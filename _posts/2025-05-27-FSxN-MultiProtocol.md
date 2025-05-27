---
title: "Multiprotocol environment with FSxN"
date:  2025-05-27
draft: false
categories: howto
tags: ["NetApp","FileServices","Ransomware","FSxN","AWS","Cloud"]
banner: /assets/images/content/cloud-security.png
layout: post
toc: false

author: "Fabian Born"
---

## Situation
Customer operates its entire production environment in AWS. Users are managed centrally in the company's Active Directory and access to resources is controlled via group memberships. The server environment consists primarily of Linux systems that are joined to the Active Directory with SSSD. FSx for NetApp ONTAP (FSxN) provides shared file systems for applications and employees. The pre-defined SVM (Storage Virtual Machine) is also an Active Directory member to enable authentication against the customer’s directory services.

## Requirements
The challenge now is to make the SVM multi-protocol ready, which basically works out of the box. that the following requirements are met:
- Authorization is based on group membership in Active Directory
- The leading access protocol is SMB
- Linux hosts should be able to mount the shared file systems via SMB and NFSv4.1
- Each shared file system must allow authorized users to create files and folders, read all data but limit them to only change or delete their own files and folders.


## How was it solved?
To solve the requirement, two things need to be configured:
1. NFSv 4.1 including Kerberos and Active Directory
2. NTFS file system permissions must be set.

NetApp provides various documentation and technical reports for the configuration of NFSv4, which you can work through completely, or you can use the next steps.

#### Configuration of the FSxN system
**note:** the following steps need the "advanced priviledge": ```set adv```

#### Enable Kerberos on FSxN
```
kerberos realm create -vserver filestore -realm ad.epicshit.io -kdc-vendor Microsoft -kdc-ip 192.168.4.134 -kdc-port 88 -clock-skew 5 -adminserver-ip 192.168.4.134 -adminserver-port 749 -passwordserver-ip 192.168.4.134 -passwordserver-port 464 -adserver-ip 192.168.4.134 -adserver-name dc01.ad.epicshit.io
```

#### Enable SPN for NFS interface
```
kerberos interface enable -vserver filestore -lif nfs_smb_management_1 -spn nfs/filestore.ad.epicshit.io@AD.EPICSHIT.IO -admin-username fboadm
```

**important** This is the DNS name that will be used to mount the share later. This name must point to the IP address of the nfs_smb_management_1 interface

```
net int show -vserver filestore 
  (network interface show)
            Logical    Status     Network            Current       Current Is
Vserver     Interface  Admin/Oper Address/Mask       Node          Port    Home
----------- ---------- ---------- ------------------ ------------- ------- ----
filestore
            iscsi_1      up/up    10.64.22.173/24    FsxId0ebcb5e155ad2b3cd-01 
                                                                   e0e     true
            iscsi_2      up/up    10.64.22.182/24    FsxId0ebcb5e155ad2b3cd-02 
                                                                   e0e     true
            nfs_smb_management_1 
                         up/up    10.64.22.99/24     FsxId0ebcb5e155ad2b3cd-01 
                                                                   e0e     true


nslookup filestore.ad.epicshit.io                                                                                                               ─╯
Server:		192.168.4.134
Address:	192.168.4.134#53

Non-authoritative answer:
Name:	filestore.ad.epicshit.io
Address: 10.64.22.99
```

#### Configure ldap client
```
vserver services name-service ldap client create -vserver filestore -client-config filestore -ad-domain ad.epicshit.io -bind-as-cifs-server true -schema MS-AD-BIS
vserver services name-service ns-switch modify -vserver filestore  -database passwd,group -sources ldap,files 
vserver services name-service ldap create -vserver filestore -client-config filestore  
```

#### Change the NFSv4 Domain of the SVM
```
vserver nfs modify -vserver filestore -v4-id-domain ad.epicshit.io
```

#### Now create the user mapping for Linux - Kerberos
```
vserver name-mapping create -vserver filestore -direction krb-unix -position 1 -pattern (.+)\$@.* -replacement pcuser
vserver name-mapping create -vserver filestore -direction krb-unix -position 2 -pattern (.+)@.* -replacement \1
vserver name-mapping create -vserver filestore -direction win-unix -position 1 -pattern AD\\(.+) -replacement \1
vserver name-mapping create -vserver filestore -direction unix-win -position 2 -pattern (.+) -replacement AD\\\1
```

#### Verify user mapping on FSxN
```
vserver services access-check authentication show-creds -vserver filestore -win-name fabian

 UNIX UID: fabian <> Windows User: AD\fabian (Windows Domain User)

 GID: users
 Supplementary GIDs: 
  users

 Primary Group SID: AD\Domain Users (Windows Domain group)

 Windows Membership:
  AD\linux_allow_sudo_leitbache (Windows Domain group)
  AD\Domain Users (Windows Domain group)
  AD\linux_allow_sudo (Windows Domain group)
  Service asserted identity (Windows Well known group)
  BUILTIN\Users (Windows Alias)
 User is also a member of Everyone, Authenticated Users, and Network Users

 Privileges (0x2080):
  SeChangeNotifyPrivilege
```
or 
```
vserver services access-check authentication show-creds -vserver filestore -win-name ad\jodoe

 UNIX UID: pcuser <> Windows User: AD\jodoe (Windows Domain User)

 GID: pcuser
 Supplementary GIDs: 
  pcuser
```

As you can see in the second output, the user exists in the Active Directory, but not as a Unix user in ONTAP. To have a clean Win->Unix and Unix->Win user mapping, you have to create the user in the SVM on FSxN:
```
vserver services unix-user create -vserver filestore -user jodoe -id <AD uidNumber> -primary-gid <AD gidNumber>
```

#### Mount the exports / shares
Especially in the AD and NFSv4 context, it is important to work with the correct DNS names. **Note** for mounting use the active directory dns name and not the management name from the AWS console!

The file system can now be mounted.
```
nfs:    mount -t nfs -o vers=4.1,sec=krb5 filestore.ad.epicshit.io:/nfs1 /share/
smb:    mount -t cifs -o username=fabian,multiuser,sec=krb5 //filestore.ad.epicshit.io/smbonnfs1 /share/smb1/
```

Additional notes:
Possible options for sec= with Kerberos:
**krb5**	Use Kerberos for authentication only
**krb5i** 	Use Kerberos for authentication and hash traffic between client and server to ensure integrity
**krb5p** 	Use Kerberos for authentication and encrypt traffic between client and server


## Set the NTFS permissions
#### Creating an NTFS security descriptor
```
vserver security file-directory ntfs create -vserver filestore -ntfs-sd sd01 -owner AD\Administrator 
```
Adding ‘-control-flags-raw 0x9014’ disables inheritance, only the defined ACL are set. Use this if permissions are not applied to a volume.

```
vserver security file-directory ntfs create -vserver filestore -ntfs-sd sd01 -owner AD\Administrator -control-flags-raw 0x9014
```

#### Removing BUILTIN* from DACL list
```
vserver security file-directory ntfs dacl remove -ntfs-sd sd01 -access-type allow -account BUILTIN\* -vserver filestore
```

#### Adding NTFS DACL access control entries to the NTFS security descriptor
```
vserver security file-directory ntfs dacl add -ntfs-sd sd01 -access-type allow -account "AD\Domain Users" -advanced-rights read-data, execute-file, read-ea, read-attr, read-perm, write-data, append-data, write-attr -vserver filestore -apply-to this-folder,sub-folders,files
```

#### Verifying DACL
```
vserver security file-directory ntfs dacl show -ntfs-sd sd01

Vserver: filestore
  NTFS Security Descriptor Name: sd01

    Account Name     Access   Access             Apply To
                     Type     Rights
    --------------   -------  -------            -----------
    AD\Domain Users   
                     allow    read-data, execute-file, read-ea, read-attr, read-perm, write-data, append-data, write-attr   
                                                 this-folder, sub-folders, files
    CREATOR OWNER    allow    full-control       this-folder, sub-folders, files
    NT AUTHORITY\SYSTEM   
                     allow    full-control       this-folder, sub-folders, files
3 entries were displayed.
```

#### Creating a security policy and adding a task 
```
vserver security file-directory policy create -policy-name sd01-policy -vserver filestore

vserver security file-directory policy task add -policy-name sd01-policy -path /nfs1/group1 -ntfs-mode propagate -security-type ntfs -ntfs-sd sd01  -access-control file-directory -vserver filestore
```

#### Applying the security policy on NTFS files and folders using the CLI
```
vserver security file-directory apply -vserver filestore -policy-name sd01-policy
```

#### Monitoring the security policy job
```
vserver security file-directory job show  -vserver filestore
 
vserver security file-directory job show  -vserver filestore                     
                            Owning
Job ID Name                 Vserver    Node           State
------ -------------------- ---------- -------------- ----------
2690   Fsecurity Apply      filestore  FsxId0ebcb5e155ad2b3cd-01 
                                                      Success
       Description: File Directory Security Apply Job
```

#### Verifying the applied file security
```
vserver security file-directory show -vserver filestore -path /nfs1/group1

                  Vserver: filestore
Dummy index for tree walk: -
                File Path: /nfs1/group1
        File Inode Number: 99
           Security Style: ntfs
          Effective Style: ntfs
           DOS Attributes: 10
   DOS Attributes in Text: ----D---
  Expanded Dos Attributes: -
             UNIX User Id: 0
            UNIX Group Id: 0
           UNIX Mode Bits: 777
   UNIX Mode Bits in Text: rwxrwxrwx
                     ACLs: NTFS Security Descriptor
                           Control:0x9014
                           Owner:AD\Administrator
                           Group:AD\Domain Users
                           DACL - ACEs
                             ALLOW-AD\Domain Users-0x1201af-OI|CI
                             ALLOW-CREATOR OWNER-0x1f01ff-OI|CI
                             ALLOW-NT AUTHORITY\SYSTEM-0x1f01ff-OI|CI
```

## Additional links
[TR-4616 - NFS Kerberos in ONTAP](https://www.netapp.com/media/19384-tr-4616.pdf)
[TR-4887 - Multiprocotol NAS in NetApp ONTAP](https://www.netapp.com/media/27436-tr-4887.pdf)
[ONTAP Documentation](https://docs.netapp.com/us-en/ontap/nfs-admin/ontap-support-kerberos-concept.html)
