# 🛡️ Conception et Sécurisation d'une Architecture Entreprise — OpenVPN, pfSense & Suricata

![Status](https://img.shields.io/badge/status-completed-brightgreen)

---

## 📋 Vue d'ensemble

Conception, déploiement et validation d'une architecture réseau d'entreprise sécurisée
sous VirtualBox, intégrant firewall, VPN, durcissement système, IDS/IPS et tests
d'intrusion pour valider chaque couche de défense.


### Composants de l'architecture

- **pfSense** : firewall/VPN assurant le filtrage du trafic et l'accès distant sécurisé via OpenVPN
- **Serveur Ubuntu Linux** : services internes (Apache, SSH, FTP)
- **Client Windows** : simule un employé distant se connectant via VPN
- **Kali Linux** : machine attaquante externe pour les tests d'intrusion

---

## 🏗️ Infrastructure

| Machine | OS | Rôle | IP |
|---|---|---|---|
| pfSense | pfSense CE 2.8.1 | Firewall + OpenVPN + IDS/IPS | LAN: 192.168.10.1 · WAN: 192.168.1.49 |
| Ubuntu Server | Ubuntu Server 24.04 | Serveur interne sécurisé | 192.168.10.10 |
| Windows Client | Windows 10 | Client OpenVPN distant | DHCP (WAN) |
| Kali Linux | Kali Linux | Machine attaquante | 192.168.1.14 |

**Réseaux** :
- LAN interne (`192.168.10.0/24`) — pfSense ↔ Ubuntu Server (Host-Only)
- WAN (Bridged) — pfSense, Kali Linux et Windows Client
- Tunnel VPN (`10.8.0.0/24`) — créé dynamiquement par OpenVPN

---


## 🔧 Détails des parties

### 1. Mise en place de l'architecture
Configuration des interfaces VirtualBox (Bridged/Host-Only), plan d'adressage IP,
configuration de pfSense, Ubuntu Server et Kali Linux, tests de connectivité entre
toutes les machines.

### 2. Déploiement OpenVPN
Création d'une Autorité de Certification interne (VPN-CA), génération du certificat
serveur, création de l'utilisateur VPN, configuration du serveur OpenVPN (Remote
Access SSL/TLS + User Auth, UDP/1194), règles firewall dédiées, export et connexion
du profil client Windows.

### 3. Durcissement du serveur Ubuntu
Mise à jour système, désactivation des services inutiles (ModemManager, multipathd,
udisks2, upower), désactivation d'IPv6, protections réseau sysctl (SYN cookies,
rp_filter, anti-redirections ICMP), pare-feu UFW (deny incoming par défaut),
sécurisation SSH (port non standard, no root login, limitation des tentatives),
politique de mots de passe PAM, protection GRUB par mot de passe.

### 4. Services sur Ubuntu Server
Déploiement et vérification d'Apache2 (service web interne), OpenSSH (administration
distante sécurisée) et vsftpd (transfert de fichiers).

### 5. Déploiement IDS/IPS avec Suricata
Installation de Suricata sur pfSense, configuration en mode **Inline IPS** sur
l'interface WAN, activation des catégories de règles Emerging Threats Open
(scan, dos, exploit), ajout de règles personnalisées pour détecter les scans Nmap,
le brute force SSH, l'ICMP flood et le SYN flood.

### 6. Simulation d'attaques depuis Kali Linux
Scan SYN (`nmap -sS`), scan de version de services (`nmap -sV`), ICMP flood
(`ping -f`), brute force SSH (Hydra). Toutes les attaques ont été soit détectées par
Suricata, soit bloquées par le firewall/UFW — validant l'efficacité de la défense en
profondeur.

### 7. Firewalling
Règles de filtrage strictes (principe du moindre privilège) sur les interfaces WAN,
LAN et OpenVPN : blocage d'ICMP et de SSH depuis le WAN, blocage explicite de l'IP
attaquante, autorisation du trafic HTTP/HTTPS/SSH uniquement via le tunnel VPN.

---

## 📊 Résultats clés

| Test | Résultat |
|---|---|
| Scans Nmap (SYN, version) | 100% détectés par Suricata |
| ICMP Flood | Bloqué — 100% packet loss côté attaquant |
| SSH depuis le WAN | Inaccessible (port filtré) |
| SSH via tunnel VPN | Accessible et fonctionnel |
| Brute force SSH (Hydra) | Connexion refusée / timeout |

---

## 🛠️ Outils utilisés

| Outil | Usage |
|---|---|
| pfSense CE | Firewall, VPN, IDS/IPS |
| OpenVPN | Tunnel VPN Client-to-Site chiffré |
| Suricata | IDS/IPS en mode inline |
| UFW | Pare-feu hôte sur Ubuntu |
| Nmap | Scan réseau et détection de services |
| Hydra | Test de brute force SSH |
| VirtualBox | Virtualisation de l'infrastructure |

---

## 📚 Références

- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [Suricata Documentation](https://suricata.readthedocs.io/)
- [OpenVPN Documentation](https://openvpn.net/community-resources/)
- [Emerging Threats Rules](https://rules.emergingthreats.net/)
- [ANSSI — Recommandations de sécurité](https://www.ssi.gouv.fr/)

---

## ⚖️ Mention légale

Ce projet a été réalisé dans un **environnement virtuel entièrement isolé**, à des fins
strictement académiques et pédagogiques. Toutes les actions ont été effectuées sur des
systèmes possédés et contrôlés par l'auteur.

**N'utilisez PAS** ces techniques sur des systèmes sans autorisation explicite.
