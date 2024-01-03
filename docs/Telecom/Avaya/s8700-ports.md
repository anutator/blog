---
title: Список портов и протоколов Avaya S8700 Media Server
share: "true"
---

This is a listing of ports and protocols used by the S8700 Media Server's interface to the Customer's LAN, i.e. the S8700-enterprise interface. On an Avaya S8700 Media Server in the Multi-connect this will the eth4. On an Avaya S8700 Media Server in the IP-Connect configuration this is on eth0, shared with the Control Network A (CNA) connection. This does not include IPSI traffic (yet).
## Входящие на интерфейсе S8700 Enterprise
Порты назначения на S8700.

| Protocol/Port     | Name     | Service                    | S8700/MV Usage                                                                                                             |
| ----------------- | -------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| TCP/00020         | ftp-data | FTP data channel           | The S8700 server can enable an ftp server to allow Upgrades and Patches to be placed on the server. [1]                    |
| TCP/00021         | ftp      | Normal FTP daemon          |                                                                                                                            |
| TCP/00022         | ssh      | Secure Shell               | Available for customer use and future use by Avaya.                                                                        |
| TCP/00023         | telnet   | Normal Telnet daemon       | Shell access to S8700, generally not required, a shell is accessible through the SAT interface on the active server.       |
| TCP/00080         | www      | Web server                 | Login screen will move to https                                                                                            |
| TCP/00443         | https    | Secure Web Server          | Primary web services port                                                                                                  |
| TCP/00512 - 01023 | rsh      | Return path for std error  | rsync over rsh is used to move translation files to the LSP. This will move to a more secure interface in a future release |
| TCP/05023         | def-sat  | DEFINITY SAT telnet daemon | Required for inbound SAT (ASA), Optionally SAT can be through a CLAN.                                                      |
| UDP/00123         | ntp      | Network time protocol      | Used to synchronize to external time server                                                                                |
| UDP/0161          | snmp     | SNMP MIB sets/gets         | If network management tool is used and native agent is available (MV1.2.1).                                                |

## Исходящие на интерфейсе S8700 Enterprise
Исходящие порты S8700 не постоянны.

| Protocol/port | Name      | Service                             | S8700/MV Usage                                                                                                                          |
| ------------- | --------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| TCP/00020     | ftp-data  | FTP data channels                   | Used for translation back-up to FTP server or getting files from ftp server.                                                            |
| TCP/00021     | ftp       | Normal FTP daemon                   |                                                                                                                                         |
| TCP/00022     | ssh       | Secure Shell                        | Not Required.                                                                                                                           |
| TCP/00023     | telnet    | Normal Telnet daemon                | Not Required; useful for trouble shooting.                                                                                              |
| TCP/00025     | smtp      | SMTP                                | Used for translation back-up via mail.                                                                                                  |
| TCP/00080     | www       | HTTP Web server.                    | Used when downloading files from an http server for updates or firmware download.                                                       |
| TCP/00514     | cmd       | Remote shell server (used by rsync) | Required for sending translations to LSP                                                                                                |
| UDP/00053     | domain    | DNS server                          | Required only if other services will be using it. e.g. Web addressing other server by name, accessing  http or ftp server for downloads |
| UDP/0162      | snmp-trap | SNMP traps                          | If network management tool is used or IP alarming is enabled.                                                                           |

[1].  When attempting to FTP a file from the S8700 server, a data session will be initiated from the server on a random high port.

### Настройки файрвола
This table contains specific recommendations and for customers (based on network configuration). This assumes that the VisAbility management server is on the same network as the S8700. All administration will be performed from specific administration stations to the S8700 (not through the CLAN). The WEB administration can also be performed from these specific administration terminals. Deny all access unless specifically permitted.

| Action                                                                                                                                                                                                                                                                                                                                                | From                        | TCP/UDP port or Protocol | To                          | TCP/UDP port or Protocol |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | ------------------------ | --------------------------- | ------------------------ |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8300 (LAN Spare Processor) | TCP any                  | S8700-Enterprise-intf       | TCP 512-1023             |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8300 (LAN Spare Processor) | TCP 514                  | S8700-Enterprise-intf       | TCP any                  |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8700-Enterprise-intf       | TCP 512-1023             | S8300 (LAN Spare Processor) | TCP any                  |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8700-Enterprise-intf       | TCP any                  | S8300 (LAN Spare Processor) | TCP 514                  |
| **These will allow the S8700 to synchronize translations with the LSP, not technically needed at 75 Wall. TCP session is initiated from the S8700 to the S8300 on port 514. A second session is then initiated from the S8300 to a port on the S8700 in the range 512-1023. There are plans to migrate away from this rsh protocol in a future release.** |                             |                          |                             |                          |
| Permit                                                                                                                                                                                                                                                                                                                                                | ASA work stations           | TCP any                  | S8700-Enterprise-intf       | TCP 5023                 |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8700-Enterprise-intf       | TCP 5023                 | ASA work stations           | TCP any                  |
| **These will allow the Admin. Workstation to log into the server.**                                                                                                                                                                                                                                                                                       |                             |                          |                             |                          |
| Permit                                                                                                                                                                                                                                                                                                                                                | Web Admin. Station          | TCP any                  | S8700-Enterprise-intf       | TCP 80                   |
| Permit                                                                                                                                                                                                                                                                                                                                                | Web Admin. Station          | TCP any                  | S8700-Enterprise-intf       | TCP 443                  |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8700-Enterprise-intf       | TCP 80                   | Web Admin. Station          | TCP any                  |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8700-Enterprise-intf       | TCP 443                  | Web Admin. Station          | TCP any                  |
| **These will allow secure and unsecure web access to the server; the server will redirect unsecure sessions to https. These permits can be allowed the entire customer network, or just the subnets or hosts where the administration will occur from.**                                                                                                  |                             |                          |                             |                          |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8700-Enterprise-intf       | UDP any                  | Customer's Network          | UDP 53                   |
| Permit                                                                                                                                                                                                                                                                                                                                                | Customer's Network          | UDP 53                   | S8700-Enterprise-intf       | UDP any                  |
| **This will allow the S8700 to perform DNS look-ups, not necessarily required, but nice if FTP or other services to the Customer's network are used.**                                                                                                                                                                                                    |                             |                          |                             |                          |
| Permit                                                                                                                                                                                                                                                                                                                                                | S8700-Enterprise-intf       | UDP any                  | Customer's Network          | UDP 123                  |
| Permit                                                                                                                                                                                                                                                                                                                                                | Customer's Network          | UDP 123                  | S8700-Enterprise-intf       | UDP any                  |
| **Требуется только если включена синхронизация времени по NTP**                                                                                                                                                                                                                                                                                                  |                             |                          |                             |                          |

Эти правила запрещают доступ извне Avaya Products, но разрешают доступ с локальной сети LAN заказчика.

| Action                                                                               | From                  | TCP/UDP port or Protocol | To                          | TCP/UDP port or Protocol |
| ------------------------------------------------------------------------------------ | --------------------- | ------------------------ | --------------------------- | ------------------------ |
| Permit                                                                               | Customer's LAN        | TCP any                  | S8700-Enterprise-int        | any                      |
| Permit                                                                               | S8700-Enterprise-int  | TCP established          | Customer's Network          | any                      |
| **These rules permit SAT, Telnet, Web and FTP Sessions to be originated into the S8700** |                       |                          |                             |                          |
| Permit                                                                               | S8700-Enterprise-intf | TCP any                  | S8300 (LAN Spare Processor) | TCP 514                  |
| **Enables translations to be sent to the LSP**                                           |                       |                          |                             |                          |
| Permit                                                                               | S8700-Enterprise-int  | TCP any                  | Customer's Network          | TCP 20                   |
| **Enables FTP from customer network to S8700**                                          |                       |                          |                             |                          |
| Permit                                                                               | S8700-Enterprise-intf | UDP any                  | Customer's Network          | UDP 53                   |
| Permit                                                                               | Customer's Network    | UDP 53                   | S8700-Enterprise-intf       | UDP any                  |
| Permit                                                                               | S8700-Enterprise-intf | UDP any                  | Customer's Network          | UDP 123                  |
| Permit                                                                               | Customer's Network    | UDP 123                  | S8700-Enterprise-intf       | UDP any                  |
| **Enables DNS and FTP service for S8700 Servers**                                       |                       |                          |                             |                          |

## CLAN
Открытые прослушивающие порты платы CLAN. Надо иметь ввиду, что S8700 общается с CLAN через плату IPSI (и TDM), а не через сетевой интерфейс CLAN.

| Port | TCP or UDP | Description                                                                              |
| ---- | ---------- | ---------------------------------------------------------------------------------------- |
| 1719 | UDP        | H.323 RAS                                                                                |
| 1720 | TCP        | H.323 Signaling                                                                          |
| 1037 | TCP        | Dual Connect CCMS (may be discontinued)                                                  |
| 2945 | TCP        | H.248                                                                                    |
| 5001 | TCP        | CMS (configurable)                                                                       |
| 5002 | TCP        | Intuity Audix                                                                            |
| 5003 | TCP        | DCS (6002-6008 for DCS/Intuity)                                                          |
| 5023 | TCP        | SAT (Telnet)                                                                             |
| XX   | TCP        | CDR - When inabled, CLAN acts as a client. Destination port is configured on CDR Server. |

Порты RTP/RTCP для медиа динамические и согласовываются во время фазы регистрации.
## IPSI
Список прослушивающих портов платы IPSI «listening» и их соответствующие порты на S8700.

| S8700 | IPSI | tcp/udp  | comment                                                                                       |
| ----- | ---- | -------- | --------------------------------------------------------------------------------------------- |
| any   | 5010 | tcp      | main control socket between PCD and SIM                                                       |
| any   | 5011 | tcp      | ipsiversion queries                                                                           |
| any   | 5012 | tcp      | serial number queries                                                                         |
| any   | 123  | udp      | ntp                                                                                           |
| 123   | any  | udp      | ntp                                                                                           |
| any   | 23   | tcp      | telnet - for configuration after enable.                                                      |
| any   | 21   | ftp      | Download of firmware                                                                          |
| any   | 20   | ftp-data | Download of firmware                                                                          |
| 3166? | 1956 | tcp      | command server (download, etc.)                                                               |
| any   | any  | icmp     | echo replies                                                                                  |
| any   | 2312 | tcp      | Used by development only for telnet shell access for debugging. Can be blocked by a firewall. |