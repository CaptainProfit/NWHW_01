# Схема
# Чего хочу
# часть1

Долго и упорно пытался заставить работать AS (или то, что под этим подразумеваю)

![](images/01b.png)

В итоге пришлось упростить схему чтобы продолжить. Теперь интернет это один коммутатор, никак не настроеный.

![[images/stage1/01c.png]]

Небольшой коллектив, строим сеть с нуля
- ~~настроить ОSPF для эмуляции WAN~~
- создать сеть для офиса на 3х человек.
- настроить днс
- запретить ssh трафик от пользователей - только от админа.
## реализация
Настройка R1:
``` R1
enable
config term
	hostname BG-A
	service password-encryption
	banner motd x hello, be nice x
	no ip domain-lookup
	
	line console 0
		password cisco
		login
	exit

	username admin privilege 15 secret admin
	line vty 0 15
		transport input all
		password cisco
		login
		login local
	exit

	interface gi 0/0/0
		ip address 11.20.202.1 255.255.255.0
		no shut
	exit

	interface gi 0/0/1
		ip address 12.22.72.11   255.0.0.0
		no shut
	exit

	ip routing
	
	ip access-list extended SEC
		30 permit tcp host 11.20.202.10 any any
		31 deny tcp any any range 22 23
		default permit ip any any
	exit
exit

clock set 11:18:00 jan 31 2026
terminal history size 256
copy ru st
```

## проверка
В результате админ имеет доступ к роутеру

![[images/stage1/02.png]]

А остальные нет

![[images/stage1/03.png]]

При этом доступ к интернету есть

![[images/stage1/04.png]]


# часть2
- У разработчиков гит не работает(ssh) - нужно поправить правила.
- Админ узнал про ssh и решил тоже на него перейти
- количество сотрудников растет, всем прописывать адреса вручную стало накладно.
## Обновление схемы
![[images/stage2/01.png]]
## Доступ к гиту
для гитхаба настраиваю эмулятор ssh на роутере ssh.com:
``` git\.com
enable
config term
	hostname git
	service password-encryption
	banner motd x hello, be nice x
	no ip domain-lookup
	
	line console 0
		password cisco
		login
	exit
	
	ip domain-name git.com
	crypto key generate rsa general-keys modulus 1024
	username CaptainProfit privilege 15 secret git
	username gitadmin privilege 15 secret admin
	line vty 0 15
		transport input all
		password cisco
		login
		login local
		transport output ssh
		transport input ssh
	exit

	interface gi 0/0/0
		ip address 12.22.72.13   255.0.0.0
		no shut
	exit

exit

clock set 11:18:00 jan 31 2026
terminal history size 256
copy ru st
```

 И он работает

![[images/stage2/02.png]]

Но только с машины админа.
``` R1
conf t
	ip access-list extended Sec
	no 31
	31 permit tcp any host 12.22.72.13
	35 deny tcp any any range 22 telnet (36 match(es))
exit
```
Теперь разработчикк доволен и всё ещё не имеет доступ к серверу.

![[images/stage2/03.png]]
## Настройка ssh

Себе тоже сделаю ssh
``` R1
config term
	hostname R1
	ip domain-name kontora.ru
	username admin privilege 15 secret admin
	crypto key generate rsa general-keys modulus 1024
	line vty 0 15
		transport output ssh
		transport input ssh
		password cisco
		login
		login local
	exit
exit
```

![[images/stage2/04.png]]

## Настройка DHCP
``` R1
conf t
	ip dhcp pool workers
		default-router 11.20.202.1
		domain-name contora.ru
		dns-server 12.22.72.10
		network 11.20.202.0 255.255.255.0
	exit
	ip dhcp excluded-address 11.20.202.1 11.20.202.19
service dhcp
```
Админу оставляю статический адрес. Работников перевожу на DHCP

![[images/stage2/05.png]]
# часть3
- Появляется новое крыло и туда нужно поставить коммутатор и на него прокинуть несколько проводов, для надежности
- В процессе работы провода перепутались и один из кабелей оказался присоединён в сеть другой конторы. 
- Появилась потребность в Nat
## Ethercahnnel в новое крыло
Хочу LACP
``` S12
ena
	conf t
		inter range fa 0/10-12
		channel-group 1 mode active
		channel-protocol lacp
	exit	
```
Заработало

![[images/stage3/02.png]]

![[images/stage3/01.png]]
# Производим ревизию схемы с помощью LLDP и ищем проблемный кабель.
У одного из пользователей не работает корпоративная почта. Какимто чудом он получил адрес 11.20.203.43.

![[images/stage3/06.png]]

![[images/stage3/03.png]]

Что покажет нам CDP `show cdp neighbours`:
![[images/stage3/05.png]]

Ничего криминального.  А`show cdp entry *`:

![[images/stage3/07.png]]

Обращаю внимание на Device ID и Interface - у fa 0/10 и fa0/11 всё правильно, а у fa0/12 какойто левый. Иду ищу его и перетыкаю.
Выяснилось, что какой-то кабель шел в сетку соседней конторы, ну как-то так вышло.

![[images/stage3/04.png]]

Поправил и сменил адрес 
![[images/stage3/08.png]]
## Реализую NAT
И тут я встрял. Раньше вся сетка была на адресах, 


Со временем контора расширяется, появляется гораздо больше сотрудников, выделяют несколько департаментов, некоторые расположены в другом крыле здания. 
- выделить каждому департаменту подсеть
- настроить NAT для выхода в сеть
	- одному разработчику нужен статический IP
- Старый сетевой инженер уходит, приходит новый. Прежде чем пойти в обход решает составить схему с помощью LLDP
## реализация
![](images/02.png)
## проверка

# часть4
![](images/03.png)
- делает дублирование первого контура
- создает двойной NAT 
- разработчику понадобился статический айпи
- в департаментах назрела нехватка адресов в подсети - прокидывает VLAN с дополнительной подсетью и часть департаментов переводит на неё, при этом пользователи должны иметь доступ друг к другу для работы.
## реализация
## проверка

# Итого

| часть | Технологии           |
| ----- | -------------------- |
| 1     | OSPF<br>DHCP<br>ACL  |
| 2     | static NAT<br>NAT    |
| 3     | etherchannel<br>LLDP |
| 4     | STP<br>VLAN<br>FHRP  |

