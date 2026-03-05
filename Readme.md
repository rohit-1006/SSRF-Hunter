# SSRF Hunter

A desktop GUI application for detecting Server-Side Request Forgery (SSRF) vulnerabilities. Built with Python and Tkinter, it combines payload generation, blind SSRF detection via built-in callback servers, cloud metadata targeting, and internal network discovery into a single tool designed for authorized penetration testing and bug bounty research.

> **For authorized use only.** Only test applications you own or have explicit written permission to assess. Unauthorized testing is illegal.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Screenshots](#screenshots)
- [Installation](#installation)
- [Usage](#usage)
- [Modules](#modules)
- [Payload Coverage](#payload-coverage)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Changelog](#changelog)
- [References](#references)
- [License](#license)

---

## Overview

SSRF Hunter Pro gives penetration testers a purpose-built environment for end-to-end SSRF testing. Rather than copy-pasting payloads manually, it automates payload generation with bypass variants, hosts its own HTTP and DNS callback servers for out-of-band detection, and provides dedicated views for targeting cloud metadata endpoints and probing internal network services — all from a single desktop application.

**What makes it different from generic testing tools:**
- Built-in OOB callback infrastructure — no Burp Collaborator or external service required
- Cloud-provider-specific metadata payloads for AWS, GCP, Azure, and more
- IP encoding variations and URL parser confusion bypasses generated automatically
- Internal service fingerprinting from SSRF responses
- Token-based tracking to correlate blind callbacks to specific payloads

---

## Features

| Module | Capability |
|--------|-----------|
| **SSRF Scanner** | Basic, blind, protocol smuggling, cloud metadata, and bypass modes |
| **Payload Generator** | IP encoding, localhost bypasses, redirect chains, DNS rebinding, Gopher/Redis |
| **Callback Server** | Built-in HTTP server (default :8888) with token-based request tracking |
| **DNS Server** | Built-in DNS server (default :5353) for DNS-only blind SSRF detection |
| **Cloud Metadata** | Pre-loaded endpoints for AWS, GCP, Azure, DigitalOcean, Alibaba, Oracle, Kubernetes |
| **Network Discovery** | Port scanning via SSRF timing, internal service payload generation |
| **Service Fingerprinting** | Identifies Redis, MySQL, Elasticsearch, Jenkins, Vault, Consul, and more from responses |
| **Results & Reports** | Export findings per session |

---

## Screenshots

> _Add screenshots of the GUI here once deployed._

---

## Installation

### Requirements

- Python 3.8 or higher
- Tkinter (included with most Python distributions)
- No additional dependencies — the tool uses the Python standard library only

### Steps

```bash
# Clone the repository
git clone https://github.com/rohit-1006/SSRF-Hunter.git
cd SSRF-Hunter

# Verify Tkinter is available
python3 -c "import tkinter; print('Tkinter OK')"

# Launch the application
python3 ssrf_hunter.py
```

### Tkinter Not Found?

On some Linux distributions Tkinter is not bundled with Python and must be installed separately:

```bash
# Debian / Ubuntu
sudo apt install python3-tk

# Fedora / RHEL
sudo dnf install python3-tkinter

# Arch Linux
sudo pacman -S tk
```

On macOS, Tkinter is included with the official Python installer from python.org. If you installed Python via Homebrew, run `brew install python-tk`.

---

## Usage

### Starting the Application

```bash
python3 ssrf_hunter.py
```

The GUI will open with six tabs across the top. A typical workflow:

**1. Set up the callback server first**
Navigate to the **Callback Server** tab, set your HTTP and DNS ports, and start both servers. Generate a tracking token and copy the callback URL — you will use this in your payloads.

**2. Configure the scanner**
In the **SSRF Scanner** tab, paste your target URL and replace the injectable parameter value with `INJECT` as a placeholder:
```
http://target.com/fetch?url=INJECT
```
Set your callback server address, choose which scan modules to enable, and click **START SCAN**.

**3. Explore cloud targets**
Use the **Cloud Metadata** tab to browse pre-loaded endpoints for your target's cloud provider, then click **Generate Bypass Payloads** to produce encoding variants ready for injection.

**4. Probe internal services**
In the **Network Discovery** tab, enter an internal host and port list, then generate service-specific payloads for Redis, MySQL, Elasticsearch, Docker, and others.

**5. Review results**
The **Results & Reports** tab collects all findings from the session for review and export.

---

## Modules

### SSRF Scanner

The core scanning tab. Accepts a target URL with `INJECT` marking the injection point. Supports five scan modes that can be enabled independently:

- **Basic SSRF Detection** — Tests direct callback payloads and response-based indicators
- **Blind SSRF (OOB Detection)** — Injects tokenized callback URLs and waits for out-of-band hits on the local HTTP or DNS server
- **Protocol Smuggling** — Tests `file://`, `gopher://`, `dict://`, `ldap://`, and `sftp://` schemes
- **Cloud Metadata Endpoints** — Submits payloads targeting provider metadata APIs across all configured cloud platforms
- **Bypass Techniques** — Applies IP encoding, URL confusion, and redirect chain wrappers to all payloads

### Payload Generator

Generates bypass-oriented payload variants from a base target:

- **IP encoding** — Decimal, hexadecimal, octal, URL-encoded, zero-padded, and IPv6-mapped forms (e.g. `2130706433`, `0x7f000001`, `0177.0.0.01` for `127.0.0.1`)
- **Localhost bypasses** — Over 20 variations including `127.1`, `0`, `::1`, `[::1]`, `localtest.me`, `spoofed.burpcollaborator.net`
- **URL parser confusion** — Credential injection (`user@callback`), fragment tricks, backslash confusion, tab/newline injection, unicode normalization
- **Redirect chains** — Payloads routed through `httpbin.org/redirect-to`
- **DNS rebinding** — Generates domains via `rbndr.us` and `1u.ms` services
- **Gopher / Redis** — Constructs raw Gopher protocol payloads for Redis command injection

### Callback Server

A self-contained HTTP server running locally that listens for incoming requests from the target application. Each generated payload embeds a unique tracking token. When the server receives a request, it records the source IP, request path, headers, and timestamp and displays them in the callback log.

Tokens follow the format `{8-char-hex}-{unix-timestamp}` and are matched against incoming requests in real time.

### DNS Callback Server

A minimal UDP DNS server that listens for DNS queries containing tracking tokens in subdomains. Useful for detecting SSRF in environments that block HTTP out-of-band but resolve DNS. Logs each query with the resolved domain and source IP.

### Cloud Metadata

Pre-loaded endpoint lists for seven cloud providers:

| Provider | Example Endpoints |
|----------|------------------|
| AWS | `/latest/meta-data/`, `/latest/meta-data/iam/security-credentials/` |
| GCP | `/computeMetadata/v1/`, `/computeMetadata/v1/instance/service-accounts/default/token` |
| Azure | `/metadata/instance?api-version=2021-02-01`, `/metadata/identity/oauth2/token` |
| DigitalOcean | `/metadata/v1/`, `/metadata/v1/id` |
| Alibaba Cloud | `http://100.100.100.200/latest/meta-data/` |
| Oracle Cloud | `/opc/v1/instance/`, `/opc/v2/instance/` |
| Kubernetes | `https://kubernetes.default.svc/`, `http://localhost:10255/pods` |

Clicking **Generate Bypass Payloads** produces encoding variants for all endpoints in the selected provider.

### Network Discovery

Generates SSRF payloads for probing internal services and performs port scanning via SSRF response timing. Includes a reference table of common internal service ports and their expected endpoints:

Redis (6379), Memcached (11211), MySQL (3306), PostgreSQL (5432), MongoDB (27017), Elasticsearch (9200), Docker API (2375), Kubernetes (10255), Consul (8500), Vault (8200), Jenkins (8080), Apache Solr (8983).

Fingerprints discovered services by matching response content against known banners and error strings.

---

## Payload Coverage

### Protocol Schemes

| Protocol | Example |
|----------|---------|
| HTTP/HTTPS | `http://169.254.169.254/latest/meta-data/` |
| File | `file:///etc/passwd`, `file:///proc/self/environ` |
| Gopher | `gopher://127.0.0.1:6379/_INFO` |
| Dict | `dict://127.0.0.1:6379/INFO` |
| LDAP | `ldap://127.0.0.1:389/` |
| SFTP | `sftp://attacker.com/` |
| TFTP | `tftp://attacker.com/file` |

### Internal Files (via `file://`)

`/etc/passwd`, `/etc/shadow`, `/etc/hosts`, `/proc/self/environ`, `/proc/self/cmdline`, `/proc/net/tcp`, `/proc/net/fib_trie`, `C:\Windows\win.ini`, `C:\Windows\System32\drivers\etc\hosts`

---

## Project Structure

```
SSRF-Hunter/
├── ssrf_hunter.py          # Full application (single-file)
│   ├── HackerTheme         # Dark UI theme configuration
│   ├── PayloadGenerator    # Payload generation for all bypass types
│   ├── CallbackTracker     # Token management and hit recording
│   ├── CallbackHTTPHandler # HTTP request handler for callback server
│   ├── CallbackServer      # HTTP callback server (default :8888)
│   ├── DNSCallbackServer   # DNS callback server (default :5353)
│   ├── SSRFScanner         # Core scanning logic and port probing
│   └── SSRFHunterGUI       # Tkinter GUI and tab layout
├── README.md
└── LICENSE
```

---

## Troubleshooting

**Application does not open / no window appears**
Confirm Tkinter is installed and working:
```bash
python3 -c "import tkinter; tkinter.Tk().mainloop()"
```
If this fails, install Tkinter for your distribution (see [Installation](#installation)).

**Callback server fails to start**
The default ports (8888 for HTTP, 5353 for DNS) may be in use or require elevated privileges (ports below 1024 on Linux). Change the port in the Callback Server tab, or run with `sudo` for ports below 1024.

**No callbacks received from the target**
- Confirm the callback server is running (green status indicator in the Callback Server tab)
- Ensure your machine is reachable from the target server — callbacks will not work if the target cannot route to your IP
- For external targets, consider port-forwarding or using a VPS as the callback host

**DNS callbacks not triggering**
DNS port 53 requires root on Linux. Either run as root or use an alternative port (5353 is the default) and configure the target payload subdomain accordingly.

**Payload injection not working**
Make sure the target URL has `INJECT` as a placeholder in the exact position where the URL parameter value should be substituted:
```
http://target.com/proxy?resource=INJECT    ✓
http://target.com/proxy?resource=          ✗
```

---

## Contributing

Contributions are welcome. To propose a change:

1. Fork the repository and create a feature branch: `git checkout -b feature/your-feature`
2. Keep changes focused — one feature or fix per pull request
3. Test against a known-vulnerable lab (DVWA, Juice Shop, or a custom app) before submitting
4. Open a pull request against `main` with a clear description of what was changed and why

For significant new modules, open an issue for discussion before writing code.

### Roadmap

- [ ] Multi-URL batch scanning from a file
- [ ] Header and POST body injection modes (not just URL parameters)
- [ ] Automated report export (HTML / JSON)
- [ ] SSRF-to-RCE chaining via Gopher/Redis
- [ ] Proxy support (route scan traffic through Burp Suite)
- [ ] Custom payload file import

### Bug Reports

Open a GitHub issue with:
- Python version and OS
- A description of the expected vs actual behavior
- Any error messages from the terminal

---

## Changelog

### v1.0.0

- Initial release
- Full Tkinter GUI with dark hacker theme
- SSRF Scanner with 5 configurable scan modes
- Payload Generator: IP encoding, localhost bypasses, URL parser confusion, redirect chains, DNS rebinding, Gopher/Redis
- Built-in HTTP callback server with token-based hit tracking
- Built-in DNS callback server for DNS-only blind SSRF
- Cloud metadata endpoints for AWS, GCP, Azure, DigitalOcean, Alibaba, Oracle, and Kubernetes
- Internal network discovery and port scanning via SSRF timing
- Internal service fingerprinting (Redis, MySQL, Elasticsearch, Jenkins, Vault, and more)

---

## References

| Resource | URL |
|----------|-----|
| OWASP SSRF Guide | https://owasp.org/www-community/attacks/Server_Side_Request_Forgery |
| PortSwigger SSRF Labs | https://portswigger.net/web-security/ssrf |
| HackTricks SSRF | https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery |
| CWE-918: SSRF | https://cwe.mitre.org/data/definitions/918.html |
| PayloadsAllTheThings SSRF | https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery |
| AWS IMDSv2 Documentation | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html |

---

## License

This project is licensed under the [MIT License](LICENSE).
