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
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a><span data-ttu-id="a3756-102">在 Azure 中选择机器学习技术</span><span class="sxs-lookup"><span data-stu-id="a3756-102">Choosing a machine learning technology in Azure</span></span>

<span data-ttu-id="a3756-103">数据科学和机器学习是数据科学家经常采用的工作负荷。</span><span class="sxs-lookup"><span data-stu-id="a3756-103">Data science and machine learning is a workload that is usually undertaken by data scientists.</span></span> <span data-ttu-id="a3756-104">它需要专业化的工具，其中的许多工具专门针对数据科学家必须执行的交互式数据探索和建模任务而设计。</span><span class="sxs-lookup"><span data-stu-id="a3756-104">It requires specialist tools, many of which are designed specifically for the type of interactive data exploration and modeling tasks that a data scientist must perform.</span></span>

<span data-ttu-id="a3756-105">机器学习解决方案以迭代方式构建，包括两个不同的阶段：</span><span class="sxs-lookup"><span data-stu-id="a3756-105">Machine learning solutions are built iteratively, and have two distinct phases:</span></span>
* <span data-ttu-id="a3756-106">数据准备和建模。</span><span class="sxs-lookup"><span data-stu-id="a3756-106">Data preparation and modeling.</span></span>
* <span data-ttu-id="a3756-107">预测服务的部署和使用。</span><span class="sxs-lookup"><span data-stu-id="a3756-107">Deployment and consumption of predictive services.</span></span>

## <a name="tools-and-services-for-data-preparation-and-modeling"></a><span data-ttu-id="a3756-108">用于数据准备和建模的工具与服务</span><span class="sxs-lookup"><span data-stu-id="a3756-108">Tools and services for data preparation and modeling</span></span>
<span data-ttu-id="a3756-109">数据科学家通常偏好使用以 Python 或 R 编写的自定义代码来处理数据。此代码通常以交互方式运行，数据科学家可以用它来查询和探索数据、生成可视化效果和统计信息，以帮助确定数据之间的关系。</span><span class="sxs-lookup"><span data-stu-id="a3756-109">Data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships with it.</span></span> <span data-ttu-id="a3756-110">数据科学家可以使用适用于 R 和 Python 的许多交互式环境。</span><span class="sxs-lookup"><span data-stu-id="a3756-110">There are many interactive environments for R and Python that data scientists can use.</span></span> <span data-ttu-id="a3756-111">特别受欢迎的是 **Jupyter Notebook**，它提供基于浏览器的 shell，可让数据科学家创建包含 R 或 Python 代码和 Markdown 文本的笔记本文件。</span><span class="sxs-lookup"><span data-stu-id="a3756-111">A particular favorite is **Jupyter Notebooks** that provides a browser-based shell that enables data scientists to create *notebook* files that contain R or Python code and markdown text.</span></span> <span data-ttu-id="a3756-112">这可以在单个文档中共享和阐述代码与结果，从而展开有效的协作。</span><span class="sxs-lookup"><span data-stu-id="a3756-112">This is an effective way to collaborate by sharing and documenting code and results in a single document.</span></span>

<span data-ttu-id="a3756-113">其他常用的工具包括：</span><span class="sxs-lookup"><span data-stu-id="a3756-113">Other commonly used tools include:</span></span>
* <span data-ttu-id="a3756-114">**Spyder**：Anaconda Python 分发版随附的适用于 Python 的交互式开发环境 (IDE)。</span><span class="sxs-lookup"><span data-stu-id="a3756-114">**Spyder**: The interactive development environment (IDE) for Python provided with the Anaconda Python distribution.</span></span>
* <span data-ttu-id="a3756-115">**R Studio**：适用于 R 编程语言的 IDE。</span><span class="sxs-lookup"><span data-stu-id="a3756-115">**R Studio**: An IDE for the R programming language.</span></span>
* <span data-ttu-id="a3756-116">**Visual Studio Code**：轻量、跨平台的编码环境，它支持 Python，以及机器学习和 AI 开发中常用的框架。</span><span class="sxs-lookup"><span data-stu-id="a3756-116">**Visual Studio Code**: A lightweight, cross-platform coding environment that supports Python as well as commonly used frameworks for machine learning and AI development.</span></span>

<span data-ttu-id="a3756-117">除了这些工具以外，数据科学家还可以利用 Azure 服务来简化代码和模型管理。</span><span class="sxs-lookup"><span data-stu-id="a3756-117">In addition to these tools, data scientists can leverage Azure services to simplify code and model management.</span></span>

### <a name="azure-notebooks"></a><span data-ttu-id="a3756-118">Azure Notebook</span><span class="sxs-lookup"><span data-stu-id="a3756-118">Azure Notebooks</span></span>
<span data-ttu-id="a3756-119">Azure Notebook 是一个在线 Jupyter Notebook 服务，可让数据科学家在基于云的库中创建、运行和共享 Jupyter Notebook。</span><span class="sxs-lookup"><span data-stu-id="a3756-119">Azure Notebooks is an online Jupyter Notebooks service that enables data scientists to create, run, and share Jupyter Notebooks in cloud-based libraries.</span></span>

<span data-ttu-id="a3756-120">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-120">Key benefits:</span></span>

* <span data-ttu-id="a3756-121">免费服务 &mdash; 不需要 Azure 订阅。</span><span class="sxs-lookup"><span data-stu-id="a3756-121">Free service&mdash;no Azure subscription required.</span></span>
* <span data-ttu-id="a3756-122">无需在本地安装 Jupyter 和提供支持的 R 或 Python 发行版 &mdash; 只需使用浏览器即可。</span><span class="sxs-lookup"><span data-stu-id="a3756-122">No need to install Jupyter and the supporting R or Python distributions locally&mdash;just use a browser.</span></span>
* <span data-ttu-id="a3756-123">可以从任何设备管理和访问自己的联机库。</span><span class="sxs-lookup"><span data-stu-id="a3756-123">Manage your own online libraries and access them from any device.</span></span>
* <span data-ttu-id="a3756-124">可与协作者共享 Notebook。</span><span class="sxs-lookup"><span data-stu-id="a3756-124">Share your notebooks with collaborators.</span></span>

<span data-ttu-id="a3756-125">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-125">Considerations:</span></span>

* <span data-ttu-id="a3756-126">无法在脱机时访问 Notebook。</span><span class="sxs-lookup"><span data-stu-id="a3756-126">You will be unable to access your notebooks when offline.</span></span>
* <span data-ttu-id="a3756-127">免费 Notebook 服务的有限处理功能可能不足以训练大型或复杂模型。</span><span class="sxs-lookup"><span data-stu-id="a3756-127">Limited processing capabilities of the free notebook service may not be enough to train large or complex models.</span></span>

### <a name="data-science-virtual-machine"></a><span data-ttu-id="a3756-128">数据科学虚拟机</span><span class="sxs-lookup"><span data-stu-id="a3756-128">Data science virtual machine</span></span>
<span data-ttu-id="a3756-129">数据科学虚拟机是一个 Azure 虚拟机映像，其中包含数据科学家经常使用的工具和框架，例如 R、Python、Jupyter Notebook、Visual Studio Code，以及用于机器学习建模的库（如 Microsoft 认知工具包）。</span><span class="sxs-lookup"><span data-stu-id="a3756-129">The data science virtual machine is an Azure virtual machine image that includes the tools and frameworks commonly used by data scientists, including R, Python, Jupyter Notebooks, Visual Studio Code, and libraries for machine learning modeling such as the Microsoft Cognitive Toolkit.</span></span> <span data-ttu-id="a3756-130">这些工具的安装可以比较复杂且耗时，并且存在各种相互依赖关系，导致版本管理问题。</span><span class="sxs-lookup"><span data-stu-id="a3756-130">These tools can be complex and time consuming to install, and contain many interdependencies that often lead to version management issues.</span></span> <span data-ttu-id="a3756-131">预装的映像可以减少数据科学家花费在排查环境问题上的时间，让他们专注于执行数据探索和建模任务。</span><span class="sxs-lookup"><span data-stu-id="a3756-131">Having a preinstalled image can reduce the time data scientists spend troubleshooting environment issues, allowing them to focus on the data exploration and modeling tasks they need to perform.</span></span>

<span data-ttu-id="a3756-132">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-132">Key benefits:</span></span>
* <span data-ttu-id="a3756-133">减少安装、管理数据科学工具和框架及其故障排除的时间。</span><span class="sxs-lookup"><span data-stu-id="a3756-133">Reduced time to install, manage, and troubleshoot data science tools and frameworks.</span></span>
* <span data-ttu-id="a3756-134">包含所有常用工具和框架的最新版本。</span><span class="sxs-lookup"><span data-stu-id="a3756-134">The latest versions of all commonly used tools and frameworks are included.</span></span>
* <span data-ttu-id="a3756-135">虚拟机选项包括高度可缩放的映像和 GPU 功能用于密集型数据建模。</span><span class="sxs-lookup"><span data-stu-id="a3756-135">Virtual machine options include highly scalable images with GPU capabilities for intensive data modeling.</span></span>

<span data-ttu-id="a3756-136">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-136">Considerations:</span></span>
* <span data-ttu-id="a3756-137">脱机时无法访问虚拟机。</span><span class="sxs-lookup"><span data-stu-id="a3756-137">The virtual machine cannot be accessed when offline.</span></span>
* <span data-ttu-id="a3756-138">运行虚拟机会产生 Azure 费用，因此请注意，只在有需要时才运行。</span><span class="sxs-lookup"><span data-stu-id="a3756-138">Running a virtual machine incurs Azure charges, so you must be careful to have it running only when required.</span></span>

### <a name="azure-machine-learning"></a><span data-ttu-id="a3756-139">Azure 机器学习</span><span class="sxs-lookup"><span data-stu-id="a3756-139">Azure Machine Learning</span></span>

<span data-ttu-id="a3756-140">Azure 机器学习是用于管理机器学习试验和模型的基于云的服务。</span><span class="sxs-lookup"><span data-stu-id="a3756-140">Azure Machine Learning is a cloud-based service for managing machine learning experiments and models.</span></span> <span data-ttu-id="a3756-141">它包含一个试验服务用于跟踪数据准备和模型训练脚本，并维护所有执行的历史记录，以便可以在不同的迭代中比较模型性能。</span><span class="sxs-lookup"><span data-stu-id="a3756-141">It includes an experimentation service that tracks data preparation and modeling training scripts, maintaining a history of all executions so you can compare model performance across iterations.</span></span> <span data-ttu-id="a3756-142">名为 Azure Machine Learning Workbench 的跨平台客户端工具提供一个中心界面用于脚本管理和历史记录，同时可让数据科学家在所选的工具（例如 Jupyter Notebook 或 Visual Studio Code）中创建脚本。</span><span class="sxs-lookup"><span data-stu-id="a3756-142">A cross-platform client tool named Azure Machine Learning Workbench provides a central interface for script management and history, while still enabling data scientists to create scripts in their tool of choice, such as Jupyter Notebooks or Visual Studio Code.</span></span>

<span data-ttu-id="a3756-143">在 Azure Machine Learning Workbench 中，可以使用交互式数据准备工具来简化常见的数据转换任务，并可以将脚本执行环境配置为在可缩放的 Docker 容器或 Spark 中本地运行模型训练脚本。</span><span class="sxs-lookup"><span data-stu-id="a3756-143">From Azure Machine Learning Workbench, you can use the interactive data preparation tools to simplify common data transformation tasks, and you can configure the script execution environment to run model training scripts locally, in a scalable Docker container, or in Spark.</span></span>

<span data-ttu-id="a3756-144">准备好部署模型时，可以使用 Workbench 环境打包模型，并将其作为 Web 服务部署到 Docker 容器、Azure HDinsight 上的 Spark、Microsoft Machine Learning Server 或 SQL Server。</span><span class="sxs-lookup"><span data-stu-id="a3756-144">When you are ready to deploy your model, use the Workbench environment to package the model and deploy it as a web service to a Docker container, Spark on Azure HDinsight, Microsoft Machine Learning Server, or SQL Server.</span></span> <span data-ttu-id="a3756-145">然后，可以使用 Azure 机器学习模型管理服务来跟踪和管理云中、边缘设备或整个企业中的模型部署。</span><span class="sxs-lookup"><span data-stu-id="a3756-145">The Azure Machine Learning Model Management service then enables you to track and manage model deployments in the cloud, on edge devices, or across the enterprise.</span></span>

<span data-ttu-id="a3756-146">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-146">Key benefits:</span></span>

* <span data-ttu-id="a3756-147">集中管理脚本和运行历史记录，轻松比较模型版本。</span><span class="sxs-lookup"><span data-stu-id="a3756-147">Central management of scripts and run history, making it easy to compare model versions.</span></span>
* <span data-ttu-id="a3756-148">通过可视编辑器进行交互式数据转换。</span><span class="sxs-lookup"><span data-stu-id="a3756-148">Interactive data transformation through a visual editor.</span></span>
* <span data-ttu-id="a3756-149">轻松部署和管理云中或边缘设备上的模型。</span><span class="sxs-lookup"><span data-stu-id="a3756-149">Easy deployment and management of models to the cloud or edge devices.</span></span>

<span data-ttu-id="a3756-150">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-150">Considerations:</span></span>
* <span data-ttu-id="a3756-151">需要对模型管理模型和 Workbench 工具环境有一定的了解。</span><span class="sxs-lookup"><span data-stu-id="a3756-151">Requires some familiarity with the model management model and Workbench tool environment.</span></span>

### <a name="azure-batch-ai"></a><span data-ttu-id="a3756-152">Azure Batch AI</span><span class="sxs-lookup"><span data-stu-id="a3756-152">Azure Batch AI</span></span>

<span data-ttu-id="a3756-153">使用 Azure Batch AI 可以并行运行机器学习试验，并使用 GPU 在整个虚拟机群集中执行大规模的模型训练。</span><span class="sxs-lookup"><span data-stu-id="a3756-153">Azure Batch AI enables you to run your machine learning experiments in parallel, and perform model training at scale across a cluster of virtual machines with GPUs.</span></span> <span data-ttu-id="a3756-154">通过 Batch AI 训练，可以使用认知工具包、Caffe、Chainer 和 TensorFlow 等框架在群集化 GPU 中扩展深度学习作业。</span><span class="sxs-lookup"><span data-stu-id="a3756-154">Batch AI training enables you to scale out deep learning jobs across clustered GPUs, using frameworks such as Cognitive Toolkit, Caffe, Chainer, and TensorFlow.</span></span> 

<span data-ttu-id="a3756-155">Azure 机器学习模型管理可用于从 Batch AI 训练中提取模型，以便对其进行部署、管理和监视。</span><span class="sxs-lookup"><span data-stu-id="a3756-155">Azure Machine Learning Model Management can be used to take models from Batch AI training to deploy, manage, and monitor them.</span></span> 

### <a name="azure-machine-learning-studio"></a><span data-ttu-id="a3756-156">Azure 机器学习工作室</span><span class="sxs-lookup"><span data-stu-id="a3756-156">Azure Machine Learning Studio</span></span>

<span data-ttu-id="a3756-157">Azure 机器学习工作室是一个基于云的可视开发环境，用于创建数据试验、训练机器学习模型，并将其作为 Web 服务发布到 Azure 中。</span><span class="sxs-lookup"><span data-stu-id="a3756-157">Azure Machine Learning Studio is a cloud-based, visual development environment for creating data experiments, training machine learning models, and publishing them as web services in Azure.</span></span> <span data-ttu-id="a3756-158">数据科学家和高级用户可以使用其可视化拖放界面快速创建机器学习解决方案，同时支持自定义 R 和 Python 逻辑，以及用于机器学习建模任务的各种成熟统计算法和技术，并且原生支持 Jupyter Notebook。</span><span class="sxs-lookup"><span data-stu-id="a3756-158">Its visual drag-and-drop interface lets data scientists and power users create machine learning solutions quickly, while supporting custom R and Python logic, a wide range of established statistical algorithms and techniques for machine learning modeling tasks, and built-in support for Jupyter Notebooks.</span></span>

<span data-ttu-id="a3756-159">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-159">Key benefits:</span></span>

* <span data-ttu-id="a3756-160">交互式可视化界面可让用户以极少量的代码实现机器学习建模。</span><span class="sxs-lookup"><span data-stu-id="a3756-160">Interactive visual interface enables machine learning modeling with minimal code.</span></span>
* <span data-ttu-id="a3756-161">内置 Jupyter Notebook 用于数据探索。</span><span class="sxs-lookup"><span data-stu-id="a3756-161">Built-in Jupyter Notebooks for data exploration.</span></span>
* <span data-ttu-id="a3756-162">将训练的模型作为 Azure Web 服务直接部署。</span><span class="sxs-lookup"><span data-stu-id="a3756-162">Direct deployment of trained models as Azure web services.</span></span>

<span data-ttu-id="a3756-163">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-163">Considerations:</span></span>

* <span data-ttu-id="a3756-164">可伸缩性有限。</span><span class="sxs-lookup"><span data-stu-id="a3756-164">Limited scalability.</span></span> <span data-ttu-id="a3756-165">训练数据集的最大大小为 10 GB。</span><span class="sxs-lookup"><span data-stu-id="a3756-165">The maximum size of a training dataset is 10 GB.</span></span>
* <span data-ttu-id="a3756-166">只能联机使用。</span><span class="sxs-lookup"><span data-stu-id="a3756-166">Online only.</span></span> <span data-ttu-id="a3756-167">不支持脱机开发环境。</span><span class="sxs-lookup"><span data-stu-id="a3756-167">No offline development environment.</span></span>

## <a name="tools-and-services-for-deploying-machine-learning-models"></a><span data-ttu-id="a3756-168">肜于部署机器学习模型的工具和服务</span><span class="sxs-lookup"><span data-stu-id="a3756-168">Tools and services for deploying machine learning models</span></span>

<span data-ttu-id="a3756-169">在据科学家创建机器学习模型后，你通常需要部署它，并从应用程序或其他数据流使用它。</span><span class="sxs-lookup"><span data-stu-id="a3756-169">After a data scientist has created a machine learning model, you will typically need to deploy it and consume it from applications or in other data flows.</span></span> <span data-ttu-id="a3756-170">机器学习模型有许多潜在的部署目标。</span><span class="sxs-lookup"><span data-stu-id="a3756-170">There are a number of potential deployment targets for machine learning models.</span></span>

### <a name="spark-on-azure-hdinsight"></a><span data-ttu-id="a3756-171">Azure HDInsight 上的 Spark</span><span class="sxs-lookup"><span data-stu-id="a3756-171">Spark on Azure HDInsight</span></span>

<span data-ttu-id="a3756-172">Apache Spark 包括 Spark MLlib（机器学习模型的框架和库）。</span><span class="sxs-lookup"><span data-stu-id="a3756-172">Apache Spark includes Spark MLlib, a framework and library for machine learning models.</span></span> <span data-ttu-id="a3756-173">适用于 Spark 的 Microsoft 机器学习库 (MMLSpark) 还针对 Spark 中的预测模型提供深度学习算法支持。</span><span class="sxs-lookup"><span data-stu-id="a3756-173">The Microsoft Machine Learning library for Spark (MMLSpark) also provides deep learning algorithm support for predictive models in Spark.</span></span>

<span data-ttu-id="a3756-174">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-174">Key benefits:</span></span>

* <span data-ttu-id="a3756-175">Spark 是一个分布式平台，为大规模机器学习流程提供高度可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="a3756-175">Spark is a distributed platform that offers high scalability for high-volume machine learning processes.</span></span>
* <span data-ttu-id="a3756-176">可以通过 Azure Machine Learning Workbench 将模型直接部署到 HDinsight 中的 Spark，然后使用 Azure 机器学习模型管理服务来管理模型。</span><span class="sxs-lookup"><span data-stu-id="a3756-176">You can deploy models directly to Spark in HDinsight from Azure Machine Learning Workbench, and manage them using the Azure Machine Learning Model Management service.</span></span>

<span data-ttu-id="a3756-177">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-177">Considerations:</span></span>

* <span data-ttu-id="a3756-178">在 HDinsght 群集中运行 Spark 全程都会产生费用。</span><span class="sxs-lookup"><span data-stu-id="a3756-178">Spark runs in an HDinsght cluster that incurs charges the whole time it is running.</span></span> <span data-ttu-id="a3756-179">如果只是偶尔才使用机器学习服务，这可能会造成不必要的费用。</span><span class="sxs-lookup"><span data-stu-id="a3756-179">If the machine learning service will only be used occasionally, this may result in unnecessary costs.</span></span>

### <a name="web-service-in-a-container"></a><span data-ttu-id="a3756-180">容器中的 Web 服务</span><span class="sxs-lookup"><span data-stu-id="a3756-180">Web service in a container</span></span>

<span data-ttu-id="a3756-181">可将机器学习模型作为 Python Web 服务部署在 Docker 容器中。</span><span class="sxs-lookup"><span data-stu-id="a3756-181">You can deploy a machine learning model as a Python web service in a Docker container.</span></span> <span data-ttu-id="a3756-182">可将模型部署到 Azure 或边缘设备，然后在其中使用它来处理数据。</span><span class="sxs-lookup"><span data-stu-id="a3756-182">You can deploy the model to Azure or to an edge device, where it can be used locally with the data on which it operates.</span></span>

<span data-ttu-id="a3756-183">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-183">Key Benefits:</span></span>

* <span data-ttu-id="a3756-184">容器是打包和部署服务的简便方式，通常也比较经济高效。</span><span class="sxs-lookup"><span data-stu-id="a3756-184">Containers are a lightweight and generally cost effective way to package and deploy services.</span></span>
* <span data-ttu-id="a3756-185">将模型部署到边缘设备可以使预测逻辑更靠近数据。</span><span class="sxs-lookup"><span data-stu-id="a3756-185">The ability to deploy to an edge device enables you to move your predictive logic closer to the data.</span></span>
* <span data-ttu-id="a3756-186">可以直接从 Azure Machine Learning Workbench 部署到容器。</span><span class="sxs-lookup"><span data-stu-id="a3756-186">You can deploy to a container directly from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="a3756-187">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-187">Considerations:</span></span>

* <span data-ttu-id="a3756-188">此部署模型基于 Docker 容器，因此，以这种方式部署 Web 服务之前，请先熟悉此技术。</span><span class="sxs-lookup"><span data-stu-id="a3756-188">This deployment model is based on Docker containers, so you should be familiar with this technology before deploying a web service this way.</span></span>

### <a name="microsoft-machine-learning-server"></a><span data-ttu-id="a3756-189">Microsoft 机器学习服务器</span><span class="sxs-lookup"><span data-stu-id="a3756-189">Microsoft Machine Learning Server</span></span>

<span data-ttu-id="a3756-190">Machine Learning Server（以前称为 Microsoft R Server）是适用于 R 和 Python 代码的可缩放平台，专门针对机器学习方案而设计。</span><span class="sxs-lookup"><span data-stu-id="a3756-190">Machine Learning Server (formerly Microsoft R Server) is a scalable platform for R and Python code, specifically designed for machine learning scenarios.</span></span>

<span data-ttu-id="a3756-191">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-191">Key benefits:</span></span>

* <span data-ttu-id="a3756-192">高度可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="a3756-192">High scalability.</span></span>
* <span data-ttu-id="a3756-193">直接从 Azure Machine Learning Workbench 部署。</span><span class="sxs-lookup"><span data-stu-id="a3756-193">Direct deployment from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="a3756-194">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-194">Considerations:</span></span>

* <span data-ttu-id="a3756-195">需要在企业中部署和管理 Machine Learning Server。</span><span class="sxs-lookup"><span data-stu-id="a3756-195">You need to deploy and manage Machine Learning Server in your enterprise.</span></span>

### <a name="microsoft-sql-server"></a><span data-ttu-id="a3756-196">Microsoft SQL Server</span><span class="sxs-lookup"><span data-stu-id="a3756-196">Microsoft SQL Server</span></span>

<span data-ttu-id="a3756-197">Microsoft SQL Server 原生支持 R 和 Python，可用于在数据库中作为 Transact-SQL 函数封装以这些语言构建的机器学习模型。</span><span class="sxs-lookup"><span data-stu-id="a3756-197">Microsoft SQL Server supports R and Python natively, enabling you to encapsulate machine learning models built in these languages as Transact-SQL functions in a database.</span></span>

<span data-ttu-id="a3756-198">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-198">Key benefits:</span></span>

* <span data-ttu-id="a3756-199">在数据库函数中封装预测逻辑可以轻松加入数据层逻辑。</span><span class="sxs-lookup"><span data-stu-id="a3756-199">Encapsulate predictive logic in a database function, making it easy to include in data-tier logic.</span></span>

<span data-ttu-id="a3756-200">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-200">Considerations:</span></span>

* <span data-ttu-id="a3756-201">采用 SQL Server 数据库作为应用程序的数据层。</span><span class="sxs-lookup"><span data-stu-id="a3756-201">Assumes a SQL Server database as the data tier for your application.</span></span>

### <a name="azure-machine-learning-web-service"></a><span data-ttu-id="a3756-202">Azure 机器学习 Web 服务</span><span class="sxs-lookup"><span data-stu-id="a3756-202">Azure Machine Learning web service</span></span>

<span data-ttu-id="a3756-203">使用 Azure 机器学习工作室创建机器学习模型时，可将其部署为 Web 服务。</span><span class="sxs-lookup"><span data-stu-id="a3756-203">When you create a machine learning model using Azure Machine Learning Studio, you can deploy it as a web service.</span></span> <span data-ttu-id="a3756-204">然后，可以在支持 HTTP 通信的任何客户端应用程序中通过 REST 接口来使用该模型。</span><span class="sxs-lookup"><span data-stu-id="a3756-204">This can then be consumed through a REST interface from any client applications capable of communicating by HTTP.</span></span>

<span data-ttu-id="a3756-205">主要优势：</span><span class="sxs-lookup"><span data-stu-id="a3756-205">Key benefits:</span></span>

* <span data-ttu-id="a3756-206">易于开发和部署。</span><span class="sxs-lookup"><span data-stu-id="a3756-206">Ease of development and deployment.</span></span>
* <span data-ttu-id="a3756-207">包含基本监视指标的 Web 服务管理门户。</span><span class="sxs-lookup"><span data-stu-id="a3756-207">Web service management portal with basic monitoring metrics.</span></span>
* <span data-ttu-id="a3756-208">原生支持从 Azure Data Lake Analytics、Azure 数据工厂和 Azure 流分析调用 Azure 机器学习 Web 服务。</span><span class="sxs-lookup"><span data-stu-id="a3756-208">Built-in support for calling Azure Machine Learning web services from Azure Data Lake Analytics, Azure Data Factory, and Azure Stream Analytics.</span></span>

<span data-ttu-id="a3756-209">注意事项：</span><span class="sxs-lookup"><span data-stu-id="a3756-209">Considerations:</span></span>

* <span data-ttu-id="a3756-210">仅适用于使用 Azure 机器学习工作室构建的模型。</span><span class="sxs-lookup"><span data-stu-id="a3756-210">Only available for models built using Azure Machine Learning Studio.</span></span>
* <span data-ttu-id="a3756-211">仅限基于 Web 的访问，训练的模型无法在本地或脱机运行。</span><span class="sxs-lookup"><span data-stu-id="a3756-211">Web-based access only, trained models cannot run on-premises or offline.</span></span>

