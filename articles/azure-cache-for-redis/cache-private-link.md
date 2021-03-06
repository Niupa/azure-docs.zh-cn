---
title: '适用于 Redis 的 azure Cache 与 Azure Private Link (预览版) '
description: Azure 专用终结点是一个网络接口，该接口将你私下并安全地连接到 Azure 缓存，以供 Azure 专用链接提供支持的 Redis。 在本文中，你将了解如何使用 Azure 门户创建 Azure 缓存、Azure 虚拟网络和专用终结点。
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: efba69372f46c9b8a7f2857e37b34ec8c88654a0
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546273"
---
# <a name="azure-cache-for-redis-with-azure-private-link-public-preview"></a>适用于 Redis 的 azure Cache for Azure Private Link (公共预览版) 
在本文中，你将了解如何使用 Azure 门户通过专用终结点为 Redis 实例创建虚拟网络和 Azure Cache。 你还将了解如何将专用终结点添加到现有 Azure Cache for Redis 实例。

Azure 专用终结点是一个网络接口，该接口将你私下并安全地连接到 Azure 缓存，以供 Azure 专用链接提供支持的 Redis。 

## <a name="prerequisites"></a>先决条件
* Azure 订阅 - [创建免费帐户](https://azure.microsoft.com/free/)

> [!IMPORTANT]
> 若要使用专用终结点，需要在2020年7月28日后创建用于 Redis 的 Azure 缓存实例。
> 目前，异地复制、防火墙规则、门户控制台支持、每个群集缓存有多个终结点、不支持永久性到防火墙和 VNet 注入的缓存。 
>
>

## <a name="create-a-private-endpoint-with-a-new-azure-cache-for-redis-instance"></a>使用新的 Azure Cache for Redis 实例创建专用终结点 

在本部分中，将使用专用终结点为 Redis 实例创建新的 Azure Cache。

### <a name="create-a-virtual-network"></a>创建虚拟网络 

1. 登录到 [Azure 门户](https://portal.azure.com) ，然后选择 " **创建资源** "。

    :::image type="content" source="media/cache-private-link/1-create-resource.png" alt-text="选择 &quot;创建资源&quot;。":::

2. 在 " **新建** " 页上，选择 " **网络** "，然后选择 " **虚拟网络** "。

3. 选择 " **添加** " 以创建虚拟网络。

4. 在“创建虚拟网络”  的“基本信息”选项卡中输入或选择以下信息  ：

   | 设置      | 建议的值  | 说明 |
   | ------------ |  ------- | -------------------------------------------------- |
   | **订阅** | 单击下拉箭头并选择你的订阅。 | 要在其下创建此虚拟网络的订阅。 | 
   | **资源组** | 单击下拉箭头并选择一个资源组，或者选择“新建”并输入新的资源组名称。 | 要在其中创建虚拟网络和其他资源的资源组的名称。 将所有应用资源放入一个资源组可以轻松地统一管理或删除这些资源。 | 
   | **名称** | 输入虚拟网络名称。 | 名称必须以字母或数字开头，以字母、数字或下划线结尾，并且只能包含字母、数字、下划线、句点或连字符。 | 
   | **区域** | 下拉，然后选择一个区域。 | 选择其他要使用虚拟网络的服务附近的 [区域](https://azure.microsoft.com/regions/) 。 |

5. 选择 " **Ip 地址** " 选项卡，或单击页面底部的 " **下一步： ip 地址** " 按钮。

6. 在 " **IP 地址** " 选项卡中，将 **IPv4 地址空间** 指定为 CIDR 表示法中的一个或多个地址前缀 (例如 192.168.1.0/24) 。

7. 在 " **子网名称** " 下，单击 " **默认** " 以编辑子网的属性。

8. 在 " **编辑子网** " 窗格中，指定 **子网名称** 和 **子网地址范围** 。 子网的地址范围应为 CIDR 表示法 (例如 192.168.1.0/24) 。 它必须包含在虚拟网络的地址空间中。

9. 选择“保存” 。

10. 选择 " **查看** " 和 "创建" 选项卡，或者单击 " **查看 + 创建** " 按钮。

11. 验证所有信息都正确，然后单击 " **创建** " 以预配虚拟网络。

### <a name="create-an-azure-cache-for-redis-instance-with-a-private-endpoint"></a>使用专用终结点为 Redis 实例创建 Azure 缓存
若要创建缓存实例，请按照以下步骤操作。

1. 返回 Azure 门户主页，或打开 "侧栏" 菜单，然后选择 " **创建资源** "。 
   
1. 在“新建”页上选择“数据库”，然后选择“Azure Cache for Redis”。

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="选择 Azure Cache for Redis。":::
   
1. 在“新建 Redis 缓存”页上配置新缓存的设置。
   
   | 设置      | 建议的值  | 说明 |
   | ------------ |  ------- | -------------------------------------------------- |
   | **DNS 名称** | 输入任何全局唯一的名称。 | 缓存名称必须是包含 1 到 63 个字符的字符串，只能包含数字、字母或连字符。 该名称必须以数字或字母开头和结尾，且不能包含连续的连字符。 缓存实例的主机名是 *\<DNS name> .redis.cache.windows.net* 。 | 
   | **订阅** | 单击下拉箭头并选择你的订阅。 | 要在其下创建此新的 Azure Cache for Redis 实例的订阅。 | 
   | **资源组** | 单击下拉箭头并选择一个资源组，或者选择“新建”并输入新的资源组名称。 | 要在其中创建缓存和其他资源的资源组的名称。 将所有应用资源放入一个资源组可以轻松地统一管理或删除这些资源。 | 
   | **位置** | 单击下拉箭头并选择一个位置。 | 选择与要使用该缓存的其他服务靠近的[区域](https://azure.microsoft.com/regions/)。 |
   | **定价层** | 单击下拉箭头并选择一个[定价层](https://azure.microsoft.com/pricing/details/cache/)。 |  定价层决定可用于缓存的大小、性能和功能。 有关详细信息，请参阅[用于 Redis 的 Azure 缓存概述](cache-overview.md)。 |

1. 选择“网络”选项卡，或单击页面底部的“网络”按钮 。

1. 在 " **网络** " 选项卡中，为连接方法选择 " **专用终结点** "。

1. 单击 " **添加** " 按钮创建专用终结点。

    :::image type="content" source="media/cache-private-link/3-add-private-endpoint.png" alt-text="在 &quot;网络&quot; 中，添加专用终结点。":::

1. 在 " **创建专用终结点** " 页上，使用在上一部分中创建的虚拟网络和子网配置专用终结点的设置，然后选择 **"确定"** 。 

1. 选择页面底部的“下一步:高级”选项卡，或者单击页面底部的“下一步:高级”按钮。

1. 在基本或标准缓存实例的“高级”选项卡中，如果想要启用非 TLS 端口，请选择启用开关。

1. 在高级缓存实例的“高级”选项卡中，配置非 TLS 端口、群集和数据持久性的设置。


1. 选择页面底部的“下一步:标记”选项卡，或者单击“下一步:标记”按钮。

1. 或者，在“标记”选项卡中，如果希望对资源分类，请输入名称或值。 

1. 选择“查看 + 创建”  。 随后你会转到“查看 + 创建”选项卡，Azure 将在此处验证配置。

1. 显示绿色的“已通过验证”消息后，选择“创建”。

创建缓存需要花费片刻时间。 可以在 Azure Cache for Redis 的“概述”页上监视进度。  如果“状态”显示为“正在运行”，则表示该缓存可供使用。  
    
> [!IMPORTANT]
> 
> `publicNetworkAccess`默认情况下有一个标志 `Enabled` 。 
> 此标志旨在允许公共和私有终结点访问缓存（如果将其设置为） `Enabled` 。 如果设置为 `Disabled` ，则它将只允许专用终结点访问。 你可以将值设置为， `Disabled` 并在下面提供修补程序请求。
> ```http
> PATCH  https://management.azure.com/subscriptions/{subscription}/resourceGroups/{resourcegroup}/providers/Microsoft.Cache/Redis/{cache}?api-version=2020-06-01
> {    "properties": {
>        "publicNetworkAccess":"Disabled"
>    }
> }
> ```
>

> [!IMPORTANT]
> 
> 若要连接到群集缓存， `publicNetworkAccess` 需要将设置为 `Disabled` ，并且只能有一个专用终结点连接。 
>

## <a name="create-a-private-endpoint-with-an-existing-azure-cache-for-redis-instance"></a>使用现有的 Redis 实例的 Azure 缓存创建专用终结点 

在本部分中，你将向 Redis 实例的现有 Azure 缓存添加专用终结点。 

### <a name="create-a-virtual-network"></a>创建虚拟网络 
若要创建虚拟网络，请执行以下步骤。

1. 登录到 [Azure 门户](https://portal.azure.com) ，然后选择 " **创建资源** "。

2. 在 " **新建** " 页上，选择 " **网络** "，然后选择 " **虚拟网络** "。

3. 选择 " **添加** " 以创建虚拟网络。

4. 在“创建虚拟网络”  的“基本信息”选项卡中输入或选择以下信息  ：

   | 设置      | 建议的值  | 说明 |
   | ------------ |  ------- | -------------------------------------------------- |
   | **订阅** | 单击下拉箭头并选择你的订阅。 | 要在其下创建此虚拟网络的订阅。 | 
   | **资源组** | 单击下拉箭头并选择一个资源组，或者选择“新建”并输入新的资源组名称。 | 要在其中创建虚拟网络和其他资源的资源组的名称。 将所有应用资源放入一个资源组可以轻松地统一管理或删除这些资源。 | 
   | **名称** | 输入虚拟网络名称。 | 名称必须以字母或数字开头，以字母、数字或下划线结尾，并且只能包含字母、数字、下划线、句点或连字符。 | 
   | **区域** | 下拉，然后选择一个区域。 | 选择其他要使用虚拟网络的服务附近的 [区域](https://azure.microsoft.com/regions/) 。 |

5. 选择 " **Ip 地址** " 选项卡，或单击页面底部的 " **下一步： ip 地址** " 按钮。

6. 在 " **IP 地址** " 选项卡中，将 **IPv4 地址空间** 指定为 CIDR 表示法中的一个或多个地址前缀 (例如 192.168.1.0/24) 。

7. 在 " **子网名称** " 下，单击 " **默认** " 以编辑子网的属性。

8. 在 " **编辑子网** " 窗格中，指定 **子网名称** 和 **子网地址范围** 。 子网的地址范围应为 CIDR 表示法 (例如 192.168.1.0/24) 。 它必须包含在虚拟网络的地址空间中。

9. 选择“保存” 。

10. 选择 " **查看** " 和 "创建" 选项卡，或者单击 " **查看 + 创建** " 按钮。

11. 验证所有信息都正确，然后单击 " **创建** " 以预配虚拟网络。

### <a name="create-a-private-endpoint"></a>创建专用终结点 

若要创建专用终结点，请执行以下步骤。

1. 在 Azure 门户中，搜索 **Redis 的 Azure Cache** ，并按 enter 或从搜索建议中选择它。

    :::image type="content" source="media/cache-private-link/4-search-for-cache.png" alt-text="搜索适用于 Redis 的 Azure Cache。":::

2. 选择要向其添加专用终结点的缓存实例。

3. 在屏幕左侧，选择 **(预览 ") 专用终结点** "。

4. 单击 " **专用终结点** " 按钮以创建专用终结点。

    :::image type="content" source="media/cache-private-link/5-add-private-endpoint.png" alt-text="添加专用终结点。":::

5. 在 " **创建专用终结点" 页** 上，配置专用终结点的设置。

   | 设置      | 建议的值  | 说明 |
   | ------------ |  ------- | -------------------------------------------------- |
   | **订阅** | 单击下拉箭头并选择你的订阅。 | 用于创建此专用终结点的订阅。 | 
   | **资源组** | 单击下拉箭头并选择一个资源组，或者选择“新建”并输入新的资源组名称。 | 要在其中创建专用终结点和其他资源的资源组的名称。 将所有应用资源放入一个资源组可以轻松地统一管理或删除这些资源。 | 
   | **名称** | 输入专用终结点名称。 | 名称必须以字母或数字开头，以字母、数字或下划线结尾，并且只能包含字母、数字、下划线、句点或连字符。 | 
   | **区域** | 下拉，然后选择一个区域。 | 选择其他将使用您的专用终结点的服务附近的 [区域](https://azure.microsoft.com/regions/) 。 |

6. 单击页面底部的 " **下一步：资源** " 按钮。

7. 在 " **资源** " 选项卡中，选择 "订阅"，选择 "资源类型" `Microsoft.Cache/Redis` ，然后选择要将专用终结点连接到的缓存。

8. 单击页面底部的 " **下一步：配置** " 按钮。

9. 在 " **配置** " 选项卡中，选择在上一部分中创建的虚拟网络和子网。

10. 单击页面底部的 " **下一步：标记** " 按钮。

11. 或者，在“标记”选项卡中，如果希望对资源分类，请输入名称或值。

12. 选择“查看 + 创建”  。 你将转到 " **查看** " 和 "创建" 选项卡，Azure 将在其中验证你的配置。

13. 显示绿色 **验证通过** 消息后，选择 " **创建** "。


## <a name="next-steps"></a>后续步骤

若要了解有关 Azure 专用链接的详细信息，请参阅 [Azure 专用链接文档](../private-link/private-link-overview.md)。