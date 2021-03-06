---
title: Azure AD Connect Health 代理安装 | Microsoft 文档
description: 本页与 Azure AD Connect Health 相关，介绍如何安装用于 AD FS 和同步的代理。
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: daveba
editor: curtand
ms.assetid: 1cc8ae90-607d-4925-9c30-6770a4bd1b4e
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/20/2020
ms.topic: how-to
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: da3ae5e86833eb3e7eb71d7e47cb6f963d37b9cf
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410719"
---
# <a name="azure-ad-connect-health-agent-installation"></a>Azure AD Connect Health 代理安装

本文档逐步讲解如何安装和配置 Azure AD Connect Health 代理。 可以从 [此处](how-to-connect-install-roadmap.md#download-and-install-azure-ad-connect-health-agent)下载代理。

## <a name="requirements"></a>要求

下表是使用 Azure AD Connect Health 的要求列表。

| 要求 | 说明 |
| --- | --- |
| Azure AD Premium |Azure AD Connect Health 是 Azure AD Premium 的一个功能，它需要与 Azure AD Premium 配合使用。 <br /><br />有关详细信息，请参阅[Azure AD Premium](../fundamentals/active-directory-get-started-premium.md)入门 <br />若要免费试用 30 天，请参阅 [开始试用](https://azure.microsoft.com/trial/get-started-active-directory/)。 |
| 必须是 Azure AD 的全局管理员才能开始使用 Azure AD Connect Health |默认情况下，只有全局管理员才能安装和配置运行状况代理、访问信息，以及在 Azure AD Connect Health 中执行任何操作。 有关详细信息，请参阅 [Administering your Azure AD directory](../fundamentals/active-directory-whatis.md)（管理 Azure AD 目录）。 <br /><br /> 使用 azure RBAC)  (azure 基于角色的访问控制，你可以允许 Azure AD Connect Health 到组织中的其他用户的访问。 有关详细信息，请参阅 [适用于 Azure AD Connect Health 的 AZURE RBAC) 的 azure 基于角色的访问控制 (。](how-to-connect-health-operations.md#manage-access-with-azure-rbac) <br /><br />**重要说明：** 在安装代理时使用的帐户必须是工作帐户或学校帐户， 而不能是 Microsoft 帐户。 有关详细信息，请参阅 [Sign up for Azure as an organization](../fundamentals/sign-up-organization.md)（以组织身份注册 Azure） |
| Azure AD Connect Health 代理已安装在每台目标服务器上 | Azure AD Connect Health 要求在目标服务器上安装和配置 Health 代理，以便接收数据并提供监视和分析功能。 <br /><br />例如，要从 AD FS 基础结构获取数据，必须将代理安装在 AD FS 和 Web 应用程序代理服务器上。 同样，要获取 AD DS 本地基础结构的相关数据，必须将代理安装在域控制器上。 <br /><br /> |
| Azure 服务终结点的出站连接 | 在安装期间和运行时，代理需要连接到 Azure AD Connect Health 服务终结点。 如果使用防火墙阻止出站连接，请确保在允许列表中添加以下终结点。 请参阅[出站连接终结点](how-to-connect-health-agent-install.md#outbound-connectivity-to-the-azure-service-endpoints) |
|基于 IP 地址的出站连接 | 若要了解如何基于 IP 地址在防火墙上进行筛选，请参阅 [Azure IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653)（Azure IP 范围）。|
| 筛选或禁用出站流量的 TLS 检查 | 如果网络层上的出站流量存在 TLS 检查或终止，则代理注册步骤或数据上传操作可能会失败。 阅读有关[如何设置 TLS 检查的](/previous-versions/tn-archive/ee796230(v=technet.10))详细信息 |
| 运行代理的服务器上的防火墙端口 |为了使代理能够与 Azure AD Health 服务终结点通信，代理要求打开以下防火墙端口。<br /><br /><li>TCP 端口 443</li><li>TCP 端口 5671</li> <br />请注意，代理的最新版本不再需要端口 5671。 升级到最新版本，此时仅需要端口 443。 了解有关[启用防火墙端口](/previous-versions/sql/sql-server-2008/ms345310(v=sql.100))的更多信息 |
| 如果启用了 IE 增强安全性，请允许以下网站 |如果在要安装代理的服务器上启用了“IE 增强的安全性”，则必须允许访问以下网站。<br /><br /><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com</li><li>https:\//login.windows.net</li><li>https： \/ /aadcdn.msftauth.net</li><li>Azure Active Directory 信任的组织联合服务器。 例如：https:\//sts.contoso.com</li> 阅读有关 [如何配置 IE](https://support.microsoft.com/help/815141/internet-explorer-enhanced-security-configuration-changes-the-browsing)的详细信息。 如果网络中有一个代理，请参阅下面的说明。|
| 确保已安装 PowerShell v4.0 或更高版本 | <li>Windows Server 2012 附带 PowerShell v3.0，后者对代理来说不够用。</li><li>Windows Server 2012 R2 和更高版本附带足够使用的 PowerShell 最新版本。</li>|
|禁用 FIPS|Azure AD Connect Health 代理不支持 FIPS。|

> [!IMPORTANT]
> 不支持在 Windows Server Core 上安装 Azure AD Connect Health 代理。


> [!NOTE]
> 如果有高度锁定的环境，则需要添加下面的服务终结点列表中提到的 Url，以及上述 "允许的 IE 增强的安全配置" 中列出的 Url。 


### <a name="outbound-connectivity-to-the-azure-service-endpoints"></a>Azure 服务终结点的出站连接

 在安装期间和运行时，代理需要连接到 Azure AD Connect Health 服务终结点。 如果使用防火墙阻止出站连接，请确保默认情况下不会阻止以下 URL。 请勿禁用对这些 URL 的安全监视或检查，但就像允许其他 Internet 流量一样允许这些 URL。 它们允许与 Azure AD Connect Health 服务终结点通信。 了解如何 [通过 test-azureadconnecthealthconnectivity 检查出站连接](#test-connectivity-to-azure-ad-connect-health-service)。

| 域环境 | 所需的 Azure 服务终结点 |
| --- | --- |
| 一般公众 | <li>&#42;.blob.core.windows.net </li><li>&#42;.aadconnecthealth.azure.com </li><li>在最新版本的代理中不需要 &#42; servicebus.windows.net-端口： 5671 () </li><li>&#42;.adhybridhealth.azure.com/</li><li>https:\//management.azure.com </li><li>https:\//policykeyservice.dc.ad.msft.net/</li><li>https:\//login.windows.net</li><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com *此终结点仅用于在注册过程中进行相关发现。</li> |
| Azure 德国 | <li>&#42;.blob.core.cloudapi.de </li><li>&#42;.servicebus.cloudapi.de </li> <li>&#42;.aadconnecthealth.microsoftazure.de </li><li>https:\//management.microsoftazure.de </li><li>https:\//policykeyservice.aadcdi.microsoftazure.de </li><li>https:\//login.microsoftonline.de </li><li>https:\//secure.aadcdn.microsoftonline-p.de </li><li>https:\//www.office.de *此终结点仅用于在注册过程中进行相关发现。</li> |
| Azure Government | <li>&#42;.blob.core.usgovcloudapi.net </li> <li>&#42;.servicebus.usgovcloudapi.net </li> <li>&#42;.aadconnecthealth.microsoftazure.us </li> <li>https:\//management.usgovcloudapi.net </li><li>https:\//policykeyservice.aadcdi.azure.us </li><li>https:\//login.microsoftonline.us </li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com *此终结点仅用于在注册过程中进行相关发现。</li> |


## <a name="download-and-install-the-azure-ad-connect-health-agent"></a>下载并安装 Azure AD Connect Health 代理

* 请务必[满足 Azure AD Connect Health 的要求](how-to-connect-health-agent-install.md#requirements)。
* 适用于 AD FS 的 Azure AD Connect Health 使用入门
    * [下载适用于 AD FS 的 Azure AD Connect Health 代理。](https://go.microsoft.com/fwlink/?LinkID=518973)
    * [请参阅安装说明](#installing-the-azure-ad-connect-health-agent-for-ad-fs)。
* 适用于同步的 Azure AD Connect Health 使用入门
    * [下载并安装最新版 Azure AD Connect](https://go.microsoft.com/fwlink/?linkid=615771)。 在安装 Azure AD Connect 的过程中，会安装适用于同步的 Health 代理（1.0.9125.0 或更高版本）。
* 适用于 AD DS 的 Azure AD Connect Health 使用入门
    * [下载 AD DS Azure AD Connect Health 代理](https://go.microsoft.com/fwlink/?LinkID=820540)。
    * [请参阅安装说明](#installing-the-azure-ad-connect-health-agent-for-ad-ds)。

## <a name="installing-the-azure-ad-connect-health-agent-for-ad-fs"></a>安装适用于 AD FS 的 Azure AD Connect Health 代理

> [!NOTE]
> AD FS 服务器应与同步服务器不同。 不要将 AD FS 代理安装到同步服务器。
>

在安装之前，请确保 AD FS 服务器主机名是唯一的，并且未存在于 AD FS 服务中。
若要启动代理安装，请双击下载的 .exe 文件。 在第一个屏幕上，单击“安装”。

![Azure AD Connect Health AD FS 安装开始](./media/how-to-connect-health-agent-install/install1.png)

安装完成后，单击“立即配置”。

![Azure AD Connect Health AD FS 安装完成](./media/how-to-connect-health-agent-install/install2.png)

此时会启动 PowerShell 窗口，从而启动代理注册过程。 出现提示时，请使用 Azure AD 帐户登录，该帐户具有执行代理注册所需的访问权限。 默认情况下，全局管理员帐户具有访问权限。

![Azure AD Connect Health AD FS 配置登录](./media/how-to-connect-health-agent-install/install3.png)

登录后，将继续 PowerShell。 完成后，可以关闭 PowerShell，因为配置已完成。

此时会自动启动代理服务，以便代理以安全方式将所需数据上传到云服务。

如果不满足前面部分中所述的所有先决条件，PowerShell 窗口中会出现警告。 请务必在安装代理之前满足[要求](how-to-connect-health-agent-install.md#requirements)。 以下屏幕截图是这些错误的一个示例。

![Azure AD Connect Health AD FS 配置脚本](./media/how-to-connect-health-agent-install/install4.png)

若要验证代理是否已安装，请在服务器上查看以下服务。 如果已完成配置，这些服务应已运行。 否则，它们会停止，直到完成配置。

* Azure AD Connect Health AD FS Diagnostics 服务
* Azure AD Connect Health AD FS Insights 服务
* Azure AD Connect Health AD FS Monitoring 服务

![Azure AD Connect Health AD FS 服务](./media/how-to-connect-health-agent-install/install5.png)


### <a name="enable-auditing-for-ad-fs"></a>为 AD FS 启用审核

> [!NOTE]
> 本部分仅适用于 AD FS 服务器。 不需对 Web 应用程序代理服务器执行以下步骤。
>

若要通过使用情况分析功能收集和分析数据，必须向 Azure AD Connect Health 代理提供 AD FS 审核日志中的信息。 默认情况下未启用这些日志。 使用以下过程在 AD FS 服务器上启用 AD FS 审核和查找 AD FS 审核日志。

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2012-r2"></a>在 Windows Server 2012 R2 上启用 AD FS 审核的步骤

1. 通过在“开始”屏幕上打开“服务器管理器”或在桌面的任务栏中打开“服务器管理器”的方式打开“本地安全策略”，并单击“工具/本地安全策略”。
2. 导航到 **“安全设置\本地策略\用户权限分配”** 文件夹，然后双击 **“生成安全审核”** 。
3. 在 **“本地安全设置”** 选项卡上，验证是否列出了 AD FS 服务帐户。 如果该服务帐户不存在，请单击 **“添加用户或组”** 并将其添加到列表中，然后单击 **“确定”** 。
4. 若要启用审核，请使用提升的权限打开命令提示符，并运行以下命令：```auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable```
5. 关闭“本地安全策略”。
<br />   -- **仅主 AD FS 服务器需要执行以下步骤。** -- <br />
6. 打开“AD FS 管理”管理单元（在“服务器管理器”中，单击“工具”，并选择“AD FS 管理”）。
7. 在“操作”窗格中，单击“编辑联合身份验证服务属性”。
8. 在 **“联合身份验证服务属性”** 对话框中，单击 **“事件”** 选项卡。
9. 选择“成功审核”和“失败审核”复选框，并单击“确定”。
10. 可以使用命令通过 powershell 启用详细日志记录 ```Set-AdfsProperties -LOGLevel Verbose``` 。

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2016"></a>在 Windows Server 2016 上针对 AD FS 启用审核

1. 通过在“开始”屏幕上打开“服务器管理器”或在桌面的任务栏中打开“服务器管理器”的方式打开“本地安全策略”，并单击“工具/本地安全策略”。
2. 导航到 **“安全设置\本地策略\用户权限分配”** 文件夹，然后双击 **“生成安全审核”** 。
3. 在 **“本地安全设置”** 选项卡上，验证是否列出了 AD FS 服务帐户。 如果该帐户不存在，请单击“添加用户或组”将 AD FS 服务帐户添加到列表中，然后单击“确定”。
4. 若要启用审核，请使用提升的权限打开命令提示符，并运行以下命令：<code>auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable</code>
5. 关闭“本地安全策略”。
<br />   -- **仅主 AD FS 服务器需要执行以下步骤。** -- <br />
6. 打开“AD FS 管理”管理单元（在“服务器管理器”中，单击“工具”，并选择“AD FS 管理”）。
7. 在“操作”窗格中，单击“编辑联合身份验证服务属性”。
8. 在 **“联合身份验证服务属性”** 对话框中，单击 **“事件”** 选项卡。
9. 选择“成功审核”和“失败审核”复选框，并单击“确定”。 默认情况下，会启用此项。
10. 打开 PowerShell 窗口并运行以下命令：```Set-AdfsProperties -AuditLevel Verbose```。

请注意，默认情况下启用“基本”审核级别。 请阅读有关 [Windows Server 2016 中的 AD FS 审核增强功能](/windows-server/identity/ad-fs/technical-reference/auditing-enhancements-to-ad-fs-in-windows-server)的更多内容


#### <a name="to-locate-the-ad-fs-audit-logs"></a>查找 AD FS 审核日志的步骤

1. 打开“ **事件查看器** ”。
2. 转到“Windows 日志”，并选择“安全”。
3. 在右侧单击“筛选当前日志”。
4. 在“事件源”下选择“AD FS 审核”。

    选择简短的适用于审核日志的[常见问题解答说明](reference-connect-health-faq.md#operations-questions)。

![AD FS 审核日志](./media/how-to-connect-health-agent-install/adfsaudit.png)

> [!WARNING]
> 组策略可以禁用 AD FS 审核。 如果禁用了 AD FS 审核，则不能使用有关登录活动的使用情况分析。 请确保没有组策略禁用 AD FS 审核。>
>


## <a name="installing-the-azure-ad-connect-health-agent-for-sync"></a>安装用于同步的 Azure AD Connect Health 代理

安装最新版本的 Azure AD Connect 时，会自动安装用于同步的 Azure AD Connect Health 代理。 若要使用 Azure AD Connect 进行同步，需要下载并安装最新版本的 Azure AD Connect。 可以在 [此处](https://www.microsoft.com/download/details.aspx?id=47594)下载最新版本。

若要验证代理是否已安装，请在服务器上查看以下服务。 如果已完成配置，这些服务应已运行。 否则，它们会停止，直到完成配置。

* Azure AD Connect Health Sync Insights 服务
* Azure AD Connect Health Sync 监视服务

![验证用于同步的 Azure AD Connect Health](./media/how-to-connect-health-agent-install/services.png)

> [!NOTE]
> 请记住，Azure AD Connect Health 需要与 Azure AD Premium 配合使用。 如果没有 Azure AD Premium，则无法在 Azure 门户中完成配置。 有关详细信息，请参阅[要求页](how-to-connect-health-agent-install.md#requirements)。
>
>

## <a name="manual-azure-ad-connect-health-for-sync-registration"></a>手动注册用于同步的 Azure AD Connect Health

如果用于同步的 Azure AD Connect Health 代理在成功安装 Azure AD Connect 后注册失败，可以使用以下 PowerShell 命令来手动注册代理。

> [!IMPORTANT]
> 只有代理在安装 Azure AD Connect 之后注册失败时，才需要使用此 PowerShell 命令。
>
>

只有 Health 代理注册失败（即使在成功安装及设置 Azure AD Connect 之后）时，才需要以下 PowerShell 命令。 成功注册代理之后，Azure AD Connect Health 服务会启动。

可以使用以下 PowerShell 命令，手动注册用于同步的 Azure AD Connect Health 代理：

`Register-AzureADConnectHealthSyncAgent -AttributeFiltering $false -StagingMode $false`

此命令采用以下参数：

* AttributeFiltering：$true（默认值）- 如果 Azure AD Connect 未同步默认属性集，且已自定义为使用筛选的属性集。 否则为 $false。
* StagingMode：$false（默认值）- 如果 Azure AD Connect 服务器不处于暂存模式。如果服务器已配置为处于暂存模式，则为 $true。

当系统提示进行身份验证时，应该使用用于配置 Azure AD Connect 的同一全局管理员帐户（例如 admin@domain.onmicrosoft.com）。

## <a name="installing-the-azure-ad-connect-health-agent-for-ad-ds"></a>为 AD DS 安装 Azure AD Connect Health 代理

若要启动代理安装，请双击下载的 .exe 文件。 在第一个屏幕上，单击“安装”。

![AD DS 安装开始 Azure AD Connect Health 代理](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install1.png)

安装完成后，单击“立即配置”。

![AD DS 安装完成的 Azure AD Connect Health 代理](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install2.png)

此时会启动一个命令提示符，其后面是执行 Register-AzureADConnectHealthADDSAgent 的一些 PowerShell 命令。 根据系统的提示登录到 Azure。

![AD DS 配置登录的 Azure AD Connect Health 代理](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install3.png)

登录后，将继续 PowerShell。 完成后，可以关闭 PowerShell，因为配置已完成。

此时，服务应会自动启动，使代理能够监视和收集数据。 如果不满足前面部分中所述的所有先决条件，PowerShell 窗口中会出现警告。 请务必在安装代理之前满足[要求](how-to-connect-health-agent-install.md#requirements)。 以下屏幕截图是这些错误的一个示例。

![AD DS 配置脚本的 Azure AD Connect Health 代理](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install4.png)

若要验证代理是否已安装，请在域控制器上查看以下服务。

* Azure AD Connect Health AD DS Insights 服务
* Azure AD Connect Health AD DS 监视服务

如果已完成配置，这些服务应已运行。 否则，它们会停止，直到完成配置。

![AD DS 服务的 Azure AD Connect Health 代理](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install5.png)

### <a name="quick-agent-installation-in-multiple-servers"></a>在多个服务器中安装快速代理

1. 使用密码在 Azure AD 中创建用户帐户。
2. 通过门户为 Azure AD Connect Health 中的本地 AAD 帐户分配 **所有者** 角色。 按照[此处](how-to-connect-health-operations.md#manage-access-with-azure-rbac)的步骤操作。 将角色分配给所有服务实例。 
3. 在本地域控制器中下载 .exe MSI 文件以进行安装。
4. 运行以下脚本以进行注册。 将参数替换为创建的新用户帐户及其密码。 

```powershell
AdHealthAddsAgentSetup.exe /quiet
Start-Sleep 30
$userName = "NEWUSER@DOMAIN"
$secpasswd = ConvertTo-SecureString "PASSWORD" -AsPlainText -Force
$myCreds = New-Object System.Management.Automation.PSCredential ($userName, $secpasswd)
import-module "C:\Program Files\Azure Ad Connect Health Adds Agent\PowerShell\AdHealthAdds"
 
Register-AzureADConnectHealthADDSAgent -Credential $myCreds

```

1. 完成后，可以通过执行下列一项或多项操作来删除本地帐户的访问权限： 
    * 删除 AAD Connect Health 的本地帐户的角色分配
    * 旋转本地帐户的密码。 
    * 禁用 AAD 本地帐户
    * 删除 AAD 本地帐户  

## <a name="agent-registration-using-powershell"></a>使用 PowerShell 进行的代理注册

在安装适当的代理 setup.exe 后，可以根据角色使用以下 PowerShell 命令执行代理注册步骤。 打开 PowerShell 窗口并执行相应的命令：

```powershell
    Register-AzureADConnectHealthADFSAgent
    Register-AzureADConnectHealthADDSAgent
    Register-AzureADConnectHealthSyncAgent

```

这些命令接受“凭据”作为参数，以便以非交互方式完成注册，或者在服务器-核心计算机上进行该操作。
* 可以在作为参数传递的 PowerShell 变量中捕获凭据。
* 可以提供任何 Azure AD 标识，只要该标识具有注册代理所需的访问权限且尚未启用 MFA。
* 默认情况下，全局管理员有权执行代理注册。 也可允许其他特权较小的标识执行此步骤。 详细了解 azure [RBAC)  (azure 基于角色的访问控制 ](how-to-connect-health-operations.md#manage-access-with-azure-rbac)。

```powershell
    $cred = Get-Credential
    Register-AzureADConnectHealthADFSAgent -Credential $cred

```

## <a name="configure-azure-ad-connect-health-agents-to-use-http-proxy"></a>将 Azure AD Connect Health 代理配置为使用 HTTP 代理

可以将 Azure AD Connect Health 代理配置为使用 HTTP 代理。

> [!NOTE]
> * 不支持使用“Netsh WinHttp set ProxyServerAddress”，因为代理使用 System.Net（而不是 Microsoft Windows HTTP 服务）发出 Web 请求。
> * 配置的 Http 代理地址用于传递加密的 Https 消息。
> * 不支持经过身份验证的代理（使用 HTTPBasic）。
>
>

### <a name="change-health-agent-proxy-configuration"></a>更改 Health 代理的代理配置

可以使用以下选项将 Azure AD Connect Health 代理配置为使用 HTTP 代理。

> [!NOTE]
> 必须重新启动所有 Azure AD Connect Health 代理服务才能更新代理设置。 运行以下命令：<br />
> Restart-Service AzureADConnectHealth *
>
>

#### <a name="import-existing-proxy-settings"></a>导入现有的代理设置

##### <a name="import-from-internet-explorer"></a>从 Internet Explorer 导入

可以导入 Internet Explorer HTTP 代理设置供 Azure AD Connect Health 代理使用。 在运行 Health 代理的每台服务器上执行以下 PowerShell 命令：

```powershell
Set-AzureAdConnectHealthProxySettings -ImportFromInternetSettings
```

##### <a name="import-from-winhttp"></a>从 WinHTTP 导入

可以导入 WinHTTP 代理设置供 Azure AD Connect Health 代理使用。 在运行 Health 代理的每台服务器上执行以下 PowerShell 命令：

```powershell
Set-AzureAdConnectHealthProxySettings -ImportFromWinHttp
```

#### <a name="specify-proxy-addresses-manually"></a>手动指定代理地址

可以执行以下 PowerShell 命令，在运行 Health 代理的每台服务器上手动指定代理服务器：

```powershell
Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress address:port
```

示例： *Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress myproxyserver: 443*

* “地址”可以是 DNS 可解析的服务器名称或 IPv4 地址
* 可以省略“端口”。 如果省略端口，则会选择 443 作为默认端口。

#### <a name="clear-existing-proxy-configuration"></a>清除现有的代理配置

可以运行以下命令清除现有的代理配置：

```powershell
Set-AzureAdConnectHealthProxySettings -NoProxy
```

### <a name="read-current-proxy-settings"></a>读取当前的代理设置

可以运行以下命令读取当前配置的代理设置：

```powershell
Get-AzureAdConnectHealthProxySettings
```

## <a name="test-connectivity-to-azure-ad-connect-health-service"></a>测试与 Azure AD Connect Health 服务的连接

问题可能会导致 Azure AD Connect Health 代理与 Azure AD Connect Health 服务连接断开。 这些问题包括网络问题、权限问题或其他各种原因。

如果代理无法将数据发送到 Azure AD Connect Health 服务达 2 小时以上，门户中会出现以下警报：“Health 服务数据不是最新的”。 可以运行以下 PowerShell 命令，确认受影响的 Azure AD Connect Health 代理是否能够将数据上传到 Azure AD Connect Health 服务：

```powershell
Test-AzureADConnectHealthConnectivity -Role ADFS
```

role 参数目前可接受以下值：

* ADFS
* 同步
* ADDS

> [!NOTE]
> 若要使用连接工具，必须先完成代理注册。 如果无法完成代理注册，请确保符合 Azure AD Connect Health 的所有[要求](how-to-connect-health-agent-install.md#requirements)。 默认情况下，此连接测试会在代理注册期间执行。
>
>

## <a name="related-links"></a>相关链接

* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Azure AD Connect Health 操作](how-to-connect-health-operations.md)
* [在 AD FS 中使用 Azure AD Connect Health](how-to-connect-health-adfs.md)
* [使用用于同步的 Azure AD Connect Health](how-to-connect-health-sync.md)
* [在 AD DS 中使用 Azure AD Connect Health](how-to-connect-health-adds.md)
* [Azure AD Connect Health 常见问题](reference-connect-health-faq.md)
* [Azure AD Connect Health 版本历史记录](reference-connect-health-version-history.md)
