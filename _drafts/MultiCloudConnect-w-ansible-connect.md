---
title: "Multi Cloud Connect with Ansible - connect the hyperscaler"
date: 2019-09-30T17:00:05+02:00
draft: false

tags : ["cloud", "ansible", "automation"]
categories : ["cloud", "ansible", "automation"]
banner : "img/cloud_small_all.png"
author : "Fabian Born"
---

The last two posts shows the setup of the native vpn services on Azure and Google Cloud Platform. Now I will show you how the connection can etablish  between both hyperscaler.

### Create VPN Tunnel on GCP


```yaml
    - name: "GCP: create a vpn tunnel"
      gcp_compute_vpn_tunnel:
        name: "testobject"
        region: "{{ gcp_region }}"
        peer_ip: "{{ output_ip_address.state.ip_address }}"
        target_vpn_gateway: "{{ gateway }}"
        #remote_traffic_selector: "{{ az_network }}"
        local_traffic_selector: "{{ gcp_subnet }}"
        shared_secret: "{{ shared_key }}"
        project: "{{ gcp_project }}"
        auth_kind: "{{ gcp_cred_kind }}"
        service_account_file: "{{ gcp_cred_file }}"
        state: present
      when: google == "present"
```

### Create VPN Tunnel on Azure

For creating the the vpn gateway you cannot use ansible, because there are no modules available yet. So for this case I have installed the AzureCli. We will be executed through Ansible. 

Make sure that AzureCLI access to azure:
```
az account list -o table
```
or
```
az login
```

Now this part will create a local gateway, which will be used for the connectivity. After that creating the local gateway the connection between both sites will be etablished 

```yaml
    - name: "script: Setup VPN Gateway
      script: az network local-gateway create --gateway-ip-address {{address.address}} --name google --resource-group {{ rg }} --local-address-prefixes {{gcp_subnet}}"

    - name: "script: Etablish connection to Google
      script: az network vpn-connection create --name connect2google --resource-group {{ rg }} --vnet-gateway1 {{ rg }}_vpngw -l {{ az_region }} --shared-key {{ shared_key }} --local-gateway2 google"
```

### Fazit
I hope this post series helps you to implement multi cloud connection. The full ansible scribt can be found on [github]



 <!---  
    Reference/Links which are included in text
 --->

[awsdocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_aws.html "AWS docs"
[gcpdocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_gce.html "GCP ansible docs"
[azuredocs]: https://docs.ansible.com/ansible/2.8/scenario_guides/guide_azure.html "Azure ansible docs"

[aws]: https://aws.amazon.com "AWS"
[gcp]: https://cloud.google.com "Google"
[azure]: https://portal.azure.com "Azure"
[github]: https://github.com/fabian-born/vpn-multicloud-connect "Github repository"