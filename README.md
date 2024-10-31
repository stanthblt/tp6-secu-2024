# TP6 : Un peu de root-me

## Sommaire

  - [Sommaire](#sommaire)
  - [I. DNS Rebinding](#i-dns-rebinding)
  - [II. Netfilter erreurs courantes](#ii-netfilter-erreurs-courantes)
  - [III. ARP Spoofing Ecoute active](#iii-arp-spoofing-ecoute-active)
  - [IV. Bonus : Trafic Global System for Mobile communications](#iv-bonus--trafic-global-system-for-mobile-communications)

## I. DNS Rebinding

> [**Lien vers l'épreuve root-me.**](https://www.root-me.org/fr/Challenges/Reseau/HTTP-DNS-Rebinding)

- utilisez l'app web et comprendre à quoi elle sert
- lire le code ligne par ligne et comprendre chaque ligne
  - en particulier : comment/quand est récupérée la page qu'on demande
- se renseigner sur la technique DNS rebinding

🌞 **Write-up de l'épreuve**

🌞 **Proposer une version du code qui n'est pas vulnérable**

- les fonctionnalités doivent être maintenues
  - genre le site doit toujours marcher
  - dans sa qualité actuelle
    - on laisse donc le délire de `/admin` joignable qu'en `127.0.0.1`
    - c'est un choix effectué ça, on le remet pas en question
- mais l'app web ne doit plus être sensible à l'attaque

## II. Netfilter erreurs courantes

> [**Lien vers l'épreuve root-me.**](https://www.root-me.org/fr/Challenges/Reseau/Netfilter-erreurs-courantes)

- à chaque paquet reçu, un firewall parcourt les règles qui ont été configurées afin de savoir s'il accepte ou non le paquet
- une règle c'est genre "si un paquet vient de telle IP alors je drop"
- à chaque paquet reçu, il lit la liste des règles **de haut en bas** et dès qu'une règle match, il effectue l'action
- autrement dit, l'ordre des règles est important
- on cherche donc à match une règle qui est en ACCEPT

🌞 **Write-up de l'épreuve**

🌞 **Proposer un jeu de règles firewall**

- on doit là encore aboutir au même fonctionnalités : pas de régression
- mais la protection était voulue est vraiment mise en place (limitation du bruteforce)

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
Flag 1 = l1tter4lly_4_c4ptur3_th3_fl4g
Salt 1 = 3e082f332c43590d
Salt 2 = 6401724a0c3f4a3007644077
Password = a4e04c017bc9326e078959908b9d962ea9119ff8
```

odd-hash
```bash
odd-crack 'hex(sha1_raw($p)+sha1_raw($s.sha1_raw(sha1_raw($p))))' --salt hex:3e082f332c43590d6401724a0c3f4a3007644077 rockyou.txt a4e04c017bc9326e078959908b9d962ea9119ff8
---
Password: heyheyhey
---
```

Flag complet
```bash
l1tter4lly_4_c4ptur3_th3_fl4g:heyheyhey
```

🌞 **Proposer une configuration pour empêcher votre attaque**

- empêcher la première partie avec le Poisoning/MITM
- empêcher la seconde partie (empêcher de retrouver le password de base de données)

> *Faites vos recherches, et appelez moi pour blabla sur ce sujet :)*

## IV. Bonus : Trafic Global System for Mobile communications

> [**Lien vers l'épreuve root-me.**](https://www.root-me.org/fr/Challenges/Reseau/Trafic-Global-System-for-Mobile-communications)

⭐ **BONUS : Write-up de l'épreuve**