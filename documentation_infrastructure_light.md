# Documentation de l'infrastructure réseau et systèmes



**Lycée Paul-Louis Courier — Section SIO** 
**Version :** 1.0 — Juin 2026 
**Auteur :** Ewen Gadonnaud (stagiaire BTS SIO 1ère année) 
**Responsable de complétion :** M. BALNY David

> **Usage :** document de référence pour les nouveaux arrivants dans la section SIO. Aucun identifiant ou mot de passe ne doit figurer en clair — référencer uniquement le gestionnaire de secrets de la section.

---
## 1. Vue d'ensemble

![Topologie réseau](assets/maquette.png)


| Salle         | Rôle                | Équipement réseau principal                      |
| ------------- | ------------------- | ------------------------------------------------ |
| Salle 406     | Salle infra (SISR)  | HP A5120 EI (Comware 5)                          |
| Salle 407     | Salle dev (SLAM)    | FlexNetwork 5120 EI (Comware 5)                  |
| Salle 408     | Salle dev (SLAM)    | FlexNetwork 5120 EI (Comware 5)                  |
| Salle 409     | Salle infra (SISR)  | HPE 5140 EI (Comware 7)                          |
| Baie serveurs | Infrastructure prod | Aruba 3810M (cœur), cluster Proxmox, serveurs AD |

---

## 2. Plan d'adressage et VLAN

| ID VLAN       | Nom         | Usage                                        | Sous-réseau    |
| ------------- | ----------- | -------------------------------------------- | -------------- |
| 20            | VLAN-PRO    | Services de production (AD, Zabbix, Grafana) | 172.16.20.0/24 |
| 32            | VLAN-WANSIO | Supervision et gestion infrastructure        | 172.16.32.0/24 |
| _À compléter_ |             |                                              |                |

### Adresses IP des équipements

| Équipement          | Adresse IP                                   | VLAN | Rôle                             |
| ------------------- | -------------------------------------------- | ---- | -------------------------------- |
| Aruba 3810M         | 172.16.99.254                                | 99   | Commutateur cœur                 |
| HPE 5140 EI         | 172.16.99.252                                | 99   | Commutateur salle 409            |
| FlexNetwork 5120 EI | 172.16.99.253                                | 99   | Commutateur salle 407/408        |
| HP A5120 EI         | 172.16.99.251                                | 99   | Commutateur salle 406            |
| pve0 / pve1 / pve2  | 172.16.99.18<br>172.16.99.19<br>172.16.99.20 | 20   | Nœuds Proxmox VE                 |
| AD-01               | 172.16.20.21                                 | 20   | Contrôleur de domaine principal  |
| AD-02               | 172.16.20.22                                 | 20   | Contrôleur de domaine secondaire |
| PRODUCTION-ZABBIX   | 172.16.20.7                                  | 20   | Supervision Zabbix               |
| PRODUCTION-GRAFANA  | 172.16.20.6                                  | 20   | Tableaux de bord Grafana         |

---

## 3. Configurations des commutateurs

> Les mots de passe ne figurent pas ici. Se référer au gestionnaire de secrets de la section.

### Aruba 3810M — Cœur (ArubaOS-Switch)

```
! Accès SSH
ip ssh
username <NOM> privilege 15 password <voir gestionnaire>

! VLANs
vlan <ID>
   name "<NOM_VLAN>"
exit

! Port trunk vers commutateur de salle
interface <PORT>
   vlan trunk allowed <ID_VLAN_1>,<ID_VLAN_2>
exit

! Agrégation de liens (LACP)
trunk <PORTS> trk<N> lacp
interface trk<N>
   vlan trunk allowed <ID_VLAN_1>,<ID_VLAN_2>
exit
```

**Particularités :** _À compléter (ex. : ports d'agrégation, VLANs natifs…)_

---

### HPE 5140 EI / FlexNetwork 5120 EI / HP A5120 EI — Salles (Comware)

```
# Accès SSH
ssh server enable
local-user <NOM> class manage
 password hash <voir gestionnaire>
 service-type ssh
 authorization-attribute user-role network-admin

# VLAN
vlan <ID>
 name <NOM_VLAN>
quit

# Port access (poste étudiant)
interface GigabitEthernet<X/X>
 description <DESCRIPTION>
 port link-type access
 port access vlan <ID>
quit

# Port trunk (lien montant vers cœur)
interface GigabitEthernet<X/X>
 description <DESCRIPTION>
 port link-type trunk
 port trunk permit vlan <ID_VLAN_1> <ID_VLAN_2>
quit
```

---

## 4. Accès et identifiants

|Ressource|Accès|Gestionnaire de secrets|
|---|---|---|
|Switches (SSH)|`ssh admin@<IP>`|_À compléter_|
|Proxmox VE|`https://<IP_NODE>:8006`|_À compléter_|
|Zabbix|`https://172.16.20.7`|_À compléter_|
|Grafana|`https://172.16.20.6:3000`|_À compléter_|
|Active Directory|_À compléter_|_À compléter_|
|FOG|`http://<IP_FOG>/fog/management`|_À compléter_|

---

## 5. Historique des modifications

|Version|Date|Auteur|Modifications|
|---|---|---|---|
|1.0|Juin 2026|Ewen Gadonnaud|Création — version synthétique|
|||||
