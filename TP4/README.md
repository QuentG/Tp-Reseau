# TP4 - Infra small/medium office

# Sommaire 

* [1. Pourquoi avoir choisi ce sujet ?](#1-pourquoi-avoir-choisi-ce-sujet)
* [2. Schema de la topologie](#2-schema-de-la-topologie)
* [3. Maquette GNS3](#3-maquette-gns3)
* [4. Plan d'adressage IP](#4-plan-dadressage-ip)
* [5. Plan des VLANs](#5-plan-des-vlans)
* [6. Matériel nécessaire](#6-matériel-nécessaire)
* [7. Progression](#7-progression)

## 1. Pourquoi avoir choisi ce sujet ?



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

- On a définit une `IP statique` pour nos serveurs (parce qu'on ne veut pas les définir via un `DHCP`, on veut connaitre nos ips)

- Sur notre router, on a configuré un serveur `DHCP` qui va distribuer des IPs pour nos clients, RH et imprimantes.
