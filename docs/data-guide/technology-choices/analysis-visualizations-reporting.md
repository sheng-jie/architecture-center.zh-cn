---
title: 选择数据分析和报告技术
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 830c61bba64a6971c815330887e5cdcc4f2b5f56
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>在 Azure 中选择数据分析技术

大多数大数据解决方案的目标是通过分析和报告提供对数据的见解。 这可以包括预配置的报表和可视化效果或交互式数据探索。 

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>选择数据分析技术时有哪些选项？

Azure 中有多个用于分析、可视化和报告的选项，具体取决于你的需求：

- [Power BI](/power-bi/)
- [Jupyter 笔记本](https://jupyter.readthedocs.io/en/latest/index.html)
- [Zeppelin Notebook](https://zeppelin.apache.org/)
- [Microsoft Azure Notebook](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

[Power BI](/power-bi/) 是一个业务分析工具套件。 它可以连接到数百个数据源，并且可以用于临时分析。 请参阅[此列表](/power-bi/desktop-data-sources)中列出的当前可用的数据源。 使用 [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/) 将 Power BI 集成到自己的应用程序中，不需要任何额外的授权。

组织可以使用 Power BI 生成报表并将其发布到组织。 每个人都可以创建具有治理机制和[内置安全性](/power-bi/service-admin-power-bi-security)的个性化仪表板。 每当用户尝试访问要求进行身份验证的资源时，Power BI 都使用 [Azure Active Directory](/azure/active-directory/) (Azure AD) 对登录 Power BI 服务的用户进行身份验证，并使用 Power BI 登录凭据。

### <a name="jupyter-notebooks"></a>Jupyter 笔记本 

[Jupyter Notebook](https://jupyter.readthedocs.io/en/latest/index.html) 提供了一个基于浏览器的 shell，它允许数据科学家创建包含 Python、Scala 或 R 代码和标记文本的 *Notebook* 文件，这使得它成为通过在单个文档中共享和记录代码及结果来进行协作的有效方式。

HDInsight 群集的大多数变体（例如 Spark 或 Hadoop）都[预先配置了 Jupyter Notebook](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels)，用于与数据进行交互以及提交作业进行处理。 根据你使用的 HDInsight 群集的类型，将提供一个或多个用于解释和运行你的代码的内核。 例如，HDInsight 上的 Spark 群集提供了与 Spark 相关的内核，你可以从中进行选择以使用 Spark 引擎执行 Python 或 Scala 代码。

Jupyter Notebook 提供了一个很棒的环境，在使用 Power BI 之类的商业智能/报告工具构建更高级的可视化效果之前，可以在该环境中对数据进行分析、可视化和处理。

### <a name="zeppelin-notebooks"></a>Zeppelin Notebook

[Zeppelin Notebook](https://zeppelin.apache.org/) 是适用于基于浏览器的 shell 的另一个选项，其功能类似于 Jupyter。 某些 HDInsight 群集[预先配置了 Zeppelin Notebook](/azure/hdinsight/spark/apache-spark-zeppelin-notebook)。 但是，如果使用 [HDInsight 交互式查询](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) (Hive LLAP) 群集，则 [Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) 是当前唯一可以用来运行交互式 Hive 查询的 notebook。 另外，如果使用[已加入域的 HDInsight 群集](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)，若要分配不同的用户登录名来控制对 Notebook 和基础 Hive 表的访问，则 Zeppelin Notebook 是唯一可实现此目标的类型。

### <a name="microsoft-azure-notebooks"></a>Microsoft Azure Notebook

[Azure Notebook](https://notebooks.azure.com/) 是一个基于 Jupyter Notebook 的联机服务，它使得数据科学家能够在基于云的库中创建、运行和共享 Jupyter Notebook。 Azure Notebook 为 Python 2、Python 3、F# 和 R 提供了执行环境，并且提供了用来使数据可视化的几个图表库，例如 ggplot、matplotlib、bokeh 和 seaborn。

不同于在 HDInsight 群集上运行的 Jupyter Notebook（它们连接到群集的默认存储帐户），Azure Notebook 不提供任何数据。 你必须以各种方式[加载数据](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb)，例如，从联机源下载数据、与 Azure Blob 或表存储进行交互、连接到 SQL 数据库，或者使用 Azure 数据工厂的复制向导加载数据。

主要优势：

* 免费服务&mdash;不需要具有 Azure 订阅。
* 无需在本地安装 Jupyter 和提供支持的 R 或 Python 发行版&mdash;只需使用浏览器即可。
* 可以从任何设备管理自己的联机库并访问它们。
* 可以与协作者共享 Notebook。

注意事项：

* 无法在脱机时访问 Notebook。
* 免费 Notebook 服务的有限处理功能可能不足以训练大型或复杂模型。

## <a name="key-selection-criteria"></a>关键选择条件

若要缩小选择范围，请先回答以下问题：

- 是否需要连接到许多数据源，提供一个集中的位置来为分布在整个域中的数据创建报表？ 如果是，请选择一个允许连接到数百个数据源的选项。

- 是否要在外部网站或应用程序中嵌入动态可视化效果？ 如果是，请选择一个提供了嵌入功能的选项。

- 是否要在脱机状态下设计可视化效果和报表？ 如果是，请选择一个具有脱机功能的选项。

- 是否需要强大的处理能力来训练大型或复杂人工智能模型或使用非常大的数据集？ 如果是，请选择一个可以连接到大数据群集的选项。

## <a name="capability-matrix"></a>功能矩阵

以下各表汇总了功能上的关键差异。 

### <a name="general-capabilities"></a>常规功能

| | Power BI | Jupyter 笔记本 | Zeppelin Notebook | Microsoft Azure Notebook |
| --- | --- | --- | --- | --- |
| 连接到大数据群集以进行高级处理 | 是 | 是 | 是 | 否 |
| 托管服务 | 是 | 是 <sup>1</sup> | 是 <sup>1</sup> | 是 |
| 连接到数百个数据源 | 是 | 否 | 否 | 否 |
| 脱机功能 | 是 <sup>2</sup> | 否 | 否 | 否 |
| 嵌入功能 | 是 | 否 | 否 | 否 |
| 自动化数据刷新 | 是 | 否 | 否 | 否 |
| 对大量开源包的访问权限 | 否 | 是 <sup>3</sup> | 是 <sup>3</sup> | 是 <sup>4</sup> |
| 数据转换/清理选项 | [Power Query](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/)、R | 40 种语言，包括 Python、R、Julia 和 Scala | 20 多个解释器，包括 Python、JDBC 和 R | Python、F#、R |
| 定价 | 对于 Power BI Desktop（创作）免费，请参阅各个托管选项的[定价](https://powerbi.microsoft.com/pricing/) | 免费 | 免费 | 免费 |
| 多用户协作 | [是](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | 是（通过进行共享或使用诸如 [JupyterHub](https://github.com/jupyterhub/jupyterhub) 的多用户服务器） | 是 | 是（通过进行共享） |

[1] 当用作托管 HDInsight 群集的一部分时。

[2] 通过使用 Power BI Desktop。

[2] 可以在 [Maven 存储库](http://search.maven.org/)中搜索由社区贡献的包。

[3] 可以使用 pip 或 conda 安装 Python 包。 可以从 CRAN 或 GitHub 安装 R 包。 可以使用 [Paket 依存关系管理器](https://fsprojects.github.io/Paket/)通过 nuget.org 安装 F# 中的包。

