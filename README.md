# LUDUS Templates for project-construct (German / de-DE)

These directories are self-contained Ludus templates, adapted for the **German language** and a **German keyboard layout** (`de-DE` / `xkb de`, timezone `Europe/Berlin` / `W. Europe Standard Time`).

For more information see: https://docs.ludus.cloud/docs/templates

This is a fork of the great [Croko-fr](https://github.com/Croko-fr) Ludus template repository. The Windows and Kali packer/build logic tracks the upstream [badsectorlabs/ludus](https://github.com/badsectorlabs/ludus) implementation. The main goal of this fork is to localize the templates for German-speaking ranges and to further develop them.

Each template lives in its own directory under `templates/<vm-identifier>/` and is fully self-contained (packer config, unattended install answer file, provisioning scripts and ansible playbooks).

## Quick Start

```bash
# Add this source to your Ludus server via an interactive installer
ludus source add https://github.com/BLGHT/project-construct-templates.git

# Or, script the install of source resources
ludus source add https://github.com/BLGHT/project-construct-templates.git --all

# Build source templates for your ranges
ludus templates build
```

## Manual template installation steps

1. Clone this repository

`git clone https://github.com/BLGHT/project-construct-templates.git`

2. Enter the directory

`cd project-construct-templates`

3. Add the templates from a directory

```bash
ludus templates add -d templates/debian-12-13-x64-de-server
ludus templates add -d templates/win2022-server-x64-de
# Or add every template at once:
for filename in templates/*/; do ludus templates add -d "$filename"; done
```

4. Show the templates list

```bash
ludus templates list
```

5. Build a template

```bash
ludus templates build -n debian-12-13-x64-de-server-template
[INFO]  Template building started - this will take a while. Building 1 template(s) at a time.
```

6. Follow the build logs

```bash
ludus templates logs -f
```

## Localization notes

Every template is configured for German:

- **Linux** (all `debian-*`, `kali-*`, `ubuntu-*`): locale `de_DE.UTF-8`, keymap `de`, timezone `Europe/Berlin`, set via the preseed / cloud-init autoinstall answer files and the packer `boot_command`.
- **Windows** (all `win*`): `de-DE` `InputLocale`/`SystemLocale`/`UILanguage`/`UserLocale`, German evaluation ISOs (`_de-de.iso` / `DE-DE.ISO`), timezone `W. Europe Standard Time`, and the localized `Administratoren` group name in `Autounattend.xml`.

Default credentials match the upstream templates:

| OS family | Username | Password |
|:---:|:---:|:---:|
| Debian | `debian` | `debian` |
| Kali | `kali` | `kali` |
| Ubuntu | `localuser` | `password` |
| Windows | `localuser` | `password` |

## Template coverage

### Linux

| Directory | Distribution | Version | ISO | Language |
|:---|:---:|:---:|:---:|:---:|
| `debian-11-11-x64-de-server` | Debian | 11.11 | netinst | de |
| `debian-12-13-x64-de-server` | Debian | 12.13 | netinst | de |
| `kali-2026-1-x64-de-desktop` | Kali | 2026.1 | installer-netinst | de |
| `ubuntu-20.04.1-x64-de-server` | Ubuntu Server | 20.04.1 | legacy-server | de |
| `ubuntu-22.04.5-x64-de-server` | Ubuntu Server | 22.04.5 | live-server | de |
| `ubuntu-24.04.2-x64-de-server` | Ubuntu Server | 24.04.2 | live-server | de |

### Windows

| Directory | Distribution | Version | Firmware | Language | Info |
|:---|:---:|:---:|:---:|:---:|:---:|
| `win10-21h2-x64-de-enterprise` | Windows 10 | 21H2 Enterprise Eval | BIOS | de | |
| `win10-22h2-x64-de-enterprise` | Windows 10 | 22H2 Enterprise Eval | BIOS | de | |
| `win11-22h2-x64-de-enterprise` | Windows 11 | 22H2 Enterprise Eval | UEFI (OVMF) | de | TPM/SecureBoot bypass |
| `win11-23h2-x64-de-enterprise` | Windows 11 | 23H2 Enterprise Eval | UEFI (OVMF) | de | TPM/SecureBoot bypass |
| `win2016-server-x64-de` | Windows Server | 2016 Eval | BIOS | de | |
| `win2019-server-x64-de` | Windows Server | 2019 Eval | BIOS | de | |
| `win2019-server-x64-de-no-security-updates` | Windows Server | 2019 Eval (17763.737) | BIOS | de | No security updates |
| `win2022-server-x64-de` | Windows Server | 2022 Eval | BIOS | de | |

## Unordinary devices

Templates that emulate non-PC network devices instead of a plain OS. Each one is a normal Linux image with extra services layered on top, so IP/VLAN placement is left to the range config like any other template.

### Available

- **`debian-12-13-x64-de-printer`** — Debian 12 that emulates an HP LaserJet network printer. Runs miniprint (raw/PJL on 9100), CUPS (IPP on 631, LPD on 515), snmpd, and a static nginx status page. Details in [Emulated printer](#emulated-printer-debian-12-13-x64-de-printer) below.

### Further development

Not implemented yet:

- **IP camera** — Debian with a fake RTSP stream (e.g. [mediamtx](https://github.com/bluenviron/mediamtx)) and a web login page. Stands in for an IoT camera.
- **ICS/PLC (conpot)** — Debian running [conpot](https://github.com/mushorg/conpot), which speaks Modbus (502), S7comm, and SNMP. Stands in for an industrial controller.

## Emulated printer (`debian-12-13-x64-de-printer`)

A Debian 12 VM that presents itself as an **HP LaserJet** network printer, built as an attack/enumeration target for a range. It is provisioned by `ansible/printer-setup.yml` and exposes a coherent printer identity across the usual printer services:

| Service | Port | Engine | Purpose |
|:---|:---:|:---|:---|
| Raw / JetDirect (PJL, PostScript) | 9100/tcp | [`miniprint`](https://github.com/sa7mon/miniprint) systemd service | Core target — PRET connects here; emulates a PJL virtual filesystem |
| IPP + web admin | 631/tcp | CUPS with a shared virtual PDF queue | Real print server + admin UI; accepts jobs |
| LPD | 515/tcp | CUPS `cups-lpd` (systemd socket) | Legacy print protocol / PRET LPD transport |
| SNMP | 161/udp | `snmpd` with a printer `sysDescr`/identity | Device fingerprinting and discovery |
| Fake control panel | 80/tcp | nginx static status/login page | HTTP recon realism (`http-title`) |

Default credentials are the Debian base `debian` / `debian`. All services bind to the range network on purpose — this template is meant for isolated lab use only.

**Attacking it from Kali** (PRET is the canonical tool):

```bash
git clone https://github.com/RUB-NDS/PRET
python3 PRET/pret.py <printer-ip> pjl     # PJL over port 9100 (miniprint)
python3 PRET/pret.py <printer-ip> ps      # PostScript
```

`nmap -p 9100,515,631,80,161 -sU -sV <printer-ip>` will fingerprint the printer services; the CUPS admin UI is reachable at `https://<printer-ip>:631/`.

### Registering the printer in Active Directory

The printer device itself does **not** join the domain — network printers never do. It appears in AD only when a **domain-joined Windows print server** shares it and publishes it to the directory. Do this on a Windows print-server VM in the range (not on this template):

1. **Add the printer:** *Settings → Printers → Add a printer → Add a local printer with manual settings → Create a new port → Standard TCP/IP Port*, and point it at this VM's IP. Windows probes the device over **SNMP** during this step — because `snmpd` is running, it is detected as a real device and a proper port is created (RAW port 9100, or select LPR for 515). Install any PostScript/PCL driver (e.g. a generic "HP LaserJet PS" driver).
2. **Share it:** open the printer's *Properties → Sharing*, tick **Share this printer**, and tick **List in the directory**. This publishes a `printQueue` object into Active Directory.
3. **Find / deploy it:** users can now locate it via *Add Printer → Search the directory*, and admins can push it with Group Policy (*Print Management → Deploy with Group Policy*).

This is exactly how printers are managed in real AD environments, which is why the template only needs to speak the print protocols and answer SNMP — the AD object lives on the print server.

## Directory layout

```
templates/<vm-identifier>/
├── <vm-identifier>.pkr.hcl     # packer proxmox-iso source + build
├── http/                       # preseed.cfg / cloud-init user-data (Linux)
├── Autounattend.xml            # unattended answer file (Windows)
├── iso/                        # first-boot provisioning scripts (Windows)
├── scripts/                    # windows-shell / powershell provisioners (Windows)
└── ansible/                    # provisioning playbooks
```

## ISO checksums

The Windows evaluation ISOs are not published with official Microsoft SHA256 hashes. The checksums pinned in the `win*` `*.pkr.hcl` files are community-documented (via [files.rg-adguard.net](https://files.rg-adguard.net)) and were cross-checked against the live download `Content-Length`. If Microsoft rotates an evaluation ISO, update both `iso_url` and `iso_checksum` in the affected template.
