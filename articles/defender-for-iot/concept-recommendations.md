---
title: 安全建议
description: 了解安全建议的概念，以及如何在用于 IoT 的 Defender 中使用它们。
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 0eccab6c3d59ad68ddc8f96c3d84c57dc1bbeeca
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "90934484"
---
# <a name="security-recommendations"></a>安全建议

Defender for IoT 扫描 Azure 资源和 IoT 设备，并提供安全建议以减少攻击面。
安全建议是可操作的，旨在帮助客户遵守安全最佳做法。

本文介绍可在 IoT 中心和/或 IoT 设备上触发的建议列表。

## <a name="recommendations-for-iot-devices"></a>IoT 设备建议

设备建议提供有关改进设备安全状况的见解和建议。

| 严重性 | 名称                                                      | 数据源 | 说明                                                                                                                                                                                           |
|----------|-----------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 中型   | 打开设备上的端口                                      | 代理       | 在设备上找到侦听终结点。                                                                                                                                                        |
| 中型   | 在其中一个链中找到的可许可防火墙策略。 | 代理       |  (输入/输出) 找到允许的防火墙策略。 默认情况下，防火墙策略应拒绝所有流量，并定义规则以允许与设备进行必要的通信。                               |
| 中型   | 找到输入链中的可许可防火墙规则     | 代理       | 已发现防火墙中包含广泛 IP 地址或端口的许可模式的规则。                                                                                    |
| 中型   | 找到输出链中的可许可防火墙规则    | 代理       | 已发现防火墙中包含广泛 IP 地址或端口的许可模式的规则。                                                                                   |
| 中型   | 操作系统基线验证失败           | 代理       | 设备不符合 [CIS Linux 基准](https://www.cisecurity.org/cis-benchmarks/)。                                                                                                        |

### <a name="operational-recommendations-for-iot-devices"></a>IoT 设备的操作建议

操作建议提供有关改进安全代理配置的见解和建议。

| 严重性 | 名称                                    | 数据源 | 说明                                                                       |
|----------|-----------------------------------------|-------------|-----------------------------------------------------------------------------------|
| 低      | 代理发送未利用消息          | 代理       | 10% 或更多的安全消息在过去24小时内小于 4 KB。  |
| 低      | 安全克隆配置不最佳 | 代理       | 安全克隆配置不是最佳的。                                        |
| 低      | 安全克隆配置冲突    | 代理       | 安全克隆配置中标识了冲突。 |                          |
|

## <a name="recommendations-for-iot-hub"></a>IoT 中心建议

建议警报提供用于改进环境安全状况的操作的见解和建议。

| 严重性 | 名称                                                     | 数据源 | 说明                                                                                                                                                                                                             |
|----------|----------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 高     | 多设备使用相同的身份验证凭据 | IoT 中心     | IoT 中心身份验证凭据由多个设备使用。 这可能表示某个非法设备模拟了合法设备。 重复的凭据使用增加了恶意执行组件对设备进行模拟的风险。 |
| 中型   | 默认 IP 筛选器策略应为 deny                  | IoT 中心     | IP 筛选器配置应为允许的流量定义规则，默认情况下，默认情况下，拒绝所有其他流量。                                                                                                     |
| 中型   | IP 筛选器规则包含大型 IP 范围                   | IoT 中心     | 允许 IP 筛选规则源 IP 范围太大。 过度许可的规则可以将 IoT 中心暴露给恶意参与者。                                                                                       |
| 低      | 在 IoT 中心启用诊断日志                       | IoT 中心     | 启用日志并将其保留长达一年的时间。 保留日志使你可以在安全事件发生或网络泄露时重新创建用于调查的活动跟踪。                                       |
|

## <a name="next-steps"></a>后续步骤

- Defender for IoT 服务 [概述](overview.md)
- 了解如何 [访问安全数据](how-to-security-data-access.md)
- 了解有关[调查设备的](how-to-investigate-device.md)详细信息
