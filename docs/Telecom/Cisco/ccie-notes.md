---
title: CCIE notes
share: "true"
tags:
  - cisco
---

# Traceroute / Ping
**Traceroute**
- Traceroute has default TTL of 30.
- Steps performed when traceroute is executed:
  - Sends 3 packets with `TTL=1` to first-hop router. FH router responds with time-exceeded (ICMP Type-11).
  - In response sends 3 packets with `TTL=2` to FH router, second-hop router responds with TTL message.
  - Continues until packets arrive at destination, last-hop router responds with unreachable (ICMP Type-3).
  - The LH router sends back a unreachable message because the destination is an unreachable port.

**Traceroute Output**
The `*` means that ICMP rate limit is enabled at the last-hop router. The default timeout is 500 msec.
- The reason only the LH router shows this is because intermediate routers send a time exceeded TTL message.
- The second traceroute packet usually times out because that one is within the 500 msec interval, the third packet is not.
- The same applies to ping with U.U.U output, the 1st message is sent back as unreachable by the LH router. The 2nd times out because it is within the 500 msec interval, the 3rd is unreachable again and so on ...

```cisco
ip icmp rate-limit unreachable 500
show ip icmp rate-limit
```

Traceroute Responses

|   | Описание  |
|---|---|
|`*` |The probe timed out|
|A|Administratively prohibited (ACL)|
|U|Port unreachable|
|H|Host unreachable|
|N|Network unreachable|
**Ping**
- Time exceeded on ping means TTL expired (ICMP Type-11). This is used in traceroute.
- Specify how often ICMP unreachable messages are sent to neighbors with the `ip icmp rate-limit unreachable` command.
- ICMP redirect messages are used to notify hosts that a better route (other router) is available for a particular destination.
- The kernel is configured to send redirects by default. Disable with the interface command `no ip redirects`.

Cisco routers send ICMP redirects when all of these conditions are met:
- The ingress interface is the same as the egress interface of the packet.
- The source is on the same subnet as the better next-hop.
- The source does not use source-routing.

ICMP types

| Тип  | Описание  |
|---|---|
|0|Echo Reply|
|3|Destination Unreachable|
|5|Redirect|
|8|Echo|
|11|Time Exceeded (TTL)|

ICMP Responses

|   | Описание  |
|---|---|
|!|Reply|
|.|Timed Out|
|U|Destination Unreachable|
# AAA
**Authentication, Authorization and Accounting (AAA)**
- Place local login before group authentication so that the specified usernames are authenticated first.

```
aaa authentication login default local-case group radius
aaa authentication login default local group tacacs+
```

**AAA Auto-Command**
- Automatically logout particular users (PPP usernames for example) and prevent them from managing the router.
- The `autocommand` function will only work if authorization is configured using AAA.
- The console is treated differently by default, and requires additional commands in order to automatically logout users.
 - The `aaa authorization console applies` AAA rules to the console as well.

```
username R1 autocommand logout
aaa authorization exec default if-authenticated

aaa authorization console
line con 0
 authorization exec default
```

## Lines / SSH

**Login**
- Log both successful and failed login attempts.
- 3-second delay between successive login attempts.
- Do not allow login attempts for 10 seconds if two tries fail within 15 seconds.
- During login block allow hosts in ACL.

```
login on-failure log
login on-success log
login delay 3
login block-for 10 attempts 2 within 15
login quiet-mode access-class QUIET

ip access-list standard QUIET
 permit host 192.168.0.1
```

Secure Shell (SSH) 
- Enable SSH without ip domain-name by using the label keyword.
- Normally the first generated RSA key is linked to SSH. Override this with the keypair-name command.

```
crypto key generate rsa modulus 768 label R1.lab.local
ip ssh version 2
ip ssh rsa keypair-name R1.lab.local
username admin privilege 15 password cisco
line vty 0 4
 login local

show crypto key mypubkey rsa
ssh -v 2 -l admin 192.168.0.1
```

Telnet
- Default ToS is 192 (C0). Change the Telnet ToS with the ip telnet tos command.

Hide IP / hostname information when establishing Telnet sessions:

```
ip telnet hidden addresses
ip telnet hidden hostnames
```

Or:

```
service hide-telnet-addresses
```

Show the line connected to when Telnet establishes:

```cisco
service linenumber

show users
show line
```
## Privilege
**Privilege Access Control**
- The all keyword specifies the configuration and all underlying sub-configurations.
- This will give access to all interfaces and all configurations under the interfaces.
- Every level has access to all the commands available underneath it. 8 has access to privilege 1-7 and 8
- In IOS there are 3 default privilege levels:
  - Privilege 0.
  - Privilege 1. User exec.
  - Privilege 15. Privilege exec.

Create custom commands for level 10:

```cisco
privilege exec level 10 [commands]
privilege configure level 10 [commands]
privilege interface level 10 [commands]
```

## Radius
**Radius**
- By default all defined radius servers will be used if you configure aaa...default group radius.
- Servers are consulted in the order in which they were configured.
- Either set default key, timeout and retransmit settings, or per host.
- Host specific setting will override the default. Default timeout is 5 sec, default retransmit amount is 3 times.
- Default radius ports are 1645 for authentication and 1646 for accounting (newer ports are 1812 and 1813).

```
aaa new-model
radius-server host 1.1.1.1 auth-port 1645 acct-port 1646 timeout 5 retransmit 3 key cisco

radius server RADIUS
 address ipv6 1::1 auth-port 1812 acct-port 1813
 timeout 3
 retransmit 0
 key cisco

aaa authentication login default group radius local
aaa authorization exec default group radius local

line vty 0 4
 login authentication default
 authorization exec default

show radius server-group all
debug radius
```

**AAA Server Groups**
- Group servers to use for a single purpose. For example a group used only for PPP authentication.
- Grouped private servers do not have to exist in the global config. And are used for a single purpose by a single AAA server group.
- Grouped public servers have to exist in the global config.
  - Public servers configured with the tacacs server command, have to be linked with the server name command.
  - Public servers configured with the tacacs-server host command, have to be linked with the server command.

```
aaa new-model
radius-server host 1.1.1.1 auth-port 1645 acct-port 1646 timeout 5 retransmit 3 key cisco

radius server RADIUS
 address ipv6 1::1 auth-port 1812 acct-port 1813
 timeout 3
 retransmit 0
 key cisco

aaa group server radius MYRADIUS
 server name RADIUS
 server 1.1.1.1
 server-private 2.2.2.2 timeout 5 retransmit 0 key cisco

aaa authentication login default group MYRADIUS local
aaa authorization exec default group MYRADIUS local

show aaa servers
```

**Radius Change of Authorization (CoA)**
- The CoA feature provides a mechanism to change the attributes of a AAA session after it is authenticated.
- The change is initiated on the radius server and 'pushed' to the router.
- The default port for packet of disconnect is 1700, ACS uses 3379.
- The client is the radius server that sends the CoA request.
- Requests allow for:
  - Session identification
  - Host re-authentication
  - Session termination

```
aaa new-model
aaa server radius dynamic-author
 client 1.1.1.1 server-key cisco
 port 3799
 auth-type all

show aaa clients
```

## RBAC
**Role-Based Access Control (Parser Views)**
- Requires AAA enabled.
- AAA authorization is recommended.
- Requires enable 'root' password configured.
- A superview combines two more more parser views.

```
enable secret cisco
enable view root

username bpin view VIEW privilege 15 password cisco

aaa new-model
aaa authentication login default local
aaa authorization exec default local

parser view VIEW
 secret cisco
 commands exec include configure terminal
 commands configure ...

show parser view
enable view VIEW

parser view SUPERVIEW superview
 secret cisco
 view VIEW1
 view VIEW2
```

## Tacacs+
**Tacacs+**
- The tacacs-server host command will be replaced by tacacs server. The old-style command has no option for IPv6.
- This new form will only show up after aaa new-model has been configured.
- The single-connection option will reuse the TCP session. This is more efficient because the tcp session doesn't have to be rebuild.
- Either set default key and timeout settings, or per host. Host specific setting will override the default. Default timeout is 5 sec.
- By default all defined tacacs+ servers will be used if you configure aaa... default group tacacs+.
- Servers are consulted in the order in which they were configured.

```
aaa new-model
tacacs-server host 1.1.1.1 single-connection port 49 timeout 5 key cisco

tacacs server TACACS
 address ipv6 1::1
 key cisco
 timeout 5
 single-connection

aaa authentication login default group tacacs+ local
aaa authorization exec default group tacacs+ local
aaa accounting commands 15 default start-stop group tacacs+

line vty 0 4
 login authentication default
 authorization exec default

show tacacs
debug tacacs
debug tacacs events
debug tacacs packets
```

**AAA Server Groups**
- Group servers to use for a single purpose. For example a group used only for PPP authentication.
- Grouped private servers do not have to exist in the global config. And are used for a single purpose by a single AAA server group.
- Grouped public servers have to exist in the global config.
  - Public servers configured with the tacacs server command, have to be linked with the server name command.
  - Public servers configured with the tacacs-server host command, have to be linked with the server command.

```
tacacs-server host 1.1.1.1 single-connection port 49 timeout 5 key cisco
tacacs server TACACS
 address ipv6 1::1
 key cisco
 timeout 5
 single-connection

aaa group server tacacs+ MYTACACS
 server name TACACS
 server 1.1.1.1
 server-private 2.2.2.2 single-connection port 49 timeout 5 key cisco

aaa authentication login default group MYTACACS local
aaa authorization exec default group MYTACACS local
aaa accounting commands 15 default start-stop group MYTACACS   

show aaa servers
```

# Access-Lists
**Compiled Access-Lists**
- The turbo ACL feature is designed in order to process ACLs more efficiently.
- Applied on all access-lists.

```
access-list compiled
show access-list compiled
```

**VTY Access-Lists**
- Outbound ACL on interface only filters transit traffic, not locally generated traffic.
- Outbound ACL on VTY lines only applies to connections generated from the existing VTY session.
- Inbound ACL on VTY is applied to outside connections coming into the router.

```
ip access-list standard VTY_IN
 10 permit host 192.168.0.1
 99 deny any log

ip access-list standard VTY_OUT
 10 permit host 192.168.0.2
 99 deny any log

ip access-list logging interval 1
ip access-list log-update threshold 1  

line vty 0 4
 access-class VTY_OUT out
 access-class VTY_IN in
```

## Dynamic
**Dynamic Access-Lists (Lock-and-Key)**
- Blocks traffic until users telnet into the router and are authenticated.
- A single time-based dynamic entry is added to the existing ACL.  
- Idle timeouts are configured with the `autocommand`. Absolute timeouts are configured in the ACL.
- The absolute timeout value must be greater than the idle timeout value, if using both.
- If using none, the access-list entry will remain indefinitely.
- It is also possible to set the `autocommand` access-enable host timeout directly on the VTY line.
- Absolute timers can be extended by 6 minutes using the `access-list dynamic-extended` command. Requires re-authentication.

```
username bpin password cisco
username bpin autocommand access-enable host timeout 4

ip access-list extended DYNAMIC
 10 permit ospf any any
 20 permit tcp host 10.0.12.1 host 10.0.12.2 eq telnet
 30 dynamic ICMP timeout 8 permit icmp host 10.0.12.1 any
 99 deny ip any any log-input
 access-list dynamic-extended

int fa0/0
 ip access-group DYNAMIC in
```

## IPv6
**IPv6 Access-Lists**
- By default IPv6 access-lists add (hidden) permit statements that are necessary for IPv6 to function.
- IPv6 only supports named and extended ACLs.
- If using an explicit deny, make sure to manually include the ND and RA/RS statements.

```
ipv6 access-list ALLOW_TELNET_OSPF
 sequence 10 permit 89 host FE80::2 any
 sequence 20 permit tcp host 12::2 any eq telnet
 sequence 30 permit icmp any any nd-na
 sequence 40 permit icmp any any nd-ns
 sequence 50 permit icmp any any router-advertisement
 sequence 60 permit icmp any any router-solicitation
 sequence 99 deny ipv6 any any log-input

int fa0/0
 ipv6 traffic-filter ALLOW_TELNET_OSPF in
```

## Reflexive
**Reflexive Access-Lists**
- Only allow return traffic if inside source initiated the traffic.
- The `reflect` keyword links ACLs together, the `evaluate` entry is dynamically created based on this reflect entry.
- By default the dynamic entry will timeout after 60 seconds.

```
ip access-list extended TRAFFIC_FROM_R3
 10 permit ospf any any
 20 permit icmp host 192.168.0.3 any reflect ICMP

ip access-list extended TRAFFIC_TO_R3
 10 evaluate ICMP
 99 deny ip any any log-input

ip access-list logging interval 1
ip access-list log-update threshold 1
ip reflexive-list timeout 60

int fa0/0
 description LINK_TO_R3
 ip access-group TRAFFIC_FROM_R3 in
 ip access-group TRAFFIC_TO_R3 out
```

## Time-Based
**Time-Based Access-Lists**
- Specify time ranges that will allow traffic during specific periods.
- Based on local system time.

```
time-range DAILY
 periodic daily 09:00 to 17:00

ip access-list extended ALLOW_ICMP_TIME
 10 permit ospf any any
 20 permit icmp host 192.168.0.3 any time-range DAILY
 30 deny ip any any log-input

int fa0/0
 description LINK_TO_R3
 ip access-group ALLOW_ICMP_TIME in
```

# BGP
**AS Range**
- Private AS range is 64512 - 65534, 65535 is reserved for special use.

|   |   |
|---|---|
|asplain|2-byte: 1 to 65535  <br>4-byte: 65536 to 4294967295|
|asdot|2-byte: 1 to 65535  <br>4-byte: 1.0 to 65535.65535|

**BGP Best Path Selection**
- A peer Autonomous System (AS) is what defines an external or internal route.
- A route learned from an external peer will be external, until the point where the route is forwarded (or redistributed) into own AS.

**First criteria is that the NEXT_HOP is reachable, meaning that it is present in the routing table.**
**Which protocol was responsible for generating the route is irrelevant.**
- WEIGHT (highest)
- LOCAL_PREF (highest)
- Locally injected (network, aggregate) over remotely learned
- AS_PATH (shortest)
- ORIGIN (lowest) (0 over 1 over 2) 0 is internal, 1 is external (absent), 2 is redistributed (incomplete)
- MUTLI_EXIT-DISC / MED (lowest)
- eBGP over iBGP learned routes
- Lowest IGP cost to the NEXT_HOP

**Best-Path Tie Breakers (No Multipath)**
- If both paths are external, prefer the older one
- If both paths are internal, prefer the lowest ROUTER_ID
  - If ORIGINATOR_ID is the same, prefer one with the shorter CLUSTER_LIST
  - Finally, prefer the one with the lowest neighbor's IP-address

**Multipath is enabled**
Weight, Local preference, AS_PATH length, Origin, MED, AS PATH content must match + eBGP and iBGP specific additional rules.

**BGP Synchronization**
- Synchronization is only relevant in iBGP. Disabled by default.
- Routes will only be passed on if they are synchronized.
- All iBGP learned prefixes must have an identical route in the routing table learned from an IGP.
- The originator of the route in the routing table has a router-id (RID).
- This IGP RID has to match the BGP neighbor that the prefix was received from otherwise its not synchronized.
- To fix synchronization problems change either the originators IGP RID or the neighbors BGP RID to the same value.

**BGP Attributes**
BGP uses Network Layer Reach-ability Information (NLRI), which is the route and prefix and certain attributes.

|   |   |   |
|---|---|---|
|Well known, Mandatory|Must appear in every UPDATE message.<br>Must be supported by all BGP software implementations.<br>If missing, a NOTIFICATION message must be sent to the peer. |NEXT_HOP<br>AS_PATH<br>ORIGIN |
|Well-Known, Discretionary|May or may not appear in an UPDATE message.<br>Must be supported by any BGP software implementation. |LOCAL_PREF<br>AT_AGGREGATE |
|Optional, Transitive|May or may not be supported in all BGP implementations.<br>Will be passed on if not recognized by the receiver. |AGGREGATOR<br>COMMUNITY |
|Optional,<br>Non-Transitive |May or may not be supported in all BGP implementations.<br>Not required to pass on, may be safely ignored. |MED<br>ORIGINATOR_ID<br>CLUSTER_LIST |

**BGP Attributes Scope**
- Transitive, these attributes are across AS boundaries.
- Non-transitive, these attributes are restricted to the same AS.
- Local, these attributes are local to the router only (weight).

Transitive attributes can be altered to be local only, non-transitive attributes cannot be altered.
MED can be transitive but only to one neighbor AS, not more.

**NEXT_HOP Attribute**
This is a well-know, mandatory, transitive attribute that must be present in all updates.
- Is the peers IP-address if remotely learned.
- Is 0.0.0.0 for routes advertised using the network or aggregate commands.
- Is the IP next-hop for redistributed routes.
- The next-hop must be reachable, meaning that it must be present in the routing table.
- Remains unchanged in the same AS by default, but can (or should) be modified.
- Is changed by default when forwarded between different AS. Will become the IP address of the router that passed on the route.

**AS_PATH Attribute**
This is a well-know, mandatory, transitive attribute that must be present in all updates.
- Ordered List (AS_SEQUENCE).
- Read from right to left.
- Can have an unordered component AS_SET.
- Used for loop prevention only.
- Shorter count is better (hop count).
 - Local ASN is added when advertised to an external peer.
- Can be modified using route-maps.
- AS Path Prepending.
- In special cases can be shortened using neighbor "neighbor-ip-address" remove-private-as. This will remove the private AS that is connected to a non-private as from appearing in the AS_PATH. Neighboring AS will assume that the route originated from the non-private AS.

**ORIGIN Attribute**
This is a well-know, mandatory, transitive attribute that must be present in all updates. Origin is not part of AS_PATH.
Three possible values:
- IGP (0, shown in IOS as "i").
- EGP (1, shown in IOS as "e").
- Incomplete (2, shown in IOS as "?").

Set at the injection point:
- Using "network" or "aggregate" will result in IGP origin code.
- Using redistribution will result in incomplete origin code.

Can be modified using route-maps using the neighbor statement or at the insertion point.

**WEIGHT Attribute**
This is a proprietary, optional, non-transitive attribute that is Cisco only and is only of local significance.
Higher value preferred, default values are:
- 32768 for locally inserted.
- 0 for remotely learned.

Can be changed using neighbor statement (directly) or using route-maps (only in the inbound direction)

**LOCAL_PREF Attribute**
This is a well-known, discretionary, non-transitive attribute that must be supported by all vendors, but may not appear in every UPDATE message. The local-preference is only of significance to the same AS.
- Higher value preferred, default value is 100. Not always visible using show commands.
- Takes effect in the entire autonomous system, even in confederations (sub-autonomous systems)
- Most effective way to influence path preference for incoming routes, meaning outgoing traffic paths.

Can be modified using route-maps.

**ATOMIC_AGGREGATE Attribute**
- Well-known discretionary attribute
- Must be recognized by all BGP implementations, but does not have to appear in all UPDATES.
- This attribute is set when routes are aggregated (summarized) at a BGP speaker and forwarded to another AS.
- Alerts BGP peers that information may have been lost in the aggregation and that it might not be the best path to the destination.
- For example, if 10.0.1.0/24 and 10.0.2.0/24 are summarized to 10.0.0.0/22 it will include the 10.0.0.0/24 and 10.0.3.0/24.
- Aggregated routes will inherit the attributes of the component routes.
- When routes are aggregated, the aggregator attaches its RID to the route into the AGGREGATOR_ID attribute, unless the AS_PATH is set using the AS_SET statement.
- It is not possible to disable the discard route in BGP.

**Communities Attribute**
- This is an optional, transitive attribute that is comparable to route-tags.
- Not used for best-path selection.
- Large number (32 bits) that are conventionally displayed as ASN:ID (for example 64512:100).
- This notation however must be enabled using the ip bgp-community new-format command.
- This community "tag" is stripped by neighbors by default (even within the same AS). Use neighbor statement with route-maps.
- Routes can be in multiple communities.

**MULTI EXIT_DISC/MED/METRIC Attribute**
This is an optional, non-transitive attribute meaning that its not required to pass on when received from another AS.

MED can actually be transitive but only to one neighbor AS, not more.
- Lower value preferred, but the default value is missing in IOS (treated as 0 which is equal and thus best). This can be modified so that 0 is treated as worst. Use the bgp bestpath med missing-as-worst command.
- Influences preferred "exit point" when peering with the same AS at multiple locations.
- Requirements are that WEIGHT, LOCAL_PREF, AS_PATH length and peer AS must be the same. These requirements can also be modified, using the bgp always-compare-med command.

Can be modified (increased from 0) using route-maps or redistribution.
- Setting the MED outbound will influence the remote AS.
- Setting the MED inbound will influence own AS.

**ORIGINATOR_ID**
Route reflectors use the ORIGINATOR_ID and CLUSTER_LIST attributes for loop prevention.
- Each routing table entry will have the originating routers RID inserted by the RR.
- The RR will pass along routes to the RR-Clients, but it is these clients themselves who check the update for the presence of the RID (ORIGINATOR_ID). If this ORIGINATOR_ID matches their own, the update is denied.

**CLUSTER_LIST**
Set by RR by default to the value of the RID. The CLUSTER_ID identifies a group, or a single RR.
- RR can use this information to discover other RR present on the network.
- Prevents the installation of multiple routes in the BGP table that were reflected by RR neighbor.

## Advertise-Map
**BGP Advertise-Map**
Advertise prefixes based on the existence of other prefixes in the BGP table.
- Network in the non-exist map has to be present in the BGP table.
- Possible to advertise default route if another route is present.
- This does not work with the `default-information originate` command.

```
route-map NON_EXIST permit 10
 match ip address prefix-list Lo2
route-map ADV_MAP permit 10
 match ip address prefix-list ADV_PREFIX

ip prefix-list ADV_PREFIX permit 192.168.0.1/32
ip prefix-list Lo2 permit 192.168.0.2/32

router bgp 1
 address-family ipv4
  network 192.168.0.1 mask 255.255.255.255
  neighbor 10.0.13.3 advertise-map ADV_MAP non-exist-map NON_EXIST

show ip bgp neighbors 10.0.13.3 | i Condition
```

Advertise default:

```
route-map EXIST permit 10
 match ip address prefix-list Lo2
route-map ADV_MAP permit 10
 match ip address prefix-list ADV_DEFAULT

ip prefix-list ADV_DEFAULT permit 0.0.0.0/0
ip prefix-list Lo2 permit 192.168.0.2/32

ip route 0.0.0.0 0.0.0.0 null0

router bgp 1
 address-family ipv4
  network 0.0.0.0 mask 0.0.0.0
  neighbor 10.0.13.3 advertise-map ADV_MAP exist-map EXIST
```

## Aggregation
**ATOMIC_AGGREGATE Attribute**
- Well-known discretionary attribute
- Must be recognized by all BGP implementations, but does not have to appear in all UPDATES.
- This attribute is set when routes are aggregated (summarized) at a BGP speaker and forwarded to another AS.
- Alerts BGP peers that information may have been lost in the aggregation and that it might not be the best path to the destination.
- For example, if 10.0.1.0/24 and 10.0.2.0/24 are summarized to 10.0.0.0/22 it will include the 10.0.0.0/24 and 10.0.3.0/24.
- Aggregated routes will inherit the attributes of the component routes.
- When routes are aggregated, the aggregator attaches its RID to the route into the AGGREGATOR_ID attribute, unless the AS_PATH is set using the AS_SET statement.
- It is not possible to disable the discard route in BGP.

The `as_set `keyword preserves the AS_PATH information, meaning that the AS information is not overwritten by the aggregator.
- In this case the AT_AGGREGATE is not set and the original AS (or multiple AS) is still present in the aggregated route.
- The `as-confed-set` keyword is the same as as-set, but applies to confederations.

By default the more specific routes are still advertised to peers in addition to the summary.
- This behavior can be altered with the summary-only command to not advertise the more specific routes.

**BGP Aggregation Suppress-Map**
- Accomplishes the same as summary-only, except only subnets matched in the suppress-map will not be advertised.

```
ip prefix-list 1 permit 3.0.0.1/32
ip prefix-list 3 permit 3.0.0.3/32

route-map SUPPRESS permit 10
 match ip address prefix-list 1
 match ip address prefix-list 3

router bgp 1
 address-family ipv4
  aggregate-address 3.0.0.0 255.255.255.252 as-set suppress-map SUPPRESS
```

**BGP Aggregation Unsuppress-Map**
- Subnets matched will be advertised alongside the suppressed summary.
- Applied on neighbor statement instead of aggregate statement.
- Will not inherit attributes set by other neighbor statements such as community values.
- In order to send attributes, neighbor route-map configurations have to be copied to the unsuppress route-map.

```
ip prefix-list 1 permit 3.0.0.1/32
ip prefix-list 3 permit 3.0.0.3/32

route-map UNSUPPRESS permit 10
 match ip address prefix-list 1
 match ip address prefix-list 3
 set community 1:1 additive

router bgp 1
 address-family ipv4
  aggregate-address 3.0.0.0 255.255.255.252 as-set summary-only
  neighbor 10.0.12.2 unsuppress-map UNSUPPRESS
```

**BGP Inject-Map**
- Routers can conditionally inject a more specific route based on the presence of an aggregate in the BGP table.
- The injected subnets must fall within the range of the aggregate.
- Copy-attributes will also transfer AS information, otherwise origin will be incomplete.
- Route-source must match the neighbor the aggregate was received from, NOT the originator.

```
ip prefix-list R3 permit 10.0.13.3/32
ip prefix-list AGG permit 3.0.0.0/29
ip prefix-list INJECT permit 3.0.0.5/32  
ip prefix-list INJECT permit 3.0.0.6/32  
ip prefix-list INJECT permit 3.0.0.7/32

route-map R3_AGG
 match ip address prefix-list AGG
 match ip route-source prefix-list R3
route-map INJECT
 set ip address prefix-list INJECT   

router bgp 1
 address-family ipv4
  bgp inject-map INJECT exist-map R3_AGG copy-attributes

show ip bgp injected-paths
```

**BGP Aggregation Advertise-map**
- This function works alongside AS_SET and summary-only.
- Used when multiple AS share the same prefixes and are both included in the aggregation.
- AS that are filtered do not become unreachable, they are just hidden in the aggregate.

```
show ip bgp regexp _4$
ip as-path access-list 4 permit _4$

route-map AGG_HIDE_AS4 deny 10
 match as-path 4
route-map AGG_HIDE_AS4 permit 99

router bgp 1
 add ipv4
 aggregate-address 34.0.0.0 255.255.255.248 summary-only as-set advertise-map AGG_HIDE_AS4
```

BGP Aggregation Attribute-Map
- Optionally add additional attributes to the aggregate with the attribute-map keyword.
- Using a route-map instead will provide the same results, and will be converted to attribute-map in the config.

```
route-map AGG_ATTRIBUTE permit 10
 set metric 500
 set local-preference 200
 set origin igp
 set community 3:1
 etc..

router bgp 1
 address-family ipv4
  aggregate-address 3.0.0.0 255.255.255.252 attribute-map AGG_ATTRIBUTE
  aggregate-address 3.0.0.0 255.255.255.252 route-map AGG_ATTRIBUTE
```

## AS_PATH
**BGP Prepend AS_PATH**
Prepend AS 3 five times to the AS_PATH:

```
route-map PREPEND_AS permit 10
 set as-path prepend 3 3 3 3 3

router bgp 1
 neighbor 10.0.12.2 route-map PREPEND_AS out
```

Prepend the last AS in the path 4 times. This will lead to 5 entries on neighbors (1 original + 4 prepend)

```
route-map PREPEND_LAST_AS permit 10
 set as-path prepend last-as 4

router bgp 1
 neighbor 10.0.12.2 route-map PREPEND_LAST_AS out
```

**BGP AS_PATH Tagging**
- The AS_PATH is converted to a tag by default when redistributing BGP into IGP.
- Convert this tag back when redistributing the IGP back into another BGP process on another router with `set as-path` tag.
- Configure `set automatic-tag` on the original redistributing router (BGP->OSPF) to preserve the origin code as well (i instead of ?).
- The automatic tag has to match a specific AS in the route-map.
- It is not possible to prepend these redistributed routes using the same route-map.

AS_PATH tag:

```
route-map AS_PATH_TAG permit 10
 set as-path tag

router bgp 1
 add ipv4
  redistribute ospf 1 route-map AS_PATH_TAG
```

Automatic Tag:

```
ip as-path access-list 1 permit .*
route-map AS_ORIGIN_TABLE_MAP permit 10
 match as-path 1
 set automatic-tag

router bgp 2
 add ipv4
  table-map AS_ORIGIN_TABLE_MAP

clear ip bgp ipv4 unicast table-map
```

BGP Local-AS
- Use a different AS then is configured when neighboring with peers.

By default R2 will see AS 2 prepended before the AS 1 on routes received from R1.
- AS 2 will also be prepended to all R1 routes passed on to other BGP peers.
- To override this behavior, configure `no-prepend` on R2.

By default R1 will see AS 2 prepended before the actual AS of 64512.
- To override this behavior, configure `replace-as` on R2.
- R1 will see prefixes from R2 with AS2, other peers will see AS 64512.

The dual-as keyword will allow R1 to peer with either the correct AS 64512 or the local-as 2.

```
router bgp 1
 neighbor 10.0.12.2 remote-as 2

router bgp 64512
 neighbor 10.0.12.1 remote-as 1
 neighbor 10.0.12.1 local-as 2 no-prepend replace-as dual-as
```

## Communities
**Communities Attribute**
- This is an optional, transitive attribute that is comparable to route-tags.
- Not used for best-path selection.
- Large number (32 bits) that are conventionally displayed as ASN:ID (for example 64512:100).
- This notation however must be enabled using the ip bgp-community new-format command.
- This community "tag" is stripped by neighbors by default (even within the same AS). Use neighbor statement with route-maps.
- Routes can be in multiple communities.

**BGP Filtering using Communities**
There are three well-known BGP communities:
- No-advertise. Do not advertise route to any peers. CxFFFFFF02
- No-export. Do not advertise route to external peers. Advertise to internal and confederation peers only. OxFFFFF01
- Local-as. Do not advertise route to external and confederation peers. Advertise to internal peers only. OxFFFFFF03

It is possible to add the communities at the network statement or redistribution point.
- This will allow the router to advertise routes to specific neighbors only.
- With the neighbor statement the routes are advertised to that neighbor but the community is applied after reception.
- With the network statement the community is applied immediately.

Advertise routes with incomplete origin (alternative to redistribution):

```
route-map BGP_ORIGIN permit 10
 set origin incomplete

router bgp 1
 address-family ipv4
  network 1.0.1.0 mask 255.255.255.0 route-map BGP_ORIGIN
  network 1.0.2.0 mask 255.255.255.0 route-map BGP_ORIGIN
  network 1.0.3.0 mask 255.255.255.0 route-map BGP_ORIGIN
  network 1.0.4.0 mask 255.255.255.0 route-map BGP_ORIGIN

ip prefix-list Lo1 permit 1.0.1.0/24
ip prefix-list Lo2 permit 1.0.2.0/24
ip prefix-list Lo3 permit 1.0.3.0/24
ip prefix-list Lo4 permit 1.0.4.0/24

route-map BGP_COMMUNITY permit 10
 match ip address prefix-list Lo1
 set community no-advertise 1:1

route-map BGP_COMMUNITY permit 20
 match ip address prefix-list Lo2
 set community local-as 1:1

route-map BGP_COMMUNITY permit 30
 match ip address prefix-list Lo3
 set community no-export 1:1

route-map BGP_COMMUNITY permit 99

router bgp 1
 address-family ipv4
  neighbor 10.0.12.2 route-map BGP_COMMUNITY out

show ip bgp community 1:1
```

Match community values and modify on other routers:
- The set comm-list 1 delete keyword will only delete the community matched in the list.
- The additive keyword will add the 'internet' community to the existing communities, and not overwrite them.

```
ip community-list 1 permit no-export
ip prefix-list Lo4 permit 1.0.4.0/24

route-map MODIFY_COMMUNITY permit 10
 match ip address prefix-list Lo4
 match community 1
 set comm-list 1 delete
 set community internet additive
route-map RM1 permit 100

router bgp 2
 address-family ipv4
  neighbor 10.0.12.1 route-map MODIFY_COMMUNITY in
```

## Confederations
**iBGP Confederations**
- Divides autonomous system into smaller sub-autonomous systems.
- Large (container) AS is a confederation, smaller systems are "members".
- Inside the member, regular iBGP rules apply using Full-mesh and/or RR.
- Perceived externally only as the confederation AS
- Local Preference (LOCAL_PREF) Applies to the entire confederation.

Configuration Considerations
- All routers must be configured to be aware of the confederation identifier.
- All routers should be configured to be aware of all the confederation peers (members).
- Can be configured alongside RR.

```
router bgp 1
 bgp confederation identifier 123
 bgp confederation peers 2
 bgp confederation peers 3
neighbor 192.168.0.2 remote-as 2
neighbor 192.168.0.3 remote-as 3
neighbor 192.168.0.2 update-source Lo0
neighbor 192.168.0.3 update-source Lo0
neighbor 192.168.0.2 ebgp-multihop
neighbor 192.168.0.3 ebgp-multihop
 address-family ipv4
  neighbor 192.168.0.2 next-hop-self
  neighbor 192.168.0.3 next-hop-self
```

The peering between AS1 and AS2, AS3 is an internal peering using different AS.
- Meaning that routers R3 and R2 are unaware of the routes injected by R1, unless a full-mesh is configured.
- Another option is to use `next-hop-self` for all peers.

When peering with loopbacks between confederation peers the peering is basically the same as eBGP.
- Meaning that `ebgp-multihop` has to be configured.
## eBGP Peering
**eBGP Peering Rules**
- Neighbor must be in a different AS.
- Neighbor should be directly connected. Override with `ebgp-multihop` which changes the default TTL.
- If not, an underlying routing protocol must provide reachability (static, IGP). Beware of recursive routing.
- NEXT_HOP is changed when routes are advertised (by the advertising router).
- Routes with own AS are rejected if observed in incoming AS_PATH updates (AS_SEQUENCE and / or AS_SET).
- All routes are advertised and accepted.

If using an IGP to advertise loopbacks for the external peering (multi-hop). It is important not to advertise the same loopbacks into BGP.
- BGP has a lower AD, meaning that the IGP routes used for the peering will become BGP routes. This will lead to recursive routing.
- Solve by increasing the AD of eBGP (not recommended), decreasing the AD of the IGP (possible solution) or use backdoor routes.

The backdoor route will set the iBGP administrative distance (200) for the route instead of the eBGP distance.
- This ensures that it is higher than the IGP AD.
- The `backdoor` statement is specified when advertising the loopback of the neighbor into BGP.

```
int se1/0
 description EBGP_LINK_TO_R2
 ip add 10.0.12.1 255.255.255.0
interface lo0
 description EBGP_PEERING_LOOPBACK
 ip address 192.168.0.1 255.255.255.255

router eigrp 1
 network 10.0.12.0 0.0.0.255
 network 192.168.0.1 0.0.0.0

ip bgp-community new-format
router bgp 1
 bgp router-id 192.168.0.1
 neighbor 192.168.0.2 remote-as 2
 neighbor 192.168.0.2 ebgp-multihop 2
 neighbor 192.168.0.2 update-source Loopback0
 address-family ipv4
  network 192.168.0.1 mask 255.255.255.255
  network 192.168.0.2 mask 255.255.255.255 backdoor
  neighbor 192.168.0.2 activate
  neighbor 192.168.0.2 send-community
```

**eBGP Peering using Default Route**
BGP will not actively open (initiate) BGP session with a peer if the only route to it is the default route.
- BGP will initiate a session if the active BGP peer has a more specific route to its peer.
- The passive BGP peer is allowed to only have a default route.
- One side has to have a specific route for this to work.

Force active/passive peerings between peers:

```
router bgp 1
 neighbor 192.168.0.2 transport connection-mode active

router bgp 2
 neighbor 192.168.0.1 transport connection-mode passive
```

If Peer 1 (being the active peer by force) only has a default route to Peer 2, the session will not form.

```
ip route 0.0.0.0 0.0.0.0 10.0.12.2

ip bgp-community new-format
router bgp 1
 bgp router-id 192.168.0.1
 neighbor 192.168.0.2 remote-as 2
 neighbor 192.168.0.2 ebgp-multihop
 neighbor 192.168.0.2 update-source Loopback0
 neighbor 192.168.0.2 transport connection-mode passive
 address-family ipv4
  neighbor 192.168.0.2 activate
  neighbor 192.168.0.2 send-community
```

**BGP Connected-Check**
- Allows directly connected neighbors to peer with loopback addresses without configuring multi-hop.

```
router bgp 1
 neighbor 10.0.12.2 remote-as 2
 neighbor 10.0.12.2 disable-connected-check
```
  
## iBGP Peering
**BGP Internal Routing**
- Does not replace IGP. No direct connectivity required.
- Works alongside IGPs, this is why direct connectivity is not required.
- Carry external prefixes throughout the AS.
- "Split-horizon" as the loop-prevention. iBGP routes are not forwarded to other iBGP peers.
- Does not re-advertise internally-learned prefixes to other iBGP peers, because the AS_PATH is not modified inside the AS.
- Multiple sessions to same neighbor are not permitted, use loopback destinations instead.

**Peering Rules**
- Neighbor must be in the same AS.
- Neighbors do not have to be directly connected, if not a underlying routing protocol must provide reachability.

Default Policy Behavior
- NEXT_HOP is not changed when routes are advertised.
- External routes are advertised.
- Routes learned from other internal peers are not advertised to other iBGP neighbors. But are advertised to eBGP neighbors.

**iBGP Next-Hop-Self**
- Because NEXT_HOP is not changed, IGPs are needed to reach neighbors.
- Another solution is to change the NEXT_HOP to the local router address.
- The address that will be chosen for this is the loopback address (in case of next-hop-self) or the internal address used to peer with the neighbors.
- Make sure the networks next_hop_self is set to are advertised into iBGP. Another solution is to redistribute the IGP.

```
router bgp 1
 neighbor 10.0.12.2 remote-as 2
 address-family ipv4
   neighbor 10.0.12.2 next-hop-self all
   network 10.0.12.0 mask 255.255.255.0
   network 10.0.13.0 mask 255.255.255.0
   redistribute ospf 1 metric 2 match internal
```

The `all` keyword enables next-hop-self for both eBGP and iBGP received paths. Default is only for iBGP received paths.
- BGP only redistributes ebgp routes into an IGP
- BGP does not redistribute ospf external routes by default
- If no metric is specified it will be equal to the number of hops a peer needs to reach the networks.
- If a metric is specified, it will be applied to all routes advertised, meaning that all routes in the OSPF domain will receive the same metric.
- The default is to only match OSPF internal routes, redistributed routes into OSPF are not included.

**iBGP Peer-Groups**

```
router bgp 123
 bgp listen range 10.0.123.0/24 peer-group PEERS
 bgp listen limit 2
 neighbor PEERS peer-group
 neighbor PEERS remote-as 123
 address-family ipv4
  neighbor PEERS activate
  neighbor PEERS send-community
  neighbor PEERS etc..
```

## IPv6
**IPv6 Networks over IPv4**

```
router bgp 1
 neighbor 10.0.12.2 remote-as 2
 address-family ipv4
  neighbor 10.0.12.2 activate
 address-family ipv6
  neighbor 10.0.12.2 activate
  neighbor 10.0.12.2 route-map IPV6_NEXT_HOP out
  network 1::1/128

route-map IPV6_NEXT_HOP permit 10
 set ipv6 next-hop 2001:10:0:12::1
```

IPv6 Peer Link-Local

```
router bgp 1
  neighbor FE80::2%Serial1/0 remote-as 2
 address-family ipv6
  neighbor FE80::2%Serial1/0 activate
  network 1::1/128
```

**IPv6 Peer Loopback**
- Same as IPv4, IPv6 neighbors are not automatically activated.
- Specify router-id if no IPv4 addresses are used on the router.

```
ipv6 route 2::2/128 2001:10:0:12::2

router bgp 1
 bgp router-id 192.168.0.1
 neighbor 2::2 remote-as 2
 neighbor 2::2 update-source Loopback 0
 neighbor 2::2 disable-connected-check
 address-family ipv6
  neighbor 2::2 activate
```

**IPv4 Networks over IPv6**
- Peer with directly connected interfaces (NOT loopbacks) when advertising IPv4 prefixes over an IPv6 connection.

```
router bgp 1
 neighbor 2001:10:0:12::2 remote-as 2
 address-family ipv4
  neighbor 2001:10:0:12::2 activate
  neighbor 2001:10:0:12::2 route-map IPV4_NEXT_HOP in
  network 192.168.0.1 mask 255.255.255.255
 address-family ipv6
  neighbor 2001:10:0:12::2 activate

route—map IPV4_NEXT_HOP permit 10
 set ip next—hop 10.0.12.2
```

## Filtering AS
AS-Path Filters

| Фильтр | Описание |
| ---- | ---- |
| `_` | White space, start of string or end of string. |
| `^` | Start of string, PEER AS. |
| `$` | End of string, ORIGINATING AS. |
| `.` | Matches any character. |
| `+` | repeats the previous character one or more times. |
| `*` | repeats the previous character zero or many times |
| `?` | repeats the previous character one or zero times. |
| `I` | A logical "or" statement. |
| `\|` | Removes special meanings. |
| `()` | Matches a group of characters. |
| `[]` | Matches a range of characters. |

Definition and Use of AS-Path Access-Lists
- AS-Path access-lists match a single character, every number in an AS is a single character.
  - Example. AS 2456 consists of the single characters 2,4,5 and 6.
- Can match a range of characters using brackets [].
  - Example. [0-3] consists of 0 or 1 or 2 or 3
- Can match a group of characters.
  - Example. (123) matches only 123 when the characters follow each other. However they can be part of a greater AS such as 55123 or 12300.
  - Example. (123_) matches only 123 when its at the end of the AS, such as 55123 but not 12300.
  - Example. (_123_) matches only AS123 and it cannot be part of a greater AS.

AS-Path AS-Sequence
- In IOS the AS-Sequence is the AS_PATH that a route travels through.
- The AS that the router receives the route from is called the PEER AS.
- The router that originated the route is called the ORIGIN AS, others are called TRANSIT AS.
o Example. ^500_400_300_200_100$. 500 is the PEER AS, 100 is the ORIGIN AS, others are TRANSIT AS.

AS-Path Filtering
Match prefixes that originated in the connected AS:

```
ip as-path access-list 1 permit ^[0-9]+$
route-map BGP_CONNECTED_AS permit 10
 match as-path 1

router bgp 1
 address-family ipv4
  neighbor 10.0.12.2 route-map BGP_CONNECTED_AS in
  neighbor 10.0.12.2 filter-list 1 in
```

This matches all numbers but does not allow blank spaces. Meaning that there can only be one AS in the path, which is the neighbors AS. The `+` means that the pattern must appear, so blank AS (our own) will not match.

| Фильтр  | Описание  |
|---|---|
|`^$` |Local AS|
|`.*` |All AS|
|`^5_` |Directly connected AS 5|
|`_5_` |Transferring AS 5|
|`_5$` |Originated in AS 5|
|`^[0-9]+$` |Originated in directly connected AS|
|`^[0-9]*$` |Originated in directly connected AS + empty AS (local AS)|
|`^([0-9]+)_5` |AS 5 which passed through directly connected AS|
|`^2_([0-9]+)` |Directly connected to AS 2|
|`^\(1234\)` |Confederation peer 1234.|
|`_(4\|5)$` |Originated in AS 4 or AS 5|
|`^(2\|3)$` |Originated in AS 2 or AS 3 that are directly connected|
|`_2_(4\|5)$` |Originated in AS 4 or AS 5 that passed through AS 2|
|`^[0-9]+([0-9]+)?$` |Originated in directly connected AS or directly connected to our directly connected AS|

`?` Basically means true or false, the secondary AS that are being matched can appear or not.

## Filtering PL / ACL

**BGP Extended Access-Lists Filtering**

```
ip access-list extended PREFIXES
 deny ip 2.0.0.0 0.0.0.255 host 255.255.255.252
 deny ip 2.0.0.0 0.0.0.255 host 255.255.255.254
 permit ip any any

router bgp 1
 address-family ipv4
  neighbor 10.0.12.2 distribute-list PREFIXES in
```

**BGP Prefix-List Filtering**

```
ip prefix-list PREFIXES deny 1.0.0.2/31
ip prefix-list PREFIXES deny 1.0.0.4/30
ip prefix-list PREFIXES permit 0.0.0.0/0 le 32

router bgp 2
 address-family ipv4
  neighbor 10.0.12.1 prefix-list PREFIXES in
```

**BGP Outbound Route Filtering (ORF)**
- Normally when a router filters incoming routes the peer will still advertise them.
- To signal the peer that some routes do not need to be advertised, configure an ORF based on the same prefix-list that is filtering.

```
ip prefix-list PREFIXES deny 1.0.0.1/32
ip prefix-list PREFIXES deny 1.0.0.2/32
ip prefix-list PREFIXES deny 1.0.0.3/32
ip prefix-list PREFIXES deny 1.0.0.4/32
ip prefix-list PREFIXES permit 0.0.0.0/0 le 32

router bgp 2
 neighbor 10.0.12.1 remote-as 1
 address-family ipv4
  network 192.168.0.2 m 255.255.255.255
  neighbor 10.0.12.1 activate
  neighbor 10.0.12.1 capability orf prefix-list send
  neighbor 10.0.12.1 prefix-list PREFIXES in

router bgp 1
 neighbor 10.0.12.2 remote-as 2
 address-family ipv4
  neighbor 10.0.12.2 activate
  neighbor 10.0.12.2 capability orf prefix-list receive
  network 192.168.0.1 m 255.255.255.255
  network 1.0.0.1 m 255.255.255.255
  network 1.0.0.2 m 255.255.255.255
  network 1.0.0.3 m 255.255.255.255
  network 1.0.0.4 m 255.255.255.255

show ip bgp neighbors 10.0.12.1 | s capabilities
show ip bgp neighbors 10.0.12.2 advertised-routes
```

## Misc
**BGP Fall-over**
- Holdtime does not have to expire in order to tear down the session.
- The fall-over will still allow bgp to form neighborships using a default or summary route.
- Configure a route-map to make sure that the specific route has to be present in the routing table.

```
ip route 192.168.0.2 255.255.255.255 10.0.12.2

ip prefix-list R2_LOOPBACK permit 192.168.0.2/32
route-map R2_FALLOVER permit 10
 match ip address prefix-list R2_LOOPBACK

router bgp 1
 neighbor 192.168.0.2 fall-over route-map R2_FALLOVER
```

**BGP Next Hop Tracking (NHT)**
- On-by-default feature that notifies BGP to a change in routing for BGP prefix next-hops.
- NHT makes the process of dealing with next-hop changes event-driven, instead of using the 60 second scanner process.
- The bgp nexthop trigger delay defines how long for the NHT process to delay updating BGP.

```
router bgp 1
 bgp nexthop trigger enable
 bgp nexthop trigger delay 5
```

**BGP Selective Address Tracking (SAT)**
- The route-map determines what prefixes can be seen as valid prefixes for next-hops.
- Allows for specific addresses as viable next-hops, and will pull prefixes from the bgp table if the next-hop does not match.
- Re-use the same route-map in fall-over to also tear down the session.

```
ip prefix-list LOOPBACKS permit 0.0.0.0/0 ge 29
route-map SAT permit 10
 match ip address prefix-list LOOPBACKS

router bgp 1
 address-family ipv4
  bgp nexthop route-map SAT
  neighbor 10.0.13.3 fall-over route-map SAT
  neighbor 10.0.12.2 fall-over route-map SAT
```

**BGP Multi-Session**
Use a different TCP session for each address-family:

```
router bgp 1
 neighbor 192.168.0.2 transport multi-session (must agree on both sides)
```

BGP TTL-Security
- Configuring TTL-Security automatically allows peering over multiple hops (without explicitly configuring multi-hop).
- Configuring a TTL-Security hop-count of 2 will cause the router to expect incoming packets have a TTL value of 253.
- If packets arrive with a TTL of lower than 253, they will be discarded.

```
router bgp 1
 bgp router-id 192.168.0.1
 neighbor 192.168.0.2 remote-as 2
 neighbor 192.168.0.2 update-source Loopback0
 neighbor 192.168.0.2 ttl-security hops 2
 address-family ipv4
  neighbor 192.168.0.2 activate
```

## Route-Reflector
**iBGP Route-Reflector**
- Because routes learned from other internal peers are not advertised to other iBGP neighbors, iBGP needs a full-mesh topology.
- This approach has drawbacks however when a lot of routers are present in the AS.
- Another solution is to use a Route-Reflector (RR) to basically re-advertise the routes to internal peers.

BGP Route-Reflectors Terminology:
- Route-Reflector (RR), the router that has to feature enabled on some or all interfaces.
- Route-Reflector Client, router peering with the RR for which the RR has the feature enabled.
- Non-Client, router that is in the same AS but is not enabled for the feature on the RR.

A Route Reflector reflects routes considered as best routes only.
- If more than one update is received for the same destination, only the BGP best route is reflected.
- A Route Reflector is not allowed to change attributes of the reflected routes including the next-hop attribute.

**Route Target Constraint**
With Route Target Constraint (RTC) the Route Reflector sends only wanted VPN4/6 prefixes to the PE.
-  'Wanted' means that the PE has a VRF importing the specific prefixes.

**ORIGINATOR_ID**
Route reflectors use the ORIGINATOR_ID and CLUSTER_LIST attributes for loop prevention.
- Each routing table entry will have the originating routers RID inserted by the RR.
- The RR will pass along routes to the RR-Clients, but it is these clients themselves who check the update for the presence of the RID (ORIGINATOR_ID). If this ORIGINATOR_ID matches their own, the update is denied.

**CLUSTER_LIST**
Set by RR by default to the value of the RID. The CLUSTER_ID identifies a group, or a single RR.
- RR can use this information to discover other RR present on the network.
- Prevents the installation of multiple routes in the BGP table that were reflected by RR neighbors.

Set the CLUSTER_ID to be equal on all reflectors:

```
router bgp 1
 bgp cluster-id 13.13.13.13
```

**Advertisement Rules**
- Routes received from iBGP are advertised to external peers.
- So there is no need for RR in this case.

Non-client -> iBGP = Forwarded to eBGP and RR-Clients.
- When a NC receives a route from an eBGP peer it will be forwarded to the RR, which forwards it to all eBGP peers and iBGP RR-Clients.

External route from RR-Client -> iBGP = Forwarded to eBGP, RR-Clients and non-clients
- When a RR-Client receives a route from an eBGP peer  it will be forwarded to the RR, which forwards it to all eBGP peers and iBGP peers.

External route from RR -> iBGP = Forwarded to eBGP, RR-Clients and non-clients
- When a RR receives a route from an eBGP peer it will be forwarded to all eBGP and iBGP peers.

## Session
BGP Session
- BGP uses TCP port 179 for destination and a random source port.
- Commands that affect the BGP session are: shutdown, password, update-source and modification of timers.
- When using the network statement, BGP looks in the routing table and verifies that the route exists.
- The subnet used in the network statement must match exactly in order to inserted into BGP.

If two BGP peers initiate a session, the router that initiated the session will use a random TCP port to set up  the TCP connection, the router that received the session request will use TCP port 179.
- The initiating router message will be OPEN_ACTIVE.
- The receiving router message will be OPEN_PASSIVE.
- The receiving router will also send the OPEN_ACTIVE message, but this will fail because it is no longer needed.

BGP Messages

| Сообщение  | Описание  |
|---|---|
|OPEN|First message sent between peers after a TCP connection has been established.<br>Confirmed by KEEPALIVE message. |
|KEEPALIVE|Sent periodically to ensure that the connection is live.|
|NOTIFICATION|Sent in response to errors or special conditions.<br>If a connection encounters an error, a NOTIFICATION message is sent and the connection is closed. |
|UPDATE|Advertises routes between BGP speakers.<br>Multiple routes that have the same path attributes can be advertised in a single message.<br>A single UPDATE message may simultaneously advertise a feasible route and withdraw multiple unfeasible routes from service. |

When BGP fast peering session deactivation is enabled, BGP will monitor the peering session with the specified neighbor.
- Adjacency changes are detected, and terminated peering sessions are deactivated in between the default or configured BGP scanning interval.

BGP States

| Статус  | Описание  |
|---|---|
|IDLE|No peering, router is looking for neighbor. Connections refused.<br>Idle (admin) means that the neighbor relationship has been administratively shut down. |
|CONNECT|Establishing TCP session. Transitions to ACTIVE if TCP session has failed.|
|ACTIVE|An OPEN message is periodically sent to try to establish the TCP session peering.|
|OPENSENT|TCP session connection is established. Router is OPEN and expects an OPEN message from peer.|
|OPENCONFIRM|Expecting either KEEPALIVE (meaning that session is approved), or NOTIFICATION.|
|ESTABLISHED|Routers have a BGP peering session. UPDATES are sent between BGP speakers.|
## Table-Map
**BGP Table-Map**
- Sits between the BGP table and the RIB. Does not filter bgp prefixes received.
- Can be used to match prefixes and apply tags or communities.
- Can also be used to filter BGP prefixes from the local RIB with the filter keyword.
- Typically used to apply QoS settings and tags/communities.
- Applies to all prefixes received from all peers.

Filter specific prefixes from the RIB:

```
ip prefix-list DENY_192 permit 192.168.0.2/32
route-map TABLE_MAP_DENY deny 10
 match ip add prefix DENY_192

router bgp 1
 add ipv4
  table-map TABLE_MAP_DENY filter

clear ip bgp ipv4 unicast table-map
```

Apply QoS to specific communities:

```
ip community-list standard 2:2 permit 2:2
ip community-list standard 3:3 permit 3:3

route-map TABLE_MAP_QOS permit 10
 match community 2:2
 set ip precedence 2
route-map TABLE_MAP_QOS permit 20
 match community 3:3
 set ip precedence 3
route-map TABLE_MAP_QOS permit 99

router bgp 1
 add ipv4
  table-map TABLE_MAP_QOS
```


Verify QoS application to routes:

```
show ip cef 192.168.0.1/32
```

Configure the ingress interface to perform classification based on IPP:

```
int fa0/0
 bgp-policy source ip-prec-map
 bgp-policy destination ip-prec-map
```

**BGP Table-Map Automatic-Tag**

```cisco
ip as-path access-list 1 permit .*
route-map AS_ORIGIN_TABLE_MAP permit 10
 match as-path 1
 set automatic-tag

router bgp 2
 add ipv4
  table-map AS_ORIGIN_TABLE_MAP

clear ip bgp ipv4 unicast table-map
```

# Cryptography
Internet Key Exchange (IKE)
- IKEv1 uses UDP port 500. UDP port 4500 is used for NAT-Traversal in IKEv2.
- Encapsulation Header (ESP) is UDP port 50. Authentication Header (AH) is UDP port 51.

IKE Phase 1 (P1)
- IKE authenticates IPsec peers.
- Negotiates IKE Security Associations (SAs).
- Sets up a secure channel for negotiating IPsec SAs in P2.
- Protects the identities of IPsec peers.
- Main mode hides the identities of the two sides, but takes more time to negotiate. Default mode.
- Aggressive mode takes less time to negotiate at the cost of less security.
- MM and AG states differ, aggressive mode skips the SA_SETUP state. Both lead to QM_IDLE.

IKE Phase 2 (P2)
- IKE negotiates IPSec SA parameters.
- Sets up matching IPSec SAs in the peers.
- Sets up protection suite using ESP or AH.
- Two SAs are set up. One for sending and one for receiving traffic.
- Multiple P2 SAs can be established over the same P1 SA.
- Only one state, QM_IDLE.

Main Mode States
- MM_NO_STATE.
- MM_SA_SETUP. The peers have agreed on parameters for the ISAKMP SA.
- MM_KEY_EXCH. Exchanged DH information but remains unauthenticated.
- MM_KEY_AUTH. Authenticated.

Aggressive Mode States
- AG_NO_STATE.
- AG_INIT_EXCH. Exchanged DH information but remains unauthenticated.
- AG_AUTH. Authenticated.

## Crypto Maps
**Crypto Maps**
- Requires specification of traffic (ACL) in order to function.
- Applied on physical interface.
- The VPN will only be initiated when traffic is generated that matches the ACL.

```
show crypto map
show crypto isakmp profile

debug crypto isakmp
debug crypto ipsec
```

**Main Mode**

```
crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 life time 3600

crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto isakmp key cisco address 10.0.12.2

crypto map CMAP 10 ipsec-isakmp
 set peer 10.0.12.2
 set transform-set TS
 match address VPN
 qos pre-classify

ip access-list extended VPN
 permit ip host 192.168.0.1 host 192.168.0.2

ip route 192.168.0.2 255.255.255.255 10.0.12.2

int fa0/0
 crypto map CMAP
```

**Aggressive Mode**

```
crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 life time 3600

crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto keyring KEYRING
 pre-shared-key address 10.0.12.2 key cisco

crypto isakmp profile ISAKMP
 keyring KEYRING
 initiate mode aggressive
 match identity address 10.0.12.2

crypto map CMAP 10 ipsec-isakmp
 set peer 10.0.12.2
 set transform-set TS
 match address VPN
 qos pre-classify
 set isakmp ISAKMP

ip access-list extended VPN
 permit ip host 192.168.0.1 host 192.168.0.2

ip route 192.168.0.2 255.255.255.255 10.0.12.2

int fa0/0
 crypto map CMAP
```

## Dynamic VTI
**Dynamic Virtual Tunnel Interfaces (VTI)**
- Provides a separate virtual interface for each VPN session cloned from virtual template.
- When using EIGRP the split-horizon and next-hop-self configuration needs to be placed on the virtual-template.
- The VPN will be initiated even if no traffic is generated.

Hub configuration:

```
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 3600

crypto keyring KEYRING
 pre-shared-key address 123.0.0.2 key cisco
 pre-shared-key address 123.0.0.3 key cisco

crypto isakmp profile ISAKMP
   keyring KEYRING
   match identity address 123.0.0.2  
   match identity address 123.0.0.3  
   virtual-template 123

crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile IPSEC
 set transform-set TS
 set isakmp-profile ISAKMP

int lo1
 ip add 10.0.123.1 255.255.255.0

int virtual-template 123 type tunnel
 ip unnumbered loopback 1
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC
 no ip next-hop-self eigrp 123
 no ip split-horizon eigrp 123

router eigrp 123
 network 10.0.123.0 0.0.0.255
 network 192.168.0.1 0.0.0.0
```

Spoke configuration:

```
crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 3600

crypto isakmp key cisco address 123.0.0.1
crypto isakmp key cisco address 123.0.0.3

crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile IPSEC
 set transform-set TS  

interface tun0
 ip add 10.0.123.2 255.255.255.0
 tunnel source fa0/0
 tunnel destination 123.0.0.1
 tunnel mode ipsec ipv4
 ip mtu 1400
 tunnel protection ipsec profile IPSEC

router eigrp 123
 network 10.0.123.0 0.0.0.255
 network 192.168.0.2 0.0.0.0
```

## Static VTI
**Static Virtual Tunnel Interfaces  (VTI)**
- Default tunnel mode is GRE, optionally change to IPsec IPv4 / IPv6.
- The VPN will be initiated even if no traffic is generated.

```
crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 3600

crypto isakmp key cisco address 12.0.0.2

crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile IPSEC
 set transform-set TS  

int tun0
 ip add 10.0.12.1 255.255.255.0
 tunnel source 12.0.0.1
 tunnel destination 12.0.0.2
 ip mtu 1400
 tunnel protection ipsec profile IPSEC
 qos pre-classify

router eigrp 12
  network 10.0.12.0 0.0.0.255
  network 192.168.0.1 0.0.0.0
```

# DMVPN
Dynamic Multipoint VPN (DMVPN) Phases

| Phase | GRE-Mode | Dynamic Tunnels | Summarization / Default-Route |
| ---- | ---- | ---- | ---- |
| 1 | mGRE on hub, GRE on spokes | no | Allowed |
| 2 | mGRE on all | yes | Allowed, but lose dynamic tunnel functionality |
| 3 | mGRE on all | yes | Allowed |

DMVPN Phase 1
- Basically traditional Hub-and-Spoke topology without dynamic tunnels.
- Configure spokes with `tunnel destination <hub nbma address>` and `tunnel mode gre` on the tunnel interface.

DMVPN Phase 3 Additions
- The `ip nhrp redirect` is configured on the hub and works similar to IP redirect. Informs spokes of the location of others.
- When a hub receives and forwards packet out of same interface it will sent a NHRP redirect message back to the source.
- The original packet from the source is not dropped but forwarded down to other spoke via RIB.
- The `ip nhrp shortcut` is configured on spokes and rewrites the CEF entry after getting the redirect message.

Next Hop Resolution Protocol (NHRP)
- ARP-like protocol that dynamically maps a NBMA network.
- NHRP works similar to Frame-Relay DLCI but instead maps NMBA addresses to tunnel addresses.
- The hub can only communicate with spokes after spokes have registered to the hub.
- The hub is the Next Hop Server (NHS) and the spokes are the Next Hop Clients (NHCs).
- NHRP allows one NHC to dynamically discover another NHC within the network and build tunnels dynamically (resolution).

NHRP Dynamic Flags

|   |   |
|---|---|
|Authoritative|Obtained from NHS.|
|Implicit|Obtained from forwarded NHRP packet .|
|Negative|Could not be obtained.|
|Unique|Request packets are unique, disable if spoke has a dynamic outside IP address.|
|Registered|Obtained from NHRP registration request (Seen on hub).<br><br>The spoke has instructed the hub not to take a registration from another identical NBMA address.|
|Used|Set when data packets are process switched and mapping entry is in use, 60s timer.|
|Router|NHRP mapping entries for the remote router for access to network.|
|Local|Local network mapping.|

## BGP
**iBGP DMVPN Recommendations**
- Configure Hub as Route-Reflector.
- Configure Spokes as Route-Reflector Clients.
- Use Peer-Groups for scalable config.

P3 hub configuration:

```
int fa0/0
 ip add 123.0.0.1 255.255.255.0

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip address 10.0.123.1 255.255.255.0
 ip nhrp redirect

router bgp 123
 bgp listen range 10.0.123.0/24 peer-group DMVPN
 bgp listen limit 2
 neighbor DMVPN peer-group
 neighbor DMVPN remote-as 123
 address-family ipv4
  neighbor DMVPN route-reflector-client
  neighbor DMVPN activate
```

P3 spoke configuration:

```
int fa0/0
 ip add 123.0.0.2 255.255.255.0

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoin
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast 123.0.0.1
 ip nhrp nhs 10.0.123.1 nbma 123.0.0.1
 ip address 10.0.123.2 255.255.255.0
 ip nhrp shortcut

router bgp 123
 neighbor 10.0.123.1 remote-as 123
 address-family ipv4
  neighbor 10.0.123.1 activate
```

## EIGRP
EIGRP DMVPN Recommendations

| Phase | Next-Hop-Self | Split-Horizon |
| ---- | ---- | ---- |
| 1 | Enabled | Disable when using specific routes  <br>Enable when using default summary |
| 2 | Disable | Disable |
| 3 | Enabled | Disable when using specific routes  <br>Enable when using default summary |

**EIGRP add-path support for DMVPN**
- Enables hubs to advertise up to four best paths to connected spokes.
- Disable next-hop-self for add-paths to operate.
- Can only be enabled named configuration.
- Should not be configured alongside variance.

P3 hub configuration:

```
int fa0/0
 ip add 123.0.0.1 255.255.255.0

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip address 10.0.123.1 255.255.255.0
 ip nhrp redirect

router eigrp DMVPN
 add ipv4 au 123
  network 10.0.123.0 0.0.0.255
 af-interface tun0
  summary-address 0.0.0.0/0
  no next-hop-self no-ecmp-mode
  no split-horizon
  add-paths 4
```

P3 spoke configuration:

```
int fa0/0
 ip add 123.0.0.2 255.255.255.0

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast 123.0.0.1
 ip nhrp nhs 10.0.123.1 nbma 123.0.0.1
 ip address 10.0.123.2 255.255.255.0
 ip nhrp shortcut

router eigrp DMVPN
 add ipv4 au 123
  network 10.0.123.0 0.0.0.255
```
  
## Misc
**DMVPN Encryption**
- Mode tunnel adds 20 bytes overhead and is only used in a multi-tier DMVPN hub.
- One set of routers running cryptography and another set performing NHRP services.
- Mode transport is preferred for single-hub configurations.

```
crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode transport
```

**DMVPN QoS**
- Apply a QoS policy on a DMVPN hub on a tunnel instance in the egress direction.
- Shape traffic for individual spokes (parent policy). Policy data flows going though the tunnel (child policy).
- Defined by NHRP group, each spoke can be managed individually using Per-Tunnel QoS.
- The spoke can belong to only one NHRP group per GRE tunnel interface.

```
policy-map R2_PM
 class class-default
  shape average 50 m
policy-map R3_POLICY
 class class-default
  shape average 25 m

int tun0
 ip nhrp map group R2 service-policy output R2_PM
 ip nhrp map group R3 service-policy output R3_PM

show policy-map multipoint
```

Spoke configuration:

```
int tun0
 ip nhrp group R2
```

**DMVPN PIMv2**
- PIM does not allow multicast traffic to leave the same interface it was received on.
- Override this behavior with ip pim nbma-mode configured on the tunnel interface.

## ODR
**DMVPN On-Demand Routing (ODR)**
- Disable CDP on outside (physical) interfaces. Enable CDP on tunnel interfaces.
- ODR timers and CDP timers should be the same.
- Configure on hub only, spokes will receive a default route via ODR and will advertise their connected subnets.
- In P2 all traffic will still flow through the hub, ODR needs P3 in order to create dynamic tunnels.

P3 hub configuration:

```
int fa0/0
 ip add 123.0.0.1 255.255.255.0

cdp timer 20
cdp holdtime 60

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip address 10.0.123.1 255.255.255.0
 ip nhrp redirect
 cdp enable

router odr
 timers 20 60 60 90
```

P3 spoke configuration:

```
int fa0/0
 ip add 123.0.0.2 255.255.255.0

cdp timer 20
cdp holdtime 60

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast 123.0.0.1
 ip nhrp nhs 10.0.123.1 nbma 123.0.0.1
 ip address 10.0.123.2 255.255.255.0
 ip nhrp shortcut
 cdp enable
```

## OSPF
**OSPF DMVPN Recommendations**
- It is vital to the workings of OSPF that the hub is the DR in broadcast and non-broadcast networks.
- OSPF is not recommended for DMVPN because it is not possible to summarize within the same area.
- The DMVPN topology is most likely going to be part of the backbone area.
- PTMP will not create dynamic tunnels in P2. Broadcast is recommended for P2.
- PTMP is recommended for P1 and P3.

P3 hub configuration:

```
int fa0/0
 ip add 123.0.0.1 255.255.255.0

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip address 10.0.123.1 255.255.255.0
 ip ospf network point-to-multipoint
 ip ospf 123 area 0
 ip nhrp redirect
```

P3 spoke configuration:

```
int fa0/0
 ip add 123.0.0.2 255.255.255.0

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast 123.0.0.1
 ip address 10.0.123.2 255.255.255.0
 ip nhrp nhs 10.0.123.1 nbma 123.0.0.1
 ip ospf network point-to-multipoint
 ip ospf 123 area 0
 ip nhrp shortcut
```

**OSPF DMVPN Filter**
- You can only filter routes from being added to the RIB, not LSAs from being received.
- Basically filter every route on the spokes except for the default route.

Hub configuration:

```
router ospf 123
 default-information originate always
```

Spoke configuration:

```
access-list 1 permit host 0.0.0.0
router ospf 123
 distribute-list 1 in
```

## RIP
**RIP DMVPN Recommendations**

| Phase | Split-Horizon |
| ---- | ---- |
| 1 | Disable when using specific routes  <br>Enable when using default summary |
| 2 | Disable |
| 3 | Disable when using specific routes  <br>Enable when using default summary |

P3 hub configuration:

```
int fa0/0
 ip add 123.0.0.1 255.255.255.0

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip address 10.0.123.1 255.255.255.0
 ip nhrp redirect

router rip
 version 2
 network 10.0.0.0
 no auto-summary
```

P3 spoke configuration:

```
int fa0/0
 ip add 123.0.0.2 255.255.255.0

int tun0
 tunnel source fa0/0
 tunnel mode gre multipoint
 ip nhrp network-id 123
 ip mtu 1400
 ip nhrp map multicast 123.0.0.1
 ip nhrp nhs 10.0.123.1 nbma 123.0.0.1
 ip address 10.0.123.2 255.255.255.0
 ip nhrp shortcut

router rip
 version 2
 network 10.0.0.0
 no auto-summary
```

## IPv6
**IPv6 DMVPN over IPv4 NBMA**
- IPv4 for the physical interface.
- IPv6 for the tunnel interface.
- Uses IPv4 NBMA address on nhs command.
- Uses IPv6 NHRP commands.

P3 hub configuration:

```
int fa0/0
 ip add 10.0.123.1 255.255.255.0

interface tun0
 tunnel source 10.0.123.1
 tunnel mode gre multipoint ipv6
 ipv6 nhrp map multicast dynamic
 ipv6 nhrp network-id 123
 ipv6 mtu 1400
 ipv6 address FE80::1 link-local
 ipv6 address 2001:10:0:123::1/64
 ipv6 nhrp redirect
```

P3 spoke configuration:

```
int fa0/0
 ip add 10.0.123.2 255.255.255.0

interface tun0
 tunnel source 10.0.123.2
 tunnel mode gre multipoint
 ipv6 nhrp map multicast 123.0.0.1
 ipv6 nhrp network-id 123
 ipv6 nhrp nhs 2001:10:0:123::1 nbma 123.0.0.1
 ipv6 mtu 1400
 ipv6 address FE80::2 link-local
 ipv6 address 2001:10:0:123::2/64
 ipv6 nhrp shortcut
```

**IPv6 DMVPN over IPv6 NBMA**
- IPv6 for the physical interface.
- IPv6 for the tunnel interface.
- Uses IPv6 NBMA address on nhs command.
- Uses IPv6 NHRP commands.

P3 hub configuration:

```
int fa0/0
 ipv6 add 2001:123::1/64
 ipv6 address FE80::1 link-local

int tun0
 ipv6 mtu 1400
 tunnel source 2001:123::1/64
 tunnel mode gre multipoint ipv6
 ipv6 address FE80::1 link-local
 ipv6 address 2001:10:0:123::1/64
 ipv6 mtu 1400
 ipv6 nhrp map multicast dynamic
 ipv6 nhrp network-id 123
 ipv6 nhrp redirect
```

P3 spoke configuration:

```
int fa0/0
 ipv6 add 2001:123::2/64
 ipv6 address FE80::2 link-local

int tun0
 ipv6 mtu 1400
 tunnel source 2001:123::2/64
 tunnel mode gre multipoint ipv6
 ipv6 address FE80::2 link-local
 ipv6 address 2001:10:0:123::2/64
 ipv6 nhrp map multicast 2001:123:0:0::1
 ipv6 nhrp network-id 123
 ipv6 nhrp nhs 2001:10:0:123::1 nbma 2001:123::1
 ipv6 mtu 1400
 ipv6 nhrp shortcut
```

**IPv4 DMVPN over IPv6 NBMA**
- IPv6 for the physical interface.
- IPv4 for the tunnel interface.
- Uses IPv6 NBMA address on `nhs` command.
- Uses IPv4 NHRP commands.

P3 hub configuration:

```
int fa0/0
 ipv6 add 2001:123::1/64
 ipv6 address FE80::1 link-local

interface tun0
 ip mtu 1400
 tunnel source 2001:123::1
 tunnel mode gre multipoint ipv6
 ip nhrp map multicast dynamic
 ip nhrp network-id 123
 ip address 10.0.123.1 255.255.255.0
 ip nhrp redirect
```

P3 spoke configuration:

```
int fa0/0
 ipv6 add 2001:123::2/64
 ipv6 address FE80::2 link-local

interface tun0
 ip mtu 1400
 tunnel source 2001:123::2
 tunnel mode gre multipoint ipv6
 ip nhrp map multicast dynamic
 ip nhrp network-id 123
 ip address 10.0.123.2 255.255.255.0
 ip nhrp nhs 10.0.123.1 nbma 2001:123::1
 ip nhrp shortcut
```

# EIGRP
**EIGRP Vector Metrics**
- Changing the `bandwidth` or `delay` on the interface will cause the router to re-advertise the route.
- Changing the `load` or `reliability` will not cause to router to re-advertise the route.

**EIGRP Best Path Selection**
- EIGRP uses Reliable Transport Protocol (RTP) for guaranteed, ordered delivery of EIGRP packets to all neighbors.
- Supports multicast and unicast, and uses IP protocol 88. Multicast address is `224.0.0.10 and FF02::A.`
- Only some EIGRP packets are sent reliably. For efficiency, reliability is provided only when necessary.
- EIGRP updates are sent to the multicast address and acknowledgements are replied via unicast.

**EIGRP Timers**
- The configured `hold-time` is communicated to the neighbor on the segment. This hold-time is included in the hello message.
- The neighbor receives this and will expect a new hello from the router within this time. These timers do not have to match.

## Filtering
**Extended Access-List**

Deny even from R2 and odd from R3:

```
ip access-list extended 100
 deny ip host 10.0.12.2 4.0.0.0 0.0.0.254
 deny ip host 10.0.13.3 4.0.0.1 0.0.0.254
 permit ip any any

router eigrp 1
 distribute-list 100 in
```

**Prefix-List**

Deny all prefixes from R2:

```
ip prefix-list DENY_R2 deny 10.0.12.2/32
ip prefix-list DENY_R2 permit 0.0.0.0/0 le 32
ip prefix-list PREFIXES permit 0.0.0.0/0 le 32

router eigrp 1
  distribute-list prefix PREFIXES gateway DENY_R2 in
```

Only accept prefixes from R3:

```
ip prefix-list R3 permit 10.0.13.3/32

router eigrp 1
 distribute-list gateway R3 in
```

Deny specific prefixes from R2:

```
ip prefix-list R2 permit 10.0.12.2/32
ip prefix-list NETWORKS deny 4.0.0.1/32
ip prefix-list NETWORKS deny 4.0.0.2/32
ip prefix-list NETWORKS permit 0.0.0.0/0 le 32

router eigrp 1
  distribute-list prefix NETWORKS gateway R2 in fa0/0
```

## Metric
EIGRP Composite Metric (Weight Calculation)
- EIGRP uses metric weights along with a set of vector metrics to compute the composite metric for local RIB and route selections.
- Type of service (first K value) must always be zero.  
- The formula is `[K1*bandwidth + (K2*bandwidth)/(256 - load) + K3*delay] * [K5/(reliability + K4)]`.

`256*[(10^7/slowest bandwidth in kbps) + all link delays in tens microseconds]`

```
router eigrp
 metric weights 0 1 0 1 0 0
```

**EIGRP Wide Metrics**
- The EIGRP Wide Metric feature supports 64-bit metric calculations and Routing Information Base (RIB) scaling.
- The lowest delay that can be configured for an interface is 10 microseconds with 32-bit metrics (normal).
- Use on interfaces that are faster than 1Gbps.
- The new metrics no longer fit in the output of the RIB, because this is limited to 32bit numbers.
- The topology table will show the correct metric numbers which is divided by the value specified in the rib-scaling.
- The formula is `[K1*bandwidth + (K2*bandwidth)/(256 - load) + K3*delay +K6*Ext Attr] * [K5/(reliability + K4)]`
- K6 is an additional K value for future use. Other K values remain the same with K1 and K3 set to 1, and K2,K4,K5 set to 0.

```
router eigrp EIGRP
 address-family ipv4 unicast autonomous-system 1
  metric rib-scale 128
```

The 64-bit metric calculations work only in EIGRP named mode configurations. EIGRP classic mode uses 32-bit metric calculations.
- EIGRP named mode automatically uses wide metrics when speaking to another EIGRP named mode process.

**EIGRP Offset Lists**
- The offset list increases the existing metric by a specified amount. It is not possible to decrease a metric.
- The offset list modifies the `delay` value, not the `bandwidth` value. Meaning that it is included in the cumulative delay.
- Using `offset-list 0` will apply the offset to all networks. Using an empty access list has the same effect.

An offset list will only influence the calculated metric, and thus the composite metric. The composite metric is only used for local calculation and is not communicated to neighbors. However, routing paths through the router between neighbors will still add the offset value to the metric. This is because the offset list changes the delay value which is cumulative in the total metric for the route.

EIGRP Unequal Cost Multi-Path Load-Sharing
- Only feasible successors are candidate for load balancing.
- The `traffic-share min accross-interfaces` only uses the primary path in case of a UCMP configuration using variance.
- This will stop the route from using UCMP. The point of this configuration is to speed up convergence.
- When calculating the eigrp traffic share count, divide numbers to figure out metric.
- A preffered traffic share count of two routes, one 10 the other `3 -> 10/3 = 3.33 * metric value`.

```
router eigrp 1
 variance 5
 traffic-share min accross-interfaces
```

**Default Metric & Redistribution**
- Only connected and static routes can be redistributed without a default metric. This metric is set to 0.
- IPv6 networks add the `include-connected` keyword to include connected networks as well.
- In IPv4 connected networks are included in the redistribution by default.

**Metric Maximum-Hops**
- The maximum hops is a greater than statement. Default is 100 hops.
- If 10 is entered for example, the prefixes 10 hops away are still valid. The prefixes 11 hops away are denied.

```
router eigrp 1
 metric maximum-hops 10
```

## Misc
**EIGRP Neighbor Statement**
- Limit neighbor ship with specific neighbors by configuring neighbor statements (on the same segment).
- Alternative is to use ACL to block specific link-local addresses (IPv6).

```
ipv6 router eigrp 1
 neighbor FE80::2 fa0/0
 neighbor FE80::3 fa0/0
```

Or:

```
ipv6 access-list EIGRP_BLOCK
 deny 88 fe80::2/128  any
 deny 88 fe80::3/128  any
 permit ipv6 any any

int fa0/0
 ipv6 traffic-filter EIGRP_BLOCK in
```

**EIGRP Router-ID**
- Duplicate router-IDs (RID) do not show up in the logging or debug.  Use the `show ip eigrp events | i Ignored` command.
- EIGRP uses the concept of the RID as a loop-prevention mechanism to filter out a routers own routes.
- In the event of a duplicate RID the neighbors routes will not be installed.

**EIGRP Authentication**
- It is possible to configure multiple keys. Only one authentication packet is sent, regardless of how many keys exist.
- The software examines the key numbers in the order from lowest to highest, and uses the first valid key that it encounters.
- HMAC-SHA-256 authentication is only available in named mode. Does not support key-chains.
- Named mode ignores all authentication (and other) commands configured on the local interface using the old method.

```
key chain EIGRP_KEY
 key 1
  key-string cisco

int fa0/0
 ip authentication key-chain eigrp 1 EIGRP_KEY
 ip authentication mode eigrp 1 md5
```

HMAC-SHA-256 using named configuration:

```
router eigrp EIGRP
 address-family ipv4 autonomous-system 1
  af-interface fa0/0
   authentication mode hmac-sha-256 cisco
```

**Dampening Change & Interval**
- Dampening controls the update of metric changes of routes advertised by neighbors.
- Dampening compares the old metric for the route with the new metric.
- Using `dampening-change`, if this new metric is within the percentage threshold, the update will be ignored.
- Using `dampening-interval`, if this new metric is within the configured time interval, the update will be ignored.
- Dampening is disabled by default.

```
int fa0/0
 ip dampening-change eigrp 1 percent 75
 ip dampening-interval eigrp 1 seconds 60
```

**Next-Hop-Self**
- Disable next-hop-self on the redistributing router when using 3rd party next hop.
- This is used when OSPF and EIGRP coexist on the same segment, and one router is used for redistributing both protocols.
- Normally all traffic would go through the redistributing router, with 3rd party next hop the neighbors can communicate directly.
- Requirement is to disable `next-hop-self` on the interface of the redistributing router towards the shared segment.
- Other situations where `next-hop-self` might be disabled is in DMVPN solutions.

```
int fa0/0
 description SHARED_OSPF_EIGRP_SEGMENT
 ip add 10.0.123.1 255.255.255.0
 no ip next-hop-self eigrp 1

router eigrp 1
 network 10.0.123.0 0.0.0.255
 redistribute ospf 1

router ospf 1
 network 10.0.123.0 0.0.0.255 area 0
 redistribute eigrp 1 subnets
```

**Bandwidth Percentage**
- By default, EIGRP packets are allowed to consume a maximum of 50 percent of the link bandwidth.
- The bandwidth is the configured `bandwidth`, not the original interface bandwidth.
- Note that values greater than 100 percent may be configured.
- This configuration option may be useful if the bandwidth is set artificially low for other reasons.

```
int fa0/0
 ip bandwidth-percent eigrp 75
```

**Administrative Distance**
- Changes to the administrative distance of all internal and external routes is limited to the router itself.
- Can be limited to a specific neighbor, only one specific distance command can be entered per neighbor.

```
ip access-list standard NETWORK
 permit host 172.16.0.0

router eigrp 1
 distance eigrp 90 170
 distance 85 10.0.12.2 0.0.0.0
 distance 75 10.0.13.3 0.0.0.0 NETWORK
```

**EIGRP Loop-Free Alternate Fast Reroute (FRR)**
- Uses repair paths or backup routes and installs these paths in the RIB.
- FRR picks the best feasible successor and places it in the FIB as a backup route.

```
router eigrp EIGRP
 address-family ipv4 autonomous-system 1
  topology base
   fast-reroute per-prefix all
```

## Route-Tags
**Route Tag Enhancements**
- The route tag enhancements allow the route tag to be formatted as a dotted decimal tag.
- These can me matched either directly (in the traditional route tag method in route-map) or via a route-tag list.
- EIGRP named mode provides the option of assigning a default-route-tag to all routes sourced by the router.

```
route-tag notation dotted-decimal
ip access-list standard Lo1
 permit host 4.0.0.1
ip access-list standard Lo2
 permit host 4.0.0.2

ip prefix-list Lo3 permit 4.0.0.3/32

route-map LOOPBACKS permit 10
 match ip address Lo1
 set tag 44.44.1.1
route-map LOOPBACKS permit 20
 match ip address Lo2
 set tag 44.44.2.1
route-map LOOPBACKS permit 30
 match ip address prefix-list Lo3
 set tag 44.44.3.1
route-map LOOPBACKS permit 40
 match interface loopback 4
 set tag 44.44.4.1

router eigrp 1
 redistribute connected route-map LOOPBACKS
```

Route-tag Even Filtering:

```
route-tag notation dotted-decimal
route-tag list LOOPBACKS permit 44.44.0.0 0.0.254.255

route-map LOOPBACKS permit 10
 match tag list LOOPBACKS
 set metric 50000 581 255 1 1500
route-map LOOPBACKS permit 20

router eigrp 1
 distribute-list route-map LOOPBACKS in fa0/0
```

Default Route-Tags:

```
router eigrp EIGRP
 address-family ipv4 unicast autonomous-system 1
  eigrp default-route-tag 192.168.0.1
```

## Summarization
**EIGRP Route Summarization**
- Summary routes are always internal, even if external routes are summarized.
- The summary route has an AD of 5 and is called the discard route which points to Null0 on the summarizing router.
- The neighbor will receive an internal route with an AD of 90.
- Disable the discard route by specifying a `summary-metric` distance of 255.
- Configure a `leak-map` to allow components of the summary to advertised alongside the summary.
- Stop IP ICMP unreachables based on discard route with no `ip unreachable`s configured on null0 interface.

```
int fa0/0
 ip summary-address eigrp 1 0.0.0.0/0

router eigrp 1
 summary-metric 0.0.0.0/0 distance 255
```

Leak-Map
Allow the 10.0.12.0/24 network to be advertised alongside the summary:

```
ip prefix-list NETWORK permit 10.0.12.0/24
route-map LEAK_MAP permit 10
 match ip address prefix-list NETWORK

int fa0/0
 ip summary-address eigrp 1 10.0.0.0/16 leak-map LEAK_MAP
```

## Stub
EIGRP Stub
- EIGRP query messages live for 3 minutes by default.  Can be modified with the `timers active-time` command.
- Queries are not sent if a feasible successor is present in the topology for the route.
- A router will wait for a reply for all neighbors before failing over to a different path. This is called Stuck in Active (SIA).
- Fix SIA with summary routes or stub routers.
- A leak map can allow routes that would normally have been suppressed.

Allow prefixes from 172.16.1.0/24 alongside the connected and summary routes:

```
int fa0/0
 ip summary-address eigrp 1 172.16.0.0/16

ip prefix-list STUB_PREFIX permit 172.16.1.0/24

route-map STUB_LEAK_MAP permit 10
 match ip address prefix STUB_PREFIX

router eigrp 1
 eigrp stub connected summary leak-map STUB_LEAK_MAP
```

# Frame-Relay (v4)
**L1 and Frame-Relay Terminology**
- L1 DTE = Data Termination Equipment
- L1 DCE = Data Communication Equipment
- Frame-Relay switch is called the DTE, endpoints are called DCE. This is unrelated to L1 DCE/DTE.
- FR DCE responds to LMI inquiries by sending LMI status, never sends LMI inquiries.
- FR DTE sends LMI inquiries, never sends LMI status.
- DLCIs are only locally significant.

Inverse ARP
- Because serial interfaces and FR are NBMA networks, there is no ARP flooding to discover neighbor addresses.
- Instead FR relies on inverse ARP to allow clients to notify neighbors of their presence and reply to requests.
- Disabling inverse ARP will disable the notification, these are called dynamic mappings.
- Has to be disabled on all routers to disable automatic learning of mappings.
- Inverse-ARP does not allow self-ping.

```
int s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp

clear frame inarp
```

**Frame-Relay Mappings**
- If dynamic mappings have been disabled, static ones need to be created on the clients (R1,R3).
- The `broadcast` keyword allows broadcast packets over NBMA.
- Only configure `broadcast` on one statement, preferably the statement that configures the router to ping itself.

```
int se1/0
 ip address 10.0.13.1 255.255.255.0
 frame-relay map ip 10.0.13.1 103 broadcast
 frame-relay map ip 10.0.13.3 103

int se1/0
 ip address 10.0.13.3 255.255.255.0
 frame-relay map ip 10.0.13.1 301
 frame-relay map ip 10.0.13.3 301 broadcast

show frame pvc
show frame map
```

**Frame-Relay Switch**
- Set encapsulation to frame-relay.
- Configure DCE on interfaces
- Create FR PVCs using either frame routes (FR switching framework) or connections (L2VPN framework).

Create FR PVCs using  frame routes on FR Switch:

```
int se1/0
 encapsulation frame-relay
 frame-relay intf-type dce
 frame route 103 interface se1/1 301

int se1/1
 encapsulation frame-relay
 frame-relay intf-type dce
 frame route 301 interface se1/0 103

show frame route
show frame pvc
```

Create FR PVCs using connections on FR Switch:

```
connect R1-R3 se1/0 103 se1/1 301
show connection all
```

**Frame-Relay with Sub-interfaces**
- If sub-interfaces are being used, the DLCI needs to be linked to the sub-interface.
- Inverse ARP configuration is not inherited by sub-interfaces.
- With PTP mode sub-interfaces are configured on the hub, each to single location.
- With PTMP mode a single interface is configured  on the hub to multiple locations. Disable `split-horizon` if using EIGRP.
- The `no inverse-arp` configuration on the physical interface is not inherit by the sub-interface.

```
int se1/0.123 point-to-multipoint
 frame-relay interface-dlci 123

int se1/0.102 point-to-point
 frame-relay interface-dlci 102

int se1/0.103 point-to-point
 frame-relay interface-dlci 103
```

**Frame-Relay Authentication**
- Use point-to-point sub-interfaces configured with PPP encapsulation.
- Use `ip unnumbered` in order to enable self-ping.

R1 configuration:

```
username R3 password cisco
int se1/0.103 point-to-point
 frame-relay interface-dlci 103 ppp virtual-template 1

interface virtual-template 1
 ip address 10.0.13.1 255.255.255.0
 encapsulation ppp
 ppp authentication chap
 ppp pap sent-username R1 password cisco
```

R3 configuration:

```cisco
username R1 password cisco
int s1/1.301 point-to-point
 frame-relay interface-dlci 301 ppp virtual-template 1

interface virtual-template 1
 ip address 10.0.13.3 255.255.255.0
 encapsulation ppp
 ppp chap hostname R2
 ppp chap password cisco
 ppp authentication pap
```

# IP Routing
**Administrative Distance and Route Selection**
- Hardcoded original administrative distance will win if different routing protocols are configured to use the same AD.
- The metrics between different routing protocols or different routing processes are not compared in the route selection.
- When receiving the same route from different OSPF  processes with the same AD, the route learned first wins.
- OSPF does not differentiate between internal and external routes to the same destination. Only the AD matters.
- When receiving the same route from different EIGRP ASs with the same AD, the route from the lower AS wins.

**IP Source-Routing**
- Allows the originator of a packet to decide which routers the packet will flow through.
- Basically a custom path of all hops specified at the source and set in the actual IP header by the source.
- Enabled by default but is a security risk. Disable with no ip source-route command.

**IP Accounting**
- Counts the number of IP packets and logs source/destination.
- Only works for transit egress traffic, not local traffic.

```
int fa0/0
 ip accounting output-packets

show ip accounting
```

**IP Redirects**
- ICMP redirect messages are send by default when routers have to forward a packet on the same interface it was received.
- Routers will notify hosts of better next-hop through redirects.

```
int fa0/0
 no ip redirects
 no ipv6 redirects
```

**IP Unreachables**
- By default the router will respond with an IP unreachable ICMP message in case the neighbor router pings an unknown address.
- Disable to block UDP port scans. These have a destination of an unused or unreachable UDP port.
- IP packets for unknown destinations are sent to null0. Disable redirects for unknown traffic on the null0 interface.
- Other interfaces will still respond with unreachable if traffic is destined to the local interface address.

```
int fa0/0
 no ip unreachables  
 no ipv6 unreachables

int null0
 no ip unreachables  
 no ipv6 unreachables
```

**IP Local Proxy-ARP**
- Enable proxy of ARP request on the same subnet with the `ip local proxy-arp` interface command.
- The `ip proxy-arp` feature is for ARP requests to different subnets. Enabled by default.
- Use in Private-VLANs to allow communication between isolated hosts. Configure on the promiscuous port.
- Instead of configuring local proxy-arp, you can also statically configured the IP to MAC mappings for individual hosts.

```
arp 10.0.123.2 abcd.1234.abcd arpa
arp 10.0.123.3 1234.abcd.1234 arpa
```

**IP Directed-Broadcast (SMURF)**
- Disabled by default. Exploited in SMURF attacks.
- Enable with `ip directed-broadcasts` command on interfaces.

**Gratuitous ARP**
- Update ARP tables after a MAC address for an IP changes, or a MAC address is now on a different port.
- Sends special ARP packet when interface goes up to notify other hosts in advance so that ARP requests are not needed.
- Does not expect a reply. When a reply is received there is an IP address conflict in the network.
- Used by FHRP to update MAC tables on L2 devices with the virtual MAC address.

**IP Event Dampening**
- Suppress flapping interface effects on routing protocols and routing tables.
- Can only be configured on physical interfaces, not on sub-interfaces or virtual-templates.

```
int fa0/0
 dampening 5 1000 2000 20

show interfaces dampening
```

## Redistribution
Redistribution
- Redistribution only redistributes routes that are present in the RIB.
- There is no direct redistribution between protocols.
- Routing protocol redistribution also redistributes the connected networks that the protocol is enabled for.
- Include IGP interfaces when filtering redistributed connected routes (loopbacks).
- Another way to include the connected interfaces is to advertise them into the protocols and optionally configure as passive.
- OSPF default static route cannot be redistributed with the `redistribute static` command, even if a route-map is specified.
- Always redistribute the default route into OSPF using the `default-information originate` command.

BGP Redistribution
- Only internal OSPF routes will be redistributed into BGP by default.
- A default route learned by an IGP is not redistributed into BGP by default.
- Advertise the redistributed default route with network 0.0.0.0 mask 0.0.0.0.

```
ip route profile
show ip route profile
debug ip routing
```

Redistribution using Direct Tags

```
route-map EIGRP_ROUTES deny 10
 match tag 90
route-map EIGRP_ROUTES permit 99

router ospf 1
 redistribute eigrp 1 subnets tag 90
  distribute-list route-map EIGRP_ROUTES in
```

Mutual MultiPoint Redistribution using Prefix-Lists

```
ip prefix-list OSPF_ROUTES permit 3.0.0.1/32
ip prefix-list OSPF_ROUTES permit 3.0.0.2/32
ip prefix-list OSPF_ROUTES permit 3.0.0.3/32

ip prefix-list EIGRP_ROUTES permit 4.0.0.1/32
ip prefix-list EIGRP_ROUTES permit 4.0.0.2/32
ip prefix-list EIGRP_ROUTES permit 4.0.0.3/32

route-map EIGRP_TO_OSPF deny 10
 match ip address prefix-list OSPF_ROUTES
route-map EIGRP_TO_OSPF permit 20
 match ip address prefix-list EIGRP_ROUTES

route-map OSPF_TO_EIGRP deny 10
  match ip address prefix-list EIGRP_ROUTES
route-map OSPF_TO_EIGRP permit 20
 match ip address prefix-list OSPF_ROUTES

router eigrp 1
 redistribute ospf 1 metric 1000000 10 255 1 1500 route-map OSPF_TO_EIGRP

router ospf 1
 redistribute eigrp 1 metric-type 1 subnets route-map EIGRP_TO_OSPF
```

**Three-Way Redistribution using Tags**

```
route-map EIGRP_TO_RIP deny 10
 match tag 120
route-map EIGRP_TO_RIP permit 20
 match tag 110
 set tag 110
route-map EIGRP_TO_RIP permit 30
 set tag 90

route-map RIP_TO_EIGRP deny 10
 match tag 90
route-map RIP_TO_EIGRP permit 20
 match tag 110
 set tag 110
route-map RIP_TO_EIGRP permit 30
 set tag 120

route-map OSPF_TO_RIP deny 10
 match tag 120
route-map OSPF_TO_RIP permit 20
 match tag 90
 set tag 90
route-map OSPF_TO_RIP permit 30
 set tag 110

route-map RIP_TO_OSPF deny 10
 match tag 110
route-map RIP_TO_OSPF permit 20
 match tag 90
 set tag 90
route-map RIP_TO_OSPF permit 30
 set tag 120

route-map OSPF_TO_EIGRP deny 10
 match tag 90
route-map OSPF_TO_EIGRP permit 20
 match tag 120
 set tag 120
route-map OSPF_TO_EIGRP permit 30
 set tag 110

route-map EIGRP_TO_OSPF deny 10
 match tag 110
route-map EIGRP_TO_OSPF permit 20
 match tag 120
 set tag 120
route-map EIGRP_TO_OSPF permit 30
 set tag 90

router eigrp 1
 redistribute ospf 1 metric 1000000 10 255 1 1500 route-map OSPF_TO_EIGRP
 redistribute rip metric 1000000 10 255 1 1500 route-map RIP_TO_EIGRP

router ospf 1
 redistribute eigrp 1 metric-type 1 subnets route-map EIGRP_TO_OSPF
 redistribute rip metric 1 subnets route-map RIP_TO_OSPF

router rip
 redistribute eigrp 1 metric 3 route-map EIGRP_TO_RIP
 redistribute ospf 1 metric 3 route-map OSPF_TO_RIP
```

## VRF-Lite
**VRF-Lite**
- Divide interfaces into VRFs and create separate routing tables.
- Do not implement L3VPN afterwards, this is why its called VRF-Lite.

```
vrf definition 10
 add ipv4
vrf definition 172
 add ipv4
int fa0/0
 vrf forwarding 10
 ip add 10.0.12.1 255.255.255.0
int fa0/1
 vrf forwarding 10
 ip add 10.0.13.1 255.255.255.0
int se1/0
 vrf forwarding 172
 ip add 172.0.12.1 255.255.255.0
int se1/1
 vrf forwarding 172
 ip add 172.0.13.1 255.255.255.0

router eigrp EIGRP
 address-family ipv4 unicast vrf 10 autonomous-system 10
  no auto-summary
  network 10.0.12.0 0.0.0.255
  network 10.0.13.0 0.0.0.255
address-family ipv4 unicast vrf 172 autonomous-system 172
  no auto-summary
  network 172.0.12.0 0.0.0.255
  network 172.0.13.0 0.0.0.255
```

Non-VRF neighbors:

```
router eigrp 10
  no auto-summary
  network 10.0.12.0 0.0.0.255
router eigrp 172
  no auto-summary
  network 172.0.12.0 0.0.0.255
```

# IP services
**Secure Copy Protocol (SCP)**
- Requires SSH and AAA Authorization.
- Has to be enabled on both routers to allow mutual copying.

```
aaa new-model  
aaa authentication login default local  
aaa authorization exec default local
username bpin privilege 15 password cisco
ip scp server enable

copy scp://bpin@10.0.12.1/nvram:startup-config null:
```

**RCMD Remote-Copy (RCP) and Remote-Shell (RSH)**
- Allow remote users to copy files to router with RCP.
- Allow remote users to execute commands with RSH.
- Server side has two names in the rcmd command.
  - First one must match /user on client.
  - Second one must match client hostname or client remote-username command.
  - The enable keyword allows execution of exec commands.

Configure the server:

```
ip rcmd rcp-enable
ip rcmd rsh-enable  
ip rcmd remote-host remoteadmin 10.0.12.2 R2 enable
```

Configure the client:

```
ip rcmd remote-username R2

rsh 10.0.12.1 /user remoteadmin show ip interface brief
copy rcp://remoteadmin@10.0.12.1/nvram:startup-config null:
```

The boot/service config enables auto-loading of configuration files from a network server:

```
service config
ip rcmd remote-username R2
boot network rcp://10.0.12.1/BOOT
boot network tftp:BOOT
```

**Local TFTP-Server**
- Specify all files that are eligible for TFTP transfer separately.
- Optionally create an `alias` for the file and limit access.

```
access-list 1 permit host 192.168.0.2
tftp-server nvram:startup-config alias STARTUP 1
```

Configure client:

```
ip tftp source-interface loopback0
copy tftp://10.0.12.1/STARTUP null:
```

**DNS Services**
- Create individual host entries on the DNS server.

Server:

```
ip domain-lookup
ip domain-name lab.local
ip dns server
ip host Server1 2.2.2.2
ipv6 host Server2 2::2
```

Client:

```
ip domain-lookup
ip name-server 1.1.1.1
ip name-server 1::1
```

Configuration Generation Performance Enhancement (Parser)
- Caches interface configuration in memory, thus allowing faster execution of show run, write memory, etc.
- Enable with the `parser config cache interface` command.

**TCP small servers**
- Echo (7): Echoes back whatever you type.
- Chargen (19): Generates a stream of ASCII data.
- Discard (9): Throws away whatever you type.
- Daytime (13): Returns system date and time.

```
telnet 10.0.12.1 19
```

Terminate on server with:

```
show tcp brief
clear tcp tcb <tcb-value>
```

**UDP small servers:**
- Echo (7): Echoes the payload of the datagram you send.
- Discard (9): Silently pitches the datagram you send.
- Chargen (19): Pitches the datagram you send, and generates a stream of ASCII data.

**Misc. Services**
The X28 editor is enabled by default:

```
no service pad
```

Ensure that abnormally terminated TCP sessions are removed:

```
service tcp-keepalives-in
service tcp-keepalives-out
```

The finger service (TCP port 79) gives line information and is disabled by default:

```
ip finger
service finger
```

## BFD
**Bidirectional Forwarding Detection (BFD)**
- Requires CEF, sent unicast to UDP 3784.
- Only supports asynchronous mode, must be enabled on both sides.
- Only works for directly connected neighbors, BFD itself has no neighbor detection.
- Is not tied to any routing protocol, and can be used as a generic and consistent failure detection mechanism.
- Parts of BFD can be distributed to the data plane (echo), better than reduced IGP timers that exist only at the control plane.

BFD Echo Mode
- BFD echo packets are are sourced from UDP 3785 and sent to 3785.
- Enabled by default and can be enabled on either side. Does not work alongside ip redirects or uRPF (or IPv6 on CSR).
- Echo mode is supported on single-hop only. The packets are sent on the negotiated BFD timer interval.
- BFD packets are processed in fast switching instead of the control plane.
- Control plane packets are still sent but they are transmitted at the slow timers speed (1000 ms by default).

BFD Timers
- The time at which 'hello' messages are sent is configured with the `interval` timer.
- The `min_rx` timer is the receive timer, if no message is received within this time the neighbor is considered timed-out
- The `multiplier` specifies how many BFD messages can be missed before neighbor interface is considered down.
- BFD timers work like EIGRP. The send and receive timer do not have to match on both sides
- The slower receive timer of the neighbor will decide the value of the local send timer.

```
bfd slow-timers 1000
int gi0/0
 bfd echo
 bfd interval 500 min_rx 500 multiplier 3
```

**BFD Authentication**

```
key chain BFD_KEY
 key 1
  key-string cisco

bfd-template single-hop BFD
 echo
 interval both 500 multiplier 3
 authentication md5 keychain BFD_KEY

int gi0/0
 bfd template BFD
```

**BFD Static**
- Static routes that support BFD must specify an egress interface in single-hop mode.
- The neighbor must point back with a static route, or an unassociated route.
- Static routes can be dependent on a group. If one location becomes inaccessible the depending (passive) routes are also removed from the routing table.

Configure R1:

```
ip route static bfd gi0/0 10.0.12.2
ip route 0.0.0.0 0.0.0.0 gi0/0 10.0.12.2
```

Configure R2:

```
ip route static bfd gi0/0 10.0.12.1 unassociate
```

BFD Static Groups

```
ip route static bfd gi0/0 10.0.12.2 group BFD
ip route 33.33.33.0 255.255.255.0 gi0/1 10.0.13.3
ip route static bfd gi0/1 10.0.13.3 group BFD passive
```

## CPPr
**IOS Control Plane**
- Handles packets that are not CEF switched, meaning the CPU takes time to handle these packets.
- Maintains keep-alives for routing adjacencies.
- Handles traffic directed at the device itself (management traffic).

**Control Plane Protection (CPPr)**
- Framework that consists of traffic classifiers, protection and policing.
- Improvement over Control Plane Policing (CoPP) by allowing finer policing granularity.
- Management Plane Protection (MPP) is a part of CPPr and is basically just specifying a `management-interface`.
- Depends on CEF. When disabled, CPPr is disabled on sub-interfaces but not on the `aggregate` interface.

**Control Plane Interfaces**
- Host. Handles traffic destined for the router or one of its own interfaces (MGMT, EIGRP, iBGP)
- Transit. Handles software switched IP traffic.
- CEF-Exception. Handles non-IP related packets such as OSPF, eBGP, ARP, LDP and CDP (or packets with TTL <=1) .
- Aggregate interface `<cr>`.  Configuration applied here applies to all the sub-interfaces.
- It is not possible to apply a L3 policy-map to the `aggregate` and any of the other interfaces at the same time.
- A L3 policy-map applied to the control plane can only use `police` or `drop`, not shape...etc.
- The `port-filter` keyword polices packets going to closed/non-listening TCP/UDP ports.
- The `queue-threshold` keyword limits the number of protocol packets that are allowed in the input queue.
  - Rate limit OSPF and eBGP on the `cef-exception` sub-interface, iBGP on the `host` and EIGRP on the `aggregate`.

Police all ICMP traffic:

```
ip access-list extended ICMP_ACL
 permit icmp any any

class-map match-all ICMP_CM
 match access-group name ICMP_ACL
policy-map ICMP_PM
 class ICMP_CM
  police 10000 conform-action transmit exceed-action drop

control-plane host
 service-policy input ICMP_PM
```

Drop connections to closed ports:

```
class-map type port-filter match-all CLOSED_PORTS_CM
 match closed-ports

policy-map type port-filter CLOSED_PORTS_PM
 class CLOSED_PORTS_CM
  drop

control-plane host
 service-policy type port-filter input CLOSED_PORTS_PM
```

Queue SNMP traffic to 75 and any other open UDP/TCP ports to 100:

```
class-map type queue-threshold SNMP_CM
 match protocol snmp
class-map type queue-threshold HOST_CM
 match host-protocols

policy-map type queue-threshold QUEUE_PM
 class SNMP_CM
  queue-limit 75
 class HOST_CM
  queue-limit 100

control-plane host
 service-policy type queue-threshold input QUEUE_PM
```

Rate limit EIGRP traffic (requires egress direction):

```
ip access-list extended EIGRP
permit eigrp any any

class-map match-all EIGRP_CM
 match access-group name EIGRP

policy-map EIGRP_PM
 class EIGRP_CM
  police 10000 conform-action transmit exceed-action drop

control-plane
 service-policy output EIGRP_PM
```

**Management Plane Protection (MPP)**
Multiple interfaces can be specified for different protocols:

```
control-plane host
 management-interface fa0/0 allow ssh
 management-interface fa0/1 allow snmp
```

## DHCPv4
DHCP Messages

| Сообщение  | Описание  |
|---|---|
|DHCPDiscover|Sent by client to 0.0.0.0 to find a DHCP server (broadcast).|
|DHCPOffer|Response to client with DHCP server information and IP address assignment (unicast or broadcast).|
|DHCPRequest|Sent by client in response to DHCPOffer, client accepts IP address assignment (broadcast).|
|DHCPAcknowledge|Acknowledgement by the DHCP server (unicast or broadcast).|

The DHCPOFFER and DHCPACK are sent broadcast by default. Disable with the `no ip dhcp-client broadcast-flag` command.

Disable DHCP and do not reply to Bootstrap Protocol request packets received:

```
no service dhcp
ip dhcp bootp ignore
```

**DHCP Conflict Logging**
Similar function to excluded-addresses. Logs conflicts with a syslog message and stores the address on an exclusion list.
 - Conflicted addresses are stored and need clearing or restart to become usable again.
 - Enabled by default. Disable with the `no ip dhcp conflict logging` command.

```
service dhcp
no ip dhcp conflict logging
ip dhcp excluded-address 10.0.123.1 10.0.123.99
ip dhcp excluded-address 10.0.123.200 10.0.123.254
ip dhcp pool DHCP
 default-router 10.0.123.1
 network 10.0.123.0 /24
 dns-server 192.168.0.1

int fa0/0
 no ip dhcp-client broadcast-flag
 ip address dhcp
```

**DHCP Reservation**
Cisco routers use the Client-Identifier to identify themselves.
 - This is a combination of the hardware address, interface name and cisco.
 - This client-identifier is then turned into a HEX string and presented to the server.
 - Add 00 to the beginning of a client-identifiers hex string.
 - The easiest way to acquire the client-identifier is by first giving out a regular DHCP address and debug ip dhcp server packets.

```
ip dhcp pool R2
 host 10.0.123.2 /24  
 client-identifier 0063.6973.636f.2d63.6130.332e.3065.3434.2e30.3030.382d.4661.302f.30
ip dhcp pool R3
 host 10.0.123.3 /24  
 client-identifier 0063.6973.636f.2d63.6130.342e.3138.6638.2e30.3030.382d.4661.302f.30
```

**DHCP Relay Agent and Information Option 82**
 - Relay agents receive DHCP messages and then generate a new DHCP message to send out on another interface.
 - DHCP option 82 (relay information option) identifies hosts by both the MAC-Address and the switchport. Disabled by default on routers, enabled by default on switches. It allows DHCP relays to inform the DHCP server where the original request came from.
 - The reply from the server is forwarded back to the client after removing option 82.
 - Enable the option with either the global or interface level command, `ip dhcp relay information` option. (interface takes preference)

```
service dhcp
int fa0/0
 description CLIENTS
 ip helper address 10.0.12.1
```

A DHCP relay agent may receive a message from another DHCP relay agent that already contains relay information (Relayed twice).
 - By default, the relay information from the previous agent is replaced. Customize with `ip dhcp relay information policy` command.
 - If the information policy is changed, also disable the information check with the `no ip dhcp relay information check` command.

## EEM
Embedded Event Manager (EEM)
 - The `skip` keyword prevents the command from being executed. Default is `skip no`.
 - The `sync` keyword runs the script before the command. Default is `sync yes`.
 - `_exit_status 1` means that the command is run.
 - `_exit_status 0` means that the command is skipped.
 - `$_cli_result` is the outcome of a cli command that was executed, can be pasted into the console with the `puts` keyword.
 - `$_cli_msg` is the pattern matched with the event keyword. Can be pasted into the console with the `syslog msg` keyword.
 - The cli command is not executed until the EEM policy exits.

```
show event manager policy registered
debug event manager action cli
debug event manager action mail
```

**EEM Examples**

Disable show running-config command:

```
event manager applet DIS_SH_RUN
 event cli pattern "show run" skip yes sync no
 action 1.0 cli command "enable"
 action 1.1 syslog msg "$_cli_msg not executed, function disabled"
 action 1.2 mail server ....
```

Hide interfaces from the running configuration:

```
event manager applet SH_RUN_NO_INT
 event cli pattern "show run" sync yes
 action 1.0 syslog msg "$_cli_msg executed"
 action 1.1 cli command "enable"
 action 1.2 cli command "show run | section exclude interface"
 action 1.3 puts "$_cli_result"
```

Re-enable manually shut down interfaces:

```
event manager applet NO_SHUT_INT
 event syslog pattern "Interface FastEthernet0/0, changed state to administratively down"
 action 1.0 cli command "enable"
 action 1.1 cli command "configure terminal"
 action 1.2 cli command "interface Fa0/0"
 action 1.3 cli command "no shut"
```

Print confirmation to the terminal:

```
event manager applet WRITE_MEMORY
 event cli pattern "write memory" sync yes
 action 1.0 syslog msg "$_cli_msg Command Executed"
 set 2.0 _exit_status 1
```

Disable OSPF and EIGRP:

```
event manager applet DIS_OSPF_EIGRP
 event cli pattern "router [eEoO].*" sync no skip yes
 action 1.0 syslog msg "Routing protocols OSPF and EIGRP have been disabled"
```

Send ICMP requests based on tracking object:

```
event manager applet TRACK_1_DOWN
 event syslog pattern "1 ip sla 1 state Up->Down"
 action 1.0 syslog msg "IP SLA 1 Transferred to Down State, Testing ICMP"
 action 1.1 cli command "enable"
 action 1.2 cli command "ping 10.0.12.2 repeat 5 time 1"
 action 1.3 syslog msg "ping 10.0.12.2 repeat 5 time 1"
 action 1.4 puts "$_cli_result"

event manager applet TRACK_1_UP
 event syslog pattern "1 ip sla 1 state Down->Up"
 action 1.0 syslog msg "IP SLA 1 Returned to UP State, Testing ICMP"
 action 1.1 cli command "enable"
 action 1.2 cli command "ping 10.0.12.2 repeat 5 time 1"
 action 1.3 syslog msg "ping 10.0.12.2 repeat 5 time 1"
 action 1.4 puts "$_cli_result"
```

Or:

```
event manager applet TRACK_1_DOWN
 event track 1 state down
 ...
```

Periodically send output to the console:

```
ip sla 1
 udp-jitter 192.168.0.2 16384 codec g729a
  frequency 5

event manager applet IP_SLA_1
 event timer watchdog time 3600
 action 1.0 cli command "show ip sla statistics 1"
 action 1.2 puts "$_cli_result"
 action 1.3 mail ...
```

Create a log message based on added routes (does not disable function):

```
event manager applet STATIC_ROUTES
 event routing type add protocol static network 0.0.0.0/0  le 32
 action 1.0 syslog msg "Static routes are not allowed, notifying admin"
 action 1.1 mail server ....
```

## EPC
**Old Method**
Association (and disassociation) are actions that can be performed in order to bind a capture point to a capture buffer.
 - A capture point can only be associated with one capture buffer (an ACL filter can also be applied).
 - A capture buffer can be associated with many capture points.
 - A buffer can collect data from many points but a point can send data to only one buffer.
 - Capture local traffic with the `monitor capture point ip process-switched LOCAL from-u`s command.

```
monitor capture buffer BUFFER
monitor capture point ip cef PCAP fa0/0 both
monitor capture point associate PCAP BUFFER

show monitor capture buffer BUFFER dump
show monitor capture buffer BUFFER parameters
show monitor capture point PCAP

monitor capture buffer BUFFER export
```

New Method

```
monitor capture PCAP match any int gi0/0 both
monitor capture PCAP start

show monitor capture PCAP

monitor capture PCAP export
```

## IPCP
**Internet Protocol Control Protocol (IPCP)**
 - IPCP relies on PPP.
 - IOS ignores mask requests and offers. This is a problem when running RIP.
 - Use PPPoE with import IPCP into DHCP to acquire the correct mask.
 - Disable `validate-update source` in RIP when using IPCP.

```
ip local pool IPCP 10.0.12.2
int se1/0
 ip add 10.0.12.1 255.255.255.0
 encapsulation ppp
 peer default ip address pool IPCP
 peer default ip address 10.0.12.2
 ppp ipcp mask 255.255.255.0
 peer neighbor-route

int se1/0
 encapsulation ppp
 ip address negotiated
 ppp ipcp mask request
 peer neighbor-route
 no shut

router rip
 no auto
 version 2
 network 10.0.12.0
 network 192.168.0.0
 no validate-update source
```

**Import IPCP subnet settings to local DHCP**
 - This will allow the import of the correct subnet mask.
 - The imported IPCP pool will always start at the first address (.1) even if only a single address is specified in the pool.
 - This will ignore additional IPCP settings, such as the default route installed through IPCP.

```
ip dhcp pool LOCAL
 import all
 origin ipcp

int se1/0
 encapsulation ppp
 ppp ipcp mask request
 no ppp ipcp route default
 no ip add negotiated
 ip add pool LOCAL
```

## IRDP
ICMP Router Discovery Protocol (IRDP)
 - Allows routers to discover gateways.
 - IP routing has to be disabled on client.

```
int fa0/0
 ip irdp
 ip irdp maxadvertinterval 30
 ip irdp minadvertinterval 10
 ip irdp holdtime 90
 ip irdp preference 200

show ip irdp fa0/0
```

Client:

```
no ip routing
ip gdp irdp
```

## FHRP
**Hot Standby Router Protocol (HSRP)**
 - Preemption is disabled by default.
 - Highest priority router is active, if priority is equal the highest IP-address wins.
 - Hello timer is 3 seconds by default, and hold timer is 10 seconds by default.

|   |   |   |   |
|---|---|---|---|
|HSRPv1|0000.0c07.acXX|UDP 1985|224.0.0.2|
|HSRPv2|0000.0C9F.F000 through 0000.0C9F.FFFF|UDP 1985|224.0.0.102|
|HSRPv6|0005.73A0.0000 through 0005.73A0.0FFF|UDP 2029|FF02::66|

XX = Group number in HEX

**Virtual Router Redundancy Protocol (VRRP)**
 - Preemption is enabled by default.
 - Highest priority router is active, if priority is equal the highest IP-address wins.
 - Advertisement interval (hello timer) is 1 second by default, automatically sets the hold-timer to 3x hello.
 - IPv6 VRRP requires version3.

|   |   |   |   |
|---|---|---|---|
|VRRPv2|0000.5e00.01XX|UDP 112|224.0.0.18|
|VRRPv3|0000.5e00.01XX|UDP 112|FF02::12|

XX = Group number in HEX

```
fhrp version vrrp v3
int fa0/0
 vrrp 1 address-family ipv6
  address 2001:10:0:12::254/64
  address fe80::254 primary
```

**Gateway Load Balancing Protocol (GLBP)**
 - AVG preemption is disabled by default.
 - AVF preemption is enabled by default, with a delay of 30 seconds. Useful when preempting lower weighted routers.
 - Highest priority router is AVG, if priority is equal the highest IP-address wins.
 - Hello timer is 3 seconds by default, and hold timer is 10 seconds by default.
 - Up to 4 AVF per group, this includes the AVG.

|   |   |   |   |
|---|---|---|---|
|GLBP|0007.b400.XXYY|UDP 3222|224.0.0.102|
|GLBPv6|0007.b400.XXYY|UDP 3222|FF02::66|

XX = Group number in HEX
YY = AVF number

**GLBP Load-Balancing and Weighting**
- Host-dependent. Same AVF is used for the same host (based on mac address).
- Round-robin. Default. Each AVF is used in turn.
- Weighted. Dependent on the weighting value.
  - The maximum weighting value is the default 'normal' value (100 by default).
  - If devices go below the lower weighting value, they lose their AVF status (1 by default)
  - If devices go above the upper weighting value, they regain their AVF status after a loss (same as maximum by default)

Device will lose AVF status if only 1 tracking object is up. Will regain status if 2/3 tracking objects are up:

```
int fa0/0
 glbp 1 load-balancing weighted
 glbp 1 weighting 100 51 74
 glbp 1 weighting track 1 decrement 25
 glbp 1 weighting track 2 decrement 25
 glbp 1 weighting track 3 decrement 25
```

**GLBP Redirection**
 - Redirect timer is 10 minutes, timeout timer is 4 hours by default.

In case of an unreachable AVF the AVG redirects traffic:
 - During redirecting time, the AVG points a new AVF for any new request with old virtual MAC address.
 - After the redirect timer expires, the AVG stops pointing a new AVF for any new request with old virtual MAC address.
 - Hosts that using old mac-address can get responses and are able to use old mac address until the timeout timer expires.
 - If the AVF doesn't return until the timeout timer expires, all GLBP peers flush the record of the old MAC address and old AVF.

## KRON
**Command Scheduler (KRON)**
 - Only works for exec mode commands, not global or interface configuration commands.
 - Choose either the `oneshot` or recurr`ing keyword to schedule KRON occurrence once or repeatedly.
 - The `system-startup` keyword will set the occurrence to be at system startup.

Show routes every 5 minutes:

```
kron policy-list KRON_POLICY
 cli show ip route

kron occurrence KRON_OCC in 5 recurring
 policy-list KRON_POLICY

sh kron schedule
debug kron all
```

## Logging
**Archive Log**

Configure archiving and optionally log commands to syslog:

```
archive
 log config
  logging enable
  notify syslog
  exit
alias exec sal show archive log config all provisioning
```

**Archive Config**
Configure archiving of configs to TFTP server:

```
archive
 path tftp://192.168.10.1/archive

archive config

show archive
show archive config differences
```

**Logging**
 - Logging to the console and or buffer might be disabled in the lab. Verify with the `show run all | i logging` command.
 - Show timestamps (on by default) on debug messages with the `service timestamps debug datetime` command.
 - Show timestamps (on by default) on log messages with the `service timestamps log datetime` command.
 - Show sequence numbers (off by default) on log messages with the s`ervice sequence-numbers` command.

Enable logging:

```
logging console guaranteed 
logging buffered 8192 debugging
logging console debugging
logging monitor debugging
logging on
```

History

```
show run all | i history
show history
```

Reset history:

```
term history 0
term history 10
```

## NetFlow
NetFlow Versions

| Версия  | Описание  |
|---|---|
|5|NetFlow v5 is fixed format, cannot be extended or added to. IPv4 only.<br>Added BGP AS information and sequence numbers.<br>Exports data from main cache only. |
|8|Added support for data export from aggregation caches.|
|9|NetFlow v9 can add additional information to flows, template based.<br>Added support for MPLS, BGP next-hop and IPv6 headers.<br>Exports data from main and aggregation cache. |

**NetFlow IP Flows**
- NetFlow Requires CEF in order to function.
- In original NetFlow if all of characteristics match, they're considered the same flow.
- An IP Flow can be characterized by a set of 5 and up to 7 packet attributes:
  - Source / destination IP address
  - Source / destination port
  - L3 protocol type
  - Class of Service
  - Router or switch interface

**NetFlow Collector**
 - NetFlow Collector = NetFlow server.
 - The Collection Engine (local) sends NetFlow data to the collector with 1.5% export data overhead.
 - The NetFlow Cache creates cache entries (flow records) for every active flow.
 - Flow records store IP flow information.
 - NetFlow export, unlike SNMP polling, pushes information periodically to the collector.
 - Flows that have terminated or expired (Based on cache) are exported as well.

NetFlow v5
 - NetFlow v5 does not have a concept of 'ingress' and 'egress' flows.
 - The collector engine reverses the information behind the scenes without any additional configuration.
 - The ip route-cache flow command is the old way of configuring NetFlow. This is called the Flow fast-switching cache.
 - The old command will also enable NetFlow on all sub-interfaces, the newer command does not.

```
int fa0/0
 description Inside
 ip flow ingress

int fa0/0
 description Outside
 ip flow ingress

show ip flow interface
show ip cache flow
```

NetFlow v9
 - NetFlow v9 introduces the 'egress' flows concept.
 - With egress, it is possible to configure ingress and egress on the same interface and capture both traffic directions.
 - Multicast traffic can't be effectively matched on ingress.
 - Ingress calculates before compression, this is a problem if WAN links are using compression of packets.
 - Egress calculates flows after compression.
 - Configuring NetFlow v9 with the egress keyword uses a default template behind the scenes.
 - Only when templates are specified Flexible NetFlow is being used.

```
int fa0/0
 description Inside
 ip flow ingress

int fa0/0
 description Outside
 ip flow egress
```

Export flows to collector:

```
ip flow-export destination 1.1.1.1 9995
ip flow-export version 9
ip flow-export source loopback 0

show ip flow export
```

**NetFlow Aggregation Cache (v8 and v9)**
 - Enables specification of which type of flows will be exported to the collector.
 - All flows are still captured on the device (using v5 or v9) but only when flows are exported they are filtered.

Only export flow entries that have a /32 mask:

```
ip flow-aggregation cache destination-prefix
 cache entries 1024
 export version 9
 export destination 2.2.2.2 9995
 mask destination minimum 32
 enabled

show ip cache flow aggregation destination-prefix
```

**NetFlow Top Talkers**
 - Useful if no collector server is present to analyze data flows.
 - Shows top talkers based on bytes or packets.

```
ip flow-top-talkers
 top 10
 sort by packets

show ip flow top-talkers
```

**NetFlow Sampler**
 - Sampled mode lets you collect only for a subset of traffic.
 - Can be linked directly to the interface, or be part of a `policy-map`.
 - Can not be used alongside the `ingress` command. Either capture all flows or a subset of flows.

```
flow-sampler-map RANDOM
 mode random one-out-of 10

int fa0/0
 flow-sampler RANDOM

show flow-sampler
```

Add a sampler to a policy-map and match one out of 10 ICMP packets:

```
flow-sampler-map RANDOM
 mode random one-out-of 10
flow-sampler-map ONE_ONE
 mode random one-out-of 1

class-map match-all ICMP
 match protocol icmp

policy-map SAMPLER
 class ICMP
  netflow-sampler RANDOM
 class class-default
  netflow-sampler ONE_ONE

int fa0/0
 service-policy input SAMPLER
```

**Flexible NetFlow (FNF)**
Consists of three parts:
 - Flow Records, which set key and non-key fields.
 - Flow Exporter, which details where and how to send the exports.
 - Flow Monitors, which match the flow records and exporters, and are then applied to an interface.

```
flow exporter FNF_EXPORT
 destination 1.1.1.1
 transport udp 9995
 export-protocol netflow-v9

flow monitor FNF
 record netflow ipv4 original-input
 exporter FNF_EXPORT

int fa0/0
 ip flow monitor FNF input

show flow exporter
show flow monitor FNF cache format table
```

**FNF Sampler**

```
sampler FNF_SAMPLER
 mode random 1 out-of 10

int fa0/0
 ip flow monitor FNF sampler FNF_SAMPLER input

show sampler
```

## NTP
**Network Time Protocol (NTP)**
 - NTP Peers can both act as either a client or a server at the same time and offer bidirectional synchronization.
 - When the connection to the NTP server fails, the peer will be regarded as the new server.
 - Masters on older IOS versions (12.4) use the 127.127.7.1 local address to peer with itself.
 - Newer IOS versions use the 127.127.1.1 local address.
 - The source local address is always one stratum lower than the configured value. Default configured stratum is 8.
 - Stratum is the tie-breaker. If two servers offer the same stratum, the prefer keyword can be added to prefer one over the other.

```
ntp master 8
ntp server 192.168.0.1 prefer
ntp peer 192.168.0.2

show ntp status     
show ntp association detail
debug ntp packet
debug ntp events
```

The `offset value` is the time difference in milliseconds between the local clock and the NTP server's reference clock.
 - The offset must be < 1000 msec (1 second) off in order for the server to be considered sane.
 - NTP does not shift the clock instantaneously, instead the router slowly drifts towards the time.
 - If the offset value between the client and the server is large, this process can take a long time.
 - After the offset value is < 1 second off, the router will adjust its stratum from 16 (infinite) to the appropriate stratum.

Time Zones
 - NTP updates are always sent in UTC/GMT.
 - EU and US summer time dates are different. Default is US, configure with clock summer-time US recurring.
 - US summer time begins second Sunday in March, ends first Sunday in November.
 - EU summer time begins last Sunday in March, ends last Sunday in October.

```
clock timezone CaPc -8
clock summer-time US recurring 2 Sun Mar 02:00 First Sun Nov 02:00
clock summer-time EU recurring Last Sun Mar 02:00 Last Sun Oct 02:00 
```

**NTP Authentication**
 - The client authenticates the server, it is more important to receive time from the correct source over giving time to devices.
 - Other NTP clients will still be able to request time without authentication.

```
ntp trusted-keys 1
ntp authenthication-key 1 md5 cisco
```

Client:

```
ntp authenthication-key 1 md5 cisco
ntp trusted-keys 1
ntp authenticate
ntp server 192.168.0.1 key 1
```

**NTP Access Control**
 - Control messages (queries) are for reading and writing internal NTP variables and status information. Not synchronization.
 - NTP request/update messages are used for actual time synchronization.
 - The `serve-only` keyword allows only time requests from NTP clients.
 - The `peer` keyword allows time requests and NTP control queries from clients. But also allows bidirectional synchronization.
 - Masters on older IOS versions (12.4) need to specifically allow peering with the own loopback address (127.127.7.1).
 - Access-groups associated with access types are scanned in the order most permissive to most restrictive. Peer -> Serve-Only.
  - This means that denying a client in `serve-only` but allowing with `peer`, the client will still be able to peer.

```
ntp master
access-list 1 deny host 192.168.0.2
access-list 2 permit host 192.168.0.2
ntp access-group serve-only 1
ntp access-group peer 2

access-list 127 permit host 127.127.7.1
ntp access-group peer 127
```

**NTP Broadcast and Multicast**
 - Default multicast address is 224.0.1.1.

```
int fa0/0
 ntp broadcast
 ntp broadcast destination 10.0.12.2
 ntp multicast 224.0.1.1
```

Client:

```
int fa0/0
 ntp broadcast client
 ntp multicast client
```

## PPP
PPP Authentication
 - The router that enables PPP authentication requests credentials from the remote router.
 - The credentials supplied by the remote router has to match the local user database.
 - Local authentication is based on usernames.
 - EAP requires the addition of the `local` keyword to authenticate using the local database.
 - PPP usernames can still be used for line management. Use the `username PPP-USER autocommand logout` to prevent this.

R1 requests CHAP from R2:

```
username R2 password cisco
username R3 password cisco
int se1/0
 encapsulation ppp
 ppp authentication chap
 ppp pap sent-username R1 password cisco
```

R2 requests PAP from R1:

```
username R1 password cisco
int se1/0
 encapsulation ppp
 ppp chap hostname R2
 ppp chap password cisco
 ppp authentication pap

show users
who
debug ppp authentication
```

**AAA Authentication for PPP**
 - When using AAA the `autocommand` will only function if `aaa authorization` is also configured.
 - Preferably use a private Radius or Tacacs+ server in combination with PPP authentication.

R1 requests EAP from R2 and authenticates using Radius:

```
aaa new-model
aaa group server radius MYRADIUS
 server-private 1.1.1.1 timeout 5 retransmit 0 key cisco

aaa authentication ppp PPP_R1_R2 group MYRADIUS local
aaa authorization exec default group MYRADIUS local
username R2 password cisco
username R2 autocommand logout
int se1/0
 encapsulation ppp
 ppp authentication eap PPP_R1_R2
 ppp eap local
 ppp chap hostname R1
 ppp chap password cisco

show aaa servers
show radius server-group all
debug radius
```

R2 requests MS-CHAP-V2 from R2 and authenticates using Tacacs+:

```
aaa new-model
aaa group server tacacs+ MYTACACS
 server-private 2.2.2.2 single-connection key cisco

aaa authentication ppp PPP_R1_R2 group MYTACACS local-case
aaa authorization exec default group MYTACACS local
username R1 password cisco
username R1 autocommand logout
int se1/0
 encapsulation ppp
 ppp eap identity R2
 ppp eap password cisco
 ppp authentication ms-chap-v2 PPP_R1_R2

show aaa servers
show tacacs private
debug tacacs
debug tacacs events
debug tacacs packets
```

**Multilink PPP (MLPPP)**
 - Uses a fragmentation scheme where large packets are sliced in pieces and sequence numbers are added using headers.
 - Fragments are sent over multiple links and reassembled at the opposite end.
 - Small voice packets are interleaved with fragments of large packets using a special priority queue.

The `interleave` keyword enables real-time packet interleaving.
 - Allows large packets to be MLPPP encapsulated and fragmented into a small enough size to satisfy delay requirements.

```
int multilink 1
 ppp multilink interleave
 ppp multilink

int se1/0
 ppp multilink
 ppp multilink group 1
```

## PPPoE
**PPP over Ethernet (PPPoE)**
 - PPPoE provides a standard method of employing the authentication methods of PPP over an Ethernet network.
 - Allows authenticated assignment of IP addresses.
 - The MTU size is automatically set to 1492 bytes.

**PPPoE IPCP**
 - IOS ignores mask requests and offers. This is a problem when running RIP.
 - Use PPPoE with import IPCP into DHCP to acquire the correct mask.

Server:

```
bba-group pppoe R2
 virtual-template 12
int fa0/0
 pppoe enable group R2

interface Virtual-Template 12
 description R2
 ip address 10.0.12.1
 ip mtu 1492
 encapsulation ppp
 ppp authentication chap
 peer default ip address pool IPCP
 ppp ipcp mask 255.255.255.0

username R2 password cisco
ip local pool IPCP 10.0.12.2
```

Client:

```
int fa0/0
  pppoe-client dial-pool-number 12

interface Dialer 1
 ip address negotiated
 ip mtu 1492
 encapsulation ppp
 ppp chap username R2
 ppp chap password cisco
 dialer pool 12
 ppp ipcp mask request
 ppp ipcp route default
```

**PPPoE IPCP with local DHCP**
 - Import IPCP subnet settings to local DHCP. Will allow the import of the correct subnet mask.
 - The imported IPCP pool will always start at the default first address (.1) even if only a single address is specified in a pool.
 - This will ignore additional IPCP settings, such as the default route installed through IPCP.
 - Requires bouncing of the interfaces if overwriting an existing IPCP configuration.

Client:

```
ip dhcp pool IMPORT_IPCP
 import all
 origin ipcp

int dialer1
 dialer pool 12
 ip mtu 1492
 encapsulation ppp
 ppp chap username R2
 ppp chap password cisco
 ppp ipcp mask request
 ip address pool IMPORT_IPCP

int fa0/0
 pppoe-client dial-pool-number 12
```

**PPPoE DHCP**

Server:

```
bba-group pppoe R2
 virtual-template 12
int fa0/1
 pppoe enable group R2

interface Virtual-Template12
 description R2
 ip address 10.0.12.1 255.255.255.0
 ip mtu 1492
 peer default ip address dhcp-pool DHCP
 ppp authentication pap

username R2 password cisco
ip dhcp excluded-address 10.0.12.1
ip dhcp excluded-address 10.0.12.3 10.0.12.254
ip dhcp pool DHCP
 network 10.0.12.0 /24
 default-router 10.0.12.1
```

Client:

```
int fa0/1
  pppoe-client dial-pool-number 12

interface Dialer 1
 dialer pool 12
 ip mtu 1492
 encapsulation ppp
 ppp pap sent-username R2 password cisco
 ip address dhcp
```

## PBR
**Policy-Based Routing (PBR)**
 - The `set ip next-hop` and `set interface` are used unconditionally, meaning that the RIB is not used in case of failure.
 - The `set ip default next-hop` and `set default interface` apply to the default route and are used before the regular default route.
 - The idea is to specify an alternate default route for hosts matched in the access-list.

There's an order of operations to PBR set statements. `ip next-hop` -> `interface` -> `ip default next-hop` -> `default interface`.
 - If the first statement fails the next will be evaluated. Remember that addresses are preferred over interfaces.
 - The `recursive` keyword can be used to specify a next-hop that is not directly connected.
 - Only statement 10 is matched in route-maps. Meaning that if there is no match in 10, traffic is routed normally.
 - For this reason PBR route-maps allow more than one `set` statement in route-map sequences.

```
ip access-list standard PBR_ACL
 permit 172.16.0.0 0.0.0.255

route-map PBR permit 10
 match ip address PBR_ACL
 set ip default next-hop 10.0.12.2
 set default interface se1/0

interface fa0/0
 ip policy route-map PBR
```

**Local PBR**
 - Applying the PBR on an interface does not affect traffic locally generated by the router (even when sourcing the interface)
 - Create an `ip local policy` using loopbacks to policy-route local traffic.
 - Use extended access-lists to have more granular control over which local traffic is policy routed.
 - Using standard access-lists will policy route all local traffic.

```
ip access-list standard PBR_LOCAL_ACL
 permit host 192.168.0.1

route-map PBR_LOCAL permit 10
 match ip address PBR_LOCAL_ACL
 set ip next-hop 10.0.12.2
 set interface se1/0
ip local policy route-map PBR_LOCAL

show ip local policy
```

## SLA
**TCP Connect**
 - Control messages communicates the port that will be used from the sender to the receiver (enabled by default).
 - Disable control packets when using a well-known TCP port (telnet for example). This also does not need a responder.
 - SLA messages to an unknown port requires a SLA responder at the destination.

```
ip sla 1
 tcp-connect 192.168.0.2 23 control disable
 threshold 500
 timeout 1000
 frequency 5
ip sla schedule 1 life forever start-time now

show ip sla statistics
show ip sla configuration
show tcp brief
```

**IP SLA Responder**
 - Does not calculate processing time, allowing for more accurate measurements on the speed of the link.
 - Enable globally with the `ip sla responder` command. General IP SLA responder uses port 1967 for control messages.
 - Can also be configured to listen on a specific port for UDP or TCP. However this specific port must be configured on both sides. It is not possible to use control messages at sender and specific port at the receiver.

**UPP Echo**
 - UDP echo always requires a responder at the destination.
 - If control messages are disabled, the responder must be configured to listen on the specific port (55555 in this case).

Source:

```
ip sla 1
 udp-echo 192.168.0.2 55555 control disable
 threshold 500
 timeout 1000
 frequency 5
ip sla schedule 1 life forever start-time now
```

Destination:

```
ip sla responder udp-echo ipaddress 192.168.0.2 port 55555

show ip sla responder
```

**UDP Jitter**
 - Per-direction jitter (source to destination and destination to source).
 - Per-direction packet loss.
 - Per-direction delay (one-way delay).
 - Round-trip delay (average round-trip time).
 - Same as UDP echo, also requires a responder configured at the destination.
 - Success/failures will only be updated when all packets are analyzed (10 packets are sent by default).

**IP SLA Authentication**
 - The authentication hash is MD5.
 - Enabled for all SLAs present on the device.
 - Only applied on SLAs where both sides need to participate (using a responder).

```cisco
key chain SLA
 key 1
  key-string cisco

ip sla key-chain SLA
show ip sla authentication
```