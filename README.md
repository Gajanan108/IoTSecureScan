# IoTSecureScan

**An automated, end-to-end IoT security assessment framework** — Firmware SAST, Firmware Composition Analysis (CVE mapping), live-device DAST, and optional internet-exposure recon (Shodan), all mapped to the **OWASP IoT Top 10** and **CWE**, with automated PDF/JSON/Excel reporting.

> Runs entirely inside a single Jupyter notebook (`ISTF.ipynb`) with **zero manual setup** — every
> dependency installs automatically, and the assessment runs out-of-the-box against a bundled,
> safe local demo target. `Run All` and you have a full report in under a minute.

## Why this project

IoT security tooling remains one of the most underinvested areas of application security — billions
of deployed devices, and comparatively few open tools that connect **firmware-level** weaknesses to
**network-level** exposure the way a real assessment would. Most public "IoT scanners" only do one
of these things (a static rules pass, or a port scan) and call it a day.

IoTSecureScan intentionally combines four disciplines that are normally separate tools into a single
reproducible pipeline, so it reflects how a real IoT penetration test is actually scoped:

| Stage | What it does | Techniques |
|---|---|---|
| 1. Firmware Acquisition | Unpacks firmware images (filesystem, if any) | `binwalk`, structure walk, raw-blob fallback |
| 2. SAST | Finds insecure code/config baked into firmware | Pattern rules, Shannon-entropy analysis, permission checks |
| 3. FCA (Firmware Composition Analysis) | Fingerprints embedded 3rd-party components & flags known CVEs | Version-string extraction + curated offline CVE map |
| 4. DAST | Probes the *running* device/service over the network | `nmap` service/version detection, protocol-specific checks, TLS/cert inspection, banner grabbing |
| 5. Internet Exposure (optional) | Checks if similar devices/components are exposed publicly | Shodan API (bring your own key) |
| 6. Risk Scoring & Reporting | Aggregates findings into a single risk posture | CVSS-like scoring, OWASP IoT Top 10 + CWE mapping, PDF/JSON/Excel export |

## Architecture

```
                       ┌────────────────────┐
                       │   CONFIG (1 cell)  │
                       └─────────┬──────────┘
                                 │
       ┌─────────────────────────┼─────────────────────────┐
       │                         │                          │
┌──────▼───────┐        ┌────────▼────────┐        ┌────────▼────────┐
│  Firmware     │        │  Local "digital  │        │  (optional)      │
│  Acquisition  │        │   twin" device   │        │  Shodan recon    │
│  (binwalk)    │        │   simulator      │        │  of fingerprints │
└──────┬────────┘        └────────┬─────────┘        └────────┬────────┘
       │                          │                            │
┌──────▼────────┐        ┌────────▼─────────┐                  │
│  SAST engine  │        │   DAST engine     │                  │
│  + FCA (CVE)  │        │  (nmap + checks)  │                  │
└──────┬────────┘        └────────┬─────────┘                  │
       │                          │                             │
       └────────────┬─────────────┴─────────────────────────────┘
                     │
             ┌───────▼────────┐
             │  Risk Engine    │  (severity scoring, OWASP/CWE rollup, charts)
             └───────┬────────┘
                     │
             ┌───────▼────────┐
             │  Report Engine  │  → report.pdf / findings.json / findings.xlsx
             └────────────────┘
```

## Key design decisions (and why they matter for a real assessment)

- **No external services required.** Earlier iterations of this tool depended on a running SonarQube
  server + API token just to do a SAST pass. That's a real assessment-tooling anti-pattern: it means
  the tool can't run standalone, can't be handed to someone else, and can't be demoed without extra
  infrastructure. This version replaces that with a self-contained, offline rule engine (pattern
  matching + entropy analysis + permission checks) mapped directly to CWE/OWASP IoT Top 10 — no server,
  no token, no internet dependency for the core SAST pass.
- **A legal, reproducible DAST target by default.** Scanning a real, third-party IoT device without
  written authorization is illegal in most jurisdictions. Rather than asking the user to point the
  scanner at "some IoT device" (which invites exactly that mistake), the notebook spins up an
  in-process **"digital-twin" device simulator** on `127.0.0.1` that mimics real, well-documented IoT
  weaknesses (an open Telnet service with a BusyBox-style banner, an HTTP admin page referencing
  default credentials, and an unauthenticated MQTT-style broker port). This gives the DAST engine
  something real and legal to find, every time, with zero setup — and the `DAST_TARGET` config value
  makes it a one-line change to point at an authorized real target instead.
- **Real firmware, not just fixtures.** In addition to a synthetic vulnerable firmware tree (`demo`
  mode, used to guarantee rich findings for demonstration), the tool also ships a real sample IoT
  firmware image (`sample` mode, from the OWASP DVID / community hardware project) so the pipeline is
  exercised against an actual binary firmware blob — including the realistic case where `binwalk` finds
  no extractable filesystem and the tool has to fall back to raw-binary string/entropy analysis.
- **CVE mapping without requiring an NVD/OSV API key.** FCA uses a small, curated, offline
  vulnerable-component database (e.g. Heartbleed for old OpenSSL, known Dropbear/BusyBox CVEs) so the
  demo works with zero network dependencies. Swapping in a live OSV.dev or NVD lookup is a small,
  clearly-marked extension point in the FCA cell.
- **Findings are a first-class, structured object**, not printed strings — every finding carries a
  module, CWE ID, OWASP IoT Top 10 category, severity, location, evidence, and remediation
  recommendation, which is what actually makes CVSS-style risk scoring and multi-format reporting
  possible.

## What you get after `Run All`

Everything lands in `iotsecurescan_output/` (configurable):

- **`IoTSecureScan_Report.pdf`** — cover page with overall risk score, severity/OWASP charts, full
  findings detail (CWE + OWASP IoT Top 10 mapping + remediation per finding), and an OWASP IoT Top 10
  reference appendix.
- **`IoTSecureScan_Findings.json`** — every finding as structured data, for feeding into a ticketing
  system, CI gate, or another tool.
- **`IoTSecureScan_Findings.xlsx`** — a findings sheet (sorted by severity) plus a summary sheet, for
  stakeholders who live in Excel.
- **`severity_chart.png` / `owasp_chart.png`** — the charts embedded in the PDF, also available
  standalone.

A pre-generated example of all of the above (from a demo run) is included in
[`sample_output/`](./sample_output) so you can see what a report looks like without running anything.

## Quickstart

1. Open `ISTF.ipynb` in **Google Colab** or **local Jupyter/VS Code**.
2. Run all cells (`Runtime > Run all` in Colab, or `Run All` in Jupyter). That's it — no API keys,
   no external servers, no manual dependency installs.
3. Read the printed executive summary, and open the generated report in `iotsecurescan_output/`.

### Running a real assessment (against firmware/devices you're authorized to test)

Edit the single `CONFIG` cell:

```python
FIRMWARE_SOURCE = "/path/to/your/firmware.bin"   # file or extracted directory
DAST_TARGET     = "192.168.1.50"                  # an IP/host you are AUTHORIZED to test
SHODAN_API_KEY  = "your_key_here"                 # optional — enables internet-exposure recon
```

## Ethics & scope

This notebook only ever scans **localhost services that it starts itself** by default, or the
**publicly available sample IoT firmware** it ships with. If you point `DAST_TARGET` at a real
device or network, **you are responsible for having explicit authorization to test it** —
unauthorized scanning of devices/networks you do not own or have written permission to test is
illegal in most jurisdictions. This tool is intended for authorized security assessments, security
research, and education.

## Roadmap / extension points

The codebase is deliberately modular so each stage is a natural place to extend:

- Swap the offline FCA database for a live [OSV.dev](https://osv.dev) or NVD API lookup.
- Add MQTT/CoAP payload-level fuzzing (not just port/protocol exposure checks) — good pairing with
  something like `PyRIT` or a custom fuzzer.
- Add firmware update-mechanism verification (checking for real signature verification, not just the
  heuristic used today).
- Add UART/JTAG hardware-interface checklist output for physical-access assessments (OWASP IoT Top 10
  I10 — Lack of Physical Hardening — is currently only lightly covered).

## Background: what is IoT, and why does this matter?

See [`Intro`](./Intro) for a primer on IoT architecture, MQTT/ZigBee, and why constrained,
massively-deployed devices make such an attractive attack surface (including real-world incidents
like the Mirai botnet).
