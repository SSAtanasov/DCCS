# LAB 2: VLAN КОНФИГУРАЦИЯ И INTER-VLAN ROUTING
## Сегментиране на мрежата с VLAN-и

**Цел:** Да се научите да създавате VLAN-и, да ги назначавате на портове и да конфигурирате Router-on-a-Stick.

**Продължителност:** 90-120 минути

**Prerequisite:** Завършен Lab 1

---

## ЧАСТ 1: Топология (15 мин)

### Устройства:
- 1x Router 2911 (R1)
- 1x Switch 2960 (SW1)
- 4x PC (PC1-Admin, PC2-Admin, PC3-IT, PC4-IT)

### Физическа свързаност:
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
        Admin  Admin   IT     IT
```

### VLAN дизайн:
```
VLAN 10 - Administration (192.168.10.0/24)
VLAN 20 - IT Department (192.168.20.0/24)
VLAN 99 - Management (192.168.99.0/24)
```

---

## ЧАСТ 2: Основна конфигурация (20 мин)

### Router базова настройка:
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

### Switch базова настройка:
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

---

## ЧАСТ 3: Създаване на VLAN-и на Switch (20 мин)

### Стъпка 1: Създаване на VLAN-и
```cisco
SW1(config)# vlan 10
SW1(config-vlan)# name Administration
SW1(config-vlan)# exit
!
SW1(config)# vlan 20
SW1(config-vlan)# name IT_Department
SW1(config-vlan)# exit
!
SW1(config)# vlan 99
SW1(config-vlan)# name Management
SW1(config-vlan)# exit
```

### Стъпка 2: Проверка на VLAN-ите
```cisco
SW1# show vlan brief
```

**Очакван резултат:**
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/6-24, Gig0/1-2
10   Administration                   active    
20   IT_Department                    active    
99   Management                       active
```

### Стъпка 3: Назначаване на портове към VLAN-и
```cisco
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description PC1-Admin
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description PC2-Admin
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/4
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC3-IT
SW1(config-if)# exit
!
SW1(config)# interface FastEthernet0/5
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC4-IT
SW1(config-if)# exit
```

### Стъпка 4: Конфигурация на trunk порт към Router
```cisco
SW1(config)# interface FastEthernet0/1
SW1(config-if)# description Trunk to R1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,99
SW1(config-if)# exit
```

**Обяснение:**
- `switchport mode trunk` - прави порта trunk (носи трафик за множество VLAN-и)
- `native vlan 99` - untagged трафик се приема като VLAN 99
- `allowed vlan` - само тези VLAN-и могат да минат през trunk-а

### Стъпка 5: Management VLAN адрес
```cisco
SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.99.1
```

### Стъпка 6: Записване
```cisco
SW1# copy running-config startup-config
```

---

## ЧАСТ 4: Router-on-a-Stick конфигурация (25 мин)

### Концепция:
Router-on-a-Stick е метод, при който един физически интерфейс на рутера се използва за routing между множество VLAN-и чрез subinterfaces.

### Стъпка 1: Конфигурация на subinterfaces
```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# no shutdown
R1(config-if)# description Trunk to SW1
R1(config-if)# exit
!
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# description Gateway for VLAN 10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit
!
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# description Gateway for VLAN 20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# exit
!
R1(config)# interface GigabitEthernet0/0.99
R1(config-subif)# description Gateway for Management VLAN
R1(config-subif)# encapsulation dot1Q 99 native
R1(config-subif)# ip address 192.168.99.1 255.255.255.0
R1(config-subif)# exit
```

**Важно:** 
- Номерът на subinterface (напр. `.10`) обикновено съвпада с VLAN ID
- `encapsulation dot1Q` указва IEEE 802.1Q tagging
- `native` keyword се използва само за native VLAN

### Стъпка 2: Записване
```cisco
R1# copy running-config startup-config
```

---

## ЧАСТ 5: Конфигурация на PC-та (15 мин)

### PC1-Admin (VLAN 10):
```
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
DNS Server: 192.168.10.1 (optional)
```

### PC2-Admin (VLAN 10):
```
IP Address: 192.168.10.11
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

### PC3-IT (VLAN 20):
```
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
```

### PC4-IT (VLAN 20):
```
IP Address: 192.168.20.11
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
```

---

## ЧАСТ 6: Тестване и верификация (20 мин)

### Test 1: Ping в рамките на същия VLAN
```
От PC1 (VLAN 10):
ping 192.168.10.11    (към PC2)
```

**Очакван резултат:** ✅ Успешен

### Test 2: Ping между различни VLAN-и
```
От PC1 (VLAN 10):
ping 192.168.20.10    (към PC3 в VLAN 20)
```

**Очакван резултат:** ✅ Успешен (чрез router)

### Test 3: Ping към gateway-ове
```
От PC1:
ping 192.168.10.1    (собствен gateway)
ping 192.168.20.1    (gateway на другия VLAN)
```

### Test 4: Проверка на routing
```
R1# show ip route
```

**Трябва да видите:**
```
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
C    192.168.99.0/24 is directly connected, GigabitEthernet0/0.99
```

### Test 5: Проверка на VLAN assignment
```
SW1# show vlan brief
SW1# show interfaces trunk
```

### Test 6: Traceroute между VLAN-и
```
От PC1:
tracert 192.168.20.10
```

**Очакван път:**
```
1    192.168.10.1    (gateway - router)
2    192.168.20.10   (destination PC3)
```

---

## ЧАСТ 7: Верификационни команди

### На Router:
```cisco
R1# show ip interface brief
R1# show vlans
R1# show interfaces trunk
```

### На Switch:
```cisco
SW1# show vlan brief
SW1# show interfaces trunk
SW1# show interfaces FastEthernet0/1 switchport
SW1# show mac address-table
```

---

## ДОПЪЛНИТЕЛНА КОНФИГУРАЦИЯ: VTP (Voice VLAN) - BONUS

### Ако искате да добавите IP Phone (optional):
```cisco
SW1(config)# vlan 100
SW1(config-vlan)# name Voice
SW1(config-vlan)# exit
!
SW1(config)# interface FastEthernet0/6
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# switchport voice vlan 100
SW1(config-if)# exit
```

---

## ЗАДАЧИ ЗА САМОСТОЯТЕЛНА РАБОТА

### Задача 1: Добавете трети VLAN
- Създайте VLAN 30 - Guest Network
- Назначете порт Fa0/6 към този VLAN
- Адресно пространство: 192.168.30.0/24
- Добавете subinterface на Router
- Добавете PC и тествайте

### Задача 2: Портова сигурност
Добавете Port Security на access портовете:
```cisco
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 1
SW1(config-if)# switchport port-security mac-address sticky
SW1(config-if)# switchport port-security violation restrict
```

### Задача 3: VLAN Hopping защита
```cisco
SW1(config)# interface range FastEthernet0/6-24
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 999
SW1(config-if-range)# shutdown
```

### Задача 4: Документация
Създайте таблица с:
- VLAN ID, име, subnet, gateway
- Port assignments
- Device IPs

---

## ЧЕСТО СРЕЩАНИ ПРОБЛЕМИ

### Проблем 1: Няма ping между VLAN-и
**Причини:**
- Router интерфейсът е shutdown
- Грешен encapsulation
- Native VLAN не съвпада

**Решение:**
```cisco
R1# show ip interface brief
R1# show vlans
SW1# show interfaces trunk
```

### Проблем 2: Trunk не работи
**Причина:** Native VLAN mismatch
**Решение:** 
```cisco
SW1(config-if)# switchport trunk native vlan 99
```

### Проблем 3: PC не може да стигне gateway
**Причина:** Грешен default gateway или subnet mask
**Решение:** Проверете IP конфигурацията на PC

---

## АНАЛИЗ НА ТРАФИКА

### Какво се случва при ping от PC1 (VLAN 10) към PC3 (VLAN 20)?

1. **PC1 → Switch:**
   - PC1 изпраща ICMP към 192.168.20.10
   - Разбира че не е в локалната мрежа → изпраща към gateway
   - Ethernet frame с Destination MAC = R1's MAC

2. **Switch → Router:**
   - Switch-ът получава frame на порт Fa0/2 (access VLAN 10)
   - Форвардва го през trunk (Fa0/1) с 802.1Q tag = 10

3. **Router (Inter-VLAN routing):**
   - Приема frame на GigE0/0.10
   - Routing decision: destination е в 192.168.20.0/24 → GigE0/0.20
   - Изпраща frame обратно към switch с tag = 20

4. **Switch → PC3:**
   - Получава tagged frame (VLAN 20) на trunk порт
   - Премахва tag-а
   - Форвардва на Fa0/4 (access port за PC3)

---

## CHECKLIST ЗА ЗАВЪРШВАНЕ

```
☐ Създадени са VLAN 10, 20, 99
☐ Портовете са правилно назначени към VLAN-ите
☐ Trunk е конфигуриран между SW1 и R1
☐ Native VLAN е 99 и на двете устройства
☐ Subinterfaces на Router са активни
☐ Всички PC имат правилни IP настройки
☐ Ping работи в рамките на един VLAN
☐ Ping работи между различни VLAN-и
☐ Show команди потвърждават конфигурацията
☐ Traceroute показва routing през R1
```

---

## КАКВО НАУЧИХМЕ

1. ✅ Как се създават и управляват VLAN-и
2. ✅ Access vs Trunk портове
3. ✅ IEEE 802.1Q tagging
4. ✅ Router-on-a-Stick концепция
5. ✅ Subinterfaces и encapsulation
6. ✅ Inter-VLAN routing
7. ✅ Native VLAN и security implications

---

## СЛЕДВАЩА СТЪПКА

В **Lab 3** ще научим:
- DHCP конфигурация (Router като DHCP server)
- DHCP Relay Agent
- Автоматично IP адресиране

**Запазете файла като:** `Lab2_VLANs_YourName.pkt`
