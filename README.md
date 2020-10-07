sapcar
======

SAPCAR is a compress utility (similar to winzip, tar, etc.), that is used by SAP to compress
and decompress delivered files. SAP Kernel Programs. SAPCAR in general. The compressed files
have the extension ".CAR" or ".SAR".

This role installs `sapcar.exe` on Windows platforms, by downloading installation file from
a local url.

Requirements
------------

No requirement.

Role Variables
--------------

Available variables are listed below, along with default values:

    sapcar_url: http://repo/install_files/sapcar_721_win_x64/SAPCAR_1320-80000938.EXE
    sapcar_url_username: null
    sapcar_url_password: null

URL to download SAPCAR installation file. Username and password are optional variables.

    sapcar_executable_dest: C:\Windows\sapcar.exe

Destination for the executable. This should not be changed. If it's really required,
then use a location included in windows environement variable `PATH`.

Dependencies
------------

No dependency.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: sapservers
      vars:
        sapcar_url: http://local_apache/install_files/sapcar_721_win_x64/SAPCAR_1320-80000938.EXE
      roles:
        - jpmat296.sapcar

License
-------

BSD

Author Information
------------------

This role was created in 2020 by [Jean-Pierre Matsumoto](https://fr.linkedin.com/in/jpmatsumoto).
