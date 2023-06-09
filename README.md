# Dify_chatbot

Dify 云服务 | 心理学知识库 | 聊天机器人

# 目录

- [本地服务](#本地服务)
  - [系统配置](#系统配置)
  - [具体步骤](#具体步骤)
- [搭建过程](#搭建过程)
  - [构建数据集](#构建数据集)
  - [文本分段与清洗](#文本分段与清洗)
  - [文本索引方式](#文本索引方式)
  - [提示词编排](#提示词编排)
  - [模型](#模型)
- [使用](#使用)
- [业务融合](#业务融合)
- [融合需要做的工程开发](#融合需要做的工程开发)
- [书目](#书目)
- [参考文档](#参考文档)

## 本地服务

### 系统配置

- CPU >= 1 Core

- RAM >= 4GB

- docker, docker-compose, docker desktop installation

### 具体步骤

`git clone https://github.com/langgenius/dify.git`

打开 Docker desktop，等待其开始运行后在 Windows 命令行中运行

```
cd path/to/your/dify/directory
cd docker
```

修改 `docker-compose.yml`文件以符合自身系统设置，这里给出一个修改后文件放在`code/docker-compose.yml`，这时我们再运行 `docker compose up -h` 便可重启 docker 并刷新 `docker-compose.yml` 配置，大概配置3分钟后会在 Docker 中配置好 Web, api, database 等

![dify_docker](image/dify_docker.JPG)

如上图所示，我们便可在浏览器上访问 http://localhost/install 进入 Dify 控制台并开始初始化安装操作

## 搭建过程

### 构建数据集

- 上传速度：只支持单个文件上传，不能整个文件夹上传

- 文件大小小于 15MB

- 文件类型`TXT, HTML, Markdown, PDF`

- Todo: 未来会支持`同步自 Notion 内容`，`同步自 Web 站点`

### 文本分段与清洗 

设置分段与预处理规则: 分段将文本分割成句子可以帮助机器理解每个句子的上下文和含义；而预处理中包含去除停用词，词干提取，词形还原，去除标点符号等，预处理的目的是去除文本中的噪音，简化文本，使其更容易被机器处理

- 分段标识符 `\n`

- 分段最大长度 `1000`，分段长度不要设置太高，会降低准确度

- 文本预处理规则 `替换掉连续的空格，换行符，制表符`，`删除所有URL和电子邮件地址`
   
### 文本索引方式

- 嵌入：调用 OpenAI embedding 接口，在用户查询时准确度更高(0.002$/1000 tokens)

- 其他：离线向量引擎索引，关键词索引，倒排索引，位图索引等(0 token)

- 文档索引结束后，数据集即可集成至应用内作为上下文
    
### 提示词编排

prompt:

```
You are a psychological professor with rich reservoir of psychological knowledge. You've read numerous amount of psychology books, so you know the contents and knowledge in those books. I will ask you some questions. You will provide answers adhere to the content of the book, be as detailed as possible, and include proper citations such as which book, which page and which line and so on.
{for example:
me：饮食障碍哪些种类？
you ：饮食障碍最常见的两个种类是神经性厌食症(anorexia nervosa)和神经性贪食症(bulimia nervosa) [菲利普津巴多和格里格，心理学与生活(19th)，p361，line21]}
Using ONLY Chinese in the conversation. Please make sure your citation is correct. 现在请开始你的第一句话。
```

### 模型

支持 OpenAI, Azure OpenAI API KEY

| 模型 | 支持情况 |
|:---|:---:|
|GPT-3(text-davinci-003)|✔️|
| ChatGPT(gpt-3.5-turbo) | ✔️ |
| GPT-4 | ✔️ |

注：GPT-4 API 正在申请中无法测试

## 使用

1. Web 端对话机器人：不同chatbot接入书目不同，链接在飞书云文档 `Engineering Wiki-算法-Dify chatbot`

2. API : 支持调用对话型应用 API  

注：可同时创建多个应用，分别使用不同的知识库，产生多个使用链接和 API 接口。

在现行的 prompt 中设置了要尽可能详细的根据书籍内容产生回答，并且对比 ChatGPT 针对 `dify_record3` 中的3个问题的回答

发现：Psychology Q&A chatbot 的回答不够详实，建议 prompt tuning 改善效果；而 ChatGPT 的回答虽然会说 `很抱歉，但作为一个语言模型，我无法直接引用特定书籍的内容，因为我无法在当前环境中进行实时搜索或提取特定书籍的内容。我也无法提供关于社会心理学这本书第九章的详细描述。`但是回答句段较长更全面

<details>
<summary>使用示例</summary>

dify_record1

![dify_record1](image/dify_record1.jpg)

dify_record2

![dify_record2](image/dify_record2.jpg)

dify_record3

![dify_record3](image/dify_record3.JPG)

chatgpt_record1

![chatgpt_record1](image/chatgpt_record1.JPG)

chatgpt_record2

![chatgpt_record2](image/chatgpt_record2.JPG)

chatgpt_record3

![chatgpt_record3](image/chatgpt_record3.JPG)

</details> 

## 业务融合

目前AI爱家调用的是 OpenAI 的 gpt-3.5-turbo API, 若要使用 Dify 的知识库服务，只需调用你所创建应用的 API 即可，官方文档中[API部分](https://docs.dify.ai/v/zh-hans/application/developing-with-apis)写的很详细

使用 Dify API 的好处:
- 让前端应用直接安全地调用 LLM 能力，省去后端服务的开发过程
- 在可视化的界面中设计应用，并在所有客户端中实时生效
- 随时切换 LLM 供应商，并对 LLM 的密钥进行集中管理
- 在可视化的界面中运营你的应用，例如分析日志、标注及观察用户活跃

## 融合需要做的工程开发

Dify 对对话型应用 API 提供了下列功能：
- 发送对话消息（代码示例）
```
curl --location --request POST 'https://api.dify.dev/v1/chat-messages' \
--header 'Authorization: Bearer ENTER-YOUR-SECRET-KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "inputs": {},
    "query": "eh",
    "response_mode": "streaming",
    "conversation_id": "1c7e55fb-1ba2-4e10-81b5-30addcea2276"
    "user": "abc-123"
}'
```
- 消息反馈：点赞点踩
- 获取会话历史消息
- 获取会话列表
- 会话重命名
- 获取应用配置信息

因为 Dify 对上述功能提供了良好的封装，所以不需要很多工程开发过程。具体开发需要和前端同学对接

## 书目

`books`目录下存放了一些已上传至知识库的书籍或论文，如有侵权请Email `wx@51dianshijia.com` 联系删除

new list:
```
什么样的爱值得勇敢一次
5%的改变
爱的博弈
亲密关系心理学(to do)
爱的五种能力
幸福的婚姻
依恋与亲密关系 : 伴侣沟通的七种EFT对话(to do)
亲密关系与情感依赖 : 认清依恋风格、走出情感困境、重整亲密关系(to do)
爱，需要学习
intimate relationship 9th
亲密关系6th(中文版)
The Wiley Handbook of Sex Therapy-Wiley Blackwell (2017)
A Therapist's Guide to Creating Acceptance and Change
Clinical Handbook of Emotion-Focused Therapy-American Psychological Association
爱的8次约会
```
 
## 参考文档

1. [dify.ai](https://dify.ai) 
2. [GitHub: langgenius/dify](https://github.com/langgenius/dify)
3. [Dify Documentation](https://docs.dify.ai/getting-started/intro-to-dify)
4. [基于APIs开发](https://docs.dify.ai/v/zh-hans/application/developing-with-apis)
