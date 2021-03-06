---
title: 教程：为 Fuze 配置自动用户预配 Azure Active Directory |Microsoft Docs
description: 了解如何配置 Azure Active Directory 以自动将用户帐户预配到 Fuze 以及取消其预配。
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 07/26/2019
ms.author: zhchia
ms.openlocfilehash: c92d7afb4c1de2596b4c98f50a003fe31176fbb7
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92449766"
---
# <a name="tutorial-configure-fuze-for-automatic-user-provisioning"></a>教程：为 Fuze 配置自动用户预配

本教程的目的是演示要在 Fuze 和 Azure Active Directory (Azure AD) 中执行的步骤，以将 Azure AD 自动预配和取消预配到 [Fuze](https://www.fuze.com/)。 有关此服务的功能、工作原理以及常见问题的重要详细信息，请参阅[使用 Azure Active Directory 自动将用户预配到 SaaS 应用程序和取消预配](../app-provisioning/user-provisioning.md)。

> [!NOTE]
> 此连接器目前以公共预览版提供。 若要详细了解 Microsoft Azure 预览版功能的一般使用条款，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。


## <a name="capabilities-supported"></a>支持的功能
> [!div class="checklist"]
> * 在 Fuze 中创建用户
> * 当用户不再需要访问权限时，删除 Fuze 中的用户
> * 使用户属性在 Azure AD 和 Fuze 之间保持同步
> * [单一登录](./fuze-tutorial.md) 到 Fuze (建议) 

## <a name="prerequisites"></a>先决条件

本教程中概述的方案假定你已具有以下先决条件：

* [Azure AD 租户](../develop/quickstart-create-new-tenant.md)。
* 具有配置预配[权限](../users-groups-roles/directory-assign-admin-roles.md)的 Azure AD 用户帐户（例如应用程序管理员、云应用程序管理员、应用程序所有者或全局管理员）。
* [Fuze 租户](https://www.fuze.com/)。
* Fuze 中具有管理员权限的用户帐户。


## <a name="step-1-plan-your-provisioning-deployment"></a>步骤 1。 规划预配部署
1. 了解[预配服务的工作原理](../app-provisioning/user-provisioning.md)。
2. 确定谁在[预配范围](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中。
3. 确定要 [在 Azure AD 与 Fuze 之间映射](../app-provisioning/customize-application-attributes.md)的数据。 

## <a name="step-2-configure-fuze-to-support-provisioning-with-azure-ad"></a>步骤 2. 配置 Fuze 以支持 Azure AD 的预配

将 Fuze 配置为使用 Azure AD 进行自动用户预配之前，需要在 Fuze 上启用 SCIM 设置。 

1. 首先联系 Fuze 代表以获取以下必需信息：

    * 当前在公司中使用的 Fuze Product Sku 列表。
    * 公司位置的位置代码列表。
    * 公司的部门代码列表。

2. 你可以在 Fuze 合同和配置文档中找到这些 Sku 和代码，或者联系 Fuze 代表。

3. 收到要求后，Fuze 代表将为你提供启用集成所需的 Fuze authentication 令牌。 此值将在 Azure 门户的 Fuze 应用程序的 "预配" 选项卡的 "机密令牌" 字段中输入。

## <a name="step-3-add-fuze-from-the-azure-ad-application-gallery"></a>步骤 3. 从 Azure AD 应用程序库添加 Fuze

从 Azure AD 应用程序库中添加 Fuze，开始管理预配到 Fuze。 如果以前为 SSO 设置了 Fuze，则可以使用相同的应用程序。 但建议你在最初测试集成时创建一个单独的应用。 可在[此处](../manage-apps/add-application-portal.md)详细了解如何从库中添加应用程序。

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>步骤 4. 定义谁在预配范围中 

使用 Azure AD 预配服务，可以根据对应用程序的分配和/或用户/组的属性来限定谁在预配范围内。 如果选择根据分配来查看要将谁预配到应用，则可以使用以下[步骤](../manage-apps/assign-user-or-group-access-portal.md)将用户和组分配给应用程序。 如果选择仅根据用户或组的属性来限定要对谁进行预配，可以使用[此处](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)所述的范围筛选器。 

* 将用户分配到 Fuze 时，必须选择 " **默认" 访问权限**以外的角色。 具有“默认访问”角色的用户将从预配中排除，并在预配日志中被标记为未有效授权。 如果应用程序上唯一可用的角色是默认访问角色，则可以[更新应用程序清单](../develop/howto-add-app-roles-in-azure-ad-apps.md)以添加其他角色。 

* 先小部分测试。 使用少量用户进行测试，然后再向所有人推出。 如果设置的作用域设置为 "分配的用户"，则可以通过将一个或两个用户分配到应用来对此进行控制。 如果作用域设置为 "所有用户"，则可以指定 [基于属性的范围筛选器](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)。 

## <a name="step-5-configuring-automatic-user-provisioning-to-fuze"></a>步骤 5。 配置 Fuze 的自动用户预配 

本部分将指导你完成以下步骤：配置 Azure AD 预配服务，以便基于 Azure AD 中的用户和/或组分配在 Fuze 中创建、更新和禁用用户和/或组。

### <a name="to-configure-automatic-user-provisioning-for-fuze-in-azure-ad"></a>若要在 Azure AD 中配置 Fuze 的自动用户预配：

1. 登录 [Azure 门户](https://portal.azure.com)。 依次选择“企业应用程序”、“所有应用程序” 。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“Fuze”****。

    ![应用程序列表中的 Fuze 链接](common/all-applications.png)

3. 选择“预配”  选项卡。

    ![带有称为 "预配" 选项的 "管理" 选项的屏幕截图。](common/provisioning.png)

4. 将“预配模式”  设置为“自动”  。

    ![具有 "自动" 选项的 "预配模式" 下拉列表屏幕截图。](common/provisioning-automatic.png)

5. 在 "**管理员凭据**" 部分下，输入先前从**租户 Url**和**机密令牌**中的 Fuze 代表那里检索到的**SCIM 2.0 基 url 和 SCIM Authentication 令牌**值。 单击 " **测试连接** " 以确保 Azure AD 可以连接到 Fuze。 如果连接失败，请确保 Fuze 帐户具有管理员权限，然后重试。

    ![租户 URL 标记](common/provisioning-testconnection-tenanturltoken.png)

6. 在“通知电子邮件”字段中，输入应接收预配错误通知的个人或组的电子邮件地址，并选中复选框“发生故障时发送电子邮件通知”   。

    ![通知电子邮件](common/provisioning-notification-email.png)

7. 单击“ **保存**”。

8. 在 " **映射** " 部分下，选择 " **将 Azure Active Directory 用户同步到 Fuze**"。

9. 在 " **属性映射** " 部分中，查看从 Azure AD 同步到 Fuze 的用户属性。 选为 " **匹配** " 属性的特性用于匹配 Fuze 中的用户帐户以执行更新操作。 选择“保存”按钮以提交任何更改。

   |Attribute|类型|
   |---|---|
   |userName|字符串|
   |name.givenName|字符串|
   |name.familyName|字符串|
   |emails[type eq "work"].value|字符串|
   |活动|Boolean|

10. 若要配置范围筛选器，请参阅[范围筛选器教程](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中提供的以下说明。

11. 若要为 Fuze 启用 Azure AD 预配服务，请在 "**设置**" 部分中将 "**预配状态**" 更改为 **"打开**"。

    ![预配状态已打开](common/provisioning-toggle-on.png)

12. 通过在 "**设置**" 部分的 "**范围**" 中选择所需的值，定义要预配到 Fuze 的用户和/或组。

    ![预配范围](common/provisioning-scope.png)

15. 已准备好预配时，单击“保存”  。

    ![保存预配配置](common/provisioning-configuration-save.png)

此操作会对“设置”部分的“范围”中定义的所有用户和/或组启动初始同步   。 初始同步执行的时间比后续同步长，只要 Azure AD 预配服务正在运行，大约每隔 40 分钟就会进行一次同步。

## <a name="step-6-monitor-your-deployment"></a>步骤 6. 监视部署
配置预配后，请使用以下资源来监视部署：

1. 通过[预配日志](../reports-monitoring/concept-provisioning-logs.md)来确定哪些用户已预配成功或失败
2. 检查[进度栏](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md)来查看预配周期的状态以及完成进度
3. 如果怀疑预配配置处于非正常状态，则应用程序将进入隔离状态。 有关隔离状态的详细信息，请访问[此处](../app-provisioning/application-provisioning-quarantine-status.md)。

## <a name="connector-limitations"></a>连接器限制

* Fuze 支持称为 " **权利**" 的自定义 SCIM 属性。 仅可创建和不更新这些属性。 

## <a name="change-log"></a>更改日志

* 06/15/2020-集成的速率限制为每秒10个请求。

## <a name="additional-resources"></a>其他资源

* [管理企业应用的用户帐户设置](../app-provisioning/configure-automatic-user-provisioning-portal.md)。
* [Azure Active Directory 的应用程序访问与单一登录是什么？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>后续步骤

* [了解如何查看日志并获取有关预配活动的报告](../app-provisioning/check-status-user-account-provisioning.md)。