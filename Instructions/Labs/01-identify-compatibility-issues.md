---
lab:
  title: Identifier les problèmes de compatibilité dans la migration SQL
---

# Identifier les problèmes de compatibilité dans la migration SQL

Dans notre scénario, vous avez été chargé d’évaluer la préparation d’une base de données SQL Server héritée en vue de sa migration vers Azure SQL Database. Votre tâche consiste à évaluer la base de données héritée et à identifier les éventuels problèmes de compatibilité ou les modifications à apporter avant la migration. Vous devez également passer en revue le schéma de la base de données et identifier les fonctionnalités ou configurations qui ne sont pas prises en charge dans Azure SQL Database.

Cet exercice prend environ **15** minutes.

> **Remarque** : pour effectuer cet exercice, vous devez avoir accès à un abonnement Azure pour créer des ressources Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?azure-portal=true) avant de commencer.

## Avant de commencer

Avant de commencer cet exercice, vérifiez que vous remplissez les prérequis suivants :

- Vous aurez besoin de SQL Server 2019 ou d’une version ultérieure, ainsi que la base de données légère [**AdventureWorksLT**](https://learn.microsoft.com/sql/samples/adventureworks-install-configure#download-backup-files) compatible avec votre instance SQL Server spécifique.
- Téléchargez et installez [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio). Si l'application est déjà installée, mettez-la à jour pour vous assurer que vous utilisez la version la plus récente.
- Un utilisateur SQL disposant d’un accès en lecture à la base de données source.

## Restaurer une base de données SQL Server et exécuter une commande

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

1. Exécutez la commande suivante sur la base de données **AdventureWorksLT** dans l’instance SQL Server.

```sql
ALTER TABLE [SalesLT].[Customer] ADD [Next] VARCHAR(5);
```

## Installer et lancer l’extension de migration Azure pour Azure Data Studio

Pour installer l’extension de migration, suivez les étapes suivantes. Si l’extension de migration Azure est déjà installée, vous pouvez ignorer ces étapes.

1. Ouvrez le gestionnaire d’extensions dans Azure Data Studio. s

1. Recherchez ***Azure SQL Migration***, puis installez l’extension. Une fois installée, l’extension de migration Azure SQL s'affiche dans la liste des extensions installées.

1. Sélectionnez l'icône **Connexions**, puis **Nouvelle connexion**. 

1. Dans le nouvel onglet **Connexion**, tapez le nom du serveur. Sélectionnez **Facultatif (False)** sous l’option **Chiffrer**.

1. Sélectionnez **Se connecter**. 

1. Pour lancer l’extension de migration Azure, cliquez avec le bouton droit sur le nom de l’instance source, puis sélectionnez **Gérer**. 

1. Dans le menu du serveur, sous **Général**, sélectionnez **Migration Azure SQL**. Vous accédez à la page principale de l’extension de migration Azure SQL.

    > **Remarque** : si vous ne parvenez pas à voir l’option **Migration Azure SQL** dans le menu du serveur, ou si la page Migration Azure SQL ne se charge pas, rouvrez Azure Data Studio.

## Effectuer l’évaluation de la compatibilité

L’évaluation de compatibilité permet d’identifier les problèmes de migration potentiels et fournit des instructions détaillées sur la façon de les résoudre avant de commencer le processus de migration. Cela permet de gagner beaucoup de temps et de ressources. 

Vous allez exécuter l’extension de migration Azure pour Azure Data Studio, exécuter l’évaluation de compatibilité, puis afficher les résultats d'une base de données Azure SQL cible.

1. Dans le tableau de bord de l’extension de migration Azure SQL, sélectionnez **Migrer vers Azure SQL** pour ouvrir l’Assistant Migration.

1. Dans **Étape 1 : Bases de données pour l’évaluation**, sélectionnez la base de données *AdventureWorks*, puis sélectionnez **Suivant**.

1. Dans **Étape 2 : Résultats de l’évaluation et recommandations**, attendez la fin du processus d’évaluation.

## Examiner les résultats d’évaluation

Vous pouvez maintenant évaluer les suggestions générées par l’extension de migration.

1. Dans **Étape 2 : résultats de l’évaluation et recommandations**, sélectionnez **Azure SQL Database** comme plateforme cible.

1. En bas de la page, sélectionnez **Afficher/Sélectionner** pour voir les résultats de l’évaluation. 

1. Sélectionnez la base de données *AdventureWorks*. Prenez un moment pour examiner les résultats de l’évaluation affichés à droite.
    
    > **Remarque :** nous voyons que la colonne `Next` qui a été ajoutée précédemment a été marquée d’un indicateur, car elle peut provoquer une erreur dans Azure SQL Database.

1. Sélectionnez **Annuler** et choisissez **Azure SQL Managed Instance** à la place comme plateforme cible **Azure SQL**.
    
    > **Remarque :** la colonne `Next` n’est plus marquée pour Azure SQL Managed Instance. Pourquoi ? 
    >
    >Cela signifie que la colonne `Next` peut être utilisée sans problème sur Azure SQL Managed Instance.

1. Sélectionnez **Enregistrer le rapport d’évaluation** pour enregistrer le rapport au format JSON.

1. Prenez un moment pour passer en revue le fichier JSON et ses propriétés.

## Résoudre le problème

1. Exécutez la commande T-SQL suivante sur la base de données *AdventureWorks*.

    ```sql
    ALTER TABLE [SalesLT].[Customer] DROP COLUMN [Next];
    ```

1. Revenez à la page **Étape 2 : résultats de l’évaluation et recommandations** dans l’Assistant, puis sélectionnez **Actualiser l’évaluation**.

1. Sélectionnez **Azure SQL Database** comme plateforme cible **Azure SQL**.

1. Sélectionnez **Afficher/Sélectionner** pour afficher les résultats de l’évaluation.

    > **Remarque :** le problème n’est plus marqué.

Vous avez appris à évaluer la préparation d’une base de données SQL Server en vue de sa migration vers Azure SQL Database. En traitant les problèmes de compatibilité et en apportant des modifications de schéma essentielles, ou en les signalant, vous avez franchi une étape importante pour atténuer les problèmes techniques potentiels qui pourraient survenir à l’avenir sur Azure SQL Database.
