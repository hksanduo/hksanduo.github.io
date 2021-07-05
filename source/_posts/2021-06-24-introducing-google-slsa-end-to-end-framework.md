---
title: "谷歌SLSA供应链框架介绍"
date:   2021-06-23  16:09:00 +0800
layout: post
tag:
- Supply Chain
categories:
- Security
---

谷歌SLSA供应链框架介绍

------

## 背景
软件供应链攻击事件（对软件包进行未经授权的修改）在过去的两年中呈上升趋势，并且被证明是影响所有用户的常见且可靠的攻击方式。软件开发和供应链部署都是相当复杂的，从源代码到构建再到发布，在整个工作流程中会存在众多威胁。虽然确实存在针对某些特定漏洞的单点解决方案，但没有全面的端到端框架来定义如何减轻整个软件供应链中的威胁，并提供合理的安全保障方案。面对近几个月令人大开眼界的损失数十亿美元骇客攻击（如：SolarWinds、Codecov)，如果软件开发人员和用户采用类似供应链的安全框架，其中一些攻击可能会被阻止或变得利用起来更加困难。
谷歌提出的解决方案是软件开发的供应链级别（Supply chain Levels for Software Artifacts 简称 SLSA，发音为“salsa”），这是一个端到端框架，用于保证整个软件供应链中组件的完整性。它的灵感来自于谷歌内部的“ Borg二进制授权系统”，它已经在谷歌内部使用了8年多，并且强制托管谷歌所有生产工作负载。SLSA的目标是改善行业状况，尤其是开源软件安全状况，进而抵御最要紧的供应链完整性威胁。使用SLSA，用户方便根据他们目前使用软件的安全状况做出明智的选择。

## SLSA用途
SLSA帮助防范常见的软件供应链攻击。下图展示了一个典型的软件供应链，包括可能发生在供应链各个环节的攻击示例。在过去的几年中，每种类型的攻击均发生过，不幸的是，随着时间的推移，软件供应链事件攻击仍在增加。
![20210624-01.png](/images/20210624-01.png)

| 序号 | 威胁类型 | 威胁案例 | SLSA防御 |
| --- | --- | --- | --- | 
| A |  将恶意代码提交到源存储库 | [Linux恶意上传带有漏洞的patch](https://lore.kernel.org/lkml/202105051005.49BFABCE@keescook/)：研究人员试图通过邮件列表上的补丁程序故意将漏洞引入Linux内核。| 双人审查可以审核大多数（但不是全部的）漏洞。|
| B | 恶意托管源代码管理平台 | [PHP官方git被攻陷](https://news-web.php.net/php.internals/113838)：攻击者破坏了PHP的自托管git服务器，并注入了两个恶意提交。| 防御较好的源代码平台增加了攻击者难度 |
| C | 使用正式流程构建，但源代码与源代码管理不匹配 | [Webmin](https://www.webmin.com/exploit.html):攻击者修改了构建工作流以使用与源代码管理不匹配的源文件。| 一个符合SLSA的构建服务器会识别实际使用的源代码，允许用户检测此类篡改。| 
| D | 被攻陷的构建平台 | [SolarWinds](https://www.crowdstrike.com/blog/sunspot-malware-technical-analysis/)：攻击者破坏了构建平台，并植入了一个在每次构建过程中注入恶意代码的程序。| 更高SLSA 级别要求构建平台具有[强有力的安全控制措施](https://github.com/slsa-framework/slsa/blob/main/build-requirements.md)，确保构建平台可以持久性并且难以被破坏 | 
| E | 使用恶意的依赖(即A-H，递归) | [事件流攻击](https://schneider.dev/blog/event-stream-vulnerability-explained/)：攻击者添加了一个无害的依赖项，然后更新该依赖并添加恶意代码。更新与提交给GitHub的代码不匹配（即攻击F）| 在所有依赖项中使用SLSA，可能会阻止特定的攻击，因为源代码可能会提示它不是由正确的构建环境构建的，或者源代码不是来自GitHub。|
| F |  上传不是由CI/CD系统生成的中间件 | [CodeCov](https://about.codecov.io/apr-2021-post-mortem/)：攻击者使用泄露的证书将恶意中间件上传到GCS存储库，用户可以从上面直接下载。| GCS存储库中，中间件的出处会显示中间件不是以预期的方式从预期的源构建的。|
| G | 恶意的软件仓库 | [攻击软件镜像站](https://theupdateframework.io/papers/attacks-on-package-managers-ccs2008.pdf)：研究人员测试几个流行的软件镜像站，这些镜像站可能会被用来托管恶意软件 | 与上面的（F）类似，恶意组件的出处会显示它们不是按照预期构建的，或者不是从预期的源repo构建的。|
| H | 欺骗用户使用恶意的软件包 | [包名抢注](https://blog.sonatype.com/damaging-linux-mac-malware-bundled-within-browserify-npm-brandjack-attempt) 攻击者上传了一个与原始文件名相似的恶意软件。|  SLSA不能直接解决这个威胁，但是通过控制源代码链接和增加其他解决方案来解决。|

## SLSA 介绍
在其当前状态下，SLSA是一套逐渐采用的安全准则，由业界一致认可。在最终形式中，SLSA在可执行性方面与最佳实践列表有所不同：它将支持自动创建可审核元数据，这些元数据可以输入到策略引擎中，以便为特定包或构建平台提供“SLSA认证”。SLSA被设计成持续的和可操作的，并且在每一步都提供最优措施。一旦一个组件达到了最高级别，用户就可以确信它没有被篡改，并且可以安全地追溯到源代码，这对于今天的大多数软件来说是很困难的，甚至是不可能的。

SLSA由四个级别组成，其中SLSA 4表示理想状态。较低级别表示具有相应增量完整性保证的增量里程碑。这些要求目前定义如下。
![20210624-02.png](/images/20210624-02.png)

* **SLSA 1**要求构建过程完全脚本化/自动化，并生成出处。出处是关于如何构建中间件的元数据，包括构建过程、顶级源代码和依赖关系。了解出处可以让用户做出基于风险的安全决策。尽管SLSA 1的源代码不能防止篡改，但它提供了基本级别的源代码识别，有利于漏洞管理。

* **SLSA 2**需要使用版本控制和生成经过身份验证出处的托管构建服务。这些额外的要求使用户对软件的来源有了更大的信心。在这个级别上，源代码在构建受信任服务过程中可以防止被篡改。**SLSA 2**还提供了到**SLSA 3**的简单升级途径。

* **SLSA 3** 进一步要求源代码和构建平台分别满足特定标准，以保证源代码的可审计性和完整性。设想有这么一个认证过程，通过这个过程，审计人员可以证明平台符合要求，用户就可以直接依赖它了。**SLSA 3**通过防御特定类型的威胁（如交叉编译污染），提供了比早期级别更强大的防篡改保护。

* **SLSA 4**目前是最高级别的，需要双人对所有变更进行审查，并且需要一个封闭的、可复制的构建流程。双人评审是一种行业最佳实践，可以发现错误并阻止不法行为。封闭的构建保证了源代码的依赖列表是完整的。可复制的构建，虽然不是严格要求的，但是提供了许多可审计性和可靠性的好处。总的来说，**SLSA 4**让用户对软件没有被篡改有绝对的信心。

用户可以在 [GitHub 存储库](https://github.com/slsa-framework/slsa)中找到有关这些建议级别的更多详细信息，包括相应的源和构建或来源要求。

## 论证
今天，我们发布了 SLSA 1 来源生成器（repo、marketplace）的来论证我们的观点。 这将允许用户在他们的构建组件的同时创建和上传出处，从而实现SLSA 1级别要求。如果要使用它，请将以下代码段添加到您的工作流程中：
```
- name: Generate provenance  
uses: slsa-framework/github-actions-demo@v0.1  
with:    artifact_path: <path-to-artifact/directory>
```
未来，我们计划与流行的源、构建和打包平台合作，以尽可能轻松地达到更高级别的 SLSA。 这些计划包括在构建系统中自动生成出处、在包存储库中本地传播来源以及跨平台添加安全功能。 我们的长期目标是提高整个行业的安全标准，以便默认期望的安全级别就是更高级别的 SLSA 安全标准，进而减少软件生产商的工作量。

## 总结
SLSA是一个用于端到端软件供应链完整性的实用框架，基于一个在世界上最大的软件工程组织中被证明可以大规模工作的模型。对于大多数项目来说，实现最高级别的SLSA可能很困难，但是较低级别的SLSA所认可的持续改进将在很大程度上提高开放源代码生态系统的安全性。


## 参考
- [https://security.googleblog.com/2021/06/introducing-slsa-end-to-end-framework.html](https://security.googleblog.com/2021/06/introducing-slsa-end-to-end-framework.html)【Introducing SLSA, an End-to-End Framework for Supply Chain Integrity】
- [https://github.com/slsa-framework/slsa](https://github.com/slsa-framework/slsa)【slsa】
- [https://security.googleblog.com/2021/06/introducing-slsa-end-to-end-framework.html](https://security.googleblog.com/2021/06/introducing-slsa-end-to-end-framework.html)【Introducing SLSA End to End Framework】
