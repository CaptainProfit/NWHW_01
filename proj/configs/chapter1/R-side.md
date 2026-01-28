```
enable
config term
	hostname R-side
	service password-encryption
	banner motd x client001, RTK, restricted area x
	no ip domain-lookup
	
	line console 0
		password cisco
		login
	exit

	line vty 0 15
		password cisco
		login
	exit
	
	interface gi 0/0
		ip address 11.20.201.1 255.255.255.0
		no shut
		ip ospf 1 area 3
	exit

	interface gi 0/1
		ip address 10.12.32.2   255.255.255.248
		no shut
		ip ospf 1 area 0
		ip ospf network point-to-point
	exit

	ip routing
	router ospf 1
		router-id 0.0.0.4
		network 10.12.32.0   0.0.0.7 area 0
		network 11.20.201.0 0.0.0.255 area 3
		default-information originate 
	exit
	ip route 0.0.0.0 0.0.0.0 gi 0/1
exit
copy ru st
```