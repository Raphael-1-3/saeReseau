# SAE RESEAU 
#### Raphael BROUSSEAU groupe 11B

## Choix implémentation

Pour les routeurs, l’interface e0/0 est utilisée pour les sous-réseaux, et l’interface e0/1 est utilisée pour établir le lien entre les autres réseaux via le réseau principal, lorsque cela est possible.

J’ai par ailleurs choisi arbitrairement le routeur R3 pour la configuration DHCP. Plus précisément, le réseau lanPrcpl, car c’est celui qui relie la plupart des sous-réseaux.

Dans un premier temps, j’ai procédé au découpage de l’adresse IP fournie (54.98.152.0/23) en plusieurs sous-réseaux adaptés aux besoins de l’entreprise. Une fois les plages d’adresses définies, j’ai attribué des adresses statiques aux interfaces des routeurs, ce qui m’a permis de configurer le service DHCP de manière centralisée.

Ensuite, j’ai mis en place le routage dynamique avec RIP v2 sur tous les routeurs internes, afin que chacun connaisse les routes vers les autres réseaux. Cette étape était nécessaire pour assurer le bon fonctionnement du DHCP et des communications inter-réseaux.

Une fois le routage fonctionnel, j’ai configuré le serveur DHCP sur le routeur R3, en prenant soin d’exclure les premières adresses de chaque sous-réseau pour les équipements à IP fixe (comme les serveurs). J’ai ensuite configuré les relais DHCP (ip helper-address) sur les autres routeurs afin que les clients puissent recevoir leurs adresses automatiquement.

Enfin, j’ai appliqué une liste de contrôle d’accès (ACL) nommée PAREFEU sur le routeur R3. Ces ACL respectes les règles de sécurité demandées :

- Le réseau LibreService ne peut pas communiquer avec les autres réseaux internes, mais garde un accès à internet

- Le réseau Recherche ne peut échanger qu’avec le réseau Développement,

- Et les machines de la DMZ sont isolées des autres réseaux internes, tout en gardant un accès à Internet.

D'ailleurs après avoir rencontré des problèmes avec le serveur dhcp suite à l'activation de ces règle. J'ai autorisé explicitement les échanges du protocole UDP dans chacune d'entre elles.
## Adressage des réseaux 

**Adresse de base** : 54.98.152.0/23. Actuellement cette adresse permet d'acceuillir 512 hôtes.

J'ai choisi de découper ce réseau en 5 fois.

            /23            - réseau de base
          /24   /24
      */25*  */25*  */25*  /25
          --   --   */26*  /26
               --   */27*  */27*
Chaque reseau entre étoile est un réseau choisi et gardé pour notre implémentation.



| Nom réseau       | Adresse réseau   | Adresse broadcast  | Masque             |
|:------------------|:------------------:|:---------------------:|:----------------------:|
| `Developpement`   | 54.98.152.0        | 54.98.152.127         | 255.255.255.128 (/25)  |
| `Commercial`      | 54.98.152.128      | 54.98.152.255         | 255.255.255.128 (/25)  |
| `LibreService`    | 54.98.153.0        | 54.98.153.127         | 255.255.255.128 (/25)  |
| `Recherche`       | 54.98.153.128      | 54.98.153.191         | 255.255.255.192 (/26)  |
| `DMZ`             | 54.98.153.192      | 54.98.153.223         | 255.255.255.224 (/27)  |
| `lanPrcpl`        | 54.98.153.224      | 54.98.153.255         | 255.255.255.224 (/27)  |




## Adressage des routeurs 

| Routeur | Developpement | Recherche | DMZ | Commercial | LibreService | ReseauPrincipale |ReseauPublique|
|:-------:|:-------------:|:---------:|:---:|:----------:|:------------:|:----------------:| :-: | 
| `R1`      | 54.98.152.1 | 54.98.153.129 | X | X | X | 54.98.153.225 | X |
| `R2`      | X | X | X | 54.98.152.129  | X | 54.98.153.226 | X |
| `R3`      | X | X | 54.98.153.193 | X | X | 54.98.153.227 | 10.0.0.2 /24 |
| `R4`      | X | X | X | X | 54.98.153.1 | 54.98.153.228 | X |

## Adressage des pc Statique 
| PC | Developpement | Recherche | DMZ | Commercial | LibreService | ReseauPrincipale |
|:--:|:-------------:|:---------:|:---:|:----------:|:------------:|:----------------:|
| `StatiqueDev` | 54.98.152.2 | X | X | X | X | X | 
| `StatiqueRech` | X | 54.98.153.130| X | X | X | X |
| `StatiqueCom` | X | X | X | 54.98.152.130 | X | X |
| `StatiqueLS` | X | X | X | X | 54.98.153.2 | X |
| `StatiqueDMZ` | X | X | 54.98.153.194 | X | X | X |
| `StatiquePrcl` | X | X | X | X | X | 54.98.153.229 |

## Annexe
### Adressage routeur
```cisco
Adressage routeur R1 (developpement / recherche)
    
    conf term
    interface e0/0
    ip address 54.98.152.1 255.255.255.128
    no shutdown

    interface e0/1
    ip address 54.98.153.225 255.255.255.224
    no shutdown

    interface e0/2
    ip address 54.98.153.129 255.255.255.192
    no shutdown
    end
``` 
```cisco
Adressage routeur R2 (commercial)
    
    conf term 
    interface e0/0
    ip address 54.98.152.129 255.255.255.128	
    no shutdown

    interface e0/1
    ip address 54.98.153.226 255.255.255.224
    no shutdown
    end 
``` 
```cisco
Adressage routeur R3 (DMZ)
    
    conf term
    interface e0/0
    ip address 54.98.153.193 255.255.255.224
    no shutdown

    interface e0/1
    ip address 54.98.153.227 255.255.255.224
    no shutdown

    interface e0/2
    ip address 10.0.0.2 255.255.255.0
    no shutdown
    end
``` 
```cisco
Adressage routeur R4 (Libre Service)
    
    conf term
    interface e0/0
    ip address 54.98.153.1 255.255.255.128
    no shutdown

    no shutdown 
    interface e0/1
    ip address 54.98.153.228 255.255.255.224
    no shutdown
    end  

``` 
```cisco
Adressage routeurPublique
    
    conf term
    interface e0/0
    ip address 10.0.0.1 255.255.255.0
    no shutdown
    end
```
### Adressage des machines stastique
```cisco
pc Satstique developpement
    ip 54.98.152.2
```
```cisco
pc Satstique recherche
    ip 54.98.153.130
```
```cisco
pc Satstique DMZ
    ip 54.98.153.194
```
```cisco
pc Satstique commercial
    ip 54.98.152.130
```
```cisco
pc Satstique Principale
    ip 54.98.153.229
```

```cisco
pc Satstique LibreService
    ip 54.98.153.2
```

### Configuration routage dynamique RIP
```cisco 
    Routage rip R1

    conf term 
    router rip 
    version 2
    no auto-summary
    network  54.98.152.0
    network 54.98.153.128
    network 54.98.153.224
    end
```
```cisco 
    Routage rip R2

    conf term 
    router rip 
    version 2
    no auto-summary
    network 54.98.152.128
    network 54.98.153.224
    end
```
```cisco 
    Routage rip R3

    conf term 
    router rip 
    version 2
    no auto-summary
    network  54.98.153.192
    network 10.0.0.0
    network 54.98.153.224
    end
```
```cisco 
    Routage rip R4

    conf term 
    router rip 
    version 2
    no auto-summary
    network  54.98.153.0
    network 54.98.153.224
    end
```

```cisco 
    Routage rip ReseauPublique

    conf term 
    router rip 
    version 2
    no auto-summary
    network  10.0.0.0
    network 54.98.153.224
    end
```

### Configuration  DHCP sur le routeur R3 
#### Serveur DHCP : R3


```cisco
Serveur dhcp R3
    conf term
    service dhcp
    ip dhcp pool lanPrcpl
    network 54.98.153.224 255.255.255.224
    lease 69 
    default-router 54.98.153.227 
    
    ip dhcp pool DMZ
    network 54.98.153.192 255.255.255.224
    lease 69 
    default-router 54.98.153.193
    
    ip dhcp pool Developpement
    network 54.98.152.0 255.255.255.128
    lease 69 
    default-router 54.98.152.1

    ip dhcp pool Recherche
    network 54.98.153.128 255.255.255.192
    lease 69
    default-router 54.98.153.129

    ip dhcp pool Commercial
    network 54.98.152.128 255.255.255.128
    lease 69 
    default-router 54.98.152.129

    ip dhcp pool LibreService
    network 54.98.153.0 255.255.255.128
    lease 69 
    default-router 54.98.153.1
    
    ip dhcp excluded-address 54.98.153.225 54.98.153.235
    ip dhcp excluded-address 54.98.152.1 54.98.152.11
    ip dhcp excluded-address 54.98.152.129 54.98.152.140
    ip dhcp excluded-address 54.98.153.1 54.98.153.11
    ip dhcp excluded-address 54.98.153.129 54.98.153.140
    ip dhcp excluded-address 54.98.153.193 54.98.153.204
    end
```
#### Configuration relai DHCP : R1
```cisco
Relais dhcp R1

    conf term
    interface e0/0
    ip helper-address 54.98.153.227
    interface e0/2
    ip helper-address 54.98.153.227
    end
```
#### Configuration relai DHCP : R2
```cisco
Relais dhcp R2

    conf term
    interface e0/0
    ip helper-address 54.98.153.227
    end
```
#### Configuration relai DHCP : R4
```cisco
Relais dhcp R4

    conf term
    interface e0/0
    ip helper-address 54.98.153.227
    end
```

Le réseau Libre-service ne doit pas pouvoir communiquer avec les autres services, mais peut
communiquer sur Internet. Le service Recherche ne peut communiquer qu’avec le service Développement. Vous devez empêcher les machines du DMZ d’accéder aux autres réseaux.
## Configuration acl etendue
### configuration acl pour le reseau LibreService
```cisco
configuration acl pour le reseau libre service
    
conf term
ip access-list extended ACL_LibreServiceVersAutre
permit udp any eq 68 any eq 67
permit udp any eq 67 any eq 68
deny ip 54.98.153.0 0.0.0.127 54.98.152.0 0.0.0.127
deny ip 54.98.153.0 0.0.0.127 54.98.152.128 0.0.0.127
permit ip any any
exit
interface e0/0
ip access-group ACL_LibreServiceVersAutre in
ip access-list extended ACL_AutreVersLibreService
permit udp any eq 68 any eq 67
permit udp any eq 67 any eq 68
deny ip  54.98.152.0 0.0.0.127 54.98.153.0 0.0.0.127
deny ip  54.98.152.128 0.0.0.127 54.98.153.0 0.0.0.127
permit ip any any
exit
interface e0/1
ip access-group ACL_AutreVersLibreService in 
end
```
### Configuration acl pour le reseau recherche
```cisco
configuration acl pour le reseau recherche R1

conf t
ip access-list extended ACL_RechercheVersAutre
permit udp any eq 68 any eq 67
permit udp any eq 67 any eq 68
deny ip 54.98.153.128 0.0.0.63 54.98.153.192 0.0.0.31
deny ip 54.98.153.128 0.0.0.63 54.98.153.0 0.0.0.127
deny ip 54.98.153.128 0.0.0.63 54.98.152.128 0.0.0.127
permit ip any any
exit
interface e0/2
ip access-group ACL_RechercheVersAutre in
ip access-list extended ACL_AutreVersRecherche
permit udp any eq 68 any eq 67
permit udp any eq 67 any eq 68
deny ip 54.98.153.192 0.0.0.31 54.98.153.128 0.0.0.63
deny ip 54.98.153.0 0.0.0.127 54.98.153.128 0.0.0.63
deny ip 54.98.152.128 0.0.0.127 54.98.153.128 0.0.0.63
permit ip any any
exit
interface e0/1
ip access-group ACL_AutreVersRecherche in
end

```
### Configuration acl pour le resesau dmz
```cisco
configuration acl pour le reseau dmz

conf term 
ip access-list extended ACL_DMZVersAutre
permit udp any eq 68 any eq 67
permit udp any eq 67 any eq 68  
deny ip 54.98.153.192 0.0.0.31 54.98.153.0 0.0.0.127
deny ip 54.98.153.192 0.0.0.31 54.98.152.128 0.0.0.127
deny ip 54.98.153.192 0.0.0.31 54.98.152.0 0.0.0.127
permit ip any any
exit
interface e0/0
ip access-group ACL_DMZVersAutre in 
ip access-list extended ACL_AutreVersDMZ
permit udp any eq 68 any eq 67
permit udp any eq 67 any eq 68  
deny ip  54.98.153.0 0.0.0.127 54.98.153.192 0.0.0.31
deny ip  54.98.152.128 0.0.0.127 54.98.153.192 0.0.0.31
deny ip  54.98.152.0 0.0.0.127 54.98.153.192 0.0.0.31
permit ip any any
exit
interface e0/1
ip access-group ACL_AutreVersDMZ in 
end
```