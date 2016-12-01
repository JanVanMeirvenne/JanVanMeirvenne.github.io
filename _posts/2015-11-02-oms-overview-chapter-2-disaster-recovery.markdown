---
author: janvanmeirvenne
comments: true
date: 2015-11-02 07:23:19+00:00
layout: post
link: http://scug.be/jan/2015/11/02/oms-overview-chapter-2-disaster-recovery/
slug: oms-overview-chapter-2-disaster-recovery
title: 'OMS overview: chapter 2 - disaster recovery'
wordpress_id: 325
categories:
- '#msoms'
- '#sysctr'
tags:
- '#msoms #sysctr'
---

### OMS Blog Series Index

 

Since there is much to say about each of the Operations Management Suite aka OMS services I will break up this post in a blog series:

 

**[Chapter 1: Introduction to OMS](http://scug.be/jan/2015/11/01/oms-overview-chapter-1-introduction/)         
Chapter 2: Disaster Recovery with OMS (this post)         
Chapter 3: Backup with OMS         
Chapter 4: Automation with OMS         
Chapter 5: Monitoring and Analysis with OMS         
Chapter 6: Conclusion and additional resources overview**

 

This series is actually a recap of a live presentation I gave on the last [SCUG event](http://scug.be/events/2015/09/30/scugbe-system-center-night-fall-edition-2210/) which was shared with another session on SCOM – SCCM better together scenario’s presented by SCUG colleagues [Tim](https://twitter.com/Tim_DK) and [Dieter](https://twitter.com/DieterWijckmans). You can find my slides [here](http://www.slideshare.net/JanVanMeirvenne/oms-overview-54622885), demo content that is shareable (the ones I don’t need to pay for ![Smile](http://scug.be/jan/files/2015/11/wlEmoticon-smile.png)) will be made available in the applicable chapters.

 

Disaster Recovery is one of the bigger money sinks of IT. First you need yourself a big secondary datacenter able to maintain your business in case your primary one bites the dust. Not only you need to throw money at something you might never need in your entire career, but even have to invest in setting up DR plans for every service you are planning to protect. This usually requires both additional design and implementation work, and separate tooling that allows you to perform the DR scenario you envisioned. Especially in the world of hybrid cloud, protecting services across platform boundaries might seem like a complex thing to do: needing to integrate multiple platforms in preferably one single DR solution.

 

## Meet [Azure Site Recovery](https://azure.microsoft.com/en-us/services/site-recovery/)

 

Azure Site Recovery or ASR provides 2 types of DR capabilities:

 

  
  * **Allowing the replication and failover orchestration of services between 2 physical sites of your own **
   
  * **Allowing the replication and failover orchestration of services between a main site that you own and the Azure IaaS platform**
 

Bear in mind that while the platform is advertised as a DR solution, it is also possible to use it as a migration tool to move workloads to Azure or other on-premise sites.

 

The big advantage here is that the solution is platform-agnostic, providing scenario’s to protect virtually any type of IT infrastructure platform you use. The DR site can be a secondary VMWare or Hyper-V (_with SCVMM_) Cloud, or Azure. Vendor-lock-in is becoming a non-issue this way!

 

## Supported Scenario’s

 

Here is a full overview of the supported flows:

 

**_Infrastructure_**

 

_To Azure_

 

  
  * [Hyper-V Site –> Azure](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-hyper-v-site-to-azure/)
   
  * [SCVMM + Hyper-V Site –> Azure](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-vmm-to-azure/)
   
  * [Physical Site –> Azure](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-vmware-to-azure/)
   
  * [VMWare Site –> Azure](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-vmware-to-azure/)
   
  * [Azure / Other Cloud –> Azure](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-migrate-azure-to-azure/)
 

_To an own DR-site_

 

  
  * [SCVMM + Hyper-V Site –> SCVMM + Hyper-V Site](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-vmm-to-vmm/)
   
  * [(Geo-clustered) SCVMM + Hyper-V Site –> Hyper-V Site](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-single-vmm/)
   
  * [VMWare Site –> VMWare Site (using InMage Scout)](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-vmware-to-vmware/)
   
  * [SCVMM + Hyper-V Site –> SCVMM + Hyper-V Site with SAN replication](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-vmm-san/)
 

**_Application_**

 

  
  * [SQL Always-On](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-sql/)
   
  * [Other](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-workload/#workload-guidance-summary) application types must be orchestrated by using the recovery plan feature or by doing a side-by-side migration / failover 
 

## Architecture

 

Basically, there are 2 major ‘streams’ within ASR to facilitate DR operations, but in any case you'll always need a Recovery Vault. The recovery vault is an encrypted container that sits on top on a (selectable) storage account. If the target DR site is Azure, the vault will store the replicated data and use it to deploy Azure IaaS VM's in case of a failover. If the target DR site is another on-premise site, the vault will only store the metadata needed for ASR to protect the main site. The vault is accessed by the on-premise systems using downloadable vault encryption keys. These keys are used during the setup and are accompanied by a passphrase the user must enter and securely store on-premise. Without this passphrase the vault becomes inaccessible should systems be (re-)attached to the vault, so it is very important to double-, no triple-backup this key!

 

All communication between the different sites is also encrypted using SSL.

 

In the case that Azure is the target DR site, you must specify an Azure size for each VM you want to protect along with an Azure virtual network to connect it to. This allows you to control the cost impact.

 

**_The Microsoft Azure Recovery Services Agent (MARS) and the Microsoft Azure Site Recovery Provider (MASR)_**

 

This setup is applicable to any scenario where Hyper-V (with or without SCVMM) is the source site in the DR plan.

 

The MARS agent needs to be installed on every Hyper-V server which will take part in the DR-scenario (both source and target). This agent will facilitate the replication of the actual VM data from the source Hyper-V servers to the target site (Azure or other Hyper-V server).

 

The MASR agent needs to be installed on the SCVMM server(s) or in case of a Hyper-V site to Azure scenario it needs to be co-located on the source Hyper-V server together with the MARS agent. The MASR agent is responsible for orchestrating the replication and failover execution and primarily sync meta-data to ASR (actual replication data is done by the MARS agent).

 

[![image](http://scug.be/jan/files/2015/11/image_thumb6.png)](http://scug.be/jan/files/2015/11/image6.png)

 

[![image](http://scug.be/jan/files/2015/11/image_thumb7.png)](http://scug.be/jan/files/2015/11/image7.png)

 

_**The Process, Master Target and Configuration Server**_

 

This setup is used for any scenario with a VMWare (with some additional components described later on), Cloud (Azure or other) or Physical site as a source. These are the components which facilitate the Replication and DR Orchestration. Note: they are all IaaS level components.

 

_Process Server_

 

This component is placed in the source site and is responsible for pushing the mobility service to the protected servers, and to collect replication-data from the same servers. The process server will store, compress, encrypt and forward the data to the Master Target Server running in Azure.

 

_Mobility Service_

 

This is a helper-agent installed on all systems (Windows or Linux) to be protected. It leverages VSS (on Windows) to capture application-consistent snapshots and upload them to the process server. The initial sync is snapshot-based, but the subsequent replication is done by storing writes in-memory and mirroring them to the process server.

 

_Master Target Server_

 

The Master Target Server is an Azure-based system that receives replication-data from the source site's process server and stores it in Azure Blob Storage. As a failover will incur heavy resource demands on this system (rollout of the replica's into Azure IaaS VMs) it is important to choose the correct sizing in regards of storage (standard or premium) to ensure a service can failover within the established RTO.

 

_Configuration Server_

 

This is another Azure-based component that integrates with the other components (Master Target, Mobility Service, Process Server) to both setup and coordinate failover operations.

 

Failback to VMWare (or even failover to a [DR VMWare site](https://azure.microsoft.com/en-us/blog/azure-site-recovery-now-offers-disaster-recovery-for-any-physical-or-virtualized-it-environment-with-inmage-scout-2/) instead) is possible with this topology with some additional components. It is nice to see that Microsoft is really upping the ante in regards of providing a truly heterogeneous DR solution in the cloud!

 

[![image](http://scug.be/jan/files/2015/11/image_thumb8.png)](http://scug.be/jan/files/2015/11/image8.png)

 

### 

 

### Orchestrating workload failover/migration using the Recovery Plan feature

 

Of course, while you can protect you entire on-prem environment in one go, this is not an application-aware setup. If you want to make sure your services are failed over with respect of their topology (backend -> middleware -> application layer -> front-end) you need to use the recovery plan feature of ASR.

 

[Recovery Plans](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-create-recovery-plans/) allows you to define an ordered chain of VMs along with actions needing to be taken at source site shutdown (pre and post) and target site startup (pre and post). Such an action can be the [execution of an automation runbook](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-runbook-automation/) hosted by [Azure Automation](https://azure.microsoft.com/en-us/services/automation/), or a manual action to be performed by an operator (the failover will actually halt until the action is marked as completed).

 

![](https://acomdpsstorage.blob.core.windows.net/dpsmedia-prod/azure.microsoft.com/en-us/documentation/articles/site-recovery-runbook-automation/20151007045150/17.png)       
Source: [https://azure.microsoft.com/en-us/documentation/articles/site-recovery-runbook-automation/](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-runbook-automation/)

 

## 

 

### Failing Over

 

When in the end you want to perform an actual [failover operation](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-failover/) you can perform 3 types of actions:

 

- Test Failover: this keeps the source service/system online while booting the replica so you can validate it. Keep in mind that you should take possible resource conflicts (DNS, Network, connected systems) into account.

 

- Planned Failover: this makes sure that the replica is fully in-sync with the source service/system before shutting it down and then boots the replica. This ensures no data loss occurs. This action can be done when migrating workloads or protecting against a foreseen disaster (storm, flood,...) the protected service will be offline during the failover

 

- Unplanned Failover: this type only brings online the replica from the last sync. Data loss will be present as a gap between the failure moment and the last sync. This is only for instances where the disaster already occurred and you need to bring the service online at the DR ASAP.

 

A failover can be executed on the per-VM level or via a recovery plan.

 

### Caveats and gotcha's

 

Although the ASR service is production-ready and covers a lot of ground in terms of features, there are some [limitations](https://azure.microsoft.com/en-us/documentation/articles/site-recovery-best-practices/) to take into account:

 

Here are some of the bigger limits:

 

_When using Azure as a DR site_

 

- Azure IaaS uses the VHD format as storage disk format, limiting the protectable size of the VHD or VHDX (conversion is done automatically) to 1024Gb. Larger sizes are not supported      
- the amount of per-VM resources (CPU Cores, RAM, Disks) is limited by the supported resources provided by the largest Azure IaaS sizing (eg if you have 64 attached disks in your on-prem you might not be able to protect it if Azure's maximum is 32)

 

_Overall Restrictions_

 

- Attached Storage setups like Fiber Channel, Pass-through Disks or iSCSI are not supported      
- Gen2 Linux VMs are not yet supported

 

### This looks nice! But how much does it cost?

 

The nice thing about using Azure as a DR-site is that you only pay a basic fee for the service, including storage and WAN traffic, but only pay the full price for IaaS compute resources when an actual failover occurs. This embodies the concept of ‘Pay what you use’ that is one of the big benefits of public cloud. Even better: you only start paying the basic fee after 31 days. So in case you would use ASR as a migration tool (moving workloads to the cloud or another site) you will have a pretty cost-effective solution! Bear in mind that used storage and WAN traffic is _always_ billed.

 

I won't bother to list the pricing here as it is as volatile in nature as the service itself. You can use the [Azure pricing calculator](https://azure.microsoft.com/en-us/pricing/calculator/) to figure out the costs.

 

[![image](http://scug.be/jan/files/2015/11/image_thumb9.png)](http://scug.be/jan/files/2015/11/image9.png)

 

If you have one or more System Center licenses, check out the [OMS suite pricing calculator](http://omscalculator.azurewebsites.net/) instead to asses if you can benefit from the bundle-pricing.

 

### 

 

### Ok, I'll bite, but how do I get started?

 

The service for now is only accessible from the 'old' Azure Portal on [https://manage.windowsazure.com](https://manage.windowsazure.com)

 

Log in with an account that is associated with an Azure subscription, and click the 'new'-button in the bottom-left corner.

 

[![image](http://scug.be/jan/files/2015/11/image_thumb10.png)](http://scug.be/jan/files/2015/11/image10.png)

 

Choose 'Data Services' -> 'Recovery Services' -> 'Site Recovery Vault'

 

[![image](http://scug.be/jan/files/2015/11/image_thumb11.png)](http://scug.be/jan/files/2015/11/image11.png)

 

Click 'Quick Create' and then enter a unique name and choose the applicable region where you want to host the service. Then, click 'Create Vault'.

 

[![image](http://scug.be/jan/files/2015/11/image_thumb12.png)](http://scug.be/jan/files/2015/11/image12.png)

 

This will create the vault from where you can start the DR setup

 

[![image](http://scug.be/jan/files/2015/11/image_thumb13.png)](http://scug.be/jan/files/2015/11/image13.png)

 

When the creation is done, go to 'Recovery Services' in the left-side Azure Service bar and then click on the vault you created.

 

[![image](http://scug.be/jan/files/2015/11/image_thumb14.png)](http://scug.be/jan/files/2015/11/image14.png)

 

The first thing you must do is to pick the appropriate scenario you want to execute

 

[![image](http://scug.be/jan/files/2015/11/image_thumb15.png)](http://scug.be/jan/files/2015/11/image15.png)

 

This will actually provide you with a tutorial to set up the chosen scenario!

 

[![image](http://scug.be/jan/files/2015/11/image_thumb16.png)](http://scug.be/jan/files/2015/11/image16.png)

 

To re-visit or change this tutorial during operational mode, just click the 'cloud icon' in the ASR interface

 

[![image](http://scug.be/jan/files/2015/11/image_thumb17.png)](http://scug.be/jan/files/2015/11/image17.png)

 

I won't cover the further steps needed as the tutorials provided by Azure are exhaustive enough. I might add specific tutorials later on in a dedicated post in case that I encounter some advanced subjects.

 

 

### Final Thoughts on ASR

 

While I am surely not a data protection guy, setting this puppy up was a breeze to me! This service, which is now part of OMS, embodies the core advantages of cloud: immediate value, low complexity and cross-platform. I have already seen several implementations, confirming that this solution is here to stay and will likely be a go-to option for companies looking for a cost-effective DR platform.

 

Thanks for the long read! And see you next time when we will touch ASR's sister service [Azure Backup](https://azure.microsoft.com/en-us/services/backup/)! Jan out.
