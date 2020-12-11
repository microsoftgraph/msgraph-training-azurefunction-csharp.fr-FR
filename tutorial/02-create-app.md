---
ms.openlocfilehash: 17c93f353c84ea2db28cd2e0203d30c5f320e36e
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655232"
---
<!-- markdownlint-disable MD002 MD041 -->

Dans ce didacticiel, vous allez créer une fonction Azure simple qui implémente les fonctions de déclencheur HTTP qui appellent Microsoft Graph. Ces fonctions couvrent les scénarios suivants :

- Implémente une API pour accéder à la boîte de réception d’un utilisateur en utilisant l’authentification [de flux de la part de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) .
- Implémente une API pour s’abonner et annuler l’abonnement aux notifications dans la boîte de réception d’un utilisateur, à l’aide de l’authentification de [flux d’octroi d’informations d’identification client](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) .
- Implémente un webhook pour recevoir [des notifications de modification](https://docs.microsoft.com/graph/webhooks) de Microsoft Graph et accéder à des données à l’aide du flux d’octroi d’informations d’identification du client.

Vous allez également créer une application JavaScript simple à page unique (SPA) pour appeler les API implémentées dans la fonction Azure.

## <a name="create-azure-functions-project"></a>Créer un projet de fonctions Azure

1. Ouvrez votre interface de ligne de commande (CLI) dans un répertoire où vous souhaitez créer le projet. Exécutez la commande suivante.

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. Remplacez le répertoire actuel de votre interface CLI par le répertoire **GraphTutorial** et exécutez les commandes suivantes pour créer trois fonctions dans le projet.

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. Ouvrez **local.settings.jssur** et ajoutez le code suivant au fichier pour autoriser cors à partir de `http://localhost:8080` , l’URL de l’application de test.

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. Exécutez la commande suivante pour exécuter le projet localement.

    ```Shell
    func start
    ```

1. Si tout fonctionne, vous verrez le résultat suivant :

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. Vérifiez que les fonctions fonctionnent correctement en ouvrant votre navigateur et en accédant aux URL de fonction indiquées dans la sortie. Le message suivant s’affiche dans votre navigateur : `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` .

## <a name="create-single-page-application"></a>Créer une application à page unique

1. Ouvrez l’interface de commande dans le répertoire dans lequel vous souhaitez créer le projet. Créez un répertoire nommé **TestClient** pour stocker vos fichiers HTML et JavaScript.

1. Créez un fichier nommé **index.html** dans le répertoire **TestClient** et ajoutez le code suivant.

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    Cette définition définit la disposition de base de l’application, y compris une barre de navigation. Il ajoute également les éléments suivants :

    - [Bootstrap](https://getbootstrap.com/) et son code JavaScript de prise en charge
    - [FontAwesome](https://fontawesome.com/)
    - [Bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) 2,0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > La page inclut un favorite, ( `<link rel="shortcut icon" href="g-raph.png">` ). Vous pouvez supprimer cette ligne, ou vous pouvez télécharger le fichier **g-raph.png** à partir de [GitHub](https://github.com/microsoftgraph/g-raph).

1. Créez un fichier nommé **style. CSS** dans le répertoire **TestClient** et ajoutez le code suivant.

    :::code language="css" source="../demo/TestClient/style.css":::

1. Créez un fichier nommé **ui.js** dans le répertoire **TestClient** et ajoutez le code suivant.

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    Ce code utilise JavaScript pour afficher la page actuelle en fonction de la vue sélectionnée.

### <a name="test-the-single-page-application"></a>Tester l’application à page unique

> [!NOTE]
> Cette section contient des instructions sur l’utilisation de [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) pour exécuter un serveur http de test simple sur votre ordinateur de développement. L’utilisation de cet outil spécifique n’est pas obligatoire. Vous pouvez utiliser n’importe quel serveur d’évaluation pour servir le répertoire **TestClient** .

1. Exécutez la commande suivante dans votre interface CLI pour installer **dotnet-serve**.

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. Remplacez le répertoire actuel de votre interface CLI par le répertoire **TestClient** et exécutez la commande suivante pour démarrer un serveur http.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. Ouvrez votre navigateur et accédez à `http://localhost:8080`. La page doit s’afficher, mais aucun bouton ne fonctionne.

## <a name="add-nuget-packages"></a>Ajouter des packages NuGet

Avant de poursuivre, installez des packages NuGet supplémentaires que vous utiliserez plus tard.

- [Microsoft. Azure. Functions. extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) pour activer l’injection de dépendance dans le projet de fonctions Azure.
- [Microsoft.Extensions.Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) pour lire la configuration de l’application à partir du [magasin de secrets de développement .net](https://docs.microsoft.com/aspnet/core/security/app-secrets).
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) pour effectuer des appels Microsoft Graph.
- [Microsoft. Identity. client](https://www.nuget.org/packages/Microsoft.Identity.Client/) pour l’authentification et la gestion des jetons.
- [Microsoft. IdentityModel. Protocols. OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) pour récupérer la configuration de OpenID pour la validation de jeton.
- [System. IdentityModel. Tokens. JWT](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) pour la validation des jetons envoyés à l’API Web.

1. Remplacez le répertoire actuel de votre interface CLI par le répertoire **GraphTutorial** et exécutez les commandes suivantes.

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
