# LAB 6: IoT ИНТЕГРАЦИЯ И АВТОМАТИЗАЦИЯ
## Smart Office с IoT устройства в Packet Tracer

**Цел:** Да се научите да интегрирате IoT устройства, да ги програмирате и да създавате автоматизация.

**Продължителност:** 90-120 минути

**Prerequisite:** Завършени Lab 1-5

---

## ЧАСТ 1: IoT в Packet Tracer - Въведение (15 мин)

### Какво е IoT (Internet of Things)?
Мрежа от физически устройства с вградени сензори, софтуер и мрежова свързаност, които събират и обменят данни.

### IoT устройства в Packet Tracer:
- **Сензори:** Motion, Door, Temperature, Smoke, CO2, Light
- **Актуатори:** Fan, Lamp, Door, Siren, Garage Door
- **Контролери:** Home Gateway, MCU, SBC (Single Board Computer)

### Архитектура:
```
IoT Устройства → Home Gateway/MCU → Network → IoT Server → Мониторинг/Контрол
```

---

## ЧАСТ 2: Разширяване на топологията (20 мин)

### Нова топология:
```
                [Router R1]
                     |
              GigE0/0 (trunk)
                     |
                [Switch SW1]
          /     |     |      \      \
       DNS   Admin  IoT    IoT    IoT
      Server  PC   Gateway Sensor Lamp
```

### Устройства за добавяне:

**IoT компоненти:**
1. **Home Gateway** (от Network Devices → Wireless Devices)
2. **Motion Detector** (от Components → Sensors)
3. **Smart LED** (от Components → Actuators)
4. **Door** (от Components → Actuators)
5. **Temperature Monitor** (от Components → Sensors)

**Кабели:**
- Wireless или Ethernet (зависи от устройството)

---

## ЧАСТ 3: Конфигурация на IoT Gateway (25 мин)

### Стъпка 1: Физическа свързаност

**Home Gateway свързан към Switch:**
- Ethernet port на Gateway → SW1 FastEthernet0/7
- Назначете порта към VLAN 20 (IT VLAN)

```cisco
SW1(config)# interface FastEthernet0/7
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description IoT-Gateway
SW1(config-if)# exit
```

### Стъпка 2: IP конфигурация на Gateway

**Кликнете на Home Gateway → Config tab:**

**LAN (Wired):**
```
Статично
IP Address: 192.168.20.50
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
```

**Wireless0 (за IoT устройства):**
```
SSID: SmartOffice
Authentication: WPA2-PSK
Password: SecureIoT123
```

### Стъпка 3: Проверка на свързаността

**От Admin PC3:**
```
ping 192.168.20.50
```

Трябва да работи!

---

## ЧАСТ 4: Добавяне на IoT сензор - Motion Detector (20 мин)

### Стъпка 1: Добавяне на Motion Detector

**Добавете от Sensors → Motion Detector**
- Поставете го в топологията

### Стъпка 2: Wireless конфигурация

**Кликнете на Motion Detector → Config:**

```
Wireless Connection:
SSID: SmartOffice
Authentication: WPA2-PSK
Password: SecureIoT123
```

**Advanced:**
```
Клик на Advanced Tab
```

### Стъпка 3: Регистрация към Gateway

**На Motion Detector → Config → IoT Server:**
```
Изберете Home Gateway
```

### Стъпка 4: Верификация

**На PC3 → Web Browser → 192.168.20.50**
- Трябва да видите Motion Detector в списъка с регистрирани устройства

---

## ЧАСТ 5: Добавяне на актуатор - Smart LED (15 мин)

### Стъпка 1: Добавяне на Smart LED

**От Actuators → Smart LED (или Old Lamp за по-прост вариант)**

### Стъпка 2: Wireless конфигурация

**Същата като Motion Detector:**
```
SSID: SmartOffice
Password: SecureIoT123
```

### Стъпка 3: Регистрация

**IoT Server:**
```
Registration Server: 192.168.20.50
Username: admin
Password: admin
```

**Кликнете: Connect**

### Стъпка 4: Тест на ръчно управление

**На Home Gateway → IoT Server:**
- Намерете Smart LED в списъка
- Кликнете върху него
- **Toggle** бутон → LED трябва да светне/угасне

---

## ЧАСТ 6: Създаване на автоматизация (30 мин)

### Scenario: При засичане на движение → запали лампата

### Стъпка 1: На Home Gateway → Programming tab

**Изберете:**
- Language: **Blockly** (визуално програмиране)

### Стъпка 2: Програмиране на логиката

**Drag and drop блокове:**

```
┌─────────────────────────────────────────┐
│ When Motion Detector detects motion     │
│   Do:                                    │
│     Set Smart LED → ON                   │
└─────────────────────────────────────────┘
```

**Blockly код (visual blocks):**
1. От **Events** → Drag "When sensor detects..."
2. Изберете Motion Detector
3. От **Actions** → Drag "Set device status..."
4. Изберете Smart LED
5. Set to: ON

**Ако искате и изключване:**
```
┌─────────────────────────────────────────┐
│ When Motion Detector detects NO motion  │
│   Do:                                    │
│     Set Smart LED → OFF                  │
└─────────────────────────────────────────┘
```

### Стъпка 3: Run Programming

**Кликнете: Run**

### Стъпка 4: Тестване

**В Simulation Mode:**
1. Превключете към **Simulation Mode** (долу дясно)
2. Кликнете на Motion Detector
3. Активирайте го (кликнете в зоната пред сензора)
4. LED трябва да светне автоматично! 

---

## ЧАСТ 7: Advanced - Temperature Monitor и Fan Control (BONUS)

### Scenario: При температура > 25°C → включи вентилатора

### Стъпка 1: Добавяне на устройства
- Temperature Monitor (Sensor)
- Fan (Actuator)

### Стъпка 2: Регистрация към Gateway
(Същата процедура като Motion Detector)

### Стъпка 3: Програмиране

**Blockly логика:**
```python
# Pseudocode
if temperature > 25:
    set Fan to ON
else:
    set Fan to OFF
```

**Visual blocks:**
1. **Control** → "if... then... else..."
2. **Sensors** → "Temperature value"
3. **Operators** → ">" (greater than)
4. **Actions** → "Set Fan status"

### Стъпка 4: Симулация
- Кликнете на Temperature Monitor
- Променете температурата ръчно на 28°C
- Fan трябва да стартира!

---

## ЧАСТ 8: IoT Server за централен контрол (BONUS)

### Добавяне на IoT Server за мониторинг

### Стъпка 1: Добавяне на Server-PT

**Server конфигурация:**
```
IP: 192.168.20.100
Subnet: 255.255.255.0
Gateway: 192.168.20.1
```

### Стъпка 2: Активиране на IoT Service

**Server → Services → IoT:**
- Turn ON
- Set Port: 8080

### Стъпка 3: Регистрация на устройства

**На всяко IoT устройство:**
```
IoT Server: 192.168.20.100:8080
Username: admin
Password: admin
```

### Стъпка 4: Dashboard view

**От Admin PC → Web Browser:**
```
http://192.168.20.100:8080
```

Ще видите dashboard с всички IoT устройства!

---

## ЧАСТ 9: Security за IoT мрежата (15 мин)

### Best Practices:

### 1. Отделен VLAN за IoT
```cisco
R1(config)# vlan 30
R1(config-vlan)# name IoT_Devices
R1(config-vlan)# exit
```

### 2. ACL за ограничаване на достъпа

**Позволи IoT да достъпва само Gateway:**
```cisco
R1(config)# access-list 110 permit ip 192.168.30.0 0.0.0.255 host 192.168.20.50
R1(config)# access-list 110 deny ip 192.168.30.0 0.0.0.255 any
R1(config)# access-list 110 permit ip any any
!
R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# ip access-group 110 in
```

### 3. Rate limiting (ако е поддържано)

### 4. Strong passwords на Gateway

---

## ЧАСТ 10: Примерни IoT сценарии за проект

### Сценарий 1: Smart Security System
```
Motion Detector → Siren + Camera
Door Sensor → Alert + Log
Smoke Detector → Siren + Notification
```

### Сценарий 2: Energy Management
```
Light Sensor → Auto dimming lights
Temperature → HVAC control
Presence detection → Power saving mode
```

### Сценарий 3: Access Control
```
RFID Reader → Smart Door unlock
Camera → Face recognition (simulated)
Log all access events
```

### Сценарий 4: Environmental Monitoring
```
Temperature → Fan + Heater
Humidity → Dehumidifier
Air Quality → Ventilation system
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Комплексна автоматизация
Създайте система, която:
- При движение → включва лампата
- След 5 минути без движение → изключва лампата
- При температура > 26°C → включва вентилатор
- При температура < 20°C → изключва вентилатор

### Задача 2: Door Access System
- Добавете Smart Door
- Програмирайте да се отваря с Motion Detector
- Добавете Timer - след 10 сек да се затваря автоматично

### Задача 3: Fire Alarm System
- Smoke Detector → Siren + LED (червена)
- Log event на IoT Server
- Send alert (simulate с console message)

### Задача 4: Интеграция с Web Interface
- Създайте HTML страница на Web Server
- Добавете бутони за контрол на IoT устройства
- (Advanced - може да изисква JavaScript)

---

## ДОКУМЕНТАЦИЯ ЗА ПРОЕКТА

### Таблица с IoT устройства:

| Device Name       | Type      | IP Address    | Connected To | Function                    |
|-------------------|-----------|---------------|--------------|----------------------------|
| Home Gateway      | Gateway   | 192.168.20.50 | SW1 Fa0/7    | IoT hub and controller     |
| Motion-Det-1      | Sensor    | (Wireless)    | Gateway      | Detect movement            |
| Smart-LED-1       | Actuator  | (Wireless)    | Gateway      | Lighting control           |
| Temp-Monitor-1    | Sensor    | (Wireless)    | Gateway      | Temperature sensing        |
| Fan-1             | Actuator  | (Wireless)    | Gateway      | Cooling system             |

### IoT Automation Logic:

```python
# Motion-activated lighting
if motion_detected:
    smart_led.turn_on()
    wait(5 minutes)
    if not motion_detected:
        smart_led.turn_off()

# Temperature-based fan control
if temperature > 25:
    fan.turn_on()
elif temperature < 22:
    fan.turn_off()
```

---

## TROUBLESHOOTING IoT

### Проблем 1: Устройство не се регистрира

**Проверки:**
```
✓ SSID и пароla правилни ли са?
✓ Gateway IP address правилен ли е?
✓ Wireless signal силен ли е?
✓ Gateway online ли е?
```

**Решение:**
- Проверете Wireless settings
- Ping gateway-a
- Reset устройството и опитайте отново

### Проблем 2: Автоматизацията не работи

**Проверки:**
```
✓ Programming е стартиран (Run)?
✓ Устройствата са регистрирани?
✓ Simulation mode е активиран?
```

**Решение:**
- Проверете Blockly кода за грешки
- Тествайте ръчно всяко устройство
- Прегледайте Gateway logs

### Проблем 3: Intermittent connectivity

**Причина:** Wireless interference (симулирано)
**Решение:**
- Намалете разстоянието
- Проверете wireless settings

---

## IoT vs Traditional Networking - Разлики

| Aspect              | Traditional IT  | IoT                          |
|---------------------|-----------------|------------------------------|
| Protocol            | TCP/IP          | MQTT, CoAP, lightweight      |
| Power consumption   | High            | Low (battery-powered)        |
| Processing          | High            | Limited (микроконтролери)    |
| Security            | Established     | Challenging (edge devices)   |
| Scale               | Hundreds        | Thousands/millions           |
| Update mechanism    | Regular         | OTA (Over-The-Air)           |

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ Home Gateway конфигуриран и свързан
☐ Wireless SSID създаден с WPA2
☐ Поне 2 IoT сензора добавени
☐ Поне 2 IoT актуатора добавени
☐ Всички устройства регистрирани към Gateway
☐ Blockly automation програмирана
☐ Автоматизацията тествана и работи
☐ ACL за IoT security конфигуриран (optional)
☐ IoT устройствата са документирани
☐ Simulation mode тестове успешни
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Какво е IoT и как работи
2. ✅ IoT архитектура (сензори → gateway → server)
3. ✅ Wireless конфигурация за IoT устройства
4. ✅ Регистрация на устройства към Gateway
5. ✅ Blockly програмиране за автоматизация
6. ✅ Сензори и актуатори в Packet Tracer
7. ✅ IoT security considerations
8. ✅ Практически IoT сценарии

---

## ПОДГОТОВКА ЗА КУРСОВИЯ ПРОЕКТ

### Вие вече знаете:
- ✅ Мрежов дизайн (Hierarchical model)
- ✅ VLAN създаване и Inter-VLAN routing
- ✅ DHCP автоматично адресиране
- ✅ DNS name resolution
- ✅ ACL за сигурност и access control
- ✅ IoT интеграция и автоматизация

### За курсовия проект комбинирайте всичко:

**Пример структура:**
```
1. Network topology (Router + 2 Switches)
2. 3 VLAN-а (Admin, Users, IoT)
3. DHCP за автоматично адресиране
4. DNS server за name resolution
5. ACL за security policies
6. 3-5 IoT устройства с автоматизация
7. Документация (5 стр PDF)
8. Презентация и demo
```

---

## СЛЕДВАЩА СТЪПКА

**Вие сте готови за курсовия проект! **

Комбинирайте всичко научено от Lab 1-6 и създайте:
- Пълна функционална мрежа
- С VLAN сегментация
- Автоматични услуги (DHCP, DNS)
- Сигурност (ACL, SSH)
- IoT компоненти с автоматизация

**Успех! **

**Запазете файла като:** `Lab6_IoT_YourName.pkt`
