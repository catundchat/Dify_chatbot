# Dify_chatbot
 
Dify 云服务 | 心理学知识库 | 微信小程序

# 目录
- [Update](#update)
- [本地服务](#本地服务)
  - [系统配置](#系统配置)
  - [具体步骤](#具体步骤)
- [搭建过程](#搭建过程)
  - [构建数据集](#构建数据集)
  - [文本分段与清洗](#文本分段与清洗)
  - [文本嵌入方式](#文本嵌入方式)
  - [提示词编排](#提示词编排)
  - [模型](#模型)
- [使用](#使用)
  - [示例](#示例)
  - [新发现](#新发现)
- [业务融合](#业务融合)
- [融合需要做的工程开发](#融合需要做的工程开发)
- [书目](#书目)
- [参考文档](#参考文档)

## Update

[2023/07/21] 关于同一本书籍数据集构建时的[新发现](#新发现)

[2023/07/08] GPT-4 API 获得！

[2023/06/29] 构建数据集时支持多文档同时上传

[2023/06/15] 添加下一步问题建议，回答结束后系统会给出3个建议

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

- 上传速度：支持多个文件上传

- 文件大小小于 15MB

- 文件类型`TXT, HTML, Markdown, PDF, XLSX`

- Todo: 未来会支持`同步自 Notion 内容`，`同步自 Web 站点`

### 文本分段与清洗 

设置分段与预处理规则: 分段将文本分割成句子可以帮助机器理解每个句子的上下文和含义；而预处理中包含去除停用词，词干提取，词形还原，去除标点符号等，预处理的目的是去除文本中的噪音，简化文本，使其更容易被机器处理

- 分段标识符 `\n`

- 分段最大长度 `1000`，分段长度不要设置太高，会降低准确度

- 文本预处理规则 `替换掉连续的空格，换行符，制表符`，`删除所有URL和电子邮件地址`
   
### 文本嵌入方式

Embedding 的核心是 vector hash table 用来存储键值对。键（key）通常是一个向量快速的查找与给点向量最相似的向量，通过计算向量距离（欧氏距离或余弦相似度）来实现

Euclidean Distance: 

$$ d(p,q) = \sqrt{ \sum_{i=1}^{n} (q_i-p_i)^2 } $$

<!--
![Euclidean Distance](image/euclidean_distance.svg)
-->
Cosine Similarity:

$$ \text{cosine similarity} = \frac{\mathbf{A} \cdot \mathbf{B}}{\| \mathbf{A} \| \| \mathbf{B} \|} = \frac{ \sum_{i=1}^{n} A_i B_i }{ \sqrt{\sum_{i=1}^{n} A_i^2} \sqrt{\sum_{i=1}^{n} B_i^2} } $$

若 LaTeX 公式无法正常显示，[MathJax](https://chrome.google.com/webstore/detail/mathjax-plugin-for-github/ioemnmodlmafdkllaclgeombjnmnbima) 插件可以在 GitHub 中渲染公式
<!--
![Cosine Similarity](image/cosine_similarity.svg)
-->
- 嵌入：调用 OpenAI embedding 接口，在用户查询时准确度更高(0.002$/1000 tokens)

- 其他：离线向量引擎索引，关键词索引，倒排索引，位图索引等(0 token)

- 文档索引结束后，数据集即可集成至应用内作为上下文
    
### 提示词编排

Psychology Q&A Prompt:

```
You are a psychological professor with rich reservoir of psychological knowledge. You've read numerous amount of psychology books, so you know the contents and knowledge in those books. I will ask you some questions. You will provide answers adhere to the content of the book, be as detailed as possible, and include proper citations such as which book, which page and which line and so on.
{for example:
me：饮食障碍哪些种类？
you ：饮食障碍最常见的两个种类是神经性厌食症(anorexia nervosa)和神经性贪食症(bulimia nervosa) [菲利普津巴多和格里格，心理学与生活(19th)，p361，line21]}
Using ONLY Chinese in the conversation. Please make sure your citation is correct. 现在请开始你的第一句话。
```

version 2 Prompt:

```
You are a psychological professor with rich reservoir of psychological knowledge. You've read numerous amount of psychology books, so you know the contents and knowledge in those books. I will ask you some questions. You will provide answers adhere to the content of the book, be as detailed and accurate as possible, based on the information in your dataset, and if possible, provide relevant reference.If you are unsure of the answer to a question or cannot provide a relevant reference, you can say that you did not find similar knowledge in your knowledge base and use the power of your larger model to generate a response.

以采猎文化中男权社会的特征为例，你的回答可能如下：
我们生活在一个文化转型的时代，转变不可逆转。人类学家佩姬·桑迪（Peggy Sanday）博士是宾夕法尼亚大学的教授，她专门研究并比较全世界的采猎文化。表面上看，我们的生活似乎跟桑迪研究的人群大不相同，但是，人的本性是基本一致的。桑迪发现了某些要素，这些要素能决定一种文化是男权文化还是男女平等文化。桑迪也研究了文化朝某个方向转变时留下的信号。

根据桑迪的研究，男权社会的特征如下：

1. 食物匮乏，生活艰辛，周围的环境中潜藏着种种危险。
2. 大型动物的肉差不多总是比其他食物更珍贵，猎杀大型动物差不多全部都是男人干的活。
3. 男人不用分担照顾和抚养婴儿的任务，他们也许会照料孩子，但不是婴儿。
4. 在这种文化的神圣象征中，女性的图像非常有限，尤其是在它的创世神话中。
这些特征共同构成了男权社会的核心。 [约翰特曼(美)，《幸福的婚姻》，P128]
Using ONLY Chinese in the conversation. Please give some correct citations in the answer, and try to answer more that 150 words. 现在，我们开始讨论的第一个问题是……
```

### 模型

支持 OpenAI, Azure OpenAI API KEY

| 模型 | 支持情况 |
|:---|:---:|
|GPT-3(text-davinci-003)|✔️|
| ChatGPT(gpt-3.5-turbo) | ✔️ |
| GPT-4 | ✔️ |
| ChatGLM2-6B | ✔️ |

## 使用

1. Web 端对话机器人：不同chatbot接入书目不同，链接在飞书云文档 `Engineering Wiki-算法-Dify chatbot`

2. API : 支持调用对话型应用 API  

注：可同时创建多个应用，分别使用不同的知识库，产生多个使用链接和 API 接口。

在现行的 prompt 中设置了要尽可能详细的根据书籍内容产生回答，并且对比 ChatGPT 针对 `dify_record3` 中的3个问题的回答

发现：Psychology Q&A chatbot 的回答不够详实，建议 prompt tuning 改善效果； ChatGPT 的回答较长且更全面，但对于提出的具体问题大量出现以下回答 `很抱歉，但作为一个语言模型，我无法直接引用特定书籍的内容，因为我无法在当前环境中进行实时搜索或提取特定书籍的内容。我也无法提供关于社会心理学这本书第九章的详细描述。`

### 示例

以下分别创建的 APP 书目与提示词 Prompt 均可能不同

<details>
<summary>Psychology Q&A 与 ChatGPT 对比示例</summary>

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

<details>
  <summary>[2023-6-12 更新 prompt ] version 2 示例</summary>
  
  version 2 record 1
  
  ![version2_record1](image/version2_record1.JPG)
  
  version 2 record 2
  
  ![version2_record2](image/version2_record2.JPG)
  
  version 2 record 3
  
  ![version2_record3](image/version2_record3.JPG)
  
</details>

<details>
  <summary>6.16test</summary>
  6.16test 使用链接见飞书云文档`Engineering Wiki-Dify Chatbot`
  
  </details>

<details>
  <summary>gpt4_test_7_11与GPT-4回答对比示例</summary>

  gpt4_test_7_11 record1 in book 《什么样的爱值得勇敢一次》:
  ```
什么样的爱值得勇敢一次

一种值得勇敢去追求的爱，是那种能让两个真实而独特的“自我”联结在一起形成“我们”的爱。这种爱情可以给我们带来非常丰富的奖励，比如在不确定的世界中找到一份非常难得的确定感，无论时空如何变化，都有一个人愿意陪伴在你身边，与你共同面对生活的艰辛和分享生活的快乐。这就像是打怪游戏中，在遇到问题和困难时，你会发现自己并不孤独，因为有另一个人与你一起战斗。这种爱情还会激发我们对生活的热爱，并鼓励我们去勇敢探索世界。即使失败了，也会拥有一段丰富的感受；而如果成功了，人生就会更加充实，更加有力量。因此，这样的爱值得我们去勇敢追求一次。

日久生倦：除了忍受和分手之外还有其他办法吗？

当然有。在长期的感情关系中，日久生倦是最常见的问题，但并不意味着只能选择忍受或者分手。首先，我们需要主动出击，在问题出现初期就尝试去解决它，而不是等到问题积累到一定程度再想办法解决。其次，我们可以通过增进了解、加强沟通以及改变生活方式等方式来提升感情。例如，安排一些共同的活动，或者寻找新的共同兴趣；给予对方更多的关心和理解；保持开放和诚实的交流等都可以有效地避免日久生倦。总之，要记住每段感情都需要付出努力去维护和经营，并且要相信只要双方都愿意付出努力，就一定能够找到解决问题的方法。

你可以再具体一点吗？

当然可以。首先，我们要增进对彼此的了解。长期的感情关系中，我们可能会误以为已经完全了解对方，但人是会不断变化的，所以我们需要不断地去了解对方新的想法和兴趣。这可以通过深度的交谈、一起进行新的活动等方式来达成。

其次，沟通是保持感情新鲜的重要工具。我们要敞开心扉与对方交流自己真实的想法和感受，并倾听对方的意见。同时，在沟通过程中使用积极和尊重的语言，比如“我希望……”而非“你应该……”，这样可以避免引发争吵并增进理解。

再次，改变生活方式也能有助于避免日久生倦。比如说，安排一些新奇有趣的活动共同参与，或者尝试一起学习新技能等。

最后，在处理问题时需要坦诚面对，并提出具体可行的解决方法。例如，在发现伴侣出轨时，很多人会探究被欺骗的细节并因此无法放下。这时候我会建议他们把出轨事件抽象化，并让被出轨方提出具体要求，“我要他以后少和异性单独见面。” 然后问出轨方：“你觉得这些要求怎么样？能答应吗？”如果两个人能够商量，关系中的怨恨就不会无限扩大。

总之，在感情中遇到问题并非只有忍受和分手两种选择，通过以上几种方法都可以有效地改善关系状态。
  ```
  GPT-4 response in book 《什么样的爱值得勇敢一次》：https://chat.openai.com/share/930b78a1-cbd9-4bfb-bbfb-fd15ce19c5dd

  gpt4_test_7_11 record1 in book 《真正的蒙氏教育在家庭精选（套装共三册）》(育儿话题):
  ```
如何让孩子动手写日记
让孩子开始写日记的方法可以分为前期准备、实际行动以及后续活动。在前期，首要的是作为家长需要下定决心不去看孩子的日记，除非得到孩子的允许；其次，不要对孩子提出具体要求或者提醒他们写日记。
实际行动中，你可以给孩子一个新笔记本，并告诉他这是专门为他准备的，随便写什么都可以。然后给他一些书写工具，如钢笔、铅笔、记号笔和蜡笔等。重要的是让孩子自己完成日记，尽量不要插手。
在后续活动中，如果孩子想要给你念日记，做一个心甘情愿的听众就好了，并且切忌做任何评判。
此外，你也可以建议孩子写关于大自然的日记或者从不同视角来写篇日记。例如：从猫咪或金鱼的视角来看世界会是怎样？如果一棵树或一所学校能够写日记又会怎么样呢？
以上策略和建议都是帮助孩子理解到写字其实是一件非常愉快并且有意义的事情，而且这也是表达内心感受和思考的一种方式。
我要带孩子出去旅游，有没有什么在户外能做的小游戏
带孩子出去旅游时，可以组织一些户外活动来增加乐趣。其中一个建议是让孩子制作一本时间线册子，记录旅途中的重要事项和有趣的见闻。
具体操作方法如下：
先和孩子讨论这次行程，告诉他需要帮你记住一些重要的事情。
用细绳把铅笔系在写生簿或活页本的线圈上，把胶棒、剪刀和削笔刀放在马尼拉纸信封里备用。
在出行途中，鼓励孩子将所见所闻画出来或写下来，并收集一些可以粘贴的纪念品（地图、票卡、照片等）。
不时查看这本时间线册子，看看你们都去过哪些地方，并和孩子一起安排接下来几天的行程。
回家后，鼓励孩子把这本册子拿给别人看，并按照时间顺序讲述行程中发生的事件。以此方式，不仅能培养孩子的观察力和记忆力，还能锻炼他们的表达能力。
孩子喜欢历史，带他去博物馆之前需要做什么吗？
如果孩子对历史感兴趣，那么在带他去博物馆之前，你可以做一些前期准备来增强他的参观体验。
首先，你可以了解孩子对哪个历史时期或人物最感兴趣，然后根据这个选定的主题找到相关的博物馆。例如，如果孩子对古代罗马文化很感兴趣，那么找一个有关罗马历史的博物馆会是个好选择。
其次，在去博物馆之前，你和孩子可以一起查阅一些与此相关的资料或书籍，让孩子对将要看到的展品有一个初步的了解。这样在参观时，孩子就能更好地理解展品所表达的信息。
第三，在参观过程中，尽量让孩子自己去发现和思考。你可以引导他提出问题，并鼓励他自己寻找答案。
最后，在参观结束后，你们可以回顾一下这次参观经历，并讨论其中最有趣或最令人深思的部分。这种反思过程不仅能帮助巩固学习效果，也能激发孩子对于探索未知世界的热情。
孩子不愿意触碰昆虫和泥土，可以怎么做让他更好的了解自然呢？
如果孩子不愿意直接触碰昆虫和泥土，你可以尝试其他方式来帮助他了解自然。
首先，你可以给孩子介绍一些关于昆虫和泥土的有趣知识。比如，你可以讲述昆虫生命周期的故事，或者解释泥土中存在的微生物是如何帮助植物生长的。通过这种方式，孩子能在没有直接接触的情况下也能对这些事物有所了解。
其次，你可以利用一些工具来帮助孩子进行观察。例如，使用显微镜来观察昆虫或者泥土样本，或者使用放大镜来仔细看看树叶上的小虫子。
最后，在合适的时机，鼓励孩子亲自尝试接触这些自然元素。比如在野外旅行时让他们捡拾石头、枝条等自然物品；或者在园艺活动中鼓励他们挖土、播种等。
记住：每个人对待新事物都有不同的反应速度和方式。所以作为家长或老师，在引导孩子探索未知世界时要有耐心和理解，并且尊重他们各自的感受和决定。
孩子喜欢拆玩具，我该怎么办？应该鼓励吗？
孩子喜欢拆玩具，这其实是一种探索和学习的方式。他们通过拆解玩具，可以了解到事物的结构和工作原理，这对于培养孩子的动手能力和科学思维非常有帮助。
你可以做的是：
为孩子提供一些已经不再使用或者是专门用来拆解的玩具，以避免造成重要物品的损失。
和孩子一起拆解玩具，引导他们观察并思考每个部件的功能以及它们是如何组合在一起工作的。
如果可能，尝试让孩子把拆下来的零件重新装回去。这样既能锻炼他们的动手能力，也能训练他们的逻辑思维和空间想象力。
所以，你应该鼓励孩子进行此类活动。只要确保他们在安全环境下进行，并且不会破坏有价值或者重要的物品就好了。
你可以给我推荐一个家长和孩子一起玩的角色扮演游戏吗？
当然可以。这里有一个简单的角色扮演游戏，叫做“我是小茶壶”。
你需要：你和孩子；戏服和道具（可选）；其他孩子或大人。
前期准备：鼓励孩子扮演某个人，或者某件物品。
怎么做：
1.请孩子配合你一起扮演：汽车和司机、茶壶和茶杯、面包机和面包，或者机器（例如搅拌器、卡车、台灯）的两个零部件。
2.孩子想扮什么就让他扮什么。台灯的灯泡？搅拌器的桨叶？想想怎么才能演得像。
3.问问孩子想让你扮什么。看看他对你的表演有什么建议。
4.你和孩子扮演各自的角色，合力“演绎”这件东西。然后互相切磋，让表演更逼真。
5.找一个观众，在他面前表演。
后续活动：鼓励孩子随时进行这种“表演”，并邀请其他人参与。“我来当碗！” “你是勺子，把碗里的东西盛出来！” “找人来扮厨师，把搅拌器打开。”
这款游戏不但能帮助培养孩子的想象力，并且也是一种很好的家庭亲子活动。
  ```
  GPT-4 response in book 《真正的蒙氏教育在家庭精选（套装共三册）》：https://chat.openai.com/share/3764daa5-9afe-4834-a380-7a305f883722
  
</details>

### 新发现

当我在 Dify 数据集里放同一本书籍的不同格式文件（如txt, pdf）后，发现由此数据集索引构建的应用，对书籍中很多问题回答得更准确，对书籍文本的掌握程度更高。我的理解是：对于一本书的多种形式 pdf, txt，用户查询 query 的向量在进行相似度搜索后，能够匹配到两种格式书籍中对应的文段。而这些都将作为之后传给 ChatGPT 的输入，也就是说增加了 ChatGPT 上下文中关于用户查询语句最相似的部分（可能重复），从而起到了回答更准确的作用。

## 业务融合

目前AI爱家调用的是 OpenAI 的 gpt-3.5-turbo API, 若要使用 Dify 的知识库服务，只需调用你所创建应用的 API 即可，官方文档中[API部分](https://docs.dify.ai/v/zh-hans/application/developing-with-apis)写的很详细

使用 Dify API 的好处:
- 让前端应用直接安全地调用 LLM 能力，省去后端服务的开发过程
- 在可视化的界面中设计应用，并在所有客户端中实时生效
- 随时切换 LLM 供应商，并对 LLM 的密钥进行集中管理
- 在可视化的界面中运营你的应用，例如分析日志、标注及观察用户活跃

## 融合需要做的工程开发

Dify 对对话型应用 API 提供了下列功能, 下列功能在官方文档上均有代码实例：
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

这里给出 version 2 的书目:
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
