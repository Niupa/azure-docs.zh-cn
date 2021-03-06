---
title: 常见问题解答 - Azure Synapse Analytics（工作区预览版）
description: Azure Synapse Analytics（工作区预览版）的常见问题解答
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: overview
ms.date: 04/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: d7e8ea231dd6db9eab6cf9115f6ce2e62893371a
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305405"
---
# <a name="azure-synapse-analytics-workspaces-preview-frequently-asked-questions"></a>Azure Synapse Analytics（工作区预览版）常见问题解答

本指南解答有关 Synapse Analytics 的最常见问题。

[!INCLUDE [preview](includes/note-preview.md)]

## <a name="general"></a>常规

### <a name="q-what-is-azure-synapse-analytics"></a>问：什么是 Azure Synapse Analytics

答：Azure Synapse 是实现 BI、AI 和持续智能的集成式数据平台。 它通过单个平台连接各种分析运行时（例如 SQL 和 Apache Spark），该平台提供了统一的方式来实现以下目的：

- 保护分析资源（包括网络），管理对池、数据和开发项目的单一登录访问。
- 轻松监视和快速优化、应对和调试在任意层的工作区活动中发生的事件。
- 跨引擎管理元数据。 创建无服务器 Apache Spark 池表，该表将在 Azure Synapse 数据库中自动提供。
- 通过统一的用户体验来与数据交互。 Synapse Studio 将大数据开发人员、数据工程师、数据库管理员 (DBA)、数据分析师和数据科学家聚合到了同一个平台。

有关详细信息，请参阅[什么是 Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics/overview-what-is)。

### <a name="q-what-are-the-main-components-of-azure-synapse-analytics"></a>问：Azure Synapse Analytics 有哪些主要组件

答：Azure Synapse 具有以下功能：

- 分析功能是通过专用 SQL 池或无服务器 SQL 池（预览版）提供的。
- 完全支持 Scala、Python、Spark、SQL 和 C# 的无服务器 Apache Spark 池（预览版）
- 提供无代码大数据转换体验的数据流
- 用于集成数据以及将所有代码开发操作化的数据集成和业务流程
- 用于通过单个 Web UI 访问所有这些功能的 Studio

### <a name="q-how-does-azure-synapse-analytics-relate-to-azure-sql-data-warehouse"></a>问：Azure Synapse Analytics 与 Azure SQL 数据仓库之间有何关系

答：Azure Synapse Analytics 是在 Azure SQL 数据仓库基础上演进而来的分析平台，其中包含用作数据仓库解决方案的专用 SQL 池。 此平台结合了数据探索、引入、转换、准备和一个为分析提供服务的层。

## <a name="use-cases"></a>用例

### <a name="q-how-do-i-rename-a-published-artifact-dataset-notebook-sql-script-and-so-on-in-azure-synapse"></a>问：如何实现在 Azure Synapse 中重命名已发布的项目（数据集、笔记本、sql 脚本等）？

答：若要重命名已发布的项目文件，请首先克隆该文件，然后将新文件重命名为你喜欢的名称。 需要将项目的所有引用手动更新为新文件名并删除旧文件名。

### <a name="q-what-is-a-good-use-case-for-dedicated-sql-pool"></a>问：专用 SQL 池的典型用例是什么

答：专用 SQL 池从本质上满足了你的数据仓库需求。 从[性价比](https://azure.microsoft.com/services/sql-data-warehouse/compare/)而言，它是领先的数据仓库解决方案。 专用 SQL 池是行业领先的云数据仓库解决方案，因为它可以让你：

- 为混合类型的大型工作负荷提供服务，而不会对性能造成影响，这得益于它提供高并发性和工作负荷隔离
- 通过从网络安全性到精细访问的各种高级功能轻松保护数据
- 受益于各种生态系统

### <a name="q-what-is-a-good-use-case-for-serverless-apache-spark-pool-in-azure-synapse"></a>问：Azure Synapse 中无服务器 Apache Spark 池的典型用例是什么

答：我们的首要目标是提供极佳的数据工程体验，从而以批或流的形式通过数据湖转换数据。 Spark 与我们的数据业务流程之间存在简单而又紧密的集成，使开发工作的操作化变得十分简单。

Apache Spark 的另一个用例是使数据科学家能够：

- 提取特征
- 探索数据
- 训练模型

### <a name="q-what-is-a-good-use-case-for-serverless-sql-pool-in-azure-synapse"></a>问：Azure Synapse 中无服务器 SQL 池的典型用例是什么

答：无服务器 SQL 池是针对数据湖中的数据运行的查询服务。 它提供熟悉的 T-SQL 语法，用于就地查询数据而无需将数据复制或加载到专用存储，使你能够让对所有数据的访问都变得简单易行。

用例包括：

- 基本发现和探索 - 为数据分析师、越来越多的数据科学家和数据工程师提供一种简单的途径，让他们可以在读取 T-SQL 查询中使用架构来初步洞察其数据湖中驻留的数据
- 逻辑数据仓库 - 数据分析师可以运用 T-SQL 语言的完全表达性来直接查询和分析 Azure 存储中驻留的数据，并使用熟悉的 BI 工具（例如 Azure Analyses Services、Power BI Premium 等）通过重新运行 Starlight Query 查询来刷新仪表板
- “单一查询”ETL - 可让数据工程师将基于 Azure 存储的数据从一种格式转换为另一种格式，以大规模并行处理方式执行筛选、聚合等操作，将查询结果保存到 Azure 存储，并使其立即可用于 Starlight Query 或其他相关服务中的进一步处理

### <a name="q-what-is-a-good-use-case-for-data-flow-in-azure-synapse"></a>问：Azure Synapse 中的数据流的典型用例是什么

答：数据工程师可以使用数据流开发图形化数据转换逻辑，而无需编写代码。 生成的数据流将在数据集成和业务流程中作为活动执行。 可以通过现有的计划、控制、流和监视功能，将数据流活动操作化。

## <a name="security-and-access"></a>安全性和访问

答：端到端单一登录体验是 Synapse Analytics 中的一个重要身份验证过程。 必须通过完整的 Azure AD 集成来管理和传递标识。

### <a name="q-how-do-i-get-access-to-files-and-folders-in-the-adls-gen2"></a>问：如何获取对 ADLS Gen2 中的文件和文件夹的访问权限

答：对文件和文件夹的访问权限目前是通过 ADLS Gen2 管理的。 有关详细信息，请参阅 [Data Lake Storage 访问控制](../storage/blobs/data-lake-storage-access-control.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)。

### <a name="q-can-i-use-third-party-business-intelligence-tools-to-access-azure-synapse-analytics"></a>问：是否可以使用第三方商业智能工具来访问 Azure Synapse Analytics

答：是的，可以使用第三方商业应用程序（例如 Tableau 和 Power BI）连接到专用 SQL 池和无服务器 SQL 池。 Spark 支持 IntelliJ。

## <a name="next-steps"></a>后续步骤

- [创建工作区](quickstart-create-workspace.md)
- [使用 Synapse Studio](quickstart-synapse-studio.md)
- [创建专用 SQL 池](quickstart-create-sql-pool-portal.md)
- [使用无服务器 SQL 池](quickstart-sql-on-demand.md)
- [创建无服务器 Apache Spark 池](quickstart-create-apache-spark-pool-portal.md) 
