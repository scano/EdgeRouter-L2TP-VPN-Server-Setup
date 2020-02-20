# EdgeRouter-L2TP-VPN-Server-Setup

Setup a L2TP VPN Server with static DNS mapping fixed allowing to resolve from VPN connections
 
Firmware version: 2.0.8

Need customize:
- USER
- PASSWORD
- SHARED-SECRET
- ROUTER-IP
- IP-POOL-START
- IP-POOL-END

Be sure that IP-POOL-START and IP-POOL-END no interfer on local network IPs.

My example on network 10.0.0.1/24 with a DHCP in range 10.0.0.32 to 10.0.0.199.
Assuming that internet connection is on *eth0*

- USER: vpnuser
- PASSWORD: vpnpass
- SHARED-SECRET: thesecret
- ROUTER-IP: 10.0.0.1
- IP-POOL-START: 10.0.0.200
- IP-POOL-END: 10.0.0.249

Access to router via ssh:

`$ ssh admin@10.0.0.1`

Then execute the scripts:

```
configure 

set firewall name WAN_LOCAL rule 30 action accept
set firewall name WAN_LOCAL rule 30 description ike
set firewall name WAN_LOCAL rule 30 destination port 500
set firewall name WAN_LOCAL rule 30 log disable
set firewall name WAN_LOCAL rule 30 protocol udp

set firewall name WAN_LOCAL rule 40 action accept
set firewall name WAN_LOCAL rule 40 description esp
set firewall name WAN_LOCAL rule 40 log disable
set firewall name WAN_LOCAL rule 40 protocol esp

set firewall name WAN_LOCAL rule 50 action accept
set firewall name WAN_LOCAL rule 50 description nat-t
set firewall name WAN_LOCAL rule 50 destination port 4500
set firewall name WAN_LOCAL rule 50 log disable
set firewall name WAN_LOCAL rule 50 protocol udp

set firewall name WAN_LOCAL rule 60 action accept
set firewall name WAN_LOCAL rule 60 description l2tp
set firewall name WAN_LOCAL rule 60 destination port 1701
set firewall name WAN_LOCAL rule 60 ipsec match-ipsec
set firewall name WAN_LOCAL rule 60 log disable
set firewall name WAN_LOCAL rule 60 protocol udp

set vpn l2tp remote-access ipsec-settings authentication mode pre-shared-secret
set vpn l2tp remote-access ipsec-settings authentication pre-shared-secret SHARED-SECRET

set vpn l2tp remote-access authentication mode local
set vpn l2tp remote-access authentication local-users username USER password PASSWORD

set vpn l2tp remote-access client-ip-pool start IP-POOL-START
set vpn l2tp remote-access client-ip-pool stop IP-POOL-END

set vpn l2tp remote-access dns-servers server-1 ROUTER-IP
set vpn l2tp remote-access dns-servers server-2 1.1.1.1

set vpn l2tp remote-access outside-address 0.0.0.0

set vpn ipsec ipsec-interfaces interface eth0

set service dns forwarding options "listen-address=ROUTER-IP"

commit ; save

exit

```

Commands for check VPN Access

`$ show vpn remote-access`

`$ show vpn ipsec sa`

