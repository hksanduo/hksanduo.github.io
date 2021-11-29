---
title: "nodejs code audit"
date:   2020-06-28  14:39:00 +0800
layout: post
tag:
- Nodejs
categories:
- Security
- Code Audit
---

nodejs 源代码安全审计

-------
## 背景
最近碰到几个nodejs的代码审计项目，看来nodejs在企业内部还是有一定的用户群体，之前翻译过国外的一篇nodejs安全清单的文章，但是文章并未阐述如何审计一个nodejs项目，这次借审计的机会，把个人对于nodejs项目审计的经验积累一下，欢迎来喷。

## 准备
1、ide(atom、sublime、vscode、nodepad++等等)，根据个人爱好选择
2、被审计的源代码
3、nodejs环境
4、静态分析工具(dependency check、fortify sca、npm、nodejsscan、Retire.js等)

## nodejs源代码审计思路
代码审计通常采用人工审查+SCA工具扫描的的方式进行审计，人工审计的方法有以下四种：
* 正向数据流分析
* 逆向数据流分析
* 根据功能点定向分析
* 通读全文

### 正向数据流分析
#### 确定获取数据方法
1、获取GET请求内容

由于GET请求直接被嵌入在路径中，URL是完整的请求路径，包括了?后面的部分，因此你可以手动解析后面的内容作为GET请求的参数。node.js 中url模块中的 *parse* 函数提供了这个功能，我们可以使用 *url.parse* 方法来解析URL中的参数。
实例：
```
var http = require('http');
var url = require('url');
var util = require('util');

http.createServer(function(req, res){
    res.writeHead(200, {'Content-Type': 'text/plain; charset=utf-8'});
    res.end(util.inspect(url.parse(req.url, true)));
}).listen(3000);
```

2、获取POST请求内容

POST 请求的内容全部的都在请求体中，http.ServerRequest 并没有一个属性内容为请求体，原因是等待请求体传输可能是一件耗时的工作。比如上传文件，而很多时候我们可能并不需要理会请求体的内容，恶意的POST请求会大大消耗服务器的资源，所以node.js默认是不会解析请求体的，当你需要的时候，需要手动来做。
实例：
```
var http = require('http');
var querystring = require('querystring');
var util = require('util');

http.createServer(function(req, res){
    // 定义了一个post变量，用于暂存请求体的信息
    var post = '';

    // 通过req的data事件监听函数，每当接受到请求体的数据，就累加到post变量中
    req.on('data', function(chunk){
        post += chunk;
    });

    // 在end事件触发后，通过querystring.parse将post解析为真正的POST请求格式，然后向客户端返回。
    req.on('end', function(){
        post = querystring.parse(post);
        res.end(util.inspect(post));
    });
}).listen(3000);
```
后端系统通过GET或者POST获取提交的参数，进行逻辑处理，我们按照正向数据流，分析数据是否进行安全处理，逻辑流程是否合理，是否可以导致敏感信息泄露。
#### 查询api list
通过查看系统对外提供的api接口，确定以下几点：
* 接口权限校验
  - 审计对外提供服务的接口是否存在权限缺陷，接口如果对互联网开放则需要对接口的使用进行权限验证，如在内部则需要在网络层面对接口服务进行控制。
* 接口请求频率校验
  - 审计对外提供服务接口是否有防滥用机制，例如查询相关信息接口，一方面如果未对接口进行查询次数控制，则会导致大量信息泄露，另一方面，频繁的查询会对服务器性能造成影响如对方频繁恶意进行接口调用，则会导致接口性能下降，影响业务（基于业务的DDOS攻击）。建议进行接口设计时设计接口阈值，对接口访问频率设置阈值，超出设定的访问频率时返回错误码，对超过阈值的请求进行屏蔽及预警，可以一定程度上防止CC攻击。
* 接口数据格式校验
  - 审计对外提供服务接口是否接口数据校验机制，安全的接口应该有数据白名单校验机制，对数据的数据类型，格式，长度、合法性进行校验。白名单校验机制验证数据确保不出现异常数据和注入攻击。
* 接口防重放
  - 审计对外提供服务接口进行交易时，是否防具备防重放措施，防止关键交易被重放，导致业务风险。
* 接口完整性校验（以系统安全级别及客户需求为准）
  - 对外提供服务接口时，是否具备报文签名机制（将时间戳加上报文其他参数再用MD5或SHA-1算法计算哈希值，这个哈希值就是本次请求的签名sign，服务端接收到请求后以同样的算法得到签名，并跟当前的签名进行比对，如果不一样，说明参数被更改过，直接返回错误标识）。签名机制保证了数据不会被篡改。
* 接口保密性校验（判断传输过程是否进行安全防护）
  - 对外提供服务接口时，如在互联网提供接口服务，则应该具备SSL加密保护或者通过VPN通道进行数据传输。如暂时未实现SSL以及VPN，则需要对敏感数据采用秘钥进行加密。
* 接口数据防泄漏
  - 检查对外提供服务接口是否有报错信息隐藏提示，安全的接口，当出现交易报错或者其他异常报错时系统返回报错信息的返回码，避免堆栈信息泄露（包括框架信息、中间件信息）。

### 逆向数据流分析
根据敏感函数，逆向追踪参数的传递过程，是代码审计最常用的方法，通过全局检索关键字，关键函数，如：
  * sql查询关键字(select、insert、update)
  * 文件上传下载关键字（upload、download）
  * 执行系统命令关键字（exec、execSyn、execFile、execFileSync、spawn、spawnSync）
  * 解析js,导致执行任意命令(eval、setInteval、setTimeout、Function)

这种方式的有点是只需要检索相应的关键字，即可快熟的挖掘想要的漏洞，具有可定向挖掘、高效、高质量的特点；缺点也很明显，对于系统框架整体把握不准确，对系统了解薄弱，在挖掘系统逻辑漏洞可能存在覆盖不全等问题。

### 根据功能点定向分析
通过特定功能点，分析系统，可能是文件上传、文件下载、富文本编辑、交易、找回密码（密码重置）等功能进行定向分析。

* 文件上传：是否限制文件上传的大小，类型，如果未限制，可能存在任意文件上传，如果上传的文件可以被执行，可获取远程服务器权限。
* 文件下载：如果系统文件下载功能设计不完善，存在通过拼接路径进行文件下载，可能存在任意文件下载漏洞
* 富文本编辑：富文本编辑框可能是审计人员容易忽略的地方，由于富文本编辑框的特性，无法使用全局过滤器进行处理，可能会存在跨站脚本编制漏洞
* 交易：对交易的数据未做非负处理，可能存在负值反冲，正负值对冲等风险
* 找回密码(密码重置)：找回密码功能可能存在敏感信息泄露，流程跳跃，短信或邮件轰炸，用户名枚举，重置任意账户密码等问题
* .....

### 通读全文
通读全文主要是为了保证审计范围全部被覆盖到，另一点是发现定点分析发现不了的逻辑漏洞。在企业中对自身系统做安全审计，需要了解真个系统的业务逻辑，这样才能挖掘到更多有价值的漏洞。

通读全文也有一定的技巧，首先需要查看系统的代码架构，比如系统的目录结构，确定系统的架构，判断那些是框架的程序文件，那些是业务的程序文件，那些配置文件。查看是否安全组件，比如：xss等；其次确定系统入口文件，判断数据的正向流向，通过分析系统的业务，查看可导致安全风险的功能点、结合业务导致的逻辑漏洞、研发人员遗漏的功能点等。

通读全文的好处仙儿意见，可以更好的了解程序的架构以及业务逻辑，能够挖掘到更多高质量的漏洞，缺点也很明显，耗费的时间过长，如果代码编写不规范，对于审计人员来说也是一种折磨。通常甲方会采用这种方式，乙方由于工时任务等因素，在实施的过程中很少采用这种方式，主要还得看项目经理安排，说到这里，得批评某些行业搅屎棍，低价抢代码项目，本来一周的任务，压缩到2天，几十万行的代码两天看完，还得写报告，几十万行小说也得看几天，这种项目个人看法就是昧着良心干活，纯属耍流氓。喷子勿扰，这只是个人观点，不要和我说全部依赖静态代码扫描工具，如果你真做过代码审计，你就会清楚SCA工具的误报有多高，发现漏洞的数量有多低，复核这些漏洞都是需要花费时间的。

## nodejs常见安全控制措施识别
### sql注入
为了防止SQL注入，可以将SQL中传入参数进行过滤或预编译，而不是直接进行字符串拼接。

1、使用escape()对查询参数进行转义
参数编码方法有如下三个：

* mysql.escape(param)
* connection.escape(param)
* pool.escape(param)

例如：
```
var userId = 'some user provided value';//前端传入的userid参数
var sql = 'SELECT * FROM users WHERE id = ' + connection.escape(userId);
connection.query(sql, function (error, results, fields) {
  if (error) throw error;
  // ...
});
```
escape()方法转义规则如下：
  * Numbers不进行转换；
  * Booleans转换为true/false；
  * Date对象转换为’YYYY-mm-dd HH:ii:ss’字符串；
  * Buffers转换为hex字符串，如X’0fa5’；
  * Strings进行安全转义；
  * Arrays转换为列表，如[‘a’, ‘b’]会转换为’a’, ‘b’；
  * 多维数组转换为组列表，如[[‘a’, ‘b’], [‘c’, ‘d’]]会转换为’a’, ‘b’), (‘c’, ‘d’)；
  * Objects会转换为key=value键值对的形式。
  * 嵌套的对象转换为字符串；
  * undefined/null会转换为NULL；
  * MySQL不支持NaN/Infinity，并且会触发MySQL错误。

2、使用escapeId()对查询标识符进行转义：
如果系统不信任用户传入的SQL标识符（数据库、表、字段名），可以使用escapeId()方法进行转义，最常用于排序等场景。escapeId()有如下三种使用方法：

  * mysql.escapeId(identifier)
  * connection.escapeId(identifier)
  * pool.escapeId(identifier)

例如：
```
var sorter = 'date';
var sql    = 'SELECT * FROM posts ORDER BY ' + connection.escapeId(sorter);
connection.query(sql, function(err, results) {
  // ...
});
```
3、使用占位符进行预编译
可使用 ? 做为查询参数占位符。在使用查询参数占位符时，在其内部自动调用 connection.escape() 方法对传入参数进行预编译。

如：
```
connection.query('SELECT * FROM users WHERE id = ?', [userId], function (error, results, fields) {
  if (error) throw error;
  // ...
});
```
也可以是以下这种查询方式，其中foo值映射为a，bar映射为b,baz映射为c,id映射为userId
```
connection.query('UPDATE users SET foo = ?, bar = ?, baz = ? WHERE id = ?', ['a', 'b', 'c', userId], function (error, results, fields) {
  if (error) throw error;
  // ...
});
```
这个查询方式看起来和mysql的预编译相似，但是只在connection.escape()方法的内部实现类似预编译机制，需要注意的是，与mysql预编译不同的是，所有“？”号均被替换，甚至在注释和参数中的问号也不例外。

4、预查询
使用mysql.format()转义参数，该函数会选择合适的转义方法转义参数 mysql.format()用于准备查询语句。

例如：
```
var userId = 1;
var sql = "SELECT * FROM ?? WHERE ?? = ?";
var inserts = ['users', 'id', userId];
sql = mysql.format(sql, inserts); // SELECT * FROM users WHERE id = 1
```
### XSS
Node环境下，安装：
```
$ npm install xss
```
然后修改：
```
const xss = require('xss')
const inputValue = content  // 未进行 xss 防御
const inputValue = xss(content)  // 已进行 xss 防御
```
通过全局检索，是否安装xss依赖库。

### express过滤器
Express 是一个简洁而灵活的 node.js Web应用框架, 提供了一系列强大特性帮助你创建各种 Web 应用，和丰富的 HTTP 工具，使用 Express 可以快速地搭建一个完整功能的网站。

Express 框架核心特性：
  * 可以设置中间件来响应 HTTP 请求。
  * 定义了路由表用于执行不同的 HTTP 请求动作。
  * 可以通过向模板传递参数来动态渲染 HTML 页面。

简单过滤器
```
const express = require('express')
const app = express();

let filter = (req, res, next) => {
    if(req.params.name == 'admin' && req.params.pwd == 'admin'){
        next()
    } else {
        next('用户名密码不正确')
    }
}

app.get('/:name/:pwd', filter, (req, res) => {
    res.send('ok')
}).listen(88)
```
运行规则

* 访问 http://localhost:4000/admin/admin
* 首先会进入过滤器方法 filter
* next()，不带任选参数，表示会直接进入目标路由，执行路由逻辑
* next('')，带参数，表示不会进入目标路由，并抛出错误。



## 常见nodejs SCA工具
### NPM Audit
#### 介绍
`npm audit`命令NPM工具提供的用于审计当前依赖组件是否存在安全风险，使用`npm install`安装依赖库，也会触发安全扫描，会返回当前依赖库存在的安全风险，提示用户进行升级。我们审计不会修改用户的项目数据，但是对于组件的问题，需要提出来，方便研发人员或者其他运维支持部对系统依赖组件进行升级。
#### 使用
检查依赖组件安全风险
```
$ npm audit
```
![20200624-nodejs-code-review-02.png](/img/20200624-nodejs-code-review-02.png)

修复组件安全风险
```
$ npm audit fix
```
#### 分析扫描结果

    1、如果组件不依赖应用系统，判断当前组件的安全风险及漏洞利用方式，确定漏洞等级
    2、如果组件安全风险必须依赖应用系统，判断是否影响应用系统。

### owasp dependency check
#### 介绍
Dependency-Check是OWASP（Open Web Application Security Project）的一个实用开源程序，用于识别项目依赖项并检查是否存在任何已知的，公开披露的漏洞。目前，已支持Java、.NET、Ruby、Node.js、Python等语言编写的程序，并为C/C++构建系统（autoconf和cmake）提供了有限的支持。
Dependency-Check支持面广（支持多种语言）、可集成性强，作为一款开源工具，在多年来的发展中已经支持和许多主流的软件进行集成，比如：命令行、Ant、Maven、Gradle、Jenkins、Sonar等；具备使用方便，落地简单等优势。
#### 使用
dependency check提供windows和linux两种运行方式，用户可以自行下载编译文成的二进制包，也可以下载源代码自行进行编译，方式不固定。

github地址：[https://github.com/jeremylong/DependencyCheck](https://github.com/jeremylong/DependencyCheck)
dependency 5.3.2下载地址:[https://dl.bintray.com/jeremy-long/owasp/dependency-check-5.3.2-release.zip](https://dl.bintray.com/jeremy-long/owasp/dependency-check-5.3.2-release.zip)
使用方式：
在bin目录下面有bat和shell两个脚本，根据操作系统运行对应的脚本

![20200624-nodejs-code-review-03.png](/img/20200624-nodejs-code-review-03.png)
```
$ .\dependency-check.bat --help
usage: Dependency-Check Core [--advancedHelp] [--enableExperimental]
       [--exclude <pattern>] [-f <format>] [--failOnCVSS <score>] [-h]
       [--junitFailOnCVSS <score>] [-l <file>] [-n] [-o <path>]
       [--prettyPrint] [--project <name>] [-s <path>] [--suppression
       <file>] [-v]

Dependency-Check Core can be used to identify if there are any known CVE
vulnerabilities in libraries utilized by an application. Dependency-Check
Core will automatically update required data from the Internet, such as
the CVE and CPE data files from nvd.nist.gov.

    --advancedHelp              Print the advanced help message.
    --enableExperimental        Enables the experimental analyzers.
    --exclude <pattern>         Specify an exclusion pattern. This option
                                can be specified multiple times and it
                                accepts Ant style exclusions.
 -f,--format <format>           The report format (HTML, XML, CSV, JSON,
                                JUNIT, or ALL). The default is HTML.
                                Multiple format parameters can be
                                specified.
    --failOnCVSS <score>        Specifies if the build should be failed if
                                a CVSS score above a specified level is
                                identified. The default is 11; since the
                                CVSS scores are 0-10, by default the build
                                will never fail.
 -h,--help                      Print this message.
    --junitFailOnCVSS <score>   Specifies the CVSS score that is
                                considered a failure when generating the
                                junit report. The default is 0.
 -l,--log <file>                The file path to write verbose logging
                                information.
 -n,--noupdate                  Disables the automatic updating of the CPE
                                data.
 -o,--out <path>                The folder to write reports to. This
                                defaults to the current directory. It is
                                possible to set this to a specific file
                                name if the format argument is not set to
                                ALL.
    --prettyPrint               When specified the JSON and XML report
                                formats will be pretty printed.
    --project <name>            The name of the project being scanned.
 -s,--scan <path>               The path to scan - this option can be
                                specified multiple times. Ant style paths
                                are supported (e.g. 'path/**/*.jar'); if
                                using Ant style paths it is highly
                                recommended to quote the argument value.
    --suppression <file>        The file path to the suppression XML file.
                                This can be specified more then once to
                                utilize multiple suppression files
 -v,--version                   Print the version information.
```
可根据实际情况，自行构建命令进行扫描,如下:
```
.\bin\dependency-check.bat --project Testing --out . --scan [path to  files to be scanned]
```
注意一点，dependency check扫描nodejs，需要使用`node install`命令先安装相关依赖包，然后才可以使用dependency check进行扫描
#### 分析扫描结果
dependency check会匹配当前依赖对应的CVE及CWE漏洞，分析这些风险问题，会花费很长时间，建议对高风险以上问题进行分析，确定漏洞是否存在。
![20200624-nodejs-code-review-04.png](/img/20200624-nodejs-code-review-04.png)

### fortify sca
#### 介绍
Fortify SCA 是一个静态的、白盒的源代码安全测试工具。它通过内置的五大主要分析引擎对源代码进行静态的分析和检测，分析的过程中与其特有的软件安全漏洞规则集进行全面地匹配、查找，从而将源代码中存在的安全漏洞扫描出来，并整理生成完整的报告。扫描的结果中不但包括详细的安全漏洞的信息，还会有相关的安全知识的说明，并提供相应的修复建议。

支持常见开发语言的检测和测试，扫描和分析出有安全漏洞和隐患的源代码。支持C/C++/C#，Java，VB，数据库开发语言Transact-SQL，PL/SQL，大型项目和管理平台开发语言COBOL，ColdFusion，ABAP，Flex，脚本语言 JSP，JavaScript/Ajax，VBScript，Python，网络和网页开发语言 ASP.NET，VB.NET，ASP，PHP，HTML，以及移动应用开发语言 Android，Objective-C等。
#### 使用
fortify sca 扫描nodejs项目，使用以下命令进行构建和扫描，个人感觉对js扫描效果一般
```
sourceanalyzer -b nodejs -clean
sourceanalyzer -b nodejs [扫描目录] -Dcom.fortify.sca.EnableDOMModeling=true  -Dcom.fortify.sca.hoa.Enable=true
sourceanalyzer -b nodejs -scan -Dcom.fortify.sca.Phase0HigherOrder.Languages=javascript,typescript -f results.fpr
```
#### 分析扫描结果
和其他SCA工具分析的思路一样，按照漏洞风险，从高到低，逐条分析。
![fortify scan result analysis](/img/20200624-nodejs-code-review-01.png)

### nodejsscan
#### 介绍
NodeJsScan是专门针对Nodejs应用程序静态代码扫描工具。
#### 使用
该工具安装的方式有多种，推荐使用容器的方式运行此工具,docker使用方法这里不做阐述。
```
docker pull opensecurity/nodejsscan:latest
docker run -it -p 9090:9090 opensecurity/nodejsscan:latest
```
访问对应站点，上传被扫描程序

![nodejsscan upload file](/img/20200624-nodejs-code-review-05.png)

等待片刻，获取扫描结果，进行分析

![nodejsscan result](/img/20200624-nodejs-code-review-06.png)
![nodejsscan result](/img/20200624-nodejs-code-review-07.png)
![nodejsscan result](/img/20200624-nodejs-code-review-08.png)
![nodejsscan result](/img/20200624-nodejs-code-review-09.png)
#### 分析扫描结果
个人感觉nodejsscan对于漏洞识别能力一般，但是还是可以辅助安全人员发现漏洞，毕竟没多少可以用的，有就不错了。

### Retire.js
#### 介绍
Retire.js通过扫描Web应用程序判断是否使用易受攻击的JavaScript库，可以帮助检测已知漏洞的版本的使用情况。
#### 使用
安装：
```
$ npm install -g retire
```
![retire install](/img/20200624-nodejs-code-review-10.png)
使用：
在项目的目录下执行以下指令，即可扫描。
```
$ retire
```
具体参数如下：
```
λ  npx retire --help

  Usage: retire [options]

  Options:

    -h, --help               output usage information
    -V, --version            output the version number

    -p, --package            limit node scan to packages where parent is mentioned in package.json (ignore node_modules)
    -n, --node               Run node dependency scan only
    -j, --js                 Run scan of JavaScript files only
    -v, --verbose            Show identified files (by default only vulnerable files are shown)
    -x, --dropexternal       Don't include project provided vulnerability repository
    -c, --nocache            Don't use local cache

    --jspath <path>          Folder to scan for javascript files
    --nodepath <path>        Folder to scan for node files
    --path <path>            Folder to scan for both
    --jsrepo <path|url>      Local or internal version of repo
    --noderepo <path|url>    Local or internal version of repo
    --cachedir <path>        Path to use for local cache instead of /tmp/.retire-cache
    --proxy <url>            Proxy url (http://some.sever:8080)
    --outputformat <format>  Valid formats: text, json, jsonsimple, depcheck (experimental) and cyclonedx
    --outputpath <path>      File to which output should be written
    --ignore <paths>         Comma delimited list of paths to ignore
    --ignorefile <path>      Custom ignore file, defaults to .retireignore / .retireignore.json
    --severity <level>       Specify the bug severity level from which the process fails. Allowed levels none, low, medium, high, critical. Default: none
    --exitwith <code>        Custom exit code (default: 13) when vulnerabilities are found
    --colors                 Enable color output (console output only)
    --insecure               Enable fetching remote jsrepo/noderepo files from hosts using an insecure or self-signed SSL (TLS) certificate
    --cacert <path>          Use the specified certificate file to verify the peer used for fetching remote jsrepo/noderepo files
```
#### 分析扫描结果
和npm扫描结果分析方式相同，基本上大同小异

## nodejs 修复措施

  1、调用安全组件，如：xss，防止跨站脚本编制<br>
  2、对于sql注入问题，按照编码要求进行编码<br>
  3、部分无法使用安全组件进行修复的场景，可以自行构建过滤器进行处理<br>
  4、对于逻辑漏洞，只能根据实际情况进行处理。


## 参考
- [https://hksanduo.github.io/2019/06/11/2019-06-11-Node-js-security-checklist/) 【nodejs安全清单】
- [https://github.com/analysis-tools-dev/static-analysis] 【static analysis tools list】
- [https://www.microfocus.com/documentation/fortify-static-code-analyzer-and-tools/1910/SCA_Guide_19.1.0.pdf] 【fortify sca 19.0.1用户手册】
- [https://medium.com/@manjula.aw/nodejs-security-tools-de0d0c937ec0] 【nodejs security tools】
- [https://medium.com/ngconf/how-to-analyze-an-angular-project-with-fortify-a3abc729e8d1]
- [https://github.com/mysqljs/mysql#escaping-query-values] 【node-mysql】
- [https://juejin.im/post/5b07eb1c5188254e28710d80] 【Node.js执行系统命令】
- [https://xz.aliyun.com/t/7184#toc-0] 【Node.js 常见漏洞学习与总结】
- [https://docs.npmjs.com/cli/audit] 【npm audit介绍】
- [https://github.com/OpenSecurityIN/nodejsscan] 【nodejsscan github地址】
- [https://retirejs.github.io/retire.js/] 【Retire.js】
