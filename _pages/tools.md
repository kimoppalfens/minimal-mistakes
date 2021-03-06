---
title: Products
author: OSCC
layout: single
excerpt: "Page not found. Your pixels are in another canvas."
sitemap: false
permalink: /tools/
author_profile: true
gallery3:
  - image_path: AdminPasswordRandomizer-CMTSStep.png
    alt: "placeholder image 2"
  - image_path: AdminPasswordRandomizer-CMTSStep2.png
    alt: "placeholder image 2"
---


## What is Secure Manage?

Secure manage is a solution that aims to eliminate the need for highly privileged accounts to access workstations by randomizing the local administrator password on workstations and make that password easily consumable through pre-build actions. Much effort has been put in making the solution as comprehensible as possible and to allow the solution to integrate into an organization’s current support workflow. 

Secure Manage integrates into System Center Configuration Manager and the admin UI that comes with it to deliver 5 easilly accessible actions:

![alt]({{ site.url }}{{ site.baseurl }}/images//SecureManage1.png)

- Request the local admin password
- Open the C$ share
- Open the Admin$ share
- Open a remote desktop session
- Open a PowerShell remoting session


The first action shows you the randomized local administrator password in cleartext, whereas all the other actions request the password for the operator and transparently use that password to perform the operation. This leads to increased security by eliminating the need for highly privileged accounts, while eliminating many re-authentications and credential re-typing when an operator needs to access remote systems in his day-to-day work.

## Where are these randomized passwords stored?
The randomized local administrator passwords are stored in Active Directory as a property of the computer object they apply to. This can be a pre-existing property, or customers that so desire can specify their own property to work with.

## How are the passwords secured in Active Directory?

The solution encrypts the passwords using a customer specific public/private key pair, or, for those customers that so desire, using their own provided public/private key pair. The operator needs to have access to the private key to perform any of the operations, and additionally needs permissions to the Active Directory property of the object he wants to execute the operation on. This 2-step protection is audited to know who executed which action when and against which resource.

## When should I use Secure Manage?

We suggest that everyone that is interested in increasing its security posture, and looks at reducing its exposure to highly privileged accounts implements Secure Manage. Highly privileged accounts, like helpdesk accounts and the re-use of identical local administrator passwords make later traversal attacks in a company ridiculously easy. Both local administrator passwords and passwords from highly privileged users are attacked in a multitude of ways. 

## What can you tell me about the audit trail that was previously mentioned?

Whenever one of the operator actions is triggered, a status message is generated by the machine the operation originated from. This audit trail contains the user account of the operator, date & time of the action, the target of the action and what exact action was triggered. These details can subsequently be used in a pre-configured report in System Center Configuration Manager, or for customers that so desire by using SCCM status message queries.

## Pre- & Post actions for an increased security posture ##

The latest version comes with the ability to run scripts pre- & post one of the secure manage actions. This allows you to keep all firewall ports needed for connectivity closed, but open them on the fly when connectivity is needed. Given the flaws discovered in RDP over the past couple of years this limits your exposure to security vulnerabilities in time. It feels unnatural to us to lower the security posture on 100% of the devices in an environment because IT personnel needs the ability to connect to small portion on them once a year for providing support. This solution eliminates that need by enabling the prerequisites needed to connect only when connecting.


## I already use a different tool/ portal/… in my organization, can Secure Manage be integrated in that?

The answer to that is most likely yes. Secure Manage comes with a “standalone” PowerShell module to trigger the actions previously described. As with the Configuration Manager Admin UI integrated actions, the standalone versions generate an audit trail. To be able to do that, it does need to have the Configuration Manager Admin UI installed.

## How does Secure Manage relate to Credential Guard?

We still strongly suggest Credential Guard is enabled, as it is a good defense against other forms of credential leakage. However, Credential Guard is a software feature, meaning flaws can be discovered in it and other flows in the operating system can impact its trustworthiness, as was demonstrated using the Meltdown & Spectre vulnerabilities. Additionally, Credential Guard can be disabled on a system once an attacker achieves gaining administrative control over a workstation. This would give said attacker the opportunity to gather the password of a highly privileged account again. For more info ( [https://tinyurl.com/cgmimikatz](https://tinyurl.com/cgmimikatz))
In summary, we like Credential Guard, but there’s no better way to protect against the leakage of highly privileged accounts than not having those accounts in the first place. Eliminating the need for these types of accounts as much is possible is what Secure Manage was built for.


## How does Secure Manage relate to LAPS?

Although LAPS randomizes the local admin password, it makes the password unusable as opposed to integrating it into an operator’s day to day workflow. In doing so it doesn’t eliminate the need for other high-privilege accounts, the explicit goal of Secure Manage. On top of that, LAPS stores the passwords in clear text, and falls short in auditing capabilities.

## Software requirements

- System Center Configuration Manager
- Active Directory
- .Net Framework 4.5.2+

## How much does Secure Manage cost?

0-500 Clients: 785.50€ Flat fee/Year

500-2.500 Clients: 1.32 €/Client/Year

2.500 Clients – 5.000 Clients: 1.05 €/Client/Year

5.000 Clients – 10.000 Clients: 0.97 €/Client/Year

10.000 Clients – 50.000 Clients: 0.91 €/Client/Year

50.000 Clients and more: 0.84 €/Client/Year