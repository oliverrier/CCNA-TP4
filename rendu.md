# TP 4 - Spéléologie réseau : descente dans les couches

## Sommaire

* I. [Mise en place du lab](#i-mise-en-place-du-lab)

* II. [Spéléologie Réseau](#ii-spéléologie-réseau)
  * 1. [ARP](#1-arp)
  * 2. [Interception de trafic avec Wireshark](#2-wireshark)
    * [ARP et `ping`](#a-interception-darp-et-ping)
    * [netcat](#b-interception-dune-communication-netcat)
    * [Trafic Web (HTTP)](#c-interception-dun-trafic-http)

**Petit tableau récapitulatif** :

Machine | `net1` | `net2`
--- | --- | ---
`client1.tp4` | `10.1.0.10` | X
`router1.tp4` | `10.1.0.254` | `10.2.0.254`
`server1.tp4` | X | `10.2.0.10`

## I. Mise en place du lab

* **`client1` ping `router1` sur l'IP 10.1.0.254:**

    ````bash[iroh@client1 etc]$ ping router1
    PING router1 (10.1.0.254) 56(84) bytes of data.
    64 bytes from router1 (10.1.0.254): icmp_seq=1 ttl=64 time=0.354 ms
    64 bytes from router1 (10.1.0.254): icmp_seq=2 ttl=64 time=0.638 ms
    ^C
    --- router1 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1003ms
    rtt min/avg/max/mdev = 0.354/0.496/0.638/0.142 ms
    ```

* **`server1` ping `router1` sur l'IP 10.2.0.254:**
    ```bash
    [iroh@server1 etc]$ ping router1
    PING router1 (10.2.0.254) 56(84) bytes of data.
    64 bytes from router1 (10.2.0.254): icmp_seq=1 ttl=64 time=0.347 ms
    64 bytes from router1 (10.2.0.254): icmp_seq=2 ttl=64 time=0.580 ms
    ^C
    --- router1 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1000ms
    rtt min/avg/max/mdev = 0.347/0.463/0.580/0.118 ms
    ```

* **`client1` ping `server1` sur l'IP 10.2.0.10:**
    ```bash
    [iroh@client1 network-scripts]$ ping server1
    PING server1 (10.2.0.10) 56(84) bytes of data.
    64 bytes from server1 (10.2.0.10): icmp_seq=1 ttl=63 time=0.594 ms
    64 bytes from server1 (10.2.0.10): icmp_seq=2 ttl=63 time=2.22 ms
    ^C
    --- server1 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1001ms
    rtt min/avg/max/mdev = 0.594/1.411/2.228/0.817 ms
    ```

* **`server1` ping `client1` sur l'IP 10.1.0.10:**
    ```bash
    [iroh@server1 ~]$ ping client1
    PING client1 (10.1.0.10) 56(84) bytes of data.
    64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.591 ms
    64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=1.67 ms
    ^C
    --- client1 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1001ms
    rtt min/avg/max/mdev = 0.591/1.131/1.672/0.541 ms
    ```

* **`server1` ping `client1` sur l'IP 10.1.0.10:**
    ```bash
    [iroh@client1 network-scripts]$ traceroute -4 server1
    traceroute to server1 (10.2.0.10), 30 hops max, 60 byte packets
    1  router1 (10.1.0.254)  0.271 ms  0.293 ms  0.250 ms
    2  server1 (10.2.0.10)  0.416 ms !X  0.275 ms !X  0.333 ms !X
    ```

## II. Spéléologie réseau

### A - Manip 1

* **Table ARP `client1`**
    ```bash
    [iroh@client1 ~]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:45 REACHABLE
    ```

    On a ici qu'une seule ligne car le client n'a pas fait de ping et n'a donc pas enregistrer les adresses du server et du router.

* **Table ARP `server1`**
    ```bash
    [iroh@server1 ~]$ ip neigh show
    10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:4a DELAY
    ```

    On a ici qu'une seule ligne car comme le client, le server n'a pas fait de ping et donc n'a pas les autres adresses dans sa table ARP.

* **Table ARP `client1` après ping**
    ```bash
    [iroh@client1 ~]$ ping server1
    PING server1 (10.2.0.10) 56(84) bytes of data.
    64 bytes from server1 (10.2.0.10): icmp_seq=1 ttl=63 time=1.22 ms
    64 bytes from server1 (10.2.0.10): icmp_seq=2 ttl=63 time=1.07 ms
    ^C
    --- server1 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1003ms
    rtt min/avg/max/mdev = 1.072/1.150/1.229/0.085 ms
    [iroh@client1 ~]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:45 REACHABLE
    10.1.0.254 dev enp0s8 lladdr 08:00:27:75:d5:29 REACHABLE
    ```

    Suite au ping, le client a enregistré l'adresse MAC du server, on a donc une nouvelle ligne dans la table ARP du client.

* **Table ARP `server1` après ping**
    ```bash
    [iroh@server1 ~]$ ip neigh show
    10.2.0.254 dev enp0s8 lladdr 08:00:27:4f:a8:f4 STALE
    10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:4a DELAY
    ```

    Suite au ping du client, le server a aussi enregistré l'adresse MAC du client, on a donc une nouvelle ligne dans la table ARP du server.

### B - Manip 2

* **Table ARP `router1`**
    ```bash
    [iroh@router1 ~]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:45 REACHABLE
    ```

    Cette ligne nous montre que le router ne connait qu'une seule adresse MAC, celle qu'il utilise pour communiquer avec l'hôte.

* **Table ARP `router1` après ping**
    ```bash
    [iroh@router1 ~]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:45 REACHABLE
    10.2.0.10 dev enp0s9 lladdr 08:00:27:55:e7:4e STALE
    10.1.0.10 dev enp0s8 lladdr 08:00:27:ef:c8:63 STALE
    ```

    Suite au ping, on voit que le `router1` à récupérer les adresses MAC de l'interface réseau du `client1` et du `server1`. En effet pour que le `client1` communique avec le `server1` il doit passer par le `router1`, ainsi le `router1` a besoin des adresses MAC du `client1` et du `server1`.

### C - Manip 3

* **Table ARP `hôte` avant flush**

    ```powershell
    PS C:\Users\Irohn> arp -a

    Interface : 10.1.0.1 --- 0x45
    Adresse Internet      Adresse physique      Type
    10.1.0.10             08-00-27-ef-c8-63     dynamique
    10.1.0.254            08-00-27-75-d5-29     dynamique
    10.1.0.255            ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique

    Interface : 10.2.0.1 --- 0x4a
    Adresse Internet      Adresse physique      Type
    10.2.0.10             08-00-27-55-e7-4e     dynamique
    10.2.0.255            ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
    ```

* **Table ARP `hôte` après flush**

    ```powershell
    PS C:\WINDOWS\system32> arp -d
    PS C:\WINDOWS\system32> arp -a

    Interface : 10.1.0.1 --- 0x45
    Adresse Internet      Adresse physique      Type
    224.0.0.22            01-00-5e-00-00-16     statique

    Interface : 10.2.0.1 --- 0x4a
    Adresse Internet      Adresse physique      Type
    224.0.0.22            01-00-5e-00-00-16     statique
    ```

* **Table ARP `hôte` après attente**

    ```powershell
    PS C:\WINDOWS\system32> arp -a

    Interface : 10.1.0.1 --- 0x45
    Adresse Internet      Adresse physique      Type
    10.1.0.255            ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique

    Interface : 10.2.0.1 --- 0x4a
    Adresse Internet      Adresse physique      Type
    10.2.0.10             08-00-27-55-e7-4e     dynamique
    10.2.0.255            ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    ```

### D - Manip 4

* **Affichage table ARP `client1`**
    ```bash
    [iroh@client1 ~]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:45 REACHABLE
    ```

* **Activation carte NAT `client1`**
    ```bash
    [iroh@client1 ~]$ sudo ifup enp0s3
    Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
    ```

* **Communication avec Internet sur `client1`**
    ```bash
    [iroh@client1 ~]$ curl google.com
    <HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
    <TITLE>301 Moved</TITLE></HEAD><BODY>
    <H1>301 Moved</H1>
    The document has moved
    <A HREF="http://www.google.com/">here</A>.
    </BODY></HTML>
    ```

* **Affichage Table ARP après curl**
    ```bash
    [iroh@client1 ~]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:45 DELAY
    10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
    ```

    La machine qui vient de pop est notre hôte car c'est celui qui nous permet de nous connecter à internet.

**Nouveau tableau récapitulatif** :

Machine | `net1` | `net2` | Adresse MAC enp0s8 | Adresse MAC enp0s9
--- | --- | --- | --- | ---
`client1.tp4` | `10.1.0.10` | X | `08:00:27:ef:c8:63` | X
`router1.tp4` | `10.1.0.254` | `10.2.0.254` |`08:00:27:75:d5:29` | `08:00:27:4f:a8:f4`
`server1.tp4` | X | `10.2.0.10` | `08:00:27:55:e7:4e` | X