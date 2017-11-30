---
title: "在 Azure 与 Internet 之间实施外围网络"
description: "如何在 Azure 中实施一个提供 Internet 访问方式的安全混合网络体系结构。"
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 372d5bb0fc0e3c272843e062210dec5c15b2b78a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="dmz-between-azure-and-the-internet"></a><span data-ttu-id="eb9c7-103">Azure 与 Internet 之间的外围网络</span><span class="sxs-lookup"><span data-stu-id="eb9c7-103">DMZ between Azure and the Internet</span></span>

<span data-ttu-id="eb9c7-104">此参考体系结构显示了一个可将本地网络扩展到 Azure 并接受 Internet 流量的安全混合网络。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> 

<span data-ttu-id="eb9c7-105">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="eb9c7-105">[![0]][0]</span></span> 

<span data-ttu-id="eb9c7-106">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="eb9c7-107">此参考体系结构扩展[在 Azure 与本地数据中心之间实施外围网络][implementing-a-secure-hybrid-network-architecture]中所述的体系结构。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-107">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="eb9c7-108">它添加了一个用于处理 Internet 流量的公共外围网络，以及一个用于处理来自本地网络的流量的专用外围网络</span><span class="sxs-lookup"><span data-stu-id="eb9c7-108">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network</span></span> 

<span data-ttu-id="eb9c7-109">此体系结构的典型用途包括：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-109">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="eb9c7-110">在本地运行一部分工作负荷，在 Azure 中运行一部分工作负荷的混合应用程序。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-110">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="eb9c7-111">可路由来自本地和 Internet 的传入流量的 Azure 基础结构。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-111">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="eb9c7-112">体系结构</span><span class="sxs-lookup"><span data-stu-id="eb9c7-112">Architecture</span></span>

<span data-ttu-id="eb9c7-113">该体系结构包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="eb9c7-114">**公共 IP 地址 (PIP)**。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-114">**Public IP address (PIP)**.</span></span> <span data-ttu-id="eb9c7-115">公共终结点的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-115">The IP address of the public endpoint.</span></span> <span data-ttu-id="eb9c7-116">连接到 Internet 的外部用户可通过此地址访问系统。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-116">External users connected to the Internet can access the system through this address.</span></span>
* <span data-ttu-id="eb9c7-117">**网络虚拟设备 (NVA)**。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-117">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="eb9c7-118">此体系结构包含一个用于处理源自 Internet 的流量的独立 NVA 池。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-118">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
* <span data-ttu-id="eb9c7-119">**Azure 负载均衡器**。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-119">**Azure load balancer**.</span></span> <span data-ttu-id="eb9c7-120">来自 Internet 的所有传入请求通过负载均衡器，并分发到公共外围网络中的 NVA。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-120">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="eb9c7-121">**公共外围网络入站子网**。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-121">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="eb9c7-122">此子网接受来自 Azure 负载均衡器的请求。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-122">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="eb9c7-123">传入的请求传递到公共外围网络中某个 NVA。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-123">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="eb9c7-124">**公共外围网络出站子网**。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-124">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="eb9c7-125">NVA 批准的请求通过此子网传递到 Web 层的内部负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-125">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="eb9c7-126">建议</span><span class="sxs-lookup"><span data-stu-id="eb9c7-126">Recommendations</span></span>

<span data-ttu-id="eb9c7-127">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-127">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="eb9c7-128">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-128">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="nva-recommendations"></a><span data-ttu-id="eb9c7-129">有关 NVA 的建议</span><span class="sxs-lookup"><span data-stu-id="eb9c7-129">NVA recommendations</span></span>

<span data-ttu-id="eb9c7-130">请对源自 Internet 的流量使用一组 NVA，对源自本地的流量使用另一组 NVA。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-130">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="eb9c7-131">若仅对两组网络流量使用一组 NVA，则存在安全风险，因为它不会在两组网络流量之间提供安全周边。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-131">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="eb9c7-132">使用不同 NVA 可以降低检查安全规则的复杂性，并指明哪项规则与每个传入网络请求相对应。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-132">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="eb9c7-133">一组 NVA 仅对 Internet 流量实施规则，另一组 NVA 仅对本地流量实施规则。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-133">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="eb9c7-134">包含第 7 层 NVA，以在 NVA 级别终止应用程序连接，并保持与后端层的兼容性。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-134">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="eb9c7-135">这可以保证建立对称连接，使得来自后端层的响应流量可通过 NVA 返回。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-135">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>  

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="eb9c7-136">有关公共负载均衡器的建议</span><span class="sxs-lookup"><span data-stu-id="eb9c7-136">Public load balancer recommendations</span></span>

<span data-ttu-id="eb9c7-137">为了实现可伸缩性和可用性，请在[可用性集][availability-set]中部署公共外围网络 NVA，并使用[面向 Internet 的负载均衡器][load-balancer]在可用性集中的 NVA 之间分发 Internet 请求。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-137">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>  

<span data-ttu-id="eb9c7-138">将负载均衡器配置为仅接受传送 Internet 流量时所要开放的端口上的请求。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-138">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="eb9c7-139">例如，将入站 HTTP 请求限制为端口 80，将入站 HTTPS 请求限制为端口 443。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-139">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="eb9c7-140">有关可伸缩性的注意事项</span><span class="sxs-lookup"><span data-stu-id="eb9c7-140">Scalability considerations</span></span>

<span data-ttu-id="eb9c7-141">即使体系结构最初只需在公共外围网络中部署单个 NVA，我们也还是建议一开始就在公共外围网络的前面部署一个负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-141">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="eb9c7-142">这样，以后便可以根据需要更轻松地扩展到多个 NVA。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-142">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="eb9c7-143">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="eb9c7-143">Availability considerations</span></span>

<span data-ttu-id="eb9c7-144">面向 Internet 的负载均衡器要求将每个 NVA 部署在公共外围网络入站子网中，以实施[运行状况探测][lb-probe]。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-144">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="eb9c7-145">无法在此终结点上做出响应的运行状况探测被视为不可用，负载均衡器会将请求定向到同一可用性集中的其他 NVA。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-145">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="eb9c7-146">请注意，如果所有 NVA 都无法做出响应，应用程序将会失败，因此必须配置监视功能，以便在正常的 NVA 实例数低于定义的阈值时，向 DevOps 发出警报。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-146">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="eb9c7-147">有关可管理性的注意事项</span><span class="sxs-lookup"><span data-stu-id="eb9c7-147">Manageability considerations</span></span>

<span data-ttu-id="eb9c7-148">应该通过管理子网中的 Jumpbox 对公共外围网络中的 NVA 执行所有监视和管理。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-148">All monitoring and management for the NVAs in the public DMZ should be be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="eb9c7-149">如[在 Azure 与本地数据中心之间实施外围网络][implementing-a-secure-hybrid-network-architecture]中所述，定义从本地网络到网关再到 Jumpbox 的单个网络路由，以限制访问。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-149">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="eb9c7-150">如果从本地网络到 Azure 的网关连接断开，仍可以通过部署公共 IP 地址、将该地址添加到 Jumpbox，然后从 Internet 登录，来连接 Jumpbox。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-150">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="eb9c7-151">有关安全性的注意事项</span><span class="sxs-lookup"><span data-stu-id="eb9c7-151">Security considerations</span></span>

<span data-ttu-id="eb9c7-152">此参考体系结构实施多个安全级别：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-152">This reference architecture implements multiple levels of security:</span></span>

* <span data-ttu-id="eb9c7-153">面向 Internet 的负载均衡器将请求定向到入站公共外围网络子网中的、且仅位于需要为应用程序开放的端口上的 NVA。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-153">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
* <span data-ttu-id="eb9c7-154">针对入站和出站公共外围网络子网实施的 NSG 通过阻止超出 NSG 规则范围的请求，来防止 NVA 受到威胁。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-154">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
* <span data-ttu-id="eb9c7-155">NVA 的 NAT 路由配置将端口 80 和端口 443 上的传入请求定向到 Web 层负载均衡器，但忽略其他所有端口上的请求。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-155">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="eb9c7-156">应该记录所有端口上的所有传入请求。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-156">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="eb9c7-157">定期审核日志，并关注超出预期参数范围的请求，因为这些请求可能暗示着发生了入侵企图。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-157">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>

## <a name="solution-deployment"></a><span data-ttu-id="eb9c7-158">解决方案部署</span><span class="sxs-lookup"><span data-stu-id="eb9c7-158">Solution deployment</span></span>

<span data-ttu-id="eb9c7-159">[GitHub][github-folder] 上提供了可实施这些建议的参考体系结构部署。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-159">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span> <span data-ttu-id="eb9c7-160">可以遵照以下指导，使用 Windows 或 Linux VM 部署该参考体系结构：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-160">The reference architecture can be deployed either with Windows or Linux VMs by following the directions below:</span></span>

1. <span data-ttu-id="eb9c7-161">单击下面的按钮：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-161">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="eb9c7-162">在 Azure 门户中打开该链接后，必须输入某些设置的值：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-162">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="eb9c7-163">参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-public-dmz-network-rg`。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-163">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-public-dmz-network-rg` in the text box.</span></span>
   * <span data-ttu-id="eb9c7-164">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-164">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="eb9c7-165">不要编辑“模板根 URI”或“参数根 URI”文本框。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-165">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="eb9c7-166">从下拉框中选择“OS 类型”：“Windows”或“Linux”。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-166">Select the **Os Type** from the drop down box, **windows** or **linux**.</span></span>
   * <span data-ttu-id="eb9c7-167">查看条款和条件，并单击“我同意上述条款和条件”复选框。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-167">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="eb9c7-168">单击“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-168">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="eb9c7-169">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-169">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="eb9c7-170">单击下面的按钮：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-170">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="eb9c7-171">在 Azure 门户中打开该链接后，必须输入某些设置的值：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-171">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="eb9c7-172">参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-public-dmz-wl-rg`。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-172">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-public-dmz-wl-rg` in the text box.</span></span>
   * <span data-ttu-id="eb9c7-173">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-173">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="eb9c7-174">不要编辑“模板根 URI”或“参数根 URI”文本框。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-174">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="eb9c7-175">查看条款和条件，并单击“我同意上述条款和条件”复选框。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-175">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="eb9c7-176">单击“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-176">Click the **Purchase** button.</span></span>
6. <span data-ttu-id="eb9c7-177">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-177">Wait for the deployment to complete.</span></span>
7. <span data-ttu-id="eb9c7-178">单击下面的按钮：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-178">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. <span data-ttu-id="eb9c7-179">在 Azure 门户中打开该链接后，必须输入某些设置的值：</span><span class="sxs-lookup"><span data-stu-id="eb9c7-179">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="eb9c7-180">参数文件中已定义了**资源组**名称，因此请选择“使用现有”，并在文本框中输入 `ra-public-dmz-network-rg`。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-180">The **Resource group** name is already defined in the parameter file, so select **Use Existing** and enter `ra-public-dmz-network-rg` in the text box.</span></span>
   * <span data-ttu-id="eb9c7-181">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-181">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="eb9c7-182">不要编辑“模板根 URI”或“参数根 URI”文本框。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-182">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="eb9c7-183">查看条款和条件，并单击“我同意上述条款和条件”复选框。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-183">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="eb9c7-184">单击“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-184">Click the **Purchase** button.</span></span>
9. <span data-ttu-id="eb9c7-185">等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-185">Wait for the deployment to complete.</span></span>
10. <span data-ttu-id="eb9c7-186">参数文件包括所有 VM 的硬编码管理员用户名和密码，我们强烈建议立即更改。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-186">The parameter files include hard-coded administrator user name and password for all VMs, and it is strongly recommended that you immediately change both.</span></span> <span data-ttu-id="eb9c7-187">针对部署中的每个 VM，请在 Azure 门户选择该 VM，并在“支持 + 故障排除”边栏选项卡中单击“重置密码”。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-187">For each VM in the deployment, select it in the Azure portal and then click  **Reset password** in the **Support + troubleshooting** blade.</span></span> <span data-ttu-id="eb9c7-188">在“模式”下拉框中选择“重置密码”，并选择新**用户名**和**密码**。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-188">Select **Reset password** in the **Mode** drop down box, then select a new **User name** and **Password**.</span></span> <span data-ttu-id="eb9c7-189">单击“更新”按钮保存设置。</span><span class="sxs-lookup"><span data-stu-id="eb9c7-189">Click the **Update** button to save.</span></span>


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.azureedge.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "保护混合网络体系结构"