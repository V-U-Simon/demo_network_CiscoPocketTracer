---
aliases: [ ]
created: 2022.09.28 10:05:04 вечера
modified: 2022.09.28 10:06:08 вечера
---
#🌐network

>[!cite]- Source
> - ❌[НАСТРОЙКА GRE ТУННЕЛЯ НА CISCO](https://wiki.merionet.ru/seti/22/nastroyka-gre-tunnelya-na-cisco/?ysclid=l7x5o427uk817773036#)
> - ⭐[CISCO GRE](https://arny.ru/education/ccna-rs/cisco-gre/?ysclid=l7xes7yvs961697034)

Релазизация туннелирования ([[🌐 GRE]]) на [[📡 cisco]]

```Bash
# 192.168.1.1 | 1.1.1.10 ---------------------------------- 2.2.2.10 | 172.20.1.1
#                        \__172.16.0.1 tunnel 172.16.0.2__/
```

# Настройка туннеля gre

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

# Настройка туннеля gre ipsec (с шифрованием)

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
