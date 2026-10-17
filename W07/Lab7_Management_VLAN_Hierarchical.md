# LAB 7: Management VLAN, йерархична топология и Layer 3 switching

**Продължителност:** 120-150 минути (Стъпки 1-6) + около 30 минути (Стъпка 7 – отказоустойчивост)
**Цел:** Изграждане на йерархична мрежа (Access/Distribution/Core) с Layer 3 комутатори, управление през Management VLAN 99 и SSH, HSRP и резервирани връзки
**Prerequisite:** Lab 2 (VLAN, trunk, Част 3А – два комутатора) и Lab 5 (SSH)

> **Надграждащо упражнение:** Lab 7–9 не са задължителни за курсовия проект; основата за проекта са Lab 1–6. Lab 7 няма рутер и използва 5 VLAN-а – в курсовия проект важат ограниченията на заданието (1 Router, 2–4 VLAN-а).

Упражнението се изпълнява в **нов празен файл**.

---

## ЦЕЛИ НА УПРАЖНЕНИЕТО

След завършване на този лаб ще можете да:
- ✅ Създавате правилна йерархична топология с L3 routing
- ✅ Конфигурирате inter-VLAN routing с SVI (Switch Virtual Interface)
- ✅ Прилагате HSRP за gateway redundancy
- ✅ Управлявате комутаторите през Management VLAN 99 и SSH
- ✅ Изграждате redundant links с Spanning Tree Protocol
- ✅ Тествате fault tolerance при отказ на връзка

---

## ТЕОРЕТИЧНА ОСНОВА

### Йерархичен модел - правилна имплементация

```
┌─────────────────────────────────────────────────────────┐
│            CORE LAYER (Гръбнак)                         │
│  • Multilayer Switch 3650 (L3)                          │
│  • IP routing enabled                                   │
│  • SVI за Management VLAN                               │
│  • Root Bridge за Spanning Tree                         │
│  • Redundant uplinks към Distribution                   │
└─────────────────────────────────────────────────────────┘
              ↕ Redundant ↕     ↕ Redundant ↕
┌─────────────────────────────────────────────────────────┐
│         DISTRIBUTION LAYER (Разпределение)               │
│  • Multilayer Switch 3650 (L3) x2                       │
│  • Inter-VLAN routing (SVI за всички VLAN)              │
│  • HSRP за gateway redundancy                           │
│  • QoS policies                                          │
│  • ACL filtering                                         │
│  • Routing protocols (OSPF/EIGRP)                       │
└─────────────────────────────────────────────────────────┘
       ↕         ↕         ↕         ↕
┌─────────────────────────────────────────────────────────┐
│          ACCESS LAYER (Достъп)                          │
│  • Switch 2960 (L2) x4                                  │
│  • VLAN assignment                                       │
│  • PortFast, BPDU Guard                                 │
│  • Port Security                                         │
└─────────────────────────────────────────────────────────┘
    ↕    ↕    ↕    ↕    ↕    ↕    ↕    ↕
  [PC] [PC] [PC] [PC] [PC] [PC] [PC] [PC]
```

---

## ТОПОЛОГИЯ

### Целева топология:

```
                              [Core-SW]
                       (Multilayer Switch 3650)
                     10.99.99.10/24 - Root Bridge
                  Gi1/0/1-2 /                \ Gi1/0/3-4
                 (2 връзки)/                  \(2 връзки)
                          /                    \
             [Dist-SW1 L3]                      [Dist-SW2 L3]
             10.99.99.11                        10.99.99.12
             HSRP Active (110)                  HSRP Standby (100)
             Gi1/0/21-24                        Gi1/0/21-24
                  |                                  |
                  +-> Gi0/1 на Access-SW1...SW4      +-> Gi0/2 на Access-SW1...SW4

  HSRP виртуални gateway-и: 192.168.10.1, 192.168.20.1, 192.168.30.1,
                            192.168.40.1 и 10.99.99.1 (Management)

  [Access-SW1]     [Access-SW2]     [Access-SW3]     [Access-SW4]
    (2960)           (2960)           (2960)           (2960)
  10.99.99.21      10.99.99.22      10.99.99.23      10.99.99.24
       |                |                |                |
    VLAN 10          VLAN 20          VLAN 30          VLAN 40
     Sales         Engineering          HR               IT
    PC1-PC2          PC3-PC4          PC5-PC6          PC7-PC8
```

Всеки Access комутатор има **две** uplink връзки: Gi0/1 към Dist-SW1 и Gi0/2 към Dist-SW2.

**VLAN дизайн:**
- VLAN 10: Sales (192.168.10.0/24) - Gateway: .1 (HSRP)
- VLAN 20: Engineering (192.168.20.0/24) - Gateway: .1 (HSRP)
- VLAN 30: HR (192.168.30.0/24) - Gateway: .1 (HSRP)
- VLAN 40: IT (192.168.40.0/24) - Gateway: .1 (HSRP)
- **VLAN 99: Management** (10.99.99.0/24) - Gateway: 10.99.99.1 (HSRP на Dist-SW1/Dist-SW2)

---

## СТЪПКА 1: Изграждане на топологията

### 1.1 Добавете устройства

В Packet Tracer добавете:
- **3x Multilayer Switch 3650-24PS** (Core-SW, Dist-SW1, Dist-SW2)
- **4x 2960-24TT Switch** (Access-SW1 до Access-SW4)
- **8x PC** (по 2 на всеки Access switch)

**Захранване на 3650:** ако след добавянето портовете на 3650 са червени и CLI не отговаря, отворете **Physical**, плъзнете модула **AC-POWER-SUPPLY** в празния слот за захранване и включете устройството.

**Имена на портовете на 3650:** GigabitEthernet**1/0/1** – **1/0/24** (обикновени портове) и Gi1/1/1 – 1/1/4 (SFP слотове, изискват модул). В упражнението използваме само Gi1/0/x. На 2960 портовете остават Fa0/1 – Fa0/24 и Gi0/1 – Gi0/2.

### 1.2 Кабелиране

Между комутаторите използвайте кабел **Copper Cross-Over**, към PC – **Copper Straight-Through**.

**Core към Distribution (по две паралелни връзки):**
```
Core-SW Gi1/0/1 ↔ Dist-SW1 Gi1/0/1
Core-SW Gi1/0/2 ↔ Dist-SW1 Gi1/0/2  (Redundant link)
Core-SW Gi1/0/3 ↔ Dist-SW2 Gi1/0/1
Core-SW Gi1/0/4 ↔ Dist-SW2 Gi1/0/2  (Redundant link)
```

**Distribution към Access (всеки Access комутатор към двата Distribution):**
```
Dist-SW1 Gi1/0/21 ↔ Access-SW1 Gi0/1     Dist-SW2 Gi1/0/21 ↔ Access-SW1 Gi0/2
Dist-SW1 Gi1/0/22 ↔ Access-SW2 Gi0/1     Dist-SW2 Gi1/0/22 ↔ Access-SW2 Gi0/2
Dist-SW1 Gi1/0/23 ↔ Access-SW3 Gi0/1     Dist-SW2 Gi1/0/23 ↔ Access-SW3 Gi0/2
Dist-SW1 Gi1/0/24 ↔ Access-SW4 Gi0/1     Dist-SW2 Gi1/0/24 ↔ Access-SW4 Gi0/2
```

**Защо две uplink връзки?** HSRP премества gateway-я на Dist-SW2 при отказ на Dist-SW1, но това помага само ако крайните устройства имат физически път до Dist-SW2. STP блокира едната връзка, докато другата работи.

**Access към PC:**
```
Access-SW1: Fa0/1-2 → PC1-PC2 (VLAN 10)
Access-SW2: Fa0/1-2 → PC3-PC4 (VLAN 20)
Access-SW3: Fa0/1-2 → PC5-PC6 (VLAN 30)
Access-SW4: Fa0/1-2 → PC7-PC8 (VLAN 40)
```

---

## СТЪПКА 2: Конфигуриране на Core Switch (Layer 3)

### 2.1 Базова конфигурация и Layer 3 активиране

```cisco
enable
configure terminal
hostname Core-SW
no ip domain-lookup

! Enable secret password
enable secret class123

! Console security
line console 0
 password cisco123
 login
 logging synchronous
 exit

! КРИТИЧНО: Enable IP routing!
ip routing

! VLAN-и (нужни, за да ги пренасят trunk-овете и STP)
vlan 10
 name Sales
vlan 20
 name Engineering
vlan 30
 name HR
vlan 40
 name IT
vlan 99
 name Management
 exit

! SVI за Management VLAN
interface vlan 99
 description Management-Interface
 ip address 10.99.99.10 255.255.255.0
 no shutdown
 exit

! Маршрут към потребителските VLAN-и през HSRP gateway-я на VLAN 99
ip route 0.0.0.0 0.0.0.0 10.99.99.1

! Spanning Tree - Rapid PVST+, Core е Root Bridge
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,99 priority 4096

! Trunk портове към Distribution switches
interface range GigabitEthernet1/0/1 - 4
 description Trunk-to-Distribution
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! Запазване на конфигурацията
end
copy running-config startup-config
```

**Важни бележки:**
- `ip routing` - КРИТИЧНО за L3 functionality
- `spanning-tree vlan ... priority 4096` - Core става Root Bridge
- На комутатор с `ip routing` командата `ip default-gateway` не се използва – затова Core получава маршрут по подразбиране към 10.99.99.1
- На 3650 няма команда `switchport trunk encapsulation` (поддържа се само 802.1Q). На по-старите 3560 тя е задължителна преди `switchport mode trunk`

---

## СТЪПКА 3: Конфигуриране на Distribution Switches (Layer 3)

### 3.1 Distribution-SW1 (PRIMARY за HSRP)

```cisco
enable
configure terminal
hostname Dist-SW1
no ip domain-lookup
enable secret class123

line console 0
 password cisco123
 login
 logging synchronous
 exit

! КРИТИЧНО: Enable IP routing!
ip routing

! VLAN-и
vlan 10
 name Sales
vlan 20
 name Engineering
vlan 30
 name HR
vlan 40
 name IT
vlan 99
 name Management
 exit

! STP: Rapid PVST+; Dist-SW1 е резервен Root след Core
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,99 priority 8192

! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
! MANAGEMENT VLAN 99 + HSRP (gateway за комутаторите)
! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
interface vlan 99
 description Management-Gateway
 ip address 10.99.99.11 255.255.255.0
 standby 99 ip 10.99.99.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
 exit

! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
! INTER-VLAN ROUTING - SVI за всеки VLAN + HSRP
! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

! VLAN 10 - Sales
interface vlan 10
 description Sales-Gateway
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
 no shutdown
 exit

! VLAN 20 - Engineering
interface vlan 20
 description Engineering-Gateway
 ip address 192.168.20.2 255.255.255.0
 standby 20 ip 192.168.20.1
 standby 20 priority 110
 standby 20 preempt
 no shutdown
 exit

! VLAN 30 - HR
interface vlan 30
 description HR-Gateway
 ip address 192.168.30.2 255.255.255.0
 standby 30 ip 192.168.30.1
 standby 30 priority 110
 standby 30 preempt
 no shutdown
 exit

! VLAN 40 - IT
interface vlan 40
 description IT-Gateway
 ip address 192.168.40.2 255.255.255.0
 standby 40 ip 192.168.40.1
 standby 40 priority 110
 standby 40 preempt
 no shutdown
 exit

! Trunk към Core (две паралелни връзки)
interface range GigabitEthernet1/0/1 - 2
 description Trunk-to-Core
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! Trunk към четирите Access switches
interface range GigabitEthernet1/0/21 - 24
 description Trunk-to-Access
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

end
copy running-config startup-config
```

### 3.2 Distribution-SW2 (SECONDARY за HSRP)

```cisco
enable
configure terminal
hostname Dist-SW2
no ip domain-lookup
enable secret class123

line console 0
 password cisco123
 login
 logging synchronous
 exit

! КРИТИЧНО: Enable IP routing!
ip routing

! VLAN-и
vlan 10
 name Sales
vlan 20
 name Engineering
vlan 30
 name HR
vlan 40
 name IT
vlan 99
 name Management
 exit

! STP: Rapid PVST+; Dist-SW2 е след Core и Dist-SW1
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,99 priority 16384

! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
! MANAGEMENT VLAN 99 + HSRP (gateway за комутаторите)
! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
interface vlan 99
 description Management-Gateway
 ip address 10.99.99.12 255.255.255.0
 standby 99 ip 10.99.99.1
 standby 99 priority 100
 standby 99 preempt
 no shutdown
 exit

! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
! INTER-VLAN ROUTING - SVI за всеки VLAN + HSRP
! Тук priority е 100 (по-нисък от Dist-SW1)
! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

! VLAN 10 - Sales
interface vlan 10
 description Sales-Gateway
 ip address 192.168.10.3 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 100
 standby 10 preempt
 no shutdown
 exit

! VLAN 20 - Engineering
interface vlan 20
 description Engineering-Gateway
 ip address 192.168.20.3 255.255.255.0
 standby 20 ip 192.168.20.1
 standby 20 priority 100
 standby 20 preempt
 no shutdown
 exit

! VLAN 30 - HR
interface vlan 30
 description HR-Gateway
 ip address 192.168.30.3 255.255.255.0
 standby 30 ip 192.168.30.1
 standby 30 priority 100
 standby 30 preempt
 no shutdown
 exit

! VLAN 40 - IT
interface vlan 40
 description IT-Gateway
 ip address 192.168.40.3 255.255.255.0
 standby 40 ip 192.168.40.1
 standby 40 priority 100
 standby 40 preempt
 no shutdown
 exit

! Trunk към Core (две паралелни връзки)
interface range GigabitEthernet1/0/1 - 2
 description Trunk-to-Core
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! Trunk към четирите Access switches
interface range GigabitEthernet1/0/21 - 24
 description Trunk-to-Access
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

end
copy running-config startup-config
```

**Защо STP priority 8192 на Dist-SW1?** Така всеки Access комутатор избира за Root port връзката към Dist-SW1 (HSRP Active), а връзката към Dist-SW2 остава резервна (Alternate, BLK).

**QoS:** В основната част QoS не се конфигурира. Вижте Задача 3 (теоретична).

---

## СТЪПКА 4: Конфигуриране на Access Switches (Layer 2)

### 4.1 Access-SW1 (VLAN 10 - Sales)

```cisco
enable
configure terminal
hostname Access-SW1
no ip domain-lookup
enable secret class123

line console 0
 password cisco123
 login
 logging synchronous
 exit

! VLAN-и
vlan 10
 name Sales
vlan 99
 name Management
 exit

spanning-tree mode rapid-pvst

! Management IP
interface vlan 99
 ip address 10.99.99.21 255.255.255.0
 no shutdown
 exit

! Default gateway (HSRP виртуален адрес на VLAN 99)
ip default-gateway 10.99.99.1

! Uplinks: Gi0/1 към Dist-SW1, Gi0/2 към Dist-SW2
interface range gigabitEthernet 0/1 - 2
 description Trunk-to-Distribution
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,99
 no shutdown
 exit

! Access портове за PC-та
interface range fastEthernet 0/1 - 2
 description Access-Ports-VLAN10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

! Изключване на неизползваните портове
interface range fastEthernet 0/3 - 24
 shutdown
 exit

end
copy running-config startup-config
```

**Бележка:** Allowed списъкът на Distribution е по-широк (10,20,30,40,99). По trunk-а минават само VLAN-ите, разрешени и от двете страни, затова към Access-SW1 отиват само VLAN 10 и 99.

### 4.2 Access-SW2, SW3, SW4

**Access-SW2 (VLAN 20):**
- `hostname Access-SW2`
- Management IP: `10.99.99.22`
- VLAN 20 вместо 10
- Trunk allowed (Gi0/1 и Gi0/2): `vlan 20,99`
- Access ports: `switchport access vlan 20`

**Access-SW3 (VLAN 30):**
- `hostname Access-SW3`
- Management IP: `10.99.99.23`
- VLAN 30
- Trunk allowed (Gi0/1 и Gi0/2): `vlan 30,99`
- Access ports: `switchport access vlan 30`

**Access-SW4 (VLAN 40):**
- `hostname Access-SW4`
- Management IP: `10.99.99.24`
- VLAN 40
- Trunk allowed (Gi0/1 и Gi0/2): `vlan 40,99`
- Access ports: `switchport access vlan 40`

### 4.3 SSH на всички комутатори

Изпълнете на **всичките 7 комутатора** (Core-SW, Dist-SW1, Dist-SW2, Access-SW1 – SW4):

```cisco
ip domain-name company.local
crypto key generate rsa
How many bits in the modulus [512]: 1024
ip ssh version 2
username admin secret cisco123
line vty 0 15
 transport input ssh
 login local
 exit
end
copy running-config startup-config
```

**Предпоставка:** адрес във VLAN 99 и gateway (Стъпки 2-4). Тестът е в Стъпка 6.5.

**Пароли:** `class123` и `cisco123` са само примерни. В курсовия проект слабите пароли са забранени.

---

## СТЪПКА 5: Конфигуриране на PC-тата

**Конфигурирайте PC-тата статично (gateway е HSRP Virtual IP!):**

| PC | VLAN | IP Address | Subnet Mask | **Gateway (HSRP)** |
|----|------|------------|-------------|-------------------|
| PC1 | 10 | 192.168.10.10 | 255.255.255.0 | **192.168.10.1** |
| PC2 | 10 | 192.168.10.11 | 255.255.255.0 | **192.168.10.1** |
| PC3 | 20 | 192.168.20.10 | 255.255.255.0 | **192.168.20.1** |
| PC4 | 20 | 192.168.20.11 | 255.255.255.0 | **192.168.20.1** |
| PC5 | 30 | 192.168.30.10 | 255.255.255.0 | **192.168.30.1** |
| PC6 | 30 | 192.168.30.11 | 255.255.255.0 | **192.168.30.1** |
| PC7 | 40 | 192.168.40.10 | 255.255.255.0 | **192.168.40.1** |
| PC8 | 40 | 192.168.40.11 | 255.255.255.0 | **192.168.40.1** |

---

## СТЪПКА 6: Тестване на Layer 3 routing

### 6.1 Проверка на routing tables

От Dist-SW1:
```cisco
show ip route

! Очакван резултат (плюс редове L за собствените адреси):
! C 10.99.99.0/24 is directly connected, Vlan99
! C 192.168.10.0/24 is directly connected, Vlan10
! C 192.168.20.0/24 is directly connected, Vlan20
! C 192.168.30.0/24 is directly connected, Vlan30
! C 192.168.40.0/24 is directly connected, Vlan40
```

От Core-SW:
```cisco
show ip route

! Очакван резултат:
! S* 0.0.0.0/0 [1/0] via 10.99.99.1
! C 10.99.99.0/24 is directly connected, Vlan99
```

### 6.2 Тест на inter-VLAN routing

**От PC1 (VLAN 10) ping PC3 (VLAN 20):**
```
C:\> ping 192.168.20.10

! Това ТРЯБВА да работи! (различни VLAN-и!)
```

**От PC1 tracert до PC5:**
```
C:\> tracert 192.168.30.10

! Очакван резултат:
! 1  192.168.10.2   (активният Dist-SW1; някои версии показват виртуалния 192.168.10.1)
! 2  192.168.30.10  (destination)
```

### 6.3 Проверка на HSRP status

От Dist-SW1:
```cisco
show standby brief

! Очакван резултат (съкратен):
! Interface   Grp  Pri P State    Active          Standby         Virtual IP
! Vl10        10   110 P Active   local           192.168.10.3    192.168.10.1
! Vl20        20   110 P Active   local           192.168.20.3    192.168.20.1
! Vl99        99   110 P Active   local           10.99.99.12     10.99.99.1
```

От Dist-SW2:
```cisco
show standby brief

! Очакван резултат (съкратен):
! Vl10        10   100 P Standby  192.168.10.2    local           192.168.10.1
! Vl20        20   100 P Standby  192.168.20.2    local           192.168.20.1
```

### 6.4 Проверка на Spanning Tree

От Core-SW:
```cisco
show spanning-tree summary

! Core-SW е Root за всички VLAN-и
```

От Access-SW1:
```cisco
show spanning-tree vlan 10

! Gi0/1 (към Dist-SW1): Root FWD
! Gi0/2 (към Dist-SW2): Altn BLK  ← резервна връзка
```

### 6.5 Управление през SSH

От PC1:
```
C:\> ssh -l admin 10.99.99.21
C:\> ssh -l admin 10.99.99.10
```

**Очакван резултат:** вход в Access-SW1 и в Core-SW. Ако няма отговор, проверете HSRP група 99 (`show standby brief`), маршрута на Core (`show ip route`) и `ip default-gateway` на Access комутатора.

---

## СТЪПКА 7: Тестване на Redundancy

### 7.1 Отказ на връзка Core–Distribution

```cisco
! На Dist-SW1: коя връзка към Core е активна?
show spanning-tree vlan 99
! Единият от Gi1/0/1 и Gi1/0/2 е Root FWD, другият е Altn BLK

! От PC1:
C:\> ping 10.99.99.10

! На Dist-SW1 изключете активната връзка (напр. Gi1/0/1):
interface gigabitEthernet 1/0/1
 shutdown
 exit

! От PC1 отново:
C:\> ping 10.99.99.10

! Трябва ДА РАБОТИ! Резервната връзка става Root port (RSTP, няколко секунди)

! Възстановете връзката:
interface gigabitEthernet 1/0/1
 no shutdown
```

### 7.2 Отказ на целия Dist-SW1 (HSRP failover)

```cisco
! От PC1:
C:\> ping 192.168.30.10

! На Dist-SW1 (симулира пълен отказ):
reload

! Докато Dist-SW1 се рестартира, от PC1:
C:\> ping 192.168.30.10

! Първите заявки могат да изтекат, после ping-ът ТРЯБВА ДА РАБОТИ:
! RSTP активира Gi0/2 на Access комутаторите, HSRP прави Dist-SW2 Active

! На Dist-SW2:
show standby brief
! Dist-SW2 е Active за всички групи

! На Access-SW1:
show spanning-tree vlan 10
! Gi0/2 (към Dist-SW2) вече е Root FWD
```

**Бележки:**
- `reload` пита за потвърждение (Enter). Ако рестартът е твърде кратък за теста, изключете Dist-SW1 от бутона за захранване в **Physical** и после го включете.
- След връщането на Dist-SW1 `preempt` го прави отново Active.
- Без втората uplink връзка (Gi0/2) PC1 би загубил връзка при отказ на Dist-SW1: HSRP не възстановява прекъснат физически път.

---

## VERIFICATION CHECKLIST

### Layer 3 Routing:
```
☐ ip routing enabled на Core и Distribution
☐ SVI конфигурирани за всички VLAN-и
☐ PC в различни VLAN могат да ping-ват
☐ Routing table показва connected routes
☐ Core има маршрут по подразбиране към 10.99.99.1
☐ Traceroute показва правилния път
```

### Management и SSH:
```
☐ Всички комутатори имат адрес във VLAN 99 и gateway 10.99.99.1
☐ SSH работи към всички 7 комутатора; Telnet е изключен
```

### HSRP Redundancy:
```
☐ HSRP virtual IP конфигуриран за всеки VLAN (включително VLAN 99)
☐ Dist-SW1 е Active (priority 110)
☐ Dist-SW2 е Standby (priority 100)
☐ При отказ на Active - Standby поема ролята
☐ Preempt enabled - възстановяване при връщане
```

### Spanning Tree:
```
☐ rapid-pvst на всички комутатори
☐ Core-SW е Root Bridge (priority 4096), Dist-SW1 е резервен (8192)
☐ На всеки Access комутатор Gi0/1 е Root FWD, Gi0/2 е Altn BLK
☐ При отказ на линк - автоматично unblock
☐ Convergence: няколко секунди при RSTP (до 50 секунди при класически STP)
```

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ

### Проблем 1: Портовете на 3650 са червени, CLI не отговаря

**Причина:** Няма захранващ модул
**Решение:** Physical → плъзнете **AC-POWER-SUPPLY** в слота за захранване → включете устройството

### Проблем 2: PC не могат да ping между VLAN-и

**Причина:** `ip routing` не е enabled на Distribution
**Решение:**
```cisco
! На Dist-SW1 и Dist-SW2:
configure terminal
ip routing
end
```

### Проблем 3: "% Invalid input" при `interface gigabitEthernet 0/1` на 3650

**Причина:** На 3650 портовете са Gi1/0/x
**Решение:** Проверете имената с `show ip interface brief` и използвайте Gi1/0/1 – Gi1/0/24

### Проблем 4: "% Incomplete command" при SVI creation

**Причина:** Switch не е multilayer или не е в L3 mode
**Решение:**
```cisco
! Проверка на модела:
show version

! Трябва да е 3560/3650/3750 (multilayer)
! В PT, използвайте Multilayer Switch, не обикновен 2960!
```

### Проблем 5: HSRP не работи (двата комутатора са Active или няма Standby)

**Причина:** Различен номер на група или виртуален IP, VLAN-ът липсва в trunk-а между Distribution комутаторите (през Core) или SVI е down
**Решение:**
```cisco
show standby brief
show interfaces trunk
show ip interface brief
```
Tracking (`standby track`) е допълнителен механизъм и не е нужен за основната работа на HSRP.

### Проблем 6: Root Bridge не е Core-SW

**Причина:** Не е зададен priority; Root става комутаторът с най-нисък MAC адрес
**Последица:** Неоптимален път на трафика (не loop – STP продължава да предотвратява цикли)
**Решение:**
```cisco
! На Core-SW:
spanning-tree vlan 10,20,30,40,99 priority 4096

! Проверка:
show spanning-tree root
```

### Проблем 7: Комутатор не се достига по SSH от PC

**Причина:** Липсва `ip default-gateway 10.99.99.1` (Access), маршрутът на Core или HSRP група 99
**Решение:** `show standby brief` на Dist-SW1 (група 99), `show ip route` на Core-SW, ping 10.99.99.1 от комутатора

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: OSPF между L3 комутаторите

В основната топология OSPF не е нужен: Dist-SW1 и Dist-SW2 имат директно свързани всички VLAN-и, а Core използва статичен маршрут. Задачата показва как Core научава потребителските мрежи динамично.

```cisco
! Core-SW:
interface loopback 0
 ip address 10.0.0.1 255.255.255.255
router ospf 1
 router-id 1.1.1.1
 network 10.99.99.0 0.0.0.255 area 0
 network 10.0.0.1 0.0.0.0 area 0

! Dist-SW1 (router-id 2.2.2.2) и Dist-SW2 (router-id 3.3.3.3):
router ospf 1
 router-id 2.2.2.2
 network 192.168.0.0 0.0.255.255 area 0
 network 10.99.99.0 0.0.0.255 area 0
```

**Проверка:** `show ip ospf neighbor`; на Core-SW `show ip route ospf` показва маршрути **O** към 192.168.10.0 – 192.168.40.0. След това статичният маршрут на Core може да се премахне (`no ip route 0.0.0.0 0.0.0.0 10.99.99.1`).

### Задача 2: ACL за Security
Ограничете достъпа между отделите:
```cisco
! Пример: HR не може да достъпва Sales
ip access-list extended BLOCK-HR-TO-SALES
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 permit ip any any

interface vlan 30
 ip access-group BLOCK-HR-TO-SALES in
```

**Важно:** Приложете ACL на **Dist-SW1 и Dist-SW2**. Иначе след HSRP failover филтрирането изчезва.

### Задача 3: QoS (теоретична, BONUS)

3650 използва MQC (class-map/policy-map) на IOS XE; командите `mls qos` от 3560/3750 не се прилагат. Примерът показва синтаксиса за обсъждане – поддръжката му в Packet Tracer 8.2.2 трябва да се провери, а на Catalyst комутаторите част от опциите (напр. `fair-queue`) не се поддържат.

```cisco
class-map match-any VOICE
 match ip dscp ef

class-map match-any VIDEO
 match ip dscp af41

policy-map QOS-POLICY
 class VOICE
  priority percent 30
 class VIDEO
  bandwidth percent 40
 class class-default
  fair-queue

interface vlan 10
 service-policy output QOS-POLICY
```

---

## КЛЮЧОВИ КОНЦЕПЦИИ

### Inter-VLAN Routing (Router-on-a-stick vs SVI):
- ✅ **SVI (Switch Virtual Interface)** - използвано в този лаб
- Създава се virtual interface за всеки VLAN
- По-ефективно от router-on-a-stick
- Поддържа по-висока throughput

### HSRP (Hot Standby Router Protocol):
- Осигурява gateway redundancy
- Два (или повече) routers споделят Virtual IP
- При отказ на Active → Standby става Active
- Transparent за крайните устройства

### Spanning Tree Protocol (STP):
- Превенция на L2 loops
- Root Bridge е референтна точка
- Портове в Blocking state за redundancy
- RSTP (Rapid STP) - по-бърз convergence (няколко секунди, ако `rapid-pvst` е на всички комутатори)

---

## СЛЕДВАЩИ СТЪПКИ

Сега можете да:
- Изграждате правилна йерархична топология с L3 routing
- Конфигурирате inter-VLAN routing със SVI
- Управлявате комутаторите през Management VLAN и SSH
- Прилагате HSRP за gateway redundancy
- Тествате fault tolerance и failover

**Следващ лаб:** Lab 8 - NAT Configuration with WAN connectivity

---

**Важно:** Запазете .pkt файла като `Lab7_YourName.pkt`

---

---

**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>