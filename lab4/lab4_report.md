University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33202

Author: Malkov Evgeny Vitalevich

Lab: Lab4

Date of create: 10.01.2023

Date of finished: 13.01.2023

## Отчёт по лабораторной работе №4 "Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS"

**Цель работы**: Изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.
**Артефакты выполнения**

1. Полный текст топологии сети:

```
name: lab4

mgmt:
  network: lab4
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
      mgmt_ipv4: 172.20.20.21

    PC2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.22

    PC3:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.23

  links:
    - endpoints: ["R01.SPB:eth1","PC1:eth1"]
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

- Схема связи 1 части работы
  ![screenshot](./assets/schema1.png)
- Схема связи 2 части работы
  ![screenshot](./assets/schema2.png)

3. Тексты конфигурации для сетевых устройств.

1) 1 Часть работы

- Роутер R01.SPB

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=1.1.1.1
/routing ospf instance
set [ find default=yes ] router-id=1.1.1.1
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.1/24 interface=ether2 network=192.168.10.0
add address=10.0.1.1/30 interface=ether3 network=10.0.1.0
add address=1.1.1.1 interface=Lo0 network=1.1.1.1
/ip dhcp-client
add disabled=no interface=ether2
/ip route vrf
add export-route-targets=65530:777 import-route-targets=65530:777 interfaces=ether2 \
    route-distinguisher=65530:777 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=1.1.1.1
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=\
    65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```

- Роутер R01.HKI

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=2.2.2.2
/routing ospf instance
set [ find default=yes ] router-id=2.2.2.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.1.2/30 interface=ether2 network=10.0.1.0
add address=10.0.2.1/30 interface=ether3 network=10.0.2.0
add address=10.0.3.1/30 interface=ether4 network=10.0.3.0
add address=2.2.2.2 interface=Lo0 network=2.2.2.2
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=2.2.2.2
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=1.1.1.1 remote-as=\
    65530 update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=4.4.4.4 remote-as=\
    65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=3.3.3.3 remote-as=\
    65530 route-reflect=yes update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```

- Роутер R01.LBN

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=3.3.3.3
/routing ospf instance
set [ find default=yes ] router-id=3.3.3.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.3.2/30 interface=ether2 network=10.0.3.0
add address=10.0.4.1/30 interface=ether3 network=10.0.4.0
add address=10.0.6.1/30 interface=ether4 network=10.0.6.0
add address=3.3.3.3 interface=Lo0 network=3.3.3.3
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=3.3.3.3
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=2.2.2.2 remote-as=\
    65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer4 remote-address=4.4.4.4 remote-as=\
    65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer6 remote-address=6.6.6.6 remote-as=\
    65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```

- Роутер R01.LND

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=4.4.4.4
/routing ospf instance
set [ find default=yes ] router-id=4.4.4.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.2.2/30 interface=ether2 network=10.0.2.0
add address=10.0.4.2/30 interface=ether3 network=10.0.4.0
add address=10.0.5.1/30 interface=ether4 network=10.0.5.0
add address=4.4.4.4 interface=Lo0 network=4.4.4.4
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=4.4.4.4
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=2.2.2.2 remote-as=\
    65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer4 remote-address=3.3.3.3 remote-as=\
    65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer5 remote-address=5.5.5.5 remote-as=\
    65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```

- Роутер R01.NY

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=5.5.5.5
/routing ospf instance
set [ find default=yes ] router-id=5.5.5.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.5.2/30 interface=ether2 network=10.0.5.0
add address=192.168.20.1/24 interface=ether3 network=192.168.20.0
add address=5.5.5.5 interface=Lo0 network=5.5.5.5
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:777 import-route-targets=65530:777 interfaces=ether3 \
    route-distinguisher=65530:777 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=5.5.5.5
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer5 remote-address=4.4.4.4 remote-as=\
    65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

- Роутер R01.SVL

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=6.6.6.6
/routing ospf instance
set [ find default=yes ] router-id=6.6.6.6
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.6.2/30 interface=ether2 network=10.0.6.0
add address=192.168.30.1/24 interface=ether3 network=192.168.30.0
add address=6.6.6.6 interface=Lo0 network=6.6.6.6
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:777 import-route-targets=65530:777 interfaces=ether3 \
    route-distinguisher=65530:777 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=6.6.6.6
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer6 remote-address=3.3.3.3 remote-as=\
    65530 update-source=Lo0
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
add address=192.168.10.2/24 interface=ether2 network=192.168.10.0
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
add address=192.168.20.2/24 interface=ether2 network=192.168.20.0
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
add address=192.168.30.2/24 interface=ether2 network=192.168.30.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC3
```

2. 2 часть работы

- R01.SPB

```
/interface bridge
add name=VPLSb
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:39:C2:02:4A:28 name=VPLS1 \
    remote-peer=5.5.5.5 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:07:2A:3D:7A:3F name=VPLS2 \
    remote-peer=6.6.6.6 vpls-id=10:0
/interface bridge port
add bridge=VPLSb interface=ether2
add bridge=VPLSb interface=VPLS1
add bridge=VPLSb interface=VPLS2
```

- R01.NY

```
/interface bridge
add name=VPLSb
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:74:E7:AB:72:59 name=VPLS1 \
    remote-peer=1.1.1.1 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:16:82:73:0D:BE name=VPLS3 \
    remote-peer=6.6.6.6 vpls-id=10:0
/interface bridge port
add bridge=VPLSb interface=ether3
add bridge=VPLSb interface=VPLS1
add bridge=VPLSb interface=VPLS3
```

- R01.SVL

```
/interface bridge
add name=VPLSb
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:38:4E:AE:2D:A8 name=VPLS2 \
    remote-peer=1.1.1.1 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:B2:40:AD:5D:2B name=VPLS3 \
    remote-peer=5.5.5.5 vpls-id=10:0
/interface bridge port
add bridge=VPLSb interface=ether3
add bridge=VPLSb interface=VPLS2
add bridge=VPLSb interface=VPLS3
```

- PC1

```
/ip address
add address=192.168.0.1/24 interface=ether2 network=192.168.0.0
```

- PC2

```
/ip address
add address=192.168.0.2/24 interface=ether2 network=192.168.0.0
```

- PC3

```
/ip address
add address=192.168.0.3/24 interface=ether2 network=192.168.0.0
```

4. Результаты пингов, проверки локальной связности.

- Результаты пингов на роутерах
  ![screenshot](./assets/Screenshot_1.png)
  ![screenshot](./assets/Screenshot_2.png)
  ![screenshot](./assets/Screenshot_3.png)
  ![screenshot](./assets/Screenshot_4.png)
- bgp peer на роутере LBN
  ![screenshot](./assets/Screenshot_5.png)
- Проверка связности между VRF в 1 части
  ![screenshot](./assets/Screenshot_16.png)
  ![screenshot](./assets/Screenshot_17.png)
- Проверка связности между компьютерами
  ![screenshot](./assets/Screenshot_18.png)
  ![screenshot](./assets/Screenshot_19.png)

**Вывод**:Выполнив данную работу я изучил протоколы BGP, MPLS и правила организации L3VPN и VPLS.
