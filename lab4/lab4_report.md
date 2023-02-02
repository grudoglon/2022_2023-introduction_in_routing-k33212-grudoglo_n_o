# Отчет по лабораторной работе №4 "Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33212

Author: Grudoglo Nikita Olegovich

Lab: Lab4

Date of create: 20.01.2023

Date of finished: ...

**Цель работы:** Изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.


**Ход работы:**

1. Текст файла .yaml

```
name: lab4

mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24

topology:
  nodes:
    R01.SPB:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.11

    R01.HKI:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.12

    R01.LBN:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.13

    R01.LND:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.14
    
    R01.NY:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.15

    R01.SVL:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.16
    
    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.17

    PC2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.18
    
    PC3:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.19
    
  links:
    - endpoints: ["PC1:eth1","R01.SPB:eth1"]
    - endpoints: ["R01.SPB:eth2","R01.HKI:eth1"]
    - endpoints: ["R01.HKI:eth2","R01.LND:eth1"]
    - endpoints: ["R01.HKI:eth3","R01.LBN:eth1"]
    - endpoints: ["R01.LBN:eth2","R01.LND:eth2"]
    - endpoints: ["R01.LND:eth3","R01.NY:eth1"]
    - endpoints: ["R01.NY:eth2","PC2:eth1"]
    - endpoints: ["R01.LBN:eth3","R01.SVL:eth1"]
    - endpoints: ["R01.SVL:eth2","PC3:eth1"]
```

2. Схема связи

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab4/pics/lab4.drawio.png "Схема связи")

3. Текст конфигураций сетевых устройств 

  **1 часть:** настройка VRF

- Роутер R01.SPB

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=1.1.1.1
/routing ospf instance
set [ find default=yes ] router-id=1.1.1.1
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.1/24 interface=ether2 network=192.168.10.0
add address=10.10.10.1/30 interface=ether3 network=10.10.10.0
add address=1.1.1.1 interface=Lo network=1.1.1.1
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=1.1.1.1
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=65530 update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```

- Роутер R01.HKI

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=2.2.2.2
/routing ospf instance
set [ find default=yes ] router-id=2.2.2.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.2/30 interface=ether2 network=10.10.10.0
add address=10.10.20.1/30 interface=ether3 network=10.10.20.0
add address=10.10.30.1/30 interface=ether4 network=10.10.30.0
add address=2.2.2.2 interface=Lo network=2.2.2.2
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=2.2.2.2
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=1.1.1.1 remote-as=65530 update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=3.3.3.3 remote-as=65530 route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=4.4.4.4 remote-as=65530 route-reflect=yes update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```

- Роутер R01.LBN

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=3.3.3.3
/routing ospf instance
set [ find default=yes ] router-id=3.3.3.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.30.2/30 interface=ether2 network=10.10.30.0
add address=10.10.40.1/30 interface=ether3 network=10.10.40.0
add address=10.10.50.1/30 interface=ether4 network=10.10.50.0
add address=3.3.3.3 interface=Lo network=3.3.3.3
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=3.3.3.3
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=2.2.2.2 remote-as=65530 route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer4 remote-address=4.4.4.4 remote-as=65530 route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer5 remote-address=6.6.6.6 remote-as=65530 update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```

- Роутер R01.LND

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=4.4.4.4
/routing ospf instance
set [ find default=yes ] router-id=4.4.4.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.20.2/30 interface=ether2 network=10.10.20.0
add address=10.10.40.2/30 interface=ether3 network=10.10.40.0
add address=10.10.60.1/30 interface=ether4 network=10.10.60.0
add address=4.4.4.4 interface=Lo network=4.4.4.4
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=4.4.4.4
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=2.2.2.2 remote-as=65530 route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer4 remote-address=3.3.3.3 remote-as=65530 route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer6 remote-address=5.5.5.5 remote-as=65530 update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```

- Роутер R01.NY

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=5.5.5.5
/routing ospf instance
set [ find default=yes ] router-id=5.5.5.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.60.2/30 interface=ether2 network=10.10.60.0
add address=192.168.20.1/24 interface=ether3 network=192.168.20.0
add address=5.5.5.5 interface=Lo network=5.5.5.5
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether3 route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=5.5.5.5
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer6 remote-address=4.4.4.4 remote-as=65530 update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

- Роутер R01.SVL

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=6.6.6.6
/routing ospf instance
set [ find default=yes ] router-id=6.6.6.6
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.50.2/30 interface=ether2 network=10.10.50.0
add address=192.168.30.1/24 interface=ether3 network=192.168.30.0
add address=6.6.6.6 interface=Lo network=6.6.6.6
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether3 route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=6.6.6.6
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer5 remote-address=3.3.3.3 remote-as=65530 update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.SVL
```

- PC1

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.2/30 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC1
```

- PC2

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.20.2/30 interface=ether2 network=192.168.20.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC2
```

- PC3

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.30.2/30 interface=ether2 network=192.168.30.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC3
```

  **2 часть:** разбор VRF и настройка VPLS (изменение настроек только на устройствах R01.SPB, R01.NY, R01.SVL и PC1, PC2, PC3)

- Роутер R01.SPB

```
/interface bridge
add name=VPLSb
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:99:BA:D4:40:99 name=VPLS10 remote-peer=5.5.5.5 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:92:8A:E8:5D:79 name=VPLS20 remote-peer=6.6.6.6 vpls-id=10:0
/interface bridge port
add bridge=VPLSb interface=ether2
add bridge=VPLSb interface=VPLS10
add bridge=VPLSb interface=VPLS20
```

- Роутер R01.NY

```
/interface bridge
add name=VPLSb
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:2B:19:A5:33:42 name=VPLS10 remote-peer=1.1.1.1 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:B9:86:DD:5E:BA name=VPLS30 remote-peer=6.6.6.6 vpls-id=10:0
/interface bridge port
add bridge=VPLSb interface=ether3
add bridge=VPLSb interface=VPLS10
add bridge=VPLSb interface=VPLS30
```

- Роутер R01.SVL

```
/interface bridge
add name=Lo
add name=VPLSb
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:28:7B:96:5C:B6 name=VPLS20 remote-peer=1.1.1.1 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:91:C0:22:FD:2C name=VPLS30 remote-peer=5.5.5.5 vpls-id=10:0
/interface bridge port
add bridge=VPLSb interface=ether3
add bridge=VPLSb interface=VPLS20
add bridge=VPLSb interface=VPLS30
```

- PC1

```
/ip address
add address=192.168.10.1/24 interface=ether2 network=192.168.10.0
```

- PC2

```
/ip address
add address=192.168.10.2/24 interface=ether2 network=192.168.10.0
```

- PC3

```
/ip address
add address=192.168.10.3/24 interface=ether2 network=192.168.10.0
```

4. Проверки локальной связности

- Результаты пингов на некоторых роутерах

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab4/pics/R01.HKI.jpeg)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab4/pics/R01.SVL.jpeg)

- Проверка связности между VRF (для 1 части)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab4/pics/R01.SPB_VRF.jpeg)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab4/pics/R01.NY_VRF.jpeg)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab4/pics/R01.SVL_VRF.jpeg)

- Проверка связности между VPLS (для 2 части)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab4/pics/PC1.jpeg)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab4/pics/PC3.jpeg)

**Вывод:** при выполнени лабораторной работы были получены навыки по настройке BGP, MPLS и организации L3VPN и VPLS.
