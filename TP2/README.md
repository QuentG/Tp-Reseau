# TP 2 - Routage statique et services d'infra

## Sommaire

* [I. Mise en place du lab](#i-mise-en-place-du-lab)
    * [1. Création des Vms et adressage IP](#1-création-des-vms-et-adressage-ip)
    * [2. Routage statique](#2-routage-statique)
    * [3. Visualisation du routage avec Whireshark](#3-visualisation-du-routage-avec-whireshark)
* [II. NAT et services d'infra](#ii-nat-et-services-dinfra)
    * [1. Mise en place du NAT](#1-mise-en-place-du-nat)
    * [2. DHCP server](#2-dhcp-server)
    * [3. NTP server](#3-ntp-server)
    * [4. Web server](#4-web-server)

# I. Mise en place du lab

## 1. Création des Vms et adressage IP

* 4 Clones du patron de VM 
    * client1.net1.b2
        * SANS carte NAT
    * server1.net2.b2
        * SANS carte NAT
    * router1.net12.b2
        * AVEC carte NAT
    * router2.net12.b2
        * SANS carte NAT

* 3 réseaux host only 
    * net1 : ```10.2.1.0/24```
    
    * net2 : ```10.2.2.0/24```

    * net12 : ```10.2.12.0/29```

### Checklist IP VMs

_Exemple pour client1_

- Definition ip statique
    ```
    NAME=enp0s8
    DEVICE=enp0s8

    BOOTPROTO=static
    ONBOOT=yes

    IPADDR=10.2.1.10
    NETMASK=255.255.255.0
    ```

- Definition du hostname 
    ```
    sudo hostname client1
    ```

- Remplissage de /etc/hosts
    ```
    [quentin@client1 ~]$ cat /etc/hosts
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    10.2.2.10   server1 server1.net2.b2
    10.2.1.254  router1 router1.net1.b2
    10.2.2.254  router2 router2.net2.b2
    ```

## 2. Routage statique

- Activation de l'IPv4 forwarding #OnTransformeEnRouteur

    * Sur router1 && router2 :

        ```
        [quentin@router1 ~]$ sudo sysctl -w net.ipv4.conf.all.forwarding=1
        [sudo] Mot de passe de quentin : 
        net.ipv4.conf.all.forwarding = 1
        ```

- Ajout de routes statiques
    * Sur router1 : 

        ```
        [quentin@router1 network-scripts]$ cat route-enp0s9 
        10.2.2.0/24 via 10.2.12.3 dev enp0s9

        [quentin@router1 ~]$ ip route show
        default via 10.0.2.2 dev enp0s3 proto static metric 100 
        10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
        10.2.2.0/24 via 10.2.12.3 dev enp0s9 proto static metric 100 
        10.2.12.0/29 dev enp0s9 proto kernel scope link src 10.2.12.2 metric 100 
        ```

     * Sur router2 : 
    
        ```
        [quentin@router2 ~]$ cat /etc/sysconfig/network-scripts/route-enp0s9 
        10.2.1.0/24 via 10.2.12.2 dev enp0s9

        [quentin@router2 ~]$ ip r s 
        10.2.1.0/24 via 10.2.12.2 dev enp0s9 proto static metric 100 
        10.2.2.0/24 dev enp0s8 proto kernel scope link src 10.2.2.254 metric 100 
        10.2.12.0/29 dev enp0s9 proto kernel scope link src 10.2.12.3 metric 100
        ```
    
     * Sur client1 : 
    
        ```
        [quentin@client1 network-scripts]$ cat route-enp0s8 
        10.2.2.0/24 via 10.2.1.254 dev enp0s8
        ```
     * On test le ping de server1 depuis client1 :

        ```
        [quentin@client1 network-scripts]$ ping server1
        PING server1 (10.2.2.10) 56(84) bytes of data.
        64 bytes from server1 (10.2.2.10): icmp_seq=1 ttl=62 time=1.69 ms
        64 bytes from server1 (10.2.2.10): icmp_seq=2 ttl=62 time=1.31 ms
        64 bytes from server1 (10.2.2.10): icmp_seq=3 ttl=62 time=0.882 ms
        ^C
        --- server1 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2003ms
        rtt min/avg/max/mdev = 0.882/1.298/1.695/0.333 ms
        ```

     * Sur server1 : 
    
        ```
        [quentin@server1 network-scripts]$ cat route-enp0s8 
        10.2.1.0/24 via 10.2.2.254 dev enp0s8
        ```

     * On test le ping de client1 depuis server1 :

        ```
        [quentin@server1 network-scripts]$ ping client1
        PING client1 (10.2.1.10) 56(84) bytes of data.
        64 bytes from client1 (10.2.1.10): icmp_seq=1 ttl=62 time=1.21 ms
        64 bytes from client1 (10.2.1.10): icmp_seq=2 ttl=62 time=1.90 ms
        ^C
        --- client1 ping statistics ---
        2 packets transmitted, 2 received, 0% packet loss, time 1004ms
        rtt min/avg/max/mdev = 1.216/1.561/1.907/0.347 ms
        ```

## 3. Visualisation du routage avec Whireshark

* Capture sur net12 : 

    [Voir net12.pcap](/TP2/pcap/net12.pcap)

    ![alt text](/TP2/screens/net12.png)

* Capture sur net2 : 

    [Voir net12.pcap](/TP2/pcap/net2.pcap)

    ![alt text](/TP2/screens/net2.png)

# II. NAT et services d'infra



## 1. Mise en place du NAT

## 2. DHCP server

## 3. NTP server

## 4. Web server
