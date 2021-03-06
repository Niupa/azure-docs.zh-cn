---
title: 用于为 Azure Cosmos MongoDB API 数据库和集合创建资源锁的 PowerShell 脚本
description: 为 Azure Cosmos MongoDB API 数据库和集合创建资源锁
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 06/12/2020
ms.openlocfilehash: b9b743e9f293455c09557348c53cc76b1f904b0a
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92489126"
---
# <a name="create-a-resource-lock-for-azure-cosmos-mongodb-api-database-and-collection-using-azure-powershell"></a>使用 Azure PowerShell 为 Azure Cosmos MongoDB API 数据库和集合创建资源锁

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

> [!IMPORTANT]
> 除非在启用了 `disableKeyBasedMetadataWriteAccess` 属性的情况下先锁定 Cosmos DB 帐户，否则资源锁对使用任何 MongoDB SDK、Mongoshell、任何工具或 Azure 门户进行连接的用户所做的更改均无效。 要详细了解如何启用此属性，请参阅[防止 SDK 更改](../../../role-based-access-control.md#prevent-sdk-changes)。

## <a name="sample-script"></a>示例脚本

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/mongodb/ps-mongodb-lock.ps1 "Create, list, and remove resource locks")]

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
|**Azure 资源**| |
| [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock) | 创建资源锁。 |
| [Get-AzResourceLock](/powershell/module/az.resources/get-azresourcelock) | 获取资源锁，或列出资源锁。 |
| [Remove-AzResourceLock](/powershell/module/az.resources/remove-azresourcelock) | 删除资源锁。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](/powershell/)。