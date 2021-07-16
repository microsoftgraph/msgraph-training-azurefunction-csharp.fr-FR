---
ms.openlocfilehash: d35318e05a5ebae2316afeb84b731da5c8342ddc
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445968"
---
<!-- markdownlint-disable MD002 MD041 -->

Ce didacticiel vous apprend à créer une fonction Azure qui utilise l’API Microsoft Graph pour récupérer des informations de calendrier pour un utilisateur.

> [!TIP]
> Si vous préférez simplement télécharger le didacticiel terminé, vous pouvez télécharger ou cloner [GitHub référentiel.](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp) Consultez le fichier README dans le dossier **de** démonstration pour obtenir des instructions sur la configuration de l’application avec un ID d’application et une secret.

## <a name="prerequisites"></a>Conditions préalables

Avant de commencer ce didacticiel, les outils suivants doivent être installés sur votre ordinateur de développement.

- [SDK .NET Core](https://dotnet.microsoft.com/download)
- [Outils azure Functions Core](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

Vous devez également avoir un compte scolaire ou scolaire Microsoft, avec accès à un compte d’administrateur général dans la même organisation. Si vous n’avez pas de [](https://developer.microsoft.com/office/dev-program) compte Microsoft, vous pouvez vous inscrire au programme Office 365 développeur pour obtenir un abonnement Office 365 gratuit.

> [!NOTE]
> Ce didacticiel a été écrit avec les versions suivantes des outils ci-dessus. Les étapes de ce guide peuvent fonctionner avec d’autres versions, mais elles n’ont pas été testées.
>
> - .NET Core SDK 5.0.203
> - Azure Functions Core Tools 3.0.3442
> - Azure CLI 2.23.0
> - ngrok 2.3.40

## <a name="feedback"></a>Commentaires

Veuillez fournir des commentaires sur ce didacticiel dans [GitHub référentiel.](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)
