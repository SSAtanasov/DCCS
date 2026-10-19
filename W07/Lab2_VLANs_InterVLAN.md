# LAB 2: VLAN КОНФИГУРАЦИЯ И INTER-VLAN ROUTING
## Сегментиране на мрежата с VLAN-и

**Цел:** Да се научите да създавате VLAN-и, да ги назначавате на портове и да конфигурирате Router-on-a-Stick.

**Продължителност:** 120-150 минути (две учебни занятия)

**Предварителни изисквания:** Завършен Lab 1

**Начален файл:** Отворете `Lab1_YourName.pkt` (записан след добавянето на PC4) и веднага го запазете като `Lab2_VLANs_YourName.pkt`.

---

## ЩО Е ТО VLAN? (ВАЖНА ТЕОРИЯ)

### Какво представлява VLAN?

**VLAN (Virtual Local Area Network)** е **виртуална локална мрежа**, създадена на ниво switch, която **логически разделя** една физическа мрежа на няколко отделни broadcast домейна.


### Преход от Lab 1: Защо НЕ използваме VLAN 1?

**В Lab 1 използвахме VLAN 1 за всички устройства. Защо сега го променяме?**
```
┌─────────────────────────────────────────┐
│ Lab 1: VLAN 1 (default)                 │
│ ❌ Всички в един VLAN                   │
│ ❌ Няма изолация                        │
│ ❌ Няма сегментация между отделите      │
└─────────────────────────────────────────┘
              ↓ ПРЕХОД
┌─────────────────────────────────────────┐
│ Lab 2: VLAN 10, 20, 99                  │
│ ✅ Логическо разделяне                  │
│ ✅ Отделни broadcast домейни            │
│ ✅ Management отделен (VLAN 99)         │
└─────────────────────────────────────────┘
```

**Защо VLAN 1 е проблем в production:**

| Риск | Обяснение |
|------|-----------|
| **Security vulnerability** | VLAN 1 не може да се изтрие - винаги съществува |
| **Management traffic exposure** | Контролни протоколи на Cisco (CDP, VTP, DTP) използват VLAN 1 на trunk портовете |
| **Default за всички портове** | Нови портове автоматично влизат в VLAN 1 |
| **Best practice violation** | Cisco препоръчва VLAN 1 да не се използва за user data |

**Оттук нататък (Lab 2-9) НИКОГА не използваме VLAN 1 за user трафик!**

---

### Защо имаме нужда от VLAN-и?

**Без VLAN-и (всички в една мрежа):**
```
┌──────────────────────────────────────┐
│        192.168.1.0/24                │
│                                       │
│  PC-Admin  PC-IT  PC-Guest  Server  │
│     ↓        ↓       ↓        ↓      │
│  [====== ВСИЧКИ В ЕДИН SWITCH =====] │
│                                       │
│  ❌ Всички виждат broadcast-ите      │
│  ❌ Няма сигурност/изолация          │
│  ❌ Труден контрол на достъпа        │
└──────────────────────────────────────┘
```

**С VLAN-и (логическо разделяне):**
```
┌──────────────────────────────────────┐
│         VLAN 10 (Admin)              │
│       192.168.10.0/24                │
│   PC1-Admin    PC2-Admin             │
│      ↓            ↓                  │
├──────────────────────────────────────┤
│         VLAN 20 (IT)                 │
│       192.168.20.0/24                │
│   PC3-IT      PC4-IT                 │
│      ↓            ↓                  │
├──────────────────────────────────────┤
│  [===== СЪЩИЯ ФИЗИЧЕСКИ SWITCH =====] │
│                                       │
│  ✅ Отделни broadcast домейни        │
│  ✅ По-малко broadcast трафик        │
│  ✅ Сигурност чрез разделяне         │
└──────────────────────────────────────┘
```

**Важно:** VLAN-ите разделят устройствата в отделни broadcast домейни. Комуникацията между тях е възможна чрез Inter-VLAN routing (Част 4), а достъпът се ограничава с ACL (Lab 5).

### VLAN = "Виртуална подмрежа"?

**ДА, но с нюанси:**

| Характеристика | Обикновена подмрежа | VLAN |
|----------------|---------------------|------|
| **Дефиниция** | IP адресно пространство (Layer 3) | Логическа сегментация (Layer 2) |
| **Пример** | 192.168.10.0/24 | VLAN 10 |
| **Изолация** | Чрез Router | На ниво Switch |
| **Broadcast домейн** | Да | Да |
| **Физическа изолация?** | НЕ (логическа) | НЕ (логическа) |

**Проста аналогия:**
- **Физическа подмрежа** = Отделни сгради с различни адреси
- **VLAN** = Апартаменти в една сграда, но с отделни входове и ключове

### Защо VLAN-ите са полезни?

1. **Сигурност:** 
   - Административният отдел (VLAN 10) не може да вижда трафика на IT (VLAN 20)
   
2. **Намаляване на broadcast трафик:**
   - Broadcast-ите в VLAN 10 НЕ достигат до VLAN 20
   
3. **Гъвкавост:**
   - Един служител може да смени отдел без да се сменят кабели
   
4. **Икономия:**
   - Не трябват отделни switch-ове за всеки отдел

---

## ЩО Е ТО SUBINTERFACE?

### Проблемът без subinterfaces:

Представете си, че имате 3 VLAN-а:
```
VLAN 10 → Router порт 1
VLAN 20 → Router порт 2  
VLAN 30 → Router порт 3
```

**Проблем:** Трябват ви **3 физически порта** на Router-а! 💸

### Решението: Subinterfaces (виртуални подинтерфейси)

**Subinterface** е **виртуален интерфейс** създаден върху един физически порт на Router-а. Това ви позволява да използвате **ЕДИН физически порт** за routing между **МНОЖЕСТВО VLAN-и**.

```
          Router R1
         ┌─────────┐
         │ GigE0/0 │ ← ЕДИН физически порт
         └────┬────┘
              │
    ┌─────────┼─────────┐
    │         │         │
 .10 │      .20 │      .99 │  ← 3 ВИРТУАЛНИ subinterfaces
    │         │         │
 VLAN 10   VLAN 20   VLAN 99
```

### Как работят subinterfaces?

**Физическият интерфейс (GigE0/0):**
- Реалният хардуерен порт
- Трябва да е `no shutdown`
- НЕ получава IP адрес (само subinterfaces-ите получават)

**Subinterfaces (GigE0/0.10, GigE0/0.20, и т.н.):**
- Виртуални интерфейси "върху" физическия порт
- Всеки има **собствен IP адрес** (gateway за съответния VLAN)
- Всеки е "маркиран" за конкретен VLAN чрез **encapsulation dot1Q**

**Аналогия:**
- Физическият порт = Главна телефонна линия
- Subinterfaces = Вътрешни телефонни номера (extension 10, 20, 99)

---

## ЩО Е ТО TRUNK?

### Access vs Trunk портове

#### Access Port (порт за достъп):
```
[PC] ───────► [Switch порт Fa0/2]
             (access VLAN 10)

- Свързва КРАЙНО устройство (PC, принтер)
- Принадлежи на ЕДИН VLAN
- Трафикът НЕ е тагван (no 802.1Q tag)
```

#### Trunk Port (магистрален порт):
```
[Switch] ────────► [Router/Switch]
       (trunk - носи VLAN 10,20,99)

- Свързва мрежови устройства (switch-router, switch-switch)
- Носи трафик за МНОЖЕСТВО VLAN-и ЕДНОВРЕМЕННО
- Трафикът Е тагван (802.1Q tag указва към кой VLAN принадлежи)
```

### IEEE 802.1Q Tagging (как се "маркира" трафикът)

Когато frame-ът минава през trunk, switch-ът **добавя 4-byte tag**:

```
Обикновен Ethernet frame (access port):
┌──────────┬─────────┬──────┬─────┐
│ Dest MAC │ Src MAC │ Type │ Data│
└──────────┴─────────┴──────┴─────┘

802.1Q Tagged frame (trunk port):
┌──────────┬─────────┬─────────────┬──────┬─────┐
│ Dest MAC │ Src MAC │ 802.1Q Tag  │ Type │ Data│
│          │         │ (VLAN ID)   │      │     │
└──────────┴─────────┴─────────────┴──────┴─────┘
                      ↑
                Contains VLAN 10 or 20 (native VLAN 99 - no tag)
```

**Пример:**
1. PC1 (VLAN 10) изпраща ping
2. Switch получава на **access порт** Fa0/2 → знае че е VLAN 10
3. Switch изпраща през **trunk порт** Fa0/1 → **добавя tag "VLAN 10"**
4. Router получава tagged frame → чете tag-а → знае да го насочи към subinterface GigE0/0.10

---

## ЩО Е ТО NATIVE VLAN? 🏷️

**Native VLAN** е специален VLAN, чийто трафик **НЕ се тагва** на trunk порта.

```
VLAN 10 traffic  ──► trunk ──► 802.1Q tag = 10 ✅
VLAN 20 traffic  ──► trunk ──► 802.1Q tag = 20 ✅
VLAN 99 traffic  ──► trunk ──► NO TAG (native) ❌
```

### Защо има native VLAN?

1. **Backward compatibility** - за стари устройства без 802.1Q
2. **Management traffic** - CDP, VTP, DTP протоколи
3. **Untagged frames** - всеки нетагнат frame, получен на trunk, се причислява към native VLAN

### Важно правило:

**Native VLAN ТРЯБВА ДА СЪВПАДА** на двата края на trunk-а!

```
Switch Fa0/1:  native VLAN 99 ✅
   │
   └──► trunk ──►
                  │
Router GigE0/0: native VLAN 99 ✅

ДОБРЕ! ✅
```

```
Switch Fa0/1:  native VLAN 99 ❌
   │
   └──► trunk ──►
                  │
Router GigE0/0: native VLAN 1 ❌

ГРЕШКА! Native VLAN mismatch ⚠️
```

---

## ЧАСТ 1: Топология (15 мин)

### Устройства:
- 1x Router 2911 (R1)
- 2x Switch 2960 (SW1 - Distribution, SW2 - Access)
- 6x PC (PC1-Admin, PC2-Admin, PC3-IT, PC4-IT, PC5-Admin, PC6-IT)

### Физическа свързаност:
```
              [Router R1]
                   |
            GigE0/0 (trunk - носи VLAN 10,20,99)
                   |
              [Switch SW1]
               /   |   \   \
        Fa0/2 Fa0/3 Fa0/4 Fa0/5 (access портове)
          /     |     \     \
     [PC1]  [PC2]  [PC3]  [PC4]
     Admin  Admin   IT     IT
   VLAN 10 VLAN 10 VLAN 20 VLAN 20
```

**+ SW2 (Access, Част 3А):** свързан към SW1 Gi0/1 с trunk; PC5 (VLAN 10) на Fa0/2 и PC6 (VLAN 20) на Fa0/3.

### VLAN дизайн:
```
VLAN 10 - Administration (192.168.10.0/24)
  ├─ PC1-Admin: 192.168.10.10
  ├─ PC2-Admin: 192.168.10.11
  ├─ PC5-Admin: 192.168.10.12 (на SW2)
  └─ Gateway: 192.168.10.1 (Router subinterface)

VLAN 20 - IT Department (192.168.20.0/24)
  ├─ PC3-IT: 192.168.20.10
  ├─ PC4-IT: 192.168.20.11
  ├─ PC6-IT: 192.168.20.12 (на SW2)
  └─ Gateway: 192.168.20.1 (Router subinterface)

VLAN 99 - Management (192.168.99.0/24) [Native VLAN]
  ├─ SW1: 192.168.99.2
  ├─ SW2: 192.168.99.3
  └─ Gateway: 192.168.99.1 (Router subinterface)
```
**⚠️ Забележка:** Използваме VLAN 99 за management, НЕ VLAN 1!

### Адресен план и subnetting (основа за курсовия проект)

Всяка VLAN е отделна IP подмрежа. Размерът на подмрежата се избира според броя на устройствата в нея:

| Маска | Префикс | Използваеми хостове | Подходяща за |
|-------|---------|---------------------|--------------|
| 255.255.255.0 | /24 | 254 | голям отдел |
| 255.255.255.128 | /25 | 126 | среден отдел |
| 255.255.255.192 | /26 | 62 | малък отдел |
| 255.255.255.224 | /27 | 30 | сървъри, малка група |
| 255.255.255.240 | /28 | 14 | management |
| 255.255.255.248 | /29 | 6 | няколко устройства |
| 255.255.255.252 | /30 | 2 | връзка рутер-рутер |

**Формула:** използваеми хостове = 2^(32 − префикс) − 2 (адресът на мрежата и broadcast адресът не се дават на устройства).

**Пример:** 192.168.10.0/26 → мрежа 192.168.10.0, gateway 192.168.10.1, хостове .1–.62, broadcast 192.168.10.63. Следващата подмрежа започва от 192.168.10.64.

В упражненията използваме /24 за простота. В курсовия проект изберете маски според броя на устройствата във всяка VLAN и ги документирайте в адресния план (мрежа, маска, gateway, DHCP диапазон, broadcast).

---

## ЧАСТ 2: Основна конфигурация (20 мин)

**Забележка:** Ако продължавате от `Lab1_YourName.pkt`, hostname, паролите и `no ip domain-lookup` вече са зададени (prompt-ът е `R1>` и `SW1>`). Тогава прескочете тази част.

### Router базова настройка:
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# no ip domain-lookup
R1(config)# enable secret class123
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# logging synchronous
R1(config-line)# exit
```

### Switch базова настройка:
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# no ip domain-lookup
SW1(config)# enable secret class123
SW1(config)# line console 0
SW1(config-line)# password cisco
SW1(config-line)# login
SW1(config-line)# logging synchronous
SW1(config-line)# exit
```

---

## ЧАСТ 3: Създаване на VLAN-и на Switch (20 мин)

### Стъпка 1: Създаване на VLAN-и

**Важно:** Не създаваме VLAN 1 - той вече съществува по подразбиране.
Ние създаваме нови VLAN-и (10, 20, 99) за правилна сегментация.

```cisco
SW1(config)# vlan 10
SW1(config-vlan)# name Administration
SW1(config-vlan)# exit
!
SW1(config)# vlan 20
SW1(config-vlan)# name IT_Department
SW1(config-vlan)# exit
!
SW1(config)# vlan 99
SW1(config-vlan)# name Management
SW1(config-vlan)# exit
```

**Обяснение на командите:**

| Команда | Какво прави |
|---------|-------------|
| `vlan 10` | Създава VLAN с ID = 10 (може да е 1-4094) |
| `name Administration` | Дава описателно име (за улеснение, незадължително) |
| `exit` | Излиза от vlan конфигурационен режим |

**Важно:** 
- VLAN ID **1** е default VLAN (всички портове са там по подразбиране)
- VLAN **1002-1005** са резервирани (Token Ring, FDDI)
- Може да използвате ID от **2 до 1001** (стандартен диапазон)

### Стъпка 2: Проверка на VLAN-ите
```cisco
SW1# show vlan brief
```

**Очакван резултат:**
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6-24, Gig0/1-2
10   Administration                   active    
20   IT_Department                    active    
99   Management                       active
```

**Забележка:** VLAN-ите са създадени, но **все още нямат портове** (ще назначим след малко)

---

### Стъпка 3: Назначаване на портове към VLAN-и

```cisco
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description PC1-Admin
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description PC2-Admin
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/4
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC3-IT
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/5
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC4-IT
SW1(config-if)# exit
```

**Обяснение на командите:**

| Команда | Какво прави | Защо е важно |
|---------|-------------|--------------|
| `interface FastEthernet0/2` | Влиза в конфигурация на конкретния порт | Трябва да сте В интерфейсния режим |
| `switchport mode access` | Казва "този порт е ACCESS (не trunk)" | Дефинира поведението на порта |
| `switchport access vlan 10` | Назначава порта към VLAN 10 | Всички frames на този порт са VLAN 10 |
| `description PC1-Admin` | Добавя описание (за документация) | Помага при troubleshooting |

**Важно:** 
- **Access портове** се свързват към **крайни устройства** (PC, принтери, сървъри)
- Всеки access порт принадлежи на **ТОЧНО ЕДИН VLAN**
- Ако не укажете VLAN, портът остава в **VLAN 1** (default)

---

### Стъпка 4: Конфигурация на trunk порт към Router

```cisco
SW1(config)# interface FastEthernet0/1
SW1(config-if)# description Trunk to R1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,99
SW1(config-if)# exit
```

**Обяснение на командите:**

| Команда | Какво прави | Пример |
|---------|-------------|--------|
| `switchport mode trunk` | Прави порта **trunk** (носи множество VLAN-и) | Порт Fa0/1 вече може да носи VLAN 10, 20, 99 едновременно |
| `switchport trunk native vlan 99` | Указва кой VLAN е **untagged** (native) | Трафик без 802.1Q tag се третира като VLAN 99 |
| `switchport trunk allowed vlan 10,20,99` | Разрешава САМО тези VLAN-и през trunk-а | VLAN 1, 30, 50 и т.н. НЕ могат да минат |

**Защо `allowed vlan`?**
- **Сигурност:** Ограничаваме кои VLAN-и могат да "пътуват"
- **Performance:** По-малко broadcast трафик
- **Best practice:** Винаги указвайте само нужните VLAN-и

**Важно за native VLAN:**
```
Switch:  native vlan 99  ✅
Router:  native vlan 99  ✅  ← ТРЯБВА ДА СЪВПАДА!

Ако не съвпада → Native VLAN mismatch warning ⚠️
```

---

### Стъпка 4.1: Премахване на управлението от Lab 1 (VLAN 1)

```cisco
SW1(config)# interface vlan 1
SW1(config-if)# no ip address
SW1(config-if)# shutdown
SW1(config-if)# exit
```

**Защо?** Управлението преминава във VLAN 99. Старият адрес 192.168.1.2 от Lab 1 не трябва да остане.

### Стъпка 5: Management VLAN адрес

```cisco
SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.99.1
SW1(config)# exit
```

**Обяснение:**

| Команда | Какво прави | Защо |
|---------|-------------|------|
| `interface vlan 99` | Създава **SVI (Switch Virtual Interface)** | За remote управление на switch-а |
| `ip address 192.168.99.2 ...` | Дава IP адрес на switch-а | За SSH/Telnet достъп |
| `no shutdown` | Активира интерфейса | По подразбиране е shutdown |
| `ip default-gateway 192.168.99.1` | Gateway за switch-а | За достъп до други мрежи (напр. за updates) |

**За какво е Management VLAN?**
- За **администриране** на switch-а (SSH, Telnet, SNMP)
- **НЕ** е за крайни потребители
- Best practice: Използвай VLAN различен от 1 (security)

### Стъпка 6: Записване
```cisco
SW1# copy running-config startup-config
```

---

## ЧАСТ 3А: Втори комутатор и йерархия Access/Distribution (25 мин)

### Защо втори комутатор?

Курсовият проект изисква **минимум 2 комутатора** и **йерархична структура** (поне Access и Distribution слой):

```
         [R1]   ← Core: маршрутизиране между VLAN-ите
          |
        [SW1]   ← Distribution: събира trunk-овете, връзка към R1
          |  trunk (Gi0/1 ↔ Gi0/1)
        [SW2]   ← Access: крайни устройства
        /    \
    [PC5]    [PC6]
   VLAN 10   VLAN 20
```

В малка мрежа R1 изпълнява ролята на Core, SW1 е Distribution, а SW2 е Access. В по-голяма мрежа всеки слой има отделни устройства.

### Стъпка 1: Добавяне и свързване

**Проверка:** ако в началния файл вече има SW2 от Задача 4 на Lab 1 (връзка SW1 Fa0/24 ↔ SW2 Fa0/24), изтрийте този SW2 и PC-тата към него, преди да продължите.

- Добавете Switch 2960 (SW2), PC5 и PC6
- SW1 **Gi0/1** ←→ SW2 **Gi0/1** (кабел **Copper Cross-Over**)
- SW2 Fa0/2 ←→ PC5, SW2 Fa0/3 ←→ PC6 (Copper Straight-Through)

### Стъпка 2: Trunk на SW1 към SW2

```cisco
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description Trunk to SW2
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,99
SW1(config-if)# exit
```

### Стъпка 3: Конфигурация на SW2

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
SW2(config)# no ip domain-lookup
SW2(config)# enable secret class123
SW2(config)# line console 0
SW2(config-line)# password cisco
SW2(config-line)# login
SW2(config-line)# exit
!
SW2(config)# vlan 10
SW2(config-vlan)# name Administration
SW2(config-vlan)# exit
SW2(config)# vlan 20
SW2(config-vlan)# name IT_Department
SW2(config-vlan)# exit
SW2(config)# vlan 99
SW2(config-vlan)# name Management
SW2(config-vlan)# exit
!
SW2(config)# interface GigabitEthernet0/1
SW2(config-if)# description Trunk to SW1
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk native vlan 99
SW2(config-if)# switchport trunk allowed vlan 10,20,99
SW2(config-if)# exit
!
SW2(config)# interface FastEthernet0/2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 10
SW2(config-if)# description PC5-Admin
SW2(config-if)# exit
SW2(config)# interface FastEthernet0/3
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
SW2(config-if)# description PC6-IT
SW2(config-if)# exit
!
SW2(config)# interface vlan 99
SW2(config-if)# ip address 192.168.99.3 255.255.255.0
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# ip default-gateway 192.168.99.1
SW2(config)# end
SW2# copy running-config startup-config
```

**Важно:**
- VLAN-ите се създават на **всеки** комутатор, през който минава трафикът им (VTP не се използва в тези упражнения)
- Native VLAN и allowed списъкът трябва да **съвпадат** на двата края на trunk-а

### Стъпка 4: IP адреси на PC5 и PC6

```
PC5 (VLAN 10): 192.168.10.12 / 255.255.255.0, Gateway 192.168.10.1
PC6 (VLAN 20): 192.168.20.12 / 255.255.255.0, Gateway 192.168.20.1
```
<!--
### Стъпка 5: Проверка (след Част 4)

```cisco
SW2# show interfaces trunk      ← Gi0/1 trunking, native VLAN 99
SW2# show vlan brief            ← Fa0/2 във VLAN 10, Fa0/3 във VLAN 20
```

- PC5 → PC1 (същата VLAN, през trunk-а SW2–SW1): успешен ping
- PC5 → PC3 (друга VLAN, през R1): успешен ping
- PC1 → 192.168.99.3 (управлението на SW2): успешен ping

**Ако PC5 не достига gateway-я си:** проверете trunk-а на двата края (`show interfaces trunk`), дали VLAN 10 съществува на SW2 (`show vlan brief`) и дали е в allowed списъка.
-->

---

## ЧАСТ 4: Router-on-a-Stick конфигурация (25 мин)

### Концепция: Какво е Router-on-a-Stick?

**Router-on-a-Stick** е метод за **Inter-VLAN routing**, при който:
- **ЕДИН** физически порт на Router-а
- Обработва трафик за **МНОЖЕСТВО VLAN-и**
- Използвайки **subinterfaces** (виртуални подинтерфейси)

```
Как работи Inter-VLAN routing?

    VLAN 10                VLAN 20
    PC1 (192.168.10.10)   PC3 (192.168.20.10)
     │                     │
     ↓                     ↓
  [Switch SW1 - не може да route между VLAN-и]
         │
         │ trunk (VLAN 10 + VLAN 20)
         ↓
    [Router R1]
         ↓
   GigE0/0 (физически порт)
    ├─ .10 → gateway за VLAN 10 (192.168.10.1)
    └─ .20 → gateway за VLAN 20 (192.168.20.1)

PC1 иска да ping PC3:
1. PC1 → gateway 192.168.10.1 (subinterface .10)
2. Router routing decision
3. Router → gateway 192.168.20.1 (subinterface .20) → PC3 ✅
```

---

### Стъпка 1: Конфигурация на subinterfaces

**Първо: Активираме ФИЗИЧЕСКИЯ интерфейс**

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# no ip address
R1(config-if)# no shutdown
R1(config-if)# description Trunk to SW1
R1(config-if)# exit
```

**Обяснение:**
- Физическият порт **НЕ получава IP адрес**
- `no ip address` премахва адреса 192.168.1.1, останал от Lab 1
- Трябва да е `no shutdown` за да работят subinterfaces-ите
- Той е "контейнер" за виртуалните подинтерфейси

---

**Създаваме SUBINTERFACE за VLAN 10:**

```cisco
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# description Gateway for VLAN 10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit
```

**Обяснение на командите:**

| Команда | Какво прави | Детайли |
|---------|-------------|---------|
| `interface GigabitEthernet0/0.10` | Създава **subinterface** с номер `.10` | Номерът може да е произволен, но за яснота използваме VLAN ID |
| `encapsulation dot1Q 10` | Казва "този subinterface обработва **802.1Q tagged frames с VLAN ID = 10**" | Това е **КЛЮЧОВАТА** команда! Без нея router-ът не знае кои frames да насочи къде |
| `ip address 192.168.10.1 ...` | IP адресът на subinterface (това е **gateway** за VLAN 10) | PC-тата в VLAN 10 ще използват този IP като default gateway |

**Важно:** 
- Командата `encapsulation dot1Q 10` казва:
  - "Когато получиш frame с tag = 10, насочи го към ТОЗИ subinterface"
  - "Когато изпращаш frame ОТ този subinterface, добави tag = 10"

---

**Създаваме SUBINTERFACE за VLAN 20:**

```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# description Gateway for VLAN 20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# exit
```

**Същата логика, но за VLAN 20.**

---

**Създаваме SUBINTERFACE за VLAN 99 (Management - NATIVE VLAN):**

```cisco
R1(config)# interface GigabitEthernet0/0.99
R1(config-subif)# description Gateway for Management VLAN
R1(config-subif)# encapsulation dot1Q 99 native
R1(config-subif)# ip address 192.168.99.1 255.255.255.0
R1(config-subif)# exit
R1(config)# exit
```

**Обяснение на `native` keyword:**

| Команда | Без `native` | С `native` |
|---------|--------------|------------|
| `encapsulation dot1Q 99` | Очаква **tagged** frames (tag=99) | Обработва **untagged** frames |
| Поведение | Frame-ове с tag = 99 → този subinterface | Frame-ове БЕЗ tag → този subinterface (native VLAN) |

**Защо е важно:**
- Native VLAN трафикът НЕ се тагва
- Трябва да укажете `native` keyword за да работи правилно
- **Native VLAN трябва да съвпада с този на switch-а** (и двете са VLAN 99)

---

### Стъпка 2: Записване
```cisco
R1# copy running-config startup-config
```

---

### Проверка на конфигурацията:

```cisco
R1# show ip interface brief
```

**Очакван резултат:**
```
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up
GigabitEthernet0/0.10  192.168.10.1    YES manual up                    up
GigabitEthernet0/0.20  192.168.20.1    YES manual up                    up
GigabitEthernet0/0.99  192.168.99.1    YES manual up                    up
```

**Забележка:**
- Физическият интерфейс (GigE0/0) е `up/up` но **няма IP**
- Трите subinterfaces имат IP адреси и са активни

```cisco
R1# show running-config
```

**Търсете секциите на subinterfaces:**
```
interface GigabitEthernet0/0.10
 description Gateway for VLAN 10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 description Gateway for VLAN 20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
interface GigabitEthernet0/0.99
 description Gateway for Management VLAN
 encapsulation dot1Q 99 native
 ip address 192.168.99.1 255.255.255.0
```

**Забележка:** Официалните упражнения на Cisco за Packet Tracer проверяват subinterfaces с `show ip interface brief` и `show running-config`; командата `show vlans` на рутера не е сред тях.

---

## ЧАСТ 5: Конфигурация на PC-та (15 мин)

### PC1-Admin (VLAN 10):
```
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1   ← subinterface GigE0/0.10
DNS Server: (не се попълва; DNS сървър се добавя в Lab 4)
```

### PC2-Admin (VLAN 10):
```
IP Address: 192.168.10.11
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1   ← същият gateway (subinterface)
```

### PC3-IT (VLAN 20):
```
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1   ← subinterface GigE0/0.20
```

### PC4-IT (VLAN 20):
```
IP Address: 192.168.20.11
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
```

---

## ЧАСТ 6: Тестване и верификация (20 мин)

### Test 1: Ping в рамките на същия VLAN
```
От PC1 (VLAN 10):
ping 192.168.10.11    (към PC2 - СЪЩО в VLAN 10)
```

**Очакван резултат:** ✅ Успешен

**Какво се случва:**
1. PC1 вижда че 192.168.10.11 е в същата мрежа (192.168.10.0/24)
2. Изпраща директно към PC2 (без router)
3. Switch получава frame на Fa0/2 (VLAN 10) → форвардва на Fa0/3 (VLAN 10)
4. **Router НЕ участва** (Layer 2 switching)

---

### Test 2: Ping между различни VLAN-и
```
От PC1 (VLAN 10):
ping 192.168.20.10    (към PC3 в VLAN 20)
```

**Очакван резултат:** ✅ Успешен (чрез router)

**Какво се случва (СТЪПКА ПО СТЪПКА):**

```
1. PC1 изпраща ICMP към 192.168.20.10
   └─ PC1 вижда че това е РАЗЛИЧНА мрежа → трябва през gateway
   └─ Изпраща frame с Destination MAC = Gateway (192.168.10.1)

2. Switch SW1 получава frame на Fa0/2 (access VLAN 10)
   └─ Форвардва през trunk Fa0/1
   └─ Добавя 802.1Q tag = 10

3. Router R1 получава tagged frame на GigE0/0
   └─ Чете tag = 10 → насочва към subinterface GigE0/0.10
   └─ Прави routing decision: 192.168.20.0/24 е свързана към GigE0/0.20
   └─ Изпраща frame ОБРАТНО към switch, но с tag = 20

4. Switch SW1 получава frame с tag = 20 на trunk Fa0/1
   └─ Премахва tag-а
   └─ Форвардва на Fa0/4 (access VLAN 20) → PC3

5. PC3 получава ICMP request → изпраща ICMP reply
   └─ Същият процес, но в обратна посока
```

**Ключов момент:** 
- **Switch НЕ може** да route между VLAN-и (Layer 2 устройство)
- **Router МОЖЕ** (Layer 3 устройство)
- Frame-ът "излиза" от switch-а, преминава през router и се "връща" (затова "Router-on-a-Stick")

---

### Test 3: Ping към gateway-ове
```
От PC1:
ping 192.168.10.1    (собствен gateway)
ping 192.168.20.1    (gateway на другия VLAN)
```

**И двете трябва да работят!**

---

### Test 4: Проверка на routing таблицата
```cisco
R1# show ip route
```

**Очакван резултат:**
```
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L       192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
     192.168.99.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.99.0/24 is directly connected, GigabitEthernet0/0.99
L       192.168.99.1/32 is directly connected, GigabitEthernet0/0.99
```

**Обяснение:**
- `C` = **Connected route** (директно свързана мрежа)
- `L` = **Local route** (адресът на самия subinterface, /32)
- Router-ът "знае" за тези 3 мрежи защото има subinterfaces с IP адреси в тях

---

### Test 5: Проверка на VLAN assignments
```cisco
SW1# show vlan brief
```

**Трябва да видите:**
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/6-24, Gig0/2
10   Administration                   active    Fa0/2, Fa0/3
20   IT_Department                    active    Fa0/4, Fa0/5
99   Management                       active
```

```cisco
SW1# show interfaces trunk
```

**Трябва да видите:**
```
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      99
Gig0/1      on           802.1q         trunking      99

Port        Vlans allowed on trunk
Fa0/1       10,20,99
Gig0/1      10,20,99
```

---

### Test 6: Traceroute между VLAN-и
```
От PC1:
tracert 192.168.20.10
```

**Очакван резултат:**
```
1    192.168.10.1    (gateway - router subinterface)
2    192.168.20.10   (destination PC3)
```

**Това потвърждава че трафикът минава през Router-а!**

---

## ЧАСТ 7: Верификационни команди (Обобщение)

### На Router:
```cisco
R1# show ip interface brief        ← IP адреси и статус на интерфейси
R1# show ip route                   ← Routing таблица
R1# show running-config             ← Пълна конфигурация (вкл. encapsulation dot1Q)
```

### На Switch:
```cisco
SW1# show vlan brief                ← Всички VLAN-и и портове
SW1# show interfaces trunk          ← Trunk портове и allowed VLANs
SW1# show interfaces Fa0/1 switchport  ← Детайли за trunk порт
SW1# show mac address-table         ← MAC адреси и техните портове
SW1# show running-config            ← Пълна конфигурация
```

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ И РЕШЕНИЯ

### Проблем 1: Няма ping между VLAN-и ❌

**Симптоми:**
- PC1 (VLAN 10) → PC3 (VLAN 20): Request timed out

**Възможни причини и решения:**

| Причина | Проверка | Решение |
|---------|----------|---------|
| Router интерфейсът е shutdown | `R1# show ip int brief` | `R1(config-if)# no shutdown` |
| Грешен encapsulation | `R1# show running-config` | Проверете дали VLAN ID съвпада |
| Native VLAN не съвпада | `SW1# show int trunk` | Направете еднакви (напр. 99 и на двете) |
| Trunk не е конфигуриран | `SW1# show int Fa0/1 switchport` | Конфигурирайте като trunk |

**Debugging стъпки:**
```cisco
R1# show ip interface brief   ← Всички ли subinterfaces са up/up?
R1# show running-config        ← encapsulation dot1Q правилен ли е?
SW1# show interfaces trunk    ← Trunk-ът active ли е?
SW1# show vlan brief           ← Портовете назначени ли са към VLAN-и?
```

---

### Проблем 2: Trunk не работи ❌

**Симптоми:**
- Native VLAN mismatch warning
- Trunk-ът е активен, но трафикът на VLAN 99 не преминава

**Причина:** Native VLAN не съвпада

**Забележка:** Ако `show interfaces trunk` не показва нищо, портът изобщо не е trunk (вижте Проблем 1, последния ред).

**Решение:**
```cisco
! На Switch-а:
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport trunk native vlan 99

! На Router-а:
R1(config)# interface g0/0.99
R1(config-subif)# encapsulation dot1Q 99 native
```

**Проверка:**
```cisco
SW1# show interfaces Fa0/1 switchport
```

Трябва да видите:
```
Name: Fa0/1
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Trunking Native Mode VLAN: 99
```

---

### Проблем 3: PC не може да стигне gateway ❌

**Симптоми:**
- `ping 192.168.10.1` от PC1 → Request timed out

**Причини и решения:**

| Причина | Проверка на PC | Решение |
|---------|----------------|---------|
| Грешен default gateway | `ipconfig` | Променете на правилния (напр. 192.168.10.1) |
| Грешна subnet mask | `ipconfig` | Трябва да е 255.255.255.0 |
| Порт не е в правилния VLAN | `SW1# show vlan brief` | Назначете порта към правилния VLAN |
| Cabling issue | Visual inspection | Проверете дали кабелът е свързан |

---

## АНАЛИЗ НА ТРАФИКА (PACKET FLOW) - Обяснение

### Сценарий: PC1 (VLAN 10) ping PC3 (VLAN 20)

```
┌─────────────────────────────────────────────────────────┐
│ СТЪПКА 1: PC1 създава ICMP Echo Request                │
└─────────────────────────────────────────────────────────┘
PC1 (192.168.10.10):
  └─ Destination: 192.168.20.10 (не е в локалната мрежа)
  └─ Изпраща към gateway: 192.168.10.1
  └─ Frame:
      Source MAC: PC1's MAC
      Dest MAC: R1's MAC (gateway)
      Data: ICMP Echo Request

┌─────────────────────────────────────────────────────────┐
│ СТЪПКА 2: Switch SW1 получава на Fa0/2 (access VLAN 10)│
└─────────────────────────────────────────────────────────┘
Switch SW1:
  └─ Получава untagged frame на Fa0/2
  └─ Знае че Fa0/2 е в VLAN 10
  └─ MAC таблица: R1's MAC е на Fa0/1 (trunk)
  └─ Добавя 802.1Q tag = 10
  └─ Изпраща през Fa0/1 (trunk)

┌─────────────────────────────────────────────────────────┐
│ СТЪПКА 3: Router R1 получава tagged frame              │
└─────────────────────────────────────────────────────────┘
Router R1:
  └─ Получава frame с 802.1Q tag = 10 на GigE0/0
  └─ Насочва към subinterface GigE0/0.10
  └─ Routing decision:
      Destination 192.168.20.10 е в мрежа 192.168.20.0/24
      Тази мрежа е connected на GigE0/0.20
  └─ Изпраща frame към GigE0/0.20
  └─ Добавя 802.1Q tag = 20
  └─ Frame излиза обратно на физическия порт GigE0/0

┌─────────────────────────────────────────────────────────┐
│ СТЪПКА 4: Switch SW1 получава frame с tag = 20         │
└─────────────────────────────────────────────────────────┘
Switch SW1:
  └─ Получава tagged frame (VLAN 20) на Fa0/1 (trunk)
  └─ MAC таблица: PC3's MAC е на Fa0/4
  └─ Премахва 802.1Q tag
  └─ Форвардва untagged frame на Fa0/4 (access VLAN 20)

┌─────────────────────────────────────────────────────────┐
│ СТЪПКА 5: PC3 получава ICMP Echo Request               │
└─────────────────────────────────────────────────────────┘
PC3 (192.168.20.10):
  └─ Получава ICMP Echo Request
  └─ Генерира ICMP Echo Reply
  └─ [Същият процес в обратна посока]
```

**Ключови наблюдения:**
1. Frame-ът "излиза" от switch → router → "връща се" в switch
2. 802.1Q tag се добавя/премахва динамично
3. Router прави routing между двете мрежи (Inter-VLAN routing)
4. Без router, PC1 и PC3 **НЕ МОГАТ** да комуникират

---

## ЗАПИС НА ОСНОВНИЯ ФАЙЛ (за Lab 3)

**Контролен тест:** PC1 → PC3 и PC5 → PC3 ping успешни; `show interfaces trunk` на SW2 показва Gi0/1; `show interfaces trunk` показва Fa0/1 с native VLAN 99; `show ip interface brief` на R1 показва .10, .20 и .99 up/up.

**Запазете файла като:** `Lab2_VLANs_YourName.pkt` – това е началният файл за Lab 3.

**Бонусът и задачите по-долу изпълнявайте в отделен файл** (`Lab2_Extra_YourName.pkt`): те използват Fa0/6 и VLAN 30 и изключват Fa0/7–24, а тези портове са нужни в Lab 3–6.

---

## ДОПЪЛНИТЕЛНА КОНФИГУРАЦИЯ: Voice VLAN - BONUS

### Ако искате да добавите IP Phone (optional):
```cisco
SW1(config)# vlan 100
SW1(config-vlan)# name Voice
SW1(config-vlan)# exit
!
SW1(config)# interface FastEthernet0/6
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10    ← PC в VLAN 10
SW1(config-if)# switchport voice vlan 100    ← Phone в VLAN 100
SW1(config-if)# exit
```

**Как работи:**
- **PC** → VLAN 10 (untagged)
- **IP Phone** → VLAN 100 (tagged)
- Едно физическо свързване, но **два VLAN-а** едновременно!
- За връзка извън switch-а VLAN 100 трябва да се добави към trunk-а (`switchport trunk allowed vlan add 100`) и да има subinterface на R1

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Добавете трети VLAN
- Създайте VLAN 30 - Guest Network
- Назначете порт Fa0/6 към този VLAN
- Адресно пространство: 192.168.30.0/24
- Добавете VLAN 30 към trunk-а (Fa0/1): `switchport trunk allowed vlan add 30`
- Добавете subinterface на Router (GigE0/0.30)
- Добавете PC и тествайте connectivity

### Задача 2: Портова сигурност (Port Security)
Добавете Port Security на access портовете:
```cisco
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 1
SW1(config-if)# switchport port-security mac-address sticky
SW1(config-if)# switchport port-security violation restrict
```

### Задача 3: VLAN Hopping защита
Shutdown неизползваните портове:
```cisco
SW1(config)# interface range FastEthernet0/7-24
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 999
SW1(config-if-range)# shutdown
```

### Задача 4: Документация
Създайте таблица с:
- VLAN ID, име, subnet, gateway IP
- Port assignments (кой порт в кой VLAN)
- Device IPs (всички крайни устройства)

### Задача 5: Адресен план с различни маски
В 192.168.50.0/24 направете план за: VLAN A - 40 устройства, VLAN B - 25 устройства, VLAN C - 10 устройства, Management - 5 устройства. За всяка подмрежа определете маска, адрес на мрежата, gateway, DHCP диапазон и broadcast адрес. Подмрежите не трябва да се припокриват.

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ Създадени са VLAN 10, 20, 99 на switch-а
☐ Портовете са правилно назначени към VLAN-ите (Fa0/2-5)
☐ SW2 е свързан с trunk (Gi0/1 ↔ Gi0/1), VLAN-ите са създадени и на SW2
☐ PC5 и PC6 (на SW2) достигат своите gateway-и
☐ Trunk е конфигуриран между SW1 Fa0/1 и R1 GigE0/0
☐ Native VLAN е 99 и на switch-а, и на router-а
☐ Subinterfaces на Router са активни (show ip int brief)
☐ Encapsulation dot1Q е правилно настроен за всеки VLAN
☐ Всички PC имат правилни IP настройки и gateway
☐ Ping работи в рамките на един VLAN (PC1 → PC2)
☐ Ping работи между различни VLAN-и (PC1 → PC3)
☐ Traceroute показва routing през R1 (2 hops)
☐ Show команди потвърждават конфигурацията
```

---

## КАКВО НАУЧИХМЕ

1. ✅ **VLAN** е виртуална локална мрежа (логически broadcast домейн на ниво switch)
2. ✅ **Subinterface** е виртуален интерфейс върху физически порт на router
3. ✅ **Access портове** свързват крайни устройства (един VLAN)
4. ✅ **Trunk портове** носят трафик за множество VLAN-и (802.1Q tagging)
5. ✅ **Native VLAN** е untagged VLAN на trunk-а (трябва да съвпада на двата края)
6. ✅ **Router-on-a-Stick** е метод за Inter-VLAN routing с един физически порт
7. ✅ **802.1Q tagging** маркира към кой VLAN принадлежи frame-ът
8. ✅ **Inter-VLAN routing** позволява комуникация между различни VLAN-и

---

## СЛЕДВАЩА СТЪПКА

В **Lab 3** ще научим:
- **DHCP конфигурация** (автоматично IP адресиране)
- Router като DHCP server за множество VLAN-и
- DHCP Relay Agent
- IP Helper address

**Запазете файла като:** `Lab2_VLANs_YourName.pkt` (преди бонуса и задачите; тях запазете отделно като `Lab2_Extra_YourName.pkt`)

---

## БЪРЗИ КОМАНДИ ЗА REFERENCE

### Switch:
```cisco
! Създаване на VLAN
vlan 10
 name Administration

! Access порт
interface Fa0/2
 switchport mode access
 switchport access vlan 10

! Trunk порт
interface Fa0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99

! Management VLAN
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown
ip default-gateway 192.168.99.1
```

### Router:
```cisco
! Физически интерфейс
interface g0/0
 no shutdown

! Subinterface
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

! Native VLAN subinterface
interface g0/0.99
 encapsulation dot1Q 99 native
 ip address 192.168.99.1 255.255.255.0
```

---

**Успех с упражнението!**


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
