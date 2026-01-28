```
enable
config term
	hostname Router7
	no ip domain-lookup
		
	router ospf 12
		network 1.1.4.0 0.0.0.255 area 2
		network 1.1.5.0 0.0.0.255 area 2
		default-information originate
		router-id 1.1.1.1
	exit

	interface gi 0/0/1
		ip address 1.1.4.2 255.255.255.0
		ip ospf 12 area 2
		no shut
	exit

	interface gi 0/0/0
		ip address 1.1.5.1 255.255.255.0
		ip ospf 12 area 2
		no shut
	exit

	ip routing
exit
copy ru st
```