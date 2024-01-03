---
title: Exam Avaya Breeze 3175T
share: "true"
tags:
  - exam
---

Вопросы и ответы. Подсказки, найденный в документации.

**1.Avaya Breeze is Avaya\'s new Collaboration Environment and Engagement Platform (EDP). What statement correctly describes Avaya Breeze?**
- A platform for Avaya, partners and enterprises to build and deploy collaboration applications.

*Подсказка*: Avaya Breeze is a development and execution platform for creating rich, omni-channel real-time communications applications.

**2.What component provides SIP load balancing for multiple nodes in an Avaya Breese high availability installation?**
- Avaya Aura Session Manager

*Подсказка*: *For SIP traffic coming into Avaya Breeze, Session Manager serves as the load balancer*.

SIP high availability is possible with Avaya Breeze by administering a cluster of Avaya Breeze servers and connecting each member of the cluster to each Session Manager.

You can then route users of a service to the cluster rather than to a specific Avaya Breeze server so it is likely that at least one will be available.

For Avaya Breeze the HTTP Load Balancer "routes" traffic to the Avaya Breeze servers in the cluster. It then follows a round-robin distribution method. If the Avaya Breeze server is offline then the Avaya Breeze server sits in an inactive pool for 15 seconds and then requests are sent to a different Avaya Breeze server in the cluster.

Primary and Secondary Balancers heartbeat each other. In case of a Primary server outage, the secondary server assumes usage of the Cluster IP and begins balancing the HTTP traffic. Once it is back online, the Primary Avaya Breeze server resumes control. In the event of a Primary server failure, a MAJOR alarm is raised.

**3.Which three Avaya Breeze Snap-in scenarios require an AAMS? (choose three)**
- When and application needs to play an announcement
- When 2-Party Make Call (Call-me / Call-you) is included in application
- When there is a need to support Text to Speech

*Подсказка*: The Avaya Aura Media Server delivers advanced multimedia processing features to a broad range of products and applications. One design concern is do we need to add an Avaya Aura Media Server to the configuration for a solution. So when is an Avaya Aura Media Server required for an Avaya Breeze Snap-in or application?

*Here are some obvious scenarios:*
Then and application needs to play an announcement to one or both parties in a call, when there is a need too collect DTMF digits, when there is a need to support Real-Time Speech, when there is a need to support Text to Speech, which, by the way, requires Nuance, or when there is a need to support Automatic Speech Recognition, which also requires Nuance.

*And here are some less obvious scenarios:*
Then the WebRTC Snap-in is used, when 2-Party Make Call (Call-me / Call-you) is included in application, when Sequential Forking (Find-me / follow-me) is used in the application, and when Flexible Call Leg Control (Drop one participant, add another) is used in an application.

**4.In addition to each user requiring a named user license, which design rule applies to an Avaya Breeze solution with or without an Avaya Aura Media Server (AAMS)?**
- For a high availability deployment with AAMS, both Breeze and AAMS must be high availability.

*Подсказка*:
Based on the size of the customer and the requirement for high availability, to support the WebRTC, Engagement Designer, Call Blocker and Smart Caller ID Outbound Snap-ins we need the following:
2 Avaya Breeze virtualized instances in an HA configuration.
2 AMS virtualized instances in an HA configuration
We will place the additional Avaya Breeze and AMS instances in Venice.

**5.In the era of digital transformation, which two statements correctly describe how Avaya Breeze provides customer\'s high-touch, personal business interactions? (choose two)**
- Supports a multi-channel development environment
- Supports context aware, historical experience

*Подсказка*: This next era is the digital transformation era. All of our customers should have a digital transformation strategy that we are a part of. In every vertical business, there are initiatives to make sure they are relevant in this era, and provide innovation in their space because there is so much disruption happening with traditional verticals exploiting new digital technologies. Just look at Uber or Airbnb as examples.

This era is line of business focused and about driving contextual experiences and outcome, hard dollar Return on Investment (ROI) and Key Performance Indicators (KPIs). With the diverse nature of our customer base, for them to drive the necessary business results in this digital era, it requires openness with technology that we have like Avaya Breeze.

The capabilities from the unified communications era and the mobile era are still required in the digital transformation era, and we have a superior solution at whatever stage your customer is in. And our solution is ready now for your customer's transformation. We are defining the future of UC. We are making it natural and a part of how we communicate every day on our device of choice, within a browser or embedded in the applications we live in; *Simple, transparent, in context and user defined*.

...The Avaya Engagement Designer Snap-in allows individuals with limited developer experience to easily integrate Avaya Breeze and Snap-in functionality on a single canvas.

The Graphical User Interface or GUI allows coding without any Java knowledge. *This results in the ability to quickly deploy high value, multi-channel use cases*. The resulting workflow can then be saved or stored in memory for days, weeks, or months.

**6.Avaya Breeze offers a free trial. This allows a customer/partner to use Avaya Breeze on a non-production system. Which statement correctly describes the free Avaya Breeze trial offer?**
- The Avaya Breeze Trial offer includes 50 named user licenses and is good for 90 days.

*Подсказка*: Avaya Breeze R3.4 will be Generally Available (GA) effective 11 Dec 2017 and the material code numbering and price is the same as EDP/Breeze 3.x.

Avaya Breeze R3.x is sold as an a la carte option to any of the suites (Core, Power, Foundation, Mobility, and Collaboration) along with the line-up of Avaya Aura applications.

*A Trial option is available and provides the ability to utilize this capability on a non-production system free of charge for 90 calendar days.*

The Avaya Breeze User Licenses are included in the Avaya Aura Core and Power Suite License. Additional User Licenses can be ordered if needed.

The Media Server Bundle should only be ordered if required. The Avaya Aura Media Server section describes scenarios where you might include this in your order. Avaya Aura Media Server is optional.

Nuance Text to Speech (TTS) and Automatic Speech Recognition (ASR) is supported with Avaya Breeze and the section below includes Nuance ordering process and material codes.

А также номер для заказа бесплатной лицензии:

***307286** BREEZE R3 TRIAL NON PRODUCTION LIC:DS,NU (50 Users, 1 Breeze Instance, and 1 AAMS Instance that will expire in 90 days) \$0.00*

**7.Which method or methods support serial calls and serial forking of calls by allowing one participant to be replaced with a different participant on the same call?**
- Flexible Call Leg Control Method

*Подсказка*: When the WebRTC Snap-in is used, when 2-Party Make Call (Call-me / Call-you) is included in application, when Sequential Forking (Find-me / follow-me) is used in the application, and when *Flexible Call Leg Control (Drop one participant, add another)* is used in an application.

**8.Which two statements correctly describe current market trends in the Unified Communications market? (choose two)**
- There is increasing interest in mobility, collaboration, video.
- There is increasing interest in integration of communications and collaboration functionality with business processes and applications

*Подсказка*:

![](image281.png)

IDC sees growth of the overall UC&C market being especially driven by organizations' increasing interest in mobility, collaboration, video, and the integration of communications and collaboration functionalities with their business processes and applications. (IDC — Worldwide Mobile Unified Communications and Collaboration, (UC&C) Forecast, 2016—2020, September 2016.)

**9.An Avaya Aura customer wants to add Click-to-call to their web site. The customer has: 1) three survivable remote sites supporting SIP endpoints 2) Presence deployed on Avaya Breeze servers instances in a high availability configuration at their main site and 3) Avaya Aura Conferencing (AAC) servers and Avaya Aura Media Servers (AAMS) supporting AAC at their main site. What is needed to integrate Click-to-Call from their website?**
- They need to install additional Avaya Breeze instances and add the WebRTC Snap-in to the new Avaya Breeze instances.
- They need to install additional Avaya Breeze instances and add the Smart Caller ID Outbound Snap-in to the new Avaya Breeze instances. No AAMS instances are needed.
- **They need to add the WebRTC Snap-in to the existing Avaya Breeze instances and utilize the existing AAMS servers.** (не уверена)
- They need to install additional Avaya Breeze instances and add the WebRTC Snap-in to the new Avaya Breeze instances. They also need to install new AAMS instances.

**Подсказка*:*
The Snap-in takes advantage of WebRTC to deliver a mobile experience across a wide array of iOS, Android™, and WebRTC-supported browsers. It integrates "click-to-call" or video chat directly back to agents on Avaya Aura ® Call Center Elite using Avaya one-X Agent. Developers can build services that tap into rich context for intelligent routing and personalized real-time engagement with Avaya Breeze Context Store and Work Assignment Snap-ins.

WebRTC adds real-time communications to any website making customer contact and support easier and giving those without immediate access to a telephone another way to connect. It allows Click-to-Call. WebRTC is available as a Snap-in on Avaya Breeze
- Support for external 'Click to Call' applications from a website with an audio path.
- Inclusive in the integration is relevant payload with the call that can be used for contextual treatment by Context Store Snap-in.

With Avaya Breeze 3.2 we will offer a WebRTC Debug license that will allow an Engagement Designer Workflow to be tested with a single WebRTC call. This license is for non-production use and needs to be selected during the Avaya Once Source configuration. Until Jan-2017 please contact productops@avaya.com to activate this WebRTC Debug license.

**10.In Avaya Aura environment, Avaya Breeze applications can function as sequenced applications. Sequenced applications are invoked by which Avaya Aura Component?**
- Session Manager

*Подсказка*: With Sequenced Applications calls can be influenced before alerting the called party.

Logic can be inserted. The calling or called party IDs can be manipulated. We can play messages to individual participants or the entire call.

*Sequenced Applications are invoked by Avaya Aura Session Manager* on behalf of the originator of a SIP session and or on behalf of the terminator of a SIP session.

This diagram gives a simple illustration of the flow of a sequenced application.

The originating user initiates a call. The originating user could be a local user or a user coming in on a PSTN trunk. Session Manger routes the calling and called party ID to Communication Manager for processing. Communication Manger sends information to Session Manger indicating that the call needs to go to Avaya Breeze for additional processing. The Avaya Breeze applications adds it treatment to the call and returns it to Session Manager for termination processing to the Termination point. As these are SIP endpoints Direct Media is used for communications between the endpoints.

This same flow can apply to incoming or outgoing calls.

**11.What is the deployment choice for Avaya Breeze 3.2 when not using Avaya Aura Presence Services?**
- VMWare virtualized application

*Подсказка*: The Avaya Breeze Server license entitles a customer to 1 instance of a virtualized Avaya Breeze Server instance.

*The server will only run on a customer's provided VMware environment. There are no offers to provide a hardware appliance for general Avaya Breeze use*. Avaya Aura 7.x Presence Services is offered on the Application Virtualization Platform (AVP) and leverages Avaya Breeze as its infrastructure. Presence Services on AVP uses a predefined VMware footprint which does not support the VCenter Agent at this time and cannot be used got general Avaya Breeze usage.

**12.Which Avaya Engagement Designer component is installed as a Snap-in on an Avaya Breeze platform?**
- Workflow Engine

*Подсказка*: Engagement Designer needs to be ordered as a separate line item from Avaya Breeze along with any other snap-ins purchased. Avaya Engagement Designer can be delivered along with Avaya Breeze and any other snap-ins ordered by the customer. Let's talk a little about components of the Engagement Designer:
- *Workflow Engine* is installed as a snap-in on Avaya Breeze. It subscribes to Avaya Breeze events and executes the Workflow Instances for deployed Workflow Definitions.
- *Workflow Definitions* describe business processes used in your enterprise. You use Workflow Definitions to create complex, end-to-end communications-enabled business processes across mobile, collaboration and contact center applications.
- *Workflow Instance* is created each time a Workflow Definition is invoked by a triggering event.

Events that trigger a particular Workflow Definition are specified in the Start event.

**13.Where is information stored to complete the action of the failed server in the event of a cluster node failure in an N+M Redundant Breeze solution?**
- Data Grid

*Подсказка*: If any node in the cluster goes down the *data grid* contains information from that node and complete the actions that of that node. Each Avaya Breeze Cluster is N+M Redundant which supports multiple Avaya Breeze failures. Avaya Breeze takes over for each other by re-using the media paths for existing sessions, preserving existing call connections, sending INVITE is replaced to re-create the signaling paths, and Calls In-Queue for a contact center are preserved.

**14.Which statement correctly describes a Snap-in?**
- Applications, connectors and modular reusable code that can quickly create or integrate new capabilities into processes and applications.

*Подсказка*: A Snap-in is an Avaya Breeze application or connector extension that expands Avaya Breeze capabilities.

**15.Which Statement correctly describes writing or developing Avaya Breeze Applications**
- They can be written in Java or deployed using Avaya Engagement Designer.

*Подсказка*: Snap-ins may include such things as: complete out of the box communications applications, modular and re-usable APIs and SDKs for developers to mix and match in order to build their own unique applications. Snap-ins may also include media and application connectors.

*Developers can create their own Snap-in applications in Java or non-Developer Business Analysts can use Avaya Engagement Designer workflow design capabilities to build communications enabled services and applications.*

To reduce development time and cost, applications can be developed by "snapping in" applications, connectors or code from Avaya or 3 rd party developers.

**16.Which Snap-ins support Avaya Breeze geo-redundancy?**
- Context Store and Presence Snap-ins

*Подсказка*: From the standpoint of geo-redundancy for Snap-ins, currently only Context Store and Presence today support HA and Geo-Redundancy. They use the GigaSpaces WAN Gateway which synchronizes data between two data centers.

**17.Which three tools are available through the Avaya Breeze Software Development Kit (SDK)? (choose three)**
- Java application program interfaces (API)
- Sample Services
- Eclipse Integrated Development Environment (IDE) Integration.

*Подсказка*:
Avaya Breeze includes a Software Development Kit (SDK) for Java developers to create their own collaboration snap-ins to run on the Avaya Breeze ® server. Any Java programmer can build, test and deploy a custom snap-in. No specialized telecommunications expertise is needed.

The Developer SDK, located on Avaya DevConnect, provides a rich set of tools to aid in the developing of an Avaya Breeze application.
- There is Eclipse integrated development environment (IDE) Integration and there is the Engagement Designer Snap-in which enables business analysts within the enterprise the ability to map out and create complex workflows.
- There are Java APIs which include APIs for the various Connectors such as E-mail, SMS and Conferencing. There are Javadocs for each API.
- There are Code examples for Whitelisting, HelloWorld, Multi-channel broadcast and dynamic team formation.

The Avaya Breeze ® SDK has all the documentation on our Java APIs, developer guides such as how to's, video tutorials, FAQs, discussion forums, and Admin Guides on how to set up the server.

**18.Which license includes an Avaya Breeze user license?**
- Avaya Aura Core and Power Suite license

*Подсказка*: 12.1.8. Avaya Breeze User Licenses

The Avaya Aura Suite License was enhanced by adding an Avaya Breeze user license in the suite license. This will, Avaya Breeze *enable every Avaya Aura Core and Power suite license user*. If additional Avaya Breeze user licenses are required a \$0 Avaya Breeze User can be ordered.

**19.Which two statements correctly distinguish Avaya from its competitors in the marketplace?**
- Focus on highly-reliable, feature-rich real-time communications
- People-centric innovation

*Подсказка*:

Today's user experience reality is still inconsistent and highly disjointed. The existing environment in enterprises is full of multi-vendor, fragmented tools and solutions, and monolithic systems, with long cycles for innovation and change. Customers are asking for:
- Consolidated solution efficiency plus the best-of-breed flexibility,
- Solutions that can deliver a seamless engagement experience, one that is tailored to an organizations unique business processes, and also can leverage knowledge of prior and in-progress interactions to drive proactive engagement treatments and flows
- And finally, the customer is asking for solutions that can keep up with the speed of business, and not take months to implement changes and new interaction flows, but days.... and at most weeks

Avaya is addressing these needs by providing unbundled infrastructure and applications, software defined engagement approach, and new and innovative sources for easy, rapid customization.

Avaya is unbundling and providing both the stable enterprise class infrastructure and the flexible platform for user-centric best-of-breed applications from Avaya or third parties.

Avaya Aura core will focus on scale, security, resiliency

*New capabilities will be customizable, user-centric, innovative and rapidly developed:*
- By Avaya and third parties allowing user choice of experience
- Enabled by software defined engagement through Avaya Breeze, Avaya Client SDK and Zang
- Avaya clients will let users collaborate right in the applications they use every day
- Avaya will provide a turn-key experience alternative
- Analytics will provide proactive and pre-emptive engagement across the enterprise
- Avaya will focus on collection across the customer, team and the platform
- Leverage 3-rd party solutions for analysis and presentation

Zang will provide a new Communication Platform as a Service (CPaaS) channel to market and a source of easy, rapid customization:
- cloud-only, pay-as-you-go source of infinitely customizable applications
- CPaaS and APIs as a Service
- OTT stand-alone applications -- such as persistent group collaboration Zang Spaces

**20.Which three statements are strength of Avaya in the application development market? (choose three)**
- Avaya has on an premise and a cloud based offer for application development and deployment
- Avaya applications are flexible and open
- Avaya offers full depth and breadth of real-time communications functionality.

**21. Which statement correctly describes the relationship between Avaya Breeze, Avaya Aura, Avaya Aura Media Server.**
Avaya Breeze can share AAMS instances with Avaya Aura Application ???

**22. Which two sources of data arrive at the Start Event in an Avaya Engagement Designer Workflow? (choose two)**
- An event administered in the Event Catalog
- A website or REST client

*Подсказка*:
The data mapper manages the flow of data from the Start event through the different tasks of the Workflow Definition. The data arrives at the Start event from:
- An event administered in the Event Catalog
- An event created by the completion of another WFD
- A process created by a Create Process task
- Or a website or REST client

Start events have an associated schema that describes the available data.

**23.Where is the Avaya Breeze Presence Services Snap-in installed?**
- On the Core Platform Cluster

*Подсказка*:
Clusters contain one or more nodes. You can have multiple instances of various cluster types. You can have from 1 to 35 Avaya Breeze servers and 1 to 5 Avaya Breeze nodes per cluster.

*Avaya Presence services requires a dedicated Core Platform cluster* which cannot be shared by other applications.

When Presence Services is deployed on its closed Core Platform cluster we have engineered this specific configuration to support up to 8 nodes in a cluster given its very controlled and well defined usage model.

**24.Which Avaya Aura component must be upgraded to release 7.0.1 to support Avaya Breeze 3.2?**
- System Manager

**25.Which statement correctly describes the Avaya Breeze Engagement Call Control (ECC) Snap-in?**
- ECC in an Avaya Breeze entitlement and requires Application Enablement Services (AES).

Вопрос с подковыркой, т.к. в будущем релизе Breeze (но непонятно каком, а в тексте ниже речь о 3.4) обещают опцию прямого соединения с CM без AES. Поэтому при сдаче экзамена хорошо бы проверить Product Offer Definition and Business Partner Proposition, если вдруг будут вопросы по новым релизам.

*Подсказка*:
At this time the Engagement Call Control (ECC) Snap-in is an entitlement the Avaya Breeze offer.

The Engagement Call Control (ECC) Snap-in provides REST APIs for CTI style call control. The ECC Snap-in provides functional equivalency to the ACE SOAP Web Services 3 rd Party Call Control. *ECC uses a protocol called Application Switch Adjunct Interface (ASAI) through the Application Enablement Services (AES) to hook into Communication Manager call processing. In a future release ECC will provide an option to use ASAI directly to communicate to CM without requiring AES*. ECC has control and visibility of all CM calls including station to station, station to trunk, or trunk to station. With the RESTful Engagement Call Control snap-in you will be able to perform the following functions in Avaya Breeze 3.1:
- Make a call, cancel a call, answer a call, end a call
- Consult transfer, single step transfer
- Hold or Retrieve a call
- Set or cancel call forwarding
- Voice mail connector to Avaya Aura Messaging

*The ECC Snap-in uses the call model provided by UCM (Unified Collaboration Model) and uses the DMCC (Device, Media and Call Control) client to communicate to AES (Application Enablement Services) through the Call Server Connector (CSC*).

**26. Scopia Connector requires integration with which release of Avaya Scopia?**
- Scopia 8.3

*Подсказка*. Scopia Connector
- Allows services to send API requests to the Scopia Management Server (by using the Create Conference API)
- Requires integration to Scopia Solution 8.3.

**27. Which Snap-ins can use Persistence Database or Cluster Database?**
- Any Avaya or Avaya Presence Services (APS) developed Snap-in
(проверить вариант Any 3rd Party Snap-in, Avaya Engagement Designer Snapin or Avaya Presence Service Snap-in)

*Подсказка*: Persistence Database (*only available to select Avaya Snap-ins*)

Avaya Breeze will introduce a database (DB) that can be used to store data for the cluster. Every cluster has one node designated as the Primary. All data access to snap-ins is provided from the Primary node. In multi-node clusters, the persistent DB is highly available (HA) because one other node that is not the primary node is designated as the standby node. The primary asynchronously streams transactions to the standby so that the standby has an in sync copy of the DB. If the primary fails or is taken down, the standby can take over as the primary quickly with very minimal data loss.

For release 3.1, the database can mirror GigaSpaces data grids, thus enabling select Avaya snap-ins that use GigaSpaces to benefit from persistent storage. *The Engagement Designer and Presence Services snap-in can use it as a general purpose DB*.

**28. How many designer licenses does the Avaya Engagement Designer basic offer include?**
1

Engagement Designer needs to be ordered as a separate line item from Avaya Breeze along with any other snap-ins purchased. Avaya Engagement Designer can be delivered along with Avaya Breeze and any other snap-ins ordered by the customer.

**29. How many concurrent workflow instances does Avaya Engagement Designer Workflow Engine Support?**
- 4200

Подсказка из Avaya Engagement Designer Reference Release 3.5 Issue 1 August 2018:
Chapter 7: Performance
Capacities
• 4200 Workflow Instance (WFI) limit per cluster
• 100 Workflow Definition (WFD) limit per cluster

Memory allocation
Avaya Engagement Designer Release 3.5 has a minimum required memory allocation of 8 GB.
Avaya Breeze disk size
The minimum Avaya Breeze server disk size for Avaya Engagement Designer is 100 GB. In your VM machine properties, increase the provisioned size of the hard disk to 100 GB. See your VM documentation for additional information. Alternatively, if you are deploying Avaya Breeze using the Solution Deployment Manager (SDM), select profile 2b. This will eliminate the need to increase the disk size.

For additional information, see the Hardware requirements section of this document.

**30.Which Avaya Breeze Snap-in provides Click-to-Call capabilities from a web page?**
- WebRTC Snap-in

*Подсказка*:
Purchasable snap-ins include:
• Avaya Context Store Snap-in
• Avaya Work Assignment Snap-in
• Avaya WebRTC Snap-in
• Avaya Real-Time Speech Snap-in

3.1.10. WebRTC Snap-in
WebRTC Snap-in allows consumers 'click to call' capabilities from any supported browser by connecting an audio path along with the associated web page information to an agent supported off of Avaya Breeze.

**31.Which configuration requires STUN or TURN servers with the Avaya WebRTC Snap-in?**
- NATs are used под вопросом (ответила так)

(проверить Browsers do not have routable IP addresses)

*Подсказка*.
STUN server address. The following IP address and port based on the enterprise network:
- If you use Avaya SBCE, the Avaya SBCE IP addresses and port.
- If web browsers are located outside the enterprise network, the external IP address.
- If web browsers are located within the enterprise network, the private IP address.
- If there are web browsers that are located outside and within the enterprise network, the external IP address.

The enterprise network must be connected to the external IP address of Avaya SBCE. The format of the IP addresses and ports that you enter must be address:port. Use comma (,) as a delimiter. The default STUN port is 3478. The default port must be accessible in the firewall.

!!! note
    Avaya SBCE configured as a TURN/STUN server does not support NAT servers for WebRTC. To mitigate this lack of support for NAT servers, the external interface of Avaya SBCE must be configured with a public IP address.
    The TURN server relay address must be configured on the external interface of Avaya SBCE, which is connected to the external firewall of the enterprise DMZ. The external firewall provides Layer 3 security to the TURN relay address.

The enterprise gateway must be configured to send all data packets through the external firewall of DMZ. These data packets reach the Avaya SBCE external interface which is visible to public networks.
WebRTC Snap-in does not support hiding the external interface IP address of Avaya SBCE from public networks.

Сервер STUN (Simple Traversal of User Datagram Protocol \[UDP-протокол пользовательских датаграмм\] через сервер NAT \[транслятор сетевых адресов\]) позволяет клиентам NAT (т.e. компьютерам за сетевым экраном) устанавливать сеансы связи с провайдером VOIP, находящимся за пределами локальной сети.

![STUN-сервер](image282.png)
Сервер STUN позволяет клиентам находить свой адрес общего доступа, тип NAT, за которым они находятся и порт Интернета, связываемый NAT с конкретным локальным портом. Эта информация используется для настройки связи UDP между клиентом и провайдером VOIP и организации сеанса. Протокол STUN определяется стандартом RFC 3489.

Соединение с сервером STUN устанавливается через UDP-порт 3478, однако сервер предлагает клиентам выполнить проверку также и альтернативного IP и номера порта (у серверов STUN есть два IP-адреса). В RFC говорится, что выбор порта и IP является произвольным.

Что такое STUN и зачем он нужен?

Так случилось, что очень много пользователей сидят в интернете за NAT (Network Address Translation). Например, наш компьютер в локальной сети имеет IP-адрес 192.168.1.100. Адресов диапазона 192.168.x.x в Интернете быть не может (он зарезервирован для локальных сетей), и такой пакет к отправителю не вернется. Когда пакет из локалки уходит в интернет, NAT (обычно это часть функционала шлюза в интернет) подменяет в нем адрес отправителя на внешний (публичный) адрес NATа, например 1.2.3.4. Когда получатель пакета получит его и решит отправить ответ, то он пошлет его на внешний адрес NATа. А внутри себя NAT заменит адрес обратно на 192.168.1.100 и дошлет пакет компьютеру в локалку.

Чтобы знать, какой пакет какому компьютеру внутри прикрытой NATом сети предназначен, NAT подменяет еще и номер порта, и помещает его в таблицу соответствий номеров портов и внутренних адресов.

Вроде бы все работает прозрачно для сидящих за NATом компьютеров. Проблема возникает тогда, когда компьютеры сначала договариваются о передаче данных по одному протоколу, сообщая друг другу свои IP-адреса, через которые пойдет обмен (конечно, NAT про протоколы не знает, ему важны номера портов), а потом начинается собственно передача данных. Именно так происходит установление соединения в VoIP - договариваются хосты о соединении по SIP, сообщая друг другу адреса и номера портов для голосовых RTP-потоков по SDP, а обмен идет уже по RTP.

Компьютер с адресом 192.168.1.100 посылает VoIP-шлюзу во внешней сети свой адрес и порт, допустим, 192.168.1.100:40000. Шлюз после установления соединения начинает отправлять RTP поток\... куда? Адрес-то левый! В итоге абонент в телефонной сети со стороны шлюза слышит голос из локалки, а его - не слышат. Типичный случай \"one-way-voice\".

Поэтому компьютер в локальной сети должен сначала узнать свой внешний адрес и номер порта. Для этого в интернете существуют STUN-серверы. Упрощенно это выглядит так: компьютер из локалки посылает STUN-серверу пакет, тот его получает и отправляет обратно, запихнув внутрь адрес и номер порта, с которых он их получил. Дальше предпринимаются еще несколько действий для выяснения типа NATa, но это уже не так важно и хорошо описано в статье по ссылке выше.

Теперь дело техники: компьютер в локалке передаст в SDP не свой локальный адрес, а полученный через STUN. Путь пакетам уже «пробит» запросом к STUN-серверу (т.е. NAT установил соответствие локальный хост : порт ←→ внешний порт), поэтому входящие извне пакеты попадут в локалку и слышимость будет обоюдная.

**32.Which two are the feature of the Avaya Engagement Assistant Snap-in? (choose two)**
- Conference Assistant
- Seamless Transfer

*Подсказка*: The Avaya Breeze Suite will also support Avaya Aura 6.x and 7.x and will require the latest Avaya Breeze and System Manager versions as described in section 7.3.2 Interoperability.
384191 (new) (Breeze GP, AAMS, RTS, CB, EA) \$9300
384197 (add)

Avaya Breeze Suite includes the Avaya Breeze core software and 25 Snap in entitlements:
Breeze General Purpose (GP) server instance
- Avaya Aura Media Server instance
- (5) Real Time Speech entitlements
- (10) Co Browse
- (10) Engagement Assistant

Engagement Assistant requires two Breeze instances to incorporate the speech engine, an additional Avaya Breeze Suite or an a la carte Breeze instance is required

3.1.4. Engagement Assistant Snap-in
Engagement Assistant Snap-in takes the headache out of manually entering conference numbers and long bridge passcodes. Engagement Assistant provides users with "one-number conferencing" for all meetings on their Microsoft Outlook calendar. It consults a user's Outlook calendar, helps select which meeting to join and *seamlessly connects* them to the desired audio or video conference. Mobile business people in particular will appreciate the ease and flexibility to connect to important meetings without the pain of manually entering conference numbers and passcodes! Engagement Assistant integrates with Microsoft Outlook calendaring and supports Avaya Aura Conferencing, Avaya Scopia, and other external conference bridge systems.
...

The Engagement Assistant Snap-in has two components:
- The Conference Assistant provides an easy, seamless way for conference callers to access their calendar and connect to an audio conference using voice means only.
- Seamless Transfer allows users of Scopia Desktop Mobile to seamlessly transition conferences between desktop and mobile utilizing Plantronics Voyager Legend UC Headset.

**33.Which two enhancements does the Avaya Call Park and Page Snap-in offer in an Avaya Aura Solution? (choose two) ?????**
- Scheduled Park and Page ???
- Complex user retrieval rules
- **Network call Park and Retrieval**
- **Simplified user retrieval нет явного указания в доке**
- Ability to page individuals

*Подсказки*

Call Park and Page Snap-in provides the same Avaya Call Park and Page functionality provided by CS1000 using Avaya Aura core (CM, SM etc) via Avaya Breeze . While CM contains call park and retrieval functions (park retrieval on CM is often referred to as "Answer Back"), operation is slightly different from CS1000. This Snap-in will allow the current base of CS1000 customers to embrace Avaya Aura and retain the valuable park and paging operation while moving users or locations onto Avaya Aura as part of the CS1000 modernization effort.
Doc ID: FAQ110942

Breeze: Is there a Breeze snap-in available that can emulate CS1000 Call Park and Page functionality to enable migrations to Communication Manager?

Yes, Avaya Call Park and Page Snap-in for Breeze provides enhanced call park capabilities for Avaya Aura Platform users and provides a familiar user experience for Avaya Communication Server 1000 users transitioning to the Avaya Aura Platform. Call Park and Page allows an operator to put incoming calls on hold in a queue while a page is sent to the needed person to call and pick up the call from another phone. Call Park and Page leverages existing paging systems for the operator to page out the park directory number, as well as existing audio source (music, announcements) to optionally provide audio to the parked caller.

The Call Park and Page Snap-in is a telephony feature on Engagement Development Platform.

With the Call Park and Page Snap-in:
- An attendant who parks the call from an attendant station or any other endpoint, can park a call to a fixed or dynamic Park Extension number.
- A paged retrieving party, for whom the intended call is intended, can dial the Park Extension number to connect with the caller.
- A paged retrieving party can dial the Park Extension number from any number within or outside the network.

After the attendant successfully parks a call, their endpoint displays the Park Extension number at which the system parks the call. The attendant who parked the call must then use existing notification or paging services to page the called party. The page informs the called party that a call is waiting at the Park Extension number. While the call is parked, the caller hears music or audio, if configured. The paged retrieving party can retrieve the call by dialing the Park Extension number from any number within or outside the network.

If the paged retrieving party does not respond within a configured time, the system sends the call back to the number that parked the call. You can then repark the call or press the asterisk key (\*) after answering the call, and connect to the parked caller.

Call Park and Page architecture
The Call Park and Page Snap-in obtains installation, update, and patching procedures from the Engagement Development Platform cluster. To upgrade, the customer deploys a new version of the Call Park and Page Snap-in.

Then the attendant dials a pilot number before parking a call, Avaya Aura Media Server plays an announcement. After the call is parked, the media resources, which provide music on hold, reside on Avaya Aura ® Media Server and are controlled by the Call Park and Page Snap-in.

The Call Park and Page Snap-in can share Avaya Aura Media Server resources with other snap-ins in the General Purpose, General Purpose Large, or Core Platform cluster.

However, the Call Park and Page Snap-in cannot share Avaya Aura Media Server resources with Communication Manager. Avaya Aura 7.0 onwards, Communication Manager can take advantage of Avaya Aura Media Server for the same purposes that it currently uses H.248 Media Gateways.

**34. Which infrastructure for call routing is used by the Avaya call park and page Snap-in?**
Avaya Aura infrastructure.

**35. Which two revenue sources are part of the Avaya Snapp Store revenue model?**
- Revenue from Avaya Snap-ins
- Revenue from 3rd party Snap-ins commissions

*Подсказка*:
The Avaya Snapp Store is a focused, ecommerce-enabled marketplace for Avaya Breeze applications or Snap-ins from Avaya and 3 rd party developers. Snap-ins are easily consumable, prebuilt connectors, fit-for-purpose applications and or developer code that enable companies to even further accelerate creation of innovation game changing business processes, customer journeys and other unique applications.

The Avaya Snapp Store makes it easy to find, access and download Snap-ins by simply using a credit card for purchase. Developers can create Snap-ins and be on-boarded in the store within weeks.

At launch 5 key partners have been highlighted that have been working with Avaya to make Snapp Store a reality. The first five partners with Snap-ins in Snapp Store today are Approved Contact, WebText, eGain, Moxtra, Verbio.

**36.Which statement correctly describes the Avaya Aura Web Gateway?**
- The Avaya Aura Web Gateway acts as a gateway to Avaya Aura for clients and applications utilizing WebRTC signaling and media

[Подсказка](https://www.devconnectprogram.com/site/global/products_resources/avaya_client_sdk/programming_docs/current/introduction/index.gsp)
The Avaya Aura Web Gateway provides an active-active and highly scale-able set of capabilities comprise of a Endpoint Service Gateway and Web Portal.
- The Endpoint Service Gateway (ESG) provides the abstraction of the Avaya infrastructure for developers creating web based / JavaScript applications. This abstraction includes the conversion of WebRTC signaling to SIP which is required by the Avaya infrastructure. When acting as a WebRTC Signaling Gateway the Endpoint Service Gateway controls Audio and Video calls and provides a full range of call / conferencing controls. In addition Presence is mediated through the gateway.
- A Web Portal host the Avaya Equinox Conferencing, Meetings for Web client. This portal functionality is utilized by applications created by Avaya and is not accessible to third party developers

The Avaya Aura Media Server provide the Adaption and transcoding of WebRTC Media.

**37.Which Avaya Breeze Software Development Kit (SDK) package provides access to Avaya Breeze Co-browse Snap-in?**
- Sharing Services

*Подсказка*.
[Список всех сервисов и назначения](https://www.devconnectprogram.com/site/global/products_resources/avaya_client_sdk/programming_docs/current/introduction/index.gsp):
**Customer Interaction Utilities** — Provides additional Customer Interaction functionality: eMail services.

**Customer Interaction Services** — Agent, Supervisor, Work/Interactions and Team Services
[Avaya Client SDK](https://www.devconnectprogram.com/site/global/products_resources/avaya_client_sdk/programming_docs/current/javascript/customer_interaction/index.gsp)

**Communication Services** (For IP Office deployments available from Client SDK 4.1) ---Calls ( Audio, Video ), Signaling Features (MWI, feature buttons, \...), Conferencing, Collaboration, Messaging Service

**Sharing Services** — Sharing Web Pages, Co Browse. Provides a set of consolidated services for sharing a webpage session. Using Javascript Sharing Services Package, two users can simultaneously browse the same webpages to collaborate and accomplish certain tasks. The agent can assist the customer to navigate through the webpages and filling the forms.

Co-browsing Snap-in enables developers to build customer engagement solutions *that allow users to share and browse web page content simultaneously* to collaborate and accomplish tasks. Built to enrich customers-agent web and mobile interactions, the Snap-in was purpose built as a common capability for integration with existing Avaya multichannel contact center solutions like Call Center Elite Multichannel, Interaction Center, and Avaya Aura Contact Center.

Avaya Co-Browsing Snap-in leverages the Document Object Model (DOM), which is an application programming interface (API) for valid HTML documents.

Avaya Co-Browsing Snap-in runs on Avaya Breeze, and you do not need to install any additional software or plug-in to use the snap-in.

Avaya Co-Browsing Snap-in provides the following functionality:
- A standard REST Web Service API to provide access to the Avaya Co-Browsing Snap-in services.
- For more information about APIs, see Avaya Co-Browsing Snap-in Developer and API Reference Guide at http://support.avaya.com.
- A developer SDK, including a sample reference client that provides co-browsing capabilities.
- Out-of-the box summary reports about agents, sessions, and customers.

Avaya Co-Browsing Snap-in utilizes Avaya Breeze ™ to provide all security configurations to access all Avaya Breeze ™ services. Avaya Breeze ™ provides configuration for HTTPS, Mutual TLS (Client Certificate Challenge), Cross Origin Resource Sharing (CORS), Whitelists, and Trust Certificates. In addition, System Manager provides a flexible platform for administering certificates and authorities. For more information about the security configuration, see the Avaya Breeze and System Manager product documentation.

**38. Which Avaya Breeze Snap-in performs CTI control of Avaya Endpoints for Applications developed using Avaya Breeze SDK?**
- Avaya Breeze Unified Collaboration Model Snap-in

*Подсказка*:
The following diagram shows the additional functionality available through the Breeze Client SDK when the Avaya Oceana Omni-Channel Contact Center infrastructure is also deployed. These Oceana Omni-Channel capabilities are developed on the Avaya Breeze platform utilizing several Breeze Snap-ins and are exposed to the developer community through the Avaya Breeze Client SDK JavaScript API\'s. *The Avaya Breeze Unified Collaboration Model (UCM) Snap-in utilizes a Computer Telephony Integration (CTI) frame work to control of Avaya endpoints.*

The solution can avail of browser based WebRTC media by addition of the Avaya Aura Web Gateway (AAWG). In this solution the browser using http registers to Aura via the AAWG as a SIP endpoint. The Agent or supervisor application controls this SIP endpoint using UCM and CTI.

**39. Which three items does the Avaya Breeze Client SDK include? (choose three)**
API reference
SDK libraries
Sample applications

**40.With Avaya Breeze Client SDK which statement correctly defines a logical grouping of functionality that can be used independently or in combination?**
Packages

Еще варианты вопросов

**Q. Which release of Avaya Aura Session Manager supports applications written using the Avaya Breeze Client 3.0 Software Development Kit (SDK) interaction with Avaya Aura**
- 7.0.1 or higher

*Подсказка*: [матрица совместимости](https://secureservices.avaya.com/compatibility-matrix/menus/product.xhtml?name=Breeze+Client+SDK&version=3.0) Client SDK 3.0 
[Все матрицы](https://secureservices.avaya.com/compatibility-matrix/menus/product.xhtml) совместимости ПО: 

**Q. When a customer deploys an application or client which they or a third party developed using the Avaya Breeze Software Development Kit (SDK), which licensing statement is correct?**
- Avaya Breeze Client SDK Basic License is always an additional charge to the Core and Power Suite License

[Подсказка](https://www.devconnectprogram.com/site/global/products_resources/avaya_client_sdk/licensing/index.gsp) (к сожалению, текст не слишком явно указывает):

The current commercial model for the Avaya Breeze Client SDK provides the SDK free of charge to DevConnect (Licensee) members to access, create, and distribute applications and client solutions utilizing the Breeze Client SDK.

Customer / End-users, which may or may not also be the Licensee, are required to obtain certain runtime licenses for Avaya systems in order to effectively utilize functionality derived from applications incorporating Avaya Breeze Client SDK capabilities.

Licensees are required to includes within their End User License Agreement (EULA) certain flow down terms from Avaya, and should provide notice in their product or solution documentation notice to their Customers (End Users) regarding the need to purchase runtime licenses from Avaya to enable the use of Licensee's application, created using the SDK, on an Avaya product, service or network; and the applicable material codes for the named user licenses, server licenses, and agent licenses.

Licensees may also be required to obtain additional 3rd party licenses, based on use of certain codecs (including G.729 and H.264), and should consult with their own legal counsel for further guidance.

*End User Licensing details*
Then a Customer wishes to deploy an application or client which they or a third party has developed, using the Avaya Breeze™ Client SDK, they must purchase runtime licenses for their users. There are two types of licenses supported available as either a perpetual or subscription license.
- Avaya Breeze Client SDK Basic
- Avaya Breeze Client SDK Advanced

Basic licenses are purchased for Avaya Aura® named users. Each of these users must have either a Avaya Aura® Core or Power suite license. The minimum purchase is 50 Basic licenses and thereafter individual licenses can be purchased.

Advanced Licenses are purchased based on the number of concurrent Avaya Oceana™ Agent bundles purchased. The number of Advanced licenses held must always equal to the number of Avaya Oceana Agent bundles.

These licenses allow up to 5 third party applications to be deployed against each licensed user/agent.

The following diagram explains the association of Client SDK packages to the Basic and Advanced Licenses.

![Avaya Breeze Client SDK Basic and Advanced Licenses](image283.png)

**Q. Which two components are required by the Avaya WebRTC Snap-in? (choose two)**
- Avaya Media Server 7.6+
- Avaya Breeze R3+

*Подсказка*:
Avaya WebRTC Snap-in supports applications built with earlier versions of the JavaScript libraries. The applications work without modifications, but you must update the applications to use the latest SDK version. The latest version gives you access to the latest features and functions of the snap-in.

Avaya WebRTC Snap-in needs the following products:
- Avaya Breeze platform Release 3.5
- Avaya Aura System Manager Release 7.1.2
- Avaya Aura Media Server Release 7.8
- Avaya Aura Communication Manager Release 6.3.5 or later
- Avaya Session Border Controller for Enterprise Release 6.3 or later

Avaya SBCE requires separate Advanced and Standard licenses from the license pool for each concurrent session.

**Q. Which licenses are required when Avaya Breeze application program interfaces (API) are used to create audio/video conferences?**
- Scopia a la carte licenses (маленькое сомнение, но вроде так)

*Подсказка*:
12.1.7. Avaya Scopia for audio/video conferencing (из документа Offer Definition, правда про лицензии ни слова)
If the application to develop wishes to leverage Avaya Breeze APIs to create audio/video conferences, Scopia v8.3 (a la carte version) would be required.

**Q. Avaya Breeze provides optional Avaya Snap-ins. Match the Snap-in name with the correct Snap-in description**
Ответ

|                      |                                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------------- |
| Engagement Designer  | A graphical drag and drop tool to create multichannel  workflows                                  |
| Real-Time Speech     | Monitors Parties in a two-party call and notifies an application when specific phrases are spoken |
| Engagement Assistant | Integrates with Microsoft Outlook Calendar to provide  one number conferencing                    |
| WebRTC               | Enables click-to-call on any supported browser                                                    |
| Call Part and Page   | Emulates the CS1000 function within Avaya Aura                                                    | 

**Q. Avaya Breeze supports clustering of groups of servers or nodes. Design rules include capacity limitation such as a maximum of 35 Breeze servers or nodes and up to Snap-ins sequenced per user. Which additional Avaya Breeze design rule must be followed?**
- Server in the same cluster can support different VMWare requirements.
- Only the master node(s) requie(s) a server license
- Geo-redundant deployments support servers across data centers in a synchronized model
- **Load balancing across Breeze nodes requires all nodes to be in the same LAN subnet - выбрала, но не уверена**
- Each Snap-in within a cluster includes a node designated as master

**Q. Which application program interface (API) is used by the Avaya Breeze Data Access Methods to access data from an external database?**
- Java Persistence API

*Подсказка* из Avaya Breeze Overview and Specification:
Data Access Methods
Enable a snap-in to access a user, a service, or global data from the provisioning database as administered on System Manager. Data Access Methods also enable the snap-in to access data from an external database by using Java Persistence API (JPA).

**Q. For Avaya Breeze solutions, which statement correctly describes including Automatic Speech Recognition (ASR) and Text To Speech (TTS) capabilities in Snap-ins?**
- A separate Nuance license is required on the Avaya Aura Media Server on a per port basis

**Q. Which three areas include Team Engagement needs and solutions? (choose three)**
- Growth Enablement
- Worker and Team Productivity
- Communications Optimization

**Q. Which Snap-in stores a customer\'s journey history allowing the customer to make subsequent calls without having to repeat any of the history?**
- Context Store Snap-in

*Подсказка*:
Avaya Context Store is an Avaya Breeze snap-in that enables context-sensitive, real-time customer contact information to be updated from multiple sources and shared between the various components and touch points in the enterprise through which a customer passes.

The Context Store Snap-in allows the capturing of every step of the customer journey with full 360 degree context.

In this example a customer experienced a login failure, sent an SMS to customer service who in turn sent an SMS with a password re-set link. These interactions where all placed in the Context Store. The customer then searched the web site for a new product issue and did not find an answer. They then sent an email to get an answer to the new product issue. When the customer places a call to resolve the issue, the customer service agent sees all of the interactions during the customer's journey and is able to resolve the issue without having to ask the customer to repeat any of the history. This leaves the customer with a very good impression.