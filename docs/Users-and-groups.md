# Users and Groups

## Users

Each member of BU-ISCIII is assigned unique credentials to access various infrastructure resources. Here are the primary user accounts you will encounter:

- **ISCIII Domain User:** This is your institutional account, which includes your username and password. It allows access to any Windows domain-connected PC, email, and other resources like shared folders on Neptuno, SFTP, etc.
- **Workstation User:** This is your local Linux account for accessing workstations. We aim to match this username with your institutional one, though they may differ, and the password will likely vary.
- **HPC User:** This local account is for accessing the High-Performance Computing (HPC) system. It typically uses the same username as your institutional account, though the password may differ. To simplify, we suggest using the same password for both the workstation and the HPC to avoid the need to remember multiple passwords.

## Groups

We have some groups we use for managing shared folders, permissions, in HPC and/or workstations.

- bi / bioinfo: all bu-isciii members are included in this group for which we have
- bioinfoaccess: this is kind of deprecated but it is used for temporary bu-isciii members that may only have access to some folders.
- sudo: group for granting sudo users (just in workstations).
- sshaccess: group for granting access for ssh to the workstations (just in workstations).

## Permissions

We need to make sure everyone has access to all the shared resources that is why we use the next permissions set:

- files -> 664 (rw-rw-r)
- folders -> 775 (rwxrwxr-x)

If you want to fix permissions in a folder, you can use:

```Bash
chown -R user_name:bi path-folder/
find path-folder/ -type d -exec chmod 775 {} \;
find path-folder/ -type d -exec chmod 664 {} \;
```
