---
title: "Multi Cloud Connect with Ansible - configure AWS"
date: 2019-09-11T22:50:05+02:00
draft: true

tags : ["cloud", "ansible", "automation"]
categories : ["cloud", "ansible", "automation"]
banner : "img/cloud_small_aws.png"
author : "Fabian Born"
---

The last two post shows how you can configure a VPN Gateway in Azure and Google. Now I will show you how it works in AWS.

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

