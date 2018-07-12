---
title: 在 Azure 与 Internet 之间实施外围网络
description: 如何在 Azure 中实施一个提供 Internet 访问方式的安全混合网络体系结构。
author: telmosampaio
ms.date: 07/02/2018
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 7a062d2394ae8b3bd1b17c19cbdf512327f9a766
ms.sourcegitcommit: 9b459f75254d97617e16eddd0d411d1f80b7fe90
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/03/2018
ms.locfileid: "37403141"
---
# <a name="dmz-between-azure-and-the-internet"></a>Azure 与 Internet 之间的外围网络

此参考体系结构显示了一个可将本地网络扩展到 Azure 并接受 Internet 流量的安全混合网络。 [**部署此解决方案**。](#deploy-the-solution)

[![0]][0] 

下载此体系结构的 [Visio 文件][visio-download]。

此参考体系结构扩展[在 Azure 与本地数据中心之间实施外围网络][implementing-a-secure-hybrid-network-architecture]中所述的体系结构。 它添加了一个用于处理 Internet 流量的公共外围网络，以及一个用于处理来自本地网络的流量的专用外围网络 

此体系结构的典型用途包括：

* 在本地运行一部分工作负荷，在 Azure 中运行一部分工作负荷的混合应用程序。
* 可路由来自本地和 Internet 的传入流量的 Azure 基础结构。

## <a name="architecture"></a>体系结构

该体系结构包括以下组件。

* **公共 IP 地址 (PIP)**。 公共终结点的 IP 地址。 连接到 Internet 的外部用户可通过此地址访问系统。
* **网络虚拟设备 (NVA)**。 此体系结构包含一个用于处理源自 Internet 的流量的独立 NVA 池。
* **Azure 负载均衡器**。 来自 Internet 的所有传入请求通过负载均衡器，并分发到公共外围网络中的 NVA。
* **公共外围网络入站子网**。 此子网接受来自 Azure 负载均衡器的请求。 传入的请求传递到公共外围网络中某个 NVA。
* **公共外围网络出站子网**。 NVA 批准的请求通过此子网传递到 Web 层的内部负载均衡器。

## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。 

### <a name="nva-recommendations"></a>有关 NVA 的建议

请对源自 Internet 的流量使用一组 NVA，对源自本地的流量使用另一组 NVA。 若仅对两组网络流量使用一组 NVA，则存在安全风险，因为它不会在两组网络流量之间提供安全周边。 使用不同 NVA 可以降低检查安全规则的复杂性，并指明哪项规则与每个传入网络请求相对应。 一组 NVA 仅对 Internet 流量实施规则，另一组 NVA 仅对本地流量实施规则。

包含第 7 层 NVA，以在 NVA 级别终止应用程序连接，并保持与后端层的兼容性。 这可以保证建立对称连接，使得来自后端层的响应流量可通过 NVA 返回。  

### <a name="public-load-balancer-recommendations"></a>有关公共负载均衡器的建议

为了实现可伸缩性和可用性，请在[可用性集][availability-set]中部署公共外围网络 NVA，并使用[面向 Internet 的负载均衡器][load-balancer]在可用性集中的 NVA 之间分发 Internet 请求。  

将负载均衡器配置为仅接受传送 Internet 流量时所要开放的端口上的请求。 例如，将入站 HTTP 请求限制为端口 80，将入站 HTTPS 请求限制为端口 443。

## <a name="scalability-considerations"></a>可伸缩性注意事项

即使体系结构最初只需在公共外围网络中部署单个 NVA，我们也还是建议一开始就在公共外围网络的前面部署一个负载均衡器。 这样，以后便可以根据需要更轻松地扩展到多个 NVA。

## <a name="availability-considerations"></a>可用性注意事项

面向 Internet 的负载均衡器要求将每个 NVA 部署在公共外围网络入站子网中，以实施[运行状况探测][lb-probe]。 无法在此终结点上做出响应的运行状况探测被视为不可用，负载均衡器会将请求定向到同一可用性集中的其他 NVA。 请注意，如果所有 NVA 都无法做出响应，应用程序将会失败，因此必须配置监视功能，以便在正常的 NVA 实例数低于定义的阈值时，向 DevOps 发出警报。

## <a name="manageability-considerations"></a>可管理性注意事项

应该通过管理子网中的 Jumpbox 对公共外围网络中的 NVA 执行所有监视和管理。 如[在 Azure 与本地数据中心之间实施外围网络][implementing-a-secure-hybrid-network-architecture]中所述，定义从本地网络到网关再到 Jumpbox 的单个网络路由，以限制访问。

如果从本地网络到 Azure 的网关连接断开，仍可以通过部署公共 IP 地址、将该地址添加到 Jumpbox，然后从 Internet 登录，来连接 Jumpbox。

## <a name="security-considerations"></a>安全注意事项

此参考体系结构实施多个安全级别：

* 面向 Internet 的负载均衡器将请求定向到入站公共外围网络子网中的、且仅位于需要为应用程序开放的端口上的 NVA。
* 针对入站和出站公共外围网络子网实施的 NSG 通过阻止超出 NSG 规则范围的请求，来防止 NVA 受到威胁。
* NVA 的 NAT 路由配置将端口 80 和端口 443 上的传入请求定向到 Web 层负载均衡器，但忽略其他所有端口上的请求。

应该记录所有端口上的所有传入请求。 定期审核日志，并关注超出预期参数范围的请求，因为这些请求可能暗示着发生了入侵企图。


## <a name="deploy-the-solution"></a>部署解决方案

[GitHub][github-folder] 上提供了可实施这些建议的参考体系结构部署。 

### <a name="prerequisites"></a>先决条件

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a>部署资源

1. 导航到参考体系结构 GitHub 存储库的 `/dmz/secure-vnet-hybrid` 文件夹。

2. 运行以下命令：

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. 运行以下命令：

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a>连接本地网关和 Azure 网关

此步骤连接两个本地网络网关。

1. 在 Azure 门户中，导航到已创建的资源组。 

2. 找到名为 `ra-vpn-vgw-pip` 的资源，并复制“概述”边栏选项卡中显示的 IP 地址。

3. 找到名为 `onprem-vpn-lgw` 的资源。

4. 单击“配置”边栏选项卡。 在“IP 地址”下，粘贴步骤 2 中获取的 IP 地址。

    ![](./images/local-net-gw.png)

5. 单击“保存”并等待操作完成。 可能需要大约 5 分钟。

6. 找到名为 `onprem-vpn-gateway1-pip` 的资源。 复制“概述”边栏选项卡中显示的 IP 地址。

7. 找到名为 `ra-vpn-lgw` 的资源。 

8. 单击“配置”边栏选项卡。 在“IP 地址”下，粘贴步骤 6 中获取的 IP 地址。

9. 单击“保存”并等待操作完成。

10. 若要验证连接，请转到每个网关的“连接”边栏选项卡。 状态应为“已连接”。

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a>验证网络流量是否抵达 Web 层

1. 在 Azure 门户中，导航到已创建的资源组。 

2. 找到名为 `pub-dmz-lb` 的资源，即公共外围网络前面的负载均衡器。 

3. 复制“概述”边栏选项卡中的公共 IP 地址，并在 Web 浏览器中打开此地址。 应会看到默认的 Apache2 服务器主页。

4. 找到名为 `int-dmz-lb` 的资源，即专用外围网络前面的负载均衡器。 复制“概述”边栏选项卡中的专用 IP 地址。

5. 找到名为 `jb-vm1` 的 VM。 单击“连接”，使用远程桌面连接到 VM。 用户名和密码已在 onprem.json 文件中指定。

6. 在远程桌面会话中打开 Web 浏览器，并导航到步骤 4 中获取的 IP 地址。 应会看到默认的 Apache2 服务器主页。

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "保护混合网络体系结构"
