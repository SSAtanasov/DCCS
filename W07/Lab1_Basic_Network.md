# LAB 1: ОСНОВНА МРЕЖОВА ТОПОЛОГИЯ
## Базова конфигурация на Switch и Router

**Цел:** Да се научите да създавате проста мрежа и да конфигурирате основните параметри на устройствата.

**Продължителност:** 60-90 минути

**Необходимо:** Cisco Packet Tracer

---

## ЧАСТ 1: Изграждане на топологията (15 мин)

### Устройства за добавяне:
- 1x Router 2911
- 1x Switch 2960
- 3x PC (PC1, PC2, PC3)
- Cables: Copper Straight-Through

### Свързване:
```
Router GigabitEthernet0/0 ←→ Switch FastEthernet0/1
Switch FastEthernet0/2 ←→ PC1
Switch FastEthernet0/3 ←→ PC2  
Switch FastEthernet0/4 ←→ PC3
```

### Схема:
```
              [Router R1]
                   |
               GigE0/0
                   |
              [Switch SW1]
               /    |    \
           Fa0/2 Fa0/3 Fa0/4
             /     |     \
         [PC1]  [PC2]  [PC3]
```

---

## ЧАСТ 2: Конфигурация на Router (20 мин)

### Стъпка 1: Влизане в Router
```
Кликнете върху Router → CLI tab
Натиснете Enter
```

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

**Обяснение:**
- `hostname R1` - задава име на устройството
- `no ip domain-lookup` - изключва DNS търсенето (за по-бърза работа)
- `enable secret` - парола за privileged mode
- `logging synchronous` - предотвратява прекъсване на командите от log съобщения

### Стъпка 3: Конфигурация на интерфейса
```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description Connection to SW1
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Забележка:** Винаги използвайте `no shutdown` за да активирате интерфейса!

### Стъпка 4: Записване на конфигурацията
```cisco
R1(config)# exit
R1# copy running-config startup-config
[Enter]
```

**Защо е важно?** При рестартиране конфигурацията ще остане запазена.

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

### Стъпка 2: VLAN 1 management адрес
```cisco
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

### Стъпка 3: Default gateway
```cisco
SW1(config)# ip default-gateway 192.168.1.1
```

**Защо е нужно?** Switch-ът да може да комуникира с устройства извън своята мрежа.

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

### Стъпка 5: Записване
```cisco
SW1(config)# exit
SW1# copy running-config startup-config
```

---

## ЧАСТ 4: Конфигурация на PC-та (10 мин)

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

---

## ЧАСТ 5: Тестване на свързаността (15 мин)

### Test 1: Ping от PC1 до Router
```
PC1 Command Prompt:
ping 192.168.1.1
```

**Очакван резултат:** 4/4 успешни отговора

### Test 2: Ping между PC-та
```
От PC1:
ping 192.168.1.11
ping 192.168.1.12
```

### Test 3: Ping от Router до PC
```
R1# ping 192.168.1.10
```

### Test 4: Проверка на ARP таблицата
```
R1# show arp
```

**Какво трябва да видите:** MAC адресите на свързаните устройства

---

## ЧАСТ 6: Верификационни команди (10 мин)

### На Router:
```cisco
R1# show ip interface brief
R1# show running-config
R1# show ip route
```

### На Switch:
```cisco
SW1# show ip interface brief
SW1# show vlan brief
SW1# show mac address-table
SW1# show interfaces status
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Добави още един PC
- Добавете PC4 към Switch на порт Fa0/5
- IP: 192.168.1.13
- Тествайте свързаността

### Задача 2: Променете hostname-овете
- Променете Router на MyRouter
- Променете Switch на MySwitch

### Задача 3: Banner съобщение
Добавете приветствено съобщение:
```cisco
R1(config)# banner motd #
Enter TEXT message. End with the character '#'.
************************************************
*  Unauthorized access is strictly prohibited  *
*  All activities are logged                   *
************************************************
#
```

### Задача 4: Създайте второ мрежово устройство
- Добавете още 1 Switch
- Свържете го към първия Switch
- Добавете 2 PC към новия Switch
- IP адреси: 192.168.1.14 и 192.168.1.15

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ И РЕШЕНИЯ

### Проблем 1: "Destination host unreachable"
**Причина:** Интерфейсът не е активен
**Решение:** `no shutdown` на интерфейса

### Проблем 2: Червени триъгълници на връзките
**Причина:** Интерфейсите може да не са активирани
**Решение:** Изчакайте ~30 сек или използвайте Fast Forward Time

### Проблем 3: PC не получава IP
**Причина:** Грешен subnet mask или gateway
**Решение:** Проверете всички 3 параметъра внимателно

### Проблем 4: Switch няма IP connectivity
**Причина:** Липсва default-gateway
**Решение:** `ip default-gateway 192.168.1.1`

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ Топологията е създадена правилно
☐ Router има hostname R1
☐ Switch има hostname SW1
☐ Всички интерфейси са описани (description)
☐ Router интерфейс GigE0/0 е активен
☐ Switch VLAN1 има IP адрес
☐ Всички PC-та имат правилни IP настройки
☐ Ping работи между всички устройства
☐ Конфигурацията е запазена (startup-config)
☐ Show командите показват правилна информация
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Как се създава основна мрежова топология
2. ✅ Базова конфигурация на Router и Switch
3. ✅ Как се задават IP адреси
4. ✅ Важността на `no shutdown` команда
5. ✅ Как се тества свързаността с ping
6. ✅ Как се записва конфигурацията
7. ✅ Основни show команди за проверка

---

## СЛЕДВАЩА СТЪПКА

В **Lab 2** ще научим:
- Как да създаваме VLAN-и
- Inter-VLAN routing
- Trunk портове между switches

**Запазете Packet Tracer файла си като:** `Lab1_YourName.pkt`
