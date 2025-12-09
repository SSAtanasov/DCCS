# УПРАЖНЕНИЕ 6: IoT ИНТЕГРАЦИЯ И АВТОМАТИЗАЦИЯ
## Smart Office с IoT устройства в Packet Tracer

**Цел:** Да се научите да интегрирате IoT устройства, да ги програмирате и да създавате автоматизация.

**Продължителност:** 75-90 минути

**Prerequisite:** Завършени Упражнения 1-5

---

## ЧАСТ 1: IoT архитектура в Packet Tracer (15 мин)

### Какво е IoT (Internet of Things)?

Мрежа от физически устройства с вградени сензори, софтуер и мрежова свързаност, които събират и обменят данни.

### IoT архитектура в Packet Tracer:

```
IoT Устройства → WiFi → IoT Server → SBC-PT/MCU-PT → Автоматизация
     (сензори,           (регистрация,    (програмиране,
     актуатори)          контрол)         логика)
```

### Основни компоненти:

**Мрежова инфраструктура:**
- **IoT Server** - централен сървър за регистрация и управление на устройства
- **Access Point** - безжична точка за достъп

**Контролери за автоматизация:**
- **SBC-PT** (Single Board Computer) - контролер с Blockly и Python за програмиране
- **MCU-PT** (Microcontroller Unit) - микроконтролер само с Blockly

**IoT устройства:**
- **IoT Sensors** - сензори (движение, температура, светлина, дим и др.)
- **IoT Actuators** - актуатори (лампи, вентилатори, врати, сирени и др.)

### Разлика между SBC-PT и MCU-PT

#### Сравнителна таблица:

| Характеристика | MCU-PT | SBC-PT |
|----------------|--------|--------|
| **Реален аналог** | Arduino, ESP32, ESP8266 | Raspberry Pi, BeagleBone, Jetson Nano |
| **Програмиране** | Само Blockly | Blockly + Python + JavaScript |
| **Сложност на логиката** | Прости if-then условия | Сложна логика, променливи, функции, класове |
| **Брой устройства** | 2-3 сензора/актуатора | Множество устройства едновременно (10+) |
| **Памет и производителност** | Ограничена (KB-MB RAM) | По-голяма (GB RAM, многоядрен процесор) |
| **Мрежова свързаност** | WiFi основно | WiFi + Ethernet + множество протоколи |
| **Обработка на данни** | Базова филтрация | Статистика, машинно обучение, бази данни |
| **Операционна система** | Firmware (вграден код) | Linux-базирана ОС |
| **Употреба** | Основни автоматизации | Професионални и комплексни проекти |
| **Енергопотребление** | Много ниско (mW-W) | Умерено (W) |
| **Цена (реален свят)** | $5-20 | $35-100 |
| **Време за разработка** | Бързо, прост код | По-дълго, но по-мощно |
| **Обучителна стойност** | Бърз старт за начинаещи | Професионални умения |

#### Препоръка за упражнението:

**Използваме SBC-PT**, защото:
- ✅ По-универсален и мощен контролер
- ✅ Подходящ за курсови проекти с множество устройства
- ✅ Python като допълнителна възможност за напреднали студенти
- ✅ Подготовка за професионална IoT разработка

#### Кога да използваме MCU-PT:

- ✓ Прости автоматизации с 1-2 устройства
- ✓ Енергийно ефективни проекти
- ✓ Бърз прототип без сложна логика

#### Кога да използваме SBC-PT:

- ✓ Комплексни системи с множество IoT устройства (5+)
- ✓ Необходимост от обработка на данни и статистика
- ✓ Интеграция с външни системи
- ✓ Професионални и индустриални приложения

### Процес на работа:

1. **Регистрация:** IoT устройствата се регистрират към IoT Server
2. **Ръчен контрол:** Чрез Web Browser можем да управляваме устройствата
3. **Автоматизация:** SBC-PT програмираме за автоматични реакции

---

## ЧАСТ 2: Топология на мрежата (20 мин)

### Разширена топология:

```
                [Router R1]
                     |
              GigE0/0 (trunk)
                     |
                [Switch SW1]
          /     |      |       \        \
       DNS   Admin   IoT    Access   SBC-PT
      Server  PC3   Server   Point
                                |
                           (Wireless)
                           /    |    \
                      Motion  Smart  Temp
                      Detect   LED   Sensor
```

### Устройства за добавяне:

**Мрежови компоненти:**
1. **Access Point-PT** - за WiFi свързаност на IoT устройствата
2. **Server-PT** (IoT Server) - за регистрация и централизирано управление
3. **SBC-PT** - за програмиране на автоматизация

**IoT устройства:**
1. **Motion Detector** (Sensors → Motion Detector)
2. **Smart LED** (Actuators → Smart LED или Street Lamp)
3. **Temperature Monitor** (Sensors → Temperature Monitor)
4. **Fan** (Actuators → Fan)

---


## ЧАСТ 3: Конфигурация на Access Point (15 мин)

### Стъпка 1: Добавяне и свързване

**Добавете Access Point-PT:**
- Devices → Network Devices → Wireless Devices → **Access Point-PT**
- Плъзнете го в топологията

**Физическа свързаност:**
- Copper Straight-Through кабел
- **FastEthernet0** на Access Point → **SW1 FastEthernet0/8**

### Стъпка 2: Конфигурация на порта на комутатора

```cisco
SW1# configure terminal
SW1(config)# interface FastEthernet0/8
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description IoT-AccessPoint
SW1(config-if)# exit
```

**Защо VLAN 20?** IoT устройствата ще бъдат в отделен IoT VLAN за сигурност.

### Стъпка 3: IP и Wireless конфигурация на Access Point

⚠️ **ВАЖНО:** Access Point-PT се конфигурира през **GUI таб**, НЕ през Config!

**1. Кликнете на Access Point в топологията**

**2. Ще се отвори прозорец с табове:**
- Physical
- Config  
- **GUI** ← Отидете тук!

**3. В GUI таба ще видите конфигурационен екран:**

```
┌─────────────────────────────────────────┐
│ Access Point Setup                      │
├─────────────────────────────────────────┤
│ Port 1 (Network Settings)               │
│   ( ) DHCP                              │
│   (•) Static                            │
│                                         │
│   IP Address:    [192.168.20.51    ]   │
│   Subnet Mask:   [255.255.255.0    ]   │
│   Gateway:       [192.168.20.1     ]   │
│                                         │
├─────────────────────────────────────────┤
│ Wireless Settings                       │
│   SSID:          [SmartOffice      ]   │
│   Authentication:[WPA2-PSK ▼]          │
│   Passphrase:    [SecureIoT123     ]   │
│                                         │
│        [Apply]  [OK]  [Cancel]          │
└─────────────────────────────────────────┘
```

**4. Попълнете полетата:**

**Port 1 (Network Settings):**
- Изберете: **Static** (махнете точката от DHCP)
- IP Address: `192.168.20.51`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.20.1`

**Wireless Settings:**
- SSID: `SmartOffice` (внимавайте - case sensitive!)
- Authentication: Изберете `WPA2-PSK` от dropdown
- PSK Pass Phrase: `SecureIoT123`

**5. Кликнете Apply или OK**

**6. Изчакайте 2-3 секунди**

**Забележка:** Ако не виждате GUI таба или показва само физическия изглед:
- Затворете прозореца
- Кликнете отново на Access Point
- Изчакайте 2-3 секунди

### Стъпка 4: Верификация

**От PC3 (Admin VLAN 20):**

Първо проверете дали PC3 има IP:
```
ipconfig
```

Ако няма, направете:
```
ipconfig /release
ipconfig /renew
```

След това:
```
ping 192.168.20.51
```

**Очакван резултат:**
```
Pinging 192.168.20.51 with 32 bytes of data:
Reply from 192.168.20.51: bytes=32 time<1ms TTL=64
Reply from 192.168.20.51: bytes=32 time<1ms TTL=64

Ping statistics:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

✅ Access Point е конфигуриран правилно!
```

**Ако ping НЕ работи:**
1. Проверете дали Access Point има зелена светлина (Power ON)
2. Проверете physical кабела (трябва да е свързан)
3. Върнете се в GUI и проверете IP адреса
4. Проверете VLAN на SW1 Fa0/8 (`show vlan brief`)

---

---

## ЧАСТ 4: Конфигурация на IoT Server (20 мин)

### Стъпка 1: Добавяне на сървъра

**Добавете Server-PT:**
- End Devices → Server-PT
- Свържете към SW1 FastEthernet0/9

### Стъпка 2: Конфигурация на порта на комутатора

```cisco
SW1(config)# interface FastEthernet0/9
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description IoT-Server
SW1(config-if)# exit
```

### Стъпка 3: IP конфигурация на сървъра

**Server → Config → FastEthernet0:**

```
IP Configuration: Static
IP Address: 192.168.20.100
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
DNS Server: 192.168.10.2
```

### Стъпка 4: Активиране на IoT услугата

**Server → Services → IoT:**

```
Service: ON
Port: 8080
Username: admin (по подразбиране)
Password: admin (по подразбиране)
```

### Стъпка 5: Достъп до IoT Dashboard

**От Admin PC3 → Desktop → Web Browser:**

```
URL: http://192.168.20.100:8080
```

**Трябва да видите:**
- Login екран (ако има)
- IoT Dashboard интерфейс
- Списък с устройства (все още празен)

---

## ЧАСТ 5: Добавяне и регистрация на IoT сензор (20 мин)

### Стъпка 1: Добавяне на Motion Detector

**Добавете устройството:**
- IoT → Sensors → Motion Detector
- Поставете го в топологията (близо до Access Point за добър сигнал)

### Стъпка 2: Конфигурация на WiFi връзката

**Motion Detector → Config → Wireless0:**

```
SSID: SmartOffice
Authentication: WPA2-PSK
PSK Pass Phrase: SecureIoT123
```

**Изчакайте:** Устройството трябва да се свърже към Access Point (виждате икона за WiFi).

### Стъпка 3: Регистрация към IoT Server

**Motion Detector → Config → IoT Server:**

```
Server IP: 192.168.20.100
Registration Port: 8080
User: admin
Password: admin
```

**Кликнете бутон: Connect**

**Резултат:** Съобщение "Connected to IoT Server" или подобно.

### Стъпка 4: Проверка на регистрацията

**От Admin PC3 → Web Browser → http://192.168.20.100:8080**

**Трябва да видите:**
- Motion Detector в списъка с устройства
- Неговото име (напр. "MotionDetector0")
- Състояние: Connected
- Текуща стойност на сензора

---

## ЧАСТ 6: Добавяне на IoT актуатор (15 мин)

### Стъпка 1: Добавяне на Smart LED

**Добавете устройството:**
- IoT → Actuators → Smart LED (или Street Lamp за по-визуален ефект)
- Поставете го в топологията

### Стъпка 2: Конфигурация на WiFi връзката

**Smart LED → Config → Wireless0:**

```
SSID: SmartOffice
Authentication: WPA2-PSK
PSK Pass Phrase: SecureIoT123
```

### Стъпка 3: Регистрация към IoT Server

**Smart LED → Config → IoT Server:**

```
Server IP: 192.168.20.100
Registration Port: 8080
User: admin
Password: admin
```

**Кликнете: Connect**

### Стъпка 4: Тест на ръчно управление

**От Admin PC3 → Web Browser → http://192.168.20.100:8080**

1. Намерете Smart LED в списъка
2. Кликнете върху устройството
3. Потърсете бутон **ON/OFF** или **Toggle**
4. Кликнете бутона

**Резултат:** LED трябва да светне/угасне визуално в топологията!

**Това е основният начин за ръчен контрол на IoT устройствата.**

---

## ЧАСТ 7: Добавяне на SBC-PT контролер (25 мин)

### Какво е SBC-PT?

Single Board Computer - контролер, който може да програмирате за автоматизация на IoT устройства.

**Възможности:**
- Blockly (визуално програмиране)
- Python (текстово програмиране)
- Достъп до всички устройства регистрирани към IoT Server

### Стъпка 1: Добавяне на SBC-PT

**Добавете:**
- End Devices → SBC-PT
- Свържете FastEthernet към SW1 FastEthernet0/10

### Стъпка 2: Конфигурация на порта на комутатора

```cisco
SW1(config)# interface FastEthernet0/10
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description IoT-Controller-SBC
SW1(config-if)# exit
```

### Стъпка 3: IP конфигурация на SBC-PT

**SBC-PT → Config → FastEthernet0:**

```
IP Configuration: Static
IP Address: 192.168.20.60
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
```

### Стъпка 4: Свързване на SBC към IoT Server

**SBC-PT → Config → Settings (или IoT Server раздел):**

```
IoT Server IP: 192.168.20.100
Port: 8080
Username: admin
Password: admin
```

**ВАЖНО:** SBC може да управлява всички устройства, регистрирани към IoT Server.

### Стъпка 5: Проверка на връзката

**SBC-PT → Config → Settings**

Трябва да видите статус: "Connected to IoT Server"

---

## ЧАСТ 8: Програмиране на автоматизация с Blockly (30 мин)

### Сценарий: При засичане на движение → включи лампата

### Стъпка 1: Отваряне на Programming интерфейса

**SBC-PT → Programming таб**

**Изберете език:** Blockly

### Стъпка 2: Идентифициране на устройствата

**КРИТИЧНО ВАЖНО:** Трябва да знаете точните имена на устройствата!

**От Admin PC3 → Web Browser → http://192.168.20.100:8080**

Запишете имената:
- Motion Detector може да е: "MotionDetector0" или "MotionDetector1"
- Smart LED може да е: "SmartLED0" или "StreetLamp0"

**Използвайте ТОЧНО тези имена в Blockly!**

### Стъпка 3: Създаване на Blockly програмата

**Логика:**
```
When MotionDetector0 detects motion
    → Set SmartLED0 to ON
```

**Визуално представяне:**
```
┌─────────────────────────────────────────┐
│ Events:                                  │
│   When [MotionDetector0] detects motion │
│                                          │
│ Actions:                                 │
│   Set device [SmartLED0] state to ON    │
└─────────────────────────────────────────┘
```

**Стъпки в Blockly:**

1. От **Events** секция → Drag "When device detects..."
2. В dropdown менюто изберете вашия Motion Detector (напр. "MotionDetector0")
3. Изберете тип събитие: "detects motion"
4. От **Actions** секция → Drag "Set device status..."
5. Изберете Smart LED от dropdown менюто
6. Изберете състояние: ON

### Стъпка 4: Добавяне на изключване

**За автоматично изключване при липса на движение:**

```
┌─────────────────────────────────────────┐
│ Events:                                  │
│   When [MotionDetector0] no motion      │
│                                          │
│ Actions:                                 │
│   Set device [SmartLED0] state to OFF   │
└─────────────────────────────────────────┘
```

**Това ще създаде втори блок под първия.**

### Стъпка 5: Стартиране на програмата

**Кликнете бутон: Start** (или Run, зависи от интерфейса)

**Резултат:** 
- Програмата започва да работи
- SBC започва да "слуша" за събития от Motion Detector

### Стъпка 6: Тестване на автоматизацията

**Превключете към Simulation Mode:**
1. Долу дясно → Simulation Mode
2. Кликнете на Motion Detector в топологията
3. Активирайте сензора (зона пред него - виждате червен конус/област)
4. **Smart LED трябва да светне автоматично!**
5. Деактивирайте движението
6. **Smart LED трябва да угасне автоматично!**

**Ако не работи - вижте раздел Troubleshooting по-долу.**

---

## ЧАСТ 9: Разширена автоматизация - Temperature Control (20 мин)

### Сценарий: При температура > 26°C → включи вентилатора

### Стъпка 1: Добавяне на допълнителни устройства

**Добавете:**
1. **Temperature Monitor** (IoT → Sensors → Temperature Monitor)
2. **Fan** (IoT → Actuators → Fan)

**За двете устройства:**
- Конфигурирайте WiFi (SSID: SmartOffice)
- Регистрирайте към IoT Server (192.168.20.100:8080)
- Проверете в Dashboard че са регистрирани

### Стъпка 2: Програмиране в Blockly

**Псевдокод:**
```python
if TemperatureSensor.value > 26:
    Fan.state = ON
else:
    Fan.state = OFF
```

**Blockly блокове:**

```
┌─────────────────────────────────────────────┐
│ Control:                                     │
│   If [TemperatureSensor0] value > 26        │
│   then:                                      │
│       Set device [Fan0] state to ON         │
│   else:                                      │
│       Set device [Fan0] state to OFF        │
└─────────────────────────────────────────────┘
```

### Стъпка 3: Блокове за използване

1. **Control** → "if... then... else..."
2. **Sensors** → "Get sensor value" → Изберете TemperatureSensor0
3. **Operators** → ">" (по-голямо от)
4. **Math** → Number → Въведете 26
5. **Actions** → "Set device status" → Fan0 → ON
6. **Actions** → "Set device status" → Fan0 → OFF (в else частта)

### Стъпка 4: Тестване

**В Simulation Mode:**
1. Кликнете на Temperature Monitor
2. Променете температурата ръчно → 28°C
3. **Fan трябва да се включи!**
4. Намалете температурата → 24°C
5. **Fan трябва да се изключи!**

### Стъпка 5: Комбинирана програма

**Можете да имате и двете автоматизации едновременно:**
- Движение → LED
- Температура → Fan

И двете ще работят паралелно!

---

## ЧАСТ 10: IoT Security - Защита на мрежата (15 мин)

### Best Practices за IoT сигурност:

#### 1. Отделен VLAN за IoT устройства

```cisco
R1(config)# vlan 30
R1(config-vlan)# name IoT_Devices
R1(config-vlan)# exit
!
R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 192.168.30.1 255.255.255.0
R1(config-subif)# exit
!
SW1(config)# interface FastEthernet0/8
SW1(config-if)# switchport access vlan 30
```

#### 2. ACL за ограничаване на достъпа

**Цел:** IoT устройствата могат да комуникират само с IoT Server и SBC-PT.

```cisco
R1(config)# access-list 120 remark IoT-Security-Policy
R1(config)# access-list 120 permit ip 192.168.30.0 0.0.0.255 host 192.168.20.100
R1(config)# access-list 120 permit ip 192.168.30.0 0.0.0.255 host 192.168.20.60
R1(config)# access-list 120 deny ip 192.168.30.0 0.0.0.255 any log
R1(config)# access-list 120 permit ip any any
!
R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# ip access-group 120 in
```

#### 3. Силни пароли и автентикация

**Променете паролите по подразбиране:**
- IoT Server: username/password → силни стойности (мин. 12 символа)
- WiFi PSK: SecureIoT123 → по-силна парола (мин. 16 символа)
- Access Point: admin пароли → уникални стойности

#### 4. Допълнителни мерки

- **Мониторинг:** Проверявайте логовете на IoT Server редовно
- **Актуализации:** Поддържайте firmware-а актуален (когато е възможно)
- **Минимални привилегии:** Всяко устройство има достъп само до необходимото
- **Network Segmentation:** IoT мрежата е изолирана от корпоративната

---

## ЧАСТ 11: Примерни IoT сценарии за курсов проект

### Идеи за автоматизация в Smart Office/Home:

| # | Сценарий | Необходими устройства | Логика | Сложност |
|---|----------|----------------------|--------|----------|
| 1 | **Smart Lighting** | Motion Sensor, Light Level Sensor, LED Лампи | IF motion detected AND light < 30% → LED ON | ★☆☆ |
| 2 | **Climate Control** | Temperature Sensor, Humidity Sensor, Fan, AC | IF temp > 25°C OR humidity > 70% → Fan ON | ★★☆ |
| 3 | **Security System** | Motion Sensor, Door Sensor, Siren, LED | IF motion AND door=OPEN → Siren ON + LED Blink | ★★☆ |
| 4 | **Energy Saving** | Motion, Light Level, ALL devices | IF no motion 5min → Turn OFF all devices | ★★★ |

### Подробни примери:

#### Пример 1: Smart Lighting System

**Устройства:**
- Motion Detector (Sensor0)
- Light Level Sensor (Sensor1)
- LED Lamp (Actuator0)

**Blockly логика:**
```
WHEN Sensor0.motion = DETECTED
  IF Sensor1.lightLevel < 30
    SET Actuator0.status = ON
  ELSE
    SET Actuator0.status = OFF
```

**Бизнес логика:** Лампата се включва само когато има движение И светлината е под 30%. Икономия на енергия!

---

#### Пример 2: Temperature Control

**Устройства:**
- Temperature Sensor (TempSensor0)
- Fan (Fan0)

**Blockly логика:**
```
WHEN TempSensor0.temperature > 25
  SET Fan0.status = ON
ELSE
  SET Fan0.status = OFF
```

**Бизнес логика:** Вентилаторът се включва автоматично при висока температура.

---

#### Пример 3: Integrated Security + Comfort

**Устройства:**
- Motion Sensor
- Door Sensor
- Temperature Sensor
- LED, Siren, Fan

**Сложна логика:** Комбинация от сигурност и комфорт. Когато вратата се отвори:
1. Ако има движение → включи LED (добре дошъл)
2. Ако няма движение → включи сирена (нарушител)
3. При висока температура → включи вентилатор

---

### Съвети за курсовия проект:

1. **Започнете просто:** Един сензор + един актуатор
2. **Тествайте стъпка по стъпка:** Проверете всяко устройство поотделно
3. **Документирайте логиката:** Нарисувайте flowchart преди да кодирате
4. **Добавяйте сложност постепенно:** След като основата работи
5. **Използвайте смислени имена:** MotionDetector1, LivingRoomLED

---

## ЧАСТ 12: Документация за проекта

### Таблица с IoT устройства:

| Име в Dashboard | Тип | IP/ID | VLAN | Местоположение | Функция |
|----------------|-----|-------|------|----------------|---------|
| MotionDetector0 | Sensor | - | 20 | Entrance | Детектор за движение |
| SmartLED0 | Actuator | - | 20 | Entrance | LED лампа |
| TempSensor0 | Sensor | - | 20 | Office | Температурен сензор |
| Fan0 | Actuator | - | 20 | Office | Вентилатор |

### Мрежова конфигурация:

| Устройство | IP Address | Gateway | VLAN | Порт |
|------------|------------|---------|------|------|
| IoT Server | 192.168.20.100 | 192.168.20.1 | 20 | Fa0/7 |
| Access Point | 192.168.20.51 | 192.168.20.1 | 20 | Fa0/8 |
| SBC-PT | 192.168.20.60 | 192.168.20.1 | 20 | Fa0/9 |
| PC3 (Test) | 192.168.20.11 | 192.168.20.1 | 20 | Fa0/4 |

### Blockly програма (описание):

```
Програма: Smart_Office_Automation
Версия: 1.0
Автор: [Вашето име]

Логика:
1. WHEN Motion.status = DETECTED
   - IF LightLevel < 30 → LED ON
   - ELSE → LED OFF

2. WHEN Temperature > 25
   - Fan ON
   
3. WHEN Motion.status = NOT_DETECTED FOR 5min
   - ALL devices OFF (energy saving)
```

### Тестови сценарии:

| Test ID | Действие | Очакван резултат | Статус |
|---------|----------|------------------|--------|
| T01 | Симулирай движение | LED ON | ✅ Pass |
| T02 | Температура > 25°C | Fan ON | ✅ Pass |
| T03 | Няма движение 5min | All OFF | ✅ Pass |

---

## ЧАСТ 13: Troubleshooting - Отстраняване на проблеми

### Често срещани проблеми и решения:

| # | Проблем | Симптом | Проверка | Решение | Време |
|---|---------|---------|----------|---------|-------|
| 1 | **WiFi не се свързва** | Няма WiFi икона на устройството | SSID правилен? (SmartOffice - case sensitive!) | Изтрий SSID/парола изцяло, въведи отново. Измести по-близо до AP. | 2-5 мин |
| 2 | **Не се регистрира** | WiFi ОК, но не в Dashboard | IoT Server ON? Port 8080? | Проверка на http://192.168.20.100:8080. Refresh. Re-connect от устройството. | 3-5 мин |
| 3 | **Blockly не работи** | Програмата стартира, но не реагира | Имената в код съвпадат с Dashboard? | Провери ТОЧНИТЕ имена (case sensitive!). Проверка на логиката в Blockly. | 5-10 мин |
| 4 | **Сензор не реагира** | В Dashboard, но без events | Status в Dashboard "Active"? | Refresh Dashboard. Manual trigger test. Рестарт на устройство. | 3-5 мин |
| 5 | **Актуатор не се управлява** | Вижда се, но не реагира на команди | Connection status? | Test с Manual control от Dashboard. Проверка на WiFi. Restart. | 3-5 мин |
| 6 | **SBC-PT не вижда устройства** | SBC свързан, но списъкът празен | SBC свързан към IoT Server? | Config → IoT Server → Re-connect. Проверка на 192.168.20.100:8080 достъпност. | 5 мин |

---

### Детайлни стъпки за основни проблеми:

#### Проблем 1: WiFi връзка

**Стъпки:**
1. Устройство → Config → Wireless0
2. **Изтрийте** SSID и паролата изцяло
3. Въведете отново **ТОЧНО**:
   - SSID: `SmartOffice`
   - Authentication: WPA2-PSK
   - PSK Pass Phrase: `SecureIoT123`
4. Изчакайте 10-15 секунди
5. Проверете за WiFi икона
6. Ако не работи → измести устройството по-близо до Access Point

---

#### Проблем 2: Регистрация в IoT Server

**Стъпки:**
1. Проверете IoT Server:
   - Server → Services → IoT → **ON**
   - Port: 8080

2. Тест на свързаност:
   - PC3 → ping 192.168.20.100
   - PC3 → Web Browser → http://192.168.20.100:8080

3. Регистрация на устройството:
   - IoT устройство → Config → IoT Server
   - Server IP: 192.168.20.100
   - Registration Port: 8080
   - User: admin | Password: admin
   - Кликнете **Connect**

4. Проверка:
   - Refresh Dashboard
   - Устройството трябва да се появи

---

#### Проблем 3: Blockly програма

**Стъпки:**
1. Спрете програмата: SBC-PT → Programming → **Stop**

2. Проверете имената:
   - PC3 → Web Browser → http://192.168.20.100:8080
   - Запишете ТОЧНИТЕ имена (case sensitive!)
   - Пример: "MotionDetector0", "SmartLED0"

3. Проверете логиката:
   - Events блок → правилно устройство?
   - Actions блок → правилно устройство?
   - Състоянието (ON/OFF) правилно?

4. Тест:
   - Ръчно тествайте устройствата от Dashboard
   - Стартирайте програмата отново

---

### Debug съвети:

**Общи проверки:**
```
✓ Access Point включен? (Power ON, зелен индикатор)
✓ IoT Server Services → IoT активирана?
✓ Всички устройства в една WiFi мрежа? (SSID: SmartOffice)
✓ SBC-PT свързан към IoT Server?
✓ Имената в Blockly код съвпадат с Dashboard?
✓ Simulation Mode активиран за тестване?
```

**Бързи команди:**
- **Refresh Dashboard:** F5 или Ctrl+R в Web Browser
- **Рестарт на устройство:** Config → Power OFF → Power ON
- **Проверка на връзка:** ping 192.168.20.100

**Логове:**
- IoT Server → Services → IoT → вижте логовете за грешки
- SBC-PT → Programming → Console за debug съобщения

---

## ЧАСТ 14: Checklist за завършване

### Мрежова инфраструктура:
```
☐ Router конфигуриран (VLAN 20)
☐ Switch портове назначени
☐ Access Point добавен (192.168.20.51)
☐ IoT Server добавен (192.168.20.100)
☐ IoT Server Services → IoT = ON
☐ Dashboard достъпен (http://192.168.20.100:8080)
```

### IoT устройства:
```
☐ Минимум 1 сензор добавен
☐ Минимум 1 актуатор добавен
☐ Всички устройства свързани към WiFi (SmartOffice)
☐ Всички устройства регистрирани в IoT Server
☐ Устройствата се виждат в Dashboard
☐ Ръчният контрол работи от Dashboard
```

### Автоматизация:
```
☐ SBC-PT добавен (192.168.20.60)
☐ SBC-PT свързан към IoT Server
☐ Blockly програма създадена
☐ Логиката тествана и работи
☐ Автоматизацията реагира на събития
☐ Програмата записана
```

### Документация:
```
☐ Таблица с IoT устройства попълнена
☐ Мрежова схема нарисувана
☐ Blockly код документиран
☐ Тестови сценарии изпълнени
☐ Troubleshooting стъпки записани
```

---

## ЧАСТ 15: Какво научихме

1. ✅ IoT архитектура и компоненти (Server, AP, SBC, устройства)
2. ✅ Разлика между MCU-PT и SBC-PT
3. ✅ Конфигурация на Access Point за WiFi
4. ✅ Настройка на IoT Server за регистрация
5. ✅ Добавяне и регистрация на IoT сензори
6. ✅ Добавяне и контрол на IoT актуатори
7. ✅ Програмиране с Blockly (visual programming)
8. ✅ Създаване на автоматизация (events → actions)
9. ✅ IoT сигурност (VLAN, ACL, пароли)
10. ✅ Troubleshooting на IoT системи

---

## ЧАСТ 16: Подготовка за курсовия проект

### Изисквания за проекта:

**Минимални изисквания:**
- 2+ сензора
- 2+ актуатора
- SBC-PT с Blockly автоматизация
- Работеща логика (if-then правила)
- Документация (схема + таблици)

**Препоръчителни допълнения:**
- 3+ различни типа сензори
- Сложна логика с AND/OR условия
- Temperature control или Security система
- Python програмиране (напреднали)

### Стъпки за успешен проект:

1. **Планиране (1-2 часа)**
   - Избор на сценарий (Smart Home/Office/Security)
   - Списък с устройства
   - Flowchart на логиката

2. **Мрежова инфраструктура (1 час)**
   - Router, Switch, VLAN конфигурация
   - Access Point настройка
   - IoT Server инсталация

3. **IoT устройства (1-2 часа)**
   - Добавяне на сензори и актуатори
   - WiFi връзка
   - Регистрация в Dashboard
   - Тестване на ръчен контрол

4. **Автоматизация (2-3 часа)**
   - SBC-PT конфигурация
   - Blockly програмиране
   - Тестване на логиката
   - Debug и оптимизация

5. **Документация (1 час)**
   - Мрежова схема
   - Таблици с устройства
   - Описание на логиката
   - Тестови резултати

### Съвети:

- Започнете рано (не оставяйте за последния момент)
- Тествайте всяка стъпка преди да продължите
- Правете backup на Packet Tracer файла редовно
- Документирайте проблемите и решенията
- Използвайте смислени имена на устройствата

---

## ЧАСТ 17: Полезни ресурси

| Ресурс | URL | Описание |
|--------|-----|----------|
| Cisco Packet Tracer | netacad.com | Официален сайт за download |
| IoT Documentation | cisco.com/c/en/us/td/docs/cloud_services/iot | Официална документация |
| Blockly Guide | developers.google.com/blockly | Visual programming guide |
| Курсови проекти (примери) | github.com/cisco-iot-pt | Примерни проекти |

---

## Бърз старт за начинаещи (30 мин)

### Минимална работеща конфигурация:

**Цел:** 1 сензор + 1 актуатор + проста автоматизация

**Устройства:**
- Access Point (192.168.20.51)
- IoT Server (192.168.20.100)
- Motion Detector (Sensor)
- LED Lamp (Actuator)
- SBC-PT (192.168.20.60)

**Стъпки:**

1. **Мрежа:** Добави AP и Server в VLAN 20
2. **WiFi:** SSID: SmartOffice, Парола: SecureIoT123
3. **Регистрация:** Свържи устройствата към IoT Server
4. **Blockly:**
   ```
   WHEN MotionDetector0.motion = DETECTED
     SET SmartLED0.status = ON
   ```
5. **Тест:** Симулирай движение → LED трябва да светне

**Време:** 30 минути от нулата до работеща автоматизация!

---

## Заключение

Завършихте успешно **Упражнение 6: IoT Интеграция и Автоматизация**!

Вие сте готови за:
- ✅ Курсов проект с IoT устройства
- ✅ Програмиране на автоматизация
- ✅ Професионална IoT разработка (основи)

**Следваща стъпка:** Започнете работа по курсовия проект!

**Запазете файла като:** `Lab6_IoT_YourName.pkt`

---

**Успех с IoT автоматизацията! **



**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>