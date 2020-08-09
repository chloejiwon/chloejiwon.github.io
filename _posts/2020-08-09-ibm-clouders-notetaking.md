---
title: "IBM Cloud Essentials Badge 획득하기"
excerpt: "IBM CLOUD essentials 무료강의 듣고 badge 따자!"

categories:
  - 클라우드
tags:
  - cloud
last_modified_at: 2020-08-09T01:11:00-05:00


---



> ibm CLouders로 활동하면서 ibm CLoud Essentials 강의 들으며 note taking한 거 공유하는 글이다. 클라우드의 개략적인 개념을 이해할 수 있어서 좋았다.

# Iaas / Paas / Saas

aaS = as a Service, how you consume

* Iaas : Infrastructure. For example, my computer, some random other person's computer running somewhere else ... 
  * persona of Iaas is "System admin"
  * It's like a "leasing a car"
* Paas : Platform.
  * persona is "Developer"
  * renting a car 
* Saas : Software
  * taking taxi / Uber

<img src="/assets/images/image-20200809084416216.png" alt="image-20200809084416216" style="zoom:50%;" />



# Cloud Native

<img src="https://d1fto35gcfffzn.cloudfront.net/images/topics/cloudnative/diagram-cloud-native.png" alt="Cloud-Native" style="zoom:33%;" />



Microservice, Container를 Cloud native application으로 개발 



# Infrastructure Services

* Bare Metal Server ? 
  * A dedicated physical server which is yours to use and manage from the 'metal up'
  * No sharing of underlying hardware

## Virtual Private Cloud (VPC)

 traditional cloud works like this. Network engineers have to do all network system things. routers, VPN, ... 

<img src="/assets/images/image-20200809092240180.png" alt="image-20200809092240180" style="zoom:50%;" />

However <<Virtual Networking>> , all of these capabilities is given as a service. User can create these functions and the segmentation with a UI or CLI or API without knowing any proprietary interfaces. 



VPC is isolated logical network that you create. this includes Multi Region Zone(MZR), Sequrity Groups , Connectivity. Other services are provided to support VPC like Load Balancing. 



## VMWare

What IBM cloud gives:

* Data sovereignty compliance - Geo-Fencing for workloads,  Data doeson't cross borders
* Compliance and regulatory control







# Reference

* https://tanzu.vmware.com/kr/cloud-native
* 