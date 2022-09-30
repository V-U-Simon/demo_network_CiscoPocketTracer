---
aliases: [ ]
created: 2022.09.28 10:01:25 вечера
modified: 2022.09.28 10:02:50 вечера
---
#🌐network

>[!cite]- Source
> - ⭐⭐ youtube - [Cisco часть I, урок 11. Cisco VLAN](https://www.youtube.com/watch?v=NriV67DU6eg&t=684s)

Настройка [[🌐 VLAN|виртуальной локальной сети]] на [[📡 cisco]]

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
