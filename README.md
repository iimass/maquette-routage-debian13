# Projet 1 : Maquette d'un réseau entre 3 machines

## Présentation de la Maquette :

Ce projet a pour but d'avoir une table de routage entre 3 machines A, B et C

Nous avons que les machines A et B sont connectées sur le réseau 1 qui est en NAT ce qui signifie donc qu'elles ont accès a internet et qui a pour adresse réseau 192.168.10.0/24.

Nous avons aussi les machines B et C qui sont elles connectées au réseau 2 qui est en Host Only ce qui signifie qu'elles communiquent uniquement entre elles et avec la machine hôte sans accès à internet et qui a pour adresse réseau 192.168.20.0/24.

La machine B est connectée aux 2 réseaux et elle a donc 2 cartes réseau dont une qui est sur le R1 et l'autre sur le R2. Comme nous avons les machines A et C qui ne sont pas sur le même réseau elles ne peuvent pas communiquer directement entre elles, elle sont obligées de passer par la machine B car la machine B est la seule machine présente sur les 2 réseaux. C'est pour cela que le RPD (Routeur Par Défaut) de A est B et celui de C est B aussi.

![Maquette du réseau](images/01-maquette.png)

## Configuration des 3 machines :

### Configuration d'une interface :

Alors pour configurer l'interface de notre machines on doit mettre notre configuration dans le fichier interfaces a l'emplacement `/etc/network/interfaces` et pour pouvoir l'éditer j'utilise vi (vim),
donc je tape `vi /etc/network/interfaces` et la je peux ajouter mes configuration que je souhaite mettre.

### Pour la machine A :

on mettra d'abord dans nos configuration `auto ens33` afin que a chaque démarrage de la machine les configuration ens33 qu'on aura fait ce mettrons automatiquement, ens33 est le nom que Linux donne à notre carte réseau, ce nom dépend du slot de la carte dans la machine. En fessent `ip a` on a pue constater qu'il y a 2 interfaces le lo et le ens33 ce qui est normale car on a bien qu'une carte réseau et le premier interfaces est toujours la c'est notre interfaces de bouclage. A la suite on met `iface ens33 inet static`, iface pour notre interfaces ens33 , inet signifie qu'on va le configurer en ipv4 et le static car on les met manuellement.

Ensuite `address 192.168.10.1/24` on met l'adresse de notre machine nous constatons qu'elle est bien sur R1 (192.168.10.0/24), et pour finir on met `gateway 192.168.10.2` ce qui signifie qu'on a crée un chemin vers une machines qui a cette adresse ce qui correspond a la machine B

![Fichier interfaces de la machine A](images/02-interfaces-machine-a.png)

Maintenant qu'on a bien mis nos configuration on a juste un faire `ifup ens33` pour bien executer nos configuration de notre interfaces ens33.

En fessant `ip a`, cela va afficher directement nos interfacers :

![Sortie de ip a](images/03-ip-a-machine-a.png)

Mais aussi en fessant `ip route`, nous voyons directement les routes de notre machines envers dans machines dans notre cas par exemple :

![Sortie de ip route](images/04-ip-route-machine-a.png)

Nous voyons clairement que la configurations de notre interfaces ens33 a bien été appliquée que maintenant l'adresse ip de notre machines est 192.168.10.1/24, qu'elle est bien sur le réseau et que aussi sont routeurs par défauts est 192.168.10.2 qui est la machine B.
