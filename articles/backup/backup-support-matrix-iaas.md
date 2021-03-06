---
title: Azure VM 备份的支持矩阵
description: 提供有关在使用 Azure 备份服务备份 Azure VM 时的支持设置和限制摘要。
ms.topic: conceptual
ms.date: 09/13/2019
ms.custom: references_regions
ms.openlocfilehash: 5988cc7bdc34521bfa75e9f179f88bfbe881b882
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/29/2020
ms.locfileid: "92925639"
---
# <a name="support-matrix-for-azure-vm-backup"></a>Azure VM 备份的支持矩阵

可以使用 [Azure 备份服务](backup-overview.md)备份本地计算机和工作负荷以及 Azure 虚拟机 (VM)。 本文汇总了使用 Azure 备份服务备份 Azure VM 时的支持设置和限制。

其他支持矩阵：

- Azure 备份的[常规支持矩阵](backup-support-matrix.md)
- Azure 备份服务器 / System Center Data Protection Manager (DPM) 备份的[支持矩阵](backup-support-matrix-mabs-dpm.md)
- 使用 Microsoft Azure 恢复服务 (MARS) 代理进行备份的[支持矩阵](backup-support-matrix-mars-agent.md)

## <a name="supported-scenarios"></a>支持的方案

下面描述了使用 Azure 备份服务备份和还原 Azure VM 的方式。

**方案** | **备份** | **代理** |**还原**
--- | --- | --- | ---
直接备份 Azure VM  | 备份整个 VM。  | 无需在 Azure VM 上安装其他代理。 Azure 备份将为 VM 上运行的 [Azure VM 代理](../virtual-machines/extensions/agent-windows.md)安装一个扩展并使用该扩展。 | 按如下所述进行还原：<br/><br/> - **创建基本的 VM** 。 如果 VM 不使用特殊配置（例如多个 IP 地址），则此操作非常有用。<br/><br/> - **还原 VM 磁盘** 。 还原磁盘。 然后将磁盘附加到现有 VM，或使用 PowerShell 从磁盘创建新的 VM。<br/><br/> - **替换 VM 磁盘** 。 如果 VM 存在并使用托管磁盘（未加密），则你可以还原磁盘并使用它来替换 VM 上的现有磁盘。<br/><br/> - **还原特定的文件/文件夹** 。 可以从 VM 还原文件/文件夹，而不是从整个 VM 还原。
直接备份 Azure VM（仅适用于 Windows）  | 备份特定的文件/文件夹/卷。 | 安装 [Azure 恢复服务代理](backup-azure-file-folder-backup-faq.md)。<br/><br/> 可将 MARS 代理与适用于 Azure VM 代理的备份扩展一同运行，以便在文件/文件夹级别备份 VM。 | 还原特定的文件夹/文件。
将 Azure VM 备份到备份服务器  | 将文件/文件夹/卷、系统状态/裸机文件和应用数据备份到 System Center DPM 或 Microsoft Azure 备份服务器 (MABS)。<br/><br/> 然后，DPM/MABS 将备份到备份保管库。 | 在 VM 上安装 DPM/MABS 保护代理。 在 DPM/MABS 上安装 MARS 代理。| 还原文件/文件夹/卷；系统状态/裸机文件；应用数据。

详细了解如何[使用备份服务器](backup-architecture.md#architecture-back-up-to-dpmmabs)进行备份和[支持要求](backup-support-matrix-mabs-dpm.md)。

## <a name="supported-backup-actions"></a>支持的备份操作

**操作** | **支持**
--- | ---
备份已关闭/脱机的 VM | 。<br/><br/> 仅创建崩溃一致性快照，不会创建应用一致性快照。
迁移到托管磁盘后备份磁盘 | 。<br/><br/> 备份将继续工作。 因此不需要执行任何操作。
启用资源组锁定后备份托管磁盘 | 不支持。<br/><br/> Azure 备份无法删除旧的还原点；如果达到了还原点数目的上限，备份将开始失败。
修改 VM 的备份策略 | 。<br/><br/> 将使用新策略中的计划和保留期设置备份 VM。 如果保留期设置已延长，则会标记并保留现有的恢复点。 如果保留期设置已缩短，则会在下一个清理作业中清理现有的恢复点，并最终将其删除。
取消备份作业| 在快照过程中受支持。<br/><br/> 快照正在传输到保管库时不受支持。
将 VM 备份到其他区域或订阅 |不支持。<br><br>若要成功备份，虚拟机必须与用于备份的保管库位于同一订阅中。
每日备份（通过 Azure VM 扩展） | 每日进行一次计划的备份。<br/><br/>Azure 备份服务每天最多支持 9 个按需备份，但 Microsoft 建议每天不超过四个按需备份，以确保获得最佳性能。
每日备份（通过 MARS 代理） | 每日进行三次计划的备份。
每日备份（通过 DPM/MABS） | 每日进行两次计划的备份。
每月/每年备份| 使用 Azure VM 扩展备份时不受支持。 仅支持每日和每周备份。<br/><br/> 可将策略设置为按每月/每年保留期保留每日/每周备份。
自动时钟调整 | 不支持。<br/><br/> 备份 VM 时，Azure 备份不会根据夏令时自动调整。<br/><br/>  请根据需要手动修改策略。
[混合备份的安全功能](./backup-azure-security-feature.md) |不支持禁用安全功能。
备份其计算机时间已发生更改的 VM | 不支持。<br/><br/> 如果为该 VM 启用备份后，计算机时间变为将来的日期时间，那么即使还原时间更改，也无法保证成功备份。
[虚拟机规模集](../virtual-machine-scale-sets/overview.md)中的 Azure VM | [业务流程模式](../virtual-machine-scale-sets/orchestration-modes.md#orchestration-modes)设置为 3 的 VM 支持备份和还原。 <br><br>不支持可用性集。

## <a name="operating-system-support-windows"></a>操作系统支持 (Windows)

下表汇总了备份 Windows Azure VM 时支持的操作系统。

**方案** | **OS 支持**
--- | ---
使用 Azure VM 代理扩展进行备份 | - Windows 10 客户端（仅 64 位） <br/><br/>- Windows Server 2019 (Datacenter/Datacenter Core/Standard) <br/><br/> - Windows Server 2016 (Datacenter/Datacenter Core/Standard) <br/><br/> - Windows Server 2012 R2 (Datacenter/Standard) <br/><br/> - Windows Server 2012 (Datacenter/Standard) <br/><br/> - Windows Server 2008 R2（RTM 和 SP1 Standard）  <br/><br/> - Windows Server 2008（仅限 64 位）
使用 MARS 代理进行备份 | [支持](backup-support-matrix-mars-agent.md#supported-operating-systems)的操作系统。
使用 DPM/MABS 备份 | 使用 [MABS](backup-mabs-protection-matrix.md) 和 [DPM](/system-center/dpm/dpm-protection-matrix) 备份时支持的操作系统。

Azure 备份不支持 32 位操作系统。

## <a name="support-for-linux-backup"></a>Linux 备份支持

下面是在需要备份 Linux 计算机的情况下的支持项目。

**操作** | **支持**
--- | ---
使用 Linux Azure VM 代理备份 Linux Azure VM | 文件一致性备份。<br/><br/> 使用[自定义脚本](backup-azure-linux-app-consistent.md)的应用一致性备份。<br/><br/> 在还原过程中，可以创建新的 VM、还原某个磁盘并使用它来创建 VM，或者还原某个磁盘并使用它来替换现有 VM 上的磁盘。 还可以还原单个文件和文件夹。
使用 MARS 代理备份 Linux Azure VM | 不支持。<br/><br/> MARS 代理只能安装在 Windows 计算机上。
使用 DPM/MABS 备份 Linux Azure VM | 不支持。

## <a name="operating-system-support-linux"></a>操作系统支持 (Linux)

对于 Azure VM Linux 备份，Azure 备份支持 [Azure 认可的 Linux 分发版](../virtual-machines/linux/endorsed-distros.md)列表。 注意以下事项：

- Azure 备份不支持 Core OS Linux。
- Azure 备份不支持 32 位操作系统。
- 只要 VM 上装有[适用于 Linux 的 Azure VM 代理](../virtual-machines/extensions/agent-linux.md)且支持 Python，其他自带 Linux 分发版也能正常运行。
- Azure 备份不支持未安装 Python 版本 2.7 且配置了代理的 Linux VM。

## <a name="backup-frequency-and-retention"></a>备份频率和保留期

**设置** | **限制**
--- | ---
每个受保护实例（计算机/工作负荷）的恢复点数上限 | 9999。
恢复点的最长过期时间 |  (99 年) 无限制。
备份到保管库时的最高备份频率（Azure VM 扩展） | 每天一次。
备份到保管库时的最高备份频率（MARS 代理） | 每日三次备份。
备份到 DPM/MABS 时的最高备份频率 | SQL Server 每隔 15 分钟。<br/><br/> 其他工作负荷每小时一次。
恢复点保留期 | 每日、每周、每月和每年。
最大保留期 | 取决于备份频率。
DPM/MABS 磁盘上的恢复点数 | 文件服务器为 64 个，应用服务器为 448 个。<br/><br/> 对于本地 DPM 来说，磁带恢复点没有限制。

## <a name="supported-restore-methods"></a>支持的还原方法

**还原选项** | **详细信息**
--- | ---
**创建新 VM** | 从还原点快速创建并正常运行一个基本的 VM。<br/><br/> 可以指定 VM 的名称，选择要将此 VM 放入到的资源组和虚拟网络 (VNet)，以及为已还原的 VM 指定存储帐户。 新 VM 必须在源 VM 所在的区域创建。
**还原磁盘** | 还原某个 VM 磁盘，然后使用它来创建新的 VM。<br/><br/> Azure 备份提供一个模板来帮助你自定义和创建 VM。 <br/><br> 还原作业会生成一个模板，可以下载该模板，并使用它来指定自定义的 VM 设置和创建 VM。<br/><br/> 磁盘将复制到指定的资源组。<br/><br/> 或者，可将磁盘附加到现有 VM，或使用 PowerShell 创建新的 VM。<br/><br/> 若要自定义 VM、添加在备份时不存在的配置设置，或添加必须使用模板或 PowerShell 配置的设置，则此选项非常有用。
**替换现有项** | 可以还原某个磁盘，并使用它来替换现有 VM 上的磁盘。<br/><br/> 当前 VM 必须存在。 如果已将其删除，则无法使用此选项。<br/><br/> Azure 备份会在替换磁盘之前获取现有 VM 的快照，并将其存储在指定的暂存位置。 连接到该 VM 的现有磁盘将替换为所选的还原点。<br/><br/> 快照将复制到保管库，并根据保留策略进行保留。 <br/><br/> 替换磁盘操作完成后，原始磁盘会保留在资源组中。 如果不需要原始磁盘，可选择手动删除它们。 <br/><br/>支持替换未加密的托管 VM 的现有磁盘。 不支持对非托管磁盘、[通用化 VM](../virtual-machines/windows/capture-image-resource.md) 或[使用自定义映像创建的](https://azure.microsoft.com/resources/videos/create-a-custom-virtual-machine-image-in-azure-resource-manager-with-powershell/) VM 使用该选项。<br/><br/> 如果还原点中的磁盘数多于或少于当前 VM 中的磁盘数，则还原点中的磁盘数只反映 VM 配置。<br><br> 对于包含链接资源的 VM（例如[用户分配的托管标识](../active-directory/managed-identities-azure-resources/overview.md)和 [Key Vault](../key-vault/general/overview.md)），还支持替换现有磁盘。
**跨区域（次要区域）** | 跨区域还原可用于还原次要区域（[Azure 配对区域](../best-practices-availability-paired-regions.md#what-are-paired-regions)）中的 Azure VM。<br><br> 如果备份在次要区域中完成，则可针对所选恢复点还原所有 Azure VM。<br><br> 此功能适用于以下选项：<br> <li> [创建 VM](./backup-azure-arm-restore-vms.md#create-a-vm) <br> <li> [还原磁盘](./backup-azure-arm-restore-vms.md#restore-disks) <br><br> 当前不支持[替换现有磁盘](./backup-azure-arm-restore-vms.md#replace-existing-disks)选项。<br><br> 权限<br> 次要区域中的还原操作可以由备份管理员和应用管理员执行。

## <a name="support-for-file-level-restore"></a>文件级还原支持

**还原** | **支持**
--- | ---
跨操作系统还原文件 | 可以在与备份的 VM 使用相同（或兼容）OS 的任何计算机上还原文件。 请参阅[兼容 OS 表](backup-azure-restore-files-from-vm.md#system-requirements)。
还原已加密 VM 中的文件 | 不支持。
还原网络受限的存储帐户中的文件 | 不支持。
使用 Windows 存储空间还原 VM 上的文件 | 不支持在同一 VM 上还原。<br/><br/> 应在兼容的 VM 上还原文件。
使用 LVM/RAID 阵列还原 Linux VM 上的文件 | 不支持在同一 VM 上还原。<br/><br/> 在兼容的 VM 上还原。
使用特殊网络设置还原文件 | 不支持在同一 VM 上还原。 <br/><br/> 在兼容的 VM 上还原。

## <a name="support-for-vm-management"></a>VM 管理支持

下表汇总了在执行 VM 管理任务（例如添加或替换 VM 磁盘）期间的备份支持。

**还原** | **支持**
--- | ---
跨订阅/区域/局部区域还原。 | 不支持。
还原到现有 VM | 使用“替换磁盘”选项。
在为存储帐户启用了 Azure 存储服务加密 (SSE) 的情况下还原磁盘 | 不支持。<br/><br/> 还原到未启用 SSE 的帐户。
还原到混合存储帐户 |不支持。<br/><br/> 根据存储帐户类型，所有已还原的磁盘将是高级或标准类型，而不是混合类型。
将 VM 直接还原到可用性集 | 对于托管磁盘，可以还原磁盘，并在模板中使用可用性集选项。<br/><br/> 不支持非托管磁盘。 对于非托管磁盘，可以还原磁盘，然后在可用性集中创建 VM。
升级到托管 VM 后还原非托管 VM 的备份| 。<br/><br/> 可以还原磁盘，然后创建托管 VM。
在将 VM 迁移到托管磁盘之前将 VM 还原到还原点 | 。<br/><br/> 还原到非托管磁盘（默认设置），将已还原的磁盘转换为托管磁盘，然后使用托管磁盘创建 VM。
还原已删除的 VM。 | 。<br/><br/> 可以从恢复点还原 VM。
还原域控制器 VM  | 。 有关详细信息，请参阅 [还原域控制器 vm](backup-azure-arm-restore-vms.md#restore-domain-controller-vms)。
还原不同虚拟网络中的 VM |。<br/><br/> 虚拟网络必须位于同一订阅和区域中。

## <a name="vm-compute-support"></a>VM 计算支持

**计算** | **支持**
--- | ---
VM 大小 |至少有 2 个 CPU 核心和 1-GB RAM 的任意 Azure VM 大小。<br/><br/> [了解详细信息。](../virtual-machines/sizes.md)
备份[可用性集](../virtual-machines/availability.md#availability-sets)中的 VM | 。<br/><br/> 无法使用快速创建 VM 的选项来还原可用性集中的 VM。 还原 VM 时，请还原磁盘并用它来部署 VM，或者还原磁盘并用它来替换现有磁盘。
备份使用[混合使用权益 (HUB)](../virtual-machines/windows/hybrid-use-benefit-licensing.md) 部署的 VM | 。
备份[规模集](../virtual-machine-scale-sets/overview.md)中部署的 VM |。 对于容错域，[业务流程模式](../virtual-machine-scale-sets/orchestration-modes.md)应设置为 2。 不支持可用性集。
备份从 [Azure 市场](https://azuremarketplace.microsoft.com/marketplace/apps?filters=virtual-machine-images)部署的 VM<br/><br/> （由 Microsoft 或第三方发布） |。<br/><br/> VM 必须运行受支持的操作系统。<br/><br/> 恢复 VM 上的文件时，可以仅还原到兼容的 OS（不是早期版本或更高版本的 OS）。 我们不会还原作为 VM 备份的 Azure 市场 VM，因为这需要购买信息。 它们仅作为磁盘还原。
备份从自定义映像部署的 VM（第三方） |。<br/><br/> VM 必须运行受支持的操作系统。<br/><br/> 恢复 VM 上的文件时，可以仅还原到兼容的 OS（不是早期版本或更高版本的 OS）。
备份已迁移到 Azure 的 VM| 。<br/><br/> 若要备份 VM，必须在迁移的计算机上安装 VM 代理。
备份多 VM 一致性 | Azure 备份不提供跨多个 VM 的数据和应用程序一致性。
备份包含[诊断设置](../azure-monitor/platform/platform-logs-overview.md)的 VM  | 不支持。 <br/><br/> 如果使用[新建](backup-azure-arm-restore-vms.md#create-a-vm)选项触发包含诊断设置的 Azure VM 的还原，还原将会失败。
还原区域固定 VM | 支持 (，适用于在2019年1月和 [可用性区域](https://azure.microsoft.com/global-infrastructure/availability-zones/) 可用) 备份的 VM。<br/><br/>目前支持还原到 Vm 中固定的同一区域。 但是，如果该区域不可用，则还原将失败。
Gen2 VM | 支持 <br> Azure 备份支持备份和还原 [Gen2 VM](https://azure.microsoft.com/updates/generation-2-virtual-machines-in-azure-public-preview/)。 从恢复点还原这些 VM 时，它们作为 [Gen2 VM](https://azure.microsoft.com/updates/generation-2-virtual-machines-in-azure-public-preview/) 还原。
带锁的 Azure VM 的备份 | 非托管 VM 不支持。 <br><br> 托管 VM 支持。
[点 VM](../virtual-machines/spot-vms.md) | 不支持。 Azure 备份会将专色 Vm 恢复为常规 Azure Vm。

## <a name="vm-storage-support"></a>VM 存储支持

**组件** | **支持**
--- | ---
Azure VM 数据磁盘 | 支持备份最多包含 32 个磁盘的 Azure VM。<br><br> 备份包含非托管磁盘的 Azure VM 或经典 VM 时，最多支持 16 个磁盘。
数据磁盘大小 | 对于 VM 中的所有磁盘，单个磁盘大小最大为 32 TB，组合磁盘大小最大为 256 TB。
存储类型 | 标准 HDD、标准 SSD、高级 SSD。
托管磁盘 | 。
加密的磁盘 | 。<br/><br/> 可以备份已启用 Azure 磁盘加密的 Azure VM（包含或不包含 Azure AD 应用）。<br/><br/> 无法在文件/文件夹级别恢复已加密的 VM。 必须恢复整个 VM。<br/><br/> 可以在已受 Azure 备份保护的 VM 上启用加密。
已启用写入加速器的磁盘 | 不支持。<br/><br/> Azure 备份会在备份期间自动排除已启用写入加速器 (WA) 的磁盘。 由于未对其进行备份，无法从 VM 的恢复点还原这些磁盘。 <br><br> **重要说明** ：具有 WA 磁盘的虚拟机需要连接 Internet 才能成功进行备份（即使已从备份中排除这些磁盘）。
备份和还原已删除重复数据的 VM/磁盘 | Azure 备份不支持删除重复数据。 有关详细信息，请参阅[此文](./backup-support-matrix.md#disk-deduplication-support) <br/> <br/>  - Azure 备份不会跨恢复服务保管库中的 VM 删除重复数据 <br/> <br/>  - 如果在还原期间有处于重复数据删除状态的 VM，则无法还原文件，因为保管库不能识别该格式。 但是，你能够成功执行完整的 VM 还原。
将磁盘添加到受保护的 VM | 。
调整受保护 VM 上的磁盘大小 | 。
共享存储| 不支持使用群集共享卷 (CSV) 或横向扩展文件服务器备份 VM。 在备份过程中，CSV 写入器可能会失败。 还原时，包含 CSV 卷的磁盘可能不会启动。
[共享磁盘](../virtual-machines/disks-shared-enable.md) | 不支持。
超级 SSD 磁盘 | 不支持。 有关更多详细信息，请参阅这些 [限制](selective-disk-backup-restore.md#limitations)。

## <a name="vm-network-support"></a>VM 网络支持

**组件** | **支持**
--- | ---
网络接口 (NIC) 数 | 特定 Azure VM 大小支持最大数量的 NIC。<br/><br/> NIC 是在还原过程中创建 VM 时创建的。<br/><br/> 已还原 VM 上的 NIC 数目与启用保护时 VM 上的 NIC 数目相同。 启用保护后移除 NIC 不影响计数。
内部/外部负载均衡器 |。 <br/><br/> [详细了解](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations)如何还原使用特殊网络设置的 VM。
多个保留的 IP 地址 |。 <br/><br/> [详细了解](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations)如何还原使用特殊网络设置的 VM。
具有多个网络适配器的 VM| 。 <br/><br/> [详细了解](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations)如何还原使用特殊网络设置的 VM。
具有公共 IP 地址的 VM| 。<br/><br/> 将现有的公共 IP 地址与 NIC 相关联，或者在完成还原后创建一个地址，并将其与 NIC 相关联。
NIC/子网上的网络安全组 (NSG)。 |。
静态 IP 地址 | 不支持。<br/><br/> 将为从还原点创建的新 VM 分配动态 IP 地址。<br/><br/> 对于经典 VM，无法备份具有保留 IP 地址且未定义终结点的 VM。
动态 IP 地址 |。<br/><br/> 如果源 VM 上的 NIC 使用动态 IP 地址，则默认情况下，已还原 VM 上的 NIC 也使用动态 IP 地址。
Azure 流量管理器| 。<br/><br/>如果备份的 VM 位于流量管理器中，请手动将已还原的 VM 添加到同一个流量管理器实例。
Azure DNS |。
自定义 DNS |。
通过 HTTP 代理建立出站连接 | 。<br/><br/> 不支持经过身份验证的代理。
虚拟网络服务终结点| 。<br/><br/> 防火墙和虚拟网络存储帐户设置应允许从所有网络进行访问。

## <a name="vm-security-and-encryption-support"></a>VM 安全与加密支持

Azure 备份支持针对传输中数据和静态数据的加密：

发往 Azure 的网络流量：

- 从服务器到恢复服务保管库的备份流量通过高级加密标准 256 进行加密。
- 备份数据通过安全 HTTPS 链接进行发送。
- 备份数据以加密格式存储在恢复服务保管库中。
- 只有你有解锁此数据的加密密钥。 Microsoft 无法解密任何恢复点的备份数据。

  > [!WARNING]
  > 设置保管库后，只有你才能访问加密密钥。 Microsoft 不保留副本，且没有访问该密钥的权限。 如果客户丢失了密钥，Microsoft 无法恢复备份数据。

数据安全：

- 备份 Azure VM 时，需要在虚拟机内部设置加密。
- Azure 备份支持 Azure 磁盘加密，后者在 Windows 虚拟机上使用 BitLocker，在 Linux 虚拟机上使用 **dm-crypt** 。
- 在后端，Azure 备份使用 [Azure 存储服务加密](../storage/common/storage-service-encryption.md)来保护静态数据。

**计算机** | **传输中** | **静态**
--- | --- | ---
没有 DPM/MABS 的本地 Windows 计算机 | ![是][green] | ![是][green]
Azure VM | ![是][green] | ![是][green]
本地计算机/装有 DPM 的 Azure VM | ![是][green] | ![是][green]
本地计算机/装有 MABS 的 Azure VM | ![是][green] | ![是][green]

## <a name="vm-compression-support"></a>VM 压缩支持

Azure 备份支持对备份流量进行压缩，详细情况汇总在下表中。 注意以下事项：

- 就 Azure VM 来说，VM 扩展会通过存储网络直接读取 Azure 存储帐户中的数据。 无需压缩此流量。
- 如果使用 DPM 或 MABS，则可先压缩数据，然后再将其备份到 DPM/MABS，以节省带宽。

**计算机** | **压缩到 MABS/DPM (TCP)** | **压缩到保管库 (HTTPS)**
--- | --- | ---
没有 DPM/MABS 的本地 Windows 计算机 | 不可用 | ![是][green]
Azure VM | 不可用 | 不可用
本地计算机/装有 DPM 的 Azure VM | ![是][green] | ![是][green]
本地计算机/装有 MABS 的 Azure VM | ![是][green] | ![是][green]

## <a name="next-steps"></a>后续步骤

- [备份 Azure VM](backup-azure-arm-vms-prepare.md)。
- 不使用备份服务器[直接备份 Windows 计算机](tutorial-backup-windows-server-to-azure.md)。
- [设置 MABS](backup-azure-microsoft-azure-backup.md) 以备份到 Azure，然后将工作负荷备份到 MABS。
- [设置 DPM](backup-azure-dpm-introduction.md) 以备份到 Azure，然后将工作负荷备份到 DPM。

[green]: ./media/backup-support-matrix/green.png
[yellow]: ./media/backup-support-matrix/yellow.png
[red]: ./media/backup-support-matrix/red.png
