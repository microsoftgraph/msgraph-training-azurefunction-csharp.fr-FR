---
ms.openlocfilehash: 914957809f268ad29f8cfc44c21dd9fb63699322
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445961"
---
<!-- markdownlint-disable MD002 MD041 -->

Dans ce didacticiel, vous allez créer une fonction Azure simple qui implémente des fonctions de déclencheur HTTP qui appellent Microsoft Graph. Ces fonctions couvrent les scénarios suivants :

- Implémente une API pour accéder à la boîte de réception d’un utilisateur à l’aide de [l’authentification de flux](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) « de la part de ».
- Implémente une API pour s’abonner et se désabonner pour les notifications dans la boîte de réception d’un utilisateur, à l’aide de l’authentification de flux d’octroi d’informations d’identification [client.](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)
- Implémente un webhook pour recevoir des [notifications](https://docs.microsoft.com/graph/webhooks) de modification de Microsoft Graph et accéder aux données à l’aide du flux d’octroi d’informations d’identification client.

Vous allez également créer une application Mono-page JavaScript simple pour appeler les API implémentées dans la fonction Azure.

## <a name="create-azure-functions-project"></a>Créer un projet Azure Functions

1. Ouvrez votre interface de ligne de commande (CLI) dans un répertoire où vous souhaitez créer le projet. Exécutez la commande suivante :

    ```Shell
    func init GraphTutorial --worker-runtime dotnetisolated
    ```

1. Modifiez le répertoire actuel de votre CLI en répertoire **GraphTutorial** et exécutez les commandes suivantes pour créer trois fonctions dans le projet.

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger"
    func new --name SetSubscription --template "HTTP trigger"
    func new --name Notify --template "HTTP trigger"
    ```

1. Ouvrez **local.settings.jset** ajoutez ce qui suit au fichier pour autoriser CORS à partir `http://localhost:8080` de , l’URL de l’application de test.

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. Exécutez la commande suivante pour exécuter le projet localement.

    ```Shell
    func start
    ```

1. Si tout fonctionne, vous verrez le résultat suivant :

    ```Shell
    Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. Vérifiez que les fonctions fonctionnent correctement en ouvrant votre navigateur et en naviguant vers les URL de fonction affichées dans la sortie. Vous devriez voir le message suivant dans votre navigateur : `Welcome to Azure Functions!` .

## <a name="create-single-page-application"></a>Créer une application à page unique

1. Ouvrez votre CLI dans un répertoire où vous souhaitez créer le projet. Créez un répertoire nommé **TestClient** pour contenir vos fichiers HTML et JavaScript.

1. Créez un fichier nommé **index.html** dans le répertoire **TestClient** et ajoutez le code suivant.

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    Cela définit la disposition de base de l’application, y compris une barre de navigation. Il ajoute également ce qui suit :

    - [Bootstrap](https://getbootstrap.com/) et son javaScript de prise en charge
    - [FontAwesome](https://fontawesome.com/)
    - [Bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) 2.0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > La page inclut une favorite, ( `<link rel="shortcut icon" href="g-raph.png">` ). Vous pouvez supprimer cette ligne ou télécharger le fichier **g-raph.png** à partir de [GitHub](https://github.com/microsoftgraph/g-raph).

1. Créez un fichier nommé **style.css** dans le répertoire **TestClient** et ajoutez le code suivant.

    :::code language="css" source="../demo/TestClient/style.css":::

1. Créez un fichier nommé **ui.js** dans le **répertoire TestClient** et ajoutez le code suivant.

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    Ce code utilise JavaScript pour afficher la page actuelle en fonction de l’affichage sélectionné.

### <a name="test-the-single-page-application"></a>Tester l’application à page unique

> [!NOTE]
> Cette section contient des instructions sur l’utilisation [de dotnet-serve](https://github.com/natemcmaster/dotnet-serve) pour exécuter un serveur HTTP de test simple sur votre ordinateur de développement. L’utilisation de cet outil spécifique n’est pas requise. Vous pouvez utiliser n’importe quel serveur de test de votre préférence pour servir le **répertoire TestClient.**

1. Exécutez la commande suivante dans votre CLI pour installer **dotnet-serve**.

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. Modifiez le répertoire actuel de votre CLI en répertoire **TestClient** et exécutez la commande suivante pour démarrer un serveur HTTP.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. Ouvrez votre navigateur et accédez à `http://localhost:8080`. La page doit s’restituer, mais aucun des boutons ne fonctionne actuellement.

## <a name="add-nuget-packages"></a>Ajouter des packages NuGet

Avant de passer à la suite, installez des packages NuGet supplémentaires que vous utiliserez ultérieurement.

- [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) pour activer l’injection de dépendances dans le projet Fonctions Azure.
- [Microsoft.Extensions.Config'uration. UserSecrets pour](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) lire la configuration de l’application à partir du magasin de secrets [de développement .NET](https://docs.microsoft.com/aspnet/core/security/app-secrets).
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) pour effectuer des appels Microsoft Graph.
- [Microsoft.Identity.Client pour](https://www.nuget.org/packages/Microsoft.Identity.Client/) l’authentification et la gestion des jetons.
- [Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) pour récupérer la configuration OpenID pour la validation du jeton.
- [System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) pour la validation des jetons envoyés à l’API web.

1. Modifiez le répertoire actuel de votre CLI en répertoire **GraphTutorial** et exécutez les commandes suivantes.

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.1.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 5.0.0
    dotnet add package Microsoft.Graph --version 4.0.0
    dotnet add package Microsoft.Identity.Client --version 4.34.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.11.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.11.1
    ```
