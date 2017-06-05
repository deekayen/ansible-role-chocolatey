Chocolatey
==========

Install the Chocolatey package manager on Microsoft Windows.

Role Variables
--------------

The tasks look for `choco.exe` at the `chocolatey_path`. The installer Powershell script defaults to downloading from chocolatey.org; change `chocolatey_installer` if your target machine doesn't have Internet access or uses a customized install script.

    chocolatey_path: c:/ProgramData/chocolatey
    chocolatey_installer: https://chocolatey.org/install.ps1

Example Playbook
----------------

    - hosts: servers
      roles:
         - role: deekayen.chocolatey
           chocolatey_path: c:/ProgramData/chocolatey
           chocolatey_installer: https://chocolatey.org/install.ps1


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
