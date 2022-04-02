
# CCNP Security SENSS 300-206

__created_date__: 01.01.2015  
__last_update__: 14.07.2015  
__author__: @yugoslavskiy

## Contents

- [Description](#description)
- [Port Security](#port-security)
  + [CAM table overflow](#cam-table-overflow)
  + [DHCP Starvation/Exhaustion](#dhcp-starvationexhaustion)
- [DHCP Snooping](#dhcp-snooping)
  + [Rogue DHCP Servers](#rogue-dhcp-servers)
- [Dynamic ARP Inspection (DAI)](#dynamic-arp-inspection-dai)
  + [ARP-Spoofing/Poisonong](#arp-spoofingpoisonong)
- [Source Guard](#source-guard)
- [Storm Control](#storm-control)
- [Private VLANs](#private-vlans)
- [Private VLAN Edge a.k.a. Protected Ports](#private-vlan-edge-aka-protected-ports)
- [Secure Remote Management](#secure-remote-management)
  + [SSH Secure Configuration](#ssh-secure-configuration)
  + [SSH RSA Authentication](#ssh-rsa-authentication)
  + [CCP/HTTPS Secure Configuration](#ccphttps-secure-configuration)
- [NTP](#ntp)
- [Logging](#logging)
- [SNMP(v3)](#snmpv3)
- [CPPr](#cppr)
- [iACLs a.k.a. Flavors](#iacls-aka-flavors)
- [uRPF](#urpf)
- [NetFlow](#netflow)
- [ZBF on IOS Router](#zbf-on-ios-router)
  + [Configuration L3-L4 Inspection](#configuration-l3-l4-inspection)
  + [Configuration L4-L7 Inspection](#configuration-l4-l7-inspection)
- [AAA & Securing Management Plane](#aaa-securing-management-plane)
  + [AAA](#aaa)
  + [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
  + [Securing the Cisco IOS Image and Configuration Files](#securing-the-cisco-ios-image-and-configuration-files)
- [Best Practices a.k.a. Hardering Network Devices](#best-practices-aka-hardering-network-devices)
- [ASA 8.4 CLI Configuration & Troubleshooting](#asa-84-cli-configuration-troubleshooting)
- [Botnet Filtering](#botnet-filtering)
- [Packet Tracer](#packet-tracer)
- [Content Directiry Agent](#content-directiry-agent)
- [Management Tools](#management-tools)
- [IPv6 Security](#ipv6-security)
- [Useful Staff](#useful-staff)

## Description

Этот документ изначально являлся конспектом видеолекций Кита Баркера (Keith Barker) курса [CBT Nuggets Cisco CCNP Security 300-206 SENSS](https://www.cbtnuggets.com/it-training/cisco-ccnp-security-300-206-senss), содержащих в себе теоретические материалы необходимые для сдачи одного из четырех экзаменов трека CCNP Security.

В последствии, он был дополнен материалами о сетевых атаках, и, надеюсь, в дальнейшем будет продолжать дополняться.

Мы будем говорить о защите внутренней сети от различных атак канального/сетевого уровней (а также о реализации этих атак), защите сетевых устройств от DoS-атак, и в целом приведения внутрненней сетевой инфраструктуры в более защищенное состояния средствами коммутаторов и маршрутизаторов. 

Отталкиваться я буду от механизмов защиты (все-таки курс строителей), а от них уже (по возможности) расписывать атаки.

### Requirements

Для того чтобы понять материал на 100%, **необходимо**:

- обладать знаниями уровня CCNA R&S и опытом работы с сетями. Если без выебонов - необходимо понимать как ходит трафик, что такое коммутация и маршрутизация
- иметь под боком виртуалку [Kali Linux](https://www.kali.org/downloads/) и какую-нибудь сетевую железку (пусть даже домашний wi-fi "роутер") для отработки атак 
- при проведении каких-либо активных действий смотреть трафик с помощью [Wireshark](http://habrahabr.ru/company/pentestit/blog/204274/)

~~Тогда мы гарантируем Вам результат уже через две недели!~~

#### TODO:

- расписать все возможные сетевые атаки (MITM, Wi-Fi)
- синтаксис атак для всех утилит
- разбор методов атак/защиты IPv6

#### HOWTODO: 

- законспектировать курс того же Кита - [Penetration Testing with Linux Tools](https://www.cbtnuggets.com/it-training/penetration-testing-backtrack-kali-linux)

## Port Security

### Attacks

#### CAM table overflow:

##### Tool: ***macof***

Переполнение таблицы коммутации коммутатора (пара порт:mac-адрес). Перевод в режим постоянного обучения (коммутатор постоянно бродкастит фреймы), что позволяет нам, находясь за одним линком, видеть весь трафик, проходящий через все порты.

Условия выполнения: непосредственное подключение к коммутатору (L2-устройству) атакуемой сети.

##### Implementation:

+ Запускаем macof: ```(sudo) macof -i <интерфейс на который флудить>```
+ Захватываем входящий трафик с помощью wireshark

#### DHCP Starvation/Exhaustion:

##### Tool: ***yersinia***

DoS-атака, отправка DHCP-DISCOVER от разных MAC-адресов (исчерпание адресов).

Условия выполнения: различные, зависят от настроек. в некоторых случаях, в крупных сетях DHCP-сервер всего один, и из удаленных площадок все ходят на него через туннели, на роутерах настроена функция ip-helper, позволяющая DHCP Discover'ам ходить между подсетями.

##### Implementation

+ Запускаем yersinia через командрую строку: ```(sudo) yersinia -I```
+ Выбираемм NIC, нажав "i"
+ Выбираем режим атаки DHCP, нажав "g":
  - нажимаем "x" для открытия окна выбора атак
  - нажимаем "1" для запуска атаки

---

### Defense

Port Security - механизм cisco, который в том или ином виде реализован у всех вендоров. Позволяет управлять киличеством mac-адресов за конкретным портом коммутатора, и реакцией на нарушение установленных политик.

- Методы получения доверенных мак адресов:
  + Dynamic:
    - По умолчанию, 1 мак-адрес из таблиц динамически изученных
  + Static:
    - Захуячить статически mac-адрес/адреса. если захуячим 1, но укажем что может быть пять - будут добавлены динамически изученные.
  + Sticky:
    - Также подхватит динамически определенные мак-адреса (указать количество) и заебенит их в run config. необходимо сохранить конфиг для применения настроек.
- Violation Actions (действия при превышении кол-ва mac-аресов):
  + Protect (без уведомлений, тупо режет трафик)
  + Restrict (режет трафик + snmp, syslog, counters)
  + Shutdown interface (действие по умолчанию. если больше чем 1 было обнаружено. + snmp, syslog, counters)
  + Shutdown vlan (для конкретного VLAN + snmp, syslog, counters)
- Настройка на различных типах портов:
  + Access:
    - тут все по умолчанию
  + Trunk:
    - есть опции, можно установить различное число mac-адресов для каждого VLAN: `switchport port-security maximum 50 vlan 123`

### Configuration
```
!
! --- входим в режим конфигурирования:
!
conf t
!
! --- устанавливаем параметры интерфейса по умолчанию:
!
default int gig 0/2
!
! --- входим в режим конфигурирования интерфейса:
!
int gig 0/2
!
! --- устанавливаем интерфейс в режим access:
!
 switchport mode access
!
! --- устанавливаем VLAN (если не установим VLAN, применится к Native):
!
 switchport access vlan 123
!
! --- устанавливаем максимальное количество MAC-адресов, по умолчанию - 1:
!
 switchport port-security maximum 5
!
! --- устанавливаем тип старения MAC-адресов в таблице
! --- для этого порта по неактивности в течение 5 минут:
!
 switchport port-security aging type inactivity
 switchport port-security aging time 5
!
! --- устанавливаем действие при превышении порога-уведомление
! --- и дроп новых МАС-ов(по умолчанию - shutdown):
!
 switchport port-security violation restrict
!
! --- добавляем статический MAC-адрес:
!
 switchport port-security mac-address xxxx.xxxx.xxxx
!
! --- включаем режим Sticky, для запоминания и добавления 
! --- новых (еще 4) МАС-адресов в конфиг:
!
 switchport port-security mac-address sticky
!
! --- не забыть включить фичу!
!
 switchport port-security
!
```
### Tshoot
```
!
! --- проверим раннинг конфиг для интерфейса:
!
show run int gig 0/2

Building configuration...

Currennt configuration : 
!
interface GigabitEthernet0/2
 switchport access vlan 123
 switchport mode access
 switchport port-security maximum 5
 switchport port-security
 switchport port-security aging time 5
 switchport port-security violation restrict
 switchport port-security aging type inactivity
 switchport port-security mac-address xxxx.xxxx.xxxx
end
!
! --- проверим настройку port-security в целом:
!
show port-security address
!
! --- посмотрим настройку port-security для конкретного интерфейса:
!
show port-security interface gig 0/2
!
```

## DHCP Snooping

### Attacks:

#### DHCP Starvation/Exhaustion

Описание приведено выше.

#### Rogue DHCP Servers

##### Tool: ***yersinia***

Создание атакующего (подмена) DHCP-сервера. Может быть развитием DHCP Starvation/Exhaustion Attack - исчерпав легетимные ip-адреса, начать выдавать ответы (DHCP-OFFER и DHCP-ACK) клиентам, содержащие нужную атакующему информацию, реализовав в последситвии MITM-атаку посредством подмены Default Gateway или DNS-серверов, либо что-то иное на усмотрение атакующего.

Условия выполнения: нахождение в одном бродкаст-домене с жертвами.

##### Implementation

- Запускаем yersinia через командрую строку: ```(sudo) yersinia -I```
- Выбираемм NIC, нажав "i"
- Выбираем режим атаки DHCP, нажав "g":
  + нажимаем "x" для открытия окна выбора атаки
  + нажимаем "2" для выбора атаки Rogue DHCP server
- Выбираем параметры для атаки
- Запускаем wireshark для дампа трафика/проверки импакта атаки

---

### Defense

DHCP Snooping: 

- Enable on Switch
- Enable on VLAN
- ID Trusted Ports

Процесс запроса адреса по DHCP:

| Клиент     |  Действие  |  DHCP-Сервер |
| -----------|:------------:|--------------:
| D ->->->-> |  DISCOVER  | ->->->-> D   |
| O <<<<<<<- |  OFFER   | <<<<<<<- O   |
| R ->->->-> |  REQUEST   | ->->->->  R  |
| A <<<<<<<- |  ACK     | <<<<<<<-  A  |


Суть DHCP Snooping - что-то вроде firewall на уровне коммутации. Есть доверенные порты, и не доверенные. Если с не доверенного порта идет некая серверная DHCP-информация, типо DHCP-OFFER - такие кадры будут дропаться. Указывается доверенный порт, за которым тусуется DHCP, только оттуда могут гулять покеты типо DHCP-OFFER и DHCP-ACK. 

АХТУНГ!!! При активации функционала DHCP Snooping, активируется база-таблица DHCP Snooping binding, которая вдет учет получения юзерами ip-адресов от DHCP-сервера. В этой таблице хранятся mac-ip-адреса пользователей и порт(интерфейс), за которым находится клиент, получивший адрес.

Эта фича - лишь верхушка айсберга, на самом деле посредством нее будет осуществляться фильтрование на уровне коммутации, и это реально как firewall. На нее будут накатываться другие технологии, но в конечном счете блокировкой будет заниматься именно она. Так что сожми булки, юный читатель, это очень важная крутая хрень на которой дальше все будет строиться.

### Configuration
```
!
! --- входим в режим конфигурирования:
!
conf t
!
! --- включаем функционал DHCP-Snooping:
!
ip dhcp snooping
!
! --- указываем на файлик, в котором будет инфа по всем 
! --- MAC-IP-адресам, портам, VLAN'ам и так далее. это таблица:
!
ip dhcp snooping database flash:/snoopy.db
!
! --- также эта таблица может храниться на ? (+другие опции):
!
ip dhcp snooping database ?
  flash:        нужен URL
  ftp:          нужен URL
  https:        нужен URL 
  rcp:          нужен URL
  scp:          нужен URL
  tftp:         нужен URL
  timeout:      конфигурация таймаута прерывания
  write-delay   конфигурация таймера задержки для записи в URL
!
! --- яснопонятно, что надо юзать шифрованные каналы связи. scp или https.
!
! --- активируем для конкретного VLAN:
!
ip dhcp snooping vlan 123
!
! --- переходим в режим конфигурирования интерфейса:
!
interface g0/3
!
! --- хуячим ему описание:
!
description trunk leading to real DHCP server
!
! --- говорим что этот порт доверенный:
!
ip dhcp snooping trust
!
! --- уёбываем:
!
exit
!
```
По умолчанию, если функционал DHCP Snooping активирован, и в нем указан один доверенный порт, остальные начинают считаться НЕ доверенными, даже если в явном виде это указано не было.

Настройка ограничения скорости DHCP-Discover сразу на пачке портов (типо,юзерских портов):
```
!
! --- входим в режим конфигурирования пачки портов:
!
interface range g0/5-6
!
! --- дропаем все пакеты DHCP-DISCOVER, которые будут
! --- превышать частоту 10 пакетов в секунду:
!
ip dhcp snooping limit rate 10
!
! --- ну и накидываем port-security, чтобы избежать исчерпания
! --- пула адресов DHCP-сервера(DHCP Starvation/Exhaustion Attack):
!
switchport port-security maximum 5
switchport port-security
exit
!
```

### Tshoot
```
!
! --- инфа по работе фичи:
! 
show ip dhcp snooping
!
! --- посмотреть таблицу соотвествия DHCP Snooping binding 
! --- (ip-mac-interface - клиенты,которые получили от сервера адрес):
!
show ip dhcp snooping binding
!
! --- посмотреть что в таблице(базе данных) DHCP Snooping:
!
dir
Directory of flash:/
!

тут типо вывод всех файлов, а ля "ls -l"
!
! --- ну и поглядеть это добро можно так:
!
more flash:/snoopy.db
!
```

## Dynamic ARP Inspection (DAI)

### Attack

#### ARP-Spoofing/Poisonong

##### Tools: ***ettercap, Cain&Abel, intercepter-ng***

[Подробная статья](http://xgu.ru/wiki/ARP-spoofing)

Старо как мир, но пусть будет. Протокол ARP имеет одну неприятную особенность - доверие. Вне зависимости от того, спрашивает хост "ху из такой-то ip-адрес?", мы (атакующий) можем отправить хосту специально сформированный ответ, поместив тем самым в ARP-таблицу запись о том, что некий ip-адрес теперь имеет другой mac-адрес. 
Попсовый вариант атаки - отправлять ответы хосту о том что шлюз - это мы (атакующий), следовательно, трафик в интернет (или просто другие подсети) будет идти через нас. Средствами специальных утилит мы роутим получаемый от жертвы трафик на настоящий Default Gateway, тем самым, не нарушая его работу, при этом, обладая возможностью работать с трафиком жертвы (типичный MITM).

Условия выполнения: нахождение в одном широковещательном домене с жертвами.

##### Implementation

```
sudo ettercap -T -i <NIC> -w </path/to/pcap_file> -M ARP /<ip_1>/ /<ip_2>/

-T - текстовый интерфейс (есть GUI)
-i - выбор интерфейса
-M - вид атаки
ARP - метод реализации
-w - записать перехваченный трафик в файл 
ip_1 - жертва
ip_2 - шлюз
```

---
### Defense

**Dynamic ARP Inspection.**

+ Фиксит:
  - ARP Poisoning/Spoofing Attacks
  - Потенциально может пресекать сетевое сканирование о.О (arp-scan (-sn) - опция по умолчанию у nmap)
  - И на самом деле дохуя чего еще. [Интересная статья на русском, с живым примером](http://habrahabr.ru/post/192136/)

+ Dynamic ARP Inspection работает посредством:
  - DHCP SNOOPNIG DB/Table
  - ARP ACL - это если порт ведет к роутеру, который не является DHCP-клиентом и имеет статический ip-адрес. На доверенных портах НЕ РАБОТАЕТ.
  - Также, все проблемы решаются захуяриванием врукопашку всех пар mac/ip в arp-таблицу, что в небольших конторках может быть и неплохим решением.

Если мы врубаем DAI для конкретного VLAN, все порты в этом VLAN, по умолчанию, становятся НЕ ДОВЕРЕННЫМИ (untrusted).
Чтобы сделать какой-то порт доверенным, необходимо врукопашку в его настройках это прописать.
Короче, если дохуя свичей соединено транками, надо каждый транк ебошить в доверенные порты, ибо все перестанет работать.
Также, по умолчанию, на не доверенных портах выставляется ограничение частоты отправки ARP-запросов - 15 в секунду.
АХТУНГ! ПРИ ПРЕВЫШЕНИИ ЧАСТОТЫ ОТПРАВКИ ПАКЕТОВ ПОРТ ПЕРЕЙДЕТ В СОСТОЯНИЕ ERROR-DISABLE, АНАЛОГИЧНОЕ СОСТОЯНИЮ SHUTDOWN. 
Порт сам не поднимется, необходимо будет врукопашку его опускать и поднимать, подключаясь непосредственно к консоли.
Можно настроить автовосстановление, конфиг ниже.

### Configuration

```
!
! --- предполагается, что функционал DHCP Snooping включен, 
! --- ip-адреса розданы, табличка фичи имеет записи с MAC-адресами
! 
show ip dhcp snooping binding

MacAddress       IpAddress   Lease(sec)  Type       VLAN  Interface
-------------    -------     --------   ------    ----  ---------
B8:27:EB:51:1A:F6 | 10.123.0.2 | 82454 | dhcp-snooping | 123 | GigEther0/2<-- Это типо атакующий, по легенде

00:0C:29:16:57:AC | 10.123.0.3 | 85546 | dhcp-snooping | 123 | GigEther0/1<-- Это жертва, ок?

Total numbers of bindings: 2

!
! -- входим в режим конфигурирования:
!
conf t
!
! --- включаем DAI для VLAN 123:
!
ip arp inspection vlan 123
!
! --- ну, типо сразу глянем чо каво:
!
show ip arp inspection vlan 123

Source Mac Validation   : Disabled
Destination Mac Validation  : Disabled
IP Address Validation   : Disabled

Vlan    Configuration     Operation     ACL Match     Static ACL
----    -------------     ---------     ---------     ----------  
123     Enabled       Active      (ЩАС БУДЕТ)

Vlan    ACL Logging     DHCP Logging    Probe Logging
----    -----------     ------------    ------------- 
123     Deny        Deny        Off

!
! --- ахтунг! Щас все порты, включая транки, стали не доверенными для DAI
! --- крч, настраиваем интерфейсы:
!
int g0/1
 description trunk port
!
! --- так как это внатуре транк:
!
 ip arp inspection trust
!
! --- хитрожопо перепрыгиваем в режим конфигурирования другого интерфейса:
!
 int g0/2
 description user access port
!
! --- так как это рили акцесс порт, режем ему частоту 
! --- отправки ARP-запросов(10 в секунду). 
!
! --- он по умолчанию не доверенный, не будем это менять:
!
ip arp inspection limit rate 10 
!
! --- уёбываем в режим глобальной конфигурации:
!
 exit
!
! --- также, у нас висит роутер, порт к которому 
! --- он подключен - не доверенный (untrusted), хотя транк.
!
int g0/3
 description trunk to R1
 exit
!
! --- для него сделаем ARP ACL:
!
arp access-list OUR-ARP-ACL
 permit ip host 10.123.0.1 mac host 001f.9e00.ee89
 exit
!
! --- чекнем его:
!
show arp access-list OUR-ARP-ACL
ARP access list OUR-ARP-ACL
  permit ip host 10.123.0.1 mac host 001f.9e00.ee89

!
! --- ну и применим его к vlan 123:
!
ip arp inspection filter OUR-ARP-ACL vlan 123
!
! --- чекаем:
!
show ip arp inspection vlan 123

Source Mac Validation   : Disabled
Destination Mac Validation  : Disabled
IP Address Validation   : Disabled

Vlan    Configuration     Operation     ACL Match     Static ACL
----    -------------     ---------     ---------     ----------  
123     Enabled       Active      OUR-ARP-ACL   No

Vlan    ACL Logging     DHCP Logging    Probe Logging
----    -----------     ------------    ------------- 
123     Deny        Deny        Off
```
Итак. Если при данной настройке чувак с Kali (10.123.0.2) попытается заспуфить пользователя с адресом 10.123.0.3, представившись шлюзом (10.123.0.1), у него ничего не получится, и нам будут сыпаться логи о том, что DHCP Snooping заблочил конкретные кадры, ибо они не проходят по правилу ARP ACL.

При этом, злоумышленник, или жертва взлома, не теряют сетевой связности ни с кем, порт не блочится, пресекаются только попытки непосредственных атак.

Если же злоумышленник или жертва взлома попробуют, к примеру, просканировать сеть nmap-ом или другим сканнером, то они превысят установленный нами предел в 10 ARP-запросов в секунду для не доверенных портов (nmap при дефолтном сканиваронии использует ARP для определения живых хостов), и произойдет то, о чем я писал в начале - порт уйдет в down с состоянием ERROR-DISABLE. Для того, чтобы вернуть его к жизни, необходимо будет подключиться к консоли, зайти на интерфейс, врукопашку опустить его (shutdown) и поднять (no shutdown). После этого все заработает. НО! Можно настроить автоматическое восстановление, дабы не париться с этими штуками:
```
!
! --- восстановить порт из состояния errdisabled через 300 секунд,
! --- при условии, что несоответствие MAC-IP будет прекращено:
!
errdisable recovery cause arp-inspection
!
! --- можем указать интервал поменьше, в секундах:
!
errdisable recovery interval 30         
!
```
Также, можно включить проверку содержимого arp-запросов-ответов на предмет валидности MAC-адреса отправителя, получателя и ip-адреса. 

```
!
! --- анализировать будет информацию второго уровня,
! ---  откуда пришел кадр и всетакое
ip arp inspection validate ?
  dst-mac
  ip
  src-mac
!
! --- короче, можем все три сразу включить, одновременно
! 
ip arp inspection validate dst-macip src-mac
!
```

### Tshoot

```
!
! --- (общее состояние):
!
show ip arp inspection vlan 123
!
! --- (дропы, матчи, и пр):
!
show ip arp inspection statistics vlan 123
!
! --- просмотреть trusted и untrusted порты и статистику можно командой 
!
show ip arp inspection interfaces
!
```

## Source Guard

### Attack

#### IP Spoofing

утилиты: встроенные, либо packet crafting (scapy etc).

условия выполнения: различные.

#### MAC Spoofing

утилиты: встроенные, либо packet crafting (scapy etc).

условия выполнения: нахождение в одном широковещательном домене с жертвами.

### Defense

Source Guard.

Возможные опции:
- Source IP
- Source MAC

Работает посредством:
- DHCP Snooping Table/DB
- IP Source Binding Table 

Описание принципов работы:

Тут все еще проще, и завязка на той же технологии - DHCP Snooping. Если мы активируем этот функционал - начинается автоматическая валидация всех кадров на предмет соответствия IP-адреса - MAC-адресу кадра-порта-vlan-а, короче, согласно таблице DHCP Snooping binding. Если что-то не так - кадры дропаются.
Пояснение - все перечисленные с самого начала технологии могут работать (и должны работать) вместе. То есть, получается такой типо стек.

### Configuration

```
!
! --- заходим на интерфейс:
! 
int g0/2
!
! --- включаем фичу:
!
ip verify source port-security
!
! --- source - это ip-адрес. port-security... mac-адрес)
!
! --- IP Source Binding Table - эта штука создается руками! 
! --- если, к примеру, у нас нет DHCP, или надо доавить в некую запись,
! --- а мы это не можем сделать с DHCP Snooping binding Table. 
! --- не беда, крч, записываем всё таким образом:
!
ip source binding xxxx.xxxx.xxxx vlan 123 10.123.0.50 gig 0/2
!
! --- эта запись... Ну понятно, предоставляет нашей фиче 
! --- статическую запись соответствия, без DHCP Snooping table
!
```

### Tshoot

```
!
! --- посмотреть текущие работающие с этой технологией хосты:
!
show ip verify source
!
! --- посмотреть таблицы соответствия (ОБЕ. И DHCP SNOOP И IP Source Binding)
!
show ip verify source binding
!
! --- тут будет показано откуда взята запись, также 
! --- соответствие MAC-IP-VLAN-INTERFACE и вся фигня
!
```

## Storm Control

Фиксит:

- Сетевые штормы. Они же широковещательные/бродкаст штормы.

Работает посредством мониторинга:

- Бродкаста
- Мультикаста
- Юникаста

Срезание вышеперечисленного на основе:
  
- Процента  от общего трафика
- PPS (пакетов в секунду)
- BPS (бит в секунду)

Действия:

- Замедлить работу (срезанием части траифка)
- Завалиить порт нафиг

В общем-то, вылечить шторм можно лишь пофиксив петли, иначе это будет вечное зализывание ран. включаем stp и радуемся жизни.
```
Проверка на наличие шторма:
!
! --- сбрасываем счетчики
!
clear counters
!
! --- смотрим что у нас по пакетам. за секунду возрастает 
! --- в несколько раз, миллионы пакетов ниоткуда:
!
show int g0/12 | i packets
!
! --- чекнуть процессорную нагрузку:
!
show proccess cpu
!
! --- ништяковые графики за 60 секунд, 60 минут и 72 часа:
!
show proccess cpu history
!
```

### Configuration

```
!
! --- заходим на пачку интерфейсов:
!
int g0/2-28
!
! --- включаем фичу:
!
storm-control broadcast level ?
  <0.00 - 100.00>   - проценты
  bps         - бит в секунду
  pps         - пакетов в секунду 
!
! --- один из вариантов - уровнями. описать нижний и верхний пределы:
!
storm-control broadcast level pps 500 100
!
```
500 - верхний предел, 100 - нижний. так себе вариант, им не надо пользоваться. согласно лучшим практикам, оптимальным вариантом будет резать по процентному соотношению к общему трафику. бродкаст не должен превышать 10% от общего трафика. 
```
!
! --- вот так::
!
storm-control broadcast level 10
!
```
Еще одно чудное решение - класть порты при превышении уровня, то есть, при обнаружении шторма:
```
!
! --- с другой стороны сыпятся логи. сыпятся ли они без этой команды... хм
!
storm-control action shutdown
!
```
Ситуация будет аналогичной - порт в error-disable, либо восстанавливать руками, либо прописывать автовосстановление:
```
!
! --- посмотреть все порты в этом состоянии
! --- будет указана причина блокировки:
!
show int status err
!
! --- фигарим автовосстановление
!
errdisable recovery cause storm-control
!
! --- поправляем таймер чтобы было порезвее:
!
errdisable recovery interval 60
!
```
### Tshoot
```
!
! --- общие настройки
!
show storm-control broadcast
!
! --- посмотреть процесс автовосстановления из errordisable
! --- интерфейсов cколько секунд осталось и для каких причин
! --- блокировок разрешено автовосстановление:
!
show errdisable recovery
!
```

## Private VLANs

Чо делает:
Изолирует хосты/порты внутри подсети-VLANа. Дробление VLAN-а на субVLANы - домены (технология чем-то напоминает Q-in-Q).
Короче, основная цель - разделить подсеть на уровне L2. 

pVLAN работает посредством:

- Primary VLAN - несущая, представляется обычным VLAN-ом
- Secondary - подVLAN, субVLAN, короче, их может быть много и они гуляют внутри Primary VLAN. Бывают:
  + Isolated - изолированные VLAN, порты в них общаются только promiscuous-портами. ни к нему, ни у него нет доступа больше никуда по L2:
  + Community - групповые VLAN, в них порты общаются как между собой, так и с promiscuous-портами
- Различных типов портов:
  + Promiscuous - типо транк для pVLAN - общается со всеми pVLAN и с обычными. надо настраивать на аплинках
  + Isolated - изолированные порты, общаются только promiscuous-портами(аплинками). ни к нему, ни у него нет доступа больше никуда по L2
  + Community - групповые порты, они общаются как между собой, так и с promiscuous-портами(аплинками)

### Configuration
```
!
! --- cбрасываем настройки пачки интерфейсов в дефолт:
!
default int range g0/11-15
!
! --- pVLANs требуют VTP Transparent Mode, включаем:
!
 vtp mode transparent
!
! --- cоздаем VLAN
!
 vlan 500
!
! --- говорим что это групповой (Community) pVLAN:
!
 private-vlan community
!
! --- выходим из режима конфигурирования vlan:
!
 exit
!
! --- создаем еще пачку таких VLAN'ов:
!
vlan 400
 private-vlan community
 exit
vlan 300
 private-vlan community
 exit
! 
! --- создадим изолированный (Isolated) VLAN:
!
vlan 200
 private-vlan isolated
 exit
!
! --- создаем Primary VLAN, по которому все будет ходить наружу:
!
vlan 100
 private-vlan primary
!
! --- связываем все Secomdary VLAN'ы с Primary VLAN:
!
 private-vlan association 200,300,400,500
 exit
!
! --- зайдем в режим конфигурации аплинка, ведущего к роутеру:
!
int g0/11
!
! --- вводим его в режим Promiscuous, тем самым, активируем технологию pVLAN:
!
 switchport mode private-vlan promiscuous
!
! --- включаем в него те pVLAN'ы, которые необходимо будет связывать. 
! --- сначала primary, затем все остальные через пробел:
!
! --- еще раз. сначала Primary VLAN, затем пробел и
! --- через запятую все Secondary:
!
 switchport private-vlan mapping 100 200,300,400,500
 exit
!
! --- вешаем Secondary Isolated pVLAN 200 на интерфейсы:
!
int range g0/12-13
!
! --- активируем pVLAN и говорим что это режим "host", 
! --- противоположность "promiscuous":
!
 switchport mode private-vlan host
!
! --- свяжем Secondary Isolated pVLAN 200 с Primary pVLAN 100. 
! --- для этого сначала укажем Primary, затем, через пробел, Secondary:
!
 switchport private-vlan host-association 100 200
 exit
! 
! --- вешаем Secondary Community pVLAN 300 на интерфейс 
! --- и свяжем его с Primary по аналогии:
!
int g0/14
 switchport mode private-vlan host
 switchport private-vlan host-association 100 300
 exit
!
! --- повторяем для 400 и для 500:
!
int g0/15
 switchport mode private-vlan host
 switchport private-vlan host-association 100 400
 exit
int g0/16
 switchport mode private-vlan host
 switchport private-vlan host-association 100 500
 exit
!
```
Примечаение:
Все хосты, висящие ЗА портом Community смогут общаться между собой по L2, то есть, как угодно. Это означает, что настройка производится на коммутаторах уровня ядра-дистрибьюции, то есть предполагается, что на портах висят не хосты, а, допустим, неуправляемые коммутаторы.

### Tshoot
```
!
! --- Общая инфа по технологии покажет Primary, Secondary
! --- и Isolated VLAN, порты на которых висят:
!
show vlan private-vlan
!
! --- подробная инфа по интерфейсу в целом
! --- также будет указано что используется данная технология
! --- и подробности о маппинге, режиме, и тд и тп.
!
show int g0/11 switchport
!
```

## Private VLAN Edge a.k.a. Protected Ports

Чо делает:
То же самое что и Private VLAN Isolated, только в две команды. 
То есть, у нас УЖЕ присутствует некая настройка, два хоста на коммутаторе в одном VLANе и типо имеют доступ друг к другу.
В две команды можно сделать то же самое, что Private VLAN Isolated, то есть, сделать их какбы в одном VLAN'e но чтобы они не имели доступа друг к другу. 
Для пачки портов это делается командой range.

Все это работает только для одного коммутатора. Если необходимо задействовать данный функционал на двух коммутаторах - нужно использовать pVLAN Community.

### Configuration

```
!
! --- в режиме конфигурирования интерфейса, или диапазона интерфейсов:
!
switchport protected
!
```
ВСЁ.


## VACL

VLAN Access Control List = VACL = VLAN MAP

### Configuration

```
!
! --- глобально создадим расширенный ACL и обзовем его:
!
ip access-list extended UNWANTED-IP-PORT
!
! --- тут же провалимся в режим его конфигурирования:
!
 permit tcp 10.1.2.0 0.0.0.255 any eq 456
 permit udp 10.1.2.0 0.0.0.255 any eq 678
 exit
! 
! --- добавим MAC-ACL:
!
mac access-list extended UNWANTED-MAC
 permit host xxxx.xxxx.xxxx any
 exit
!
! --- создаем еще один ACL с целью разрешения трафика:
!
ip access-list extended ALLOWED-TRAFFIC
 permit ip any any
 exit
!
! --- теперь создаем VLAM MAP в которую добавим все VACL и MACL))
!
vlan access-map VACL 10
 match ip address UNWANTED-IP-PORT
 action drop
 exit
!
! --- не долбись в шары, команды отличаются! не перепутай!!!
!
vlan access-map VACL 20
 match mac address UNWANTED-MAC
 action drop
 exit
!
! --- не долбись в шары, команды отличаются! не перепутай!!!
!
vlan access-map VACL 30
 match ip address ALLOWED-TRAFFIC
 action forward
 exit
!
! --- прикрепляем это добро к VLAN:
!
vlan filter VACL vlan 55
!
```
Примечание:
Правило permit в ACL не означает ровным счетом ничего. 
То есть, пишем permit в ACL по умолчанию, а уже действие (drop/forward) определяем в vlan access map.


## pACL

Port Based Access Control List - ну тут все ясно

### Configuration:
```
!
! --- создаем ACL:
!
ip access-list extended NO-PING-TO-11
 deny icmp any host 10.123.0.11
 permit ip any any
 exit
!
! --- применяем его к интерфейсу:
!
int g0/2
 ip access-group NO-PING-TO-11 in
 exit
!
```
Примечание:
Тут все слава богу по стандарту, нормальный ACL, нормально применяешь его к интерфейсу и все огонь.


## MACsec

MACsec L2 Hop be Hop encription - симметричное шифрование о.О 
правда, не указано нифига что за алгоритм и пр...

### Configuration

```
!
! --- ахтунг. будем юзать 2 свича, SW1 и SW4.
! --- тут все завязано на неком CTS - Cisco TrustSec (проприетарный). 
! --- MACsec - это типо открытая технология.
!
! --- погнали. убедимся что сосед жив и мы его видим:
!
show cdp neighbor
Capability Codes:   
          R - Router, T - Trans Bridge, B - Source Route Bridge
          S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone,
          D - Remote, C - CVTA, M - Two-port MAC Relay

Device ID   Local Intrfce   Holdtime  Capability Platform   Port ID
SW4     Gig 0/26    130   S I     WS-C3550-   Gig 0/1
SW2     Gig 0/25    172   S I     WS-C3550-   Gig 0/1

!
! --- все ок, начинаем настройку. заходим на интерфейс:
!
int g0/26
!
! --- врубаем фичу:
!
cts manual
?
CTS manual configuration commands:
  default   Sec a command to its defaults  
  exit    Exit from CTS manual sub mode
  no      Negare a command or set its defaults
  policy    CTS policy for manual mode
  sap     CTS SAP configuration for manual mode

!
! --- установим ключ:
!
 sap pmk ABCD
!
! --- глянем как это выглядит в конфиге:
! 
do show run int g0/26
Building configuration...

Currennt configuration : 186 bytes
!
interface GigabitEthernet0/26
 switchport trunk encapsulation dot1q
 switchport mode trunk
 cts manual
  sap pmk 0000000000000000000000000000000000000000000000000000000000000ABCD

!
! --- как мы можем заметить, все в открытом виде хранится. 
! --- а как мы знаем, чтобы это пофиксить, надо глобально активировать 
! --- service password-encryption
!
!
! --- тут мы глядим что происходит с этой фичей,
! --- согласовала ли ключи с пиром, активна ли и пр:
!
show cts in
!
! --- сейчас на нашей стороне настроено шифрование, 
! --- на той-нет, связности по L2 нет.
! --- как подтверждение - глядим cdp:
!
show cdp neighbor
Capability Codes:   R - Router, T - Trans Bridge, B - Source Route Bridge
          S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone,
          D - Remote, C - CVTA, M - Two-port MAC Relay

Device ID   Local Intrfce   Holdtime  Capability Platform   Port ID
SW2     Gig 0/25    177     S I      WS-C3550-  Gig 0/1

!
! --- мы не получили ни одного кадра cdp за весь
! --- holdtime период и SW4 отвалился. таки дела.
! 
! --- чтобы вылечить, скопипейстить настройку
! --- MACsec/TrustSec на нужный интерфейс в SW4
!
```

### Tshoot

```
!
! --- глянуть на самом интерфейсе в конфиге:
!
show run int g0/26
!
! --- тут мы траблушим состояние туннеля. ну или типо того -.-
!
show cts in
!
```

## Secure Remote Management

Фиксит:
Brute Force атаки на ssh, перехват паролей, в целом укрепляет безопасность удаленного управления.

Короче, тут спич про усиление защищенности удаленного доступа (ssh).
Защита Management Plane.

Password Length
Local Privilege 15 User
Hide Plain Text Passwords
SSH Support
CCP Support -> Эта херня позволяет конфигурировать устройство по HTTP/HTTPS. Cisco Configuration Professional
ACLs for Mgmt. Access
Slowing Brute Force Attacks
Clipping Failed Login Attempts

НЕ ЗАБЫВАТЬ, ЧТО ПО ХОРОШЕМУ НУЖЕН Out-Of-Band (OOB) канал управления устройством (независимый от пользовательского трафика).

### SSH Secure Configuration

```
!
! --- для создания RSA-ключей необходим domain name (лобой)
! --- и пользователь, под которым будем заходить по ssh:
!
ip domain name NGSDistrbtn.com
!
! --- жестко зададим минимальную длину любого пароля = 10 символам
!
security passwords min-length 10
!
! --- создадим юзера с использованием команды (secret),
! --- дадим ему максимальные привелегии и стойкий пароль.
! --- что касается привилегий - тут какбе да, типо, даем рута.
! --- но иначе нельзя, тут не OS, чтобы права ограничивать,
! --- в конце концов мы лезем править конфиг, окей?
!
! --- "secret" означает что в раннинг конфиг попадет MD5 hash пароля,
! --- а не плоский текст, как в случае, если мы будем использовать "password".
! --- ныне, конеш, под командой "secret" cisco понимает
! --- SHA-1, но только для К9-х, которые, вроде как, по прежнему
! --- запрещены на ввоз ФСБ.
!
username admin privilege 15 secret SuPeR$Tr0nGP@55W0rD
!
! --- таким же "суперсильным"" методом мы защитим наш PEXEC (enable).
! --- юзай другой пароль:
!
enable secret Drug0iSuPeR$Tr0nGP@55W0rD
!
! --- создадим RSA-ключи:
!
crypto key generate rsa modulus 2048 label Our-RSA-Keys
!
! --- естественно, указываем что юзаем только ssh v2, ибо v1 имеет уязвимости:
!
ip ssh version 2
!
! --- задаем таймаут на попытку авторизации. типо, если пользователь
! --- будет пытаться зайти больше 30 секунд - вылетит по неактивности.
! --- 30 секунд:
!
ip ssh time-out 30
!
! --- указываем максимальное количество попыток входа за указанные 30 секунд:
!
ip ssh authentication-retries 5
!
! --- укажем задержку между попытками (любыми) аутентификации = 5  секунд:
!
login delay 5
!
! --- укажем блокировку пользователя на 30 секунд после 3 неудачных
! --- попыток авторизации за 30 секунд:
!
login block-for 30 attempts 3 within 30
!
! --- создадим ACL для подключения по ssh, котором опишем админские тачки
! --- т.к. описать нужно только источники - хватит и standard ACL:
!
ip access-list standard 5
 permit host 10.1.0.25
 permit host 10.1.0.26
 permit host 192.168.1.23
!
! --- укажем что хотим логи всех попыток зайти от левых адресов
!
 deny any log
!
! --- ну и как мы помним, все остальное будет отбрасываться
!
 exit
!
! --- активируем и настриваем сервис ssh.
! --- конфигурируем все виртуальные линии, от 0 до 15й:
!
line vty 0 15
!
! --- говорим что к ним можно подключаться только по ssh
!
 transport input ssh
!
! --- говорим что используем локальную базу для аутентификации:
!
 login local
!
! --- говорим, что подключаться по ssh могут только чуваки из ACL 5:
!
 access-class 5 in
 exit
!
! --- прогуляемся сами куданить по ssh
!
ssh -l ЮЗЕР -v 2 <ip-адрес>
!
```

### SSH RSA Authentication

Для того чтобы по простому гулять на родные циски без паролей. 
Фича поддерживается начиная с версии 15.X IOS.
По итогу мы увидим в раннинг конфиге hash RSA ключа.

На клиенте:

- Сгенерировать RSA-пару
- Юзать RSA аутентификацию

На роутере:

- Настроить ssh
- Закрепить за локальным пользователем публичный ключ, сгенерированный на клиенте

Короч, как на линуксах и маках сгенерировать RSA ключи - понятно.
На винде это может сделать PuTTY Key Generator (puttygen.exe)
Также, аутентификацию по ключам на винде можно задействовать с помощью Secure CRT ^^

#### Конфигурация SSH RSA Authentication

> Source: https://networklessons.com/uncategorized/ssh-public-key-authentication-cisco-ios/

> The key is printed on a single line, that’s fine but Cisco IOS only supports a maximum of 254 characters on a single line so you won’t be able to paste this in one go. There’s a useful Linux command you can use to break the public key in multiple parts:
```
fold -b -w 72 /home/ubuntu/.ssh/id_rsa.pub

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC80DsOF4nkk15V0V2U7r4Q2MyAwIbgQX/7
rqdUyNCTulliYZWdxnQHaI0WpvcEHQTrSXCauFOBqUrLZglI2VExOgu0TmmWCajW/vnp8J5b
ArzwIk83ct35IHFozPtl3Rj79U58HwMlJ2JhBTkyTrZYRmsP+r9VF7pYMVcuKgFS+gDvhbux
M8DNLmS1+eHDw9DNHYBA+dIaEIC+ozxDV7kF6wKOx59E/Ni2/dT9TJ5Qge+Rw7zn+O0i1Ib9
5djzNfVdHq+174mchGx3zV6l/6EXvc7G7MyXj89ffLdXIp/Xy/wdWkc1P9Ei8feFBVLTWijX
iilbYWwdLhrk7L2EQv5x ubuntu@HOST1
```
>We can remove the “ssh-rsa” part at the beginning and the comment at the end. This is just the public key:
```
AAAAB3NzaC1yc2EAAAADAQABAAABAQC80DsOF4nkk15V0V2U7r4Q2MyAwIbgQX/7
rqdUyNCTulliYZWdxnQHaI0WpvcEHQTrSXCauFOBqUrLZglI2VExOgu0TmmWCajW/vnp8J5b
ArzwIk83ct35IHFozPtl3Rj79U58HwMlJ2JhBTkyTrZYRmsP+r9VF7pYMVcuKgFS+gDvhbux
M8DNLmS1+eHDw9DNHYBA+dIaEIC+ozxDV7kF6wKOx59E/Ni2/dT9TJ5Qge+Rw7zn+O0i1Ib9
5djzNfVdHq+174mchGx3zV6l/6EXvc7G7MyXj89ffLdXIp/Xy/wdWkc1P9Ei8feFBVLTWijX
iilbYWwdLhrk7L2EQv5x
```
>Let’s add it to the router, I will use the username “LINUX_USER”:
```
R1(config)#ip ssh pubkey-chain 
R1(conf-ssh-pubkey)#username LINUX_USER
R1(conf-ssh-pubkey-user)#key-string
R1(conf-ssh-pubkey-data)#AAAAB3NzaC1yc2EAAAADAQABAAABAQC80DsOF4nkk15V0V2U7r4Q2MyAwIbgQX/7    
R1(conf-ssh-pubkey-data)#rqdUyNCTulliYZWdxnQHaI0WpvcEHQTrSXCauFOBqUrLZglI2VExOgu0TmmWCajW/vnp8J5b
R1(conf-ssh-pubkey-data)#ArzwIk83ct35IHFozPtl3Rj79U58HwMlJ2JhBTkyTrZYRmsP+r9VF7pYMVcuKgFS+gDvhbux
R1(conf-ssh-pubkey-data)#M8DNLmS1+eHDw9DNHYBA+dIaEIC+ozxDV7kF6wKOx59E/Ni2/dT9TJ5Qge+Rw7zn+O0i1Ib9
R1(conf-ssh-pubkey-data)#5djzNfVdHq+174mchGx3zV6l/6EXvc7G7MyXj89ffLdXIp/Xy/wdWkc1P9Ei8feFBVLTWijX
R1(conf-ssh-pubkey-data)#iilbYWwdLhrk7L2EQv5x
R1(conf-ssh-pubkey-data)#exit
R1(conf-ssh-pubkey-user)#exit
R1(conf-ssh-pubkey)#exit
```

Теперь делаем как надо:

```
!
! --- короче, предполагается что локальный пользователь (admin)
! --- создан и ssh настроен:
!
ip ssh pubkey-chain
#ip ssh public-chain
 username admin
  key-string 
  <сюда вставляем ssh pub key>
!
! --- конец ключа мы объявляем роутеру командой exit:
!
  exit
 end
!
```

Также, можно в явном виде указать с каких интерфейсов можно заходить по ssh или https.
К слову, на ASA в любом случае необходимо указывать интерфейс, с которого можно подключаться по ssh.
Все это называется защитой Management Plane.

### SSH RSA Authentication ASA

```
username <user> password <password> privilege 15
username <user> attributes
  ssh authentication publickey <тут сам ключ без ssh-rsa и коммента в конце>
```

#### Конфигурация ограничения доступа по SSH

```
!
! --- настроим это ограничение. заходим в настройку control plane:
!
control-plane host
!
! --- говорим что можно использовать подключение по SSH и HTTPS
! --- (все неуказанные здесь протоколы будут запрещены)
! --- ТОЛЬКО на интерфейсе gigeth 0/1. 
! --- на всех остальных интерфейсах указанные протоколы будут запрещены:
!
 management-interface g0/1 allow https ssh
 exit
!
```

### Tshoot

```
!
! --- посмотреть текущие подключения по ssh
!
show ssh
!
! --- посмотреть под кем зашли
!
who
!
! --- посмотреть наши привилегии в системе:
!
show privilege
!
! --- посмотреть настройки аутентификации пользователей:
!
show login
!
! --- посмотреть количество срабатываний ACL:
! 
show access-list 5
!
! --- поглядеть какие у нас привилегии при коннекте (who):
!
show privilege
!
! --- посмотреть созданные ключи (RSA, HTTPS):
!
show crypto key mypubkey rsa
!
```

### CCP/HTTPS Secure Configuration

```
!
! --- блочим http сервер (по умолчанию включен)
!
no ip http server
!
! --- включаем https
!
ip http secure-server
!
! --- говорим что используем локальную базу для аутентификации:
!
ip http authentication local
!
! --- говорим, что подключаться по http могут только чуваки из ACL 5:
!
ip http access-class 5
!
```

## NTP

NTP - нутыпонял.

### Configuration

```
!
clock timezone MSK
clock summer-time PDT recurring
do clock set 11:57:48 Jan 4 2015
!
! --- используем NTP-сервер:
!
ntp servet 38.229.71.1
!
! --- чекнуть соединение с ntp:
!
show ntp associations
!
! --- чекнуть статус работы ntp
!
show ntp status
!
! --- настройка аутентификации ntp. должна поддерживаться ntp-сервером:
!
ntp authentication-key 1 md5 SuperPassword
ntp trusted-key 1
ntp authenticate
!
```

### Tshoot

```
!
! --- чекнуть соединение с ntp:
!
show ntp associations
!
! --- чекнуть статус работы ntp:
!
show ntp status
!
```

## Logging

Уровни "важности" (severity) логов. Указываются при настройке отправки логов, типо, какого уровня логи будем принимать и обрабатывать:

| Level | Name      | Description               |
| ------|-------------------|---------------------------------------|
| 0   | Emergencies   |System is unusable.          |
| 1   | Alerts      |Immediate action needed.         |
| 2   | Critical    |Critical conditions.           |
| 3   | Errors      |Error conditions.            |
| 4   | Warnings    |Warning conditions.          |
| 5   | Notifications   |Normal, but significant conditions.  |
| 6   | Informational   |Informational messages.        |
| 7   | Debugging     |Highly detailed information based on current debugging that is turned on.      |

В качестве получателей логов можем указывать:

- console - тупо сыпет в консоль, обычно отрубают чтобы не мешала править конфиг
- vty lines - может отправлять на некий ssh/telnet клиент, допустим, если самописная балалайка работающая при подключении к консоли, на каком-нибудь bash+expect+grep+sed и тд
- buffer - будет тупо кешить, при перезагрузке - перетрет
- SNMP-сервер/менеджер - будем отправлять trap'ы
- Syslog-сервер - нутыпонял

### Configuration

```
!
! --- включение логирования:
!
logging on
!
! --- если лог-сервер будет лежать, циска какое-то 
! --- время будет сохранять логи в буфере:
!
logging buffered informational
!
! --- указываем лог-сервер:
!
logging host 192.168.1.23
!
! --- указываем уровень сообщений, которые будем отправлять на сервер (0-7)
! 
! --- в данном примере указываем максимальный, то есть логи будут
! --- литься по поводу и без:
!
logging trap debugging
!
```

### Tshoot

```
!
! --- поглядеть всю инфу по логированию, включая активность 
! --- сервера, кол-во отправленных сообщений и пр:
!
show logging
!
! --- есть под винду прога - 3CDaemon, типо лог сервер.
! --- с ней можно оперативно разобраться, шлет циска логи или нет.
! --- укажем наш комп с этой тулзой в качестве лог сервера
! --- и сделаем следующее:
!
debug ip icmp
ping <наш 3CDaemon>
!
! --- циска должна будет разродиться кучей логов, 
! --- а мы - увидеть из у нас на демоне.
!
! --- отключаем дебаг:
!
undebug all
!
```

## SNMP(v3)

SNMPv3 - поддерживает аутентификацию и шифрование в отличие от v2 и v1.

У SNMP есть три типа сообщений:

- GET - забираем инфу с управляемого устройства
- SET - устанавливаем-конфигурируем устройство
- Trap - эту штуку шлет УСТРОЙСТВО серверу, если с ним чото происходит и отправка трапов настроена.

### Configuration

```
!
! --- активируем функционал:
!
snmp-server engineID local 123456789A
!
! --- создаем группу G1 и глянем возможные настройки:
!
snmp-server group G1 v3 ? 
  auth  - аутентификация без шифрования 
  noauth  - без аутентификации и без шифрования
  priv  - и аутентификация и шифрования

!
! --- укажем максимально безопасный вариант + защитим ACL 5
!
snmp-server group G1 v3 prive access 5
!
! --- описываем алгоритмы аутентификации и шифрования.
! --- U1 - это юзер группы G1, он будет создан автоматически:
!
snmp-server user U1 G1 v3 auth sha AuthP@SS priv aes 128 CrypT0P@ss
!
! --- чекнем SNMP-юзера, что он создался и всетакое
!
show snmp user
! 
! --- укажем ip-адрес сервера, версию snmp и локального юзера:
!
snmp-server host 192.168.1.23 traps version 3 auth U1
snmp-server enable traps ? 
!
! --- вывалится список миллионов сервисов, которые можно запихнуть в трапы
! --- и отправить на сервер
!
snmp-server enable traps syslog
!
! --- указываем за каким интерфейсом тусуется наш сервер:
!
control-plane host
management-interface <интерфейс> allow snmp
!
! --- отправка SNMP-трапов в случае загрузки CPU под потолок:
!
process cpu threshold type total rising 80 interval 5 falling 20 interval 5
!
```

### Tshoot

```
!
! --- чекнуть SNMP-юзеров
!
show snmp user
!
! --- ставим себе ManageEngine MibBrowser Free Tool
! --- (это тулза под винду) и ей чекаем работу всего этого добра
! --- на самом деле это самая детальная инфа о железе которая только может быть
! --- один косяк - SNMP вообще нечитаемый, ничего не понять без тулзовин.
! --- также, насколько я помню, этим протоколом можно УПРАВЛЯТЬ железом.
! --- о, ништяк! Wireshark умеет декодить информацию об SNMP даже 
! --- если она была зашифрована если мы укажем в настройках SNMP в WireShark
! --- (правой кнопкой по шифрованному SNMP-пакету) все пароли и пр херню. 
! --- ну, если мы их знаем ._.
!
```

## CPPr

Control Plane Protection

Cisco разделяет роутер на три логических плоскости с целью обеспечения 
безопасности каждой из них и упрощения понимания данного процесса:

- DATA PLANE:
  + тут гуляет пользовательский трафик
  + защита:
     - шифрование, туннелирование и пр. 
     - ACL 
     - все виды межсетевого экранирования и томуподобное

- MANAGEMENT PLANE:
  + тут мы копошимся при настройке устройства, это CLI
  + защита:
     - Role Based CLI Access (AAA)
     - использование SSH, HTTPS, сложных паролей
     - использование ACL на доступ по SSH и HTTPS
     - общий hardening устройства (усложнение доступа в CLI)

- CONTROL PLANE:
  + тут мозг (CPU) маршрутизатора. разделяют субинтерфейсы:
    - host - трафик, предназначенный самому роутеру - ssh, ntp, snmp, eigrp, ospf и пр:
      имеется ввиду, что этот трафик и работа завязана на демонах которые жрут вычислительные мощности. 
    - transit - трафик, который проходит сквозь маршрутизатор но при этом обрабатываться CPU:
      не является пользовательским трафиком, может быть мультикастом протоколов динамической маршрутизации (DRP) не запущенных на роутере, TTL=1 и тд и тп.
    - CEF-exception - не IP-трафик, CDP, ARP и томуподобное - все это будет обрабатываться CPU.

Eсли в пункте MANAGEMENT PLANE мы говорили про ssh в разрезе безопасности ДОСТУПА к MANAGEMENT PLANE, то здесь мы говорим об ssh как о процессе, который будет требовать к себе внимания со стороны CPU, так как будет обрабатываться именно ЭТОЙ ПЛОСКОСТЬЮ, мозгами.

Даже если мы отключим все сервисы, трафик, предназначенный роутеру будет обрабатываться CONTROL PLANE. Это означает, что потенциально можно загрузить устройство мусором, заставив его затрачивать CPU на отбрасывание лишнего.Наша задача - отбрасывать мусор на уровне DATA PLANE и ограничить возможную cкорость служебного трафика на CONTROL PLANE с целью предотвращения возможных перегрузок. Да, сейчас мы будем резать служебный трафик во имя жизни, ибо если нас прижмут гигабитным потоком флуда HTTP-запросов на подключение по CCP при выключенном http-сервере, и у нас не будет ограничений на доступ к Control Plane, с очень большой вероятностью, размазанный CP не даст нам подключиться по ssh с целью что-то починить даже если это будет OOB соединение, ибо CPU перегружен. 

Так что защищаем CONTROL PLANE ^^

### Configuration

Создадим ограничение по скорости трафика для необходимых нам служб, 
таких как SSH и SNMP и для всего остального трафика, идущего в Control Plane роутера.

```
!
! --- создаем ACL, с которым в последствии будем защищать CP:
!
ip access-list extended LIMIT-ACL
!
! ---  говорим что у нас можно snmp и ssh
 permit udp any any eq snmp
 permit tcp any any eq 22
 exit
!
! --- cоздаем class-map и обзываем его:
class-map LIMIT-CLASS
 match access-group name LIMIT-ACL
 exit
!
! --- создаем policy-map и обзываем его:
!
policy-map LIMIT-POLICY
!
! --- говорим что политика юзает class-map LIMIT-CLASS
!
 class LIMIT-CLASS
!
! --- говорим что если трафик по указанному классу class LIMIT-CLASS 
! --- который описывается ACL LIMIT-ACL, который описывает SSH и SNMP,
! --- превысит 64000 бит в секунду - мы будем его дропать:
!
  police rate 64000 bps
!
! --- описываем действия для всего остального, что мы не описали:
!
  class class-default
!
! --- если в классе по умолчанию (весь неописанный нами, 
! --- но попадающий на сам роутер) трафик начнет превышать
! --- указанный порог - он будет дропаться:
!
  police rate 512000 bps
  exit
 exit
!
! --- применяем все это добро к Control Plane:
!
R1(config)# control-plane host
R1(config-cp-host)# service-policy input LIMIT-POLICY
!
! --- забаним трафик, идущий к заблокированным портам-сервисам:
! --- cоздаем class-map и обзываем его:
!
class-map type port-filter ClosedPorts
!
! --- скажем что этот класс особенный - соответствует 
! --- закрытым портам. без ACL, крч:
!
  match closed-ports
!
! --- для него создадим политику
!
policy-map type port-filter ClosedPorts
!
! --- в ней скажем что юзаем class-map (тип трафика) ClosedPorts:
!
 class ClosedPorts
!
! --- описываем действие к исполнению - сброс:
!
 drop
!
! --- применяем все это добро к CP:
!
control-plane host
service-policy type port-filter input ClosedPorts
!
```

### Tshoot

```
!
! --- вся настройка + увидим превышения установленных 
! --- нами порогов и другую статистику:
!
show policy-map control-plane 
!
```

## iACLs a.k.a. Flavors

Тут спич пойдет почти про то же самое, что и в предыдущем пункте. 

Мы занимались CPPr - Control Plane PROTECTION.
А сейчас будет заниматься CoPP - Control Plane POLICYNG. 
Да, цискари любят пизданутое количество схожих на вид акронимов. Они думают что чем больше акронимов знаешь - тем ты умнее. Ога.

CONTROL PLANE - тут мозг (CPU) маршрутизатора. разделяют субинтерфейсы:

- **host** - трафик, предназначенный самому роутеру - ssh, ntp, snmp, eigrp, ospf и пр: 
  + имеется ввиду, что этот трафик и работа завязана на демонах которые жрут вычислительные мощности. 
- **transit** - трафик, который проходит сквозь маршрутизатор но при этом обрабатываться CPU:
  + не является пользовательским трафиком, может быть мультикастом протоколов динамической маршрутизации (DRP) не запущенных на роутере, TTL=1 и тд и тп.
- **CEF-exception** - не IP-трафик, CDP, ARP и томуподобное - все это будет обрабатываться CPU.

Разница между CPPr и CoPP в том, что первый призван защищать только host-интерфейс CP, в то время как у него есть еще два. 
CoPP будет работать с трафиком всех интерфейсов CP, что называется Aggregets.

Чо тут будет:
  
- ACL - ну ясно
- ACE - Access Control Entry - это строчка в ACL
- VACL - все ясно
- rACL  (recieve) - если мы не юзаем CPPr или CoPP - можем создать rACL, который будет замедлять или дропать трафик перед тем, как он будет обрабатываться CPU CP. То есть, тут спич о защите роутером самого себя, трафика, предназначенного его ip-адресу.
- iACL (infrastructure) - предназначен для защиты других устройств внутри сети:
  + Anti-spoofing записи - мы знаем что некие ip-адреса расположены за одним интерфейсом, а трафик с этими ip-адресами в сорцах прет с другого интерфейса (возможная асимметрия) - дропаем его нафиг.
  + Запрет Special Purpose Adresses - то что прописано в RFC 3330 - дропать трафик, если кто-то пытается слать его на 127.0.0.1 (лупбэк), на 192.0.2.Х (резервный), 224.х.х.х - мультикаст или 0.0.0.0 - это вообще хз что. туда же все серые адреса (мы сейчас говорим об интерфейсе, который смотрит наружу) описанные в RFC 1918.
  Также можем резать основываясь на CIDR (блоками).
  Не забываем, что через нас может ходить BGP-трафик, и его, естественно, резать не надо. Необходимо помнить о таких сервисах и прописывать их разрешающими правилами выше запрещающих для функционала iACL.
  ВАЖНО! в iACL мы описываем то, что ЗАПРЕЩАЕМ, остальное разрешено по умолчанию, т.е. в конце permit any any.
- tACL (transit) - в отличие от iACL, тут мы описываем то, что РАЗРЕШАЕМ.
  т.е. в конце мы говорим deny any any. 

### Configuration

У нас есть удаленный офис за интерфейсом 0/3, в котором установлены два роутера с адресами 55.44.33.195 и 55.44.33.200.
Их (роутеры на другом сайте), типо, и защищаем. Или защищаем нас от имперсонализации.
Скажем что с другого интерфейса (0/2), смотрящего также в интернет, пакеты от ip адресов 55.44.33.195 и 55.44.33.200 к нам приходить не могут. 
Также исключим адреса из RFC 3330 и RFC 1918

```
R2(config)# 
!
! --- антиспуфинг:
!
access-list 110 deny ip host 55.44.33.195 any
access-list 110 deny ip host 55.44.33.200 any
!
! --- исключаем специальные адреса из RFC 3330:
!
access-list 110 deny ip host 0.0.0.0 any
access-list 110 deny ip 127.0.0.1 0.255.255.255 any
access-list 110 deny ip 192.0.2.0 0.0.0.255 any
access-list 110 deny ip 224.0.0.0 31.255.255.255 any
!
! --- исключаем серые адреса из RFC 1918:
!
access-list 110 deny ip 10.0.0.0 0.255.255.255 any
access-list 110 deny ip 172.16.0.0 0.15.255.255 any
access-list 110 deny ip  192.168.0.0 0.0.255.255 any
!
! --- дропаем весь трафик на интернет-интерфейсе, 
! --- у которого в источниках указан ip-адрес НАШЕЙ подсети:
!
R2(config)# access-list 110 deny ip 55.44.33.192 0.0.0.15 any
!
! --- разрешаем BGP из интерфейса 0/2 в 0/3, то есть от интернет-интерфейса, 
! --- к другому, за которым наши роутеры на удаленном сайте.
! --- в источниках указан известный нам роутер провайдера,
! --- а в назначении - известный нам наш удаленный роутер, который юзает BGP:
!
access-list 110 permit tcp host 12.23.34.45 host 55.44.33.195 eq bgp
access-list 110 permit tcp host 12.23.34.45 eq bgp host 55.44.33.195
!
! --- теперь режем весь остальной трафик к нашим удаленным
! --- роутерам, который будет идти через нас:
!
access-list 110 deny ip any host 55.44.33.195
access-list 110 deny ip any host 55.44.33.200
!
! --- разрешаем все остальное:
!
access-list 110 permit ip any any
!
! --- устанавливаем его на интерфейс 0/2 и типо счастье.
!
```

## uRPF

uRPF - Unicast Reverse Path Forwarding

(Динамическое предотвращение spoofing-атак на основе таблицы маршрутизации)

Фиксит:
ip-spoofing

Работает посредством:

Reverse Path Forwarding Check - эта херня смотрит в таблицу маршрутизации и ip-адрес отправителя с интерфейсом, с которого он получен.
Если некто с одного интерфейса ломится к нам с ip-адресом источника, который согласно нашей таблицы маршрутизации находится за ДРУГИМ интерфейсом - такие пакеты дропаются, соответственно, не дойдя даже до непосредственного определения того, куда их отправить(в таблицу маршрутизации роутер не полезет с целью обычного форвардинга перед тем как провести RPF Check). Да, стоит сразу упомянуть о том, что если асимметрия для нашей сети - это необходимая фича, то uRPF ее порежет и фича отвалится, если мы настроим не тот режим:

uRPF режимы:

- Strict - описанное выше - это как раз тот случай. Дропаем, если ip-адрес отправителя пакета у нас за другим интерфейсом.
- Loose - для того чтобы юзать при необходимости оставить асимметрию. Тут работа не с интерфейсами, а с маршрутами. Мы смотрим, есть ли ip-адрес отправителя во всей нашей таблице маршрутизации. если есть - все ок. Если у нас 5 интерфейсов с одинаковой стоимостью до какой-то подсети (ну, ассимметрия) - тогда все будет ок, пакет будет пропущен.

Самое главное - мы можем использовать разные режимы на разных интерфейсах! Там где асимметрия нужна - выбираем Loose, где нет - Strict.

Опции:

- Allow self ping - спецом для любителей пинговать самих себя сделана такая вот опция. позволит роутеру пинговать свои интерфейсы.
- Allow default route - по умолчанию uRPF не учитывает Default GW. Влючаем эту фичу - начнет учитывать.
- ACL - В ACL можно прописать разрешения для трафика, который без явного разрешения него будет резаться uRPF. То есть, сможем добавить исключения. Этот ACL будет завязан именно на технологии uRPF. Также, можно использовать его в разрезе [deny ip any any log-input] - то есть смотреть, срабатывания uRPF-технологии. Еще раз: deny any any не будет резать весь трафик, ACL будет рассматривать трафик только в рамках обнаружения пакетов, попадающих под дефолтный дроп uRPF. ALC тут как последняя надежда для трафика, который собираются дропнуть, правило [deny ip any any log-input] не вносит никаких изменений, лишь дает логирование срабатываний.

### Configuration

Схема сети:

```
INTERNAL SUBNET         BRAS            INTERNET
[PC-1,PC-2] ----------> gig 2/0-[R1]-gig 1/0 <-----------[cloud-router-R2]
    |                     |            |                      |
10.1.0.25-26/24   10.123.0.1/24     10.123.0.4/24       тут мы колдуем с 
                              ip-адресом, типо 
                              это атакующий
```
Конфигурация атакующего:

Для того, чтобы заDoSить один из хостов внутренней подсети, мы создадим на R2 лупбэк-интерфейс с адресом одного из хостов этой подсети и начнем пинг с указанием этого лупбека в качестве выходного интерфейса:
```
cloud-router-R2(config)#
!
! --- эта команда создает интерфейс и закидывает
! --- нас в режим его конифгурирования:
!
int loop 10
!
! --- пилим ip-адрес c маской 255.255.255.255, которая для
! --- хоста-получателя будет означать что отправитель находится
! --- в той же подсети - directly connected. то есть, хосты внутри
! --- одной подсети, между собой общаются с масками /32:
!
cloud-router-R2(config-if)# ip address 10.1.0.26 255.255.255.255
!
! --- поднимать ничо не надо, лупбек не может быть в дауне, сваливаем
!
cloud-router-R2(config-if)# end
!
! --- атакуем! "timeout 0" означает что слать пакеты будем 
! --- без задержки и ожидания ответа
! --- с такой скоростью, с которой нам позволит это делать среда передачи:
!
cloud-router-R2# ping 10.1.0.25 repeat 999999999999 source loop 10 timeout 0
```
Теперь конфигурируем uRPF на BRAS - R1, с целью пресечения подобных телодвижений:
```
R1(config)# 
!
! --- создаем ACL для логирования срабатываний uRPF:
!
R1(config)# access-list 123 deny ip any any
!
! --- активируем uRPF:
!
int g1/0
 ip verify unicast source reachable-via ?
  any - проверять доступность источника пакета для любого интерфейса - это режим Loose
  rx - только для интерфейса, с которого он получен - это режим Strict
!
! --- самое главное - мы можем использовать разные 
! --- режимы на разных интерфейсах. тут выбираем Strict:
!
 ip verify unicast source reachable-via rx ?
  <1-199>       IP access list (standard or extended)
  <1300-2699>     IP expanded access list (standard or extended)
  allow-default     добавляем Default GW к рассмотрению (по умолчанию не рассматривается)
  allow-self-ping   разрешаем роутеру пингвать самого себя
!
! --- мы вибираем две опции:
!
 ip verify unicast source reachable-via rx allow-default 123
 exit
!
! --- говорим почаще обновлять саммари, проще говоря, при каждом
! --- срабатывании по умолчанию просто срабатывания аггрегируются, 
! --- иначе логов сыпалась бы тьма:
!
ip access-list log-update threshold 1
!
! --- ну все, нам начинают валиться логи что все плохо. 
! --- то есть, что всё хорошо - мы же уже режем этот трафик)
!
```

### Tshoot

```
!
! --- смотрим дропы, если не настроили логирование ACL'м:
!
show ip int g1/0
!
! --- тут же смотрим настройку всего uRPF, его ACL, 
! --- режим и дополнительные функции
!
```

## NetFlow

NetFlow - это архитектура, технология, и концепция, введенная циско.

Зачем мы юзаем нетфлоу? 

- чтобы у нас была статистика по трафику:
  + проценты по протоколам
  + топ талкеры
  + средняя загрузка по времени и пр
- чтобы у нас была инфа о том, что происходит в сети. мониторинг в целом.

Отсебятина:
нетфлоу не панацея, нужны полноценные мониторилки трафика, ибо нетфлоу покажет нам только:

- ip источника/получателя трафика
- время начала/конца сессии
- протокол по которому идет трафик
- количество переданных байт

Компоненты:

- Монитор - интерфейс/интерфейсы роутера или устройства, поддерживающего NetFlow. будет генерить NetFlow на основе трафика, проходящего сквозь него. также, монитор будет кэшить NetFlow для последующей передачи
- Exporter - штука, которая будет выдергивать кэш NetFlow и отдавать его коллектору. чаще всего, это будет сам роутер -.-
- Collector - он же анализатор, на него будет стекаться NetFlow, и он будет выдавать статистику. Примеры софтин:
  + plixer
  + Scrutinizer
  + PRTG

Optional Sampler - можем сказать роутеру, что хотим обрабатывать для статистики только 1/10 трафика, то есть, исключаем часть трафика из мониторинга для разгрузки процессора, если у нас нагруженный канал связи

### Configuration

В общем, на роутере настраивается Monitor и Exporter.

- Монитор:
  + какой интерфейс? 
  + in или out или оба направления трафика мониторить?
    тут важно ничо не напутать, иначе будем собирать одну и ту же информацию с двух интерфейсов роутера -.- короче, если у роутера два интерфейса, не надо фигарить один на входящий трафик а другой на исходящий))))
  + весь ли трафик? (в процентах, по дефолту будет 100)
  + какой тип трафика - ipv4 или v6?
  + ху из экспортер?
- Экспортер:
  + ip-адрес коллектора
  + UDP порт коллектора (9996 + еще пара по умолчанию)
  + версия NetFlow
  + откуда будем слать NetFlow (с какого интерфейса)
  + опциональные плюхи
- Сэмплер:
  + указываем какую часть от трафика стоит обрабатывать netflow

Не забыть разрешить все это добро на межсетевых экранах, если есть на пути NetFlow.
```
!
! --- создаем экспортер:
!
flow exporter EXPORT-1
 destination 192.168.1.23
 transport udp 9996
 export-protocol netflow-v9
 source gig 1/0
 exit
!
! --- чекнем его настройки:
!
show flow exporter EXPORT-1
!
! --- создаем монитор:
!
flow monitor MONITOR-1
 record netflow ?
  ipv4
  ipv6
!
! --- тут мы активируем набор опций для мониторинга. 
! --- если хотим все - можем включить все))
!
 record netflow ipv4 ?
  as
  as-tos
  bgp-nexthop-tos
  destination-prefix
  destination-prefix-tos
  original-input        - стандартный набор netflow информации
  original-output
  prefix
  prefix-ports
  prefix-ports-tos
  source-prefix
  source-prefix-tos

 record netflow ipv4 original-input
 exporter EXPORT-1
 exit
!
! --- чекнем его настройки:
!
show flow monitor name MONITOR-1
!
! --- вешаем монитор на интерфейс и говорим что мониторим входящий трафик:
!
int g1/0
 ip flow monitor MONITOR-1 input
 end
!
! --- создадим сэмплера для смягчения влияния netlow на CPU
!
sampler OUR-SAMPLER
 description our custom sampler for NetFlow
 mode deterministic 1 out-of 10
 exit
!
! --- проверяем настройки сэмплера:
!
show sampler OUR_SAMPLER
!
! --- зафигарим его на интерфейс:
!
int g1/0
 ip flow monitor MONITOR-1 sampler OUR-SAMPLER input
!
! --- короче, тут циска может ругнуться, отменяем первоначальные настройки
! --- no ip flow monitor MONITOR-1 input и колбасим команду повторно
! --- ip flow monitor MONITOR-1 sampler OUR-SAMPLER input
!
! --- все должно заработать
!
 exit
!
! --- проверить настройку всего этого добра:
!
show flow int g1/0
!
```

### Tshoot

```
!
! --- проверка всех настроек экспортера:
!
show flow exporter <имя экспортера> 
!
! --- смотрим статистику экспортера:
!
show flow exporter <имя экспортера> statistics
!
! --- проверка всех настроек монитора:
!
show flow monitor name <имя монитора>
!
! --- посмотреть инфу netflow из кэша монитора:
!
show flow monitor name <имя монитора> cache ?
  aggregate
  filter
  format ?
    csv
    record
    table
  sort ?
    counters
    flow
    highest
    interface
    ipv4
    lowest
    routing
    timestamp
    transport
!
! --- также, кэш периодически отправляется на экспортер,
! --- и затирается после отправки. не забыть разрешить все это 
! --- добро на межсетевых экранах. проверяем настройки сэмплера:
!
show sampler <имя сэмплера>
!
! --- проверить настройку потока NetFlow на интерфейсе:
!
R1(config)# do show flow int g1/0
!
```

## ZBF on IOS Router

Сначала мы делим интерфейсы роутера (нашу сеть) на зоны - IN, OUT, DMZ.
Определяем политики доступа между ними. Затем:
```
классифицируем трафик, который хотим инспектировать   - Class Map (class-map)
создаем политику для описанного трафика               - Policy Map  (policy-map)
выбираем действие, применяемое к нему                 - Inspect:
                              + pass
                              + drop
                              + log 
```
Inspect - **разрешает** прохождение трафика, при этом **контролирует состояние сессий** типо как **Stateful Packet Inspection**. А вообще много кто этого не знает, и подразумевает под "инспектированием трафика" некую, блять, магию, типо там крутая циска щас вытащит всю малварь.

То есть тут все на сессиях и ACL-ях, ничего особенного и удивительного. 
```

            /   ZBF:
           /    - Zones
L3-L4 INSPECTION    /   - Inspect Class Maps
            \   - Inspect Policy Maps
           \    - Zone-Pairs
            \   - Inspect Service Policy

А вот тут уже анализ контента:
             /    Application Layer Options:      
L4-L7 INSPECTION  |   - APP inspect Class Maps
             \    - APP inspect Policy Maps
```

В общем, у циски все очень плохо с анализом на уровне приложений)
Все что они могут предложить - отслеживание работы ряда приложений на предмет соответствия RFC (protocol-violation) и поиск в потоке данных по регулярным выражениям.

### Configuration L3-L4 Inspection

```
!
! --- создаем зоны
!
zone security IN
 exit
zone security OUT
 exit
zone security DMZ
 exit
!
! --- накидываем на интерфейсы
!
int g1/0
 zone-member security OUT
 exit
int g2/0
 zone-member security IN
 exit
int g3/0
 zone-member security DMZ
 exit
!
! --- проверяем настройку зон:
!
show zone security
!
! --- классификация и инспектирование трафика
!
ip access-list extended IN-OUT-ACL
 permit ip any any
 exit
class-map type inspect match-all IN-OUT-CLASS
 match access-group name IN-OUT-ACL
 exit
!
! --- приверим настройку class map:
!
class-map type inspect
policy-map type inspect IN-OUT-POLICY
 class type inspect IN-OUT-CLASS
  inspect
  exit
 exit
!
! --- приверим настройку policy map:
!
show policy-map type inspect
!
! --- создаем пары зон:
!
zone-pair security IN-OUT-ZP source IN dest OUT
 service-policy type inspect IN-OUT-POLICY
 exit
!
! --- проверить настройку пары зон:
!
show zone-pair security
!
! --- МУТИМ ИНСПЕКТИРОВАНИЕ WEB-ТРАФИКА:
!
class-map type inspect match-any WEB-CLASS
 match protocol http
 exit
policy-map type inspect WEB-POLICY
 class type inspect WEB-CLASS
  inspect
  exit
 exit
zone-pair security OUT-DMZ-ZP source OUT dest DMZ
 service-policy type inspect WEB-POLICY
 exit
!
```

### Configuration L4-L7 Inspection

```
!
! --- создаем регулярку под .com
!
parametr-map type regex COM
 pattern .+\.[Cc][Oo][Mm]
 exit
!
! --- описываем трафик http содержащий .com как класс:
!
class-map type inspect http match-any BAD-HTTP-CLASS
 match request uri regex COM
!
! --- также, активируем сервис проверки работы по HTTP на соответствие с RFC:
!
 match req-resp protocol-violation
 exit
!
! --- создаем политику для этого класса:
!
policy-map type inspect http BAD-HTTP-POLICY
 class type inspect http BAD-HTTP-CLASS
!
! --- говоримч что хотим логи на срабатывания
!
  log
!
! --- и сброс сессий на уровне TCP (RST-пакеты):
!
  reset
  exit
 exit
!
```

### Configuration L3-L7 Inspection

```
!
policy-map type inspect WEB-POLICY
 class type inspect WEB-CLASS
  inspect
!
! --- вот самый сочный момент - мы фигарим политику в другую политику
!
  service-policy http BAD-HTTP-POLICY
  exit
 exit
!
```

### Tshoot

```
!
! --- настройка зон:
!
show zone security
!
! --- настройка class map:
!
class-map type inspect
!
! --- настройка policy map:
!
show policy-map type inspect
!
! --- настройка пары зон:
!
show zone-pair security
!
! --- статистика срабатываний для пар:
!
show policy-map type inspect zone-pair ?
  sessions - больше инфы
!
! --- посмотреть карту портов:
!
show ip port-map
!
```

## AAA & Securing Management Plane

Authentication, Authorization, Accounting - технологии для контроля и аудита доступа к MANAGEMENT PLANE (MP).

Используется для Role-Based Access Control (RBAC) и Centralized Management.

> Мельком пробежавшись по видео, могу с уверенностью сказать что касательно ААА в курсе CCNP-S ничего такого, чего не было в CCNA-S. Но листинг приведу, лишним не будет. 

- Установка:
  + Включить AAA
  + Установить Method List для AAA
  + Включить Method List в AAA

- Методы защиты MP:
  + Сильные пароли
  + AAA
  + RBAC
  + SSH/HTTPS
  + Logging (ну, ведение журнала):
    - для этого нужен NTP
  + Защита системных файлов (конифг)
  + OOB-канал управления как данность

### AAA

- Authentication:
  + Может быть локальная база (на роутере)
  + Есть сервер авторизации под винду - Cisco Secure ACS, типо RADIUS
  + Также есть и самостоятельная железка - Identity Services Engine (ISE)
  + Но, как ты понимаешь, мы юзаем линукса и опенсорс)
  + В общем и целом для аваторизации юает RADIUS и TACACS+
- Authorization:
  + Тут мы говорим о привилегиях пользователей. они указаны там же, где и сами пользовательские учетки - локально, на RADIUS или TACACS+ серверах.
  + Тут же речь идет об удаленном доступе по VPN - также можно юзать для этого AAA
- Accounting:
  + шта?

#### Configuration

Синтаксис настройки технологии:
```
aaa type {default | list-name} method-1 [method-2 method-3 method-4]

type ?        - выбираем технологию
  authentication
  authorization
  accounting

list-name       - имя политики AAA. его мы будем применять к интерфейсу или vty
default       - это если мы не укажем имя, его не будет -.-
method-# ?      - метод реализации технологии
   local      - локальная база
   local-case   - будет чувствительна к регистру. удивительно, что в local не чувствительна -.-
   enable     - юзать пароль на EXEC в качестве пароля. можно указать как последний метод, типо, если остальные не прошли
   krb5       - будет юзать Kerberos 5 для аутентификации (если в type выберем authentication)
   krb5-telnet  - вообще какая-то хитрожопость. 
   line       - line password. я хз что это
   none       - не юзать аутентификацию о.О
   group tacats   - сервер или сервера TACACS+ для аутентификации и авторизации.
   group radius   - сервер или сервера RADIUS для аутентификации и авторизации.
!
! --- врубаем AAA, если еще не включена
!
aaa new-model
!
! --- определяем пару серваков tacacs для AAA
!
tacacs-server host 50.50.4.101
tacacs-server key ToUgHPaSsW0rD-1# 7
!
! --- указываем метод аутентификации по умолчанию - локальный
!
aaa authentication login default local enable
!
! --- теперь укажем что будем юзать лист. циска будет искать
! --- юзеров в раннинг-конфиге мол, указано же локал? указано.
! --- а enable в конце указывает что если пользователя нет
! --- в раннинг-конфиге (lacal) или на сервере (tacacs), то пустим 
! --- его по enable secret паролю (пароль который пропускает нас в PEXEC), 
! --- но для этого, как ты понимаешь, это пароль должен быть настроен
!
aaa authentication login MY-LIST-1 group tacacs local enable
!
! создаем группу, описывающую доступные команды и откуда их брать:
!
aaa authorization exec M-EXEC group tacacs+ local
!
! --- тут мы говорим что необходимо проверять доступные
! --- для пользователя команд в режиме user mode
!
! --- (privileged level 1) на сервере такакс или в локальной базе
!
aaa authorization commands 1 TAC1 group tacacs+ local
!
! --- тут мы говорим что необходимо проверять доступные 
! --- для пользователя команд в режиме privileged exec mode (PEXEC)
! --- на сервере такакс или в локальной базе короче, там, где мы 
! --- создавали этого пользователя, ибо его права описываются 
! --- в команде создания его учетки
!
aaa authorization commands 15 TAC15 group tacacs+ local
!
! --- также, надо создать хотя бы одного ЛОКАЛЬНОГО пользователя-админа 
! --- на случай падения AAA-сервера:
!
username admin privilege 15 secret 4Je7*1swEsf
!
! --- создаем группу аккаунтинга и описываем как он будет происходить:
!
aaa accounting exec M-ACCT-EXEC start-stop group tacacs+ 
!
! --- тут мы говорим что будем запоминать все команды, введенные кем-либо в 
! --- user mode и PEXEC, и отправлять их на tacacs-сервер
!
aaa accounting commands 1 TAC-act1 start-stop group tacacs+ 
aaa accounting commands 15 TAC-act15 start-stop group tacacs+
!
! --- а тут мы вешаем весь функционал AAA на виртуальные линии 0-4:
!
line vty 0 4
 login authentication MY-LIST-1
 authorization exec M-EXEC
 authorization commands 1 TAC1
 authorization commands 15 TAC15
 accounting exec M-ACCT-EXEC
 accounting commands 1 TAC-act1
 accounting commands 15 TAC-act15
!
```

#### Tshoot

```
!
! --- чекаем функционал. нам ответят есть такая пара логин-пароль или нет
!
test aaa group tacacs+ ПОЛЬЗОВАТЕЛЬ ПАРОЛЬ legacy
!
```

### Role-Based Access Control (RBAC)

Права пользователей разделяются на 15 уровней. 15 - максимальный.
Можно создать множество пользователей, объединять их в группы, и давать им различные права к управлению устройством на основе их группы и учетной записи в принципе. один момент - команды которые относятся к конкретному уровню привилегий придется определять руками... 

Это удобно, если необходимо предоставить кому-то доступ на просмотр конфигурации железа, логов и пр, не давая при этом возможности что-то изменить. Именно такую учетку мне выделили бы в МОЭСКе для доступа на их ядро, если бы тамашние админы это понимали и умели. Но, как видим, проще сказать "ты это, не делай там тока ничо..."

Важная херня - есть дофига дефолтных ролей, которые можно раздавать. также, можно установить время действия учетки - по его истечению, будет уделаена.

#### Configuration

```
!
! --- говорим что PEXEC  - это 8й уровень привилегий.
!
R2(config)# privilege exec level 8 configure terminal
R2(config)# enable secret level 8 0 NewPa5s123&
R2(config)# end
R2# 
R2# disable
R2> show privilege
Current privilege level is 1
R2> enable ?
  <0-15>  Enable level
  view    Set into the existing view
  <cr>

R2> enable 8
!
! ---тут надо буде ввести пароль. получается что-то типо chroot, 
! --- это нифига не полноценный уровень управления. даже если мы после 
! --- enable 8 зайдем в conf t - мы получим не полный функционал.
!
! --- cоздаем новый режим просмотра:
!
enable secret aBc!2# &iU 
aaa new-model 
end
enable view
Password: вводим aBc!2# &iU
!
! --- ебать 0_о циска назорвет нас рутом
!
%PARSER-6-VIEW_SWITCH: successfully set to view 'root'. <- это типо лог.
!
conf t
!
! --- создаем новый режим просмотра:
!
parser view New_VIEW
!
! --- создаем пароль для этого режима
!
 secret New_VIEW_PASSWORD
!
! --- перечисляем команды, которые может исполнять 
! --- пользователь в этом режиме просмотра:
!
 commands exec include ping
 commands exec include all show
!
! --- это вообще-то можно и не вводить, если нам
! --- нужен просто просмотр:
!
 commands exec include configure
!
! --- мы говорим что позволяем настраивать только ACL
! 
commands configure include access-list
exit
exit
!
! --- выходим из EXEC в user mode
!
disable
!
! --- заходим в новосозданный режим:
!
R2>enable view New_VIEW
Password: - нутыпонял
!
! --- посмотрит куда это мы попали:
!
R2# show parser view
Current view is 'New_VIEW'
!
! --- надежда умирает последней, попробуем чонить настроить:
!
R2# configure terminal
!
! --- посмотрим что нам доступно:
!
R2(config)# ?
Configure commands:
  access-list   Add an access list entry
  do        To run exec commands in config mode 
  exit      Exit from configure mode
```
В общем, в этом режиме максимум что можно сделать - насоздавать ACL-ей досмерти, перегрузив память. 
Применить их к интерфейсу, естественно, не получится. 
Можно не давать доступ к conf t в принципе, тут рассматривается такой пример с целью дать понимание,
что можно создать не только вьюера, но и, блин, создателя поканеработающих ACL-ей.
```
!
! --- а вот тут мы создаем пользователя который только 
! --- с этим режимом и будет жить.
! --- на месте уровня привилегий описываем его режим и радуемся жизни.
!
username Lois view New_VIEW secret cisco123
!
```

### Securing the Cisco IOS Image and Configuration Files

```
Secure the IOS image
!
! --- запрет на удаление IOS из NVRAM
!
secure boot-image
!
! --- cохраним стартап конфиг в шифрованный архив на флеше:
!
secure boot-config
%IOS_RESILIENCE-5-CONFIG_RESIL_ACTIVE: Successfully secured config archive
 [flash:.runcfg-20111222-230018.ar]
!
! --- чекнем эти дела:
!
show secure bootset IOS resilience router id FTX1036A13J
!
```

## Best Practices a.k.a. Hardering Network Devices

```
!
! --- установим интерфейс, по которому можно ходить протоколами управления:
!
control-plane host
 management-interface g1/0 allow ssh https snmp
 exit
!
! --- посмотреть активные фичи по защите CP и время их активации:
!
show control-plane host feachures
!
! --- таймауты для сессий управления:
!
line vty 0 15
 exec-timeout 10
 exit
!
! --- отключение AUX-порта:
!
line aux 0
 no exec
 no transport in
 no transport out
 exit
auxdown
!
! --- вешаем баннер:
!
R1(config)# banner ?
  LINE        c текста баннера c, где 'c' - разделитель для текста
  config-save     сообщение при сохранении конфигурации
  exec        при входе в терминал
  incoming      приветствие
  login         при логировании
  motd        создать сообщение ДНЯ. ват?
  prompt-timeout    сообщение при выкидывании по таймауту
  slip-ppp      сообщение для SLIP/PPP - ну, видимо, для авторизации по этим протоколам       
!
! --- сброс неактивных сессий:
!
service tcp-keepalives-in
service tcp-keepalives-out
!
! --- посмотреть все введенные команды (даже ошибочные):
!
show history
!
! --- зафигарить в лог метку времени (ват?)
!
R1(config)# service timestamps log datetime
R1(config)# service timestamps debug datetime
!
! --- запилить автосохранение раннинг-конфига:
!
archive
R1(config-archive)# ?
  default       установить в значение по умолчанию
  exit        выйти из режима конфигурирования архивирования
  log         логировать команды
  maximum       максимальное количество резервных копий
  no          нутыпонял
  path        путь к бекапам
  rollback      откатить параметры
  time-period     задать периодичность автосохранений раннинг-конфига
  write-memory    включить автосохранение после ввода команды write-mem 
            (copy run start), короче, после рукопашного сохранения конфига

 path tftp://192.168.1.23/$h
 time-period 60
 write-memory
 end
!
! --- чекнуть текущие архивы, наиболее свежий и как будет обозван следующий:
!
show archive
The next erchive file will be named tftp://192.168.1.23/R1-1
!
! --- аутентификация протоколов маршрутизации:
!
! --- BGP:
!
router bgp 123
 neighbor 192.168.1.50 remote-as 555
!
! --- максимальный TTL = 253, то есть, 2 хопа. 
! --- это поможет предотвратить атаку, если атакующий находится намного дальше
! --- чем в двух хопах
!
 neighbor 192.168.1.50 ttl-security hops 2
 neighbor 192.168.1.50 password <пароль>
 exit
!
! --- OSPF:
!
router ospf 1
 area 0 authentication message-digest
 exit
int g1/0
 ip ospf message-digest-key 1 md5 <пароль>
 exit
!
! --- HSRP/GLBP:
!
int g1/0 
 standby 1 authen md5 key-string <пароль>
 glbp 1 authen md5 key-string <пароль>
 exit
!
! --- отклюить отправку пакета по маршруту вне таблицы маршрутизации
! --- взятому из опции ip-пакета (icmp-redirect?):
!
no ip source-route
!
! дропать пакеты с ip-опциями
!
ip options drop
!
! --- отключение неиспользуемых сервисов в диалоговом режиме:
!
auto secure
!
```

## ASA 8.4 CLI Configuration & Troubleshooting

Версии:

8.2 -> | ДИКИЕ НОВОВВЕДЕНИЯ | -> 8.3x, 8.4, 8.x -> 9.x <- тут синтаксис пока не меняли.

Новые фичи в -X серии (CX):

- IPS теперь не слот, а встроенный. нужно только лицензировать.
- AVC - Application Visibility Control, короче, можно резать трафик на основе его типа, например, запретить конкретные действия на facebook.com
- WSE - Web Security Essentials - фильтрация по типам сайтов, спискам сайтов. URL-фильтрация.

### Configuration

```
!
int m0/0
 no shutdown
 security-level 100
 ip address 192.168.1.100 255.255.255.0
 exit
int g0/0 
 no sh
 nameif outside
 security-level 0
 ip address 10.123.0.100 255.255.255.0
 exit
int g0/1
 no shutdown
 nameif inside
 security-level 100
 ip address 10.1.0.100 255.255.255.0
 exit 
int g0/2
 no shutdown
 nameif dmz
 security-level 50
 ip address 172.16.5.100 255.255.255.0
 exit 
http server enable
htto 192.168.1.0 255.255.255.0 management
sh ip int bri
!
! --- в конце - AD, административная дистанция:
!
route outside 0.0.0.0 0.0.0.0 10.123.0.1 5
sh route
!
snmp-server location MOSCOW LOLOLO
snmp-server contact Dan
snmp-server group G1 v3 priv
snmp-server user U1 G1 auth sha <пароль> priv aes 123 <пароль>
snmp-server host manahement 192.168.1.23 version 3 U1
snmp cpu threshold rising 80 1
snmp-server enable traps cpu threshold rising
sh snmp user 
sh snmp group
!
sh logging
logging enable
logging host management 192.168.1.23
logging trap 5
logging console 4
logging buffered 6
clear logging buffer
sh logging
sh log
!
! --- отрубаем конкретные лог-сообщения:
!
no log message 111005
!
! --- меняем уровень у конкретных лог-сообщений. с 5го уровня на 6й:
!
logging message 111007 level Informational
!
! --- или у пачки сообщений:
!
logging list Our-Event-List message 101001-101003
logging list Our-Event-List level Informational
smtp-server 192.168.1.23
logging from-address ASA@Here.net
logging recipient-addresss it@ngsec.ru level Informational
!
! --- отправлять только вот эти сообщения:
!
logging mail Our-Event-List
clock timezome MSK 4
clock summertime PDT recurring 2 Sun Mar 2:00 Sun Nov 2:00
ntp server 66.187.233.4 source outside
sh ntp associations
sh ntp status
!
flow-export destination management 192.168.1.23 9996
!
class-map global-class
 match any 
 exit
policy-map global-policy
 class global-class
  flow-export event-type all destination 192.168.1.23
  exit
 exit
!
object network Srv-1
 host 172.16.5.5
 nat static 10.123.0.5 net-to-net
 object network Srv-2
 host 172.16.5.6
 object network Srv-3
 host 172.16.5.7
 exit
!
object-group network DMZ-Servers
 network-object object Srv-1
 network-object object Srv-2
 network-object object Srv-3
!
 object-group service WEB-Services
 service-object tcp destination eq http
 service-object tcp destination eq https
 exit
!
access-list outside_access_in permit object-group WEB-Services any object-group DMZ-Servers
!
access-group outside_access_in in interface outside
!
logging console 7
logging console
!
regex Looking-4-EXE ".+\.[Ee][Xx][Ee]"
class-map type match-any REGEX-CLASS-MAP
 match regex Looking-4-EXE
 exit
class-map type inspect http match-all HTTP-CLASS-MAP
 match request uri regex class REGEX-CLASS-MAP
 exit
policy-map type inspect http HTTP-Policy-Map
 paremeters
  protocol-violation action reset log
  exit
 class HTTP-CLASS-MAP
  reset log
  exit
 exit
!
policy-map global_policy
 class inspection_default
  inspect http HTTP-Policy-Map
  exit
 exit
!
show run policy-map
show service-policy
capture OUR-CAPTURE ?

exec mode commands/options:
  access-list      Capture packets that match access-list
  headers-only     Capture only L2, L3 and L4 headers of packet without data in them
  interface        Capture packets on a specific interface
  circular-buffer  Overwrite buffer from beginning when full, default is non-circular
  trace            Trace the captured packets
  buffer           Configure size of capture buffer, default is 512 KB:
              <1534-33554432>  Size of capture buffer in bytes


  ethernet-type    Capture Ethernet packets of a particular type, default is IP:
            802.1Q     
            <0-65535>  Ethernet type
            arp        
            ip         
            ip6        
            ipx        
            pppoed     
            pppoes     
            rarp       
            vlan   

  match            Capture packets matching five-tuple: 
  <0-255>  Enter protocol number (0 - 255)
            ah       
            eigrp
            esp
            gre
            icmp
            icmp6
            igmp
            igrp
            ip
            ipinip
            ipsec
            nos
            ospf
            pcp
            pim
            pptp
            snp
            tcp
            udp

  packet-length    Configure maximum length to save from each packet, default is 1518 bytes:
  
  <14-9216>  Maximum length to save from each packet in bytes

  real-time   

Display captured packets in real-time. Warning: using this option with a
slow console connection may result in an excessive amount of non-displayed
packets due to performance limitations:
  
  detail  Display more information for each packet
  dump    Display hex dump for each packet
  match   Capture packets matching five-tuple
  trace   Display extended trace information for each packet

  type          Capture packets based on a particular type
  asp-drop      Capture packets dropped with a particular reaso
  isakmp        Capture encrypted and decrypted ISAKMP payload
  raw-data      Capture inbound and outbound packets on one or more interface
  tls-proxy     Capture decrypted inbound and outbound data from TLS Proxy on one or more interfaces
  webvpn        Capture WebVPN transactions for a specified user

!
! --- будем ловить весь трафик предназначенный конкретному хосту
!
capture OUR-CAPTURE match tcp host 10.1.0.25 any eq 80
!
! --- на таком-то интерфейсе:
!
capture OUR-CAPTURE interface inside
show capture OUT-CAPTURE
!
! --- скачать это добро можно через веб-интерфейс ASA
! --- ну, предварительно остановив дамп:
!
no capture OUR-CAPTURE
!
! --- тут мы смотрим дропы трафика (всего, вместе с причинами)
! --- также, поведенческую самодеятельность ASA:
!
ciscoasa(config)# sh asp drop
ciscoasa(config)# capture DROPPED type asp-drop all
!
! --- если введем знак вопроса вывалится лютая портянка. 
! --- кстати, лишней не будет -.-
!
```

#### Причины по которым ASA молча дропает трафик

```
ciscoasa(config)# capture DROPPED type asp-drop ?
exec mode commands/options:
  acl-drop                                    Flow is denied by configured rule
  all                                         All packet drop reasons
  async-lock-queue-limit                      Async lock queue limit exceeded
  bad-crypto                                  Bad crypto return in packet
  bad-ipsec-natt                              Bad IPSEC NATT packet
  bad-ipsec-prot                              IPSEC not AH or ESP
  bad-ipsec-udp                               Bad IPSEC UDP packet
  bad-tcp-cksum                               Bad TCP checksum
  bad-tcp-flags                               Bad TCP flags
  buffer                                      Configure size of capture buffer,default is 512 KB
  channel-closed                              Data path channel closed
  circular-buffer                             Overwrite buffer from beginning when full, default is non-circular
  conn-limit                                  Connection limit reached
  connection-lock                             Unable to obtain connection lock
  cp-event-queue-error                        CP event queue error
  cp-syslog-event-queue-error                 CP syslog event queue error
  ctm-error                                   CTM returned error
  dispatch-block-alloc                        Core local block alloc failure
  dispatch-decode-err                         Diapatch decode error
  dns-guard-id-not-matched                    DNS Guard id not matched
  dns-guard-out-of-app-id                     DNS Guard out of app id
  dst-l2_lookup-fail                          Dst MAC L2 Lookup Failed
  flow-being-freed                            Flow is being freed
  flow-expired                                Expired flow
  fo-standby                                  Dropped by standby unit
  fragment-reassembly-failed                  Fragment reassembly failed
  headers-only                                Capture only L2, L3 and L4 headers of packet without data in them
  host-limit                                  Host limit
  host-move-pkt                               FP host move packet
  ifc-classify                                Virtual firewall classification failed
  inspect-dns-id-not-matched                  DNS Inspect id not matched
  inspect-dns-invalid-domain-label            DNS Inspect invalid domain label
  inspect-dns-invalid-pak                     DNS Inspect invalid packet
  inspect-dns-out-of-app-id                   DNS Inspect out of app id
  inspect-dns-pak-too-long                    DNS Inspect packet too long
  inspect-icmp-error-different-embedded-conn  ICMP Error Inspect different embedded conn
  inspect-icmp-error-no-existing-conn         ICMP Error Inspect no existing  conn
  inspect-icmp-out-of-app-id                  ICMP Inspect out of app id
  inspect-icmp-seq-num-not-matched            ICMP Inspect seq num not matched
  inspect-icmpv6-error-invalid-pak            ICMPv6 Error Inspect invalid packet
  inspect-icmpv6-error-no-existing-conn       ICMPv6 Error Inspect no existing conn
  inspect-rtcp-invalid-length                 Inspect RTCP invalid packet length
  inspect-rtcp-invalid-payload-type           Inspect RTCP invalid payload-type
  inspect-rtcp-invalid-version                Inspect RTCP invalid version
  inspect-rtp-invalid-length                  Inspect RTP invalid packet length
  inspect-rtp-invalid-payload-type            Inspect RTP invalid payload-type
  inspect-rtp-invalid-version                 Inspect RTP invalid version
  inspect-rtp-max-outofseq-paks-probation     Inspect RTP Max out of sequence paks in probation
  inspect-rtp-sequence-num-outofrange         Inspect RTP Sequence number out of range
  inspect-rtp-ssrc-mismatch                   Inspect RTP SSRC mismatch
  inspect-srtp-client-port-not-present        Inspect SRTP Client port not populated
  inspect-srtp-decrypt-failed                 Inspect SRTP decrypt failed
  inspect-srtp-encrypt-failed                 Inspect SRTP encrypt failed
  inspect-srtp-generate-authtag-failed        Inspect SRTP generate authentication tag failed
  inspect-srtp-no-media-session               Inspect SRTP Media session not found
  inspect-srtp-no-output-flow                 Inspect SRTP find output flow failed
  inspect-srtp-no-remote-phone-proxy-ip       Inspect SRTP Remote Phone Proxy IP not found
  inspect-srtp-one-part-no-key                Inspect SRTP keys not available for one party
  inspect-srtp-setup-srtp-failed              Inspect SRTP failed to setup SRTP with CTM
  inspect-srtp-validate-authtag-failed        Inspect SRTP validate authentication tag failed
  intercept-unexpected                        Intercept unexpected packet
  interface-down                              Interface is down
  invalid-app-length                          Invalid app length
  invalid-encap                               Invalid encapsulation
  invalid-ethertype                           Invalid Ethertype
  invalid-ip-header                           Invalid IP header
  invalid-ip-length                           Invalid IP length
  invalid-ip-option                           IP option drop
  invalid-tcp-hdr-length                      Invalid TCP Length
  invalid-udp-length                          Invalid UDP Length
  ips-fail                                    IPS config removed for flow
  ips-fail-close                              IPS card is down
  ips-no-ipv6                                 Executing IPS software does not support IPv6
  ips-request                                 IPS Module requested drop
  ipsec-clearpkt-notun                        IPSEC Clear Pkt w/no tunnel
  ipsec-ipv6                                  IPSEC via IPV6
  ipsec-lock-error                            IPSec locking error
  ipsec-need-sa                               IPSEC SA Not negotiated yet
  ipsec-spoof                                 IPSEC Spoof detected
  ipsec-tun-down                              IPSEC tunnel is down
  ipsecudp-keepalive                          IPSEC/UDP keepalive message
  ipv6-ah-denied                              IPv6 AH EH denied by config
  ipv6-bad-eh                                 IPv6 bad EH
  ipv6-bad-eh-order                           IPv6 EH not in proper order
  ipv6-dest-option-denied                     IPv6 destination option EH denied by config
  ipv6-eh-count-denied                        IPv6 EH count denied by config
  ipv6-eh-inspect-failed                      IPv6 EH could not be inspected
  ipv6-esp-denied                             IPv6 ESP EH denied by config
  ipv6-fragment-denied                        IPv6 fragment EH denied by config
  ipv6-hop-by-hop-denied                      IPv6 hop-by-hop EH denied by config
  ipv6-mobility-denied                        IPv6 mobility EH denied by config
  ipv6-mobility-type-denied                   IPv6 mobility type EH denied by config
  ipv6-routing-address-denied                 IPv6 routing addresses exceed limit and denied by config
  ipv6-routing-type-denied                    IPv6 routing type EH denied by config
  ipv6-sp-security-failed                     IPv6 slowpath security checks failed
  l2_acl                                      FP L2 rule drop
  l2_same-lan-port                            L2 Src/Dst same LAN port
  loopback-buffer-full                        Loopback buffer full
  loopback-count-exceeded                     Loopback count exceeded
  loopback-ifc-not-found                      Loopback output interface not found
  loopback-lock-failed                        Loopback lock failed
  lu-invalid-pkt                              Invalid LU packet
  match                                       Capture packets matching five-tuple
  mp-pf-queue-full                            PF Module queue full
  mp-svc-addr-renew-response                  SVC Module received address renew response data frame
  mp-svc-bad-framing                          SVC Module received badly framed data
  mp-svc-bad-length                           SVC Module received bad data length
  mp-svc-compress-error                       SVC Module compression error
  mp-svc-decompres-error                      SVC Module decompression error
  mp-svc-delete-in-progress                   SVC Module received data while connection was being deleted
  mp-svc-flow-control                         SVC Module is in flow control
  mp-svc-invalid-mac                          SVC Module found invalid L2 data in the frame
  mp-svc-invalid-mac-len                      SVC Module found invalid L2 data length in the frame
  mp-svc-no-channel                           SVC Module does not have a channel for reinjection
  mp-svc-no-fragment                          SVC Module unable to fragment packet
  mp-svc-no-mac                               SVC Module unable to find L2 data for frame
  mp-svc-no-prepend                           SVC Module does not have enough space to insert header
  mp-svc-no-session                           SVC Module does not have a session
  mp-svc-unknown-type                         SVC Module received unknown data frame
  natt-keepalive                              NAT-T keepalive message
  no-adjacency                                No valid adjacency
  no-mcast-entry                              FP no mcast entry
  no-mcast-intrf                              FP no mcast output intrf
  no-route                                    No route to host
  non-ip-pkt-in-routed-mode                   Non-IP packet received in routed mode
  np-socket-closed                            Dropped pending packets in a closed socket
  np-sp-invalid-spi                           Invalid SPI
  packet-length                               Configure maximum length to save from each packet, default is 1518 bytes
  punt-no-mem                                 Punt no memory
  punt-queue-limit                            Punt queue limit exceeded
  punt-rate-limit                             Punt rate limit exceeded
  queue-removed                               Rate-limiter queued packet dropped
  rate-exceeded                               Output QoS rate exceeded
  real-time                                   Display captured packets in real-time. Warning: using this option with a slow console connection may result in an excessive amount of non-displayed packets due to performance limitations.
  rm-conn-limit                               RM connection limit reached
  rm-conn-rate-limit                          RM connection rate limit reached
  rpf-violated                                Reverse-path verify failed
  security-failed                             Early security checks failed
  send-ctm-error                              Send to CTM returned error
  shunned                                     Packet shunned
  sp-security-failed                          Slowpath security checks failed
  ssm-app-fail                                Service module is down
  ssm-app-request                             Service module requested drop
  ssm-asdp-invalid                            Invalid ASDP packet received from SSM card
  ssm-dpp-invalid                             Invalid packet received from SSM card
  tcp-3whs-failed                             TCP failed 3 way handshake
  tcp-ack-syn-diff                            TCP ACK in SYNACK invalid
  tcp-acked                                   TCP DUP and has been ACKed
  tcp-bad-option-list                         TCP option list invalid
  tcp-buffer-full                             TCP Out-of-Order packet buffer full
  tcp-buffer-timeout                          TCP Out-of-Order packet buffer timeout
  tcp-conn-limit                              TCP Connection limit reached
  tcp-data-past-fin                           TCP data send after FIN
  tcp-discarded-ooo                           TCP ACK in 3 way handshake invalid
  tcp-dual-open                               TCP Dual open denied
  tcp-dup-in-queue                            TCP dup of packet in Out-of-Order queue
  tcp-fo-drop                                 TCP replicated flow pak drop
  tcp-global-buffer-full                      TCP global Out-of-Order packet buffer full
  tcp-invalid-ack                             TCP invalid ACK
  tcp-mss-exceeded                            TCP data exceeded MSS
  tcp-not-syn                                 First TCP packet not SYN
  tcp-paws-fail                               TCP packet failed PAWS test
  tcp-reserved-set                            TCP reserved flags set
  tcp-rst-syn-in-win                          TCP RST/SYN in window
  tcp-rstfin-ooo                              TCP RST/FIN out of order
  tcp-seq-past-win                            TCP packet SEQ past window
  tcp-seq-syn-diff                            TCP SEQ in SYN/SYNACK invalid
  tcp-syn-data                                TCP SYN with data
  tcp-syn-ooo                                 TCP SYN on established conn
  tcp-synack-data                             TCP SYNACK with data
  tcp-synack-ooo                              TCP SYNACK on established conn
  tcp_xmit_partial                            TCP retransmission partial
  tcpnorm-rexmit-bad                          TCP bad retransmission
  tcpnorm-win-variation                       TCP unexpected window size variation
  telnet-not-permitted                        Telnet not permitted on least secure interface
  tfw-no-mgmt-ip-config                       No management IP address configured for TFW
  unable-to-add-flow                          Flow hash full
  unable-to-create-flow                       Flow denied due to resource limitation
  unexpected-packet                           Unexpected packet
  unsupport-ipv6-hdr                          Unsupported IPV6 header
  unsupported-ip-version                      Unsupported IP version
  vpn-handle-error                            VPN Handle Error
  vpn-handle-mismatch                         VPN Handle Mismatch Error
  wccp-redirect-no-route                      WCCP redirect packet no route
  wccp-return-no-route                        WCCP returned packet no route

!
! --- посмотреть количество сдампенных байт
! --- кстати, очень важная штука, в случае, если ASA 
! --- что-то роняет непонятно почему!
!
show capture 
!
! --- посмотреть причины дропов
!
show capture DROPPED 
!
! --- отправим дамп на tftp в диалоговом режиме:
!
ciscoasa(config)# copy /pcap capture:DROPPED ?

exec mode commands/options:
  /noconfirm      Do not prompt for confirmation
  /pcap           Raw packet capture dump
  capture:        Copyout capture buffer
  disk0:          Copy from disk0: file system
  flash:          Copy from flash: file system
  ftp:            Copy from ftp: file system
  http:           Copy from http: file system
  https:          Copy from https: file system
  running-config  Copy from current system configuration
  smb:            Copy from smb: file system
  startup-config  Copy from startup configuration
  system:         Copy from system: file system
  tftp:           Copy from tftp: file system

 copy /pcap capture:DROPPED tftp:

Source capture name [DROPPED]?

Address or name remote host []? 192.168.1.22

Destination filename [DROPPED]? DROPPED.pcap
!!!!!!!!
```

та-да, файл отправлен

## Botnet Filtering 

Есть некая база SIO - Security Intelligence Operations, некий центр, который собирает информацию о ботнетах. 
Эта база динамически обновляется. ASA на связи с SIO. У ASA динамически обновляется фильтр от ботнетов, причем, это дело управляется из того самого мистического SIO. Делается это, например, путем обнаружения SIO ip-адреса центра управления, который тут же появляется в списках доступа на запрет коммуникаций с ним во всех ASA, подписанных на SIO, т.е. с активированным функционалом анти-ботнет.

Конфигурация:

+ установить лицензию
+ включить DNS Client / Snooping (снупинг - основываясь на динамической базе данных SIO)
+ активировать динамическое обновление данных
+ включить белые/черные списки:
  - если мы вносим некий ip в белый список, даже если мы получим из ISO автоматическую команду на запрет этого ip, он не будет заблокирован.

## Packet Tracer

Packet Tracer - аналог Traffic Simulation Query в AlgoSec только это фича в ASDM для прохождения трафика внутри самой ASA из одного интерфейса в другой с отслеживанием всего, что будет с ним происходить на входящих-исходящих ACL обоих интерфейсов, NAT, в Route-Forwarding'е (ну, типо, куда марщрутизировать и маршрутизировать ли вообще), а также в VPN и QoS-модулях. 

Короче, процесс прохождения трафика не так прост, поэтому нужны такие инструменты. Детализация всего что происходит сыпется в окно ASDM и доступно в один клик. 

Соответственно, есть консольная версия:

```
ASA# packet-tracer ?

  input  Ingress interface on which to trace packet

ASA# packet-tracer input ?

  Current available interface(s):
  amt      Name of interface GigabitEthernet1/1
  isdprod  Name of interface GigabitEthernet1/2.20
  mgmt     Name of interface GigabitEthernet1/2.101
  outside  Name of interface GigabitEthernet1/3
  socdev   Name of interface GigabitEthernet1/2.10
  tst      Name of interface GigabitEthernet1/2.30

ASA# packet-tracer input outside ?

  icmp   Enter this keyword if the trace packet is ICMP
  rawip  Enter this keyword if the trace packet is RAW IP
  sctp   Enter this keyword if the trace packet is SCTP
  tcp    Enter this keyword if the trace packet is TCP
  udp    Enter this keyword if the trace packet is UDP

ASA# packet-tracer input outside tcp ?

  A.B.C.D          Enter the Source address if ipv4
  X:X:X:X::X       Enter the Source address if ipv6
  fqdn             Enter this keyword if an FQDN is specified as source address
  inline-tag       Enter this keyword if trace packet is embedded with L2 CMD Header
  security-group   Enter this keyword if a security group is specified as source address
  user             Enter this keyword if a user is specified as source address

ASA# packet-tracer input outside tcp 10.1.1.1 ?

  <0-65535>        finger           kshell              pptp
  aol              ftp              ldap                rsh
  bgp              ftp-data         ldaps               rtsp
  chargen          gopher           login               sip
  cifs             h323             lotusnotes          smtp
  citrix-ica       hostname         lpd                 sqlnet
  cmd              http             netbios-ssn         ssh
  ctiqbe           https            nfs                 sunrpc
  daytime          ident            nntp                tacacs
  discard          imap4            pcanywhere-data     talk
  domain           irc              pim-auto-rp         telnet
  echo             kerberos         pop2                uucp
  exec             klogin           pop3                whois
                                                        www
ASA# packet-tracer input outside tcp 10.1.1.1 514 ?

  A.B.C.D         Enter the destination ipv4 address
  fqdn            Enter this keyword if an FQDN is specified as destination address
  security-group  Enter this keyword if a security group is specified as destination address

ASA# packet-tracer input outside tcp 10.1.1.1 514 192.168.0.1 ?

  (порт. те же что выше)
                                                         
ASA# packet-tracer input outside tcp 10.1.1.1 514 192.168.0.1 8080 ?

  detailed     Dump more detailed information
  vlan-id      Specify VLAN id for the flow
  vxlan-inner  Specify inner packet using VLXN encapsulation
  xml          Output in xml format
  <cr>

ASA# packet-tracer input outside tcp 10.1.1.1 514 192.168.0.1 8080 detailed ?

  xml  Output in xml format
  <cr>

ASA# packet-tracer input outside tcp 10.1.1.1 514 192.168.0.1 8080 detailed xml ?

  <cr>
```

## Content Directiry Agent

Эта штука призвана совместить AD (Active Directory) и ASA.
Для этого и нужен Context Directory Agent. То есть, на основе пользовательских данных (групповых политик) из AD мы будем управлять его доступом в сети.

Необходимо настроить контакт AD с CDA, а CDA уже будет отправлять непосредственные указания ASA.

- Пошагово:
  + На AD надо произвести дофига изменений, для этого есть отдельный гайд + даже пропатчить надо -.-
  + Развертывание CDA:
    - CDA рекомендуют ставить на виртуалку CentOS 32-бит, понадобится 30 ГБ памяти
    - CDA будет взаимодействовать с ASA как RADIUS-сервер
    - CDA будет взаимодействовать с AD по WMI
  + Настроить на ASA:
    - Взаимодействие с CDA по RADIUS с целью получения ip-mapping information
    - Взаимодействие по LDAP с AD с целью выяснения какие пользователи в каких группах
    - IP Based Groups, and Rules

короче. если на CDA можно каким-то хером откуда-то слать логи о подключении новых пользователей, которые будут содержать в себе ip-адреса и пр. подробности - можно будет обойтись и без WMI и коннекта CDA с AD.

Настройка показана через web-интерфейс и ASDM, заучивать такое нет смысла.

## Management Tools

Вот, это интересно. Тут то, чему противопоставляет себя AlgoSec - системы для мониторинга и управления политиками всей сети. Ну, только если это Cisco.

- Tools:
  + Adaptive Security Device Manager (ASDM)
  + Cisco Configuration Professional (CCP)
  + IPS Device Manager (IDM)
  + IPS Manager Express (IME)
  + Cisco Security Manager (CSM):
    - ASA 5500-5500-X (cx не поддерживается)
    - IOS
    - Switches
    - IPS 
    - пушим политики одновременно на несколько девайсов
  + Cisco Prime Infrastructure & Cisco Prime Security Manager (PRSM):
    - все тоже самое что и в CSM плюс:
      + ASA CX
      + Secure Mobility
      + IPSec RA (AnyConnect)
      + красивые отчеты касательно ASA CX (еще ее называют NGFW, lol)
  + Access Control Server (ACS):
    - AAA-сервер
    - 802.1X 
    - TACACS+
    - RADIUS
  + Identity Services Engine (ISE) - эта штука типо призвана заменить ACS:
    - ТОЛЬКО RADIUS
    - предлагают юзать совместно с ACS

## IPv6 Security

### Attack

+ Alive6:
  - Find all local IPv6 systems, checks Find all local IPv6 systems, checks aliveness of re aliveness of remote systems mote systems
+ PARSITE6:
  - ICMP Neighbor ICMP Neighbor Spoofer Spoofer for Man for Man-In-The-Middle attacks Middle attacks
+ REDIR6:
  - Redirect traffic to your system on a LAN
+ FAKE_ROUTER6:
  - Fake a router, implant routes, become the default router
+ DETECT-NEW-IPv6:
  - Detect new IPv6 systems on the LAN, automatically launch a script
+ DOS-NEW-IPv6:
  - Denial any new IPv6 system access on the LAN (DAD Spoofing)
+ SMURF6:
  - Local Smurf Tool (attack you own LAN)
+ RSMURF6:
  - Remote Smurf Tool (attack a remote LAN)
+ TOOBIG6:
  - Reduce the MTU of a target
+ FAKE_MLD6:
  - Play around with Multicast Listener Discovery Reports
+ FAKE_MIPv6:
  - Reroute mobile IPv6 nodes where you want them if no IPSEC is required
+ SENDPEES6:
  - Neighbor solicitations with lots of CGAs
+ Protocol Implementation:
  - Various tests, more to come
 
Новый заголовок лишает нас следующего:
  - IP AD field - никаких проверок аптайма, как раньше
  - IP Record Route Options - нет трейсроутов
Никаких бродкастов
Мальтикасты не гуляют в удаленные сети

Короче, можно представиться роутером, раздать ipv6 адреса и осуществить MITM

[Больше инфы](https://www.thc.org/papers/vh_thc-ipv6_attack.pdf)

### Defense

+ First Hop Security:
  - ND Snooping  (типо ARP инспект)
  - ND Inspect (типо ARP инспект)
  - RA Guard (почти то же самое что DHCP Snooping, настраивается на свичах)
  - Per Port Adress Limits - короче, это как Port Security
  - Secure ND (SeND) - криптованнй NDP

NDP - Network Discovery Protocol (замена ARP), поиск соседей.
RA - Router Advertisement
RS - Router Solicitation
DAD - Duplicate Address Detection
SLAAC - ???

DHCPv6 Guard
Destination Guard - блокирует трафик на основе ND Snooping
IPv6 ACLs и pACLs - короче, тут как в IPv4

IPv6 ACLs - всегда разрешать надо две штуки:
  - NS - Neighbor Solicitation
  - NA - Neighbor Advertisement
Да, по умолчанию у IPv6 ACL нет правила deny any any в конце.

```
!
ipv6 access-list FILTERv6
 permit icmp any any nd-na
 permit icmp any any nd-ns
! 
```
ну и дальше по пунктам...

[Подробная статья на хабре](http://habrahabr.ru/post/64592/)

## Useful Staff

```
!
! --- отдает весь конфиг + все талицы по загрузке интерфейсов, 
! --- текущего состояния системы и пр пр пр. Все что нужно для аудита
!
show tech-support
!
show run | section crypto
!
show run all | section crypto
!
! --- посчитать shasum на винде:
!
fciv -sha1 c:\windows\system32\mstsc.exe
!
```
