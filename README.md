# Static-and-dynamic-routing-OSPF-

- Поднять три виртуалки

- Объединить их разными vlan

- поднять OSPF между машинами на базе Quagga;

- изобразить ассиметричный роутинг;

- сделать один из линков "дорогим", но что бы при этом роутинг был симметричным.

---


Дано:

![image](https://github.com/user-attachments/assets/1c4634b7-36ac-4321-add2-932c10d34b49)



![image](https://github.com/user-attachments/assets/dddaba8d-0740-45d2-8ca8-e953088004ff)


Выполним плейбук:

https://github.com/spbsupprt/Static-and-dynamic-routing-OSPF/blob/main/ospf.yml

![image](https://github.com/user-attachments/assets/bd45d209-2713-41a2-b604-c3269941d17f)


Итоговые проверки:

- router1

```
router1# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/300] via 10.0.12.2, enp0s9, weight 1, 00:00:22
O>* 10.0.11.0/30 [110/200] via 10.0.12.2, enp0s9, weight 1, 00:00:22
O   10.0.12.0/30 [110/100] is directly connected, enp0s9, weight 1, 00:23:41
O   192.168.10.0/24 [110/100] is directly connected, enp0s10, weight 1, 00:23:41
O>* 192.168.20.0/24 [110/300] via 10.0.12.2, enp0s9, weight 1, 00:00:22
O>* 192.168.30.0/24 [110/200] via 10.0.12.2, enp0s9, weight 1, 00:12:15
router1# exit

root@router1:# ping -I 192.168.10.1 192.168.20.1
PING 192.168.20.1 (192.168.20.1) from 192.168.10.1 : 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=64 time=4.23 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=64 time=1.40 ms
........................................................
64 bytes from 192.168.20.1: icmp_seq=73 ttl=64 time=0.982 ms
64 bytes from 192.168.20.1: icmp_seq=74 ttl=64 time=1.96 ms
64 bytes from 192.168.20.1: icmp_seq=75 ttl=64 time=2.72 ms
64 bytes from 192.168.20.1: icmp_seq=76 ttl=64 time=1.08 ms
^Z
[1]+  Stopped                 ping -I 192.168.10.1 192.168.20.1

Ещё раз попингуем,чтобы насладиться симметричным маршрутом.

root@router1:# ping -I 192.168.10.1 192.168.20.1
PING 192.168.20.1 (192.168.20.1) from 192.168.10.1 : 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=63 time=6.04 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=63 time=2.31 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=63 time=14.2 ms
```

- router2

```
router2# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/100] is directly connected, enp0s8, weight 1, 00:19:52
O   10.0.11.0/30 [110/100] is directly connected, enp0s9, weight 1, 00:20:23
O>* 10.0.12.0/30 [110/200] via 10.0.10.1, enp0s8, weight 1, 00:13:01
  *                        via 10.0.11.1, enp0s9, weight 1, 00:13:01
O>* 192.168.10.0/24 [110/200] via 10.0.10.1, enp0s8, weight 1, 00:19:48
O   192.168.20.0/24 [110/100] is directly connected, enp0s10, weight 1, 00:20:23
O>* 192.168.30.0/24 [110/200] via 10.0.11.1, enp0s9, weight 1, 00:14:37


root@router2:# tcpdump -i enp0s8
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
08:51:11.054027 IP router2 > 192.168.10.1: ICMP echo reply, id 1, seq 18, length 64
08:51:12.044476 IP router2 > 192.168.10.1: ICMP echo reply, id 1, seq 19, length 64
08:51:13.046314 IP router2 > 192.168.10.1: ICMP echo reply, id 1, seq 20, length 64



root@router2:# tcpdump -i enp0s9
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
08:51:49.032882 IP 10.0.11.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
08:51:49.192973 IP 192.168.10.1 > router2: ICMP echo request, id 1, seq 56, length 64
08:51:50.192462 IP 192.168.10.1 > router2: ICMP echo request, id 1, seq 57, length 64


Поднимем стоимость маршрута, чтобы пакеты ходили через соседний роутер.


root@router2:# vtysh

Hello, this is FRRouting (version 10.2.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router2# conf t
router2(config)# int enp0s8
router2(config-if)# ip ospf cost 1000
router2(config-if)# exit
router2(config)# exit
router2# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/1000] is directly connected, enp0s8, weight 1, 00:00:17
O   10.0.11.0/30 [110/100] is directly connected, enp0s9, weight 1, 00:28:25
O>* 10.0.12.0/30 [110/200] via 10.0.11.1, enp0s9, weight 1, 00:00:17
O>* 192.168.10.0/24 [110/300] via 10.0.11.1, enp0s9, weight 1, 00:00:17
O   192.168.20.0/24 [110/100] is directly connected, enp0s10, weight 1, 00:28:25
O>* 192.168.30.0/24 [110/200] via 10.0.11.1, enp0s9, weight 1, 00:22:39



root@router2:# tcpdump -i enp0s9
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
08:56:18.834270 IP router2 > ospf-all.mcast.net: OSPFv2, Hello, length 48
08:56:19.006278 IP 10.0.11.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
08:56:21.337380 IP 192.168.10.1 > router2: ICMP echo request, id 2, seq 1, length 64
08:56:21.337544 IP router2 > 192.168.10.1: ICMP echo reply, id 2, seq 1, length 64
08:56:22.352555 IP 192.168.10.1 > router2: ICMP echo request, id 2, seq 2, length 64
```

