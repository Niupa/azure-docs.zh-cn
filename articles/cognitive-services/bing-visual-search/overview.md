---
title: 什么是必应视觉搜索 API？
titleSuffix: Azure Cognitive Services
description: 必应视觉搜索提供与图像相关的详细信息或见解，如类似图像或购物源。
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: overview
ms.date: 12/19/2019
ms.author: scottwhi
ms.openlocfilehash: 2eab79d79a287bc8a92133c6901c420dfaee2fd5
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93082039"
---
# <a name="what-is-the-bing-visual-search-api"></a>什么是必应视觉搜索 API？

> [!WARNING]
> 必应搜索 API 将从认知服务迁移到必应搜索服务。 从 2020 年 10 月 30 日开始，必应搜索的任何新实例都需按照[此处](https://aka.ms/cogsvcs/bingmove)所述的过程进行预配。
> 使用认知服务进行预配的必应搜索 API 将在未来三年或在企业协议结束前（以先发生者为准）得到支持。
> 有关迁移说明，请参阅[必应搜索服务](https://aka.ms/cogsvcs/bingmigration)。

必应视觉搜索 API 返回图像见解。 你可以上传图像，也可以提供图像的 URL。 见解是视觉上类似的图像、购物源、包含图像的网页，等等。 必应视觉搜索 API 返回的见解类似于 Bing.com/images 上显示的见解。 

如果使用[必应图像搜索 API](../bing-image-search/overview.md)，则可以使用该 API 的搜索结果中的见解标记，而无需上传图像。

> [!IMPORTANT]
> 如果使用必应图像搜索 API 获取图像见解，请考虑切换到必应视觉搜索 API，因为它可以提供更全面的见解。

## <a name="insights"></a>洞察力

可以使用必应视觉搜索发现以下见解：

| 见解                              | 说明 |
|--------------------------------------|-------------|
| 视觉上相似的图像              | 与输入图像在外观上相似的图像列表。 |
| 外观相似的产品            | 与所示产品外观相似的产品。            |
| 购物源                     | 可以购买输入图像中所示项的位置。            |
| 相关搜索                     | 其他人执行的相关搜索或基于图像内容的相关搜索。            |
| 包含图像的网页     | 包含输入图像的网页。            |
| 食谱                              | 包含输入图像中所示食物的食谱的网页。            |
| 实体                             | 知名人物、场所和事物。 |

除了这些见解，必应视觉搜索还可以返回各种派生自输入图像的不同字词（即标记）。 这些标记可让用户浏览在图像中找到的概念。 例如，如果输入图像是一位著名运动员，则其中一个标记可能是运动员的姓名，另一个标记可能是“体育”。 或者，如果输入图像是一个苹果馅饼，则标记可能是“苹果馅饼”、“馅饼”和“甜点”。

必应视觉搜索结果还包含图像中感兴趣区域对应的边框。 例如，如果图像包含多个名人，结果可能包含每个已识别名人对应的边框。 或者，如果必应识别图像中的产品或服装，结果可能包含已识别项对应的边框。

## <a name="workflow"></a>工作流

必应视觉搜索 API 是一项 RESTful Web 服务，可以轻松地通过任何编程语言调用，只要该语言能够发出 HTTP 请求和分析 JSON 即可。 可以通过 REST API 或 SDK 使用此服务。

1. 创建一个[认知服务帐户](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)用于访问必应搜索 API。 如果没有 Azure 订阅，可以[免费创建一个帐户](https://azure.microsoft.com/free/cognitive-services/)。
2. 使用有效的搜索查询向 API 发送请求。
3. 通过分析返回的 JSON 消息处理 API 响应。

## <a name="next-steps"></a>后续步骤

首先，请尝试必应视觉搜索 API [交互式演示](https://azure.microsoft.com/services/cognitive-services/bing-visual-search/)。
该演示介绍如何快速自定义搜索查询并在 Web 中搜索图像。

若要快速了解如何使用第一个请求，请参阅快速入门：

* [C#](quickstarts/csharp.md)

* [Java](quickstarts/java.md)

* [node.js](quickstarts/nodejs.md)

* [Python](quickstarts/python.md)

## <a name="see-also"></a>另请参阅

* [图像 - 视觉搜索](https://docs.microsoft.com/rest/api/cognitiveservices/bingvisualsearch/images/visualsearch)参考文档介绍了有关终结点、请求标头、响应和查询参数的定义和信息，这些都可以用来请求基于图像的搜索结果。

* [必应搜索 API 用法和显示要求](../bing-web-search/use-display-requirements.md)指定了允许用户如何使用通过必应搜索 API 获得的内容和信息。

* 请访问[必应搜索 API 中心页](../bing-web-search/search-the-web.md)，浏览其他可用的 API。