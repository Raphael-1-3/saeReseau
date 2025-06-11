# SAE RESEAU 

## Choix implémemtation

Pour les routeur, c'est l'interface e0/0 qui est utilisé pour les sous-réseaux.
Et donc l'interface e0/1 qui est utilisé pour établir le lien entre les autres réseau par le reseau principale quand cela est possible.

J'ai par ailleur choisi le routeur R3 comme serveur DHCP de façon purement arbitraire. Plus prescisement le lanPrcpl car c'est celui qui relie la plus part des sous réseaux.

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
| `StastiqueDev` | 54.98.152.2 | X | X | X | X | X | 
| `StastiqueRech` | X | 54.98.153.130| X | X | X | X |
| `StastiqueCom` | X | X | X | 54.98.152.130 | X | X |
| `StastiqueLS` | X | X | X | X | 54.98.153.2 | X |
| `StastiqueDMZ` | X | X | 54.98.153.194 | X | X | X |
| `StastiquePrcl` | X | X | X | X | X | 54.98.153.229 |

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

### Configuration  DHCP sur le routeur R3 
#### Serveur DHCP : R3
les dix premières
adresses de chaque sous-réseau sont réservées pour l’installation de serveurs avec des adresses statiques.

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


## Configuration parfeu 
Le réseau Libre-service ne doit pas pouvoir communiquer avec les autres services, mais peut
communiquer sur Internet. Le service Recherche ne peut communiquer qu’avec le service Développement. Vous devez empêcher les machines du DMZ d’accéder aux autres réseaux.
! (autorisation implicite vers 10.0.0.0/24)
```cisco
Configuration de l'ACL PAREFEU sur le routeur R3

    conf term
    ip access-list extended PAREFEU
    ! --- Autoriser Recherche vers Developpement ---
    permit ip 54.98.153.128 0.0.0.63 54.98.152.0 0.0.0.127
    permit ip 54.98.152.0 0.0.0.127 54.98.153.128 0.0.0.63

    ! --- Interdire Recherche vers tout le reste (hors Developpement) ---
    deny ip 54.98.153.128 0.0.0.63 54.98.152.128 0.0.0.127     ! vers Commercial
    deny ip 54.98.153.128 0.0.0.63 54.98.153.0 0.0.0.127       ! vers LibreService
    deny ip 54.98.153.128 0.0.0.63 54.98.153.192 0.0.0.31      ! vers DMZ

    ! --- Interdire DMZ vers tout (sauf Internet) ---
    deny ip 54.98.153.192 0.0.0.31 54.98.152.0 0.0.1.255       ! vers tous 54.98.152.x et .153.x

    ! --- Interdire LibreService vers tous les réseaux sauf Internet (10.0.0.0/24) ---
    deny ip 54.98.153.0 0.0.0.127 54.98.152.0 0.0.1.255        ! bloque LibreService → internes
    
    ! --- Autoriser tout le reste ---
    permit ip any any
    
    exit
    
    interface e0/1
    ip access-group PAREFEU in
    end
```