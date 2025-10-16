# LAB 3: DHCP КОНФИГУРАЦИЯ
## Dynamic Host Configuration Protocol

**Цел:** Да се научите да конфигурирате DHCP на Router и да автоматизирате IP адресирането на устройствата.

**Продължителност:** 60-90 минути

**Prerequisite:** Завършен Lab 2 (VLAN-и и Inter-VLAN routing)

---

## ЧАСТ 1: Топология - продължаваме от Lab 2

### Използваме същата топология:
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

### VLAN-и:
- VLAN 10 - Administration (192.168.10.0/24)
- VLAN 20 - IT Department (192.168.20.0/24)
- VLAN 99 - Management (192.168.99.0/24)

---

## ЧАСТ 2: Какво е DHCP? (Теория - 10 мин)

### DHCP процесът (DORA):
```
1. DISCOVER  - Client: "Има ли тук DHCP server?"
2. OFFER     - Server: "Да! Ето ти IP адрес"
3. REQUEST   - Client: "OK, искам този адрес"
4. ACK       - Server: "Потвърдено! Адресът е твой"
```

### Какво получава клиентът:
- IP Address
- Subnet Mask
- Default Gateway
- DNS Server(s)
- Lease time (колко време е валиден IP-то)

---

## ЧАСТ 3: Конфигурация на DHCP pools (25 мин)

### Стъпка 1: Изключване на DNS lookup (ако не е направено)
```cisco
R1# configure terminal
R1(config)# no ip domain-lookup
```

### Стъпка 2: Създаване на DHCP pool за VLAN 10
```cisco
R1(config)# ip dhcp pool ADMIN_POOL
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
R1(dhcp-config)# domain-name company.local
R1(dhcp-config)# lease 7
R1(dhcp-config)# exit
```

**Обяснение на параметрите:**
- `network` - адресното пространство на мрежата
- `default-router` - gateway за клиентите
- `dns-server` - DNS сървъри (Google DNS в примера)
- `domain-name` - домейн за локалното име
- `lease 7` - IP адресът е валиден за 7 дни

### Стъпка 3: Изключване на адреси (exclude addresses)
```cisco
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10
```

**Защо?** Запазваме 192.168.10.1-10 за статични устройства:
- .1 = Gateway (Router)
- .2-.10 = Сървъри, принтери, мрежови устройства

### Стъпка 4: DHCP pool за VLAN 20
```cisco
R1(config)# ip dhcp pool IT_POOL
R1(dhcp-config)# network 192.168.20.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.20.1
R1(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
R1(dhcp-config)# domain-name company.local
R1(dhcp-config)# lease 7
R1(dhcp-config)# exit
!
R1(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.10
```

### Стъпка 5: Записване на конфигурацията
```cisco
R1# copy running-config startup-config
```

---

## ЧАСТ 4: Конфигурация на PC-та за DHCP (15 мин)

### За всеки PC (PC1, PC2, PC3, PC4):

1. Кликнете на PC → Desktop → IP Configuration
2. Изберете: **DHCP** (вместо Static)
3. Кликнете: **DHCP** бутон
4. Изчакайте за адрес

**След 3-5 секунди трябва да видите:**
```
DHCP Request Successful
IP Address: 192.168.10.11 (примерно)
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
DNS Server: 8.8.8.8
```

### Ако не работи веднага:
```
Command Prompt на PC:
ipconfig /release
ipconfig /renew
```

---

## ЧАСТ 5: Верификация и тестване (20 мин)

### Test 1: Проверка на получените адреси

**На всеки PC (Command Prompt):**
```
ipconfig
```

**Очакван резултат за PC1 (VLAN 10):**
```
IP Address: 192.168.10.11 (или друг в диапазона 11-254)
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

### Test 2: Проверка на DHCP сървъра

**На Router:**
```cisco
R1# show ip dhcp binding
```

**Очакван изход:**
```
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.10.11    0001.6373.7ABF          Jun 15 2024 12:30 PM    Automatic
192.168.10.12    0002.16CE.E0F3          Jun 15 2024 12:31 PM    Automatic
192.168.20.11    00D0.BC0F.0D4C          Jun 15 2024 12:32 PM    Automatic
```

### Test 3: Проверка на DHCP pool статистики
```cisco
R1# show ip dhcp pool

Pool ADMIN_POOL :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 2
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.10.12        192.168.10.1     - 192.168.10.254   2
```

### Test 4: Проверка на DHCP статистиката
```cisco
R1# show ip dhcp server statistics

Memory usage         12345
Address pools        2
Database agents      0
Automatic bindings   4
Manual bindings      0
Expired bindings     0
Malformed messages   0
```

### Test 5: Тестване на connectivity
```
От PC1:
ping 192.168.10.1     (gateway)
ping 192.168.20.1     (другия gateway)
ping 8.8.8.8          (DNS server)
```

---

## ЧАСТ 6: DHCP Relay Agent (BONUS - Advanced)

### Scenario:
Ако имаме DHCP server на отделна мрежа или на втори router, използваме DHCP Relay.

### Пример - DHCP сървър на друга мрежа:

```cisco
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# ip helper-address 192.168.100.10
R1(config-subif)# exit
!
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# ip helper-address 192.168.100.10
R1(config-subif)# exit
```

**Какво прави `ip helper-address`?**
- Препраща DHCP DISCOVER broadcast като unicast към посочения сървър
- Позволява централизиран DHCP сървър за множество subnet-и

---

## ЧАСТ 7: Troubleshooting DHCP (15 мин)

### Проблем 1: PC не получава адрес

**Debug на Router:**
```cisco
R1# debug ip dhcp server events
R1# debug ip dhcp server packet
```

**Тествайте отново на PC:**
```
ipconfig /release
ipconfig /renew
```

**Гледайте output-а на Router - трябва да видите:**
```
DHCPD: DHCPDISCOVER received from client 0001.6373.7ABF
DHCPD: Sending DHCPOFFER to client 0001.6373.7ABF (192.168.10.11)
DHCPD: DHCPREQUEST received from client 0001.6373.7ABF
DHCPD: Sending DHCPACK to client 0001.6373.7ABF (192.168.10.11)
```

**Изключване на debug:**
```cisco
R1# undebug all
```

### Проблем 2: Грешен gateway

**Проверка на pool конфигурацията:**
```cisco
R1# show running-config | section dhcp
```

**Корекция:**
```cisco
R1(config)# ip dhcp pool ADMIN_POOL
R1(dhcp-config)# default-router 192.168.10.1
```

### Проблем 3: Адресите са изчерпани

**Проверка:**
```cisco
R1# show ip dhcp pool

Pool ADMIN_POOL :
 Total addresses                : 244
 Leased addresses               : 244    ← ПРОБЛЕМ!
```

**Решение:**
```cisco
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.5
! Намалихме excluded range
```

---

## ЧАСТ 8: Статична резервация (DHCP Reservation)

### Scenario: 
Искате принтер винаги да получава 192.168.10.50

### Стъпка 1: Намерете MAC адреса на устройството
```
На PC свързан към принтера:
arp -a
```

### Стъпка 2: Създайте ръчен binding
```cisco
R1(config)# ip dhcp pool PRINTER
R1(dhcp-config)# host 192.168.10.50 255.255.255.0
R1(dhcp-config)# client-identifier 01aa.bbcc.ddee.ff
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# exit
```

**Алтернатива с hardware-address:**
```cisco
R1(config)# ip dhcp pool PRINTER
R1(dhcp-config)# host 192.168.10.50 255.255.255.0
R1(dhcp-config)# hardware-address aabb.ccdd.eeff
```

---

## ЧАСТ 9: DHCP Snooping (Security - BONUS)

### Защита срещу Rogue DHCP Servers:

```cisco
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 10,20
!
SW1(config)# interface FastEthernet0/1
SW1(config-if)# description Trusted - to Router
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit
!
! Всички останали портове са untrusted по default
```

**Какво прави:**
- Блокира DHCP OFFER пакети от untrusted портове
- Позволява само от trusted (към легитимния DHCP сървър)

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Добавете трети DHCP pool
- Създайте VLAN 30 - Guest (192.168.30.0/24)
- Конфигурирайте DHCP pool с lease 1 ден
- Exclude първите 20 адреса
- Тествайте с нов PC

### Задача 2: Променете DNS сървърите
- Използвайте Cloudflare DNS: 1.1.1.1 и 1.0.0.1
- Обновете и двата pool-а
- Release и renew на всички PC-та

### Задача 3: Намалете lease time
- Променете lease на 2 часа за Guest VLAN
- Защо е полезно за guest мрежа?

### Задача 4: DHCP Options
Добавете допълнителни опции:
```cisco
R1(dhcp-config)# option 42 ip 192.168.10.5   ! NTP server
R1(dhcp-config)# option 150 ip 192.168.10.6  ! TFTP server
```

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ

### Проблем: "DHCP Request Failed"
**Причини:**
- Няма DHCP pool за този subnet
- Всички адреси са заети
- Interface на router е down

**Решение:**
```cisco
R1# show ip dhcp pool
R1# show ip interface brief
```

### Проблем: Получава адрес от грешен pool
**Причина:** PC е на грешен VLAN
**Решение:**
```cisco
SW1# show vlan brief
SW1# show mac address-table
```

### Проблем: Lease не се обновява
**Причина:** Router clock не е настроен
**Решение:**
```cisco
R1# clock set 14:30:00 15 June 2024
```

---

## ДОКУМЕНТАЦИЯ ЗА ПРОЕКТА

### Таблица с DHCP Pools:

| Pool Name   | VLAN | Network          | Gateway       | DNS           | Lease | Excluded Range      |
|-------------|------|------------------|---------------|---------------|-------|---------------------|
| ADMIN_POOL  | 10   | 192.168.10.0/24  | 192.168.10.1  | 8.8.8.8       | 7 d   | .1-.10              |
| IT_POOL     | 20   | 192.168.20.0/24  | 192.168.20.1  | 8.8.8.8       | 7 d   | .1-.10              |
| GUEST_POOL  | 30   | 192.168.30.0/24  | 192.168.30.1  | 1.1.1.1       | 1 d   | .1-.20              |

### Команди за верификация:
```cisco
show ip dhcp binding
show ip dhcp pool
show ip dhcp server statistics
show ip dhcp conflict
```

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ DHCP pool създаден за всеки VLAN
☐ Excluded addresses конфигурирани
☐ Default gateway правилно зададен
☐ DNS servers добавени
☐ Lease time определен
☐ PC-тата получават адреси автоматично
☐ Ping работи между всички устройства
☐ Show ip dhcp binding показва всички клиенти
☐ Документирани са всички pools
☐ Конфигурацията е запазена
```

---

## КАКВО НАУЧИХМЕ

1. ✅ DHCP концепция и DORA процес
2. ✅ Конфигурация на DHCP pools на Router
3. ✅ Excluded addresses
4. ✅ DHCP options (gateway, DNS, lease time)
5. ✅ DHCP binding и статистики
6. ✅ Troubleshooting с debug команди
7. ✅ DHCP Relay Agent (ip helper-address)
8. ✅ DHCP Snooping за сигурност

---

## СЛЕДВАЩА СТЪПКА

В **Lab 4** ще научим:
- DNS конфигурация
- Как да създадем локален DNS сървър
- Name resolution в мрежата

**Запазете файла като:** `Lab3_DHCP_YourName.pkt`
