# Rules-Based-IDS
🛡️ Rule-Based Network Intrusion Detection System (NIDS)A lightweight, passive, rule-based Network Intrusion Detection System (NIDS) built on Kali Linux using Suricata, a custom 1,183-signature rule set, and the EveBox web dashboard. Both Suricata and EveBox are configured as systemd services so the entire monitoring pipeline starts automatically at boot without manual intervention.Designed as an affordable, transparent, and extensible security monitoring platform for laboratories, small organizations, and security training.📌 Table of ContentsArchitectureKey FeaturesCustom Rule Set BreakdownTech StackRepository StructureInstallation & SetupTesting & ValidationScreenshotsLimitations & Future WorkDisclaimerAuthor🏗️ Architecture                Network Traffic (eth0)
                          │
                          ▼
┌───────────────────────────────────────────────────┐
│             SURICATA IDS (systemd)                │
│  Packet Capture ─► Decode ─► TCP Stream ─► HTTP   │
│  Parser ─► Detection Engine (1,183 Custom Rules)  │
└─────────────────────────┬─────────────────────────┘
                          │  EVE JSON (/var/log/suricata/eve.json)
                          ▼
┌───────────────────────────────────────────────────┐
│            EVEBOX DASHBOARD (systemd)             │
│    Embedded SQLite DB ─► Search, Filter, Triage   │
└─────────────────────────┬─────────────────────────┘
                          │
                          ▼
         Web Browser (https://localhost:5636)
✨ Key Features1,183 Custom Detection Signatures: Covers both HTTP application-layer attacks (1,167 rules) and network-layer behavior (16 rules).Wide CVE Coverage: Includes 627 CVE-based exploit signatures spanning vulnerabilities from 2000 to 2026.Structured Telemetry: Outputs rich alerts and protocol metadata in Suricata's EVE JSON format.Lightweight Visualization: Powered by EveBox with an embedded SQLite backend—providing real-time alert search, filtering, and inspection without the heavy resource overhead of an ELK stack.Always-On Automation: Registered and enabled as systemd services (suricata.service and evebox.service) for automatic startup at boot.Passive IDS Mode: Monitors network traffic non-intrusively without delaying or blocking legitimate connections.📊 Custom Rule Set BreakdownAll 1,183 signatures are organized into structured Signature ID (SID) blocks for easy auditing and collision-free expansion:SID BlockCategoryRule CountShare22,xxx,xxxCVE-Based Exploit Rules (2000–2026)62753.0%50,xxx,xxx – 51,xxx,xxxGeneric Attack Patterns (SQLi, XSS, Command Injection, LFI, Path Traversal, SSRF, XXE, SSTI, Deserialization, Log4Shell/JNDI, Open Redirect, CRLF, NoSQLi)17715.0%30,000,xxxDefault-Login Attempts (150 products & consoles)15012.7%60,000,xxxProduct-Specific Web Attack Rules14812.5%12,0xx,xxxCNVD-Based Vulnerability Rules393.3%40,000,xxxReconnaissance & Information Disclosure262.2%70,000,xxxNetwork-Layer Rules (SYN/NULL/FIN/XMAS Scans, ICMP/SYN Floods, SSH/FTP/RDP Brute Force, Telnet, Nmap/Nikto, Reverse Shell, DNS Tunnelling)161.4%Total1,183100%🛠️ Tech StackComponentPurposeVersion / DetailKali LinuxHost operating system & network sensorLinux 7.0.12+kali-amd64SuricataMulti-threaded NIDS & HTTP protocol parser8.0.6EveBoxWeb-based alert inbox & dashboard0.28.0SQLiteEmbedded event storage for EveBoxBuilt-insystemdService automation & boot persistenceIncluded with Kali Linux📂 Repository Structurerule-based-network-ids/
├── config/
│   ├── suricata.yaml        # Suricata configuration (HOME_NET, interface, EVE JSON)
│   └── evebox.yaml          # EveBox configuration (SQLite backend & eve.json input)
├── rules/
│   └── local.rules          # 1,183 custom HTTP & network-layer Suricata signatures
├── systemd/
│   ├── suricata.service     # systemd unit file for Suricata
│   └── evebox.service       # systemd unit file for EveBox
├── screenshots/             # Dashboard, service status, and test alert screenshots
└── README.md
🚀 Installation & SetupPrerequisitesKali Linux (or any Debian-based Linux distribution)Suricata (sudo apt install suricata)EveBox installed on the sensor1. Clone the Repositorygit clone https://github.com/<YOUR_USERNAME>/rule-based-network-ids.git
cd rule-based-network-ids
2. Deploy Rules and Configuration Filessudo cp rules/local.rules /etc/suricata/rules/local.rules
sudo cp config/suricata.yaml /etc/suricata/suricata.yaml
sudo cp config/evebox.yaml /etc/evebox/evebox.yaml
Note: Open /etc/suricata/suricata.yaml and update HOME_NET (e.g., [192.168.35.0/24]) and the af-packet interface (e.g., eth0) to match your network environment.3. Validate the Suricata ConfigurationAlways test the configuration and custom rules before starting the service:sudo suricata -T -c /etc/suricata/suricata.yaml -v
Confirm that the output ends with Configuration provided was successfully loaded and 0 rules failed.4. Enable and Start the Servicessudo systemctl daemon-reload
sudo systemctl enable --now suricata
sudo systemctl enable --now evebox
Verify that both services are active and running:sudo systemctl status suricata evebox
5. Access the EveBox DashboardOpen your web browser and go to:https://localhost:5636/#/inbox
🧪 Testing & ValidationRun the following controlled tests from a separate machine on the monitored network to verify real-time detection:Test CaseCommandExpected SIDTCP SYN Port Scansudo nmap -sS -p 1-1000 <TARGET_IP>70000001TCP NULL Scansudo nmap -sN -p 1-1000 <TARGET_IP>70000002Reconnaissance (robots.txt)curl http://<TARGET_IP>/robots.txt40000001SQLMap User-Agentcurl -A "sqlmap/1.7" http://<TARGET_IP>/50000001XSS (<script>)curl "http://<TARGET_IP>/?q=%3Cscript%3Ealert(1)%3C/script%3E"50100001Command Injectioncurl "http://<TARGET_IP>/?cmd=;whoami"50400001Path Traversalcurl "http://<TARGET_IP>/?file=../../../../etc/passwd"50500001SSRF (Cloud Metadata)curl "http://<TARGET_IP>/?url=http://169.254.169.254/"50800001After running any test, refresh the EveBox inbox to inspect the generated alert and its underlying EVE JSON record.📸 Screenshots(Add your screenshots to the screenshots/ folder and update the filenames below)![EveBox Dashboard](screenshots/evebox_dashboard.png)
![Suricata Validation](screenshots/suricata_test.png)
⚠️ Limitations & Future WorkEncrypted Traffic (HTTPS): Content-based HTTP signatures inspect clear-text traffic only. Network-layer rules (scans, floods, brute force) remain effective regardless of encryption. Future work includes deploying behind a TLS termination point.Signature-Based Scope: Detects known attack patterns; requires periodic rule updates for newly disclosed CVEs.Passive Mode: Operates strictly as an IDS (alert-only). Inline IPS blocking can be evaluated after production tuning.Future Enhancements: SIEM/email alert forwarding, automated PCAP regression testing, and hybrid anomaly/ML detection.⚖️ DisclaimerThis project is intended strictly for defensive security monitoring, educational purposes, and authorized laboratory testing. Only capture traffic and run test simulations on networks and systems you own or have explicit permission to test.👤 AuthorParamvir SinghB.E. Computer Science & Engineering — Chandigarh University
