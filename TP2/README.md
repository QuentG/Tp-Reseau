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

    [Voir net2.pcap](/TP2/pcap/net2.pcap)

    ![alt text](/TP2/screens/net2.png)

    Maintenant quand l'on double clique sur la première tram par exemple de chacune des captures nous voyons que nous n'avons pas les mêmes adresses MAC.

* Trame net12 :

    ![alt text](/TP2/screens/net12_trame.png)

* Trame net2 :

    ![alt text](/TP2/screens/net2_trame.png)

    Les MAC ne sont pas les mêmes à l'entrée, le routeur s'est occupé de changer les MAC source et destination (IPv4_forwarding) ✅

# II. NAT et services d'infra

## 1. Mise en place du NAT

* Tout d'abord nous vérifions si router1 à accès à internet : 

    ```
    [quentin@router1 ~]$ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=63 time=19.7 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=63 time=45.1 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=63 time=19.4 ms
    ^C
    --- 8.8.8.8 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2005ms
    rtt min/avg/max/mdev = 19.403/28.097/45.160/12.066 ms
    ```
    _Le ping passe bien, router1 a accès à internet_

* Utilisation des zones du Firewall CentOS
    * L'on met l'interface NAT / net1 en **ZONE=public** : 

        ```
        [quentin@router1 network-scripts]$ cat ifcfg-enp0s3 | grep 'ZONE'
        ZONE=public

        [quentin@router1 network-scripts]$ cat ifcfg-enp0s8 | grep 'ZONE'
        ZONE=public
        ```

    * Et l'interface net12 en **ZONE=internal**: 

        ```
        [quentin@router1 network-scripts]$ cat ifcfg-enp0s9 | grep 'ZONE'
        ZONE=internal
        ```

* Activation de la NAT dans la zone public :

    ``` 
    [quentin@router1 network-scripts]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
    [sudo] Mot de passe de quentin : 
    success
    [quentin@router1 network-scripts]$ sudo firewall-cmd --reload
    success
    ```

* Ajout d'une route par défaut sur router2 : 

    ```
    sudo ip route add default via 10.2.12.2 dev enp0s9
    ```

    *  Ajout du DNS 

        ```
        [quentin@router2 network-scripts]# sudo vi /etc/resolv.conf
        ```
        ```
        [root@router2 network-scripts]# cat /etc/resolv.conf 
        # Generated by NetworkManager
        nameserver 8.8.8.8
        ```

    Sur client1 :

    ```
    sudo ip route add default via 10.2.1.254 dev enp0s8
    ```

    **router2** et **client1** ont accés à internet

## 2. DHCP server

* Renommage de l'hostname : 

    ```
    [quentin@client1 ~]$ echo "dhcp-server.net1.b2 | sudo tee /etc/hostname
    [quentin@client1 ~]$ sudo hostname dhcp-server.net1.b2
    ```

* Explication du dhcpd.conf
    ```
    # dhcpd.conf

    # option definitions common to all supported networks
    option domain-name "net1.tp2";

    default-lease-time 600;
    max-lease-time 7200;

    # If this DHCP server is the official DHCP server for the local
    # network, the authoritative directive should be uncommented.
    authoritative;

    # Use this to send dhcp log messages to a different log file (you also
    # have to hack syslog.conf to complete the redirection).
    log-facility local7;

    subnet 10.2.1.0 netmask 255.255.255.0 {
    range 10.2.1.50 10.2.1.70;
    option domain-name "net1.tp2";
    option routers 10.2.1.254;
    option broadcast-address 10.2.1.255;
    }
    ```

    * option domain-name = définition du nom de domaine

    * Ensuite on va retrouver le temps par défault d'un bail dhcp et le temps max

    * Ensuite on passe sur la **configuration du réseau**

    * La range correspond aux addresses IP clients dispo

    * On définit le nom de domaine / gateway /l'adresse de broadcast


* On clone la VM patron encore #client2 :

    * On lui configure l'interface en DHCP :
        ```
        [quentin@client2 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8 
        TYPE=Ethernet
        BOOTPROTO=dhcp

        NAME=enp0s8
        DEVICE=enp0s8

        ONBOOT=yes
        ```
    
    * On fait un ifdown / ifup enp0s8

    * On vérifie que l'on a bien récup une ip dans la plage d'IP définie : 
        ```
        [quentin@client2 ~]$ ip a | grep "inet "
            inet 127.0.0.1/8 scope host lo
            inet 10.2.1.50/24 brd 10.2.1.255 scope global dynamic enp0s8
        ```
    
    * On check si on a une route par défault qui c'est add : 
        ```
        [quentin@client2 ~]$ ip r s
        default via 10.2.1.254 dev enp0s8 proto static metric 100 
        10.2.1.0/24 dev enp0s8 proto kernel scope link src 10.2.1.50 metric 100
        ```
## 3. NTP server

* On ajoute le pool de serveur français dans le fichier **chrony.conf** : 
    ```
    # Replace XXX with needed server names
    server 0.europe.pool.ntp.org
    server 1.europe.pool.ntp.org
    server 2.europe.pool.ntp.org
    server 3.europe.pool.ntp.org
    ```

* On ouvre le port utilisé par NTP + reload: 
    ```
    [quentin@router1 ~]$ sudo firewall-cmd --add-port=123/udp --permanent
    [sudo] Mot de passe de quentin : 
    success
    [quentin@router1 ~]$ sudo firewall-cmd --reload
    success
    ```

* On installer chrony + start : 
    ```
    [quentin@router1 ~]$ sudo yum -y install chrony
    ```

    ```
    [quentin@router1 ~]$ sudo systemctl start chronyd
    [quentin@router1 ~]$ systemctl status chronyd
    ● chronyd.service - NTP client/server
    Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
    Active: active (running) since lun. 2019-03-04 11:18:52 CET; 2s ago
    ```

* **Etat de synchronisation NTP** : 

    * Chronyc sources **router1** :

        ```
        [quentin@router1 ~]$ chronyc sources
        210 Number of sources = 4
        MS Name/IP address         Stratum Poll Reach LastRx Last sample               
        ===============================================================================
        ^? ptbtime1.ptb.de               1   6     1     3   -939ms[ -939ms] +/-   23ms
        ^? 134-249-140-99.broadband>     2   6     3     2   -922ms[ -922ms] +/-  100ms
        ^? ip235.ip-151-80-165.eu        2   6     3     3   -928ms[ -928ms] +/-   40ms
        ^? ntp12.berlin-provider.de      2   6     3     3   -929ms[ -929ms] +/-   47ms
        ```

    * Chronyc tracking **router1** : 
        ```
        [quentin@router1 ~]$ chronyc tracking
        Reference ID    : C035676C (ptbtime1.ptb.de)
        Stratum         : 2
        Ref time (UTC)  : Mon Mar 04 10:31:49 2019
        System time     : 0.000641622 seconds fast of NTP time
        Last offset     : -0.000115246 seconds
        RMS offset      : 0.003322741 seconds
        Frequency       : 1.602 ppm fast
        Residual freq   : -0.022 ppm
        Skew            : 4.410 ppm
        Root delay      : 0.035957448 seconds
        Root dispersion : 0.001017101 seconds
        Update interval : 64.9 seconds
        Leap status     : Normal
        ```

* Exemple sur **dhcp-server** : 

    * Chronyc sources : 
        ```
        [quentin@dhcp-server ~]$ chronyc sources
        210 Number of sources = 1
        MS Name/IP address         Stratum Poll Reach LastRx Last sample               
        ===============================================================================
        ^? router1                       2   6     1     2    +76us[  +76us] +/-   19ms
        ```

        _Il passe bien par router1_

    * Chronyc tracking
        ```
        [quentin@dhcp-server ~]$ chronyc tracking
        Reference ID    : 7F7F0101 ()
        Stratum         : 10
        Ref time (UTC)  : Mon Mar 04 10:41:18 2019
        System time     : 0.000000000 seconds slow of NTP time
        Last offset     : +0.000000000 seconds
        RMS offset      : 0.000000000 seconds
        Frequency       : 0.000 ppm slow
        Residual freq   : +0.000 ppm
        Skew            : 0.000 ppm
        Root delay      : 0.000000000 seconds
        Root dispersion : 0.000000000 seconds
        Update interval : 0.0 seconds
        Leap status     : Normal
        ```

* Pour server1 : 

Sur server1 nous n'avons pas internet donc nous allons ajouter l'accès à internet : 

* Activation du masquerading dans la zone internal :
    ```
    [quentin@router1 ~]$ sudo firewall-cmd --add-masquerade --zone=internal --permanent
    [sudo] Mot de passe de quentin : 
    success
    [quentin@router1 ~]$ sudo firewall-cmd --reload
    success
    ```

    * Sur router2 :

        ```
        [root@router2 network-scripts]# sudo firewall-cmd --add-masquerade --permanent
        success
        [root@router2 network-scripts]# sudo firewall-cmd --reload
        success
        ```

    * Maintenant sur server1 on autorise le http && le https : 

        ```
        [quentin@server1 ~]$ sudo firewall-cmd --add-service=http --permanent
        success
        [quentin@server1 ~]$ sudo firewall-cmd --add-service=https --permanent
        success
        [quentin@server1 ~]$ sudo firewall-cmd --reload
        success
        ```

    * On ajouter une route par défault : 

        ``` 
        [quentin@server1 ~]$ ip r s
        default via 10.2.2.254 dev enp0s8 proto static metric 100 
        10.2.1.0/24 via 10.2.2.254 dev enp0s8 proto static metric 100 
        10.2.2.0/24 dev enp0s8 proto kernel scope link src 10.2.2.10 metric 100 
        ```

        _Et maintenant nous avons accès à internet sur server1_ 😎

## 4. Web server

* Vérification que le server web soit bien lancé : 
    ```
    [quentin@server1 ~]$ sudo ss -altnp4
    State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
    LISTEN      0      128      *:80                   *:*                   users:(("nginx",pid=2423,fd=6),("nginx",pid=2422,fd=6))
    LISTEN      0      128      *:22                   *:*                   users:(("sshd",pid=940,fd=3))
    LISTEN      0      100    127.0.0.1:25                   *:*                   users:(("master",pid=1040,fd=13))
    ```

* On va accéder depuis un client au server web avec un **curl** :  

    ```
    [quentin@dhcp-server ~]$ curl 10.2.2.10
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
        <head>
            <title>Test Page for the Nginx HTTP Server on Fedora</title>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        </head>

        <body>
            <h1>C'est la fin du <strong>TP</strong> <3 !</h1>

        </body>
    </html>
    ```