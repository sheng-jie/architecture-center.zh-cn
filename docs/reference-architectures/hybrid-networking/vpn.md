---
title: 使用 VPN 将本地网络连接到 Azure
description: 如何实现这样一个安全的站点到站点网络体系结构：跨 Azure 虚拟网络，以及使用 VPN 建立连接的本地网络。
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: dafcee6607d9cc7c56c332f9ed5d9568ff70f0e7
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/30/2018
ms.locfileid: "30270687"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>使用 VPN 网关将本地网络连接到 Azure

此参考体系结构演示如何使用站点到站点虚拟专用网络 (VPN) 将本地网络扩展到 Azure。 本地网络与 Azure 虚拟专用网络 (VNet) 之间的流量通过 IPSec VPN 隧道传送。 [**部署此解决方案**。](#deploy-the-solution)

![[0]][0]

下载此体系结构的 [Visio 文件][visio-download]。

## <a name="architecture"></a>体系结构 

该体系结构包括以下组件。

* **本地网络**。 组织中运行的专用局域网。

* **VPN 设备**。 用于与本地网络建立外部连接的设备或服务。 该 VPN 设备可以是硬件设备，也可以是软件解决方案，例如 Windows Server 2012 中的路由和远程访问服务 (RRAS)。 有关支持的 VPN 设备的列表以及有关如何配置它们以连接到 Azure VPN 网关的信息，请参阅[关于站点到站点 VPN 网关连接的 VPN 设备][vpn-appliance]一文中针对所选设备的说明。

* **虚拟网络 (VNet)**。 云应用程序和 Azure VPN 网关的组件驻留在相同 [VNet][azure-virtual-network] 中。

* **Azure VPN 网关**。 [VPN 网关][azure-vpn-gateway]服务使你可以通过 VPN 设备将 VNet 连接到本地网络。 有关详细信息，请参阅[将本地网络连接到 Microsoft Azure 虚拟网络][connect-to-an-Azure-vnet]。 VPN 网关包括以下元素：
  
  * **虚拟网络网关**。 为 VNet 提供虚拟 VPN 设备的资源。 它负责将流量从本地网络路由到 VNet。
  * **本地网络网关**。 本地 VPN 设备的抽象。 从云应用程序到本地网络的网络流量通过此网关进行路由。
  * **连接**。 该连接包含一些属性，这些属性指定连接类型 (IPSec)，以及与本地 VPN 设备共享的、用于加密流量的密钥。
  * **网关子网**。 虚拟网络网关保留在自己的子网中，该子网需满足下面“建议”一节中所述的要求。

* **云应用程序**。 Azure 中托管的应用程序。 它可以包含多个层，以及通过 Azure 负载均衡器连接的多个子网。 有关应用程序基础结构的详细信息，请参阅[运行 Windows VM 工作负荷][windows-vm-ra]和[运行 Linux VM 工作负荷][linux-vm-ra]。

* **内部负载均衡器**。 来自 VPN 网关的网络流量通过内部负载均衡器路由到云应用程序。 该负载均衡器位于应用程序的前端子网中。

## <a name="recommendations"></a>建议

以下建议适用于大多数方案。 除非有优先于这些建议的特定要求，否则请遵循这些建议。

### <a name="vnet-and-gateway-subnet"></a>VNet 和网关子网

创建地址空间的大小足以容纳所有所需资源的 Azure VNet。 确保 VNet 地址空间具有足够空间，以便在将来可能需要更多 VM 时增长。 VNet 的地址空间不得与本地网络重叠。 例如，上图将地址空间 10.20.0.0/16 用于 VNet。

创建一个名为 *GatewaySubnet* 的子网，使其地址范围为 /27。 此子网是虚拟网络网关所必需的。 向此子网分配 32 个地址将有助于防止将来达到网关大小限制。 此外，避免将此子网置于地址空间中间。 一个好的做法是将网关子网的地址空间设置在 VNet 地址空间上端。 图中所示的示例使用 10.20.255.224/27。  下面是用于计算 [CIDR] 的快速过程：

1. 将 VNet 地址空间中的变量位设置为 1（直到网关子网所使用的位），然后将剩余位设置为 0。
2. 将得到的位转换为十进制，然后将它表示为前缀长度设置为网关子网大小的地址空间。

例如，对于 IP 地址范围为 10.20.0.0/16 的 VNet，应用上面的步骤 1 会成为 10.20.0b11111111.0b11100000。  将该数字转换为十进制并将它表示为地址空间会生成 10.20.255.224/27。 

> [!WARNING]
> 请勿向网关子网部署任何 VM。 此外，请勿向此子网分配 NSG，因为它会导致网关停止工作。
> 
> 

### <a name="virtual-network-gateway"></a>虚拟网络网关

为虚拟网络网关分配公共 IP 地址。

在网关子网中创建虚拟网络网关，并将新分配的公共 IP 地址分配给它。 使用最符合你要求并且通过 VPN 设备启用的网关类型：

- 如果需要基于策略条件（如地址前缀）密切控制如何对请求进行路由，请创建[基于策略的网关][policy-based-routing]。 基于策略的网关使用静态路由，仅适用于站点到站点连接。

- 如果使用 RRAS 连接到本地网络、支持多站点或跨区域连接或是实现 VNet 到 VNet 连接（包含遍历多个 VNet 的路由），请创建[基于路由的网关][route-based-routing]。 基于路由的网关使用动态路由定向网络之间的流量。 它们在网络路径中的容错能力好于静态路由，因为它们可以尝试备用路由。 基于路由的网关还可以减少管理开销，因为路由在网络地址更改时可以无需手动更新。

有关支持的 VPN 设备的列表，请参阅[关于站点到站点 VPN 网关连接的 VPN 设备][vpn-appliances]。

> [!NOTE]
> 创建网关后，必须先删除并重新创建网关才能更改网关类型。
> 
> 

选择最符合你吞吐量要求的 Azure VPN 网关 SKU。 Azure VPN 网关提供三个 SKU，如下表所示。 

| SKU | VPN 吞吐量 | 最大 IPSec 隧道数 |
| --- | --- | --- |
| 基本 |100 Mbps |10 |
| 标准 |100 Mbps |10 |
| 高性能 |200 Mbps |30 |

> [!NOTE]
> 基本 SKU 不与 Azure ExpressRoute 兼容。 可以在创建网关之后[更改 SKU][changing-SKUs]。
> 
> 

会基于预配和提供网关的时间量进行收费。 请参阅 [VPN 网关定价][azure-gateway-charges]。

为网关子网创建将传入应用程序流量从网关定向到内部负载均衡器，而不是允许请求直接传递到应用程序 VM 的路由规则。

### <a name="on-premises-network-connection"></a>本地网络连接

创建本地网络网关。 指定本地 VPN 设备的公共 IP 地址以及本地网络的地址空间。 请注意，本地 VPN 设备必须具有可由 Azure VPN 网关中的本地网络网关访问的公共 IP 地址。 VPN 设备不能位于网络地址转换 (NAT) 设备后面。

为虚拟网络网关和本地网络网关创建站点到站点连接。 选择站点到站点 (IPSec) 连接类型，并指定共享密钥。 具有 Azure VPN 网关的站点到站点加密基于 IPSec 协议，使用预共享密钥进行身份验证。 创建 Azure VPN 网关时需指定密钥。 必须使用相同密钥配置在本地运行的 VPN 设备。 当前不支持其他身份验证机制。

确保本地路由基础结构配置为将目标为 Azure VNet 中的地址的请求转发到 VPN 设备。

在本地网络中打开云应用程序所需的任何端口。

测试连接以验证以下事项：

* 本地 VPN 设备通过 Azure VPN 网关将流量正确路由到云应用程序。
* VNet 将流量正确路由回本地网络。
* 正确阻止两个方向上的禁止流量。

## <a name="scalability-considerations"></a>可伸缩性注意事项

可以通过从基本或标准 VPN 网关 SKU 升级到高性能 VPN SKU，来实现有限纵向可缩放性。

对于预期存在大量 VPN 流量的 VNet，请考虑将不同工作负载分发到单独的较小 VNet 以及为每个 VNet 都配置 VPN 网关。

可以按水平或垂直方式对 VNet 进行分区。 若要进行水平分区，请将某些 VM 实例从每个层移动到新 VNet 的子网中。 结果是每个 VNet 都具有相同结构和功能。 若要进行垂直分区，请重新设计每个层，以将功能划分为不同逻辑区域（如处理订单、开发票、客户帐户管理等）。 每个功能区域随后可以放置在自己的 VNet 中。

在 VNet 中复制本地 Active Directory 域控制器以及在 VNet 中实现 DNS 可以帮助减少从本地到云的某些安全相关和管理流量。 有关详细信息，请参阅[将 Active Directory 域服务 (AD DS) 扩展到 Azure][adds-extend-domain]。

## <a name="availability-considerations"></a>可用性注意事项

如果需要确保本地网络对 Azure VPN 网关保持可用，请为本地 VPN 网关实现故障转移群集。

如果组织具有多个本地站点，请创建与一个或多个 Azure VNet 之间的[多站点连接][vpn-gateway-multi-site]。 此方法需要动态（基于路由的）路由，因此请确保本地 VPN 网关支持此功能。

有关服务级别协议的详细信息，请参阅 [VPN 网关的 SLA][sla-for-vpn-gateway]。 

## <a name="manageability-considerations"></a>可管理性注意事项

监视来自本地 VPN 设备的诊断信息。 此过程取决于 VPN 设备提供的功能。 例如，如果在 Windows Server 2012 上使用路由和远程访问服务，则可使用 [RRAS 日志记录][rras-logging]。

使用 [Azure VPN 网关诊断][gateway-diagnostic-logs]可捕获有关连接问题的信息。 这些日志可以用于跟踪信息，如连接请求的源和目标、使用的协议以及连接的建立方式（或尝试失败的原因）。

使用 Azure 门户中提供的审核日志监视 Azure VPN 网关的运行日志。 为本地网络网关、Azure 网络网关和连接分别提供了单独的日志。 此信息可以用于跟踪对网关进行的任何更改，并且在以前正常运行的网关由于某种原因而停止工作时可能会十分有用。

![[2]][2]

监视连接，并跟踪连接失败事件。 可以使用监视包（如 [Nagios][nagios]）捕获并报告此信息。

## <a name="security-considerations"></a>安全注意事项

为每个 VPN 网关生成不同的共享密钥。 使用强共享密钥可帮助抵御暴力攻击。

> [!NOTE]
> 当前无法使用 Azure Key Vault 为 Azure VPN 网关预共享密钥。
> 
> 

确保本地 VPN 设备使用的加密方法[与 Azure VPN 网关兼容][vpn-appliance-ipsec]。 对于基于策略的路由，Azure VPN 网关支持 AES256、AES128 和 3DES 加密算法。 基于路由的网关支持 AES256 和 3DES。

如果本地 VPN 设备位于在外围网络 (DMZ) 与 Internet 之间具有防火墙的外围网络中，则可能必须配置[其他防火墙规则][additional-firewall-rules]以允许实现站点到站点 VPN 连接。

如果 VNet 中的应用程序将数据发送到 Internet，请考虑[实现强制隧道][forced-tunneling]以通过本地网络路由所有 Internet 绑定流量。 此方法使你可以审核应用程序从本地基础结构进行的传出请求。

> [!NOTE]
> 强制隧道可能会影响与 Azure 服务（例如存储服务）和 Windows 许可证管理器之间的连接。
> 
> 


## <a name="troubleshooting"></a>故障排除 

有关常见 VPN 相关错误故障排除的常规信息，请参阅[常见 VPN 相关错误故障排除][troubleshooting-vpn-errors]。

以下建议可用于确定本地 VPN 设备是否正常运行。

- **在 VPN 设备生成的任何日志文件中检查是否存在错误或故障。**

    这会帮助确定 VPN 设备是否正常运行。 此信息的位置因设备而异。 例如，如果在 Windows Server 2012 上使用 RRAS，则可以使用以下 PowerShell 命令显示 RRAS 服务的错误事件信息：

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    每个条目的 Message 属性提供错误的说明。 一些常见示例包括：

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    还可以使用以下 PowerShell 命令获取有关通过 RRAS 服务尝试连接的事件日志信息： 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    如果出现连接故障，则此日志会包含类似于以下内容的错误：

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- **验证跨 VPN 网关的连接和路由。**

    VPN 设备可能无法通过 Azure VPN 网关正确路由流量。 使用 [PsPing][psping] 这类工具可验证跨 VPN 网关的连接和路由。 例如，若要测试从本地计算机到位于 VNet 上的 Web 服务器的连接，请运行以下命令（将 `<<web-server-address>>` 替换为 Web 服务器的地址）：

    ```
    PsPing -t <<web-server-address>>:80
    ```

    如果本地计算机可以将流量路由到 Web 服务器，则你应看到类似于以下内容的输出：

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    如果本地计算机无法与指定目标进行通信，则你会看到如下消息：

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- **验证本地防火墙是否允许 VPN 流量通过以及正确的端口是否打开。**

- **验证本地 VPN 设备使用的加密方法是否[与 Azure VPN 网关兼容][vpn-appliance]。** 对于基于策略的路由，Azure VPN 网关支持 AES256、AES128 和 3DES 加密算法。 基于路由的网关支持 AES256 和 3DES。

以下建议可用于确定是否存在与 Azure VPN 网关有关的问题：

- **在 [Azure VPN 网关诊断日志][gateway-diagnostic-logs]中检查是否存在潜在问题。**

- **验证 Azure VPN 网关和本地 VPN 设备是否使用相同的共享身份验证密钥进行配置。**

    可以使用以下 Azure CLI 命令查看 Azure VPN 网关存储的共享密钥：

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    使用适合于本地 VPN 设备的命令显示为该设备配置的共享密钥。

    验证包含 Azure VPN 网关的 GatewaySubnet 子网是否不与 NSG 关联。

    可以使用以下 Azure CLI 命令查看子网详细信息：

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    确保没有名为 Network Security Group id 的数据字段。以下示例显示分配了 NSG (VPN-Gateway-Group) 的 GatewaySubnet 实例的结果。 如果没有为此 NSG 定义任何规则，则这可能会阻止网关正常工作。

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- **验证 Azure VNet 中的虚拟机是否配置为允许来自 VNet 外部的流量进入。**

    检查与包含这些虚拟机的子网关联的任何 NSG 规则。 可以使用以下 Azure CLI 命令查看所有 NSG 规则：

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **验证 Azure VPN 网关是否已连接。**

    可以使用以下 Azure PowerShell 命令检查 Azure VPN 连接的当前状态。 `<<connection-name>>` 参数是链接虚拟网络网关和本地网关的 Azure VPN 连接的名称。

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    以下代码片段突出显示在网关已连接（第一个示例）和断开连接（第二个示例）时生成的输出：

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

以下建议可用于确定主机 VM 配置、网络带宽利用率或应用程序性能是否存在问题：

- **验证在子网中 Azure VM 上运行的来宾操作系统中的防火墙是否正确配置为允许来自本地 IP 范围的流量。**

- **验证流量是否未接近可供 Azure VPN 网关使用的带宽限制。**

    如何进行验证取决于在本地运行的 VPN 设备。 例如，如果在 Windows Server 2012 上使用 RRAS，则可以使用性能监视器跟踪通过 VPN 连接接收和传输的数据量。 使用 RAS Total 对象，选择 Bytes Received/Sec 和 Bytes Transmitted/Sec 计数器：

    ![[3]][3]

    应将结果与可供 VPN 网关使用的带宽（对于基本和标准 SKU 为 100 Mbps，对于高性能 SKU 为 200 Mbps）进行比较：

    ![[4]][4]

- **验证是否已为应用程序负载部署了正确数量和大小的 VM。**

    确定 Azure VNet 中是否有任何虚拟机运行缓慢。 如果有，则它们可能过载、可能数量太少而无法处理负载或负载均衡器未正确配置。 若要确定这种情况，请[捕获并分析诊断信息][azure-vm-diagnostics]。 可以使用 Azure 门户检查结果，不过还有许多可以提供有关性能数据的详细深入信息的第三方工具可用。

- **验证该应用程序是否在高效使用云资源。**

    检测每个 VM 上运行的应用程序代码以确定应用程序是否在充分利用资源。 可以使用 [Application Insights][application-insights] 这类工具。

## <a name="deploy-the-solution"></a>部署解决方案


**先决条件。** 必须提供一个已配置适当网络设备的现有本地基础结构。

若要部署该解决方案，请执行以下步骤。

1. 单击下面的按钮：<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. 等待该链接在 Azure 门户中打开，然后执行以下步骤： 
   * 参数文件中已定义**资源组**名称，因此请选择“新建”，并在文本框中输入 `ra-hybrid-vpn-rg`。
   * 从“位置”下拉框中选择区域。
   * 不要编辑“模板根 URI”或“参数根 URI”文本框。
   * 查看条款和条件，并单击“我同意上述条款和条件”复选框。
   * 单击“购买”按钮。
3. 等待部署完成。



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "跨本地和 Azure 基础结构的混合网络"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Azure 门户中的审核日志"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "用于监视 VPN 网络流量的性能计数器"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "示例 VPN 网络性能图"