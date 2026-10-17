# LAB 4: DNS КОНФИГУРАЦИЯ
## Domain Name System - Name Resolution

**Цел:** Да се научите да конфигурирате DNS сървър и да разберете как работи преобразуването име → IP адрес.

**Продължителност:** 60-75 минути

**Предварителни изисквания:** Завършени Lab 1-3

**Начален файл:** Отворете `Lab3_DHCP_YourName.pkt` (основния файл: R1 е единственият DHCP сървър, Fa0/11 и 192.168.10.2 са свободни) и го запазете като `Lab4_DNS_YourName.pkt`.

---

## ЧАСТ 1: Какво е DNS? (Теория - 10 мин)

### Проблем без DNS:

```
Искате да отворите www.google.com

❌ Трябва да помните: 142.250.185.206
❌ За Facebook: 157.240.241.35
❌ За YouTube: 142.250.185.78
❌ За всеки сайт - различен IP адрес!

Невъзможно е да запомните стотици IP адреси! 
```

### Решение с DNS:

```
✅ Пишете: www.google.com
✅ DNS превежда → 142.250.185.206
✅ Browser се свързва с IP адреса

Вие помните ИМЕНА, DNS помни IP АДРЕСИТЕ! 
```

### Аналогия: DNS е телефонен указател

```
┌─────────────────────────────────┐
│   ТЕЛЕФОНЕН УКАЗАТЕЛ            │
├─────────────────────────────────┤
│ Иван Петров    → 0888 123 456  │
│ Мария Георгиева → 0899 987 654  │
│ Пицария "Вкусно"→ 02 123 4567   │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│   DNS SERVER                    │
├─────────────────────────────────┤
│ www.google.com → 142.250.185.206│
│ web.company.local → 192.168.10.3│
│ server.company.local → 192.168.10.2│
└─────────────────────────────────┘
```

Вместо да помните номера, търсите по име!

---

### Как работи DNS? (Опростен процес)

```
Стъпка 1: Потребител пише име
┌─────────┐
│  PC1    │  "Искам да отворя www.company.local"
└────┬────┘
     │
     │
Стъпка 2: PC проверява локален кеш
     │  "Имам ли запазен IP за това име?"
     │  Не → Продължава към Стъпка 3
     │
     │
Стъпка 3: PC пита DNS Server
     │  "Какъв е IP адресът на www.company.local?"
     │
     ↓
┌─────────┐
│DNS Server│
└────┬────┘
     │
     │
Стъпка 4: DNS Server отговаря
     │  "www.company.local е 192.168.10.3"
     │
     ↓
┌─────────┐
│  PC1    │  Сега зная IP-то! Свързвам се...
└─────────┘
```

---

### Типове DNS записи

DNS използва различни типове записи за различни цели:

| Тип | Име | Какво прави | Пример |
|-----|-----|-------------|--------|
| **A** | Address | Име → IPv4 адрес | `web.company.local` → `192.168.10.3` |
| **AAAA** | IPv6 Address | Име → IPv6 адрес | `web.company.local` → `2001:db8::1` |
| **CNAME** | Canonical Name | Псевдоним → Друго име | `www.company.local` → `web.company.local` |
| **MX** | Mail Exchange | Mail сървър за домейна | `company.local` → `mail.company.local` |
| **NS** | Name Server | DNS сървърът на домейна | `company.local` → `ns1.company.local` |
| **PTR** | Pointer | IP → Име (обратно) | `192.168.10.3` → `web.company.local` |

**Забележка:** Таблицата описва реален DNS. В Packet Tracer DNS сървърът поддържа записи **A, CNAME, SOA и NS**; AAAA, MX и PTR не могат да се създадат.

### Най-важните за нашия Lab:

**A Record (Address):**
```
server.company.local  →  192.168.10.2
```
Директна връзка: **име** → **IP адрес**

**CNAME Record (Alias):**
```
www.company.local  →  web.company.local  →  192.168.10.3
      ↓                      ↓
  псевдоним          реално име (A record)
```
CNAME сочи към ДРУГО ИМЕ, което има A record.

---

### TTL (Time To Live)

**TTL** = Колко време да се пази записът в кеша

```
Пример: TTL = 3600 секунди (1 час)

1. PC пита DNS за www.company.local
2. DNS отговаря: "192.168.10.3" (TTL: 3600)
3. PC запазва в кеша си за 1 час
4. Следващите заявки → Използва кеша
5. След 1 час → Пита DNS отново
```

**Типични стойности:**
- 3600 секунди (1 час) - стандартно за нашия Lab
- 86400 секунди (1 ден) - стабилни записи

**Забележка:** В Packet Tracer TTL не се въвежда за всеки A или CNAME запис; задава се в SOA записа на зоната.

---

## ЧАСТ 2: Топология (5 мин)

### Разширяване на топологията

Добавяме DNS Server към нашата мрежа:

```
              [Router R1]
                   |
            GigE0/0 (trunk)
                   |        Gi0/1 (trunk)
              [Switch SW1] ---------------------------[Switch SW2]
       /       |     |     |     \     \     \          /      \
   Fa0/11  Fa0/6 Fa0/2 Fa0/3 Fa0/4 Fa0/5 Fa0/7        Fa0/2  Fa0/3
      |       |     |     |     |     |     |           |      |
  [Server] [Server][PC1] [PC2] [PC3] [PC4] (празен)   [PC5]  [PC6]
  DNS(10.2) File   Admin Admin  IT    IT              VLAN10 VLAN20
           (10.50)
           бонус Lab3
```

**VLAN-и:**
- VLAN 10: Administration (192.168.10.0/24)
- VLAN 20: IT Department (192.168.20.0/24)

**+ SW2 (Access):** свързан към SW1 Gi0/1 с trunk; PC5 (VLAN 10) и PC6 (VLAN 20) - от Lab 2, Част 3А.

**Устройства:**
- DNS Server (нов): 192.168.10.2
- File Server (ако е изпълнен бонусът от Lab 3): 192.168.10.50
- Web Server (в ЧАСТ 7): 192.168.10.3 (ще го добавим)

---

## ЧАСТ 3: Конфигурация на Switch (5 мин)

### Назначаване на порта към VLAN

```cisco
SW1# configure terminal
SW1(config)# interface FastEthernet0/11
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description DNS-Server
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

**Защо VLAN 10?** 
Сървърите обикновено са в администраторски VLAN за по-добра сигурност и управление.

---

## ЧАСТ 4: Конфигурация на DNS Server (20 мин)

### Стъпка 1: IP конфигурация на сървъра

**Кликнете: Server → Desktop → IP Configuration**

```
IP Address: 192.168.10.2
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
DNS Server: 127.0.0.1  (посочва към себе си)
```

**Защо DNS = 127.0.0.1?**
- 127.0.0.1 = localhost (самия сървър)
- DNS Server използва себе си за name resolution

### Стъпка 2: Активиране на DNS Service

**Кликнете: Server → Services → DNS**

1. Уверете се че **DNS Service е ON**
2. Ще видите празна таблица с DNS records

### Стъпка 3: Добавяне на A records

**A Record** = Име → IP адрес

**Добавете следните записи:**

| Name | Type | Address |
|------|------|---------|
| server.company.local | A | 192.168.10.2 |
| router.company.local | A | 192.168.10.1 |
| switch.company.local | A | 192.168.99.2 |
| web.company.local | A | 192.168.10.3 |
| pc1.company.local | A | 192.168.10.11 |
| pc2.company.local | A | 192.168.10.12 |

**Забележка:** Адресите на PC1 и PC2 са получени от DHCP и могат да се различават. Проверете ги с `ipconfig` и въведете реалните. Тези записи не се обновяват автоматично при смяна на адреса.

**Как се добавят:**
1. Name: `server.company.local`
2. Type: **A Record** (по подразбиране)
3. Address: `192.168.10.2`
4. Кликнете **Add**
5. Повторете за всички

### Стъпка 4: Добавяне на CNAME records (Aliases)

**CNAME Record** = Псевдоним → Друго име

| Name | Type | Host Name |
|------|------|-----------|
| www.company.local | CNAME | web.company.local |
| mail.company.local | CNAME | server.company.local |
| dns.company.local | CNAME | server.company.local |

**Забележка:** За CNAME записа Packet Tracer показва поле **Host Name** (каноничното име), а не Address.

**Обяснение:**
```
www.company.local → web.company.local → 192.168.10.3
      ↓                    ↓
   CNAME               A Record
```

---

## ЧАСТ 5: Обновяване на DHCP (10 мин)

### Промяна на DHCP да сочи към DNS сървъра

**Защо?** PC-тата автоматично да получават DNS Server адрес.

```cisco
R1# configure terminal
R1(config)# ip dhcp pool ADMIN_POOL
R1(dhcp-config)# dns-server 192.168.10.2
R1(dhcp-config)# domain-name company.local
R1(dhcp-config)# exit
!
R1(config)# ip dhcp pool IT_POOL
R1(dhcp-config)# dns-server 192.168.10.2
R1(dhcp-config)# domain-name company.local
R1(dhcp-config)# exit
```

**Забележка:** Ако Packet Tracer не приеме `domain-name` в DHCP pool-а (`% Invalid input`), пропуснете реда. DNS работи и без него, защото в тестовете се използват пълните имена (напр. `server.company.local`).

**Ако в Lab 3 сте избрали Server DHCP (Метод 2):** горните команди не се изпълняват. Сменете DNS Server на 192.168.10.2 в pool-овете на DHCP сървъра (Services → DHCP).

### Release и Renew на PC-тата

**На всеки PC (Command Prompt):**
```
ipconfig /release
ipconfig /renew
```

**Проверка:**
```
ipconfig /all
```

Трябва да видите:
```
DNS Server: 192.168.10.2
```

---

## ЧАСТ 6: Тестване на DNS (15 мин)

### Test 1: Ping по име (вместо IP)

**От PC1 (Command Prompt):**
```
ping server.company.local
```

**Очакван резултат:**
```
Pinging server.company.local [192.168.10.2] with 32 bytes of data:
Reply from 192.168.10.2: bytes=32 time<1ms TTL=128
Reply from 192.168.10.2: bytes=32 time<1ms TTL=128
```

✅ Виждате IP адреса в скоби → DNS работи!

### Test 2: nslookup команда

**nslookup** = Name Server Lookup (проверка на DNS)

**От PC (Command Prompt):**
```
nslookup server.company.local
```

**Очакван изход:**
```
Server: 192.168.10.2
Address: 192.168.10.2

Name: server.company.local
Address: 192.168.10.2
```

**Обяснение:**
- **Server:** Кой DNS server отговори
- **Name:** Името което търсихме
- **Address:** IP адресът на името

### Test 3: CNAME resolution

```
nslookup www.company.local
```

**Очакван изход:**
```
Server: 192.168.10.2
Address: 192.168.10.2

Name: web.company.local
Address: 192.168.10.3
Aliases: www.company.local
```

**Обяснение на процеса:**
```
1. Търсим: www.company.local
2. DNS намира: CNAME → web.company.local
3. Търси A record за: web.company.local
4. Намира: 192.168.10.3
5. Връща: 192.168.10.3 (Alias: www.company.local)
```

### Test 4: Тестване на несъществуващо име

```
ping fake.company.local
```

**Очакван резултат:**
```
Ping request could not find host fake.company.local.
```

❌ DNS не може да намери това име → грешка

---

## ЧАСТ 7: Добавяне на Web Server (BONUS - 15 мин)

**Забележка:** Това е НОВ сървър (не File Server-ът от Lab 3).

### Стъпка 1: Добавете Web Server

**Устройство:**
- 1x Server-PT (НОВ!)
- Име: Web Server
- IP: 192.168.10.3
- Subnet: 255.255.255.0
- Gateway: 192.168.10.1
- DNS: 192.168.10.2

**Свързване:**
```cisco
SW1(config)# interface FastEthernet0/7
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description Web-Server
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

**Забележка:** Използваме Fa0/7 (Fa0/6 е зает от File Server, ако е изпълнен бонусът от Lab 3).

### Стъпка 2: Конфигурация на HTTP Service

**Server → Services → HTTP:**

1. **HTTP Service:** ON
2. Кликнете **index.html**
3. **Изтрийте** стария код
4. **Копирайте** следния HTML:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Company Website</title>
    <style>
        body { font-family: Arial; text-align: center; padding: 50px; }
        h1 { color: #0066cc; }
    </style>
</head>
<body>
    <h1>Welcome to Company.Local</h1>
    <p>This is the internal company website.</p>
    <p>DNS Resolution is working! 🎉</p>
    <hr>
    <p>Server: 192.168.10.3</p>
</body>
</html>
```

5. Кликнете **Save**

### Стъпка 3: Актуализирайте DNS

**На DNS Server (192.168.10.2):**
- Вече има запис: `web.company.local` → `192.168.10.3`

### Стъпка 4: Тестване от браузър

**От PC1 → Desktop → Web Browser:**

**URL:** `http://www.company.local`

**Трябва да видите:**
```
Welcome to Company.Local
This is the internal company website.
DNS Resolution is working! 
───────────────────────────
Server: 192.168.10.3
```

✅ **Успех!** DNS преобразува името → IP → браузърът зарежда страницата!

---

## ЧАСТ 8: Troubleshooting (10 мин)

### Проблем 1: "Ping request could not find host"

**Симптоми:**
```
ping server.company.local
Ping request could not find host server.company.local
```

**Проверки:**

**1. DNS Server правилен ли е?**
```
ipconfig /all

DNS Server: 192.168.10.2  ← Трябва да сочи към DNS Server
```

**2. DNS Server достъпен ли е?**
```
ping 192.168.10.2
```

**3. DNS Server има ли записа?**
- **Server → Services → DNS**
- Проверете дали `server.company.local` съществува в таблицата

### Проблем 2: DNS не resolve-ва

**На DNS Server:**
1. **Services → DNS**
2. Проверете дали **DNS Service е ON**
3. Проверете дали записите са добавени правилно (Name, Type, Address)

### Проблем 3: Работи от един PC, не от друг

**Проверка на PC:**
```
ipconfig /all
```

Вероятно DHCP не е обновен:
```
ipconfig /release
ipconfig /renew
ipconfig /all   ← Проверете DNS Server
```

### Проблем 4: CNAME не работи

**Уверете се че:**
1. CNAME записът съществува
2. Type е избран правилно като **CNAME**
3. Address сочи към **съществуващ A record**

**Пример:**
```
❌ ГРЕШНО:
www.company.local (CNAME) → 192.168.10.3
                            ↑
                    Директно към IP (трябва A record!)

✅ ПРАВИЛНО:
www.company.local (CNAME) → web.company.local (A) → 192.168.10.3
```

---

## ДОКУМЕНТАЦИЯ

### Таблица с DNS Records:

| Record Type | Name | Points To | IP Address |
|-------------|------|-----------|------------|
| A | server.company.local | Self | 192.168.10.2 |
| A | router.company.local | R1 | 192.168.10.1 |
| A | web.company.local | Web Server | 192.168.10.3 |
| CNAME | www.company.local | web.company.local | - |
| CNAME | dns.company.local | server.company.local | - |

### DHCP конфигурация със DNS:

```cisco
ip dhcp pool ADMIN_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.10.2
 domain-name company.local
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Добавете повече DNS записи
Създайте записи за:
- `printer.company.local` → 192.168.10.4
- `camera.company.local` → 192.168.20.5

(Адреси от резервирания диапазон .1–.10: 192.168.10.50 е File Server от Lab 3, а 192.168.20.100 е IoT Server в Lab 6.)

### Задача 2: Алтернативен домейн
- Добавете записи за `internal.net`
- Създайте A records с новия домейн
- Тествайте с ping

### Задача 3: Mail Server (MX Record - Advanced)

**Само теоретично:** Packet Tracer не поддържа MX записи. Освен това MX трябва да сочи към име с A запис, а не към CNAME (в този лаб `mail.company.local` е CNAME).
```
Type: MX
Name: company.local
Address: mail.company.local
Priority: 10
```

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ DNS Server добавен (192.168.10.2)
☐ DNS Service е активиран (ON)
☐ A records са създадени
☐ CNAME records работят
☐ DHCP pools сочат към DNS server
☐ PC-тата могат да ping по име
☐ nslookup работи правилно
☐ (BONUS) Web browser зарежда www.company.local
☐ DNS records са документирани
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Какво е DNS и защо е важен (Име → IP)
2. ✅ Как работи DNS Query процес
3. ✅ Типове DNS records (A, CNAME)
4. ✅ TTL (Time To Live)
5. ✅ Конфигурация на DNS Server в Packet Tracer
6. ✅ Интеграция с DHCP (dns-server option)
7. ✅ nslookup команда за troubleshooting
8. ✅ Web browser с DNS resolution

---

## КОНТРОЛЕН ТЕСТ ПРЕДИ LAB 5

- `nslookup server.company.local` работи от PC1 **и** от PC3
- `ipconfig /all` показва DNS Server 192.168.10.2
- (ако има Web Server) `http://www.company.local` се отваря

---

## СЛЕДВАЩА СТЪПКА

В **Lab 5** ще научим:
- Access Control Lists (ACL)
- Контрол на трафика между VLAN-и
- Security policies

**Запазете файла като:** `Lab4_DNS_YourName.pkt`

---

**Успех! DNS е критичен компонент на всяка мрежа!**



**гл. ас. Светослав Атанасов**  
svetoslav.atanasov@trakia-uni.bg


<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
