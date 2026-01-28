```
enable
config term
	hostname Router5
	no ip domain-lookup
		
	router ospf 12
		network 1.1.2.0 0.0.0.255 area 1
		network 1.1.3.0 0.0.0.255 area 0
		default-information originate
		router-id 2.2.2.2
	exit

	interface gi 0/0/1
		ip address 1.1.2.2 255.255.255.0
		ip ospf 12 area 1
		no shut
	exit

	interface gi 0/0/0
		ip address 1.1.3.1 255.255.255.0
		ip ospf 12 area 0
		ip ospf network point-to-point
		no shut
	exit

	ip route 0.0.0.0 0.0.0.0 gi 0/0/0
	ip routing
exit
copy ru st
```