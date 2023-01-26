# Отчет по лабораторной работе №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33212

Author: Grudoglo Nikita Olegovich

Lab: Lab3

Date of create: 16.01.2023

Date of finished: ...

**Цель работы:** изучить протоколы OSPF и MPLS, механизмы организации EoMPLS.

**Ход работы:**

1. Текст файла .yaml

```
name: lab3

mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24

topology:
  nodes:
    R01.NY:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.11

    R01.LND:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.12

    R01.LBN:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.13

    R01.HKI:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.14
    
    R01.MSK:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.15

    R01.SPB:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.16
    
    SGI-Prism:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.17

    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.18
    
  links:
    - endpoints: ["SGI-Prism:eth1","R01.NY:eth1"]
    - endpoints: ["R01.NY:eth2","R01.LND:eth1"]
    - endpoints: ["R01.NY:eth3","R01.LBN:eth1"]
    - endpoints: ["R01.LND:eth2","R01.HKI:eth1"]
    - endpoints: ["R01.LBN:eth2","R01.MSK:eth1"]
    - endpoints: ["R01.LBN:eth3","R01.HKI:eth3"]
    - endpoints: ["R01.HKI:eth2","R01.SPB:eth2"]
    - endpoints: ["R01.MSK:eth2","R01.SPB:eth1"]
    - endpoints: ["R01.SPB:eth3","PC1:eth1"]
```

2. Схема связи

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab3/pics/lab3.drawio.png "Схема связи")

3. Текст конфигураций сетевых устройств

- R01.NY

```
/interface bridge
add name=EoMPLSb
add name=Lo
/interface vpls
add cisco-style=yes cisco-style-id=100 disabled=no l2mtu=1500 mac-address=02:13:B5:3B:90:BD name=EoMPLS remote-peer=6.6.6.6
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=1.1.1.1
/interface bridge port
add bridge=EoMPLSb interface=ether2
add bridge=EoMPLSb interface=EoMPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.1/30 interface=ether3 network=10.10.10.0
add address=10.10.20.1/30 interface=ether4 network=10.10.20.0
add address=1.1.1.1 interface=Lo network=1.1.1.1
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=1.1.1.1
/mpls ldp interface
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

- R01.LND

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=2.2.2.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.2/30 interface=ether2 network=10.10.10.0
add address=10.10.30.1/30 interface=ether3 network=10.10.30.0
add address=2.2.2.2 interface=Lo network=2.2.2.2
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=2.2.2.2
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```

- R01.LBN

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=3.3.3.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.20.2/30 interface=ether2 network=10.10.20.0
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
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```

- R01.HKI

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=4.4.4.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.30.2/30 interface=ether2 network=10.10.30.0
add address=10.10.60.1/30 interface=ether3 network=10.10.60.0
add address=10.10.50.2/30 interface=ether4 network=10.10.50.0
add address=4.4.4.4 interface=Lo network=4.4.4.4
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=4.4.4.4
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```

- R01.MSK

```
/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=5.5.5.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.40.2/30 interface=ether2 network=10.10.40.0
add address=10.10.70.1/30 interface=ether3 network=10.10.70.0
add address=5.5.5.5 interface=Lo network=5.5.5.5
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=5.5.5.5
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.MSK
```

- R01.SPB

```
/interface bridge
add name=EoMPLSb
add name=Lo
/interface vpls
add cisco-style=yes cisco-style-id=100 disabled=no l2mtu=1500 mac-address=02:26:B4:96:F6:EB name=EoMPLS remote-peer=1.1.1.1
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=6.6.6.6
/interface bridge port
add bridge=EoMPLSb interface=ether4
add bridge=EoMPLSb interface=EoMPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.70.2/30 interface=ether2 network=10.10.70.0
add address=10.10.60.2/30 interface=ether3 network=10.10.60.0
add address=6.6.6.6 interface=Lo network=6.6.6.6
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=6.6.6.6
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```

- SGI Prism

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.11/24 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=SGI-Prism
```

- PC1

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.12/24 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC1
```

4. Проверки локальной связности

- MPLS forwarding table на некоторых роутерах

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab3/pics/R01.NY.jpeg)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab3/pics/R01.MSK.jpeg)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab3/pics/R01.SPB.jpeg)

- Проверка связи между компьютером и сервером

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab3/pics/PC1.jpeg)

![](https://github.com/grudoglon/2022_2023-introduction_in_routing-k33212-grudoglo_n_o/blob/main/lab3/pics/SGI-Prism.jpeg)

**Вывод:** при выполнении лабораторной работы были получены навыки по настройке OSPF, MPLS И EoMPLS.
