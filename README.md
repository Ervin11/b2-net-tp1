

# TP1 - Remise dans le bain
---  

#### Configuration de VirtualBox  

Création de deux interfaces host-only **net1** : 10.1.1.0/24 et **net2** : 10.1.2.0/30.  
  
Il y a 254 adresses disponibles dans un /24 et 2 adresses dans un /30.  

Un /30 sert à n'avoir que deux adresses IP disponibles, donc pour qu'il n'y ai pas plus de connections possibles, par sécurité par exemple. 

---
#### Allumage et configuration de la VM

**Client 1**
```sh
[ervin@vm1 ~]$ ping -c 2 host.net1.tp1
64 bytes from host.net1.tp1 (10.1.1.1): icmp_seq=1 ttl=64 time=0.229 ms  
64 bytes from host.net1.tp1 (10.1.1.1): icmp_seq=2 ttl=64 time=0.253 ms
2 packets transmitted, 2 received, 0% packet loss, time 1003ms


[ervin@vm1 ~]$ ping -c 2 host.net2.tp1
64 bytes from host.net2.tp1 (10.1.2.1): icmp_seq=1 ttl=64 time=0.768 ms
64 bytes from host.net2.tp1 (10.1.2.1): icmp_seq=2 ttl=64 time=0.241 ms
2 packets transmitted, 2 received, 0% packet loss, time 1007ms


[ervin@vm1 ~]$ ping -c 2 client1.net1.tp1
64 bytes from client1.net1.tp1 (10.1.1.2): icmp_seq=1 ttl=64 time=0.069 ms
64 bytes from client1.net1.tp1 (10.1.1.2): icmp_seq=2 ttl=64 time=0.068 ms
2 packets transmitted, 2 received, 0% packet loss, time 1003ms


[ervin@vm1 ~]$ ping -c 2 client2.net1.tp1
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=1 ttl=64 time=0.671 ms
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=2 ttl=64 time=0.587 ms
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
```

**Client 2**

```sh
[ervin@vm2 ~]$ ping -c 2 host.net1.tp1
64 bytes from host.net1.tp1 (10.1.1.1): icmp_seq=1 ttl=64 time=0.263 ms
64 bytes from host.net1.tp1 (10.1.1.1): icmp_seq=2 ttl=64 time=0.336 ms
2 packets transmitted, 2 received, 0% packet loss, time 1000ms


[ervin@vm2 ~]$ ping -c 2 client1.net1.tp1
64 bytes from client1.net1.tp1 (10.1.1.2): icmp_seq=1 ttl=64 time=0.472 ms
64 bytes from client1.net1.tp1 (10.1.1.2): icmp_seq=2 ttl=64 time=0.462 ms
2 packets transmitted, 2 received, 0% packet loss, time 1000ms

[ervin@vm2 ~]$ ping -c 2 client2.net1.tp1
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=1 ttl=64 time=0.056 ms
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=2 ttl=64 time=0.172 ms
2 packets transmitted, 2 received, 0% packet loss, time 999ms

```
---  
#### Basics

**Table de routage Client 1**

```sh
[ervin@vm1 ~]$ ip route show
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 101 
10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 102
```
**Opérations sur la table de routage**

```sh
# Test de ping vers le client 2
[ervin@vm1 ~]$ ping -c 2 client2.net1.tp1
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=1 ttl=64 time=0.652 ms
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=2 ttl=64 time=0.775 ms
2 packets transmitted, 2 received, 0% packet loss, time 1002ms

# Suppression de la route vers net1
[ervin@vm1 ~]$ sudo ip route del 10.1.1.0/24
[ervin@vm1 ~]$ ip r s
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.1.2.0/24 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 102 

# Test du non-fonctionnement
[ervin@vm1 ~]$ ping -c 2 client2.net1.tp1
connect: Network is unreachable

# Ré-insertion de la route 
[ervin@vm1 ~]$ sudo ip route add 10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 101

# Test du fonctionnement
[vagrant@client1 ~]$ ping -c 2 client2.net1.tp1
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=1 ttl=64 time=0.477 ms
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=2 ttl=64 time=0.461 ms
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
```

**Table ARP**

```sh
[ervin@vm1 ~]$ ip neigh show
# NAT
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
# client2.net1.tp1
10.1.1.3 dev enp0s8 lladdr 08:00:27:76:36:ac STALE
# host.net1.tp2
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:02 REACHABLE
# host.net2.tp1
10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:00 STALE
```

```sh
[ervin@vm1 ~]$ sudo ip neigh flush all
[ervin@vm1 ~]$ ip neigh show
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:02 REACHABLE
```

```sh
[ervin@vm1 ~]$ ping -c 2 10.1.2.1
64 bytes from 10.1.2.1: icmp_seq=1 ttl=64 time=0.308 ms
64 bytes from 10.1.2.1: icmp_seq=2 ttl=64 time=0.643 ms
2 packets transmitted, 2 received, 0% packet loss, time 1002ms

[ervin@vm1 ~]$ ip neigh show
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:02 REACHABLE
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
# Une ligne correspondant à la MAC du host dans le net2 a été ajoutée
10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:00 STALE
```

**Capture réseau**

[Capture de requête ARP + Ping de client 1 vers client 2](https://github.com/Ervin11/b2-net-tp1/blob/master/Captures/ping.pcap)

```sh
# Shell 1
[ervin@vm1 ~]$ sudo ip n flush all
[ervin@vm1 ~]$ sudo tcpdump -i enp0s8 -w ping.pcap
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C10 packets captured
10 packets received by filter
0 packets dropped by kernel

# Shell 2

[ervin@vm1 ~]$ ping -c 4 host.net1.tp1
64 bytes from host.net1.tp1 (10.1.1.1): icmp_seq=1 ttl=64 time=1.84 ms
64 bytes from host.net1.tp1 (10.1.1.1): icmp_seq=2 ttl=64 time=0.550 ms
64 bytes from host.net1.tp1 (10.1.1.1): icmp_seq=3 ttl=64 time=0.253 ms
64 bytes from host.net1.tp1 (10.1.1.1): icmp_seq=4 ttl=64 time=0.286 ms
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
```

#### Communication simple entre 2 machines

[Capture de requête ARP + Ping de client 1 vers client 2](https://github.com/Ervin11/b2-net-tp1/blob/master/Captures/ping-2.pcap)

```sh
# Shell 1
[ervin@vm1 ~]$ sudo ip neigh flush all
[ervin@vm1 ~]$ sudo tcpdump -i enp0s8 -w ping-2.pcap
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C10 packets captured
10 packets received by filter
0 packets dropped by kernel

# Shell 2
[ervin@vm1 ~]$ ping -c 4 client2.net1.tp1
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=1 ttl=64 time=0.976 ms
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=2 ttl=64 time=0.523 ms
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=3 ttl=64 time=0.955 ms
64 bytes from client2.net1.tp1 (10.1.1.3): icmp_seq=4 ttl=64 time=0.489 ms
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
```

**Firewall**

Ouverture du port 8888 en UDP et TCP
```sh
[ervin@vm1 ~]$ sudo firewall-cmd --add-port=8888/tcp --permanent
success
[ervin@vm1 ~]$ sudo firewall-cmd --add-port=8888/udp --permanent
success
[ervin@vm1 ~]$ sudo firewall-cmd --reload
success
```

**Netcat**

***UDP***

[Capture UDP](https://github.com/Ervin11/b2-net-tp1/blob/master/Captures/nc-udp.pcap)

```sh
# Client 1
# Shell 1
[ervin@vm1 ~]$ nc -l -u 8888
yes
yo

# Shell 2
[ervin@vm1 ~]$ sudo tcpdump -i enp0s8 -w nc-udp.pcap
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C2 packets captured
2 packets received by filter
0 packets dropped by kernel

# Client 2
[ervin@vm2 ~]$ nc -u client1.net1.tp1 8888
yes
yo
```

***TCP***

[Capture TCP](https://github.com/Ervin11/b2-net-tp1/blob/master/Captures/nc-tcp.pcap)

```sh
# Client 1
# Shell 1
[ervin@vm1 ~]$ nc -l 8888
yes
yo

# Shell 2
[ervin@vm1 ~]$ sudo tcpdump -i enp0s8 -w nc-tcp.pcap
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C2 packets captured
2 packets received by filter
0 packets dropped by kernel

# Client 2
[ervin@vm2 ~]$ nc client1.net1.tp1 8888
yes
yo
```
[Capture Firewall](https://github.com/Ervin11/b2-net-tp1/blob/master/Captures/firewall.pcap)

#### Routage statique et simple

```sh
# Activation du IP forwarding sur client 1
[ervin@vm1 ~]$ sysctl -w net.ipv4.ip_forward=1`

# Client 2
[ervin@vm2 ~]$ ip r s
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 102 
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.3 metric 101 

# Ajout d'une route vers net2
[ervin@vm2 ~]$ sudo ip route add 10.1.2.0/30 via 10.1.1.2 dev enp0s8

[ervin@vm2 ~]$ ping 10.1.2.1
64 bytes from 10.1.2.1: icmp_seq=1 ttl=64 time=1.18 ms
64 bytes from 10.1.2.1: icmp_seq=2 ttl=64 time=0.898 ms
64 bytes from 10.1.2.1: icmp_seq=3 ttl=64 time=1.02 ms
3 packets transmitted, 3 received, 0% packet loss, time 2003ms

[ervin@vm2 ~]$ sudo traceroute -I 10.1.2.1
traceroute to 10.1.2.1 (10.1.2.1), 30 hops max, 60 byte packets
 1  client1.net1.tp1 (10.1.1.2)  0.289 ms  0.143 ms  0.087 ms
 2  10.1.2.1 (10.1.2.1)  0.142 ms  0.165 ms  0.121 ms

```


