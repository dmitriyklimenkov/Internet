# Основные протоколы сети интернет

# Схема сегмента сети:

![](https://github.com/dmitriyklimenkov/Internet/blob/main/%D0%9E%D0%B1%D1%89%D0%B0%D1%8F%20%D1%81%D1%85%D0%B5%D0%BC%D0%B0.PNG)

# Задание:
1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001
2. Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042
3. Настроите статический NAT для R20
4. Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления
5. Настроите статический NAT(PAT) для офиса Чокурдах
6. Настроите для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP
7. Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13

# 1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
Небходимо выделить блок адресов для NAT, создать ACL и пул, и объявить их в BGP. У меня будет сеть 14.20.20.0/24 для R15 и 15.20.20.0/24.
Конфигурация R14:
```
!
interface Ethernet0/0
 ip address 152.95.24.50 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::14 link-local
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/1
 ip address 152.95.24.5 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::14 link-local
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/2
 ip address 14.20.20.10 255.255.255.0 secondary
 ip address 194.14.123.1 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address FE80::14 link-local
 ipv6 address 200C:C0FE:1111:10::1/64
 ipv6 enable
!
interface Ethernet0/3
 ip address 152.95.24.9 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::14 link-local
 ipv6 enable
 ipv6 ospf 1 area 101
!

router bgp 1001
address-family ipv4
network 14.20.20.0 mask 255.255.255.0

ip nat pool POOL1 14.20.20.1 14.20.20.1 netmask 255.255.255.0
ip nat source list 10 interface Ethernet0/2 overload

ip route 14.20.20.0 255.255.255.0 Null0

access-list 10 permit 152.95.0.0 0.0.31.255
```

Конфигурация R15:
```
interface Ethernet0/0
 ip address 152.95.24.13 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::15 link-local
 ipv6 enable
!
interface Ethernet0/1
 ip address 152.95.24.6 255.255.255.252
 ipv6 address FE80::15 link-local
 ipv6 enable
!
interface Ethernet0/2
 ip address 15.20.20.10 255.255.255.0 secondary
 ip address 194.14.123.5 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address FE80::15 link-local
 ipv6 address 200C:C0FE:1111:20::1/64
 ipv6 enable
!
interface Ethernet0/3
 ip address 152.95.24.21 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::15 link-local
 ipv6 enable
!
ip nat pool POOL1 15.20.20.1 15.20.20.1 netmask 255.255.255.0
ip nat inside source list 11 pool POOL1 overload

ip route 15.20.20.0 255.255.255.0 Null0

access-list 11 permit 152.95.0.0 0.0.31.255
```

Пропингуем маршрутизатор Чокурдах  и посмотрим в NAT на R15
```
VPCS> ping 194.14.123.34

84 bytes from 194.14.123.34 icmp_seq=1 ttl=250 time=2.897 ms
84 bytes from 194.14.123.34 icmp_seq=2 ttl=250 time=2.442 ms
84 bytes from 194.14.123.34 icmp_seq=3 ttl=250 time=2.726 ms
84 bytes from 194.14.123.34 icmp_seq=4 ttl=250 time=2.613 ms
84 bytes from 194.14.123.34 icmp_seq=5 ttl=250 time=2.453 ms

R15#sh ip nat tra
Pro Inside global      Inside local       Outside local      Outside global
udp 15.20.20.1:59313   152.95.0.20:59313  194.14.123.34:59314 194.14.123.34:59314
```

# 2. Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
Небходимо выделить блок адресов для NAT, создать ACL и пул, и объявить их в BGP. У меня будет сеть 18.20.20.0/24.

```
interface Ethernet0/0
 ip address 31.52.6.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::18 link-local
 ipv6 enable
!
interface Ethernet0/1
 ip address 31.52.6.5 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::18 link-local
 ipv6 enable
!
interface Ethernet0/2
 ip address 194.14.123.9 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address FE80::18 link-local
 ipv6 address 200C:C0FE:1111:50::1/64
 ipv6 enable
 crypto map MAP
!
interface Ethernet0/3
 ip address 194.14.123.13 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address FE80::18 link-local
 ipv6 address 200C:C0FE:1111:60::1/64
 ipv6 enable
!
ip nat pool POOL1 18.20.20.1 18.20.20.5 netmask 255.255.255.0
ip nat inside source list 11 pool POOL1 overload

ip route 18.20.20.0 255.255.255.0 Null0

access-list 11 permit 31.52.0.0 0.0.7.255
```

Пропингуем маршрутизатор Чокурдах  и посмотрим в NAT на R18
```
R18#sh ip nat tra
Pro Inside global      Inside local       Outside local      Outside global
icmp 18.20.20.1:30     31.52.6.6:30       194.14.123.34:30   194.14.123.34:30
```

# 3. Настроите статический NAT для R20.

```
!
interface Ethernet0/0
 ip address 152.95.24.22 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ip ospf 1 area 102
 ipv6 address FE80::20 link-local
 ipv6 enable
 ipv6 ospf 1 area 102
!

ip nat inside source static 152.95.24.22 15.20.20.10
```
# 4. Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
Необходимо настроить secondary адреса на R14 и R15 и пробросить порт 23 для подключения по telnet.
Для R15:
```
!
interface Ethernet0/2
 ip address 15.20.20.10 255.255.255.0 secondary
 ip nat outside
 
ip nat inside source static tcp 152.95.24.10 23 15.20.20.10 23
```
Для R14:
```
interface Ethernet0/2
 ip address 14.20.20.10 255.255.255.0 secondary
 ip nat outside

ip nat inside source static tcp 152.95.24.10 23 14.20.20.10 23
```
С R18 подключимся по telnet:
```
R18#telnet 14.20.20.10
Trying 14.20.20.10 ... Open

User Access Verification

Username:
```
Сессия поднимается.

# 5. Настроите статический NAT(PAT) для офиса Чокурдах.
Т.к. в офисе Чокурдах нет BGP, необходимо создать secondary адреса, пул выделенных адресов, настроить prefix-list, через route-map матчить адреса и привязать route-map к NAT.

```
!
interface Ethernet0/1
 ip address 28.20.30.10 255.255.255.0 secondary
 ip address 194.14.123.30 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address FE80::28 link-local
 ipv6 address 200C:C0FE:1111:80::1/64
 ipv6 enable
!
interface Ethernet0/2
 no ip address
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::28 link-local
 ipv6 enable
!
interface Ethernet0/2.3
 encapsulation dot1Q 3
 ip address 201.193.45.1 255.255.255.192
 ip nat inside
 ip virtual-reassembly in
 ip policy route-map TO_R26
!
interface Ethernet0/2.4
 encapsulation dot1Q 4
 ip address 201.193.45.65 255.255.255.192
 ip nat inside
 ip virtual-reassembly in
 ip policy route-map TO_R25
!

ip nat pool POOL1 28.20.20.10 28.20.20.10 netmask 255.255.255.0
ip nat pool POOL2 28.20.20.20 28.20.20.20 netmask 255.255.255.0
ip nat source route-map NAT30 pool POOL1 overload
ip nat source route-map NAT300 pool POOL2 overload

ip route 28.20.20.0 255.255.255.0 null0

!
route-map NAT30 permit 10
 match ip address prefix-list NAT1
 match interface Ethernet0/0
!
route-map NAT300 permit 10
 match ip address prefix-list NAT1
 match interface Ethernet0/1
!
ip prefix-list NAT1 seq 5 permit 201.193.45.0/24
```

# 6. Настроите для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
Настройка на R12:
```
!
ip dhcp excluded-address 152.95.0.1 152.95.0.10
!
ip dhcp pool POOL
 network 152.95.0.0 255.255.240.0
 default-router 152.95.0.1
!
```
Проверим:
```
VPCS> dhcp -r
DORA IP 152.95.0.13/20 GW 152.95.0.1
```
Настройка на R13 такая же.

# 7. Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.
На R12 и R13:
```
clock timezone MSK 3
clock calendar-valid
ntp master 3
```
На остальных команда ntp server {ip-сервера}.
Проверка
```
R15#
*Feb 17 22:04:47.408: %SYS-5-CONFIG_I: Configured from console by console
R15#sh ntp sta
Clock is synchronized, stratum 4, reference is 12.12.12.1
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 2400 (1/100 of seconds), resolution is 4000
reference time is E3D81180.933334C8 (00:04:48.575 EET Thu Feb 18 2021)
clock offset is 0.5000 msec, root delay is 1.00 msec
root dispersion is 4005.31 msec, peer dispersion is 439.41 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is 0.000000000 s/s
system poll interval is 64, last update was 8 sec ago.
```



