# LAB 8: NAT (Network Address Translation) КОНФИГУРАЦИЯ
## Симулиране на интернет свързаност

**Продължителност:** 90-120 минути  
**Цел:** Научаване на NAT конфигурация за симулиране на връзка с интернет и споделяне на публични IP адреси  
**Prerequisite:** Завършени Lab 1-5 (особено Lab 5 за ACL)

> **🔗 ВРЪЗКА С LAB 7 И LAB 9:**
> Този лаб се изпълнява **независимо** в нов празен файл (отделна топология).
> В **Lab 9** изграждате нова интегрирана мрежа (10.10.x.x). WAN частта там е същата като тук: ISP-Router, 209.165.200.0/30 и симулираните адреси 8.8.8.8 и 1.1.1.1.

> **ЗА КУРСОВИЯ ПРОЕКТ:**
> NAT е **ОПЦИОНАЛЕН** за КП – задължителен е само ако сценарият включва симулирана връзка с интернет (кратко въведение има в Lab 5, Част 9). Lab 7–9 са надграждащи и не са задължителни за проекта. NAT демонстрира:
> - Реалистична enterprise мрежа с интернет достъп
> - Professional network design skills
> - Разбиране на WAN connectivity

---
## ТЕОРЕТИЧНА ОСНОВА (РАЗШИРЕНА)

### Какво е NAT?

**Network Address Translation (NAT)** е механизъм за **преобразуване на IP адреси** при преминаване на пакети между мрежи с различни адресни пространства.

**Проста аналогия:**
```
NAT е като портиер на хотел:

Вие (192.168.1.10) искате да изпратите писмо
↓
Портиерът (NAT Router) го взема и пише своя адрес като изпращач
↓
Писмото излиза с адрес на хотела (203.0.113.1)
↓
Отговорът се връща на хотела
↓
Портиерът го препраща на вас в стая 192.168.1.10
```

---

### Защо е нужен NAT? (IPv4 Address Exhaustion)

**Историята на IPv4 кризата:**

IPv4 предлага **4.3 милиарда адреса** (2³² = 4,294,967,296)

```
1980s: "4 милиарда адреса ще стигнат завинаги!"
1990s: "Хм, може би не..."
2011: IANA обяви ИЗЧЕРПВАНЕ на IPv4 адресите
Днес: Все още използваме IPv4 с NAT като временно решение
```

**Без NAT:**
- Всяко устройство трябва да има публичен IP адрес
- Компания със 500 компютъра = 500 публични IP адреса
- Домакинство с 10 устройства = 10 публични IP адреса
- **НЕВЪЗМОЖНО** - няма толкова адреси!

**С NAT:**
- Цяла компания споделя 1-5 публични IP адреса
- Домакинството споделя 1 публичен IP
- **РАБОТИ** - PAT използва портове за разграничаване

---

### Private vs Public IP адреси

**RFC 1918 дефинира PRIVATE IP ranges:**

| Клас | Private Range | Subnet | Брой адреси |
|------|---------------|--------|-------------|
| Клас A | 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 | 16,777,216 |
| Клас B | 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 | 1,048,576 |
| Клас C | 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | 65,536 |

**Правила за private IPs:**
- ✅ **Може** да се използват вътре в локална мрежа (LAN)
- ❌ **НЕ може** да се рутират в интернет
- ✅ **Може** да се повтарят в различни мрежи (всеки има 192.168.1.1)
- 🔄 **Изискват NAT** за интернет комуникация

**Публични IP адреси:**
- Уникални в целия интернет
- Трябва да се закупят/наемат от ISP
- Скъпи и ограничени
- Необходими за услуги достъпни от интернет

**Забележка:** 203.0.113.0/24 (TEST-NET-3), 198.51.100.0/24 и 192.0.2.0/24 са запазени за документация и учебни примери (RFC 5737); в упражнението ги използваме като „публични“ адреси. 8.8.8.8 и 1.1.1.1 са loopback адреси на ISP-Router: симулират достижимост до външна мрежа, но не предоставят DNS услуга.

---

### Как работи NAT? (Процес стъпка по стъпка)

**Сценарий:** PC1 (192.168.10.10) иска да достъпи Google (8.8.8.8)

```
СТЪПКА 1: PC изпраща пакет
┌────────────────────────────────┐
│ Source: 192.168.10.10:54321   │ ← Private IP + random port
│ Dest:   8.8.8.8:80            │
└────────────────────────────────┘
        │
        ↓
    [NAT Router]
        │
        ↓
СТЪПКА 2: NAT Router транслира
┌────────────────────────────────┐
│ Source: 203.0.113.1:54321     │ ← Public IP + същия/нов port
│ Dest:   8.8.8.8:80            │
└────────────────────────────────┘
        │
        ↓
    [INTERNET]
        │
        ↓
СТЪПКА 3: Google отговаря
┌────────────────────────────────┐
│ Source: 8.8.8.8:80            │
│ Dest:   203.0.113.1:54321     │ ← Публичен IP на Router-а
└────────────────────────────────┘
        │
        ↓
    [NAT Router]
        │
        ↓ (проверява NAT таблицата: 54321 → 192.168.10.10:54321)
        │
СТЪПКА 4: Router препраща към PC
┌────────────────────────────────┐
│ Source: 8.8.8.8:80            │
│ Dest:   192.168.10.10:54321   │ ← Обратно към private IP
└────────────────────────────────┘
```

**NAT Translation Table:**
```
Inside Local      Inside Global      Outside Global
192.168.10.10:54321 → 203.0.113.1:54321 → 8.8.8.8:80
192.168.10.11:54322 → 203.0.113.1:54322 → 8.8.8.8:80
192.168.10.12:54323 → 203.0.113.1:54323 → 1.1.1.1:443
```

---

### NAT Терминология (Важно!)

| Термин | Значение | Пример |
|--------|----------|--------|
| **Inside Local** | Private IP адрес на устройството | 192.168.10.10 |
| **Inside Global** | Public IP адрес след NAT | 203.0.113.1 |
| **Outside Global** | Public IP адрес на external host | 8.8.8.8 |
| **Outside Local** | Как external host се вижда от inside | 8.8.8.8 (обикновено същото) |

**Inside vs Outside:**
- **Inside** = вашата локална мрежа (LAN)
- **Outside** = интернет (WAN)

**Local vs Global:**
- **Local** = преди NAT translation
- **Global** = след NAT translation

---

### Типове NAT (Детайлно)

#### 1. Static NAT (1:1 Permanent Mapping)

```
Преди NAT:                     След NAT:
192.168.10.50 (Web Server) → 203.0.113.10 (винаги!)
192.168.10.51 (Mail Server) → 203.0.113.11 (винаги!)
```

**Характеристики:**
- Постоянно mapping (не се променя)
- 1 private IP → 1 public IP
- Винаги активно (дори без трафик)

**Кога се използва:**
- ✅ Web Servers (трябва постоянен публичен адрес)
- ✅ Mail Servers
- ✅ FTP Servers
- ✅ Всяко устройство което трябва да е достъпно от интернет

**Конфигурация:**
```cisco
ip nat inside source static 192.168.10.50 203.0.113.10
```

---

#### 2. Dynamic NAT (Pool-Based)

```
Pool от публични IPs: 203.0.113.10 - 203.0.113.15 (6 IPs)

Първи клиент:  192.168.10.10 → 203.0.113.10  (вземе първия)
Втори клиент:  192.168.10.11 → 203.0.113.11  (вземе втория)
...
Седми клиент:  192.168.10.16 → ❌ NO IP AVAILABLE!
```

**Характеристики:**
- Временно mapping (само докато има трафик)
- First-come, first-served
- Ако pool-ът е пълен → новите заявки се отхвърлят

**Проблеми:**
- ❌ Изисква много публични IP адреси
- ❌ Waste на адреси (1:1 mapping)
- ❌ Ако pool е пълен → никой нов не може да излезе

**Рядко се използва** в практиката!

**Конфигурация:**
```cisco
ip nat pool MY_POOL 203.0.113.10 203.0.113.15 netmask 255.255.255.0
access-list 1 permit 192.168.10.0 0.0.0.255
ip nat inside source list 1 pool MY_POOL
```

---

#### 3. PAT (Port Address Translation) - Overload

**Най-често използваният метод!**

```
ЕДИН публичен IP → хиляди устройства

192.168.10.10:54321 → 203.0.113.1:54321
192.168.10.11:54322 → 203.0.113.1:54322  ← СЪЩИЯ IP!
192.168.10.12:54323 → 203.0.113.1:54323  ← СЪЩИЯ IP!
...
192.168.10.100:60000 → 203.0.113.1:60000 ← СЪЩИЯ IP!
```

**Как работи портовете:**
- Всеки TCP/UDP комуникация използва **Source Port** (случаен номер 49152-65535)
- NAT Router запомня: "Source Port 54321 → PC1"
- Може да поддържа **65,000+ едновременни връзки** с 1 публичен IP!

**Характеристики:**
-  Много устройства → 1 публичен IP
-  Използва портове за разграничаване
-  Най-ефективен метод

**Кога се използва:**
- ✅ Домашна мрежа (всички устройства споделят 1 IP)
- ✅ Малки офиси
- ✅ Средни компании (десетки/стотици PC-та → 1-5 публични IPs)

**Конфигурация:**
```cisco
access-list 1 permit 192.168.10.0 0.0.0.255
ip nat inside source list 1 interface Serial0/0/0 overload
```

Ключовата дума **`overload`** активира PAT!

---

#### 4. Port Forwarding (Static PAT)

**Комбинира Static NAT с PAT**

```
Интернет клиент иска: http://203.0.113.1:8080
                               ↓
                        NAT Router препраща
                               ↓
                      Web Server 192.168.10.50:80
```

**Употреба:**
- ✅ Hosting на множество услуги с 1 публичен IP
- ✅ Port 8080 → Internal Web Server (port 80)
- ✅ Port 2222 → Internal SSH Server (port 22)
- ✅ Port 3389 → Internal RDP Server (port 3389)

**Конфигурация:**
```cisco
ip nat inside source static tcp 192.168.10.50 80 203.0.113.1 8080
```

---

### Защо е важен NAT? (Обобщение)

✅ **IPv4 Conservation** - позволява милиарди устройства с ограничени IPs  
✅ **Сигурност** - скрива вътрешната топология  
✅ **Гъвкавост** - лесна промяна на ISP без re-IP  
✅ **Икономия** - не е нужно да купувате много публични IPs  
❌ **Забавя трафика** - NAT translation overhead  
❌ **Усложнява troubleshooting** - hidden layer  
❌ **Проблеми с P2P** - изисква port forwarding  

**В бъдеще:** IPv6 ще елиминира нуждата от NAT (340 undecillion адреси!)

---

## ТОПОЛОГИЯ

```
                [Outside-PC] 198.51.100.10/24 (външен клиент)
                     │
                     │ Gi0/0: 198.51.100.1/24
    ┌────────────────┴────────────┐
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
        └─┬─────┬───┬─┘
          │     │   └───────┐
    ┌─────▼──┐ ┌▼───────┐ ┌─▼───────┐
    │  PC1   │ │  PC2   │ │ Server1 │
    │ .10.10 │ │ .10.11 │ │ .10.50  │
    └────────┘ └────────┘ └─────────┘
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
- **2x Router 2911** (ISP-Router и R1-NAT) – с модул **HWIC-2T**
- **1x Switch 2960-24TT** (SW1)
- **2x PC** (PC1, PC2) и **1x Server-PT** (Server1) във вътрешната мрежа
- **1x PC** (Outside-PC) – външен клиент зад ISP-Router

**Serial модул (задължително):** Router 2911 в Packet Tracer няма Serial портове по подразбиране. На **всеки** от двата рутера: Physical → изключете захранването → плъзнете **HWIC-2T** в слот HWIC 0 → включете. Портовете стават Serial0/0/0 и Serial0/0/1 (проверка: `show ip interface brief`).

**Кабелиране:**
```
ISP-Router Se0/0/0 (DCE) ↔ R1-NAT Se0/0/0 (DTE)
ISP-Router Gi0/0 ↔ Outside-PC (Copper Cross-Over)
R1-NAT Gi0/0 ↔ SW1 Gi0/1
SW1 Fa0/1 ↔ PC1
SW1 Fa0/2 ↔ Server1
SW1 Fa0/3 ↔ PC2
```

**ВАЖНО:** ISP-Router Se0/0/0 е **DCE** (има `clock rate 64000`). При кабела **Serial DCE** краят, който свържете първи, става DCE – започнете от ISP-Router.

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

! LAN към външния клиент (Outside-PC)
interface gigabitEthernet 0/0
 ip address 198.51.100.1 255.255.255.0
 no shutdown
 exit

! Static route към публичния блок на клиента (NAT адресите)
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

### СТЪПКА 1А: IP адреси на крайните устройства

| Устройство | IP адрес | Маска | Gateway |
|------------|----------|-------|---------|
| PC1 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC2 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| Server1 (Server-PT, Services → HTTP **ON**) | 192.168.10.50 | 255.255.255.0 | 192.168.10.1 |
| Outside-PC | 198.51.100.10 | 255.255.255.0 | 198.51.100.1 |

**Преди NAT:** от Outside-PC `ping 192.168.10.50` е неуспешен – ISP-Router няма (и не трябва да има) маршрут към частни адреси.

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

! Маркиране на inside/outside интерфейсите
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

**Тест от външната страна (Outside-PC):**
- `ping 203.0.113.50` → успешен
- Web Browser → `http://203.0.113.50` → страницата на Server1

```cisco
show ip nat translations

! Очакван изход (след теста се появяват и редове icmp/tcp):
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

! Свързване на ACL с pool-а
ip nat inside source list 1 pool PUBLIC_POOL

end
```

**Тест:** от PC1 и PC2 `ping 8.8.8.8`, после:

```cisco
show ip nat translations

! Очаквано: PC1 и PC2 получават различни адреси от pool-а
! (напр. 203.0.113.10 и 203.0.113.11); Server1 остава 203.0.113.50 (Static NAT от Част 1)
```

**Проблем:** Ако има 11+ устройства, но само 10 IPs → последните нямат достъп!
**Решение:** Използвайте PAT (следващата секция)

---

## ЧАСТ 3: PAT (NAT Overload) - Най-често използван

### СТЪПКА 6: Конфигуриране на PAT

**Сценарий:** Един публичен IP за всички вътрешни устройства

```cisco
! Изчистване на активните преводи (privileged EXEC)
clear ip nat translation *

configure terminal

! Премахване на Dynamic NAT
no ip nat inside source list 1 pool PUBLIC_POOL
no ip nat pool PUBLIC_POOL

! PAT using outside interface IP
ip nat inside source list 1 interface serial 0/0/0 overload

end
copy running-config startup-config
```

**Ключова дума:** `overload` - това е PAT!

**Бележка:** Static NAT за Server1 от Част 1 остава активен. Затова Server1 продължава да излиза като 203.0.113.50, а не през PAT.

### СТЪПКА 7: Тестване на PAT

От PC1: `ping 8.8.8.8`
От PC2: `ping 1.1.1.1`

```cisco
show ip nat translations

! Очакван изход:
Pro  Inside global          Inside local          Outside local         Outside global
icmp 209.165.200.2:1        192.168.10.10:1       8.8.8.8:1             8.8.8.8:1
icmp 209.165.200.2:2        192.168.10.11:1       1.1.1.1:2             1.1.1.1:2
---  203.0.113.50           192.168.10.50         ---                   ---
     ^^^^^^^^^^^^^^         ^^^^^^^^^^^^^^^^
     СЪЩИЯТ публичен IP!    РАЗЛИЧНИ private IPs!
```

Номерата след „:“ (ICMP идентификатори и портове) може да се различават от примера.

---

## ЧАСТ 4: Комбиниране на Static NAT + PAT

### СТЪПКА 8: Real-World сценарий

**Изисквания:**
1. Web Server (192.168.10.50) достъпен отвън на 203.0.113.50
2. Всички служители имат интернет чрез PAT

```cisco
configure terminal

! 1. Static NAT за Web Server (вече зададен в Част 1 - проверете с show running-config)
ip nat inside source static 192.168.10.50 203.0.113.50

! 2. ACL за служителите (exclude Web Server)
!    ACL 1 вече съществува от Част 2, затова първо се изтрива
no access-list 1
access-list 1 deny   host 192.168.10.50
access-list 1 permit 192.168.10.0 0.0.0.255

! 3. PAT за служителите
ip nat inside source list 1 interface serial 0/0/0 overload

end
copy running-config startup-config
```

**Защо `no access-list 1`?** Редът на правилата е важен. Ако добавите `deny` към съществуващия ACL 1, той застава **след** `permit 192.168.10.0` и никога не се изпълнява.

**Бележки:**
- ACL 1 тук само избира адресите за NAT; не филтрира трафика (не е приложен с `ip access-group`)
- Статичният превод има предимство и без `deny`, но изричното изключване прави политиката ясна

**Тест:** PC1 `ping 8.8.8.8` → излиза като 209.165.200.2; Outside-PC `ping 203.0.113.50` → успешен.

---

## ЧАСТ 5: Port Forwarding (Static PAT)

### СТЪПКА 9: Конфигуриране на Port Forwarding

**Сценарий:** вместо пълния 1:1 Static NAT, отвън се отваря **само HTTP** (TCP 80) на Server1.

```cisco
! Изчистване на активните преводи (privileged EXEC)
clear ip nat translation *

configure terminal

! 1. Премахване на пълния Static NAT (иначе влиза в конфликт със static PAT за същия адрес)
no ip nat inside source static 192.168.10.50 203.0.113.50

! 2. Static PAT: 203.0.113.50:80 → 192.168.10.50:80
ip nat inside source static tcp 192.168.10.50 80 203.0.113.50 80

end
```

**Тест от Outside-PC:**
- Web Browser → `http://203.0.113.50` → **работи**
- `ping 203.0.113.50` → **неуспешен** (препраща се само TCP 80)

`ping` не проверява port forwarding – нужна е HTTP заявка от външен клиент.

**Външен порт 8080 → вътрешен 80 (както в теорията):** първо премахнете правилото за порт 80 – един вътрешен адрес и порт не може да се свърже с два външни порта (без `extendable`):
```cisco
no ip nat inside source static tcp 192.168.10.50 80 203.0.113.50 80
ip nat inside source static tcp 192.168.10.50 80 203.0.113.50 8080
```
В браузъра: `http://203.0.113.50:8080` (проверете дали браузърът на Packet Tracer приема порт в адреса).

**Примери (само за илюстрация – Server-PT няма SSH и RDP услуга; не ги въвеждайте в тази топология):**
```cisco
! SSH
ip nat inside source static tcp 192.168.10.50 22 203.0.113.50 22

! RDP
ip nat inside source static tcp 192.168.10.51 3389 203.0.113.50 3389
```

---

## VERIFICATION COMMANDS

```cisco
! NAT translations
show ip nat translations

! Статистики
show ip nat statistics

! ACL
show access-lists

! NAT в running-config
show running-config | include nat

! Дебъг (внимание - много output!)
debug ip nat
undebug all

! Изчистване на translations
clear ip nat translation *
```

---

## VERIFICATION CHECKLIST

### ✅ Static NAT:
```
☐ ip nat inside/outside са зададени
☐ Static mapping е конфигуриран
☐ Ping от Outside-PC към 203.0.113.50 работи
☐ show ip nat translations показва mapping
```

### ✅ PAT (Overload):
```
☐ ACL permit вътрешни адреси (deny за сървъра е преди permit)
☐ "overload" keyword е използван
☐ Множество устройства ping интернет едновременно
☐ show ip nat translations показва различни портове
```

### ✅ Port Forwarding:
```
☐ HTTP от Outside-PC към 203.0.113.50 работи
☐ Ping към 203.0.113.50 не работи (отворен е само TCP 80)
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
! Проверете дали ACL permit-ва правилния subnet
access-list 1 permit 192.168.10.0 0.0.0.255
```

### Проблем 3: Serial интерфейсът липсва или е down

**Причина:** Липсва модул HWIC-2T (няма Serial0/0/0) или липсва clock rate на DCE страната

**Решение:**
```cisco
! На ISP-Router (DCE страна):
interface serial 0/0/0
 clock rate 64000
 no shutdown
```

### Проблем 4: Dynamic NAT pool е пълен

**Решение:** Използвайте PAT (overload)
```cisco
ip nat inside source list 1 pool PUBLIC_POOL overload
```

### Проблем 5: Outside-PC не достига 203.0.113.50

**Причина:** ISP-Router няма маршрут към публичния блок на клиента или Static NAT е премахнат (Част 5)

**Решение:**
```cisco
ISP-Router# show ip route
! Трябва да има: S 203.0.113.0/24 [1/0] via 209.165.200.2
R1-NAT# show ip nat translations
```

---

## TROUBLESHOOTING WORKFLOW

```
1. Проверете интерфейсите: show ip interface brief
   └─ Всички up/up? Има ли Serial0/0/0 (модул HWIC-2T)?

2. Проверете ip nat inside/outside
   └─ show ip interface Gi0/0 | include NAT

3. Проверете ACL: show access-lists
   └─ permit правилния subnet? deny преди permit?

4. Проверете NAT config: show run | include nat
   └─ Има ли "overload" за PAT?

5. Проверете routing: show ip route
   └─ R1: има ли default route? ISP: има ли маршрут към 203.0.113.0/24?

6. Debug: debug ip nat
   └─ Виждате ли translations?
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

Изпълнявайте задачите в отделен файл (`Lab8_Extra_YourName.pkt`), за да не се смесват с NAT правилата от основната част.

### Задача 1: Port Forwarding за multiple services

Конфигурирайте Port Forwarding за:
- Web Server (HTTP): 192.168.10.50:80 → 203.0.113.1:80
- Web Server (HTTPS): 192.168.10.50:443 → 203.0.113.1:443 (Server-PT няма SSH услуга)

Тест от Outside-PC: браузър към `http://203.0.113.1` и `https://203.0.113.1`.

**Бележка:** Ако второто правило за същия вътрешен и външен адрес бъде отказано, добавете `extendable` в края на двете команди (в реален IOS ключовата дума често се добавя автоматично).

### Задача 2: Multiple Static NAT mappings

Създайте Static NAT за:
- Web Server: 192.168.10.50 → 203.0.113.50
- Mail Server: 192.168.10.51 → 203.0.113.51 (добавете втори Server-PT)

Добавете PAT за останалите устройства и тествайте.

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

1. ✅ **Използвайте PAT за повечето случаи** - икономия на IPs
2. ✅ **Static NAT само за сървъри** - когато трябва входяща връзка
3. ✅ **ACL трябва да е конкретен** - не `permit any`
4. ✅ **Документирайте NAT конфигурацията**
5. ✅ **NAT не е firewall!** - използвайте ACL за сигурност

---

## ВРЪЗКА С LAB 9

> **🔗 ПРОДЪЛЖЕНИЕ В LAB 9:**

В **Lab 9: Integrated Network Design** ще:
- Изградите нова интегрирана мрежа (10.10.x.x) с ISP-Router като в Lab 8
- Конфигурирате PAT за всички потребителски VLAN-и
- Комбинирате NAT с Inter-VLAN routing, DHCP, DNS, ACL
- Създадете професионална enterprise мрежа с интернет достъп

**WAN частта от Lab 8 (ISP-Router, Serial връзка, 8.8.8.8) се използва отново в Lab 9; вътрешната мрежа е нова.**

> **ЦЯЛОСТНА ПРОГРЕСИЯ:**
> - Lab 1 → Основи (Router, Switch, базова конфигурация, IP адресиране)
> - Lab 2 → VLAN 10, 20 и Management VLAN 99, Router-on-a-Stick, втори комутатор, subnetting
> - Lab 3 → DHCP (pools, excluded addresses)
> - Lab 4 → DNS (A и CNAME записи, name resolution)
> - Lab 5 → ACL, SSH и Port Security (+ NAT като BONUS)
> - Lab 6 → IoT интеграция
> - Lab 7 → Йерархична топология с L3 комутатори и HSRP
> - Lab 8 → NAT (интернет свързаност + ISP simulation) ✓
> - Lab 9 → Финална интеграция (всички концепции заедно)

---

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

**Запазете .pkt файла като `Lab8_YourName.pkt`**

**ВАЖНО:** Този лаб използва ISP Router вместо Cloud устройство за реалистична симулация.



**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>