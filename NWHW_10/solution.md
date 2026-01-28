# Лабораторная работа. Настройка протокола OSPFv2 для одной области
## Часть 1. Создание сети и настройка основных параметров устройства
собираю топологию

![](01.png)

таблица адресов

| Устройство | Интерфейс | IP-адрес    | Маска подсети |
| ---------- | --------- | ----------- | ------------- |
| R1         | G0/0/1    | 10.53.0.1   | 255.255.255.0 |
| R1         | Loopback1 | 172.16.1.1  | 255.255.255.0 |
| R2         | G0/0/1    | 10.53.0.2   | 255.255.255.0 |
| R2         | Loopback1 | 192.168.1.1 | 255.255.255.0 |
Базовые настройки
```
enable
config terminal
	hostname R1
	no ip domain-lookup
	enable secret class
	line console 0
		password cisco
		login
		exit
	line vty 0 15
		password cisco
		login
		exit
	service password-encryption
	banner motd x hello, be nice x
exit
copy ru st

clock set 12:21:00 jan 17 2026
```
## Часть 2. Настройка и проверка базовой работы протокола OSPFv2 для одной области
Настраиваю интерфейсы интерфейсов
на R1:
```
interface gi 0/0/1
	ip address 10.53.0.1 255.255.255.0
	no shut
exit

interface loopback 1
	ip address 172.16.1.1 255.255.255.0
	no shut
exit
```
на R2:
```
interface gi 0/0/1
	ip address 10.53.0.2 255.255.255.0
	no shut
exit

interface loopback 1
	ip address 192.168.1.1 255.255.255.0
	no shut
exit
```
Настраиваю с OSPF
На R1
```
router ospf 56
	router-id 1.1.1.1
exit

interface gi 0/0/1
	ip ospf 56 area 0
exit

```
На R2 
```
router ospf 56
	router-id 2.2.2.2
exit

interface gi 0/0/1
	ip ospf 56 area 0
exit

interface loopback 1
	ip ospf 56 area 0
exit
```
Проверяю связности
`show ip route` на R1 появилась сеть с R2

![](02.png)

Отвечаю на каверзные вопросы
проверяю параметры `ip ospf interface` и `ip ospf neighbor`

![](03.png)

![](04.png)

Значит - R1 - BDR, R2 - DR, что согласуется с правилами выбора - у кого айдишник больше, тот и DR.

пинг до сети работает:

![](05.png)

## Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области
### Шаг1. Оптимизации
- меняю приоритет на R1
```
interface gi 0/0/1
	ip ospf priority 50
end

clear ip ospf process
```
теперь роли поменялись

![](06.png)

- настраиваю таймеры 
```
interface gi 0/0/1
	ip ospf hello-interval 30
exit
```
в процессе соседство разорвалось, изза несогласованности параметров, потом восстановилось

![](07.png)

на R1:
```
ip route 0.0.0.0 0.0.0.0 loopback 1
*%Default route without gateway, if not a point-to-point interface, may impact performance*

router ospf 56
	default-information originate
exit
```

спустя несколько мгновений в R2 обновляется таблица маршрутизации

![](08.png)

Теперь убираю loopback из ospf на R2
```
interface lo 1
	no ip ospf 56 area 0
exit
```

![](09.png)

Настраиваю R2:lo 1 как точка-точка:
`ip ospf network point-to-point`

Устанавливаю базовую пропускную способность и обновляю соседство на обоих маршрутизаторах:
```
interface gi 0/0/1
	bandwidth 10000000000
end

clear ip ospf process 
```
### Шаг 2. проверяю
**a.** Выполните команду **show ip ospf interface g0/0/1** на R1 и R2
Предварительно поправлю dead-interval
```
interface gi 0/0/1
	ip ospf dead-interval 120
exit
```
- priority 50,
- временной интервал Hello 30
- временной интервал Dead 120
- тип сети по умолчанию — Broadcast
Всё так.

![[10.png]]

**b.** На R1 выполните команду **show ip route ospf**, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации. Обратите внимание на разницу в метрике между этим выходным и предыдущим выходным. Также обратите внимание, что маска теперь составляет 24 бита, в отличие от 32 битов, ранее объявленных.
не пингуется, не присутствует.
``` R1
router ospf 56
	network 172.16.1.0 255.255.255.0 area 0
exit
```
``` R2
router ospf 56
	network 192.168.1.0 255.255.255.0 area 0
exit

```
о, появились сети и пинги пошли.

![[11.png]]

3. Введите команду **show ip route ospf** на маршрутизаторе R2. Единственная информация о маршруте OSPF должна быть распространяемый по умолчанию маршрут R1.
есть такое.
    
4. Запустите Ping до адреса интерфейса R1 Loopback 1 из R2. Выполнение команды ping должно быть успешным.
есть такое.
#### Вопрос:
Почему стоимость OSPF для маршрута по умолчанию отличается от стоимости OSPF в R1 для сети 192.168.1.0/24?
А где эта стоимость? Вообще говоря у маршрута по умолчанию маска подсети меньше, поэтому он менее приоритетен.
#### Итого:
После 5 собраных схем OSPF я не понимаю логику, по которой он живет - то работает, то нет.