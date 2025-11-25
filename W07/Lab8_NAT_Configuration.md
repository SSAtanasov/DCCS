# LAB 8: NAT (Network Address Translation) конфигурация

**Продължителност:** 75-90 минути  
**Цел:** Научаване на NAT конфигурация за симулиране на връзка с интернет и споделяне на публични IP адреси  
**Prerequisite:** Завършени Lab 1-5 (особено Lab 5 за ACL)

---

## ЦЕЛИ НА УПРАЖНЕНИЕТО

След завършване на този лаб ще можете да:
- ✅ Разбирате какво е NAT и защо се използва
- ✅ Конфигурирате Static NAT (1:1 mapping)
- ✅ Конфигурирате Dynamic NAT (pool-based)
- ✅ Конфигурирате PAT (Port Address Translation) - най-често използван
- ✅ Симулирате реалистична връзка с интернет в Packet Tracer
- ✅ Troubleshoot NAT проблеми

---

## ТЕОРЕТИЧНА ОСНОВА

### Какво е NAT?

**Network Address Translation (NAT)** е техника за транслиране на private IP адреси към public IP адреси при комуникация с интернет.

**Защо е нужен NAT?**
- 📊 Липса на достатъчно IPv4 адреси
- 🔒 Сигурност (скрива вътрешната топология)
- 💰 Икономия (не е нужен публичен IP за всяко устройство)

### Типове NAT:

| Тип | Описание | Употреба |
|-----|----------|----------|
| **Static NAT** | 1:1 mapping (постоянно) | Сървъри (Web, Mail, FTP) |
| **Dynamic NAT** | Pool от IPs, first-come first-served | Рядко се използва |
| **PAT (Overload)** | Много устройства → 1 IP (чрез портове) | Най-често използван |

```
PAT пример:
192.168.1.10:54321 → 203.0.113.1:54321
192.168.1.11:54322 → 203.0.113.1:54322  ← СЪЩИЯ публичен IP!
192.168.1.12:54323 → 203.0.113.1:54323
```

---

## ТОПОЛОГИЯ

```
    ┌─────────────────────────────┐
    │   INTERNET SIMULATION       │
    │   ISP-Router (2911)         │
    │   - Loopback0: 8.8.8.8/32   │
    │   - Loopback1: 1.1.1.1/32   │
    └──────────┬──────────────────┘
               │ Se0/0/0 (DCE) 
               │ 209.165.200.1/30
               │ clock rate 64000
               │
               │ Se0/0/0 (DTE)
               │ 209.165.200.2/30
        ┌──────▼──────┐
        │   R1-NAT    │ (NAT Router 2911)
        └──────┬──────┘
               │ G0/0 (inside)
               │ 192.168.10.1/24
        ┌──────▼──────┐
        │    SW1      │ (Switch 2960-24TT)
        └─┬────────┬──┘
          │        │
    ┌─────▼──┐  ┌──▼──────┐
    │  PC1   │  │ Server1 │
    │ .10.10 │  │ .10.50  │
    └────────┘  └─────────┘
```

### Защо ISP Router вместо Cloud?

- ✅ Реалистична симулация на ISP
- ✅ Loopback интерфейси за симулация на интернет услуги (8.8.8.8, 1.1.1.1)
- ✅ Пълна routing конфигурация
- ✅ Compatibility с Lab 9

---

## ЧАСТ 0: Изграждане на ISP инфраструктура

### СТЪПКА 0: Създаване на топологията

**Устройства:**
- **2x Router 2911** (ISP-Router и R1-NAT)
- **1x Switch 2960-24TT** (SW1)
- **2x PC** (PC1, Server1)

**Кабелиране:**
```
ISP-Router Se0/0/0 (DCE) ↔ R1-NAT Se0/0/0 (DTE)
R1-NAT Gi0/0 ↔ SW1 Gi0/1
SW1 Fa0/1 ↔ PC1
SW1 Fa0/2 ↔ Server1
```

**ВАЖНО:** ISP-Router Se0/0/0 е **DCE** (има `clock rate 64000`)

---

### СТЪПКА 0.1: Конфигурация на ISP Router

```cisco
enable
configure terminal
hostname ISP-Router
no ip domain-lookup

! Loopback интерфейси (симулират интернет)
interface loopback 0
 ip address 8.8.8.8 255.255.255.255
 exit

interface loopback 1
 ip address 1.1.1.1 255.255.255.255
 exit

! WAN интерфейс (DCE страна)
interface serial 0/0/0
 ip address 209.165.200.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! Static route към customer network
ip route 203.0.113.0 255.255.255.0 209.165.200.2

end
copy running-config startup-config
```

---

## ЧАСТ 1: Static NAT (за сървъри)

### СТЪПКА 1: Базова конфигурация на R1-NAT Router

```cisco
enable
configure terminal
hostname R1-NAT
no ip domain-lookup

! Inside interface
interface gigabitEthernet 0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

! Outside interface (DTE страна - БЕЗ clock rate!)
interface serial 0/0/0
 ip address 209.165.200.2 255.255.255.252
 no shutdown
 exit

! Default route към ISP
ip route 0.0.0.0 0.0.0.0 209.165.200.1

end
copy running-config startup-config
```

### СТЪПКА 2: Проверка на connectivity

```cisco
ping 209.165.200.1
ping 8.8.8.8
```
✅ Ако работи, WAN връзката е правилна!

---

### СТЪПКА 3: Конфигуриране на Static NAT

**Сценарий:** Server1 (192.168.10.50) → публичен IP 203.0.113.50

```cisco
configure terminal

! Маркирай inside/outside интерфейси
interface gigabitEthernet 0/0
 ip nat inside
 exit

interface serial 0/0/0
 ip nat outside
 exit

! Static NAT mapping
ip nat inside source static 192.168.10.50 203.0.113.50

end
copy running-config startup-config
```

### СТЪПКА 4: Тестване

**Конфигурирай Server1:** IP: 192.168.10.50, Gateway: 192.168.10.1

```cisco
show ip nat translations

! Очакван изход:
Pro Inside global      Inside local       Outside local      Outside global
--- 203.0.113.50       192.168.10.50      ---                ---
```

---

## ЧАСТ 2: Dynamic NAT (pool-based)

### СТЪПКА 5: Конфигуриране на Dynamic NAT

**Сценарий:** Pool от 10 публични IP адреса (203.0.113.10-19)

```cisco
configure terminal

! ACL за inside addresses
access-list 1 permit 192.168.10.0 0.0.0.255

! NAT pool
ip nat pool PUBLIC_POOL 203.0.113.10 203.0.113.19 netmask 255.255.255.0

! Свържи ACL с pool
ip nat inside source list 1 pool PUBLIC_POOL

end
```

**Проблем:** Ако имаш 11+ устройства, но само 10 IPs → последните нямат достъп!  
**Решение:** Използвай PAT (следващата секция)

---

## ЧАСТ 3: PAT (NAT Overload) - Най-често използван

### СТЪПКА 6: Конфигуриране на PAT

**Сценарий:** Един публичен IP за всички вътрешни устройства

```cisco
configure terminal

! Премахни предишните NAT конфигурации
no ip nat inside source list 1 pool PUBLIC_POOL
no ip nat pool PUBLIC_POOL

! PAT using outside interface IP
ip nat inside source list 1 interface serial 0/0/0 overload

end
copy running-config startup-config
```

**Ключова дума:** `overload` - това е PAT!

### СТЪПКА 7: Тестване на PAT

От PC1: `ping 8.8.8.8`  
От Server1: `ping 1.1.1.1`

```cisco
show ip nat translations

! Очакван изход:
Pro Inside global           Inside local          Outside local         Outside global
icmp 209.165.200.2:1        192.168.10.10:1       8.8.8.8:1             8.8.8.8:1
icmp 209.165.200.2:2        192.168.10.50:1       1.1.1.1:2             1.1.1.1:2
     ^^^^^^^^^^^^^^         ^^^^^^^^^^^^^^^^
     СЪЩИЯТ публичен IP!    РАЗЛИЧНИ private IPs!
```

---

## ЧАСТ 4: Комбиниране на Static NAT + PAT

### СТЪПКА 8: Real-World сценарий

**Изисквания:**
1. Web Server (192.168.10.50) достъпен отвън на 203.0.113.50
2. Всички служители имат интернет чрез PAT

```cisco
configure terminal

! 1. Static NAT за Web Server
ip nat inside source static 192.168.10.50 203.0.113.50

! 2. ACL за служителите (exclude Web Server)
access-list 1 deny   192.168.10.50
access-list 1 permit 192.168.10.0 0.0.0.255

! 3. PAT за служителите
ip nat inside source list 1 interface serial 0/0/0 overload

end
copy running-config startup-config
```

---

## ЧАСТ 5: Port Forwarding (Static PAT)

### СТЪПКА 9: Конфигуриране на Port Forwarding

**Сценарий:** Web server на порт 8080 вътрешно → порт 80 външно

```cisco
ip nat inside source static tcp 192.168.10.50 8080 203.0.113.50 80 extendable
```

**Примери:**
```cisco
! SSH
ip nat inside source static tcp 192.168.10.50 22 203.0.113.50 22

! RDP
ip nat inside source static tcp 192.168.10.51 3389 203.0.113.50 3389
```

---

## VERIFICATION COMMANDS

```cisco
! Виж NAT translations
show ip nat translations

! Статистики
show ip nat statistics

! Виж ACL
show access-lists

! NAT в running-config
show running-config | include nat

! Дебъг (внимавай - много output!)
debug ip nat
undebug all

! Изчисти translations
clear ip nat translation *
```

---

## VERIFICATION CHECKLIST

### ✅ Static NAT:
```
☐ ip nat inside/outside са зададени
☐ Static mapping е конфигуриран
☐ Ping от ISP към public IP работи
☐ show ip nat translations показва mapping
```

### ✅ PAT (Overload):
```
☐ ACL permit вътрешни адреси
☐ "overload" keyword е използван
☐ Множество устройства ping интернет едновременно
☐ show ip nat translations показва различни портове
```

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ

### Проблем 1: NAT не работи

**Причина:** Inside/Outside интерфейси не са конфигурирани

**Решение:**
```cisco
interface gigabitEthernet 0/0
 ip nat inside
interface serial 0/0/0
 ip nat outside
```

### Проблем 2: Няма translations

**Причина:** ACL не permit-ва трафика

**Решение:**
```cisco
show access-lists
! Провери дали ACL permit-ва правилния subnet
access-list 1 permit 192.168.10.0 0.0.0.255
```

### Проблем 3: Serial интерфейсът е down

**Причина:** Липсва clock rate на DCE страната

**Решение:**
```cisco
! На ISP-Router (DCE страна):
interface serial 0/0/0
 clock rate 64000
 no shutdown
```

### Проблем 4: Dynamic NAT pool е пълен

**Решение:** Използвай PAT (overload)
```cisco
ip nat inside source list 1 pool PUBLIC_POOL overload
```

---

## TROUBLESHOOTING WORKFLOW

```
1. Провери интерфейси: show ip interface brief
   └─ Всички up/up?

2. Провери ip nat inside/outside
   └─ show ip interface Gi0/0 | include NAT

3. Провери ACL: show access-lists
   └─ permit правилния subnet?

4. Провери NAT config: show run | include nat
   └─ Има ли "overload" за PAT?

5. Провери routing: show ip route
   └─ Има ли default route?

6. Debug: debug ip nat
   └─ Виждаш ли translations?
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Port Forwarding за multiple services

Конфигурирай Port Forwarding за:
- Web Server (HTTP): 192.168.10.50:80 → 203.0.113.1:80
- SSH Server: 192.168.10.50:22 → 203.0.113.1:22

### Задача 2: Multiple Static NAT mappings

Създай Static NAT за:
- Web Server: 192.168.10.50 → 203.0.113.50
- Mail Server: 192.168.10.51 → 203.0.113.51

Добави PAT за останалите устройства и тествай.

---

## РЕАЛЕН СЦЕНАРИЙ: Компания със собствен Web сървър

**Инфраструктура:**
- 3 публични IP адреса (203.0.113.10-12)
- Web Server: 192.168.1.10
- Mail Server: 192.168.1.11
- 100 служители

**Решение:**
```cisco
! Static NAT за сървъри
ip nat inside source static 192.168.1.10 203.0.113.10
ip nat inside source static 192.168.1.11 203.0.113.11

! PAT за служители (exclude сървърите)
access-list 1 deny   192.168.1.10
access-list 1 deny   192.168.1.11
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

---

## NAT ТЕРМИНОЛОГИЯ

```
┌─────────────────────────────────────────────────────────┐
│                    NAT ROUTER                           │
│                                                         │
│  Inside Local ──────NAT──────► Inside Global           │
│  (Private IP)                  (Public IP)             │
│  192.168.10.10 ──────────────► 203.0.113.10            │
└─────────────────────────────────────────────────────────┘
```

- **Inside Local:** Private IP в локалната мрежа
- **Inside Global:** Public IP след NAT
- **Outside Global:** Действителният IP на външното устройство

---

## BEST PRACTICES

1. ✅ **Използвай PAT за повечето случаи** - икономия на IPs
2. ✅ **Static NAT само за сървъри** - когато трябва входяща връзка
3. ✅ **ACL трябва да е конкретен** - не `permit any`
4. ✅ **Документирай NAT конфигурацията**
5. ✅ **NAT не е firewall!** - използвай ACL за сигурност

---

## ВРЪЗКА С LAB 9

Този лаб е подготовка за Lab 9, където ще:
- Използваш същата ISP Router структура
- Конфигурираш PAT за множество VLANs
- Комбинираш NAT с Inter-VLAN routing и ACL

---

## КАКВО НАУЧИХМЕ

1. ✅ Какво е NAT и типовете (Static, Dynamic, PAT)
2. ✅ Конфигурация на ISP Router за симулация
3. ✅ Static NAT за сървъри
4. ✅ PAT (Overload) за споделяне на IP
5. ✅ Port Forwarding (Static PAT)
6. ✅ Комбиниране на Static NAT + PAT
7. ✅ Troubleshooting NAT проблеми

---

**Запази .pkt файла като `Lab8_NAT_Configuration.pkt`**

**ВАЖНО:** Този лаб използва ISP Router вместо Cloud устройство за реалистична симулация.


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
