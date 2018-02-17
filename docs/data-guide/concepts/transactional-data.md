---
title: "事务数据"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 6277badde1845c42858e69f6c8ecc73331e7a945
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="transactional-data"></a>事务数据

事务数据是指用于跟踪与组织活动相关的交互的信息。 这些交互通常是业务事务，如从客户收到的付款、对供应商进行的付款、从库存移动的产品、接受的订单或交付的服务。 表示事务本身的事务事件通常包含时间维度、一些数值和对其他数据的引用。 

事务通常需要*原子性*和*一致性*。 原子性意味着整个事务始终作为一个工作单元成功或失败，永远不会处于半完成状态。 如果无法完成某个事务，数据库系统必须回退任何已作为该事务的一部分完成的步骤。 在传统 RDBMS 中，如果某个事务无法完成，这种回退会自动发生。 一致性意味着事务始终让数据处于有效状态。 （这些是原子性和一致性的非常不正式的说明。 这些属性有更正式的定义，如 [ACID](https://en.wikipedia.org/wiki/ACID)。）

事务数据库可以使用各种锁定策略（如悲观锁定）支持事务的强一致性，以确保所有用户和进程的所有数据在企业的上下文中具有强一致性。 

使用事务数据的最常见部署体系结构是 3 层体系结构中的数据存储层。 3 层体系结构通常包含表示层、业务逻辑层和数据存储层。 相关的部署体系结构是 [N 层](/azure/architecture/guide/architecture-styles/n-tier)体系结构，它可能具有多个处理业务逻辑的中间层。

![3 层应用程序的示例](./images/three-tier-application.png)

## <a name="typical-traits-of-transactional-data"></a>事务数据的典型特征

事务数据往往具有以下特征：

| 要求 | 说明 |
| --- | --- |
| 规范化 | 高度规范化 |
| 架构 | 写入架构，强制执行|
| 一致性 | 强一致性，保证 ACID |
| 完整性 | 高完整性 |
| 使用事务 | 是 |
| 锁定策略 | 悲观或乐观|
| 可更新 | 是 |
| 可追加 | 是 |
| 工作负载 | 大量写入，适度读取 |
| 索引 | 主要和辅助索引 |
| 基准大小 | 中小型 |
| 模型 | 关系 |
| 数据形状 | 表格 |
| 查询灵活性 | 高度灵活 |
| 缩放 | 小型 (MB) 到大型（几 TB） | 

## <a name="see-also"></a>另请参阅

[联机事务处理](../scenarios/online-transaction-processing.md)