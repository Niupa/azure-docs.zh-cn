---
title: Azure 混合权益和 Linux Vm
description: Azure 混合权益允许在 Azure 上运行的 Linux 虚拟机上节省资金。
services: virtual-machines
documentationcenter: ''
author: mathapli
manager: westonh
ms.service: virtual-machines-linux
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 09/22/2020
ms.author: mathapli
ms.openlocfilehash: feaa2471f2867257deb06ab32ed5fc0a26a0d37e
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443426"
---
# <a name="public-preview-azure-hybrid-benefit--how-it-applies-for-linux-virtual-machines"></a>公共预览版： Azure 混合权益–适用于 Linux 虚拟机的方式

## <a name="overview"></a>概述

Azure 混合权益使你可以通过使用自己预先存在的 Red Hat 或 SUSE 软件订阅，更轻松地将本地 Red Hat Enterprise Linux (RHEL) 并 SUSE Linux Enterprise Server (的虚拟机) 到 Azure。 利用此权益，你只需为 VM 的基础结构成本付费，因为你的 RHEL 或 SLES 订阅会涵盖软件费用。 该权益适用于所有 RHEL 和 SLES Marketplace 即用即付 (PAYG) 映像。

> [!IMPORTANT]
> 适用于 Linux Vm 的 Azure 混合权益目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="benefit-description"></a>权益说明

通过 Azure 混合权益，你可以更轻松地将本地 RHEL 和 SLES 服务器迁移到 Azure，方法是将 Azure 上的现有 RHEL 和 SLES PAYG Vm 转换为自带订阅 (BYOS) 计费。 通常，从 Azure 上的 PAYG 映像部署的 Vm 将同时收取基础结构费用和软件费用。 使用 Azure 混合权益，可以在不重新部署的情况下将 PAYG Vm 转换为 BYOS 计费模型，以避免任何停机风险。

:::image type="content" source="./media/ahb-linux/azure-hybrid-benefit-cost.png" alt-text="Azure 混合权益 Linux Vm 的成本可视化。":::

启用 RHEL 或 SLES VM 权益后，你将不再需要支付 PAYG VM 上通常产生的额外软件费用。 相反，你的 VM 会开始发出 BYOS 费用，这只包括计算硬件费用和不收取软件费用。

如果需要，还可以将已启用权益的 VM 转换回 PAYG 计费模型。

## <a name="scope-of-azure-hybrid-benefit-eligibility-for-linux-vms"></a>适用于 Linux Vm 的 Azure 混合权益资格范围

Azure 混合权益适用于所有 RHEL 和 SLES Marketplace PAYG 映像。 此权益尚不适用于 RHEL 或 SLES Marketplace BYOS 映像或自定义映像。

如果已在使用 Linux Vm 的权益，则保留实例、专用主机和 SQL 混合权益不符合 Azure 混合权益。

## <a name="how-to-get-started"></a>如何开始使用

Azure 混合权益目前处于 Linux Vm 的预览阶段。 获取预览版的访问权限后，可以使用 Azure CLI 启用此权益。

### <a name="public-preview"></a>公共预览版

Linux) Azure 混合权益 (目前处于公开预览阶段。 你可以使用以下步骤来启用 Red Hat 和 SUSE 分发的权益。 

### <a name="red-hat-customers"></a>Red Hat 客户

1.    注册 [Red Hat 云访问计划](https://aka.ms/rhel-cloud-access)
1.    启用 Azure 订阅 (的云访问) ，并启用包含你要使用其权益的 Vm 的订阅
1.    通过 Azure CLI 将权益应用于现有 Vm
1.    使用单独的更新源注册 Vm，接收权益


### <a name="suse-customers"></a>SUSE 客户

1.    向 SUSE 公有云计划注册
1.    通过 Azure CLI 将权益应用于现有 Vm
1.    使用单独的更新源注册 Vm，接收权益


### <a name="enable-and-disable-the-benefit-in-the-azure-cli"></a>在 Azure CLI 中启用和禁用权益

你可以使用 "az vm update" 命令来更新现有的 Vm。 对于 RHEL Vm，请使用--license 类型参数 "RHEL_BYOS" 运行该命令。 对于 SLES Vm，请使用--license 类型参数 "SLES_BYOS" 运行该命令。

#### <a name="cli-example-to-enable-the-benefit"></a>启用此权益的 CLI 示例：
```azurecli
# This will enable the benefit on a RHEL VM
az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS

# This will enable the benefit on a SLES VM
az vm update -g myResourceGroup -n myVmName --license-type SLES_BYOS
```
#### <a name="cli-example-to-disable-the-benefit"></a>用于禁用权益的 CLI 示例：
若要禁用权益，请使用许可证类型值 "None"
```azurecli
# This will disable the benefit on a VM
az vm update -g myResourceGroup -n myVmName --license-type None
```

#### <a name="cli-example-to-enable-the-benefit-on-a-large-number-of-vms"></a>启用大量 Vm 权益的 CLI 示例
若要在大量 Vm 上启用权益，可以使用 `--ids` Azure CLI 中的参数。

```azurecli
# This will enable the benefit on a RHEL VM. In this example, ids.txt is an
# existing text file containing a delimited list of resource IDs corresponding
# to the VMs using the benefit
az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS --ids $(cat ids.txt)
```

下面的示例演示了两种获取资源 Id 列表的方法-一个在资源组级别（一个在订阅级别）。
```azurecli
# To get a list of all the resource IDs in a resource group:
$(az vm list -g MyResourceGroup --query "[].id" -o tsv)

# To get a list of all the resource IDs of VMs in a subscription:
az vm list -o json | jq '.[] | {VMName: .name, ResourceID: .id}'
```

## <a name="check-ahb-status-of-a-vm"></a>检查 VM 的 AHB 状态
可以通过两种方式查看 VM 的 AHB 状态：使用 Azure CLI 或使用 Azure IMDS)  (azure 实例元数据服务。


### <a name="azure-cli"></a>Azure CLI

`az vm get-instance-view`此命令可用于实现此目的。 在响应中查找 licenseType 字段。 如果 licenseType 字段存在并且值为 "RHEL_BYOS" 或 "SLES_BYOS"，则 VM 启用了权益。

```azurecli
az vm get-instance-view -g MyResourceGroup -n MyVm
```

### <a name="azure-instance-metadata-service"></a>Azure 实例元数据服务

可以从 VM 本身中查询 IMDS 证明元数据，以确定 VM 的 licenseType。 "RHEL_BYOS" 或 "SLES_BYOS" 的 licenseType 值将指示 VM 启用了权益。 [在此处](./instance-metadata-service.md#attested-data)了解有关证明元数据的详细信息

## <a name="compliance"></a>合规性

### <a name="red-hat"></a>Red Hat

若要为 RHEL Vm 使用 Azure 混合权益，必须先向 Red Hat 云访问计划注册。 可以通过此处的 Red Hat 云访问站点来执行此操作。 启用 VM 权益后，必须使用 Red Hat 订阅管理器或 Red Hat 卫星将 VM 注册到自己的更新源。 注册更新将确保仍处于受支持状态。

### <a name="suse"></a>SUSE

若要将 Azure 混合权益用于 SLES Vm，必须先向 SUSE 公有云计划注册。 在此处了解有关程序的详细信息。 购买 SUSE 订阅后，必须使用 SUSE 客户中心、订阅管理工具服务器或 SUSE 管理器将这些订阅的 Vm 注册到自己的更新源。

## <a name="frequently-asked-questions"></a>常见问题
*问：我是否可以将 "RHEL_BYOS" 的许可证类型用于 SLES 映像，反之亦然？*

答：不能。 尝试输入不正确匹配 VM 上运行的发行版的许可证类型将不会更新任何计费元数据。 但是，如果意外输入错误的许可证类型，则再次将 VM 更新为正确的许可证类型将仍会启用此权益。

*问：我已注册 Red Hat 云访问权限，但仍无法启用我的 RHEL Vm 权益。我该怎么办？*

答： Red Hat 云访问订阅注册可能需要一些时间才能从 Red Hat 传播到 Azure。 如果在一个工作日内仍看到此错误，请联系 Microsoft 支持部门。

## <a name="common-issues"></a>常见问题
本部分包含可能遇到的常见问题的列表以及缓解步骤。

| 错误 | 缓解措施 |
| ----- | ---------- |
| "该操作无法完成，因为我们的记录显示你尚未在 Azure 订阅上成功启用 Red Hat 云访问 ..." | 若要将权益与 RHEL Vm 一起使用，必须先将 Azure 订阅注册 () ，并使用 Red Hat 云访问权限。 访问此链接，了解有关如何注册适用于 Red Hat 云访问的 Azure 订阅的详细信息

## <a name="next-steps"></a>后续步骤
* 了解如何创建和更新 Vm，并 (RHEL_BYOS 中添加许可证类型，SLES_BYOS[在此处使用 Azure 混合权益](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest&preserve-view=true)) Azure CLI。
