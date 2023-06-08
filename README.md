# Dify_chatbot
Dify 云服务 | 心理学知识库 | 聊天机器人

## 搭建过程

1. 构建数据集：只支持单个文件上传，不能整个文件夹上传，单个文件大小小于15MB，支持文件类型`TXT, HTML, Markdown, PDF`

2. 文本分段与清洗：设置分段与预处理规则
   
3. 文档索引：

    a. 嵌入：调用 OpenAI embedding 接口，在用户查询时准确度更高(0.002$/1000 tokens)
   
    b. 其他索引方法：离线向量引擎索引，关键词索引，倒排索引，位图索引等(0 token)
    
    c. 文档索引结束后，数据集即可集成至应用内作为上下文
    
4. 提示词编排：设置prompt

5. 模型选择：支持 OpenAI, Azure OpenAI API KEY

| 模型 | 能否使用 |
|:---|:---:|
| gpt-3.5-turbo | ✔️ |
| gpt-4 | ✔️ |

注：GPT-4 API 正在申请中无法测试

## 使用

1. Web 端对话机器人：Psychology Q&A，链接在飞书云文档 `Engineering Wiki-算法-Dify chatbot`

2. API : 支持调用对话型应用 API  [官网](https://dify.ai) 
