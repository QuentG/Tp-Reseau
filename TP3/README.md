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

#### > Réseau(x)

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

## 2. Configuration du routage statique

# III. Mise en place d'OSPF

## 1. Mise en place du lab

## 2. Configuration de OSPF

# IV. Lab Final
