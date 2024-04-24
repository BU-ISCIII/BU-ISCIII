# 1. Connect to the SFTP manager

The SFTP manager is in this URL: [https://sftpbioinfo.isciii.es:6443/admin/sign-in](https://sftpbioinfo.isciii.es:6443/admin/sign-in)

> [!WARNING]
> This website will be recognized as Not secure.

Sign in credentials:

- Username: bioinfoadm
- Password: YKWIM
- Virtual site: bioinfo

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/e1511203-3315-4434-8c9e-3a97a3f28aa8)

# 2. Create a virtual folder in the SFTP manager

1. Create the folder in `/data/bi/sftp/` with the command line `mkdir NewFolder`.

> [!NOTE]
> The criteria to name the folder is the laboratory of the scientist.

2. Once in the SFTP manager, select `VFS` in the left panel
3. Select `Add VFS` button

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/4d61ae40-c4db-4e76-8703-a425b45bb26a)

4. Fill-in the gaps as follow (only showing the fields you have to change, leave the others as defaults):
    - Name VFS: Name of the folder you want to create (e.g. NewFolder)
    - Type: `Disk`
    - Full filesystem path: `H:\hpc-bioinfo\NewFolder`

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/bb679852-812c-4023-bc47-5f88bfb161d6)

5. Select `➕ Add Disk`

> [!WARNING]
> Now remember to add the folder to the user

# 3. Create a new SFTP user

1. Once in the SFTP manager, select `Users` in the left panel
2. Select `Add user` button

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/47f27a00-494f-459c-bb96-48aec4dd1286)

3. Fill-in `Account information` gaps as follow (only showing the fields you have to change, leave the others as defaults):
    - Name: ISCIII's e-mail name (without @isciii.es)
    - EMail and Description: :warning: **OPTIONALS**
    - User type: Two options:
        - Normal user: External people, allows to set new password. Use some password generator, with 16 characters.
        - LDAP user: For ISCIII users, uses their ISCIII password:
            - LDAP Server ID: ldap1.isciii.es
            - LDAP Query: sAMAcct
    - User status: Enabled
    - **Next**

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/749a384e-939e-46a6-a81b-d4940ad1207a)

4. Fill-in `Home foolder and permisssions` gaps as follow (only showing the fields you have to change, leave the others as defaults):
    - Subsystems: SFTP
    - Home VFS: `InfoSFTP`
    - Directory permisssions: List
    - File permisssions: Get

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/4c63983f-bdb2-4619-82dc-f67ff8bde662)

5. **Add user**

# 4. Add existing Virtual folder to existing SFTP user

1. Once in the SFTP manager, select `Users` in the left panel
2. Select the user to whom you want to add the VFS

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/aa3b9149-deaa-4400-9d90-ab232651b1e7)

3. Inside User's settings go to `Virutal folders`
4. Select `➕ Add virtual folder`

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/3cdd2063-deb1-43b2-b853-66844fea2264)

5. Fill-in `Adding a New Virtual Folder` gaps as follow (only showing the fields you have to change, leave the others as defaults):
    - Mount point: Back slash followed by the folder's name (e.g. /Blobo_tmp) ⚠️ It's important to include the back slash (/) before the folder's name.
    - VFS: Select folder from list (e.g. Blobo_tmp)
    - Directory permisssions: List
    - File permisssions: Get

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/25959991/2cb2d872-c26e-4c17-8f59-c41586d3111c)

6. Select `➕ Add`

And that's all, you can add the virtual folder to your own user the same way you just did for the researcher in order to test with Filezilla if everything looks fine. You can add as many virtualfolders as you want to any users, you'll just see more folders once connected to the sftpbioinfo through Filezilla.
