---
title: Exam Avaya 3171T
share: "true"
tags:
  - exam
---

## 3171T

**2.Which statement about the deployment of Avaya Aura Presence Services with Avaya Aura 8 is true?**
- Presence Services always require one or more dedicated (closed) Avaya Breeze instances.

The Avaya Breeze based deployment model also allows for rapid application development in a robust, scalable environment which improves time-to-market by extending the Presence and IM messaging capabilities to 3rd party developers.

Presence Services generally requires one or more dedicated (closed) Avaya Breeze instances. Optionally the Presence Services Connector snap-in can be deployed on a general-purpose Avaya Breeze instance/cluster to provide presence related API access to Avaya Breeze application developers or co-exist with other snapins.

**3.Which two characteristics of Avaya strategy provide value and lower total cost of ownership for its customers? (choose two)**
- Vendor operability
- Minimal hardware in favor of virtualization options

**4.Which two factors distinguish Avaya in the market? (choose two)**
Open/multi-vendor
People-centric innovation

**5.The Multiple Device Access (MDA) feature allows multiple devices to register with the same extension number but use only one user license. Which three applications are requied for the MDA feature? (choose three)**
- Session Manager
- System Manager
- Communication Manager

Avaya Aura 6.2 Feature Pack 2 (FP2) introduced the Multi Device Access (MDA) feature that allows users to leverage multiple devices (endpoints) simultaneously to meet their communication needs. Users can receive and place calls at multiple devices, and move calls between devices as needed. The MDA feature spans multiple Avaya Aura components, including Avaya Aura Session Manager (SM), Avaya Aura Communication Manager (CM), Avaya Aura Conferencing (AAC), Avaya Aura Presence Server (PS) and supported Avaya SIP endpoints. The MDA feature leverages IETF RFC 3261 for core SIP and the parallel forking capability built into Session Manager. Communication Manager enhances the user experience by supporting the ability to bridge-on to calls at any of the user's devices.

Multiple Device Access requirements

Avaya Communicator supports Multiple Device Access (MDA) to provide the capability to:
- Log on to the same extension from multiple devices.
- Answer a call from multiple devices.
- Join a call from other logged in devices.
- Simultaneously ring all logged in devices when you receive a call on your extension.

The number of devices that can log in simultaneously depends on the Avaya Aura ® configuration for an extension.

Block New Registration When Maximum Registrations Active is an Avaya Aura ® feature. If you select the Block New Registration When Maximum Registrations Active check box and an endpoint attempts to register after the number of registration requests exceed the administered limit, the system denies any new request. The system sends a warning message and stops SIP service to the endpoint.

Then the user reaches the maximum simultaneous device limit, the Avaya Aura ® configuration determines whether the first or the last logged in device is denied access.

Avaya Communicator supports the Block New Registration When Maximum Registrations Active feature only with Session Manager 6.3.3, that is, Avaya Aura ® 6.2 FP2 SP1 or later. If you are using Avaya Communicator with an earlier version of Session Manager, you must disable the Block New Registration When Maximum Registrations Active feature.

MDA requires the use of TLS endpoints. Users can log in to their extension with a TCP device. However, a new incoming call does not ring on the TCP device and the user cannot join the call from that device.

Some MDA limitations exist for IM and Presence between Avaya Communicator and other applications. For more information, see Multiple Device Access White Paper.

For more information about other MDA user limitations, see:
- Using Avaya Communicator for Android
- Using Avaya Communicator for iPad (18-603943)
- Using Avaya Communicator for iPhone
- Using Avaya Communicator for Windows (18--604158)

**6.Avaya Aura Session Manager has many benefits such as offering a single, centralized dial plan. What is another benefit of Session Manager?**
- It provides SIP application interoperability across multi-vendor equipment

**7.With Communication Manager 8.0, how many digits are supported for call processing?**
- 16

*Подсказка*:
Branch Gateway Release 8.0 supports the following new features and enhancements:
- 16-digit dial plan extension
- The Standard Local Survivability (SLS) feature is updated to support 16-digit extensions.
- Login authentication password complexity
- Login authentication password complexity is enhanced to set the maximum number of consecutive
- repeated characters and consecutive characters of the same class (uppercase, lowercase, digits, symbols) in a password.
- Inbound DTMF digit processing
- Branch Gateway is enhanced to allow mobile users to dial the feature access code without putting the ongoing call on hold.

**8.Avaya Aura Application Enablement Services (AES) requires direct access with which Avaya Aura component?**
- Communication Manager

**9.What is licensing of the Avaya Session Border Controller for Enterprise is based on?**
- The number of simultaneous sessions

ASBCE 7.2 introduces dynamic licenses, in which licenses can be shared across ASBCEs.

Avaya SBCE supports pooled licensing. As opposed to static license allocation, Avaya SBCE dynamically reserves and unreserves pooled licenses when needed. For example, customers with multiple Avaya SBCE devices can use a pool of licenses dynamically across the devices as required. All features that are enabled for pooled licensing operate in a pooled mode. Avaya SBCE instances fetch the licenses based on traffic and need. Avaya SBCE releases these licenses when they are not in use. Avaya SBCE raises an alarm if the license usage exceeds the configured threshold for: Standard sessions • Advanced sessions • Scopia sessions • CES sessions • Transcoding sessions

Licensing requirements
Avaya SBCE uses WebLM version 8.0.0.0 for licensing requirements. You can install the Avaya SBCE license file on Element Management System (EMS) using the Device Management page.

Ensure that the license file of the WebLM server displays the product code Session Border Controller E AE. Before you configure the license file, you can view the License State, Grace Period State, and Grace Period Expiration Date fields on the Dashboard page. You have a 30-day grace period from the day of installation or upgrade to install the license. Avaya SBCE works normally during the grace period. The license file contains the following information:
- Product name
- Supported software version
- Expiration date
- Host ID

The primary host ID of WebLM is used for creating the license file.
- Licensed features
- Licensed capacity

All hardware Avaya SBCE devices can use a local WebLM server for licenses. However, for mixed deployment environments with EMS on VMware and Avaya SBCE on hardware, use a WebLM server installed on VMware or System Manager WebLM.

There are two licensed versions of Avaya SBCE:
• Standard Services delivers secure SIP trunking.
• Advanced Services adds Mobile Workspace User, Media Replication and other features to the Standard Services offer.

Avaya Aura ® Mobility Suite and Collaboration Suite licenses include Avaya SBCE.

**10.A customer is licensed for 2000 sessions in an Avaya Session Border Controller for Enterprise (ASBCE) 7.2 geo-redundant deployment. There are two ASBCE servers. If dynamic licensing is configures for the deployment, how many sessions can the server in Data Center 2 use, if the server in Data Center 1 is currently servicing 700 sessions?**
- 1300

Geographic-redundant deployment

In a Geographic-redundant deployment, you can deploy two different Avaya SBCE devices in two different data centers. You can deploy the devices as individual Avaya SBCE devices or devices managed by their own EMS. You can deploy these Avaya SBCE devices in a High Availability mode or a non-High Availability mode.

Geographic-redundant deployment in the non-HA mode

In the following diagram, SBCE1 and SBCE2 are two different physical devices deployed in different data centers. The endpoints have one connection with SBCE1 corresponding to the primary Session Manager, SM1. The second connection with SBCE2 corresponds to the secondary Session Manager, SM2.

Geographic-redundant deployment in the HA mode

In the following diagram, SBCE1 and SBCE2 are two different physical devices that are deployed in an HA mode in different data centers. The endpoints have one connection with SBCE1-A, that is Active SBCE corresponding to the primary Session Manager, SM1. The second connection is with SBCE2-A, Active SBCE corresponding to the secondary Session Manager, SM2.

During an SBCE1-A fail over, SBCE1-S, which is the standby Avaya SBCE, handles the media of the active calls. During an SBCE2-A fail over, SBCE2-S, which is the standby Avaya SBCE, handles the media of the active calls.

ASBCE 7.2 introduces dynamic licenses, in which licenses can be shared across ASBCEs. Session licenses that are actually in use will be tracked and unused session licenses will be released back to the shareable pool of licenses in WebLM. The number of dynamic licenses purchased must be the same as the number of Standard licenses. However, once purchased, all the license types (Standard, Advanced, Transcoding, Scopia) become dynamic.

**11. Although the Avaya Aura Media Server adoption of Communications Manager (AAMS-CM) provides many of the resource capabilities of the G-series Media Gateways (MG), it is not a direct replacement. Therefore, with Avaya Aura and Communications Manager (CM) 8.0, both AAMS-CM and MGs are sold as media resource options. AAMS-CM can support more channels per instance than a G series media gateway. For which requirement should AAMS-CM be deployed instead of a MG?**
- G722 codec used for ad hoc conferencing

**12. All Avaya Aura System Manager redundancy options utilize two servers. Although one server is designated as a primary and one server is designated as secondary, it is possible for them both to be active. Under which circumstances or deployment model would you expect to have both servers operate?**
- Operating with a split WAN ???
- For a geo-redundancy deployment model
- For a large deployment model with load balancing
- Operating with a high availability deployment model

**13. A customer has Avaya Aura components at release 6.3 and they would like to use Avaya Equinox clients to take advantage of Avaya Aura Device Services (AADS) in their deployment. What is Avaya Aura requirement for AADS?**
- Session Manager 6.3 or later
- Session Manager 7.0.1.2 or later
- Session Manager 7.1 or later
- Session Manager 7.0 or later ???

**14. In an Enterprise design, which two design requirements call for the installation of a G430 or G450 Gateway? (Choose two)**
- TDM trunks ??
- DSP resources ??
- SIP endpoints
- T.38 fax
- Sequenced Applications

**15. A customer is interested in taking advantage of Avaya Vantage. Which three components are required for a minimum level deployment? (Choose three)**
- **Session Manager (SM)**
- Avaya Aura Device Services (AADS)
- **System Manager (SMGR)**
- **Communication Manager (CM)**
- Presence Services (PS)
- Equinox Manager

*Подсказка из руководства по установке*:
The following components must be installed and configured in your network:
- Avaya Aura Session Manager
- Avaya Aura Communication Manager
- Avaya Aura System Manager
- Avaya Aura Presence Services
- Avaya Aura Session Border Controller
- Avaya Aura Device Services
- A DHCP server for providing dynamic IP addresses
- A file server, an HTTP, HTTPS, or the Avaya Aura Utility Services for downloading software distribution packages and the settings file
- One or both of the following conference servers for audio and video conference:\ - Avaya Aura Conferencing\ - Scopia Elite MCU

Avaya Vantage ™ requires Avaya Aura ® Release 7.0.1.2. For detailed information about supported product releases, see the Avaya Compatibility Matrix.

**16. With Avaya Aura 8.0 which two CS100 endpoints are supported for migration to Avaya Aura? (Choose two)**
- 9640
- **1120**
- **1140E**
- 3280
- I2033

*Подсказка*: Support for new CS 1000 endpoints
With the Release 8.0.1, Avaya Device Adapter Snap-in supports new CS 1000 endpoints in addition to CS1K-IP endpoints. The following are the supported new CS 1000 endpoints:

Set type Endpoints
**Unistim** IP 1110, 1120, 1140, 1150, 1165, 1210, 1220, 1230, 2001, 2002, 2004, and 2050 (softphone)
**Digital** 2006, 2008, 2216, 2616, 3110, 3310, 3820, 3901, 3902, 3903, 3904, and 3905
**Analog** 500, and 2500
Note: All expansion modules for the sets are supported.

Communication Manager supports several new endpoint types for CS 1000 endpoints. The following are the supported new CS 1000 endpoint types for Communication Manager:

Set type Endpoints
**CS1k-IP** For IP phones introduced in Avaya Device Adapter Snap-in and Communication Manager 8.0
**CS1k-39xx** For 39xx family of digital phones.
**CS1k-2col** For other digital phones with 2 columns of keys/buttons
**CS1k-1col** For other digital phones with 1 columns of keys/buttons
**CS1k-ana** For analog phones

**16.To provide CS1000 Mobile-X like feature experience, which change was made to EC500 with Avaya Aura 8.0?**

- You can now dial a feature access code without putting an ongoing call on hold.

*Подсказка*: EC500 in-call feature invocation
When you connect an EC500 mobile phone to another phone in Communication Manager, you can use the in-call features, such as hold and consult, transfer, and conference. To enable the in-call features, you must dial the feature access code that the administrator configures.

**17.Existing customers may be using old licenses including Standard Edition, Enterprise Edition, Foundation Suite, Mobility, Collaboration, analog, Basic IPT and Enhanced IPT. Which type of licenses can or cannot be mixed on the same Communication Manager instance**
- Core and Power with Foundation, Mobility and Collaboration suites can be mixed

*Подсказка*.
**Avaya Aura ® 7.1/7.1.1/7.1.2 Suites License commercial offer updates and changes.**

The primary Suites offer in Avaya Aura 7 is Suites Version 2 (V2) Core and Power. Also available are Analog and IPT (Basic and Enhanced) licensing (specific use) see Offer Definition Avaya Aura IPT Licensing Offer Definition

Customer system instances with active Upgrade Subscription, Upgrade Advantage, SS+U , Pass+/SRS, have the option to upgrade and land on the Avaya Aura 7 at Suites V1 Foundation as the \$0 subscription landing point or uplift at loyalty pricing to Suites V2 Core and Power. Transactional (paid) upgrades will go to Suites V2 Core and Power.

Adds will be in Suites V2 Core and Power only on Avaya Aura 7. *These can be mixed on a system that have Suites V1 Foundation resulting from a subscription upgrade*.

Suites V1 Mobility and Collaboration are not offered in Avaya Aura 7. SV2 Core and Power are the license op for this user segment and offer substantial value over Mobility and Collaboration Suites in both pricing and overall TCO. *As part of the upgrade to Avaya Aura 7 a customers existing Mobility and Collaboration suite licenses will automatically be converted to Core and Power respectively*.

To upgrade any product components to Avaya Aura 7, the Avaya Aura Suites Upgrade (Subscription or Paid Transaction) must be part of the order. See Compatibility table for individual product compatibility or minimum levels.

See Appendix E: for Upgrade Path Flowcharts

License Constructs / Transactions **not available** in Avaya Aura 7
- Legacy Standard Edition (SE) and Enterprise Edition (EE) Licenses;
    - As per existing suites offer definitions upgrades transition into the Avaya Aura® Suites
- Suites Version 1 (V1) Mobility and Collaboration Suite Licenses;
    - Replaced by Suites V2 Core and Power which offer more favorable value both pricing and TCO as maintenance and a la carte bundling with suite offer
    - Avaya Aura 6.x Mobility and Collaboration licenses will be converted to Core and Power respectively on upgrade to Avaya Aura 7
    - Suites Version 1 (SV1) Foundation Suites License for Add/Expansion\*;\ All Add/Expansions in Avaya Aura are in the SV2 Core/Power Constructs (Definition\*Add/Expansion: purchase of new licenses for an existing system instance. Not the same as a Move/Merge transaction)

Avaya Aura 7 Suites License Offer Summary

| Transaction Type                       | Definition                                                                                                     | Change from Avaya Aura R6.x Suites Offer?                                     | Avaya Aura 7.1/7.1.2 License Offer                                                                                                                                                                                                                                             |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| New                                    | New Licenses for New system instance purchase                                                                  | No                                                                            | -Suites V2 Core and Power Suite licenses<br><br>-Analog and IPT Basic and Enhanced (specific use) licenses Avaya Aura IPT Licensing Offer Definition                                                                                                                           |
| Add/Expansion (not same as Move/Merge) | New licenses purchased for an Existing system Instance                                                         | Yes<br><br>All Adds in Avaya Aura 7 are SV2 Core/Power                        | Suites V2 Core and Power Suite licenses<br><br>_Can be mixed on systems that have existing Suites V1 and mixed SV1/SV2 system instances._                                                                                                                                      |
| Upgrade: Transactional (Paid)          | Paid Upgrade from previous release to Avaya Aura  7                                                            | No                                                                            | -Suites V2 Core and Power Suite licenses<br><br>-Existing Suites V1 Mobility and Collaboration licenses convert automatically to Core/Power respectively.                                                                                                                      |
| Upgrade: Subscription                  | Subscription upgrade from previous release to Avaya Aura 7 with active Upgrade Advantage (UA), SS+U, Pass+/SRS | No` *`<br><br>Yes `**`                                                            | -Avaya Aura® 7 Suites V1 Foundation only; $0 subscription landing point `*`<br><br>-Avaya Aura® 7 Suites V2 Core and Power; paid uplift with loyalty pricing `*`<br><br>-Existing Suites V1 Mobility and Collaboration licenses convert automatically to Core/Power respectively`**` |
| Uplift                                 | Uplift license level to a higher one in release                                                                | No `*`<br><br>Yes `**` uplifts from Analog/IPT Basic/Enhanced to Core/Power only | -Avaya Aura SV2 Core to Power<br><br>-Avaya Aura ® 7 SV1 Foundation to SV2 Core/Power<br><br>-Analog to SV2 Core or Power`**`<br><br>-IPT Basic to IPT Enhanced, SV2 Core or Power`**`<br><br>-IPT Enhanced to SV2 Core or Power`** `                                                |


![](image284.png)

По лицензированию:

Avaya Aura provides Avaya Aura Suite Licensing V2 for Unified Communications (UC) applications. This suite provides:
- Simplified Unified Communications licensing for customers and channels.
- New products and capabilities in an easily scalable structure.

| Product                                                                                | Core Suite                                                                                                        | Power Suite                                                                                                        |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Communication Manager, System Manager, Session Manager, Survivability                  | Y                                                                                                                 | Y                                                                                                                  |
| Application Enablement Services Unified Desktop                                        | Y                                                                                                                 | Y                                                                                                                  |
| Avaya Breeze™                                                                          | Y<br><br>Concurrent user Right To Use                                                                             | Y<br><br>Concurrent user Right To Use                                                                              |
| Avaya Aura® Presence Services (Instant Messaging and Presence)                         | Y                                                                                                                 | Y                                                                                                                  |
| Avaya Multimedia Messaging                                                             | Basic                                                                                                             | Enhanced                                                                                                           |
| Voice Messaging (Avaya Aura® Messaging)                                                | Basic VM<br><br>Avaya Aura® Messaging - Basic license                                                             | Enhanced VM<br><br>Avaya Aura® Messaging - Mainstream license                                                      |
| Avaya Equinox™ - for Windows                                                           | Y                                                                                                                 | Y                                                                                                                  |
| Avaya Equinox™ for Skype for business                                                  | Y                                                                                                                 | Y                                                                                                                  |
| Avaya Equinox™ - for Android and iOS                                                   | Y                                                                                                                 | Y                                                                                                                  |
| Avaya Session Border Controller for Enterprise Remote Worker and SIP Trunking Sessions | One High Availability Remote Worker license and One High Availability SIP Session for every 7 Core Suite licenses | One High Availability Remote Worker license and One High Availability SIP Session for every 7 Power Suite licenses |
| AvayaLive Video                                                                        | Right to purchase one Video Meeting Room at a discount for every 25 Core Suite licenses                           | Right to purchase one Video Meeting Room at a discount for every 25 Power Suite licenses                           |
| Avaya Aura® Conferencing (Audio, Video and Web)                                        | Optional                                                                                                          | Y                                                                                                                  |
| Multidevice Access (MDA) for SIP Devices/users                                         | 10                                                                                                                | 10                                                                                                                 |
| Peer to Peer Video                                                                     | Y                                                                                                                 | Y                                                                                                                  |
| Extension to Cellular (EC500)                                                          | Y                                                                                                                 | Y                                                                                                                  |


The following table shows the mapping of features in the Communication Manager Messaging license file to features.

| **License feature**                                | **Communication Manager Messaging features**                                               |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| CM Messaging Offer (VALUE_CMM_OFFER)               | `EMBEDDED` allows up to 6000 mailboxes.<br><br>`FEDERAL_MARKET` allows up to 15,000 mailboxes. |
| Maximum CM Messaging Mailboxes (VALUE_CMM_MAILBOX) | Maps directly to the CM Messaging Mailboxes feature.                                       |

A specific feature in the Communication Manager license file may enable or map to multiple features on the Customer Options form. For example, ASAI Features (FEAT_CM_ASAI_PCKG) in the license file enables multiple ASAI-related features on the Customer Options form, including ASAI Link Core Capabilities and ASAI Link Plus Capabilities. Unlicensed features are available to all customers and are excluded from the license file.

The following table summarizes the mapping of features in the Communication Manager license file to Customer Option features.

| License feature                                                 | Communication Manager Customer Option features                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Edition (VALUE_CM_EDITION)                                      | Standard enables all unlicensed Customer Option features.<br><br>Enterprise maps to the Multinational Locations Customer Option feature. Also enables all unlicensed Customer Option features.                                                                                                                                                                                                                                                                                                                 |
| Maximum Stations (VALUE_CM_STA)                                 | Maps to multiple Customer Option features, notably Maximum Stations.                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Maximum Analog Stations (VALUE_CM_ANALOG)                       | Specifies the number of analog stations to which the customer is entitled.                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Maximum Survivable Processors (VALUE_CM_SP)                     | Maps directly to the Maximum Survivable Processors Customer Option feature.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Maximum ESS Stations (VALUE_CM_ESS_STA)                         | Specifies the number of Survivable Core station licenses to which the customer is entitled.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Maximum LSP Stations (VALUE_CM_LSP_STA)                         | Specifies the number of Survivable Remote station licenses to which the customer is entitled.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Maximum Mobility Enabled Stations (VALUE_CM_MOBILITY)           | Maps to multiple Off-PBX Telephones Customer Option features.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Maximum Video Capable IP Softphones (VALUE_CM_VC_IPSP)          | Maps to the Maximum Video Capable IP Softphones Customer Option feature.                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ASAI Features (FEAT_CM_ASAI_PCKG)                               | Maps to ASAI-related Customer Option features including ASAI Link Core Capabilities and ASAI Link Plus Capabilities.                                                                                                                                                                                                                                                                                                                                                                                           |
| Maximum Expanded Meet-Me Conference Ports (VALUE_CM_EMMC_PORTS) | Maps to the Maximum Number of Expanded Meet-Me Conference Ports Customer Option feature.                                                                                                                                                                                                                                                                                                                                                                                                                       |
| IP Endpoint Registration Features (for example, IP_Soft)        | Map directly to Customer Option features of the same name, for example, IP_Soft.                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Support End Date (VALUE_CM_SED)                                 | Specifies the Support End Date (SED) used for Avaya Service Pack and Dot Release Guardian.<br><br>If the Support End Date feature is available in the Communication Manager license file, the value is in DD-Month-YYYY format (for example, 01 June 2012).<br><br>If the Support End Date feature is not available in the license file, Communication Manager does not perform the Support End Date validations.<br><br>For more information, see the Service Pack and Dot Release Guardian overview section. |

The following table summarizes the mapping of features in the Call Center license file to Customer Option features.

| License feature                                                       | Call Center Customer Option features                                                                               |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Maximum Elite Agents (VALUE_CC_ELITE)                                 | Maps to multiple Customer Option features, most notably Logged-In ACD Agents.                                      |
| Maximum Advocate Agents (VALUE_CC_ADVOCATE)                           | Maps to multiple Customer Option features, most notably Logged-In Advocate Agents.                                 |
| Proprietary (FEAT_CC_PROPRIETARY)                                     | Maps directly to the Proprietary Customer Option feature (renamed from Agent States in Communication Manager 6.0). |
| Call Center IP Endpoint Registration Features (for example, IP_Agent) | Maps directly to Customer Option features of the same name, for example, IP_Agent.                                 |


**18. With the Avaya Aura R8 version 2 Suite License Offering Power license, which messaging entitlement is included?**
не нашла

**Q. Engagement Development Platform (EDP) 3.1 supports server clustering. Some rules include up to 35 EDP servers limit and 5 snap-ins sequenced per user. In addition to these what other rules need to be followed?**
- Load Balancing of EDP nodes requires all nodes in the same LAN subnet network.

**Q. The Engagement Development Platform (EDP) is a deployment model based on Avaya Aura 7\'s Aura Presence Service. For the deployment of Presence Services in Avaya Aura 7, which is the correct description?**
- Presence Services always requires more than one special EDP.

**Q. For an Avaya Equinox solution which two components are required? (Choose two)**
- Avaya Equinox Media Server
- Avaya Equinox Management Server

**Q. For an existing customer with existing Elite 6000 MCUs what might be a reason to add an Avaya Equinox? Media Server? (Choose two.)**
- To add WebRTC Gateway functionality
- To add Web Collaboration functionality

**Q. A customer has 9650 users. You need to use "nine (9) user-per-session model" for SIP trunks. You hope that at a time, 10% of users can use the mobile function. What is the optimal number of ASBCE security visits in the design?**
- 1073 Standard Sessions with 107 Advanced Sessions
- 1073 Standard Sessions with 965 Advanced Sessions
- 2038 Standard Sessions with 204 Advanced Sessions
- **2038 Standard Sessions with 965 Advanced Sessions**

**Q. In addition to requiring Named User permission for each user, what must the EDP solution follow in the presence or absence of Avaya Aura Media Server (AAMS)? (Please choose two options)**
- Only one master node in each cluster requires a Server license
- **If AAMS is HA deployment, and then both AAMS and EDP are HA**
- If AAMS is HA deployment, and then AAMS or EDP can be HA
- AAMS and EDP can be used by all snap-in or other Avaya solutions
- **CapEx-oriented Named User License and OpEx-oriented Named User cannot be mixed in the same solution**

**Q. The Engagement Development Platform (EDP) offers a 90-day trial option. This allows the client/partner to take advantage of this platform on non-production systems. In addition to 50 named user licenses, what are there in the trial package?**
**1 EDP server license, 1 Avaya Aura Media Server license**
1 EDP server license with HA, 1 Avaya Aura Media Server license with HA
1 EDP server license, 1 Avaya Aura Media Server license, plus an optional Avaya Snap-in
1 EDP server license, 1 Avaya Aura Media Server license, plus Engagement Designer optional Snap-in

**Q. You are designing for a customer with 11,000SIP users. Assume that the user has two devices registered at the same time and need redundancy. What is the minimum configuration to meet this requirement?**
- 1 Session Manager
- **2 Session Managers**
- 3 Session Managers
- 4 Session Managers

**Q. Customers need Microsoft Exchange as a back-end voice message. Customers do not require solutions for geographic redundancy, but require hardware redundancy. The customer has 8,000 users. Assuming that there are 60 users per port, at least how many Avaya Aura Messaging Application Servers will be required to meet the requirements?**
- 2 Application Servers
- **3 Application Servers**
- 4 Application Servers
- 5 Application Servers

**Q. You are in the process of sizing an ASBCE for a customer location. They are using a 7 user per SIP session model for sizing the solution. There are 5000 users at the location. They expect 40% of the users to be mobile at any given time.Following best practice, how many Standard Sessions and how may Advanced Sessions will be needed to satisfy the customers\' requirements?**
A. 714 Standard Sessions with 2000 Advanced Sessions ???
B. 2714 Standard Sessions with 2000 Advanced Sessions
C. 2000 Standard Sessions with 2714 Advanced Sessions

Готовые проверенные ответы:
**1. For the Unified Communications market, which is the correct description of the current market trends? (Please Choose Two)**
- *All companies hope to provide a solution through a vendor (тоже вроде как правильный, опционально)*
- **Many companies are eager to unite B2P and B2C** (business to partner, business to customer)
- UC solution is to attract some specific small users
- Use the solution with integrated management and management driver
- The reduction in the number of communication devices is important to the user experience
- **All device types (mobile, desktop, BYOD) are intended to be integrated into UC applications**

**2. What are the three aspects of the enterprise's Team Engagement requirements and solutions? (Please Select Three)**
- **The realization of growth**
- **Employee and team productivity**
- **Communication optimization**
- Implementation of call center
- Optimization of Fabric Networking
- Network virtualization

**3. Which of the following are the strategic features of Avaya reduce the total owing cost and the offering value for the customers? (Please Choose Two)**
- **Reduce the number of hardware via virtualization**
- **Interoperability between manufacturers**
- Complex server cluster
- «rip and replace»
- Patent standards

**4. Which of the following is used to highlight Avaya's market value? (Please Select Two)**
- **People-oriented innovation**
- Closed single supplier
- Independent core, application and terminal
- Distributed deployment and management
- **Focus on high reliability, feature-rich real-time communication**

**5. Avaya Aura Messaging supports a variety of high reliability and disaster recovery options.**
**Which of the following servers can support N+1 configurations?**
- Storage server
- Message store
- **Application server** (сервер приложений)
- Avaya X Connector (AxC)

**6. A user is more concerned about the new voicemail power failure resolution mechanism, because they have encountered voicemail system is not available. This user network has two sets of PBX, the average distribution of users in two systems. Users are interested in the function of Avaya Aura Messaging (AAM). From a design point of view, which Avaya Aura Messaging (AAM) deployment architecture can better meet the needs of users to avoid failure?**
- Each branch's PBX connected to the same application server and the same centralized storage server
- **Each branch is a cluster of two application servers, and then connected to the same centralized storage server**
- The same site deploy a cluster of two application servers and a centralized storage server
- Each branch of an application server connected to the storage server cluster

**7. You recommend a single server version Avaya Aura Messaging (AAM) solution to customers, the customers want to know what to do if it had exceeded the system capacity limit or if it would like to decide to deploy Unified Messaging in the future. You tell the user can migrate the user license of a single server version to multi-server version of the AAM, then how should deal with the existing hardware server?**
- Cannot be used again
- **Can be used for application servers or storage servers** (существующей сервер для серверов приложений и хранилища)
- Can be used as a single server version of the backup
- Can be used as a one-x Speech server

**8. Avaya Aura Messaging (AAM) 6.3 Deployment only allows the Exchange as the storage destination for the AAM system. In view of this deployment, which description is correct? (Please Select Three)**
- **AxC connector can also be application roles on the server**
- AxC connector must be running on a standalone server
- **Occupy at least one server**
- Occupy at least two servers
- **AxC connector is responsible for distributing the message**
- AxC connector can also be information storage roles on the server

**9. Your customers believe that when employees are telecommuting, getting their status information can improve productivity. Their current communication system is provided by several suppliers. Assuming you are preparing for the next meeting, you have carefully studied Avaya Aura Presence Services. Which of the following is correct description of Presence Services? (Please Select Two)**
- **In this solution, Avaya Aura System Manager is required to configure the state system.**
- Avaya Communicator for Android cannot use state services in Avaya Aura System Manager 7.0
- **SIP users of CM can use the instant messaging and status service functions, but the users of CM 323 cannot**
- Presence Services release 7.0 is compatible with Microsoft OCS/Lync

**10. With the popularity of computers, many users have shifted from mobile-centric to more computer-centric. In response to this change, Avaya has a soft client. But it is not just a computer-based soft terminal, more attention to collaboration and display a variety of information. In addition to providing intelligent online status information, what function the Presence Services can provide also?**
- Visual messaging
- Synchronized call logs
- **Instant messaging**
- Unified contacts

**11. You are designing a scenario that requires Avaya Aura and Microsoft systems to share internal status. Which of the following products must be included in your program? (Please Select Three)**
- **Avaya Aura System Manager**
- Avaya Aura Application Enablement Services (AES)
- **Avaya Aura Presence Services**
- **Microsoft OCS/Lync**
- Avaya Agile Communication Environment (ACE)
- Avaya Session Border Controller for Enterprise

**12. The Engagement Development Platform (EDP) is a deployment model based on Avaya Aura 7's Aura Presence Service. For the deployment of Presence Services in Avaya Aura 7, which is the correct description?**
- **Presence Services always requires more than one special EDP**
- Presence Services does not require special EDP
- Presence Services requires more than one universal EDP
- Presence Services can only be installed on a special EDP instance

**13. Why does the Avaya Aura Presence Service Connector snap-in be deployed on a common EDP or cluster?**
- **To provide information about the status of the connection to the EDP application development interface**
- To provide information about the status of the unique EDP
- To provide an interface to connect to the Session Manager
- To provide status information of Avaya Communicator for Windows

**14. What are the standard protocols for Avaya Aura Presence Services? (Please Select Two)**
Вопрос может звучать так: **Which two standard protocols are used by Avaya Aura Presence Services?**
- *MSRP (тоже стандартный протокол, но достаточно выбрать два)*
- PRIM
- **SIMPLE**
- **XMPP**
- 323

*Подсказка*:
Avaya Aura Presence Services work in concert with other presence-based applications, including Microsoft Office Communication Server, IBM Lotus Sametime, and other third-party applications using open standards SIMPLE (Session Initiation Protocol for Instant Messaging and Presence Leveraging Extensions) and XMPP (Extensible Messaging and Presence Protocol). This allows consistent presence visibility and the use of a wide array of business communications applications including Avaya one-X clients and deskphones to provide fully aggregated presence capabilities. Full aggregated presence is provided in Avaya one-X UC clients and Avaya one-X deskphones.

Более того: Protocols, such SIP/SIMPLE, XMPP and REST. These protocols enable Presence Services to aggregate and federate presence with major IM and messaging solutions and a number of user-productivity tools.

**15. Your customer is interested in the Avaya Aura Session Manager's Branch SIP trunk function, which provides SIP trunks as a shared source with local self-surviving capabilities. You need to explain what happens to the branch when the core network fails. Which of the following statements should be reflected in your report? (Please Select Two)**
- **To the control of the dry branch of the SIP phone, from the core of Session Manager move to the branch of Session Manager**
- Outbound call of the branch, before routing to the SIP Service Provider, through the LSP and branch gateway first
- Local Survivable Processor (LSP) directly connects to the SIP Service Provider
- Branch of the gateway should be connected to the SIP Service Provider
- **The branch gateway is connected to the local surviving server LSP and the LSP is the call controller**

## 3171T старые вопросы
**1. In a typical Avaya Aura 6.2 environment. Session Manager provides the call admission control functionality. But with the addition of Scopia based collaboration, Scopia Management (iView) provides that functionality for the Scopia environment. In addition to call admission control, another function it provides is bandwidth management. For what type of calls can Scopia Management manage or limit the bandwidth used?**

A. For only SIP internal calls
**B. For only SIP external calls**
C. For any SIP call
**D. For only H.323 internal calls**
E. For only H.323 external calls
F. For any H.323 call
G. For any SIP or H.323 call

**2. In connection with a CCT implementation project for the Government of Mourito, Avaya has partnered with a leading Distributor in the country. Avaya is required to import certain telecom equipment into Mourito. Avaya arranges for the shipment and same reaches Mourito port. In order to release the shipment, a no-objection letter is required from the customs unit in charge of the port. This is standard operating procedure in Mourito vis-a-vis overseas shipments. Typically, it takes about 7-14 working days to receive the letter. The Distributor, citing project exigency, pays a sum of \$150 to a senior customs official and obtains the NOC. What prompts the Distributor to make the payment is that facilitation payments are customary and legal in Mourito. What breach, if any, has the Distributor committed?**

A. None, the payment made by the Distributor constituted facilitation payment which is customary under the laws of Mourito
**B. The Distributor has breached Avaya\'s policy since Avaya prohibits facilitation payments**

**3. In addition to the Scopia@ Gateway for Microsoft@ Lync and the Elite MCU, there are other Scopia products that could be included in a Microsoft Lync and OCS2 environment. This includes evident and Scopia PathFinder. In this type of Microsoft environment, where would the customer use the PathFinder for firewall traversal?**
A. Customer\'s partners with Lync endpoints
B. Customer\'s partners with H.323 endpoints
**C. Customer's Lync endpoints**
D. Customer\'s partners with SIP endpoints

**4. The XT Executive 240 supports additional features not available on the VC240 it replaced. One of those features is an additional HDMI display. What is a characteristic or requirement of that feature?**
A. It requires an optional license.
B. It requires the optional camera.
**C. One display shows video and one display shows content.**
D. Both displays must show the same image.

**5. Avaya is bidding for a telecommunications project with the Government of India (\"GOI\") through one of its Partners. The GOI official who is in charge of the tender requests that the Partner arrange a site visit to Avaya premises to check out our facilities. This is part of the bidding process which authorizes the concerned department to undertake a capability study of all the bidders. The Partner and Avaya take the three (3) member GOI team on a tour of Avaya facilities and conduct a demo of our core offerings. At the end of the demo. Partner and Avaya serve refreshments i.e. tea/coffee and biscuits for the GOI team. Have the Partner and Avaya conducted themselves in compliance with Avaya policy?**

**A. Yes, because there was nothing wrong with facilitating the above since it was arranged pursuant to a legitimate government process (of reviewing bidder capabilities).**
B. No, because by hosting government officials, both the Partner and Avaya attempted to influence the government to secure a favorable response bid response.

**6. Using the red numbers in the Avaya Aura@ 6.2 & Scopia@ Interoperability diagram, determine the correct signaling flow for an Scopia Desktop client to join a meet-me videoconference call on an Elite 5000 MCU. (Use the magnifying glass icon to enlarge the diagram.)**

![](image285.png)

A. 3 → 13 → 15
B. 3→ 5→ 6 →13 →15
**C. 3→6 →13→15**
D. 3→6 →15
E. 3 →5→13→15

**7. Using the red numbers in the Avaya Scopia@ Video Conferencing diagram, with a standalone ECS gatekeeper, determine the correct signaling flow for an XTE240 to join a meet-me videoconference call on an Elite 6000 MCU. {Use the magnifying glass icon to enlarge the diagram.)**

![](image286.png)

A. 14 → 7
B. 14 →12→5→7
C. 14→12→9→7
**D. 14→9→7**
E. 14→6→7

**8. You are helping Don learn about the Scopia@ Elite MCU. He wants to know when you need MCU cascading. What are two reasons for MCU cascading? (Choose 2)**

A. To maintain the conference when a master MCU fails
**B. To save bandwidth when several participants are near each location**
C. To support meetings that exceed the maximum, single MCU capacity
D. To connect telepresence systems from multiple manufacturers
E. To allow a mix of telepresence users and standard room system users

.....


**1. You are proposing videoconferencing for a customer with 15 large meeting rooms, 25 small meeting rooms, and 4000 employees dispersed over three continents: North America, Asia, and Europe. Thirty percent of the workforce will be video-enabled, and you are proposing XT5000s for the large meeting rooms and XT4200 for the small meeting rooms. Using the normal 1:10 ratio for simultaneous rooms and users, how many ports (including cascading) and Elite 5000 MCUs should be included in the design?**

A. 440 352p ports or 4 Elite 5230 MCUs
B. 280 352p ports or 2 Elite 5230 MCUs
**C. 152 352p ports or 3 Elite 5115 MCUs**
D. 140 352p ports or 4 Elite 5110 MCUs
E. 136 352p ports or 3 Elite 5110 MCUs

**2. Your customer, Jay, is reviewing your proposal for Scopia® video conferencing. He notices that within Scopia Management, there is a SIP Back-to-Back User Agent and an internal gatekeeper that could be external. When would you tell him he would use an external gatekeeper instead of an internal gatekeeper?**

A. In order to work with an external Microsoft SQL database
B. When running Scopia Management (iView) on a Linux server
C. To support configurations with multiple cascaded Elite MCUs
**D. To support Scopia Management (iView) redundancy**

**3. Your customer is concerned about the ease of use for the infrequent video collaboration user. You explain that your solution includes Scopia' Control. What is Scopia Control?**

**A. An iPad app for conference control.**
B. An Android mobile device app for conference control.
C. An Android mobile device app for configuring the user\'s virtual room.
D. An iPad app for configuring the user's virtual room.

**4. You are meeting with your Account Team and discussing a small SMB customer. You\'re hesitant to select the Scopia® SMB solution with the MCU embedded in the XT1200, because it has some differences from a configuration with an Elite MCU and Scopia Management (iView). Select three capabilities the SMB solution does not support that you would discuss with the Account Team. (Choose 3)**

A. Support for Scopia Mobile users
B. Support for internal Scopia Desktop Client users
**C. Support recording and streaming of conferences**
D. Support for encryption of conferences over 4
**E. Support for external Scopia Desktop Client users**
**F. Support multiple concurrent conferences**

**5. For users who operate out of the office, Scopia® offers desktop client and mobile applications. Your friend Oliver, another SE, calls to ask you about a statement in the Scopia marketing materials that says that Scopia is the best meet-me client because it is more than an endpoint. Although there are many reasons, what two would you want to tell Oliver about? (Choose 2)**

A. Error resiliency for both the desktop and mobile clients uses SVC (scalable video coding) and Netsense
B. Users can download the presentation using the slider feature
C. User features such as chat, FECC (far end camera control), and raise hand
**D. Best user experience with calendar integration and one tap to join**
**E. Simple and secure firewall traversal using HTTPS (hypertext transfer protocol secure)**

**6. What is the built-in feature of Scopia® Management (iView) that allows the creation of multiple organizations with separate administrators?**

A. Administrator redundancy
B. Cost center segmentation
C. Shared system
**D. Multi-tenant**

**7. Your IT contact, Jessie, calls you with some questions about the evident solution your team has proposed. She remembers you talking about the ability for a third-party application to control the evident solution. She wants to know what protocol or method must the third party application support in order to control evident. What protocol would you tell Jessie is required?**

A. XMPP
B. SNMP manager/agent
C. JTAPI
**D. TCP/XML**

**8. Oliver, another SE, calls to ask you about licensing for Scopia Mobile and Desktop clients. In the Avaya Solution Designer, he sees both a desktop license and a mobile license. Since most of his remote users will be using both a desktop PC and a mobile device, he wants to know if they will need both licenses. You assure him that he will not need both of these licenses for every remote user. Select the correct statements about desktop and mobile licenses. (Choose one for each license type: desktop and mobile)**

A. A Mobile license is required for every Mobile user with a virtual room.
B. A Mobile license is required for every Desktop PC, laptop PC, and Mobile user with the client installed.
**C. A Mobile license is required for every concurrent user on a Mobile device.**
D. A Desktop license is required for every Desktop PC or laptop PC user.
E. A Desktop license is required for every Desktop PC, laptop PC, and Mobile user with the client installed.
**F. A Desktop license is required for every concurrent Desktop PC connection in the Desktop Server.**

**9. Your customer Andy calls with some questions on the centralized recording aspect of your Scopia proposal. He likes the ability of the recording function because it records audio, video, and presentation or content. He saw in your proposal a cost for the number of concurrent recording sessions, but no hardware. Where would you tell Andy these recordings are being stored?**

A. In the Scopia XT Desktop Server.
B. In the Scopia Management server.
**C. In the Scopia Desktop Server.**
D. In the Scopia Elite MCU.

**10. You have proposed Scopia® PathFinder to a customer. Your contact, Quinn, remembers you only discussed the PathFinder server. Quinn had called one of your reference accounts and the person mentioned a PathFinder client. Why did you not mention the PathFinder client to Quinn.**

A. It is not required for customers who also deploy Desktop Server.
B. It is a Java-based runtime app that installs automatically if required.
**C. All remote H.323 endpoints can now work directly with PathFinder Server.**
D. The PathFinder client is now a software app that is free to install on all remote endpoints.

**11. Conference, the initial admission request goes to Scopia Management. For each type of call, SIP or H.323, which one of the four main components within Scopia Management (core, gatekeeper, back-to-back user agent, internal database) handles the admission request? (Choose one answer for each call type: SIP and H.323)**

A. For H.323 based calls, the internal database handles admission requests.
B. For H.323 based calls, the core handles admission requests.
C. For H.323 based calls, the back to back user agent handles admission requests.
**D. For H.323 based calls, the gatekeeper handles admission requests.**
E. For SIP based calls, the gatekeeper handles admission requests.
F. For SIP based calls, the internal database handles admission requests.
G. For SIP based calls, the core handles admission requests.
**H. For SIP based calls, the back to back user agent handles admission requests.**

**12. You are helping Don learn about the Scopia® Elite MCU. He wants to know when you need MCU cascading. What are two reasons for MCU cascading? (Choose 2)**

A. To maintain the conference when a master MCU fails
**B. To save bandwidth when several participants are near each location**
C. To support meetings that exceed the maximum, single MCU capacity
D. To connect telepresence systems from multiple manufacturers
E. To allow a mix of telepresence users and standard room system users

**13. One of the members of your account team, Karen, read that the Scopia® Elite MCU supports an Auto Attendant function. She is confused about why an MCU needs an AutoVideo Attendant feature. In addition to answering the call, what would you tell her that this feature does?**

A. provides options for the user\'s video layout
**B. provides a menu of the current meetings that the user can join.**
C. greets or welcomes the user
D. provides a «waiting room» prior to the moderator joining the call.

**14. During a meeting on the Scopia® Elite MCU, the users see a user-definable video layout. At the top of the layout are icons and indications. Which of the following are three indications or icons the user can see?(Choose 3)**

A. Encrypted conference
B. Audio volume level
**C. Number of audio only participants**
D. Bandwidth being utilized by each participant
**E. List of meeting participants**
**F. Recording notification**

**15. Your customer, Joe, was reading about Scopia® endpoints and calls you to ask several questions. He knows that one of the unique features of Scopia XT5000 is 1080p60 dual video. But he does not understand what the function of dual video is. What would you tell him dual video allows?**

A. Allows the location to see more of the participants using a second display when the number exceeds the capacity of a single display
**B. Allow the HD content to be on one display and HD video of the users on a second**
C. Allows a second display that can be used as the user\'s PC screen
D. Allows the HD content to contain motion that can span across two displays
E. Allows a single display to show both the HD video of the users and the HD presentation or content

**16. Your customer is concerned about getting a consistent video equipment deployment in the conference room in each of their multiple offices globally. They are not interested in immersive telepresence. You are considering the Scopia® XT Meeting Center. Select the statement that correctly describes the XT Meeting Center.**

A. It is a complete, single display, self-contained videoconferencing solution for the SMB market.
B. It is a single display endpoint based on the XT5000 that can be placed on the desktop in any conference room.
C. It is a kit based on the XT5000 that includes assembly instructions and sources to buy the displays and cart.
**D. It is a single or dual display endpoint based on the XT5000 that is pre-installed in a cart/display stand.**

Correct Answer: D

**17. Pat went to a local user\'s group meeting and heard that Scopia® offers a family of serial and ISDN gateways, but she is still confused about the purpose or function of these gateways. What is the basic function of the ISDN gateway?**

**A. It allows H.323 to H.320 connectivity near the MCU location.**
B. It adds H.460 support to H.323 endpoints for firewall traversal over ISDN.
C. It allows H.323 to H.324 connectivity near the MCU location.
D. It allows H.323 to SIP connectivity near the MCU location.

**18. You are helping another SE, Fran, learn about Scopia® PathFinder. She wants to know when a design might include multiple PathFinder Servers. You start by telling her that if you want redundancy or load balancing, you always have to have multiple servers or appliances. In addition to redundancy, what are two configurations when multiple PathFinder Servers would be required? (Choose 2)**

**A. When you want provide firewall traversal for both H.323 and SIP endpoints**
B. When there are more than 90 concurrent calls or over 499 registered devices
C. When you have large sub-branches or geographic dispersion and you want to go to the Internet directly
**D. When there are more than 100 concurrent calls or 600 registered devices**
E. When you want to support Scopia® Mobile or Desktop Clients

**19. Your customer, Steve, has a LyncTM 2010 environment and they have been using it for point to point videoconferencing. They want to be able to have multipoint videoconferences and Steve wants to know about any differences between using the Lync A/V MCU and the Scopia® Elite MCU. You tell him that adding the Scopia Gateway for Microsoft® Lync is required to use the Elite MCU, but their existing standard Microsoft licensing with point-to-point video should be sufficient. What are two other differences in the user experience between the A/V MCU and the Elite MCU? (Choose 2)**

**A. 1080p HD resolution is only available with the Elite MCU**
B. 1080p HD resolution is only available with the Lync A/V MCU
C. Continuous Presence of 28 endpoints is only available using the Lync A/V MCU
**D. Continuous Presence of 28 endpoints is only available using the Elite MCU**
E. With content sharing, Lync clients can simultaneously see both the content or data and the video with the Elite MCU
F. With content sharing, Lync clients can simultaneously see both the content or data and the video with the Lync A/V MCU

**20. You are proposing video conferencing for a customer based in the United States. They want to have 3 small meeting rooms in their offices located in San Francisco, Dallas, and Chicago, and five percent of their 1000 employees video-enabled. They plan to have no more than 3 simultaneous conferences with 6 to 8 maximum participants. The customer does not need encryption, but wants high profile and SVC (scalable video coding). Your initial design used a Scopia® Elite 6110 MCU, Desktop Server and clients, and XT4200s for each meeting room. The Account Manager wants to know if there is a way to reduce the design cost and simplify the solution. Which alternative solution would meet the customer's requirements?**

A. Three XTSOOOs with MCU9
B. Three XT4200S with SMB9
**C. Three XT4200s with MCU9**
D. Three XT1200S with SMB9
E. Three XT1200s with MCU9
F. Three XT5000S with SMB9

**21. You are hoping to sell Scopia® products to a healthcare provider who has a small amount of video deployed in their network. However, the IT Manager says that he is having support problems with his existing conferences and is not going to expand his video deployment until he is comfortable supporting what he has currently. From his perspective, video quality is a very subjective trait. So his issue is around knowing the magnitude or severity of the quality issues being reported and isolating the trouble. What might be good quality to one person might be poor quality to another. What solution would you suggest he deploy?**

A. PreVideo and simulate the video traffic and verify the network supports it.
B. Scopia Management (iView) since it can work with several manufacturers\' endpoints.
C. VQInsider and analyze captures done with a packet sniffer program (like Wireshark).
**D. RVMon to capture real-time information of each video conference call.**

**22. Hans, the IT Manager at one of your high-tech customers, calls to tell you that they acquired another company. This new acquisition has both Tandberg and Cisco telepresence rooms. Although they would eventually want to reduce the variety of systems, Hans would initially like to have it just integrated with the existing Polycom® and LifeSize® telepresence and the Scopia® endpoints and infrastructure you have proposed. So in order to connect to an Elite MCU meet-me conference with full functionality, which of the following telepresence systems will require a TIP Gateway?**

**A. Cisco telepresence**
B. LifeSize telepresence
C. Tandberg telepresence
D. Polycom telepresence

**23. High Profile (HP) and Scalable Video Coding (SVC) are both part of the H.264 standard. Unlike many competitors, the XT4200 and XT5000 support both SVC and High profile. Select the correct statements about the functionality of SVC and High Profile. (Choose one statement for SVC and one for HP)**

A. SVC improves resilience against network jitter.
B. SVC provides bandwidth savings for a high resolution video call.
**C. SVC improves resilience against network packet loss.**
D. SVC improves resilience against network delay, and HP improves resilience against network jitter.
**E. HP provides bandwidth savings for a high resolution video call.**
F. HP improves resilience against network packet loss.
G. HP improves resilience against network delay.

**24. The capacity required for a Scopia® Management (iView) system with internal gatekeeper is based on three factors. In addition to the number of MCU ports, the number of  \_\_\_\_\_\_\_\_\_\_  and \_\_\_\_\_\_\_\_\_\_ are two other factors on which the capacity is based? (Select two answers that correctly fill in the blanks. Choose 2)**

A. gatekeeper registrations
**B. redundant servers**
**C. point to point ports**
D. virtual rooms in the database
E. organizations defined with the multi-tenant feature

**25. The Scopia® Collaboration Suite solution includes both Guest and Named endpoint licenses. When is the Guest license required?**

**A. For H.323 based room system endpoints only**
B. For SIP based room system endpoints only
C. For any desktop endpoints
D. For room system endpoints not having a named license

**26. Roman is creating a Scopia® design with four geographically dispersed MCUs. His customer wants the ability to connect all of them together for larger conferences. How many ports would be required for cascading?**

A. Three 720p ports
B. Four 720p ports
**C. Eight 352p ports**
D. Twelve 352p ports
E. Twenty-four 352p ports

**27. In addition to centralized recording of conferences the Scopia® solution allows an XT endpoint to record conferences to a USB key. Using the USB recording option, which of the following is a characteristic or requirement?**

**A. If MCU9 or SMB9 is used it requires 2 MCU ports**
B. Can be only be played back on an XT endpoint
C. Conferences must be encrypted to be recorded
D. Recorded conferences are in an MP3 format

**28. The Scopia® 8.3 release changed the number of endpoints supported by the SMB9 option. In addition to the one local endpoint, what is the maximum number of remote endpoints that the SMB9 option with the 8.3 and later release supports?**

A. 7 remote endpoints or 7 mixed remote endpoint, PC and mobile clients
B. 7 remote endpoints or 8 mixed remote endpoint, PC and mobile clients
C. 8 remote endpoints or 7 mixed remote endpoint, PC and mobile clients
**D. 8 remote endpoints or 8 mixed remote endpoint, PC and mobile clients**

**29. The 8.3 version of Scopia® Management included several improvements to reporting. In addition to the GUI-based interface and adjustable colors, there were also some new reports added. Two new reports are the top endpoints and top MCU. What criteria is used to determine the top MCUs?**

A. size of the meeting
B. number of meetings
**C. greatest availability or lease downtime**
D. minutes of usage

**30. Support Advantage and the Avaya Video Support Services support include some of the same coverages. For Support Advantage, possible parts and onsite support coverages include:
Which of these coverages is available with Avaya Video Support Services?**

A. Advanced Parts Replacement next business day and onsite support 8x5 next business day
B. Advanced Parts Replacement next business day and onsite support 24 x 7
**C. Advanced Parts Replacement 24 x 7 x 4 and onsite support 8x5 next business day**
D. Advanced Parts Replacement 24 x 7 x 4 and onsite support 24 x 7 next business day

**31. Avaya is trying to get payments worth \$100,000 released from a public sector client in connection with an implementation project Avaya delivered successfully three (3) months back. The Channel Partner through whom Avaya bid for the project is the primary interface conducting regular follow-ups with the client. The client contact promises to get the payment released within 45 days provided the Partner or Avaya pay up $150 to expedite release. The Avaya channel account manager encourages Partner to do «whatever it takes» to get the money. What should the Partner do?**

A. The Partner should ask Avaya to pay the money to get the payments released.
B. The Partner should negotiate the proposed \"sum\" with the client contact and try and agree on a sum of not more than \$60 which could be deemed reasonable.
**C. The Partner should immediately report the matter by using Avaya Ethics Hotline or emailing compliance@avaya.com**
D. The Partner should report the matter to their own management.

**32. Pursuant to opening a new branch office in an emerging market in South Asia, the distributor («Distributor») engaged by Avaya comes in contact with a leading businessman in the country who claims to have sufficient contacts within the non-government space and offers to promote Avaya and our offerings in the country. For the above purpose, the businessman demands a cash payment of \$1000 which the Distributor pays on behalf of Avaya without seeking Avaya's express approval. Is the Distributor's conduct appropriate?**

A. Yes, since these are typically facilitation \"grease\" payments aimed to speed up things in the country and are acceptable as exceptions under certain anti-bribery/anti-corruption laws in various countries
B. No, the Distributor has violated our Avaya\'s policies since he did not receive Avaya\'s approval before effecting the payment
**C. No, because the Distributor violated Avaya\'s policies on anti-bribery/anti-corruption because he did not conduct any due diligence on the businessman and without Avaya\'s knowledge engaged in conduct designed to improperly influence a commercial customer**

**33. Avaya has set up a branch office in an emerging market in South Asia. The company engages a channel partner (\"Partner\" or \"Channel Partner\") who has contacts within the government to promote and sell its products and services in the above market. In connection with the above, the Partner pays two (2) government officials \$250 each. The Partner takes the position that these payments were not made to secure any government business but rather to build relationships to position Avaya\'s business in the market. Is the Partner potentially in breach of anti-bribery/anti-corruption laws and regulations and Avaya\'s Anti-Bribery/Anti-Corruption policy?**

A. Yes, because anti-bribery/anti-corruption statutes and Avaya policy prohibit the improper influencing of... ТУТ ПОТЕРЯЛОСЬ