# LAB 3: DHCP КОНФИГУРАЦИЯ
## Dynamic Host Configuration Protocol - Автоматично IP адресиране

**Цел:** Да се научите да конфигурирате DHCP за автоматично раздаване на IP адреси.

**Продължителност:** 60 минути

**Предварителни изисквания:** Завършен Lab 2 (VLAN-и и Inter-VLAN routing)

---

## ЧАСТ 1: Какво е DHCP? (5 мин)

### Проблем без DHCP:

```
❌ Администраторът трябва РЪЧНО да конфигурира всеки PC:
   - IP Address
   - Subnet Mask  
   - Default Gateway
   - DNS Server

❌ Голям риск от грешки (duplicate IP)
❌ Много време при 50+ компютъра
```

### Решение с DHCP:

```
✅ PC се включва в мрежата
✅ Автоматично получава всички настройки
✅ Няма грешки, няма duplicate IP-та
✅ Бърза конфигурация
```

### Как работи? (DORA процес)

```
1. PC:      "Има ли тук DHCP server?" (DISCOVER)
2. Server:  "Да! Ето ти IP: 192.168.10.11" (OFFER)
3. PC:      "OK, приемам го" (REQUEST)
4. Server:  "Потвърдено!" (ACK)
```

### Какво получава PC-то?

| Параметър | Стойност | Обяснение |
|-----------|----------|-----------|
| IP Address | 192.168.10.11 | Адресът на PC-то |
| Subnet Mask | 255.255.255.0 | Размер на мрежата |
| Default Gateway | 192.168.10.1 | Router за излизане от мрежата |
| DNS Server | 8.8.8.8 | Google DNS за имена |

---

## ЧАСТ 2: Топология (5 мин)

Продължаваме със същата топология от Lab 2:

```
              [Router R1]
                   |
            GigE0/0 (trunk)
                   |
              [Switch SW1]
               /   |   \   \
           Fa0/2 Fa0/3 Fa0/4 Fa0/5
             /     |     \     \
        [PC1]  [PC2]  [PC3]  [PC4]
       VLAN10 VLAN10 VLAN20 VLAN20
```

**VLAN-и:**
- VLAN 10: 192.168.10.0/24
- VLAN 20: 192.168.20.0/24

---

## ЧАСТ 3: Метод 1 - Router като DHCP сървър (25 мин)

### Стъпка 1: Excluded addresses (запазени адреси)

**Защо?** Запазваме първите 10 адреса за статични устройства:

```
.1 = Router (Gateway)
.2 = DNS Server (ако имаме)
.3 = Web Server (ако имаме)
.4-.10 = Резерв за други сървъри
```

```cisco
R1# configure terminal
R1(config)# no ip domain-lookup
!
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10
R1(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.10
```

### Стъпка 2: DHCP pool за VLAN 10

```cisco
R1(config)# ip dhcp pool ADMIN_POOL
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# dns-server 8.8.8.8
R1(dhcp-config)# exit
```

**Обяснение:**
- `network` → Коя мрежа обслужва този pool
- `default-router` → Gateway-ят (Router interface IP)
- `dns-server` → DNS за name resolution

### Стъпка 3: DHCP pool за VLAN 20

```cisco
R1(config)# ip dhcp pool IT_POOL
R1(dhcp-config)# network 192.168.20.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.20.1
R1(dhcp-config)# dns-server 8.8.8.8
R1(dhcp-config)# exit
```

### Стъпка 4: Запазване

```cisco
R1(config)# exit
R1# copy running-config startup-config
```

**Забележка:** На Router НЕ можем да настроим lease time в Packet Tracer.

---

## ЧАСТ 4: Метод 2 - Server като DHCP (BONUS - 15 мин)

### Предимства на Server метода:

✅ Има GUI (по-лесно за начинаещи)  
✅ **Lease Time работи!** (можем да настроим часове/дни)  
✅ Виждаме всички настройки наведнъж  

### Топология със Server:

```
              [Router R1]
                   |
            GigE0/0 (trunk)
                   |
              [Switch SW1]
          /     |     |     \     \
       Fa0/11 Fa0/2 Fa0/3 Fa0/4 Fa0/5
         |     |     |     |     |
     [Server] [PC1] [PC2] [PC3] [PC4]
      DHCP   VLAN10 VLAN10 VLAN20 VLAN20
```

### Стъпка 1: Добавяне на Server

1. **Devices → End Devices → Server-PT**
2. Свържете към **SW1 FastEthernet0/11**

```cisco
SW1(config)# interface FastEthernet0/11
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description DHCP-Server
SW1(config-if)# exit
```

### Стъпка 2: IP на Server-а

**Кликнете: Server → Desktop → IP Configuration**

```
IP Address: 192.168.10.2
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

### Стъпка 3: Конфигурация на DHCP Service

**Кликнете: Server → Services → DHCP**

1. **Включете DHCP:** ON

2. **Pool за VLAN 10:**
   ```
   Pool Name: ADMIN_POOL
   Default Gateway: 192.168.10.1
   DNS Server: 8.8.8.8
   Start IP Address: 192.168.10.11
   Subnet Mask: 255.255.255.0
   Maximum Number of Users: 50
   TFTP Server: (празно)
   WLC Address: (празно)
   Lease Time: 1 Days (или колкото искате!)
   ```
   → Кликнете **Add**

3. **Pool за VLAN 20:**
   ```
   Pool Name: IT_POOL
   Default Gateway: 192.168.20.1
   DNS Server: 8.8.8.8
   Start IP Address: 192.168.20.11
   Subnet Mask: 255.255.255.0
   Maximum Number of Users: 50
   Lease Time: 7 Days
   ```
   → Кликнете **Add**

### Стъпка 4: DHCP Relay на Router

**Защо?** Server е в VLAN 10, но трябва да обслужва и VLAN 20.

```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# ip helper-address 192.168.10.2
R1(config-subif)# exit
```

**Какво прави:** Препраща DHCP заявки от VLAN 20 към Server-а.

---

## ЧАСТ 5: Конфигурация на PC-тата (10 мин)

### За всеки PC (PC1, PC2, PC3, PC4):

1. **Кликнете:** PC → Desktop → IP Configuration
2. **Изберете:** DHCP (вместо Static)
3. **Кликнете:** Бутона DHCP
4. **Изчакайте** 3-5 секунди

**Трябва да видите:**
```
IP Address: 192.168.10.11 (или друг)
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
DNS Server: 8.8.8.8
```

### Ако не работи:

**Command Prompt:**
```
ipconfig /release
ipconfig /renew
```

---

## ЧАСТ 6: Проверка (10 мин)

### Test 1: Проверка на получените адреси

**На PC (Command Prompt):**
```
ipconfig
```

**Очаквано за PC1 (VLAN 10):**
```
IP Address: 192.168.10.11
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

### Test 2: Проверка на DHCP bindings

**Метод 1 (Router като DHCP):**
```cisco
R1# show ip dhcp binding
```

**Очакван изход:**
```
IP address       Hardware address        Type
192.168.10.11    0001.6373.7ABF         Automatic
192.168.10.12    0002.16CE.E0F3         Automatic
192.168.20.11    00D0.BC0F.0D4C         Automatic
```

**Метод 2 (Server като DHCP):**
- **Server → Services → DHCP**
- Вижте таблицата **Current Leases**

### Test 3: Ping тест

**От PC1:**
```
ping 192.168.10.1     (gateway)
ping 192.168.10.12    (друг PC в VLAN 10)
ping 192.168.20.1     (gateway VLAN 20)
ping 192.168.20.11    (PC в VLAN 20)
```

**Всички трябва да работят!** ✅

---

## ЧАСТ 7: Troubleshooting (10 мин)

### Проблем 1: PC не получава адрес

**Симптоми:**
```
DHCP Request Failed
IP Address: 0.0.0.0
```

**Проверки:**
```cisco
! Има ли DHCP pool за този subnet?
R1# show ip dhcp pool

! Интерфейсът UP ли е?
R1# show ip interface brief

! Правилен ли е VLAN на порта?
SW1# show vlan brief
```

**Решение:**
1. Проверете DHCP pool конфигурацията
2. Проверете VLAN assignment на switch port-а
3. Release и renew отново

### Проблем 2: PC получава 169.254.x.x

**Какво значи?**
- Windows автоматично раздава **APIPA адрес**
- Появява се когато DHCP server **НЕ е достъпен**

**Решение:**
1. Проверете дали DHCP service е ON (ако ползвате Server)
2. Проверете физическата свързаност
3. Проверете ip helper-address (ако Server е в друг VLAN)

### Проблем 3: Грешен gateway

**Симптоми:**
- PC получава адрес
- Ping в същия VLAN работи
- Ping извън VLAN-а НЕ работи

**Решение:**
```cisco
! Проверка на pool
R1# show running-config | section dhcp

! Трябва да видите:
ip dhcp pool ADMIN_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1   ← Проверете!
```

---

## ЧАСТ 8: Статична резервация за Server (BONUS - 10 мин)

### Scenario:
Искаме File Server винаги да получава **192.168.10.50**

**Защо File Server, не Web Server?**
- Web Server обикновено е на **.3** (ще го добавим в Lab 4)
- File Server е по-подходящ за **.50** (сървърна зона)

### Стъпка 1: Добавете File Server

1. **Добавете Server-PT** в VLAN 10
2. Име: File Server
3. Свържете към **SW1 Fa0/6**

```cisco
SW1(config)# interface FastEthernet0/6
SW1(config-if)# switchport mode access  
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description File-Server
SW1(config-if)# exit
```

### Стъпка 2: Намерете MAC адреса

**От Router:**
```cisco
R1# show ip dhcp binding

IP address       Hardware address
192.168.10.13    00D0.974B.97FC    ← Това е File Server
```

### Стъпка 3: Excluded address

```cisco
R1(config)# ip dhcp excluded-address 192.168.10.50
```

**Защо?** За да не го раздава pool-ът на друго устройство.

### Стъпка 4: Manual binding

```cisco
R1(config)# ip dhcp pool FILE_SERVER
R1(dhcp-config)# host 192.168.10.50 255.255.255.0
R1(dhcp-config)# client-identifier 0100.d097.4b97.fc
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# exit
```

**Забележка:** Client-identifier = "01" + MAC адрес

**⚠️ ВАЖНО:** В Packet Tracer използвайте **client-identifier** (НЕ hardware-address)!

### Стъпка 5: DHCP на Server

**File Server → Desktop → IP Configuration**
- Изберете **DHCP**
- Кликнете бутона

**Трябва да получи:**
```
IP Address: 192.168.10.50
```

**Забележка:** Web Server ще добавим в **Lab 4** на адрес **192.168.10.3**

---

## СРАВНЕНИЕ: Router vs Server метод

| Аспект | Router DHCP | Server DHCP |
|--------|-------------|-------------|
| **Конфигурация** | CLI команди | GUI (по-лесно) |
| **Lease Time** | ❌ Не работи в PT | ✅ Работи! |
| **Flexibility** | По-гъвкав | Основни настройки |
| **За бакалаври** | Трябва да знаят IOS | По-достъпно |
| **Реален свят** | Често се използва | При малки мрежи |
| **За Lab** | Основен метод | BONUS |

**Препоръка:** Научете и двата метода!

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Трети VLAN
- Създайте VLAN 30 (Guest: 192.168.30.0/24)
- Добавете DHCP pool
- Тествайте с нов PC

### Задача 2: Променете DNS
- Използвайте Cloudflare DNS: **1.1.1.1**
- Обновете pools
- Release/renew на PC-тата

### Задача 3: Server DHCP
- Пробвайте Server метода (ЧАСТ 4)
- Настройте lease time на 2 дни
- Сравнете с Router метода

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ DHCP pools създадени (VLAN 10 и 20)
☐ Excluded addresses конфигурирани
☐ PC-тата получават адреси автоматично
☐ Ping работи между всички VLAN-и
☐ show ip dhcp binding показва клиентите
☐ (BONUS) Server DHCP тестван
☐ (BONUS) Manual binding за File Server
☐ Конфигурацията е запазена (copy run start)
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Какво е DHCP и защо е важен
2. ✅ DORA процес (Discover, Offer, Request, ACK)
3. ✅ Конфигурация на DHCP на Router
4. ✅ Excluded addresses
5. ✅ (BONUS) Server като DHCP с lease time
6. ✅ (BONUS) Manual binding за сървъри
7. ✅ Troubleshooting

---

## СЛЕДВАЩА СТЪПКА

В **Lab 4** ще научим:
- DNS конфигурация
- Name resolution (ping по име вместо IP)
- Интеграция на DHCP със DNS

**Запазете файла като:** `Lab3_DHCP_YourName.pkt`

---

**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
