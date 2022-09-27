---
aliases: [ ]
created: 2022.09.05 8:00:17 вечера
modified: 2022.09.13 9:25:41 утра
---
#🌐network

>[!cite]- Source
> - [CISCO IOS: СОХРАНЕНИЕ КОНФИГУРАЦИИ](https://wiki.merionet.ru/seti/62/cisco-ios-soxranenie-konfiguracii/?ysclid=l7r63sqxop817630303)

Cisco IOS (Internetwork Operating System — Межсетевая операционная система)

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

show arp	      # таблицы ARP (на роутерах)
show mac          # таблицы коммутации (на свичах) address-table
show dhcp	      # настроек DHCP
show ip protocols # 
# Команды для show:
sh vlan
sh int gigabitEthernet 0/1 switchport

# ДРУГИЕ КОМАНДЫ
ping -t 192.168.0.1 # бесконечный пинг (-t)
traceroute 192.168.0.1 # прослеживание пути
nslookup # утилита позволяет обратиться к DNS-серверу и узнать у него информацию о имени или IP-адресе
hostname R1 # переименовать хост
reload #  перезагрузка
```

# Настройка интерфейсов

```Bash
# 1. Первоначально переходим в режим конфигурации
enable             # привилегированный          | en
configure terminal # глобальной конфигурации    | conf ter


# 2. Настройка IP
interface fastEthernet 0/0   # выбор интерфейса | int fa0/0
interface loopback 0   # виртуальный интерфейс  | int loopback 0

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

# Статическая маршрутизация

```Bash
# 10.0.0.0 | 192.168.0.1 - 192.168.0.2 | 10.0.1.0
```

```bash 
# Роутер 1
en
conf term
ip addr 192.168.0.1 255.255.255.0
no sh

interface FastEthernet0/1
ip route 10.0.0.0 255.255.255.0 192.168.0.2 # подсеть / маска / адрес  следующего устройства 
```

```Bash
# Роутер 2
en
conf term
ip addr 192.168.0.2 255.255.255.0
no sh

interface FastEthernet0/1
ip route 10.0.1.0 255.255.255.0 192.168.0.1 # подсеть / маска / адрес  следующего устройства 
```

# Маршрутизация [[🌐 OSPF]]

>[!cite]- Source
> - ⭐⭐youtube - [Протокол динамической маршрутизации OSPF](https://www.youtube.com/watch?v=AcL_5BI3U_c&list=PLvGOckMhKg76rMzrows3RqNBmQeeBK-sn&index=4)

>[!example]- Команды
>
> ```Bash
> router ospf 1 # включение OSPF процесса на роутре (номер процесса может быть любой)
> router-id 1.1.1.1 # назаначение Router-ID (просто число для удобочитаемости)
> # У кого болье ID тот и будет DR, если не указан приоритет
> 
> 
> # Добавление сетей
> #       |__сеть___| |_mask__| - wildcard mask (обратная маска подсети)
> network 192.168.1.0 0.0.0.255 area 0     # включить  сеть      в процесс   OSPF
> # интерфейсы смотрящие в оконечные сети следует делать пассивными
> passive-interface gigabitEthernet 0/0    # исключить интерфейс из процесса OSPF
> no passive-interface gigabitEthernet 0/0 # включить  интерфейс из процесса OSPF
> 
> 
> # 👀 ПРОСМОТР:
> show ip ospf int brief # интерфейсов, включенных в OSPF процесс
> show ip ospf int       # интерфейсов, включенных в OSPF процесс
> show ip ospf neighbor  # соседей по OSPF
> show ip ospf database  # 
> show ip route ospf     # маршрутов, внесенных в таблицу маршрутизации по протоколу OSPF
> ```


```Bash
# 192.168.1.1 | 10.0.1.1 - switch - 10.0.1.2 | 192.168.2.1
```

```Bash
# Роутер 1 - 192.168.1.1 | 10.0.1.1
en
conf term
router ospf 1 # включение OSPF процесса на роутре (номер процесса может быть любой)
# Необходимо аннонсировать все подлключенные сети учавствовующие в OSPF
network 192.168.1.0 0.0.0.255 area 0     # аннонсировать сеть в процесс OSPF
network 10.0.1.0    0.0.0.255 area 0     # аннонсировать сеть в процесс OSPF
```

```Bash
# Роутер 2 - 192.168.2.1 | 10.0.1.2
en
conf term
router ospf 1
network 192.168.2.0 0.0.0.255 area 0
network 10.0.1.0    0.0.0.255 area 0
```

OSPF c несколькими регионами

```Bash
# 192.168.1.1 | 10.0.1.1 - 10.0.1.2 ABR router 10.0.2.1 - 10.0.2.2 | 172.20.1.1
# \____________area 0_____________/            \____________area 1_____________/
```

```Bash
# ABR router
en
conf term
router ospf 1 # включение OSPF процесса на роутре (номер процесса может быть любой)
# Необходимо аннонсировать все подлключенные сети учавствовующие в OSPF
network 10.0.1.0  0.0.0.255 area 0     # аннонсировать сеть в процесс OSPF
network 10.0.2.0  0.0.0.255 area 1     # аннонсировать сеть в процесс OSPF
exit

# ВЫБОР DR (рассылающий ASL сообщения, для посройки карты сети)
# настройка интерфейса смотрящего в area 0
conf term 
interfase fa0/0
ip ospf priority 100 # увеличить приоритет для выбора DR роутером
exit
# настройка интерфейса смотрящего в area 1
conf term
interfase fa0/1
ip ospf priority 100 # увеличить приоритет для выбора DR роутером
exit

clear ip ospf process # перезагрузить OSPF для перевыборов DR
```

```Bash
# Роутер 1 - area 0
en
conf term
router ospf 1 # включение OSPF процесса на роутре (номер процесса может быть любой)
# Необходимо аннонсировать все подлключенные сети учавствовующие в OSPF
network 192.168.1.0 0.0.0.255 area 0     # аннонсировать сеть в процесс OSPF
network 10.0.1.0    0.0.0.255 area 0     # аннонсировать сеть в процесс OSPF
```

```Bash
# Роутер 2 - area 1
en
conf term
router ospf 1 # включение OSPF процесса на роутре (номер процесса может быть любой)
# Необходимо аннонсировать все подлключенные сети учавствовующие в OSPF
network 172.20.1.0  0.0.0.255 area 1     # аннонсировать сеть в процесс OSPF
network 10.0.2.0    0.0.0.255 area 1     # аннонсировать сеть в процесс OSPF
```

# Настройка [[🌐 VLAN]]

>[!cite]- Source
> - ⭐⭐ youtube - [Cisco часть I, урок 11. Cisco VLAN](https://www.youtube.com/watch?v=NriV67DU6eg&t=684s)

```Bash
# 1. VLAN на свиче
vlan 2      # ✅ создаем VLAN 2 (VLAN 1 по умолчанию зарезервирован)
no vlan 10  # ❌ удалить VLAN
name Admins #    имя     VLAN


# 👀 ПРОСМОТР 
show interfaces trunk # информации о trunk портах
show vlan             # таблица VLAN
show vlan brief       # короткая таблица VLAN
# VLAN Name                             Status    Ports
# ---- -------------------------------- --------- -------------------------------
# 1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
#                                                 Fa0/5, Fa0/6, Fa0/7, Fa0/8
#                                                 Fa0/9, Fa0/10, Fa0/11, Fa0/12
#                                                 Fa0/13, Fa0/14, Fa0/15, Fa0/16
#                                                 Fa0/17, Fa0/18, Fa0/19, Fa0/20
#                                                 Fa0/21, Fa0/22, Fa0/23, Fa0/24
# 300  Admins                           active    
# 1002 fddi-default                     active    
# 1003 token-ring-default               active    
# 1004 fddinet-default                  active    
# 1005 trnet-default                    active  
# *******************************************************************************
# 1. номер VLAN
# 2. имя VLAN    | name Admins
# 3. статус
# 4. порты       | каким VLAN-ам принадлежат порты
```

```Bash
# Настройка 1го порта - ACCESS
interface fastEthernet 0/1 # выбор интерфейса
switchport mode access     # перевод в режим access
switchport access vlan 2   # закрепляем 2ой VLAN
# Настройка с 2 по 5  порты - ACCESS
interface range fastEthernet 0/2-5 # выбираем пул интерфейсов
switchport mode access
switchport access vlan 2
# Настройка порта смотрящего на другое сетевое оборудование - TRUNK
interface ge 0/1      
switchport mode trunk      # переводим порт в режим TRUNK 
# фильтрация пакетов в trunk
switchport trunk allowed vlan 2, 3     # объявить access для 2 и 3 vlan
switchport trunk allowed vlan add 4    # добавить
switchport trunk allowed vlan remove 4 # удалить


# 2. VLAN на роутере:
interface fastEthernet 0/0.2 # субинтерфейс - виртуальный (логический) интерфейс
encapsulation dot1Q 2        # определяем VLAN, тут 2ой
ip address 192.168.1.1 255.255.255.0 # добавляем к нему IP

interface fastEthernet 0/0.3
encapsulation dot1Q 3
ip address 192.168.2.1 255.255.255.0

interface fastEthernet 0/0.4
encapsulation dot1Q 4
ip address 192.168.3.1 255.255.255.0
```

> Вся информациях о VLAN хранится в flash памяти в файле `vlan.dat`

# Настройка [[🌐 DHCP]]

```Bash
# Настройка DHCP-сервера на роутере. DHCP будет раздавать адреса, если настроен интерфейс в той же сети, адреса которого раздаются.
ip dhcp excluded-address 192.168.0.1 192.168.0.10   # исключаем диапазон адресов, которые раздавать не будем (нарпимер адрес самого роутера)
ip dhcp pool TEST			   # создаем пул с именем
network 10.0.0.0 255.255.255.0 # описываем из какой сети раздаем адреса
default-router 10.0.0.1		   # какой шлюз по умолчанию будем раздавать
dns-server 8.8.8.8			   # какой DNS по умолчанию будем раздавать
ip dhcp pool 192_NET		   # пулов может быть несколько
network 192.168.0.0 255.255.255.0
default-router 192.168.0.1
dns-server 4.4.4.4

# На роутере
# ip helper-address 192.168.3.2 # куда стоит обращаться для получения IP 

```

# Настройка [[🌐telnet]]

```Bash
# TELNET
enable # переход в привилегированный режим
configure terminal # переход в режим глобальной конфигурации. 
# В этом режиме возможен ввод команд, позволяющих конфигурировать общие характеристики системы
# Из режима глобальной конфигурации можно перейти во множество режимов конфигурации, специфических для конкретного протокола или функции.
username admin secret cisco # создаем пользователя с именем admin и паролем cisco.

# Настройка интерфейса - подключение по telnet
interface vlan 1 # переходим в виртуальный интерфейс и повесим на него IP-адрес. 
ip address 192.168.1.254 255.255.255.0 # присваиваем адрес 192.168.1.254 с маской 255.255.255.0
no shutdown # по умолчанию интерфейс выключен, поэтому включаем его
# В IOS 90% команд отменяются или выключаются путем приписывания перед командой «no».

line vty 0 15 # переходим в настройки виртуальных линий, где как раз живет Telnet. 
# От 0 до 15 означает, что применяем это для всех линий. 
# Всего можно установить на нем до 16 одновременных соединений.

transport input all # разрешаем соединение для всех протоколов. 

login local # указываем, что учетная запись локальная, и он будет проверять ее с той, что мы создали
copy running-config startup-config # обязательно сохраняем конфигурацию. Иначе после перезагрузки коммутатора все сбросится.
telnet 192.168.1.254       # подключиться по telnet
```

# Настройка [[🌐 SSH]]

```Bash
# SSH 
enable # переход в привилегированный режим
configure terminal # переход в режим глобальной конфигурации. 
username admin secret cisco # создаем пользователя с именем admin и паролем cisco.

# Настройка интерфейса - подключение по ssh
interface vlan 1 # переходим в виртуальный интерфейс и повесим на него IP-адрес. 
ip address 192.168.1.254 255.255.255.0 # присваиваем адрес 192.168.1.254 с маской 255.255.255.0
no shutdown # по умолчанию интерфейс выключен, поэтому включаем его
# .......шаги из telnet

transport input all # разрешаем соединение для всех протоколов. 
hostname SW1 # меняем имя коммутатора. С этим стандартным именем нельзя прописать домен, который нужен для генерации ключей.
ip domain-name cisadmin.ru # прописываем домен
crypto key generate rsa # генерируем RSA ключи
1024 # Указываем размер ключа. По умолчанию предлагается 512, но я введу 1024.

ssh -l admin 192.168.1.254 # подключиться по ssh
```

# Настройка трансляций [[🌐 NAT]] и [[🌐 PAT]]

>[!cite]- Source
> - [PORT FORWARDING: ТЕОРИЯ И НАСТРОЙКА CISCO](https://wiki.merionet.ru/seti/14/port-forwarding-teoriya-i-nastrojka-cisco/?ysclid=l7w2sznn7x239314828#)
> - [NAT на Cisco. Часть 1](https://habr.com/ru/post/108931/?ysclid=l7w2tk3tcr85297178)

```Bash
show ip nat statistics     # 👀 просмотр inside/outside интерфесов
show ip nat translations   # 👀 просмотр трансляций NAT
# Pro	Inside Global		    Inside Local	Outside local		    Outside global
# tcp	212.193.249.136:8080	192.168.1.10:80	212.193.249.17:46088	212.193.249.17:46088 

no ip nat create flow-entries #  Отключить информативных записи о статической трансляции 
ip nat translation ? # ⏱️ изменить различные таймауты для NAT

clear ip nat translation * # ❌ удаление динамических записей в NAT
```

## Статический

Static NAT

```Bash
# 1. Настраиваем и Маркируем интерфейсы
interface GigabitEthernet0/0/0
ip address 8.8.8.1 255.255.255.0
ip nat outside # со стороны интернета
duplex auto
speed auto

interface GigabitEthernet0/0/1
ip address 192.168.0.1 255.255.255.0
ip nat inside # со стороны локальной сети
duplex auto
speed auto

# 2. Создаем трансляцию | Static NAT
ip nat inside source static 192.168.0.100 7.7.7.1     # статическая трансляция 7.7.7.1     в 192.168.0.100 
#                           ^ что         ^ куда
```

Static PAT

```Bash
# 1. Настраиваем и Маркируем интерфейсы
# 2. Создаем трансляцию | static PAT (Port Forwarding)
ip nat inside source static tcp 192.168.0.100 80  6.6.6.1 80  # статическая трансляция 6.6.6.1:80  в 192.168.0.100:80
ip nat inside source static tcp 192.168.0.101 443 6.6.6.1 443 # статическая трансляция 6.6.6.1:443 в 192.168.0.101:443
#                           ^ что                 ^ куда
```

## Динамический

Dynamic NAT

```Bash
# inside source dynamic NAT
# 1. Указываем что транслировать (Access list)
# ip access-list standard NAME # пример имени (ACL_NAME) - NET_10.0.0.0/24
access-list 100 permit tcp 192.168.0.0 0.0.0.255 any # создаем access-list, перечисляющий трафик


# 2. Указываем куда транслировать (Создаем пул из адресов)
ip nat pool NAME_OF_POOL 11.1.1.10 11.1.1.20 netmask 255.255.255.0 # от 11.1.1.10 до 11.1.1.20 включительно


# 3. Маркируем интерфейсы
interface fa 0/0
ip nat inside

interface fa 0/1  
ip nat outside


# 4. Создаем трансляцию
ip nat inside source list 100 pool NAME_OF_POOL # маршрутизатор будет менять IP-адреса
#                    ^ что     ^ куда


# 5. Результат
sh ip nat translations # обратимся с хоста 10.1.1.1 к хосту 11.1.1.2 
#     Источник после   Источник до     Назначение до  Назначение после
# Pro Inside global    Inside local    Outside local  Outside global  
# tcp 11.1.1.10:55209  10.0.1.1:55209  11.1.1.2:23    11.1.1.2:23
# 
#     из роутера       в роутер
```

Dynamic PAT

```Bash
# inside source dynamic NAT with overload
# 1. Указываем, что транслировать (Access list - NAME)
# 2. Указываем куда транслировать (Создаем пул из адресов, если один то пропускаем)
# 3. Маркируем интерфейсы 
# 4. Создаем трансляцию | маршрутизатор будет менять не только IP-адреса и порты
ip nat inside source list NAME pool NAME_OF_POOL overload # если пул  IP
ip nat inside source list NAME int fa0/1         overload # если один IP
#                    ^ что     ^ куда


# 5. Результат
sh ip nat translations   
#     Источник после   Источник до     Назначение до  Назначение после
# Pro Inside global    Inside local    Outside local  Outside global  
# tcp 11.1.1.11:21545  10.0.1.1:21545  11.1.1.2:23    11.1.1.2:23  
# tcp 11.1.1.11:49000  10.0.2.1:49000  11.1.1.2:23    11.1.1.2:23
```

# Туннелирование ([[🌐 GRE]])

>[!cite]- Source
> - ❌[НАСТРОЙКА GRE ТУННЕЛЯ НА CISCO](https://wiki.merionet.ru/seti/22/nastroyka-gre-tunnelya-na-cisco/?ysclid=l7x5o427uk817773036#)
> - ⭐[CISCO GRE](https://arny.ru/education/ccna-rs/cisco-gre/?ysclid=l7xes7yvs961697034)

```Bash
# 192.168.1.1 | 1.1.1.10 ---------------------------------- 2.2.2.10 | 172.20.1.1
#                        \__172.16.0.1 tunnel 172.16.0.2__/
```

Настройка туннеля GRE

```Bash
show ip eigrp neighbors
show ip route
# ...D - EIGRP...
# D 192.168.2.0/24 [90/3584] via 10.10.10.2, 00:12:38, GigabitEthernet0/0
```

```Bash
# Роутер 1
interface Tunnel0
 ip address 172.16.0.1 255.255.255.0 # IP в сети туннельного интерфейса
 mtu 1476 # уменьшанный MTU (Maximum Transfer Unit) | 1400 байт
 tunnel source GigabitEthernet0/0/1
 tunnel destination 4.4.4.2 # публичный IP назначение
 
router eigrp 1
 network 172.16.0.0 0.0.0.255
ip route 192.168.2.0 255.255.255.0 172.16.0.2 # Статический маршрут


# Роутер 2
interface Tunnel0
 ip address 172.16.0.2 255.255.255.0
 mtu 1476
 tunnel source GigabitEthernet0/0/0
 tunnel destination 2.2.2.2 # публичный IP назначение
 
router eigrp 1
 network 172.16.0.0 0.0.0.255
ip route 10.0.0.0 255.255.255.0 172.16.0.2 # Статический маршрут
```

Настройка туннеля GRE IPSec (с шифрованием)

```bash
# Роутер 1
# Настройка ISAKMP (ISAKMP Phase 1)
R1(config)# crypto isakmp policy 1
R1(config-isakmp)# encr 3des 
R1(config-isakmp)# hash md5
R1(config-isakmp)# authentication pre-share
R1(config-isakmp)# group 2
R1(config-isakmp)# lifetime 86400

R1(config)# crypto isakmp key merionet address 2.2.2.10
R1(config)# crypto ipsec transform-set TS esp-3des esp-md5-hmac
R1(cfg-crypto-trans)# mode transport

# создаем профиль IPSec
R1(config)# crypto ipsec profile protect-gre
R1(ipsec-profile)# set security-association lifetime seconds 86400
R1(ipsec-profile)# set transform-set TS

# применить шифрование IPSec к интерфейсу туннеля
R1(config)# interface Tunnel 0
R1(config-if)# tunnel protection ipsec profile protect-gre



# Роутер 2
R2(config)# crypto isakmp policy 1
R2(config-isakmp)# encr 3des
R2(config-isakmp)# hash md5
R2(config-isakmp)# authentication pre-share
R2(config-isakmp)# group 2
R2(config-isakmp)# lifetime 86400
 
R2(config)# crypto isakmp key merionet address 1.1.1.10
R2(config)# crypto ipsec transform-set TS esp-3des esp-md5-hmac
R2(cfg-crypto-trans)# mode transport

R2(config)# crypto ipsec profile protect-gre
R2(ipsec-profile)# set security-association lifetime seconds 86400
R2(ipsec-profile)# set transform-set TS

R2(config)# interface Tunnel 0
R2(config-if)# tunnel protection ipsec profile protect-gre
```
