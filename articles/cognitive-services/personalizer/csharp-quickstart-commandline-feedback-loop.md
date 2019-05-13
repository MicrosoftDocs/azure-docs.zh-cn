---
title: 反馈循环 - 个性化体验创建服务
titleSuffix: Azure Cognitive Services
description: 在本 C# 快速入门中使用个性化体验创建服务个性化内容。
services: cognitive-services
author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/08/2019
ms.author: edjez
ms.openlocfilehash: 85252680fcc4d2592d242762d01040c3859b14a2
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442083"
---
# <a name="quickstart-personalize-content-using-c"></a>快速入门：使用 C# 个性化内容 

在本 C# 快速入门中使用个性化体验创建服务显示个性化内容。

本示例演示了如何使用适用于 C# 的个性化体验创建服务客户端库执行以下操作： 

 * 为一系列个性化操作排名。
 * 基于用户对指定事件所做的选择，报告要分配给排名最高的操作的奖励。

开始使用个性化体验创建服务功能涉及到以下步骤：

1. 引用 SDK 
1. 编写代码以排名要显示给用户的操作
1. 编写代码以发送奖励来训练循环。

## <a name="prerequisites"></a>先决条件

* 需要有[个性化体验创建服务](how-to-settings.md)才能获取订阅密钥和终结点服务 URL。 
* [Visual Studio 2015 或 2017](https://visualstudio.microsoft.com/downloads/)。
* Microsoft.Azure.CognitiveServices.Personalizer SDK NuGet 包。 下面提供了安装说明。

## <a name="change-the-model-update-frequency"></a>更改模型更新频率

在 Azure 门户中的个性化体验创建服务中，将“模型更新频率”更改为 10 秒。 这将快速训练服务，使你可以看到顶部操作如何针对每次迭代而变化

![更改模型更新频率](./media/settings/configure-model-update-frequency-settings.png)

## <a name="creating-a-new-console-app-and-referencing-the-personalizer-sdk"></a>创建新的控制台应用并引用个性化体验创建服务 SDK 

<!--
Get the latest code as a Visual Studio solution from [GitHub] (add link).
-->

1. 在 Visual Studio 中创建一个新的 Visual C# 控制台应用。
1. 安装个性化体验创建服务客户端库 NuGet 包。 在菜单中，依次选择“工具”、“NuGet 包管理器”、“管理解决方案的 NuGet 包”。
1. 选择“浏览”选项卡，在“搜索”框中键入 `Microsoft.Azure.CognitiveServices.Personalizer`。
1. 当 **Microsoft.Azure.CognitiveServices.Personalizer** 显示时，将其选中。
1. 选中项目名称旁边的复选框，然后选择“安装”。

## <a name="add-the-code-and-put-in-your-personalizer-and-azure-keys"></a>添加代码并输入个性化体验创建服务和 Azure 密钥

1. 将 Program.cs 替换为以下代码： 
1. 将 `serviceKey` 值替换为有效的个性化体验创建服务订阅密钥。
1. 将 `serviceEndpoint` 替换为你的服务终结点。 例如 `https://westus2.api.cognitive.microsoft.com/`。
1. 运行该程序。

## <a name="add-code-to-rank-the-actions-you-want-to-show-to-your-users"></a>添加代码以排名要显示给用户的操作

以下 C# 代码完整演示了如何使用 SDK 向个性化体验创建服务传递用户信息、特征以及有关内容和操作的信息。 个性化体验创建服务将返回要显示给用户的排名最高的操作。  

```csharp
using Microsoft.Azure.CognitiveServices.Personalizer;
using Microsoft.Azure.CognitiveServices.Personalizer.Models;
using System;
using System.Collections.Generic;
using System.Linq;

namespace PersonalizerExample
{
    class Program
    {
        // The key specific to your personalizer service instance; e.g. "0123456789abcdef0123456789ABCDEF"
        private const string ApiKey = "";

        // The endpoint specific to your personalizer service instance; e.g. https://westus2.api.cognitive.microsoft.com/
        private const string ServiceEndpoint = "";

        static void Main(string[] args)
        {
            int iteration = 1;
            bool runLoop = true;

            // Get the actions list to choose from personalizer with their features.
            IList<RankableAction> actions = GetActions();

            // Initialize Personalizer client.
            PersonalizerClient client = InitializePersonalizerClient(ServiceEndpoint);

            do
            {
                Console.WriteLine("\nIteration: " + iteration++);

                // Get context information from the user.
                string timeOfDayFeature = GetUsersTimeOfDay();
                string tasteFeature = GetUsersTastePreference();

                // Create current context from user specified data.
                IList<object> currentContext = new List<object>() {
                    new { time = timeOfDayFeature },
                    new { taste = tasteFeature }
                };

                // Exclude an action for personalizer ranking. This action will be held at its current position.
                IList<string> excludeActions = new List<string> { "juice" };

                // Generate an ID to associate with the request.
                string eventId = Guid.NewGuid().ToString();

                // Rank the actions
                var request = new RankRequest(actions, currentContext, excludeActions, eventId);
                RankResponse response = client.Rank(request);

                Console.WriteLine("\nPersonalizer service thinks you would like to have: " + response.RewardActionId + ". Is this correct? (y/n)");

                float reward = 0.0f;
                string answer = GetKey();

                if (answer == "Y")
                {
                    reward = 1;
                    Console.WriteLine("\nGreat! Enjoy your food.");
                }
                else if (answer == "N")
                {
                    reward = 0;
                    Console.WriteLine("\nYou didn't like the recommended food choice.");
                }
                else
                {
                    Console.WriteLine("\nEntered choice is invalid. Service assumes that you didn't like the recommended food choice.");
                }

                Console.WriteLine("\nPersonalizer service ranked the actions with the probabilities as below:");
                foreach (var rankedResponse in response.Ranking)
                {
                    Console.WriteLine(rankedResponse.Id + " " + rankedResponse.Probability);
                }

                // Send the reward for the action based on user response.
                client.Reward(response.EventId, new RewardRequest(reward));

                Console.WriteLine("\nPress q to break, any other key to continue:");
                runLoop = !(GetKey() == "Q");

            } while (runLoop);
        }

        /// <summary>
        /// Initializes the personalizer client.
        /// </summary>
        /// <param name="url">Azure endpoint</param>
        /// <returns>Personalizer client instance</returns>
        static PersonalizerClient InitializePersonalizerClient(string url)
        {
            PersonalizerClient client = new PersonalizerClient(
                new ApiKeyServiceClientCredentials(ApiKey)) {Endpoint = url};

            return client;
        }

        /// <summary>
        /// Get users time of the day context.
        /// </summary>
        /// <returns>Time of day feature selected by the user.</returns>
        static string GetUsersTimeOfDay()
        {
            string[] timeOfDayFeatures = new string[] { "morning", "afternoon", "evening", "night" };

            Console.WriteLine("\nWhat time of day is it (enter number)? 1. morning 2. afternoon 3. evening 4. night");
            if (!int.TryParse(GetKey(), out int timeIndex) || timeIndex < 1 || timeIndex > timeOfDayFeatures.Length)
            {
                Console.WriteLine("\nEntered value is invalid. Setting feature value to " + timeOfDayFeatures[0] + ".");
                timeIndex = 1;
            }

            return timeOfDayFeatures[timeIndex - 1];
        }

        /// <summary>
        /// Gets user food preference.
        /// </summary>
        /// <returns>Food taste feature selected by the user.</returns>
        static string GetUsersTastePreference()
        {
            string[] tasteFeatures = new string[] { "salty", "sweet" };

            Console.WriteLine("\nWhat type of food would you prefer (enter number)? 1. salty 2. sweet");
            if (!int.TryParse(GetKey(), out int tasteIndex) || tasteIndex < 1 || tasteIndex > tasteFeatures.Length)
            {
                Console.WriteLine("\nEntered value is invalid. Setting feature value to " + tasteFeatures[0] + ".");
                tasteIndex = 1;
            }

            return tasteFeatures[tasteIndex - 1];
        }

        /// <summary>
        /// Creates personalizer actions feature list.
        /// </summary>
        /// <returns>List of actions for personalizer.</returns>
        static IList<RankableAction> GetActions()
        {
            IList<RankableAction> actions = new List<RankableAction>
            {
                new RankableAction
                {
                    Id = "pasta",
                    Features =
                    new List<object>() { new { taste = "salty", spiceLevel = "medium" }, new { nutritionLevel = 5, cuisine = "italian" } }
                },

                new RankableAction
                {
                    Id = "ice cream",
                    Features =
                    new List<object>() { new { taste = "sweet", spiceLevel = "none" }, new { nutritionalLevel = 2 } }
                },

                new RankableAction
                {
                    Id = "juice",
                    Features =
                    new List<object>() { new { taste = "sweet", spiceLevel = "none" }, new { nutritionLevel = 5 }, new { drink = true } }
                },

                new RankableAction
                {
                    Id = "salad",
                    Features =
                    new List<object>() { new { taste = "salty", spiceLevel = "low" }, new { nutritionLevel = 8 } }
                }
            };

            return actions;
        }

        private static string GetKey()
        {
            return Console.ReadKey().Key.ToString().Last().ToString().ToUpper();
        }
    }
}
```

## <a name="run-the-program"></a>运行程序

生成并运行程序。 快速入门程序会提出一些问题来收集用户的偏好（称为“特征”），然后提供排名最高的操作。

![快速入门程序会提出一些问题来收集用户的偏好（称为“特征”），然后提供排名最高的操作。](media/csharp-quickstart-commandline-feedback-loop/quickstart-program-feedback-loop-example.png)

## <a name="clean-up-resources"></a>清理资源
完成本快速入门后，删除在本快速入门中创建的所有文件。 

## <a name="next-steps"></a>后续步骤

[个性化体验创建服务的工作原理](how-personalizer-works.md)


