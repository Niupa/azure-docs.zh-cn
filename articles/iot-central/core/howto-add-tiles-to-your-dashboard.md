---
title: 将磁贴添加到 Azure IoT Central 仪表板 |Microsoft Docs
description: 作为生成器，了解如何使用磁贴配置默认的 Azure IoT Central 应用程序仪表板。
author: Haley-Rowland
ms.author: harowl
ms.date: 05/27/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 5276f9b8c6dd3bdc305142e5b9452cd9c5d60bce
ms.sourcegitcommit: 051908e18ce42b3b5d09822f8cfcac094e1f93c2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/09/2020
ms.locfileid: "94376974"
---
# <a name="configure-the-application-dashboard"></a>配置应用程序仪表板

**仪表板** 是连接到 IoT Central 应用程序时看到的第一页。 如果从以行业为中心的 [应用程序模板](./concepts-app-templates.md)创建应用程序，应用程序将具有开始预定义的仪表板。 如果从自定义 [应用程序模板](./concepts-app-templates.md)创建应用程序，你的仪表板将显示一些入门提示。

> [!TIP]
> 除了默认应用程序仪表板外，用户还可以[创建多个仪表板](howto-create-personal-dashboards.md)。 这些仪表板可以仅供用户使用，也可以在应用程序的所有用户之间共享。  

## <a name="add-tiles"></a>添加磁贴

以下屏幕截图显示了从 **自定义应用程序** 模板创建的应用程序中的仪表板。 若要自定义当前仪表板，请选择 " **编辑** "，添加自定义个人或共享仪表板，选择 " **新建** "：

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/dashboard-sample-contoso.png" alt-text="基于自定义应用程序模板的应用程序的面板":::

选择 " **编辑** " 或 " **新建** " 后，仪表板处于 *编辑* 模式。 你可以使用 " **编辑仪表板** " 面板中的工具将磁贴添加到仪表板，并在仪表板上自定义和删除磁贴。 例如，若要添加 **遥测** 磁贴以显示由一个或多个设备报告的当前温度：

1. 在 " **编辑仪表板** " 面板中，选择一个 **设备组** 。
1. 在 " **设备** " 下拉列表中选择一个或多个设备以在磁贴上显示。 现在，你可以看到设备中的可用遥测、属性和命令。
1. 在 "遥测" 部分选择 " **温度** "，然后选择 " **添加磁贴** "。 现在，磁贴显示在仪表板上，你可以在其中更改可视化效果、调整磁贴的大小并对其进行配置：

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/device-details.png" alt-text="将温度遥测磁贴添加到仪表板":::

在仪表板上添加并自定义磁贴后，请选择 " **保存** "。

## <a name="customize-tiles"></a>自定义磁贴

要在仪表板上自定义磁贴，仪表板必须处于编辑模式。 可用的自定义选项取决于 [磁贴类型](#tile-types)：

* 磁贴上的标尺图标使你可以更改可视化效果。 可视化效果包括折线图、上一已知值和热度地图。

* 利用方形图标可以调整磁贴的大小。

* 齿轮图标使你可以配置可视化效果。 例如，对于折线图视觉对象，可以选择显示图例和轴，并选择要绘制的时间范围。

## <a name="tile-types"></a>磁贴类型

下表描述了可以添加到仪表板的不同类型的磁贴：

| 磁贴             | 说明 |
| ---------------- | ----------- |
| Markdown         | Markdown 磁贴是可单击的磁贴，用于显示使用 Markdown 设置格式的标题和说明文本。 URL 可以是指向应用程序中其他页面的相对链接，也可以是指向外部站点的绝对链接。|
| 图像            | 图像磁贴显示自定义图像，可以单击。 URL 可以是指向应用程序中其他页面的相对链接，也可以是指向外部站点的绝对链接。|
| Label            | 标签磁贴在仪表板上显示自定义文本。 你可以选择文本的大小。 使用标签磁贴向仪表板添加相关信息，例如说明、联系人详细信息或帮助。|
| 计数            | 计数磁贴显示设备组中的设备数。|
| 映射              | 地图磁贴显示一个或多个设备在地图上的位置。 你还可以显示设备位置历史记录的 100 个点。 例如，可以显示设备在过去一周内的抽样路线。|
| KPI              |  KPI 磁贴显示一段时间内一个或多个设备的聚合遥测值。 例如，你可以使用它来显示最近一小时内一个或多个设备达到的最大温度和压力。|
| 折线图       | 折线图磁贴将一个或多个设备的一个或多个聚合遥测值绘制一段时间。 例如，可以显示一个折线图，用于绘制最近一小时内一个或多个设备的平均温度和压力。|
| 条形图        | 条形图磁贴将一个或多个设备的一个或多个聚合遥测值绘制一段时间。 例如，您可以显示一个条形图来显示最近一小时内一个或多个设备的平均温度和压力。|
| 饼图        | 饼图磁贴显示一个或多个设备在一段时间内的一个或多个聚合遥测值。|
| 热度地图         | 热度地图磁贴显示有关一个或多个设备的信息，表示为颜色。|
| 上一个已知值 | "最后一个已知值" 磁贴显示一个或多个设备的最新遥测值。 例如，你可以使用此磁贴显示一个或多个设备的最新温度、压力和湿度值。 |
| 事件历史记录    | “事件历史记录”磁贴显示设备在一段时间内的事件。 例如，你可以使用它来显示最近一小时内一个或多个设备的所有阀打开和关闭事件。|
| 属性         |  属性磁贴显示一个或多个设备的属性和云属性的当前值。 例如，你可以使用此磁贴来显示设备的属性，例如设备的制造商或固件版本。 |

目前，最多可以向支持多个设备的磁贴添加10个设备。

### <a name="customizing-visualizations"></a>自定义可视化效果

默认情况下，折线图显示时间范围内的数据。 所选时间范围拆分为50相等大小的存储桶，并按 bucket 聚合设备数据，以在所选时间范围内为50个数据点。 若要查看原始数据，可以更改所选内容以查看最后一个100值。 若要更改时间范围或选择原始数据可视化，请使用 " **配置图表** " 面板中的 "显示范围" 下拉列表。

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/display-range.png" alt-text="更改折线图的显示范围":::

对于显示聚合值的磁贴，请在 " **配置图表** " 面板中选择 "遥测" 类型旁边的齿轮图标以选择聚合。 可以从 average、sum、maximum、maximum 和 count 中进行选择。

对于折线图、条形图和饼图，可以自定义不同遥测值的颜色。 选择要自定义的遥测旁边的 "调色板" 图标：

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/color-customization.png" alt-text="更改遥测值的颜色":::

对于显示字符串属性或遥测值的磁贴，你可以选择显示文本的方式。 例如，如果设备将 URL 存储在字符串属性中，则可以将其显示为可单击的链接。 如果 URL 引用图像，则可以在上一个已知值或属性磁贴中呈现图像。 若要更改字符串的显示方式，请在 "磁贴配置" 中选择遥测类型或属性旁边的齿轮图标：

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/string-customization.png" alt-text="更改字符串在磁贴上的显示方式":::

对于数值 **KPI** 、 **上一个已知值** 和 **属性** 磁贴，你可以使用条件格式基于其当前值自定义磁贴的颜色。 若要添加条件格式，请在磁贴上选择 " **配置** "，然后选择要自定义的值旁边的 **条件格式** 图标：

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-1.png" alt-text="显示如何查找磁贴的 &quot;配置&quot; 选项，然后选择 &quot;条件格式&quot; 图标的屏幕截图":::

添加条件格式规则：

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-2.png" alt-text="显示平均流的条件格式规则的屏幕截图。有三个规则-不小于50是绿色，小于为黄色，超过50的任何内容为红色":::

以下屏幕截图显示了条件格式规则的效果：

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-3.png" alt-text="显示平均水流量磁贴上黄色背景色的屏幕截图。磁贴上的数字为40.84":::

## <a name="next-steps"></a>后续步骤

现在，你已了解如何配置 Azure IoT Central 默认应用程序仪表板，接下来可以[了解如何创建个人仪表板](howto-create-personal-dashboards.md)。
