---
ms.openlocfilehash: eb227079656e2a57550511c3abfacb49935fe46a
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655239"
---
<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez terminer l’implémentation des fonctions Azure `SetSubscription` et `Notify` mettre à jour l’application de test pour vous abonner et annuler l’abonnement aux modifications apportées dans la boîte de réception d’un utilisateur.

- La `SetSubscription` fonction se comporte comme une API, ce qui permet à l’application de test de créer ou supprimer un [abonnement](https://docs.microsoft.com/graph/webhooks) à des modifications apportées à la boîte de réception d’un utilisateur.
- La `Notify` fonction se comporte comme le webhook qui reçoit les notifications de modification générées par l’abonnement.

Les deux fonctions utilisent le [flux d’octroi d’informations d’identification du client](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) pour obtenir un jeton d’application uniquement pour appeler Microsoft Graph. Étant donné qu’un administrateur a accordé le consentement de l’administrateur aux étendues d’autorisation requises, aucune interaction de l’utilisateur n’est requise pour obtenir le jeton.

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>Ajouter l’authentification des informations d’identification du client au projet de fonctions Azure

Dans cette section, vous allez implémenter le flux des informations d’identification du client dans le projet de fonctions Azure pour obtenir un jeton d’accès compatible avec Microsoft Graph.

1. Ouvrez la CLI dans le répertoire qui contient **GraphTutorial. csproj**.

1. Ajoutez votre ID d’application webhook et votre clé secrète au magasin secret à l’aide des commandes suivantes. Remplacez `YOUR_WEBHOOK_APP_ID_HERE` par l’ID de l’application pour le **webhook de fonction Azure de Graph**. Remplacez `YOUR_WEBHOOK_APP_SECRET_HERE` par le secret d’application que vous avez créé dans le portail Azure pour le **webhook de fonction Azure de Graph**.

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>Créer un fournisseur d’authentification des informations d’identification du client

1. Créez un fichier dans le répertoire **./GraphTutorial/Authentication** nommé **ClientCredentialsAuthProvider.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

Prenez le temps de prendre en compte le rôle du code dans **ClientCredentialsAuthProvider.cs** .

- Dans le constructeur, il initialise un **ConfidentialClientApplication** à partir du `Microsoft.Identity.Client` Package. Il utilise les `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` `.WithTenantId(tenantId)` fonctions et pour limiter l’audience de connexion à l’organisation Microsoft 365 spécifiée uniquement.
- Dans la `GetAccessToken` fonction, il appelle `AcquireTokenForClient` pour obtenir un jeton pour l’application. Le flux de jeton des informations d’identification du client est toujours non interactif.
- Elle implémente l' `Microsoft.Graph.IAuthenticationProvider` interface, ce qui permet de transmettre cette classe dans le constructeur du `GraphServiceClient` pour authentifier les demandes sortantes.

## <a name="update-graphclientservice"></a>Mettre à jour GraphClientService

1. Ouvrez **GraphClientService.cs** et ajoutez la propriété suivante à la classe.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. Remplacez la fonction `GetAppGraphClient` existante par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientFunctions":::

## <a name="implement-notify-function"></a>Implémenter la fonction Notify

Dans cette section, vous allez implémenter la `Notify` fonction, qui sera utilisée comme URL de notification pour les notifications de modification.

1. Créez un répertoire dans le répertoire **GraphTutorials** nommé **modèles**.

1. Créez un fichier dans le répertoire des **modèles** nommé **ResourceData.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. Créez un fichier dans le répertoire des **modèles** nommé **ChangeNotification.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotification.cs" id="ChangeNotificationSnippet":::

1. Créez un fichier dans le répertoire des **modèles** nommé **NotificationList.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. Ouvrez **./GraphTutorial/Notify.cs** et remplacez l’intégralité de son contenu par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

Prenez le temps de prendre en compte le rôle du code dans **Notify.cs** .

- La `Run` fonction vérifie la présence d’un `validationToken` paramètre de requête. Si ce paramètre est présent, il traite la demande en tant que [demande de validation](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)et répond en conséquence.
- Si la demande n’est pas une demande de validation, la charge utile JSON est désérialisée en un `NotificationList` .
- Chaque notification de la liste est vérifiée pour la valeur d’état de client attendue et est traitée.
- Le message qui a déclenché la notification est récupéré avec Microsoft Graph.

## <a name="implement-setsubscription-function"></a>Implémenter la fonction SetSubscription

Dans cette section, vous allez implémenter la fonction SetSubscription. Cette fonction se comporte comme une API appelée par l’application de test pour créer ou supprimer un abonnement dans la boîte de réception d’un utilisateur.

1. Créez un fichier dans le répertoire des **modèles** nommé **SetSubscriptionPayload.cs** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. Ouvrez **./GraphTutorial/SetSubscription.cs** et remplacez l’intégralité de son contenu par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

Prenez le temps de prendre en compte le rôle du code dans **SetSubscription.cs** .

- La `Run` fonction lit la charge utile JSON envoyée dans la demande post pour déterminer le type de demande (subscribe ou unsubscribe), l’ID d’utilisateur auquel s’abonner et l’ID d’abonnement pour annuler l’abonnement.
- Si la demande est une demande subscribe, elle utilise le kit de développement logiciel (SDK) Microsoft Graph pour créer un nouvel abonnement dans la boîte de réception de l’utilisateur spécifié. L’abonnement indiquera quand les messages seront créés ou mis à jour. Le nouvel abonnement est renvoyé dans la charge utile JSON de la réponse.
- Si la demande est une demande de désinscription, elle utilise le kit de développement logiciel (SDK) Microsoft Graph pour supprimer l’abonnement spécifié.

## <a name="call-setsubscription-from-the-test-app"></a>Appeler SetSubscription à partir de l’application de test

Dans cette section, vous allez implémenter des fonctions pour créer et supprimer des abonnements dans l’application de test.

1. Ouvrez **/TestClient/azurefunctions.js** et ajoutez la fonction suivante.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    Ce code appelle la `SetSubscription` fonction Azure pour s’abonner et ajoute le nouvel abonnement au tableau des abonnements dans la session.

1. Ajoutez la fonction suivante à **azurefunctions.js**.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    Ce code appelle la `SetSubscription` fonction Azure et supprime l’abonnement du tableau des abonnements dans la session.

1. Si ngrok n’est pas en cours d’exécution, exécutez ngrok ( `ngrok http 7071` ) et copiez l’URL de transfert HTTPS.

1. Ajoutez l’URL ngrok au magasin de secrets d’utilisateur en exécutant la commande suivante.

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > Si vous redémarrez ngrok, vous devrez répéter cette commande pour mettre à jour votre URL ngrok.

1. Remplacez le répertoire actuel de votre interface CLI par le répertoire **./GraphTutorial** et exécutez la commande suivante pour démarrer la fonction Azure localement.

    ```Shell
    func start
    ```

1. Actualisez le SPA et sélectionnez l’élément de navigation **abonnements** . Entrez un ID d’utilisateur pour un utilisateur de votre organisation Microsoft 365 qui dispose d’une boîte aux lettres Exchange Online. Il peut s’agir de l’utilisateur `id` (à partir de Microsoft Graph) ou de l’utilisateur `userPrincipalName` . Cliquez sur **s’abonner**.

1. La page est actualisée et affiche le nouvel abonnement dans le tableau.

1. Envoyer un courrier électronique à l’utilisateur. Après un bref moment, la `Notify` fonction doit être appelée. Vous pouvez vérifier cela dans l’interface Web ngrok ( `http://localhost:4040` ) ou dans la sortie de débogage du projet de fonction Azure.

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. Dans l’application de test, cliquez sur **supprimer** dans la ligne du tableau pour l’abonnement. La page est actualisée et l’abonnement n’est plus dans le tableau.
