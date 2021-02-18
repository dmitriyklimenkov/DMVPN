# DMVPN

# Лабораторная работа: DMVPN.
# Задание:
1. Настроить GRE между офисами Москва и С.-Петербург.
2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги.
3. Все узлы в офисах в лабораторной работе должны иметь IP связность.
4. План работы и изменения зафиксированы в документации.

Схема сегмента сети:

![](https://github.com/dmitriyklimenkov/DMVPN/blob/main/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D0%B0.PNG)

# 1. Настроим GRE между офисом Москва-R15 и Петербург-R18.

Сеть туннеля - 10.10.10.0/24.
Внешний IP адрес офиса Москва 194.14.123.5, адрес туннеля 10.10.10.1.
Внешний IP адрес офиса Москва 194.14.123.9, адрес туннеля 10.10.10.2.

Конфигурация R15:
```
!
interface Tunnel1
 ip address 10.10.10.1 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 194.14.123.5
 tunnel destination 194.14.123.9
end
```

Конфигурация R18:
```
!
interface Tunnel1
 ip address 10.10.10.2 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 194.14.123.9
 tunnel destination 194.14.123.5
end
```
Проверка связности:

![](https://github.com/dmitriyklimenkov/DMVPN/blob/main/ping%20R15-R18.PNG)

Пинг проходит, связность есть.

# 2. Настроить DMVMN Phase 2 между офисами Москва-R15 и Чокурдах-R28, Лабытнанги-R27 и поднять OSPF.

Сеть туннеля - 10.10.20.0/24.
Внешний IP адрес офиса Москва 194.14.123.5, адрес туннеля 10.10.20.1.
Внешний IP адрес офиса Чокурдах 194.14.123.34, адрес туннеля 10.10.20.2.
Внешний IP адрес офиса Лабытнанги 194.14.123.26, адрес туннеля 10.10.20.3.

Конфигурация R15:
```
!
interface Tunnel2
 ip address 10.10.20.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 50
 ip tcp adjust-mss 1360
 ip ospf network broadcast
 ip ospf priority 255
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
end

router ospf 1
 network 10.10.20.0 0.0.0.255 area 0
 network 12.12.12.0 0.0.0.255 area 0
 network 152.95.0.0 0.0.31.255 area 0
 network 10.10.20.0 0.0.0.255 area 0
 network 12.12.12.0 0.0.0.255 area 0
```

Конфигурация R28:
```
interface Tunnel2
 ip address 10.10.20.2 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 194.14.123.5
 ip nhrp map 10.10.20.1 194.14.123.5
 ip nhrp network-id 50
 ip nhrp nhs 10.10.20.1
 ip tcp adjust-mss 1360
 ip ospf network broadcast
 ip ospf priority 0
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
end

router ospf 100
 network 10.10.20.0 0.0.0.255 area 0
 network 201.193.45.0 0.0.0.255 area 0
```

Конфигурация R27:
```
interface Tunnel2
 ip address 10.10.20.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 194.14.123.5
 ip nhrp map 10.10.20.1 194.14.123.5
 ip nhrp network-id 50
 ip nhrp nhs 10.10.20.1
 ip tcp adjust-mss 1360
 ip ospf network broadcast
 ip ospf priority 0
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
end

router ospf 100
 network 10.10.20.0 0.0.0.255 area 0
 network 27.27.27.27 0.0.0.0 area 0
```
Проверим связность:

![](https://github.com/dmitriyklimenkov/DMVPN/blob/main/ping%20R15-R27-28.PNG)

Запустим трассировку из офиса Чокурдах в Лабытнанги:

![](https://github.com/dmitriyklimenkov/DMVPN/blob/main/trace%20R28-R27.PNG)

Видно, что сначала пакеты пошли через хаб, который настроен в Москве, при втором запуске пакеты пошли напрямую в Лабытнанги.

Проверим таблицу маршрутизации в Чокурдахе:
```
28#sh ip rou os
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 194.14.123.33 to network 0.0.0.0

      12.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
O IA     12.12.12.1/32 [110/1011] via 10.10.20.1, 00:00:04, Tunnel2
O        12.12.12.2/32 [110/1001] via 10.10.20.1, 00:00:04, Tunnel2
      152.95.0.0/16 is variably subnetted, 7 subnets, 3 masks
O IA     152.95.0.0/20 [110/1020] via 10.10.20.1, 00:00:04, Tunnel2
O        152.95.24.4/30 [110/1010] via 10.10.20.1, 00:00:04, Tunnel2
O IA     152.95.24.8/30 [110/1020] via 10.10.20.1, 00:00:04, Tunnel2
O        152.95.24.12/30 [110/1010] via 10.10.20.1, 00:00:04, Tunnel2
O        152.95.24.24/30 [110/1020] via 10.10.20.1, 00:00:04, Tunnel2
O        152.95.24.48/30 [110/1020] via 10.10.20.1, 00:00:04, Tunnel2
O        152.95.25.1/32 [110/1011] via 10.10.20.1, 00:00:04, Tunnel2
```

Все маршруты прилетели.

Запустим пинг с VPC7 из Москвы до VPC30 Чокурдах:

VPCS> ping 201.193.45.2
```
84 bytes from 201.193.45.2 icmp_seq=1 ttl=57 time=3.274 ms
84 bytes from 201.193.45.2 icmp_seq=2 ttl=57 time=4.757 ms
84 bytes from 201.193.45.2 icmp_seq=3 ttl=57 time=3.231 ms
84 bytes from 201.193.45.2 icmp_seq=4 ttl=57 time=2.570 ms
```
