---
title: "Garter-《软件工程领导者如何降低软件供应链风险》分享"
date:   2022-05-11  13:52:00 +0800
layout: post
tag:
- Software Supply Chain
categories:
- Security
---

Garter-《软件工程领导者如何降低软件供应链风险》分享

------

## 背景
Garter发布《软件工程领导者如何降低软件供应链分享》的调研文章，有幸研读，感觉对供应链安全的理解更深了，这里把里面的内容进行整理和分析，其中翻译不准确，不乏博主个人观点，请见谅。

## 概述
调研关键发现共分为以下三点：    
* 恶意代码注入威胁的增加使得保护内部代码和外部依赖项（开源和商业）变得至关重要。    
* 软件构建和交付流程受到破坏的后将导致泄露机密或其他敏感数据和代码被篡改。    
* 未能强制执行最低权限和扁平化网络架构（对应的是分层网络架构或隔离网络）会使攻击者横向移动到生产环境，从而使企业面临更大的风险。

根据关键发现，调研给出以下建议，软件工程领导者应该与他们的安全和风险管理团队合作，具体分为以下三点：
* 在整个交付生命周期中，通过强制执行健全的版本控制策略、使用制品库来存储受信任的组件以及管理供应商风险，进而来保护内部和外部代码的完整性。
* 通过在 CI/CD 中配置安全工具，保护机密以及代码和对容器镜像签名，最终来强化软件交付管道流程。
* 通过使用最小特权原则和零信任安全模型来管理资源的访问，保护软件工程师的操作环境。

客户关注的供应链风险维度分为以下三点：
* CI/CD系统的威胁
* 恶意代码注入风险
* 包含漏洞和恶意代码的依赖项

> 从SolarWinds (2020)、NetBeans IDE (2020)、Kaseya (2021) 和 Codecov (2021)代表软件供应链攻击的这四个突出示例来看。Gartner 认为，到 2025 年，全球 45% 的组织将受到软件供应链遭受攻击，比 2021 年增加了三倍。


## 潜在软件供应链安全风险
软件供应链攻击是在软件开发、交付和使用的任何阶段破坏软件或其依赖项之一的行为。 尽管每种情况下精确的攻击向量可能不同，但攻击者通常会未授权访问开发环境和基础设施，包括版本控制系统、制品仓库、开源软件存储库、持续集成管道、构建服务器或应用程序服务器等，这导致攻击者可以通过修改源代码、脚本和依赖软件包，并建立后门窃取受害者环境中的数据。攻击包括但不仅限于外部攻击者，也可能来自内部威胁。

![潜在软件供应链安全风险](/img/20220511-01.png)
### 内部和外部代码风险（开源组件）
#### 开发阶段
* 证书丢失
* 密钥硬编码
* 密钥丢失
* 弱加密
* 代码注入
* 固件串改

#### 集成阶段
* 开源组件漏洞
* 包名抢注
* 命名空间冲突
* 不安全的第三方SDK和API

### 交付流程风险
* 签名证书被篡改
* 自动化脚本被篡改

### 生产环境风险
* 未授权访问
* 二进制代码逆向
* 表单劫持
* 提权
* 网络端口扫描
* 更新劫持
......

## 软件开发和部署中降低供应链安全风险的最佳安全开发实践 
针对这三类风险，Gartner给出了部分应对措施和建议。
![软件开发和部署中降低供应链安全风险的最佳安全开发实践](/img/20220511-02.png)

### 内部和外部代码风险应对措施
软件研发团队使用版本控制系统（git/svn等）和制品库来维护内部代码开发和外部制品分发，如果未将这些版本控制系统和制品库进行安全控制，可导致源代码和组件被篡改或劫持。Garter推荐以下三种方式来保障代码和组件的完整性。
* 严格的版本控制
* 受信任的组件库
* 第三方风险管理

#### 严格的版本控制
基于 Git 的版本管理系统（VCS），包括 BitBucket、GitHub 和 GitLab，提供源代码托管和访问权限控制能力，软件工程团队必须启用访问策略控制、分支保护和敏感扫描功能。 这些控件策略默认情况下不启用，必须进行配置。
![健全的版本控制策略](/img/20220511-03.png)

由于部分研发人员安全意思孱弱，无意将密钥信息，证书等上传到github或者gitlab上，任何能访问到源代码的用户均可获取相关敏感信息,Garter推荐部分基于Git存储库的敏感信息扫描工具
| 开源工具 | 厂商 |
| --- | --- |
| git-secrets: Open sourced by AWS Labs | Github Secrets Scanning |
| Repo Supervisor: Open sourced by Auth0 | GitLab Secret Detection |
| truffleHog: Searches for secrets in Git repos | Bitbuchet Secrets Scan |
| Gitleaks: Scans repos and commits for secrets | GitGuardian |
| Deadshot: Open sourced by Twilio | SpectralOps |


#### 受信任的组件库
建议使用制品（或容器镜像）存储库、软件成分分析工具和源代码扫描工具，其中制品存储库可以对组件进行分发版本控制，软件成分分析工具（SCA）可以对当前组件及源代码进行成分分析，获取项目的依赖风险信息，并及时进行升级修复，对于源代码扫描工具(SAST)，可以对用户自己编写的源代码进行合规和安全行分析。     
常见的制品库管理工具如下：
| 组件管理平台 | 容器管理平台 |
| --- | --- |
| Azure Artifacts | Azure Container Registry |
| AWS CodeArtifacts | Amazon ECR |
| GitHub | CNCF Harbor |
| GitLab | Docker Trusted Registry |
| Google Artifact Registry | GitHub |
| JFrog Artifactory | GitLab |
| Sonatype Nexus Repository | Google Container Registry |
| Tidelift Catalogs | JFrog Artifactory |
| | Red Hat Quay |

常用企业级的SAST工具如下（这里没必要抬杠，工具有上百种，只列举常见的SAST平台，数据来源互联网、个人审计工作和客户反馈，杠精勿扰）：
| 工具名称 | 类型 | 公司 | 地址 |
| --- | --- | --- | --- |
| Fortify sca | 商业 | Hp Security | https://www.microfocus.com/zh-cn/products/static-code-analysis-sast/overview |
| checkmarx sast | 商业 | checkmarx | https://checkmarx.com/product/cxsast-source-code-scanning/ |
| Sonarqube | 开源 | Sonaqube | https://www.sonarqube.org/ |
| Veracode Static Analysis (SAST) | 商业 | Veracode | https://www.veracode.com/products/binary-static-analysis-sast |
| Coverity | 商业 | synopsys | https://www.synopsys.com/software-integrity/security-testing/static-analysis-sast.html |
| 奇安信代码卫士 | 商业 | qax | https://www.qianxin.com/product/detail/pid/14 | 
| DMSCA | 商业 | 端玛科技 | http://www.dumasecurity.com/goods.html |
| AppScan Source | 商业 | HCL AppScan | https://www.hcltechsw.com/appscan/offerings/source |

* 对于其他SAST工具，比如codeql之类小众优秀的工具，这里不做推荐，暂时无法满足企业级需求，对于其他国产SAST工具，这里不做说明和推荐，勿杠。

常用企业级SCA工具（同上）：
| 工具名称 | 类型 | 公司 | 地址 |
| --- | --- | --- | --- |
| Black Duck Software Composition Analysis | 商业 | synopsys | https://www.synopsys.com/software-integrity/security-testing/software-composition-analysis.html |
| nexus lifecycle | 商业 | Sonatype | https://www.sonatype.com/products/open-source-security-dependency-management?topnav=true |
| Veracode sca | 商业 | Veracode | https://www.veracode.com/products/software-composition-analysis |
| Jfrog Xray | 商业 | Jfrog | https://jfrog.com/xray/ |
| Mend sca | 商业 | Mend | https://www.mend.io/sca/ |
| checkmarx sca | 商业 | Checkmarx | https://checkmarx.com/product/cxsca-open-source-scanning/  |
| Dependency Scanning | 商业 | Gitlab | https://docs.gitlab.com/ee/user/application_security/dependency_scanning/ |
| Dependency Track | 开源 | Owasp | https://dependencytrack.org/ |
| 雳鉴SCA | 商业 | 默安 | https://www.moresec.cn/product/sdl-sca |
| 悬镜源鉴OSS | 商业 | 悬镜 | https://oss.xmirror.cn/ | 
| CoBot | 商业 | 北大库博 | https://www.pkuse.com.cn/multi/521.html |

The Forrester Wave SCA 
![Forrester Wave SCA Analysis ](/img/20220511-04.jpeg)

![Forrester Wave SCA Analysis ](/img/20220511-05.jpeg)
> TOP 10 SCA工具中有5款支持软件包开源软件SCA检查能力(synopsys/ Sonatype/ Veracode/ Jfrog/ GitLab)，其他工具只支持源代码SCA检查能力。
> 5款支持软件包SCA检查工具中，对C/C++、Java、.Net语言支持的比较好，但对Golang、python、JavaScript语言支持能力偏弱，比如：synopsys支持的组件对象中前面3种语言占大头90%+，相应的检测率也高，而Golang语言的组件检出率则低很多。

#### 第三方风险管理

常见的与第三方软件相关的两种供应链风险：
1. 由于第三方或开源依赖项中的已知漏洞而导致的风险
2. 由于外部采购软件中植入后门/恶意软件的风险 

Garter提供的解决措施：
1. 检查第三方是否遵循标准或获得认证
2. 检查供应商是否有必要的措施开展SDLC流程
3. 供应商遵循什么流程来修补自己的软件及其依赖项
4. 第三方软件的更新机制是否受到保护
5. 对于第三方软件或依赖项中发现的漏洞的SLA（软件服务协议）是什么？

软件供应链安全评估框架和标准
| 评估名称 | 简介  | 地址 |
| --- | --- | --- | 
| Evaluating Your Supply Chain Security | A Checklist by Cloud Native Computing Foundation (CNCF) | https://github.com/cncf/tag-security/blob/main/supply-chain-security/supply-chain-security-paper/secure-supply-chain-assessment.md | 
| NIST Secure Software Development Framework | Secure Software Development Framework (SSDF) Version 1.1:Recommendations for Mitigating the Risk of SoftwareVulnerabilities | https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.04232020.pdf |
| NIST, Security and Privacy Controls for Information Systems and Organizations | Security and Privacy Controls. | https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf |
| Ul 2900 for IoT Certification | UL 2900 series of standards was developed as part of UL’s Cybersecurity Assurance Program which provides manufacturers testable and measureable criteria  | https://www.cybersecuritysummit.org/wp-content/uploads/2017/10/4.00-Justin-Heyl.pdf |
| ISO/IEC 27034 | Information technology — Security techniques — Application security | https://www.iso.org/standard/44378.html |

软件供应链开源项目安全评估
| 评估名称 | 简介  | 地址 |
| --- | --- | --- | 
| Open Source Insights | Open Source Insights 会展示软件包的相关信息，而无需用户预先安装软件包。开发人员可以看到该依赖包对项目的重要程度，依赖组件流行程度，查找源代码的链接，然后决定是否应安装该组件。 | https://opensource.googleblog.com/2021/06/introducing-open-source-insights-project.html | 
| OSSF Scorecard  | OSSF Scorecard 是一个通过多种维度来评估开源项目的安全性的工具. | https://github.com/ossf/scorecard | 
| Supply chain Levels for Software Artifacts (SLSA, pronounced “salsa”) | 确保软件供应链中组件完整性的端到端保护框架。 | https://security.googleblog.com/2021/06/introducing-slsa-end-to-end-framework.html | 

### 交付流程风险应对措施
#### 1. 使用密钥管理工具：

密钥管理通过规范的方法来管理和保护如凭证、密码、API 令牌和证书等机密。 Garter建议使用密钥管理工具来自动创建、存储、检索和撤销秘密。 这有助于避免在源代码、配置文件和基础设施自动化脚本中嵌入（硬编码）密钥信息等。
常见的密钥管理工具如下：

| 场景 | 密钥管理工具  |
| --- | --- | 
| 与平台无关 | Akeyless </br> CyberArk Conjur  </br> Thycotic Secrets Vault </br> HashiCorp Vault | 
| 云厂商提供 | AWS Secrets Manager </br> Azure Key Vault </br> GCP Secret Manager | 
| 容器原生环境 | Kubernetes Secrets (etcd) </br> Sealed Secrets | 
| 配置管理 |  Ansible Vault </br>  Chef Data Bag </br> Puppet Hiera | 

#### 2. 通过签名和散列函数来验证源码的完整性
1. 哈希和签名可用于验证源代码和二进制文件的完整性。     
    VCS（版本控制系统）在提交时会对单个文件生成哈希，该哈希可以用于验证文件在传输过程中是否被篡改。另外，编译器在编译时也生成哈希，可以通过将提交时和编译时的哈希进行对比，以保证代码在提交阶段和编译阶段未被篡改。
2. 提交签名     
    由于哈希不能验证来源，所以需要通过VCS的签名提交功能来验证提交代码人的身份。
3. 容器签名     
    当前越来越多的系统开始通过容器进行部署，因此需要保证容器未被篡改过。即使用户的公司或组织自行构建和维护内部镜像，Gartner 也建议对容器镜像进行签名。这是因为第三方代码或依赖项中的任何问题都会影响正在运行的应用程序的安全状况

![Kubernetes 容器镜像漏洞的传播 ](/img/20220511-06.png)

容器签名的工具如下： 
| 工具名称 | 介绍  | 地址 |
| --- | --- | --- | 
| Grafeas | Grafeas定义用于管理有关软件资源的元数据的API格式，例如容器镜像，VM镜像，JAR包和脚本，为构建代码到容器供应链的组件，包括组件的来源、漏洞、依赖关系等提供了一个集中的知识库。 | https://github.com/grafeas/grafeas | 
| Kritis | Kritis 是一个Kubernetes准入控制器，它在运行时运行由Kubernetes集群管理员定义的策略检查，然后根据镜像中的漏洞或镜像不是从可信来源获得的，批准或拒绝要启动的容器。 | https://github.com/grafeas/kritis | 
| Kritis Singer | 是一个为容器图像创建认证的命令行工具。 | https://github.com/grafeas/kritis/blob/master/docs/signer.md | 
| Cosign | Cosign对容器图像签名。Cosign是由Linux基金会主办的sigstore项目的一部分。 | https://github.com/sigstore/cosign | 


#### 3. 在CI/CD管道中配置安全控制
攻击者可以通过攻击CI/CD系统来绕过对代码的检查和扫描，因此需要保证CI/CD系统的安全性。可以通过对CI/CD系统的安全配置来防范风险。常见工具：Apiiro, Argon,Cycode, Garantir, GrammaTech, JFrog (Vdoo)、RunSafe Security。     
保护 CI/CD 管道有以下两点措施： 
1. 可复现的构建过程
确保相同的代码始终构建相同的软件，其中包含以下三点原则： 

    确定性构建：确保相同的源代码必须编译构建相同的软件      
    强化构建工具：构建管道中的工具是安全稳定的且不可更改的      
    可验证输出：能够检测和验证预期构建和实际构建之间的差异      

2. 在构建管道中创建不可变的、可验证的制品签名任务
支持在管道运行期间生成制品的签名，以确保一致性并在管道执行结束时验证出处。

对于IDE的保护，可以使用基于浏览器的IDE（远程开发桌面），防止开发人员本地安装的IDE工具存在风险。

### 生产环境风险应对措施
操作环境风险是指在整个软件开发过程中所涉及的操作环境的风险，如开发环境、代码仓库、流水线系统、测试环境等。针对操作环境的风险防范措施包括：
#### 1. 最小权限访问策略
网络上连接着不同设备，特权提升允许攻击者一旦获得对一个系统的访问，就可以渗透到其他机器和服务中。此外，除非实施了正确的访问控制，否则受到攻击的可执行文件可能会未经授权与其他核心系统建立连接。因此，Garter建议使用基于角色的身份验证和授权、使用零信任安全模型的自适应访问和特权访问管理。      
![管理用户访问和特权帐户的方法](/img/20220511-07.png)

#### 2. 机器身份管理
对分布式应用、云原生、API服务等架构体系的使用使得应用系统的部署变得更细颗粒度且数量增多。机器身份管理是对主机、容器、虚拟机、应用程序、数据库、API服务等机器的身份进行统一管理、统一验证，确定机器的唯一身份。包括：密钥管理、证书管理等。常见的机器身份识别系统如下：
| 使用场景 | 作用域  | 应用名称 |
| --- | --- | --- | 
| 静态数据加密，对称密钥管理 | 密钥管理系统 | Akeyless <br> AWS <br> KMS <br> Azure <br> Twilio (Ionic) <br> Fortanix <br> PKWARE <br> Thales and Townsend Security | 
| 存储DevOps管道中使用的密钥，将机器标识发送给容器 | 机密管理 | Akeyless <br> AWS <br> Microsoft Azure <br> BeyondTrust <br> CyberArk <br> Fortanix <br> Google Cloud Platform (GCP) <br> HashiCorp <br> ThycoticCentrify | 
| 用于代码签名的身份验证、加密和签名 | PKI和证书管理 | AppViewX <br> AWS <br> DigiCert <br> Entrust <br> GlobalSign <br> Keyfactor <br> Microsoft <br> The Nexus Group <br> Sectigo <br> Venafi | 
| 发现和控制对关键系统的特权访问 | 特权访问管理 | Akeyless <br> BeyondTrust <br> Broadcom <br> CyberArk <br> One Identity <br> ThycoticCentrify | 

#### 3. 异常检测和自动响应
软件工程领导者必须与安全和风险团队密切合作，以了解和定义其开发平台和工具的预期行为，以便他们能够实时检测异常。 例如，EDR、CWPP、NDR 或 squery 等工具可以监控系统异常。 构建系统，包括软件工程师使用的 PC，不应免除 EPP/EDR 保护。异常检测和响应在容器原生、基于 GitOps 的部署中尤其重要，可以自动化部署完整的代码到容器工作流程。 尽管处于开发阶段的容器镜像扫描工具有助于检测已知漏洞，但软件工程团队必须部署相关工具来可视化容器流量、识别集群错误配置并对异常容器行为和安全事件进行监测。        
对异常活动进行监测，并及时响应和处理有以下四点：
1. 可执行文件与访问控制系统创建不必要的连接
2. 特定机器上的进程、线程以及CPU和内存的利用率增加
3. 针对网络访问、存储库的上传及下载、非常用目录的流访问量激增
4. 监控软件的异常告警（SIEM、EPP、CASB）

## 总结
从SolarWinds (2020)、NetBeans IDE (2020)、Kaseya (2021) 和 Codecov (2021)攻击案例来看，软件供应链安全攻击范围较广，难度较高，周期较长，影响较远，防护较弱。根据信息安全的“木桶理论”：“信息的安全就像一个‘木桶’，整体的安全性取决于最薄弱的一个环节，否则即使其它方面做得再强，但在某一方面留下一个漏洞，也可能被他人利用，导致信息的失窃。”供应链当前安全现状也如此。为了提升软件供应链安全，建议软件工程团队根据自身情况，循序渐进提升软件供应链安全能力。


## 参考
- [https://www.gartner.com/en/documents/4003625](https://www.gartner.com/en/documents/4003625)【How Software Engineering Leaders Can Mitigate Software Supply Chain Security Risks】
- [https://cloud.tencent.com/developer/article/1839537](https://cloud.tencent.com/developer/article/1839537)【企业级静态代码分析工具清单】
- [https://www.gartner.com/reviews/market/software-composition-analysis-sca](https://www.gartner.com/reviews/market/software-composition-analysis-sca)【Products In Software Composition Analysis (SCA) reviews Market】
- [https://sudonull.com/post/27911-Forrester-Research-A-Comparison-of-Ten-Top-Software-Composition-Analysis-Vendors](https://sudonull.com/post/27911-Forrester-Research-A-Comparison-of-Ten-Top-Software-Composition-Analysis-Vendors)【Forrester Research: A Comparison of Ten Top Software Composition Analysis Vendors】
- [https://blog.csdn.net/m0_50579386/article/details/123507873](https://blog.csdn.net/m0_50579386/article/details/123507873)【国内外软件成分分析SCA产品评测】
- [https://developer.aliyun.com/article/738408](https://developer.aliyun.com/article/738408)【Kubernetes 时代的安全软件供应链】