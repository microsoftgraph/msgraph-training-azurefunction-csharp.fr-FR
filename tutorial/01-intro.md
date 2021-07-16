---
ms.openlocfilehash: d35318e05a5ebae2316afeb84b731da5c8342ddc
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445968"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f5a2d-101">Ce didacticiel vous apprend à créer une fonction Azure qui utilise l’API Microsoft Graph pour récupérer des informations de calendrier pour un utilisateur.</span><span class="sxs-lookup"><span data-stu-id="f5a2d-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="f5a2d-102">Si vous préférez simplement télécharger le didacticiel terminé, vous pouvez télécharger ou cloner [GitHub référentiel.](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)</span><span class="sxs-lookup"><span data-stu-id="f5a2d-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="f5a2d-103">Consultez le fichier README dans le dossier **de** démonstration pour obtenir des instructions sur la configuration de l’application avec un ID d’application et une secret.</span><span class="sxs-lookup"><span data-stu-id="f5a2d-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="f5a2d-104">Conditions préalables</span><span class="sxs-lookup"><span data-stu-id="f5a2d-104">Prerequisites</span></span>

<span data-ttu-id="f5a2d-105">Avant de commencer ce didacticiel, les outils suivants doivent être installés sur votre ordinateur de développement.</span><span class="sxs-lookup"><span data-stu-id="f5a2d-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="f5a2d-106">SDK .NET Core</span><span class="sxs-lookup"><span data-stu-id="f5a2d-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="f5a2d-107">Outils azure Functions Core</span><span class="sxs-lookup"><span data-stu-id="f5a2d-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="f5a2d-108">Azure CLI</span><span class="sxs-lookup"><span data-stu-id="f5a2d-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="f5a2d-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="f5a2d-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="f5a2d-110">Vous devez également avoir un compte scolaire ou scolaire Microsoft, avec accès à un compte d’administrateur général dans la même organisation.</span><span class="sxs-lookup"><span data-stu-id="f5a2d-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="f5a2d-111">Si vous n’avez pas de [](https://developer.microsoft.com/office/dev-program) compte Microsoft, vous pouvez vous inscrire au programme Office 365 développeur pour obtenir un abonnement Office 365 gratuit.</span><span class="sxs-lookup"><span data-stu-id="f5a2d-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="f5a2d-112">Ce didacticiel a été écrit avec les versions suivantes des outils ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="f5a2d-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="f5a2d-113">Les étapes de ce guide peuvent fonctionner avec d’autres versions, mais elles n’ont pas été testées.</span><span class="sxs-lookup"><span data-stu-id="f5a2d-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="f5a2d-114">.NET Core SDK 5.0.203</span><span class="sxs-lookup"><span data-stu-id="f5a2d-114">.NET Core SDK 5.0.203</span></span>
> - <span data-ttu-id="f5a2d-115">Azure Functions Core Tools 3.0.3442</span><span class="sxs-lookup"><span data-stu-id="f5a2d-115">Azure Functions Core Tools 3.0.3442</span></span>
> - <span data-ttu-id="f5a2d-116">Azure CLI 2.23.0</span><span class="sxs-lookup"><span data-stu-id="f5a2d-116">Azure CLI 2.23.0</span></span>
> - <span data-ttu-id="f5a2d-117">ngrok 2.3.40</span><span class="sxs-lookup"><span data-stu-id="f5a2d-117">ngrok 2.3.40</span></span>

## <a name="feedback"></a><span data-ttu-id="f5a2d-118">Commentaires</span><span class="sxs-lookup"><span data-stu-id="f5a2d-118">Feedback</span></span>

<span data-ttu-id="f5a2d-119">Veuillez fournir des commentaires sur ce didacticiel dans [GitHub référentiel.](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)</span><span class="sxs-lookup"><span data-stu-id="f5a2d-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>
