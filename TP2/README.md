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

    * Sur routeur1 && routeur2 :

        ```
        [quentin@router1 ~]$ sudo sysctl -w net.ipv4.conf.all.forwarding=1
        [sudo] Mot de passe de quentin : 
        net.ipv4.conf.all.forwarding = 1
        ```

- Ajout de routes statiques
    * Sur routeur1 : 

        ```

        ```

     * Sur routeur2 : 
    
        ```

        ```
    
     * Sur client1 : 
    
        ```

        ```

     * Sur server1 : 
    
        ```

        ```

## 3. Visualisation du routage avec Whireshark

# II. NAT et services d'infra

## 1. Mise en place du NAT

## 2. DHCP server

## 3. NTP server

## 4. Web server