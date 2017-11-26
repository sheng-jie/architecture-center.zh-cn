---
title: "将本地网络连接到 Azure"
description: "本地网络与 Azure 之间的安全可靠网络连接的建议体系结构。"
layout: LandingPage
ms.openlocfilehash: 0707d17295e338af0176bd0cea806615ef05f9ad
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure"></a>将本地网络连接到 Azure

这些参考体系结构介绍了有关在本地网络与 Azure 之间创建可靠网络连接的成熟做法。 [应该选择什么？](./considerations.md)

<ul class="panelContent">
    <li>
        <a href="./vpn.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/vpn.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>VPN</h3>
                            <p>使用站点到站点虚拟专用网络 (VPN) 将本地网络扩展到 Azure。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ExpressRoute</h3>
                            <p>使用 Azure ExpressRoute 将本地网络扩展到 Azure</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute-vpn-failover.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute-vpn-failover.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>使用 ExpressRoute 和 VPN 实现故障转移</h3>
                            <p>使用 Azure ExpressRoute 和 VPN 作为故障转移连接，将本地网络扩展到 Azure。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./hub-spoke.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/hub-spoke.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>中心辐射型拓扑</h3>
                            <p>中心是本地网络的中心连接点。 分支是与中心对等互连的 VNet，可用于隔离工作负荷。 </p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
</ul>

