# УПРАЖНЕНИЕ 6: IoT ИНТЕГРАЦИЯ И АВТОМАТИЗАЦИЯ
## Smart Office с IoT устройства в Packet Tracer

**Цел:** Да се научите да интегрирате IoT устройства, да ги програмирате и да създавате автоматизация.

**Продължителност:** около 120 минути за основната част (Части 1–6А). Частите, означени като BONUS, са за допълнителна работа.

**Версия:** инструкциите са за Cisco Packet Tracer 8.2.2.

**Предварителни изисквания:** Завършени Упражнения 1-5

**Начален файл:** Отворете `Lab5_ACL_YourName.pkt` и го запазете като `Lab6_IoT_YourName.pkt`.

---

## ЧАСТ 1: IoT архитектура в Packet Tracer (15 мин)

### Какво е IoT (Internet of Things)?

Мрежа от физически устройства с вградени сензори, софтуер и мрежова свързаност, които събират и обменят данни.

### IoT архитектура в Packet Tracer:

```
Основен път (Smart Things):
IoT Устройства → WiFi → IoT Server → Conditions → Автоматизация
(сензори,               (регистрация,  (правила
 актуатори)              контрол)       IF → THEN)

BONUS (Components):
Сензори/актуатори → IoT Custom Cable → SBC-PT/MCU-PT → Автоматизация
                                       (програмиране, логика)
```

### Основни компоненти:

**Мрежова инфраструктура:**
- **IoT Server** - централен сървър за регистрация и управление на устройства
- **Access Point** - безжична точка за достъп

**Контролери за автоматизация:**
- **SBC-PT** (Single Board Computer) - програмируем контролер (Blockly, Python, JavaScript)
- **MCU-PT** (Microcontroller Unit) - програмируем микроконтролер

**Важно:** Платките управляват **Components** (сензори и актуатори без мрежов интерфейс), свързани към цифровите и аналоговите им слотове с **IoT Custom Cable**. Мрежовите **Smart Things**, регистрирани на IoT Server, се автоматизират с **Conditions** на сървъра.

**IoT устройства:**
- **IoT Sensors** - сензори (движение, температура, светлина, дим и др.)
- **IoT Actuators** - актуатори (лампи, вентилатори, врати, сирени и др.)

### Разлика между SBC-PT и MCU-PT

#### Сравнителна таблица:

| Характеристика | MCU-PT | SBC-PT |
|----------------|--------|--------|
| **Реален аналог** | Arduino, ESP32, ESP8266 | Raspberry Pi, BeagleBone, Jetson Nano |
| **Програмиране** | Blockly + Python + JavaScript (проверете в Programming таба) | Blockly + Python + JavaScript |
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

**Забележка:** Редовете за реален аналог, памет, ОС, енергия и цена описват реалните платки (аналогии), а не симулационните модели в Packet Tracer.

#### Препоръка за упражнението:

**В BONUS частите използваме SBC-PT**, защото:
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
3. **Автоматизация:** правила (Conditions) на IoT Server; програмиране на SBC-PT - в BONUS частите

---

## ЧАСТ 2: Топология на мрежата (20 мин)

### Разширена топология:

```
                [Router R1]
                     |
              GigE0/0 (trunk)
                     |        Gi0/1 (trunk)
                [Switch SW1] ------------------[Switch SW2]
          /     |      |       \        \        /      \
       DNS    IT     IoT    Access   SBC-PT    Fa0/2  Fa0/3
      Server  PC3   Server   Point   (BONUS)     |      |
                                |              [PC5]  [PC6]
                           (Wireless)          VLAN10 VLAN20
                           /    |    \
                      Motion  Light  Temp
                      Detect         Sensor
```

### Устройства за добавяне:

**Мрежови компоненти:**
1. **Access Point-PT** - за WiFi свързаност на IoT устройствата
2. **Server-PT** (IoT Server) - за регистрация и централизирано управление
3. **SBC-PT** - за програмиране на автоматизация (BONUS)

**IoT устройства (Smart Things от End Devices → Home):**
1. **Motion Detector**
2. **Light** (лампа)
3. **Temperature Monitor** (BONUS)
4. **Fan** (BONUS)

**Важно:** Използвайте **Smart Things** (End Devices → Home / Smart City), които имат мрежов интерфейс и се регистрират на IoT Server. **Components** (Components → Sensors / Actuators) нямат мрежов интерфейс и се свързват само към MCU/SBC с IoT Custom Cable.

### Резервиране на статичните адреси (задължително)

В Lab 3 R1 раздава адреси от 192.168.20.11 нагоре. Статичните адреси в това упражнение трябва да се изключат от раздаването, преди да ги използвате:

```cisco
R1# configure terminal
R1(config)# ip dhcp excluded-address 192.168.20.51
R1(config)# ip dhcp excluded-address 192.168.20.60
R1(config)# ip dhcp excluded-address 192.168.20.100
R1(config)# end
```

Ако някой от тези адреси вече е раздаден (`show ip dhcp binding`), обновете адреса на съответния клиент (`ipconfig /release` и `ipconfig /renew`).

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
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

**Защо VLAN 20?** В основния сценарий IoT устройствата използват съществуващата VLAN 20 (IT), без допълнителна изолация. Отделен IoT VLAN (VLAN 40) е BONUS в Част 10.

### Стъпка 3: Wireless конфигурация на Access Point

**Access Point-PT → Config → Port 1:**

- SSID: `SmartOffice` (внимавайте - case sensitive!)
- Authentication: **WPA2-PSK**
- PSK Pass Phrase: `SecureIoT123`

**Забележка:** Access Point-PT работи като мост (Layer 2) между безжичните устройства и VLAN 20. Стандартният модел няма GUI таб и IP адрес за управление (GUI имат безжичните маршрутизатори и Home Gateway). Ако вашата версия има поле за IP управление, използвайте резервирания адрес 192.168.20.51.

### Стъпка 4: Верификация

Access Point-PT няма IP адрес, затова не се проверява с ping. Връзката се проверява в Част 5: безжичното IoT устройство показва връзка към AP (пунктирна линия в топологията) и получава IP адрес от VLAN 20.

**Ако по-късно устройствата не се свързват:**
1. Проверете кабела AP ↔ SW1 Fa0/8 (зелени индикатори)
2. Проверете SSID и паролата в Config → Port 1
3. Проверете VLAN на SW1 Fa0/8 (`show vlan brief`)

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
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

### Стъпка 3: IP конфигурация на сървъра

**Server → Desktop → IP Configuration:**

```
IP Configuration: Static
IP Address: 192.168.20.100
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
DNS Server: 192.168.10.2
```

### Стъпка 4: Активиране на IoT услугата

**Server → Services → IoT:** Registration Server **ON**

### Стъпка 5: Създаване на акаунт и достъп до IoT Dashboard

**От PC3 (IT VLAN 20):** първо проверете дали PC3 има IP адрес (`ipconfig`; при нужда `ipconfig /release` и `ipconfig /renew`).

**PC3 → Desktop → Web Browser:**

```
URL: http://192.168.20.100
```

1. Кликнете **Sign up now** (долу на страницата за вход)
2. Създайте потребител и парола за упражнението (напр. `iotadmin` / `Lab6Pass`) и ги запишете
3. Влезте с тях

**Трябва да видите:**
- IoT Dashboard (страница Home)
- Списък с устройства (все още празен)

**Важно:** Сървърът няма фабричен акаунт `admin/admin` (такъв има само Home Gateway). Същите потребител и парола се въвеждат после във всяко IoT устройство.

---

## ЧАСТ 5: Добавяне и регистрация на IoT сензор (20 мин)

### Стъпка 1: Добавяне на Motion Detector

**Добавете устройството:**
- End Devices → Home → **Motion Detector** (Smart Thing)
- Поставете го в топологията (близо до Access Point за добър сигнал)

### Стъпка 2: Конфигурация на WiFi връзката

**Безжичен адаптер:** Motion Detector → Config → **Advanced** → таб **I/O Config** → Network Adapter: **PT-IOT-NM-1W**. (По подразбиране устройството е с Ethernet адаптер.)

**Motion Detector → Config → Wireless0:**

```
SSID: SmartOffice
Authentication: WPA2-PSK
PSK Pass Phrase: SecureIoT123
IP Configuration: DHCP
```

**Изчакайте:** Устройството трябва да се свърже към Access Point (виждате икона за WiFi).

### Стъпка 3: Регистрация към IoT Server

**Motion Detector → Config → Settings → IoT Server → Remote Server:**

```
Server Address: 192.168.20.100
User Name: (потребителят от Част 4)
Password: (паролата от Част 4)
```

**Кликнете бутон: Connect**

**Резултат:** Съобщение "Connected to IoT Server" или подобно.

### Стъпка 4: Проверка на регистрацията

**От PC3 (IT) → Web Browser → http://192.168.20.100**

**Трябва да видите:**
- Motion Detector в списъка с устройства
- Неговото име (напр. "MotionDetector0")
- Състояние: Connected
- Текуща стойност на сензора

---

## ЧАСТ 6: Добавяне на IoT актуатор (15 мин)

### Стъпка 1: Добавяне на лампа

**Добавете устройството:**
- End Devices → Home → **Light** (Smart Thing; или Street Lamp от Smart City)
- Поставете го в топологията

**Внимание:** Smart LED от Components → Actuators е компонент без мрежов интерфейс и не може да се регистрира на IoT Server. Той се използва само с SBC/MCU (BONUS).

### Стъпка 2: Конфигурация на WiFi връзката

**Безжичен адаптер:** Light → Config → Advanced → I/O Config → Network Adapter: **PT-IOT-NM-1W**.

**Light → Config → Wireless0:**

```
SSID: SmartOffice
Authentication: WPA2-PSK
PSK Pass Phrase: SecureIoT123
```

### Стъпка 3: Регистрация към IoT Server

**Light → Config → Settings → IoT Server → Remote Server:**

```
Server Address: 192.168.20.100
User Name: (потребителят от Част 4)
Password: (паролата от Част 4)
```

**Кликнете: Connect**

### Стъпка 4: Тест на ръчно управление

**От PC3 (IT) → Web Browser → http://192.168.20.100**

1. Намерете Light в списъка
2. Кликнете върху устройството
3. Потърсете бутон **ON/OFF** или **Toggle**
4. Кликнете бутона

**Резултат:** Лампата трябва да светне/угасне визуално в топологията!

**Това е основният начин за ръчен контрол на IoT устройствата.**

---

## ЧАСТ 6А: Автоматизация с Conditions на IoT Server (15 мин)

### Сценарий: При засичане на движение → включи лампата

**Conditions** е вграденият в IoT Server механизъм за правила IF → THEN между регистрираните Smart Things.

**PC3 (IT) → Web Browser → http://192.168.20.100 → Conditions → Add:**

**Правило 1:**
- Name: `Motion_ON`
- IF: Motion Detector - On (засечено движение)
- THEN: Light - Status: On

**Правило 2:**
- Name: `Motion_OFF`
- IF: Motion Detector - Off (няма движение)
- THEN: Light - Status: Off

(Имената на устройствата и стойностите изберете от падащите менюта - те зависят от реалните устройства.)

### Тестване

1. Работете в режим **Realtime** (Simulation Mode не е нужен)
2. Задръжте **Alt** и движете мишката пред Motion Detector
3. **Лампата трябва да светне автоматично!**
4. Спрете движението → **лампата угасва**

**След записа:** проверете, че двете правила се виждат в списъка Conditions. (За Packet Tracer 8.2.2 има потребителски доклади, че ново правило понякога не се показва; тогава презаредете страницата или го създайте отново.)

**Ако не работи:** проверете дали двете устройства са в списъка на Dashboard, дали ръчното управление работи (Част 6, Стъпка 4) и дали двете правила са записани.

**Това завършва основната част на упражнението.** Частите 7–9 (SBC-PT, Blockly и температурно управление) и разделите от Част 10, означени като BONUS, са за допълнителна работа.

---

## ЧАСТ 7 (BONUS): Добавяне на SBC-PT контролер (25 мин)

⚠️ **Важно за Части 7–8:** SBC-PT управлява **Components**, свързани към неговите цифрови и аналогови слотове с **IoT Custom Cable** (напр. Components → Sensors и Components → Actuators). Регистрацията на SBC-PT към IoT Server не дава автоматично на Blockly достъп по име до всички регистрирани Smart Things. Основната автоматизация на регистрираните устройства е с Conditions (Част 6А).

### Какво е SBC-PT?

Single Board Computer - контролер, който може да програмирате за автоматизация на IoT устройства.

**Възможности:**
- Blockly (визуално програмиране)
- Python (текстово програмиране)
- Управление на Components, свързани към слотовете му с IoT Custom Cable

### Стъпка 1: Добавяне на SBC-PT

**Добавете:**
- Components → Boards → **SBC-PT**
- (по избор) Свържете FastEthernet0 към SW1 FastEthernet0/10

**Забележка:** Мрежовото свързване и регистрацията (Стъпки 2–5) са по избор. Програмата в Част 8 управлява устройства, свързани директно към слотовете на SBC-PT, и не изисква мрежа.

### Стъпка 2: Конфигурация на порта на комутатора

```cisco
SW1(config)# interface FastEthernet0/10
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description IoT-Controller-SBC
SW1(config-if)# no shutdown
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
User Name / Password: акаунтът от Част 4
```

**ВАЖНО:** Регистрацията на SBC-PT към IoT Server не му дава автоматично управление на всички регистрирани устройства (вижте предупреждението в началото на Част 7).

### Стъпка 5: Проверка на връзката

**SBC-PT → Config → Settings**

Трябва да видите статус: "Connected to IoT Server"

---

## ЧАСТ 8 (BONUS): Програмиране на автоматизация със SBC-PT (30 мин)

### Сценарий: При засичане на движение → включи лампата (локално, без IoT Server)

Сценарият следва официалното упражнение на Cisco NetAcad „SBC Actuate“: SBC-PT чете сензор и управлява лампа, свързани директно към неговите слотове.

### Стъпка 1: Свързване на устройствата към SBC-PT

1. Добавете **Motion Sensor** (Components → Sensors → Motion Sensor)
2. Добавете **Lamp** (End Devices → Home → Lamp)
3. Свържете ги към SBC-PT с **IoT Custom Cable** (Connections):
   - Motion Sensor → слот **D0** на SBC-PT
   - Lamp → слот **D1** на SBC-PT

**Забележка:** Тези устройства не се регистрират на IoT Server и не се нуждаят от WiFi.

### Стъпка 2: Програма

**SBC-PT → Programming → New:** име `main.py`, шаблон **Empty - Python** (или **Empty - Visual** за Blockly) → **Open**.

**Python (по официалния пример на Cisco):**

```python
from gpio import *
from time import *

def main():
    while True:
        if digitalRead(0) == HIGH:   # движение на D0
            customWrite(1, 1)        # лампата на D1: ON
        else:
            customWrite(1, 0)        # лампата на D1: OFF
        delay(500)

if __name__ == "__main__":
    main()
```

**Blockly (шаблон Empty - Visual):** изградете същата логика с блоковете за пинове: безкраен цикъл → `if digitalRead(0) = HIGH` → `customWrite(1, 1)`, иначе `customWrite(1, 0)` → `delay(500)`. Точните имена на блоковете проверете в палитрата на вашата версия на Packet Tracer.

### Стъпка 3: Стартиране и тест

1. Кликнете **Run**
2. В режим **Realtime** задръжте **Alt**, докато мишката е над Motion Sensor
3. **Лампата трябва да светне автоматично!**
4. Отдалечете мишката и пуснете Alt → **лампата угасва**

**Забележка:** Проверете в Specifications на лампата какво означават стойностите на `customWrite` (някои модели: 0 = Off, 1 = Dim, 2 = On). Ако 1 означава Dim, използвайте `customWrite(1, 2)` за пълно включване.

**Ако не работи - вижте раздел Troubleshooting по-долу (Проблем 3).**

---

## ЧАСТ 9 (BONUS): Разширена автоматизация - Temperature Control (20 мин)

### Сценарий: При температура > 26°C → включи вентилатора

### Стъпка 1: Добавяне на допълнителни устройства

**Добавете:**
1. **Temperature Monitor** (IoT → Sensors → Temperature Monitor)
2. **Fan** (IoT → Actuators → Fan)

**За двете устройства:**
- Безжичен адаптер PT-IOT-NM-1W и WiFi (SSID: SmartOffice)
- Регистрирайте към IoT Server (192.168.20.100)
- Проверете в Dashboard че са регистрирани

### Стъпка 2: Правила в Conditions

**PC3 (IT) → Web Browser → http://192.168.20.100 → Conditions → Add:**

- **Правило 1:** IF Temperature Monitor > 26 → THEN Fan - On
- **Правило 2:** IF Temperature Monitor <= 26 → THEN Fan - Off

**Логика (за справка):**
```python
if TemperatureSensor.value > 26:
    Fan.state = ON
else:
    Fan.state = OFF
```

### Стъпка 3: Тестване

**Тестване (Realtime):**

Temperature Monitor отчита температурата на средата. Ако стойността не може да се промени с клик, променете температурата на средата (Environment) или използвайте нагревател.

1. Повишете температурата → 28°C
2. **Fan трябва да се включи!**
3. Намалете температурата → 24°C
4. **Fan трябва да се изключи!**

### Стъпка 4: Комбинирана програма

**Можете да имате и двете автоматизации едновременно:**
- Движение → LED
- Температура → Fan

И двете ще работят паралелно!

---

## ЧАСТ 10: IoT Security - Защита на мрежата (15 мин)

### Best Practices за IoT сигурност:

#### 1. Отделен VLAN за IoT устройства (BONUS - в отделен файл)

⚠️ Изпълнявайте само в копие на завършения файл (`Lab6_Bonus_IoTVLAN.pkt`). VLAN 30 вече е Guest в задачите на Lab 2 и Lab 3, затова за IoT използваме **VLAN 40** (192.168.40.0/24).

```cisco
! SW1
SW1(config)# vlan 40
SW1(config-vlan)# name IoT_Devices
SW1(config-vlan)# exit
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport trunk allowed vlan add 40
SW1(config-if)# exit
SW1(config)# interface range FastEthernet0/8 - 10
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 40
SW1(config-if-range)# no shutdown
SW1(config-if-range)# exit
!
! R1
R1(config)# interface GigabitEthernet0/0.40
R1(config-subif)# encapsulation dot1Q 40
R1(config-subif)# ip address 192.168.40.1 255.255.255.0
R1(config-subif)# exit
R1(config)# ip dhcp excluded-address 192.168.40.1 192.168.40.100
R1(config)# ip dhcp pool IOT_POOL
R1(dhcp-config)# network 192.168.40.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.40.1
R1(dhcp-config)# dns-server 192.168.10.2
R1(dhcp-config)# exit
```

**След това задължително:**
- IoT Server: IP 192.168.40.100, Gateway 192.168.40.1 (и SBC-PT: 192.168.40.60, ако го използвате)
- IoT устройствата: превключете IP Configuration (Wireless0) на Static и обратно на DHCP, за да получат адрес от 192.168.40.0/24, и ги регистрирайте отново към **192.168.40.100**
- Тест от PC3: `http://192.168.40.100`

#### 2. ACL за ограничаване на достъпа (BONUS, след VLAN 40)

**Цел:** IoT устройствата (VLAN 40) не достъпват останалите мрежи, но IT (VLAN 20) може да отваря Dashboard.

```cisco
R1(config)# access-list 140 remark IoT-Security-Policy
R1(config)# access-list 140 permit udp any eq 68 any eq 67
R1(config)# access-list 140 permit udp 192.168.40.0 0.0.0.255 host 192.168.10.2 eq 53
R1(config)# access-list 140 permit tcp 192.168.40.0 0.0.0.255 host 192.168.10.2 eq 53
R1(config)# access-list 140 permit tcp host 192.168.40.100 eq 80 192.168.20.0 0.0.0.255
R1(config)# access-list 140 permit icmp 192.168.40.0 0.0.0.255 host 192.168.40.1
R1(config)# access-list 140 deny ip 192.168.40.0 0.0.0.255 any
R1(config)# access-list 140 permit ip any any
!
R1(config)# interface GigabitEthernet0/0.40
R1(config-subif)# ip access-group 140 in
```

**Обяснение:**
- Правило 1: всички DHCP заявки на клиентите (порт 68 → 67), включително първоначалната от 0.0.0.0
- Правила 2–3: DNS към вътрешния DNS сървър 192.168.10.2 (DHCP го дава на IoT устройствата)
- Правило 4: отговорите на IoT Server към браузъра на PC-тата от VLAN 20. ACL не помни връзките (stateless): без това правило отговорите на сървъра биха били блокирани
- Правило 5: ping към gateway-я 192.168.40.1 (диагностика)
- Правило 6: всичко останало от VLAN 40 към други мрежи е забранено
- IoT устройствата и IoT Server са в една VLAN, затова комуникацията помежду им не минава през ACL

**Проверка:** включете нов DHCP клиент във VLAN 40 и следете броячите с `show access-lists 140`.

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
| Light0 | Actuator | - | 20 | Entrance | Лампа |
| TempSensor0 | Sensor | - | 20 | Office | Температурен сензор |
| Fan0 | Actuator | - | 20 | Office | Вентилатор |

### Мрежова конфигурация:

| Устройство | IP Address | Gateway | VLAN | Порт |
|------------|------------|---------|------|------|
| IoT Server | 192.168.20.100 | 192.168.20.1 | 20 | Fa0/9 |
| Access Point | няма (Layer 2) | - | 20 | Fa0/8 |
| SBC-PT (BONUS) | 192.168.20.60 | 192.168.20.1 | 20 | Fa0/10 |
| PC3 (Test) | 192.168.20.11 | 192.168.20.1 | 20 | Fa0/4 |

### Програма за автоматизация (описание):

```
Програма: Smart_Office_Automation
Версия: 1.0
Автор: [Вашето име]

Логика (основна част - Conditions на IoT Server):
1. IF Motion Detector = On   → THEN Light = On
2. IF Motion Detector = Off  → THEN Light = Off

BONUS:
3. IF Temperature > 26°C     → THEN Fan = On
4. IF Temperature <= 26°C    → THEN Fan = Off
```

### Тестови сценарии:

| Test ID | Действие | Очакван резултат | Статус |
|---------|----------|------------------|--------|
| T01 | Движение (Alt над Motion Detector) | Light ON | ☐ За попълване |
| T02 | Няма движение | Light OFF | ☐ За попълване |
| T03 (BONUS) | Температура > 26°C | Fan ON | ☐ За попълване |
| T04 (BONUS) | Температура ≤ 26°C | Fan OFF | ☐ За попълване |

---

## ЧАСТ 13: Troubleshooting - Отстраняване на проблеми

### Често срещани проблеми и решения:

| # | Проблем | Симптом | Проверка | Решение | Време |
|---|---------|---------|----------|---------|-------|
| 1 | **WiFi не се свързва** | Няма WiFi икона на устройството | SSID правилен? (SmartOffice - case sensitive!) | Изтрий SSID/парола изцяло, въведи отново. Измести по-близо до AP. | 2-5 мин |
| 2 | **Не се регистрира** | WiFi ОК, но не в Dashboard | IoT Server ON? Правилни потребител/парола? | Проверка на http://192.168.20.100. Refresh. Re-connect от устройството. | 3-5 мин |
| 3 | **SBC-PT програмата не работи** (BONUS) | Програмата стартира, но не реагира | Слотовете в кода съвпадат с кабелите? | Проверете IoT Custom Cable (D0, D1) и номерата в кода. | 5-10 мин |
| 4 | **Сензор не реагира** | В Dashboard, но без events | Status в Dashboard "Active"? | Refresh Dashboard. Manual trigger test. Рестарт на устройство. | 3-5 мин |
| 5 | **Актуатор не се управлява** | Вижда се, но не реагира на команди | Connection status? | Test с Manual control от Dashboard. Проверка на WiFi. Restart. | 3-5 мин |
| 6 | **Лампата не реагира на SBC-PT** (BONUS) | Програмата върви, лампата не реагира | Кабелът е в слот D1? | Свържете лампата с IoT Custom Cable към D1; в кода `customWrite(1, ...)`. | 5 мин |

---

### Детайлни стъпки за основни проблеми:

#### Проблем 1: WiFi връзка

**Първо:** проверете дали устройството е с безжичен адаптер (Config → Advanced → I/O Config → **PT-IOT-NM-1W**).

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

2. Тест на свързаност:
   - PC3 → ping 192.168.20.100
   - PC3 → Web Browser → http://192.168.20.100

3. Регистрация на устройството:
   - IoT устройство → Config → Settings → IoT Server → **Remote Server**
   - Server Address: 192.168.20.100
   - User Name / Password: акаунтът от Част 4
   - Кликнете **Connect**

4. Проверка:
   - Refresh Dashboard
   - Устройството трябва да се появи

---

#### Проблем 3: SBC-PT програма (BONUS)

**Стъпки:**
1. Спрете програмата: SBC-PT → Programming → **Stop**

2. Проверете свързването:
   - Motion Sensor → слот **D0**, Lamp → слот **D1** (IoT Custom Cable)
   - Номерата на слотовете в кода съвпадат ли с кабелите?

3. Проверете логиката:
   - `digitalRead(0)` чете сензора
   - `customWrite(1, 1)` и `customWrite(1, 0)` управляват лампата

4. Тест:
   - Стартирайте програмата отново (**Run**)
   - В Realtime задръжте **Alt** над Motion Sensor

---

### Debug съвети:

**Общи проверки:**
```
✓ Access Point включен? (Power ON, зелен индикатор)
✓ IoT Server Services → IoT активирана?
✓ Всички устройства в една WiFi мрежа? (SSID: SmartOffice)
✓ Conditions правилата записани? (SBC-PT - само в BONUS)
✓ (BONUS) Слотовете в кода на SBC-PT съвпадат с кабелите?
✓ Тествате в режим Realtime?
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
☐ Статичните адреси изключени от DHCP (.51, .60, .100)
☐ Switch портове назначени
☐ Access Point добавен (Fa0/8, SSID SmartOffice)
☐ IoT Server добавен (192.168.20.100)
☐ IoT Server Services → IoT = ON
☐ Dashboard достъпен (http://192.168.20.100)
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
☐ Conditions правило създадено (движение → лампа)
☐ Логиката тествана и работи
☐ Автоматизацията реагира на събития
☐ (BONUS) SBC-PT програмиран (Blockly или Python) с локално свързани устройства
```

### Документация:
```
☐ Таблица с IoT устройства попълнена
☐ Мрежова схема нарисувана
☐ Логиката на автоматизацията документирана
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
7. ✅ Автоматизация с Conditions (IF → THEN)
8. ✅ (BONUS) Програмиране на SBC-PT с Blockly
9. ✅ IoT сигурност (VLAN, ACL, пароли)
10. ✅ Troubleshooting на IoT системи

---

## ЧАСТ 16: Подготовка за курсовия проект

### Изисквания за проекта:

**Минимални изисквания (според заданието):**
- Минимум 1 IoT устройство за положителна оценка; за точките за автоматизация - поне 1 сензор и 1 актуатор
- 3+ IoT устройства за отлична оценка
- Автоматизация: правила (Conditions) на IoT Server **или** програма на SBC-PT/MCU-PT (Blockly или Python)
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
   - Router, 2+ комутатора (Access/Distribution), VLAN-и, адресен план
   - DHCP, DNS, ACL, SSH, Port Security
   - Access Point настройка
   - IoT Server инсталация

3. **IoT устройства (1-2 часа)**
   - Добавяне на сензори и актуатори
   - WiFi връзка
   - Регистрация в Dashboard
   - Тестване на ръчен контрол

4. **Автоматизация (2-3 часа)**
   - Conditions правила или SBC-PT конфигурация
   - Blockly програмиране (при SBC-PT)
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

### Съответствие: изискване на заданието → къде се учи

| Изискване на курсовия проект | Упражнение |
|------------------------------|------------|
| Router + 2 комутатора, йерархия Access/Distribution | Lab 2, Част 3А |
| 2–4 VLAN-а, 3+ подмрежи, Inter-VLAN routing | Lab 2 |
| Management VLAN (не VLAN 1) | Lab 2 |
| Subnetting и адресен план | Lab 2, Част 1 и Задача 5 |
| DHCP pools, excluded addresses, lease time | Lab 3 |
| DNS: A и CNAME записи, name resolution | Lab 4 |
| ACL: Standard, Extended, Named | Lab 5 |
| SSH на всички устройства, силни пароли | Lab 5, Част 6 |
| Port Security на 2 порта | Lab 5, Част 8 |
| NAT (само при симулиран интернет) | Lab 5, Част 9 (BONUS) |
| IoT устройства и автоматизация | Lab 6 |
| Тестове и таблици за документацията | Контролните тестове в Lab 2–6; Lab 6, Част 12 |

### Предпроектна интеграционна задача (самостоятелно, ориентировъчно 180 мин)

Преди проекта, в **нов празен файл** и без да следвате упражненията стъпка по стъпка, изградете малка мрежа. Задачата не се побира в едно занятие: т. 1–2 (инфраструктура и услуги) и т. 3–5 (сигурност, IoT и документация) могат да се изпълнят в две части, а довършването е възможно и извън лабораторията.

1. R1 + 2 комутатора (Distribution и Access), 3 потребителски VLAN-а + Management VLAN, адресен план с различни маски
2. DHCP за всички потребителски VLAN-и; DNS сървър с 3 A записа и 1 CNAME
3. Едно Extended ACL правило; SSH на всички мрежови устройства; Port Security на 2 порта
4. Един сензор и един актуатор с правило в Conditions
5. Кратка документация: адресна таблица, VLAN таблица и 3 теста: успешен ping между VLAN-и; трафик, блокиран от ACL; ping по име (DNS)

**Самопроверка:** можете ли да промените едно ACL правило и да обясните ефекта? Това се проверява и при защитата на проекта.

### Как да обосновете решенията в документацията

За всяко основно решение напишете по едно-две изречения: защо е избрана тази структура (VLAN-и по отдели или функции), защо са избрани тези маски (брой устройства), какво разрешава и забранява всяко ACL правило и как е проверено, че услугите работят.

---

## ЧАСТ 17: Полезни ресурси

| Ресурс | URL | Описание |
|--------|-----|----------|
| Cisco Packet Tracer | netacad.com | Официален сайт за download |
| IoT в Packet Tracer | tutorials.ptnetacad.net/help/default/iot_using_things.htm | Официална помощ на PT за IoT |
| Blockly Guide | developers.google.com/blockly | Visual programming guide |
| Сървъри в Packet Tracer | tutorials.ptnetacad.net/help/default/config_servers.htm | DNS, DHCP, HTTP на Server-PT |

---

## Бърз старт за начинаещи (30 мин)

### Минимална работеща конфигурация:

**Цел:** 1 сензор + 1 актуатор + проста автоматизация

**Устройства:**
- Access Point (Fa0/8)
- IoT Server (192.168.20.100)
- Motion Detector (Sensor)
- Light (Actuator)
- (SBC-PT не е нужен за бързия старт)

**Стъпки:**

1. **Мрежа:** Добави AP и Server в VLAN 20
2. **WiFi:** SSID: SmartOffice, Парола: SecureIoT123
3. **Регистрация:** Свържи устройствата към IoT Server
4. **Conditions:** IF Motion Detector = On → THEN Light = On (и второ правило за Off)
5. **Тест:** Alt + движение на мишката пред сензора → лампата трябва да светне

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

**Успех с IoT автоматизацията!**



**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
