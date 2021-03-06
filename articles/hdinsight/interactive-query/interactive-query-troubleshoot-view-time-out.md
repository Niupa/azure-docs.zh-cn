---
title: 从查询结果生成 Apache Hive 视图超时 - Azure HDInsight
description: 在 Azure HDInsight 中提取查询结果时，Apache Hive 视图超时
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/30/2019
ms.openlocfilehash: c642760a5f6bfa7e59d42237e1583617e322ece3
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/03/2020
ms.locfileid: "93288824"
---
# <a name="scenario-apache-hive-view-times-out-when-fetching-a-query-result-in-azure-hdinsight"></a>方案：在 Azure HDInsight 中提取查询结果时，Apache Hive 视图超时

本文介绍在 Azure HDInsight 群集中使用交互式查询组件时出现的问题的故障排除步骤和可能的解决方案。

## <a name="issue"></a>问题

从 Apache Hive 视图运行某些查询时，可能会遇到以下错误：

```
result fetch timed out
java.util.concurrent.TimeoutException: deadline passed
```

## <a name="cause"></a>原因

Hive 视图默认超时值可能不适用于正在运行的查询。 指定的时间段对于 Hive 视图而言太短，无法提取查询结果。

## <a name="resolution"></a>解决方法

通过在 `/etc/ambari-server/conf/ambari.properties` 中设置以下属性来增加 Apache Ambari Hive 视图超时。

```
views.ambari.request.read.timeout.millis=300000
views.request.read.timeout.millis=300000
views.ambari.hive.<HIVE_VIEW_INSTANCE_NAME>.result.fetch.timeout=300000
```

`HIVE_VIEW_INSTANCE_NAME` 的值可在 Hive 视图 URL 的末尾找到。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]