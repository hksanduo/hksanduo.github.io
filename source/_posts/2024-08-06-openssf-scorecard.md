---
title: "软件供应链健康度评估之scorecard"
date:   2024-08-06  14:36:00 +0800
layout: post
tag:
- Software Supply Chain
categories:
- Security
---

# 软件供应链健康度评估之scorecard 

------
# Scorecard介绍
Scorecard是一个自动化工具，评估开源项目的健康度。Scorecard通过评估与开源项目安全性相关的一系列指标项，每项10分。可以使用这些评分来了解需要改进的具体领域，以增强项目的安全性。同时，还可以评估依赖项带来的安全风险，如接受这些风险、评估替代方案或与维护者合作进行改进。

注意：由于scorecard项目不断迭代更新，检测规则也在变化，本文有效时间为：2024年08月06日前后

## 支持Scorecard项目
Scorecard已在数千个项目上运行，以监控和跟踪安全指标。主要使用Scorecard的项目包括：

* Tensorflow
* Angular
* Flutter
* sos.dev
* deps.dev

![deps fastjson results ](/img/20240806-01.png)

## 查看项目的评分
要查看Scorecard定期扫描的项目评分，请访问webviewer，并替换占位符文本为平台、用户/组织和存储库名称：
`https://scorecard.dev/viewer/?uri=<github_or_gitlab>.com/<user_name_or_org>/<repository_name>`

例如：

https://scorecard.dev/viewer/?uri=github.com/alibaba/fastjson

![fastjson score ](/img/20240806-02.png)

## 公开数据
每周对被认为是最关键的100万个开源项目进行Scorecard扫描，并将结果发布在BigQuery公开数据集中。最新的结果可在BigQuery视图openssf:scorecardcron.scorecard-v2_latest中查看。


可以使用BigQuery Explorer查询数据，导航至Add Data > Star a project by name > 'openssf'。例如，可能会对项目评分随时间的变化感兴趣：

![bigquery data query](/img/20240806-03.png)

```
SELECT date, score FROM `openssf.scorecardcron.scorecard-v2` WHERE repo.name="github.com/alibaba/fastjson" ORDER BY date ASC
```
![bigquery data query demo](/img/20240806-04.png)

还可以使用bq工具将最新结果提取到Google Cloud Storage中：

```
# 获取最新的PARTITION_ID
bq query --nouse_legacy_sql 'SELECT partition_id FROM openssf.scorecardcron.INFORMATION_SCHEMA.PARTITIONS WHERE table_name="scorecard-v2" AND partition_id!="__NULL__" ORDER BY partition_id DESC LIMIT 1'

# 提取到GCS
bq extract --destination_format=NEWLINE_DELIMITED_JSON 'openssf:scorecardcron.scorecard-v2$<partition_id>' gs://bucket-name/filename-*.json
```


# Scorecard 使用
## Scorecard GitHub Action
使用Scorecard GitHub Action是最简单的方法，在拥有的GitHub项目上运行Scorecard。该Action在任何存储库更改时运行，并在维护者可以在存储库的安全标签中查看警报。有关更多信息，请参阅Scorecard GitHub Action安装说明。

## Scorecard命令行
要在不拥有的项目上运行Scorecard扫描，请使用命令行界面安装选项。

### 安装
#### Docker
scorecard可以作为Docker容器使用：

```
docker pull gcr.io/openssf/scorecard:stable
```
要使用特定的scorecard版本（例如v3.2.1），请运行：
```
docker pull gcr.io/openssf/scorecard:v3.2.1
```
#### 本地安装
自行编译，或者直接从release下载对应的二进制文件即可
```
make build
```

使用包管理器
| 包管理器 | 支持的分发版 |	命令 |
| --- | --- | --- |
| Nix	| NixOS	| nix-shell -p nixpkgs.scorecard |
| AUR 	| Arch Linux |	yaourt install scorecard |
| Homebrew	| macOS或Linux	| brew install scorecard |

### 获取github 授权 token
GitHub对未经认证的请求实施api速率限制。为了避免这些限制，必须在运行Scorecard之前对的请求进行认证。有两种方法可以认证的请求：创建GitHub个人访问令牌或创建GitHub App安装。

创建经典的GitHub个人访问令牌。创建个人访问令牌时，建议选择public_repo范围。

```
export GITHUB_AUTH_TOKEN=<your access token>
```

### 运行
若要在特定存储库上运行scorecard：
```
scorecard --repo=<repo_url>
```

例如：
```
scorecard --repo=https://github.com/alibaba/fastjson
```
结果如下：
```
Starting [Token-Permissions]
Starting [Signed-Releases]
Starting [Packaging]
Starting [Dangerous-Workflow]
Starting [Contributors]
Starting [Security-Policy]
Starting [CI-Tests]
Starting [SAST]
Starting [License]
Starting [Binary-Artifacts]
Starting [CII-Best-Practices]
Starting [Pinned-Dependencies]
Starting [Code-Review]
Starting [Fuzzing]
Starting [Branch-Protection]
Starting [Vulnerabilities]
Starting [Dependency-Update-Tool]
Starting [Maintained]
Finished [Binary-Artifacts]
Finished [CII-Best-Practices]
Finished [Pinned-Dependencies]
Finished [Vulnerabilities]
Finished [Dependency-Update-Tool]
Finished [Maintained]
Finished [Code-Review]
Finished [Fuzzing]
Finished [Branch-Protection]
Finished [Token-Permissions]
Finished [Signed-Releases]
Finished [Packaging]
Finished [CI-Tests]
Finished [SAST]
Finished [License]
Finished [Dangerous-Workflow]
Finished [Contributors]
Finished [Security-Policy]

RESULTS
-------
Aggregate score: 3.9 / 10

Check scores:
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
|  SCORE  |          NAME          |             REASON             |                                               DOCUMENTATION/REMEDIATION                                               |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 10 / 10 | Binary-Artifacts       | no binaries found in the repo  | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#binary-artifacts       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | Branch-Protection      | branch protection not enabled  | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#branch-protection      |
|         |                        | on development/release         |                                                                                                                       |
|         |                        | branches                       |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | CI-Tests               | 0 out of 4 merged PRs          | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#ci-tests               |
|         |                        | checked by a CI test -- score  |                                                                                                                       |
|         |                        | normalized to 0                |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | CII-Best-Practices     | no effort to earn an OpenSSF   | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#cii-best-practices     |
|         |                        | best practices badge detected  |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 1 / 10  | Code-Review            | Found 3/28 approved changesets | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#code-review            |
|         |                        | -- score normalized to 1       |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 10 / 10 | Contributors           | project has 18 contributing    | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#contributors           |
|         |                        | companies or organizations     |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 10 / 10 | Dangerous-Workflow     | no dangerous workflow patterns | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#dangerous-workflow     |
|         |                        | detected                       |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 10 / 10 | Dependency-Update-Tool | update tool detected           | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#dependency-update-tool |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | Fuzzing                | project is not fuzzed          | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#fuzzing                |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 10 / 10 | License                | license file detected          | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#license                |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | Maintained             | 0 commit(s) and 0 issue        | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#maintained             |
|         |                        | activity found in the last 90  |                                                                                                                       |
|         |                        | days -- score normalized to 0  |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| ?       | Packaging              | packaging workflow not         | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#packaging              |
|         |                        | detected                       |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | Pinned-Dependencies    | dependency not pinned by hash  | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#pinned-dependencies    |
|         |                        | detected -- score normalized   |                                                                                                                       |
|         |                        | to 0                           |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | SAST                   | SAST tool is not run on all    | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#sast                   |
|         |                        | commits -- score normalized to |                                                                                                                       |
|         |                        | 0                              |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 10 / 10 | Security-Policy        | security policy file detected  | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#security-policy        |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| ?       | Signed-Releases        | no releases found              | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#signed-releases        |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | Token-Permissions      | detected GitHub workflow       | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#token-permissions      |
|         |                        | tokens with excessive          |                                                                                                                       |
|         |                        | permissions                    |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 0 / 10  | Vulnerabilities        | 22 existing vulnerabilities    | https://github.com/ossf/scorecard/blob/a8eae2d833553e93021a22f5641661cb6ad3e24c/docs/checks.md#vulnerabilities        |
|         |                        | detected                       |                                                                                                                       |
|---------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|
```

在默认情况下，Scorecard会对所有检查运行。要运行特定检查，请使用：
```
scorecard --repo=<repo_url> --checks <check1,check2>
```
例如：

```
scorecard --repo=https://github.com/ossf/scorecard --checks Code-Review,Maintained
```

使用SARIF文件导出到GitHub Security
可以将Scorecard检查结果导出到GitHub Security工具（由GitHub Actions自动生成）。为此，使用--format参数生成一个SARIF文件并上传：

```
scorecard --repo=<repo_url> --format sarif > results.sarif
gh upload results.sarif
```

使用原始数据导出
默认情况下，scorecard导出json格式的结果。可以使用以下参数导出结果：
```
scorecard --repo=<repo_url> --format json --show-details > results.json
```

## 检查项介绍
检查项详细来源：https://github.com/ossf/scorecard/blob/main/docs/checks.md

| 名称 | 描述 |  
| --- | --- |
| Binary-Artifacts(二进制组件) | 检查项目中是否包含二进制文件 |     
| Branch-Protection(分支保护) | github 是否启用分支保护 |  
| CI-Tests(CI测试) | 在合并分支请求之前是否配置CI测试 | 
| CII-Best-Practices(CII最佳实践) | 是否遵循CII最佳实践 | 
| Code-Review(代码审计) | 检测项目在合并代码之前是否进行代码审计 | 
| Contributors(贡献者) | 检查项目是否有多个贡献者 | 
| Dangerous-Workflow(不安全的工作流) | 检查github action配置是否不安全 | 
| Dependency-Update-Tool(依赖更新工具) | 检查项目是否使用依赖更新工具 | 
| Fuzzing(模糊测试) | 检查项目是否进行模糊测试 | 
| License(许可协议) | 检查项目是否声明了许可协议 | 
| Maintained(维护) | 检查项目是否积极维护 | 
| Packaging(打包) | 检查项目是否使用组件包的形式进行发布 | 
| Pinned-Dependencies(依赖标识) | 检查项目是否使用包管理器声明依赖信息 | 
| SAST(静态代码扫描) | 检查项目是否进行静态代码扫描 | 
| Security-Policy(安全策略) | 检查项目是否声明安全策略 | 
| Signed-Releases(签名发行) | 检查项目是否对发布的组件进行签名 | 
| Token-Permissions(令牌权限) | 检查workflow token是否遵顼最小权限原则 | 
| Vulnerabilities(漏洞) | 使用osv等开源漏洞库检测代码和组件是否包含漏洞 | 

## 总结
### 企业实践
1、使用 Scorecard 对第三方开源项目进行安全性评估，综合判断引入的第三方组件的健壮性。    
2、对企业开源的项目进行评估，提高被维护项目的安全性。

### 开发者
1、对自己开发的项目进行评估，综合提高项目安全性。   
2、评估引入的第三方组件，提高项目整理健壮性。

## 参考
* https://deps.dev/
* https://osv.dev/
* https://github.com/ossf/scorecard
