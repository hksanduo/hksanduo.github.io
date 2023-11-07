---
title: "使用codeshell进行代码安全评估"
date:   2023-11-17  17:00:00 +0800
layout: post
tag:
- ML
- CodeAudit
categories:
- Linux
---

# 使用codeshell进行代码安全评估

------

## 背景

北京大学软件工程国家工程研究中心知识计算实验室联合四川天府银行AI实验室，正式开源70亿参数的代码大模型CodeShell，成为同等规模最强代码基座。与此同时，团队将软件开发代码助手的完整解决方案全部开源，人手一个本地化轻量化的智能代码助手的时代已经来临！

CodeShell代码：https://github.com/WisdomShell/codeshell     
CodeShell基座模型：https://huggingface.co/WisdomShell/CodeShell-7B  
代码助手VSCode插件：https://github.com/WisdomShell/codeshell-vscode 

目前拥有以下优势：

- **强大的性能**：CodelShell在HumanEval和MBPP上达到了7B代码基座大模型的最优性能
- **完整的体系**：除了代码大模型，同时开源IDE（VS Code与JetBrains）插件，形成开源的全栈技术体系
- **轻量化部署**：支持本地C++部署，提供轻量快速的本地化软件开发助手解决方案
- **全面的评测**：提供支持完整项目上下文、覆盖代码生成、代码缺陷检测与修复、测试用例生成等常见软件开发活动的多任务评测体系（即将开源）
- **高效的训练**：基于高效的数据治理体系，CodeShell在完全冷启动情况下，只训练了五千亿Token即获得了优异的性能

安全狗，重点关注安全检测功能。codeshell号称可以检测代码中的潜在安全风险，如可能出现的SQL注入、跨站脚本攻击等，帮助排查安全性风险。对代码进行深入分析，检测潜在的错误、冗余代码和性能瓶颈，并为开发者提供相应的修复建议；基于代码逻辑，自动创建测试用例，以辅助进行代码测试和验证，确保代码的正确性和稳定性。
## 准备
获取模型
我这里选取的是CodeShell-7B-Chat，如果资源不够，可以采用量化模型。
```
git clone https://huggingface.co/WisdomShell/CodeShell-7B-Chat
```

## 安装

首先克隆项目

```
git clone https://github.com/WisdomShell/codeshell
cd codeshell
conda create -n codeshell python=3.10
conda activate codeshell
pip install requirements.txt
```

昨天作者遗漏requirements.txt，作者响应很及时，很快就补上了，如果需要评估web demo，还需要另外安装依赖

```
pip install demos/requirements_web_demo.txt
```

个人环境还缺乏部分依赖，手动安装就行，灵活处理。
```
pip install bitsandbytes
```

## 运行
```jsx
python web_demo.py --server-port=80 --server-name=0.0.0.0 -c /data/models/CodeShell-7B-Chat 
```
![加载](/img/20231107-01.png)

正常加载demo

![web demo](/img/20231107-02.png)

注意：我这里使用显卡为3090ti，显存为24G，运行全量模型占用显存大约16G，仅供参考。

## 错误：

### ImportError: Using `low_cpu_mem_usage=True` or a `device_map` requires Accelerate: `pip install accelerate`

手动安装

```
pip install accelerate 
```

### torch.cuda.OutOfMemoryError: CUDA out of memory.

换模型或者换显卡资源

## 评估
prompt参考：

```
检查以下java代码，是否存在安全性问题,请给出优化建议:
```

评估代码片段为：

```
package com.suyu.secexample.rce.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

@Controller
public class rcecontroller {

    @RequestMapping("/rce")
    public String input(){
        return "rce/rce";
    }

    @PostMapping("/rceoutput")
    public String index(@RequestParam("command") String command, Model model){
        if(command=="" | command==null){
            command= "whoami";
        }
        Process p = null;
        String result = null;
        try {
            p = Runtime.getRuntime().exec(command);
        } catch (IOException e) {
            e.printStackTrace();
        }
        InputStream is = p.getInputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(is));
        String s = null;
        while (true) {
            try {
                if (!((s = reader.readLine()) != null)) break;
            } catch (IOException e) {
                e.printStackTrace();
            }
            result = s;
        }
        model.addAttribute("result",result);
        return "rce/rceoutput";
    }

}
```

结果如下：
![结果](/img/20231107-03.png)

占用资源：

![占用资源](/img/20231107-04.png)

遇到不准确及幻觉
![回答不准确](/img/20231107-05.png)


## 使用vscode插件
### 安装配置
vscode应用市场检索：CodeShell VSCode，直接安装即可，也可以自行编译安装。
![vscode插件](/img/20231107-06.png)
使用vscode组件，有两种模式，一种使用CPU，采用llama_cpp量化的形式加载，速度稍微缓慢一些，另外一种是使用text-generation-inference(TGI)加载模型。详情详解项目README。

插件配置如下：
![插件配置](/img/20231107-07.png)
注意：
1.正常配置地址和端口
2.这里只支持llama.cpp和tgi两种连接方式，目前(2023-11-07)不支持openai格式，需要手动修改插件
3.tokens根据实际情况修改

### 编译运行llama_cpp_for_codeshell
这里尝试编译llama_cpp_for_codeshell

```
git clone https://github.com/WisdomShell/llama_cpp_for_codeshell.git
cd llama_cpp_for_codeshell
make
```

运行：

```
./server -m ./models/codeshell-chat-q4_0.gguf --host 127.0.0.1 --port 8080
```

注意，这里使用的是量化模型，加载模型要注意。结果没什么影响，只是速度有些缓慢。

### 使用TGI加载模型

```
docker run --gpus 'all' --shm-size 1g -p 9090:80 -v /data/models:/data \
        --env LOG_LEVEL="info,text_generation_router=debug" \
        ghcr.nju.edu.cn/huggingface/text-generation-inference:1.0.3 \
        --model-id /data/CodeShell-7B-Chat --num-shard 1 \
        --max-total-tokens 5000 --max-input-length 4096 \
        --max-stop-sequences 12 --trust-remote-code
```

这里使用南大的ghcr镜像源，速度不错，推荐给大家。个人设备是24G显存，使用TGI加载模型会爆显存。
![oom](/img/20231107-09.png)

### 测试
我这里由于操作系统的语言是英文，导致返回的结果为英文，codeshell是支持中文的。对于代码自动补全我就不演示了，各位自行摸索。
![测试](/img/20231107-08.png)


## 总结
1、可以辅助代码审计人员分析代码，只能辅助，没当作SAST来使用，再说SAST也有大量误报，都需要人工来复核
2、由于codeshell 上下文token限制，没法去分析业务系统，代码释义可能不准确，这些在使用的过程中需要注意
3、codeshell是首个支持代码安全分析的大模型，相当不容易。

## 参考
- [https://github.com/WisdomShell/codeshell](https://github.com/WisdomShell/codeshell)【codeshell】
- [https://mp.weixin.qq.com/s/lLKGDdslHgWhf6Skb-wldg](https://mp.weixin.qq.com/s/lLKGDdslHgWhf6Skb-wldg)【人手一个编程助手！北大最强代码大模型CodeShell-7B开源，性能霸榜，IDE插件全开源】
- [https://github.com/huggingface/text-generation-inference](https://github.com/huggingface/text-generation-inference)【text-generation-inference】
- [https://github.com/WisdomShell/codeshell-vscode](https://github.com/WisdomShell/codeshell-vscode)【codeshell-vscode】

