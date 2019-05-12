# TP4 - Infra small/medium office

# Sommaire 

* [1. Pourquoi avoir choisi ce sujet ?](#1-pourquoi-avoir-choisi-ce-sujet-?)
* [2. Schema de la topologie](#2-schema-de-la-topologie)
* [3. Maquette GNS3](#3-maquette-gns3)
* [4. Plan d'adressage IP](#4-plan-dadressage-ip)
* [5. Plan des VLANs](#5-plan-des-vlans)
* [6. Matériel nécessaire](#6-matériel-nécessaire)
* [7. Progression](#7-progression)

## 1. Pourquoi avoir choisi ce sujet ?

Nous avons tous simplement choisis ce sujet pour mettre en pratique les notions que l'on a pu voir à travers les différents [TPs](https://github.com/QuentG/Tp-Reseau) sur une infra réelle 🔥.

## 2. Schema de la topologie

```
 RH1   RH2   RH3   impr        admin     clients  impr            clients       impr        clients    impr       clients    impr
+---+ +---+ +---+ +---+        +---+ +---+ +---+ +---+       +---+ +---+ +---+ +---+      +---+ +---+ +---+      +---+ +---+ +---+
|   | |   | |   | |   |        |   | |   | |   | |   |       |   | |   | |   | |   |      |   | |   | |   |      |   | |   | |   |
+-+-+ +--++ ++--+ +-+-+        +--++ +--++ +-+-+ +-+-+       +--++ +--++ +-+-+ +-+-+      +--++ +--++ +-+-+      +--++ +-+-+ +-+-+
  |      |   |      |             |     |    |     |            |     |    |     |           |     |    |           |    |     |
  +--+   |   |  +---+             +--+  |    |  +--+            +-+   |    |  +--+           +-+   |  +-+           +-+  |  +--+
     |   |   |  |                    |  |    |  |                 |   |    |  |                |   |  |               |  |  |
   +-+---+---+--+-+                +-+--+----+--+--+           +--+---+----+--+-+            +-+---+--+---+         +-+--+--+--+
   |    switch1   +----------------+    switch2    +-----------+     switch3    +------------+   switch4  +---------+  switch5 |
   +-------+------+                +---------------+           +----------------+            +------------+         +----------+
           |
           |
           |
           |
           |
           |
           |                +----------------+                           +---------+
   +-------+------+         |                |                           |  NAT    |
   |   switch8    +---------+    router1     +---------------------------+         |
   +-------+------+         |                |                           +---------+
           |                +----------------+
           |
           |
           |
           |
   +-------+------+                  +---------------+
   |   switch6    +------------------+   switch7     |
   +-----+---+----+                  +----+---+--+---+
         |   |                            |   |  |
         |   +--------+                   |   |  |
         |            |           +-------+   |  +--------------+
         |            |           |           |                 |
   +-----+---+  +-----+---+  +----+----+  +---+-------+  +------+----+
   |         |  |         |  |         |  |           |  |           |
   |         |  |         |  |         |  |           |  |           |
   +---------+  +---------+  +---------+  +-----------+  +-----------+

     server1      server2     server3       server4        server5

```

## 3. Maquette GNS3

![!alt text](/TP4/screens/screenGNS3.png)

* L'architecture comporte bien :

   * ~ 20 personnes pro
   * 3 RH
   * 1 admin
   * 5 serveurs
   * 5 imprimantes

## 4. Plan d'adressage IP

### Tableau résumé
Hosts | `10.3.100.0/28` |  `10.3.110.0/28` |  `10.3.120.0/26` | `10.3.130.0/28` | `10.3.140.0/29`
--- | --- | --- | --- | --- | ---
`serverN` | `10.3.100.N/28` | x | x | x | x
`imprN` | x | `10.3.110.N/28` | x | x | x
`clientN` | x | x | `10.3.120.N/26` | x | x
`RHN` | x | x | x | `10.3.130.N/28` | x
`admin` | x | x | x | x | `10.3.140.1/29`

### Tableau complet
Hosts | `10.3.100.0/28` |  `10.3.110.0/28` |  `10.3.120.0/26` | `10.3.130.0/28` | `10.3.140.0/29`
--- | --- | --- | --- | --- | ---
`server1` | `10.3.100.1/28` | x | x | x | x
`server2` | `10.3.100.2/28` | x | x | x | x
`server3` | `10.3.100.3/28` | x | x | x | x
`server4` | `10.3.100.4/28` | x | x | x | x
`server5` | `10.3.100.5/28` | x | x | x | x
`impr1` | x | `10.3.110.1/28` | x | x | x
`impr2` | x | `10.3.110.2/28` | x | x | x
`impr3` | x | `10.3.110.3/28` | x | x | x
`impr4` | x | `10.3.110.4/28` | x | x | x
`impr5` | x | `10.3.110.5/28` | x | x | x
`client1` | x | x | `10.3.120.1/26` | x | x
`client2` | x | x | `10.3.120.2/26` | x | x
`client3` | x | x | `10.3.120.3/26` | x | x
`clientN` | x | x | `10.3.120.N/26` | x | x
`RH1` | x | x | x | `10.3.130.1/28` | x
`RH2` | x | x | x | `10.3.130.2/28` | x
`RH3` | x | x | x | `10.3.130.3/28` | x
`admin` | x | x | x | x | `10.3.140.1/29`

## 5. Plan des vlans

VLANs | `VLAN 10` |  `VLAN 20` |  `VLAN 30`
--- | --- | --- | --- |
`serverN (1 à 2)` | ☑ | x | x |
`serverN (3 à 5)` | x | x | ☑ |
`imprN` | ☑ | x | x |
`clientN` | x | ☑ | x |
`RHN` | x | ☑ | x |
`admin` | x | x | ☑ |

## 6. Matériel nécessaire

* Pour réaliser cette infrastructure il faudra :

  * 8 Switchs
  * 1 routeur
  * 32 câbles
  
 
## 7. Progression

* On a définit une `IP statique` pour nos serveurs (parce qu'on ne veut pas les définir via un `DHCP`, on veut connaitre nos ips)

* Sur notre router, on a configuré un serveur `DHCP` qui va distribuer des IPs et des gateways pour nos clients, RH et imprimantes :

  * Exemple sur `impr1` :

    ```
    On demande une p au serveur dhcp pour l'imrpimante

    impr2> ip dhcp -r
    DDORA IP 10.3.110.1/28 GW 10.3.110.10
    ```

    ```
    VPCS> show ip

    NAME        : VPCS[1]
    IP/MASK     : 10.3.120.3/26 -> Distribué automatiquement grâve au DHCP 🔥
    GATEWAY     : 10.3.120.40  -> IDEM 🔥
    DNS         : 8.8.8.8
    DHCP SERVER : 10.3.120.40
    DHCP LEASE  : 86381, 86400/43200/75600
    MAC         : 00:50:79:66:68:0c
    LPORT       : 10090
    RHOST:PORT  : 127.0.0.1:10091
    MTU:        : 1500
    ```

* Mise en place des `VLANs` :

  * `Vlan10` pour les `imprimantes` / `server(1-2)` / `Vlan20` pour les `pro (clients)` / `Vlan30` pour `l'admin` et `server(3-5)` :

    ```
    IOU4#show vlan br

    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    Et1/0, Et1/2, Et1/3, Et2/0
                                                   Et2/1, Et2/2, Et2/3, Et3/0
                                                   Et3/1, Et3/2, Et3/3
    10   server-impr-network              active    Et0/0
    20   client-network                   active    Et0/1, Et0/2
    30   server-admin-network             active    Et0/3
    1002 fddi-default                     act/unsup 
    1003 token-ring-default               act/unsup 
    1004 fddinet-default                  act/unsup 
    1005 trnet-default                    act/unsup 
    ```

* Mise en place de l'`inter-vlan` sur le `router1` : 

  ```
  R1(config)#interface FastEthernet0/0.10
  R1(config-subif)#encap dot1Q 10 
  R1(config-subif)#ip add 10.3.110.10 255.255.255.240
  R1(config-subif)#no shut
  R1(config-subif)#exit

  R1(config)#interface FastEthernet0/0.20
  R1(config-subif)#encap dot1Q 20
  R1(config-subif)#ip add 10.3.120.40 255.255.255.192
  R1(config-subif)#no shut
  R1(config-subif)#exit
  ```

* Petit test de ping entre les clients / imprimantes : 

  ```
  VPCS> ping 10.3.110.1
  84 bytes from 10.3.110.1 icmp_seq=3 ttl=63 time=14.616 ms
  84 bytes from 10.3.110.1 icmp_seq=4 ttl=63 time=23.003 ms
  84 bytes from 10.3.110.1 icmp_seq=5 ttl=63 time=19.038 ms
  ```

  🔥


