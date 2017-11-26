---
title: "Azure 参考体系结构"
description: "适用于 Azure 上的常见工作负荷的参考体系结构、蓝图和规范性实施指南。"
layout: LandingPage
ms.openlocfilehash: eaff531c28faeb0aac774acf70d4334fb4e1f319
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="azure-reference-architectures"></a>Azure 参考体系结构

我们的参考体系结构已根据方案进行编制，相关的体系结构已分组到一起。 每个体系结构包括建议的做法，以及有关可伸缩性、可用性、可管理性和安全性的注意事项。 大多数体系结构还包括一个可部署的解决方案。

<section class="series">
    <ul class="panelContent">
    <!--Windows VM -->
    <li>
        <a href="./virtual-machines-windows/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-windows/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Windows VM 工作负荷</h3>
                            <p>此系列文章依次介绍有关运行单个 Windows VM、多个负载均衡的 VM 和多区域 N 层应用程序的最佳做法。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Linux VM -->
    <li>
        <a href="./virtual-machines-linux/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-linux/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Linux VM 工作负荷</h3>
                            <p>此系列文章依次介绍有关运行单个 Linux VM、多个负载均衡的 VM 和多区域 N 层应用程序的最佳做法。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Hybrid network -->
    <li>
        <a href="./hybrid-networking/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>混合网络</h3>
                            <p>此系列文章演示用于在本地网络与 Azure 之间创建网络连接的选项。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- DMZ -->
    <li>
        <a href="./dmz/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>外围网络</h3>
                            <p>此系列文章介绍如何创建外围网络，以保护 Azure 虚拟网络与本地网络或 Internet 之间的边界。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Identity -->
    <li>
        <a href="./identity/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./identity/images/adds-extend-domain.svg" height="140px" >
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>身份管理</h3>
                            <p>此系列文章演示用于将本地 Active Directory (AD) 环境与 Azure 网络集成的选项。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Managed web app -->
    <li>
        <a href="./app-service-web-app/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>应用服务 Web 应用程序</h3>
                            <p>此系列文章介绍有关使用 Azure 应用服务的 Web 应用程序的最佳做法。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    </ul>

<ul class="panelContent cardsI">
<li>
    <a href="./sharepoint/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sharepoint/images/sharepoint.svg" alt="SharePoint Server 2016" height="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>SharePoint Server 2016 场</h3>
                    <p>使用 SQL Server Always On 可用性组在 Azure 上部署和运行高可用性 SharePoint Server 2016 场。</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>

<li>
    <a href="./sap/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sap/images/sap.svg" alt="SAP NetWeaver and SAP HANA" width="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>SAP NetWeaver 和 SAP HANA</h3>
                    <p>在 Azure 上的高可用性环境中部署和运行 SAP NetWeaver 与 SAP HANA。</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>
</ul>


</section>

