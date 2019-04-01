# TP3 - OSPF, VLAN, une vraie topologie

# Sommaire

* [I. Manipulation de switches et de VLAN](#i-manipulation-de-switches-et-de-vlan)
  * [1. Mise en place du lab](#1-mise-en-place-du-lab)
  * [2. Configuration des VLANs](#2-configuration-des-vlans)
* [II. Manipulation simple de routeurs](#ii-manipulation-simple-de-routeurs)
  * [1. Mise en place du lab](#1-mise-en-place-du-lab-1)
  * [2. Configuration du routage statique](#2-configuration-du-routage-statique)
* [III. Mise en place d'OSPF](#iii-mise-en-place-dospf)
  * [1. Mise en place du lab](#1-mise-en-place-du-lab-2)
  * [2. Configuration de OSPF](#2-configuration-de-ospf)
* [IV. Lab Final](#iv-lab-final)

# I. Manipulation de switches et de VLAN

## 1. Mise en place du lab

#### > Tableau d'adressage

Hosts | `10.1.1.0/24`
--- | ---
`client1.lab1.tp3` | `10.1.1.1/24`
`client2.lab1.tp3` | `10.1.1.2/24`
`client3.lab1.tp3` | `10.1.1.3/24`

* Pour se connecter en ssh à chaque VM on set les adaptateurs à 2 depuis GNS3 (il n'y en a qu'un de base) pour pouvoir ajouter un réseau host-only qui sera en ```10.1.2.0/24``` (l'hôte aura l'ip ```10.1.2.4/24```)

    * Connexion en **ssh** sur chaque VM : 

        * D'abord on définit une ip static dans le réseau host-only ```10.1.2.0/24``` :

            Exemple pour client1 :

            ```
            [axel@client1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
            TYPE=Ethernet
            BOOTPROTO=static
            NAME=enp0s8
            DEVICE=enp0s8
            ONBOOT=yes

            IPADDR=10.1.2.1
            NETMASK=255.255.255.0
            ```

        * Ensuite on va redémarer les interfaces : 

            ```
            sudo ifdown enp0s8
            sudo ifup enp0s8
            ```

    * Changement du **hostname** pour chaque VM :

        ```
        [axel@localhost ~]$ sudo hostname client1.lab1.tp3

        [axel@localhost ~]$ sudo hostname client2.lab1.tp3

        [axel@localhost ~]$ sudo hostname client2.lab1.tp3
        ```

    * Exemple de ping de client1 vers client2: 

        ```
        [axel@client1 ~]$ ping 10.1.1.2
        PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
        64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=4.89 ms
        64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=1.23 ms
        64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=1.33 ms
        ^C
        --- 10.1.1.2 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2002ms
        rtt min/avg/max/mdev = 1.231/2.486/4.892/1.702 ms
        ```

## 2. Configuration des VLANs

* Mise en place des ports en mode access :

    * Configuration de SW1 : 

        * Création d'un VLAN10 que l'on assigne à Ethernet 0/0 sur SW1 :

            ```
            SW1#conf t
            SW1(config)#vlan 10
            SW1(config-vlan)#name VLAN10
            SW1(config-vlan)#exit

            ...

            SW1(config)#interface Ethernet 0/0
            SW1(config-if)#switchport mode access
            SW1(config-if)#switchport access vlan 10
            ```

        * Création d'un VLAN20 que l'on assigne à Ethernet 0/2 sur SW1 : 

            ```
            SW1(config)#vlan 20
            SW1(config-vlan)#name VLAN20
            SW1(config-vlan)#exit

            ...

            SW1(config)#interface Ethernet 0/1
            SW1(config-if)#switchport mode access
            SW1(config-if)#switchport access vlan 20
            ```

        * Ajout d'un **TRUNK** pour connecter SW1 à SW2 depuis SW1 :

            ```
            SW1#conf t
            SW1(config-if)#interface Ethernet 0/2
            SW1(config-if)#switchport trunk encapsulation dot1q
            SW1(config-if)#switchport mode trunk
            ```

    * Configuration de SW2 : 

         * Création d'un VLAN10 que l'on assigne à Ethernet 0/0 sur SW2 :

            ```
            SW2#conf t
            SW2(config)#vlan 10
            SW2(config-vlan)#name VLAN10
            SW2(config-vlan)#exit

            ...

            SW2(config)#interface Ethernet 0/0
            SW2(config-if)#switchport mode access
            SW2(config-if)#switchport access vlan 10
            ```

        * Ajout d'un **TRUNK** pour connecter SW1 à SW2 depuis SW2 :

            ```
            SW2#conf t
            SW2(config-if)#interface Ethernet 0/1
            SW2(config-if)#switchport trunk encapsulation dot1q
            SW2(config-if)#switchport mode trunk
            ```

* Différents test de ping entre les interfaces : 

    * Ping de client1 vers client3 :

        ```
        [axel@client1 ~]$ ping 10.1.1.3
        PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
        64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=2.60 ms
        64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=1.95 ms
        64 bytes from 10.1.1.3: icmp_seq=3 ttl=64 time=1.85 ms
        ^C
        --- 10.1.1.3 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2004ms
        rtt min/avg/max/mdev = 1.851/2.138/2.607/0.338 ms
        ```

    * Traceroute de client1 vers client3 :

        ```
        [axel@client1 ~]$ traceroute 10.1.1.3
        traceroute to 10.1.1.3 (10.1.1.3), 30 hops max, 60 byte packets
        1  10.1.1.3 (10.1.1.3)  2.316 ms !X  2.192 ms !X  4.483 ms !X
        ```

    * Ping de client1 vers client2 :

        ```
        [axel@client1 ~]$ ping 10.1.1.2
        PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
        From 10.1.1.1 icmp_seq=1 Destination Host Unreachable
        From 10.1.1.1 icmp_seq=2 Destination Host Unreachable
        From 10.1.1.1 icmp_seq=3 Destination Host Unreachable
        From 10.1.1.1 icmp_seq=4 Destination Host Unreachable
        ^X^C
        --- 10.1.1.2 ping statistics ---
        5 packets transmitted, 0 received, +4 errors, 100% packet loss, time 4001ms
        pipe 4
        ```

        _client2 est bien **injoignable**_

# II. Manipulation simple de routeurs

## 1. Mise en place du lab

#### > 3 Réseaux host only

Nom | Adresse
--- | ---
`lab2-net1` | `10.2.1.0/24`
`lab2-net2` | `10.2.2.0/24`
`lab2-net12` | `10.2.12.0/30`

#### > Tableau d'adressage

Hosts | `lab2-net1` |  `lab2-net2` |  `lab2-net12` 
--- | --- | --- | ---
`client1.lab2.tp3` | `10.2.1.10/24` | x | x
`client2.lab2.tp3` | `10.2.1.11/24` | x | x
`server1.lab2.tp3` | x | `10.2.2.10/24` | x
`router1.lab2.tp3` | `10.2.1.254/24` | x | `10.2.12.1/30`
`router2.lab2.tp3` | x | `10.2.2.254/24` | `10.2.12.2/30`

* Test sur les client / serveur pour voir si ils peuvent joindre leurs **gateways** respectives :

    * client2 vers router1 :

        ```
        [axel@client2 ~]$ ping 10.2.1.254
        PING 10.2.1.254 (10.2.1.254) 56(84) bytes of data.
        64 bytes from 10.2.1.254: icmp_seq=1 ttl=255 time=2.60 ms
        64 bytes from 10.2.1.254: icmp_seq=2 ttl=255 time=1.95 ms
        64 bytes from 10.2.1.254: icmp_seq=3 ttl=255 time=1.85 ms
        ^C
        --- 10.2.1.254 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 3012ms
        rtt min/avg/max/mdev = 1.851/2.138/2.607/0.338 ms
        ```
    
    * server1 vers router2 :

        ```
        [axel@server1 ~]$ ping 10.2.2.254
        PING 10.2.2.254 (10.2.2.254) 56(84) bytes of data.
        64 bytes from 10.2.2.254: icmp_seq=1 ttl=255 time=2.60 ms
        64 bytes from 10.2.2.254: icmp_seq=2 ttl=255 time=1.95 ms
        64 bytes from 10.2.2.254: icmp_seq=3 ttl=255 time=1.85 ms
        ^C
        --- 10.2.2.254 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 3012ms
        rtt min/avg/max/mdev = 1.851/2.138/2.607/0.338 ms
        ```

        _Les **client / server** peuvent joindre leurs gateways respectives_

    * router2 vers router1 :

        ```
        router2#ping 10.2.12.1

        Type escape sequence to abort.
        Sending 5, 100-byte ICMP Echos to 10.2.12.1, timeout is 2 seconds:
        .!!!!
        Success rate is 80 percent (4/5), round-trip min/avg/max = 48/60/64 ms
        router2#
        ```

        _Les **routeurs** communiquent bien entre eux_

## 2. Configuration du routage statique

* Ajout de route :

    * Sur server1 :

        ```
        sudo ip route add 10.2.1.0/24 via 10.2.2.254 dev enp0s3
        ```

    * Sur client2 :
    
        ```
        sudo ip route add 10.2.2.0/24 via 10.2.1.254 dev enp0s3
        ```
    
    * Sur router1 :

        ```
        router1(config)#ip route 10.2.2.0 255.255.255.0 10.2.12.2
        ```

    * Sur router2 : 

        ```
        router2(config)#ip route 10.2.1.0 255.255.255.0 10.2.12.1
        ```

* Test de ping entre client / server :

    * client2 vers server1 :

        ```
        [axel@client2 ~]$ ping 10.2.2.10
        PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
        64 bytes from 10.2.2.10: icmp_seq=1 ttl=62 time=2.60 ms
        64 bytes from 10.2.2.10: icmp_seq=2 ttl=62 time=1.95 ms
        64 bytes from 10.2.2.10: icmp_seq=3 ttl=62 time=1.85 ms
        ^C
        --- 10.2.2.10 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 3012ms
        rtt min/avg/max/mdev = 26.015/33.783/42.549/6.080 ms
        ```

    * server1 vers client2 :

        ```
        [axel@client2 ~]$ ping 10.2.1.11
        PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
        64 bytes from 10.2.1.11: icmp_seq=1 ttl=62 time=2.60 ms
        64 bytes from 10.2.1.11: icmp_seq=2 ttl=62 time=1.95 ms
        64 bytes from 10.2.1.11: icmp_seq=3 ttl=62 time=1.85 ms
        ^C
        --- 10.2.1.11 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 3012ms
        rtt min/avg/max/mdev = 27.334/33.680/40.574/4.687 ms
        ```

* Topologie où tous les clients auraient la même passerelle :

```md
    router1              router2            server1

+--------+            +----------+       +-----------+
|        +------------+          +-------+           |
|        |            |          |       |           |
+---+----+            +----------+       +-----------+
    |
    |
    |
    |
    |
    |
+---+---+  switch1
|       |
|       +------------+
+---+---+            |
    |                |
    |                |
    |                |
    |                |
 +--+--+          +--+--+
 |     |          |     |
 |     |          |     |
 +-----+          +-----+

 client1           client2
```

# III. Mise en place d'OSPF
 
## 1. Mise en place du lab

#### > Tableau d'adressage

Hosts | `10.3.100.0/30` | `10.3.100.4/30` | `10.3.100.8/30` | `10.3.100.12/30` | `10.3.100.16/30` | `10.3.100.20/30` | `10.3.101.0/24` | `10.3.102.0/24`
--- | --- | --- | --- | --- | --- | --- | --- | --- 
`client1.lab3.tp3` | x | x | x | x | x | x | `10.3.101.10/24` | x 
`server1.lab3.tp3` | x | x | x | x | x | x | x | `10.3.102.10/24` 
`router1.lab3.tp3` | `10.3.100.1/30` | x | x | x | x | `10.3.100.22/30` | x | `10.3.102.254/24` 
`router2.lab3.tp3` | `10.3.100.2/30` | `10.3.100.4/30` | x | x | x | x | x | x 
`router3.lab3.tp3` | x | `10.3.100.5/30` | `10.3.100.9/30` | x | x | x | x | x 
`router4.lab3.tp3` | x | x | `10.3.100.10/30` | `10.3.100.13/30` | x | x | `10.3.101.254/24` | x 
`router5.lab3.tp3` | x | x | x | `10.3.100.14/30` | `10.3.100.17/30` | x | x | x 
`router6.lab3.tp3` | x | x | x | x | `10.3.100.18/30` | `10.3.100.21/30` | x | x 

* Test sur les client / serveur pour voir si ils peuvent joindre leurs **gateways** respectives :

    * client1 vers router4 :

        ```
        [axel@client1 ~]$ ping 10.3.101.254
        PING 10.3.101.254 (10.3.101.254) 56(84) bytes of data.
        64 bytes from 10.3.101.254: icmp_seq=1 ttl=62 time=2.85 ms
        64 bytes from 10.3.101.254: icmp_seq=2 ttl=62 time=2.20 ms
        64 bytes from 10.3.101.254: icmp_seq=3 ttl=62 time=1.70 ms
        ^C
        --- 10.3.101.254 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 3501ms
        rtt min/avg/max/mdev = 27.334/33.680/40.574/4.687 ms
        ```

    * server1 vers router1 :

        ```
        [axel@server1 ~]$ ping 10.3.102.254
        PING 10.3.102.254 (10.3.102.254) 56(84) bytes of data.
        64 bytes from 10.3.102.254: icmp_seq=1 ttl=62 time=2.60 ms
        64 bytes from 10.3.102.254: icmp_seq=2 ttl=62 time=1.95 ms
        64 bytes from 10.3.102.254: icmp_seq=3 ttl=62 time=1.85 ms
        64 bytes from 10.3.102.254: icmp_seq=4 ttl=62 time=1.85 ms
        ^C
        --- 10.3.102.254 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 2830ms
        rtt min/avg/max/mdev = 21.329/31.201/42.675/7.593 ms
        ```

* Test pour voir si les routeurs peuvent discuter entre eux (de point à point) :

    * Vu que y en a beaucoup on met qu'un screen (ping de router2 vers router3) :

        ```
        R2#ping 10.3.100.6

        Type escape sequence to abort.
        Sending 5, 100-byte ICMP Echos to 10.3.100.6, timeout is 2 seconds:
        .!!!!
        Success rate is 80 percent (4/5), round-trip min/avg/max = 16/42/68 ms
        ```

        ✅**TOUS** les autres routeurs peuvent discuter entre eux (de point à point)

## 2. Configuration de OSPF

* Activation de OSPF sur chaque router :

    ```
    (config)# router ospf 1
    ```

* Définition d'un **router-id** sur chaque router :

    ```
    (config-router)# router-id 1.1.1.1

    etc..
    ```

* Partage de **tous** les réseaux auquel le routeur est connecté : 

    * Exemple sur router2 : 

        ```
        R2#show ip protocols
        Routing Protocol is "ospf 1"
        Outgoing update filter list for all
        interfaces is not set
        Incoming update filter list for all
        interfaces is not set
        Router ID 2.2.2.2
        Number of areas in this router is 2. 2 normal 0 stub 0 nssa
        Maximum path: 4
        Routing for Networks:
            10.3.100.0 0.0.0.3 area 0
            10.3.101.0 0.0.0.3 area 1
        ```

* Ajout d'une route par défault sur client1 / server1 qui pointe vers leurs passerelles respectives : 

    * Test de ping : 

        * De client1 vers server1 : 

            ```
            [quentin@client1 ~]$ ping 10.3.102.10
            PING 10.3.102.10 (10.3.102.10) 60(86) bytes of data.
            64 bytes from 10.3.102.10: icmp_seq=1 ttl=62 time=2.85 ms
            64 bytes from 10.3.102.10: icmp_seq=2 ttl=62 time=2.02 ms
            64 bytes from 10.3.102.10: icmp_seq=3 ttl=62 time=1.97 ms
            ^C
            --- 10.3.102.10 ping statistics ---
            3 packets transmitted, 3 received, 0% packet loss, time 3120ms
            rtt min/avg/max/mdev = 23.839/33.992/44.372/9.231 ms
            ```

            _It's work 🔥_

# IV. Lab Final

#### > Topologie

```
                                                         NAT
                                                    +------------+
                                            +-------+            |
                                            |       |            |
                                            |       +------------+
                                            |
                                            |
                                  router1   |
                            +---------------++
                            |                |
                            |                |
                            |                |
                +-----------+                +------------+
                |           +----------------+            |
                |                                         |
                |                                         |
                |                                         |
                |                                         |
        +-------+-------+                        +--------+-----+
        |               |                        |              |
router2 |               +------------------------+              |router3
        |               |                        |              |
        +-------+-------+                        +--------+-----+
                |                                         |
                |                                         |
                |                                         |
                |                                         |
                |                                         |
                |                                         |
         +------+-------+                         +-------+------+
         |              |                         |              |
         |              |                         |              |
 switch1 |              |                         |              |switch2
         |              |                         |              |
         +------+-------+                         +-----+---+----+
                |                                       |   |
                |                               VLAN 20 |   | VLAN 30
                |   VLAN 10                             |   |
                |                    +------------+     |   |    +--------------+
                |                    |            |     |   |    |              |
         +------+-------+            |            +-----+   +----+              |
         |              |            |            |              |              |
         |              |            +------------+              +--------------+
  server1|              |                client1                      client2
         |              |
         +--------------+

```

#### > Tableau d'adressage IP

Hosts | `10.3.1.0/30` |  `10.3.1.4/30` |  `10.3.1.8/30` | `10.3.101.0/24` | `10.3.102.0/24` | `10.3.103.0/24`
--- | --- | --- | --- | --- | --- | ---
`router1.lab4.tp3` | `10.3.1.1/30` | x | `10.3.1.9/30` | x | x | x
`router2.lab4.tp3` | `10.3.1.2/30` | `10.3.1.5/30` | x | x | x | x
`router3.lab4.tp3` | x | `10.3.1.6/30` | `10.3.1.10/30` | x | x | x
`client1.lab4.tp3` | x | x | x | `10.3.101.10/24`| x | x
`client2.lab4.tp3` | x | x | x | x | x | `10.3.103.11/24`
`server1.lab4.tp3` | x | x | x | x | `10.3.102.10/24` | x


### 1. Configuration des VMs

* On set le bon hostname sur chacune des VMs : 

    ```
    [axel@localhost ~]$ sudo echo 'client1.lab4.tp3' | sudo tee /etc/hostname

    [axel@localhost ~]$ sudo echo 'client2.lab4.tp3' | sudo tee /etc/hostname

    [axel@localhost ~]$ sudo echo 'server1.lab4.tp3' | sudo tee /etc/hostname
    ```

* Configuration d'ip statique sur chacune des VMs : 

    * Exemple pour client1 :
        ```
        [axel@client1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
        TYPE=Ethernet
        BOOTPROTO=static
        NAME=enp0s8
        DEVICE=enp0s8
        ONBOOT=yes

        IPADDR=10.3.101.10
        NETMASK=255.255.255.0
        ```

        _etc.._


## 2. Configuration des routeurs 

* On va commencer par attribuer des **IPs** aux routers :

    * Pour router1 : 

        ```
        R1#show ip int br
        Interface                  IP-Address      OK? Method Status                Protocol
        FastEthernet0/0            unassigned      YES unset  administratively down down
        FastEthernet1/0            10.3.1.9        YES manual up                    up
        FastEthernet2/0            10.3.1.1        YES manual up                    up
        FastEthernet3/0            unassigned      YES unset  administratively down down
        ```

    * Pour router2 :

        ```
        R2#show ip int br
        Interface                  IP-Address      OK? Method Status                Protocol
        FastEthernet0/0            10.3.1.5        YES manual up                    up
        FastEthernet1/0            10.3.1.2        YES manual up                    up
        FastEthernet2/0            10.3.102.11     YES manual up                    up
        FastEthernet3/0            unassigned      YES unset  administratively down down
        ```

    * Pour router3 : 

        ```
        R3#show ip int br
        Interface                  IP-Address      OK? Method Status                Protocol
        FastEthernet0/0            10.3.1.10       YES manual up                    up
        FastEthernet1/0            10.3.101.12     YES manual up                    up
        FastEthernet2/0            10.3.1.6        YES manual up                    up
        FastEthernet3/0            unassigned      YES unset  administratively down down
        ```

        _Tous les routers arrivent à ce **ping** 🔥_

* Mise en place **OSPF** :

    * Activation **OSPF** :

        ```
        R1(config)# router ospf 1

        R2(config)# router ospf 1

        R3(config)# router ospf 1
        ```

    * Définition d'un **router-id** pour chaque routeur :

        ```
        R1(config-router)# router-id 1.1.1.1

        R2(config-router)# router-id 2.2.2.2

        R3(config-router)# router-id 3.3.3.3
        ```

    * Partage de réseaux : 

        * Exemple sur router2 : 
        
            ```
            R2(config-router)#network 10.3.102.0 0.0.0.255 area 0
            R2(config-router)#network 10.3.101.0 0.0.0.255 area 0
            R2(config-router)#network 10.3.103.0 0.0.0.255 area 0
            ```

* Ajout de gateway pour que les clients et le server se ping :

    * Sur server1 :

        ```
        GATEWAY=10.3.102.11
        ```

    * Sur client1 && client2 : 

        ```
        GATEWAY=10.3.101.12
        GATEWAY=10.3.103.12
        ```

    * Test de ping entre les machines :

        * client1 vers server1 : 

            ```
            [axel@client1 ~]$ ping 10.3.102.10
            PING 10.3.102.10(10.3.102.10) 56(84) bytes of data.
            64 bytes from 10.3.102.10: icmp_seq=1 ttl=255 time=2.62 ms
            64 bytes from 10.3.102.10: icmp_seq=2 ttl=255 time=12.2 ms

            64 bytes from 10.3.102.10: icmp_seq=3 ttl=255 time=7.15 ms
            64 bytes from 10.3.102.10: icmp_seq=4 ttl=255 time=10.6 ms
            64 bytes from 10.3.102.10: icmp_seq=5 ttl=255 time=10.4 ms
            64 bytes from 10.3.102.10: icmp_seq=6 ttl=255 time=9.60 ms
            64 bytes from 10.3.102.10: icmp_seq=7 ttl=255 time=9.14 ms
            ^C
            --- 10.3.102.10 ping statistics ---
            7 packets transmitted, 7 received, 0% packet loss, time 6017ms
            rtt min/avg/max/mdev = 2.629/8.849/12.282/2.927 ms
            ```

        * server1 vers client1 : 

            ```
            [axel@server1 ~]$ ping 10.3.101.10
            PING 10.3.101.10(10.3.101.10) 56(84) bytes of data.
            64 bytes from 10.3.101.10: icmp_seq=1 ttl=255 time=3.04 ms
            64 bytes from 10.3.101.10: icmp_seq=2 ttl=255 time=8.00 ms
            64 bytes from 10.3.101.10: icmp_seq=3 ttl=255 time=2.69 ms
            ^C
            --- 10.3.101.10 ping statistics ---
            3 packets transmitted, 3 received, 0% packet loss, time 7324ms
            rtt min/avg/max/mdev = 2.608/6.924/12.302/2.987 ms
            ```

            _Les clients et le server peuvent se **ping** 🔥_

## 3. Configuration de la NAT

* Dans notre topologie nous avons la **NAT** qui est relié au **router1**, donc tout d'abord nous allons configurer **router1** :

    * Configuration IP sur router1 : 

        ```
        R1#conf t
        Enter configuration commands, one per line.  End with CNTL/Z.
        R1(config)#interface FastEthernet0/0          
        R1(config-if)#ip address dhcp
        R1(config-if)#no shut
        ```

    * Vérification : 

        ```
        R1#show ip int br
        Interface                  IP-Address      OK? Method Status                Protocol
        FastEthernet0/0            192.168.122.251 YES DHCP   up                    up
        FastEthernet1/0            10.3.1.9        YES NVRAM  up                    up
        FastEthernet2/0            10.3.1.1        YES NVRAM  up                    up
        FastEthernet3/0            unassigned      YES NVRAM  administratively down down
        NVI0                       unassigned      NO  unset  up                    up
        ```

        _L'inferface **FastEthernet0/0** est bien configurer en **DHCP** est à récupérer une **IP**_

    * Test de ping **google** :

        ```
        R1#ping 8.8.8.8

        Type escape sequence to abort.
        Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
        !!!!!
        Success rate is 100 percent (5/5), round-trip min/avg/max = 28/52/80 ms
        ```

        _**router1** a bien accès à internet 🔥_

* Maintenant on va permettre aux deux autres routeurs d'avoir accès à internet : 

    * Sur router1 : 

        ```
        R1(config)#interface FastEthernet0/0
        R1(config-if)#ip nat outside
        R1(config-if)#exit

        R1(config)#interface FastEthernet1/0
        R1(config-if)#ip nat inside
        R1(config-if)#exit

        R1(config)#interface FastEthernet2/0
        R1(config-if)#ip nat inside
        R1(config-if)#exit

        R1(config)#ip nat inside source list 1 interface fastEthernet0/0 overload
        R1(config)#access-list 1 permit any
        ```

    * On va dire à **OSPF** que **router1** est la passerelle par défault pour accéder à internet : 

        ```
        R1(config)#router ospf 1
        R1(config-router)#default-information originate
        ```

    * Test de ping **google** : 

        * Sur client1 :

            ```
            [axel@client1 ~]$ ping 8.8.8.8
            PING 8.8.8.8(8.8.8.8) 56(84) bytes of data.
            64 bytes from 8.8.8.8: icmp_seq=1 ttl=255 time=114 ms
            64 bytes from 8.8.8.8: icmp_seq=2 ttl=255 time=117 ms
            64 bytes from 8.8.8.8: icmp_seq=3 ttl=255 time=178 ms
            ^C
            --- 8.8.8.8 ping statistics ---
            3 packets transmitted, 3 received, 0% packet loss, time 9304ms
            rtt min/avg/max/mdev = 2.859/6.220/11.678/2.393 ms
            ```

        * Sur server1 :

            ```
            [axel@server1 ~]$ ping 8.8.8.8
            PING 8.8.8.8(8.8.8.8) 56(84) bytes of data.
            64 bytes from 8.8.8.8: icmp_seq=1 ttl=255 time=120 ms
            64 bytes from 8.8.8.8: icmp_seq=2 ttl=255 time=128 ms
            64 bytes from 8.8.8.8: icmp_seq=3 ttl=255 time=143 ms
            64 bytes from 8.8.8.8: icmp_seq=4 ttl=255 time=155 ms
            64 bytes from 8.8.8.8: icmp_seq=5 ttl=255 time=168 ms
            ^C
            --- 8.8.8.8 ping statistics ---
            5 packets transmitted, 5 received, 0% packet loss, time 7439ms
            rtt min/avg/max/mdev = 2.765/6.153/12.324/2.904ms
            ```

            _Toutes nos VMs ont maintenant accès à internet 😎_

## 4. Configuration des switchs 

* Configuration du **Switch2** : 

    * Mode access entre **switch2** et **client1** / **client2** : 

        ```
        Switch2(config)#vlan 20
        Switch2(config-vlan)#name client1
        Switch2(config-vlan)#exit

        Switch2(config)#vlan 30
        Switch2(config-vlan)#name client2
        Switch2(config-vlan)#exit

        Switch2(config)#interface Ethernet0/2
        Switch2(config-if)#switchport mode access
        Switch2(config-if)#switchport access vlan 20
        Switch2(config-if)#exit

        Switch2(config)#interface Ethernet0/1
        Switch2(config-if)#switchport mode access
        Switch2(config-if)#switchport access vlan 30
        Switch2(config-if)#exit
        ```

    * Mode trunk entre **router3** et **switch2** : 

        ```
        Switch2(config)#interface Ethernet0/0
        Switch2(config-if)#switchport trunk encapsulation dot1q
        Switch2(config-if)#switchport mode trunk
        Switch2(config-if)#exit
        ```

* Pareil pour **Switch1** : 

    ```
    Switch1(config)#interface Ethernet0/1
    Switch1(config-if)#switchport mode access
    Switch1(config-if)#switchport access vlan 10
    Switch1(config-if)#exit

    Switch1(config)#interface Ethernet0/0
    Switch1(config-if)#switchport trunk encapsulation dot1q
    Switch1(config-if)#switchport mode trunk
    ```

    _Nous avons tous nos Vlans opérationnels_

* Maintenant nous allons faire de l'**inter-vlan** :

    * Sur router3 :

        D'abord nous supprimons l'**IP** que nous avons définie dans le réseau des Vms

        ```
        R3#conf t
        Enter configuration commands, one per line.  End with CNTL/Z.
        R3(config)#interface FastEthernet1/0
        R3(config-if)#no ip address
        R3(config-if)#exit
        ```

    * On crée une sous interface sur **router2** et **router3** pour chaque **vlan** :

        * Sur router3 :

            ```
            R3(config)#interface FastEthernet1/0.20
            R3(config-subif)#encap dot1Q 20 
            R3(config-subif)#ip add 10.3.101.12 255.255.255.0 
            R3(config-subif)#no shut
            R3(config-subif)#exit

            R3(config)#interface FastEthernet1/0.30
            R3(config-subif)#encap dot1Q 30
            R3(config-subif)#ip add 10.3.103.12 255.255.255.0
            R3(config-subif)#no shut
            R3(config-subif)#exit
            ```

        * Sur router2 : 

            ```
            R2(config)#interface FastEthernet2/0.10
            R2(config-subif)#encap dot1Q 10 
            R2(config-subif)#ip add 10.3.102.11 255.255.255.0
            R2(config-subif)#no shut
            R2(config-subif)#exit
            ```

_Toutes nos VMs peuvent de nouveau se ping 🔥_

## 5. Installation du service d'infra

* Installation de **nginx** sur **server1** :

    ```
    sudo yum install nginx -y
    ```

* Ouverture du port 80 en **tcp** :

    ```
    sudo firewall-cmd --add-port=80/tcp --permanent
    sudo firewall-cmd --reload
    ```

* Sur les vms clientes nous allons faire un wget du server **nginx** installer sur **server1** :

    * Exemple sur **client1** : 

        ![alt text](/TP3/screens/wget.png)

        _(Désolé pour le petit screen)_


🔥🔥 Fin du TP 🔥🔥