# TP4 - Infra small/medium office

# Sommaire 

* [1. Schema de la topologie](#1-schema-de-la-topologie)
* [2. Maquette GNS3](#2-maquette-gns3)
* [3. Plan d'adressage IP](#3-plan-dadressage-ip)
* [4. Plan des VLANs](#4-plan-des-vlans)
* [5. Matériel nécessaire](#5-matériel-nécessaire)

## 1. Schema de la topologie

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

## 2. Maquette GNS3

![!alt text](/TP4/screens/screenGNS.png)

* L'architecture comporte bien :

   * ~ 20 personnes pro
   * 3 RH
   * 1 admin
   * 5 serveurs
   * 5 imprimantes

## 3. Plan d'adressage IP

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

## 4. Plan des vlans
 
   * Les imprimantes seront dans le VLAN 10
   * Les clients (pro) et les RH seront dans le VLAN 20
   * L'admin sera dans le VLAN 30

## 5. Matériel nécessaire

