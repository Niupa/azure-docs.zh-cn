---
title: 适用于 Azure Cosmos DB 表 API 的 Azure PowerShell 示例
description: 获取用于在 Azure Cosmos DB 表 API 中执行常见任务的 Azure PowerShell 示例
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 10/13/2020
ms.author: mjbrown
ms.openlocfilehash: d48d16d13bf35636950366cba7cd946f01afcff0
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92282915"
---
# <a name="azure-powershell-samples-for-azure-cosmos-db-table-api"></a>适用于 Azure Cosmos DB 表 API 的 Azure PowerShell 示例

下表包含常用于 Azure Cosmos DB 的 Azure PowerShell 脚本的链接。 使用右侧的链接可导航到 API 特定示例。 常见示例在所有 API 间是相同的。 [Azure PowerShell 参考](/powershell/module/az.cosmosdb)中收录了所有 Azure Cosmos DB PowerShell cmdlet 的参考页。 请定期检查 `Az.CosmosDB` 是否有更新。 还可以从我们的 GitHub 存储库 [GitHub 上的 Cosmos DB PowerShell 示例](https://github.com/Azure/azure-docs-powershell-samples/tree/master/cosmosdb)创建这些适用于 Cosmos DB 的 PowerShell 示例的分支。

## <a name="common-samples"></a>常见示例

|任务 | 说明 |
|---|---|
|[更新帐户](scripts/powershell/common/account-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 更新 Cosmos DB 帐户的默认一致性级别。 |
|[更新帐户的区域](scripts/powershell/common/update-region.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 更新 Cosmos DB 帐户的区域。 |
|[更改故障转移优先级或触发故障转移](scripts/powershell/common/failover-priority-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 更改 Azure Cosmos 帐户的区域故障转移优先级或触发手动故障转移。 |
|[帐户密钥或连接字符串](scripts/powershell/common/keys-connection-strings.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 获取 Azure Cosmos DB 帐户的主密钥和辅助密钥、连接字符串或重新生成其帐户密钥。 |
|[创建启用 IP 防火墙的 Cosmos 帐户](scripts/powershell/common/firewall-create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 创建启用 IP 防火墙的 Azure Cosmos DB 帐户。 |
|||

## <a name="table-api-samples"></a>表 API 示例

|任务 | 说明 |
|---|---|
|[创建帐户和表](scripts/powershell/table/create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 创建 Azure Cosmos 帐户和表。 |
|[创建帐户和表（具有自动缩放功能）](scripts/powershell/table/autoscale.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 创建 Azure Cosmos 帐户和表（具有自动缩放功能）。 |
|[列出或获取表](scripts/powershell/table/list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 列出或获取表。 |
|[吞吐量操作](scripts/powershell/table/throughput.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 表的吞吐量操作，包括获取、更新自动缩放和标准吞吐量和在其之间进行迁移。 |
|[锁定资源以防止将其删除](scripts/powershell/table/lock.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 通过资源锁防止资源遭到删除。 |
|||
