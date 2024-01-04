---
lab:
  title: Comparer les coûts Azure locaux pour la migration
---

# Comparer les coûts Azure locaux pour la migration

Le coût total de possession (TCO) est un outil que vous pouvez utiliser lors d’un projet de modernisation des plateformes de données pour évaluer la différence de coût que la migration peut engendrer.

Au sein de votre entreprise mondiale de distribution, la modernisation des plateformes de données est censée permettre des économies considérables, mais le conseil d’administration vous a demandé d’estimer ces économies aussi précisément que possible.

Vous allez calculer ici le coût total de possession (TCO) de la migration vers Azure avec la calculatrice TCO.

Cet exercice prend environ **30** minutes.

## Calculer le coût total de possession (TCO)

1. Ouvrez un nouvel onglet de navigateur et accédez à la [calculatrice du coût TCO Azure](https://azure.microsoft.com/pricing/tco/calculator/).
1. Sous **Définir vos charges de travail**, supprimez toutes les charges de travail existantes dans la section **Serveurs**.

## Ajouter la charge de travail de la base de données

1. Sous **Bases de données**, sélectionnez **+ Ajouter une base de données**.
1. Dans la zone de texte **Nom**, tapez **Accounting** (Comptabilité).
1. Dans la section **Source**, choisissez les valeurs suivantes :

    | Propriété | Valeur |
    | --- | --- |
    | Base de données | **Microsoft SQL Server** |
    | Licence | **Entreprise** |
    | Environnement | **Serveurs physiques** |
    | Système d’exploitation | **Windows** |
    | Licence du système d’exploitation | **Centre de données** |
    | Serveurs | **1** |
    | Processeurs par serveur | **1** |
    | Noyau(x) par processeur | **4** |
    | RAM (Go) | **64** |
    | Optimiser par | **UC** |
    | SQL Server 2008/2008R2 | **Activer pour sélectionner cette valeur** |

1. Dans la section **Destination**, choisissez les valeurs suivantes :

    | Propriété | Valeur |
    | --- | --- |
    | Service | **Machine virtuelle SQL Server** |
    | Type de disque | **SSD** |
    | IOPS | **5000** |
    | Stockage SQL Server | **32 Go** |
    | Sauvegarde SQL Server | **32 Go** |

    > [!NOTE]
    > Des disques SSD sont recommandés pour les charges de travail de production dans Azure.

## Ajouter les charges de travail de stockage et de réseau

1. Sous **Stockage**, sélectionnez **+ Ajouter du stockage**.
1. Dans la zone de texte **Nom**, tapez **Accounting Local Disks** (Disques locaux de comptabilité), puis entrez les valeurs suivantes :

    | Propriété | Value |
    | --- | --- |
    | Type de stockage | **Disque local/SAN** |
    | Type du disque | **HDD** |
    | Capacité | **3 To** |
    | Backup | **1 To** |
    | Archivage | **0 To** |

1. Sous **Mise en réseau**, dans les contrôles **Bande passante sortante**, sélectionnez **1 Go**.
1. En bas de la page, sélectionnez **Suivant**.

## Ajuster les hypothèses

1. Dans la section **Ajuster les hypothèses**, dans la liste **Devise**, sélectionnez la devise de votre choix.
1. Sous **Couverture Software Assurance (offre Azure Hybrid Benefit)**, sélectionnez **Couverture Windows Server Software Assurance** en activant la bascule.
1. Sélectionnez **Couverture SQL Server Software Assurance** en activant la bascule.

    > [!NOTE]
    > Vous pouvez utiliser les liens fournis dans la section **Software Assurance** pour en savoir plus sur l’assurance disponible. 

1. Sous **Stockage géoredondant (GRS)**, vérifiez que l’option **Le stockage GRS réplique vos données vers une région secondaire située à des centaines de kilomètres de la région primaire** n’est pas activée.
1. Sous **Coûts des machines virtuelles**, assurez-vous que l’option **Activez cette option pour la calculatrice afin de ne pas recommander les machines virtuelles de la série Bs** est activée.

    > [!NOTE]
    > Les machines virtuelles de la série B n’offrent pas le ratio mémoire/vCore de 8 recommandé pour les charges de travail SQL Server.

1. Sous **Coûts liés à l’électricité**, dans la zone de texte **Tarif par Kilowatt-heure**, entrez une valeur réaliste pour votre localisation.

    > [!NOTE]
    > Vous pouvez trouver des tarifs d’électricité approximatifs dans la page des [tarifs d’électricité mondiaux](https://www.statista.com/statistics/263492/electricity-prices-in-selected-countries/). Ces prix sont en dollars US ($). Convertissez-les en une valeur approximative dans la devise de votre choix.

1. Sous **Coûts de stockage**, laissez toutes les valeurs par défaut ou ajustez-les si elles semblent déraisonnables.
1. Sous **Coûts liés au personnel informatique**, laissez toutes les valeurs par défaut ou ajustez-les si elles semblent déraisonnables.
1. Dans **Autres hypothèses**, développez chaque section et examinez les coûts associés.
1. En bas de la page, sélectionnez **Suivant**.

## Examiner le rapport sur 5 ans

1. Dans la page **Afficher le rapport**, vous pouvez constater que **Plage de temps** est défini par défaut sur **5 ans**.
1. Faites défiler le rapport et examinez la répartition estimée des coûts pour les systèmes locaux et Azure. Notez les informations suivantes :

    - Quel est le composant des coûts le plus important de l’environnement local ?
    - Quelle est l’économie la plus importante si vous décidez de migrer vers Azure ?

1. Développez chaque section tour à tour, puis examinez la répartition des coûts.

## Examiner le rapport sur 3 ans

1. Faites défiler vers le haut de la page, puis, dans la zone de texte **Plage de temps**, sélectionnez **3 ans**.
1. Faites défiler le rapport et examinez la répartition estimée des coûts pour les systèmes locaux et Azure. Notez les informations suivantes :

    - Quel est le composant des coûts le plus important de l’environnement local ?
    - Quelle est l’économie la plus importante si vous décidez de migrer vers Azure ?

1. Développez chaque section tour à tour, puis examinez la répartition des coûts.

Vous avez utilisé la calculatrice du coût TCO Azure pour identifier les différences de coût entre les déploiements locaux et Azure pour le serveur Accounting d’Adatum Corporation et les bases de données associées.
