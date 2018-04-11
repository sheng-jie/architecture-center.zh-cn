---
title: 选择实时消息引入技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2e6578b779950b5ef11bda7b8ba1fb2e45e09f4e
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/23/2018
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>在 Azure 中选择实时消息引入技术

实时处理对实时捕获的数据流进行处理并且以最低延迟对其进行处理。 许多实时处理解决方案都需要一个消息引入存储来充当消息缓冲区，以及支持横向扩展处理、可靠传递和其他消息队列语义。 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>用于实时消息引入的选项有哪些？

- [Azure 事件中心](/azure/event-hubs/)
- [Azure IoT 中心](/azure/iot-hub/)
- [Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Azure 事件中心

[Azure 事件中心](/azure/event-hubs/)是高度可缩放的数据流式处理平台和事件引入服务，能够每秒接收和处理数百万事件。 事件中心可以处理和存储分布式软件和设备生成的事件、数据或遥测。 可以使用任何实时分析提供程序或批处理/存储适配器转换和存储发送到数据中心的数据。 事件中心以低延迟大规模提供发布-订阅功能，这使得它适用于大数据方案。

## <a name="azure-iot-hub"></a>Azure IoT 中心

[Azure IoT 中心](/azure/iot-hub/)是一项托管的服务，可在数百万个 IoT 设备和一个基于云的后端之间实现安全可靠的双向通信。

IoT 中心的功能包括：

* 多个针对设备到云和云到设备通信的选项。 这些选项包括单向消息传递、文件传输以及请求-答复方法。
* 将消息传送到其他 Azure 服务。
* 为设备元数据和已同步的状态信息提供可查询存储。
* 使用每个设备的安全密钥或 X.509 证书来实现安全的通信和访问控制。
* 监视设备连接和设备标识管理事件。

在消息引入方面，IoT 中心类似于事件中心。 但是，它专门设计用于管理 IoT 设备连接，而不仅仅是用于消息引入。 有关详细信息，请参阅 [Azure IoT 中心与 Azure 事件中心的比较](/azure/iot-hub/iot-hub-compare-event-hubs)。 

## <a name="kafka-on-hdinsight"></a>Kafka on HDInsight

[Apache Kafka](https://kafka.apache.org/) 是一个开源的分布式流式处理平台，可用于构建实时数据管道和流式处理应用程序。 Kafka 还提供了类似于消息队列的消息中转站，可在其中发布和订阅命名数据流。 它可水平缩放，具有容错能力，而且速度非常快。 [Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) 将 Kafka 提供为 Azure 中的一项可高度扩展且高度可用的托管服务。 

Kafka 的一些常见用例包括：

* **消息**。 由于 Kafka 支持发布-订阅消息模式，因此它经常用作消息中转站。
* **活动跟踪**。 由于 Kafka 提供了按顺序将记录记入日志的功能，因此它还可用来跟踪和重新创建活动。
* **聚合**。 使用流处理可从不同的流中聚合信息，将信息合并和集中到运营数据中。
* **转换**。 使用流处理可将多个输入主题中的数据合并到一个或多个输出主题中，丰富其内容。

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- IoT 设备与 Azure 之间是否需要进行双向通信？ 如果是，请选择 IoT 中心。

- 是否需要管理对各个设备的访问权限，并且必须能够撤销对特定设备的访问权限？ 如果是，请选择 IoT 中心。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。 

| | IoT 中心 | 事件中心 | Kafka on HDInsight |
| --- | --- | --- | --- |
| 云到设备的通信 | 是 | 否 | 否 |
| 设备启动的文件上传 | 是 | 否 | 否 |
| 设备状态信息 | [设备孪生](/azure/iot-hub/iot-hub-devguide-device-twins) | 否 | 否 |
| 协议支持 | MQTT、AMQP、HTTPS <sup>1</sup> | AMQP、HTTPS | [Kafka 协议](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| “安全” | 每设备标识；可撤销的访问控制权限。 | 共享访问策略；通过发布者策略实现有限的撤销。 | 使用 SASL 的身份验证；可插拔身份验证；与受支持的外部身份验证服务集成。 |

[1] 还可以使用 [Azure IoT 协议网关](/azure/iot-hub/iot-hub-protocol-gateway)作为自定义网关，来为 IoT 中心启用协议自适应。

有关详细信息，请参阅 [Azure IoT 中心与 Azure 事件中心的比较](/azure/iot-hub/iot-hub-compare-event-hubs)。
