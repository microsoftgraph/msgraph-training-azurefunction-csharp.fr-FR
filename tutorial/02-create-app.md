---
ms.openlocfilehash: 17c93f353c84ea2db28cd2e0203d30c5f320e36e
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655232"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="09de4-101">Dans ce didacticiel, vous allez créer une fonction Azure simple qui implémente les fonctions de déclencheur HTTP qui appellent Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="09de4-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="09de4-102">Ces fonctions couvrent les scénarios suivants :</span><span class="sxs-lookup"><span data-stu-id="09de4-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="09de4-103">Implémente une API pour accéder à la boîte de réception d’un utilisateur en utilisant l’authentification [de flux de la part de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) .</span><span class="sxs-lookup"><span data-stu-id="09de4-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="09de4-104">Implémente une API pour s’abonner et annuler l’abonnement aux notifications dans la boîte de réception d’un utilisateur, à l’aide de l’authentification de [flux d’octroi d’informations d’identification client](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) .</span><span class="sxs-lookup"><span data-stu-id="09de4-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="09de4-105">Implémente un webhook pour recevoir [des notifications de modification](https://docs.microsoft.com/graph/webhooks) de Microsoft Graph et accéder à des données à l’aide du flux d’octroi d’informations d’identification du client.</span><span class="sxs-lookup"><span data-stu-id="09de4-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="09de4-106">Vous allez également créer une application JavaScript simple à page unique (SPA) pour appeler les API implémentées dans la fonction Azure.</span><span class="sxs-lookup"><span data-stu-id="09de4-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="09de4-107">Créer un projet de fonctions Azure</span><span class="sxs-lookup"><span data-stu-id="09de4-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="09de4-108">Ouvrez votre interface de ligne de commande (CLI) dans un répertoire où vous souhaitez créer le projet.</span><span class="sxs-lookup"><span data-stu-id="09de4-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="09de4-109">Exécutez la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="09de4-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. <span data-ttu-id="09de4-110">Remplacez le répertoire actuel de votre interface CLI par le répertoire **GraphTutorial** et exécutez les commandes suivantes pour créer trois fonctions dans le projet.</span><span class="sxs-lookup"><span data-stu-id="09de4-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. <span data-ttu-id="09de4-111">Ouvrez **local.settings.jssur** et ajoutez le code suivant au fichier pour autoriser cors à partir de `http://localhost:8080` , l’URL de l’application de test.</span><span class="sxs-lookup"><span data-stu-id="09de4-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="09de4-112">Exécutez la commande suivante pour exécuter le projet localement.</span><span class="sxs-lookup"><span data-stu-id="09de4-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="09de4-113">Si tout fonctionne, vous verrez le résultat suivant :</span><span class="sxs-lookup"><span data-stu-id="09de4-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="09de4-114">Vérifiez que les fonctions fonctionnent correctement en ouvrant votre navigateur et en accédant aux URL de fonction indiquées dans la sortie.</span><span class="sxs-lookup"><span data-stu-id="09de4-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="09de4-115">Le message suivant s’affiche dans votre navigateur : `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` .</span><span class="sxs-lookup"><span data-stu-id="09de4-115">You should see the following message in your browser: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="09de4-116">Créer une application à page unique</span><span class="sxs-lookup"><span data-stu-id="09de4-116">Create single-page application</span></span>

1. <span data-ttu-id="09de4-117">Ouvrez l’interface de commande dans le répertoire dans lequel vous souhaitez créer le projet.</span><span class="sxs-lookup"><span data-stu-id="09de4-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="09de4-118">Créez un répertoire nommé **TestClient** pour stocker vos fichiers HTML et JavaScript.</span><span class="sxs-lookup"><span data-stu-id="09de4-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="09de4-119">Créez un fichier nommé **index.html** dans le répertoire **TestClient** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="09de4-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="09de4-120">Cette définition définit la disposition de base de l’application, y compris une barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="09de4-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="09de4-121">Il ajoute également les éléments suivants :</span><span class="sxs-lookup"><span data-stu-id="09de4-121">It also adds the following:</span></span>

    - <span data-ttu-id="09de4-122">[Bootstrap](https://getbootstrap.com/) et son code JavaScript de prise en charge</span><span class="sxs-lookup"><span data-stu-id="09de4-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="09de4-123">FontAwesome</span><span class="sxs-lookup"><span data-stu-id="09de4-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="09de4-124">Bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) 2,0</span><span class="sxs-lookup"><span data-stu-id="09de4-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="09de4-125">La page inclut un favorite, ( `<link rel="shortcut icon" href="g-raph.png">` ).</span><span class="sxs-lookup"><span data-stu-id="09de4-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="09de4-126">Vous pouvez supprimer cette ligne, ou vous pouvez télécharger le fichier **g-raph.png** à partir de [GitHub](https://github.com/microsoftgraph/g-raph).</span><span class="sxs-lookup"><span data-stu-id="09de4-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="09de4-127">Créez un fichier nommé **style. CSS** dans le répertoire **TestClient** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="09de4-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="09de4-128">Créez un fichier nommé **ui.js** dans le répertoire **TestClient** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="09de4-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="09de4-129">Ce code utilise JavaScript pour afficher la page actuelle en fonction de la vue sélectionnée.</span><span class="sxs-lookup"><span data-stu-id="09de4-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="09de4-130">Tester l’application à page unique</span><span class="sxs-lookup"><span data-stu-id="09de4-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="09de4-131">Cette section contient des instructions sur l’utilisation de [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) pour exécuter un serveur http de test simple sur votre ordinateur de développement.</span><span class="sxs-lookup"><span data-stu-id="09de4-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="09de4-132">L’utilisation de cet outil spécifique n’est pas obligatoire.</span><span class="sxs-lookup"><span data-stu-id="09de4-132">Using this specific tool is not required.</span></span> <span data-ttu-id="09de4-133">Vous pouvez utiliser n’importe quel serveur d’évaluation pour servir le répertoire **TestClient** .</span><span class="sxs-lookup"><span data-stu-id="09de4-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="09de4-134">Exécutez la commande suivante dans votre interface CLI pour installer **dotnet-serve**.</span><span class="sxs-lookup"><span data-stu-id="09de4-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="09de4-135">Remplacez le répertoire actuel de votre interface CLI par le répertoire **TestClient** et exécutez la commande suivante pour démarrer un serveur http.</span><span class="sxs-lookup"><span data-stu-id="09de4-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. <span data-ttu-id="09de4-136">Ouvrez votre navigateur et accédez à `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="09de4-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="09de4-137">La page doit s’afficher, mais aucun bouton ne fonctionne.</span><span class="sxs-lookup"><span data-stu-id="09de4-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="09de4-138">Ajouter des packages NuGet</span><span class="sxs-lookup"><span data-stu-id="09de4-138">Add NuGet packages</span></span>

<span data-ttu-id="09de4-139">Avant de poursuivre, installez des packages NuGet supplémentaires que vous utiliserez plus tard.</span><span class="sxs-lookup"><span data-stu-id="09de4-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="09de4-140">[Microsoft. Azure. Functions. extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) pour activer l’injection de dépendance dans le projet de fonctions Azure.</span><span class="sxs-lookup"><span data-stu-id="09de4-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="09de4-141">[Microsoft.Extensions.Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) pour lire la configuration de l’application à partir du [magasin de secrets de développement .net](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="09de4-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="09de4-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) pour effectuer des appels Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="09de4-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="09de4-143">[Microsoft. Identity. client](https://www.nuget.org/packages/Microsoft.Identity.Client/) pour l’authentification et la gestion des jetons.</span><span class="sxs-lookup"><span data-stu-id="09de4-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="09de4-144">[Microsoft. IdentityModel. Protocols. OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) pour récupérer la configuration de OpenID pour la validation de jeton.</span><span class="sxs-lookup"><span data-stu-id="09de4-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="09de4-145">[System. IdentityModel. Tokens. JWT](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) pour la validation des jetons envoyés à l’API Web.</span><span class="sxs-lookup"><span data-stu-id="09de4-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="09de4-146">Remplacez le répertoire actuel de votre interface CLI par le répertoire **GraphTutorial** et exécutez les commandes suivantes.</span><span class="sxs-lookup"><span data-stu-id="09de4-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
