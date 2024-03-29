---
title: "SDL最佳实践原则"
date:   2021-07-05  17:54:00 +0800
layout: post
tag:
- SDL
categories:
- Security
---

SDL最佳实践原则

------
SDL是一个方法论,只是一种手段，而不是目的，安全左移理念,目的是设计与交付更安全的软件,来保护公司及用户资产,同时降低安全的成本;如果盲目为了SDL而SDL,完全照搬微软SDL流程,结果必然是悲惨的,就算你能完全实现,那最直接的就是没业务了,公司全员搞安全了... 因地制宜,提升企业自身安全能力，只有适合自己的才是最好的,我们做SDL其实真正花时间去思考解决的就是找到合适的方法将SDL方法论在客户侧落地来达到企业的安全目标,其中的顺序我们要明确,通俗来说:有些弯路是必须要走的。以下是个人总结10点SDL最佳实践原则，仅供参考。
![20210705-01.png](/img/20210705-01.png)

### 1、自顶向下
企业要推行SDL，单靠传统的信息安全部门或几个网络安全人员是不行的，必须由公司领导层支持，自顶向下去推行。之所以必须是自顶向下推行，一个重要的原因就是SDL的实施并不是只有信息安全部门投入就可以了，SDL会与产品、研发、测试、运维等相关部门深度结合，需要相关部门的积极支持和全体参与。另外，安全对于大部分企业而言，能直接看到的是成本投入增加，而产出收益却是隐性的，并不会因为做了SDL就能看到产品的直接的收益。
因此，不管是对于研发部门还是其他部门，有SDL的需求，但是很难有主动实施SDL的动力。微软在推行SDL时，是由比尔盖茨亲自发邮件宣布微软内部进入紧急状态，新版停止开发，8500名工程师参加安全编码规范培训，员工从开发代码上转移到识别和修正漏洞，将安全性作为他们工作中最为重要的一项指标。此次中断让微软付出1亿美元代价。这标志着当时世界上最大的软件公司对其产品安全性的看法已经发生了改变。微软首先开始尝试用系统化的思想来避免产生有漏洞的产品，减少”攻击者”滥用它所创建的各种操作系统和工作的机会。为了执行可信赖计算，逐步引入形成了SDL（安全开发生命周期）；而华为则是由CEO担任全球网络安全委员会主任来推行实施。也就是说，如果没有高层领导至上而下的要求，安全部门推行SDL只会是一厢情愿。相信很多安全部门在推行SDL时，都会遇到研发团队不配合而导致无法推行或推行效果不理想的情况。
除了来自高层的支持，企业还要有相应的组织结构支撑，而合理的组织结构是保障SDL实施效果的基础。因为SDL在实施过程中会产生大量新的工作内容和新的工作流程，如果这部分工作内容与工作职责模糊不清，将直接影响SDL的执行效率和实施效果。

### 2、与现有管理体系相契合
不少企业实施SDL时，将SDL作为一个独立的流程来操作，这使得企业需要投入大量额外资源来支撑SDL整个流程的运转，并且实施的质量得不到保障。因此，SDL的实施效果往往达不到预期。
安全本质上是产品的一种质量属性。在质量管理领域，业界已有成熟的方法和流程，比如：ISO9001、CMM等级，这些都用来保障产品的质量。大部分企业都设置有质量部门，并设置有质量管理人员角色。但安全往往因为专业性强，缺乏成熟的管理方法和流程，再加上安全部门的存在，因此产品质量部门通常不关心产品的安全问题。在SDL落地的过程中，将安全工程活动标准化，并纳入产品的质量体系，是保障SDL实施效果的基础。举个例子来说，当产品的某项安全指标没有达到要求时，质量部门有权否决产品的上市发布或上线运营。
部分领导对SDL存在误解，认为SDL就是灵丹妙药，觉得立刻推行SDL就可以提升应用安全能力，部分安全人员对于SDL的理解停留在纸上谈兵阶段，完全照搬微软SDL体系在业务部门推广，恶化部门间关系。当前主流SDL思想均是采用工具+体系咨询的方式进行推进，在项目伊始，咨询人员根据企业现状调研结果来梳理企业当前管理体系和安全状况，基于企业现有软件研发体系、人员管理体系，结合当前企业状况、企业安全目标、安全人员能力、SDL相关经费等因素来为企业量身制定SDL管理流程，做到与现有管理体系相契合，降低部门间隔阂，提高SDL流程可落地性。

### 3、SDL流程可视化、任务可度量
对于企业的研发高层领导来说，最关注的还是SDL实施效果。如何让SDL实施效果可视化，是SDL实施过程中需要注意的重要问题。如果研发高层领导看不到SDL的实施效果，那就意味着可能失去研发高层领导对SDL实施的持续支持和资源投入，从而导致SDL实施失败。SDL实施的效果本身就是隐性的。微软在这个问题上也没法给出立竿见影的效果，但今天Windows操作系统的安全性要比在SDL实施前的WindowsXP好多了，尽管今天的Windows操作系统还是有很多安全漏洞，但安全性的增强并不是简单地从漏洞数量上进行对比，而是漏洞发现的难度、漏洞利用的难度和漏洞被利用的影响都比之前有了明显的改善。
因此，作为SDL实施人员，需要在实施SDL前给研发部门高层领导一个相对合理的预期：世界上没有100%的安全，不能保证SDL实施后就不会再有漏洞了；也不是实施了SDL后安全就可以高枕无忧了。但这也并不意味着就完全看不到效果。如何让SDL实施的效果可视化，比较好的做法是建立一套度量体系，通过度量的方法让SDL实施的效果可视化出来。度量体系本身也是一套复杂的工程，比如说业界的OWASP SAMM和BSIMM就是复杂的评估度量体系。实施人员可以选取一些比较直观且容易实施的工程活动，体现工程能力的成熟度提升，这个和软件成熟度CMM类似。另外，也要有结果性的数据，比如：可以对测试发现的安全问题进行分级，建立一个SDL实施前的基线，再看SDL实施后每一年的问题发展趋势，针对共性高频问题针对性的提出解决方案。

### 4、安全目标及项目分类分级
完整的SDL流程包含众多的安全活动，不同单位根据自身情况实施其中部分安全活动，而同样的活动在不同企业的投入弹性空间也非常大，以威胁建模为例，有的产品可能只花半天时间，而有的产品可能需要花一个月甚至更长时间。在SDL实施的过程中遇到过很多类似问题：这个活动需不需要做？这个活动需要做到什么程度？这个活动需求投入多少人？对于这些问题，并没有统一的答案。因为不同的产品所处的环境不一样，面临的风险也不一样。但我们可以给出基本的判断原则。这些原则的基本出发点就是产品的安全目标是什么？安全目标说起来容易，但要说清楚，就不是一件容易的事了。很多专业的安全人员往往更多的考虑安全技术，而忽略了安全目标。技术应该是用来支撑目标的达成，所以当目标不清楚的情况下，很难判断一项技术的使用是否合理？这些技术是否足够？这就导致了很多企业当前的一个现象：安全的投入好像是一个无底洞，不知道什么时候才能做完。这显然不是企业领导者所要的结果。因此，在实施SDL的过程中，定义一个清晰的安全目标，量身定制契合的安全开发流程，根据项目、数据、用户的重要程度及系统的开放性等，自动对项目进行分类分级，依据分类分级结果匹配相应的SDL流程，才能使SDL的实施过程更加科学合理。

### 5、安全能力组件化
代码漏洞对于软件来说几乎是不可避免的，据数据统计，代码量与漏洞成正比。即便最早提出和实施方法论的微软，也不能保证代码百分之百没有漏洞。
漏洞问题对产品来说是最直观的（可直接利用）,也是最头痛的（消灭不了）;代码漏洞也是ADSL需要重点解决的问题。目前多数也认识到这一问题，并选择使用代码扫描工具，例如SAST和DAST等，但这类工具存在致命的缺陷：误报和漏报。误报过多造成大量研发资源的浪费，而漏报过多又会使得工具的应用效果大打折扣。代码扫描工具的漏报和误报是必然存在的，SDL中也有如何降低漏洞和误报的实践，但这更多需要依赖于新型的安全检测工具去解决。
从SDL的整体视角上看，扫描工具只能发现部分已存在的代码漏洞，并不能减少代码漏洞的产生，属于“后端被动式”的解决思路。SDL更关注如何减少代码漏洞的产生，也就是如何从“前端”主动解决问题。一个比较好的实践就是将产品中的安全特性组件化，比如：密码算法模块、认证授权模块，这些模块都是重要的缓解措施实现，一旦出问题就导致缓解措施被绕过的漏洞。因此，将这些模块组件化，让不同的产品在这些领域都使用公共组件，而不用自己开发，自然也就不会引入漏洞；而这些公共的组件则由安全专业团队重点保障，安全人员可以通过SCA工具监测项目中组件使用的情况。在微软，为了避免参数校验问题导致和缓冲区溢出问题，由专业的安全团队重写了经常导致漏洞的函数(如：​memcpy、strcpy等)，并由一系列自身带有安全校验的函数来代替，这一措施使得产品在很大程度上解决了缓冲区溢出的问题（虽不能全部解决，但效果显而易见，且投入成本不高）。

### 6、软件供应链安全管理
不论是传统的软件企业还是新型的互联网企业，在软件开发的过程中都免不了要使用第三方组件和服务。第三方组件既包含开源软件，也包含商业软件。而且随着软件越复杂，第三方软件的使用数量也越来越多。从安全的角度看，第三方软件也是一个重要的风险源（比如，最近的SolarWinds构建平台恶意代码注入事件）。第三方软件不仅是产品集成的组件，开发环境中所用到的工具也要作为第三方软件来对待（XcodeGhost事件大家应该都还记得）。
第三方软件与自主研发的软件不一样。SDL的方法和流程没法覆盖开源社区和第三方厂商。那么如何管理第三方软件的风险，也是SDL实施过程中面临的一个主要的问题。具体来说，有以下实践供大家参考：

*   (1)企业要有清单列表或者平台记录哪些产品使用了哪些第三方软件。一旦某个第三方软件出现漏洞，可以通过清单列表迅速排查。
* （2)企业要有清单列表记录禁用的第三方软件。对于那些安全问题比较多、风险较大的第三软件，应加入到这个禁用清单列表中禁止使用。
* （3)对于使用较多且开源的第三方组件，建议进行代码扫描，对于发现的漏洞，提交开源社区，并促使开源社区修复。
* （4)对于第三方软件的使用要有安全性指导（主要是规避一些因配置不当引入的安全问题）。
* （5)慎用对安全问题处理态度消极的厂商所开发的第三方软件。
* （6)由于无法确保第三方组件供应商的稳定性并且供应商拒绝提供源代码，可以采取源代码第三方托管机制

### 7、SDL服务化产品化
无论在普通开发、敏捷开发还是DevSecOps模式下，SDL落地的关键都离不开流程体系和高度自动化工具链的融合。根据OWASP SDL项目团队的实践积累，若有一个一体化的平台能准确、完整地记录、管理和追踪软件产品在SDL实施过程中的实际情况，实现软件产品开发信息在SDL流程中跨活动、跨角色流动，才能真正确保软件产品的安全需求和安全设计在开发、测试和部署运维过程中落地。而无论是需求阶段的需求库、开发与测试的安全测试工具、安全开发流程管控平台、还是其他安全工具，都将成为SDL工具链中的一环。
对于部分企业，由于其安全人员数量和能力有限，对于部分安全活动需要专业安全人员参与，可以将相关任务以外包的形式交由第三方安全厂商来实施，安全人员负责任务进度和质量把控。

### 8、SDL工具链(DevSecOps)
近年来，DevOps的开发模式已被广泛应用。DevOps的核心思想是将开发和运维一体化，开发能快速推出产品进行AB测试，通过数个版本的迭代，使产品变得成熟稳定，同时也使产品功能变得丰富。在DevOps开发模式下，传统的SDL流程在DevOps模式下显得过于厚重，那么就需要有适用于DevOps流程的SDL,这就是DevSecOps的由来。由于运维流程也一体化了，因此在传统SDL的安全成本模型也就发生了变化。举个例子来说，在传统SDL的测试过程中，我们要尽可能的发现所有的安全漏洞，因为产品一旦发布，漏洞的修复成本会很高；但在互联网企业自己开发、自己测试、自己运维的DevOps模式下，产品发布后，漏洞修复的成本并不一定增加很多。因为运维一体化后，漏洞一旦发现，响应的时间可控制在一个很短的时间内。但这并不是说DevOps之后开发过程中的安全活动就不需要做了，只是做的方式会有差异。这个差异主要来自于安全功能的服务化、工具自动化。
安全功能服务化本身符合SOA架构和微服务架构的演进方向。安全功能服务化后，就能将产品的一些安全风险转移到安全服务上。以IAM服务为例，采用成熟的IAM服务能在很大程度上降低产品在认证和授权方面的问题。AWS提供的移动应用账号服务可以让移动应用直接集成，而不用担心账号的安全问题；或是采用OAuth认证方式，采用安全性很强的Google、腾讯，字节等知名厂商的安全认证对接。这样自然就减少了产品研发过程中的安全投入，使SDL可以变得快起来。另一方面，采用工具实现自动化，也在很大程度上能减少SDL过程的投入。
另外一点就是当前SDL流程壁垒效应比较严重，在软件安全开发生命周期中，各个阶段的数据无法做到完全互通，工具检测的标准不一，各个厂商的产品无法进行数据拉通，只能做到单一规则检测，无法完全通过检测结果来自动化评估相关参与人员的工作效率和工作质量。举个例子：AST工具检测的漏洞无法自动化反推需求、设计或编码阶段相关任务的质量。这部分如果交由专业人员来分析，势必会引入新的工作量。

### 9、持续优化
SDL 的核心理念是将安全考虑集成在软件开发的每一个阶段：需求、设计、编码、测试和运维。从需求、设计到发布产品的每一个阶段每都增加了相应的安全活动，以减少软件中漏洞的数量并将安全缺陷降低到最小程度。SDL是一个持续优化的过程，在SDL初期，为企业定制的SDL流程或多或少会存在不合理的可能性，或者在技术的升级变革中需要引入其他安全活动，所以在SDL实施过程中安全人员需要保留持续优化的理念，对于不合理的流程或者指标，参考各方建议及时进行优化并公示。

### 10、建立SDL人员培养体系
在SDL实施的过程中，安全不仅仅是安全专家的职责，更是实施企业所有人的责任。仅靠几个安全专家很难保证企业所有产品的安全质量，而信息安全部门或网络安全部门面对软件开发往往也力不从心。SDL虽然整体设计软件产品的安全开发生命周期，偏重于方法和流程，但人的因素同样至关重要。对于同样的方法、同样的流程和同样的工具，如果实施人员的安全开发思想意识和技术能力不同，其产生的实施效果差异也会非常大。比如：某公司的安全部门要求所有口令都在hash后再存储，而开发人员就将口令设计成hash之后的结果，让人看了哭笑不得。
如何让所有研发人员都了解并关注软件安全开发？建立一套合适的培训体系是较好的业界实践。这里的培训强调的是体系化的软件安全开发培训，而不是安全部门内部组织的信息安全知识培训或攻防渗透技术培训，因为对于不同的部门、不同的岗位、不同的人员，其安全的认知意识和技术能力也是不一样的。简单来说，建议将安全培训分成不同的等级，且不同等级面向不同类型的人员群体。比如：软件安全开发意识培训可以面向所有人、软件安全编码培训可只面向开发和测试人员，而网络攻击技术培训可只面向安全专业人员。另外，需要让所有研发人员宏观的理解SDL方法与流程，有助于让每个研发人员了解其与SDL流程中上、下游角色的互动关系，也有助于让每个研发人员理解每一个SDL的工作环节对整体产品安全的重要性。

## 参考
- [https://news.microsoft.com/2012/01/11/memo-from-bill-gates/](https://news.microsoft.com/2012/01/11/memo-from-bill-gates/)【Memo from Bill Gates】
- [https://cloud.tencent.com/developer/article/1459352](https://cloud.tencent.com/developer/article/1459352)【微软的安全开发生命周期(SDL)】
- [https://www.secrss.com/articles/12575](https://www.secrss.com/articles/12575)【从微软、Facebook、华为的网络安全备忘录说开去】
- [http://blog.nsfocus.net/nsfocus-adsl/](http://blog.nsfocus.net/nsfocus-adsl/)【绿盟科技SDL实践】
- [https://www.freebuf.com/articles/es/232252.html](https://www.freebuf.com/articles/es/232252.html)【漫谈SDL：开篇明义】
