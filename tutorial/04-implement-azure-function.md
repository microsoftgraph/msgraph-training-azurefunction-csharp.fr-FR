---
ms.openlocfilehash: 3e6a83c19de68b6047914a68d94e66dbab95c6c4
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445954"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5ea67-101">Dans cet exercice, vous terminerez l’implémentation de la fonction Azure et mettrez à jour le `GetMyNewestMessage` client de test pour appeler la fonction.</span><span class="sxs-lookup"><span data-stu-id="5ea67-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="5ea67-102">La fonction Azure utilise le flux « de [la part de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)».</span><span class="sxs-lookup"><span data-stu-id="5ea67-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="5ea67-103">L’ordre de base des événements dans ce flux est le :</span><span class="sxs-lookup"><span data-stu-id="5ea67-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="5ea67-104">L’application de test utilise un flux d’th interactive pour permettre à l’utilisateur de se connecter et d’accorder son consentement.</span><span class="sxs-lookup"><span data-stu-id="5ea67-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="5ea67-105">Elle obtient un jeton qui est étendue à la fonction Azure.</span><span class="sxs-lookup"><span data-stu-id="5ea67-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="5ea67-106">Le jeton ne **contient pas** d’étendues Graph Microsoft.</span><span class="sxs-lookup"><span data-stu-id="5ea67-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="5ea67-107">L’application de test appelle la fonction Azure, en envoyant son jeton d’accès dans `Authorization` l’en-tête.</span><span class="sxs-lookup"><span data-stu-id="5ea67-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="5ea67-108">La fonction Azure valide le jeton, puis échange ce jeton contre un deuxième jeton d’accès qui contient les étendues Graph Microsoft.</span><span class="sxs-lookup"><span data-stu-id="5ea67-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="5ea67-109">La fonction Azure appelle Microsoft Graph au nom de l’utilisateur à l’aide du deuxième jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="5ea67-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="5ea67-110">Pour éviter de stocker l’ID d’application et la secret dans la source, vous utiliserez le gestionnaire de secret [.NET](https://docs.microsoft.com/aspnet/core/security/app-secrets) pour stocker ces valeurs.</span><span class="sxs-lookup"><span data-stu-id="5ea67-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="5ea67-111">Le Gestionnaire de secret est uniquement à des fins de développement, les applications de production doivent utiliser un gestionnaire de secret approuvé pour stocker les secrets.</span><span class="sxs-lookup"><span data-stu-id="5ea67-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="5ea67-112">Ajouter l’authentification à l’application à page unique</span><span class="sxs-lookup"><span data-stu-id="5ea67-112">Add authentication to the single page application</span></span>

<span data-ttu-id="5ea67-113">Commencez par ajouter l’authentification à la SPA.</span><span class="sxs-lookup"><span data-stu-id="5ea67-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="5ea67-114">Cela permettra à l’application d’obtenir un jeton d’accès accordant l’accès pour appeler la fonction Azure.</span><span class="sxs-lookup"><span data-stu-id="5ea67-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="5ea67-115">Comme il s’agit d’une SPA, elle utilise le flux de [code d’autorisation avec PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span><span class="sxs-lookup"><span data-stu-id="5ea67-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="5ea67-116">Créez un fichier dans le **répertoire TestClient** nommé **config.js** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="5ea67-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="5ea67-117">Remplacez par l’ID d’application que vous avez créé dans le portail `YOUR_TEST_APP_APP_ID_HERE` Azure **pour Graph’application test de fonction Azure.**</span><span class="sxs-lookup"><span data-stu-id="5ea67-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="5ea67-118">Remplacez `YOUR_TENANT_ID_HERE` par la **valeur d’ID d’annuaire (client)** que vous avez copiée à partir du portail Azure.</span><span class="sxs-lookup"><span data-stu-id="5ea67-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="5ea67-119">Remplacez `YOUR_AZURE_FUNCTION_APP_ID_HERE` par l’ID d’application **pour la Graph Azure Function**.</span><span class="sxs-lookup"><span data-stu-id="5ea67-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="5ea67-120">Si vous utilisez un contrôle source tel que Git, il est temps d’exclure le fichier **config.js** du contrôle source afin d’éviter toute fuite accidentelle de vos ID d’application et ID de client.</span><span class="sxs-lookup"><span data-stu-id="5ea67-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="5ea67-121">Créez un fichier dans le **répertoire TestClient** nommé **auth.js** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="5ea67-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="5ea67-122">Prenez en compte ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="5ea67-122">Consider what this code does.</span></span>

    - <span data-ttu-id="5ea67-123">Il initialise une à `PublicClientApplication` l’aide des valeurs stockées dans **config.js**.</span><span class="sxs-lookup"><span data-stu-id="5ea67-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="5ea67-124">Il utilise pour `loginPopup` se connecter à l’utilisateur, à l’aide de l’étendue d’autorisation pour la fonction Azure.</span><span class="sxs-lookup"><span data-stu-id="5ea67-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="5ea67-125">Il stocke le nom d’utilisateur de l’utilisateur dans la session.</span><span class="sxs-lookup"><span data-stu-id="5ea67-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="5ea67-126">Étant donné que l’application utilise , vous devrez peut-être modifier le bloqueur de fenêtres pop-up de votre navigateur pour autoriser les `loginPopup` fenêtres pop-up à partir `http://localhost:8080` de .</span><span class="sxs-lookup"><span data-stu-id="5ea67-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="5ea67-127">Actualisez la page et connectez-vous.</span><span class="sxs-lookup"><span data-stu-id="5ea67-127">Refresh the page and sign in.</span></span> <span data-ttu-id="5ea67-128">La page doit être mise à jour avec le nom d’utilisateur, ce qui indique que la signature a réussi.</span><span class="sxs-lookup"><span data-stu-id="5ea67-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="5ea67-129">Ajouter l’authentification à la fonction Azure</span><span class="sxs-lookup"><span data-stu-id="5ea67-129">Add authentication to the Azure Function</span></span>

<span data-ttu-id="5ea67-130">Dans cette section, vous allez implémenter le flux de la part de dans la fonction Azure pour obtenir un jeton d’accès compatible avec `GetMyNewestMessage` Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="5ea67-130">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="5ea67-131">Initialisez le magasin de secrets de développement .NET en ouvrant votre CLI dans le répertoire qui contient **GraphTutorial.csproj** et en exécutant la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="5ea67-131">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="5ea67-132">Ajoutez votre ID d’application, votre secret et votre ID de client au magasin secret à l’aide des commandes suivantes.</span><span class="sxs-lookup"><span data-stu-id="5ea67-132">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="5ea67-133">Remplacez `YOUR_API_FUNCTION_APP_ID_HERE` par l’ID d’application **pour la Graph Azure Function**.</span><span class="sxs-lookup"><span data-stu-id="5ea67-133">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="5ea67-134">Remplacez `YOUR_API_FUNCTION_APP_SECRET_HERE` par la secret d’application que vous avez créée dans le portail Azure **pour la Graph Azure Function**.</span><span class="sxs-lookup"><span data-stu-id="5ea67-134">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="5ea67-135">Remplacez `YOUR_TENANT_ID_HERE` par la **valeur d’ID d’annuaire (client)** que vous avez copiée à partir du portail Azure.</span><span class="sxs-lookup"><span data-stu-id="5ea67-135">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="5ea67-136">Traiter le jeton du porteur entrant</span><span class="sxs-lookup"><span data-stu-id="5ea67-136">Process the incoming bearer token</span></span>

<span data-ttu-id="5ea67-137">Dans cette section, vous allez implémenter une classe pour valider et traiter le jeton du porteur envoyé à partir de la SPA vers la fonction Azure.</span><span class="sxs-lookup"><span data-stu-id="5ea67-137">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="5ea67-138">Créez un répertoire dans le **répertoire GraphTutorial** nommé **Authentication**.</span><span class="sxs-lookup"><span data-stu-id="5ea67-138">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="5ea67-139">Créez un fichier nommé **TokenValidationResult.cs** dans le dossier **./GraphTutorial/Authentication** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="5ea67-139">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="5ea67-140">Créez un fichier nommé **TokenValidation.cs** dans le dossier **./GraphTutorial/Authentication** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="5ea67-140">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="5ea67-141">Prenez en compte ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="5ea67-141">Consider what this code does.</span></span>

- <span data-ttu-id="5ea67-142">Il s’assure qu’il existe un jeton du porteur dans `Authorization` l’en-tête.</span><span class="sxs-lookup"><span data-stu-id="5ea67-142">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="5ea67-143">Il vérifie la signature et l’émetteur de la configuration OpenID publiée d’Azure.</span><span class="sxs-lookup"><span data-stu-id="5ea67-143">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="5ea67-144">Il vérifie que l’audience `aud` (revendication) correspond à l’ID d’application de la fonction Azure.</span><span class="sxs-lookup"><span data-stu-id="5ea67-144">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="5ea67-145">Il pare le jeton et génère un ID de compte MSAL, qui sera nécessaire pour tirer parti de la mise en cache du jeton.</span><span class="sxs-lookup"><span data-stu-id="5ea67-145">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="5ea67-146">Créer un fournisseur d’authentification de la part de</span><span class="sxs-lookup"><span data-stu-id="5ea67-146">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="5ea67-147">Créez un fichier  dans le répertoire d’authentification **nommé OnBehalfOfAuthProvider.cs** et ajoutez le code suivant à ce fichier.</span><span class="sxs-lookup"><span data-stu-id="5ea67-147">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="5ea67-148">Prenez le temps de réfléchir à ce que fait le code **dans OnBehalfOfAuthProvider.cs.**</span><span class="sxs-lookup"><span data-stu-id="5ea67-148">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="5ea67-149">Dans la fonction, il tente d’abord d’obtenir un jeton utilisateur à partir du cache de jetons à `GetAccessToken` l’aide `AcquireTokenSilent` de .</span><span class="sxs-lookup"><span data-stu-id="5ea67-149">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="5ea67-150">En cas d’échec, il utilise le jeton du porteur envoyé par l’application de test à la fonction Azure pour générer une assertion utilisateur.</span><span class="sxs-lookup"><span data-stu-id="5ea67-150">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="5ea67-151">Il utilise ensuite cette assertion utilisateur pour obtenir un jeton compatible Graph à l’aide de `AcquireTokenOnBehalfOf` .</span><span class="sxs-lookup"><span data-stu-id="5ea67-151">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="5ea67-152">Il implémente l’interface, ce qui permet à cette classe d’être passée dans le constructeur du constructeur pour authentifier `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` les demandes sortantes.</span><span class="sxs-lookup"><span data-stu-id="5ea67-152">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="5ea67-153">Implémenter Graph service client</span><span class="sxs-lookup"><span data-stu-id="5ea67-153">Implement a Graph client service</span></span>

<span data-ttu-id="5ea67-154">Dans cette section, vous allez implémenter un service qui peut être inscrit pour [l’injection de dépendance.](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="5ea67-154">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="5ea67-155">Le service sera utilisé pour obtenir un client Graph authentifié.</span><span class="sxs-lookup"><span data-stu-id="5ea67-155">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="5ea67-156">Créez un répertoire dans le **répertoire GraphTutorial** nommé **Services**.</span><span class="sxs-lookup"><span data-stu-id="5ea67-156">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="5ea67-157">Créez un fichier dans le répertoire **Services** nommé **IGraphClientService.cs** et ajoutez le code suivant à ce fichier.</span><span class="sxs-lookup"><span data-stu-id="5ea67-157">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="5ea67-158">Créez un fichier dans le répertoire **Services** nommé **GraphClientService.cs** et ajoutez le code suivant à ce fichier.</span><span class="sxs-lookup"><span data-stu-id="5ea67-158">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

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

1. <span data-ttu-id="5ea67-159">Ajoutez les propriétés suivantes à la `GraphClientService` classe.</span><span class="sxs-lookup"><span data-stu-id="5ea67-159">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="5ea67-160">Ajoutez les fonctions suivantes à la `GraphClientService` classe.</span><span class="sxs-lookup"><span data-stu-id="5ea67-160">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="5ea67-161">Ajoutez une implémentation d’espace réservé pour la `GetAppGraphClient` fonction.</span><span class="sxs-lookup"><span data-stu-id="5ea67-161">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="5ea67-162">Vous l’implémenterez dans les sections ultérieures.</span><span class="sxs-lookup"><span data-stu-id="5ea67-162">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="5ea67-163">La fonction prend les résultats de la validation du jeton et crée une fonction authentifiée `GetUserGraphClient` pour `GraphServiceClient` l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="5ea67-163">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="5ea67-164">Ouvrez **./GraphTutorial/Program.cs** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="5ea67-164">Open **./GraphTutorial/Program.cs** and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="ProgramSnippet" highlight="15-23":::

    <span data-ttu-id="5ea67-165">Ce code ajoute des secrets utilisateur à la configuration et active [l’injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) de dépendances dans vos fonctions Azure, exposant le `GraphClientService` service.</span><span class="sxs-lookup"><span data-stu-id="5ea67-165">This code will add user secrets to the configuration, and enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="5ea67-166">Implémenter la fonction GetMyNewestMessage</span><span class="sxs-lookup"><span data-stu-id="5ea67-166">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="5ea67-167">Ouvrez **./GraphTutorial/GetMyNewestMessage.cs** et remplacez tout son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="5ea67-167">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="5ea67-168">Passer en revue le code dans GetMyNewestMessage.cs</span><span class="sxs-lookup"><span data-stu-id="5ea67-168">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="5ea67-169">Prenez le temps de réfléchir à ce que fait le code **dans GetMyNewestMessage.cs.**</span><span class="sxs-lookup"><span data-stu-id="5ea67-169">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="5ea67-170">Dans le constructeur, il enregistre les objets transmis via `IConfiguration` `IGraphClientService` l’injection de dépendance.</span><span class="sxs-lookup"><span data-stu-id="5ea67-170">In the constructor, it saves the `IConfiguration` and `IGraphClientService` objects passed in via dependency injection.</span></span>
- <span data-ttu-id="5ea67-171">Dans la `Run` fonction, elle fait les choses suivantes :</span><span class="sxs-lookup"><span data-stu-id="5ea67-171">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="5ea67-172">Valide que les valeurs de configuration requises sont présentes dans `IConfiguration` l’objet.</span><span class="sxs-lookup"><span data-stu-id="5ea67-172">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="5ea67-173">Valide le jeton du porteur et renvoie un `401` code d’état si le jeton n’est pas valide.</span><span class="sxs-lookup"><span data-stu-id="5ea67-173">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="5ea67-174">Obtient un client Graph de `GraphClientService` l’utilisateur qui a effectué cette demande.</span><span class="sxs-lookup"><span data-stu-id="5ea67-174">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="5ea67-175">Utilise le SDK Microsoft Graph pour obtenir le dernier message de la boîte de réception de l’utilisateur et le renvoie sous forme de corps JSON dans la réponse.</span><span class="sxs-lookup"><span data-stu-id="5ea67-175">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="5ea67-176">Appeler la fonction Azure à partir de l’application de test</span><span class="sxs-lookup"><span data-stu-id="5ea67-176">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="5ea67-177">Ouvrez **auth.js** et ajoutez la fonction suivante pour obtenir un jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="5ea67-177">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="5ea67-178">Prenez en compte ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="5ea67-178">Consider what this code does.</span></span>

    - <span data-ttu-id="5ea67-179">Il tente d’abord d’obtenir un jeton d’accès en mode silencieux, sans intervention de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="5ea67-179">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="5ea67-180">Étant donné que l’utilisateur doit déjà être signé, MSAL doit avoir des jetons pour l’utilisateur dans son cache.</span><span class="sxs-lookup"><span data-stu-id="5ea67-180">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="5ea67-181">En cas d’échec avec une erreur qui indique que l’utilisateur doit interagir, il tente d’obtenir un jeton de manière interactive.</span><span class="sxs-lookup"><span data-stu-id="5ea67-181">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

    > [!TIP]
    > <span data-ttu-id="5ea67-182">Vous pouvez consulter le jeton d’accès et vérifier que la revendication est l’ID d’application pour la fonction Azure et que la revendication contient [https://jwt.ms](https://jwt.ms) `aud` l’étendue d’autorisation de la fonction Azure, et non `scp` Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="5ea67-182">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

1. <span data-ttu-id="5ea67-183">Créez un fichier dans le **répertoire TestClient** nommé **azurefunctions.js** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="5ea67-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="5ea67-184">Modifiez le répertoire actuel de votre CLI en **répertoire ./GraphTutorial** et exécutez la commande suivante pour démarrer la fonction Azure localement.</span><span class="sxs-lookup"><span data-stu-id="5ea67-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="5ea67-185">Si la SPA n’est pas déjà en service, ouvrez une deuxième fenêtre CLI et modifiez le répertoire actuel en **répertoire ./TestClient.**</span><span class="sxs-lookup"><span data-stu-id="5ea67-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="5ea67-186">Exécutez la commande suivante pour exécuter l’application de test.</span><span class="sxs-lookup"><span data-stu-id="5ea67-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="5ea67-187">Ouvrez votre navigateur et accédez à `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="5ea67-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="5ea67-188">Connectez-vous et sélectionnez **l’élément de** navigation Dernier message.</span><span class="sxs-lookup"><span data-stu-id="5ea67-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="5ea67-189">L’application affiche des informations sur le dernier message dans la boîte de réception de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="5ea67-189">The app displays information about the newest message in the user's inbox.</span></span>
