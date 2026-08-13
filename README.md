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
