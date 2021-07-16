---
ms.openlocfilehash: f034aca537c878d988e8c4e7db1ff28d0817761a
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445947"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="e603f-101">Dans cet exercice, vous allez découvrir les modifications nécessaires à l’exemple de fonction Azure pour préparer la publication dans une [application Fonctions Azure.](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)</span><span class="sxs-lookup"><span data-stu-id="e603f-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="e603f-102">Mise à jour du code</span><span class="sxs-lookup"><span data-stu-id="e603f-102">Update code</span></span>

<span data-ttu-id="e603f-103">La configuration est lue à partir du magasin de secret utilisateur, qui s’applique uniquement à votre ordinateur de développement.</span><span class="sxs-lookup"><span data-stu-id="e603f-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="e603f-104">Avant de publier sur Azure, vous devez modifier l’endroit où vous stockez votre configuration et mettre à jour le code dans **Program.cs** en conséquence.</span><span class="sxs-lookup"><span data-stu-id="e603f-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Program.cs** accordingly.</span></span>

<span data-ttu-id="e603f-105">Les secrets de l’application doivent être stockés dans un stockage sécurisé, tel [qu’Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span><span class="sxs-lookup"><span data-stu-id="e603f-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="e603f-106">Mettre à jour le paramètre CORS pour azure Function</span><span class="sxs-lookup"><span data-stu-id="e603f-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="e603f-107">Dans cet exemple, nous avons configuré CORS **local.settings.jspour** permettre à l’application de test d’appeler la fonction.</span><span class="sxs-lookup"><span data-stu-id="e603f-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="e603f-108">Vous devez configurer votre fonction publiée pour autoriser les applications SPA qui l’appelleront.</span><span class="sxs-lookup"><span data-stu-id="e603f-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="e603f-109">Mettre à jour les inscriptions d’applications</span><span class="sxs-lookup"><span data-stu-id="e603f-109">Update app registrations</span></span>

<span data-ttu-id="e603f-110">La propriété dans le manifeste pour l’inscription Graph’application Azure Function doit être mise à jour avec les ID d’application de toutes les applications qui appelleront `knownClientApplications` la fonction Azure. </span><span class="sxs-lookup"><span data-stu-id="e603f-110">The `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="e603f-111">Recréer des abonnements existants</span><span class="sxs-lookup"><span data-stu-id="e603f-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="e603f-112">Tous les abonnements créés à l’aide de l’URL de webhook sur votre ordinateur local ou ngrok doivent être recréés à l’aide de l’URL de production de `Notify` la fonction Azure.</span><span class="sxs-lookup"><span data-stu-id="e603f-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>
