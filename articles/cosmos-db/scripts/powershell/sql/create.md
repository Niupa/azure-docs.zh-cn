---
title: 用于创建 Azure Cosmos DB SQL API 数据库和容器的 PowerShell 脚本
description: Azure PowerShell 脚本 - Azure Cosmos DB 创建 SQL API 数据库和容器
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 05/13/2020
ms.author: mjbrown
ms.openlocfilehash: 3ee4036a605e74ff43b3951b41ee3b4d1325004d
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92482037"
---
# <a name="create-a-database-and-container-for-azure-cosmos-db---sql-api"></a>创建 Azure Cosmos DB 的数据库和容器 - SQL API

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

此脚本在两个具有会话级一致性的区域、一个数据库和一个具有分区键、自定义索引策略、唯一键策略、TTL、专用吞吐量和“以最后一次写入为准”冲突解决策略的容器中使用在 `multipleWriteLocations=true` 时将使用的自定义冲突解决路径为 SQL (Core) API 创建 Cosmos 帐户。

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/sql/ps-sql-create.ps1 "Create an account, database, and container for SQL API")]

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
|**Azure Cosmos DB**| |
| [New-AzCosmosDBAccount](/powershell/module/az.cosmosdb/new-azcosmosdbaccount) | 创建 Cosmos DB 帐户。 |
| [New-AzCosmosDBSqlDatabase](/powershell/module/az.cosmosdb/new-azcosmosdbsqldatabase) | 创建 Cosmos DB SQL 数据库。 |
| [New-AzCosmosDBSqlUniqueKey](/powershell/module/az.cosmosdb/new-azcosmosdbsqluniquekey) | 创建一个 PSSqlUniqueKey 对象，用作 New-AzCosmosDBSqlUniqueKeyPolicy 的参数。 |
| [New-AzCosmosDBSqlUniqueKeyPolicy](/powershell/module/az.cosmosdb/new-azcosmosdbsqluniquekeypolicy) | 创建一个 PSSqlUniqueKeyPolicy 对象，用作 New-AzCosmosDBSqlContainer 的参数。 |
| [New-AzCosmosDBSqlCompositePath](/powershell/module/az.cosmosdb/new-azcosmosdbsqlcompositepath) | 创建一个 PSCompositePath 对象，用作 New-AzCosmosDBSqlIndexingPolicy 的参数。 |
| [New-AzCosmosDBSqlIncludedPathIndex](/powershell/module/az.cosmosdb/new-azcosmosdbsqlincludedpathindex) | 创建一个 PSIndexes 对象，用作 New-AzCosmosDBSqlIncludedPath 的参数。 |
| [New-AzCosmosDBSqlIncludedPath](/powershell/module/az.cosmosdb/new-azcosmosdbsqlincludedpath) | 创建一个 PSIncludedPath 对象，用作 New-AzCosmosDBSqlIndexingPolicy 的参数。 |
| [New-AzCosmosDBSqlIndexingPolicy](/powershell/module/az.cosmosdb/new-azcosmosdbsqlindexingpolicy) | 创建一个 PSSqlIndexingPolicy 对象，用作 New-AzCosmosDBSqlContainer 的参数。 |
| [New-AzCosmosDBSqlConflictResolutionPolicy](/powershell/module/az.cosmosdb/new-azcosmosdbsqlconflictresolutionpolicy) | 创建一个 PSSqlConflictResolutionPolicy 对象，用作 New-AzCosmosDBSqlContainer 的参数。 |
| [New-AzCosmosDBSqlContainer](/powershell/module/az.cosmosdb/new-azcosmosdbsqlcontainer) | 创建新 Cosmos DB SQL 容器。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](/powershell/)。