---
ms.openlocfilehash: f034aca537c878d988e8c4e7db1ff28d0817761a
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445947"
---
<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez découvrir les modifications nécessaires à l’exemple de fonction Azure pour préparer la publication dans une [application Fonctions Azure.](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)

## <a name="update-code"></a>Mise à jour du code

La configuration est lue à partir du magasin de secret utilisateur, qui s’applique uniquement à votre ordinateur de développement. Avant de publier sur Azure, vous devez modifier l’endroit où vous stockez votre configuration et mettre à jour le code dans **Program.cs** en conséquence.

Les secrets de l’application doivent être stockés dans un stockage sécurisé, tel [qu’Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).

## <a name="update-cors-setting-for-azure-function"></a>Mettre à jour le paramètre CORS pour azure Function

Dans cet exemple, nous avons configuré CORS **local.settings.jspour** permettre à l’application de test d’appeler la fonction. Vous devez configurer votre fonction publiée pour autoriser les applications SPA qui l’appelleront.

## <a name="update-app-registrations"></a>Mettre à jour les inscriptions d’applications

La propriété dans le manifeste pour l’inscription Graph’application Azure Function doit être mise à jour avec les ID d’application de toutes les applications qui appelleront `knownClientApplications` la fonction Azure. 

## <a name="recreate-existing-subscriptions"></a>Recréer des abonnements existants

Tous les abonnements créés à l’aide de l’URL de webhook sur votre ordinateur local ou ngrok doivent être recréés à l’aide de l’URL de production de `Notify` la fonction Azure.
