# LAB 9: Интегриран мрежов дизайн (3+ subnets + топология диаграми)

**Продължителност:** 120-150 минути  
**Цел:** Интегриране на всички знания от Lab 1-8 в един комплексен проект с множество subnets и професионална документация
**Prerequisite:** Lab 1-6 и Lab 8 (NAT); Lab 7 не е задължителен

> **Надграждащо упражнение:** Lab 7–9 не са задължителни за курсовия проект; основата за проекта са Lab 1–6. Lab 9 демонстрира разширен интегриран дизайн – броят VLAN-и (6), устройствата и WAN инфраструктурата **не са** изисквания на проекта. В курсовия проект важат ограниченията на заданието (2–4 VLAN-а), а сценарият, адресите и топологията на Lab 9 **не се използват** като готов проект.

Упражнението се изпълнява в **нов празен файл** (не е продължение на Lab 8).

---

## ЦЕЛИ НА УПРАЖНЕНИЕТО

След завършване на този лаб ще можете да:
- ✅ Проектирате мрежа с 3+ subnet-а едновременно
- ✅ Интегрирате VLAN, DHCP, DNS, ACL, NAT в една топология
- ✅ Създавате професионални логически и физически топология диаграми
- ✅ Планирате IP addressing scheme (основно /24; VLSM – в Задача А)
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
- 3 отдела: Sales, Engineering, Administration
- 28 служители общо (15 + 8 + 5)
- 2 сървъра (Web, DNS)
- 1 гостна мрежа (Guest; в упражнението – кабелни PC)

**Нужди:**
- Разделение на отделите (VLAN-и)
- DHCP за автоматично IP адресиране
- DNS за локални имена
- Интернет достъп (NAT)
- Сигурност между отделите (ACL)

> **Забележка за IP схемата:** В този лаб използваме **10.10.x.0/24** вместо 192.168.x.0/24 от предишните лабове. Това е умишлено - симулираме нов проект за компания "TechStart" с различно IP адресно пространство. В реалния свят всяка организация има уникална IP схема.

**Subnet план:**
- Management VLAN: 10.10.99.0/24
- VLAN 10 - Sales: 10.10.10.0/24 (15 users)
- VLAN 20 - Engineering: 10.10.20.0/24 (8 users)
- VLAN 30 - Administration: 10.10.30.0/24 (5 users)
- VLAN 40 - Guest: 10.10.40.0/24 (10 guests)
- VLAN 50 - Servers: 10.10.50.0/24 (2 servers)
- WAN Link: 209.165.200.0/30 (към ISP)

---

## ЧАСТ 1: IP ADDRESSING PLAN

### СТЪПКА 1: Създаване на IP addressing таблица

| Subnet | Network | First IP | Last IP | Broadcast | Gateway | VLAN | Hosts |
|--------|---------|----------|---------|-----------|---------|------|-------|
| Switch Management | 10.10.99.0/24 | 10.10.99.1 | 10.10.99.254 | 10.10.99.255 | 10.10.99.1 | 99 | R1 + 3 комутатора |
| Sales | 10.10.10.0/24 | 10.10.10.1 | 10.10.10.254 | 10.10.10.255 | 10.10.10.1 | 10 | 15 |
| Engineering | 10.10.20.0/24 | 10.10.20.1 | 10.10.20.254 | 10.10.20.255 | 10.10.20.1 | 20 | 8 |
| Administration | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.254 | 10.10.30.255 | 10.10.30.1 | 30 | 5 |
| Guest | 10.10.40.0/24 | 10.10.40.1 | 10.10.40.254 | 10.10.40.255 | 10.10.40.1 | 40 | 10 |
| Servers | 10.10.50.0/24 | 10.10.50.1 | 10.10.50.254 | 10.10.50.255 | 10.10.50.1 | 50 | 2 |
| WAN Link | 209.165.200.0/30 | 209.165.200.1 | 209.165.200.2 | 209.165.200.3 | N/A | N/A | ISP link |

**DHCP Pools** (диапазонът се задава с excluded-address за началото и края на мрежата):
- VLAN 10: 10.10.10.10 - 10.10.10.50
- VLAN 20: 10.10.20.10 - 10.10.20.30
- VLAN 30: 10.10.30.10 - 10.10.30.20
- VLAN 40: 10.10.40.10 - 10.10.40.100

**Excluded ranges:**
- 10.10.10.1-9 (gateway, printers) и 10.10.10.51-254
- 10.10.20.1-9 и 10.10.20.31-254
- 10.10.30.1-9 и 10.10.30.21-254
- 10.10.40.1-9 и 10.10.40.101-254
- VLAN 50 и VLAN 99: без DHCP pool (статични адреси)

---

## ЧАСТ 2: ЛОГИЧЕСКА ТОПОЛОГИЯ

### СТЪПКА 2: Създаване на логическа диаграма

**Как да направите диаграма в Packet Tracer:**

1. В Packet Tracer отворете вашата топология
2. Меню **File** → **Export** → **Export to PNG** (ако във вашата версия няма такава опция, използвайте **File → Print** към PDF или екранна снимка)
3. Запазете като `Logical_Topology.png`

**Алтернатива - Draw.io / Lucidchart:**

1. Отворете [draw.io](https://app.diagrams.net/)
2. Изберете **Network Diagram** template
3. Добавете устройства:
   - 2x Router (R1 и ISP-Router)
   - Switches (иконка Switch)
   - PCs grouped by VLAN
   - Servers
   - Internet symbol

**Какво да включите в логическата диаграма:**
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
      │ Gi0/0: Trunk (NAT Inside), .99 native
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
   Sales Eng     Admin Guest Servers
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
- 1x Router 2911 (ISP-Router) – с модул HWIC-2T
- 1x Router 2911 (R1) – с модул HWIC-2T
- 3x Switch 2960-24TT (SW1-Core, SW2-Access, SW3-Access)
- 20x PC за потребители (демонстрационни – по 5 на VLAN; планираният брой е в Част 1)
- 2x Server (Web, DNS)

**Serial модул:** Router 2911 в Packet Tracer няма Serial портове по подразбиране. На двата рутера: Physical → изключете захранването → плъзнете **HWIC-2T** в слот HWIC 0 → включете (както в Lab 8). При кабела Serial DCE започнете от ISP-Router, за да е той DCE страната.

**Роля на SW1:** в тази малка мрежа R1 изпълнява ролята на Core, а SW1 е Distribution слой (събира trunk-овете към Access комутаторите).

**Кабелиране:**

```
# WAN connection (към ISP-Router)
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
SW3 Fa0/1-5 → PCs (VLAN 30 - Administration)
SW3 Fa0/6-10 → PCs (VLAN 40 - Guest)
SW3 Fa0/11-12 → Servers (VLAN 50)
```

---

## ЧАСТ 4: КОНФИГУРАЦИЯ

### СТЪПКА 4A: ISP Router конфигурация

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

! Маршрут към частната мрежа 10.10.0.0/16 НЕ се задава:
! при PAT отговорите се връщат към 209.165.200.2 (директно свързана мрежа)

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
 description Administration VLAN
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

! Management VLAN 99 (native) - gateway за комутаторите
interface gigabitEthernet 0/0.99
 description Switch Management
 encapsulation dot1Q 99 native
 ip address 10.10.99.1 255.255.255.0
 exit

! DHCP Pools for VLANs
ip dhcp excluded-address 10.10.10.1 10.10.10.9
ip dhcp excluded-address 10.10.10.51 10.10.10.254
ip dhcp excluded-address 10.10.20.1 10.10.20.9
ip dhcp excluded-address 10.10.20.31 10.10.20.254
ip dhcp excluded-address 10.10.30.1 10.10.30.9
ip dhcp excluded-address 10.10.30.21 10.10.30.254
ip dhcp excluded-address 10.10.40.1 10.10.40.9
ip dhcp excluded-address 10.10.40.101 10.10.40.254

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

ip dhcp pool ADMIN_POOL
 network 10.10.30.0 255.255.255.0
 default-router 10.10.30.1
 dns-server 10.10.50.10
 domain-name techstart.local
 lease 30
 exit

ip dhcp pool GUEST_POOL
 network 10.10.40.0 255.255.255.0
 default-router 10.10.40.1
 dns-server 10.10.50.10
 lease 0 1 0
 exit

! NAT - PAT Overload
access-list 1 permit 10.10.0.0 0.0.255.255
ip nat inside source list 1 interface serial 0/0/0 overload

! Default route to Internet (към ISP-Router)
ip route 0.0.0.0 0.0.0.0 209.165.200.1

! ACL - Guest isolation (само DNS към вътрешния сървър; без достъп до вътрешните мрежи)
ip access-list extended GUEST_FILTER
 remark Allow DNS to local server
 permit udp 10.10.40.0 0.0.0.255 host 10.10.50.10 eq 53
 permit tcp 10.10.40.0 0.0.0.255 host 10.10.50.10 eq 53
 remark Deny internal networks and switch management
 deny ip 10.10.40.0 0.0.0.255 10.10.10.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.20.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.30.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.50.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.99.0 0.0.0.255
 remark Allow remaining traffic (simulated Internet)
 permit ip any any
 exit

interface gigabitEthernet 0/0.40
 ip access-group GUEST_FILTER in
 exit

! SSH Configuration (достъп само от Administration VLAN 30 и Management VLAN 99)
ip domain-name techstart.local
crypto key generate rsa
1024
ip ssh version 2

access-list 15 remark VTY_ACCESS
access-list 15 permit 10.10.30.0 0.0.0.255
access-list 15 permit 10.10.99.0 0.0.0.255

line vty 0 4
 transport input ssh
 login local
 access-class 15 in
 exit

username admin privilege 15 secret admin123

end
copy running-config startup-config
```

**Бележки:**
- **Lease:** командата `lease` се приема само ако версията на Packet Tracer я поддържа (проверка: `lease ?` в DHCP pool-а). Ако `lease` или `domain-name` не се приема, пропуснете я и опишете срока в документацията.
- **Guest DNS:** Guest получава вътрешния DNS (10.10.50.10) и GUEST_FILTER разрешава само DNS към него. 8.8.8.8 е loopback на ISP-Router и не е DNS сървър.
- **Пароли:** `class123`, `admin123` и `cisco` са само примерни. В курсовия проект слабите пароли са забранени.

---

### СТЪПКА 5: Core Switch (SW1)

```cisco
enable
configure terminal
hostname SW1-Core
no ip domain-lookup
enable secret class123

! VLAN Creation
vlan 10
 name Sales
 exit
vlan 20
 name Engineering
 exit
vlan 30
 name Administration
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

! Trunk to Router (всички VLAN-и)
interface gigabitEthernet 0/1
 description Trunk to R1-TechStart
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Trunk to SW2-Access (само VLAN-ите на SW2)
interface gigabitEthernet 0/2
 description Trunk to SW2-Access
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Trunk to SW3-Access (VLAN-ите на SW3, включително 50)
interface fastEthernet 0/24
 description Trunk to SW3-Access
 switchport mode trunk
 switchport trunk allowed vlan 30,40,50,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Shutdown unused ports
interface range fastEthernet 0/1-23
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

line vty 0 15
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
 name Switch_Management
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

interface gigabitEthernet 0/2
 shutdown
 exit

! SSH Config
ip domain-name techstart.local
crypto key generate rsa
1024
ip ssh version 2

line vty 0 15
 transport input ssh
 login local
 exit

username admin privilege 15 secret admin123

end
copy running-config startup-config
```

---

**SW3-Access (Administration + Guest + Servers):**

```cisco
enable
configure terminal
hostname SW3-Access
no ip domain-lookup
enable secret class123

! VLAN Creation
vlan 30
 name Administration
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
 ip address 10.10.99.30 255.255.255.0
 no shutdown
 exit

ip default-gateway 10.10.99.1

! Trunk to Core
interface gigabitEthernet 0/1
 description Trunk to SW1-Core
 switchport mode trunk
 switchport trunk allowed vlan 30,40,50,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Access ports for Administration
interface range fastEthernet 0/1-5
 description Administration Department
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
 description Guest
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

interface gigabitEthernet 0/2
 shutdown
 exit

! SSH Config
ip domain-name techstart.local
crypto key generate rsa
1024
ip ssh version 2

line vty 0 15
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
     portal.techstart.local  → www.techstart.local
     webmail.techstart.local → mail.techstart.local
     ```
   - За CNAME Packet Tracer показва поле **Host Name** – попълнете каноничното име (както в Lab 4)

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
   - Customize index.html по желание

---

## ЧАСТ 5: ДОКУМЕНТАЦИЯ

### СТЪПКА 9: Device Inventory таблица

| Hostname | Device Type | Model | Management IP | Location | Purpose |
|----------|-------------|-------|---------------|----------|---------|
| ISP-Router | Router | 2911 | N/A | ISP Site | Internet simulation, WAN gateway |
| R1-TechStart | Router | 2911 | 10.10.99.1 | MDF | Customer gateway, NAT, DHCP, ACL |
| SW1-Core | Switch | 2960-24TT | 10.10.99.10 | MDF | Core switching, VLAN database |
| SW2-Access | Switch | 2960-24TT | 10.10.99.20 | IDF-1 | Access layer (Sales, Engineering) |
| SW3-Access | Switch | 2960-24TT | 10.10.99.30 | IDF-2 | Access layer (Admin, Guest, Servers) |
| DNS-Server | Server | Server-PT | 10.10.50.10 | Server Room | DNS resolution services |
| Web-Server | Server | Server-PT | 10.10.50.11 | Server Room | Web/Mail/FTP services |

---

### СТЪПКА 10: VLAN Configuration таблица

| VLAN ID | VLAN Name | Subnet | Gateway | DHCP Pool | Assigned Ports |
|---------|-----------|--------|---------|-----------|----------------|
| 10 | Sales | 10.10.10.0/24 | 10.10.10.1 | .10-.50 | SW2 Fa0/1-5 |
| 20 | Engineering | 10.10.20.0/24 | 10.10.20.1 | .10-.30 | SW2 Fa0/6-10 |
| 30 | Administration | 10.10.30.0/24 | 10.10.30.1 | .10-.20 | SW3 Fa0/1-5 |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | .10-.100 | SW3 Fa0/6-10 |
| 50 | Servers | 10.10.50.0/24 | 10.10.50.1 | N/A (static) | SW3 Fa0/11-12 |
| 99 | Switch_Management | 10.10.99.0/24 | 10.10.99.1 | N/A (static) | Native VLAN на trunk-овете |

---

### СТЪПКА 11: WAN Link Configuration таблица

| Device | Interface | IP Address | Subnet Mask | Clock Rate | Description |
|--------|-----------|------------|-------------|------------|-------------|
| ISP-Router | Se0/0/0 | 209.165.200.1 | 255.255.255.252 | 64000 (DCE) | WAN to Customer |
| R1-TechStart | Se0/0/0 | 209.165.200.2 | 255.255.255.252 | N/A (DTE) | WAN to ISP |

---

### СТЪПКА 12: ACL Summary

| ACL # | Name | Type | Applied To | Direction | Purpose |
|-------|------|------|------------|-----------|---------|
| 1 | NAT_ACL | Standard | N/A | N/A | Define NAT inside addresses |
| - | GUEST_FILTER | Extended | Gi0/0.40 | in | DNS към 10.10.50.10; Guest без достъп до вътрешните мрежи и VLAN 99 |
| 15 | VTY_ACCESS | Standard | line vty 0 4 (R1) | in | SSH към R1 само от VLAN 30 и VLAN 99 |

---

### СТЪПКА 13: Физическа топология диаграма

**Какво да включите:**
- ✅ Реално кабелиране (copper straight, Serial DCE)
- ✅ Порт номера (Gi0/1, Fa0/2, Se0/0/0)
- ✅ Кабелни типове (copper, serial)
- ✅ Физическо разположение (ISP Site, MDF, IDF)
- ✅ ISP-Router устройство

**Експортирайте от Packet Tracer:**
1. File → Export → **Export to PNG** (или File → Print към PDF / екранна снимка)
2. Запазете като `Physical_Topology.png`

---

## ЧАСТ 6: ТЕСТВАНЕ

### СТЪПКА 14: Comprehensive Testing

В документацията използвайте действително получените DHCP адреси; адресите в примерите (напр. 10.10.10.10) може да се различават.

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

**Ако Test 2 е неуспешен:**
- Проверете subinterfaces на R1: `show ip interface brief`
- Проверете trunk-а към R1 на SW1: `show interfaces trunk` (командата е за комутатора; на R1 проверете subinterfaces и `encapsulation dot1Q` в `show running-config`)
- Проверете дали VLAN 50 съществува на SW1: `show vlan brief`

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

**Ако Test 3 е неуспешен:**
1. Проверете IP на DNS Server (Desktop → IP Configuration; трябва да е 10.10.50.10)
2. Ping DNS Server: `ping 10.10.50.10`
3. На PC: `ipconfig /all` (DNS Server трябва да е 10.10.50.10); на R1 – `dns-server` в DHCP pool-а (`show running-config`)
4. На SW1 проверете: `show vlan brief` (VLAN 50 трябва да съществува)
5. На SW3 проверете: `show vlan brief` и `show interfaces fastEthernet 0/11 switchport`

---

#### Test 4: Internet Access via NAT
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
2  8.8.8.8          # ISP-Router (8.8.8.8 е негов loopback)
```

Вторият hop е самият ISP-Router, затова трети hop няма. Показаните адреси може да се различават според версията на Packet Tracer.

**Ако Test 4 е неуспешен:**
1. На R1 проверете default route: `show ip route` (трябва да сочи към 209.165.200.1)
2. На R1 проверете NAT: `show ip nat translations`
3. Проверете WAN link: `ping 209.165.200.1` от R1
4. На ISP-Router: `show ip route` – маршрут към 10.10.0.0/16 **не трябва** да има. Ако ping работи и без NAT, проверете дали ISP няма такъв маршрут

---

#### Test 5: Guest Isolation via ACL
От PC в VLAN 40 (Guest):
```
C:\> ping 10.10.10.10   (Sales VLAN)
C:\> ping 10.10.20.10   (Engineering VLAN)
C:\> ping 10.10.30.10   (Administration VLAN)
C:\> ping 10.10.50.10   (DNS Server)
C:\> ping 10.10.99.10   (Switch management)
C:\> nslookup www.techstart.local
C:\> ping 8.8.8.8       (Internet)
```

**Очакван резултат:**
```
ping 10.10.10.10  → ✗ Request timed out (BLOCKED)
ping 10.10.20.10  → ✗ Request timed out (BLOCKED)
ping 10.10.30.10  → ✗ Request timed out (BLOCKED)
ping 10.10.50.10  → ✗ Request timed out (BLOCKED - разрешен е само DNS)
ping 10.10.99.10  → ✗ Request timed out (BLOCKED)
nslookup          → ✓ 10.10.50.11 (DNS е разрешен)
ping 8.8.8.8      → ✓ Reply from 8.8.8.8 (ALLOWED)
```

**Ако Test 5 е неуспешен:**
1. На R1 проверете ACL: `show ip access-lists GUEST_FILTER`
2. Проверете приложението: `show ip interface gigabitEthernet 0/0.40`
3. Ако всичко е блокирано (включително интернет), проверете че ACL завършва с `permit ip any any`

---

#### Test 6: Management Access via SSH
От PC във VLAN 30 (Administration):
```
C:\> ssh -l admin 10.10.99.10
Password: admin123
C:\> ssh -l admin 10.10.30.1
```

**Очакван резултат:** ✅ Успешен login към SW1-Core и към R1. От PC във VLAN 10 SSH към R1 е отказан (access-class 15).

**Ако Test 6 е неуспешен:**
- Проверете че SSH е enabled: `show ip ssh`
- Проверете че user е създаден: `show run | include username`
- Проверете access-class 15 на R1 (SSH е разрешен само от VLAN 30 и VLAN 99)

---

#### Test 7: WAN Connectivity и PAT
От PC1 (VLAN 10):
```
C:\> ping 8.8.8.8
```

На R1:
```
R1# show ip nat translations
R1# show ip nat statistics
```

**Очакван резултат:**
```
Pro  Inside global      Inside local       Outside local     Outside global
icmp 209.165.200.2:1    10.10.10.10:1      8.8.8.8:1         8.8.8.8:1
```

Inside local е адресът, който PC1 е получил от DHCP; номерата след „:“ може да се различават.

Отделно, само като проверка на WAN routing (не на PAT):
```
R1# ping 209.165.200.1
R1# ping 8.8.8.8
```
Ping, изпратен от самия R1, не минава през `ip nat inside` интерфейс и не създава PAT превод.

---

## ЧАСТ 7: VERIFICATION CHECKLIST

### Layer 2 (Switching):
```
☐ VLAN-ите са създадени на комутаторите, през които минават (SW1 - всички; SW2 - 10, 20, 99; SW3 - 30, 40, 50, 99)
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
☐ ISP-Router НЯМА маршрут към частната мрежа 10.10.0.0/16
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
☐ ACL блокира Guest достъп до internal networks и VLAN 99
☐ Guest има DNS (10.10.50.10) и достъп до интернет (8.8.8.8)
☐ SSH работи на всички устройства
☐ Telnet е disabled
☐ В курсовия проект - силни пароли (примерните admin123, class123 и cisco са забранени)
```

### WAN:
```
☐ ISP-Router е конфигуриран с loopback 8.8.8.8
☐ Serial link е up/up на двата края
☐ Clock rate е set на DCE страна (ISP-Router)
☐ ISP-Router няма маршрут към 10.10.0.0/16 (интернет работи само чрез NAT)
☐ Traceroute показва hop през ISP-Router
```

### Documentation:
```
☐ Логическа диаграма експортирана (с ISP-Router)
☐ Физическа диаграма експортирана (с ISP-Router)
☐ Device Inventory таблица попълнена (с ISP-Router)
☐ VLAN Configuration таблица попълнена
☐ WAN Link Configuration таблица попълнена
☐ IP Address Plan таблица попълнена
☐ ACL Summary таблица попълнена
```

---

## ЧАСТ 8: ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ И РЕШЕНИЯ

### Проблем 1: Inter-VLAN routing не работи
**Решение:**
- Проверете subinterfaces на Router (show ip interface brief)
- Проверете trunk-а на SW1 към R1 (`show interfaces trunk` на SW1)
- Проверете gateway в DHCP pools
- Проверете дали VLAN 50 съществува на SW1-Core

### Проблем 2: DHCP не работи
**Решение:**
- Проверете дали PC е set на DHCP
- Вижте DHCP bindings: `show ip dhcp binding`
- Проверете excluded addresses
- Проверете дали gateway IP е правилен

### Проблем 3: DNS не resolve-ва
**Решение:**
- Ping-нете DNS сървъра (10.10.50.10)
- Проверете дали DNS е в DHCP options
- Проверете A records на DNS Server
- Проверете дали VLAN 50 е на SW1, SW3
- Проверете дали Fa0/11-12 на SW3 са в VLAN 50

### Проблем 4: NAT/Internet не работи
**Решение:**
- Проверете че ISP-Router е конфигуриран
- Проверете Serial link: `show ip interface brief` на R1 и ISP
- Проверете default route на R1: `show ip route`
- ISP не трябва да има маршрут към 10.10.0.0/16; проверете NAT на R1, не маршрутите на ISP
- Проверете NAT translations: `show ip nat translations`
- Ping 8.8.8.8 от R1 първо, след това от клиент

### Проблем 5: Guest ACL блокира интернет
**Решение:**
- Проверете че ACL завършва с `permit ip any any`
- Проверете че ACL е applied inbound на Gi0/0.40
- Test ping към 8.8.8.8 от Guest VLAN

### Проблем 6: WAN Link е down
**Решение:**
- Проверете модула HWIC-2T и кабелирането (Serial DCE кабел)
- Проверете clock rate на DCE страна (ISP-Router Se0/0/0)
- Проверете `no shutdown` на двата интерфейса
- Проверете IP адресите (209.165.200.1 и .2)

---

## ЧАСТ 9: ФИНАЛНА ПРОВЕРКА

Преди предаване на проекта, проверете:

### Technical Functionality:
```
☐ Всички PC-та получават IP от DHCP
☐ Ping работи между разрешените VLAN-и (Guest умишлено е блокиран)
☐ DNS resolve-ва локални имена (www.techstart.local)
☐ Internet достъп работи (ping 8.8.8.8 успешен)
☐ Traceroute показва hop през ISP-Router
☐ ACL блокира Guest правилно (ping към 10.10.x.x е неуспешен, DNS работи)
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
☐ Има тестови резултати (screenshots от всички 7 теста, с действително получените адреси)
```

---

## ЧАСТ 10: КЛЮЧОВИ КОНЦЕПЦИИ

### Multi-Subnet Design:
- Всеки subnet служи за конкретна цел
- Правилно subnetting спестява IP адреси
- VLSM позволява flexibility

### Service Integration:
- DHCP автоматизира IP адресирането
- DNS улеснява достъпа (names vs IPs)
- NAT дава интернет достъп
- ISP simulation осигурява реалистична среда

### Security Layers:
- VLAN separation (Layer 2)
- ACL filtering (Layer 3)
- SSH encryption (management)
- Port Security (access layer)

### WAN Connectivity:
- Serial връзка симулира WAN link
- Clock rate на DCE край
- Static routing между клиента и ISP (default route на R1; ISP не маршрутизира частни адреси)
- NAT overload за множество клиенти

---

## ЧАСТ 10А: ДОПЪЛНИТЕЛНИ ЗАДАЧИ (BONUS)

### Задача А: VLSM адресен план

Основният пример използва /24 за простота – това не е VLSM. Направете план в 10.10.0.0/24 с маски според броя на устройствата (от най-голямата подмрежа към най-малката):

| Мрежа | Нужда | Подмрежа | Хостове | Broadcast |
|-------|-------|----------|---------|-----------|
| Sales | 15 + gateway | 10.10.0.0/27 | .1 – .30 | .31 |
| Guest | 10 + gateway | 10.10.0.32/28 | .33 – .46 | .47 |
| Engineering | 8 + gateway | 10.10.0.48/28 | .49 – .62 | .63 |
| Administration | 5 + gateway | 10.10.0.64/29 | .65 – .70 | .71 |
| Servers | 2 + gateway | 10.10.0.72/29 | .73 – .78 | .79 |
| Switch Management | R1 + 3 комутатора | 10.10.0.80/29 | .81 – .86 | .87 |

Пренастройте подинтерфейсите, DHCP pool-овете (маските в `network`) и ACL wildcard маските. Маршрутът по подразбиране и NAT остават същите.

### Задача Б: IoT интеграция

Самостоятелно добавете IoT Server (Server-PT, Services → IoT **ON**), Access Point-PT, един сензор (Motion Detector) и един актуатор (Light) с правило в Conditions (както в Lab 6). В документацията обосновете в коя VLAN са IoT устройствата и как ACL ограничава достъпа до тях. Отделна IoT VLAN прави 7 VLAN-а – допустимо в упражнението, но не и в курсовия проект (най-много 4).

---

## ЧАСТ 11: СЛЕДВАЩИ СТЪПКИ

Браво! Завършихте всички 9 упражнения!

За **КУРСОВИЯ ПРОЕКТ** основата са Lab 1–6; Lab 7–9 са надграждащи:
- Приложете наученото
- Създайте собствен сценарий (различен от Lab 9)
- Изградете мрежа в рамките на заданието (2–4 VLAN-а)
- Документирайте детайлно

**Бонус упражнения:**
1. Добавете втори ISP router за redundancy
2. Конфигурирайте HSRP на gateway-ите
3. Добавете QoS за Voice VLAN
4. Implement port-based NAT за DMZ

**Успех с проекта!**

---

## ПРИЛОЖЕНИЯ

### ПРИЛОЖЕНИЕ А: Бърза команда справка

#### ISP-Router:
```cisco
show ip interface brief    # Проверете интерфейси
show ip route              # Проверете routing table
```

#### R1-TechStart:
```cisco
show ip interface brief           # Проверете интерфейси
show ip route                     # Проверете routing
show ip dhcp binding              # Проверете DHCP clients
show ip nat translations          # Проверете NAT
show ip access-lists              # Проверете ACL
ping 209.165.200.1                # Test WAN
```

#### SW1/SW2/SW3:
```cisco
show vlan brief                   # Проверете VLAN
show interfaces trunk             # Проверете trunk
show spanning-tree brief          # Проверете STP
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

**Запазете .pkt файла като `Lab9_YourName.pkt`**

**Дата на ревизия:** 2026
**Версия:** 3.0



**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
