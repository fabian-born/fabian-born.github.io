---
title: "NetApp Ansible Authentication with Hashicorp Vault"
date:  2020-05-08T12:45:00+02:00
draft: false
categories: ["ansible","automation","hashicorp"]
tags : ["cloud", "ansible", "automation","hashicorp"]
author : "Fabian Born"
banner: "img/netapp-ansible-hasicorp.png"
---
## NetApp Ansible Authentication with Hashicorp Vault
Today I want to show how to use NetApp Ansible Modules together with a Credential Store. Although Ansible comes also with a vault, but I decided to use Hashicorp Vault today. 

The advantage of Hashicorp Vault is that you have a central management of credentials. The credential store can also be designed redundantly. The installation is easy and very fast.

All actions described here can also be done via WebUI. But the CLI is much easier ;-)


#### Preparation of credential store for NetApp systems

##### Credential Store
First you have to create the secret store. There are two possibilities here:

1. via CLI
```bash
$ vault secrets enable -path=secretstore -version=1 kv
```
oder
```bash
$ vault secrets enable -path=secretstore -version=2 kv
```


2. via WebUI
```
http(s)://<vaultserver>:8200/
```

Two versions are available for the KV (Key/Value) store. More details can be read [here](http://www.vaultproject.io).

##### ACL for the store

A policy is required for the user who is to access the credential later, or an existing policy must be adapted in the following points:


```json
path "secretstore/netapp/*"
{
  capabilities = ["read", "list"]
}
// add the right for create tokens
path "auth/token/*"
{
  capabilities = ["create", "update"]
}
```


##### Creating an account and access token for Ansible

To create an access token you need an account to access the vault:

```bash
# Login as Administrator #
##########################
$ vault write auth/userpass/users/netapp-ansible password="securepassword" policies="read-netapp-credential"
Success! Data written to: auth/userpass/users/netapp-ansible
```

##### Create token
First you have to login with your account:
```bash
# Login as netapp-ansible #
##########################
$ vault login -method=userpass username=netapp-ansible

Password (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.F7QmKFkb8BL7RltxPjb0VmeP
...
token_policies         ["default" "read-netapp-credential"]
token_meta_username    netapp-ansible
```

After the successful login, we can create an access token for the user:

```bash
$ vault token create -policy=read-netapp-credential  -display-name='NetApp Ansible Token'


Key                  Value
---                  -----
token                s.ZHlgLUVI1FVcT0VPxcCa6OTM
token_accessor       6TK5JIxqOuXOgbphf0IVgWVB
token_duration       768h
token_renewable      true
token_policies       ["default" "read-netapp-credential"]
identity_policies    []
policies             ["default" "read-netapp-credential"]
```
##### Creating the credential for an ONTAP system

Now we create the credentials for each Ontap system:
- Command 1) stores the password as plain text value
- Command 2) stores the password as base64 encoded
```bash
$ vault kv put secretstore/netapp/<clustername> username=admin-user password=my-secret-password
```

base64 encode password
```bash
$ vault kv put secretstore/netapp/v2/democluster username=admin-user password=`echo "my-secret-password" | base64`
```
Of course you can also do this via WebUI.




### NetApp Module authentication
To use Hashicorp Vault with Ansible, we need the Python module 'hvac'. This is simply done with the command "pip3 install hvac".

If you want to use the user information from the Hashicorp Vault, you just have to add the following lines to your playbook:

```yaml
    hashivault: "{{ lookup('hashi_vault', 'secret=secretstore/netapp/{{ cluster }} token=s.ZHlgLUVI1FVcT0VPxcCa6OTM url=https://vault.cloudapps.fabianborn.net') }}"
    netapp_password: "{{ hashivault.password | b64decode }}"
    netapp_username: "{{ hashivault.username }}"
```

Here is a simple example for the output:
```yaml
---
- name: Demo Ansible Hashicorp Vault
  hosts: localhost
  connection: local
  gather_facts: False
  vars:
    cluster: "democluster"
    hashivault: "{{ lookup('hashi_vault', 'secret=secretstore/netapp/{{ cluster }} token=s.ZHlgLUVI1FVcT0VPxcCa6OTM url=https://vault.cloudapps.fabianborn.net') }}"
    netapp_password: "{{ hashivault.password | b64decode }}"
    netapp_username: "{{ hashivault.username }}"
  tasks:
  - name: Show Password
    debug:
      msg: "{{ netapp_username}} : {{  netapp_password }} = crypted: {{ hashivault.password }}"
```
```output
$ ansible-playbook ansible-hashicorpvault.yaml 

PLAY [Demo Ansible Hashicorp Vault] ***************************************************************************************************************************************************

TASK [Show Password] ***************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "admin-user : netapp123 = crypted: bmV0YXBwMTIz"
}

localhost                  : ok=1    changed=0    unreachable=0    failed=0   
```


Once you have filled the variables with the values, you can authenticate to NetApp. The passwords are loaded at runtime and forgotten afterwards.

The token can be revoked or can be exchanged at any time. Of course, other authentication options such as LDAP, AD, Github or similar are also possible. Just have a look at the Hashikorp documentation for that.