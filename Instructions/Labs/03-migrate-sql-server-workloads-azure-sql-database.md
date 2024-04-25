---
lab:
  title: "Migrer des bases de données SQL\_Server vers Azure\_SQL\_Database"
---

# Migrer des bases de données SQL Server vers Azure SQL Database

Dans cet exercice, vous allez apprendre à migrer une base de données SQL Server vers Azure SQL Database en utilisant l’extension de migration Azure pour Azure Data Studio. Vous allez commencer par installer et lancer l’extension de migration Azure pour Azure Data Studio. Vous allez ensuite procéder à la migration hors connexion d’une base de données SQL Server vers Azure SQL Database. Vous apprendrez également à surveiller le processus de migration sur le portail Azure.

Cet exercice prend environ **45** minutes.

> **Remarque** : pour effectuer cet exercice, vous devez avoir accès à un abonnement Azure pour créer des ressources Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?azure-portal=true) avant de commencer.

## Avant de commencer

Voici les éléments nécessaires pour effectuer cet exercice :

| Élément | Description |
| --- | --- |
| **Serveur cible** | Un serveur Azure SQL Database. Nous allons le créer pendant cet exercice.|
| **Base de données cible** | Une base de données hébergée sur un serveur Azure SQL Database. Nous allons le créer pendant cet exercice.|
| **Serveur source** | Instance de SQL Server 2019 ou une [version plus récente](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) installée sur le serveur de votre choix. |
| **Base de données source** | La base de données légère [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure) à restaurer sur l’instance SQL Server. |
| **Azure Data Studio** | Installez [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) sur le même serveur que celui où se trouve la base de données source. Si l'application est déjà installée, mettez-la à jour pour vous assurer que vous utilisez la version la plus récente. |
| Fournisseur de ressources **Microsoft.DataMigration** | Assurez-vous que l’abonnement vous autorise à utiliser l’espace de noms **Microsoft.DataMigration**. Pour découvrir comment effectuer l’inscription d’un fournisseur de ressources, consultez [Inscrire le fournisseur de ressources](https://learn.microsoft.com/azure/dms/quickstart-create-data-migration-service-portal#register-the-resource-provider). |
| **Microsoft Integration Runtime** | Installez [Microsoft Integration Runtime](https://aka.ms/sql-migration-shir-download). |

## Restaurer une base de données SQL Server

Nous allons restaurer la base de données *AdventureWorksLT* sur l’instance SQL Server. Cette base de données servira de base de données source pour cet exercice de labo. Passez ces étapes si la base de données est déjà restaurée.

1. Sélectionnez le bouton Démarrer de Windows et tapez SSMS. Sélectionnez **Microsoft SQL Server Management Studio 18** dans la liste.  

1. Lorsque SSMS s’ouvre, notez que la boîte de dialogue **Se connecter au serveur** est préremplie avec le nom de l’instance par défaut. Sélectionnez **Se connecter**.

1. Sélectionnez le dossier **Bases de données**, puis **Nouvelle requête**.

1. Dans la fenêtre de la nouvelle requête, copiez et collez le code T-SQL ci-dessous. Vérifiez que le nom et le chemin d’accès du fichier de sauvegarde de la base de données correspondent à votre fichier de sauvegarde actuel. Dans le cas contraire, la commande échouera. Exécutez la requête pour restaurer la base de données.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\<FolderName>\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\<FolderName>\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\<FolderName>\AdventureWorksLT2019.ldf';
    ```

    > **Remarque** : Veillez à avoir le fichier de sauvegarde [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure#download-backup-files) léger sur la machine SQL Server avant d’exécuter la commande T-SQL.

1. Un message de réussite doit s’afficher une fois la restauration terminée.

## Inscrire l’espace de noms Microsoft.DataMigration

Passez ces étapes si l’espace de noms **Microsoft.DataMigration** est déjà inscrit sur votre abonnement.

1. Dans le portail Azure, recherchez **Abonnement** dans la zone de recherche en haut, puis sélectionnez **Abonnements**. Sélectionnez votre abonnement dans le panneau **Abonnements**.

1. Dans la page de votre abonnement, sous **Paramètres**, sélectionnez **Fournisseurs de ressources**. Recherchez **Microsoft.DataMigration** dans la zone de recherche en haut, puis sélectionnez **Microsoft.DataMigration**. 

    > **Remarque** : si la barre latérale **Détails du fournisseur de ressources** s’ouvre, vous pouvez la fermer.

1. Sélectionnez **Inscrire**.

## Provisionner une base de données Azure SQL

Nous allons configurer une base de données Azure SQL qui servira d’environnement cible.

1. Dans le portail Azure, recherchez **bases de données SQL** dans la zone de recherche en haut, puis sélectionnez **Base de données SQL**.

1. Dans le panneau **Bases de données SQL**, sélectionnez **+ Créer**.

1. Dans la page **Créer une base de données SQL**, sélectionnez les options suivantes.

    - **Abonnement** : &lt;votre abonnement&gt;.
    - **Groupe de ressources :**&lt;votre groupe de ressources&gt;.
    - **Nom de la base de données** : AdventureWorksLT.
    - **Serveur :**  sélectionnez le lien **Créer nouveau**. Fournissez les détails du serveur sur la page **Créer un serveur SQL Database**.
        - **Nom du serveur :**&lt;choisissez le nom du serveur&gt;. Le nom du serveur doit être globalement unique.
        - **Emplacement :**&lt;même région que votre groupe de ressources&gt;
        - **Méthode d’authentification :** utilisez l’authentification SQL.
        - **Connexion administrateur au serveur :** sqladmin
        - **Mot de passe :**&lt;votre mot de passe&gt;
        - **Confirmer le mot de passe :**&lt;votre mot de passe&gt;
        
            **Remarque :** notez le nom du serveur et les informations d’identification. Ils vous seront demandés dans les étapes suivantes.

    - **Vous souhaitez utiliser un pool élastique SQL ?** Non
    - **Environnement de charge de travail :** production

1. Dans **Calcul + stockage**, sélectionnez **Configurer la base de données**. Dans la page **Configurer**, pour la liste déroulante **Niveau de service**, sélectionnez **De base**, puis **Appliquer**.

1. Pour l’option **Redondance du stockage de sauvegarde**, conservez la valeur par défaut : **stockage de sauvegarde géoredondant**. Sélectionnez **Vérifier + créer**.

1. Passez en revue les paramètres, puis sélectionnez **Créer**.

1. Une fois le déploiement effectué, sélectionnez **Accéder à la ressource**.

## Activer l’accès à une base de données Azure SQL.

Nous allons activer l’accès à une base de données Azure SQL afin de pouvoir nous y connecter via les outils clients.

1. Dans la page **Base de données SQL**, sélectionnez la section **Vue d’ensemble**, puis cliquez sur le lien du nom du serveur dans la section supérieure :

1. Dans le panneau **SQL Server**, sous la section **Sécurité**, sélectionnez **Mise en réseau**.

1. Sous l’onglet **Accès public**, sélectionnez **Réseaux sélectionnés**. 

1. Sous **Règles de pare-feu**, sélectionnez **+ Ajouter l’adresse IPv4 de votre client**.

1. Sous **Exceptions**, sélectionnez la propriété **Autoriser les services et les ressources Azure à accéder à ce serveur**. 

1. Sélectionnez **Enregistrer**.

## Se connecter à Azure SQL Database dans Azure Data Studio

Avant de commencer à utiliser l’extension de migration Azure, nous allons nous connecter à la base de données cible.

1. Lancez Azure Data Studio.

1. Sélectionnez **Connexions**, puis **Ajouter une connexion**.

1. Renseignez les champs des **Détails de la connexion** avec le nom du serveur SQL Server et d’autres informations.

    > **Remarque** : entrez le nom du serveur SQL Server créé précédemment. Le format doit correspondre à : **<server>.database.windows.net**.

1. Dans le **Type d’authentification**, sélectionnez **Connexion SQL** et indiquez le nom d’utilisateur et le mot de passe.

1. Sélectionnez **Se connecter**.

## Installer et lancer l’extension de migration Azure pour Azure Data Studio

Pour installer l’extension de migration, suivez les étapes suivantes. Si l’extension est déjà installée, ignorez ces étapes.

1. Ouvrez le gestionnaire d’extensions dans Azure Data Studio.

1. Recherchez ***Azure SQL Migration***, puis sélectionnez l’extension.

1. Installez l’extension. Une fois installée, l’extension de migration Azure SQL s'affiche dans la liste des extensions installées.

1. Connectez-vous à une instance SQL Server dans Azure Data Studio. Dans le nouvel onglet connexion, en regard de l’option de **Chiffrement**, sélectionnez **Facultatif (False)**.

1. Pour lancer l’extension de migration Azure, cliquez avec le bouton droit sur le nom de l’instance SQL Server, puis sélectionnez **Gérer** pour accéder au tableau de bord et à la page d’arrivée de l’extension de migration Azure SQL.

    > **Remarque** : si **Migration Azure SQL** n’est pas visible dans la barre latérale du tableau de bord du serveur, rouvrez Azure Data Studio.
 
## Effectuer une migration hors connexion d’une base de données SQL Server vers Azure SQL Database

Nous sommes maintenant prêts à migrer les données. Pour effectuer une migration hors connexion en utilisant Azure Data Studio, suivez les étapes ci-dessous.

1. Lancez l’Assistant **Migration vers Azure SQL** de l’extension dans Azure Data Studio, puis sélectionnez **Migrer vers Azure SQL**.

1. À l’**Étape 1 : Bases de données pour l’évaluation**, sélectionnez la base de données *AdventureWorksLT*, puis sélectionnez **Suivant**.

1. À l’**Étape 2 : Résumé de l’évaluation et recommandations SKU**, attendez la fin de l’évaluation, puis passez en revue les résultats. Cliquez sur **Suivant**.

1. À l’**Étape 3 : Résultats de l’évaluation et de la plateforme cible**, sélectionnez **Azure SQL Database** comme type de cible. Après avoir passé en revue les résultats de l’évaluation, sélectionnez **Suivant**.

1. À l’**Étape 4 : Cible Azure SQL**, si le compte n’est pas encore lié, veillez à ajouter un compte en sélectionnant le lien **Lier un compte**. Sélectionnez ensuite un compte Azure, un locataire Microsoft Entra, un abonnement, un emplacement, un groupe de ressources, un serveur Azure SQL Database et les informations d’identification d’Azure SQL Database.

1. Sélectionnez **Connecter**, puis sélectionnez la base de données *AdventureWorksLT* comme **Base de données cible**. Cliquez sur **Suivant**.

1. À l’**Étape 5 : Azure Database Migration Service**, sélectionnez le lien **Créer** pour créer une instance d’Azure Database Migration Service à l’aide de l’Assistant. Suivez les étapes fournies par l’Assistant pour configurer un nouveau runtime d’intégration auto-hébergé. Si vous en avez créé un précédemment, réutilisez-le.

1. À l’**Étape 6 : Configuration de la source de données**, entrez les informations d’identification appropriées pour vous connecter à l’instance SQL Server à partir du runtime d’intégration auto-hébergé.

1. Sélectionnez **Modifier** pour la base de données *AdventureWorksLT*. 

1. Cochez l’option **Migrer le schéma vers la cible**, puis vérifiez que toutes les tables figurant sous l’onglet **Manquant sur la cible** sont sélectionnées. Sélectionnez **Mettre à jour**.

1. Sélectionnez **Exécuter la validation**.

    ![Capture d’écran de l’étape d’exécution de la validation dans l’extension de migration Azure pour Azure Data Studio.](../media/3-run-validation.png) 

1. Une fois la validation terminée, sélectionnez **Terminé**.

1. Cliquez sur **Suivant**.

1. À l’**Étape 7 : Résumé**, sélectionnez **Démarrer la migration**.

1. Sélectionnez **Migrations de bases de données en cours** dans le tableau de bord de la migration pour afficher l’état des migrations. 

    ![Capture d’écran du tableau de bord de la migration dans l’extension de migration Azure pour Azure Data Studio.](../media/3-data-migration-dashboard.png)

1. Sélectionnez le nom de la base de données *AdventureWorks* pour afficher plus de détails.

    ![Capture d’écran des détails de la migration dans l’extension de migration Azure pour Azure Data Studio.](../media/3-dashboard-sqldb.png)

1. Lorsque l’état de la migration est **Réussie**, accédez au serveur cible et validez la base de données cible. Vérifiez le schéma de la base de données et les données migrées.

Vous avez appris à installer l’extension de migration et à générer le schéma de la base de données à l’aide de l’Assistant Migration de données. Vous avez également appris à migrer une base de données SQL Server vers Azure SQL Database en utilisant l’extension de migration Azure pour Azure Data Studio. Une fois la migration terminée, vous pouvez commencer à utiliser votre nouvelle ressource Azure SQL Database. 

## Nettoyage

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. 

Laisser inutilement des ressources en cours d’exécution peut entraîner des coûts supplémentaires. Vous pouvez supprimer les ressources individuellement ou supprimer le groupe de ressources dans le [Portail Azure](https://portal.azure.com?azure-portal=true).

## Plus d’informations

Pour plus d’informations sur Azure SQL Database, consultez l’article [En quoi consiste Azure SQL Database ?](https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview)
