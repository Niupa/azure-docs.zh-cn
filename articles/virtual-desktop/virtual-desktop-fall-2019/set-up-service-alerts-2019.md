---
title: 为 Windows 虚拟桌面（经典）设置服务警报 - Azure
description: 如何设置 Azure 服务运行状况以接收 Windows 虚拟桌面（经典）的服务通知。
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 3f2171d3271a4ffc4770dd0c9faea16c23e44d02
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "88005496"
---
# <a name="tutorial-set-up-service-alerts-for-windows-virtual-desktop-classic"></a>教程：为 Windows 虚拟桌面（经典）设置服务警报

>[!IMPORTANT]
>本教程的内容适用于 Windows 虚拟桌面（经典），后者不支持 Azure 资源管理器 Windows 虚拟桌面对象。 要尝试管理 Azure 资源管理器 Windows 虚拟桌面对象，请参阅[本文](../set-up-service-alerts.md)。

可以使用 Azure 服务运行状况监视 Windows 虚拟桌面的服务问题和运行状况公告。 Azure 服务运行状况可以通过不同类型的警报（例如，电子邮件或短信）通知你，帮助你了解问题的影响，并在问题解决时保持更新。 此外，Azure 服务运行状况还可以帮助你减少停机时间，并为可能影响资源可用性的计划内维护和更改做好准备。

本教程介绍以下操作：

> [!div class="checklist"]
> * 创建和配置服务警报。

若要详细了解 Azure 服务运行状况，请参阅 [Azure 运行状况文档](https://docs.microsoft.com/azure/service-health/)。

## <a name="prerequisites"></a>先决条件

- [教程：在 Windows 虚拟桌面中创建租户](tenant-setup-azure-active-directory.md)
- [教程：使用 PowerShell 创建服务主体和角色分配](create-service-principal-role-powershell.md)
- [教程：通过 Azure 市场创建主机池](create-host-pools-azure-marketplace-2019.md)

## <a name="create-service-alerts"></a>创建服务警报

本部分介绍如何配置 Azure 服务运行状况以及如何设置通知，你可以在 Azure 门户上访问这些通知。 可以设置不同类型的警报，并安排它们及时通知你。

### <a name="recommended-service-alerts"></a>建议的服务警报

我们建议你为以下运行状况事件类型创建服务警报：

- **服务问题：** 接收有关影响用户与服务的连接或管理 Windows 虚拟桌面租户的能力的主要问题的通知。
- **运行状况公告：** 接收需要你注意的通知。 以下是此类通知的一些示例：
    - 虚拟机 (VM) 没有安全配置为打开端口 3389
    - 弃用的功能

### <a name="configure-service-alerts"></a>配置服务警报

若要配置服务警报，请执行以下操作：

1. 登录 [Azure 门户](https://portal.azure.com/)。
2. 选择“服务运行状况”  。
3. 按照[创建有关服务通知的活动日志警报](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log-service-notifications?toc=%2Fazure%2Fservice-health%2Ftoc.json#alert-and-new-action-group-using-azure-portal)中的说明设置警报和通知。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何设置和使用 Azure 服务运行状况来监视 Windows 虚拟桌面的服务问题和运行状况公告。 若要了解如何登录到 Windows 虚拟桌面，请继续学习“连接到 Windows 虚拟桌面”操作指南。

> [!div class="nextstepaction"]
> [连接到 Windows 7 和 Windows 10 上的远程桌面客户端](connect-windows-7-10-2019.md)
