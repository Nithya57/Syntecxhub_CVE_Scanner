[README.md](https://github.com/user-attachments/files/30054723/README.md)
# Vulnerability / CVE Scanner

Educational scanner for the Syntecxhub Cybersecurity Internship — **Task 4, Project 1**.

## What it does

1. **Banner grabbing / service detection** — connects to given ports on a
   host and grabs whatever identifying banner the service offers (SSH,
   FTP, HTTP `Server` header, etc.).
2. **Product/version parsing** — extracts a product name + version from
   the banner (e.g. `OpenSSH 8.2p1`, `nginx 1.18.0`).
3. **CVE lookup** — queries the free public **NVD (National Vulnerability
   Database) API** for CVEs matching that product/version.
4. **Reporting** — writes a JSON + text report listing open ports,
   banners, and any possible CVE matches with severity/CVSS score.

## Ethical scope

By default the scanner only targets `localhost` / private-network hosts
(127.0.0.1, 192.168.x.x, 10.x.x.x, 172.16-31.x.x), matching the brief's
"only test on allowed targets" rule. Scanning anything else needs the
explicit `--i-own-this-target` flag — an acknowledgement you have
permission. Port-scanning systems you don't own/have authorization for
is illegal in most places.

## Setup

```bash
pip install requests
```

No API key is required for light NVD usage — the script rate-limits
itself to stay under NVD's public (no-key) limit of ~5 requests/30s.

## Usage

```bash
# Scan your own machine's common ports, with CVE lookups
python3 cve_scanner.py --host 127.0.0.1 --ports 21,22,80,443,3306,8080

# Scan a port range
python3 cve_scanner.py --host 127.0.0.1 --ports 1-1024

# Banner-grab only, skip CVE lookups (no internet needed, fast)
python3 cve_scanner.py --host 127.0.0.1 --ports 22,80 --no-cve-lookup

# A host you own on your network (requires the flag)
python3 cve_scanner.py --host 192.168.1.10 --ports 22,80,443 --i-own-this-target
```

## Testing it locally (no real vulnerable service needed)

You can fake a banner to see the full pipeline work end-to-end:

```python
# fake_service.py — run this in one terminal
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
s.bind(('127.0.0.1', 2222))
s.listen(5)
while True:
    conn, addr = s.accept()
    conn.sendall(b'SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.5\r\n')
    conn.close()
```

```bash
python3 fake_service.py        # terminal 1
python3 cve_scanner.py --host 127.0.0.1 --ports 2222   # terminal 2
```

This should detect the (deliberately outdated) OpenSSH banner and — if
CVE lookups are enabled — return real CVEs associated with that version.

## Output

- `cve_report.json` — structured results
- `cve_report.txt` — readable summary per open port
- `cve_report.log` — full run log

## Responsible disclosure & triage basics

- **Don't publish exploit details publicly** before the vendor/system
  owner has had a reasonable chance to fix a real finding.
- **Report privately first** — a `security@` contact or a bug bounty
  program, if one exists. Include steps to reproduce and impact.
- **Give a reasonable disclosure timeline** — 90 days is a common
  industry default before public disclosure.
- **Stay within scope** — don't access, modify, or exfiltrate data
  beyond what's needed to prove the issue exists.
- **Triage by severity** — CVSS score bands roughly: 9.0–10.0 Critical,
  7.0–8.9 High, 4.0–6.9 Medium, 0.1–3.9 Low. Prioritize fixes accordingly.

## Disclaimer

For authorized security testing and educational use only. Only scan
systems you own or have explicit written permission to test.
