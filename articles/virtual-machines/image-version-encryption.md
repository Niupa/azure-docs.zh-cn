---
title: 预览版 - 创建使用你自己的密钥加密的映像版本
description: 使用客户管理的加密密钥在共享映像库中创建映像版本。
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/3/2020
ms.author: cynthn
ms.openlocfilehash: e0534fa6eaccbfb9318369e0a4224d84fa8de7c8
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2020
ms.locfileid: "93347703"
---
# <a name="preview-use-customer-managed-keys-for-encrypting-images"></a>预览版：使用客户管理的密钥加密映像

库映像是作为托管磁盘存储的，因此会使用服务器端加密来自动对其进行加密。 服务器端加密使用 256 位 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)，这是可用的最强分组加密技术之一，并且符合 FIPS 140-2 规范。 有关作为 Azure 托管磁盘基础的加密模块的详细信息，请参阅[加密 API：下一代](/windows/desktop/seccng/cng-portal)。

你可以依赖于平台管理的密钥来加密映像、使用你自己的密钥，或者可以同时使用这两个密钥进行双重加密。 如果你选择管理使用自己的密钥进行的加密，可以指定客户管理的密钥，用于加密和解密映像中的所有磁盘。 

通过客户管理的密钥进行的服务器端加密使用 Azure Key Vault。 可将 [RSA 密钥](../key-vault/keys/hsm-protected-keys.md)导入到 Key Vault，或者在 Azure Key Vault 中生成新的 RSA 密钥。

## <a name="prerequisites"></a>先决条件

本文要求在要将映像复制到的每个区域中已设置了磁盘加密。

- 若要仅使用客户托管的密钥，请参阅使用 [Azure 门户](./disks-enable-customer-managed-keys-portal.md)或 [PowerShell](./windows/disks-enable-customer-managed-keys-powershell.md#set-up-your-azure-key-vault-and-diskencryptionset)**启用使用服务器端加密的客户管理** 的密钥。

- 若要同时使用平台管理的密钥和客户托管的密钥 (进行双重加密) ，请参阅使用 [Azure 门户](./disks-enable-double-encryption-at-rest-portal.md)或 [PowerShell](./windows/disks-enable-double-encryption-at-rest-powershell.md)**启用静态加密** 。
    > [!IMPORTANT]
    > 必须使用此链接 [https://aka.ms/diskencryptionupdates](https://aka.ms/diskencryptionupdates) 访问 Azure 门户。 双重加密当前在公共 Azure 门户中不可见，不使用链接。

## <a name="limitations"></a>限制

使用客户管理的密钥加密共享映像库映像时存在几项限制：  

- 加密密钥集必须与映像在同一订阅中。

- 加密密钥集是区域资源，因此每个区域都需要不同的加密密钥集。

- 不能复制或共享使用客户托管密钥的映像。 

- 使用自己的密钥加密磁盘或映像后，不能重新改用平台管理的密钥来加密这些磁盘或映像。


> [!IMPORTANT]
> 使用客户管理的密钥进行的加密目前以公共预览版提供。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。


## <a name="powershell"></a>PowerShell

在公共预览版中，首先需要注册该功能。

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

只需几分钟时间即可完成注册。 使用 Get-AzProviderFeature 检查功能注册的状态。

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

当 RegistrationState 返回 Registered 时，即可转到下一步。

检查提供程序注册。 确保 RegistrationState 返回 `Registered`。

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute | Format-table -Property ResourceTypes,RegistrationState
```

如果未返回 `Registered`，请使用以下命令来注册提供程序：

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

若要指定映像版本的磁盘加密集，请将 [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion) 与 `-TargetRegion` 参数配合使用。 

```azurepowershell-interactive

$sourceId = <ID of the image version source>

$osDiskImageEncryption = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet'}

$dataDiskImageEncryption1 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet1';Lun=1}

$dataDiskImageEncryption2 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet2';Lun=2}

$dataDiskImageEncryptions = @($dataDiskImageEncryption1,$dataDiskImageEncryption2)

$encryption1 = @{OSDiskImage=$osDiskImageEncryption;DataDiskImages=$dataDiskImageEncryptions}

$region1 = @{Name='West US';ReplicaCount=1;StorageAccountType=Standard_LRS;Encryption=$encryption1}

$eastUS2osDiskImageEncryption = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet'}

$eastUS2dataDiskImageEncryption1 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet1';Lun=1}

$eastUS2dataDiskImageEncryption2 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet2';Lun=2}

$eastUS2DataDiskImageEncryptions = @($eastUS2dataDiskImageEncryption1,$eastUS2dataDiskImageEncryption2)

$encryption2 = @{OSDiskImage=$eastUS2osDiskImageEncryption;DataDiskImages=$eastUS2DataDiskImageEncryptions}

$region2 = @{Name='East US 2';ReplicaCount=1;StorageAccountType=Standard_LRS;Encryption=$encryption2}

$targetRegion = @($region1, $region2)


# Create the image
New-AzGalleryImageVersion `
   -ResourceGroupName $rgname `
   -GalleryName $galleryName `
   -GalleryImageDefinitionName $imageDefinitionName `
   -Name $versionName -Location $location `
   -SourceImageId $sourceId `
   -ReplicaCount 2 `
   -StorageAccountType Standard_LRS `
   -PublishingProfileEndOfLifeDate '2020-12-01' `
   -TargetRegion $targetRegion
```

### <a name="create-a-vm"></a>创建 VM

可以从共享映像库创建 VM，并使用客户管理的密钥来加密磁盘。 语法与从映像创建[通用化](vm-generalized-image-version-powershell.md)或[专用化](vm-specialized-image-version-powershell.md) VM 相同；需要使用扩展的参数集并将 `Set-AzVMOSDisk -Name $($vmName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage` 添加到 VM 配置。

对于数据磁盘，使用 [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk) 时需要添加 `-DiskEncryptionSetId $setID` 参数。


## <a name="cli"></a>CLI 

对于公共预览版，首先需要注册该功能。 注册花费大约30分钟。

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name SIGEncryption
```

检查功能注册的状态。

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name SIGEncryption | grep state
```

如果返回了 `"state": "Registered"`，则可转到下一步。

检查注册。

```azurecli-interactive
az provider show -n Microsoft.Compute | grep registrationState
```

如果未显示 registered，请运行以下命令：

```azurecli-interactive
az provider register -n Microsoft.Compute
```


若要指定映像版本的磁盘加密集，请将 [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create) 与 `--target-region-encryption` 参数配合使用。 的格式 `--target-region-encryption` 是以逗号分隔的密钥列表，用于对 OS 和数据磁盘进行加密。 它应如下所示： `<encryption set for the OS disk>,<Lun number of the data disk>,<encryption set for the data disk>,<Lun number for the second data disk>,<encryption set for the second data disk>`。 

如果 OS 磁盘的源是托管磁盘或 VM，请使用 `--managed-image` 指定映像版本的源。 在此示例中，源是一个托管映像，其中包含 OS 磁盘，以及位于 LUN 0 上的数据磁盘。 OS 磁盘将通过 DiskEncryptionSet1 加密，数据磁盘将通过 DiskEncryptionSet2 加密。

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --location westus \
   --target-regions westus=2=standard_lrs eastus2 \
   --target-region-encryption WestUSDiskEncryptionSet1,0,WestUSDiskEncryptionSet2 EastUS2DiskEncryptionSet1,0,EastUS2DiskEncryptionSet2 \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage \
   --managed-image "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
```

如果 OS 磁盘的源是快照，请使用 `--os-snapshot` 指定 OS 磁盘。 如果还应在映像版本中包含数据磁盘快照，请使用 `--data-snapshot-luns` 指定 LUN，并使用 `--data-snapshots` 指定快照，以添加这些快照。

在此示例中，源是磁盘快照。 有一个 OS 磁盘，还有一个位于 LUN 0 上的数据磁盘。 OS 磁盘将通过 DiskEncryptionSet1 加密，数据磁盘将通过 DiskEncryptionSet2 加密。

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --location westus\
   --target-regions westus=2=standard_lrs eastus\
   --target-region-encryption WestUSDiskEncryptionSet1,0,WestUSDiskEncryptionSet2 EastUS2DiskEncryptionSet1,0,EastUS2DiskEncryptionSet2 \
   --os-snapshot "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myOSSnapshot" \
   --data-snapshot-luns 0 \
   --data-snapshots "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myDDSnapshot" \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage 
   
```

### <a name="create-the-vm"></a>创建 VM

可以从共享映像库创建 VM，并使用客户管理的密钥来加密磁盘。 语法与从映像创建[通用化](vm-generalized-image-version-cli.md)或[专用化](vm-specialized-image-version-cli.md) VM 相同，只需添加 `--os-disk-encryption-set` 参数以及加密集的 ID。 对于数据磁盘，请添加 `--data-disk-encryption-sets`，以及数据磁盘的磁盘加密集列表（磁盘加密集以空格分隔）。


## <a name="portal"></a>门户

在门户中创建映像版本时，可以使用 " **加密** " 选项卡输入 "应用存储加密集"。

> [!IMPORTANT]
> 若要使用双加密，必须使用此链接 [https://aka.ms/diskencryptionupdates](https://aka.ms/diskencryptionupdates) 访问 Azure 门户。 双重加密当前在公共 Azure 门户中不可见，不使用链接。


1. 在“创建映像版本”页中，选择“加密”选项卡。 
2. 在 " **加密类型** " 中，选择 "静态加密"，同时选择 "使用 **客户管理的密钥** " 或 " **使用平台管理的密钥和客户托管的密钥进行双重加密** "。 
3. 对于映像中的每个磁盘，从下拉列表中选择要使用的“磁盘加密集”。 

### <a name="create-the-vm"></a>创建 VM

可以从映像版本创建 VM，并使用客户管理的密钥来加密磁盘。 当你在门户中创建 VM 时，请在 " **磁盘** " 选项卡上，选择 "以 **客户管理的密钥加密** " 或 " **通过平台管理的密钥和客户托管的密钥** 加密" 作为 **加密类型** 。 然后可以从下拉列表中选择加密集。

## <a name="next-steps"></a>后续步骤

详细了解[服务器端磁盘加密](./windows/disk-encryption.md)。

有关如何提供购买计划信息的信息，请参阅[在创建映像时提供 Azure 市场购买计划信息](marketplace-images.md)。
