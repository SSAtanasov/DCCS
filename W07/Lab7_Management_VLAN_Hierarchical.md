# LAB 7: Management VLAN и йерархична топология (Multi-tier)

**Продължителност:** 90-120 минути  
**Цел:** Научаване на концепцията за Management VLAN за сигурно управление и изграждане на йерархична мрежа (Access/Distribution/Core)

---

## ЦЕЛИ НА УПРАЖНЕНИЕТО

След завършване на този лаб ще можете да:
- ✅ Създавате и конфигурирате Management VLAN
- ✅ Разбирате защо VLAN 1 е несигурен за management
- ✅ Изграждате йерархична топология (Access, Distribution, Core слоеве)
- ✅ Конфигурирате многослойна мрежа с правилни uplink-ове
- ✅ Прилагате best practices за мрежов дизайн

---

## ТЕОРЕТИЧНА ОСНОВА

### Management VLAN

**Какво е Management VLAN?**
- Отделен VLAN специално за управление на мрежовите устройства
- Съдържа само management IP адресите на switch-ове и router-и
- Изолиран от потребителския трафик
- Подобрява сигурността и производителността

**Защо НЕ трябва да използваме VLAN 1?**
- VLAN 1 е default VLAN на Cisco устройства
- Всички управленски протоколи (CDP, VTP, DTP) по подразбиране са във VLAN 1
- Атакуващ може лесно да се включи във VLAN 1
- Best practice: Създай отделен Management VLAN (обикновено 99)

### Йерархична топология (Hierarchical Design)

**Три слоя на йерархична мрежа:**

```
┌─────────────────────────────────────────┐
│         CORE LAYER                      │  ← Бързо switching, no routing
│  (High-speed backbone)                  │     политики, redundancy
└───────────┬──────────────┬──────────────┘
            │              │
┌───────────▼──────┐  ┌────▼──────────────┐
│ DISTRIBUTION     │  │ DISTRIBUTION      │  ← Routing, policies, 
│    LAYER         │  │    LAYER          │     aggregation
└────┬──────┬──────┘  └──────┬──────┬─────┘
     │      │                │      │
┌────▼──┐ ┌─▼────┐      ┌───▼──┐ ┌─▼─────┐
│ACCESS │ │ACCESS│      │ACCESS│ │ACCESS │  ← User connectivity,
│ SW1   │ │ SW2  │      │ SW3  │ │ SW4   │     VLAN assignment
└───────┘ └──────┘      └──────┘ └───────┘
    │         │              │         │
   PCs       PCs            PCs       PCs
```

**Access Layer:**
- Свързва крайните устройства (PC, принтери, IP телефони)
- VLAN assignment
- Port Security
- PoE (Power over Ethernet)

**Distribution Layer:**
- Агрегира access switch-ове
- Routing между VLAN-и
- QoS политики
- ACL filtering
- Redundancy

**Core Layer:**
- Високоскоростен backbone
- Минимални политики (само switching/routing)
- Максимална производителност
- 24/7 availability

---

## ТОПОЛОГИЯ

### Целева топология:

```
                    [CORE-SW]
                    (Core Layer)
                   10.99.99.10/24
                        │
            ┌───────────┴───────────┐
            │                       │
        [DIST-SW1]              [DIST-SW2]
    (Distribution Layer)    (Distribution Layer)
       10.99.99.11/24          10.99.99.12/24
            │                       │
     ┌──────┴──────┐         ┌──────┴──────┐
     │             │         │             │
[ACCESS-SW1] [ACCESS-SW2] [ACCESS-SW3] [ACCESS-SW4]
10.99.99.21  10.99.99.22  10.99.99.23  10.99.99.24
     │             │         │             │
   PCs           PCs       PCs           PCs
 VLAN 10       VLAN 20   VLAN 30       VLAN 40
```

**VLAN дизайн:**
- VLAN 10: Sales (192.168.10.0/24)
- VLAN 20: Engineering (192.168.20.0/24)
- VLAN 30: HR (192.168.30.0/24)
- VLAN 40: IT (192.168.40.0/24)
- **VLAN 99: Management** (10.99.99.0/24) ← Нов!

---

## СТЪПКА 1: Изграждане на топологията

### 1.1 Добави устройства

В Packet Tracer добави:
- **1x 2960-24TT Switch** (Core-SW)
- **2x 2960-24TT Switch** (Dist-SW1, Dist-SW2)
- **4x 2960-24TT Switch** (Access-SW1 до Access-SW4)
- **8x PC** (по 2 на всеки Access switch)

### 1.2 Кабелиране

**Core към Distribution:**
```
Core-SW Gi0/1 ↔ Dist-SW1 Gi0/1
Core-SW Gi0/2 ↔ Dist-SW2 Gi0/1
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

## СТЪПКА 2: Конфигуриране на Core Switch

### 2.1 Базова конфигурация

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

! VLAN 99 за Management
vlan 99
 name Management
 exit

! Management IP на Core
interface vlan 99
 ip address 10.99.99.10 255.255.255.0
 no shutdown
 exit

! Default gateway (ако има router)
! ip default-gateway 10.99.99.1

! Trunk портове към Distribution switches
interface range gigabitEthernet 0/1-2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! Запази конфигурацията
end
copy running-config startup-config
```

**Обяснение:**
- `switchport trunk native vlan 99` - Management трафик е untagged
- `trunk allowed vlan` - Позволява само нужните VLAN-и

---

## СТЪПКА 3: Конфигуриране на Distribution Switches

### 3.1 Distribution-SW1

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

! Management IP
interface vlan 99
 ip address 10.99.99.11 255.255.255.0
 no shutdown
 exit

! Trunk към Core
interface gigabitEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

! Trunk към Access switches
interface range fastEthernet 0/23-24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
 exit

end
copy running-config startup-config
```

### 3.2 Distribution-SW2

Повтори същата конфигурация, но промени:
- `hostname Dist-SW2`
- `interface vlan 99` → `ip address 10.99.99.12 255.255.255.0`
- `interface gigabitEthernet 0/1` → trunk към Core Gi0/2

---

## СТЪПКА 4: Конфигуриране на Access Switches

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

! Trunk към Distribution
interface gigabitEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,99
 no shutdown
 exit

! Access портове за PC-та
interface range fastEthernet 0/1-2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
 exit

! Изключи неизползваните портове (security best practice)
interface range fastEthernet 0/3-24
 shutdown
 exit

end
copy running-config startup-config
```

### 4.2 Access-SW2, SW3, SW4

Повтори за останалите 3 access switch-а, като промениш:

**Access-SW2 (VLAN 20):**
- `hostname Access-SW2`
- Management IP: `10.99.99.22`
- VLAN 20 instead of 10
- Trunk allowed: `vlan 20,99`

**Access-SW3 (VLAN 30):**
- `hostname Access-SW3`
- Management IP: `10.99.99.23`
- VLAN 30
- Trunk allowed: `vlan 30,99`
- Trunk от Dist-SW2

**Access-SW4 (VLAN 40):**
- `hostname Access-SW4`
- Management IP: `10.99.99.24`
- VLAN 40
- Trunk allowed: `vlan 40,99`
- Trunk от Dist-SW2

---

## СТЪПКА 5: Конфигуриране на PC-тата

**За тестване, конфигурирай PC-тата статично:**

| PC | VLAN | IP Address | Subnet Mask | Gateway |
|----|------|------------|-------------|---------|
| PC1 | 10 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC2 | 10 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC3 | 20 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC4 | 20 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |
| PC5 | 30 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| PC6 | 30 | 192.168.30.11 | 255.255.255.0 | 192.168.30.1 |
| PC7 | 40 | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 |
| PC8 | 40 | 192.168.40.11 | 255.255.255.0 | 192.168.40.1 |

---

## СТЪПКА 6: Тестване на Management VLAN

### 6.1 Провери Management IP-тата

От всеки switch:
```cisco
show interface vlan 99
show ip interface brief
```

**Очакван резултат:**
```
Interface         IP-Address      OK? Method Status                Protocol
Vlan99            10.99.99.21     YES manual up                    up
```

### 6.2 Ping тест между switch-ове

От Access-SW1:
```cisco
ping 10.99.99.10    ! Core-SW
ping 10.99.99.11    ! Dist-SW1
ping 10.99.99.22    ! Access-SW2
```

**Всички ping-ове трябва да са успешни!**

### 6.3 Провери VLAN assignment

```cisco
show vlan brief
```

Виж че VLAN 99 е създаден и активен.

### 6.4 Провери trunk портовете

```cisco
show interfaces trunk
```

**Очакван изход:**
```
Port        Mode         Encapsulation  Status        Native vlan
Gi0/1       on           802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/1       10,20,30,40,99
```

---

## СТЪПКА 7: Тестване на йерархичната топология

### 7.1 Traceroute test

От PC1 (VLAN 10) към Access-SW1:
```
C:\> tracert 10.99.99.21
```

**Забележка:** Понеже няма Layer 3 routing още, ping между различни VLAN-и няма да работи. Това е нормално на този етап.

### 7.2 Spanning Tree проверка

От всеки switch:
```cisco
show spanning-tree

! По-кратък изглед
show spanning-tree summary
```

**Очаквано:** Core-SW трябва да е Root Bridge

---

## СТЪПКА 8: Сигурност на Management VLAN

### 8.1 SSH вместо Telnet (Best Practice)

На всеки switch:
```cisco
configure terminal

! Име на домейн (нужно за SSH)
ip domain-name lab.local

! Генерирай RSA ключове
crypto key generate rsa
! Избери 1024 when prompted

! SSH версия 2
ip ssh version 2

! VTY lines за SSH
line vty 0 4
 transport input ssh
 login local
 exit

! Създай локален потребител
username admin privilege 15 secret admin123

end
copy running-config startup-config
```

### 8.2 Тествай SSH достъп

От PC (ако поддържа) или от друг switch:
```cisco
ssh -l admin 10.99.99.21
```

Въведи парола: `admin123`

---

## VERIFICATION CHECKLIST

### Management VLAN:
```
☐ VLAN 99 създаден на всички switch-ове
☐ Management IP адреси конфигурирани (10.99.99.x)
☐ Ping между всички switch-ове успешен
☐ VLAN 1 НЕ се използва за management
☐ Native VLAN е 99 на всички trunk-ове
```

### Йерархична топология:
```
☐ 3 слоя налични: Core, Distribution, Access
☐ Core-SW е Root Bridge в Spanning Tree
☐ Trunk-овете работят между слоевете
☐ Access switch-ове са leaf nodes (само към PC и Distribution)
☐ Всяка VLAN е assigned на правилния Access switch
```

### Сигурност:
```
☐ SSH конфигуриран на всички устройства
☐ Telnet disabled (transport input ssh)
☐ Неизползвани портове са shutdown
☐ Силни пароли използвани
```

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ

### Проблем 1: Не мога да ping-на други switch-ове

**Причина:** Native VLAN mismatch или Management VLAN не е allowed на trunk
**Решение:**
```cisco
! На всеки trunk port:
show interfaces trunk
! Виж Native VLAN и Allowed VLANs

! Ако е грешно:
interface gigabitEthernet 0/1
 switchport trunk native vlan 99
 switchport trunk allowed vlan add 99
```

### Проблем 2: SSH не работи

**Причина:** RSA keys не са генерирани или SSH version не е 2
**Решение:**
```cisco
show ip ssh
! Виж дали SSH е enabled

crypto key generate rsa
ip ssh version 2
```

### Проблем 3: Spanning Tree loop

**Причина:** Грешна топология или misconfiguration
**Решение:**
```cisco
show spanning-tree

! Виж Root Bridge:
! Should be Core-SW

! Ако не е, форсирай:
spanning-tree vlan 1-4094 priority 4096
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Redundancy
Добави втори линк между Core и всеки Distribution switch за redundancy.
Провери как Spanning Tree блокира един от портовете.

### Задача 2: Port Security на Management
Конфигурирай Port Security на trunk портовете за допълнителна защита.

### Задача 3: SNMP за мониторинг
Конфигурирай SNMP community string на Management VLAN за network monitoring.

---

## КЛЮЧОВИ КОНЦЕПЦИИ

### Management VLAN Best Practices:
1. ✅ Никога не използвай VLAN 1
2. ✅ Създай отделен Management VLAN (напр. 99)
3. ✅ Използвай отделен IP subnet (напр. 10.99.99.0/24)
4. ✅ Конфигурирай Native VLAN да е Management VLAN
5. ✅ Използвай SSH, не Telnet
6. ✅ Apply ACL за достъп само от админ IP-та

### Hierarchical Design Best Practices:
1. ✅ Ясно разделение на слоевете
2. ✅ Access слой само за end devices
3. ✅ Distribution за routing и политики
4. ✅ Core за high-speed switching
5. ✅ Redundancy между слоевете
6. ✅ Spanning Tree root на Core слой

---

## СЛЕДВАЩИ СТЪПКИ

Сега знаеш как да:
- Създаваш Management VLAN за сигурно управление
- Изграждаш йерархична топология (Access/Distribution/Core)
- Прилагаш best practices за enterprise networks

**Следващ лаб:** Lab 8 - NAT Configuration

---

**Готово! Запази .pkt файла като `Lab7_Management_VLAN_Hierarchical.pkt`**
