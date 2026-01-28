```
enable
config term
	hostname BG-10A
	service password-encryption
	banner motd x BG-10A, RTK, restricted area x
	no ip domain-lookup
	
	line console 0
		password cisco
		login
	exit

	line vty 0 15
		password cisco
		login
	exit

	ip routing
	ip route 0.0.0.0 0.0.0.0 gi 0/1
	router ospf 1
		router-id 11.20.201.1
		default-information originate
	exit

	interface gi 0/0
		ip address 11.20.201.1 255.0.0.0
		ip ospf 1 area 2
		no shut
	exit

	interface gi 0/1
		ip address 10.12.32.2 255.255.255.248
		ip ospf 1 area 1
		ip ospf network point-to-point
		no shut
	exit
exit

clock set 11:18:00 jan 27 2026
terminal history size 256
copy ru st
```