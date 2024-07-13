---
title: "Multi Cloud Connect with Ansible starting with Azure"
date: 2019-09-06T18:00:00+02:00
draft: false

tags : ["cloud", "ansible", "automation"]
categories : ["cloud", "ansible", "automation"]
banner : "img/cloud_small_azure.png"
author : "Fabian Born"
---

Everyone talks about cloud in this day and age. Everyone wants to test the diverse possibilities of hyperscaler like Amazon AWS, Microsoft Azure or Google Cloud.

At least if you want to use the services of other hyperscalers safely at the same time, you should think about an encrypted connection. Each Hyperscaler offers the possibility to establish "direct connections" with your own company. All traffic will be routed through your corperate network.

In this blog post, I'll show you how to make VPN connections between the hyperscaler with their native VPN services.

The complete ansible playbook is available [github-repo]( https://github.com/fabian-born/vpn-multicloud-connect "my github repository") 

# Multi Cloud Connect
### Credentials
Before starting with Ansible script, make sure that authentication to all hyperscaler works properly.

Every hyperscale has a tutorial how you can use ansible with them:

 1. AZURE: [azuredocs]
 2. Google: [GCPdocs]
 3. AWS: [AWSdocs]

### Environment
On each Hyperscaler we will use a new subnet for the multicloud connect. This subnets are an example:

|         | | subnet          | region  |   |
|----|----|----|----|----|
| Azure   | |172.16.8.0/22   | eastus  |   |
| Google  | |172.16.16.0/22  |   |   |
| AWS     | |172.16.24.0/22  |   |   | 
| on-Prem | |172.16.32.0/22  |   |   |


### Deploying Azure
The first post will show, how you deploy a subnet / vpn gateway on azure. 
The ansible script contains different sections / task. In the beginnig there are the variables which must be defined.

```yaml 
vars:
    rg: "multicloud"
    az_region: "eastus"
    az_network: "172.16.8.0/22"
    az_subnet:
      - { subnet: '172.16.9.0/24', name: 'vmnet9' }
      - { subnet: '172.16.10.0/24', name: 'vmnet10' }
      - { subnet: '172.16.11.0/24', name: 'vmnet11' }
    az_gwsubnet: "172.16.9.0/24"
```


Later, we want to create a connection between the hyperscaler, we need an authorization key. This code snippet will create random "password":
```yaml
    secret_gen: "{{ lookup('password', '/dev/null length=32 chars=ascii_letters') }}"
```


```yaml
  tasks:
  #### Generate Secret Key for IKEv2 VPN
    - set_fact:
        shared_key: "{{ secret_gen }}"
    - debug:
        msg: "shared key: {{ shared_key }}"
```
### Create the new resource group on azure
```yaml
  #### AZURE Configuration
    - name: "Azure: Create a resource group {{ rg }}"
      azure_rm_resourcegroup:
        name: "{{ rg }}"
        location: "{{ az_region }}"
        state: "present"
        force_delete_nonempty: false
```
### Create Azure virtual network
```yaml
    - name: "Azure: Create a virtual network"
      azure_rm_virtualnetwork:
        name: "{{ rg }}_network"
        resource_group: "{{ rg }}"
        state: "{{ status }}"
        address_prefixes_cidr:
            - "{{ az_network }}"
        tags:
            testing: vpntest
            delete: on-exit
      when: status == "present"
```
### Add the subnets to the virtual network
```yaml
    - name: "Azure: Add subnet"
      azure_rm_subnet:
        resource_group: "{{ rg }}"
        name: "{{ rg }}_subnet_{{ item.name }}"
        address_prefix: "{{ item.subnet }}"
        virtual_network: "{{ rg }}_network"
      with_items: 
        "{{ az_subnet }}"
      when: status == "present"
```
### Add the required GatewaySubnet to the virtual network
```yaml
    - name: "Azure: Add subnet GatewaySubnet"
      azure_rm_subnet:
        resource_group: "{{ rg }}"
        name: "GatewaySubnet"
        address_prefix: "{{ az_gwsubnet }}"
        virtual_network: "{{ rg }}_network"
      when: status == "present"
```
### Request a dynamic public ip address for the vpn gateway
```yaml
    - name: "Azure: Create public IP address"
      azure_rm_publicipaddress:
        resource_group: "{{ rg }}"
        allocation_method: dynamic
        name: "{{ rg }}_vpn_pubip"
        state: "{{ status }}"
      register: output_ip_address
      when: status == "present"
    
    - name: "Azure: Return public IP"
      debug:
        msg: "The public IP is {{ output_ip_address.state.ip_address }}." 
      when: status == "present"

    - set_fact:
        azure_pubip: output_ip_address.state.ip_address
      when: status == "present"
```
### Create a new VPN Gateway
```yaml
    - name: "Azure: Create virtual network gateway"
      azure_rm_virtualnetworkgateway:
        resource_group: "{{ rg }}"
        name: "{{ rg }}_vpngw"
        sku: "VpnGw1"
        ip_configurations:
          - name: vpnipconfiguration
            private_ip_allocation_method: Dynamic
            public_ip_address_name: "{{ rg }}_vpn_pubip"
        virtual_network: "{{ rg }}_network"
        state: absent
      register: azure_vpn
      when: status == "present"
```

If the last task has been finished, the vpn gateway is up and running. Now you can create the the VPN gateway on [aws] or [gcp]




 <!---  
    Reference/Links which are included in text
 --->
[awsdocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_aws.html "AWS docs"
[gcpdocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_gce.html "GCP ansible docs"
[azuredocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_azure.html "Azure ansible docs"

[aws]: https://aws.amazon.com "AWS"
[gcp]: https://cloud.google.com "Google"
[azure]: https://portal.azure.com "Azure"

