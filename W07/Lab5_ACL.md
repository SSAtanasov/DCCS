# LAB 5: ACCESS CONTROL LISTS (ACL)
## Контрол на мрежовия трафик и сигурност

**Цел:** Да се научите да създавате и прилагате ACL за контрол на достъпа между мрежи и VLAN-и.

**Продължителност:** 120-135 минути (основната част; Част 9 е BONUS)

**Предварителни изисквания:** Завършени Lab 1-4 (особено Lab 2 с VLAN-и)

**Начален файл:** Отворете `Lab4_DNS_YourName.pkt` и го запазете като `Lab5_ACL_YourName.pkt`.

---

## ЩО Е ТО ACCESS CONTROL LIST (ACL)?

### Основна концепция

**ACL (Access Control List)** е **филтър за мрежов трафик** - като охранител на врата, който проверява кой може да влезе и кой не.

```
Аналогия: ACL е като нощен клуб със списък на гостите

┌─────────────────────────────┐
│    🚪 ENTRANCE (Router)     │
├─────────────────────────────┤
│ Bouncer (ACL):              │
│                             │
│ ✅ VIP John → ALLOW         │
│ ❌ Troublemaker Bob → DENY  │
│ ✅ Everyone else → ALLOW    │
└─────────────────────────────┘
```

### Защо имаме нужда от ACL?

**Примери за защита:**
- Guest VLAN не може да достъпва Admin Server ❌
- IT VLAN може да достъпва всичко ✅
- Блокиране на конкретни хостове
- Ограничаване на услуги (само HTTP, не Telnet)

---

## ВИДОВЕ ACL - СРАВНЕНИЕ

| Характеристика | Standard ACL | Extended ACL | Named ACL |
|----------------|-------------|--------------|-----------|
| **ACL номера** | 1-99, 1300-1999 | 100-199, 2000-2699 | Име (текст) |
| **Филтрира по** | Source IP **САМО** | Source, Dest, Protocol, Port | Според типа (standard или extended) |
| **Гъвкавост** | Ограничена | Много гъвкав | Най-гъвкав |
| **Къде прилагаме** | Близо до **destination** | Близо до **source** | Според типа |
| **Пример** | `access-list 10 deny host 192.168.10.11` | `access-list 100 permit tcp ... eq 80` | `ip access-list extended WEB_ACCESS` |
| **Кога използваме** | Блокиране на цели мрежи | Детайлен контрол (port, protocol) | Модерен подход (препоръчителен) |

### Standard ACL пример:
```cisco
access-list 10 deny host 192.168.10.11
access-list 10 permit any
```
**Блокира САМО един хост, позволява всички останали**

### Extended ACL пример:
```cisco
access-list 100 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 80
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 permit ip any any
```
**Позволява само HTTP от IT към Admin, блокира всичко останало**

### Named ACL пример:
```cisco
ip access-list extended WEB_ONLY
 remark Allow only HTTP and HTTPS
 permit tcp any any eq 80
 permit tcp any any eq 443
 deny ip any any
```
**По-четлив и лесен за редакция**

---

## ЩО Е ТО WILDCARD MASK?

### Subnet Mask vs Wildcard Mask

**Subnet Mask** и **Wildcard Mask** са **ОБРАТНИ** едно на друго!

| Концепция | Subnet Mask | Wildcard Mask |
|-----------|-------------|---------------|
| **Цел** | Дефинира мрежа | Дефинира кои битове да се **игнорират** |
| **Логика** | 1 = проверявай, 0 = игнорирай | 0 = проверявай, 1 = **игнорирай** |
| **Пример /24** | 255.255.255.0 | 0.0.0.255 |
| **Използване** | IP конфигурация | ACL филтриране |

### Бърза формула:
```
Wildcard Mask = 255.255.255.255 - Subnet Mask

Пример:
Subnet Mask:  255.255.255.0
Wildcard:     255.255.255.255
            - 255.255.255.0
            ─────────────────
            = 0.0.0.255 ✅
```

---

## ТРИ ОСНОВНИ ПРИМЕРА С WILDCARD

### Пример 1: Цяла мрежа /24

**Искаме:** Всички адреси в 192.168.10.0/24

```
IP:       192.168.10.0
Wildcard: 0.0.0.255

Обяснение:
- 0.0.0 → Първите 3 октета ТРЯБВА да са 192.168.10
- 255 → Последният октет МОЖЕ ДА Е ВСИЧКО (0-255)

Съвпада с:
✅ 192.168.10.1, 192.168.10.50, 192.168.10.255
❌ 192.168.11.1 (различен 3-ти октет)
```

**В ACL:**
```cisco
access-list 10 permit 192.168.10.0 0.0.0.255
```

---

### Пример 2: Един конкретен хост

**Искаме:** САМО 192.168.10.50

```
IP:       192.168.10.50
Wildcard: 0.0.0.0

Обяснение:
- Всички 4 октета ТРЯБВА да съвпадат ТОЧНО
- Нищо не се игнорира

Съвпада с:
✅ 192.168.10.50
❌ 192.168.10.51, 192.168.10.49
```

**В ACL:**
```cisco
access-list 10 permit host 192.168.10.50
! или
access-list 10 permit 192.168.10.50 0.0.0.0
```

---

### Пример 3: Всички адреси (any)

**Искаме:** Абсолютно ВСИЧКИ IP адреси

```
IP:       0.0.0.0
Wildcard: 255.255.255.255

Обяснение:
- Всички битове се игнорират
- Съвпада с ВСИЧКО
```

**В ACL:**
```cisco
access-list 10 permit any
! или
access-list 10 permit 0.0.0.0 255.255.255.255
```

---

### Често срещани Wildcard Masks:

| Subnet Mask | Wildcard Mask | Какво означава | Пример |
|-------------|---------------|----------------|---------|
| 255.255.255.255 | 0.0.0.0 | Един хост | `host 192.168.10.50` |
| 255.255.255.0 | 0.0.0.255 | /24 мрежа (254 хоста) | `192.168.10.0 0.0.0.255` |
| 255.255.0.0 | 0.0.255.255 | /16 мрежа | `192.168.0.0 0.0.255.255` |
| 0.0.0.0 | 255.255.255.255 | Всички адреси | `any` |

---

## ACL PROCESSING - КАК РАБОТИ?

### Три ключови правила:

#### 1. **Top-to-Bottom Processing (Последователна проверка)**

```
ACL правилата се проверяват ОТ ГОРЕ НАДОЛУ:

┌────────────────────────────────────┐
│ 1. permit tcp host 192.168.10.11   │ ← Първо проверява това
│         any eq 80                  │
├────────────────────────────────────┤
│ 2. deny tcp 192.168.10.0 0.0.0.255 │ ← После това
│         any                        │
├────────────────────────────────────┤
│ 3. permit ip any any               │ ← Накрая това
├────────────────────────────────────┤
│ [implicit deny ip any any]         │ ← Невидимо, винаги в края
└────────────────────────────────────┘
```

#### 2. **First Match Wins (Първото съвпадение печели)**

```
Когато пакет СЪВПАДНЕ с правило → прилага се и проверката СПИРА

Пример:
Пакет от 192.168.10.11 → порт 80
│
├─ Правило 1: permit tcp host 192.168.10.11 eq 80 → ✅ СЪВПАДА!
│  └─ Резултат: PERMIT → СПИРА тук (не проверява останалите)
│
Правило 2: deny tcp 192.168.10.0 0.0.0.255 any
└─ НЕ СЕ ПРОВЕРЯВА (вече е минал)
```

#### 3. **Implicit Deny All (Невидимо "deny all" в края)**

**В края на ВСЕКИ ACL има невидимо:**
```cisco
deny ip any any
```

**Което означава:**
```
Ако НИТО ЕДНО правило не съвпадне → автоматично DENY ❌
```

**Затова ВИНАГИ трябва:**
```cisco
access-list 10 deny host 192.168.10.11
access-list 10 permit any    ← БЕЗ ТОВА: всички останали ще бъдат блокирани!
```

---

## ЧАСТ 1: Топология - Продължаваме от Lab 4

### Използваме топологията от Lab 1-4:

```
              [Router R1]
              192.168.10.1 (VLAN 10)
              192.168.20.1 (VLAN 20)
                   |
            GigE0/0 (trunk)
                   |        Gi0/1 (trunk)
              [Switch SW1] -----------[Switch SW2]
         /     |     |     |     \      /      \
     Fa0/11 Fa0/2 Fa0/3 Fa0/4 Fa0/5   Fa0/2  Fa0/3
        |     |     |     |     |       |      |
      DNS   PC1   PC2   PC3   PC4     [PC5]  [PC6]
     Server Admin Admin  IT    IT     VLAN10 VLAN20
   .10.2  .10.11 .10.12 .20.11 .20.12
   
   VLAN 10 (Admin)       VLAN 20 (IT)
```

**+ SW2 (Access):** свързан към SW1 Gi0/1 с trunk; PC5 (VLAN 10) и PC6 (VLAN 20) - от Lab 2, Част 3А.

### Мрежова информация от предишните лабове:

| VLAN | Име | Network | Gateway | DNS Server |
|------|-----|---------|---------|------------|
| 10 | Administration | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.2 |
| 20 | IT Department | 192.168.20.0/24 | 192.168.20.1 | 192.168.10.2 |

### Устройства:
- **DNS Server:** 192.168.10.2 (на Fa0/11, VLAN 10)
- **PC1:** 192.168.10.11 (VLAN 10 - Admin)
- **PC2:** 192.168.10.12 (VLAN 10 - Admin)
- **PC3:** 192.168.20.11 (VLAN 20 - IT)
- **PC4:** 192.168.20.12 (VLAN 20 - IT)

**Забележка:** Адресите на PC-тата са получени от DHCP и са ориентировъчни. Проверете реалните с `ipconfig` и ги използвайте в примерите по-долу (особено в правилото `host 192.168.20.11` в Част 5).

---

## ЧАСТ 2: Тестване БЕЗ ACL (baseline test) (10 мин)

### Преди да добавяме ACL, тестваме какво работи:

**От PC3 (IT VLAN 20):**
```
ping 192.168.10.2     (DNS Server в Admin VLAN)
ping 192.168.10.11    (PC1 в Admin VLAN)
ping 192.168.10.12    (PC2 в Admin VLAN)
```

**Очакван резултат:**
```
✅ Всички ping-ове работят (без ACL, няма рестрикции)
```

**От PC1 (Admin VLAN 10):**
```
ping 192.168.20.11    (PC3 в IT VLAN)
ping 192.168.20.12    (PC4 в IT VLAN)
```

**Очакван резултат:**
```
✅ Всички ping-ове работят
```

**Документирайте резултатите:** Ще ги сравним след прилагане на ACL!

---


## ЧАСТ 3: Сценарий 1 - Блокиране на цял VLAN (20 мин)

### Изискване:
**IT VLAN (20) НЕ може да достъпва Admin VLAN (10)**

⚠️ **ВАЖНО:** ACL работи САМО за трафик който минава през router (между различни VLAN-и)!

### Стъпка 1: Създаване на Standard ACL

```cisco
R1# configure terminal
R1(config)# access-list 10 remark Block IT from accessing Admin VLAN  
R1(config)# access-list 10 deny 192.168.20.0 0.0.0.255
R1(config)# access-list 10 permit any
```

**Обяснение:**
- `deny 192.168.20.0 0.0.0.255` - блокира ЦЯЛАТА мрежа 192.168.20.0/24
- `permit any` - **КРИТИЧНО!** Позволява всички останали

### Стъпка 2: Прилагане на ACL

**Standard ACL прилагаме близо до DESTINATION (Admin VLAN):**

```cisco
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# ip access-group 10 out
R1(config-subif)# exit
```

**Защо `out`?**
- Трафик ИЗЛИЗАЩ към VLAN 10 (Admin)
- Блокира всичко от IT към Admin

### Стъпка 3: Записване
```cisco
R1(config)# exit
R1# copy running-config startup-config
```

### Стъпка 4: Тестване

**От PC3 (IT VLAN - 192.168.20.11):**
```
ping 192.168.10.2     ← ❌ НЕ РАБОТИ (блокиран)
ping 192.168.10.11    ← ❌ НЕ РАБОТИ (блокиран)
```

**От PC1 (Admin VLAN - 192.168.10.11):**
```
ping 192.168.20.11    ← ❌ НЕ РАБОТИ
```

**Защо?** Заявката от PC1 стига до PC3, но отговорът (ICMP Echo Reply) идва от 192.168.20.11 и се блокира от ACL 10 на изхода към VLAN 10. ACL не помни връзките (stateless): всяко правило се прилага към всеки пакет поотделно, включително към отговорите.

**От PC1 към PC2 (и двата Admin VLAN):**
```
ping 192.168.10.12    ← ✅ РАБОТИ (остава в switch, не минава през ACL)
```

### Верификация:

```cisco
R1# show access-lists
Standard IP access list 10
    10 deny   192.168.20.0, wildcard bits 0.0.0.255
    20 permit any (5 matches)

R1# show ip interface GigabitEthernet0/0.10 | include access list
  Outgoing access list is 10
```

---

## ЧАСТ 4: Extended ACL - Разрешаване на HTTP/HTTPS, DNS и ICMP (30 мин)

### Изискване:
**IT VLAN може да достъпва към Admin VLAN само HTTP (80), HTTPS (443), DNS (53) към DNS сървъра и ICMP (за тестване)**

### Стъпка 1: Премахване на предишния ACL

```cisco
R1# configure terminal
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# no ip access-group 10 out
R1(config-subif)# exit
R1(config)# no access-list 10
```

### Стъпка 2: Създаване на Extended ACL

```cisco
R1(config)# access-list 100 remark IT to Admin - HTTP/HTTPS only
R1(config)# access-list 100 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 80
R1(config)# access-list 100 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 443
R1(config)# access-list 100 permit icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
R1(config)# access-list 100 permit udp 192.168.20.0 0.0.0.255 host 192.168.10.2 eq 53
R1(config)# access-list 100 permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.2 eq 53
R1(config)# access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
R1(config)# access-list 100 permit ip any any
```

**Обяснение на wildcard mask:**
- 0.0.0.255 означава "игнорирай последния октет"
- 192.168.20.0 0.0.0.255 = всички адреси от 192.168.20.0 до .255

**Логика на ACL 100:**
1. Permit TCP port 80 (HTTP) от IT към Admin
2. Permit TCP port 443 (HTTPS) от IT към Admin
3. Permit ICMP (ping) от IT към Admin - за тестване
4. Permit DNS (UDP и TCP 53) към DNS сървъра 192.168.10.2 - иначе имената от Lab 4 спират да работят за VLAN 20
5. Deny всичко останало от IT към Admin
6. Permit всичко останало (други VLAN-и или трафик)

### Стъпка 3: Прилагане на ACL

**Extended ACL прилагаме близо до SOURCE:**

```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# ip access-group 100 in
R1(config-subif)# exit
R1(config)# exit
R1# copy running-config startup-config
```

**Защо `in`?**
- Спира трафика КОГАТО ВЛИЗА от VLAN 20 към router-а
- По-ефективно (спираме го рано)

### Стъпка 4: Тестване

**От PC3 (IT VLAN - 192.168.20.11):**
```
ping 192.168.10.2      ← DNS Server → ✅ РАБОТИ (ICMP е разрешен)
ping 192.168.10.11     ← PC1 → ✅ РАБОТИ
nslookup server.company.local  ← ✅ РАБОТИ (DNS е разрешен)
```

**Симулиране на HTTP трафик (Packet Tracer):**
- Desktop → Web Browser → http://192.168.10.2
- Ако има HTTP server на DNS-а → ✅ РАБОТИ
- Server-PT има HTTP услуга (Services → HTTP: On), затова тестът работи и без Web Server от Lab 4
- HTTPS (443) се тества само ако HTTPS услугата на сървъра е On: `https://192.168.10.2`. Разрешението в ACL не създава HTTPS сървър.

**Тестване на блокирани услуги:**
```
telnet 192.168.10.2 23  ← Telnet → ❌ НЕ РАБОТИ (блокиран)
```

**Внимание:** DNS Server няма Telnet услуга, затова неуспехът сам по себе си не доказва действието на ACL. Доказателството е броячът на правилото `deny` в `show access-lists 100`: той трябва да се увеличи след опита.

**От PC1 (Admin VLAN → 192.168.10.11):**
```
ping 192.168.20.11   ← PC3 в IT → ✅ РАБОТИ (няма рестрикции от Admin към IT)
```

### Верификация:

```cisco
R1# show access-lists 100
Extended IP access list 100
    10 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq www
    20 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 443
    30 permit icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 (3 matches)
    40 permit udp 192.168.20.0 0.0.0.255 host 192.168.10.2 eq domain
    50 permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.2 eq domain
    60 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
    70 permit ip any any (15 matches)

R1# show ip interface GigabitEthernet0/0.20
GigabitEthernet0/0.20 is up, line protocol is up
  Internet address is 192.168.20.1/24
  Inbound  access list is 100
  Outgoing access list is not set
```

**Забележка:** `(X matches)` показва колко пъти е използвано правилото

---

## ЧАСТ 5: Named ACL - Модерен подход (20 мин)

### Предимства на Named ACL:
- По-четливи имена (вместо номера)
- Лесно се редактират (add/remove/resequence rules)
- По-добра организация и документация

### Сценарий: Създаване на Named Extended ACL

### Стъпка 1: Премахване на номерирания ACL

```cisco
R1# configure terminal
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# no ip access-group 100 in
R1(config-subif)# exit
R1(config)# no access-list 100
```

### Стъпка 2: Създаване на Named ACL

```cisco
R1(config)# ip access-list extended IT_TO_ADMIN
R1(config-ext-nacl)# remark Allow HTTP, HTTPS, ICMP and DNS from IT to Admin
R1(config-ext-nacl)# permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 80
R1(config-ext-nacl)# permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 443
R1(config-ext-nacl)# permit icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
R1(config-ext-nacl)# permit udp 192.168.20.0 0.0.0.255 host 192.168.10.2 eq 53
R1(config-ext-nacl)# permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.2 eq 53
R1(config-ext-nacl)# deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
```

### Стъпка 3: Прилагане

```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# ip access-group IT_TO_ADMIN in
R1(config-subif)# exit
R1(config)# exit
R1# copy running-config startup-config
```

### Стъпка 4: Добавяне/Редактиране на правила

**Предимство:** Можем да вмъкнем ново правило на конкретна позиция!

```cisco
R1(config)# ip access-list extended IT_TO_ADMIN
R1(config-ext-nacl)# 15 permit tcp host 192.168.20.11 any eq 22
R1(config-ext-nacl)# exit
```

**Номерът 15** поставя правилото ПРЕДИ правило 20.

**Забележка:** Правило 15 е само учебен пример: то разрешава SSH от 192.168.20.11 към **всеки** адрес. Затова по-долу го изтриваме (`no 15`).

### Показване на правилата с номера:

```cisco
R1# show access-lists IT_TO_ADMIN
Extended IP access list IT_TO_ADMIN
    10 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq www
    15 permit tcp host 192.168.20.11 any eq 22
    20 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 443
    30 permit icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
    40 permit udp 192.168.20.0 0.0.0.255 host 192.168.10.2 eq domain
    50 permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.2 eq domain
    60 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
    70 permit ip any any
```

### Изтриване на конкретно правило:

```cisco
R1(config)# ip access-list extended IT_TO_ADMIN
R1(config-ext-nacl)# no 15
R1(config-ext-nacl)# exit
```

---

## ЧАСТ 6: SSH на всички мрежови устройства и контрол на достъпа (25 мин)

### Изискване:
**Само Admin VLAN (192.168.10.0/24) може да прави SSH към router-а**

⚠️ **ВНИМАНИЕ:** Преди да прилагате ACL на VTY линиите, уверете се че:
1. Конзолният достъп работи (за rescue при грешка)
2. ACL включва ВАШИЯ текущ IP адрес
3. Имате физически достъп до устройството при нужда

**Грешка в ACL може да ви блокира напълно от достъп!**

### Стъпка 1: Конфигуриране на SSH на router (ако не е направено)

```cisco
R1(config)# hostname R1
R1(config)# ip domain-name company.local
R1(config)# crypto key generate rsa
How many bits in the modulus [512]: 1024
R1(config)# username admin secret cisco123
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exit
R1(config)# service password-encryption
```

**Пароли:** `cisco123`, `cisco` и `class123` в упражненията са само примерни. В курсовия проект са **забранени** - използвайте силни пароли (поне 10 символа с букви, цифри и знаци). `service password-encryption` само маскира паролите, зададени с `password`, чрез обратим алгоритъм (type 7) и не ги прави криптографски защитени. Защитата идва от `enable secret` и `username ... secret`.

**Забележка:** Ако рутерът има и линии vty 5–15 (проверете с `show running-config`), приложете към тях същите настройки (`line vty 5 15`), както и `access-class 50 in` в Стъпка 3. Иначе тези линии остават извън защитата.

### Стъпка 2: Създаване на Standard ACL за VTY

```cisco
R1(config)# access-list 50 remark SSH Access from Admin VLAN only
R1(config)# access-list 50 permit 192.168.10.0 0.0.0.255
R1(config)# access-list 50 deny any
```

### Стъпка 3: Прилагане на VTY линиите

```cisco
R1(config)# line vty 0 4
R1(config-line)# access-class 50 in
R1(config-line)# exit
R1(config)# exit
R1# copy running-config startup-config
```

**Забележка:** За VTY използваме `access-class` (не `ip access-group`)

### Стъпка 4: Тестване

**От PC1 (Admin VLAN - 192.168.10.11):**
```
Desktop → Command Prompt:
ssh -l admin 192.168.10.1
Password: cisco123

✅ Трябва да се свърже
```

**Забележка за Packet Tracer:** Командата е `ssh -l username IP`, за разлика от реалната среда където може и `ssh username@IP`

**От PC3 (IT VLAN - 192.168.20.11):**
```
ssh -l admin 192.168.20.1

❌ Connection refused (отказ от ACL 50)
```

**Защо към 192.168.20.1?** Това е gateway-ят на PC3, затова трафикът не минава през `IT_TO_ADMIN`. Така отказът идва само от `access-class 50`. При опит към 192.168.10.1 връзката се спира още от `IT_TO_ADMIN` и тестът не доказва действието на ACL 50.

### Верификация:

```cisco
R1# show access-lists 50
Standard IP access list 50
    10 permit 192.168.10.0, wildcard bits 0.0.0.255
    20 deny   any

R1# show line vty 0 4 | include access
Access Class 50
```

### Стъпка 5: SSH на комутаторите (SW1 и SW2)

Курсовият проект изисква SSH на **всички** мрежови устройства. На всеки комутатор:

```cisco
SW1(config)# ip domain-name company.local
SW1(config)# crypto key generate rsa
How many bits in the modulus [512]: 1024
SW1(config)# username admin secret cisco123
SW1(config)# line vty 0 15
SW1(config-line)# transport input ssh
SW1(config-line)# login local
SW1(config-line)# exit
```

Повторете същото на SW2.

**Предпоставка:** IP адрес за управление (SVI във VLAN 99) и `ip default-gateway` - вече са конфигурирани в Lab 2 (SW1: 192.168.99.2, SW2: 192.168.99.3).

**Тест от PC1 (Admin):**
```
ssh -l admin 192.168.99.2
ssh -l admin 192.168.99.3
```

---

## ЧАСТ 7: Troubleshooting ACL (10 мин)

### Често срещани грешки и решения:

| Грешка | Симптом | Как да проверим | Решение |
|--------|---------|-----------------|---------|
| **Забравен `permit any`** | Всичко блокирано (implicit deny) | `show access-lists` | Добави `access-list X permit any` |
| **Грешна посока (IN/OUT)** | ACL не работи | `show ip interface g0/0.X` | Промени на правилната посока |
| **Subnet mask вместо wildcard** | Филтрира грешно | Проверка на командата | Използвай `255.255.255.255 - subnet` |
| **ACL на физически интерфейс** | Не работи за Router-on-a-Stick | `show ip interface g0/0` | Приложи на subinterface (.10, .20) |
| **Грешен ред на правилата** | Първото правило блокира всичко | `show access-lists` | Пренареди с Named ACL |

---

### Полезни команди за troubleshooting:

```cisco
! Преглед на всички ACL-и
show access-lists

! Преглед на конкретен ACL
show access-lists 100
show access-lists IT_TO_ADMIN

! Къде е приложен ACL-а
show ip interface GigabitEthernet0/0.20

! Статистика (matches)
show access-lists
! Гледай "(X matches)" - показва колко пъти е използвано правилото

! Нулиране на counters
clear ip access-list counters
```

---

### Debug команди (ВНИМАВАЙТЕ - генерират много output!):

**Забележка:** Някои debug команди (напр. `debug ip access-list`) може да не се поддържат в Packet Tracer. Основните средства за проверка са `show access-lists` и `show ip interface`.

```cisco
! Включване
R1# debug ip packet
R1# debug ip access-list

! Направете теста (ping, etc.)

! ВАЖНО: Изключете след това!
R1# undebug all
```

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ И РЕШЕНИЯ

### 🔴 Проблем 1: ACL не блокира трафика

**Симптоми:** Ping-овете минават, въпреки че трябва да са блокирани

**Проверки:**
```cisco
! 1. ACL създаден ли е?
show access-lists

! 2. ACL приложен ли е на интерфейса?
show ip interface GigabitEthernet0/0.20

! 3. Правилна посока (in/out)?
show run interface GigabitEthernet0/0.20
```

**Възможни причини:**
- ✅ ACL не е приложен към интерфейса
- ✅ Грешна посока (in вместо out или обратно)
- ✅ `permit any` в края позволява всичко
- ✅ ACL е на физическия интерфейс вместо на subinterface

---

### 🔴 Проблем 2: Всичко е блокирано (дори разрешения трафик)

**Причина:** Implicit deny all (забравено `permit any`)

**Решение:**
```cisco
R1(config)# access-list 10 permit any
! или за Extended ACL:
R1(config)# access-list 100 permit ip any any
```

**Как да проверя:**
```cisco
R1# show access-lists
! Виж последното правило - има ли "permit any"?
```

---

### 🔴 Проблем 3: SSH не работи след прилагане на ACL 50

**Причина:** ACL 50 блокира ТВОЯ IP адрес

**Симптоми:** 
- SSH connection refused
- Не можеш да влезеш в router-а

**СПЕШНО решение:**
```cisco
! Влез през CONSOLE кабел (физически достъп)
R1> enable
R1# configure terminal
R1(config)# line vty 0 4
R1(config-line)# no access-class 50 in
R1(config-line)# exit
R1(config)# no access-list 50
R1(config)# access-list 50 permit 192.168.10.0 0.0.0.255
R1(config)# access-list 50 deny any
R1(config)# line vty 0 4
R1(config-line)# access-class 50 in
```

**Защо `no access-list 50`?** Нов ред към съществуващ номериран ACL се добавя в края, след `deny any`, и няма ефект. Затова ACL-ът се създава наново в правилния ред.

**Превенция:** ВИНАГИ тествай ACL преди да го прилагаш на VTY!

---

### 🟡 Проблем 4: ACL правилата са в грешен ред

**Причина:** First match wins - първото правило блокира всичко

**Пример на грешка:**
```cisco
access-list 100 deny ip 192.168.20.0 0.0.0.255 any     ← Блокира ВСИЧКО
access-list 100 permit tcp 192.168.20.0 0.0.0.255 any eq 80  ← Никога не се достига
```

**Решение:** Редактирайте ACL-а по номерата на редовете (работи и за номериран ACL)
```cisco
R1(config)# ip access-list extended 100
R1(config-ext-nacl)# no 10   ← Изтрий грешното правило
R1(config-ext-nacl)# 5 permit tcp 192.168.20.0 0.0.0.255 any eq 80  ← Постави го ПРЕДИ deny
```

---

### 🟡 Проблем 5: Wildcard mask грешка

**Симптом:** ACL филтрира повече/по-малко отколкото трябва

**Често срещана грешка:**
```cisco
! ГРЕШНО (subnet mask вместо wildcard):
access-list 10 permit 192.168.10.0 255.255.255.0  ❌

! ПРАВИЛНО (wildcard mask):
access-list 10 permit 192.168.10.0 0.0.0.255  ✅
```

**Формула:** Wildcard = 255.255.255.255 - Subnet Mask

---

### 🟢 Проблем 6: Не виждам "(X matches)" в show access-lists

**Причина:** Трафик не преминава през ACL правилото

**Проверки:**
1. Генерирай трафик (ping, web browser)
2. Изчакай 5-10 секунди
3. Провери отново: `show access-lists`

**Нулиране на counters:**
```cisco
R1# clear ip access-list counters
```

---

## CHECKLIST ПРЕДИ ПРИЛАГАНЕ НА ACL

```
☐ Направил съм backup на конфигурацията
☐ Имам console достъп (физически кабел)
☐ Тествал съм ACL на test интерфейс първо
☐ ACL включва "permit any" (ако е нужно)
☐ Проверил съм реда на правилата (top-to-bottom)
☐ Уверен съм в посоката (in vs out)
☐ За VTY: ACL включва МОЯ IP адрес
☐ Готов съм да отменя промените ако нещо се обърка
```

---

## БЪРЗА КОМАНДА ЗА ОТМЯНА НА ACL

```cisco
! Премахване на ACL от интерфейс:
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# no ip access-group 100 in
R1(config-subif)# exit

! Изтриване на целия ACL:
R1(config)# no access-list 100
! или
R1(config)# no ip access-list extended IT_TO_ADMIN
```

---

## ЧАСТ 8: Port Security на Switch (10 мин)

### Допълнителна сигурност на Layer 2

Port Security **ограничава кои MAC адреси** могат да се свържат към порт.

```cisco
SW1(config)# interface range FastEthernet0/2 - 3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport port-security
SW1(config-if-range)# switchport port-security maximum 1
SW1(config-if-range)# switchport port-security mac-address sticky
SW1(config-if-range)# switchport port-security violation restrict
SW1(config-if-range)# exit
```

**Защо два порта?** Курсовият проект изисква Port Security на минимум 2 switch порта (тук Fa0/2 и Fa0/3 - PC1 и PC2).

**Защо `maximum 1`?** Числото е броят разрешени MAC адреси **на всеки порт**, а не броят защитени портове. Към Fa0/2 и Fa0/3 е свързан по един компютър, затова е разрешен един адрес. С `maximum 2` втори (чужд) компютър би бил приет без нарушение.

### Violation modes:

| Mode | Какво прави | Кога да използвам |
|------|-------------|-------------------|
| **shutdown** | Shutdown-ва порта (по подразбиране) | Строга сигурност |
| **restrict** | Блокира трафика, логва, НЕ shutdown | Средна сигурност |
| **protect** | Блокира трафика, БЕЗ логове | Мека сигурност |

### Проверка:

```cisco
SW1# show port-security interface FastEthernet0/2
SW1# show port-security address
```

### Тест на нарушение

1. От PC1 и PC2 изпълнете `ping 192.168.10.1`, за да се научат sticky MAC адресите
2. `show port-security address` → по един запис за Fa0/2 и Fa0/3
3. Откачете PC1 от Fa0/2 и свържете на негово място нов PC (Test-PC, статичен IP 192.168.10.30/24, Gateway 192.168.10.1)
4. От Test-PC `ping 192.168.10.1` → **неуспешен**
5. `show port-security interface FastEthernet0/2` → Security Violation Count е по-голям от 0; при `restrict` портът остава up
6. Премахнете Test-PC и свържете отново PC1 към Fa0/2 (преди записа на файла)

---

## ЗАПИС НА ОСНОВНИЯ ФАЙЛ (за Lab 6)

**Контролен тест:** от PC3 `nslookup server.company.local` работи (DNS е разрешен); PC3 → 192.168.10.2 ping работи; SSH от PC1 към R1 работи, а от PC3 към 192.168.20.1 е отказан.

**Запазете файла като:** `Lab5_ACL_YourName.pkt` – това е началният файл за Lab 6.

**Задачите по-долу изпълнявайте в отделен файл** (`Lab5_Extra_YourName.pkt`), защото променят ACL-ите.

---

## ЧАСТ 9 (BONUS): NAT към симулиран интернет (20 мин)

Изпълнява се в отделния файл. Нужно е за курсовия проект само ако сценарият включва симулиран интернет.

### Топология

```
[R1] Gi0/1 ──── Gi0/0 [ISP] Gi0/1 ──── [Internet Server]
     203.0.113.2/30     203.0.113.1/30   198.51.100.1/24     198.51.100.10/24
```

- ISP: Router 2911; Internet Server: Server-PT (IP 198.51.100.10/24, Gateway 198.51.100.1)

### Конфигурация на ISP

```cisco
ISP(config)# interface GigabitEthernet0/0
ISP(config-if)# ip address 203.0.113.1 255.255.255.252
ISP(config-if)# no shutdown
ISP(config-if)# exit
ISP(config)# interface GigabitEthernet0/1
ISP(config-if)# ip address 198.51.100.1 255.255.255.0
ISP(config-if)# no shutdown
ISP(config-if)# exit
```

### PAT (NAT Overload) на R1

```cisco
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip address 203.0.113.2 255.255.255.252
R1(config-if)# ip nat outside
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# ip nat inside
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# ip nat inside
R1(config-subif)# exit
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
R1(config)# access-list 1 permit 192.168.10.0 0.0.0.255
R1(config)# access-list 1 permit 192.168.20.0 0.0.0.255
R1(config)# ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

### Тест

- От PC1: `ping 198.51.100.10` → успешен
- `R1# show ip nat translations` → вътрешният адрес на PC1 е преведен към 203.0.113.2

Адресите 203.0.113.0/30 и 198.51.100.0/24 са запазени за документация и примери. Подробно NAT (Static, Dynamic, PAT) е в Lab 8.

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Блокирай конкретен хост към DNS

Създайте Standard ACL, който блокира PC4 (192.168.20.12) да достъпва DNS server (192.168.10.2).

**Hints:**
- Използвай Standard ACL (1-99)
- `deny host 192.168.20.12`
- НЕ забравяй `permit any`!
- Приложи на правилния subinterface и посока

---

### Задача 2: Ограничи протоколи

Създайте Extended ACL който позволява от IT към Admin:
- ✅ ICMP (ping)
- ✅ DNS (UDP port 53)
- ❌ Всичко останало

**Hints:**
- Extended ACL (100-199)
- `permit icmp ...`
- `permit udp ... eq 53`
- `deny ip ...`
- `permit ip any any` (за другите VLAN-и)

---

### Задача 3: Multi-layer security

Комбинирайте:
1. ACL на router (inter-VLAN control)
2. Port security на 2 switch порта
3. SSH access control само от Admin VLAN

**Документирайте конфигурацията в таблица!**

---

## ДОКУМЕНТАЦИЯ ЗА ПРОЕКТА

### Таблица с ACL правила:

| ACL | Тип | Име/Номер | Действие | Source | Destination | Service | Приложен на |
|-----|-----|-----------|----------|--------|-------------|---------|-------------|
| 1 | Standard | 10 | Deny | 192.168.20.0/24 | VLAN 10 | any | GigE0/0.10 out |
| 2 | Extended | 100 | Permit | 192.168.20.0/24 | 192.168.10.0/24 | HTTP/HTTPS, DNS, ICMP | GigE0/0.20 in |
| 3 | Named | IT_TO_ADMIN | Permit | 192.168.20.0/24 | 192.168.10.0/24 | HTTP/HTTPS, DNS, ICMP | GigE0/0.20 in |
| 4 | Standard | 50 | Permit | 192.168.10.0/24 | any | SSH | VTY 0-4 |

---

## BEST PRACTICES ЗА ACL

```
✅ Extended ACL близо до source (спираме трафика рано)
✅ Standard ACL близо до destination
✅ Винаги добавяйте "permit any" (ако искате да разрешите останалото)
✅ Използвайте Named ACL (по-четливо и лесно за редакция)
✅ Добавяйте remark коментари
✅ Тествайте ПРЕДИ production!
✅ Документирайте в таблица
✅ Записвайте конфигурацията след промени
✅ Проверявайте "(X matches)" в show access-lists
```

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ Standard ACL създаден и тестван
☐ Extended ACL създаден и тестван
☐ Named ACL създаден и тестван
☐ Разбирам разликата между IN и OUT
☐ Разбирам wildcard masks (0.0.0.255 за /24)
☐ Разбирам implicit deny all
☐ ACL приложен в правилна посока
☐ SSH access control конфигуриран
☐ SSH работи на R1, SW1 и SW2
☐ Port Security на Fa0/2 и Fa0/3 конфигуриран и проверен
☐ Ping тестовете потвърждават permit правилата
☐ Ping тестовете потвърждават deny правилата
☐ ACL правилата документирани в таблица
☐ Конфигурацията е запазена
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Що е то ACL (аналогия с охранител на нощен клуб)
2. ✅ Standard vs Extended ACL (разлики и кога да използваме)
3. ✅ Wildcard mask (обратното на subnet mask)
4. ✅ ACL processing (top-to-bottom, first match wins)
5. ✅ Implicit deny all (невидимото правило в края)
6. ✅ IN vs OUT direction (къде да приложим ACL)
7. ✅ Named ACL (модерен и препоръчителен подход)
8. ✅ VTY access control (SSH security)
9. ✅ Port security (Layer 2 защита)
10. ✅ Troubleshooting ACL (show commands, debug)

---

## БЪРЗИ КОМАНДИ ЗА REFERENCE

### Standard ACL:
```cisco
access-list 10 deny host 192.168.10.11
access-list 10 permit any
interface g0/0.20
 ip access-group 10 out
```

### Extended ACL:
```cisco
access-list 100 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 80
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 permit ip any any
interface g0/0.20
 ip access-group 100 in
```

### Named ACL:
```cisco
ip access-list extended WEB_ACCESS
 remark Allow HTTP and HTTPS only
 permit tcp any any eq 80
 permit tcp any any eq 443
 deny ip any any
interface g0/0.20
 ip access-group WEB_ACCESS in
```

### VTY Access:
```cisco
access-list 50 permit 192.168.10.0 0.0.0.255
access-list 50 deny any
line vty 0 4
 access-class 50 in
```

### Port Security:
```cisco
interface Fa0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

### Show Commands:
```cisco
show access-lists                    ! Всички ACL-и
show ip interface g0/0.20            ! ACL на интерфейс
show port-security interface Fa0/2   ! Port security статус
show port-security address           ! Научени MAC адреси
```

---

## СЛЕДВАЩА СТЪПКА

В **Lab 6** ще интегрираме:
- IoT устройства (sensors, actuators)
- Автоматизация с Blockly/Python
- Smart Home/Office сценарии

**Това е финалната стъпка преди курсовия проект!**

**Запазете файла като:** `Lab5_ACL_YourName.pkt` (преди задачите; тях запазете отделно като `Lab5_Extra_YourName.pkt`)

---

**Успех с упражнението! ACL-ите са мощен инструмент за мрежова сигурност!**



**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>