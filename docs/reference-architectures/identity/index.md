---
title: "身份管理"
description: "介绍并比较可用于在跨本地/Azure 云边界的混合系统中管理标识的各种方法。"
layout: LandingPage
ms.openlocfilehash: de98ee30306f5e712759ab7140bd430cb6d4cd75
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="identity-management"></a>身份管理

这些参考体系结构演示用于将本地 Active Directory (AD) 环境与 Azure 网络集成的选项。 <br/>[应该选择什么？](./considerations.md)

<section class="series">
    <ul class="panelContent">
    <!-- Integrate with Azure Active Directory -->
<li style="display: flex; flex-direction: column;">
    <a href="./azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/azure-ad.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>与 Azure Active Directory 集成</h3>
                        <p>使用 Azure AD 将本地 Active Directory 域与林集成。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Extend AD DS to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>将 AD DS 扩展到 Azure</h3>
                        <p>使用 Active Directory 域服务将 Active Directory 环境扩展到 Azure。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Create an AD DS forest in Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adds-forest.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>在 Azure 中创建 AD DS 林</h3>
                        <p>在 Azure 中创建受本地林中的域信任的独立 AD 域。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Extend AD FS to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adfs.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>将 AD FS 扩展到 Azure</h3>
                        <p>在 Azure 中使用 Active Directory 联合身份验证服务进行联合身份验证和授权。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>