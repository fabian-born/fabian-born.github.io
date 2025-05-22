---
title: "Multiprotocol environment with FSxN"
date:  2025-10-10
draft: true
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


