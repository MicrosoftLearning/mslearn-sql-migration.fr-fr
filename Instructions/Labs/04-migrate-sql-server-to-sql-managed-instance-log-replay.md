---
lab:
  title: Migrer des bases de données SQL Server vers Azure SQL Managed Instance en utilisant le service Log Replay Service
---

# Migrer des bases de données SQL Server vers Azure SQL Managed Instance en utilisant le service Log Replay Service

Dans cet exercice, vous allez apprendre à migrer une base de données SQL Server vers Azure SQL Managed Instance en tirant parti de Log Replay Service. 

Vous commencez par déployer une instance Azure SQL Managed Instance. Ensuite, vous utilisez les services Log Replay Service pour effectuer une migration en line d’une base de données SQL Server vers Azure SQL Managed Instance. Vous apprenez également à monitorer le processus de migration dans PowerShell.

Cet exercice prend environ **45** minutes.

> **Remarque** : pour effectuer cet exercice, vous devez avoir accès à un abonnement Azure pour créer des ressources Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?azure-portal=true) avant de commencer.

## Avant de commencer

Voici les éléments nécessaires pour effectuer cet exercice :

| Élément | Description |
| --- | --- |
| **Serveur cible** | Azure SQL Managed Instance. Nous allons le créer pendant cet exercice.|
| **Serveur source** | Instance de SQL Server 2019 ou une [version plus récente](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) installée sur le serveur de votre choix. |
| **Base de données source** | La base de données légère [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure) à restaurer sur l’instance SQL Server. |
| **Azure Data Studio** | Installez [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) sur le même serveur que celui où se trouve la base de données source. Si l'application est déjà installée, mettez-la à jour pour vous assurer que vous utilisez la version la plus récente. |

## Restaurer une base de données SQL Server

Nous allons restaurer la base de données *AdventureWorksLT* sur l’instance SQL Server. Cette base de données sert de base de données source pour l’exercice de ce labo. Passez ces étapes si la base de données est déjà restaurée.

1. Sélectionnez le bouton Démarrer de Windows et tapez SSMS. Sélectionnez **Microsoft SQL Server Management Studio 18** dans la liste.  

1. Une fois SQL Server Management Studio ouvert, notez que le dialogue **Se connecter au serveur** remplit automatiquement le nom d’instance par défaut. Sélectionnez **Se connecter**.

1. Sélectionnez le dossier **Bases de données**, puis **Nouvelle requête**.

1. Dans la nouvelle fenêtre de requête, copiez et collez le T-SQL suivant. Exécutez la requête pour restaurer la base de données.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\LabFiles\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\LabFiles\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\LabFiles\AdventureWorksLT2019.ldf';
    ```

    > **Remarque** : vérifiez que le nom et le chemin d'accès du fichier de sauvegarde de la base de données dans l’exemple ci-dessus correspondent à votre fichier de sauvegarde réel. Si ce n’est pas le cas, la commande peut échouer.

1. Un message de réussite doit s’afficher une fois la restauration terminée.

## Déployer une instance Azure SQL Managed Instance

Créer une instance Azure SQL Managed Instance en suivant ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true), puis sélectionnez **Créer une ressource** dans l’angle supérieur gauche.
1. Effectuez une recherche sur **managed instance**, sélectionnez **Azure SQL Managed Instance**, puis **Créer**.
1. Remplissez le formulaire d’instance managée SQL en vous aidant des informations figurant dans le tableau suivant :

    |  | Valeur suggérée |
    |---|---|
    | **Abonnement** | Votre abonnement. |
    | **nom de l’instance managée** | Nom valide. |
    | **connexion administrateur de l’instance managée** | N’importe quel nom d’utilisateur valide. N’utilisez pas « serveradmin », car il s’agit d’un rôle réservé au niveau du serveur. |
    | **Mot de passe** | Tout mot de passe supérieur à 16 caractères et répondant aux impératifs de complexité. |
    | **Fuseau horaire** | Fuseau horaire devant être respecté par votre instance managée. |
    | **Classement** | Classement à utiliser pour l’instance managée. Si vous migrez des bases de données à partir de SQL Server, vérifiez le classement de la source à l’aide de SELECT SERVERPROPERTY(N'Collation') et utilisez cette valeur. |
    | **Lieu** | Région Azure dans laquelle vous souhaitez créer l’instance managée. |
    | **Réseau virtuel** | Sélectionnez Créer un réseau virtuel, ou sélectionnez un réseau et un sous-réseau virtuels valides. |
    | **Activer le point de terminaison public** | Cochez cette option pour activer un point de terminaison public, ce qui permet aux clients externes à Azure d’accéder à la base de données. |
    | **Autoriser l’accès depuis** | Sélectionnez les services Azure, Internet ou aucun accès. |
    | **Type de connexion** | Choisissez entre le type de connexion Proxy et Redirection. |
    | **Groupe de ressources** | nouveau groupe de ressources ou groupe de ressources existant. |

1. Sélectionnez **Niveau tarifaire** pour dimensionner les ressources de calcul et de stockage et examiner les options de niveau tarifaire. 
1. Quand vous avez terminé, sélectionnez **Appliquer** pour enregistrer votre sélection, puis sélectionnez **Créer** pour déployer l’instance managée.
1. Sélectionnez l’icône **Notifications** pour voir l’état du déploiement.
1. Sélectionnez **Déploiement en cours** pour ouvrir la fenêtre de l’instance managée et superviser de façon plus approfondie la progression du déploiement.

## Créer un conteneur et un compte Stockage Blob Azure

Créez un compte Stockage Blob Azure dans la même région que votre instance Azure SQL Managed Instance. Il s’agit de l’emplacement où vous stockez les sauvegardes de votre base de données pour des migrations.

1. Accédez au [Portail Azure ](https://portal.azure.com) et connectez-vous en utilisant les informations d’identification de votre compte.
1. Dans le menu de gauche, sélectionnez **Tous les services**, puis recherchez *« Comptes de stockage »*. Sélectionnez **Comptes de stockage** pour ouvrir la page des comptes de stockage.
1. Sur la page Comptes de stockage, sélectionnez **+ Ajouter** pour créer le compte de stockage.
1. Sous l’onglet **Informations de base** de la page **Créer un compte de stockage**, sélectionnez l’abonnement que vous souhaitez utiliser pour le compte de stockage. Sélectionnez ensuite le groupe de ressources qui contient votre instance Azure SQL Managed Instance.
1. Entrez un nom unique pour le compte de stockage. 
    
    > **Remarque :** Le nom doit comporter entre 3 et 24 caractères et peut contenir uniquement des lettres minuscules et des chiffres.

1. Sélectionnez l’emplacement (région) où se trouve votre instance Azure SQL Managed Instance.
1. Spécifiez le niveau de performances pour le compte de stockage.
1. Sélectionnez **BlobStorage** pour le type de compte de stockage. 
1. Sélectionnez **Stockage localement redondant (LRS)** pour l’option de réplication du compte de stockage.
1. Passez en revue et sélectionnez **Vérifier + créer** pour créer le compte de stockage.
1. Une fois le compte de stockage créé, accédez à la page du compte de stockage, puis sélectionnez l’option **Conteneurs** dans le menu de gauche. Sélectionnez ensuite **+ Conteneur** pour créer un conteneur. Entrez un nom pour le conteneur et sélectionnez le niveau d’accès public. 
1. Sélectionnez le bouton **Créer** pour créer le conteneur.

Une fois ces étapes terminées, vous disposez d’un compte Stockage Blob Azure dans la même région que votre instance Azure SQL Managed Instance et un conteneur où vous pouvez stocker les sauvegardes de votre base de données pour des migrations.

## Sauvegarder une base de données SQL Server

Créons une sauvegarde complète de la base de données *AdventureWorksLT* sur l’instance SQL Server, suivie des sauvegardes de fichier journal et différentielle avec l’élément `CHECKSUM` activé. 

1. Sélectionnez le bouton Démarrer de Windows et tapez SSMS. Sélectionnez **Microsoft SQL Server Management Studio 18** dans la liste.  
1. Une fois la suite SSMS ouverte, notez que le dialogue **Se connecter au serveur** remplit automatiquement le nom d’instance par défaut. Sélectionnez **Se connecter**.
1. Sélectionnez le dossier **Bases de données**, puis **Nouvelle requête**.
1. Dans la fenêtre de la nouvelle requête, copiez et collez le code T-SQL ci-dessous. Exécutez la requête pour restaurer la base de données.

    ```sql
    BACKUP DATABASE AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_full.bak'
    WITH CHECKSUM;

    BACKUP DATABASE AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_diff.dif'
    WITH DIFFERENTIAL, CHECKSUM;

    BACKUP LOG AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_log.trn'
    WITH CHECKSUM;
    ```

    > **Remarque** : Vérifiez que le chemin d’accès au fichier de l’exemple ci-dessus correspond au chemin d’accès de votre fichier réel. Si ce n’est pas le cas, la commande peut échouer.

1. Un message de réussite doit s’afficher une fois la restauration terminée.
1. Si vous exécutez une version de SQL Server (à partir de SQL Server 2012 SP1 CU2 et SQL Server 2014), vous pouvez effectuer des sauvegardes à partir de SQL Server directement dans votre compte Stockage Blob en utilisant l’option `BACKUP TO URL` SQL Server native. 

    ```sql
    CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<containername>] 
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
    SECRET = '<SAS_TOKEN>';  
    GO
    
    -- Take a full database backup to a URL
    BACKUP DATABASE [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_full.bak'
    WITH INIT, COMPRESSION, CHECKSUM
    GO
    
    -- Take a differential database backup to a URL
    BACKUP DATABASE [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_diff.bak'  
    WITH DIFFERENTIAL, COMPRESSION, CHECKSUM
    GO
    
    -- Take a transactional log backup to a URL
    BACKUP LOG [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_log.trn'  
    WITH COMPRESSION, CHECKSUM
    ```

    > **Remarque :** Si vous décidez d’utiliser cette option, vous pouvez ignorer la section **Copier les fichiers de sauvegarde dans le compte Stockage Azure** suivante.

## Copier les fichiers de sauvegarde dans le compte Stockage Azure

Copions maintenant les fichiers de sauvegarde dans le compte Stockage Blob Azure créé plus tôt.

1. Accédez au [Portail Azure ](https://portal.azure.com) et connectez-vous en utilisant les informations d’identification de votre compte.
1. Dans le menu de gauche, sélectionnez **Comptes de stockage**, puis le compte de stockage créé précédemment.
1. Dans la page de vue d’ensemble du compte de stockage, faites défiler jusqu’à la section **Service BLOB**, puis sélectionnez **Conteneurs**. Sélectionnez le conteneur créé précédemment.
1. Sélectionnez **Charger** en haut de la page du conteneur. Dans la page **Charger l’objet blob**, sélectionnez **Dossier** pour sélectionner le dossier contenant les fichiers de sauvegarde ou **Fichiers** pour choisir des fichiers de sauvegarde individuels. Une fois les fichiers sélectionnés, choisissez **Charger** pour démarrer le processus de chargement.

## Valider l’accès

Il est important de vérifier si votre SQL Server et votre SQL Managed Instance peuvent correctement accéder à votre compte Stockage Blob. Pour ce faire, exécutez un exemple de requête test pour déterminer si votre instance gérée peut accéder à la sauvegarde dans le conteneur.

1. Connectez-vous à votre instance SQL Managed Instance via SSMS.
1. Ouvrez un nouvel éditeur de requête, puis exécutez la commande.

```sql
CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/databases] 
WITH IDENTITY = 'SHARED ACCESS SIGNATURE' 
, SECRET = '<sastoken>' 

RESTORE HEADERONLY 
FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<backup_file_name>.bak'
```
1. Répétez ce processus connecté sur votre instance SQL Server.

## Utiliser Log Replay Service pour restaurer des fichiers de sauvegarde

Vous utilisez le service Log Replay Service (LRS) pour restaurer les fichiers de sauvegarde à partir de Stockage Blob Azure dans votre instance Azure SQL Managed Instance. LRS est un service gratuit basé sur la technologie de copie des journaux de transaction SQL Server.

1. Dans la page de vue d’ensemble du compte de stockage, faites défiler jusqu’à la section **Service BLOB**, puis sélectionnez **Conteneurs**. Sélectionnez le conteneur de stockage des fichiers de sauvegarde.
1. Sélectionnez **Générer une signature d’accès partagé** en haut de la page de conteneur. Dans la page **Générer une signature d’accès partagé**, sélectionnez les autorisations que vous souhaitez octroyer, définissez l’heure de début et d’expiration du jeton SAP, puis sélectionnez **Générer la chaîne de connexion et SAP**. Le jeton SAP s’affiche dans le champ **Jeton SAP**. Copiez-le.
1. Utilisez PowerShell pour vous connecter à votre compte Azure en exécutant la cmdlet `Connect-AzAccount`.

    ```powershell
    Login-AzAccount
    Select-AzSubscription -SubscriptionId <subscription ID>
    ```

1. Utilisez la cmdlet `Start-AzSqlInstanceDatabaseLogReplay` pour démarrer le service Log Replay Service pour la base de données que vous souhaitez restaurer. Vous devez fournir le nom de groupe de ressources, le nom d’instance, le nom de base de données, l’URI de conteneur de stockage et le jeton SAP précédemment copié.

```PowerShell
Import-Module Az.Sql

Start-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "YourResourceGroupName" -InstanceName "YourInstanceName" -Name "YourDatabaseName" -StorageContainerUri "https://yourstorageaccount.blob.core.windows.net/yourcontainer" -StorageContainerSasToken "YourSasToken"
```

## Superviser la progression de la migration

Vous pouvez utiliser la cmdlet `Get-AzSqlInstanceDatabaseLogReplay` pour monitorer la progression de Log Replay Service. Cet cmdlet renvoie des informations sur l’état actuel du service, notamment le dernier fichier de sauvegarde de fichier journal restauré.

1. Exécutez le code PowerShell suivant.

```powershell
# Import the Az.Sql module
Import-Module Az.Sql

# Set the resource group name, instance name, and database name
$resourceGroupName = "YourResourceGroupName"
$instanceName = "YourInstanceName"
$databaseName = "YourDatabaseName"

# Get the log replay status
$logReplayStatus = Get-AzSqlInstanceDatabaseLogReplay -ResourceGroupName $resourceGroupName -InstanceName $instanceName -Name $databaseName

# Display the log replay status
$logReplayStatus | Format-List
```

## Exécution du basculement de migration

Une fois la sauvegarde de base de données complète restaurée sur l’instance managée cible d’Azure SQL Database Managed Instance, la base de données est disponible pour un basculement de migration.

1. Lorsque vous êtes prêt à effectuer la migration de base de données en ligne, sélectionnez **Démarrer le basculement**.
1. Arrêtez tout le trafic entrant dans les bases de données sources.
1. Effectuez la sauvegarde de la fin du journal, mettez le fichier de sauvegarde à disposition dans le partage réseau SMB, puis attendez que cette sauvegarde de fichier journal finale soit restaurée.
1. À ce stade, vous devez voir le paramètre **Modifications en attente** défini sur 0.
1. Sélectionnez **Confirmer**, puis **Appliquer**.

    ![Écran de basculement de migration](../media/3-migration-cutover-screen.png)

1. Dès que la migration de base de données présente l’état **Terminé**, connectez vos applications à la nouvelle instance cible d’Azure SQL Database Managed Instance.