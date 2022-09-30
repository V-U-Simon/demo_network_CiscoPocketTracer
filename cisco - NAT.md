---
aliases: [ ]
created: 2022.09.28 10:03:30 вечера
modified: 2022.09.28 11:22:51 вечера
---
#🌐network

>[!cite]- Source
> - [PORT FORWARDING: ТЕОРИЯ И НАСТРОЙКА CISCO](https://wiki.merionet.ru/seti/14/port-forwarding-teoriya-i-nastrojka-cisco/?ysclid=l7w2sznn7x239314828#)
> - [NAT на Cisco. Часть 1](https://habr.com/ru/post/108931/?ysclid=l7w2tk3tcr85297178)

Настройка трансляций [[🌐 NAT|🌐 NAT и 🌐 PAT]] на [[📡 cisco|cisco]]

```Bash
show ip nat statistics     # 👀 просмотр inside/outside интерфесов
show ip nat translations   # 👀 просмотр трансляций NAT
# Pro	Inside Global		    Inside Local	Outside local		    Outside global
# tcp	212.193.249.136:8080	192.168.1.10:80	212.193.249.17:46088	212.193.249.17:46088 

no ip nat create flow-entries #  Отключить информативных записи о статической трансляции 
ip nat translation ? # ⏱️ изменить различные таймауты для NAT

clear ip nat translation * # ❌ удаление динамических записей в NAT
```

# Статический

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
ip nat inside source static 192.168.0.100 7.7.7.1     # статическая трансляция 192.168.0.100 в 7.7.7.1
#                           ^ что         ^ куда
```

Static PAT

```Bash
# 1. Настраиваем и Маркируем интерфейсы
# 2. Создаем трансляцию | static PAT (Port Forwarding)
ip nat inside source static tcp 192.168.0.100 80  6.6.6.1 80  # статическая трансляция 192.168.0.100:80  в 6.6.6.1:80 
ip nat inside source static tcp 192.168.0.101 443 6.6.6.1 443 # статическая трансляция 192.168.0.101:443 в 6.6.6.1:443
#                           ^ что                 ^ куда
```

# Динамический

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
# 4. Создаем трансляцию | маршрутизатор будет менять не только IP-адреса, но и порты
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
