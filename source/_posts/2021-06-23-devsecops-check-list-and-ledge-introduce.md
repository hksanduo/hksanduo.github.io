---
title: "DevOps元素周期表及DevSecOps检查清单介绍"
date:   2021-06-22  17:58:00 +0800
layout: post
tag:
- DevSecOps
categories:
- Security
---

DevOps元素周期表及DevSecOps检查清单介绍

------
## DevOps元素周期表介绍
Ledge （from Know-Ledge，意指承载物）知识和工具平台，是我们基于在 ThoughtWorks 进行的一系列 DevOps 实践、敏捷实践、软件开发与测试、精益实践提炼出来的知识体系。它包含了各种最佳实践、原则与模式、实施手册、度量、工具，用于帮助您的企业在数字化时代更好地发展。

DevOps 运动的主要特点是强烈倡导对构建软件的所有环节 (从集成、测试、发布到部署和基础架构管理) 进行全面的自动化和监控。DevOps 元素周期表 协助选择 DevOps 工具，由于工具​列表会不断更新，请参考最新版本。

![20210623-01.png](/images/20210623-01.png)

您可以在这个平台上看到：
* **工具元素周期表**。帮助您进行数字化时代的 DevOps 工具选型。
* **DevOps 设计工具**。帮助您设计组织内的 DevOps 流程，涵盖了流程、人、工具、制品等等。
* **案例学习**。从社区的知识库中，我们总结了传统企业走向 DevOps 的经验，并浓缩到易于使用的内容和材料中。
* **最佳实践**。我们从海量的 DevOps 内容中，提炼出了一系列的最佳实践，以更好地帮助企业进行 DevOps 实践。
* **模式与原则**。基于我们的实践，我们提炼了位于它背后的模式与原则，帮助个人和组织更好地了解 DevOps 文化。
* **实施手册**。只凭实践与原则，无法让中小型 IT 团队进行 DevOps 转型，所以我们准备了详实的实施手册，以帮助您一步步前进。
* **度量**。KPI - 度量、度量 - KPI、KPI - 度量，帮助您更好地度量 DevOps 转型情况。
* **报告**。我们尝试从丰富地 DevOps 报告中，提炼出有用的实践和工具。
* Mobile DevOps。我们相信移动应用的 DevOps 改进，才是大多数公司的挑战。
* **工具**。工具，工具，工具是最好的生产力，工具比人的记忆力更加可靠。
* **解决方案**。即某一 DevOps 厂商的解决方案。（不收费，为了 Ledge 项目的可持续性，仅开放给将 Ledge 列为合作伙伴的厂商）

在线使用：https://devops.phodal.com/

ledge github 地址：https://github.com/phodal/ledge

## DevSecOps 介绍

DevOps是Development和Operations的缩写，而DevSecOps则是Development、Security和Operations的缩写。John Willis(DevOps Cookbook合著者之一)称Patrick Debois是DevOps的缔造者，而DevOps的传道者们认为，DevSecOps的基本理念是让每一个解决方案、开发测试的多个跨部门协作人员都融入开发运维和安全的理念，并正确地理解DevOps的做法与含义。也就是说DevSecOps是一个项目组织方式，因此并不不存在DevOps者，更不是说将开发环境和生产环境整合在一起的意思，DevSecOps是一个群体做法，核心理念是：“安全是整个IT团队（包括开发、运维及安全团队）所有成员的责任，需要贯穿整个业务生命周期（从开发到运营）的每一个环节。”
![20210623-02.png](/images/20210623-02.png)

将安全整合到DevOps的“DevSecOps”会带来思维方式、流程和技术的整体变化。安全和风险管理的领导者，必须坚持合作，DevOps的敏捷性意味着其在开发过程是无缝的和透明的，同理，“DevSecOps”中的“安全”也应当是沉默的。

企业在执行DevSecOps时通常会面临以下几个方面的挑战：

* 作为一名安全或风险管理人员，应当很清楚企业的业务发展是核心，因此向客户提供新的IT功能的产品开发团队才是王道，而不是信息安全或IT团队。
* 信息安全工作必须适应开发过程和相应的工具，而不是背道而驰。
* 组织使用DevOps生产新的应用和服务，DevOps相关的过程和工具也有责任遵照公司对其他应用程序的要求，产生安全且符合要求的代码。要求应用程序达到百分百地安全是不现实的，企业一旦陷入“将安全漏洞的数量降为零”的错误追求中，安全测试的负担识别加重，且很可能成为业务发展的一个障碍。

因此，问题的关键是风险的控制和管理，而非单纯的追求安全。想要确保应用程序和数据安全，在进行安全和风险管理（Security and risk management，SRM）时，应当注意：

* 将安全和合规测试无缝地整合到DevSecOps中，使开发者专注工作在持续集成和持续部署工具链环境中。
* 持续的扫描已知的漏洞和配置错误，对象包括所有开源的和第三方组件。理想情况下，管理并维护一套完整的资产清单，便于完成对所有软件组件的分析。
* 不要尝试删除自定义代码中所有未知的漏洞，相反，应把开发人员看作放在最严谨和最自信的人。
* 鼓励尝试使用新的工具或方式，以减少与开发人员的摩擦，例如使用交互式应用安全测试（IAST）来取代传统的静态和动态测试。
* 把信息安全团队完美地融入DevOps进程中。
* 使用相同的统一的规范来处理所有自动化脚本、模板、图像和设计，并保证安全工作覆盖了所有的源代码。

## DevSecOps 检查清单
Ledge提供DevSecOps检查清单，可以作为参考：
地址：https://devops.phodal.com/checklists/devsecops
数据来源于新思科技（Synopsys）的 《Building your DevSecOps pipeline:5 essential activities 》

下图列述了DevSecOps流水线的五个关键节点，每一个节点的用途，优势，关键推动因素和用例：
![20210623-03.png](/images/20210623-03.png)
 
### DevSecOps 检查清单

> 以下内容主要参考[@hylerrix](https://github.com/hylerrix) 大佬翻译的[DevSecOps检查清单](https://devops.phodal.com/checklists/devsecops)，发现部分翻译过于生硬，无法准确表达原文意思，鄙人斗胆重新翻译和总结清单列表，如果有所不足，请及时斧正，我将不胜感激。

DevSecOps 清单主要目的:

* 让开发人员专注于缺陷修复
* 利用IDE中的安全组件为研发人员的编码提供安全指导，在项目研发初期匹配源代码分析策略
* 促使组织在软件开发生命周期中安全左移
* 协助安全团队维护合规并集中的持续跟踪残余安全风险
* 允许 DevSecOps 团队集成检测工具链，降低安全成本

### 清单列表

- 预提交检查：将源码更改提交到源码库之前查找并修复常见的安全问题
    - 使用IDE中SAST组件进行扫描，在程序编码过程中为研发人员提供安全编码指导 
    - 威胁建模，安全需求分析
    - 架构风险分析
    - 手动代码审查
    - 项目配置文件审查
    - 通过电子邮件通知应用程序安全团队或软件安全组（SSG），研发人员已经提交核心代码

- 提交时检查：生成并执行基本的自动化应用测试，测试结果快速反馈给提交变更的开发人员
    - 编译并构建源代码代码
    - 运行SAST工具：静态应用程序安全检测（SAST）
    - 自动化安全测试并收集指标
    - 构建被破坏时向相关团队发出警报
    
- 构建时检查：执行应用程序的高级自动化测试。这包括更深层次的 SAST、开源组件管理、安全测试、基于风险的安全测试、使用 PGP 签名软件并存储到软件仓库当中。
    - 使用更全面的 SAST 规则集进行扫描：例如 OWASP Top 10
    - SCA 供应链风险检测
    - 基于风险自动化安全测试并收集关键指标
    - 构建失败时及时提示相关团队
    
- 测试时检查：从制品库中选择最新最优版本进行构建，并将其部署到类生产或测试环境中，所有测试，包括功能、集成、性能、高级 SAST 和 DAST，都在此版本上执行。
    - 使用全量的 SAST 规则集进行扫描
    - 配置 DAST 规则并使用 DAST 工具对目标系统进行测试
    - 恶意代码检测
    - 模糊测试
    - 配置并自动化将制品库中最新最优版本进行部署，部署完成后进行自动化扫描、收集关键指标
    - 构建失败时及时提示相关团队
    
- 部署时检查：提供持续的保障，确保对生产环境的更改不会引起安全问题
    - 部署前
        - 自动化配置管理
        - 自动化配置运行环境
    - 部署后
        - 持续监控，自动收集应用安全指标
        - 指定安全扫描计划
        - 定期进行漏洞扫描
        - 协助安全运营SRC
        - 构建应急响应流程
        - 持续为DevSecOps团队提供威胁预警

## 参考：
- [https://www.synopsys.com/software-integrity/polaris.html](https://www.synopsys.com/software-integrity/polaris.html)【Polaris Software Integrity Platform】
- [https://www.synopsys.com/blogs/software-security/devsecops-pipeline-checklist/](https://www.synopsys.com/blogs/software-security/devsecops-pipeline-checklist/)【Building your DevSecOps pipeline: 5 essential activities】
- [https://mp.weixin.qq.com/s/s6P7Ucv1V2oWknfkvucMOw](https://mp.weixin.qq.com/s/s6P7Ucv1V2oWknfkvucMOw)【5个关键SAST活动，构建你的DevSecOps流水线 | IDCF】
- [https://devops.phodal.com/checklists/devsecops](https://devops.phodal.com/checklists/devsecops)【
DevSecOps 检查清单】
- [https://devops.phodal.com/home](https://devops.phodal.com/home)【ledge home】
- [https://www.redhat.com/zh/topics/devops/what-is-devsecops](https://www.redhat.com/zh/topics/devops/what-is-devsecops)【DevSecOps 和 DevOps 有什么区别 ?】


