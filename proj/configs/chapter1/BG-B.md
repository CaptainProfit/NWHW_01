```
enable
config term
	hostname BG-B
	service password-encryption
	banner motd x BG-B, RTK, restricted area x
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
		router-id 11.22.72.2
	exit

	interface gi 0/0
		ip address 11.22.72.2 255.0.0.0
		ip ospf 12 area 0
		no shut
	exit

	interface gi 0/1
		ip address 10.12.32.129 255.255.255.248
		no shut
	exit

	ip route 11.20.202.0 255.255.255.0 gi 0/1
exit
copy ru st
```