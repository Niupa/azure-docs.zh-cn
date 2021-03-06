---
title: Azure Cosmos DB 中的联机备份和按需数据还原
description: 本文介绍了自动备份、按需数据还原的工作原理、如何配置备份间隔和保留，以及如何在 Azure Cosmos DB 中联系数据还原支持。
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/13/2020
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 43625a80df76ff35b8bb1804df5f5fd1524326c5
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097526"
---
# <a name="online-backup-and-on-demand-data-restore-in-azure-cosmos-db"></a>Azure Cosmos DB 中的联机备份和按需数据还原
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB 会定期自动备份数据。 自动备份不会影响数据库操作的性能或可用性。 所有备份均单独存储在一个存储服务中，并会进行全局复制，以便在发生区域性灾难时可以复原。 如果意外删除或更新了 Azure Cosmos 帐户、数据库或容器，而稍后需要恢复数据，那么在这种情况下自动备份非常有用。

## <a name="automatic-and-online-backups"></a>自动联机备份

使用 Azure Cosmos DB，数据和数据备份都高度冗余，并且具有可复原性，能抵御区域性灾难。 以下步骤演示 Azure Cosmos DB 如何执行数据备份：

* Azure Cosmos DB 每 4 小时自动完整备份数据库一次，默认情况下在任何时间点都只存储最新的 2 个备份。 如果默认间隔不足以满足工作负荷，则可以从 Azure 门户更改备份间隔和保持期。 可以在创建 Azure Cosmos 帐户期间或之后更改备份配置。 如果删除了容器或数据库，Azure Cosmos DB 会将给定容器或数据库中的现有快照保留 30 天。

* Azure Cosmos DB 将这些备份存储在 Azure Blob 存储中，而实际数据以本地形式驻留在 Azure Cosmos DB 中。

* 为了保证低延迟，你的备份快照存储在与当前写入区域相同的区域中的 Azure Blob 存储中 (或 **其中一个** 写入区域，以防具有多区域写入配置) 。 对于针对区域性灾难的恢复，Azure Blob 存储中备份数据的屏幕快照将通过异地冗余存储 (GRS) 再次复制到另一区域。 根据源区域以及与源区域关联的区域对确定备份复制的目标区域。 若要了解详细信息，请参阅 [Azure 区域的异地冗余对列表](../best-practices-availability-paired-regions.md)一文。 不能直接访问此数据库。 当你通过支持请求来请求还原后，Azure Cosmos DB 团队将还原你的备份。

   下图显示了如何在美国西部的远程 Azure Blob 存储帐户中备份 Azure Cosmos 容器（其三个主要物理分区全部位于美国西部），然后将其复制到美国东部：

  :::image type="content" source="./media/online-backup-and-restore/automatic-backup.png" alt-text="GRS Azure 存储中所有 Cosmos DB 实体的定期完整备份" border="false":::

* 备份不会影响应用程序的性能或可用性。 Azure Cosmos DB 在后台执行数据备份，不会消耗任何其他预配吞吐量 (RU)，也不会影响数据库的性能和可用性。

## <a name="modify-the-backup-interval-and-retention-period"></a><a id="configure-backup-interval-retention"></a>修改备份时间间隔和保持期

Azure Cosmos DB 每 4 小时自动对数据库执行一次完整备份，而且在任何时候都只存储最新的 2 个备份。 此配置是默认选项，提供它时不需要任何额外的费用。 你可以在创建 Azure Cosmos 帐户期间或创建帐户之后更改默认备份时间间隔和保持期。 备份配置是在 Azure Cosmos 帐户级别设置的，需要在每个帐户上配置。 为帐户配置备份选项后，该帐户将应用到该帐户中的所有容器。 目前，你只能从 Azure 门户更改它们的备份选项。

如果你意外删除或损坏了数据， **则在创建支持请求以还原数据之前，请确保将你的帐户的备份保留期至少增加了7天。最好在此事件的8小时内提高你的保留期。** 这样，Azure Cosmos DB 团队才有足够的时间来还原你的帐户。

使用以下步骤可为现有 Azure Cosmos 帐户更改默认备份选项：

1. 登录到 [Azure 门户](https://portal.azure.com/)。
1. 导航到你的 Azure Cosmos 帐户，打开“备份和还原”窗格。 根据需要更新备份间隔和备份保持期。

   * **备份间隔** -Azure Cosmos DB 尝试对数据进行备份的时间间隔。 备份需要一定的时间，在某些情况下，可能由于下游依赖项而失败。 Azure Cosmos DB 尝试在配置的时间间隔内进行备份，但它不能保证备份在该时间间隔内完成。 可以按小时或分钟配置此值。 备份间隔不能小于 1 小时且不能超过 24 小时。 更改此间隔后，新间隔将从进行上次备份时开始生效。

   * 备份保持期 - 它表示保留每个备份的时间段。 可以按小时或天配置它。 最小保持期不能小于备份间隔 (的两倍) 小时，并且不能超过720小时。

   * 保留的数据副本 - 默认情况下，会免费提供数据的两个备份副本。 如果需要两个以上的副本，则需支付额外费用。 请参阅[定价页](https://azure.microsoft.com/pricing/details/cosmos-db/)中的“已用存储”部分，了解额外副本的确切价格。

   :::image type="content" source="./media/online-backup-and-restore/configure-backup-interval-retention.png" alt-text="GRS Azure 存储中所有 Cosmos DB 实体的定期完整备份" border="true":::

如果在帐户创建过程中配置备份选项，则可以配置“备份策略”（是“定期”或“连续”）  。 定期策略使你可以配置备份间隔和备份保留期。 连续策略目前仅通过注册提供。 Azure Cosmos DB 团队会评估你的工作负载并审批你的请求。

:::image type="content" source="./media/online-backup-and-restore/configure-periodic-continuous-backup-policy.png" alt-text="GRS Azure 存储中所有 Cosmos DB 实体的定期完整备份" border="true":::

## <a name="request-data-restore-from-a-backup"></a>请求从备份还原数据

如果意外删除了数据库或容器，可以[提交支持工单](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)或[联系 Azure 支持](https://azure.microsoft.com/support/options/)，以便从自动联机备份中还原数据。 Azure 支持仅适用于选定的计划，例如“标准”、“开发人员”以及更高级别的计划 。 不适用于基本计划  。 若要了解不同的支持方案，请参阅 [Azure 支持计划](https://azure.microsoft.com/support/plans/)页。

若要还原备份的特定快照，Azure Cosmos DB 要求在该快照的备份周期的持续时间内可用。
在请求还原之前，应该了解以下详细信息：

* 准备好订阅 ID。

* 应根据数据被意外删除或修改的方式，准备好提供其他信息。 建议提前准备可用的信息，从而尽量减少可能在某些有时效的情况下造成不良影响的来回传输。

* 如果删除了整个 Azure Cosmos DB 帐户，则需要提供删除的帐户的名称。 如果创建了同名的另一个帐户，请与支持团队共享该帐户，因为这有助于确定要选择的正确帐户。 建议为删除的每个帐户提交不同的支持工单，因为这可以最大限度地减少还原状态的混乱。

* 如果删除了一个或多个数据库，应提供 Azure Cosmos 帐户及 Azure Cosmos 数据库名，并指定是否存在同名的新数据库。

* 如果删除了一个或多个容器，应提供 Azure Cosmos 帐户名、数据库名和容器名。 并指定是否存在同名容器。

* 如果意外删除或损坏了数据，则应在 8 小时内联系 [Azure 支持](https://azure.microsoft.com/support/options/)，以便 Azure Cosmos DB 团队帮助你从备份中还原数据。 **在创建用于还原数据的支持请求之前，请确保至少将帐户 [的备份保留期增加](#configure-backup-interval-retention) 到7天。最好在此事件的8小时内提高你的保留期。** 这样，Azure Cosmos DB 支持团队才有足够的时间来还原你的帐户。

除了 Azure Cosmos 帐户名、数据库名、容器名以外，还应指定数据可以还原到的时间点。 务必尽量精确，因为这有助于我们确定当时可用的最佳备份。 **指定 UTC 时间也很重要。**

下面的屏幕截图说明如何为容器（集合/图/表）创建通过 Azure 门户还原数据的支持请求。 提供其他详细信息（例如数据类型、还原目的、删除数据的时间），以帮助我们设置请求的优先级。

:::image type="content" source="./media/online-backup-and-restore/backup-support-request-portal.png" alt-text="GRS Azure 存储中所有 Cosmos DB 实体的定期完整备份":::

## <a name="considerations-for-restoring-the-data-from-a-backup"></a>从备份还原数据时的注意事项

在以下情况之一中，可能会意外删除或修改数据：  

* 删除整个 Azure Cosmos 帐户。

* 删除一个或多个 Azure Cosmos 数据库。

* 删除一个或多个 Azure Cosmos 容器。

* 删除或修改容器中的 Azure Cosmos 项（例如文档）。 此特定情况通常称为"数据损坏"。

* 删除或损坏共享产品数据库中的共享产品数据库或容器。

在上述所有情况中，Azure Cosmos DB 均可还原数据。 进行还原时，会创建一个新的 Azure Cosmos 帐户来保存还原数据。 如果未指定新帐户的名称，则其名称将采用 `<Azure_Cosmos_account_original_name>-restored1` 格式。 如果尝试多次还原，则最后一位数将递增。 不能将数据还原到预先创建的 Azure Cosmos 帐户中。

意外删除 Azure Cosmos 帐户后，如果该帐户名未被使用，则可将数据还原到相同名称的帐户中。 因此，建议不要在删除帐户后重新创建它。 因为这样不仅会阻止还原数据使用相同的名称，还会难以确定正确还原帐户。

意外删除 Azure Cosmos 数据库后，我们可以还原整个数据库或该数据库中的容器子集。 还可以跨数据库选择特定容器并将它们还原到新的 Azure Cosmos 帐户中。

如果意外删除或修改了容器中的一个或多个项（即数据损坏情况），需要指定还原到的时间。 如果数据损坏，则时间很重要。 由于容器是实时的，所以备份仍在运行，因此如果超过了保持期（默认值为 8 小时），备份将被覆盖。 若要防止备份被覆盖，请将帐户的备份保留至少增加7天。 最好在8小时内提高数据损坏的保留期。

如果意外删除或损坏了数据，则应在 8 小时内联系 [Azure 支持](https://azure.microsoft.com/support/options/)，以便 Azure Cosmos DB 团队帮助你从备份中还原数据。 这样，Azure Cosmos DB 支持团队才有足够的时间来还原你的帐户。

> [!NOTE]
> 还原数据后，并非所有源功能或设置都会传输到还原的帐户。 以下设置不会转移到新帐户：
> * VNET 访问控制列表
> * 存储过程、触发器和用户定义的函数
> * 多区域设置  

如果在数据库级别预配吞吐量，那么在这种情况下，将对整个数据库，而不是单个容器进行备份和还原。 在这种情况下，无法选择还原容器子集。

## <a name="options-to-manage-your-own-backups"></a>管理自己的备份的选项

通过 Azure Cosmos DB SQL API 帐户，还可采用以下某一方法维护备份：

* 使用 [Azure 数据工厂](../data-factory/connector-azure-cosmos-db.md)定期将数据移至所选的存储。

* 使用 Azure Cosmos DB [更改源](change-feed.md)定期读取数据来进行完整备份或增量更改，并将其存储在自己的存储中。

## <a name="post-restore-actions"></a>还原后的操作

数据还原的主要目标是恢复意外删除或修改的数据。 因此，建议先检查已还原数据，确保其中包含所需内容。 如果一切正常，您可以将数据迁移回主帐户。 尽管可以使用已还原帐户作为新的活动帐户，但如果有生产工作负载则不建议这样做。 

还原数据后，你会收到有关新帐户名（通常采用 `<original-name>-restored1` 格式）和帐户要还原到的时间的通知。 还原的帐户与原始帐户具有相同的预配吞吐量、索引策略，并且二者位于同一区域。 角色为订阅管理员或共同管理员的用户可以看到还原的帐户。

### <a name="migrate-data-to-the-original-account"></a>将数据迁移到原始帐户

以下是将数据迁移回原始帐户的不同方法：

* 使用 [Azure Cosmos DB 数据迁移工具](import-data.md)。
* 使用 [Azure 数据工厂](../data-factory/connector-azure-cosmos-db.md)。
* 使用 Azure Cosmos DB 中的[更改源](change-feed.md)。
* 可以编写自己的自定义代码。

建议在迁移数据之后立即删除容器或数据库。 如果不删除已还原的数据库或容器，它们将在请求单位、存储和流出量方面产生成本。

## <a name="next-steps"></a>后续步骤

接下来可了解如何从 Azure Cosmos 帐户还原数据，或了解如何将数据迁移到 Azure Cosmos 帐户

* 若要提出还原请求，请联系 Azure 支持，并[从 Azure 门户提交票证](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)
* [使用 Cosmos DB 更改源](change-feed.md)将数据移动到 Azure Cosmos DB。
* [使用 Azure 数据工厂](../data-factory/connector-azure-cosmos-db.md)将数据移动到 Azure Cosmos DB。

