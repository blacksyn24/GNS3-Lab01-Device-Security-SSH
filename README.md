# 🔒 Lab 1 — Device Security & SSH Configuration

![Cisco](https://img.shields.io/badge/Cisco-CCNA1-blue?style=for-the-badge&logo=cisco&logoColor=white)
![GNS3](https://img.shields.io/badge/GNS3-2.x-orange?style=for-the-badge&logo=gns3)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Lab](https://img.shields.io/badge/GNS3%20Series-Lab%201-purple?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Device%20Security-red?style=for-the-badge)

---

## 📋 Description

Premier lab de ma nouvelle série GNS3 couvrant l'ensemble des modules CCNA1/CCNA2.
L'objectif est de sécuriser l'accès à un routeur et deux switches Cisco
avant toute autre configuration réseau : mots de passe, bannière légale,
et authentification SSH sécurisée à la place de Telnet.

### Objectifs
- ✅ Configurer les **hostnames** sur R1 et les 2 switches
- ✅ Sécuriser l'accès **console** et **enable**
- ✅ Ajouter une **bannière MOTD** d'avertissement légal
- ✅ Générer une **clé RSA** et activer **SSH**
- ✅ Désactiver **Telnet** au profit de SSH uniquement
- ✅ Configurer **DHCP** pour deux LAN distincts

---

## 📖 Théorie

### Hiérarchie des mots de passe Cisco

password (ligne console/VTY)
→ Protège l'accès à la ligne de connexion

enable secret
→ Protège le passage en mode privilégié
→ Toujours chiffré (contrairement à enable password)

username + secret (local)
→ Authentification par utilisateur, requise pour SSH


### SSH vs Telnet

Telnet
→ Trafic en CLAIR (mots de passe visibles) ❌
→ Port 23

SSH
→ Trafic CHIFFRÉ ✅
→ Port 22
→ Nécessite : domain-name + clé RSA + username local


---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🔀 Routeur | c7200/vios | R1 | Routeur central |
| 🔀 Switch | IOSvL2 | Switchl2-1 | Accès LAN1 |
| 🔀 Switch | IOSvL2 | Switchl2-2 | Accès LAN2 |
| 💻 PC | VPCS | PC1 | Test LAN1 |
| 💻 PC | VPCS | PC2 | Test LAN2 |

---

## 🗺️ Topologie
                R1
          Gi1/0    Gi2/0
           /           \
  Switchl2-1        Switchl2-2
  Gi0/0                Gi0/0
      |                    |
    PC1                  PC2

<img width="1920" height="1080" alt="Topologie" src="https://github.com/user-attachments/assets/fe806465-4bcd-4db5-ac7c-52dea4b9c23a" />


---

## 📊 Plan d'adressage

| LAN | Réseau | Passerelle | Pool DHCP |
|-----|--------|------------|-----------|
| LAN1 | 192.168.10.0/24 | 192.168.10.1 | .11→.254 |
| LAN2 | 192.168.20.0/24 | 192.168.20.1 | .11→.254 |

---

## ⚙️ Configuration complète

### 🔧 Task 1 — Hostname + mots de passe de base

```cisco
! R1
enable
configure terminal
hostname R1
no ip domain-lookup
enable secret Cisco123!
line console 0
password Console123
login
exec-timeout 5 0
exit
end
```

```cisco
! Switchl2-1 (idem sur Switchl2-2)
enable
configure terminal
hostname Switchl2-1
no ip domain-lookup
enable secret Cisco123!
line console 0
password Console123
login
exec-timeout 5 0
exit
end
```

---

### 🔧 Task 2 — Bannière légale

```cisco
! Sur R1 et les 2 switches
configure terminal
banner motd #
ACCES RESERVE - BeninCore Solutions
Toute connexion non autorisee est interdite.
#
end
```

---

### 🔧 Task 3 — SSH (clé RSA + username local)

```cisco
! Sur R1 et les 2 switches
configure terminal
ip domain-name benincore.local
crypto key generate rsa
1024
username admin privilege 15 secret Admin123!
line vty 0 4
transport input ssh
login local
exec-timeout 5 0
exit
end
write
```

---

### 🔧 Task 4 — Interfaces + DHCP (R1)

```cisco
! R1
configure terminal
interface gigabitEthernet1/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface gigabitEthernet2/0
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool LAN1
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8
exit
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool LAN2
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8
exit
end
write
```

---

### 🔧 Task 5 — Interfaces switches (mode access)

```cisco
! Switchl2-1 et Switchl2-2
configure terminal
interface gigabitEthernet0/0
switchport mode access
no shutdown
exit
interface gigabitEthernet0/1
switchport mode access
no shutdown
exit
end
write
```

---

## 🧪 Tests finaux

```cisco
PC1> ip dhcp                     ✅ IP obtenue (192.168.10.11)
PC1> ping 192.168.10.1           ✅ passerelle LAN1
PC1> ping 192.168.20.11          ✅ inter-LAN via R1
R1# show ip ssh                  ✅ SSH actif, Telnet désactivé
```

---

## 💡 Points clés

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `enable secret` | Sécurise le mode privilégié (chiffré) |
| `crypto key generate rsa` | Génère la clé nécessaire à SSH |
| `transport input ssh` | Désactive Telnet, force SSH uniquement |
| `login local` | Authentifie via compte username local |
| `banner motd` | Affiche un avertissement légal à la connexion |

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
