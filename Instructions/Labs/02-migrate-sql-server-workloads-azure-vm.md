---
lab:
  title: "Migrer des bases de données SQL\_Server vers SQL\_Server sur une machine virtuelle Azure"
---

# Migrer des bases de données SQL Server vers SQL Server sur une machine virtuelle Azure

Dans cet exercice, vous allez apprendre à migrer une base de données SQL Server vers un serveur SQL Server s’exécutant sur une machine virtuelle Azure au moyen de l’extension de migration Azure pour Azure Data Studio. Vous allez commencer par installer et lancer l’extension de migration Azure pour Azure Data Studio. Vous effectuerez ensuite une migration en ligne d’une base de données SQL Server vers un serveur SQL Server s’exécutant sur une machine virtuelle Azure. Vous allez également apprendre à surveiller le processus de migration sur le portail Azure et à terminer le processus de basculement pour finaliser la migration.

Cet exercice prend environ **45** minutes.

> **Remarque** : pour effectuer cet exercice, vous devez avoir accès à un abonnement Azure pour créer des ressources Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?azure-portal=true) avant de commencer.

## Avant de commencer

Voici les éléments nécessaires pour effectuer cet exercice :

| Élément | Description |
| --- | --- |
| **Serveur cible** | Serveur SQL Server sur une machine virtuelle Azure Pour en savoir plus, consultez l’article [Approvisionner une instance de SQL Server sur une machine virtuelle Azure](https://microsoftlearning.github.io/dp-300-database-administrator/Instructions/Labs/01-provision-sql-vm.html). **Remarque :** la version de SQL Server de la cible et du serveur doit être identique. |
| **Serveur source** | La dernière version de [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) installée sur un serveur de votre choix. |
| **Base de données source** | La base de données [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure) légère à restaurer sur l’instance SQL Server 2022. |

## Approvisionner un compte de stockage Azure avec un conteneur d’objets blob

L’objectif de la création d’un compte de stockage Azure est de stocker les sauvegardes complètes et les journaux des transactions pour la migration. Nous utiliserons ce compte de stockage plus tard dans l’exercice.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Dans le menu du portail de gauche, sélectionnez **Comptes de stockage** pour afficher la liste de vos comptes de stockage. Si le menu du portail n’est pas visible, cliquez sur le bouton du menu pour l’activer.
1. Sur la page **Comptes de stockage**, sélectionnez **Créer**.
1. Sous **Détails du projet**, sélectionnez l’abonnement Azure dans lequel vous avez créé la machine virtuelle Azure.
1. Sélectionnez le même groupe de ressources que celui dans lequel vous avez créé la machine virtuelle Azure. 
1. Choisissez un nom unique pour votre compte de stockage, puis sélectionnez la même région que celle de la machine virtuelle Azure.
1. Choisissez le niveau de service **Standard**.
1. Conservez les valeurs par défaut pour les options restantes.
1. Sélectionnez **Vérifier + créer**, puis sélectionnez **Créer**.

Une fois le compte de stockage créé, vous pouvez créer un conteneur en procédant comme suit :

1. Dans le portail Azure, accédez à votre nouveau compte de stockage.
1. Dans le menu de gauche du compte de stockage, faites défiler jusqu’à la section **Service Blob**, puis sélectionnez **Conteneurs**.
1. Sélectionnez **+ Conteneur** pour créer un conteneur.
1. Sur la page Nouveau conteneur, fournissez les informations suivantes :
    - **Nom :***nom de votre choix*
    - **Niveau d’accès Public :** privé
1. Sélectionnez **Créer**.

## Installer et lancer l’extension de migration Azure pour Azure Data Studio

Avant de commencer à utiliser l’extension de migration Azure, vous devez installer [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio). Pour ce scénario, installez Azure Data Studio sur le même serveur que celui où se trouve la base de données source. L’extension est disponible dans la Place de marché Azure Data Studio.

Pour installer l’extension de migration, suivez ces étapes :

1. Ouvrez le gestionnaire d’extensions dans Azure Data Studio.
1. Recherchez ***Azure SQL Migration***, puis sélectionnez l’extension.
1. Installez l’extension. Une fois installée, l’extension de migration Azure SQL s'affiche dans la liste des extensions installées.
1. Connectez-vous à une instance SQL Server dans Azure Data Studio.
1. Pour lancer l’extension de migration Azure, cliquez avec le bouton droit sur le nom de l’instance source, puis sélectionnez **Gérer** pour accéder au tableau de bord et à la page d’arrivée de l’extension de migration Azure SQL.

## Effectuer une migration en ligne d’une base de données SQL Server vers un serveur SQL Server s’exécutant sur une machine virtuelle Azure

Pour effectuer une migration avec un temps d’arrêt minimal en utilisant Azure Data Studio, procédez comme suit :

1. Lancez l’Assistant Migration vers Azure SQL de l’extension dans Azure Data Studio.

1. Dans la page **Étape 1 : bases de données pour l’évaluation**, sélectionnez la base de données à migrer, puis cliquez sur **Suivant**.
    
    > **Remarque** : il est recommandé de collecter des données de performances et d’obtenir des recommandations Azure appropriées.

1. À l’**Étape 2 : résultats et recommandations de l’évaluation**, attendez que l’évaluation se termine, puis sélectionnez **SQL Server sur une machine virtuelle Azure** comme cible **Azure SQL**.

1. En bas de la page **Étape 2 : résultats et recommandations de l’évaluation**, sélectionnez **Afficher/Sélectionner** pour afficher les résultats de l’évaluation. Sélectionnez la base de données à migrer. 

    > **Remarque** : prenez un moment pour examiner les résultats de l’évaluation affichés à droite.

1. À l’**Étape 3 : cible Azure SQL**, sélectionnez un compte Azure et votre serveur SQL Server cible sur une machine virtuelle Azure.

    ![Capture d’écran de la configuration de la cible Azure SQL de l’extension de migration Azure pour Azure Data Studio.](../media/3-step-azure-sql-target.png)

1. À l’**Étape 4 : Azure Database Migration Service**, créez un service Azure Database Migration Service en utilisant l’Assistant Azure Data Studio. Si vous en avez créé un précédemment, réutilisez-le. Vous pouvez également créer une ressource Azure Database Migration Service dans le portail Azure.

    > **Remarque** : assurez-vous que l’abonnement est inscrit pour utiliser l’espace de noms **Microsoft.DataMigration**. Pour découvrir comment effectuer l’inscription d’un fournisseur de ressources, consultez [Inscrire le fournisseur de ressources](https://learn.microsoft.com/azure/dms/quickstart-create-data-migration-service-portal#register-the-resource-provider).

1. Sauvegardez la base de données source. Vous pouvez [sauvegarder dans le Stockage Blob Microsoft Azure à l’aide de SSMS ou T-SQL](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/sql-server-backup-to-url). Vous avez également la possibilité de copier manuellement la sauvegarde de la base de données dans un dossier conteneur à l’aide du portail Azure.

    > **Remarque** : vérifiez qu’un dossier est créé dans le conteneur avant de poursuivre la copie du fichier de sauvegarde.

1. À l’**Étape 5 : configuration de la source de données**, sélectionnez l’emplacement de vos sauvegardes de base de données, soit sur un partage réseau local, soit dans un conteneur de Stockage Blob Azure.

1. Démarrez la migration de la base de données et surveillez la progression dans Azure Data Studio. Vous pouvez également suivre la progression dans le portail Azure sous la ressource Azure Database Migration Service.

    > **Remarque** : Azure Data Migration Service orchestre et restaure automatiquement les fichiers de sauvegarde sur le serveur cible.

1. Sélectionnez **Migrations de bases de données en cours** dans le tableau de bord de la migration pour afficher les migrations en cours. 

    ![Capture d’écran du tableau de bord de la migration dans l’extension de migration Azure pour Azure Data Studio.](../media/3-data-migration-dashboard.png)

1. Sélectionnez le nom de la base de données pour obtenir plus de détails.

    ![Capture d’écran des détails de la migration dans l’extension de migration Azure pour Azure Data Studio.](../media/3-dashboard-details.png)

## Surveiller la migration dans le portail Azure

Vous pouvez également surveiller l’activité de migration à l’aide d’Azure Database Migration Service. 

1. 
    ![Capture d’écran de la page de surveillance dans Azure Database Migration Services du portail Azure.](../media/3-dms-azure-portal.png)
    
## Terminer le processus de basculement

1. Effectuez une [sauvegarde de fichier journal après défaillance](https://learn.microsoft.com/sql/relational-databases/backup-restore/tail-log-backups-sql-server) pour la base de données source.

1. Dans le portail Azure, chargez la sauvegarde du journal des transactions dans le conteneur et le dossier où se trouve le fichier de sauvegarde complet.

1. Dans l’extension de migration Azure, sélectionnez **Terminer le basculement** dans la page de surveillance.

    ![Capture d’écran de l’option de basculement de la migration de l’extension de migration Azure pour Azure Data Studio.](../media/3-dashboard-details-cutover.png)

1. Vérifiez que toutes les sauvegardes de journal ont été restaurées sur la base de données cible. La valeur de **Sauvegardes de journaux en attente de restauration** doit être égale à zéro. Cette étape termine la migration.

    ![Capture d’écran de l’option de basculement de la migration de l’extension de migration Azure pour Azure Data Studio.](../media/3-dashboard-details-cutover-extension.png)

1. La propriété État de la migration passe à **Terminée**, puis à **Réussie** une fois la migration terminée.

    > **Remarque** : vous pouvez effectuer le basculement à l’aide d’étapes similaires avec Azure Database Migration Service via le portail Azure.

1. Lorsque l’état de la migration est **Réussie**, accédez au serveur cible et validez la base de données cible. Vérifiez le schéma et les données de la base de données.

Vous avez appris à migrer une base de données SQL Server vers un serveur SQL Server s’exécutant sur une machine virtuelle Azure au moyen de l’extension de migration Azure pour Azure Data Studio. Vous avez également appris à terminer le processus de basculement pour finaliser la migration. Cela garantit que toutes les données ont été correctement migrées et que la nouvelle base de données est entièrement opérationnelle. Une fois le processus de basculement terminé, vous pouvez commencer à utiliser votre nouvelle base de données SQL Server s’exécutant sur une machine virtuelle Azure. 

## Nettoyer

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. 

Laisser inutilement des ressources en cours d’exécution peut entraîner des coûts supplémentaires. Vous pouvez supprimer les ressources individuellement ou supprimer le groupe de ressources dans le [Portail Azure](https://portal.azure.com?azure-portal=true).

## Plus d’informations

Pour plus d’informations sur l’exécution de SQL Server sur les machines virtuelles Azure, consultez l’article [Qu’est-ce que SQL Server sur les machines virtuelles Azure ?](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview?view=azuresql-vm)
