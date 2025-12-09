# LAB 9: Интегриран мрежов дизайн (3+ subnets + топология диаграми)

**Продължителност:** 120-150 минути  
**Цел:** Интегриране на всички знания от Lab 1-8 в един комплексен проект с множество subnets и професионална документация
**Prerequisite:** Завършени Lab 1-8 (всички концепции се интегрират тук)

---

## ЦЕЛИ НА УПРАЖНЕНИЕТО

След завършване на този лаб ще можете да:
- ✅ Проектирате мрежа с 3+ subnet-а едновременно
- ✅ Интегрирате VLAN, DHCP, DNS, ACL, NAT в една топология
- ✅ Създавате професионални логически и физически топология диаграми
- ✅ Планирате IP addressing scheme с VLSM
- ✅ Документирате мрежата по стандарт
- ✅ Прилагате всички best practices

---

## ТЕОРЕТИЧНА ОСНОВА

### Интегриран мрежов дизайн

**Какво означава "интегриран дизайн"?**
- Комбинация на всички мрежови технологии
- Реален work scenario
- 3+ subnet-а работещи заедно
- Пълна функционалност (routing, services, security)

### Дизайн методология:

```
1. ПЛАНИРАНЕ
   ├── Изисквания (брой users, услуги)
   ├── IP addressing scheme (subnetting plan)
   └── VLAN дизайн

2. ИЗГРАЖДАНЕ
   ├── Физическа топология
   ├── Конфигурация на устройства
   └── Тестване на connectivity

3. УСЛУГИ
   ├── DHCP
   ├── DNS
   └── NAT

4. СИГУРНОСТ
   ├── ACL
   ├── SSH
   └── Port Security

5. ДОКУМЕНТАЦИЯ
   ├── Логическа диаграма
   ├── Физическа диаграма
   └── Конфигурационни таблици
```

---

## СЦЕНАРИЙ: Малка IT компания "TechStart"

### Изисквания:

**Организация:**
- 3 отдела: Sales, Engineering, Management
- 25 служители общо
- 2 сървъра (Web, DNS)
- 1 гостна мрежа (Guest WiFi)

**Нужди:**
- Разделение на отделите (VLAN-и)
- DHCP за автоматично IP адресиране
- DNS за локални имена
- Интернет достъп (NAT)
- Сигурност между отделите (ACL)

> **Забележка за IP схемата:** В този лаб използваме **10.10.x.0/24** вместо 192.168.x.0/24 от предишните лабове. Това е умишлено - симулираме нов проект за компания "TechStart" с различно IP адресно пространство. В реалния свят всяка организация има уникална IP схема.
```

**Subnet план:**
- Management VLAN: 10.10.99.0/24
- VLAN 10 - Sales: 10.10.10.0/24 (15 users)
- VLAN 20 - Engineering: 10.10.20.0/24 (8 users)
- VLAN 30 - Management: 10.10.30.0/24 (5 users)
- VLAN 40 - Guest: 10.10.40.0/24 (10 guests)
- VLAN 50 - Servers: 10.10.50.0/24 (2 servers)
- WAN Link: 209.165.200.0/30 (към ISP)

---

## ЧАСТ 1: IP ADDRESSING PLAN

### СТЪПКА 1: Създаване на IP addressing таблица

| Subnet | Network | First IP | Last IP | Broadcast | Gateway | VLAN | Hosts |
|--------|---------|----------|---------|-----------|---------|------|-------|
| Management | 10.10.99.0/24 | 10.10.99.1 | 10.10.99.254 | 10.10.99.255 | N/A | 99 | Switch management |
| Sales | 10.10.10.0/24 | 10.10.10.1 | 10.10.10.254 | 10.10.10.255 | 10.10.10.1 | 10 | 15 |
| Engineering | 10.10.20.0/24 | 10.10.20.1 | 10.10.20.254 | 10.10.20.255 | 10.10.20.1 | 20 | 8 |
| Management | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.254 | 10.10.30.255 | 10.10.30.1 | 30 | 5 |
| Guest | 10.10.40.0/24 | 10.10.40.1 | 10.10.40.254 | 10.10.40.255 | 10.10.40.1 | 40 | 10 |
| Servers | 10.10.50.0/24 | 10.10.50.1 | 10.10.50.254 | 10.10.50.255 | 10.10.50.1 | 50 | 2 |
| WAN Link | 209.165.200.0/30 | 209.165.200.1 | 209.165.200.2 | 209.165.200.3 | N/A | N/A | ISP link |

**DHCP Pools:**
- VLAN 10: 10.10.10.10 - 10.10.10.50
- VLAN 20: 10.10.20.10 - 10.10.20.30
- VLAN 30: 10.10.30.10 - 10.10.30.20
- VLAN 40: 10.10.40.10 - 10.10.40.100

**Excluded ranges (reserved for static):**
- 10.10.10.1-9 (gateway, printers)
- 10.10.20.1-9
- 10.10.30.1-9
- 10.10.40.1-9
- 10.10.50.1-10 (servers)

---

## ЧАСТ 2: ЛОГИЧЕСКА ТОПОЛОГИЯ

### СТЪПКА 2: Създаване на логическа диаграма

**Как да направиш диаграма в Packet Tracer:**

1. В Packet Tracer, отвори твоята топология
2. Меню **File** → **Export** → **Export to PNG**
3. Запази като `Logical_Topology.png`

**Алтернатива - Draw.io / Lucidchart:**

1. Отвори [draw.io](https://app.diagrams.net/)
2. Избери **Network Diagram** template
3. Добави устройства:
   - 2x Router (R1 и ISP-Router)
   - Switches (иконка Switch)
   - PCs grouped by VLAN
   - Servers
   - Internet symbol

**Какво да включиш в логическата диаграма:**
- ✅ Всички устройства с hostname
- ✅ VLAN-и с names и IDs
- ✅ IP адреси на gateway-и
- ✅ Subnet notations (10.10.10.0/24)
- ✅ Trunk links маркирани
- ✅ WAN link маркиран
- ✅ Легенда (Legend)

**Пример логическа диаграма структура:**

```
         INTERNET (Simulated)
      ┌──────────────────────┐
      │   ISP-Router 2911    │
      │ Lo0: 8.8.8.8/32      │ Симулиран Google DNS
      │ Se0/0/0: 209.165.200.1/30
      └──────────┬───────────┘
                 │ Serial DCE (64000)
                 │ WAN Link
                 │ 209.165.200.0/30
                 │
      ┌──────────▼───────────┐
      │  R1-TechStart 2911   │
      │ Se0/0/0: 209.165.200.2/30 (NAT Outside)
      │ Gi0/0: Trunk (NAT Inside)
      │ Gateway + DHCP + ACL │
      └──────────┬───────────┘
                 │ Gi0/0 Trunk
                 │ VLAN 10,20,30,40,50,99
                 │ Subinterfaces: 10.10.x.1
      ┌──────────▼───────────┐
      │   SW1-Core 2960      │
      │ VLAN database        │
      │ Management: 10.10.99.10
      └────┬────────────┬────┘
           │ Trunk      │ Trunk
           │ VLANs:     │ VLANs:
           │ 10,20,99   │ 30,40,50,99
    ┌──────▼─────┐ ┌───▼──────┐
    │  SW2-Access│ │SW3-Access│
    │   2960     │ │  2960    │
    │10.10.99.20 │ │10.10.99.30
    └─┬──┬───────┘ └──┬──┬──┬─┘
      │  │            │  │  │
   VLAN VLAN       VLAN VLAN VLAN
    10  20          30  40  50
   Sales Eng      Mgmt Guest Servers
   (15) (8)        (5) (10)  (2)
```

**Легенда:**
- 🔵 Trunk Link (802.1Q)
- 🟢 Access Port (VLAN assigned)
- 🔴 WAN Link (Serial)
- ⚪ Management VLAN 99

---

## ЧАСТ 3: ФИЗИЧЕСКА ТОПОЛОГИЯ

### СТЪПКА 3: Изграждане в Packet Tracer

**Устройства:**
- 1x Router 2911 (ISP-Router) - нов!
- 1x Router 2911 (R1)
- 3x Switch 2960-24TT (SW1-Core, SW2-Access, SW3-Access)
- 20x PC за потребители
- 2x Server (Web, DNS)

**Кабелиране:**

```
# WAN connection (ПРОМЯНА: сега към ISP-Router)
ISP-Router Se0/0/0 (DCE) ↔ R1 Se0/0/0 (DTE)
Кабел: Serial DCE

# Router to Core Switch
R1 Gi0/0 ↔ SW1 Gi0/1 (Trunk)

# Core to Access Switches
SW1 Gi0/2 ↔ SW2 Gi0/1 (Trunk)
SW1 Fa0/24 ↔ SW3 Gi0/1 (Trunk)

# Access Switches to End Devices
SW2 Fa0/1-5 → PCs (VLAN 10 - Sales)
SW2 Fa0/6-10 → PCs (VLAN 20 - Engineering)
SW3 Fa0/1-5 → PCs (VLAN 30 - Management)
SW3 Fa0/6-10 → PCs (VLAN 40 - Guest)
SW3 Fa0/11-12 → Servers (VLAN 50)
```

---

## ЧАСТ 4: КОНФИГУРАЦИЯ

### СТЪПКА 4A: ISP Router конфигурация (НОВ!)

```cisco
enable
configure terminal
hostname ISP-Router
no ip domain-lookup
enable secret class123

! Интерфейс към клиентския рутер (R1-TechStart)
interface serial 0/0/0
 description WAN Link to Customer R1-TechStart
 ip address 209.165.200.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! Loopback интерфейс симулиращ Google DNS
interface loopback 0
 description Simulated Internet - Google DNS 8.8.8.8
 ip address 8.8.8.8 255.255.255.255
 no shutdown
 exit

! Loopback симулиращ общ интернет адрес
interface loopback 1
 description Simulated Internet Gateway
 ip address 1.1.1.1 255.255.255.255
 no shutdown
 exit

! Static route за връщане на трафик към клиентската мрежа
ip route 10.10.0.0 255.255.0.0 209.165.200.2

! Конзолна парола
line console 0
 password cisco
 login
 exit

! SSH Configuration
ip domain-name isp.example.com
crypto key generate rsa
1024
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exit

username admin privilege 15 secret admin123

end
copy running-config startup-config
```

---

### СТЪПКА 4B: Router конфигурация (R1)

```cisco
enable
configure terminal
hostname R1-TechStart
no ip domain-lookup
enable secret class123

! Outside interface (to ISP)
interface serial 0/0/0
 description WAN to ISP-Router
 ip address 209.165.200.2 255.255.255.252
 ip nat outside
 no shutdown
 exit

! Inside interface (to internal network) - Trunk with subinterfaces
interface gigabitEthernet 0/0
 description Trunk to SW1-Core
 no shutdown
 exit

! Subinterfaces for each VLAN (Router-on-a-Stick)
interface gigabitEthernet 0/0.10
 description Sales VLAN
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
 ip nat inside
 exit

interface gigabitEthernet 0/0.20
 description Engineering VLAN
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
 ip nat inside
 exit

interface gigabitEthernet 0/0.30
 description Management VLAN
 encapsulation dot1Q 30
 ip address 10.10.30.1 255.255.255.0
 ip nat inside
 exit

interface gigabitEthernet 0/0.40
 description Guest VLAN
 encapsulation dot1Q 40
 ip address 10.10.40.1 255.255.255.0
 ip nat inside
 exit

interface gigabitEthernet 0/0.50
 description Server VLAN
 encapsulation dot1Q 50
 ip address 10.10.50.1 255.255.255.0
 ip nat inside
 exit

! DHCP Pools for VLANs
ip dhcp excluded-address 10.10.10.1 10.10.10.9
ip dhcp excluded-address 10.10.20.1 10.10.20.9
ip dhcp excluded-address 10.10.30.1 10.10.30.9
ip dhcp excluded-address 10.10.40.1 10.10.40.9

ip dhcp pool SALES_POOL
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server 10.10.50.10
 domain-name techstart.local
 lease 7
 exit

ip dhcp pool ENGINEERING_POOL
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
 dns-server 10.10.50.10
 domain-name techstart.local
 lease 7
 exit

ip dhcp pool MANAGEMENT_POOL
 network 10.10.30.0 255.255.255.0
 default-router 10.10.30.1
 dns-server 10.10.50.10
 domain-name techstart.local
 lease 30
 exit

ip dhcp pool GUEST_POOL
 network 10.10.40.0 255.255.255.0
 default-router 10.10.40.1
 dns-server 8.8.8.8
 lease 0 1 0
 exit

! NAT - PAT Overload
access-list 1 permit 10.10.0.0 0.0.255.255
ip nat inside source list 1 interface serial 0/0/0 overload

! Default route to Internet (към ISP-Router)
ip route 0.0.0.0 0.0.0.0 209.165.200.1

! ACL - Guest isolation (block Guest to internal networks)
ip access-list extended GUEST_FILTER
 remark Block Guest access to internal networks
 deny ip 10.10.40.0 0.0.0.255 10.10.10.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.20.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.30.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.50.0 0.0.0.255
 permit ip any any
 exit

interface gigabitEthernet 0/0.40
 ip access-group GUEST_FILTER in
 exit

! SSH Configuration
ip domain-name techstart.local
crypto key generate rsa
1024
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exit

username admin privilege 15 secret admin123

end
copy running-config startup-config
```

---

### СТЪПКА 5: Core Switch (SW1) - КОРИГИРАНА ВЕРСИЯ

```cisco
enable
configure terminal
hostname SW1-Core
no ip domain-lookup
enable secret class123

! VLAN Creation (ДОБАВЕН VLAN 50!)
vlan 10
 name Sales
 exit
vlan 20
 name Engineering
 exit
vlan 30
 name Management_Users
 exit
vlan 40
 name Guest
 exit
vlan 50
 name Servers
 exit
vlan 99
 name Switch_Management
 exit

! Management IP
interface vlan 99
 description Management Interface
 ip address 10.10.99.10 255.255.255.0
 no shutdown
 exit

ip default-gateway 10.10.99.1

! Trunk to Router (ОБНОВЕН: добавен VLAN 50)
interface gigabitEthernet 0/1
 description Trunk to R1-TechStart
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Trunk to SW2-Access (ОБНОВЕН: добавен VLAN 50)
interface gigabitEthernet 0/2
 description Trunk to SW2-Access
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Trunk to SW3-Access (ОБНОВЕН: добавен VLAN 50)
interface fastEthernet 0/24
 description Trunk to SW3-Access
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Shutdown unused ports
interface range fastEthernet 0/1-23
 shutdown
 exit

interface range gigabitEthernet 0/3-24
 shutdown
 exit

! Spanning Tree - Set as root bridge
spanning-tree mode pvst
spanning-tree vlan 10,20,30,40,50,99 priority 4096

! SSH Configuration
ip domain-name techstart.local
crypto key generate rsa
1024
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exit

username admin privilege 15 secret admin123

end
copy running-config startup-config
```

---

### СТЪПКА 6: Access Switches (SW2 & SW3)

**SW2-Access (Sales + Engineering):**

```cisco
enable
configure terminal
hostname SW2-Access
no ip domain-lookup
enable secret class123

! VLAN Creation
vlan 10
 name Sales
 exit
vlan 20
 name Engineering
 exit
vlan 99
 name Management
 exit

! Management IP
interface vlan 99
 ip address 10.10.99.20 255.255.255.0
 no shutdown
 exit

ip default-gateway 10.10.99.1

! Trunk to Core
interface gigabitEthernet 0/1
 description Trunk to SW1-Core
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Access ports for Sales
interface range fastEthernet 0/1-5
 description Sales Department
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 no shutdown
 exit

! Access ports for Engineering
interface range fastEthernet 0/6-10
 description Engineering Department
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 no shutdown
 exit

! Shutdown unused ports
interface range fastEthernet 0/11-24
 shutdown
 exit

interface range gigabitEthernet 0/2-24
 shutdown
 exit

! SSH Config
ip domain-name techstart.local
crypto key generate rsa
1024
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exit

username admin privilege 15 secret admin123

end
copy running-config startup-config
```

---

**SW3-Access (Management + Guest + Servers):**

```cisco
enable
configure terminal
hostname SW3-Access
no ip domain-lookup
enable secret class123

! VLAN Creation (ВКЛЮЧЕН VLAN 50!)
vlan 30
 name Management_Users
 exit
vlan 40
 name Guest
 exit
vlan 50
 name Servers
 exit
vlan 99
 name Management
 exit

! Management IP
interface vlan 99
 ip address 10.10.99.30 255.255.255.0
 no shutdown
 exit

ip default-gateway 10.10.99.1

! Trunk to Core (ОБНОВЕН: добавен VLAN 50)
interface gigabitEthernet 0/1
 description Trunk to SW1-Core
 switchport mode trunk
 switchport trunk allowed vlan 30,40,50,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Access ports for Management Users
interface range fastEthernet 0/1-5
 description Management Department
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 no shutdown
 exit

! Access ports for Guest
interface range fastEthernet 0/6-10
 description Guest WiFi
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 no shutdown
 exit

! Access ports for Servers (КРИТИЧНО!)
interface range fastEthernet 0/11-12
 description Server Farm
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
 no shutdown
 exit

! Shutdown unused ports
interface range fastEthernet 0/13-24
 shutdown
 exit

interface range gigabitEthernet 0/2-24
 shutdown
 exit

! SSH Config
ip domain-name techstart.local
crypto key generate rsa
1024
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exit

username admin privilege 15 secret admin123

end
copy running-config startup-config
```

---

### СТЪПКА 7: DNS Server конфигурация

**DNS Server (10.10.50.10):**

В GUI на Server:
1. **Config** tab → **FastEthernet0**
   - IP: `10.10.50.10`
   - Mask: `255.255.255.0`
   - Gateway: `10.10.50.1`

2. **Services** tab → **DNS**
   - Turn **ON**
   - Add A Records:
     ```
     www.techstart.local    → 10.10.50.11 (Web Server)
     mail.techstart.local   → 10.10.50.11
     ftp.techstart.local    → 10.10.50.11
     intranet.techstart.local → 10.10.50.11
     router.techstart.local → 10.10.10.1
     isp.example.com        → 209.165.200.1
     ```
   - Add CNAME Records:
     ```
     portal → www.techstart.local
     webmail → mail.techstart.local
     ```

---

### СТЪПКА 8: Web Server конфигурация

**Web Server (10.10.50.11):**

В GUI на Server:
1. **Config** tab → **FastEthernet0**
   - IP: `10.10.50.11`
   - Mask: `255.255.255.0`
   - Gateway: `10.10.50.1`

2. **Services** tab → **HTTP**
   - Turn **ON**
   - Customize index.html ако желаеш

---

## ЧАСТ 5: ДОКУМЕНТАЦИЯ

### СТЪПКА 9: Device Inventory таблица (ОБНОВЕНА)

| Hostname | Device Type | Model | Management IP | Location | Purpose |
|----------|-------------|-------|---------------|----------|---------|
| ISP-Router | Router | 2911 | N/A | ISP Site | Internet simulation, WAN gateway |
| R1-TechStart | Router | 2911 | N/A | MDF | Customer gateway, NAT, DHCP, ACL |
| SW1-Core | Switch | 2960-24TT | 10.10.99.10 | MDF | Core switching, VLAN database |
| SW2-Access | Switch | 2960-24TT | 10.10.99.20 | IDF-1 | Access layer (Sales, Engineering) |
| SW3-Access | Switch | 2960-24TT | 10.10.99.30 | IDF-2 | Access layer (Mgmt, Guest, Servers) |
| DNS-Server | Server | Server-PT | 10.10.50.10 | Server Room | DNS resolution services |
| Web-Server | Server | Server-PT | 10.10.50.11 | Server Room | Web/Mail/FTP services |

---

### СТЪПКА 10: VLAN Configuration таблица

| VLAN ID | VLAN Name | Subnet | Gateway | DHCP Pool | Assigned Ports |
|---------|-----------|--------|---------|-----------|----------------|
| 10 | Sales | 10.10.10.0/24 | 10.10.10.1 | .10-.50 | SW2 Fa0/1-5 |
| 20 | Engineering | 10.10.20.0/24 | 10.10.20.1 | .10-.30 | SW2 Fa0/6-10 |
| 30 | Management_Users | 10.10.30.0/24 | 10.10.30.1 | .10-.20 | SW3 Fa0/1-5 |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | .10-.100 | SW3 Fa0/6-10 |
| 50 | Servers | 10.10.50.0/24 | 10.10.50.1 | N/A (static) | SW3 Fa0/11-12 |
| 99 | Switch_Management | 10.10.99.0/24 | N/A | N/A | Management only |

---

### СТЪПКА 11: WAN Link Configuration таблица (НОВА ТАБЛИЦА)

| Device | Interface | IP Address | Subnet Mask | Clock Rate | Description |
|--------|-----------|------------|-------------|------------|-------------|
| ISP-Router | Se0/0/0 | 209.165.200.1 | 255.255.255.252 | 64000 (DCE) | WAN to Customer |
| R1-TechStart | Se0/0/0 | 209.165.200.2 | 255.255.255.252 | N/A (DTE) | WAN to ISP |

---

### СТЪПКА 12: ACL Summary

| ACL # | Name | Type | Applied To | Direction | Purpose |
|-------|------|------|------------|-----------|---------|
| 1 | NAT_ACL | Standard | N/A | N/A | Define NAT inside addresses |
| GUEST_FILTER | Extended | Gi0/0.40 | in | Block Guest to internal networks |

---

### СТЪПКА 13: Физическа топология диаграма

**Какво да включиш:**
- ✅ Реално кабелиране (copper straight, Serial DCE)
- ✅ Порт номера (Gi0/1, Fa0/2, Se0/0/0)
- ✅ Кабелни типове (copper, serial)
- ✅ Физическо разположение (ISP Site, MDF, IDF)
- ✅ ISP-Router устройство

**Експортирай от Packet Tracer:**
1. File → Export → **Export to PNG**
2. Запази като `Physical_Topology.png`

---

## ЧАСТ 6: ТЕСТВАНЕ

### СТЪПКА 14: Comprehensive Testing (ОБНОВЕНИ ТЕСТОВЕ)

#### Test 1: Connectivity within VLAN
От PC в VLAN 10:
```
C:\> ipconfig
C:\> ping 10.10.10.1    (gateway)
C:\> ping 10.10.10.11   (друг PC в същия VLAN)
```

**Очакван резултат:** ✅ Всички ping-ове успешни

---

#### Test 2: Inter-VLAN routing
От PC в VLAN 10:
```
C:\> ping 10.10.20.10   (PC в VLAN 20)
C:\> ping 10.10.30.10   (PC в VLAN 30)
C:\> ping 10.10.50.10   (DNS Server в VLAN 50)
```

**Очакван резултат:** ✅ Всички ping-ове успешни

**Ако Test 2 фейлва:**
- Провери subinterfaces на R1: `show ip interface brief`
- Провери trunk между R1 и SW1: `show interfaces trunk`
- Провери дали VLAN 50 съществува на SW1: `show vlan brief`

---

#### Test 3: DNS Resolution (КРИТИЧЕН ТЕСТ)
От PC:
```
C:\> nslookup www.techstart.local
C:\> ping www.techstart.local
```

**Очакван резултат:**
```
Server: 10.10.50.10
Address: 10.10.50.10

Name: www.techstart.local
Address: 10.10.50.11

Reply from 10.10.50.11: bytes=32 time<1ms TTL=127
```

**Ако Test 3 фейлва:**
1. Провери IP на DNS Server: `ipconfig` (трябва да е 10.10.50.10)
2. Ping DNS Server: `ping 10.10.50.10`
3. На R1 провери: `show ip dhcp binding` (DNS трябва да е 10.10.50.10)
4. На SW1 провери: `show vlan brief` (VLAN 50 трябва да съществува)
5. На SW3 провери: `show vlan brief` и `show interfaces fastEthernet 0/11 switchport`

---

#### Test 4: Internet Access via NAT (ОБНОВЕН ТЕСТ)
От PC:
```
C:\> ping 8.8.8.8
C:\> ping 209.165.200.1
C:\> tracert 8.8.8.8
```

**Очакван резултат:**
```
C:\> ping 8.8.8.8
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254

C:\> tracert 8.8.8.8
1  10.10.10.1       # R1 gateway
2  209.165.200.1    # ISP-Router
3  8.8.8.8          # Loopback на ISP
```

**Ако Test 4 фейлва:**
1. На R1 провери default route: `show ip route` (трябва да сочи към 209.165.200.1)
2. На R1 провери NAT: `show ip nat translations`
3. Провери WAN link: `ping 209.165.200.1` от R1
4. На ISP-Router провери return route: `show ip route` (трябва да има 10.10.0.0/16)

---

#### Test 5: Guest Isolation via ACL (ОБНОВЕН ТЕСТ)
От PC в VLAN 40 (Guest):
```
C:\> ping 10.10.10.10   (Sales VLAN)
C:\> ping 10.10.20.10   (Engineering VLAN)
C:\> ping 10.10.30.10   (Management VLAN)
C:\> ping 10.10.50.10   (DNS Server)
C:\> ping 8.8.8.8       (Internet)
```

**Очакван резултат:**
```
ping 10.10.10.10  → ✗ Request timed out (BLOCKED)
ping 10.10.20.10  → ✗ Request timed out (BLOCKED)
ping 10.10.30.10  → ✗ Request timed out (BLOCKED)
ping 10.10.50.10  → ✗ Request timed out (BLOCKED)
ping 8.8.8.8      → ✓ Reply from 8.8.8.8 (ALLOWED)
```

**Ако Test 5 фейлва:**
1. На R1 провери ACL: `show ip access-lists GUEST_FILTER`
2. Провери приложението: `show ip interface gigabitEthernet 0/0.40`
3. Ако всичко е блокирано (включително интернет), провери че ACL завършва с `permit ip any any`

---

#### Test 6: Management Access via SSH
От друг switch или PC (ако е конфигуриран):
```
ssh admin@10.10.99.10
password: admin123
```

**Очакван резултат:** ✅ Успешен login към SW1-Core

**Ако Test 6 фейлва:**
- Провери че SSH е enabled: `show ip ssh`
- Провери че user е създаден: `show run | include username`

---

#### Test 7: WAN Connectivity (НОВ ТЕСТ)
От R1:
```
R1# ping 209.165.200.1
R1# ping 8.8.8.8
R1# show ip nat translations
```

**Очакван резултат:**
```
ping 209.165.200.1  → ✓ Success!
ping 8.8.8.8        → ✓ Success!

NAT translations showing:
Pro Inside global    Inside local     Outside local   Outside global
icmp 209.165.200.2  10.10.10.10      8.8.8.8         8.8.8.8
```

---

## ЧАСТ 7: VERIFICATION CHECKLIST (ОБНОВЕН)

### Layer 2 (Switching):
```
☐ Всички VLAN-и създадени на всички switches (включително VLAN 50!)
☐ Trunk портове работят (show interfaces trunk)
☐ VLAN 50 е visible на trunk портовете
☐ Access портове assigned към правилни VLAN-и
☐ Spanning Tree стабилен (show spanning-tree)
☐ Port Security активиран на критични портове
```

### Layer 3 (Routing):
```
☐ Inter-VLAN routing работи (ping между VLAN-и)
☐ Default route към ISP-Router е конфигуриран
☐ ISP-Router има return route към 10.10.0.0/16
☐ WAN link е up/up (show ip interface brief)
☐ Всички gateway-и отговарят
```

### Services:
```
☐ DHCP раздава IP адреси във всички VLAN-и
☐ DNS resolve-ва локални имена (www.techstart.local)
☐ NAT транслира private към public IP
☐ Клиенти могат да ping-нат 8.8.8.8
```

### Security:
```
☐ ACL блокира Guest достъп до internal networks
☐ Guest има достъп до интернет (8.8.8.8)
☐ SSH работи на всички устройства
☐ Telnet е disabled
☐ Силни пароли използвани
```

### WAN (НОВО):
```
☐ ISP-Router е конфигуриран с loopback 8.8.8.8
☐ Serial link е up/up на двата края
☐ Clock rate е set на DCE страна (ISP-Router)
☐ ISP-Router има static route към 10.10.0.0/16
☐ Traceroute показва hop през ISP-Router
```

### Documentation:
```
☐ Логическа диаграма експортирана (с ISP-Router)
☐ Физическа диаграма експортирана (с ISP-Router)
☐ Device Inventory таблица попълнена (с ISP-Router)
☐ VLAN Configuration таблица попълнена
☐ WAN Link Configuration таблица попълнена (НОВА)
☐ IP Address Plan таблица попълнена
☐ ACL Summary таблица попълнена
```

---

## ЧАСТ 8: ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ И РЕШЕНИЯ (ОБНОВЕНА)

### Проблем 1: Inter-VLAN routing не работи
**Решение:**
- Провери subinterfaces на Router (show ip interface brief)
- Провери trunk между Router и Switch
- Провери gateway в DHCP pools
- **НОВ ЧЕК:** Провери дали VLAN 50 съществува на SW1-Core

### Проблем 2: DHCP не работи
**Решение:**
- Провери дали PC е set на DHCP
- Виж DHCP bindings: `show ip dhcp binding`
- Провери excluded addresses
- Провери дали gateway IP е правилен

### Проблем 3: DNS не resolve-ва
**Решение:**
- Ping-ни DNS Server IP (10.10.50.10)
- Провери дали DNS е в DHCP options
- Провери A records на DNS Server
- **НОВ ЧЕК:** Провери дали VLAN 50 е на SW1, SW3
- **НОВ ЧЕК:** Провери дали Fa0/11-12 на SW3 са в VLAN 50

### Проблем 4: NAT/Internet не работи (ОБНОВЕНО)
**Решение:**
- **НОВ ЧЕК:** Провери че ISP-Router е конфигуриран
- **НОВ ЧЕК:** Провери Serial link: `show ip interface brief` на R1 и ISP
- Провери default route на R1: `show ip route`
- **НОВ ЧЕК:** Провери return route на ISP: `show ip route`
- Провери NAT translations: `show ip nat translations`
- **НОВ ЧЕК:** Ping 8.8.8.8 от R1 първо, след това от клиент

### Проблем 5: Guest ACL блокира интернет
**Решение:**
- Провери че ACL завършва с `permit ip any any`
- Провери че ACL е applied inbound на Gi0/0.40
- Test ping към 8.8.8.8 от Guest VLAN

### Проблем 6: WAN Link е down (НОВ ПРОБЛЕМ)
**Решение:**
- Провери физическото кабелиране (Serial DCE кабел)
- Провери clock rate на DCE страна (ISP-Router Se0/0/0)
- Провери `no shutdown` на двата интерфейса
- Провери IP адресите (209.165.200.1 и .2)

---

## ЧАСТ 9: ФИНАЛНА ПРОВЕРКА (ОБНОВЕНА)

Преди предаване на проекта, провери:

### Technical Functionality:
```
☐ Всички PC-та получават IP от DHCP
☐ Ping работи между всички VLAN-и (включително към VLAN 50)
☐ DNS resolve-ва локални имена (www.techstart.local)
☐ Internet достъп работи (ping 8.8.8.8 успешен)
☐ Traceroute показва hop през ISP-Router
☐ ACL блокира Guest правилно (ping към 10.10.x.x фейлва)
☐ Guest има интернет (ping 8.8.8.8 успешен)
☐ SSH достъп работи към всички устройства
☐ WAN link е up/up
```

### Documentation:
```
☐ Логическа диаграма е професионална и четлива
☐ Логическа диаграма показва ISP-Router
☐ Физическа диаграма показва всички кабели (включително Serial)
☐ Всички таблици са попълнени
☐ Конфигурациите са документирани
☐ Има тестови резултати (screenshots от всички 7 теста)
```

---

## ЧАСТ 10: КЛЮЧОВИ КОНЦЕПЦИИ (ОБНОВЕНА)

### Multi-Subnet Design:
- Всеки subnet служи за конкретна цел
- Правилно subnetting спестява IP адреси
- VLSM позволява flexibility

### Service Integration:
- DHCP автоматизира IP адресирането
- DNS улеснява достъпа (names vs IPs)
- NAT дава интернет достъп
- **НОВ:** ISP simulation осигурява реалистична среда

### Security Layers:
- VLAN separation (Layer 2)
- ACL filtering (Layer 3)
- SSH encryption (management)
- Port Security (access layer)

### WAN Connectivity (НОВО):
- Serial връзка симулира WAN link
- Clock rate на DCE край
- Static routing между AS-и (customer vs ISP)
- NAT overload за множество клиенти

---

## ЧАСТ 11: СЛЕДВАЩИ СТЪПКИ

Браво! Завърши си всички 9 лаба!

Сега си готов за **КУРСОВИЯ ПРОЕКТ**:
- Приложи всичко научено
- Създай собствен сценарий
- Изгради професионална мрежа
- Документирай детайлно

**Бонус упражнения:**
1. Добави втори ISP router за redundancy
2. Конфигурирай HSRP на gateway-ите
3. Добави QoS за Voice VLAN
4. Implement port-based NAT за DMZ

**Успех с проекта!**

---

## ПРИЛОЖЕНИЯ

### ПРИЛОЖЕНИЕ А: Бърза команда справка

#### ISP-Router:
```cisco
show ip interface brief    # Провери интерфейси
show ip route              # Провери routing table
```

#### R1-TechStart:
```cisco
show ip interface brief           # Провери интерфейси
show ip route                     # Провери routing
show ip dhcp binding              # Провери DHCP clients
show ip nat translations          # Провери NAT
show ip access-lists              # Провери ACL
ping 209.165.200.1                # Test WAN
```

#### SW1/SW2/SW3:
```cisco
show vlan brief                   # Провери VLAN
show interfaces trunk             # Провери trunk
show spanning-tree brief          # Провери STP
```

#### От PC:
```powershell
ipconfig /all                     # Пълна IP конфигурация
ping 10.10.10.1                   # Test gateway
ping 10.10.50.10                  # Test DNS
ping 8.8.8.8                      # Test Internet
nslookup www.techstart.local      # Test DNS resolution
tracert 8.8.8.8                   # Trace route to Internet
```

---

### ПРИЛОЖЕНИЕ Б: Съпоставка Cloud vs ISP Router

| Характеристика | Cloud устройство | ISP Router |
|----------------|------------------|------------|
| Симулация на интернет | Ограничена | Пълна ✓ |
| Отговаря на ping | ❌ Не | ✓ Да |
| Traceroute работи | ❌ Не | ✓ Да |
| Loopback адреси | ❌ Не | ✓ Да (8.8.8.8) |
| Static routes | ❌ Не | ✓ Да |
| Реалистичен WAN | ❌ Не | ✓ Да |
| NAT тестване | Ограничено | Пълно ✓ |
| Учебна стойност | Ниска | Висока ✓ |

**Заключение:** ISP Router е предпочитаният избор за учебни цели!

---

**Запази .pkt файла като `Lab9_Integrated_Network_Design_v2.pkt`**

**Дата на ревизия:** 2025
**Версия:** 2.0 (с ISP Router и коригиран VLAN 50)



**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
