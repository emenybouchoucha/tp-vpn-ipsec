# 🔒 Configuration et Validation d'un VPN IPSec sous GNS3

Ce dépôt contient l'architecture réseau et l'ensemble des configurations nécessaires pour la mise en place d'un tunnel **VPN IPSec** sous **GNS3** dans le cadre du TP de **RSI** (Réseaux Informatiques et Services).

L'objectif de cette maquette est de sécuriser les échanges entre deux réseaux locaux distants à travers un réseau non de confiance (simulant Internet) en s'appuyant sur les protocoles **IKE (Phase 1)** et **IPSec ESP (Phase 2)**.

---

## 📐 Topologie Réseau

La topologie s'appuie sur des routeurs Cisco c7200 interconnectés selon le schéma suivant :

* **Réseau Local A** : `PC-A` (`192.168.1.3/24`) $\leftrightarrow$ **R1** (`192.168.1.1/24`)
* **Lien WAN 1** : **R1** (`10.1.1.2/30`) $\leftrightarrow$ **R2** (`10.1.1.1/30`) *(Cœur de réseau / Internet)*
* **Lien WAN 2** : **R2** (`10.2.2.1/30`) $\leftrightarrow$ **R3** (`10.2.2.2/30`)
* **Réseau Local C** : **R3** (`192.168.3.1/24`) $\leftrightarrow$ `PC-C` (`192.168.3.3/24`)

---

## 📋 Table d'Adressage

| Équipement | Interface | Adresse IP | Masque de sous-réseau | Passerelle par défaut |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | Fa0/0 <br> S1/0 | 192.168.1.1 <br> 10.1.1.2 | 255.255.255.0 <br> 255.255.255.252 | N/A <br> N/A |
| **R2** | S1/0 <br> S1/1 | 10.1.1.1 <br> 10.2.2.1 | 255.255.255.252 <br> 255.255.255.252 | N/A <br> N/A |
| **R3** | Fa0/0 <br> S1/0 | 192.168.3.1 <br> 10.2.2.2 | 255.255.255.0 <br> 255.255.255.252 | N/A <br> N/A |
| **PC-A** | NIC | 192.168.1.3 | 255.255.255.0 | 192.168.1.1 |
| **PC-C** | NIC | 192.168.3.3 | 255.255.255.0 | 192.168.3.1 |

---

## ⚙️ Paramètres de Sécurité

### 1. IKE Phase 1 (ISAKMP)
* **Algorithme de chiffrement** : AES
* **Hachage** : SHA-1
* **Authentification** : Clé pré-partagée (`pre-share` : `cisco`)
* **Groupe Diffie-Hellman** : DH Group 2
* **Durée de vie (Lifetime)** : 86400 secondes

### 2. IKE Phase 2 (IPSec)
* **Transform Set** : `VPN-SET` (`esp-3des esp-sha-hmac`)
* **Crypto Map** : `VPN-MAP`
* **Trafic chiffré (ACL 110)** : Flux IP entre `192.168.1.0/24` et `192.168.3.0/24`

---

## 🚀 Script de Configuration Rapide (Cisco IOS)

<details>
<summary><b>Cliquez pour afficher les commandes Cisco IOS (R1 et R3)</b></summary>

### Sur R1 :
```text
crypto isakmp enable
crypto isakmp policy 10
 encryption aes
 authentication pre-share
 group 2
 exit
crypto isakmp key cisco address 10.2.2.2

access-list 110 permit ip 192.168.
