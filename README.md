<h1 align="center">TP pfSense – DMZ &amp; Addon</h1>

<p align="center">
  <b>Auteur :</b> Charles BRIENNE &nbsp;|&nbsp;
  <b>Formation :</b> BTS SIO – option SISR &nbsp;|&nbsp;
  <b>Date :</b> 15/10/2026
</p>

---

## Sommaire

1. [Plan d'adressage](#adressage)
2. [Topologie réseau](#topologie)
3. [Mise en situation 1 : Monitoring du LAN](#situation-1)
4. [Mise en situation 2 : Accès RDP à un serveur Windows en DMZ](#situation-2)
5. [Mise en situation 3 : Accès à un serveur Apache2 en DMZ](#situation-3)

---

<a id="adressage"></a>

## 📋 Plan d'adressage

| Réseau | Équipement | Adresse IP |
|:------:|-----------|-----------|
| 🟠 WAN | pfSense (interface WAN) | `192.168.20.183` |
| 🔵 LAN | pfSense (interface LAN) | `192.168.20.1` |
| 🟢 DMZ | pfSense (interface DMZ) | `192.168.6.1` |
| 🟢 DMZ | Windows Server | `192.168.6.100` |
| 🟢 DMZ | Serveur Apache2 | `192.168.6.6` |

---

<a id="topologie"></a>

## 🗺️ Topologie réseau

L'infrastructure est virtualisée sous Proxmox et comprend trois machines virtuelles : pfSense (pare-feu), un Windows Server en DMZ et un serveur Apache2 en DMZ.

<p align="center">
  <img width="2720" height="2120" alt="schema_reseau_pfsense_avec_legende" src="https://github.com/user-attachments/assets/8cf8a302-0c87-4234-aec6-071985bc0ea8" />
  <br>
  <i>Schéma logique de l'infrastructure : zones WAN, LAN et DMZ autour du pare-feu pfSense.</i>
</p>

### Récapitulatif des redirections

| Service | Port WAN | Destination | Port local |
|---------|:--------:|-------------|:----------:|
| RDP (Windows Server) | `50000` | `192.168.6.100` | `3389` |
| HTTP (Apache2) | `8080` | `192.168.6.6` | `80` |

---

<a id="situation-1"></a>

## 📊 Mise en situation 1 : Monitoring du LAN

> 🎯 **Objectif**
>
> Permettre au DSI de superviser le trafic du réseau LAN depuis une interface web accessible via un navigateur.

### Solution retenue : Darkstat

Darkstat est un outil de capture et d'analyse du trafic réseau. Il offre :

- la capture du trafic en temps réel ;
- l'analyse des flux par adresse IP ;
- une interface web intégrée et simple d'utilisation.

### Installation

1. Se rendre dans **System > Package Manager > Available Packages**.
2. Installer le paquet `darkstat`.
3. Activer le service dans **Services > Darkstat**.

### Validation

> ✅ L'interface web est accessible à l'adresse `http://192.168.20.1:666` et affiche le trafic du LAN en temps réel.

---

<a id="situation-2"></a>

## 🖥️ Mise en situation 2 : Accès RDP à un serveur Windows en DMZ

> 🎯 **Objectif**
>
> Permettre l'accès en RDP, depuis l'extérieur, à un serveur Windows hébergé dans la DMZ.

### Configuration de la DMZ

| Élément | Valeur |
|---------|--------|
| Réseau DMZ | `192.168.6.0/24` |
| pfSense (interface DMZ) | `192.168.6.1` |
| Windows Server | `192.168.6.100` |
| Service | Bureau à distance (RDP) activé |

### Règle de NAT (redirection de port)

Menu : **Firewall > NAT > Port Forward**

| Paramètre | Valeur |
|-----------|--------|
| Interface | WAN |
| Protocole | TCP |
| Port de destination (WAN) | `50000` |
| Redirect target IP | `192.168.6.100` |
| Redirect target port | `3389` |
| Description | NAT RDP DMZ |

### Règle de pare-feu associée

La règle de pare-feu WAN est générée automatiquement lors de la création de la règle NAT (option *Add associated filter rule*). Elle autorise le flux TCP entrant vers `192.168.6.100:3389`.

### Validation

Test réalisé depuis un réseau externe au LAN et à la DMZ, avec le client Bureau à distance :

```
mstsc → 192.168.20.183:50000
```

> ✅ La connexion est établie et l'accès au serveur Windows de la DMZ est fonctionnel. La commande `ipconfig` sur le serveur confirme son adresse `192.168.6.100`.

---

<a id="situation-3"></a>

## 🌐 Mise en situation 3 : Accès à un serveur Apache2 en DMZ

> 🎯 **Objectif**
>
> Rendre accessible, depuis l'extérieur, un serveur web Apache2 situé dans la DMZ, via un port WAN dédié.

### Informations du serveur

| Élément | Valeur |
|---------|--------|
| Adresse IP (DMZ) | `192.168.6.6` |
| Service | Apache2 |
| Port local | `80` |

### Règle de NAT (redirection de port)

Menu : **Firewall > NAT > Port Forward**

| Paramètre | Valeur |
|-----------|--------|
| Interface | WAN |
| Protocole | TCP |
| Port de destination (WAN) | `8080` |
| Redirect target IP | `192.168.6.6` |
| Redirect target port | `80` |
| Description | NAT Apache DMZ |

### Règle de pare-feu associée

Comme pour le RDP, la règle WAN est générée automatiquement avec la règle NAT. Elle autorise le flux TCP entrant vers `192.168.6.6:80`.

### Validation

Test réalisé depuis un réseau externe, via un navigateur :

```
http://192.168.20.183:8080
```

> ✅ La page par défaut d'Apache2 s'affiche, ce qui confirme le bon fonctionnement de la redirection WAN → DMZ.
