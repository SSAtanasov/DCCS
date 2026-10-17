# LAB 1: ОСНОВНА МРЕЖОВА ТОПОЛОГИЯ
## Базова конфигурация на Switch и Router

**Цел:** Да се научите да създавате проста мрежа и да конфигурирате основните параметри на устройствата.

**Продължителност:** 60-90 минути

**Необходимо:** Cisco Packet Tracer 8.2.2 (една и съща версия за преподавател и студенти: файл, записан в по-нова версия, може да не се отвори в по-стара)

---

## ОСНОВНИ КОНЦЕПЦИИ (ПРЕДИ ДА ЗАПОЧНЕМ)

### Какво е Router (Маршрутизатор)?

**Router** е устройство, което **свързва различни мрежи** и решава накъде да изпрати данните.

```
Аналогия: Router е като пощенски служител
- Получава пакети с адреси
- Решава по кой път да ги изпрати
- Свързва различни "квартали" (мрежи)
```

**Основни характеристики:**
- Работи на **Layer 3** (мрежов слой)
- Има **IP адреси** на интерфейсите си
- Прави **routing decisions** (къде да изпрати пакета)
- Свързва **различни IP мрежи**

---

### Какво е Switch (Комутатор)?

**Switch** е устройство, което **свързва устройства в една мрежа** и пренасочва трафик между тях.

```
Аналогия: Switch е като телефонна централа
- Свързва много устройства
- Препраща съобщения между тях
- Всички са в една "сграда" (мрежа)
```

**Основни характеристики:**
- Работи на **Layer 2** (канален слой)
- Препраща frames на база **MAC адреси**
- Създава **локална мрежа** (LAN)
- По подразбиране не прави routing между мрежи

---

### IP Адресиране - основи

**IP адрес** е уникален идентификатор на устройство в мрежата.

```
Формат: 192.168.1.1   (маска 255.255.255.0 = /24)
         │   │   │ │
         │   │   │ └─ Host част (устройство)
         ├───┴───┴─── Мрежова част (192.168.1)
         └─────────── 192 = клас C (192–223)
```

**Subnet Mask:** Определя коя част е мрежа и коя е host
```
255.255.255.0 означава:
- Първите 3 октета (192.168.1) = мрежа
- Последният октет (.1, .2, .3...) = устройства в мрежата
```

**Default Gateway:** IP адресът на Router-а (изходът към други мрежи)
```
Когато PC иска да изпрати данни извън локалната мрежа,
изпраща ги към gateway-я (Router-а)
```

---

## ЧАСТ 1: Изграждане на топологията (15 мин)

### Устройства за добавяне:
- 1x Router 2911 (R1)
- 1x Switch 2960 (SW1)
- 3x PC (PC1, PC2, PC3)
- Cables: Copper Straight-Through (прави кабели)

### Как да добавите устройства в Packet Tracer:

1. **Долен ляв ъгъл:**
   - Кликни на **Routers** → избери **2911**
   - Плъзни в работната област
   
2. **Switches:**
   - Кликни на **Switches** → избери **2960**
   - Плъзни в работната област

3. **End Devices:**
   - Кликни на **End Devices** → избери **PC**
   - Плъзни 3 пъти (за PC1, PC2, PC3)

4. **Свързване с кабели:**
   - Кликни на **Connections** (lightning bolt icon)
   - Избери **Copper Straight-Through** (черна непрекъсната линия)
   - Кликни на първо устройство → избери порт
   - Кликни на второ устройство → избери порт

---

### Свързване на устройствата:

```
Router GigabitEthernet0/0 ←→ Switch FastEthernet0/1
Switch FastEthernet0/2 ←→ PC1 FastEthernet0
Switch FastEthernet0/3 ←→ PC2 FastEthernet0
Switch FastEthernet0/4 ←→ PC3 FastEthernet0
```

**Важно:** 
- Router-Switch връзка: GigE порт на router-а (2911 има само GigabitEthernet портове)
- Switch-PC връзки: обикновени FastEthernet портове

**Какво означават имената на портовете?**

| Име | Съкращение | Скорост | Къде ги има в това упражнение |
|-----|------------|---------|------------------------|
| **FastEthernet** | `Fa` | 100 Mbps (10/100) | Switch 2960: Fa0/1 – Fa0/24; PC: FastEthernet0 |
| **GigabitEthernet** | `Gi` (в текста: GigE) | 1000 Mbps = 1 Gbps (10/100/1000) | Router 2911: Gi0/0 – Gi0/2; Switch 2960: Gi0/1 – Gi0/2 |

```
FastEthernet0/1
│           │ │
│           │ └─ Port (номер на порта) = 1
│           └─── Slot (модул) = 0
└─────────────── Тип на интерфейса = FastEthernet (100 Mbps)
```

**Аналогия:** FastEthernet е обикновен път, GigabitEthernet е магистрала с 10 пъти по-голям капацитет.

**Забележка:** Връзката работи със скоростта на по-бавния порт. Gi0/0 (R1) ←→ Fa0/1 (SW1) = 100 Mbps.

---

### Схема на топологията:

```
              [Router R1]
              192.168.1.1
                   |
               GigE0/0
                   |
              [Switch SW1]
              192.168.1.2
               /    |    \
           Fa0/2 Fa0/3 Fa0/4
             /     |     \
         [PC1]  [PC2]  [PC3]
        .10     .11     .12
    
    Мрежа: 192.168.1.0/24
    Gateway: 192.168.1.1
```

---

## ЧАСТ 2: Конфигурация на Router (20 мин)

### Стъпка 1: Влизане в Router

1. Кликнете върху **Router** икона
2. Изберете таб **CLI** (Command Line Interface)
3. Натиснете **Enter** за да видите prompt-а

```
Виждате:

Would you like to enter the initial configuration dialog? [yes/no]: 

Напишете: no [Enter]
```

**Защо "no"?** 
- Setup wizard е за начинаещи и е ограничен
- Ние ще конфигурираме ръчно (по-гъвкаво)

---

### Стъпка 2: Базова конфигурация

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
Запишете си паролите на хартия!

**Внимание:** Паролите `cisco` и `class123` са само за упражненията. В курсовия проект слабите пароли са **забранени** (вижте заданието).

**Детайлно обяснение на ВСЯКА команда:**

| Команда | Какво прави | Защо е важно | Пример резултат |
|---------|-------------|--------------|-----------------|
| `enable` | Влиза в **privileged EXEC mode** | Дава достъп до всички команди | Prompt-ът става `Router#` вместо `Router>` |
| `configure terminal` | Влиза в **global configuration mode** | Позволява промени в конфигурацията | Prompt става `Router(config)#` |
| `hostname R1` | Задава име на устройството | За идентификация (вместо "Router" вижда се "R1") | Prompt става `R1(config)#` |
| `no ip domain-lookup` | Изключва автоматичното DNS търсене | Когато напишете грешна команда, не чака DNS отговор (по-бързо) | Без delay при грешки |
| `enable secret class123` | Задава **хеширана** парола за privileged mode | Защита на устройството (enable = влизане в режим на администратор) | При `enable` ще пита за парола |
| `line console 0` | Влиза в конфигурация на **конзолната линия** (физическото свързване) | За настройка на достъпа през конзолния порт | Prompt става `R1(config-line)#` |
| `password cisco` | Задава парола за конзолен достъп | Защита при директен физически достъп | При влизане ще пита за "cisco" |
| `login` | **Активира** проверката за парола | Без това, паролата няма да се изисква! | Ще изисква парола при login |
| `logging synchronous` | Синхронизира log съобщенията | Предотвратява прекъсване на командите от системни съобщения | По-чисто писане в CLI |
| `exit` | Излиза от текущия режим | Връща към горното ниво | `R1(config-line)#` → `R1(config)#` |

---

### Режими на работа (важно!):

```
Router>                      ← User EXEC mode (ограничен)
  ↓ enable
Router#                      ← Privileged EXEC mode (пълен достъп)
  ↓ configure terminal
Router(config)#              ← Global Configuration mode
  ↓ interface g0/0
Router(config-if)#           ← Interface Configuration mode
  ↓ exit
Router(config)#              
  ↓ exit
Router#
  ↓ disable            (exit/logout тук прекратява сесията)
Router>
```

**Клавишни комбинации:**
- `Ctrl+C` → Отказ от текущата команда
- `Ctrl+Shift+6` → Прекъсване на ping/traceroute
- `Ctrl+Z` → Директен изход към privileged mode
- `exit` → Едно ниво назад
- `end` → Директно към privileged mode

---

### Стъпка 3: Конфигурация на интерфейса

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description Connection to SW1
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Детайлно обяснение:**

| Команда | Какво прави | Аналогия | Допълнително |
|---------|-------------|----------|--------------|
| `interface GigabitEthernet0/0` | Влиза в конфигурация на конкретен интерфейс | Като отваряш настройките на мрежова карта в Windows | Формат: `type slot/port` (GigE **0/0** = slot 0, port 0) |
| `ip address 192.168.1.1 255.255.255.0` | Задава IP адрес и subnet mask | Като задаваш IP на мрежова карта | **192.168.1.1** = gateway (обикновено .1), **/24** = 255.255.255.0 |
| `description Connection to SW1` | Добавя описание на порта | Като етикет "Към Switch SW1" | Не е задължително, но помага при troubleshooting |
| `no shutdown` | **Активира** интерфейса | Като "включване" на мрежовата карта | **МНОГО ВАЖНО!** По подразбиране интерфейсите на router са **shutdown** |
| `exit` | Излиза от interface режим | Връщане назад | Prompt: `R1(config-if)#` → `R1(config)#` |

---

### Защо `no shutdown` е КРИТИЧНО важна команда?

**Cisco router интерфейсите са shutdown по подразбиране!**

```
Без no shutdown:
R1# show ip interface brief
GigabitEthernet0/0  192.168.1.1  YES manual administratively down  down
                                         ↑
                                    SHUTDOWN! ❌

С no shutdown:
R1# show ip interface brief
GigabitEthernet0/0  192.168.1.1  YES manual up                    up
                                         ↑
                                    ACTIVE! ✅
```

**Забележка:** Switch портовете **не са shutdown** по подразбиране и стават up веднага щом се свърже кабел (обратното на router)

---

### Стъпка 4: Записване на конфигурацията

```cisco
R1(config)# exit
R1# copy running-config startup-config
Destination filename [startup-config]? [Enter]
Building configuration...
[OK]
```

**Обяснение:**

| Термин | Какво е | Къде се намира | Какво става при restart |
|--------|---------|----------------|-------------------------|
| **running-config** | Активната конфигурация В МОМЕНТА | RAM (временна памет) | ❌ **ИЗГУБВА СЕ** при restart |
| **startup-config** | Запазената конфигурация | NVRAM (постоянна памет) | ✅ **ОСТАВА** след restart |

**Аналогия:**
- `running-config` = Word документ преди да натиснеш Save
- `startup-config` = Word документ след Save (на диска)

**Кратка команда:**
```cisco
R1# wr           ← Съкращение за write (същото като copy run start)
```

**Проверка дали е запазена:**
```cisco
R1# show startup-config     ← Вижте какво ще се зареди при restart
```

---

## ЧАСТ 3: Конфигурация на Switch (20 мин)

### Стъпка 1: Базова конфигурация

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

**Забележка:** Командите са СЪЩИТЕ като при Router-а!

**Защо Switch също има hostname и пароли?**
- За **security** (защита на достъпа)
- За **management** (администриране)
- За **идентификация** (да знаеш кое устройство конфигурираш)

---

### Стъпка 2: VLAN 1 management адрес

```cisco
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

**Какво е VLAN 1?**

| Термин | Обяснение |
|--------|-----------|
| **VLAN** | Virtual LAN (виртуална локална мрежа) |
| **VLAN 1** | **Default VLAN** - всички портове на switch-а са тук по подразбиране |
| **interface vlan 1** | **SVI (Switch Virtual Interface)** - виртуален интерфейс за управление на switch-а |

**Защо switch-ът има нужда от IP адрес?**

Switch работи на **Layer 2** (MAC адреси), но за **remote management** (Telnet, SSH, web interface) има нужда от **IP адрес**.

```
Без IP адрес:
- Switch работи (форвардва трафик) ✅
- НЕ можеш да го управляваш отдалечено ❌

С IP адрес:
- Switch работи ✅
- SSH/Telnet: изискват допълнителна конфигурация (line vty, парола, login; за SSH и домейн, RSA ключове, потребител) ⚠️
- Можеш да го ping-неш ✅
```

**Важно:** IP адресът на switch-а трябва да е в **същата мрежа** като устройствата!

```
Router:  192.168.1.1  ✅
Switch:  192.168.1.2  ✅
PC1:     192.168.1.10 ✅

Switch:  10.0.0.1     ❌ Различна мрежа! Управлението няма да работи!
```

---

### Стъпка 3: Default gateway

```cisco
SW1(config)# ip default-gateway 192.168.1.1
```

**Какво прави `ip default-gateway`?**

**Default gateway** е "изходът" към други мрежи.

```
Сценарий: Администратор от друга мрежа (10.0.0.5) управлява SW1

SW1 (192.168.1.2) трябва да изпрати отговор към 10.0.0.5:
  ↓
"10.0.0.5 не е в моята мрежа (192.168.1.0/24)"
  ↓
"Изпращам към default gateway: 192.168.1.1"
  ↓
Router (192.168.1.1) получава пакета
  ↓
Router го препраща, ако има маршрут към 10.0.0.0
```

**Без default gateway:**
```
SW1 може да достъпи само устройства в 192.168.1.0/24 ❌
```

**С default gateway:**
```
SW1 може да достигне мрежите, до които R1 има маршрут ✅
(За интернет R1 се нуждае от външна връзка, маршрут по подразбиране и NAT.)
```

---

### Стъпка 4: Описание на портове

```cisco
SW1(config)# interface FastEthernet0/1
SW1(config-if)# description Connection to R1
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/2
SW1(config-if)# description PC1
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/3
SW1(config-if)# description PC2
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/4
SW1(config-if)# description PC3
SW1(config-if)# exit
```

**Защо да добавяш description?**

```
Без description:
SW1# show interfaces status
Port      Status       Vlan
Fa0/1     connected    1
Fa0/2     connected    1
Fa0/3     connected    1
         ↑
    Кой кабел къде отива? 🤔

С description:
SW1# show interfaces status
Port      Name                  Status       Vlan
Fa0/1     Connection to R1      connected    1
Fa0/2     PC1                   connected    1
Fa0/3     PC2                   connected    1
         ↑
    Ясно! ✅
```

**Best practice:** ВИНАГИ добавяй descriptions!
- Помага при troubleshooting
- Добра документация
- Спестява време

**Синтаксис:**
```cisco
description [text до 240 символа]
```

---

### Стъпка 5: Записване

```cisco
SW1(config)# exit
SW1# copy running-config startup-config
```

**Или кратко:**
```cisco
SW1# wr
```

---

## ЧАСТ 4: Конфигурация на PC-та (10 мин)

### Как да конфигурирате IP адрес на PC:

1. Кликнете на **PC** икона
2. Изберете таб **Desktop**
3. Кликнете на **IP Configuration**
4. Изберете **Static** (не DHCP)
5. Въведете адресите

---

### Адресна таблица:

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | GigE0/0 | 192.168.1.1 | 255.255.255.0 | N/A |
| SW1 | VLAN 1 | 192.168.1.2 | 255.255.255.0 | 192.168.1.1 |
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | NIC | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| PC3 | NIC | 192.168.1.12 | 255.255.255.0 | 192.168.1.1 |

---

### За всеки PC (Desktop → IP Configuration):

**PC1:**
- IP Address: `192.168.1.10`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.1.1`

**PC2:**
- IP Address: `192.168.1.11`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.1.1`

**PC3:**
- IP Address: `192.168.1.12`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.1.1`

**Важно:**
- **Subnet Mask** трябва да е еднаква за всички (255.255.255.0)
- **Default Gateway** е винаги IP-то на Router-а (192.168.1.1)
- **IP адресите** трябва да са уникални (няма две устройства с един и същ IP)

---

### Какво означават адресите?

```
192.168.1.10   (маска /24)
│   │   │ │
│   │   │ └─ Host част (.10) - уникална, може да е от .1 до .254
├───┴───┴─── Мрежова част (192.168.1) - обща за всички устройства
└─────────── 192.168.x.x = частен адрес (само за локални мрежи)
```

**Защо всички са 192.168.1.X?**
- Защото са в **ЕДНА мрежа** (same subnet)
- Subnet mask 255.255.255.0 означава първите 3 октета са мрежа

**Защо gateway е .1?**
- Convention (традиция): първият адрес (.1) е за gateway
- Може да е и друг (напр. .254), но .1 е стандарт

---

## ЧАСТ 5: Тестване на свързаността (15 мин)

### Test 1: Ping от PC1 до Router

1. Кликнете на **PC1**
2. Изберете таб **Desktop**
3. Кликнете на **Command Prompt**
4. Напишете:

```
C:\> ping 192.168.1.1
```

**Очакван резултат:**
```
Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Успешен ping! ✅**

---

### Какво означава ping резултатът?

| Параметър | Значение | Какво означава |
|-----------|----------|----------------|
| **bytes=32** | Размер на данните | 32 байта полезен товар на ICMP (по подразбиране), не целият IP пакет |
| **time<1ms** | Време за отговор | Много бързо (локална мрежа) |
| **TTL=255** | Time To Live | Оставащата стойност в получения отговор; всеки маршрутизатор по пътя я намалява с 1 |
| **Lost = 0** | Загубени пакети | **0% = отлично!** |

**Ако първият ping е "Request timed out":**
```
Request timed out.                                  ⚠️ Първият
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255  ✅
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255  ✅
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255  ✅

Това е НОРМАЛНО! ✅
```

**Защо?** Изпращащото устройство изпълнява ARP, за да научи MAC адреса на 192.168.1.1; докато чака отговор, първата заявка може да изтече. Switch-ът не използва ARP: той научава MAC адресите от адреса на източника в получените кадри.

---

### Test 2: Ping между PC-та

```
От PC1:
C:\> ping 192.168.1.11
C:\> ping 192.168.1.12
```

**Трябва и двата да работят!**

**Какво се случва при ping от PC1 → PC2:**

```
1. PC1 подготвя ICMP Echo Request
   └─ Destination IP: 192.168.1.11
   └─ Destination MAC: ? (не знае още)

2. PC1 изпраща ARP Request (broadcast):
   "Кой има IP 192.168.1.11? Кажете ми MAC адреса!"
   └─ Switch научава MAC на PC1 и препраща кадъра към всички портове във VLAN 1, освен входящия

3. PC2 получава ARP Request:
   "Аз съм 192.168.1.11! Моят MAC е: AA:BB:CC:DD:EE:FF"
   └─ Switch "научава" MAC адреса на PC2

4. PC1 получава ARP Reply:
   Вече знае MAC адреса → изпраща ICMP Echo Request
   └─ Switch форвардва директно на PC2 (знае MAC-а)

5. PC2 изпраща ICMP Echo Reply
   └─ Switch форвардва на PC1

6. PC1 получава отговор: "Reply from 192.168.1.11"
```

---

### Test 3: Ping от Router до PC

```
R1# ping 192.168.1.10

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```

**Обяснение на резултата:**

| Символ | Значение |
|--------|----------|
| `!` | Успешен отговор ✅ |
| `.` | Timeout (няма отговор) ❌ |
| `U` | Destination unreachable ❌ |
| `Q` | Source quench (congestion) ⚠️ |
| `?` | Неизвестен пакет ❓ |

**Пример на неуспешен ping:**
```
R1# ping 192.168.1.99
.....
Success rate is 0 percent (0/5)
```

---

### Test 4: Проверка на ARP таблицата

```
R1# show arp
```

**Примерен резултат (след ping от/към всички устройства):**
```
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.1            -    0011.1111.1111  ARPA   GigabitEthernet0/0
Internet  192.168.1.2            0    0022.2222.2222  ARPA   GigabitEthernet0/0
Internet  192.168.1.10           0    0033.3333.3333  ARPA   GigabitEthernet0/0
Internet  192.168.1.11           0    0044.4444.4444  ARPA   GigabitEthernet0/0
Internet  192.168.1.12           0    0055.5555.5555  ARPA   GigabitEthernet0/0
```

**Забележка:** R1 има записи само за устройствата, с които е комуникирал. След Test 1 и Test 3 таблицата съдържа само 192.168.1.1 и 192.168.1.10.

**Какво е ARP таблица?**

**ARP (Address Resolution Protocol)** свързва **IP адреси** с **MAC адреси**.

```
Проблем: 
Router знае IP адреса (192.168.1.10)
Но Ethernet frames се изпращат към MAC адреси!

Решение:
ARP таблица: "192.168.1.10 = MAC 00:33:33:33:33:33"
```

**Аналогия:** ARP таблицата е като телефонен указател
- IP адрес = Име на човека
- MAC адрес = Телефонен номер

---

## ЧАСТ 6: Верификационни команди (10 мин)

### На Router:

#### 1. `show ip interface brief`

```cisco
R1# show ip interface brief
```

**Показва:**
```
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
GigabitEthernet0/1     unassigned      YES unset  administratively down down
```

**Обяснение на колоните:**

| Колона | Значение | Пример |
|--------|----------|--------|
| **Interface** | Име на интерфейса | GigabitEthernet0/0 |
| **IP-Address** | IP адрес (или unassigned) | 192.168.1.1 |
| **OK?** | Валиден ли е IP адресът | YES = адресът е валиден (не доказва, че интерфейсът работи) |
| **Method** | Как е получен IP-ът | manual = ръчно конфигуриран |
| **Status** | Физическо състояние (Layer 1) | **up** = кабел свързан ✅ |
| **Protocol** | Логическо състояние (Layer 2) | **up** = протоколът работи ✅ |

**Комбинации Status/Protocol:**

| Status | Protocol | Значение |
|--------|----------|----------|
| **up** | **up** | Работи перфектно ✅ |
| **up** | **down** | Кабел свързан, но протокол не работи ⚠️ |
| **down** | **down** | Няма кабел ❌ |
| **administratively down** | **down** | Интерфейс е **shutdown** ❌ |

---

#### 2. `show running-config`

```cisco
R1# show running-config
```

**Показва ЦЯЛАТА активна конфигурация.**

**Полезни секции:**
```
hostname R1                          ← Hostname
!
interface GigabitEthernet0/0         ← Интерфейс конфиг
 description Connection to SW1
 ip address 192.168.1.1 255.255.255.0
!
line console 0                       ← Конзола конфиг
 password cisco
 login
 logging synchronous
!
```

**Забележка:** `no shutdown` не се показва в running-config. Изключен интерфейс показва ред `shutdown`.

---

#### 3. `show ip route`

```cisco
R1# show ip route
```

**Показва routing таблицата:**
```
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP

     192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.1.0/24 is directly connected, GigabitEthernet0/0
L       192.168.1.1/32 is directly connected, GigabitEthernet0/0
```

**Какво означава:**
- `C` = **Connected route** (директно свързана мрежа)
- `L` = **Local route** (собственият адрес на интерфейса, /32)
- `192.168.1.0/24` = Мрежата
- `GigabitEthernet0/0` = През кой интерфейс

---

### На Switch:

#### 1. `show ip interface brief`

```cisco
SW1# show ip interface brief
```

**Очакван резултат:**
```
Interface              IP-Address      OK? Method Status                Protocol
Vlan1                  192.168.1.2     YES manual up                    up
FastEthernet0/1        unassigned      YES unset  up                    up
FastEthernet0/2        unassigned      YES unset  up                    up
```

**Забележка:** Switch портовете **нямат** IP адреси (Layer 2 устройство)

---

#### 2. `show vlan brief`

```cisco
SW1# show vlan brief
```

**Показва:**
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5-24, Gi0/1-2
```

**Обяснение:**
- Всички портове са в **VLAN 1** (default)
- VLAN 1 е active

---

#### 3. `show mac address-table`

```cisco
SW1# show mac address-table
```

**Показва:**
```
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0011.1111.1111    DYNAMIC     Fa0/1
   1    0033.3333.3333    DYNAMIC     Fa0/2
   1    0044.4444.4444    DYNAMIC     Fa0/3
   1    0055.5555.5555    DYNAMIC     Fa0/4
```

**Какво е MAC address table?**

Switch-ът "научава" кой MAC адрес е на кой порт:
```
PC1 (MAC: 00:33:33:33:33:33) е на порт Fa0/2
PC2 (MAC: 00:44:44:44:44:44) е на порт Fa0/3

Когато получи frame за PC1:
└─ Гледа таблицата → форвардва само на Fa0/2 ✅
```

**Type: DYNAMIC** означава switch-ът е научил автоматично (не е ръчно конфигурирано)

---

#### 4. `show interfaces status`

```cisco
SW1# show interfaces status
```

**Показва:**
```
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     Connection to R1   connected    1          a-full  a-100 10/100BaseTX
Fa0/2     PC1                connected    1          a-full  a-100 10/100BaseTX
Fa0/3     PC2                connected    1          a-full  a-100 10/100BaseTX
Fa0/4     PC3                connected    1          a-full  a-100 10/100BaseTX
Fa0/5                        notconnect   1          auto    auto  10/100BaseTX
```

**Обяснение на колоните:**

| Колона | Значение | Пример |
|--------|----------|--------|
| **Port** | Номер на порта | Fa0/1 |
| **Name** | Description (ако има) | Connection to R1 |
| **Status** | connected / notconnect | connected ✅ |
| **Vlan** | Към кой VLAN принадлежи | 1 (default) |
| **Duplex** | Full/Half duplex | a-full (auto-negotiated full) |
| **Speed** | Скорост | a-100 (100 Mbps) |


---

### Добавете още един PC

**Стъпки:**
1. Добавете PC4 в топологията
2. Свържете го към Switch на порт **Fa0/5**
3. Конфигурирайте:
   - IP: `192.168.1.13`
   - Subnet Mask: `255.255.255.0`
   - Default Gateway: `192.168.1.1`
4. Добавете description на порт Fa0/5:
   ```cisco
   SW1(config)# interface FastEthernet0/5
   SW1(config-if)# description PC4
   ```
5. Тествайте:
   ```
   От PC4: ping 192.168.1.1
   От PC4: ping 192.168.1.10
   ```

**Очакван резултат:** Всички ping-ове успешни ✅

Запазете Packet Tracer файла си като: Lab1_YourName.pkt
---

## ДОПЪЛНИТЕЛНИ ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 2: Променете hostname-овете

**Стъпки:**

**На Router:**
```cisco
R1# configure terminal
R1(config)# hostname MyRouter
MyRouter(config)# exit
MyRouter# copy running-config startup-config
```

**На Switch:**
```cisco
SW1# configure terminal
SW1(config)# hostname MySwitch
MySwitch(config)# exit
MySwitch# copy running-config startup-config
```

**Проверка:**
- Prompt-ът трябва да се промени на `MyRouter#` и `MySwitch#`

---

### Задача 3: Banner съобщение

**Banner** е съобщение, което се показва преди login.

#### Метод 1: С разделител # (стандартен)

```cisco
R1# configure terminal
R1(config)# banner motd #
Enter TEXT message. End with the character '#'.
************************************************
*  Unauthorized access is strictly prohibited  *
*  All activities are logged                   *
************************************************
#
R1(config)#
```

**Важно:**
- Напишете `banner motd #` и натиснете Enter
- Напишете текста на съобщението (може да е на множество редове)
- Завършете с **#** на нов ред
- **НЕ** използвайте символа # В ТЕКСТА (иначе ще приключи по-рано)

---

#### Метод 2: С различен разделител (ако # не работи)

```cisco
R1(config)# banner motd $
Enter TEXT message. End with the character '$'.
************************************************
*  Unauthorized access is strictly prohibited  *
*  All activities are logged                   *
************************************************
$
```

**Може да използвате:** `$`, `%`, `@`, `&`, или друг символ

---

#### Метод 3: Кратък banner (ако има проблем с многоредовия)

```cisco
R1(config)# banner motd #Unauthorized access prohibited!#
```

**Всичко на един ред:** 
- Започва с #
- Текст
- Завършва с #

---

#### Проверка:

1. **Излезте от router-а:**
   ```cisco
   R1# exit
   ```

2. **Влезте отново:**
   - Кликнете на Router → CLI tab
   - Натиснете Enter

3. **Трябва да видите:**

```
************************************************
*  Unauthorized access is strictly prohibited  *
*  All activities are logged                   *
************************************************

User Access Verification
Password:
```

---

#### Ако не работи:

**Проблем:** Banner-ът не се показва

**Възможни причини:**
1. Забравили сте `#` в края
2. Използвали сте `#` В текста (прекъсва banner-а)
3. Packet Tracer bug (рядко)

**Решение:**
```cisco
! Проверете текущия banner:
R1# show running-config | include banner

! Изтрийте banner-а:
R1(config)# no banner motd

! Опитайте с различен разделител:
R1(config)# banner motd $
Unauthorized access prohibited!
$
```

---

#### Banner варианти:

| Команда | Кога се показва |
|---------|-----------------|
| `banner motd` | **Message of the Day** – при свързване, преди login банера и удостоверяването |
| `banner login` | Преди login prompt |
| `banner exec` | След успешен login |

**За лаба използвайте `banner motd` (най-често срещан)**

---

### Задача 4: Създайте второ мрежово устройство

**Стъпки:**

1. **Добавете SW2 (втори Switch):**
   - Добавете Switch 2960
   - Hostname: `SW2`

2. **Свържете SW1 → SW2:**
   - SW1 Fa0/24 ←→ SW2 Fa0/24 (кабел **Copper Cross-Over**)

3. **Конфигурирайте SW2:**
   ```cisco
   Switch> enable
   Switch# configure terminal
   Switch(config)# hostname SW2
   SW2(config)# no ip domain-lookup
   SW2(config)# enable secret class123
   SW2(config)# interface vlan 1
   SW2(config-if)# ip address 192.168.1.3 255.255.255.0
   SW2(config-if)# no shutdown
   SW2(config-if)# exit
   SW2(config)# ip default-gateway 192.168.1.1
   ```

4. **Добавете descriptions:**
   ```cisco
   SW1(config)# interface Fa0/24
   SW1(config-if)# description Connection to SW2
   
   SW2(config)# interface Fa0/24
   SW2(config-if)# description Connection to SW1
   ```

5. **Добавете PC5 и PC6:**
   - Свържете към SW2 на портове Fa0/2 и Fa0/3
   - PC5: `192.168.1.14` 
   - PC6: `192.168.1.15`

6. **Тествайте:**
   ```
   От PC5: ping 192.168.1.1  (Router)
   От PC5: ping 192.168.1.10 (PC1 на SW1)
   От PC1: ping 192.168.1.14 (PC5 на SW2)
   ```

**Очакван резултат:** Всички устройства могат да ping-ват едни други ✅

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ И РЕШЕНИЯ

### Проблем 1: "Destination host unreachable"

**Симптоми:**
```
C:\> ping 192.168.1.1
Reply from 192.168.1.10: Destination host unreachable.
```

**Причини:**

| Причина | Проверка | Решение |
|---------|----------|---------|
| Router интерфейсът е shutdown | `R1# show ip int brief` | `R1(config-if)# no shutdown` |
| Грешен subnet mask | `ipconfig` на PC | Поправете на 255.255.255.0 |
| Грешен IP адрес | `ipconfig` на PC | Проверете че е в 192.168.1.X |
| Прекъснат кабел или грешен порт | Цвят на връзката в Packet Tracer | Свържете кабела към правилния порт |

---

### Проблем 2: Червени триъгълници на връзките

**Симптоми:** Връзките са с червени, а не със зелени триъгълници

**Причини:**
- Интерфейсите не са активирани
- Packet Tracer все още "стартира"

**Решения:**

1. **Изчакайте 30-60 секунди** (устройствата се инициализират)

2. **Fast Forward Time:**
   - Долна лента, вляво до часовника → бутон **Fast Forward Time** (>>)
   - Ускорява времето в симулацията

3. **Проверете интерфейсите:**
   ```cisco
   R1# show ip interface brief
   ! Трябва да е "up up"
   ```

4. **Ако са administratively down:**
   ```cisco
   R1(config)# interface g0/0
   R1(config-if)# no shutdown
   ```

---

### Проблем 3: PC не получава IP (или не може да ping)

**Симптоми:**
```
C:\> ipconfig
IP Address: 0.0.0.0
```

**Причина:** IP конфигурацията не е запазена или е на DHCP

**Решение:**
1. PC → Desktop → IP Configuration
2. Изберете **Static** (не DHCP)
3. Въведете IP, Subnet Mask, Gateway
4. Затворете и отворете отново за да проверите

---

### Проблем 4: Switch няма IP connectivity

**Симптоми:**
```
От PC: ping 192.168.1.2
Request timed out.
```

**Причини и решения:**

| Причина | Проверка | Решение |
|---------|----------|---------|
| VLAN1 интерфейс е shutdown | `SW1# show ip int brief` | `SW1(config-if)# no shutdown` на interface vlan 1 |
| Няма активен порт във VLAN 1 (SVI остава down) | `SW1# show interfaces status` | Свържете устройство към порт във VLAN 1 |
| Грешен IP адрес | `SW1# show ip int brief` | Поправете IP адреса |

---

### Проблем 5: Banner не се показва

**Причина:** Грешен синтаксис или разделител

**Решение:**
```cisco
! Изтрий стария banner
R1(config)# no banner motd

! Направи нов с прост текст
R1(config)# banner motd $
Unauthorized Access Prohibited
$
```

---

### Проблем 6: "Invalid input detected"

**Симптоми:**
```
R1(config)# hsotname R1
              ^
% Invalid input detected at '^' marker.
```

**Причина:** Грешка в командата (typo)

**Решение:**
- Проверете spelling (в случая: `hostname`, не `hsotname`)
- Използвайте **Tab** за autocomplete
- Използвайте **?** за помощ:
  ```cisco
  R1(config)# host?
  hostname
  ```

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ Топологията е създадена правилно
☐ Router има hostname R1
☐ Switch има hostname SW1
☐ Всички интерфейси са описани (description)
☐ Router интерфейс GigE0/0 е активен (up/up)
☐ Switch VLAN1 има IP адрес 192.168.1.2
☐ Switch има default-gateway 192.168.1.1
☐ Всички PC-та имат правилни IP настройки
☐ Ping работи от PC до Router
☐ Ping работи между PC-та
☐ Ping работи от Router до PC
☐ PC4 е добавен на Fa0/5 и ping работи
☐ Файлът е запазен като Lab1_YourName.pkt (след PC4, за Lab 2)
☐ Show команди показват правилна информация
☐ Конфигурацията е запазена (copy run start)
☐ Banner съобщение е добавено (optional)
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Какво е Router и Switch (разликите между тях)
2. ✅ Как се създава основна мрежова топология
3. ✅ Базова конфигурация на Router и Switch
4. ✅ Как се задават IP адреси, subnet mask, gateway
5. ✅ Важността на `no shutdown` команда
6. ✅ Как се тества свързаността с ping
7. ✅ Как се записва конфигурацията (running vs startup)
8. ✅ Основни show команди за проверка
9. ✅ Режимите на работа (User EXEC, Privileged, Config)
10. ✅ Troubleshooting при основни проблеми

---

## ⚠️ ВАЖНО ЗА СЛЕДВАЩИ ЛАБОВЕ

### Защо в Lab 2 ще сменим от VLAN 1?

**В този лаб използвахме VLAN 1 (default VLAN), защото:**
- Това е първо упражнение - фокус върху базова конфигурация
- Все още не сме учили какво е VLAN
- По-просто е за начинаещи

**❌ В реални проекти НИКОГА не използвайте VLAN 1 за потребителски трафик!**

### Проблеми със VLAN 1:

| Проблем | Защо е опасно |
|---------|--------------|
| **Всички портове по подразбиране** | Всеки нов порт автоматично е в VLAN 1 |
| **Management трафик** | Контролни протоколи на Cisco (CDP, VTP, DTP) използват VLAN 1 на trunk портовете |
| **Не може да се изтрие** | VLAN 1 е hardcoded, не може да се премахне |
| **Лесна цел за атаки** | Нападателите знаят че VLAN 1 съществува винаги |

### Best Practice за Lab 2 и нататък:
```
✅ Създавайте отделни VLAN-и:
   - VLAN 10, 20, 30... за различни отдели
   - VLAN 99 или 999 за management на switch-а
   
✅ Shutdown неизползваните портове:
   interface range Fa0/10-24
     switchport access vlan 999
     shutdown
     
✅ Не използвайте VLAN 1 за user трафик!
```

**В Lab 2 ще създадем правилна VLAN структура!**

---

## СЛЕДВАЩА СТЪПКА

В **Lab 2** ще научим:
- **VLAN-и** (виртуални локални мрежи)
- **Inter-VLAN routing** (Router-on-a-Stick)
- **Trunk портове** между switches
- **802.1Q tagging**

**Забележка:** В Lab 2 продължавате от `Lab1_YourName.pkt` и заменяте единната мрежа 192.168.1.0/24 с VLAN 10, 20 и 99. Адресите на R1 Gi0/0 (192.168.1.1) и SW1 Vlan1 (192.168.1.2) от това упражнение ще бъдат премахнати.

**Запазете Packet Tracer файла си като:** `Lab1_YourName.pkt` (след добавянето на PC4, преди допълнителните задачи)

---

## БЪРЗИ КОМАНДИ ЗА REFERENCE

### Основна конфигурация:
```cisco
enable                              ← Privileged mode
configure terminal                  ← Config mode
hostname R1                         ← Задай име
no ip domain-lookup                 ← Изключи DNS lookup
enable secret class123              ← Парола за enable
line console 0                      ← Конзола конфиг
  password cisco
  login
  logging synchronous
```

### Интерфейс конфигурация:
```cisco
interface g0/0
  ip address 192.168.1.1 255.255.255.0
  description Connection to SW1
  no shutdown
```

### Switch VLAN1:
```cisco
interface vlan 1
  ip address 192.168.1.2 255.255.255.0
  no shutdown
ip default-gateway 192.168.1.1
```

### Записване:
```cisco
copy running-config startup-config
! или
wr
```

### Show команди:
```cisco
show ip interface brief             ← IP адреси и статус
show running-config                 ← Активна конфиг
show startup-config                 ← Запазена конфиг
show ip route                       ← Routing таблица
show vlan brief                     ← VLAN-и
show mac address-table              ← MAC адреси
show arp                            ← ARP таблица
```

---

**Успех с упражнението!**


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
