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



## 2. Configuration des VLANs

# II. Manipulation simple de routeurs

## 1. Mise en place du lab

## 2. Configuration du routage statique

# III. Mise en place d'OSPF

## 1. Mise en place du lab

## 2. Configuration de OSPF

# IV. Lab Final
