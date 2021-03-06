---
title: 适用于 v2 的文本分析 API 迁移指南
titleSuffix: Azure Cognitive Services
description: 了解如何将应用程序移到使用版本3的文本分析 API。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 10/19/2020
ms.author: aahi
ms.openlocfilehash: 172e684c6edbab4d7d47c8cf78e35ae38de3a0af
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92461781"
---
# <a name="migrate-to-version-3x-of-the-text-analytics-api"></a>迁移到文本分析 API 的版本1。x

[!INCLUDE [v3 region availability](includes/v3-region-availability.md)]

如果使用文本分析 API 版本2.1，本文将帮助你将应用程序升级为使用版本2.x。 版本3.0 已正式发布，并引入了新功能，例如扩展的 [命名实体识别 (NER) ](how-tos/text-analytics-how-to-entity-linking.md#named-entity-recognition-versions-and-features) 和 [模型版本控制](concepts/model-versioning.md)。 此外，还提供了版本 3.1)  (的版本3.1 预览版本，它可添加 [观点挖掘](how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features)等功能。 V2 中使用的模型将不会收到将来的更新。 

#### <a name="sentiment-analysis"></a>[情绪分析](#tab/sentiment-analysis)

## <a name="feature-changes"></a>功能更改 

版本2.1 中的情绪分析将为发送到 API 的每个文档返回介于0和1之间的情绪评分，分数将接近1，表示更积极的情绪。 版本3改为返回情绪标签 (如 "正值" 或 "负值" ) 作为一个整体的句子和文档及其关联的置信度分数。 

## <a name="steps-to-migrate"></a>迁移步骤

### <a name="rest-api"></a>REST API

如果应用程序使用 REST API，请将其请求终结点更新为情绪分析的 v3 终结点。 例如： `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/sentiment` 。 还需要更新应用程序，以使用 [API 响应](how-tos/text-analytics-how-to-sentiment-analysis.md#view-the-results)中返回的情绪标签。 

有关 JSON 响应的示例，请参阅参考文档。
* [版本 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c9)
* [版本 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Sentiment) 
* [版本 3.1-preview](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/Sentiment)

### <a name="client-libraries"></a>客户端库

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

#### <a name="ner-and-entity-linking"></a>[NER 和实体链接](#tab/named-entity-recognition)

## <a name="feature-changes"></a>功能更改

> [!NOTE] 
> 目前， [v3 实体类别](named-entity-types.md) 仅在英语和西班牙语文本上返回。 API 为其他语言的请求返回2.1 版结果，前提是它们在版本2.1 中受支持。

在版本2.1 中，文本分析 API 为命名实体识别使用一个终结点 (NER) 和实体链接。 版本3提供了扩展的命名实体检测，并为 NER 和实体链接请求使用了单独的终结点。 从第 3.1-NER 开始，还可以检测个人 `pii` 和健康 `phi` 信息。 

## <a name="steps-to-migrate"></a>迁移步骤

### <a name="rest-api"></a>REST API

如果应用程序使用 REST API，请将其请求终结点更新为 NER 和/或实体链接的 v3 终结点。

实体链接
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/linking`

NER
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/recognition/general`

还需要更新应用程序，以使用[API 响应](how-tos/text-analytics-how-to-entity-linking.md#view-results)中返回的[实体类别](named-entity-types.md)。

有关 JSON 响应的示例，请参阅参考文档。
* [版本 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/5ac4251d5b4ccd1554da7634)
* [版本 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/EntitiesRecognitionGeneral) 
* [版本 3.1-preview](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/EntitiesRecognitionGeneral)

### <a name="client-libraries"></a>客户端库

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]


#### <a name="language-detection"></a>[语言检测](#tab/language-detection)

## <a name="feature-changes"></a>功能更改 

此语言检测功能在终结点版本之外的 v3 中未发生更改，但 JSON 响应将包含 `ConfidenceScore` 而不是 `score` 。 V3 还仅返回输出中的一种语言。 

## <a name="steps-to-migrate"></a>迁移步骤

### <a name="rest-api"></a>REST API

如果你的应用程序使用 REST API，请将其请求终结点更新为用于语言检测的 v3 终结点。 例如： `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/languages` 。 还需要更新应用程序，以便 `ConfidenceScore` `score` 在 [API 的响应](how-tos/text-analytics-how-to-language-detection.md#step-3-view-the-results)中使用而不是。 

有关 JSON 响应的示例，请参阅参考文档。
* [版本 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7)
* [版本 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages) 
* [版本 3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/Languages)

### <a name="client-libraries"></a>客户端库

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]


#### <a name="key-phrase-extraction"></a>[关键短语提取](#tab/key-phrase-extraction)

## <a name="feature-changes"></a>功能更改 

终结点版本之外的 v3 中的关键短语提取功能尚未更改。

## <a name="steps-to-migrate"></a>迁移步骤

### <a name="rest-api"></a>REST API

如果你的应用程序使用 REST API，请将其请求终结点更新到 v3 终结点以进行关键短语提取。 例如：`https://<your-custom-subdomain>.api.cognitiveservices.azure.com/text/analytics/v3.0/keyPhrases`

有关 JSON 响应的示例，请参阅参考文档。
* [版本 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c6)
* [版本 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases) 
* [版本 3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/KeyPhrases)

### <a name="client-libraries"></a>客户端库

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

---


## <a name="see-also"></a>另请参阅

* [什么是文本分析 API](overview.md)
* [语言支持](language-support.md)
* [模型版本控制](concepts/model-versioning.md)
