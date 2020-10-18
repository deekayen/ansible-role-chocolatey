Chocolatey
==========

[![Build Status](https://travis-ci.org/deekayen/ansible-role-chocolatey.svg?branch=main)](https://travis-ci.org/deekayen/ansible-role-chocolatey) [![Project Status: Unsupported – The project has reached a stable, usable state but the author(s) have ceased all work on it. A new maintainer may be desired.](https://www.repostatus.org/badges/latest/unsupported.svg)](https://www.repostatus.org/#unsupported) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue) ![Windows platform](https://img.shields.io/badge/platform-windows-lightgrey)

Install the Chocolatey package manager on Microsoft Windows.

If Chocolatey is missing from the system, the win_chocolatey module is supposed to install it. This role may be helpful to ensure Chocolatey is available on a system, even if you don't run win_chocolatey tasks against it. See http://docs.ansible.com/ansible/win_chocolatey_module.html.

Role Variables
--------------

The tasks look for `choco.exe` at the `chocolatey_path`. The installer Powershell script defaults to downloading from chocolatey.org; change `chocolatey_installer` if your target machine doesn't have Internet access or uses a customized install script.

    chocolatey_installer: https://chocolatey.org/install.ps1
    chocolatey_path: c:/ProgramData/chocolatey
    chocolatey_version: latest
    chocolatey_windows_compression: "false"

Example Playbook
----------------

    - hosts: win_servers
      roles:
         - role: deekayen.chocolatey
           chocolatey_installer: https://chocolatey.org/install.ps1
           chocolatey_path: c:/ProgramData/chocolatey

Requirements
------------

The user running this role needs administrator access.

* Windows 7+ / Windows Server 2008+
* PowerShell v3+
* .NET Framework 4+ (the installation will attempt to install .NET 4.0 if you do not have it installed)

Dependencies
------------

None.

License
-------

BSD
