```
enable
config term
	hostname Router4
	no ip domain-lookup
		
	router ospf 12
		network 1.1.1.0 0.0.0.255 area 1
		network 1.1.2.0 0.0.0.255 area 1
		default-information originate
		router-id 1.1.1.1
	exit

	interface gi 0/0/1
		ip address 1.1.2.1 255.255.255.0
		ip ospf 12 area 1
		no shut
	exit

	interface gi 0/0/0
		ip address 1.1.1.1 255.255.255.0
		ip ospf 12 area 1
		no shut
	exit

	ip routing
exit
copy ru st
```