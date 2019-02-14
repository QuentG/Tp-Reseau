# TP 1 - Remise dans le bain !

## Sommaire

## 1.Mise en place 

### Configuration

* Combien y a-t-il d'adresses disponibles dans un `/24` ?
    * Il y en a 254 !

* Combien y a-t-il d'adresses disponibles dans un `/30` ?
    * Il y en a 2 !

[ ] s'assurer que les 3 cartes réseaux fonctionnent :

* Carte NAT : 
    * On fait un `curl google.com` 
    * On obtient donc un retour HTML avec le code d'erreur 301 qui signifie **Moved Permanetly** qui veut dire **redirection permanente**.

* Carte net1 :
    * ping 10.1.1.1
    * `PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
        64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.037 ms`

* Carte net2 :
    * ping 10.1.2.1 
    * `PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data.
64 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.035 ms`

## 2.Basics

### Routes

* Afficher les routes :

    ```
    ip route show

    default via 10.0.2.2 dev enp0s3 proto static metric 100 
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
    10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 100 
    10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 100 
    ```
    * La première route la route par défault qui va permettre l'accès au réseau `n'importe lequel`.

    * La route route est la route via  enp0s3 (NAT) qui va permettre l'accès à `10.0.2.0/24 `.

    * La troisième route est la route via la enp0s8 (net1) qui va permettre l'accès à `10.1.1.0/24 `.

    * La quatrième route est la route via enp0s9 (net2) qui va permettre l'accès à `10.1.2.0/30 `.
  
* Supprimer une route :

    `sudo ip route del 10.1.2.0/30`

    ```
    ping 10.1.2.1

    PING 10.1.2.1 (10.1.2.1) 56(84) bytes of data.
    From 10.1.1.2 icmp_seq=1 Destination Host Unreachable
    ```
    * Elle ne fonctionne plus, on ne peut plus utiliser le reseau concerné

* Remettre une route :

    `sudo ip route add 10.1.2.0/30 via 10.1.2.2 dev enp0s9`

    ```
    ip route show 

   default via 10.0.2.2 dev enp0s3 proto static metric 100 
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
    10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 100 
    10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 100 
    ```
    * On voit bien que la route `10.1.2.0/30` c'est bien rajouter.

### Table ARP

* Afficher les voisins que connaît notre machine = la table ARP :

    ```
    ip neigh show

    10.1.1.0 dev enp0s8 lladdr 0a:00:27:00:00:06 REACHABLE
    ```

    * Cette ligne veut dire que j'ai déjà discuté avec la machine en `10.1.1.0` qui est sur le même réseau.

* Vider la table ARP :

    `sudo ip neigh flush all`

    * Quand l'on fait un `ip neigh show`il n'y a plus rien.

* Effectuer une requête simple vers l'hôte :

    `ping 10.1.1.2`

    ```
    ip neigh show

    10.1.1.0 dev enp0s8 lladdr 0a:00:27:00:00:06 REACHABLE
    ```

### Capture réseau

* On capture 10 packets

    ```
    [quentin@client1 ~]$ sudo tcpdump -i enp0s9 -w ping.pcap
    [sudo] Mot de passe de quentin : 
    tcpdump: listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
    ^C10 packets captured
    10 packets received by filter
    0 packets dropped by kernel
    ```

* Screen du Wireshark : 

    ![alt text](./whireshark.png "Whireshark")

    * La ligne **1** va être une requête ARP vers l'ip `10.1.2.1` pour lui demander son adresse MAC, vu qu'au préalable nous avions vider la table ARP.
    * La ligne **2** c'est la réponse.
    * De la ligne **3 à 10** ce sont des requêtes `ping 'pong'`.

## Communication simple entre deux machines

### 1.Mise en place







  





