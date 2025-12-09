# LAB 7: Йерархична топология с Layer 3 routing (ОБНОВЕН)

**Продължителност:** 120-150 минути  
**Цел:** Изграждане на правилна йерархична мрежа (Access/Distribution/Core) с Layer 3 routing, QoS и redundancy

---

## КРИТИЧНИ ПРОМЕНИ СПРЯМО ОРИГИНАЛНАТА ВЕРСИЯ

### Основни подобрения:
1. ✅ **Layer 3 switches** на Core и Distribution слоевете
2. ✅ **Inter-VLAN routing** - PC-та в различни VLAN могат да комуникират
3. ✅ **HSRP redundancy** между Distribution switches
4. ✅ **Redundant links** между Core и Distribution
5. ✅ **QoS policies** на Distribution Layer
6. ✅ **Spanning Tree оптимизация**

---

## ЦЕЛИ НА УПРАЖНЕНИЕТО

След завършване на този лаб ще можете да:
- ✅ Създавате правилна йерархична топология с L3 routing
- ✅ Конфигурирате inter-VLAN routing с SVI (Switch Virtual Interface)
- ✅ Прилагате HSRP за gateway redundancy
- ✅ Настройвате QoS policies на Distribution Layer
- ✅ Изграждате redundant links с Spanning Tree Protocol
- ✅ Тествате fault tolerance при отказ на връзка

---

## ТЕОРЕТИЧНА ОСНОВА

### Какво е различно от предишната версия?

**ПРЕДИ (Lab 7 v1):**
- Само Layer 2 switches (2960-24TT)
- Няма routing между VLAN-и
- Няма redundancy
- Няма QoS

**СЕГА (Lab 7 v2):**
- Core и Distribution са **Layer 3 Multilayer Switches**
- Пълна inter-VLAN routing функционалност
- HSRP за gateway redundancy
- QoS на Distribution Layer
- Spanning Tree оптимизация

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
                  10.99.99.10/24
                  Root Bridge
         ┌────────────┴────────────┐
         │ (Gi0/1,Gi0/2)    (Gi0/3,Gi0/4) │
         │  Redundant      Redundant   │
    [Dist-SW1 L3]              [Dist-SW2 L3]
(Multilayer Switch 3650)  (Multilayer Switch 3650)
   10.99.99.11/24            10.99.99.12/24
   HSRP Virtual IPs:
   - VLAN 10: 192.168.10.1
   - VLAN 20: 192.168.20.1
   - VLAN 30: 192.168.30.1
   - VLAN 40: 192.168.40.1
         │                        │
    ┌────┴──────┐          ┌──────┴──────┐
    │           │          │             │
[Access-SW1] [Access-SW2] [Access-SW3] [Access-SW4]
  (2960)      (2960)       (2960)        (2960)
10.99.99.21  10.99.99.22  10.99.99.23   10.99.99.24
    │           │            │             │
  VLAN 10    VLAN 20      VLAN 30       VLAN 40
  Sales      Engineer      HR            IT
    │           │            │             │
  PC1-PC2    PC3-PC4      PC5-PC6       PC7-PC8
```

**VLAN дизайн:**
- VLAN 10: Sales (192.168.10.0/24) - Gateway: .1 (HSRP)
- VLAN 20: Engineering (192.168.20.0/24) - Gateway: .1 (HSRP)
- VLAN 30: HR (192.168.30.0/24) - Gateway: .1 (HSRP)
- VLAN 40: IT (192.168.40.0/24) - Gateway: .1 (HSRP)
- **VLAN 99: Management** (10.99.99.0/24) - Gateway: Core-SW

---

## СТЪПКА 1: Изграждане на топологията

### 1.1 Добави устройства

В Packet Tracer добави:
- **1x Multilayer Switch 3650-24PS** (Core-SW)
- **2x Multilayer Switch 3650-24PS** (Dist-SW1, Dist-SW2)
- **4x 2960-24TT Switch** (Access-SW1 до Access-SW4)
- **8x PC** (по 2 на всеки Access switch)

### 1.2 Кабелиране

**Core към Distribution (REDUNDANT):**
```
Core-SW Gi0/1 ↔ Dist-SW1 Gi0/1
Core-SW Gi0/2 ↔ Dist-SW1 Gi0/2  (Redundant link)
Core-SW Gi0/3 ↔ Dist-SW2 Gi0/1
Core-SW Gi0/4 ↔ Dist-SW2 Gi0/2  (Redundant link)
```

**Distribution към Access:**
```
Dist-SW1 Fa0/23 ↔ Access-SW1 Gi0/1
Dist-SW1 Fa0/24 ↔ Access-SW2 Gi0/1
Dist-SW2 Fa0/23 ↔ Access-SW3 Gi0/1
Dist-SW2 Fa0/24 ↔ Access-SW4 Gi0/1
```

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

! VLAN-и (само за awareness, routing става със SVI)
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

! Spanning Tree - Root Bridge за всички VLAN-и
spanning-tree mode rapid-pvst
spanning-tree vlan 1-4094 priority 4096

! Trunk портове към Distribution switches
interface range gigabitEthernet 0/1-4
 description Trunk-to-Distribution
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! Запази конфигурацията
end
copy running-config startup-config
```

**Важни бележки:**
- `ip routing` - КРИТИЧНО за L3 functionality
- `spanning-tree priority 4096` - Core става Root Bridge
- `switchport trunk encapsulation dot1q` - нужно за multilayer switches

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

! Създай VLAN-и
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

! SVI за Management
interface vlan 99
 description Management-Interface
 ip address 10.99.99.11 255.255.255.0
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

! Trunk към Core (redundant)
interface range gigabitEthernet 0/1-2
 description Trunk-to-Core
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! Trunk към Access switches
interface range fastEthernet 0/23-24
 description Trunk-to-Access
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
! QoS CONFIGURATION (критично за Distribution Layer!)
! ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

! Enable QoS globally
mls qos

! Trust DSCP on trunk ports
interface range gigabitEthernet 0/1-2
 mls qos trust dscp
 exit

interface range fastEthernet 0/23-24
 mls qos trust dscp
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

! Създай VLAN-и
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

! SVI за Management
interface vlan 99
 description Management-Interface
 ip address 10.99.99.12 255.255.255.0
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

! Trunk към Core (redundant)
interface range gigabitEthernet 0/1-2
 description Trunk-to-Core
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! Trunk към Access switches
interface range fastEthernet 0/23-24
 description Trunk-to-Access
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! QoS enable
mls qos

interface range gigabitEthernet 0/1-2
 mls qos trust dscp
 exit

interface range fastEthernet 0/23-24
 mls qos trust dscp
 exit

end
copy running-config startup-config
```

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

! Създай VLAN-и
vlan 10
 name Sales
vlan 99
 name Management
 exit

! Management IP
interface vlan 99
 ip address 10.99.99.21 255.255.255.0
 no shutdown
 exit

! Default gateway (към Core)
ip default-gateway 10.99.99.10

! Trunk към Distribution
interface gigabitEthernet 0/1
 description Trunk-to-Dist-SW1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,99
 no shutdown
 exit

! Access портове за PC-та
interface range fastEthernet 0/1-2
 description Access-Ports-VLAN10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

! Изключи неизползваните портове
interface range fastEthernet 0/3-24
 shutdown
 exit

end
copy running-config startup-config
```

### 4.2 Access-SW2, SW3, SW4

**Access-SW2 (VLAN 20):**
- `hostname Access-SW2`
- Management IP: `10.99.99.22`
- VLAN 20 вместо 10
- Trunk allowed: `vlan 20,99`
- Access ports: `switchport access vlan 20`

**Access-SW3 (VLAN 30):**
- `hostname Access-SW3`
- Management IP: `10.99.99.23`
- VLAN 30
- Trunk allowed: `vlan 30,99`
- Trunk от Dist-SW2
- Access ports: `switchport access vlan 30`

**Access-SW4 (VLAN 40):**
- `hostname Access-SW4`
- Management IP: `10.99.99.24`
- VLAN 40
- Trunk allowed: `vlan 40,99`
- Trunk от Dist-SW2
- Access ports: `switchport access vlan 40`

---

## СТЪПКА 5: Конфигуриране на PC-тата

**Конфигурирай PC-тата статично (gateway е HSRP Virtual IP!):**

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

### 6.1 Провери routing tables

От Dist-SW1:
```cisco
show ip route

! Очакван резултат:
! C 192.168.10.0/24 is directly connected, Vlan10
! C 192.168.20.0/24 is directly connected, Vlan20
! C 192.168.30.0/24 is directly connected, Vlan30
! C 192.168.40.0/24 is directly connected, Vlan40
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
! 1  192.168.10.1 (gateway - HSRP)
! 2  192.168.30.10 (destination)
```

### 6.3 Провери HSRP status

От Dist-SW1:
```cisco
show standby brief

! Очакван резултат:
! Vlan10  10  P   192.168.10.1    Active   local
! Vlan20  20  P   192.168.20.1    Active   local
```

От Dist-SW2:
```cisco
show standby brief

! Очакван резултат:
! Vlan10  10  P   192.168.10.1    Standby  192.168.10.2
! Vlan20  20  P   192.168.20.1    Standby  192.168.20.2
```

### 6.4 Провери Spanning Tree

От Core-SW:
```cisco
show spanning-tree summary

! Виж че Core-SW е Root за всички VLAN-и
```

---

## СТЪПКА 7: Тестване на Redundancy

### 7.1 Симулирай отказ на линк

**Тест 1: Disable trunk между Core и Dist-SW1**
```cisco
! На Core-SW:
interface gigabitEthernet 0/1
 shutdown
 exit

! От PC1 ping PC3:
C:\> ping 192.168.20.10

! Трябва ДА РАБОТИ! (автоматично failover)
```

**Тест 2: Disable primary Distribution switch**
```cisco
! На Dist-SW1 (симулира пълен отказ):
reload

! От PC1:
C:\> ping 192.168.20.10

! Трябва ДА РАБОТИ! (HSRP failover към Dist-SW2)

! Провери HSRP:
! На Dist-SW2:
show standby brief
! Сега Dist-SW2 трябва да е Active!
```

---

## VERIFICATION CHECKLIST

### Layer 3 Routing:
```
☐ `ip routing` enabled на Core и Distribution
☐ SVI конфигурирани за всички VLAN-и
☐ PC в различни VLAN могат да ping-ват
☐ Routing table показва connected routes
☐ Traceroute показва правилния път
```

### HSRP Redundancy:
```
☐ HSRP virtual IP конфигуриран за всеки VLAN
☐ Dist-SW1 е Active (priority 110)
☐ Dist-SW2 е Standby (priority 100)
☐ При отказ на Active - Standby поема ролята
☐ Preempt enabled - възстановяване при връщане
```

### Spanning Tree:
```
☐ Core-SW е Root Bridge (priority 4096)
☐ Redundant links са blocked от STP
☐ При отказ на линк - автоматично unblock
☐ Convergence време < 50 секунди (RSTP)
```

### QoS:
```
☐ `mls qos` enabled на Distribution switches
☐ Trunk портове trust DSCP
☐ (Optional) Class maps и policy maps конфигурирани
```

---

## РАЗЛИКИ СПРЯМО ОРИГИНАЛНАТА ВЕРСИЯ

| Аспект | Оригинална версия | Обновена версия |
|--------|-------------------|-----------------|
| **Оборудване Core** | 2960 L2 | **3650 L3** |
| **Оборудване Distribution** | 2960 L2 | **3650 L3** |
| **Routing** | ✗ Няма | ✅ Inter-VLAN routing |
| **Gateway** | ✗ Не работи | ✅ HSRP Virtual IP |
| **Redundancy** | ✗ Липсва | ✅ Redundant links + HSRP |
| **QoS** | ✗ Липсва | ✅ Enabled на Distribution |
| **Spanning Tree** | Default | ✅ Оптимизиран (Root на Core) |
| **PC connectivity** | Само в един VLAN | **Между всички VLAN** |

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ

### Проблем 1: PC не могат да ping между VLAN-и

**Причина:** `ip routing` не е enabled на Distribution
**Решение:**
```cisco
! На Dist-SW1 и Dist-SW2:
configure terminal
ip routing
end
```

### Проблем 2: "% Incomplete command" при SVI creation

**Причина:** Switch не е multilayer или не е в L3 mode
**Решение:**
```cisco
! Провери модела:
show version

! Трябва да е 3560/3650/3750 (multilayer)
! В PT, използвай Multilayer Switch, не обикновен 2960!
```

### Проблем 3: HSRP не работи

**Причина:** HSRP track на интерфейс не е конфигуриран
**Решение:**
```cisco
! Провери HSRP:
show standby

! Виж дали virtual IP е active
! Виж priority и preempt
```

### Проблем 4: Spanning Tree loop

**Причина:** Core не е Root Bridge
**Решение:**
```cisco
! На Core-SW:
spanning-tree vlan 1-4094 priority 4096

! Провери:
show spanning-tree root
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: OSPF Routing Protocol
Конфигурирай OSPF между Core и Distribution за динамично routing:
```cisco
! На всички L3 switches:
router ospf 1
 router-id X.X.X.X
 network 192.168.0.0 0.0.255.255 area 0
 network 10.99.99.0 0.0.0.255 area 0
```

### Задача 2: ACL за Security
Ограничи достъпа между отделения:
```cisco
! Пример: HR не може да достъпва Sales
ip access-list extended BLOCK-HR-TO-SALES
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 permit ip any any

interface vlan 30
 ip access-group BLOCK-HR-TO-SALES in
```

### Задача 3: Advanced QoS
Създай class maps и policy maps за приоритизиране:
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
- RSTP (Rapid STP) - по-бърз convergence

---

## СЛЕДВАЩИ СТЪПКИ

Сега знаеш как да:
- Изграждаш правилна йерархична топология с L3 routing
- Конфигурираш inter-VLAN routing със SVI
- Прилагаш HSRP за gateway redundancy
- Тестваш fault tolerance и failover

**Следващ лаб:** Lab 8 - NAT Configuration with WAN connectivity

---

**Важно:** Запази .pkt файла като `Lab7_Hierarchical_L3_Routing.pkt`

---

---

**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>