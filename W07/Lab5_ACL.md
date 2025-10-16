# LAB 5: ACCESS CONTROL LISTS (ACL)
## Контрол на мрежовия трафик и сигурност

**Цел:** Да се научите да създавате и прилагате ACL за контрол на достъпа между мрежи и VLAN-и.

**Продължителност:** 90-120 минути

**Prerequisite:** Завършени Lab 1-4

---

## ЧАСТ 1: Теория - Какво е ACL? (15 мин)

### Видове ACL:

**Standard ACL (1-99, 1300-1999):**
- Филтрира само по SOURCE IP адрес
- По-прости, ограничени възможности
- Прилагат се близо до destination

**Extended ACL (100-199, 2000-2699):**
- Филтрира по source IP, destination IP, protocol, port
- По-мощни и детайлни
- Прилагат се близо до source

### ACL Processing:
```
1. Проверява се всяко правило последователно (top-to-bottom)
2. При съвпадение (match) → действие (permit/deny)
3. Ако няма съвпадение → implicit deny (deny all)
```

### Важно правило:
**Implicit Deny All** - в края на всеки ACL има невидимо:
```
deny ip any any
```

---

## ЧАСТ 2: Топология за лаба (10 мин)

### Използваме същата топология + добавки:

```
              [Router R1]
                   |
            GigE0/0 (trunk)
                   |
              [Switch SW1]
         /     |     |     |     \
      DNS   Admin  Admin  IT    IT
     .10.2  .10.11 .10.12 .20.11 .20.12
```

### VLAN-и:
- VLAN 10 - Administration (192.168.10.0/24)
- VLAN 20 - IT Department (192.168.20.0/24)

---

## ЧАСТ 3: Standard ACL - Базов пример (20 мин)

### Scenario 1: Блокирай PC1 да достъпва IT VLAN

### Стъпка 1: Създаване на Standard ACL
```cisco
R1# configure terminal
R1(config)# access-list 10 deny host 192.168.10.11
R1(config)# access-list 10 permit any
```

**Обяснение:**
- ACL 10 (standard range 1-99)
- Deny трафик от 192.168.10.11 (PC1)
- Permit всичко останало (важно! иначе implicit deny)

### Стъпка 2: Прилагане на ACL към интерфейс
```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# ip access-group 10 out
R1(config-subif)# exit
```

**Защо OUT?** Трафикът излиза към VLAN 20 от router-а.

### Тест:
```
От PC1 (192.168.10.11):
ping 192.168.20.11     ← FAIL ❌

От PC2 (192.168.10.12):
ping 192.168.20.11     ← SUCCESS ✅
```

### Стъпка 3: Проверка на ACL
```cisco
R1# show access-lists
R1# show ip interface GigabitEthernet0/0.20
```

---

## ЧАСТ 4: Extended ACL - Advanced (30 мин)

### Scenario 2: Позволи само HTTP и HTTPS от IT към Admin VLAN

### Стъпка 1: Премахване на стария ACL (ако има)
```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# no ip access-group 10 out
R1(config-subif)# exit
R1(config)# no access-list 10
```

### Стъпка 2: Създаване на Extended ACL
```cisco
R1(config)# access-list 100 remark IT to Admin Access Control
R1(config)# access-list 100 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 80
R1(config)# access-list 100 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 443
R1(config)# access-list 100 permit icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
R1(config)# access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
R1(config)# access-list 100 permit ip any any
```

**Обяснение на wildcard mask:**
- 0.0.0.255 означава "игнорирай последния октет"
- 192.168.20.0 0.0.0.255 = всички адреси от 192.168.20.0 до .255

**Логика на ACL 100:**
1. Permit TCP port 80 (HTTP) от IT към Admin
2. Permit TCP port 443 (HTTPS) от IT към Admin
3. Permit ICMP (ping) от IT към Admin
4. Deny всичко останало от IT към Admin
5. Permit всичко останало (други VLAN-и)

### Стъпка 3: Прилагане на ACL
```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# ip access-group 100 in
R1(config-subif)# exit
```

**Защо IN?** Трафикът влиза в router от VLAN 20.

### Тестване:
```
От PC3 (IT VLAN):
ping 192.168.10.2      ← SUCCESS ✅ (ICMP permitnat)
ping 192.168.10.11     ← SUCCESS ✅

telnet 192.168.10.2 23 ← FAIL ❌ (Telnet не е разрешен)
```

---

## ЧАСТ 5: Named ACL (по-модерен подход) (20 мин)

### Предимства:
- По-четливи имена
- Лесно се редактират (add/remove rules)
- По-добра организация

### Scenario 3: Създаване на Named Extended ACL

### Стъпка 1: Създаване
```cisco
R1(config)# ip access-list extended ADMIN_TO_IT
R1(config-ext-nacl)# remark Allow admin VLAN full access to IT VLAN
R1(config-ext-nacl)# permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
R1(config-ext-nacl)# exit
```

### Стъпка 2: Прилагане
```cisco
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# ip access-group ADMIN_TO_IT out
R1(config-subif)# exit
```

### Стъпка 3: Добавяне на ново правило (на конкретна позиция)
```cisco
R1(config)# ip access-list extended ADMIN_TO_IT
R1(config-ext-nacl)# 15 deny tcp host 192.168.10.11 any eq 23
R1(config-ext-nacl)# exit
```

**Номерът 15** определя позицията в списъка.

---

## ЧАСТ 6: ACL за защита на мрежовите устройства (15 мин)

### Scenario 4: Позволи SSH само от Admin VLAN към Router

### Стъпка 1: Конфигурация на SSH на Router (ако не е направено)
```cisco
R1(config)# hostname R1
R1(config)# ip domain-name company.local
R1(config)# crypto key generate rsa
[Choose 1024]
!
R1(config)# username admin privilege 15 secret cisco123
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exit
```

### Стъпка 2: ACL за VTY линии
```cisco
R1(config)# access-list 50 remark SSH Access Control
R1(config)# access-list 50 permit 192.168.10.0 0.0.0.255
R1(config)# access-list 50 deny any log
!
R1(config)# line vty 0 4
R1(config-line)# access-class 50 in
R1(config-line)# exit
```

**Какво прави:**
- Само от 192.168.10.0/24 може SSH
- Всички останали се отказват (и се логват)

### Тестване:
```
От PC1 (192.168.10.11):
ssh -l admin 192.168.10.1     ← SUCCESS ✅

От PC3 (192.168.20.11):
ssh -l admin 192.168.10.1     ← FAIL ❌
```

---

## ЧАСТ 7: Time-based ACL (Advanced - BONUS)

### Scenario: Интернет достъп само в работно време

### Стъпка 1: Конфигурация на часовника
```cisco
R1# clock set 14:30:00 15 June 2024
```

### Стъпка 2: Дефиниране на time range
```cisco
R1(config)# time-range WORK_HOURS
R1(config-time-range)# periodic weekdays 08:00 to 18:00
R1(config-time-range)# exit
```

### Стъпка 3: ACL с time range
```cisco
R1(config)# ip access-list extended INTERNET_ACCESS
R1(config-ext-nacl)# permit ip 192.168.20.0 0.0.0.255 any time-range WORK_HOURS
R1(config-ext-nacl)# deny ip 192.168.20.0 0.0.0.255 any
R1(config-ext-nacl)# exit
```

---

## ЧАСТ 8: Troubleshooting ACL (15 мин)

### Често срещани грешки:

### Грешка 1: Implicit Deny All
**Проблем:** Създали сте ACL само с deny правила
**Решение:** Добавете `permit any` в края

```cisco
R1(config)# access-list 10 deny host 192.168.10.11
R1(config)# access-list 10 permit any     ← ВАЖНО!
```

### Грешка 2: ACL в грешна посока
**Проблем:** Прилагате ACL като IN вместо OUT
**Решение:** Помислете къде е source и къде е destination

### Грешка 3: Грешен интерфейс
**Проблем:** ACL е на GigE0/0 вместо на subinterface
**Решение:** Прилагайте на правилния subinterface

### Debug команди:
```cisco
R1# debug ip packet
R1# debug ip access-list
! Правете тест
R1# undebug all
```

### Проверка на ACL hits:
```cisco
R1# show access-lists
Access list 100
    10 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq www (5 matches)
    20 permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 443
    30 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 (12 matches)
```

**"matches"** показва колко пъти е използвано правилото.

---

## ЧАСТ 9: Port Security на Switch (BONUS)

### Допълнителна сигурност на Layer 2

```cisco
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 2
SW1(config-if)# switchport port-security mac-address sticky
SW1(config-if)# switchport port-security violation restrict
SW1(config-if)# exit
```

**Обяснение:**
- `maximum 2` - максимум 2 MAC адреса на порта
- `sticky` - автоматично запомня първите MAC-ове
- `violation restrict` - при нарушение спира трафика, но не shutdown-ва порта

### Проверка:
```cisco
SW1# show port-security interface FastEthernet0/2
SW1# show port-security address
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Блокирай конкретен хост
Създайте ACL, който блокира PC2 (192.168.10.12) да достъпва DNS server (192.168.10.2).

### Задача 2: Ограничи протоколи
Създайте ACL, който:
- Позволява ICMP (ping)
- Позволява DNS (UDP port 53)
- Блокира всичко останало от IT към Admin

### Задача 3: Multi-layer security
Комбинирайте:
- ACL на router за inter-VLAN traffic
- Port security на switch портовете
- SSH access control на управляващите устройства

### Задача 4: Log analysis
Добавете `log` keyword към deny правила:
```cisco
access-list 100 deny ip any any log
```
Направете тестове и прегледайте логовете:
```cisco
R1# show logging
```

---

## ДОКУМЕНТАЦИЯ ЗА ПРОЕКТА

### Таблица с ACL правила:

| ACL # | Name          | Type     | Purpose                    | Applied To      | Direction |
|-------|---------------|----------|----------------------------|-----------------|-----------|
| 10    | -             | Standard | Block PC1 to IT VLAN       | GigE0/0.20      | out       |
| 50    | -             | Standard | SSH access control         | VTY 0-4         | in        |
| 100   | -             | Extended | IT to Admin (HTTP/S only)  | GigE0/0.20      | in        |
| -     | ADMIN_TO_IT   | Extended | Admin full access to IT    | GigE0/0.10      | out       |

### ACL Best Practices:
```
✅ Прилагайте Extended ACL близо до source
✅ Прилагайте Standard ACL близо до destination
✅ Винаги добавяйте "permit any" в края (ако е нужно)
✅ Използвайте Named ACL за по-добра четливост
✅ Добавяйте remark за документация
✅ Тествайте преди да прилагате в production
✅ Записвайте конфигурацията след промени
```

---

## ACL Flowchart за решение

```
START
  ↓
Трябва ли да филтрирам само по source IP?
  ├─ ДА → Standard ACL (1-99)
  └─ НЕ → ↓
           ↓
     Трябва ли да филтрирам по:
     - Source IP
     - Destination IP  
     - Protocol
     - Port number
           ↓
     ДА → Extended ACL (100-199)
           ↓
     Приложи близо до SOURCE (входящ трафик)
           ↓
     Тествай с ping/telnet/traceroute
           ↓
     Провери с show access-lists
           ↓
END
```

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ Standard ACL създаден и тестван
☐ Extended ACL създаден и тестван
☐ Named ACL създаден
☐ ACL приложен в правилна посока (in/out)
☐ SSH access control конфигуриран
☐ Тестовете потвърждават permit правилата
☐ Тестовете потвърждават deny правилата
☐ Port security е активиран на критични портове
☐ ACL правилата са документирани
☐ Логовете са прегледани (show logging)
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Разликата между Standard и Extended ACL
2. ✅ Wildcard mask и как се използва
3. ✅ Как се прилага ACL към интерфейс (in/out)
4. ✅ Named ACL за по-добра четливост
5. ✅ ACL за VTY линии (SSH access control)
6. ✅ Implicit deny all правило
7. ✅ Troubleshooting с show и debug команди
8. ✅ Port security на Switch (Layer 2)

---

## СЛЕДВАЩА СТЪПКА

В **Lab 6** ще интегрираме:
- IoT устройства
- Автоматизация и логика
- Smart Home/Office сценарии

Това е финалната стъпка преди курсовия проект!

**Запазете файла като:** `Lab5_ACL_YourName.pkt`
