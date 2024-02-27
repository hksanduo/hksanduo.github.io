---
title: "安装ollama并运行QWEN1.5模型"
date:   2024-02-27  14:27:00 +0800
layout: post
tag:
- ML
categories:
- Linux
---

# 安装ollama并运行QWEN1.5模型

## 介绍

ollama是一个在本地启动并运行大型语言模型的框架。ollama 通过对模型文件进行转化，配置和优化，方便进行多平台部署，包括GPU的使用做了一定的优化，另外LangChain也对其做了集成。

注意：由于ollama及LLM社区日新月异，本文章可能存在时效性，请酌情参考。

## 部署

### linux部署

需要提前安装lspci或者lshw

```jsx
curl -fsSL [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh
```

正常安装，看提示判断是否安装成功。如果不成功，根据实际错误自行解决。

新增以下脚本，并且允许外网访问：

```jsx
#!/bin/bash
# enable global access
export OLLAMA_HOST=0.0.0.0

/usr/local/bin/ollama  serve
```

加载官方已发布镜像

```jsx
ollama run gemma:7b
```

![加载gemma](/img/20240227-01.png)

* 魔法上网，效果更好。

![加载成功](/img/20240227-02.png)

ollama会将缓冲保存到本地用户目录下：`~/.ollama/models` 文件保存类似于docker存储方式

![缓存](/img/20240227-03.png)

如果想从当前服务中移除运行的镜像，有两种方法，1、重启ollama 服务  2、使用rm删除当前镜像，第二种方法有个问题就是镜像得重新下载。

## 自定义镜像

### 导入支持CGUF格式模型

导入cguf格式模型较为简单

1、创建**`Modelfile`文件**

首先创建一个Modelfile。该文件是模型的蓝图，指定权重、参数、提示模板等。

```jsx
FROM ./mistral-7b-v0.1.Q4_0.gguf
```

（可选）许多聊天模型需要提示模板才能正确回答。可以使用Modelfile中的TEMPLATE指令指定默认提示模板：

```jsx
FROM ./mistral-7b-v0.1.Q4_0.gguf
TEMPLATE "[INST] {{ .Prompt }} [/INST]"
```

2、创建Ollama模型

```jsx
ollama create example -f Modelfile
```

3、运行模型

```jsx
ollama run example "What is your favourite condiment?"
```

### 导入支持**(PyTorch & Safetensors)格式模型**

从PyTorch和Safetensors导入的过程比从GGUF导入的过程更长。

1、克隆ollama项目

```jsx
git clone https://github.com/ollama/ollama
```

2、 fetch  `llama.cpp` 子模块:

```jsx
git submodule init
git submodule update llm/llama.cpp
```

3、安装依赖

```jsx
conda create -n ollama python=3.10
conda activate ollama
pip install -r llm/llama.cpp/requirements.txt
```

4、构建quantize工具

```jsx
make -C llm/llama.cpp quantize
```

5、获取或者下载模型到本地

6、转换模型

某些模型架构需要使用特定的转换脚本。例如，Qwen模型需要运行convert-hf-to-gguf.py而不是convert.py

```jsx
python llm/llama.cpp/convert.py ./model --outtype f16 --outfile converted.bin
```

这里不太清楚是否可以直接使用量化模型，先尝试qwen1.5-Qwen1.5-14B-Chat-GPTQ-Int8

转换模型命令如下：

```jsx
python llm/llama.cpp/convert-hf-to-gguf.py /data/models/Qwen1.5-14B-Chat-GPTQ-Int8 --outtype q8_0 --outfile converted.bin
```

目前不支持，错误如下：

[`convert-hf-to-gguf.py](http://convert-hf-to-gguf.py/): error: argument --outtype: invalid choice: 'q8_0' (choose from 'f32', 'f16')`

指定f16遇到新的错误，暂时不清楚如何解决，网上没有检索到类似的错误：

```jsx
Loading model: Qwen1.5-14B-Chat-GPTQ-Int8
gguf: This GGUF file is for Little Endian only
Set model parameters
Set model tokenizer
Special tokens have been added in the vocabulary, make sure the associated word embeddings are fine-tuned or trained.
gguf: Adding 151387 merge(s).
gguf: Setting special token type eos to 151643
gguf: Setting special token type pad to 151643
gguf: Setting special token type bos to 151643
gguf: Setting chat_template to {% for message in messages %}{{'<|im_start|>' + message['role'] + '
' + message['content'] + '<|im_end|>' + '
'}}{% endfor %}{% if add_generation_prompt %}{{ '<|im_start|>assistant
' }}{% endif %}
Exporting model to 'converted.bin'
gguf: loading model part 'model-00001-of-00005.safetensors'
token_embd.weight, n_dims = 2, torch.float16 --> float16
blk.0.attn_norm.weight, n_dims = 1, torch.float16 --> float32
blk.0.ffn_down.bias, n_dims = 1, torch.float16 --> float32
Can not map tensor 'model.layers.0.mlp.down_proj.g_idx'
```

尝试使用无量化模型Qwen1.5-14B-Chat

错误，这个原因是由于镜像内未安装git-lfs

```jsx
gguf: loading model part 'model-00001-of-00008.safetensors'
Traceback (most recent call last):
  File "/data/project/qwen-ollama/ollama/llm/llama.cpp/convert-hf-to-gguf.py", line 1937, in <module>
    main()
  File "/data/project/qwen-ollama/ollama/llm/llama.cpp/convert-hf-to-gguf.py", line 1931, in main
    model_instance.write()
  File "/data/project/qwen-ollama/ollama/llm/llama.cpp/convert-hf-to-gguf.py", line 152, in write
    self.write_tensors()
  File "/data/project/qwen-ollama/ollama/llm/llama.cpp/convert-hf-to-gguf.py", line 113, in write_tensors
    for name, data_torch in self.get_tensors():
  File "/data/project/qwen-ollama/ollama/llm/llama.cpp/convert-hf-to-gguf.py", line 71, in get_tensors
    ctx = cast(ContextManager[Any], safe_open(self.dir_model / part_name, framework="pt", device="cpu"))
safetensors_rust.SafetensorError: Error while deserializing header: HeaderTooLarge
```

执行转换，占用的是CPU，无GPU占用

![cpu占用率](/img/20240227-04.png)

cover成功

![转换成功](/img/20240227-05.png)

7、量化模型

```jsx
llm/llama.cpp/quantize converted.bin quantized.bin q4_0
```

针对qwen特殊配置，这里是量化int8

```jsx
llm/llama.cpp/quantize converted.bin qwen_v1.5_quantized_int8.bin q8_0
```

成功

![量化成功](/img/20240227-06.png)

8、构造文件 **`Modelfile`**

```jsx
FROM qwen_v1.5_quantized_int8.bin
TEMPLATE "[INST] {{ .Prompt }} [/INST]"
```

9、创建一个 **Ollama 模型**

最终，从Modelfile创建一个模型

```jsx
ollama create example -f Modelfile
```

针对qwen

```jsx
ollama create qwen1.5-int8 -f Modelfile
```

![创建模型](/img/20240227-07.png)

10、运行你的模型

```jsx
ollama run qwen1.5-int8 "who are you?"
```

![运行模型](/img/20240227-08.png)

接口测试

```jsx
curl http://192.168.3.199:11434/api/generate -d '{
  "model": "qwen1.5-int8:latest",
  "prompt": "Why is the sky blue?"
}'
```

![api测试结果](/img/20240227-09.png)

继续测试（No streaming）

```jsx
curl http://192.168.3.199:11434/api/generate -d '{
  "model": "qwen1.5-int8:latest",
  "prompt": "Why is the sky blue?",
  "stream": false
}'
```

![api测试结果](/img/20240227-10.png)

消耗资源

![GPU占用资源](/img/20240227-11.png)

此处显存是同时加载：gemma:7b和Qwen1.5-14B-Chat-GPTQ-Int8

更多接口格式，请参考ollama官方文档。

### 注意

1. 对于在docker内部运行来说，如果没有启动systemd，所以需要手动启动对应服务
2. ollama会建立一个ollama账户，用户目录位于`/usr/share/ollama`
3. ollama默认绑定127.0.0.1 11434端口，目前可以修改host，但是无法修改port，等待社区解决或者手动映射端口

## 参考

[https://github.com/ollama/ollama](https://github.com/ollama/ollama)

[https://ollama.com/download/linux](https://ollama.com/download/linux)

[http://m.tnblog.net/hb/article/details/8200](http://m.tnblog.net/hb/article/details/8200)

[https://zhuanlan.zhihu.com/p/671840823](https://zhuanlan.zhihu.com/p/671840823)

[https://github.com/ollama/ollama/blob/main/docs/api.md](https://github.com/ollama/ollama/blob/main/docs/api.md)

