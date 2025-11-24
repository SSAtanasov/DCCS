# LAB 9: Интегриран мрежов дизайн (3+ subnets + топология диаграми)

**Продължителност:** 120-150 минути  
**Цел:** Интегриране на всички знания от Lab 1-8 в един комплексен проект с множество subnets и професионална документация

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

**Subnet план:**
- Management VLAN: 10.10.99.0/24
- VLAN 10 - Sales: 10.10.10.0/24 (15 users)
- VLAN 20 - Engineering: 10.10.20.0/24 (8 users)
- VLAN 30 - Management: 10.10.30.0/24 (5 users)
- VLAN 40 - Guest: 10.10.40.0/24 (10 guests)
- Server Subnet: 10.10.50.0/24 (2 servers)

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
   - Router (иконка Router)
   - Switches (иконка Switch)
   - PCs grouped by VLAN
   - Servers
   - Cloud (интернет)

**Какво да включиш в логическата диаграма:**
- ✅ Всички устройства с hostname
- ✅ VLAN-и с names и IDs
- ✅ IP адреси на gateway-и
- ✅ Subnet notations (10.10.10.0/24)
- ✅ Trunk links маркирани
- ✅ Легенда (Legend)

**Пример логическа диаграма структура:**

```
         INTERNET
         (Cloud)
      209.165.200.1
            │
            │ Se0/0/0 (NAT outside)
            │ 209.165.200.2/30
        ┌───▼─────┐
        │   R1    │  Inter-VLAN routing
        │ Gateway │  NAT, ACL
        └────┬────┘
             │ Gi0/0 (Trunk)
             │ 10.10.x.1
        ┌────▼────┐
        │   SW1   │  Core Switch
        │  2960   │  VLAN database
        └──┬───┬──┘
           │   │
    ┌──────┘   └──────┐
    │                 │
┌───▼────┐       ┌────▼────┐
│  SW2   │       │   SW3   │  Access Switches
│ Access │       │ Access  │
└─┬──┬─┬─┘       └─┬─┬──┬──┘
  │  │ │           │ │  │
VLAN 10 20        30 40 50
Sales  Eng        Mgmt Guest Servers
```

---

## ЧАСТ 3: ФИЗИЧЕСКА ТОПОЛОГИЯ

### СТЪПКА 3: Изграждане в Packet Tracer

**Устройства:**
- 1x Router 2911 (R1)
- 3x Switch 2960-24TT (SW1-Core, SW2-Access, SW3-Access)
- 20x PC за потребители
- 2x Server (Web, DNS)
- 1x Cloud (Internet simulation)

**Кабелиране:**

```
# WAN connection
R1 Se0/0/0 ↔ Cloud

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

### СТЪПКА 4: Router конфигурация (R1)

```cisco
enable
configure terminal
hostname R1-TechStart
no ip domain-lookup
enable secret class123

! Outside interface (to Internet)
interface serial 0/0/0
 description WAN to ISP
 ip address 209.165.200.2 255.255.255.252
 ip nat outside
 clock rate 64000
 no shutdown
 exit

! Inside interface (to internal network) - Trunk with subinterfaces
interface gigabitEthernet 0/0
 description Trunk to SW1
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
 lease 7
 exit

ip dhcp pool MANAGEMENT_POOL
 network 10.10.30.0 255.255.255.0
 default-router 10.10.30.1
 dns-server 10.10.50.10
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

! Default route to Internet
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
vlan 20
 name Engineering
vlan 30
 name Management
vlan 40
 name Guest
vlan 50
 name Servers
vlan 99
 name Management
 exit

! Management IP
interface vlan 99
 ip address 10.10.99.10 255.255.255.0
 no shutdown
 exit

ip default-gateway 10.10.99.1

! Trunk to Router
interface gigabitEthernet 0/1
 description Trunk to R1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Trunk to Access Switches
interface range gigabitEthernet 0/2, fastEthernet 0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk native vlan 99
 no shutdown
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
vlan 20
 name Engineering
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
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 switchport trunk native vlan 99
 no shutdown
 exit

! Access ports for Sales
interface range fastEthernet 0/1-5
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
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
 exit

! Shutdown unused ports
interface range fastEthernet 0/11-23
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
username admin privilege 15 secret admin123

end
copy running-config startup-config
```

**SW3-Access (Management + Guest + Servers):**

Подобна конфигурация, но с VLAN 30, 40, 50

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
     ```
   - Add CNAME Records:
     ```
     portal → www.techstart.local
     webmail → mail.techstart.local
     ```

---

## ЧАСТ 5: ДОКУМЕНТАЦИЯ

### СТЪПКА 8: Device Inventory таблица

| Hostname | Device Type | Model | Management IP | Location | Purpose |
|----------|-------------|-------|---------------|----------|---------|
| R1-TechStart | Router | 2911 | N/A | MDF | Gateway, NAT, DHCP |
| SW1-Core | Switch | 2960-24TT | 10.10.99.10 | MDF | Core switching |
| SW2-Access | Switch | 2960-24TT | 10.10.99.20 | IDF-1 | Access layer |
| SW3-Access | Switch | 2960-24TT | 10.10.99.30 | IDF-2 | Access layer |
| DNS-Server | Server | Server-PT | 10.10.50.10 | Server Room | DNS services |
| Web-Server | Server | Server-PT | 10.10.50.11 | Server Room | Web/Mail/FTP |

---

### СТЪПКА 9: VLAN Configuration таблица

| VLAN ID | VLAN Name | Subnet | Gateway | DHCP Pool | Assigned Ports |
|---------|-----------|--------|---------|-----------|----------------|
| 10 | Sales | 10.10.10.0/24 | 10.10.10.1 | .10-.50 | SW2 Fa0/1-5 |
| 20 | Engineering | 10.10.20.0/24 | 10.10.20.1 | .10-.30 | SW2 Fa0/6-10 |
| 30 | Management | 10.10.30.0/24 | 10.10.30.1 | .10-.20 | SW3 Fa0/1-5 |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | .10-.100 | SW3 Fa0/6-10 |
| 50 | Servers | 10.10.50.0/24 | 10.10.50.1 | N/A (static) | SW3 Fa0/11-12 |
| 99 | Management | 10.10.99.0/24 | N/A | N/A | Management only |

---

### СТЪПКА 10: ACL Summary

| ACL # | Name | Type | Applied To | Direction | Purpose |
|-------|------|------|------------|-----------|---------|
| 1 | NAT_ACL | Standard | N/A | N/A | Define NAT inside addresses |
| GUEST_FILTER | Extended | Gi0/0.40 | in | Block Guest to internal networks |

---

### СТЪПКА 11: Физическа топология диаграма

**Какво да включиш:**
- ✅ Реално кабелиране (copper straight, crossover)
- ✅ Порт номера (Gi0/1, Fa0/2)
- ✅ Кабелни типове (fiber, copper)
- ✅ Физическо разположение (MDF, IDF)

**Експортирай от Packet Tracer:**
1. File → Export → **Export to PNG**
2. Запази като `Physical_Topology.png`

---

## ЧАСТ 6: ТЕСТВАНЕ

### СТЪПКА 12: Comprehensive Testing

#### Test 1: Connectivity within VLAN
От PC в VLAN 10:
```
C:\> ipconfig
C:\> ping 10.10.10.1    (gateway)
C:\> ping 10.10.10.11   (друг PC в същия VLAN)
```

#### Test 2: Inter-VLAN routing
От PC в VLAN 10:
```
C:\> ping 10.10.20.10   (PC в VLAN 20)
C:\> ping 10.10.30.10   (PC в VLAN 30)
```

#### Test 3: DNS Resolution
От PC:
```
C:\> ping www.techstart.local
C:\> nslookup www.techstart.local
```

#### Test 4: Internet Access (NAT)
От PC:
```
C:\> ping 8.8.8.8
C:\> tracert 8.8.8.8
```

#### Test 5: Guest Isolation (ACL)
От PC в VLAN 40 (Guest):
```
C:\> ping 10.10.10.10   (Should FAIL - blocked by ACL)
C:\> ping 8.8.8.8       (Should WORK - Internet allowed)
```

#### Test 6: Management Access
От друг switch или PC:
```
ssh admin@10.10.99.10
password: admin123
```

---

## VERIFICATION CHECKLIST

### Layer 2 (Switching):
```
☐ Всички VLAN-и създадени на всички switches
☐ Trunk портове работят (show interfaces trunk)
☐ Access портове assigned към правилни VLAN-и
☐ Spanning Tree стабилен (show spanning-tree)
☐ Port Security активиран на критични портове
```

### Layer 3 (Routing):
```
☐ Inter-VLAN routing работи (ping между VLAN-и)
☐ Default route към интернет е конфигуриран
☐ Всички gateway-и отговарят
```

### Services:
```
☐ DHCP раздава IP адреси във всички VLAN-и
☐ DNS resolve-ва локални имена
☐ NAT транслира private към public IP
```

### Security:
```
☐ ACL блокира Guest достъп до internal networks
☐ Guest има достъп до интернет
☐ SSH работи на всички устройства
☐ Telnet е disabled
☐ Силни пароли използвани
```

### Documentation:
```
☐ Логическа диаграма експортирана
☐ Физическа диаграма експортирана
☐ Device Inventory таблица попълнена
☐ VLAN Configuration таблица попълнена
☐ IP Address Plan таблица попълнена
☐ ACL Summary таблица попълнена
```

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ

### Проблем: Inter-VLAN routing не работи
**Решение:**
- Провери subinterfaces на Router (show ip interface brief)
- Провери trunk между Router и Switch
- Провери gateway в DHCP pools

### Проблем: DHCP не работи
**Решение:**
- Провери дали PC е set на DHCP
- Виж DHCP bindings: `show ip dhcp binding`
- Провери excluded addresses

### Проблем: DNS не resolve-ва
**Решение:**
- Ping-ни DNS Server IP (10.10.50.10)
- Провери дали DNS е в DHCP options
- Провери A records на DNS Server

---

## ФИНАЛНА ПРОВЕРКА

Преди предаване на проекта, провери:

### Technical Functionality:
```
☐ Всички PC-та получават IP от DHCP
☐ Ping работи между всички VLAN-и
☐ DNS resolve-ва локални имена
☐ Internet достъп работи (NAT)
☐ ACL блокира Guest правилно
☐ SSH достъп работи
```

### Documentation:
```
☐ Логическа диаграма е професионална и четлива
☐ Физическа диаграма показва всички кабели
☐ Всички таблици са попълнени
☐ Конфигурациите са документирани
☐ Има тестови резултати (screenshots)
```

---

## КЛЮЧОВИ КОНЦЕПЦИИ

### Multi-Subnet Design:
- Всеки subnet служи за конкретна цел
- Правилно subnetting спестява IP адреси
- VLSM позволява flexibility

### Service Integration:
- DHCP автоматизира IP адресирането
- DNS улеснява достъпа (names vs IPs)
- NAT дава интернет достъп

### Security Layers:
- VLAN separation
- ACL filtering
- SSH encryption
- Port Security

---

## СЛЕДВАЩИ СТЪПКИ

Браво! Завърши си всички 9 лаба!

Сега си готов за **КУРСОВИЯ ПРОЕКТ**:
- Приложи всичко научено
- Създай собствен сценарий
- Изгради професионална мрежа
- Документирай детайлно

**Успех с проекта!**

---

**Запази .pkt файла като `Lab9_Integrated_Network_Design.pkt`**
