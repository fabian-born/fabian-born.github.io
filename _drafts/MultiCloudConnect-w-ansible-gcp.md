---
title: "Multi Cloud Connect with Ansible - configure GCP"
date: 2019-09-10T18:00:00+02:00
draft: false

tags : ["cloud", "ansible", "automation"]
categories : ["cloud", "ansible", "automation"]
banner : "img/cloud_small_gcp.png"
author : "Fabian Born"
---

The last post shows how you can configure a VPN Gateway in Azure. Now I will show you how it works in Google Cloud Platform.

But first this post repeats the requirements form the last one. So we need a working authorization to the hyperscaler and our required network configuration. 

Google, Azure and AWS provides the information, how you can connect the platform with Ansible. So I linked only the documentation:

 1. AZURE: [azuredocs]
 2. Google: [GCPdocs]
 3. AWS: [AWSdocs]


### Environment
Based on the last post we will use following network/subnet definition:

|         | | subnet          | region  |   |
|----|----|----|----|----|
| Azure   | |172.16.8.0/22   | eastus  |   |
| Google  | |172.16.16.0/22  |   |   |
| AWS     | |172.16.24.0/22  |   |   | 
| on-Prem | |172.16.32.0/22  |   |   |


### Deploying google
The network configuration in Google is just a little bit easier as in Azure. So we can configure the complete subnet or we choose someone from the whole. In this example I select a shorter subnet:


```yaml 
vars:
    gcp_project: "gcp-project-xxxx"
    gcp_cred_kind: "serviceaccount"
    gcp_cred_file: "/path/to/gcp.json"
    gcp_subnet: "172.16.17.0/24"
    gcp_region: "us-east1"
```
* gcp_project: your google-cloud project name
* gcp_cred_kind: account type
* gcp_cred_file: Credential file

This is an example of a credential file: 
```json
{
  "type": "service_account",
  "project_id": "gcp-project-xxxx",
  "private_key_id": "a61ab2b5ea921024adccd4be7727fb1ef1010290",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMDyQsLYCvfrKhr3dCW6RP6Z2UFftXDu+tD5x4F7+xl02o5Y1xG0E\ngOrT8xLupGOEnji9AONTTQI=\n-----END PRIVATE KEY-----\n",
  "client_email": "ansible@gcp-project-xxxx.iam.gserviceaccount.com",
  "client_id": "11068912349840049401",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/ansible%40gcp-project-xxxx.iam.gserviceaccount.com"
}
```


### Create GCP virtual network
```yaml
#### GCP Configuration
    - name: "GCP: create a network"
      gcp_compute_network:
        name: "{{ rg }}network"
        auto_create_subnetworks: 'false'
        project: "{{ gcp_project }}"
        auth_kind: "{{ gcp_cred_kind }}"
        service_account_file: "{{ gcp_cred_file }}"
        #state: "{{ status }}"
        state: present
      register: gcpnetwork
```
### Add the subnets to the virtual network
```yaml
    - name: "GCP: create a subnetwork"
      gcp_compute_subnetwork:
        name: "multicloudconnect"
        region: "{{ gcp_region}}"
        network: "{{ gcpnetwork }}"
        ip_cidr_range: "{{ gcp_subnet}}"
        project: "{{ gcp_project }}"
        auth_kind: "{{ gcp_cred_kind }}"
        service_account_file: "{{ gcp_cred_file }}"
        state: present
```

### Request a dynamic public ip address for the vpn gateway
```yaml
    - name: create a address
      gcp_compute_address:
        name: addressvpngateway
        region: "{{ gcp_region }}"
        project: "{{ gcp_project }}"
        auth_kind: "{{ gcp_cred_kind }}"
        service_account_file: "{{ gcp_cred_file }}"
        state: present
      register: gcp_public_address
```
### Create a new VPN Gateway
```yaml
    - name: create a target vpn gateway
      gcp_compute_target_vpn_gateway:
        name: gateway-vpn-tunnel
        region: "{{ gcp_region }}"
        network: "{{ gcpnetwork }}"
        project: "{{ gcp_project }}"
        auth_kind: "{{ gcp_cred_kind }}"
        service_account_file: "{{ gcp_cred_file }}"
        state: present
      register: gcp_gateway
```

### Create Firewall rules for VPN Connect
```yaml
    - name: "GCP: create a forwarding rule ESP"
      gcp_compute_forwarding_rule:
        name: forwardingesp
        region: "{{ gcp_region }}"
        target: "{{ gcp_gateway }}"
        ip_protocol: ESP
        ip_address: "{{ gcp_public_address.address }}"
        project: "{{ gcp_project }}"
        auth_kind: "{{ gcp_cred_kind }}"
        service_account_file: "{{ gcp_cred_file }}"
        network_tier: "STANDARD"
        state: present

    - name: "GCP: create a forwarding rule UDP500"
      gcp_compute_forwarding_rule:
        name: forwardingudp500
        region: "{{ gcp_region }}"
        target: "{{ gcp_gateway }}"
        ip_protocol: UDP
        port_range: "500"
        ip_address: "{{ gcp_public_address.address }}"
        project: "{{ gcp_project }}"
        auth_kind: "{{ gcp_cred_kind }}"
        service_account_file: "{{ gcp_cred_file }}"
        network_tier: "STANDARD"
        state: present

    - name: "GCP: create a forwarding rule UDP4500"
      gcp_compute_forwarding_rule:
        name: forwardingudp4500
        region: "{{ gcp_region }}"
        target: "{{ gcp_gateway }}"
        ip_protocol: UDP
        port_range: "4500"
        ip_address: "{{ gcp_public_address.address }}"
        project: "{{ gcp_project }}"
        auth_kind: "{{ gcp_cred_kind }}"
        service_account_file: "{{ gcp_cred_file }}"
        network_tier: "STANDARD"
        state: present
```

If you start with post one, you have two VPN Gateways running. The next step is to setup the connection between Azure and Google or you can create a VPN gateway on [aws]

The next post will show you, how you can setup the relationship betweenn Azure and Google. It will be available as soon as possible :-).


Update (2019-09-30): The next post is available: ["Multicloud Connect with Ansible: The Connection"]
 <!---  
    Reference/Links which are included in text
 --->
 <!---  
    Reference/Links which are included in text
 --->
[awsdocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_aws.html "AWS docs"
[gcpdocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_gce.html "GCP ansible docs"
[azuredocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_azure.html "Azure ansible docs"

[aws]: https://aws.amazon.com "AWS"
[gcp]: https://cloud.google.com "Google"
[azure]: https://portal.azure.com "Azure"
["Multicloud Connect with Ansible: The Connection"]: /blog/2019/09/30/multicloudconnect-w-ansible-connect/

