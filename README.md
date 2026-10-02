# deekayen.chocolatey

[![CI](https://github.com/deekayen/ansible-role-chocolatey/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-role-chocolatey/actions/workflows/ci.yml) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-deekayen.chocolatey-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/deekayen/chocolatey/) [![Project Status: Unsupported – The project has reached a stable, usable state but the author(s) have ceased all work on it. A new maintainer may be desired.](https://www.repostatus.org/badges/latest/unsupported.svg)](https://www.repostatus.org/#unsupported) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue) ![Windows platform](https://img.shields.io/badge/platform-windows-lightgrey)

An Ansible role that installs the Chocolatey package manager on a Windows host when `choco.exe` is missing, then adds the Chocolatey `bin` directory to the machine `PATH`.

The role checks for `choco.exe` under `chocolatey_path` with `ansible.windows.win_stat`. If it is absent, a `raw` PowerShell command forces TLS 1.2, sets `$env:chocolateyUseWindowsCompression` (and `$env:chocolateyVersion` for a pinned version), then downloads and runs the install script at `chocolatey_installer`. The last task adds `%ALLUSERSPROFILE%\chocolatey\bin` to the machine `PATH` with `ansible.windows.win_path`.

## Requirements

- ansible-core 2.15 or newer on the controller.
- The `ansible.windows` collection: `ansible-galaxy collection install ansible.windows`.
- A WinRM or SSH connection to the target with administrative rights. The [Chocolatey install docs](https://docs.chocolatey.org/en-us/choco/setup/) require an administrative shell.
- Outbound HTTPS from the target to `chocolatey_installer` and to wherever that script downloads the Chocolatey package. The default URL redirects to `community.chocolatey.org`.
- .NET Framework 4.8 on the target for Chocolatey CLI v2.0 and later. As of October 2026, the [Chocolatey install docs](https://docs.chocolatey.org/en-us/choco/setup/) list it as a requirement and say the install script attempts to install it if missing.

## Supported platforms

| Platform | Versions |
| --- | --- |
| Windows | 2016, 2019, 2022 |

CI lints the role and runs `ansible-playbook --syntax-check`; it does not apply the role to a Windows host.

## Installation

From Ansible Galaxy:

```bash
ansible-galaxy role install deekayen.chocolatey
ansible-galaxy collection install ansible.windows
```

Or pin it in `requirements.yml`:

```yaml
---
roles:
  - name: deekayen.chocolatey
    src: https://github.com/deekayen/ansible-role-chocolatey.git
    scm: git
    version: main

collections:
  - name: ansible.windows
```

```bash
ansible-galaxy install -r requirements.yml
```

## Role variables

`meta/argument_specs.yml` declares every variable below as a string, so Ansible validates types and choices before the tasks run.

| Variable | Default | Description |
| --- | --- | --- |
| `chocolatey_installer` | `https://chocolatey.org/install.ps1` | URL of the install PowerShell script. Must start with `http://` or `https://`. Point it at an internal mirror when the host has no Internet access. |
| `chocolatey_path` | `c:/ProgramData/chocolatey` | Directory where the role looks for `choco.exe` to decide whether Chocolatey is installed. Must not be empty. See [Known issues](#known-issues). |
| `chocolatey_version` | `latest` | `latest` installs whatever version the script provides; any other value is exported as `$env:chocolateyVersion` before the script runs. Must not be empty. |
| `chocolatey_windows_compression` | `"false"` | Exported as `$env:chocolateyUseWindowsCompression`. Must be the string `"true"` or `"false"`, since it is interpolated into PowerShell. |

As of October 2026, the [Chocolatey install docs](https://docs.chocolatey.org/en-us/choco/setup/) say `chocolateyVersion` and `chocolateyUseWindowsCompression` only take effect with install methods that call `https://community.chocolatey.org/install.ps1`. A customized script at `chocolatey_installer` may ignore both.

## Behavior

- The install tasks run only when `choco.exe` is missing. A host that already has Chocolatey is left at its current version, so changing `chocolatey_version` later does not upgrade or downgrade it.
- An install run always reports `changed`, since the `raw` task sets `changed_when: true`.
- The `PATH` task runs on every play, whether or not the role installed anything.

## Dependencies

None. The `ansible.windows` collection is a requirement, not a role dependency.

## Example playbook

Fetch the install script from an internal mirror:

```yaml
---
- name: Install Chocolatey on Windows build agents.
  hosts: windows_build_agents

  vars:
    chocolatey_installer: https://chocolatey-mirror.example.internal/install.ps1

  roles:
    - deekayen.chocolatey
```

`chocolatey-mirror.example.internal` is a placeholder for an internal mirror.

## Known issues

- `chocolatey_path` only controls the `win_stat` check in `tasks/main.yml:8`. The role does not set `$env:ChocolateyInstall`, which the [Chocolatey install docs](https://docs.chocolatey.org/en-us/choco/setup/) say decides the install location, so the script installs to the default `C:\ProgramData\chocolatey` unless `ChocolateyInstall` is already set on the host. With a non-default `chocolatey_path` and no matching `ChocolateyInstall`, the role does not find `choco.exe` after the install, so the install task runs on every play.
- `tasks/main.yml:48` hard-codes `%ALLUSERSPROFILE%\chocolatey\bin` for the `PATH` entry instead of deriving it from `chocolatey_path`.

## Development

CI runs on every push to `main` and every pull request (see `.github/workflows/ci.yml`). It installs `ansible.windows`, runs `ansible-lint --profile production`, and syntax-checks `tests/test.yml`, which applies the role by its relative path. To run the same checks locally:

```bash
pip3 install ansible-lint
ansible-galaxy collection install ansible.windows
ansible-lint --profile production
ansible-playbook --syntax-check tests/test.yml -i tests/inventory
```

The repository also has a `.pre-commit-config.yaml`; run `pre-commit run --all-files` before pushing.

### Repository layout

| Path | Purpose |
| --- | --- |
| `tasks/main.yml` | Input validation, `choco.exe` check, install script run, and `PATH` update. |
| `tasks/assert.yml` | Input validation, tagged `always`. |
| `defaults/main.yml` | Every user-facing variable. |
| `meta/argument_specs.yml` | Type and choice validation for the variables. |
| `tests/` | Syntax-check playbook and inventory used by CI. |

## Releases

Pushing a git tag runs `.github/workflows/release.yml`, which imports the tagged commit into Ansible Galaxy as `deekayen.chocolatey`. The import needs a `GALAXY_API_KEY` repository or organization secret.

## License

BSD 3-Clause. See [LICENSE](LICENSE).

## Author

[David Norman](https://github.com/deekayen). Sponsorship links are in [.github/FUNDING.yml](.github/FUNDING.yml).
