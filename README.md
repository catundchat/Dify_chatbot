# Dify_chatbot

Dify 云服务 | 心理学知识库 | 聊天机器人

## 搭建过程

### 构建数据集

- 上传速度：只支持单个文件上传，不能整个文件夹上传

- 文件大小小于 15MB

- 文件类型`TXT, HTML, Markdown, PDF`

- 未来会支持`同步自 Notion 内容`，`同步自 Web 站点`

### 文本分段与清洗 

设置分段与预处理规则

- 分段标识符 `\n`

- 分段最大长度 `1000`，分段长度不要设置太高，会降低准确度

- 文本预处理规则 `替换掉连续的空格，换行符，制表符`，`删除所有URL和电子邮件地址`
   
### 文本索引方式

- 嵌入：调用 OpenAI embedding 接口，在用户查询时准确度更高(0.002$/1000 tokens)

- 其他：离线向量引擎索引，关键词索引，倒排索引，位图索引等(0 token)

- 文档索引结束后，数据集即可集成至应用内作为上下文
    
### 提示词编排

设置prompt

# 模型

支持 OpenAI, Azure OpenAI API KEY

| 模型 | 支持情况 |
|:---|:---:|
|GPT-3(text-davinci-003)|✔️|
| ChatGPT(gpt-3.5-turbo) | ✔️ |
| GPT-4 | ✔️ |

注：GPT-4 API 正在申请中无法测试

## 使用

1. Web 端对话机器人：Psychology Q&A，链接在飞书云文档 `Engineering Wiki-算法-Dify chatbot`

2. API : 支持调用对话型应用 API  
 
## 参考文档

1. [dify.ai](https://dify.ai) 
2. [GitHub: langgenius/dify](https://github.com/langgenius/dify)
