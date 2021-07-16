---
ms.openlocfilehash: 181afe9acfe45ff619a50cf874669228b421b475
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445975"
---
<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous terminerez l’implémentation des fonctions Azure et mettez à jour l’application de test pour vous abonner et vous désabonner aux modifications dans la boîte de réception `SetSubscription` `Notify` d’un utilisateur.

- La fonction agit en tant qu’API, ce qui permet à l’application de test de créer ou de supprimer un abonnement aux modifications apportées dans la boîte de réception `SetSubscription` d’un [](https://docs.microsoft.com/graph/webhooks) utilisateur.
- La `Notify` fonction agit en tant que webhook qui reçoit les notifications de modification générées par l’abonnement.

Les deux fonctions utiliseront le [flux](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) d’octroi d’informations d’identification client pour obtenir un jeton d’application uniquement pour appeler Microsoft Graph. Étant donné qu’un administrateur a accordé le consentement administrateur aux étendues d’autorisation requises, aucune interaction utilisateur n’est requise pour obtenir le jeton.

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>Ajouter l’authentification des informations d’identification client au projet Fonctions Azure

Dans cette section, vous allez implémenter le flux d’informations d’identification client dans le projet Fonctions Azure pour obtenir un jeton d’accès compatible avec Microsoft Graph.

1. Ouvrez votre CLI dans le répertoire qui contient **GraphTutorial.csproj**.

1. Ajoutez votre ID d’application webhook et votre secret au magasin de secrets à l’aide des commandes suivantes. Remplacez `YOUR_WEBHOOK_APP_ID_HERE` par l’ID d’application **pour Graph webhook de fonction Azure.** Remplacez par la secret d’application que vous avez créée dans le portail `YOUR_WEBHOOK_APP_SECRET_HERE` Azure pour le Graph **webhook de fonction Azure.**

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>Créer un fournisseur d’authentification d’informations d’identification client

1. Créez un fichier dans le répertoire **./GraphTutorial/Authentication** nommé **ClientCredentialsAuthProvider.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

Prenez le temps de réfléchir à ce que fait le code dans **ClientCredentialsAuthProvider.cs.**

- Dans le constructeur, il initialise un **ConfidentialClientApplication** à partir du `Microsoft.Identity.Client` package. Il utilise les fonctions et les fonctions pour limiter l’audience de connexion uniquement à `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` `.WithTenantId(tenantId)` l’Microsoft 365 organisation.
- Dans la `GetAccessToken` fonction, il appelle `AcquireTokenForClient` pour obtenir un jeton pour l’application. Le flux de jeton d’informations d’identification client est toujours non interactif.
- Il implémente l’interface, ce qui permet à cette classe d’être passée dans le constructeur du constructeur pour authentifier `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` les demandes sortantes.

## <a name="update-graphclientservice"></a>Mettre à jour GraphClientService

1. Ouvrez **GraphClientService.cs** et ajoutez la propriété suivante à la classe.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. Remplacez la fonction `GetAppGraphClient` existante par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientFunctions":::

## <a name="implement-notify-function"></a>Implémenter la fonction Notify

Dans cette section, vous allez implémenter la fonction, qui sera utilisée comme URL de `Notify` notification pour les notifications de modification.

1. Créez un répertoire dans le **répertoire GraphTutorials** nommé **Modèles.**

1. Créez un fichier dans le répertoire **Models** nommé **ResourceData.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. Créez un fichier dans le répertoire **Models** nommé **ChangeNotificationPayload.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotificationPayload.cs" id="ChangeNotificationSnippet":::

1. Créez un fichier dans le répertoire **Models** nommé **NotificationList.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. Ouvrez **./GraphTutorial/Notify.cs** et remplacez tout son contenu par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

Prenez le temps de réfléchir à ce que fait le code dans **Notify.cs.**

- La `Run` fonction vérifie la présence d’un paramètre de `validationToken` requête. Si ce paramètre est présent, il traite la demande en tant que demande [de validation](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)et répond en conséquence.
- Si la demande n’est pas une demande de validation, la charge utile JSON est désérialisée en `ChangeNotificationCollection` une .
- Chaque notification de la liste est vérifiée pour la valeur d’état client attendue et traitée.
- Le message ayant déclenché la notification est récupéré avec Microsoft Graph.

## <a name="implement-setsubscription-function"></a>Implémenter la fonction SetSubscription

Dans cette section, vous allez implémenter la fonction SetSubscription. Cette fonction agit comme une API appelée par l’application de test pour créer ou supprimer un abonnement dans la boîte de réception d’un utilisateur.

1. Créez un fichier dans le répertoire **Models** nommé **SetSubscriptionPayload.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. Ouvrez **./GraphTutorial/SetSubscription.cs** et remplacez tout son contenu par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

Prenez le temps de réfléchir à ce que fait le code **dans SetSubscription.cs.**

- La fonction lit la charge utile JSON envoyée dans la requête POST pour déterminer le type de demande (s’abonner ou se désabonner), l’ID d’utilisateur pour s’abonner et l’ID d’abonnement à `Run` désabonner.
- Si la demande est une demande d’abonnement, elle utilise le SDK Microsoft Graph pour créer un abonnement dans la boîte de réception de l’utilisateur spécifié. L’abonnement notifiera la création ou la mise à jour des messages. Le nouvel abonnement est renvoyé dans la charge utile JSON de la réponse.
- Si la demande est une demande de désabonnement, elle utilise le SDK Microsoft Graph pour supprimer l’abonnement spécifié.

## <a name="call-setsubscription-from-the-test-app"></a>Appeler SetSubscription à partir de l’application de test

Dans cette section, vous allez implémenter des fonctions pour créer et supprimer des abonnements dans l’application de test.

1. Ouvrez **./TestClient/azurefunctions.js** et ajoutez la fonction suivante.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    Ce code appelle la fonction Azure pour s’abonner et ajoute le nouvel abonnement au `SetSubscription` tableau des abonnements dans la session.

1. Ajoutez la fonction suivante à **azurefunctions.js**.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    Ce code appelle la fonction Azure pour se désabonner et supprime l’abonnement du tableau des `SetSubscription` abonnements dans la session.

1. Si ngrok n’est pas en cours d’exécution, exécutez ngrok ( ) et copiez l’URL de `ngrok http 7071` forwarding HTTPS.

1. Ajoutez l’URL ngrok au magasin de secrets utilisateur en exécutant la commande suivante.

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > Si vous redémarrez ngrok, vous devrez répéter cette commande pour mettre à jour votre URL ngrok.

1. Modifiez le répertoire actuel de votre CLI en **répertoire ./GraphTutorial** et exécutez la commande suivante pour démarrer la fonction Azure localement.

    ```Shell
    func start
    ```

1. Actualisez la SPA et sélectionnez **l’élément de navigation Abonnements.** Entrez un ID d’utilisateur pour un utilisateur de votre organisation Microsoft 365 qui dispose d’une boîte aux lettres Exchange Online’utilisateur. Il peut s’agit de l’utilisateur `id` (de Microsoft Graph) ou de l’utilisateur. `userPrincipalName` Cliquez sur **S’abonner.**

1. La page s’actualise en affichant le nouvel abonnement dans le tableau.

1. Envoyez un courrier électronique à l’utilisateur. Après un court instant, la `Notify` fonction doit être appelée. Vous pouvez le vérifier dans l’interface web ngrok ( ) ou dans la sortie `http://localhost:4040` de débogage du projet de fonction Azure.

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. Dans l’application de test, cliquez **sur Supprimer** dans la ligne de tableau de l’abonnement. La page est actualisée et l’abonnement n’est plus dans le tableau.
