---
title: "谷歌 bigquery 配置代理"
date:   2022-05-18  16:57:00 +0800
layout: post
tag:
- Software Supply Chain
categories:
- Security
---

谷歌 bigquery 配置代理

------

## 背景
谷歌云提供bigquery服务，BigQuery是一个RESTful的Web服务 ，可以对与Google Storage结合使用的大型数据集进行交互式分析。它是一种无服务器的平台即服务（ PaaS ），且可与MapReduce相互补充使用。我们需要获取部分数据信息进行分析，由于GFW的原因，需要配置代理服务器获取数据。

## 准备
由于项目主要开发语言为python，这里主要介绍均以python为主。根据BigQuery API Client Libraries的wiki信息，我们首先获取身份认证信息。这个按照wiki一步一步来做即可。最终获取到一个授权的json文件，我们保存到本地的项目当中。

## 构造请求demo
基于BigQuery API Client Libraries 提供的python demo程序，我们自行构造自己的查询程序，代码如下：
```
# -*- coding: utf-8 -*-
from google.cloud import bigquery
import os
import google.auth
AUTH_JSON_FILE_PATH = './bigquery-35b79efd3da3.json'

# 初始化
def bq_InitConnection():
    os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = AUTH_JSON_FILE_PATH
    os.environ["HTTP_PROXY"] = "socks5://127.0.0.1:6666"
    os.environ["HTTPS_PROXY"] = "socks5://127.0.0.1:6666"
    credentials, _ = google.auth.default()
    credentials = google.auth.credentials.with_scopes_if_required(
                  credentials, bigquery.Client.SCOPE)
    authed_http = google.auth.transport.requests.AuthorizedSession(credentials)

    bigquery_client = bigquery.Client(credentials=credentials, _http=authed_http)
    return bigquery_client

# 查询
def query_stackoverflow():
    client = bq_InitConnection()
    query_job = client.query(
        """ custom query sql """
    )

    results = query_job.result()  # Waits for job to complete.

    for row in results:
        print("{} : {} :  {} : {}".format(row.Checks, row.Date,row.Repo,row.Metadata))

if __name__ == "__main__":
    query_stackoverflow()
```
说明：  
* 其中AUTH_JSON_FILE_PATH为授权文件路径，自行配置   
* 代理配置的思路相当于手动配置当前python程序的http和https代理，如果已经配置了系统代理，这步就可以省略了     
* 查询语句根据实际情况自行构建      

![成功获取结果](/img/20220518-01.png)


## 参考
- [https://cloud.google.com/bigquery/docs/reference/libraries#client-libraries-usage-python](https://cloud.google.com/bigquery/docs/reference/libraries#client-libraries-usage-python)【BigQuery API Client Libraries】
- [https://stackoverflow.com/questions/43926668/python3-bigquery-or-google-cloud-python-through-http-proxy/43945207#43945207](https://stackoverflow.com/questions/43926668/python3-bigquery-or-google-cloud-python-through-http-proxy/43945207#43945207)【Python3 BigQuery or Google Cloud Python through HTTP Proxy】
