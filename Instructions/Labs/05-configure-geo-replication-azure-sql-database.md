---
lab:
  title: "Configurer la géoréplication pour Azure\_SQL\_Database"
---

# Configurer la géoréplication pour Azure SQL Database

Dans cet exercice, vous allez apprendre à activer la géoréplication pour une base de données Azure SQL et à effectuer un basculement vers une région secondaire. Pour ce faire, vous devez créer un réplica de votre base de données, configurer un nouveau serveur pour la base de données secondaire et lancer un basculement forcé. Vous allez également apprendre à vérifier l’état de vos déploiements et à comprendre le rôle des bases de données géosecondaires ou géoréplicas dans la gestion d’Azure SQL Database. Enfin, vous allez basculer manuellement la base de données vers une autre région à l’aide du portail Azure. Cet exercice offre une expérience pratique des aspects clés de la gestion et de la résilience de vos bases de données Azure SQL.

Cet exercice prend environ **30** minutes.

> **Remarque** : pour effectuer cet exercice, vous devez avoir accès à un abonnement Azure pour créer des ressources Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?azure-portal=true) avant de commencer.

## Avant de commencer

Pour effectuer cet exercice, nous allons utiliser de nombreuses ressources et outils. Examinons-les de plus près :

|  | Description |
| --- | --- |
| **Serveur primaire** | Un serveur Azure SQL Database que nous allons configurer dans ce labo.|
| **Base de données primaire** | L’exemple de base de données **AdventureWorksLT** créé sur le serveur secondaire.|
| **Serveur secondaire** | Un serveur Azure SQL Database supplémentaire que nous allons configurer dans ce labo. |
| **Base de données secondaire** | Il s’agit de notre réplica de base de données sur le serveur secondaire. |
| **SQL Server Management Studio** | Téléchargez et installez la dernière version de [SQL Server Management Studio](https://learn.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) sur la machine de votre choix. |

## Approvisionner des ressources Azure SQL Database

Nous allons créer les ressources Azure SQL Database en deux étapes. Tout d’abord, nous allons établir le serveur primaire et la base de données. Ensuite, nous allons répéter le processus pour configurer le serveur secondaire sous un autre nom. Nous obtiendrons ainsi deux serveurs Azure SQL, chacun disposant de ses propres règles de pare-feu. Toutefois, seul le serveur primaire possède une base de données.

1. Accédez au [portail Azure](https://portal.azure.com) et connectez-vous à l’aide des informations d’identification de votre compte Azure.

1. Sélectionnez l’option **Cloud Shell** dans la barre de menus en haut à droite (similaire à une invite d’interpréteur de commandes **`>_`**).

1. Un volet s’affiche en bas de l’écran et vous demande de choisir votre type d’interpréteur de commandes préféré. Sélectionnez **Bash**.

1. S’il s’agit de la première fois que vous lancez **Cloud Shell**, vous êtes invité à créer un compte de stockage (utilisé pour conserver vos données entre les sessions). Suivez les invites pour en créer un.

1. Une fois l’interpréteur de commandes lancé, vous disposez d’une interface de ligne de commande directement dans le portail Azure, où vous pouvez entrer vos commandes de script.

1. Sélectionnez **{}** pour ouvrir l’éditeur et copier et coller le script ci-dessous. 
 
    > **Remarque** : veillez à remplacer les valeurs d’espace réservé dans le script par vos valeurs réelles avant de l’exécuter. Si vous devez modifier le script, tapez `code` dans **Cloud Shell** pour utiliser l’éditeur de texte intégré.
        
    ```powershell
    subscription="<Your subscription>"
    resourceGroup="<Your resource group>"
    location="<Your region, same as your resource group>"
    serverName="<Your SQL server name>"
    adminLogin="sqladmin"
    password="<password>"
    databaseName="AdventureWorksLT"
    
    az account set --subscription $subscription
    az sql server create --name $serverName --resource-group $resourceGroup --location $location --admin-user $adminLogin --admin-password $password
    az sql db create --resource-group $resourceGroup --server $serverName --name $databaseName --sample-name AdventureWorksLT --service-objective Basic

    ```
    Ce script Azure CLI définit l’abonnement Azure actif, crée un serveur Azure SQL et une base de données Azure SQL contenant les exemples de données AdventureWorksLT.

1. Cliquez avec le bouton droit sur la page de l’éditeur et sélectionnez **Enregistrer**.

1. Donnez un nom au fichier. L’extension du fichier doit être **.ps1**.

1. Sur le terminal Cloud Shell, tapez et exécutez la commande.

    ```bash
    chmod +x <script_name>.ps1

    ```
    
    Remplacez *<script_name>* par le nom que vous avez donné pour le script. Cette commande modifie les autorisations du fichier que vous avez créé pour l’exécuter.

1. Exécutez le script. 
    
    ```powershell
    ./<script_name>.ps1

    ```

1. Une fois le processus terminé, accédez au serveur Azure SQL que vous venez de créer en vous rendant sur le portail Azure, puis sur la page de votre serveur SQL Server. 

1. Dans la page principale de votre serveur Azure SQL Server, sélectionnez **Mise en réseau** sur la gauche.

1. Sous l’onglet **Accès public**, sélectionnez **Réseaux sélectionnés**.

1. Dans la section **Règles de pare-feu**, sélectionnez **+ Ajouter l’adresse IPv4 de votre client**. Tapez votre adresse IP, puis sélectionnez **Enregistrer**.

    ![Capture d’écran de la page de règle de pare-feu pour Azure SQL Database.](../media/5-new-firewall-rule.png)

    À ce stade, vous devez être en mesure de vous connecter à la base de données primaire `AdventureWorksLT` via un outil client tel que SQL Management Studio.

1. Nous allons maintenant créer un serveur Azure SQL secondaire. Répétez les étapes précédentes (6 à 14), mais veillez à utiliser un autre `serverName` et `location`. En outre, ignorez le code qui crée la base de données en commentant la commande `az sql db create`. Cette procédure permet de créer un serveur dans une autre région, sans l’exemple de base de données.

## Activer la géoréplication

À présent, nous allons créer le réplica secondaire pour nos ressources Azure SQL.

1. Dans le portail Azure, accédez à votre base de données en recherchant les **bases de données SQL**.

1. Sélectionnez la base de données SQL **AdventureWorksLT**.

1. Dans la page principale de votre base de données Azure SQL, sélectionnez **Réplicas** sous **Gestion des données** à gauche.

1. Sélectionnez **+ Créer un réplica**.

1. Dans la page **Créer une base de données SQL - Géoréplica**, sous **Serveur**, sélectionnez le serveur SQL Server secondaire créé précédemment.

1. Sélectionnez **Vérifier + créer**, puis **Créer**. La base de données secondaire est alors créée et remplie. Pour vérifier l’état, regardez sous l’icône de notification en haut du portail Azure. 

1. Si l’opération a réussi, il passe de **Déploiement en cours** à **Déploiement effectué**.

1. Connectez-vous à votre serveur SQL Server secondaire en utilisant SQL Management Studio.

## Basculer une base de données SQL vers une région secondaire

Imaginez un scénario où la base de données Azure SQL primaire rencontre des problèmes en raison d’une panne régionale. Pour garantir la continuité de vos services et réduire les temps d’arrêt, vous devez effectuer un basculement forcé.

Le basculement forcé permet d’inverser les rôles des bases de données primaires et secondaires. La base de données secondaire prend le relais en tant que base de données primaire, et la base de données primaire d’origine devient la base de données secondaire. Cela permet à vos applications de continuer à fonctionner à l’aide du réplica secondaire pendant que les problèmes liés à la base de données primaire d’origine sont résolus.

Apprenons à lancer un basculement forcé suite à une panne dans une région.

1. Accédez à la page des serveurs SQL Server, puis sélectionnez le serveur secondaire.

1. Dans la section **Paramètres** à gauche, sélectionnez **Bases de données SQL**.

1. Dans la page principale de votre base de données Azure SQL, sélectionnez **Réplicas** sous **Gestion des données** à gauche. Le lien de géoréplication est maintenant établi.

1. Sélectionnez le menu **...** du serveur secondaire, puis cliquez sur **Basculement forcé**.

    > **Remarque** : le basculement forcé fait passer la base de données secondaire au rôle primaire. Toutes les sessions sont prennent fin pendant cette opération.

1. Lorsque vous y êtes invité par le message d’avertissement, sélectionnez **Oui**.

1. L’état du réplica principal passe à **En attente**, et le secondaire à **Basculement**. 

    > **Remarque** : cette opération peut prendre quelques minutes. Une fois l’opération terminée, les rôles sont inversés : le serveur secondaire devient le nouveau serveur primaire et le serveur primaire d’origine se transforme en serveur secondaire.

Tenez compte des raisons pour lesquelles vous souhaitez placer vos serveurs SQL Server primaires et secondaires dans la même région, et quand il peut être utile de choisir différentes régions.

Vous venez de voir comment activer la géoréplication pour Azure SQL Database et comment la faire basculer manuellement vers une autre région à l’aide du portail Azure.

## Nettoyer

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. 

Laisser inutilement des ressources en cours d’exécution peut entraîner des coûts supplémentaires. Vous pouvez supprimer les ressources individuellement ou supprimer le groupe de ressources dans le [Portail Azure](https://portal.azure.com?azure-portal=true).

## Plus d’informations

Pour plus d’informations sur la géoréplication pour Azure SQL Database, consultez l’article [Géoréplication active](https://review.learn.microsoft.com/azure/azure-sql/database/active-geo-replication-overview).