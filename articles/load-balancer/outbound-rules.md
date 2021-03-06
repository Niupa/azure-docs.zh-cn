---
title: Azure 负载均衡器的出站规则
description: 本文介绍如何通过 Azure 负载均衡器配置出站规则，以控制 internet 流量的出口。
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: conceptual
ms.custom: contperfq1
ms.date: 10/13/2020
ms.author: allensu
ms.openlocfilehash: 947ecaa2efbfb013f1f3e8203d1c4296b9ca329f
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422155"
---
# <a name="outbound-rules-azure-load-balancer"></a><a name="outboundrules"></a>Azure 负载均衡器的出站规则

出站规则允许你为公共标准负载均衡器显式定义 SNAT (源网络地址转换) 。 此配置允许你使用负载均衡器的公共 IP () 来提供后端实例的出站 internet 连接。

此配置启用：

* IP 伪装
* 简化允许列表。
* 减少用于部署的公共 IP 资源数。

对于出站规则，您对出站 internet 连接具有完全声明性控制。 出站规则允许你根据特定需求缩放和优化此功能。 

仅当后端 VM 没有实例级公共 IP 地址 (ILPIP) 时，才会执行出站规则。

![负载均衡器出站规则](media/load-balancer-outbound-rules-overview/load-balancer-outbound-rules.png)

对于出站规则，您可以显式定义出站 **SNAT** 行为。

使用出站规则可以控制：

* **哪些虚拟机将被转换为哪些公共 IP 地址。**
     * 后端池 A 使用 IP 地址 A 和 B，后端池 B 使用 IP 地址 C 和 D。
* **如何分配出站 SNAT 端口。**
     * 后端池 B 是建立出站连接的唯一池，将所有 SNAT 端口向后端池 B 和无连接到后端池 A。
* **要为哪些协议提供出站转换。**
     * 后端池 B 需要 UDP 端口才能进行出站。 后端池 A 需要 TCP。 为 A 和 UDP 端口指定 B 的 TCP 端口。
* **用于出站连接空闲超时的持续时间（4-120 分钟）。**
     * 如果有长时间运行的与 keepalive 的连接，请为长时间运行的连接保留空闲端口长达120分钟。 假设过时连接被放弃，并在4分钟内针对新连接释放端口 
* **是否在空闲超时后发送 TCP 重置。**
     * 当超时空闲连接时，是否向客户端和服务器发送 TCP RST，以便他们知道是否放弃了流？

## <a name="outbound-rule-definition"></a>出站规则定义

出站规则遵循用户熟悉的与负载均衡和入站 NAT 规则相同的语法： **前端** + **参数** + **后端池** 。 

出站规则为后端池识别的、要转换为前端的所有虚拟机配置出站 NAT。   

参数针对出站 NAT 算法提供更精细的控制。

## <a name="scale-outbound-nat-with-multiple-ip-addresses"></a><a name="scale"></a>使用多个 IP 地址缩放出站 NAT

前端提供的每个附加 IP 地址可提供额外的 64,000 个临时端口，供负载均衡器用作 SNAT 端口。 

使用多个 IP 地址来规划大规模方案。 使用出站规则来缓解 [SNAT 耗尽](troubleshoot-outbound-connection.md#snatexhaust)的情况。 

你还可以直接在出站规则中使用[公共 IP 前缀](https://aka.ms/lbpublicipprefix)。 

公共 IP 前缀增强了部署的缩放。 可以将前缀添加到源自 Azure 资源的流的允许列表中。 可以在负载均衡器中配置引用公共 IP 地址前缀所需的前端 IP 配置。  

负载均衡器可控制公共 IP 前缀。 出站规则会自动使用公共 IP 前缀中包含的所有公共 IP 地址来建立出站连接。 

公共 IP 前缀范围内的每个 IP 地址可提供额外的 64,000 个临时端口，供负载均衡器用作 SNAT 端口。

## <a name="outbound-flow-idle-timeout-and-tcp-reset"></a><a name="idletimeout"></a> 出站流空闲超时和 TCP 重置

出站规则提供一个配置参数用于控制出站流空闲超时，并使该超时符合应用程序的需求。 出站空闲超时默认为 4 分钟。 有关详细信息，请参阅[配置空闲超时](load-balancer-tcp-idle-timeout.md)。 

负载均衡器的默认行为是在达到了出站空闲超时时以静默方式丢弃流。 `enableTCPReset` 参数可以让应用程序的行为和控制更具可预测性。 此参数指示在发生出站空闲超时时是否要发送双向 TCP 重置 (TCP RST)。 

查看[在空闲超时时 TCP 重置](https://aka.ms/lbtcpreset)，了解详细信息，包括区域可用性。

## <a name="securing-and-controlling-outbound-connectivity-explicitly"></a><a name="preventoutbound"></a>显式保护和控制出站连接

负载均衡规则提供出站 NAT 的自动编程。 某些方案受益于或者要求通过负载均衡规则禁用出站 NAT 的自动编程。 通过该规则进行禁用可以控制或优化行为。  

可通过两种方式使用此参数：

1. 禁止将入站 IP 地址用于出站 SNAT。 在负载均衡规则中禁用出站 SNAT。
  
2. 对同时用于入站和出站连接的 IP 地址的出站 **SNAT** 参数进行优化。 必须禁用自动出站 NAT 才能让出站规则掌管控制权。 若要更改也用于入站连接的某个地址的 SNAT 端口分配，则必须将 `disableOutboundSnat` 参数设置为 true。 

如果尝试重新定义用于入站连接的 IP 地址，则配置出站规则的操作会失败。  请先禁用负载均衡规则的出站 NAT。

>[!IMPORTANT]
> 如果将此参数设置为 true，但没有任何出站规则来定义出站连接，则虚拟机将不会建立出站连接。  VM或应用程序的某些操作可能依赖于公网连接。 请务必了解方案的依赖关系，并考虑此项更改造成的影响。

有时，让 VM 创建出站流是不合需要的。 可能需要对哪些目标接收出站流或哪些目标启动入站流进行管理。 使用[网络安全组](../virtual-network/security-overview.md)可管理 VM 访问的目标。 使用 NSG 可对哪些公共目标启动入站流进行管理。

将 NSG 应用于负载均衡的 VM 时，需要注意[服务标记](../virtual-network/security-overview.md#service-tags)和[默认安全规则](../virtual-network/security-overview.md#default-security-rules)。 

请确保 VM 可以接收来自 Azure 负载均衡器的运行状况探测请求。

如果 NSG 阻止来自 AZURE_LOADBALANCER 默认标记的运行状况探测请求，则 VM 运行状况探测失败，VM 被标记为不可用。 负载均衡器会停止向该 VM 发送新流。

## <a name="scenarios-with-outbound-rules"></a>具有出站规则的方案
        

### <a name="outbound-rules-scenarios"></a>出站规则方案


* 配置与一组特定公共 Ip 或前缀的出站连接。
* 修改 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口分配。
* 仅启用出站。
* 仅 (没有入站) 的 Vm 的出站 NAT。
* 内部标准负载均衡器的出站 NAT。
* 为带有公共标准负载均衡器的出站 NAT 启用 TCP & UDP 协议。


### <a name="scenario-1-configure-outbound-connections-to-a-specific-set-of-public-ips-or-prefix"></a><a name="scenario1out"></a>方案1：配置与一组特定公共 Ip 或前缀的出站连接


#### <a name="details"></a>详细信息


使用此方案将出站连接调整成源自一组公共 IP 地址。 根据来源向允许列表或拒绝列表添加公共 IP 或前缀。


此公共 IP 或前缀可与负载均衡规则使用的相同。 


若要使用与负载均衡规则使用的公共 IP 或前缀不同的公共 IP 或前缀，请执行以下操作： 


1. 创建公共 IP 前缀或公共 IP 地址。
2. 创建公共标准负载均衡器 
3. 创建一个前端，用于引用所要使用的公共 IP 前缀或公共 IP 地址。 
4. 重复使用某个后端池或创建一个后端池，并将 VM 放入公共负载均衡器的后端池
5. 在公共负载均衡器上配置出站规则，以使用前端为这些 VM 启用出站 NAT。 不建议将负载均衡规则用于出站，请在负载均衡规则上禁用出站 SNAT。


### <a name="scenario-2-modify-snat-port-allocation"></a><a name="scenario2out"></a>方案2：修改 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口分配


#### <a name="details"></a>详细信息


可以使用出站规则[基于后端池大小优化自动 SNAT 端口分配](load-balancer-outbound-connections.md#preallocatedports)。 


如果遇到 SNAT 耗尽的情况，请增加给定的 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口数（默认值为 1024）。 


每个公共 IP 地址最多提供 64,000 个临时端口。 后端池中的 VM 数决定了分配给每个 VM 的端口数。 后端池中的一个 VM 最多可以访问 64,000 个端口。 对于两个 VM 的情况，可以使用出站规则为每个 VM 最多指定 32,000 个 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口 (2x 32,000 = 64,000)。 


可以使用出站规则来优化默认情况下给定的 SNAT 端口。 你指定的端口数可以多于或少于默认 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口分配提供的端口数。 出站规则的前端中的每个公共 IP 地址最多提供 64,000 个可用作 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口的临时端口。 


负载均衡器以 8 的倍数提供 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口。 如果提供的值不能被 8 整除，则会拒绝配置操作。 每个负载均衡规则和入站 NAT 规则将使用一系列8个端口。 如果负载平衡或入站 NAT 规则与另一规则共享同一范围8，则不会使用其他端口。


如果你尝试指定的 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口数超出了系统所能提供的端口数（具体取决于公共 IP 地址数），系统会拒绝该配置操作。 例如，如果为每个 VM 提供了10000个端口，而后端池中有7个 Vm 共享单个公共 IP，则配置将被拒绝。 7 乘以 10,000 超出了 64,000 个端口的限制。 将更多的公共 IP 地址添加到出站规则的前端即可实现该方案。 


将端口数指定为 0 即可恢复到[默认端口分配](load-balancer-outbound-connections.md#preallocatedports)。 前 50 个 VM 实例会获得 1024 个端口，而 51-100 个 VM 实例会获得 512 个端口，以此类推，直到最大实例数。 有关默认 SNAT 端口分配的详细信息，请参阅 [SNAT 端口分配表](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#preallocatedports)。


### <a name="scenario-3-enable-outbound-only"></a><a name="scenario3out"></a>方案3：仅启用出站


#### <a name="details"></a>详细信息


使用公共标准负载均衡器为一组 Vm 提供出站 NAT。 在此方案中，仅使用出站规则，而不配置任何其他规则。


> [!NOTE]
> **Azure 虚拟网络 NAT** 可以为虚拟机提供出站连接，无需使用负载均衡器。 有关详细信息，请参阅[什么是 Azure 虚拟网络 NAT？](../virtual-network/nat-overview.md)。

### <a name="scenario-4-outbound-nat-for-vms-only-no-inbound"></a><a name="scenario4out"></a>方案4：仅限 Vm 的出站 NAT (没有入站) 


> [!NOTE]
> **Azure 虚拟网络 NAT** 可以为虚拟机提供出站连接，无需使用负载均衡器。 有关详细信息，请参阅[什么是 Azure 虚拟网络 NAT？](../virtual-network/nat-overview.md)。

#### <a name="details"></a>详细信息


对于此方案： Azure 负载均衡器出站规则和虚拟网络 NAT 是可用于从虚拟网络传出的选项。


1. 创建公共 IP 或前缀。
2. 创建公共标准负载均衡器。 
3. 创建一个与专用于出站的公共 IP 或前缀关联的前端。
4. 为 VM 创建后端池。
5. 将 VM 置于后端池。
6. 配置启用出站 NAT 所需的出站规则。



使用前缀或公共 IP 来缩放 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources) 端口。 将出站连接的源添加到允许列表或拒绝列表。



### <a name="scenario-5-outbound-nat-for-internal-standard-load-balancer"></a><a name="scenario5out"></a>方案5：内部标准负载均衡器的出站 NAT


> [!NOTE]
> **Azure 虚拟网络 NAT** 可以利用内部标准负载均衡器为虚拟机提供出站连接。 有关详细信息，请参阅[什么是 Azure 虚拟网络 NAT？](../virtual-network/nat-overview.md)。

#### <a name="details"></a>详细信息


对于内部标准负载均衡器，出站连接不可用，除非已通过实例级公共 Ip 或虚拟网络 NAT 显式声明，或将后端池成员与仅出站负载平衡器配置相关联。 


有关详细信息，请参阅[仅出站的负载均衡器配置](https://docs.microsoft.com/azure/load-balancer/egress-only)。




### <a name="scenario-6-enable-both-tcp--udp-protocols-for-outbound-nat-with-a-public-standard-load-balancer"></a><a name="scenario6out"></a>方案6：启用使用公共标准负载均衡器的出站 NAT 的 TCP & UDP 协议


#### <a name="details"></a>详细信息


使用公共标准负载均衡器时，提供的自动出站 NAT 与负载均衡规则的传输协议相匹配。 


1. 在负载均衡规则中禁用出站 [SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#-sharing-ports-across-resources)。 
2. 在同一个负载均衡器上配置出站规则。
3. 重复使用 VM 已用的后端池。 
4. 指定“协议”：“所有”作为出站规则的一部分。 


只使用入站 NAT 规则时，不会提供出站 NAT。 


1. 将 VM 放入后端池。
2. 使用公共 IP 地址或公共 IP 前缀定义一个或多个前端 IP 配置 
3. 在同一个负载均衡器上配置出站规则。 
4. 指定“协议”：“所有”作为出站规则的一部分


## <a name="limitations"></a>限制

- 每个前端 IP 地址的最大可用临时端口数为 64,000。
- 可配置的出站空闲超时范围为 4 到 120 分钟（240 到 7200 秒）。
- 负载均衡器不支持将 ICMP 用于出站 NAT。
- 出站规则只能应用于 NIC 的主 IP 配置。  不能为 VM 或 NVA 的辅助 IP 创建出站规则。 支持多个 NIC。
- **可用性集中** 的所有虚拟机都必须添加到后端池以实现出站连接。 
- 必须将 **虚拟机规模集中** 的所有虚拟机添加到后端池以实现出站连接。

## <a name="next-steps"></a>后续步骤

- 了解有关[Azure 标准负载均衡器](load-balancer-overview.md)的详细信息
- 查看[有关 Azure 负载均衡器](load-balancer-faqs.md)的常见问题

