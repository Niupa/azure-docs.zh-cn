---
title: 训练模型：模块参考
titleSuffix: Azure Machine Learning
description: 了解如何使用 Azure 机器学习中的**训练模型**模块来训练分类或回归模型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/11/2020
ms.openlocfilehash: e3080836e8b9ed38e99c691c66e71a4620829c90
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "90890207"
---
# <a name="train-model-module"></a>“定型模型”模块

本文介绍 Azure 机器学习设计器中的模块。

使用此模块，可以训练分类或回归模型。 训练在你定义模型并设置其参数后进行，并且需要带标记的数据。 你还可以使用**训练模型**来使用新数据重新训练现有模型。 

## <a name="how-the-training-process-works"></a>训练过程的工作原理

在 Azure 机器学习中，创建和使用机器学习模型通常是一个三步过程。 

1. 可以通过选择特定类型的算法并定义其参数或超参数来配置模型。 选择以下任一模型类型： 

    + **分类**模型，基于神经网络、决策树，以及决策林和其他算法。
    + **回归**模型，这可能包括标准线性回归，也可能使用其他算法（包括神经网络和 Bayesian 回归）。  

2. 提供一个带标记且其数据与算法兼容的数据集。 将数据和模型都连接到**训练模型**。

    训练产生的是特定的二元格式 iLearner，它封装了从数据中获知的统计模式。 你无法直接修改或读取此格式；但是，其他模块可以使用此训练后的模型。 
    
    你还可以查看模型的属性。 有关详细信息，请参阅“结果”部分。

3. 在训练完成后，使用训练后的模型与[评分模型](./score-model.md)之一来基于新数据进行预测。

## <a name="how-to-use-train-model"></a>如何使用训练模型 
  
1.  在 Azure 机器学习中，配置一个分类模型或回归模型。
    
2. 将“训练模型”模块添加到管道。  可以在“机器学习”类别下找到此模块。 展开“训练”，然后将“训练模型”模块拖到你的管道中。
  
3.  在左侧输入中，附加未训练的模式。 将训练数据集附加到**训练模型**的右侧输入。

    训练数据集必须包含一个标签列。 不带标签的任何行都将被忽略。
  
4.  对于“标签列”，单击模块右侧面板中的“编辑列”，然后选择包含模型可用于训练的结果的单个列。
  
    - 对于分类问题，标签列必须包含**分类**值或**离散**值。 可能的一些示例如下：“是/否”评级、疾病分类代码或名称，或收入组。  如果你选择了一个非分类列，则模块在训练期间将返回错误。
  
    -   对于回归问题，标签列必须包含表示响应变量的**数字**数据。 理想情况下，数字数据表示连续标度。 
    
    可能的示例有信用风险分数、硬盘驱动器的预计故障时间，或者在给定的日期或时间内对某个呼叫中心的呼叫预测数。  如果未选择数字列，则可能会出现错误。
  
    -   如果未指定要使用的标签列，则 Azure 机器学习将尝试使用数据集的元数据推断哪个列是相应的标签列。 如果它选择了错误的列，请使用列选择器来更正它。
  
    > [!TIP] 
    > 如果使用列选择器时遇到问题，请参阅[选择数据集中的列](./select-columns-in-dataset.md)一文中的提示。 该文章介绍了使用 **WITH RULES** 和 **BY NAME** 选项的一些常见方案和提示。
  
5.  提交管道。 如果有大量数据，则可能需要一段时间。

## <a name="results"></a>结果

在对模型进行训练后：


+ 若要在其他管道中使用该模型，请选择该模块，然后在右侧面板的“输出”选项卡下选择“注册数据集”图标。 可以在“数据集”下的模块调色板中访问已保存的模型。

+ 若要在预测新值时使用模型，请将其连接到[评分模型](./score-model.md)模块以及新的输入数据。


## <a name="next-steps"></a>后续步骤

请参阅 Azure 机器学习的[可用模块集](module-reference.md)。 