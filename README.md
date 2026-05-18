# 05 — Filtrage Réseau avec Access Control Lists (ACL) Cisco

## 🎯 Objectif
Mettre en place des listes de contrôle d'accès (ACL) sur des équipements Cisco afin de filtrer le trafic réseau, restreindre les accès entre VLANs et sécuriser l'infrastructure.

---

## 🏗️ Architecture

```
[VLAN 10 - Direction]  -->  ACL in  -->  [Routeur]  -->  ACL out  -->  [VLAN 30 - Serveurs]
[VLAN 20 - Employes]   -->  ACL in  -->  [Routeur]  -->  ACL out  -->  [INTERNET]
```

---

## 📋 Politique de filtrage

| Source | Destination | Port | Action |
|---|---|---|---|
| VLAN 10 (Direction) | VLAN 30 (Serveurs) | Tous | Autorisé |
| VLAN 20 (Employes) | VLAN 30 (Serveurs) | 80, 443 | Autorisé |
| VLAN 20 (Employes) | VLAN 10 (Direction) | Tous | Refusé |
| Tout | Tout | Telnet (23) | Refusé |
| Tout | Internet | 80, 443 | Autorisé |

---

## ⚙️ Configuration ACL Standard

```
! ACL standard — filtrage par IP source uniquement
Router(config)# access-list 10 permit 192.168.10.0 0.0.0.255
Router(config)# access-list 10 deny   any

! Application sur interface
Router(config)# interface gigabitEthernet 0/0.10
Router(config-if)# ip access-group 10 in
```

---

## ⚙️ Configuration ACL Etendue

```
! ACL étendue — filtrage source, destination, port, protocole
Router(config)# ip access-list extended EMPLOYES_TO_SERVEURS
Router(config-ext-nacl)# permit tcp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255 eq 80
Router(config-ext-nacl)# permit tcp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255 eq 443
Router(config-ext-nacl)# deny   ip  192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
Router(config-ext-nacl)# permit ip  any any

! Bloquer Telnet sur tous les équipements
Router(config)# ip access-list extended BLOCK_TELNET
Router(config-ext-nacl)# deny tcp any any eq 23
Router(config-ext-nacl)# permit ip any any

! Application sur interface
Router(config)# interface gigabitEthernet 0/0.20
Router(config-if)# ip access-group EMPLOYES_TO_SERVEURS in

! Sécurisation des lignes VTY
Router(config)# line vty 0 4
Router(config-line)# access-class 10 in
Router(config-line)# transport input ssh
```

---

## ⚙️ ACL Nommée — Accès Internet

```
Router(config)# ip access-list extended INTERNET_ACCESS
Router(config-ext-nacl)# permit tcp 192.168.0.0 0.0.255.255 any eq 80
Router(config-ext-nacl)# permit tcp 192.168.0.0 0.0.255.255 any eq 443
Router(config-ext-nacl)# permit udp 192.168.0.0 0.0.255.255 any eq 53
Router(config-ext-nacl)# deny   ip  any any log

Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip access-group INTERNET_ACCESS out
```

---

## ✅ Vérification

```
! Afficher les ACL
Router# show access-lists
Router# show ip access-lists EMPLOYES_TO_SERVEURS

! Vérifier les compteurs (hits)
Router# show access-lists | include matches

! Vérifier les interfaces
Router# show ip interface gigabitEthernet 0/0.20
```

---

## 🎓 Compétences acquises

- Création et application d'ACL standard et étendues
- Filtrage par protocole, port et adresse IP
- Sécurisation des accès VTY (SSH uniquement)
- Politique de sécurité inter-VLAN
- Analyse des logs et compteurs de filtrage
