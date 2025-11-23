# LAB 4: DNS КОНФИГУРАЦИЯ
## Domain Name System - Name Resolution

**Цел:** Да се научите да конфигурирате DNS сървър в Packet Tracer и да разберете как работи name resolution.

**Продължителност:** 60-75 минути

**Prerequisite:** Завършени Lab 1-3

---

## ЧАСТ 1: Разширяване на топологията (15 мин)

### Добавяне на DNS Server

**Нова топология:**
```
              [Router R1]
                   |
            GigE0/0 (trunk)
                   |
              [Switch SW1]
         /     |     |     \     \
      Fa0/1 Fa0/2 Fa0/3 Fa0/4 Fa0/5
        |     |     |     |     |
    [Server] [PC1] [PC2] [PC3] [PC4]
     DNS    Admin Admin  IT    IT
```

### Устройства:
- Добавете 1x Server-PT (за DNS)
- Свържете го към SW1 на порт Fa0/11

---

## ЧАСТ 2: Конфигурация на Switch (10 мин)

### Назначаване на порта към VLAN
```cisco
SW1# configure terminal
SW1(config)# interface FastEthernet0/11
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description DNS-Server
SW1(config-if)# exit
```

**Защо VLAN 10?** Сървърите обикновено са в администраторски VLAN.

---

## ЧАСТ 3: Конфигурация на DNS Server (20 мин)

### Стъпка 1: IP конфигурация на сървъра

**Кликнете на Server → Desktop → IP Configuration:**
```
IP Address: 192.168.10.2
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
DNS Server: 127.0.0.1  (посочва към себе си)
```

### Стъпка 2: Активиране на DNS Service

**Кликнете на Server → Services → DNS:**

1. Уверете се че DNS Service е **ON**
2. Ще видите празна таблица с DNS records

### Стъпка 3: Добавяне на DNS records

**Add следните A records:**

| Name              | Type | Address       | TTL  |
|-------------------|------|---------------|------|
| server.company.local | A    | 192.168.10.2  | 3600 |
| router.company.local | A    | 192.168.10.1  | 3600 |
| switch.company.local | A    | 192.168.99.2  | 3600 |
| web.company.local    | A    | 192.168.10.3  | 3600 |
| ftp.company.local    | A    | 192.168.10.4  | 3600 |
| pc1.company.local    | A    | 192.168.10.11 | 3600 |
| pc2.company.local    | A    | 192.168.10.12 | 3600 |

**Как се добавят:**
1. В полето "Name" напишете: `server.company.local`
2. Type: A Record (по default)
3. Address: `192.168.10.2`
4. Кликнете **Add**
5. Повторете за всички записи

### Стъпка 4: Добавяне на CNAME records (Aliases)

| Name              | Type  | Address              | TTL  |
|-------------------|-------|----------------------|------|
| www.company.local | CNAME | web.company.local    | 3600 |
| mail.company.local| CNAME | server.company.local | 3600 |
| dns.company.local | CNAME | server.company.local | 3600 |

**CNAME = Canonical Name (псевдоним)**

---

## ЧАСТ 4: Обновяване на DHCP за DNS (10 мин)

### Промяна на DHCP pools да сочат към DNS сървъра

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

## ЧАСТ 5: Тестване на DNS Resolution (15 мин)

### Test 1: Ping по име (вместо IP)

**От PC1 (Command Prompt):**
```
ping server.company.local
```

**Очакван резултат:**
```
Pinging server.company.local [192.168.10.2] with 32 bytes of data:
Reply from 192.168.10.2: bytes=32 time<1ms TTL=128
```

### Test 2: nslookup команда

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

### Test 3: nslookup с конкретен DNS server
```
nslookup router.company.local 192.168.10.2
```

---

## ЧАСТ 6: DNS на Router (като DNS Forwarder)

### Концепция:
Router може да действа като DNS proxy/forwarder за клиентите.

### Конфигурация (BONUS):
```cisco
R1(config)# ip dns server
R1(config)# ip domain-lookup
R1(config)# ip name-server 192.168.10.2
!
R1(config)# ip host router.company.local 192.168.10.1
R1(config)# ip host switch.company.local 192.168.99.2
```

**Какво прави:**
- Router приема DNS queries от клиентите
- Проверява локалните `ip host` записи
- Ако не намери, форвардва към 192.168.10.2

---

## ЧАСТ 7: Добавяне на Web Server (BONUS - 15 мин)

### Стъпка 1: Добавете Server за Web

**Добавете още 1 Server-PT:**
- IP: 192.168.10.3
- Subnet: 255.255.255.0
- Gateway: 192.168.10.1
- DNS: 192.168.10.2
- Свържете към SW1 на порт Fa0/6 (VLAN 10)

### Стъпка 2: Конфигурирайте HTTP Service

**Server → Services → HTTP:**

1. Включете HTTP (ON)
2. Кликнете на **index.html** за редакция
3. **Изтрийте** целия стар Cisco Packet Tracer код
4. **Копирайте и поставете** следния HTML код:
```

Следва новият код (на нов ред без backticks в markdown):
```
<!DOCTYPE html>
<html>
<head><title>Company Website</title></head>
<body>
  <h1>Welcome to Company.Local</h1>
  <p>This is the internal company website.</p>
  <p>DNS Resolution is working!</p>
</body>
</html>
```
```markdown
5. Кликнете **Save**
```

### Стъпка 3: Актуализирайте DNS

**На DNS Server (192.168.10.2):**
- Добавете: `web.company.local` A record → 192.168.10.3

### Стъпка 4: Тествайте от браузъра

**От PC1 → Desktop → Web Browser:**
```
http://www.company.local
```

**Трябва да видите вашия HTML!**


### От PC1 → Test: Ping CNAME record
```
ping www.company.local
```

**Какво се случва:**
```
www.company.local → (CNAME) → web.company.local → (A) → 192.168.10.3
```

---

## ЧАСТ 8: Troubleshooting DNS (10 мин)

### Проблем 1: "Ping request could not find host"

**Проверки:**
```
1. ipconfig /all     ← DNS server правилен ли е?
2. ping 192.168.10.2 ← DNS сървърът достъпен ли е?
3. nslookup server.company.local ← DNS resolving работи ли?
```

### Проблем 2: DNS не resolve-ва

**На DNS Server:**
- Services → DNS → Проверете дали service е ON
- Проверете дали записите са добавени правилно

### Проблем 3: Работи от един PC, но не от друг

**Проверка:**
```
ipconfig /all
```

Вероятно DHCP не е обновен:
```
ipconfig /release
ipconfig /renew
```

### Проблем 4: CNAME не работи

**Уверете се че:**
- CNAME record съществува
- Сочи към съществуващ A record
- Type е избран правилно като CNAME

---

## ЧАСТ 9: Advanced DNS - Multiple Domains

### Scenario: Искате вътрешен и външен домейн

**На DNS Server добавете:**

**Internal domain (company.local):**
- server.company.local → 192.168.10.2
- www.company.local → 192.168.10.3

**External domain (company.com) - симулирано:**
- www.company.com → 8.8.8.8 (примерен external IP)

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Добавете повече DNS записи
Създайте записи за:
- printer.company.local → 192.168.10.50
- camera.company.local → 192.168.20.100
- door.company.local → 192.168.20.101

### Задача 2: Тест с различни domain names
Добавете алтернативен домейн:
- internal.net вместо company.local
- Създайте съответните записи

### Задача 3: Mail Server (MX Record)
Добавете MX record:
```
Type: MX
Name: company.local
Address: mail.company.local
Priority: 10
```

### Задача 4: Round-Robin DNS
Създайте няколко A записа с едно име:
```
lb.company.local → 192.168.10.10
lb.company.local → 192.168.10.11
lb.company.local → 192.168.10.12
```
Тествайте с nslookup - ще видите всички адреси.

---

## ДОКУМЕНТАЦИЯ ЗА ПРОЕКТА

### Таблица с DNS Records:

| Record Type | Name                 | Points To            | IP Address    |
|-------------|----------------------|----------------------|---------------|
| A           | server.company.local | Self                 | 192.168.10.2  |
| A           | router.company.local | R1                   | 192.168.10.1  |
| A           | web.company.local    | Web Server           | 192.168.10.3  |
| CNAME       | www.company.local    | web.company.local    | -             |
| CNAME       | dns.company.local    | server.company.local | -             |

### Конфигурация на DHCP със DNS:
```cisco
ip dhcp pool ADMIN_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.10.2
 domain-name company.local
```

---

## DNS Lookup Process - Детайлно обяснение

### Какво се случва при: `ping www.company.local`

1. **PC1 проверява кеша си:**
   - Има ли вече записан IP за www.company.local?
   - Ако да → използва го директно
   - Ако не → продължава към стъпка 2

2. **PC1 изпраща DNS Query към DNS Server:**
   ```
   PC1 → UDP Port 53 → 192.168.10.2
   Query: "What is the IP of www.company.local?"
   ```

3. **DNS Server обработва заявката:**
   - Проверява дали има www.company.local
   - Вижда че е CNAME → resolve-ва go web.company.local
   - Намира A record: web.company.local → 192.168.10.3

4. **DNS Server отговаря:**
   ```
   DNS Server → PC1
   Answer: "www.company.local is 192.168.10.3"
   ```

5. **PC1 прави ICMP ping:**
   ```
   PC1 → 192.168.10.3
   ICMP Echo Request
   ```

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ DNS Server е добавен на 192.168.10.2
☐ DNS Service е активиран
☐ A records са създадени
☐ CNAME records (aliases) работят
☐ DHCP pools сочат към DNS server
☐ PC-тата могат да ping по име
☐ nslookup работи правилно
☐ Web browser може да зареди www.company.local
☐ DNS records са документирани
☐ Troubleshooting е тестван
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Какво е DNS и как работи
2. ✅ Конфигурация на DNS Server в Packet Tracer
3. ✅ A records и CNAME records
4. ✅ Интеграция с DHCP (dns-server option)
5. ✅ nslookup команда за troubleshooting
6. ✅ DNS resolution process
7. ✅ Router като DNS forwarder (bonus)

---

## СЛЕДВАЩА СТЪПКА

В **Lab 5** ще научим:
- Access Control Lists (ACL)
- Как да контролираме трафика между VLAN-и
- Security policies

**Запазете файла като:** `Lab4_DNS_YourName.pkt`

<script data-goatcounter="https://satanasov.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>

<script src="/SNA/assets/js/analytics-logger.js"></script>
