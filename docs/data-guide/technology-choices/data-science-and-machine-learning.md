---
title: 选择机器学习技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 995349c795066ec3067b20ad2615e40b0fb152db
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288929"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a>在 Azure 中选择机器学习技术

数据科学和机器学习是数据科学家经常采用的工作负荷。 它需要专业化的工具，其中的许多工具专门针对数据科学家必须执行的交互式数据探索和建模任务而设计。

机器学习解决方案以迭代方式构建，包括两个不同的阶段：
* 数据准备和建模。
* 预测服务的部署和使用。

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>用于数据准备和建模的工具与服务
数据科学家通常偏好使用以 Python 或 R 编写的自定义代码来处理数据。此代码通常以交互方式运行，数据科学家可以用它来查询和探索数据、生成可视化效果和统计信息，以帮助确定数据之间的关系。 数据科学家可以使用适用于 R 和 Python 的许多交互式环境。 特别受欢迎的是 **Jupyter Notebook**，它提供基于浏览器的 shell，可让数据科学家创建包含 R 或 Python 代码和 Markdown 文本的笔记本文件。 这可以在单个文档中共享和阐述代码与结果，从而展开有效的协作。

其他常用的工具包括：
* **Spyder**：Anaconda Python 分发版随附的适用于 Python 的交互式开发环境 (IDE)。
* **R Studio**：适用于 R 编程语言的 IDE。
* **Visual Studio Code**：轻量、跨平台的编码环境，它支持 Python，以及机器学习和 AI 开发中常用的框架。

除了这些工具以外，数据科学家还可以利用 Azure 服务来简化代码和模型管理。

### <a name="azure-notebooks"></a>Azure Notebook
Azure Notebook 是一个在线 Jupyter Notebook 服务，可让数据科学家在基于云的库中创建、运行和共享 Jupyter Notebook。

主要优势：

* 免费服务 &mdash; 不需要 Azure 订阅。
* 无需在本地安装 Jupyter 和提供支持的 R 或 Python 发行版 &mdash; 只需使用浏览器即可。
* 可以从任何设备管理和访问自己的联机库。
* 可与协作者共享 Notebook。

注意事项：

* 无法在脱机时访问 Notebook。
* 免费 Notebook 服务的有限处理功能可能不足以训练大型或复杂模型。

### <a name="data-science-virtual-machine"></a>数据科学虚拟机
数据科学虚拟机是一个 Azure 虚拟机映像，其中包含数据科学家经常使用的工具和框架，例如 R、Python、Jupyter Notebook、Visual Studio Code，以及用于机器学习建模的库（如 Microsoft 认知工具包）。 这些工具的安装可以比较复杂且耗时，并且存在各种相互依赖关系，导致版本管理问题。 预装的映像可以减少数据科学家花费在排查环境问题上的时间，让他们专注于执行数据探索和建模任务。

主要优势：
* 减少安装、管理数据科学工具和框架及其故障排除的时间。
* 包含所有常用工具和框架的最新版本。
* 虚拟机选项包括高度可缩放的映像和 GPU 功能用于密集型数据建模。

注意事项：
* 脱机时无法访问虚拟机。
* 运行虚拟机会产生 Azure 费用，因此请注意，只在有需要时才运行。

### <a name="azure-machine-learning"></a>Azure 机器学习

Azure 机器学习是用于管理机器学习试验和模型的基于云的服务。 它包含一个试验服务用于跟踪数据准备和模型训练脚本，并维护所有执行的历史记录，以便可以在不同的迭代中比较模型性能。 名为 Azure Machine Learning Workbench 的跨平台客户端工具提供一个中心界面用于脚本管理和历史记录，同时可让数据科学家在所选的工具（例如 Jupyter Notebook 或 Visual Studio Code）中创建脚本。

在 Azure Machine Learning Workbench 中，可以使用交互式数据准备工具来简化常见的数据转换任务，并可以将脚本执行环境配置为在可缩放的 Docker 容器或 Spark 中本地运行模型训练脚本。

准备好部署模型时，可以使用 Workbench 环境打包模型，并将其作为 Web 服务部署到 Docker 容器、Azure HDinsight 上的 Spark、Microsoft Machine Learning Server 或 SQL Server。 然后，可以使用 Azure 机器学习模型管理服务来跟踪和管理云中、边缘设备或整个企业中的模型部署。

主要优势：

* 集中管理脚本和运行历史记录，轻松比较模型版本。
* 通过可视编辑器进行交互式数据转换。
* 轻松部署和管理云中或边缘设备上的模型。

注意事项：
* 需要对模型管理模型和 Workbench 工具环境有一定的了解。

### <a name="azure-batch-ai"></a>Azure Batch AI

使用 Azure Batch AI 可以并行运行机器学习试验，并使用 GPU 在整个虚拟机群集中执行大规模的模型训练。 通过 Batch AI 训练，可以使用认知工具包、Caffe、Chainer 和 TensorFlow 等框架在群集化 GPU 中扩展深度学习作业。 

Azure 机器学习模型管理可用于从 Batch AI 训练中提取模型，以便对其进行部署、管理和监视。 

### <a name="azure-machine-learning-studio"></a>Azure 机器学习工作室

Azure 机器学习工作室是一个基于云的可视开发环境，用于创建数据试验、训练机器学习模型，并将其作为 Web 服务发布到 Azure 中。 数据科学家和高级用户可以使用其可视化拖放界面快速创建机器学习解决方案，同时支持自定义 R 和 Python 逻辑，以及用于机器学习建模任务的各种成熟统计算法和技术，并且原生支持 Jupyter Notebook。

主要优势：

* 交互式可视化界面可让用户以极少量的代码实现机器学习建模。
* 内置 Jupyter Notebook 用于数据探索。
* 将训练的模型作为 Azure Web 服务直接部署。

注意事项：

* 可伸缩性有限。 训练数据集的最大大小为 10 GB。
* 只能联机使用。 不支持脱机开发环境。

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>肜于部署机器学习模型的工具和服务

在据科学家创建机器学习模型后，你通常需要部署它，并从应用程序或其他数据流使用它。 机器学习模型有许多潜在的部署目标。

### <a name="spark-on-azure-hdinsight"></a>Azure HDInsight 上的 Spark

Apache Spark 包括 Spark MLlib（机器学习模型的框架和库）。 适用于 Spark 的 Microsoft 机器学习库 (MMLSpark) 还针对 Spark 中的预测模型提供深度学习算法支持。

主要优势：

* Spark 是一个分布式平台，为大规模机器学习流程提供高度可伸缩性。
* 可以通过 Azure Machine Learning Workbench 将模型直接部署到 HDinsight 中的 Spark，然后使用 Azure 机器学习模型管理服务来管理模型。

注意事项：

* 在 HDinsght 群集中运行 Spark 全程都会产生费用。 如果只是偶尔才使用机器学习服务，这可能会造成不必要的费用。

### <a name="web-service-in-a-container"></a>容器中的 Web 服务

可将机器学习模型作为 Python Web 服务部署在 Docker 容器中。 可将模型部署到 Azure 或边缘设备，然后在其中使用它来处理数据。

主要优势：

* 容器是打包和部署服务的简便方式，通常也比较经济高效。
* 将模型部署到边缘设备可以使预测逻辑更靠近数据。
* 可以直接从 Azure Machine Learning Workbench 部署到容器。

注意事项：

* 此部署模型基于 Docker 容器，因此，以这种方式部署 Web 服务之前，请先熟悉此技术。

### <a name="microsoft-machine-learning-server"></a>Microsoft 机器学习服务器

Machine Learning Server（以前称为 Microsoft R Server）是适用于 R 和 Python 代码的可缩放平台，专门针对机器学习方案而设计。

主要优势：

* 高度可伸缩性。
* 直接从 Azure Machine Learning Workbench 部署。

注意事项：

* 需要在企业中部署和管理 Machine Learning Server。

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

Microsoft SQL Server 原生支持 R 和 Python，可用于在数据库中作为 Transact-SQL 函数封装以这些语言构建的机器学习模型。

主要优势：

* 在数据库函数中封装预测逻辑可以轻松加入数据层逻辑。

注意事项：

* 采用 SQL Server 数据库作为应用程序的数据层。

### <a name="azure-machine-learning-web-service"></a>Azure 机器学习 Web 服务

使用 Azure 机器学习工作室创建机器学习模型时，可将其部署为 Web 服务。 然后，可以在支持 HTTP 通信的任何客户端应用程序中通过 REST 接口来使用该模型。

主要优势：

* 易于开发和部署。
* 包含基本监视指标的 Web 服务管理门户。
* 原生支持从 Azure Data Lake Analytics、Azure 数据工厂和 Azure 流分析调用 Azure 机器学习 Web 服务。

注意事项：

* 仅适用于使用 Azure 机器学习工作室构建的模型。
* 仅限基于 Web 的访问，训练的模型无法在本地或脱机运行。

