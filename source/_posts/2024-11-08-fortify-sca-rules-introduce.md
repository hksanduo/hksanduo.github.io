---
title: "fortify sca rules 标签介绍"
date:   2024-11-08  10:05:00 +0800
layout: post
tag:
- CodeAudit
categories:
- Security
---

# fortify sca rules 标签介绍

------
# RulePack标签分析

```jsx
<?xml version="1.0" encoding="UTF-8"?>
<RulePack xmlns:mask="xmlns://www.fortifysoftware.com/schema/mask" xmlns="xmlns://www.fortifysoftware.com/schema/rules">
    <RulePackID>D044EBBB-7081-4451-BDD6-5A163AD639C3</RulePackID>
    <SKU>RUL13040</SKU>
    <Name>
        <![CDATA[Fortify 安全编码规则、核心、Java]]>
    </Name>
    <Activated>true</Activated>
    <Version>2024.2.1.0003</Version>
    <Language>java</Language>
    <Description>
        <![CDATA[为 Java 提供安全相关核心语言 API 范围。此规则包的部分内容来自 Cigital Java Rulepack (c) 2008 Cigital。 Copyright 2004 - 2024 Open Text.]]>
    </Description>
    <Locale>zh_CN</Locale>
    <Rules version="3.2" minimumSCA="17.10">
    ......
    </Rules>
    <Masks revision="0">
    ......
    </Masks>
    <Localization>
    ......
    </Localization>
</RulePack>
```

xml标签属性说明

- RulePackID：规则包的唯一标识符
- SKU：全局唯一标识符
- Name：规则包的名称
- Activated：可以使用的(猜测，没找到定义内容)
- Version：规则版本
- Language：规则包适配编程语言
- Description：规则包描述信息
- Locale：规则包语言
- Rules：规则主体，其中version是版本代号，minimumSCA 标识规则最低支持的fortify sca版本
- Masks ：掩码
- Localization：本地化

# Rules标签分析

```jsx
<Rules version="3.2" minimumSCA="17.10">
        <Notes>
        ......
        </Notes>
        <RuleDefinitions>
        ......
        </RuleDefinitions>
        <LabelDefinitions>
        ......
        </LabelDefinitions>
        <Descriptions>
        ......
        </Descriptions>
        <ControlflowStateStrings>
        ......
        </ControlflowStateStrings>
        <TaintFlagDeclarations>
        ......
        </TaintFlagDeclarations>
        <TaintFlagDescriptions>
        ......
        </TaintFlagDescriptions>
        <Coverage>
        ......
        </Coverage>
        <ScriptDefinitions/>
</Rules>
```

xml标签属性说明

- Notes：规则备注信息，许可声明信息
- RuleDefinitions：规则集合
- LabelDefinitions：标签定义
- Descriptions：描述信息
- ControlflowStateStrings：控制流状态字符串
- TaintFlagDeclarations：污染标志声明
- TaintFlagDescriptions：污染标志说明
- Coverage：覆盖度
- ScriptDefinitions：脚本定义

## DataflowSourceRule 标签分析

```jsx
<DataflowSourceRule formatVersion="3.2" language="java">
  <MetaInfo>
    <Group name="package">Java Core Accessibility</Group>
    <Group name="inputsource">Web</Group>
  </MetaInfo>
  <RuleID>58B43BC7-05D2-4289-B0CE-BECAFC28A45E</RuleID>
  <FunctionIdentifier>
    <NamespaceName>
      <Value>javax.accessibility</Value>
    </NamespaceName>
    <ClassName>
      <Value>AccessibleText</Value>
    </ClassName>
    <FunctionName>
      <Pattern>(get.+Index)|getSelectedText</Pattern>
    </FunctionName>
    <Parameters>
      <ParamType>int</ParamType>
      <ParamType>int</ParamType>
    </Parameters>
    <ApplyTo implements="true" overrides="true" extends="true" />
  </FunctionIdentifier>
  <OutArguments>return</OutArguments>
</DataflowSourceRule>
```

DataflowSourceRule识别受污染的数据进入程序的位置。

- MetaInfo：元信息，规则分类信息
    - Group：组标签
        - package：包信息，`<Group name="package">Java Core Accessibility</Group>` 表示这条规则属于 "Java Core Accessibility" 包。
        - inputsource：输入源，`<Group name="inputsource">Web</Group>` 表示这条规则关注的是来自 Web 的输入源。
- RuleID：唯一的标识符，用于区分不同的规则。
- FunctionIdentifier：函数标识
    - NamespaceName：**命名空间，**`<NamespaceName><Value>javax.accessibility</Value></NamespaceName>` 指定了方法所在的命名空间。
    - ClassName：**类名，**`<ClassName><Value>AccessibleText</Value></ClassName>` 指定了方法所在的类。
    - FunctionName：函数名，`<FunctionName><Pattern>(get.+Index)|getSelectedText</Pattern></FunctionName>` 使用正则表达式匹配方法名，包括 `getCharIndex`, `getWordIndex`, `getSentenceIndex`, 和 `getSelectedText`。
    - Parameters：参数，`<Parameters><ParamType>int</ParamType><ParamType>int</ParamType></Parameters>` 指定了方法的参数类型，这里有两个 `int` 类型的参数。
- OutArguments：输出参数，用于指定方法调用后哪些参数或返回值应被视为污染（tainted）数据，`<OutArguments>return</OutArguments>` 指定了方法的返回值应被视为污染数据。

## LabelDefinitions标签分析

```jsx
<CharacterizationRule formatVersion="3.7" language="java">
    <MetaInfo>
        <Group name="package">Java Core JMX</Group>
    </MetaInfo>
    <RuleID>04DC0B4B-6AC5-4B93-9B46-486D7B4778A8</RuleID>
    <StructuralMatch>
        <![CDATA[
        FunctionCall fc: function is [Function:
                name == "registerMBean"
                and enclosingClass.supers contains [Class: name == "javax.management.MBeanServer"]
            ]
            and arguments[0] is [Expression:
                reachingTypes contains [Type: definition is [Class jmxbean:]]
            ]
    ]]>
    </StructuralMatch>
    <Definition>
        <![CDATA[
        foreach jmxbean {
            Label(jmxbean, "JMXBean")
        }
    ]]>
    </Definition>
</CharacterizationRule>
```

字符化规则（`<CharacterizationRule>`），用于识别和标记 Java 代码中特定的结构模式。

- CharacterizationRule：
    - formatVersion：**格式版本，**`formatVersion="3.7"` 指明了规则使用的格式版本。
    - language：**语言，**`language="java"` 指明了这条规则适用于 Java 语言。
- MetaInfo：元数据，`<MetaInfo>` 元素提供了关于规则的分类信息。
    - Group：`<Group name="package">Java Core JMX</Group>` 表示这条规则属于 "Java Core JMX" 包。
- RuleID：规则唯一的标识符
- StructuralMatch：结构匹配，`<StructuralMatch>` 元素定义了要匹配的代码结构。
    - `FunctionCall fc` 表示要匹配的函数调用。
    - `function is [Function: name == "registerMBean" and enclosingClass.supers contains [Class: name == "javax.management.MBeanServer"]]` 表示要匹配的函数名为 `registerMBean`，并且该函数所在类的超类包含 `javax.management.MBeanServer`。
    - `arguments[0] is [Expression: reachingTypes contains [Type: definition is [Class jmxbean:]]]` 表示第一个参数的类型是某个类 `jmxbean`。
- Definition：**定义，**`<Definition>` 元素定义了在匹配到特定结构时要执行的操作。
    - `foreach jmxbean { Label(jmxbean, "JMXBean") }` 表示对每一个匹配到的 `jmxbean` 类型，为其添加一个标签 `"JMXBean"`。

## Descirptions标签分析

```jsx
<Descriptions>
    <Description
        id="desc.controlflow.java.intent_manipulation_implicit_internal_intent" formatVersion="3.4">
        <Abstract>
                    <![CDATA[<Paragraph>在 <Replace key="PrimaryLocation.file"/> 的第 <Replace key="PrimaryLocation.line"/> 行检测到隐式内部 <code>Intent</code>。隐式的内部意图可能会使系统遭受对内部组件的中间人攻击。<AltParagraph>检测到隐式内部 <code>Intent</code>。隐式的内部意图可能会使系统遭受对内部组件的中间人攻击。</AltParagraph></Paragraph>]]>
        </Abstract>
        <Explanation>
                    <![CDATA[内部 <code>Intent</code> 使用内部组件定义的自定义操作。隐式意图可以促进从任何给定外部组件调用意图，而无需了解特定组件。将两者结合起来使应用程序能够从所需的应用程序上下文外部访问为特定内部使用指定的意图。

                    通过外部应用程序处理内部 <code>Intent</code> 的能力可以实现各种严重程度不等的中间人攻击，从信息泄露、拒绝服务到远程代码执行，具体取决于 <code>Intent</code> 指定的内部操作的能力。

                    <b>示例 1：</b>以下代码使用隐式内部 <code>Intent</code>。

                    <pre>
                    ...
                    val imp_internal_intent_action = Intent("INTERNAL_ACTION_HERE")
                    startActivity(imp_internal_intent_action)
                    ...
                    </pre>]]>
        </Explanation>
        <Recommendations>
                    <![CDATA[仅调用具有明确意图的内部应用程序。显式 <code>Intent</code> 是显式设置其组件、类、类名和包的意图。

                    <b>示例 2：</b>以下代码使用显式内部 <code>Intent</code>。

                    <pre>
                    ...
                    val exp_internal_intent = Intent("INTERNAL_ACTION_HERE", Uri.EMPTY, this, DownloadService::class.java)
                    startActivity(exp_internal_intent)
                    ...
                    </pre>

                    <b>示例 3：</b>以下代码使用隐式内部 <code>Intent</code>，该意图已通过 setter 更新为显式。

                    <pre>
                    ...
                    val imp_internal_intent_remediate_action = Intent("INTERNAL_ACTION_HERE")

                    imp_internal_intent_remediate_action.`package` = "package"
                    imp_internal_intent_remediate_action.setClass(this, DownloadService::class.java)
                    imp_internal_intent_remediate_action.component = componentName

                    startActivity(imp_internal_intent_remediate_action)
                    ...
                    </pre>]]>
                </Recommendations>
        <References>
            <Reference>
                <Title>
                            <![CDATA[Remediation of Implicit Internal Intent Vulnerability]]>
                        </Title>
                <Source>
                            <![CDATA[https://support.google.com/faqs/answer/10399926]]>
                        </Source>
            </Reference>
            <Reference>
                <Title>
                            <![CDATA[CWE ID 99]]>
                        </Title>
                <Author>
                            <![CDATA[Standards Mapping - Common Weakness Enumeration]]>
                        </Author>
            </Reference>
            <Reference>
                <Title>
                            <![CDATA[CCI-001094]]>
                        </Title>
                <Author>
                            <![CDATA[Standards Mapping - DISA Control Correlation Identifier Version 2]]>
                        </Author>
            </Reference>
            <Reference>
                <Title>
                            <![CDATA[Indirect Access to Sensitive Data]]>
                        </Title>
                <Author>
                            <![CDATA[Standards Mapping - General Data Protection Regulation]]>
                        </Author>
            </Reference>
            <Reference>
                <Title>
                            <![CDATA[SC-5 Denial of Service Protection (P1)]]>
                        </Title>
                <Author>
                            <![CDATA[Standards Mapping - NIST Special Publication 800-53 Revision 4]]>
                        </Author>
            </Reference>
            <Reference>
                <Title>
                            <![CDATA[SC-5 Denial of Service Protection]]>
                        </Title>
                <Author>
                            <![CDATA[Standards Mapping - NIST Special Publication 800-53 Revision 5]]>
                        </Author>
            </Reference>
            <Reference>
                <Title>
                            <![CDATA[A9 Application Denial of Service]]>
                        </Title>
                <Author>
                            <![CDATA[Standards Mapping - OWASP Top 10 2004]]>
                        </Author>
            </Reference>
            <Reference>
                <Title>
                            <![CDATA[Requirement 6.5.1]]>
                        </Title>
                <Author>
                            <![CDATA[Standards Mapping - Payment Card Industry Data Security Standard Version 1.1]]>
                        </Author>
            </Reference>
        </References>
    </Description>
</Descriptions>
```

- Description：规则描述，这里将描述信息单独拎出来，使用的化直接引用就行，类似这种：<Description ref="desc.controlflow.java.code_correctness_call_to_sleep_in_lock">
    - **版本格式**：`formatVersion="3.4"` 指明了规则使用的格式版本。
    - **ID**：`id="desc.controlflow.java.intent_manipulation_implicit_internal_intent"` 是一个唯一的标识符，用于区分不同的描述性规则。
- Abstract：`<Abstract>` 元素提供了规则的简要说明。
    - `Paragraph`：`<Paragraph>` 描述了在特定文件和行号检测到隐式内部 `Intent`，并指出隐式内部意图可能会使系统遭受中间人攻击。
    - `AltParagraph`：`<AltParagraph>` 提供了一个简短的备用描述。
- Explanation：详细解释
- Recommendations：推荐措施
- References：参考

## ControlflowStateStrings标签分析

```jsx
<ControlflowStateStrings>
    <StateStrings id="statedesc.java.null_assigned">
        <Enter>%{p} is assigned null</Enter>
    </StateStrings>
</ControlflowStateStrings>
```

控制流状态字符串概述

- StateStrings：**状态字符串，**<StateStrings>元素包含了描述状态变化的字符串。
    - **Enter：进入状态，**`<Enter>%{p} is assigned null</Enter>` 描述了当变量被赋值为 `null` 时的状态变化。
    - id：`id="statedesc.java.null_assigned"` 是一个唯一的标识符，用于区分不同的控制流状态字符串。

说明：

- id：`statedesc.java.null_assigned` 这个 id 表示这是一个与 Java 语言相关的状态描述，具体描述了变量被赋值为 `null` 的情况。
- **进入状态**：`%{p} is assigned null` 这个字符串使用占位符 `%{p}` 来表示变量名。当工具检测到某个变量被赋值为 `null` 时，会用实际的变量名替换 `%{p}`，从而生成具体的描述信息。

## TaintFlagDeclarations标签分析

```jsx
<TaintFlagDescriptions>
    <FlagDescription name="ARGS">Command Line Arguments</FlagDescription>
    <FlagDescription name="PRIVATE">Private Information</FlagDescription>
</TaintFlagDescriptions>
```

`<TaintFlagDescriptions>`污点标志描述

- FlagDescription：**污点标志描述，**`<FlagDescription>` 元素定义了每个污点标志的名称和描述。
    - `name` 属性指定了污点标志的名称。
    - value描述：元素的文本内容提供了污点标志的描述。

这段 XML 代码定义了两个污点标志描述：

- `ARGS`：表示数据来源于命令行参数。
- `PRIVATE`：表示数据包含私有信息。

## Coverage 标签分析

```jsx
<Coverage>
    <FunctionIdentifier formatVersion="3.8">
        <MatchExpression>java.io/
            File.{toPath,getPath,getAbsolutePath,getCanonicalPath,getCanonicalFile}()</MatchExpression>
    </FunctionIdentifier>
    <FunctionIdentifier formatVersion="6.10">
        <MatchExpression>java.io / ObjectOutputStream.init^()</MatchExpression>
    </FunctionIdentifier>
</Coverage>
```

<Coverage>覆盖率配置

- FunctionIdentifier：**函数标识符，定义了需要覆盖的函数或方法。**
    - formatVersion：函数标识符的格式版本。
    - MatchExpression：匹配函数或方法的表达式。

表达式描述：

`java.io/File.{toPath,getPath,getAbsolutePath,getCanonicalPath,getCanonicalFile}()`

匹配表达式指定了 `java.io.File` 类中的多个方法，包括 `toPath`、`getPath`、`getAbsolutePath`、`getCanonicalPath` 和 `getCanonicalFile`。这些方法都与文件路径的获取有关。

# Masks标签分析

```jsx
<Masks revision="0">
    <mask:Mask builtin="true" name="Targeted">
        <mask:Exclusion>
            <mask:Attribute name="inputsource" value="Serialized Data" />
        </mask:Exclusion>
        <mask:Exclusion>
            <mask:Attribute name="inputsource" value="Windows Registry" />
        </mask:Exclusion>
    </mask:Mask>
    <mask:Mask builtin="true" default="true" name="Broad">
        <mask:Description>
            <![CDATA[此设置向源代码分析器提供了一套最全面的规则。通过它的使用可以发现一系列需要审核的安全问题。]]>
        </mask:Description>
    </mask:Mask>
    <mask:Mask builtin="true" name="Medium">
    </mask:Mask>
</Masks>
```

Mask：掩码，用于静态代码分析工具（如 Fortify SCA）来配置不同的分析规则集。

- `<mask:Mask>`： 元素定义了不同的分析规则集。
- `builtin="true"`： 表示这是一个预定义的掩码。
- `name` ：属性指定了掩码的名称。
- `default="true"` ：表示这是默认的掩码。

# Localization 标签分析

```jsx
<Localization>
  <Mapping>
    <KeyString>Weak Encryption</KeyString>
    <LocalString>Weak Encryption</LocalString>
  </Mapping>
  <Mapping>
    <KeyString>User-Controlled Key Size</KeyString>
    <LocalString>User-Controlled Key Size</LocalString>
  </Mapping>
</Localization>;
```

本地化配置，管理和翻译不同的字符串

- `<Localization>` 是根元素，包含多个映射（`<Mapping>`）。
- `<Mapping>` 元素定义了键字符串（`<KeyString>`）和本地化字符串（`<LocalString>`）之间的对应关系。

# 参考

-  SCACustRulesGuide23.2.0.pdf
-  SCA_Cust_Rules_Guide_23.2.0.zip
-  2024.2.1.0003-zh_CN.zip

参考文件，已托管到知识星球，自取。