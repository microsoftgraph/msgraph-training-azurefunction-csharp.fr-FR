---
ms.openlocfilehash: 3e6a83c19de68b6047914a68d94e66dbab95c6c4
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445954"
---
<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous terminerez l’implémentation de la fonction Azure et mettrez à jour le `GetMyNewestMessage` client de test pour appeler la fonction.

La fonction Azure utilise le flux « de [la part de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)». L’ordre de base des événements dans ce flux est le :

- L’application de test utilise un flux d’th interactive pour permettre à l’utilisateur de se connecter et d’accorder son consentement. Elle obtient un jeton qui est étendue à la fonction Azure. Le jeton ne **contient pas** d’étendues Graph Microsoft.
- L’application de test appelle la fonction Azure, en envoyant son jeton d’accès dans `Authorization` l’en-tête.
- La fonction Azure valide le jeton, puis échange ce jeton contre un deuxième jeton d’accès qui contient les étendues Graph Microsoft.
- La fonction Azure appelle Microsoft Graph au nom de l’utilisateur à l’aide du deuxième jeton d’accès.

> [!IMPORTANT]
> Pour éviter de stocker l’ID d’application et la secret dans la source, vous utiliserez le gestionnaire de secret [.NET](https://docs.microsoft.com/aspnet/core/security/app-secrets) pour stocker ces valeurs. Le Gestionnaire de secret est uniquement à des fins de développement, les applications de production doivent utiliser un gestionnaire de secret approuvé pour stocker les secrets.

## <a name="add-authentication-to-the-single-page-application"></a>Ajouter l’authentification à l’application à page unique

Commencez par ajouter l’authentification à la SPA. Cela permettra à l’application d’obtenir un jeton d’accès accordant l’accès pour appeler la fonction Azure. Comme il s’agit d’une SPA, elle utilise le flux de [code d’autorisation avec PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).

1. Créez un fichier dans le **répertoire TestClient** nommé **config.js** et ajoutez le code suivant.

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    Remplacez par l’ID d’application que vous avez créé dans le portail `YOUR_TEST_APP_APP_ID_HERE` Azure **pour Graph’application test de fonction Azure.** Remplacez `YOUR_TENANT_ID_HERE` par la **valeur d’ID d’annuaire (client)** que vous avez copiée à partir du portail Azure. Remplacez `YOUR_AZURE_FUNCTION_APP_ID_HERE` par l’ID d’application **pour la Graph Azure Function**.

    > [!IMPORTANT]
    > Si vous utilisez un contrôle source tel que Git, il est temps d’exclure le fichier **config.js** du contrôle source afin d’éviter toute fuite accidentelle de vos ID d’application et ID de client.

1. Créez un fichier dans le **répertoire TestClient** nommé **auth.js** et ajoutez le code suivant.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    Prenez en compte ce que fait ce code.

    - Il initialise une à `PublicClientApplication` l’aide des valeurs stockées dans **config.js**.
    - Il utilise pour `loginPopup` se connecter à l’utilisateur, à l’aide de l’étendue d’autorisation pour la fonction Azure.
    - Il stocke le nom d’utilisateur de l’utilisateur dans la session.

    > [!IMPORTANT]
    > Étant donné que l’application utilise , vous devrez peut-être modifier le bloqueur de fenêtres pop-up de votre navigateur pour autoriser les `loginPopup` fenêtres pop-up à partir `http://localhost:8080` de .

1. Actualisez la page et connectez-vous. La page doit être mise à jour avec le nom d’utilisateur, ce qui indique que la signature a réussi.

## <a name="add-authentication-to-the-azure-function"></a>Ajouter l’authentification à la fonction Azure

Dans cette section, vous allez implémenter le flux de la part de dans la fonction Azure pour obtenir un jeton d’accès compatible avec `GetMyNewestMessage` Microsoft Graph.

1. Initialisez le magasin de secrets de développement .NET en ouvrant votre CLI dans le répertoire qui contient **GraphTutorial.csproj** et en exécutant la commande suivante.

    ```Shell
    dotnet user-secrets init
    ```

1. Ajoutez votre ID d’application, votre secret et votre ID de client au magasin secret à l’aide des commandes suivantes. Remplacez `YOUR_API_FUNCTION_APP_ID_HERE` par l’ID d’application **pour la Graph Azure Function**. Remplacez `YOUR_API_FUNCTION_APP_SECRET_HERE` par la secret d’application que vous avez créée dans le portail Azure **pour la Graph Azure Function**. Remplacez `YOUR_TENANT_ID_HERE` par la **valeur d’ID d’annuaire (client)** que vous avez copiée à partir du portail Azure.

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>Traiter le jeton du porteur entrant

Dans cette section, vous allez implémenter une classe pour valider et traiter le jeton du porteur envoyé à partir de la SPA vers la fonction Azure.

1. Créez un répertoire dans le **répertoire GraphTutorial** nommé **Authentication**.

1. Créez un fichier nommé **TokenValidationResult.cs** dans le dossier **./GraphTutorial/Authentication** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. Créez un fichier nommé **TokenValidation.cs** dans le dossier **./GraphTutorial/Authentication** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

Prenez en compte ce que fait ce code.

- Il s’assure qu’il existe un jeton du porteur dans `Authorization` l’en-tête.
- Il vérifie la signature et l’émetteur de la configuration OpenID publiée d’Azure.
- Il vérifie que l’audience `aud` (revendication) correspond à l’ID d’application de la fonction Azure.
- Il pare le jeton et génère un ID de compte MSAL, qui sera nécessaire pour tirer parti de la mise en cache du jeton.

### <a name="create-an-on-behalf-of-authentication-provider"></a>Créer un fournisseur d’authentification de la part de

1. Créez un fichier  dans le répertoire d’authentification **nommé OnBehalfOfAuthProvider.cs** et ajoutez le code suivant à ce fichier.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

Prenez le temps de réfléchir à ce que fait le code **dans OnBehalfOfAuthProvider.cs.**

- Dans la fonction, il tente d’abord d’obtenir un jeton utilisateur à partir du cache de jetons à `GetAccessToken` l’aide `AcquireTokenSilent` de . En cas d’échec, il utilise le jeton du porteur envoyé par l’application de test à la fonction Azure pour générer une assertion utilisateur. Il utilise ensuite cette assertion utilisateur pour obtenir un jeton compatible Graph à l’aide de `AcquireTokenOnBehalfOf` .
- Il implémente l’interface, ce qui permet à cette classe d’être passée dans le constructeur du constructeur pour authentifier `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` les demandes sortantes.

### <a name="implement-a-graph-client-service"></a>Implémenter Graph service client

Dans cette section, vous allez implémenter un service qui peut être inscrit pour [l’injection de dépendance.](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) Le service sera utilisé pour obtenir un client Graph authentifié.

1. Créez un répertoire dans le **répertoire GraphTutorial** nommé **Services**.

1. Créez un fichier dans le répertoire **Services** nommé **IGraphClientService.cs** et ajoutez le code suivant à ce fichier.

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. Créez un fichier dans le répertoire **Services** nommé **GraphClientService.cs** et ajoutez le code suivant à ce fichier.

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. Ajoutez les propriétés suivantes à la `GraphClientService` classe.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. Ajoutez les fonctions suivantes à la `GraphClientService` classe.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. Ajoutez une implémentation d’espace réservé pour la `GetAppGraphClient` fonction. Vous l’implémenterez dans les sections ultérieures.

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    La fonction prend les résultats de la validation du jeton et crée une fonction authentifiée `GetUserGraphClient` pour `GraphServiceClient` l’utilisateur.

1. Ouvrez **./GraphTutorial/Program.cs** et remplacez son contenu par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="ProgramSnippet" highlight="15-23":::

    Ce code ajoute des secrets utilisateur à la configuration et active [l’injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) de dépendances dans vos fonctions Azure, exposant le `GraphClientService` service.

### <a name="implement-getmynewestmessage-function"></a>Implémenter la fonction GetMyNewestMessage

1. Ouvrez **./GraphTutorial/GetMyNewestMessage.cs** et remplacez tout son contenu par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>Passer en revue le code dans GetMyNewestMessage.cs

Prenez le temps de réfléchir à ce que fait le code **dans GetMyNewestMessage.cs.**

- Dans le constructeur, il enregistre les objets transmis via `IConfiguration` `IGraphClientService` l’injection de dépendance.
- Dans la `Run` fonction, elle fait les choses suivantes :
  - Valide que les valeurs de configuration requises sont présentes dans `IConfiguration` l’objet.
  - Valide le jeton du porteur et renvoie un `401` code d’état si le jeton n’est pas valide.
  - Obtient un client Graph de `GraphClientService` l’utilisateur qui a effectué cette demande.
  - Utilise le SDK Microsoft Graph pour obtenir le dernier message de la boîte de réception de l’utilisateur et le renvoie sous forme de corps JSON dans la réponse.

## <a name="call-the-azure-function-from-the-test-app"></a>Appeler la fonction Azure à partir de l’application de test

1. Ouvrez **auth.js** et ajoutez la fonction suivante pour obtenir un jeton d’accès.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    Prenez en compte ce que fait ce code.

    - Il tente d’abord d’obtenir un jeton d’accès en mode silencieux, sans intervention de l’utilisateur. Étant donné que l’utilisateur doit déjà être signé, MSAL doit avoir des jetons pour l’utilisateur dans son cache.
    - En cas d’échec avec une erreur qui indique que l’utilisateur doit interagir, il tente d’obtenir un jeton de manière interactive.

    > [!TIP]
    > Vous pouvez consulter le jeton d’accès et vérifier que la revendication est l’ID d’application pour la fonction Azure et que la revendication contient [https://jwt.ms](https://jwt.ms) `aud` l’étendue d’autorisation de la fonction Azure, et non `scp` Microsoft Graph.

1. Créez un fichier dans le **répertoire TestClient** nommé **azurefunctions.js** et ajoutez le code suivant.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. Modifiez le répertoire actuel de votre CLI en **répertoire ./GraphTutorial** et exécutez la commande suivante pour démarrer la fonction Azure localement.

    ```Shell
    func start
    ```

1. Si la SPA n’est pas déjà en service, ouvrez une deuxième fenêtre CLI et modifiez le répertoire actuel en **répertoire ./TestClient.** Exécutez la commande suivante pour exécuter l’application de test.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. Ouvrez votre navigateur et accédez à `http://localhost:8080`. Connectez-vous et sélectionnez **l’élément de** navigation Dernier message. L’application affiche des informations sur le dernier message dans la boîte de réception de l’utilisateur.
