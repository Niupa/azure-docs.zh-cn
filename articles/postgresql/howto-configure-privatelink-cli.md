---
title: 专用链接 - Azure CLI - Azure Database for PostgreSQL（单一服务器）
description: 了解如何从 Azure CLI 配置 Azure Database for PostgreSQL（单一服务器）的专用链接
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 01/09/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 45f5a7e66c80dff5e78e575463becd95bcc7fca1
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/03/2020
ms.locfileid: "93242171"
---
# <a name="create-and-manage-private-link-for-azure-database-for-postgresql---single-server-using-cli"></a>使用 CLI 创建和管理 Azure Database for PostgreSQL（单一服务器）的专用链接

专用终结点是 Azure 中专用链接的构建基块。 它使 Azure 资源（例如虚拟机 (VM)）能够以私密方式来与专用链接资源通信。 本文介绍如何使用 Azure CLI 在 Azure 虚拟网络和带有 Azure 专用终结点的 Azure Database for PostgreSQL 单一服务器中创建 VM。

> [!NOTE]
> 专用链接功能仅适用于“常规用途”或“内存优化”定价层中的 Azure Database for PostgreSQL 服务器。 请确保数据库服务器位于其中一个定价层中。

## <a name="prerequisites"></a>先决条件

若要逐步执行本操作方法指南，需要：

- [Azure Database for PostgreSQL 服务器和数据库](quickstart-create-server-database-azure-cli.md)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果决定在本地安装并使用 Azure CLI，本快速入门要求使用 Azure CLI 2.0.28 或更高版本。 若要查找已安装的版本，请运行 `az --version`。 有关安装或升级信息，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

在创建任何资源之前，必须创建一个资源组以托管虚拟网络。 使用 [az group create](/cli/azure/group) 创建资源组。 此示例在 " *westeurope* " 位置创建名为 " *myResourceGroup* " 的资源组：

```azurecli-interactive
az group create --name myResourceGroup --location westeurope
```

## <a name="create-a-virtual-network"></a>创建虚拟网络
使用 [az network vnet create](/cli/azure/network/vnet) 创建虚拟网络。 此示例创建名为 *myVirtualNetwork* 的默认虚拟网络，它具有一个名为 *mySubnet* 的子网：

```azurecli-interactive
az network vnet create \
 --name myVirtualNetwork \
 --resource-group myResourceGroup \
 --subnet-name mySubnet
```

## <a name="disable-subnet-private-endpoint-policies"></a>禁用子网专用终结点策略 
Azure 会将资源部署到虚拟网络中的子网，因此，你需要创建或更新子网，以禁用专用终结点[网络策略](../private-link/disable-private-endpoint-network-policy.md)。 使用 [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) 更新名为 *mySubnet* 的子网配置：

```azurecli-interactive
az network vnet subnet update \
 --name mySubnet \
 --resource-group myResourceGroup \
 --vnet-name myVirtualNetwork \
 --disable-private-endpoint-network-policies true
```
## <a name="create-the-vm"></a>创建 VM 
使用 az vm create 创建 VM。 出现提示时，请提供要用作 VM 登录凭据的密码。 此示例创建名为 *myVm* 的 VM： 
```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image Win2019Datacenter
```
 记下 VM 的公共 IP 地址。 在下一步中，此地址将用于从 Internet 连接到 VM。

## <a name="create-an-azure-database-for-postgresql---single-server"></a>创建 Azure Database for PostgreSQL（单一服务器） 
使用 az postgres server create 命令创建 Azure Database for PostgreSQL。 请记住，PostgreSQL 服务器的名称必须在 Azure 中是唯一的，因此请将占位符值替换为你在上面使用的唯一值： 

```azurecli-interactive
# Create a server in the resource group 
az postgres server create \
--name mydemoserver \
--resource-group myresourcegroup \
--location westeurope \
--admin-user mylogin \
--admin-password <server_admin_password> \
--sku-name GP_Gen5_2
```

## <a name="create-the-private-endpoint"></a>创建专用终结点 
为虚拟网络中的 PostgreSQL 服务器创建专用终结点： 

```azurecli-interactive
az network private-endpoint create \  
    --name myPrivateEndpoint \  
    --resource-group myResourceGroup \  
    --vnet-name myVirtualNetwork  \  
    --subnet mySubnet \  
    --private-connection-resource-id $(az resource show -g myResourcegroup -n mydemoserver --resource-type "Microsoft.DBforPostgreSQL/servers" --query "id" -o tsv) \    
    --group-id postgresqlServer \  
    --connection-name myConnection  
 ```

## <a name="configure-the-private-dns-zone"></a>配置专用 DNS 区域 
为 PostgreSQL 服务器域创建专用 DNS 区域，并创建一个与虚拟网络关联的链接。 
```azurecli-interactive
az network private-dns zone create --resource-group myResourceGroup \ 
   --name  "privatelink.postgres.database.azure.com" 
az network private-dns link vnet create --resource-group myResourceGroup \ 
   --zone-name  "privatelink.postgres.database.azure.com"\ 
   --name MyDNSLink \ 
   --virtual-network myVirtualNetwork \ 
   --registration-enabled false 

#Query for the network interface ID  
networkInterfaceId=$(az network private-endpoint show --name myPrivateEndpoint --resource-group myResourceGroup --query 'networkInterfaces[0].id' -o tsv)
 
 
az resource show --ids $networkInterfaceId --api-version 2019-04-01 -o json 
# Copy the content for privateIPAddress and FQDN matching the Azure database for PostgreSQL name 
 
 
#Create DNS records 
az network private-dns record-set a create --name myserver --zone-name privatelink.postgres.database.azure.com --resource-group myResourceGroup  
az network private-dns record-set a add-record --record-set-name myserver --zone-name privatelink.postgres.database.azure.com --resource-group myResourceGroup -a <Private IP Address>
```

> [!NOTE] 
> 客户 DNS 设置中的 FQDN 未解析为已配置的专用 IP。 你必须为已配置的 FQDN 设置一个 DNS 区域，如[此处](../dns/dns-operations-recordsets-portal.md)所示。

> [!NOTE]
> 在某些情况下，Azure Database for PostgreSQL 和 VNet 子网位于不同的订阅中。 在这些情况下，必须确保以下配置：
> - 请确保两个订阅都注册了 Microsoft.DBforPostgreSQL 资源提供程序。 有关详细信息，请参阅[资源管理器注册][resource-manager-portal]

## <a name="connect-to-a-vm-from-the-internet"></a>从 Internet 连接到 VM

从 Internet 连接到 VM *myVm* ，如下所示：

1. 在门户的搜索栏中，输入 *myVm* 。

1. 选择“连接”按钮。 选择“连接”按钮后，“连接到虚拟机”随即打开   。

1. 选择“下载 RDP 文件”。 Azure 会创建远程桌面协议 ( *.rdp* ) 文件，并将其下载到计算机。

1. 打开 downloaded.rdp  文件。

    1. 出现提示时，选择“连接”  。

    1. 输入在创建 VM 时指定的用户名和密码。

        > [!NOTE]
        > 可能需要选择“更多选择” > “使用其他帐户”，以指定在创建 VM 时输入的凭据 。

1. 选择“确定”  。

1. 你可能会在登录过程中收到证书警告。 如果收到证书警告，请选择“确定”或“继续” 。

1. VM 桌面出现后，将其最小化以返回到本地桌面。  

## <a name="access-the-postgresql-server-privately-from-the-vm"></a>从 VM 对 PostgreSQL 服务器进行私密访问

1. 在 *myVM* 的远程桌面中，打开 PowerShell。

2. 输入  `nslookup mydemopostgresserver.privatelink.postgres.database.azure.com`。 

    将收到类似于下面的消息：
    ```azurepowershell
    Server:  UnKnown
    Address:  168.63.129.16
    Non-authoritative answer:
    Name:    mydemopostgresserver.privatelink.postgres.database.azure.com
    Address:  10.1.3.4
    ```

3. 使用任何可用的客户端测试 PostgreSQL 服务器的专用链接连接。 在以下示例中，我使用了 [Azure Data Studio](/sql/azure-data-studio/download?view=sql-server-ver15) 来执行该操作。

4. 在“新建连接”中，输入或选择以下信息：

    | 设置 | Value |
    | ------- | ----- |
    | 服务器类型| 选择“PostgreSQL”。|
    | 服务器名称| 选择 *mydemopostgresserver.privatelink.postgres.database.azure.com* |
    | 用户名 | 以 username@servername 形式输入用户名，这是在 PostgreSQL 服务器创建过程中提供的。 |
    |密码 |输入创建 PostgreSQL 服务器期间提供的密码。 |
    |SSL|选择“必需”。|
    ||

5. 选择“连接”。

6. 浏览左侧菜单中的数据库。

7. （可选）创建或查询 postgreSQL 服务器中的信息。

8. 关闭与 myVm 的远程桌面连接。

## <a name="clean-up-resources"></a>清理资源 
如果不再需要资源组及其所有资源，可以使用 az group delete 将其删除： 

```azurecli-interactive
az group delete --name myResourceGroup --yes 
```

## <a name="next-steps"></a>后续步骤
- 了解[什么是 Azure 专用终结点的](../private-link/private-endpoint-overview.md)详细信息

<!-- Link references, to text, Within this same GitHub repo. -->
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md
