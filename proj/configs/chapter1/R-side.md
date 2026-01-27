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

	ip routing
		
	interface gi 0/1
		ip address 10.12.32.2 255.255.255.248
		no shut
	exit

	interface gi 0/0
		ip address 11.20.201.1 255.255.255.0
		no shut
	exit

	ip route 0.0.0.0 0.0.0.0 gi 0/1
exit
copy ru st
```