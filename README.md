# Hospital Honeypot — Community Edition releases

Download the free Hospital Honeypot Community Edition installers from the
[Releases](../../releases) page. This repository hosts release binaries only.

A deception system that simulates three networked hospital devices — a kidney dialysis machine,
an MRI scanner and a bedside cardiac monitor — to capture and classify attacker behavior against
healthcare infrastructure. All patient and session data is synthetic and generated at runtime; no
real PHI is ever involved.

**Latest release: v1.1.0** (Linux x86-64 and ARM64 / Raspberry Pi).

| Version | What changed |
| --- | --- |
| **v1.1.0** | Adds anonymous community reporting to the ThreatSpire Deception Grid (see below), on by default, with automatic retry and a resumable progress cursor so events are not dropped. Opt out with `HH_COMMUNITY_REPORTING=false`. |
| v1.0.0 | Initial release: dialysis, MRI (DICOM) and cardiac (HL7 v2 / MLLP) sensors, the admin API and web UI, and optional push to your own ThreatSpire tenant. |

## Sensors

| Sensor | Protocol | Env var (port) | Default |
| --- | --- | --- | --- |
| dialysis | proprietary TCP | `HH_DIALYSIS_PORT` | 9101 |
| mri | DICOM | `HH_MRI_PORT` | 11112 |
| cardiac | HL7 v2 / MLLP | `HH_CARDIAC_PORT` | 2575 |

Admin API and web UI: `HH_ADMIN_ADDR` (default `127.0.0.1:8080`). The commercial Hospital Honeypot
adds five more device sensors (medication dispensing cabinet, point-of-care analyzer, infusion
pump, anesthesia machine, nurse call system) and a ThreatSpire console connection.

## Install

| Platform | Asset |
| --- | --- |
| Linux / AMD64 | `hospital-honeypot-community-<version>-linux-amd64.zip` |
| Raspberry Pi / Linux ARM64 | `hospital-honeypot-community-<version>-linux-arm64.zip` |

```bash
unzip hospital-honeypot-community-<version>-linux-<arch>.zip
cd hospital-honeypot-community-<version>-linux-<arch>
sudo ./install.sh
sudo systemctl enable --now hospital-honeypot
```

The installer creates an unprivileged service account and never starts the service by itself:
opening listeners that attract attacker traffic is an operator decision. Verify your download with
the release's `SHA256SUMS.txt`:

```bash
shasum -a 256 -c hospital-honeypot-community-<version>-SHA256SUMS.txt
```

**Upgrading:** unpack the new bundle and run `sudo ./install.sh` again. It replaces the binary and
service unit and keeps your config (new defaults are written next to it as
`/etc/hospital-honeypot/hospital-honeypot.env.new`). Then `sudo systemctl restart hospital-honeypot`.

## Router / firewall port forwarding

To be reachable by attackers on the internet, forward the sensor ports to the host:

| Port (default) | Sensor | Forward it? |
| --- | --- | --- |
| `9101/tcp` | dialysis | Yes |
| `11112/tcp` | mri | Yes |
| `2575/tcp` | cardiac | Yes |
| `8080/tcp` | admin API + web UI | **No** |

**Never forward the admin port.** It has no authentication and is meant to be reached only from
`localhost` or over a private connection, e.g. an SSH tunnel
(`ssh -L 8080:127.0.0.1:8080 user@host`). If you changed a default with the `HH_*_PORT` variables,
forward the values you set.

## Community reporting (on by default, v1.1.0 and later)

The honeypot anonymously reports captured attacker events to the ThreatSpire community endpoint
(`https://console.threatspire.com`). It helps the ThreatSpire team study how attackers probe
healthcare devices and improves the community's threat data.

**Sent per event:** the sensor name (`dialysis`, `mri`, `cardiac`), event type, timestamps, the
**attacker's** source IP, MITRE ATT&CK technique IDs, and up to 4 KB of the attacker's own payload.
**Not sent:** no hostname, no operator label, no token, nothing from your network. The server sees
only the public IP the report arrives from (to count nodes by country); it is never stored with your
events.

**Turn it off:** set `HH_COMMUNITY_REPORTING=false` in `/etc/hospital-honeypot/hospital-honeypot.env`
and restart the service. Failed reports retry and never drop events.

| Env var | Purpose | Default |
| --- | --- | --- |
| `HH_COMMUNITY_REPORTING` | `false` turns community reporting off | `true` |
| `HH_COMMUNITY_PUSH_INTERVAL_SECONDS` | how often to report new events | `30` |

## Optional: push to your own ThreatSpire tenant

Off by default; there is no baked-in destination. Set these in the env file or use the web console's
"ThreatSpire Push" tab:

| Env var | Purpose | Default |
| --- | --- | --- |
| `HH_THREATSPIRE_ENABLED` | turn the feature on | `false` |
| `HH_THREATSPIRE_BASE_URL` | your ThreatSpire tenant's base URL | (none) |
| `HH_THREATSPIRE_CONTROL_TOKEN` | the deployment token from your tenant's admin UI | (none) |
| `HH_THREATSPIRE_SENSOR_ID` | a stable label for this installation | `hospital-honeypot` |
| `HH_THREATSPIRE_PUSH_INTERVAL_SECONDS` | how often to poll for new events | `30` |

## License

The Software is free to use and may be redistributed unmodified under the terms in
[LICENSE](LICENSE). The source code is proprietary. More at [threatspire.com](https://www.threatspire.com).
