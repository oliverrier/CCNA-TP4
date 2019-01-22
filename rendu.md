# TP 4 - Spéléologie réseau : descente dans les couches

## Sommaire

* I. [Mise en place du lab](#i-mise-en-place-du-lab)

* II. [Spéléologie Réseau](#ii-spéléologie-réseau)
  * 1. [ARP](#1-arp)
  * 2. [Interception de trafic avec Wireshark](#2-wireshark)
    * [ARP et `ping`](#a-interception-darp-et-ping)
    * [netcat](#b-interception-dune-communication-netcat)
    * [Trafic Web (HTTP)](#c-interception-dun-trafic-http)

## I. Mise en place du lab

* **client1 ping router1.tp4 sur l'IP 10.1.0.254:**

    ````bash[iroh@client1 etc]$ ping router1
    PING router1 (10.1.0.254) 56(84) bytes of data.
    64 bytes from router1 (10.1.0.254): icmp_seq=1 ttl=64 time=0.354 ms
    64 bytes from router1 (10.1.0.254): icmp_seq=2 ttl=64 time=0.638 ms
    ^C
    --- router1 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1003ms
    rtt min/avg/max/mdev = 0.354/0.496/0.638/0.142 ms
    ```

* **server1 ping router1.tp4 sur l'IP 10.2.0.254:**
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
