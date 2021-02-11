# DMVPN

# Лабораторная работа: DMVPN.
# Задание:
1. Настроить GRE между офисами Москва и С.-Петербург.
2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги.
3. Все узлы в офисах в лабораторной работе должны иметь IP связность.
4. План работы и изменения зафиксированы в документации.

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

# 2. Настроить DMVMN между Москва-R15 и Чокурдах-R28, Лабытнанги-R27.

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
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
end
```

Конфигурация R28:
```
!
interface Tunnel2
 ip address 10.10.20.2 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 194.14.123.5
 ip nhrp map 10.10.20.1 194.14.123.5
 ip nhrp network-id 50
 ip nhrp nhs 10.10.20.1
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
end
```

Конфигурация R27:
```
!
interface Tunnel2
 ip address 10.10.20.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 194.14.123.5
 ip nhrp map 10.10.20.1 194.14.123.5
 ip nhrp network-id 50
 ip nhrp nhs 10.10.20.1
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
end
```
Проверим связность:

![](https://github.com/dmitriyklimenkov/DMVPN/blob/main/ping%20R15-R27-28.PNG)

Запустим трассировку из офиса Чокурдах в Лабытнанги:

![](https://github.com/dmitriyklimenkov/DMVPN/blob/main/trace%20R28-R27.PNG)

Видно, что сначала пакеты пошли через хаб, который настроен в Москве, при втором запуске пакеты пошли напрямую в Лабытнанги.
