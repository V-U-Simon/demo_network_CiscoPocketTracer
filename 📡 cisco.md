---
aliases: ['cisco',  ]
created: 2022.09.05 8:00:17 вечера
modified: 2022.09.30 9:40:31 утра
---
#🌐network

# cisco

>[!cite]- Source
> - [CISCO IOS: СОХРАНЕНИЕ КОНФИГУРАЦИИ](https://wiki.merionet.ru/seti/62/cisco-ios-soxranenie-konfiguracii/?ysclid=l7r63sqxop817630303)

Cisco IOS (Internetwork Operating System — Межсетевая операционная система)

- [VLAN](cisco%20-%20VLAN.md)
- [DHCP](cisco%20-%20DHCP.md)
- [NAT](cisco%20-%20NAT.md)
- [GRE](cisco%20-%20GRE.md)
- [routing](cisco%20-%20routing.md)
- [ssh](cisco%20-%20ssh.md)
- [telnet](cisco%20-%20telnet.md)

```Bash
# РЕЖИМЫ
enable              # привилегированный       | en
configure terminal  # глобальной конфигурации | conf ter
no ip domain lookup # ⏩ выключить разрешение имен

# 👀 ПРОСМОТР:
show running-config # текущая конфигурацияБББ

show ip route           # таблицы маршрутизации - все
show ip route connected # таблицы маршрутизации - присоединенные
show ip route static    # таблицы маршрутизации - статические
show ip route ospf      # таблицы маршрутизации - OSPF

show ip int brief # список IP - интерфейс - состояние

show arp          # таблицы ARP (на роутерах)
show mac          # таблицы коммутации (на свичах) address-table
show dhcp         # настроек DHCP
show ip protocols # 
sh vlan
sh int gigabitEthernet 0/1 switchport

# ДРУГИЕ КОМАНДЫ
ping -t 192.168.0.1 # бесконечный пинг (-t)
traceroute 192.168.0.1 # прослеживание пути
nslookup # утилита позволяет обратиться к DNS-серверу и узнать у него информацию о имени или IP-адресе
hostname R1 # переименовать хост
reload #  перезагрузка
```

## Настройка интерфейсов

```Bash
# 1. Первоначально переходим в режим конфигурации
enable             # привилегированный          | en
configure terminal # глобальной конфигурации    | conf ter


# 2.1 Настройка интерфейса
interface fastEthernet 0/0   # выбор интерфейса | int fa0/0
interface loopback 0   # виртуальный интерфейс  | int loopback 0

# 2.2 Настройка IP
ip address 192.168.1.254 255.255.255.0 # привязка IP к интерфейсу
duplex auto
speed auto
no shutdown # включить этот интерфейс           | no sh
exit        # выход из настройки этого интерфейса


# 3. Дополнительные настройки различных протоколов (опционально)
# 4. Сохранение конфигурации
copy running-config startup-config # сохраниеть в загрузочную конфигурацию | copy run sta
# иначе после перезагрузки коммутатора все сбросится
```
