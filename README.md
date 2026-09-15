# Projet 1 — Maquette d'un réseau entre 3 machines

Mise en place du routage entre deux réseaux IP sous Debian, avec une machine
jouant le rôle de routeur.

## Sommaire

- [Présentation de la maquette](#présentation-de-la-maquette)
- [Plan d'adressage](#plan-dadressage)
- [Configuration des machines](#configuration-des-machines)
- [Identifier la carte NAT et la carte Host-Only](#identifier-la-carte-nat-et-la-carte-host-only)
- [Activer le routage sur B](#activer-le-routage-sur-b)

## Présentation de la maquette

Ce projet a pour but d'avoir une table de routage entre 3 machines A, B et C.

![Maquette du projet](images/01-maquette.png)

*Figure 1 — Maquette du projet : deux réseaux reliés par la machine B*

Les machines A et B sont connectées sur le réseau 1, qui est en NAT, ce qui
signifie donc qu'elles ont accès à Internet, et qui a pour adresse réseau
`192.168.10.0/24`.

Les machines B et C sont elles connectées au réseau 2, qui est en Host-Only, ce
qui signifie qu'elles communiquent uniquement entre elles et avec la machine
hôte, sans accès à Internet, et qui a pour adresse réseau `192.168.20.0/24`.

La machine B est connectée aux 2 réseaux et elle a donc 2 cartes réseau, dont
une qui est sur le R1 et l'autre sur le R2. Comme les machines A et C ne sont
pas sur le même réseau, elles ne peuvent pas communiquer directement entre
elles, elles sont obligées de passer par la machine B, car la machine B est la
seule machine présente sur les 2 réseaux. C'est pour cela que le routeur par
défaut (RPD) de A est B, et celui de C est B aussi.

## Plan d'adressage

| Machine | Interface | Réseau | Mode VMware | Adresse IP | Passerelle |
|---------|-----------|--------|-------------|------------|------------|
| A | `ens33` | R1 | NAT | `192.168.10.1/24` | `192.168.10.2` |
| B | `ens33` | R1 | NAT | `192.168.10.2/24` | — |
| B | `ens36` | R2 | Host-Only | `192.168.20.2/24` | — |
| C | `ens33` | R2 | Host-Only | `192.168.20.3/24` | `192.168.20.2` |

La machine B n'a pas de passerelle par défaut : c'est elle le routeur, elle
connaît directement les deux seuls réseaux de la maquette.

## Configuration des machines

### Configuration d'une interface

Pour configurer l'interface de notre machine, on doit mettre notre configuration
dans le fichier `interfaces`, à l'emplacement `/etc/network/interfaces`. Pour
pouvoir l'éditer, j'utilise `vi` :

```bash
vi /etc/network/interfaces
```

### Machine A

![Fichier /etc/network/interfaces de la machine A](images/02-interfaces-machine-a.png)

*Figure 2 — Fichier `/etc/network/interfaces` de la machine A*

```
auto ens33
iface ens33 inet static
    address 192.168.10.1/24
    gateway 192.168.10.2
```

On met d'abord `auto ens33`, afin qu'à chaque démarrage de la machine les
configurations d'ens33 qu'on aura faites se mettent automatiquement. `ens33` est
le nom que Linux donne à notre carte réseau ; ce nom dépend de l'emplacement de
la carte dans la machine, ce qui fait qu'il reste le même à chaque démarrage.

En faisant `ip a`, on a pu constater qu'il y a 2 interfaces, `lo` et `ens33`, ce
qui est normal car on n'a bien qu'une carte réseau, et la première interface est
toujours là : c'est notre interface de bouclage.

À la suite, on met `iface ens33 inet static` : `iface` pour notre interface
ens33, `inet` signifie qu'on va la configurer en IPv4, et `static` car on met
les adresses manuellement.

Ensuite `address 192.168.10.1/24` : on met l'adresse de notre machine, et nous
constatons qu'elle est bien sur R1 (`192.168.10.0/24`). Pour finir, on met
`gateway 192.168.10.2`, ce qui signifie qu'on a créé un chemin vers la machine
qui a cette adresse, ce qui correspond à la machine B.

### Application et vérification

Maintenant qu'on a bien mis nos configurations, on a juste à faire :

```bash
ifup ens33
```

En faisant `ip a`, cela va afficher directement nos interfaces :

![Sortie de ip a sur la machine A](images/03-ip-a-machine-a.png)

*Figure 3 — Sortie de `ip a` sur la machine A*

Mais aussi en faisant `ip route`, nous voyons directement les routes connues par
notre machine :

![Sortie de ip route sur la machine A](images/04-ip-route-machine-a.png)

*Figure 4 — Sortie de `ip route` sur la machine A*

Nous voyons clairement que la configuration de notre interface ens33 a bien été
appliquée, que maintenant l'adresse IP de notre machine est `192.168.10.1/24`,
qu'elle est bien sur le réseau R1, et aussi que son routeur par défaut est
`192.168.10.2`, qui est la machine B.

## Identifier la carte NAT et la carte Host-Only

Tout d'abord, il faut comprendre ce qu'est une carte en mode NAT et une carte en
mode Host-Only.

- **NAT** : la machine virtuelle passe par la machine hôte pour accéder à
  Internet.
- **Host-Only** : le réseau est fermé, les machines ne peuvent donc communiquer
  qu'entre elles et avec la machine hôte, sans aucune sortie vers Internet.

Donc on va vérifier quelle carte réseau aura un accès à Internet et laquelle
n'en aura pas. Pour cela, j'ai mis les deux interfaces en DHCP, afin que chacune
reçoive automatiquement une adresse du réseau sur lequel elle est déjà branchée.

```
auto ens33
iface ens33 inet dhcp

auto ens36
iface ens36 inet dhcp
```

On regarde ensuite les routes avec `ip route` :

```
default via 192.168.28.2 dev ens33 proto dhcp src 192.168.28.135 metric 1002
192.168.10.0/24 dev ens33 proto kernel scope link src 192.168.10.2
192.168.20.0/24 dev ens36 proto kernel scope link src 192.168.20.2
192.168.28.0/24 dev ens33 proto dhcp scope link src 192.168.28.135 metric 1002
192.168.139.0/24 dev ens36 proto dhcp scope link src 192.168.139.129 metric 1003
```

La carte ens33 a une route par défaut vers la passerelle du réseau NAT, tandis
qu'ens36 n'en a pas. Cela nous prouve bien qu'**ens33 est en NAT** et qu'**ens36
est en Host-Only**.

**Pourquoi ?** Car un serveur DHCP n'annonce une passerelle que s'il y a un
ailleurs où aller. Le réseau NAT mène vers Internet, il annonce donc une
passerelle. Le réseau Host-Only étant fermé, il n'a rien vers quoi router et
n'annonce donc aucune passerelle.

## Activer le routage sur B

Notre objectif, maintenant que les cartes réseau de nos 3 machines sont
configurées, est de faire en sorte qu'elles puissent s'envoyer des paquets entre
elles.

Actuellement, sans avoir fait aucun changement, un ping de A vers C ne marchera
pas : aucun de nos paquets ne sera reçu par la machine C.

```bash
sysctl net.ipv4.ip_forward
```

On voit que son statut est à 0. C'est un booléen, donc 0 pour false : nous
n'avons pas l'autorisation de faire transiter des paquets d'une interface vers
une autre.

Sans cette autorisation, A va envoyer un paquet à son routeur par défaut, qui
est sur R1 et qui est la machine B. Ce paquet sera donc reçu sur ens33. Mais ce
paquet a pour destination une machine qui est sur un autre réseau (R2), et c'est
là qu'entre en jeu `ip_forward` : il détermine si B a le droit d'envoyer ce
paquet vers son autre carte réseau (ens36), alors qu'il est arrivé sur ens33.

Du coup, avec cette autorisation, la machine B va regarder sa table de routage
et va décider d'envoyer ce paquet vers ens36, qui est elle sur R2 comme la
destination du paquet. Et ens36 va pouvoir l'envoyer à sa destination sans
souci.

Pour mettre cette autorisation à 1, et pour qu'elle reste active même en
redémarrant la machine, j'ai créé un fichier `.conf` que j'ai appelé
`99-routage.conf` dans le dossier `/etc/sysctl.d`. Les fichiers de ce dossier
sont lus dans l'ordre alphabétique, et le préfixe `99` garantit que le mien est
appliqué en dernier.

```bash
vi /etc/sysctl.d/99-routage.conf
```

```
net.ipv4.ip_forward=1
```

On applique le fichier, puis on revérifie :

```bash
sysctl -p /etc/sysctl.d/99-routage.conf
sysctl net.ipv4.ip_forward
```

La valeur est maintenant à 1 : la machine B fait bien office de routeur entre
les deux réseaux.
