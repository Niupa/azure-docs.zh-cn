---
title: 高可用性、灾难恢复、业务连续性
description: 了解适用于 Azure VM 上的 SQL Server 的高可用性、灾难恢复 (HADR) 和业务连续性选项，例如 AlwaysOn 可用性组、故障转移群集实例、数据库镜像、日志传送以及对 Azure 存储的备份和还原。
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: 53981f7e-8370-4979-b26a-93a5988d905f
ms.service: virtual-machines-sql
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/27/2020
ms.author: mathoma
ms.openlocfilehash: 81d0bddbd62f9f2d15d8404fee63b15c8ab2c0a3
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93102269"
---
# <a name="business-continuity-and-hadr-for-sql-server-on-azure-virtual-machines"></a>适用于 Azure 虚拟机上的 SQL Server 的业务连续性和 HADR
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

业务连续性意味着在发生灾难时继续你的业务、规划恢复以及确保数据高度可用。 Azure 虚拟机上的 SQL Server 可以帮助降低高可用性和灾难恢复 (HADR) 数据库解决方案的成本。 

虚拟机 (VM) 支持大多数 SQL Server HADR 解决方案，这些解决方案既可以仅限于 Azure，也可以是混合解决方案。 在仅包含 Azure 的解决方案中，整个 HADR 系统都在 Azure 中运行。 而在混合配置中，解决方案的一部分在 Azure 中运行，另一部分则在组织的本地运行。 Azure 环境具有灵活性，允许部分或完全迁移至 Azure，以满足 SQL Server 数据库系统对于预算和 HADR 的要求。

本文比较和对比了适用于 Azure VM 上的 SQL Server 的业务连续性解决方案。 

## <a name="overview"></a>概述

你有责任确保数据库系统拥有服务级别协议 (SLA) 要求的 HADR 功能。 Azure 提供了高可用性机制，例如云服务的服务修复和虚拟机的故障恢复检测，但这一事实自身并不保证你能够达到 SLA 的要求。 虽然这些机制可以帮助保护虚拟机的高可用性，但不能保护在 VM 内部运行的 SQL Server 的可用性。 

VM 联机并正常运行时，SQL Server 实例也可能会出故障。 即便是 Azure 提供的高可用性机制，也会在 VM 遇到从软件或硬件故障进行恢复、操作系统升级等事件时，为其留出一定的停机时间。

Azure 中的异地冗余存储 (GRS) 是通过一个名为异地复制的功能实现的。 GRS 可能不是适合你的数据库的灾难恢复解决方案。 因为异地复制异步发送数据，所以最近的更新可能会在灾难中丢失。 有关异地复制限制的详细信息，请阅读[异地复制支持](#geo-replication-support)部分。

## <a name="deployment-architectures"></a>部署体系结构
Azure 支持以下 SQL Server 技术以实现业务连续性：

* [AlwaysOn 可用性组](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server)
* [Always On 故障转移群集实例 (FCIs)](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
* [日志传送](/sql/database-engine/log-shipping/about-log-shipping-sql-server)
* [使用 Azure Blob 存储执行 SQL Server 备份和还原](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service)
* [数据库镜像](/sql/database-engine/database-mirroring/database-mirroring-sql-server) - SQL Server 2016 中已弃用

可将多种技术配合使用，以实现具有高可用性和灾难恢复功能的 SQL Server 解决方案。 根据所用技术的不同，混合部署可能需要使用 VPN 隧道连接 Azure 虚拟网络。 以下部分显示了某些部署体系结构的示例。

## <a name="azure-only-high-availability-solutions"></a>仅限 Azure：高可用性解决方案

可针对在数据库级别具有 Always On 可用性组的 SQL Server 提供高可用性解决方案。 还可以在具有 Always On 故障转移群集实例的实例级别创建高可用性解决方案。 对于其他保护措施，可以通过在故障转移群集实例上创建可用性组，在两个级别上创建冗余。 

| 技术 | 示例体系结构 |
| --- | --- |
| **可用性组** |在同一区域的 Azure VM 中运行的可用性副本提供高可用性。 需要配置域控制器 VM，因为 Windows 故障转移群集需要 Active Directory 域。<br/><br/> 为了实现更高的冗余和可用性，Azure Vm 可以部署在不同的 [可用性区域](../../../availability-zones/az-overview.md) 中，如 [可用性组概述](availability-group-overview.md)中所述。 如果可用性组中的 SQL Server Vm 部署在可用性区域中，请将 [azure 标准负载均衡器](../../../load-balancer/load-balancer-overview.md) 用于侦听器，如 [AZURE SQL VM CLI](./availability-group-az-commandline-configure.md) 和 [azure 快速入门模板](availability-group-quickstart-template-configure.md) 文章中所述。<br/> ![此关系图显示了 "主副本"、"辅助副本" 和 "文件共享见证" 生成的 "WSFC 群集" 上方的 "域控制器"。](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-ha-always-on.png)<br/>有关详细信息，请参阅[在 Azure 中配置可用性组 (GUI)](./availability-group-quickstart-template-configure.md)。 |
| **故障转移群集实例** |SQL Server Vm 支持故障转移群集实例。 由于 FCI 功能需要共享存储，因此，五个解决方案可用于 Azure Vm 上的 SQL Server： <br/><br/> -将 [Azure 共享磁盘](failover-cluster-instance-azure-shared-disks-manually-configure.md) 用于 Windows Server 2019。 共享托管磁盘是一种允许同时将托管磁盘附加到多个虚拟机的 Azure 产品。 群集中的 Vm 可以根据群集应用程序通过 SCSI 永久保留 (SCSI PR) 选择的保留来读取或写入附加的磁盘。 SCSI PR 是一种行业标准存储解决方案，适用于在存储区域网络上运行的应用程序 (SAN) 本地运行。 启用托管磁盘上的 SCSI PR 后，可以按原样将这些应用程序迁移到 Azure。 <br/><br/>-使用[存储空间直通 \( S2D \) ](failover-cluster-instance-storage-spaces-direct-manually-configure.md)为 Windows Server 2016 和更高版本提供基于软件的虚拟 SAN。<br/><br/>-使用适用于 Windows Server 2012 和更高版本的 [高级文件共享](failover-cluster-instance-premium-file-share-manually-configure.md) 。 高级文件共享是 SSD 支持的，具有一致的延迟，并完全支持与 FCI 配合使用。<br/><br/>-使用合作伙伴解决方案支持的用于群集的存储。 有关使用 SIOS DataKeeper 的具体示例，请参阅博客文章 [故障转移群集和 SIOS DataKeeper](https://azure.microsoft.com/blog/high-availability-for-a-file-share-using-wsfc-ilb-and-3rd-party-software-sios-datakeeper/)。<br/><br/>-通过 Azure ExpressRoute 对远程 iSCSI 目标使用共享块存储。 例如，NetApp 专用存储 (NPS) 使用 Equinix 通过 ExpressRoute 向 Azuer VM 公开 iSCSI 目标。<br/><br/>对于 Microsoft 合作伙伴提供的共享存储和数据复制解决方案，如有任何关于在故障转移时访问数据的问题，请联系供应商。<br/><br/>||

## <a name="azure-only-disaster-recovery-solutions"></a>仅限 Azure：灾难恢复解决方案
可将可用性组、数据库镜像或备份和还原与存储 Blob 配合使用，为 Azure 中的 SQL Server 数据库提供灾难恢复解决方案。

| 技术 | 示例体系结构 |
| --- | --- |
| **可用性组** |可用性副本在 Azure VM 中跨多个数据中心运行以实现灾难恢复。 这种跨区域解决方案可以帮助防止站点完全中断。 <br/> ![此图显示了两个区域，其中包含一个 "主副本" 和 "辅助副本"，由 "异步提交" 连接。](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-dr-alwayson.png)<br/>在一个区域内，所有副本应该位于同一云服务和同一虚拟网络中。 由于每个区域会有单独的虚拟网络，因此这些解决方案需要网络间连接。 有关详细信息，请参阅[使用 Azure 门户配置网络间连接](../../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)。 有关详细说明，请参阅[在不同的 Azure 区域中配置 SQL Server Always On 可用性组](availability-group-manually-configure-multiple-regions.md)。|
| **数据库镜像** |主体和镜像以及服务器在不同数据库中运行以实现灾难恢复。 必须使用服务器证书对它们进行部署。 Azure VM 上的 SQL Server 2008 和 SQL Server 2008 R2 都不支持 SQL Server 数据库镜像。 <br/>![显示一个区域中的 "主体" 的关系图，该区域连接到另一区域中的 "具有高性能的镜像"。](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-dr-dbmirroring.png) |
| **使用 Azure Blob 存储进行备份和还原** |生产数据库直接备份到不同数据中心内的 Blob 存储以实现灾难恢复。<br/>![此关系图显示了一个区域中的 "数据库"，备份到另一个区域中的 "Blob 存储"。](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-dr-backup-restore.png)<br/>有关详细信息，请参阅 [Azure VM 中 SQL Server 的备份和还原](../../../azure-sql/virtual-machines/windows/backup-restore.md)。 |
| 使用 Azure Site Recovery 将 SQL Server 复制和故障转移到 Azure |一个 Azure 数据中心的生产 SQL Server 实例直接复制到其他 Azure 数据中心中的 Azure 存储以实现灾难恢复。<br/>![此图显示了一个 Azure 数据中心内的 "数据库"，其中使用 "ASR 复制" 在另一个数据中心进行灾难恢复。 ](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-dr-standalone-sqlserver-asr.png)<br/>有关详细信息，请参阅[使用 SQL Server 灾难恢复和 Azure Site Recovery 来保护 SQL Server](../../../site-recovery/site-recovery-sql.md)。 |


## <a name="hybrid-it-disaster-recovery-solutions"></a>混合 IT：灾难恢复解决方案
可使用可用性组、数据库镜像、日志传送以及备份和还原与 Azure Blob 存储配合使用，在混合 IT 环境中为 SQL Server 数据库提供灾难恢复解决方案。

| 技术 | 示例体系结构 |
| --- | --- |
| **可用性组** |某些可用性副本运行在 Azure VM 中，另一些则在本地运行，以实现跨站点灾难恢复。 生产站点可位于本地，也可以位于 Azure 数据中心。<br/>![可用性组](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-alwayson.png)<br/>由于所有可用性副本必须在同一故障转移群集中，因此该群集必须同时跨越这两个网络（多子网故障转移群集）。 此配置需要在 Azure 与本地网络之间使用 VPN 连接。<br/><br/>为了成功地对数据库进行灾难恢复，还应在灾难恢复站点上安装副本域控制器。|
| **数据库镜像** |一个合作伙伴在 Azure VM 中运行，另一个则在本地运行，以实现使用服务器证书进行跨站点灾难恢复。 合作伙伴不必在同一 Active Directory 域中，并且不需要 VPN 连接。<br/>![数据库镜像](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-dbmirroring.png)<br/>另一种数据库镜像方案是一个合作伙伴在 Azure VM 中运行，另一个则在同一 Active Directory 域中本地运行，以实现跨站点灾难恢复。 需要[在 Azure 虚拟网络与本地网络之间使用 VPN 连接](../../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)。<br/><br/>为了成功地对数据库进行灾难恢复，还应在灾难恢复站点上安装副本域控制器。 Azure VM 上的 SQL Server 2008 和 SQL Server 2008 R2 都不支持 SQL Server 数据库镜像。 |
| **日志传送** |一个服务器在 Azure VM 中运行，另一个则在本地运行，以实现跨站点灾难恢复。 日志传送依赖于 Windows 文件共享，因此需要在 Azure 虚拟网络与本地网络之间使用 VPN 连接。<br/>![日志传送](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-log-shipping.png)<br/>为了成功地对数据库进行灾难恢复，还应在灾难恢复站点上安装副本域控制器。 |
| **使用 Azure Blob 存储进行备份和还原** |本地生产数据库直接备份到 Azure Blob 存储以实现灾难恢复。<br/>![备份和还原](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-backup-restore.png)<br/>有关详细信息，请参阅 [Azure 虚拟机上 SQL Server 的备份和还原](../../../azure-sql/virtual-machines/windows/backup-restore.md)。 |
| 使用 Azure Site Recovery 将 SQL Server 复制和故障转移到 Azure |本地生产 SQL Server 实例直接复制到 Azure 存储以实现灾难恢复。<br/>![使用 Azure Site Recovery 进行复制](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-standalone-sqlserver-asr.png)<br/>有关详细信息，请参阅[使用 SQL Server 灾难恢复和 Azure Site Recovery 来保护 SQL Server](../../../site-recovery/site-recovery-sql.md)。 |


## <a name="free-dr-replica-in-azure"></a>Azure 中的免费 DR 副本

如果具有 [软件保障](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?rtc=1&activetab=software-assurance-default-pivot:primaryr3)，则可以使用 SQL Server 实现混合灾难恢复 (DR) 计划，而不会为被动灾难恢复实例带来额外的许可成本。

在下图中，安装程序使用在 Azure 虚拟机上运行的 SQL Server，该虚拟机使用12核作为使用12个内核的本地 SQL Server 部署的灾难恢复副本。 过去，需要为本地部署和 Azure 虚拟机部署 SQL Server 12 核的。 新权益提供了在 Azure 虚拟机上运行的被动副本权益。 现在，只要满足 Azure 虚拟机上被动副本的灾难恢复条件，就只需为本地运行的 SQL Server 12 个核心提供许可。

![Azure 中的免费灾难恢复副本](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/free-dr-replica-azure.png)

有关详细信息，请参阅[产品许可条款](https://www.microsoft.com/licensing/product-licensing/products)。 

若要启用此权益，请跳到 [SQL Server 虚拟机资源](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource)。 选择 " **设置** " 下的 " **配置** "，然后在 " **SQL Server 许可证** " 下选择 " **灾难恢复** " 选项。 选中该复选框以确认该 SQL Server VM 将用作被动副本，然后选择 " **应用** " 以保存设置。 

![在 Azure 中配置灾难恢复副本](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/dr-replica-in-portal.png)


## <a name="important-considerations-for-sql-server-hadr-in-azure"></a>有关 Azure 中的 SQL Server HADR 的重要注意事项
Azure VM、存储和网络的运行特征与本地非虚拟化的 IT 基础结构不同。 需要了解这些区别并设计可适应这些区别的解决方案，才能成功地在 Azure 中实现 HADR SQL Server 解决方案。

### <a name="high-availability-nodes-in-an-availability-set"></a>可用性集中的高可用性节点
使用 Azure 中的可用性集，可以将高可用性节点放置在单独的容错域和更新域中。 Azure 平台为可用性集中的每个虚拟机分配一个更新域和一个容错域。 数据中心内的这种配置可以确保在发生计划内或计划外维护事件时，至少有一个虚拟机可用，并满足 99.95% 的 Azure SLA 要求。 

若要配置高可用性设置，请将所有参与的 SQL Server 虚拟机放在同一可用性集中，以避免在维护事件期间丢失应用程序或数据。 只有同一云服务中的节点可加入同一可用性集。 有关详细信息，请参阅[管理虚拟机的可用性](../../../virtual-machines/manage-availability.md?toc=%252fazure%252fvirtual-machines%252fwindows%252ftoc.json)。

### <a name="high-availability-nodes-in-an-availability-zone"></a>可用性区域中的高可用性节点
可用性区域是 Azure 区域中独特的物理位置。 每个区域由一个或多个数据中心组成，这些数据中心配置了独立电源以及散热和网络设备。 区域内可用性区域的物理分离有助于保护应用程序和数据免受数据中心故障，方法是确保至少有一个虚拟机可用，并满足99.99% 的 Azure SLA 要求。 

若要配置高可用性，请将参与 SQL Server 虚拟机分散到该区域中的可用性区域。 在可用性区域之间进行网络间传输时，会产生额外的费用。 有关详细信息，请参阅[可用性区域](../../../availability-zones/az-overview.md)。 


### <a name="failover-cluster-behavior-in-azure-networking"></a>故障转移群集在 Azure 网络中的行为
Azure 中不符合 RFC 的 DHCP 服务可能会导致某些故障转移群集配置创建失败。 失败的原因是为群集网络名称分配了重复的 IP 地址，例如与某个群集节点相同的 IP 地址。 当你使用可用性组时，会产生一个问题，因为它依赖于 Windows 故障转移群集功能。

创建两节点群集并使其联机时，请考虑此应用场景：

1. 群集联机，NODE1 随后会为群集网络名称请求一个动态分配的 IP 地址。
2. DHCP 服务除了 NODE1 自身的 IP 地址以外不提供任何 IP 地址，因为 DHCP 服务可以识别请求是否来自 NODE1 自身。
3. Windows 检测到同时向 NODE1 和故障转移群集网络名称分配了一个重复的地址，并且默认群集组未能联机。
4. 默认群集组会移动到 NODE2。 NODE2 将 NODE1 的 IP 地址作为群集 IP 地址，并使默认群集组联机。
5. 当 NODE2 尝试与 NODE1 建立连接时，针对 NODE1 的数据包从不离开 NODE2，因为后者将 NODE1 的 IP 地址解析为其自身。 NODE2 无法与 NODE1 建立连接，它会丢失仲裁并关闭群集。
6. NODE1 可向 NODE2 发送数据包，但 NODE2 无法回复。 NODE1 丢失仲裁并关闭群集。

可将未使用的静态 IP 地址分配给群集网络名称，让群集网络名称联机，从而避免这种情况发生。 例如，可以使用 169.254.1.1 等本地链路 IP 地址。 若要简化此过程，请参阅[在 Azure 中针对可用性组配置 Windows 故障转移群集](https://social.technet.microsoft.com/wiki/contents/articles/14776.configuring-windows-failover-cluster-in-windows-azure-for-alwayson-availability-groups.aspx)。

有关详细信息，请参阅[在 Azure 中配置可用性组 (GUI)](./availability-group-quickstart-template-configure.md)。

### <a name="support-for-availability-group-listeners"></a>支持可用性组侦听器
运行 Windows Server 2012 及更高版本的 Azure VM 支持可用性组侦听器。 这种支持的实现，是借助于在 Azure VM 上启用的负载均衡终结点，它们都是可用性组节点。 必须执行特殊的配置步骤，才能让侦听器对在 Azure 中运行和本地运行的客户端应用程序都有效。

设置侦听器时有两个主要选项：“外部(公共)”或“内部”。 外部（公共）侦听器使用面向 Internet 的负载均衡器，并与可通过 Internet 访问的公共虚拟 IP 相关联。 内部侦听器使用内部负载均衡器，仅支持同一虚拟网络内的客户端。 对于任一负载均衡器类型，都必须启用直接服务器返回。 

如果可用性组跨多个 Azure 子网（例如，跨 Azure 区域的部署），则客户端连接字符串必须包含 `MultisubnetFailover=True`。 这会导致与不同子网中的副本建立并行连接。 有关设置侦听器的说明，请参阅[为 Azure 中的可用性组配置 ILB 侦听器](availability-group-listener-powershell-configure.md)。


仍可通过直接连接到服务实例，单独连接到每个可用性副本。 此外，由于可用性组与数据库镜像客户端向后兼容，因此可以像数据库镜像伙伴一样连接到可用性副本，只要这些副本配置得类似于数据库镜像即可：

* 有一个主要副本和一个次要副本。
* 将辅助副本配置为不可读（“可读辅助副本”选项设置为“否”） 。

下面是一个示例客户端连接字符串，它对应于这个使用 ADO.NET 或 SQL Server Native Client 的类似于数据库镜像的配置：

```console
Data Source=ReplicaServer1;Failover Partner=ReplicaServer2;Initial Catalog=AvailabilityDatabase;
```

有关客户端连接的详细信息，请参阅：

* [将连接字符串关键字用于 SQL Server 本机客户端](/sql/relational-databases/native-client/applications/using-connection-string-keywords-with-sql-server-native-client)
* [将客户端连接到数据库镜像会话 (SQL Server)](/sql/database-engine/database-mirroring/connect-clients-to-a-database-mirroring-session-sql-server)
* [在混合 IT 环境中连接到可用性组侦听器](/archive/blogs/sqlalwayson/connecting-to-availability-group-listener-in-hybrid-it)
* [可用性组侦听器、客户端连接和应用程序故障转移 (SQL Server)](/sql/database-engine/availability-groups/windows/listeners-client-connectivity-application-failover)
* [将数据库镜像连接字符串用于可用性组](/sql/database-engine/availability-groups/windows/listeners-client-connectivity-application-failover)

### <a name="network-latency-in-hybrid-it"></a>混合 IT 环境中的网络延迟
在部署 HADR 解决方案时，请假设本地网络和 Azure 之间有时可能会存在很高的网络延迟。 将副本部署到 Azure 时，同步模式应使用异步提交而非同步提交。 当你同时在本地和 Azure 中部署数据库镜像服务器时，请使用高性能模式，而非高安全模式。

### <a name="geo-replication-support"></a>异地复制支持
Azure 磁盘中的异地复制不支持将同一数据库的数据文件和日志文件各自存储在不同的磁盘上。 GRS 独立并异步地复制对每个磁盘的更改。 此机制可保证单个磁盘中异地复制副本上的写顺序，但不能保证多个磁盘的异地复制副本上的写顺序。 如果将数据库配置为将其数据文件和日志文件各自存储在不同的磁盘上，则灾难后恢复的磁盘所含的数据文件副本可能比日志文件副本新，而这违反了 SQL Server 中的预写日志以及事务的 ACID 属性（原子性、一致性、隔离性和持久性）。 

如果无法对存储帐户禁用异地复制，请将数据库的所有数据和日志文件都保留在同一磁盘上。 如果因数据库较大而必须使用多个磁盘，请部署之前列出的某个灾难恢复解决方案以确保数据冗余。

## <a name="next-steps"></a>后续步骤

确定 [可用性组](availability-group-overview.md) 或 [故障转移群集实例](failover-cluster-instance-overview.md) 是否为业务的最佳业务连续性解决方案。 然后查看配置环境以实现高可用性和灾难恢复的 [最佳实践](hadr-cluster-best-practices.md) 。