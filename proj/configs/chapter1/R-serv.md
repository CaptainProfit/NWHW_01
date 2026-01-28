```
enable
config term
	hostname R-serv
	service password-encryption
	banner motd x client002, RTK, restricted area x
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
		ip address 11.20.203.1 255.255.255.0
		no shut
		ip ospf 1 area 4
	exit

	interface gi 0/1
		ip address 10.12.32.198 255.255.255.248
		no shut
		ip ospf 1 area 0
		ip ospf network point-to-point
	exit

	ip routing
	router ospf 1
		router-id 0.0.0.6
		network 10.12.32.198 0.0.0.7 area 0
		network 11.20.203.0 0.0.0.255 area 4
		default-information originate 
	exit
	ip route 0.0.0.0 0.0.0.0 gi 0/1
exit
copy ru st
```