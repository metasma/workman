This routers general sub-net is 26 `172.26.0.0/16`, but some devices still remain in sub-net 23 `172.23.0.0/16`. They should be changed to 26 sub-net.

### Backstory

There were conflict with sub-net of Sayran wireless - router (it also had sub-net 23), so I decided to change Mnogovodnaya's sub-net from 23 to 26. I forgot that not all devices get dynamic adresses, so tourniquet and camera servers and their client devices left in 23 sub-net. I made some additional instructions in Main router and Mnogovodnaya router to get in touch with those devices. Then, tourniquet ip - address was changed to actual sub-net but camera server and camera ip - addresses are not, because responsible specialist doesn't want to change it. He said: 

> "leave them in sub-net 23, so I don't have to change their addresses and suffer 2 - 3 days. If there will be some problems - they will be mine."

### Changes

1) Mnogovodnaya:

	1. Virtual gateway added to get lan connection with sub-net 23 
	
```
/ip address print
...
8    172.23.0.1/16     172.23.0.0     vlan-201-mnogovodnaya
...
```

    2. Automatic route needed to find the path
```
/ip route print
...
  DAc 172.23.0.0/16     vlan-201-mnogovodnaya  main                  0
...
```
    3. Masquerade needed to make connection between sub-net 23 and Main router
```
/ip firewall nat print
...
 4    chain=srcnat action=masquerade src-address=172.23.0.0/16 out-interface=vlan-201-mnogovodnaya 
...
```

2) Main

1. 23 subnet added to wireguard peer's allowed-addresses
```
/interface wireguard peers print
...
3 wg-core    xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=              0  10.200.200.5/32  25s                 
                                                                           172.26.0.0/16                        
                                                                           172.23.0.0/16 
...
```

2. Added 1 more route to 23 subnet but with different prefix *24*
```
/ip route print
...
 8  As   172.23.0.0/16     206.62.55.197                              1 #connection with Mnogovodnaya (cameras)
 9  As   172.23.250.0/24   10.200.200.5                               1 #connection with Sayran wireless (traffic flow)
...
```

3) Sayran wireless

1. ip pool of it's general network has changed to prevent conflict with camera servers and camera addresses in Mnogovodnaya
```
/ip pool print
...
0  pool-dhcp  172.23.0.50-172.23.249.254
...
```
Before it was the full range, make the 3rd octate 250 free.
