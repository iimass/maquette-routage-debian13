# Projet 1 : Maquette d'un réseau entre 3 machines

## Présentation de la Maquette :

Ce projet a pour but d'avoir une table de routage entre 3 machines A, B et C

Nous savons que les machines A et B sont connectées sur le réseau 1 qui est en NAT ce qui signifie donc qu'elles ont accès à internet et qui a pour adresse réseau 192.168.10.0/24.

Nous avons aussi les machines B et C qui sont elles connectées au réseau 2 qui est en Host Only ce qui signifie qu'elles communiquent uniquement entre elles et avec la machine hôte sans accès à internet et qui a pour adresse réseau 192.168.20.0/24.

La machine B est connectée aux 2 réseaux et elle a donc 2 cartes réseau dont une qui est sur le R1 et l'autre sur le R2. Comme nous avons les machines A et C qui ne sont pas sur le même réseau elles ne peuvent pas communiquer directement entre elles, elles sont obligées de passer par la machine B car la machine B est la seule machine présente sur les 2 réseaux. C'est pour cela que le RPD (Routeur Par Défaut) de A est B et celui de C est B aussi.

![Maquette du réseau](01-maquette.png)

## Configuration des 3 machines :

### Configuration d'une interface :

Alors pour configurer l'interface de notre machine on doit mettre notre configuration dans le fichier interfaces à l'emplacement `/etc/network/interfaces` et pour pouvoir l'éditer j'utilise vi (vim),
donc je tape `vi /etc/network/interfaces` et là je peux ajouter mes configurations que je souhaite mettre.

### Pour la machine A :

on mettra d'abord dans nos configurations `auto ens33` afin qu'à chaque démarrage de la machine les configurations ens33 qu'on aura faites se mettront automatiquement, ens33 est le nom que Linux donne à notre carte réseau, ce nom dépend du slot de la carte dans la machine. En faisant `ip a` on a pu constater qu'il y a 2 interfaces le lo et le ens33 ce qui est normal car on a bien qu'une carte réseau et la première interface est toujours là, c'est notre interface de bouclage. A la suite on met `iface ens33 inet static`, iface pour notre interface ens33, inet signifie qu'on va le configurer en ipv4 et le static car on les met manuellement.

Ensuite `address 192.168.10.1/24` on met l'adresse de notre machine nous constatons qu'elle est bien sur R1 (192.168.10.0/24), et pour finir on met `gateway 192.168.10.2` ce qui signifie qu'on a créé un chemin vers une machine qui a cette adresse ce qui correspond à la machine B

![Fichier interfaces de la machine A](02-interfaces-machine-a.png)

Maintenant qu'on a bien mis nos configurations, on a juste à faire `ifup ens33` pour bien exécuter nos configurations de notre interface ens33.

En faisant `ip a`, cela va afficher directement nos interfaces :

![Sortie de ip a](03-ip-a-machine-a.png)

Mais aussi en faisant `ip route`, nous voyons directement les routes de notre machine vers d'autres machines dans notre cas par exemple :

![Sortie de ip route](04-ip-route-machine-a.png)

Nous voyons clairement que la configuration de notre interface ens33 a bien été appliquée, que maintenant l'adresse ip de notre machine est 192.168.10.1/24, qu'elle est bien sur le réseau et que son routeur par défaut est bien 192.168.10.2 qui est la machine B.

### Pour la machine B :

Comme vu dans notre maquette la machine B est sur 2 réseaux à la fois et c'est complètement normal car elle a 2 cartes réseau, la première ens33 qui elle est connectée au R1 et comme R1 est en NAT cette carte réseau l'est aussi, et notre deuxième carte réseau ens36 est connectée à R2 et comme R2 est en Host Only cette carte réseau l'est aussi.

Tout d'abord on commence par la configuration de la première carte réseau qui est ens33, on fait comme tout à l'heure, on fait `auto ens33` pour qu'au démarrage on ait bien nos configurations, ensuite on fait `iface ens33 inet static` afin de configurer notre interface ens33 avec nos configurations, on met `address 192.168.10.2/24`, et on remarque bien que cette carte réseau est connectée à R1 car le troisième chiffre de son adresse, 10, correspond à notre réseau 1 (192.168.10.0/24), et le dernier chiffre, 2, correspond à l'ip de la machine B sur ce réseau.

Maintenant on fait la même chose mais pour ens36, on fait `auto ens36`, ensuite `iface ens36 inet static`, `address 192.168.20.2/24`, et on remarque que cette carte réseau est bien connectée à R2 car le réseau 2 a pour adresse réseau 192.168.20.0/24.

![Fichier interfaces de la machine B](05-interfaces-machine-b.png)

Comme vous l'avez pu remarquer on n'a pas eu besoin de faire le RPD (gateway) car ce sont les machines A et C qui auront leur route par défaut vers B donc quand on fait ce gateway le chemin est créé et donc pas besoin de le recréer en B.

Maintenant qu'on a bien mis nos configurations on a juste à faire `ifup ens33` pour bien exécuter nos configurations de notre interface ens33 et aussi `ifup ens36` pour bien exécuter nos configurations de notre interface ens36.

En faisant `ip a`, cela va afficher directement nos interfaces :

![Sortie de ip a](06-ip-a-machine-b.png)

Mais aussi en faisant `ip route`, nous voyons directement les routes reliées à notre machine :

![Sortie de ip route](07-ip-route-machine-b.png)

On peut voir que notre première carte réseau ens33, connectée en R1, et notre deuxième carte réseau ens36, connectée en R2, sont toutes les deux en `proto kernel scope link` : le noyau les a créées tout seul à partir des adresses en /24, et le `scope link` signifie que les machines de ces réseaux sont directement joignables, donc la machine A par ens33 et la machine C par ens36. Donc à partir de B on peut parler à ces deux machines.

### Pour la machine C :

Pour la machine C on va faire pareil que pour la machine A, juste maintenant nous sommes sur R2 et la machine B est aussi sur R2 et son RPD sera donc la machine B. Dans l'interface on met donc `auto ens33` comme d'habitude et ensuite `iface ens33 inet static`, ça reste toujours la même interface ens33 qui est notre carte réseau.

Là on met `address 192.168.20.3/24` vu que notre machine est bien sur le réseau 192.168.20.0/24 (R2), et pour le gateway on met `192.168.20.2` ce qui correspond à la machine B qui est notre RPD pour cette machine aussi.

![Fichier interfaces de la machine C](08-interfaces-machine-c.png)

Une fois qu'on a fait `ifup ens33` pour exécuter nos configurations, en faisant `ip a` on voit bien que ça a été appliqué et que notre interface ens33 a maintenant bien cette adresse :

![Sortie de ip a](09-ip-a-machine-c.png)

Et en faisant `ip route` ça nous montre les routes que notre machine a vers les autres machines dans notre cas :

![Sortie de ip route](10-ip-route-machine-c.png)

Donc pour conclure on voit bien que les configurations de notre interface ens33 sont bonnes, que l'adresse ip est bien 192.168.20.3/24 et que son routeur par défaut est bien 192.168.20.2 qui correspond à la machine B.

## Question : comment savoir quelle carte réseau de B est en NAT et laquelle est en Host-Only ?

> Comment pouvez-vous savoir quelle carte réseau Linux de B est la carte en mode HostOnly et quelle carte réseau Linux de B est en mode NAT ? Ainsi, si les 2 cartes vues par Linux sont appelées ens33 et ens37 (le nom dépend de votre version de VMware), comment savoir si c'est ens33 qui est en NAT ou si c'est ens37 ?

Ma méthode pour vérifier quelle carte est en mode NAT et l'autre en Host Only :

Alors tout d'abord il faut comprendre le raisonnement de ce qu'est une carte en mode NAT et Host Only.

- **NAT** : la machine virtuelle passe par la machine hôte pour accéder à Internet.
- **Host-Only** : le réseau est FERMÉ, les machines ne peuvent donc communiquer qu'entre elles et avec la machine hôte, sans aucune sortie vers Internet.

Donc on va vérifier quelle carte réseau aura un accès à Internet et l'autre non. Pour cela j'ai changé la configuration de leur interface, je les ai mises en DHCP afin que chacune reçoive automatiquement une adresse du réseau sur lequel elle est déjà branchée.

![Les 2 cartes de B en DHCP](11-interfaces-dhcp-machine-b.png)

Comme on peut le voir par ici j'ai directement changé l'interface de nos 2 cartes réseau et je les ai bien configurées en DHCP, et avec cela on saura laquelle est en NAT et l'autre en Host Only.

Maintenant qu'on a bien changé la configuration et que nos 2 cartes réseau sont bien en DHCP, on regarde avec `ip route` le chemin de ces cartes réseau et avec cela on verra laquelle est connectée directement à internet.

![Sortie de ip route avec le DHCP](12-ip-route-dhcp-machine-b.png)

Comme on peut le voir juste ici notre carte réseau ens33 a un RPD vers le chemin du réseau NAT, tandis que ens36 n'en a pas. Donc cela nous prouve bien que ens33 est bien en NAT et que ens36 est en Host Only.

Pourquoi ? Car le serveur DHCP du réseau Host Only ne donne aucune passerelle vu qu'il n'y a aucun routeur sur ce réseau : comme il est fermé, il n'a nulle part où aller. Alors que le serveur DHCP du réseau NAT donne bien l'adresse de la passerelle NAT, car c'est par elle qu'on sort vers Internet.

## Pouvoir faire en sorte que A puisse faire un ping vers C :

Notre objectif ici, maintenant qu'on a configuré nos cartes réseau de nos 3 machines, est de pouvoir faire en sorte qu'elles puissent s'envoyer des paquets entre elles. Actuellement, sans avoir fait aucun changement, si on souhaite depuis la machine A faire un ping à la machine C, ça ne marchera pas : chacun de nos paquets ne sera jamais reçu par la machine C.

Cela est lié à un problème : en faisant `sysctl net.ipv4.ip_forward` on voit que son statut est à 0.

![ip_forward à 0](13-ip-forward-0.png)

Cela veut dire que ip_forward ne nous donne pas cette autorisation, et donc 0 car c'est un booléen, donc 0 pour false, et nous on veut avoir cette autorisation.

Sans cette autorisation, dans notre cas A va envoyer un paquet à son RPD qui est en R1 et qui est la machine B, donc ce paquet sera reçu sur ens33. Mais le problème est que ce paquet a pour destination une machine qui est sur un autre réseau (R2), et c'est là qu'entre en jeu cette autorisation : elle nous dit si on a le droit d'envoyer ce paquet vers son autre carte réseau (ens36) car elle est sur R2.

Du coup, dans le cas où on a notre autorisation, la machine B va regarder sa table de routage et va donc décider d'envoyer ce paquet à ens36 qui est en R2 comme la destination de ce paquet, et donc ens36 va pouvoir envoyer ce paquet à sa destination sans souci.

Alors pour pouvoir mettre cette autorisation à 1, et pour qu'elle marche même en redémarrant la machine, j'ai créé un fichier `.conf` que j'ai appelé `99-routage.conf` dans le dossier `/etc/sysctl.d`, c'est là-dedans que va s'appliquer cette autorisation.

![Création du fichier 99-routage.conf](14-creation-99-routage-conf.png)

Maintenant qu'il a été créé, je peux mettre cette autorisation dans notre `99-routage.conf` : je vais tout simplement mettre dedans `net.ipv4.ip_forward=1` afin de bien activer notre autorisation en la mettant à 1.

![Contenu du fichier 99-routage.conf](15-contenu-99-routage-conf.png)

C'est bon, on a cette autorisation. Maintenant on peut en revérifier le statut en faisant la même commande `sysctl net.ipv4.ip_forward` et on verra bien qu'il sera à 1.

![ip_forward à 1](16-ip-forward-1.png)

Maintenant qu'on a cette autorisation, on pourra envoyer nos paquets de la machine A à la machine C tout en passant par la machine B.

On peut faire un ping de A vers B/R1 et que ça soit OK :

![Ping de A vers B/R1](17-ping-a-vers-b-r1.png)

On peut faire un ping de A vers B/R2 et que ça soit OK :

![Ping de A vers B/R2](18-ping-a-vers-b-r2.png)

On peut faire un ping de A vers C/R2 et que ça soit OK :

![Ping de A vers C/R2](19-ping-a-vers-c-r2.png)
