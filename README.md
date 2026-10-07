# Protocole DHCP : Guide Avancé Complet

**Créé par : Hackers_tchad**

---

## Table des matières

1. [Introduction](#introduction)
2. [Historique et créateurs](#historique-et-créateurs)
3. [Qu'est-ce que DHCP ?](#quest-ce-que-dhcp-)
4. [Pourquoi utiliser DHCP ?](#pourquoi-utiliser-dhcp-)
5. [Composants architecturaux](#composants-architecturaux)
6. [Fonctionnement du protocole](#fonctionnement-du-protocole)
7. [Le processus DORA](#le-processus-dora)
8. [Types de messages DHCP](#types-de-messages-dhcp)
9. [Options DHCP](#options-dhcp)
10. [Gestion des baux (leases)](#gestion-des-baux-leases)
11. [Configuration serveur DHCP](#configuration-serveur-dhcp)
12. [Configuration client DHCP](#configuration-client-dhcp)
13. [Commandes de vérification](#commandes-de-vérification)
14. [Sécurité DHCP](#sécurité-dhcp)
15. [DHCPv6](#dhcpv6)
16. [Dépannage avancé](#dépannage-avancé)
17. [Outils et ressources](#outils-et-ressources)
18. [Livres recommandés](#livres-recommandés)
19. [RFCs officielles](#rfcs-officielles)
20. [Conclusion](#conclusion)

---

## Introduction

Le protocole DHCP, acronyme de **Dynamic Host Configuration Protocol**, est l'un des piliers fondamentaux des réseaux informatiques modernes. Sans DHCP, chaque administrateur réseau serait contraint de configurer manuellement chaque hôte : adresse IP, masque de sous-réseau, passerelle par défaut, serveurs DNS, et une multitude d'autres paramètres. DHCP automatise cette tâche, permettant ainsi le déploiement rapide et fiable de milliers, voire de millions, de périphériques sur des réseaux locaux ou étendus.

Ce guide avancé, rédigé par **Hackers_tchad**, vise à fournir une compréhension profonde du protocole DHCP. Il s'adresse aux étudiants en réseaux, aux administrateurs système, aux ingénieurs DevOps, aux professionnels de la cybersécurité, ainsi qu'à tous ceux qui souhaitent maîtriser les mécanismes d'attribution dynamique des adresses IP et de configuration réseau. Ce document dépasse largement les explications de surface : il explore l'architecture, les messages, les options, la sécurité, le dépannage, et fournit des commandes pratiques pour Linux, Windows, Cisco, et autres équipements réseau.

---

## Historique et créateurs

### Origines

Avant DHCP, le protocole **BOOTP** (Bootstrap Protocol), défini dans la RFC 951 en 1985, était utilisé pour attribuer des adresses IP aux stations de travail sans disque dur (diskless workstations). BOOTP permettait à une machine de démarrer via le réseau en obtenant une adresse IP et l'adresse d'un serveur de fichiers. Cependant, BOOTP souffrait de plusieurs limitations : il nécessitait une configuration manuelle de la correspondance entre adresses MAC et adresses IP, et il n'offrait pas de mécanisme de bail temporaire.

DHCP a été conçu pour remédier à ces limitations. Il a été développé par le groupe de travail **DHC** (Dynamic Host Configuration) de l'**IETF** (Internet Engineering Task Force). Les principaux contributeurs incluent **Ralph Droms**, professeur à l'Université Bucknell, qui est considéré comme l'un des pères fondateurs de DHCP, ainsi que **Ted Lemon** de Nominum, qui a largement contribué à la définition de DHCPv4 et DHCPv6.

### Évolution chronologique

- **1985** : Publication de BOOTP dans la RFC 951.
- **1993** : Première spécification de DHCP dans la RFC 1531, puis stabilisation dans la RFC 1541.
- **1997** : DHCPv4 est standardisé dans la RFC 2131.
- **2002** : Publication de la RFC 3315 pour DHCPv6.
- **2014** : RFC 6842 et mises à jour diverses apportant des clarifications.
- **2015** : RFC 7844 concernant la confidentialité des clients DHCP.
- **2022 et au-delà** : Poursuite de l'évolution avec des RFCs axées sur la sécurité, l'authentification, et l'intégration avec IPv6.

Aujourd'hui, DHCP est omniprésent. Il équipe les box Internet domestiques, les entreprises, les datacenters, les réseaux Wi-Fi publics, les infrastructures IoT, et bien d'autres environnements.

---

## Qu'est-ce que DHCP ?

DHCP est un protocole de la couche application du modèle OSI, qui utilise les services de transport UDP. Il permet à un serveur de distribuer automatiquement des paramètres de configuration réseau aux clients. Ces paramètres incluent principalement :

- Une adresse IPv4 ou IPv6
- Le masque de sous-réseau
- La passerelle par défaut
- Les serveurs DNS
- Le nom de domaine
- Les serveurs NTP (Network Time Protocol)
- Les serveurs TFTP (Trivial File Transfer Protocol)
- Les options personnalisées (vendor-specific options)
- La durée de validité du bail (lease time)

DHCP repose sur un modèle client-serveur. Le client, généralement une machine qui démarre, envoie une requête de diffusion (broadcast) pour découvrir les serveurs DHCP disponibles. Le serveur répond en proposant une configuration. Le client accepte cette offre, et le serveur accorde le bail.

### Ports utilisés

DHCP utilise deux ports UDP bien spécifiques :

- **Port 67 UDP** : utilisé par le serveur DHCP pour écouter les requêtes.
- **Port 68 UDP** : utilisé par le client DHCP pour écouter les réponses.

Lors du processus initial de découverte, le client n'a pas encore d'adresse IP. Il utilise donc l'adresse IP source `0.0.0.0` et l'adresse de destination `255.255.255.255` (broadcast IPv4 limité).

### Relation avec d'autres protocoles

DHCP ne fonctionne pas isolément. Il interagit étroitement avec :

- **ARP** (Address Resolution Protocol) : pour résoudre les adresses IP en adresses MAC.
- **DNS** (Domain Name System) : pour fournir les serveurs DNS et, via DDNS, mettre à jour les enregistrements DNS.
- **BOOTP** : DHCP hérite du format de message BOOTP et reste rétrocompatible.
- **IPv6** : DHCPv6 est l'équivalent de DHCP pour les réseaux IPv6, souvent complété par SLAAC.
- **RADIUS / 802.1X** : dans les environnements d'accès contrôlé, DHCP peut être intégré à des mécanismes d'authentification.

---

## Pourquoi utiliser DHCP ?

L'utilisation de DHCP présente de nombreux avantages par rapport à la configuration manuelle (aussi appelée configuration statique) :

### Réduction des erreurs humaines

La saisie manuelle des paramètres réseau est sujette aux erreurs : mauvaise adresse IP, masque incorrect, conflit d'adresses, passerelle mal configurée, etc. DHCP élimine ces risques en centralisant la configuration.

### Facilité de gestion

Un administrateur peut modifier les paramètres réseau de l'ensemble du parc depuis un seul serveur. Par exemple, changer les serveurs DNS de l'entreprise ne nécessite aucune intervention sur les postes clients.

### Mobilité des utilisateurs

Dans les environnements Wi-Fi et les bureaux nomades, les utilisateurs se déplacent constamment. DHCP leur attribue automatiquement une adresse valide sur chaque sous-réseau visité.

### Optimisation de l'utilisation des adresses IP

Dans les réseaux où tous les hôtes ne sont pas connectés en permanence, DHCP permet de réutiliser les adresses IP grâce aux baux temporaires. Cela est particulièrement utile pour les pools d'adresses limités.

### Traçabilité et journalisation

Les serveurs DHCP conservent des journaux (logs) des attributions d'adresses. Ces journaux facilitent le dépannage, l'audit de sécurité, et la conformité réglementaire.

### Support du démarrage réseau

DHCP peut fournir l'adresse d'un serveur de fichiers et le nom d'un fichier de démarrage, ce qui permet le boot réseau (PXE) des postes sans système d'exploitation local.

---

## Composants architecturaux

Une infrastructure DHCP typique comprend les éléments suivants :

### Le serveur DHCP

Le serveur DHCP est le composant central qui gère le pool d'adresses IP et les options de configuration. Il peut s'agir de logiciels tels que :

- **ISC DHCP** (Internet Systems Consortium) : l'implémentation la plus répandue sur Linux et Unix.
- **dnsmasq** : serveur DHCP et DNS léger, souvent utilisé sur les routeurs et les box Internet.
- **Kea** : successeur moderne d'ISC DHCP, développé par l'ISC.
- **Microsoft DHCP Server** : intégré à Windows Server.
- **Cisco IOS DHCP** : serveur DHCP embarqué sur les équipements Cisco.
- **RouterOS / MikroTik** : serveur DHCP intégré.
- **OpenWrt / DD-WRT** : serveur DHCP embarqué dans les firmwares personnalisés.

### Le client DHCP

Le client DHCP est un logiciel présent sur chaque hôte demandeur. Exemples :

- **dhclient** : client DHCP historique sur Linux.
- **dhcpcd** : client DHCP léger et moderne, par défaut sur de nombreuses distributions.
- **systemd-networkd** : gestionnaire réseau avec support DHCP intégré.
- **NetworkManager** : client DHCP utilisé sur de nombreux bureaux Linux.
- **Client DHCP Windows** : service intégré à tous les systèmes Windows.
- **macOS DHCP client** : intégré au système d'exploitation.

### Le relais DHCP (DHCP Relay Agent)

Le relais DHCP permet de faire transiter les requêtes DHCP entre des clients et un serveur situés sur des sous-réseaux différents. Les requêtes DHCP étant des broadcasts, elles ne traversent pas les routeurs par défaut. Un agent relais, configuré sur l'interface du routeur du sous-réseau client, encapsule les requêtes en unicast vers le serveur DHCP distant.

Exemple d'agent relais :

- **dhcrelay** (ISC)
- **Cisco ip helper-address**
- **RouterOS DHCP relay**

### Le bail (lease)

Le bail est une concession temporaire d'une adresse IP à un client. Il a une durée définie par l'administrateur. À l'expiration du bail, le client doit le renouveler ou libérer l'adresse.

### La base de données des baux

Le serveur DHCP maintient une base de données des baux attribués. Cette base contient généralement :

- L'adresse IP attribuée
- L'adresse MAC du client
- La date et l'heure de début du bail
- La durée du bail
- Le nom d'hôte du client (si fourni)
- Les options distribuées

---

## Fonctionnement du protocole

### Vue d'ensemble

Lorsqu'un client DHCP démarre sur un réseau, il suit un processus standardisé pour obtenir une configuration. Ce processus peut être décomposé en quatre phases principales, souvent désignées par l'acronyme **DORA** :

1. **Discover** : le client diffuse une requête de découverte.
2. **Offer** : le serveur propose une configuration.
3. **Request** : le client sollicite formellement l'offre.
4. **Acknowledge** : le serveur confirme l'attribution du bail.

### Diffusion initiale

Étant donné que le client n'a pas encore d'adresse IP au démarrage, il utilise le broadcast pour communiquer. Cette diffusion est limitée au segment local (sous-réseau) à moins qu'un relais DHCP ne soit configuré.

### Unicast et broadcast

Après avoir obtenu une adresse IP, le client peut communiquer en unicast avec le serveur pour renouveler son bail. Les renouvellements se produisent généralement à 50 % et 87,5 % de la durée du bail.

---

## Le processus DORA

### Étape 1 : DHCP Discover

Le client envoie un message `DHCPDISCOVER` en broadcast depuis l'adresse `0.0.0.0:68` vers `255.255.255.255:67`. Ce message contient l'adresse MAC du client, un identifiant de transaction (xid), et éventuellement des options demandées.

Exemple de contenu d'un DHCP Discover :

```
Ethernet II : src=00:11:22:33:44:55, dst=ff:ff:ff:ff:ff:ff
IP : src=0.0.0.0, dst=255.255.255.255
UDP : src=68, dst=67
DHCP :
  Message Type : Boot Request (1)
  Hardware Type : Ethernet
  Client MAC : 00:11:22:33:44:55
  Options :
    DHCP Message Type : Discover
    Parameter Request List : Subnet Mask, Router, DNS, Domain Name
    Client Identifier
```

### Étape 2 : DHCP Offer

Tout serveur DHCP recevant le Discover et disposant d'une adresse disponible répond avec un message `DHCPOFFER`. Ce message est généralement envoyé en broadcast (ou parfois en unicast si le client le supporte) et contient l'adresse IP proposée, le masque, la passerelle, les DNS, et la durée du bail.

Exemple de contenu d'un DHCP Offer :

```
IP : src=192.168.1.1, dst=255.255.255.255
UDP : src=67, dst=68
DHCP :
  Message Type : Boot Reply (2)
  Your IP Address : 192.168.1.100
  Server IP Address : 192.168.1.1
  Client MAC : 00:11:22:33:44:55
  Options :
    DHCP Message Type : Offer
    Subnet Mask : 255.255.255.0
    Router : 192.168.1.1
    DNS Servers : 192.168.1.1, 8.8.8.8
    Lease Time : 86400 secondes
    Server Identifier : 192.168.1.1
```

### Étape 3 : DHCP Request

Le client sélectionne une offre (généralement la première reçue) et envoie un message `DHCPREQUEST` en broadcast pour informer tous les serveurs de sa décision. Ce broadcast est nécessaire car plusieurs serveurs pourraient avoir envoyé des offres, et les serveurs non retenus doivent libérer les adresses réservées.

### Étape 4 : DHCP Acknowledge

Le serveur sélectionné répond avec un message `DHCPACK`, confirmant l'attribution du bail. Le client configure alors son interface réseau avec les paramètres reçus.

---

## Types de messages DHCP

Les messages DHCP sont codés à l'aide de l'option 53. Voici la liste complète des types de messages définis pour DHCPv4 :

| Valeur | Message | Description |
|--------|---------|-------------|
| 1 | DHCPDISCOVER | Requête de découverte émise par le client |
| 2 | DHCPOFFER | Offre de configuration par le serveur |
| 3 | DHCPREQUEST | Demande formelle de bail par le client |
| 4 | DHCPDECLINE | Le client décline une adresse (détectée comme déjà utilisée) |
| 5 | DHCPACK | Accusé de réception positif du serveur |
| 6 | DHCPNAK | Accusé de réception négatif du serveur |
| 7 | DHCPRELEASE | Le client libère volontairement son bail |
| 8 | DHCPINFORM | Le client demande des options sans demander d'adresse IP |

### DHCPDECLINE

Le client envoie un `DHCPDECLINE` s'il détecte que l'adresse IP qui lui a été attribuée est déjà utilisée sur le réseau, généralement via un conflit ARP. Le serveur doit alors marquer cette adresse comme potentiellement problématique.

### DHCPRELEASE

Un client envoie un `DHCPRELEASE` lorsqu'il souhaite libérer son bail de manière proactive, par exemple avant une mise hors tension planifiée ou un changement de réseau.

### DHCPINFORM

Le message `DHCPINFORM` est utilisé par un client qui possède déjà une adresse IP configurée manuellement mais qui souhaite obtenir d'autres paramètres de configuration (DNS, NTP, etc.) auprès du serveur DHCP.

### DHCPNAK

Le serveur envoie un `DHCPNAK` pour refuser une demande. Cela peut se produire si l'adresse demandée n'est plus disponible, si le client a déménagé vers un autre sous-réseau, ou si le bail a expiré.

---

## Options DHCP

Les options DHCP sont des paramètres supplémentaires transportés dans les messages DHCP. Elles sont codées selon le format TLV (Type-Length-Value). Voici un panorama des options les plus courantes :

### Options fondamentales

- **Option 1** : Subnet Mask — Masque de sous-réseau.
- **Option 3** : Router — Liste des passerelles par défaut.
- **Option 6** : Domain Name Server — Liste des serveurs DNS.
- **Option 12** : Host Name — Nom d'hôte du client.
- **Option 15** : Domain Name — Nom de domaine par défaut.
- **Option 28** : Broadcast Address — Adresse de broadcast du sous-réseau.
- **Option 33** : Static Route — Routes statiques à installer.
- **Option 42** : NTP Servers — Serveurs de temps.
- **Option 50** : Requested IP Address — Adresse IP demandée par le client.
- **Option 51** : IP Address Lease Time — Durée du bail demandée ou accordée.
- **Option 52** : Option Overload — Indique que les champs sname/file transportent des options.
- **Option 53** : DHCP Message Type — Type du message DHCP.
- **Option 54** : Server Identifier — Identifiant du serveur DHCP.
- **Option 55** : Parameter Request List — Liste des options demandées par le client.
- **Option 56** : Message — Message d'erreur textuel.
- **Option 57** : Maximum DHCP Message Size — Taille maximale acceptée.
- **Option 58** : Renewal Time Value (T1) — Temps avant renouvellement.
- **Option 59** : Rebinding Time Value (T2) — Temps avant rebinding.
- **Option 60** : Vendor Class Identifier — Identifiant du constructeur.
- **Option 61** : Client Identifier — Identifiant unique du client.
- **Option 66** : TFTP Server Name — Nom du serveur TFTP.
- **Option 67** : Bootfile Name — Nom du fichier de démarrage.
- **Option 81** : Client FQDN — Nom de domaine complet du client pour DDNS.
- **Option 119** : Domain Search — Liste de domaines de recherche DNS.
- **Option 121** : Classless Static Route — Routes statiques sans classe.
- **Option 138** : CAPWAP Access Controller Addresses — Contrôleurs Wi-Fi.
- **Option 150** : TFTP Server Address — Adresses de serveurs TFTP (Cisco).
- **Option 252** : WPAD — Proxy auto-configuration URL.

### Options personnalisées

Les administrateurs peuvent définir des options spécifiques à un constructeur. Ces options utilisent généralement des numéros entre 128 et 254, réservés à des usages privés ou spécifiques.

---

## Gestion des baux (leases)

### Principe du bail

Le bail est au cœur de DHCP. Il représente la durée pendant laquelle un client est autorisé à utiliser une adresse IP. Cette durée est définie par l'administrateur et peut varier de quelques minutes à plusieurs jours, voire des mois.

### Cycle de vie d'un bail

Un bail DHCP traverse plusieurs états :

1. **Allocated (Alloué)** : le serveur a attribué l'adresse au client.
2. **Renewal (Renouvellement)** : à 50 % de la durée du bail, le client tente de renouveler son bail auprès du serveur d'origine.
3. **Rebinding (Re liaison)** : à 87,5 % de la durée du bail, si le renouvellement a échoué, le client diffuse une requête à tous les serveurs DHCP.
4. **Expiration** : si aucun serveur ne répond, le client perd son bail et doit recommencer le processus DORA.
5. **Release** : le client libère volontairement le bail.

### Calcul des timers

Pour un bail de 24 heures (86400 secondes) :

- **T1 (Renewal)** : 50 % × 86400 = 43200 secondes (12 heures)
- **T2 (Rebinding)** : 87,5 % × 86400 = 75600 secondes (21 heures)

Ces timers peuvent être configurés explicitement via les options 58 et 59, mais par défaut ils sont calculés à partir de la durée du bail.

### Base de données des baux

Sous ISC DHCP, la base de données des baux est stockée dans un fichier tel que `/var/lib/dhcp/dhcpd.leases`. Voici un exemple d'entrée :

```
lease 192.168.1.100 {
  starts 6 2024/01/15 08:30:45;
  ends 6 2024/01/16 08:30:45;
  cltt 6 2024/01/15 08:30:45;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:11:22:33:44:55;
  uid "\001\000\021\"3DU";
  client-hostname "poste-client";
}
```

---

## Configuration serveur DHCP

### Configuration ISC DHCP sous Linux

Le serveur ISC DHCP utilise généralement deux fichiers principaux :

- `/etc/dhcp/dhcpd.conf` : fichier de configuration principal.
- `/var/lib/dhcp/dhcpd.leases` : fichier des baux.

Voici un exemple de configuration complète :

```conf
# Fichier : /etc/dhcp/dhcpd.conf

# Délaration des DNS globaux
option domain-name "hackers-tchad.local";
option domain-name-servers 192.168.1.1, 8.8.8.8;

# Durée par défaut des baux
default-lease-time 86400;
max-lease-time 172800;

# Mécanisme de mise à jour DNS (DDNS)
ddns-update-style interim;
ddns-domainname "hackers-tchad.local.";
ddns-rev-domainname "in-addr.arpa.";

# Activer les logs
log-facility local7;

# Déclaration d'un sous-réseau
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.50 192.168.1.200;
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
    option domain-name-servers 192.168.1.1, 8.8.8.8;
    option ntp-servers 192.168.1.2;
    default-lease-time 86400;
    max-lease-time 172800;
}

# Réservation d'adresse (adresse fixe)
host imprimante-reseau {
    hardware ethernet 00:11:22:33:44:66;
    fixed-address 192.168.1.10;
    option host-name "imprimante";
}

# Groupe d'options personnalisées
group {
    option domain-name "wifi.hackers-tchad.local";
    option routers 192.168.2.1;
    subnet 192.168.2.0 netmask 255.255.255.0 {
        range 192.168.2.10 192.168.2.100;
    }
}
```

### Configuration dnsmasq

`dnsmasq` est un serveur DHCP/DNS léger, très utilisé sur les routeurs domestiques et les petites infrastructures.

Exemple de configuration dans `/etc/dnsmasq.conf` :

```conf
# Interface à écouter
interface=eth0

# Plage DHCP
 dhcp-range=192.168.1.50,192.168.1.200,255.255.255.0,86400

# Passerelle par défaut
 dhcp-option=3,192.168.1.1

# Serveurs DNS
 dhcp-option=6,192.168.1.1,8.8.8.8

# Nom de domaine
 domain=hackers-tchad.local

# Réservations DHCP
 dhcp-host=00:11:22:33:44:55,poste-client,192.168.1.100,infinite
 dhcp-host=00:11:22:33:44:66,imprimante,192.168.1.10

# Activation des logs
 log-dhcp
```

### Configuration Kea

Kea est le successeur moderne d'ISC DHCP. Il utilise un fichier JSON pour sa configuration.

Exemple minimal dans `/etc/kea/kea-dhcp4.conf` :

```json
{
    "Dhcp4": {
        "interfaces-config": {
            "interfaces": [ "eth0" ]
        },
        "lease-database": {
            "type": "memfile",
            "persist": true,
            "name": "/var/lib/kea/kea-leases4.csv"
        },
        "subnet4": [
            {
                "subnet": "192.168.1.0/24",
                "pools": [
                    { "pool": "192.168.1.50 - 192.168.1.200" }
                ],
                "option-data": [
                    { "name": "routers", "data": "192.168.1.1" },
                    { "name": "domain-name-servers", "data": "192.168.1.1, 8.8.8.8" },
                    { "name": "domain-name", "data": "hackers-tchad.local" }
                ]
            }
        ]
    }
}
```

### Configuration Microsoft DHCP Server

Sous Windows Server, le rôle DHCP se configure via le gestionnaire de serveur ou PowerShell.

Création d'une étendue DHCP via PowerShell :

```powershell
# Installer le rôle DHCP
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Ajouter une étendue
Add-DhcpServerv4Scope `
  -Name "LAN Hackers_tchad" `
  -StartRange 192.168.1.50 `
  -EndRange 192.168.1.200 `
  -SubnetMask 255.255.255.0 `
  -LeaseDuration 1.00:00:00

# Configurer les options d'étendue
Set-DhcpServerv4OptionValue `
  -ScopeId 192.168.1.0 `
  -Router 192.168.1.1 `
  -DnsServer 192.168.1.1, 8.8.8.8 `
  -DnsDomain "hackers-tchad.local"

# Autoriser le serveur dans Active Directory
Add-DhcpServerInDC -DnsName "dhcp.hackers-tchad.local" -IPAddress 192.168.1.5
```

### Configuration Cisco IOS

Un routeur Cisco peut agir comme serveur DHCP :

```cisco
! Activer le service DHCP
service dhcp

! Configurer une pool DHCP
ip dhcp pool LAN_HACKERS_TCHAD
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 192.168.1.1 8.8.8.8
 domain-name hackers-tchad.local
 lease 2

! Exclure des adresses de la plage DHCP
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.1.254

! Vérifier les baux
show ip dhcp binding
show ip dhcp pool
show ip dhcp server statistics
```

### Configuration relais DHCP (DHCP Relay)

Sous Linux avec `dhcrelay` :

```bash
# Syntaxe
dhcrelay -i eth0 192.168.10.5
```

Sous Cisco avec `ip helper-address` :

```cisco
interface GigabitEthernet0/1
 ip helper-address 192.168.10.5
```

Sous RouterOS MikroTik :

```routeros
/ip dhcp-relay
add name=relay-lan interface=ether2 dhcp-server=192.168.10.5 local-address=192.168.1.1
```

---

## Configuration client DHCP

### Sous Linux avec NetworkManager

La configuration DHCP est généralement automatique. Pour forcer une interface en DHCP :

```bash
nmcli connection modify "Connexion filaire 1" ipv4.method auto
nmcli connection up "Connexion filaire 1"
```

### Sous Linux avec systemd-networkd

Fichier `/etc/systemd/network/20-wired.network` :

```ini
[Match]
Name=eth0

[Network]
DHCP=yes

[DHCPv4]
UseDNS=true
UseRoutes=true
SendHostname=true
Hostname=mon-poste
```

### Sous Linux avec dhcpcd

Fichier `/etc/dhcpcd.conf` :

```conf
interface eth0
static domain_name_servers=192.168.1.1 8.8.8.8
```

### Sous Windows

1. Ouvrir les **Paramètres réseau et Internet**.
2. Cliquer sur **Modifier les options d'adaptateur**.
3. Sélectionner l'interface, puis **Propriétés**.
4. Choisir **Protocole Internet version 4 (TCP/IPv4)**.
5. Sélectionner **Obtenir une adresse IP automatiquement** et **Obtenir les adresses des serveurs DNS automatiquement**.

En ligne de commande avec `netsh` :

```cmd
netsh interface ip set address "Wi-Fi" dhcp
netsh interface ip set dns "Wi-Fi" dhcp
```

### Sous macOS

1. Ouvrir **Préférences Système** > **Réseau**.
2. Sélectionner l'interface.
3. Dans **Configurer IPv4**, choisir **Via DHCP**.

---

## Commandes de vérification

### Linux : commandes client DHCP

#### Renouveler un bail avec dhclient

```bash
# Libérer le bail actuel
sudo dhclient -r eth0

# Demander un nouveau bail
sudo dhclient eth0

# Afficher les détails de la négociation
sudo dhclient -v eth0
```

#### Renouveler avec dhcpcd

```bash
sudo dhcpcd -n eth0
sudo dhcpcd -k eth0   # Libérer
```

#### Utiliser systemd-networkd

```bash
# Redémarrer la configuration réseau
sudo networkctl renew eth0
```

### Linux : vérifier l'adresse IP

```bash
# Commande moderne
ip addr show eth0

# Commande historique
ifconfig eth0

# Afficher les routes
ip route show

# Afficher la résolution DNS
resolvectl status
systemd-resolve --status
```

### Linux : consulter les baux locaux

```bash
# Fichier des baux de dhclient
/var/lib/dhcp/dhclient.leases

# Fichier des baux de dhcpcd
/var/lib/dhcpcd/dhcpcd-eth0.lease
```

### Linux : vérifier le serveur DHCP

```bash
# Vérifier le statut du serveur ISC DHCP
sudo systemctl status isc-dhcp-server

# Vérifier la syntaxe de la configuration
sudo dhcpd -t

# Consulter les baux attribués
sudo cat /var/lib/dhcp/dhcpd.leases

# Logs DHCP
sudo journalctl -u isc-dhcp-server -f
sudo tail -f /var/log/syslog | grep dhcpd
```

### Windows : commandes de vérification

```cmd
# Afficher la configuration réseau complète
ipconfig /all

# Libérer le bail DHCP
ipconfig /release

# Renouveler le bail DHCP
ipconfig /renew

# Afficher le cache DNS
ipconfig /displaydns

# Vider le cache DNS
ipconfig /flushdns
```

### Windows : PowerShell avancé

```powershell
# Afficher les informations DHCP d'une interface
Get-NetIPConfiguration -InterfaceAlias "Wi-Fi"

# Obtenir l'adresse du serveur DHCP
Get-NetIPAddress -InterfaceAlias "Wi-Fi" -AddressFamily IPv4

# Renouveler le bail DHCP
ipconfig /renew "Wi-Fi"

# Afficher les statistiques DHCP
Get-DhcpServerv4ScopeStatistics -ScopeId 192.168.1.0
```

### Cisco : commandes de vérification

```cisco
# Afficher les baux DHCP attribués
show ip dhcp binding

# Afficher les statistiques du serveur DHCP
show ip dhcp server statistics

# Afficher la configuration des pools
show ip dhcp pool

# Afficher les conflits d'adresses détectées
show ip dhcp conflict

# Vérifier les relais DHCP configurés
show ip interface brief | include helper
```

### MikroTik / RouterOS

```routeros
# Afficher les baux DHCP
/ip dhcp-server lease print

# Afficher les statistiques
/ip dhcp-server print

# Afficher les clients connectés
/ip dhcp-server lease print detail
```

### Analyse réseau avec tcpdump / Wireshark

```bash
# Capturer le trafic DHCP sur une interface
sudo tcpdump -i eth0 port 67 or port 68 -n

# Capturer avec plus de détails
sudo tcpdump -i eth0 -v port 67 or port 68

# Sauvegarder pour analyse Wireshark
sudo tcpdump -i eth0 -w dhcp_capture.pcap port 67 or port 68
```

Filtres Wireshark utiles pour DHCP :

```
bootp.dhcp
bootp.option.dhcp == 1   # DHCP Discover
bootp.option.dhcp == 2   # DHCP Offer
bootp.option.dhcp == 3   # DHCP Request
bootp.option.dhcp == 5   # DHCP ACK
```

---

## Sécurité DHCP

### Menaces courantes

Le protocole DHCP, par sa nature de confiance, est vulnérable à plusieurs attaques :

#### Attaque par serveur DHCP rogue

Un attaquant déploie un serveur DHCP non autorisé sur le réseau. Les clients recevant une réponse plus rapide que celle du serveur légitime obtiennent une configuration malveillante : passerelle contrôlée par l'attaqueur, DNS empoisonnés, etc.

#### Starvation DHCP

Un attaquant envoie un grand nombre de requêtes DHCP Discover avec des adresses MAC falsifiées pour épuiser le pool d'adresses disponibles. Les clients légitimes ne peuvent plus obtenir d'adresse IP.

#### Usurpation d'adresse MAC

Un attaquant usurpe l'adresse MAC d'un client légitime pour obtenir son adresse IP réservée ou son accès réseau.

#### Attaque Man-in-the-Middle via DNS

En réponse à un DHCP Discover, un serveur rogue fournit des serveurs DNS contrôlés par l'attaquant, redirigeant ainsi le trafic des victimes.

### Mécanismes de protection

#### DHCP Snooping

Le DHCP Snooping est une fonctionnalité des commutateurs qui distingue les ports de confiance (trusted) des ports non de confiance (untrusted). Seuls les ports de confiance peuvent envoyer des réponses DHCP. Les offres provenant de ports non de confiance sont rejetées.

Configuration sur un switch Cisco :

```cisco
! Activer le DHCP Snooping globalement
ip dhcp snooping

! Activer pour un VLAN
ip dhcp snooping vlan 10

! Définir un port comme de confiance
interface GigabitEthernet0/1
 ip dhcp snooping trust

! Définir un port comme non de confiance
interface GigabitEthernet0/2
 ip dhcp snooping untrusted
```

#### Dynamic ARP Inspection (DAI)

DAI valide les paquets ARP en s'appuyant sur la base de données DHCP Snooping. Il empêche l'empoisonnement ARP.

```cisco
ip arp inspection vlan 10
interface GigabitEthernet0/1
 ip arp inspection trust
```

#### IP Source Guard

IP Source Guard empêche un hôte d'utiliser une adresse IP qui ne lui a pas été attribuée par DHCP.

```cisco
interface GigabitEthernet0/2
 ip verify source port-security
```

#### Authentification DHCP

Des mécanismes d'authentification DHCP existent, notamment via l'option 90 (Authentication), mais ils sont peu déployés en pratique. L'intégration avec 802.1X et RADIUS offre une sécurisation plus robuste.

#### Segmentation VLAN et ACL

Segmenter le réseau en VLAN limite la portée d'un serveur DHCP rogue. Les ACL peuvent restreindre le trafic DHCP aux seuls serveurs autorisés.

---

## DHCPv6

### Différences avec DHCPv4

DHCPv6 est l'équivalent de DHCP pour les réseaux IPv6. Contrairement à DHCPv4, DHCPv6 ne fournit pas de masque de sous-réseau (la taille du préfixe est intégrée à l'architecture IPv6) et ne fonctionne pas en broadcast mais en multicast.

### Ports utilisés

- **Port 546 UDP** : client DHCPv6
- **Port 547 UDP** : serveur DHCPv6

### Adresses multicast

- **ff02::1:2** : All_DHCP_Relay_Agents_and_Servers
- **ff05::1:3** : All_DHCP_Servers (site-local)

### Types de messages DHCPv6

| Code | Message | Description |
|------|---------|-------------|
| 1 | SOLICIT | Requête de découverte |
| 2 | ADVERTISE | Offre du serveur |
| 3 | REQUEST | Demande formelle |
| 4 | CONFIRM | Vérification de la validité des adresses |
| 5 | RENEW | Renouvellement du bail |
| 6 | REBIND | Rebinding après échec du renouvellement |
| 7 | REPLY | Réponse du serveur |
| 8 | RELEASE | Libération du bail |
| 9 | DECLINE | Refus d'une adresse en conflit |
| 10 | RECONFIGURE | Demande de reconfiguration par le serveur |
| 11 | INFORMATION-REQUEST | Demande d'options sans adresse |
| 12 | RELAY-FORW | Message relais vers le serveur |
| 13 | RELAY-REPL | Réponse relais vers le client |

### Modes d'attribution IPv6

- **Stateful DHCPv6** : le serveur attribue des adresses IPv6 complètes et gère les baux.
- **Stateless DHCPv6** : l'adresse est obtenue via SLAAC (Stateless Address Autoconfiguration), mais le serveur fournit d'autres options (DNS, NTP, etc.).

### Configuration ISC DHCPv6

```conf
# Fichier : /etc/dhcp/dhcpd6.conf

ddns-update-style none;

subnet6 2001:db8:1::/64 {
    range6 2001:db8:1::1000 2001:db8:1::1fff;
    option dhcp6.name-servers 2001:db8:1::1, 2001:4860:4860::8888;
    option dhcp6.domain-search "hackers-tchad.local";
    default-lease-time 86400;
    max-lease-time 172800;
}
```

### Configuration Kea DHCPv6

```json
{
    "Dhcp6": {
        "interfaces-config": {
            "interfaces": [ "eth0" ]
        },
        "subnet6": [
            {
                "subnet": "2001:db8:1::/64",
                "pools": [
                    { "pool": "2001:db8:1::1000 - 2001:db8:1::1fff" }
                ],
                "option-data": [
                    { "name": "dns-servers", "data": "2001:db8:1::1" },
                    { "name": "domain-search", "data": "hackers-tchad.local" }
                ]
            }
        ]
    }
}
```

---

## Dépannage avancé

### Problème : le client n'obtient pas d'adresse IP

1. Vérifier que le serveur DHCP est actif :
   ```bash
   sudo systemctl status isc-dhcp-server
   ```
2. Vérifier que l'interface du serveur est correctement configurée.
3. S'assurer qu'il reste des adresses disponibles dans le pool.
4. Vérifier les logs du serveur pour détecter les erreurs de syntaxe ou les refus.
5. Capturer le trafic DHCP avec `tcpdump` pour observer si les messages Discover atteignent le serveur.

### Problème : conflit d'adresses IP

Un conflit survient lorsque deux hôtes utilisent la même adresse IP. Les causes possibles incluent :

- Attribution d'une adresse hors de la plage DHCP sans exclusion.
- Client déclinant une adresse déjà utilisée.
- Baux expirés mais pas libérés correctement.

Solution :

- Définir des plages DHCP cohérentes.
- Utiliser des réservations DHCP (fixed-address) pour les équipements critiques.
- Activer la détection des conflits sur le serveur.

### Problème : renouvellement de bail échoué

Vérifier la connectivité entre le client et le serveur. S'assurer que le pare-feu n'obstrue pas les ports UDP 67 et 68. Vérifier que le bail n'a pas expiré et que le serveur n'a pas été redémarré avec une configuration différente.

### Problème : relais DHCP non fonctionnel

1. Confirmer que l'adresse du serveur DHCP est correctement configurée sur le relais.
2. Vérifier la connectivité IP entre le routeur relais et le serveur DHCP.
3. S'assurer que le relais modifie correctement le champ giaddr (Gateway IP Address).
4. Capturer les paquets pour vérifier l'encapsulation.

### Outils de diagnostic

- **Wireshark** : analyse détaillée des paquets DHCP.
- **tcpdump** : capture en ligne de commande.
- **nmap** : scan réseau pour détecter les serveurs DHCP.
- **dhcping** : envoi de requêtes DHCP pour tester un serveur.
- **dhtest** : outil de test et de stress du protocole DHCP.

---

## Outils et ressources

### Outils logiciels

- **ISC DHCP** : https://www.isc.org/dhcp/
- **Kea DHCP** : https://www.isc.org/kea/
- **dnsmasq** : http://www.thekelleys.org.uk/dnsmasq/doc.html
- **Wireshark** : https://www.wireshark.org/
- **tcpdump** : https://www.tcpdump.org/
- **Packet Tracer** (Cisco) : simulation réseau.
- **GNS3** : émulation réseau avancée.
- **EVE-NG** : plateforme de virtualisation réseau.

### Ressources en ligne

- IETF DHC Working Group : https://datatracker.ietf.org/wg/dhc/
- RFC Editor : https://www.rfc-editor.org/
- Cisco Documentation : https://www.cisco.com/c/en/us/support/index.html
- Microsoft DHCP Documentation : https://docs.microsoft.com/fr-fr/windows-server/networking/technologies/dhcp/dhcp-top

### Communautés

- Reddit r/networking
- Server Fault
- Stack Overflow
- GitHub repositories d'ISC DHCP et Kea

---

## Livres recommandés

1. **"TCP/IP Illustrated, Volume 1: The Protocols"** par W. Richard Stevens et Kevin R. Fall — Référence absolue sur les protocoles TCP/IP, dont DHCP.
2. **"The TCP/IP Guide"** par Charles M. Kozierok — Guide complet et accessible.
3. **"Computer Networking: A Top-Down Approach"** par James F. Kurose et Keith W. Ross — Cours universitaire classique incluant DHCP.
4. **"Cisco CCNA Routing and Switching 200-125 Official Cert Guide"** par Wendell Odom — Pour la configuration Cisco incluant DHCP.
5. **"Network Warrior"** par Gary A. Donahue — Conseils pratiques d'administration réseau.
6. **"DHCP: A Guide to Dynamic Host Configuration Protocol"** — Ouvrages spécialisés sur le sujet.
7. **"IPv6 Essentials"** par Silvia Hagen — Pour approfondir DHCPv6 et IPv6.
8. **"Practical Packet Analysis"** par Chris Sanders — Analyse de paquets avec Wireshark.

---

## RFCs officielles

- **RFC 951** — Bootstrap Protocol (BOOTP).
- **RFC 2131** — Dynamic Host Configuration Protocol (DHCPv4).
- **RFC 2132** — DHCP Options and BOOTP Vendor Extensions.
- **RFC 3046** — DHCP Relay Agent Information Option.
- **RFC 3315** — Dynamic Host Configuration Protocol for IPv6 (DHCPv6).
- **RFC 3361** — Dynamic Host Configuration Protocol (DHCP-for-IPv4) Option for Session Initiation Protocol (SIP) Servers.
- **RFC 3633** — IPv6 Prefix Options for Dynamic Host Configuration Protocol (DHCP) version 6.
- **RFC 3927** — Dynamic Configuration of IPv4 Link-Local Addresses.
- **RFC 4242** — Information Refresh Time Option for Dynamic Host Configuration Protocol for IPv6 (DHCPv6).
- **RFC 4361** — Node-specific Client Identifiers for Dynamic Host Configuration Protocol Version Four (DHCPv4).
- **RFC 4702** — The Dynamic Host Configuration Protocol (DHCP) Client Fully Qualified Domain Name (FQDN) Option.
- **RFC 6842** — DHCPv4 Server Identifier Override Suboption.
- **RFC 7844** — Anonymity Profiles for DHCP Clients.
- **RFC 8415** — Dynamic Host Configuration Protocol for IPv6 (DHCPv6), remplaçant la RFC 3315.

---

## Cas pratiques et labs

### Lab 1 : Déploiement d'un serveur DHCP Linux

Objectif : configurer un serveur ISC DHCP sur Debian/Ubuntu.

```bash
# Installation
sudo apt update
sudo apt install isc-dhcp-server

# Configuration de l'interface à écouter
sudo nano /etc/default/isc-dhcp-server
# INTERFACESv4="eth0"

# Édition du fichier de configuration
sudo nano /etc/dhcp/dhcpd.conf

# Vérification de la syntaxe
sudo dhcpd -t

# Démarrage du service
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
```

### Lab 2 : Capture et analyse Wireshark

1. Lancer Wireshark sur le segment réseau.
2. Appliquer le filtre `bootp`.
3. Redémarrer un client DHCP et observer les messages Discover, Offer, Request, ACK.
4. Analyser les options présentes dans chaque message.

### Lab 3 : DHCP Relay inter-VLAN

1. Configurer deux VLANs sur un routeur ou un switch de couche 3.
2. Placer le serveur DHCP dans un VLAN et les clients dans un autre.
3. Configurer `ip helper-address` sur l'interface client.
4. Vérifier que les clients obtiennent une adresse du serveur distant.

### Lab 4 : Sécurisation avec DHCP Snooping

1. Activer DHCP Snooping sur un switch Cisco.
2. Configurer le port connecté au serveur DHCP comme trusted.
3. Connecter un serveur DHCP rogue sur un port untrusted.
4. Vérifier que les clients ne reçoivent pas d'offre malveillante.

### Lab 5 : DHCPv6 stateful

1. Configurer un réseau IPv6 de test.
2. Installer et configurer ISC DHCPv6 ou Kea.
3. Configurer les clients pour utiliser DHCPv6.
4. Observer les messages SOLICIT, ADVERTISE, REQUEST, REPLY.

---

## Questions fréquentes (FAQ)

### Quelle est la différence entre DHCP et BOOTP ?

BOOTP attribue des adresses IP statiques configurées manuellement dans une table. DHCP ajoute l'attribution dynamique, les baux temporaires, et une grande variété d'options de configuration.

### Peut-on avoir plusieurs serveurs DHCP sur un même réseau ?

Oui, mais cela nécessite une configuration soignée pour éviter les conflits. Les plages d'adresses doivent être disjointes. Cette architecture peut offrir de la redondance via le split-scope ou le failover DHCP.

### Quelle durée de bail choisir ?

Cela dépend du contexte. Les réseaux d'entreprise utilisent souvent des baux de 8 à 24 heures. Les réseaux Wi-Fi publics ou invités peuvent utiliser des baux courts (30 minutes à 2 heures). Les équipements fixes peuvent avoir des baux longs ou infinis.

### DHCP fonctionne-t-il avec IPv6 ?

Oui, via DHCPv6. Cependant, IPv6 privilégie souvent SLAAC pour l'attribution automatique des adresses, DHCPv6 étant utilisé pour les options complémentaires ou les adresses stateful.

### Comment sécuriser un serveur DHCP ?

Utiliser DHCP Snooping, Dynamic ARP Inspection, IP Source Guard, segmenter le réseau en VLAN, surveiller les logs, et authentifier les équipements via 802.1X lorsque possible.

---

## Annexes

### Annexe A : Tableau récapitulatif des options DHCP courantes

| Option | Nom | Utilisation |
|--------|-----|-------------|
| 1 | Subnet Mask | Masque de sous-réseau |
| 3 | Router | Passerelle par défaut |
| 6 | Domain Name Server | Serveurs DNS |
| 12 | Host Name | Nom d'hôte |
| 15 | Domain Name | Nom de domaine |
| 28 | Broadcast Address | Adresse de broadcast |
| 42 | NTP Servers | Serveurs de temps |
| 51 | IP Address Lease Time | Durée du bail |
| 53 | DHCP Message Type | Type de message |
| 54 | Server Identifier | ID du serveur |
| 55 | Parameter Request List | Options demandées |
| 58 | Renewal Time | T1 |
| 59 | Rebinding Time | T2 |
| 61 | Client Identifier | ID client |
| 66 | TFTP Server Name | Serveur TFTP |
| 67 | Bootfile Name | Fichier de boot |
| 81 | Client FQDN | Nom FQDN pour DDNS |
| 119 | Domain Search | Domaines de recherche |
| 121 | Classless Static Route | Routes sans classe |
| 150 | TFTP Server Address | Cisco VoIP/TFTP |
| 252 | WPAD | Proxy auto-config |

### Annexe B : Gestion des baux sous ISC DHCP

Commandes utiles pour manipuler les baux :

```bash
# Afficher le fichier des baux
sudo cat /var/lib/dhcp/dhcpd.leases

# Rechercher un bail par adresse MAC
sudo grep -i "00:11:22:33:44:55" /var/lib/dhcp/dhcpd.leases

# Libérer manuellement un bail (arrêter le service, éditer le fichier, redémarrer)
sudo systemctl stop isc-dhcp-server
sudo nano /var/lib/dhcp/dhcpd.leases
sudo systemctl start isc-dhcp-server
```

### Annexe C : Commandes de gestion Windows Server DHCP

```powershell
# Lister les étendues
Get-DhcpServerv4Scope

# Lister les baux actifs
Get-DhcpServerv4Lease -ScopeId 192.168.1.0

# Ajouter une réservation
Add-DhcpServerv4Reservation `
  -ScopeId 192.168.1.0 `
  -IPAddress 192.168.1.100 `
  -ClientId "00-11-22-33-44-55" `
  -Description "Poste fixe Hackers_tchad"

# Sauvegarder la configuration DHCP
Backup-DhcpServer -Path "C:\BackupDHCP"
```

### Annexe D : Configuration avancée Cisco DHCP

```cisco
! Pool avec plusieurs options
ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8 8.8.4.4
 domain-name hackers-tchad.local
 option 66 ip 192.168.20.10
 option 67 ascii pxelinux.0
 lease 0 2 0

! Débogage DHCP
debug ip dhcp server events
debug ip dhcp server packets
```

---

## Conclusion

Le protocole DHCP est un élément indispensable de toute infrastructure réseau moderne. Sa capacité à automatiser l'attribution des adresses IP et des paramètres de configuration en fait un outil puissant, mais sa simplicité apparente cache une complexité réelle : options avancées, sécurité, DHCPv6, relais, haute disponibilité, et intégration DNS.

Ce guide, élaboré par **Hackers_tchad**, propose une vue d'ensemble complète, des fondamentaux aux sujets avancés. Pour aller plus loin, il est recommandé de consulter les RFCs officielles, de pratiquer sur des labs virtuels avec GNS3 ou Packet Tracer, et de surveiller constamment l'évolution des bonnes pratiques en matière de sécurité réseau.

La maîtrise de DHCP est une compétence essentielle pour tout professionnel des réseaux. Que vous soyez administrateur système, ingénieur réseau, analyste en cybersécurité ou étudiant, la compréhension approfondie de ce protocole vous permettra de concevoir, déployer, sécuriser et dépanner des infrastructures fiables et performantes.

**Rédigé par Hackers_tchad.**

---

*Fin du document.*
