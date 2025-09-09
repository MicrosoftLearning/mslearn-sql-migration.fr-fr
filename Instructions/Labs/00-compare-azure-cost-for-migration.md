---
lab:
  title: Comparer les coûts Azure pour la migration
---

# Comparer les coûts Azure pour la migration

La calculatrice de prix Azure est un outil utile que vous pouvez utiliser pendant un projet de modernisation de la plateforme de données pour estimer les coûts des différentes approches de migration et de services Azure.

Chez votre détaillant internationale, le projet de modernisation de la plateforme de données devrait réaliser des économies significatives, mais le conseil d’administration vous a demandé d’estimer les coûts des différentes options de migration Azure aussi précisément que possible.

Ici, vous allez calculer les coûts estimés de la migration vers Azure à l’aide de la [calculatrice de prix Azure](https://azure.microsoft.com/en-us/pricing/calculator/).

Cet exercice prend environ **30** minutes.

## Calculer les coûts Azure estimés

1. Ouvrez un nouvel onglet dans votre navigateur et rendez-vous sur le [calculatrice de prix Azure](https://azure.microsoft.com/en-us/pricing/calculator/).
1. La calculatrice de prix Azure vous aide à estimer les coûts des services Azure. Nous allons calculer le coût de la migration de votre charge de travail de base de données vers Azure.

## Ajouter la charge de travail de la base de données

1. Dans la section **Produits**, recherchez et sélectionnez **Azure SQL Database** (vous pouvez utiliser la barre de recherche ou parcourir la catégorie **Bases de données**).
1. Dans le panneau de configuration **Azure SQL Database** qui apparaît plus bas, renseignez les valeurs suivantes :

    | Propriété | Valeur |
    | --- | --- |
    | Région | **USA Est**(ou votre région préférée) |
    | Type | **Base de données unique** |
    | Modèle d’achat | **vCore** |
    | Niveau de service | **Usage général** |
    | Niveau de calcul | **approvisionné** |
    | Type de matériel | **Série Standard (Gen5)** |
    | Instance | **4 vCore** |
    | Récupération d’urgence | **Réplica principal ou géographique** |
    | Compute | **Localement redondant** |
    | Stockage | **32 Go** |
    | Stockage de sauvegarde | **RA-GRS** |

1. Passez en revue l’estimation des coûts mensuels affichée pour la base de données SQL.

## Ajouter une machine virtuelle à des fins de comparaison

1. De retour dans la section **Produits**, recherchez et sélectionnez **Machines virtuelles**.
1. Dans le panneau de configuration des **machines virtuelles**, saisissez les valeurs suivantes :

    | Propriété | Valeur |
    | --- | --- |
    | Région | **USA Est** (identique à la base de données) |
    | Système d’exploitation | **Windows** |
    | Type | **SQL Server** |
    | Niveau | **Standard** |
    | Instance | **D4s v3** (4 processeurs virtuels, 16 Go de RAM) |
    | Machines Virtuelles | **1** |
    | Licence | **SQL Standard** |

1. Développez **Disques managés** et ajoutez :
   - Niveau : **SSD Premium**, **128 Go**

## Ajouter un stockage pour les sauvegardes

1. Dans la section **Produits**, recherchez et sélectionnez **Comptes** de stockage.
1. Dans le panneau de configuration **Comptes de stockage**, sélectionnez ces valeurs :

    | Propriété | Valeur |
    | --- | --- |
    | Région | **USA Est** |
    | Type | **Stockage d’objets blob de blocs** |
    | Performances | **Standard** |
    | Type de compte de stockage | **V2 à usage général** |
    | Structure de fichier | **Espace de noms plat** |
    | Niveau d’accès | **Chaud** |
    | Redondance | **LRS** |
    | Capacité | **1 To** |

## Examiner et comparer les coûts

1. Passez en revue le total des coûts mensuels estimés pour tous les services que vous avez ajoutés :
   - **SQL Database** : Coûts du service de base de données managé
   - **Machines virtuelles** : Coûts d’infrastructure pour SQL Server sur une machine virtuelle
   - **Comptes de stockage** : Coûts de sauvegarde et de stockage supplémentaires

1. Tenez compte des questions suivantes lorsque vous passez en revue les estimations :
   - Comment les coûts sont-ils comparés entre SQL Database (PaaS) et SQL Server sur une machine virtuelle (IaaS) ?
   - Quels sont les compromis entre les services managés et les services d’infrastructure ?
   - Comment ces coûts peuvent-ils être mis à l’échelle avec vos besoins en charge de travail ?

1. Vous pouvez enregistrer cette estimation en sélectionnant **Enregistrer** ou **Exporter** pour partager avec les parties prenantes.

## Explorer les options d’optimisation des coûts

1. Dans chaque configuration de service, explorez différentes options pour voir leur impact sur les coûts :
   - **SQL Database** : Essayer différents niveaux de service (De base, Standard, Premium) et différentes tailles de calcul
   - **Machines virtuelles** : Comparer différentes tailles de machine virtuelle et options de stockage
   - **Stockage** : Comparer différentes options de redondance et différents niveaux d’accès

1. Sachez que la modification de ces options affecte votre estimation mensuelle globale.

Vous avez utilisé la calculatrice de prix Azure pour estimer les coûts de migration du serveur de comptabilité d’Adatum Corporation et de ses bases de données associées vers différents services Azure. Cette estimation vous donne une base pour comparer les options IaaS (Infrastructure as a Service) et PaaS (Platform-as-a-Service) et la planification de votre budget de migration.

