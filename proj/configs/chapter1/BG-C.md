```
enable
config term
	hostname BG-C
	service password-encryption
	banner motd x BG-C, RTK, restricted area x
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
		
	router ospf 12
		router-id 11.22.72.3
	exit

	interface gi 0/0/0
		ip address 11.22.72.3 255.0.0.0
		ip ospf 12 area 0
		no shut
	exit

	interface gi 0/0/1
		ip address 10.12.32.197 255.255.255.248
		no shut
	exit

	ip route 11.20.203.0 255.255.255.0 gi 0/0/1
exit
copy ru st
```