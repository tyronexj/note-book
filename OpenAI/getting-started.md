# OpenAI API 体验之初

## 简介

最近 OpenAI 推出的问答模型 ChatGPT 掀起了新的 AI 热潮，从技术问答到玩场景 play，从代写论文到聊天解闷，有趣到让人产生图灵测试已经不在话下的感觉。

点击这里来体验一下[ChatGPT](https://chat.openai.com/chat)的问答能力。

OpenAI API 顾名思义，就是 OpenAI 提供的一系列具有不同功能级别的模型，并且能够微调您自己的自定义模型。这些模型可用于从内容生成到语义搜索和分类的所有领域。

本文参考了[OpenAI 文档](https://beta.openai.com/docs/introduction)，体验了以下几个功能：

- 文本补全：了解如何使用我们的模型生成或编辑文本
- 代码补全：了解如何生成、编辑或解释代码
- 图像生成：了解如何生成或编辑图像
- 微调：了解如何为您的用例训练模型
- 嵌入：了解如何搜索、分类和比较文本

### 一点点的准备

首先，您需要在这里申请一个[账号](https://beta.openai.com/signup)。

如果您不是一个程序员，那么您可以通过访问[OpenAI API Playground](https://beta.openai.com/playground)来体验这些功能。同时请您跳过本文的代码部分。

如果您是一个程序员，那么您还可以通过编写代码来调用 OpenAI API。本文将使用 Python 语言来调用 OpenAI API。

Python library 的安装方法如下：

```bash
$ pip install -U openai
```

您还需要在[这里](https://beta.openai.com/account/api-keys)申请一个 API key，并且设置到文件或者环境变量中。

以 VS Code 为例，在.env 文件中输入以下内容。

```
OPENAI_API_KEY=您自己的key
```

在 Python 代码中引用它

```python
import os

# Load your API key from an environment variable or secret management service
openai.api_key = os.getenv("OPENAI_API_KEY")
```

## 文本补全

文本补全是 OpenAI API 的一个重要功能，它可以生成或编辑文本。

```python
prompt = "Say this is a test"

response = openai.Completion.create(
    model="text-davinci-003",
    prompt=prompt,
    temperature=0.5,
    max_tokens=1024,
    n=1,
)
```

输出

    This is indeed a test.

我们来逐一解释一下这些参数的含义：

- `model`: 模型名称，这里使用的是 `text-davinci-003`, OpenAI 提供的最强大文本模型。
- `prompt`: 输入的文本，您可以在这里尽情发挥。
- `temperature`: 取值 0-1。这个参数的值越大，生成的文本越随机，越小，生成的文本越有逻辑性。温度这个词语的灵感来自统计热力学，高温意味着更有可能遇到低能量状态。
- `max_tokens`: 顾名思义，最长的文本补全长度。
- `n`: 顾名思义，给出的文本补全的个数。

在`temperature=0`的前提下，可以看到生成的文本是有逻辑性的。

prompt

    明月几时有

output

    把酒问青天

这个输入是来自于苏轼的那首脍炙人口的《水调歌头》。因为设置了`temperature=0`，所以生成的文本基本上是确定的。如果把`temperature`设置为 1.0，那么生成的文本就会有一些随机性。多测试几次，您会发现生成的文本是不一样的。例如，

output

    中国大陆每月的新月都会出现在月初，一般是每个月的第一天凌晨0点左右。

output

    丙申年，8月中旬，大约是8月16号左右有最大的一轮明月。

“明月几时有”，除了理解为《水调歌头》中的一句诗句，也可以理解为一个问句，“月亮什么时候出来？”或者“月亮什么时候最大？”等等，因此有了以上的输出。

`prompt` 提示的设计是一个很有趣的话题，它可以让我们更好地理解 AI 的工作原理。我们来看几个例子。

### 归类 Classification

prompt

    判断下面几句话的情绪是积极，消极还是中性？
    1. 他强由他强，清风拂山冈；他横任他横，明月照大江。
    2. 人生是一场无法改变的轮回，繁华刚刚落幕，寂寞又开始上演。多少金风玉露的相逢，都成了灯火阑珊的错过。
    3. 空即是色，色即是空

output

    1. 积极
    2. 消极
    3. 中性

### 生成 Generation

prompt

    为宠物犬边牧取个名字，他是一条公狗，性格机警。

output

    1. 奥利维亚（Olivia）
    2. 卡洛斯（Carlos）
    3. 泰勒（Tyler）
    4. 拉斐尔（Raphael）
    5. 布莱克（Blake）
    6. 杰克（Jack）
    7. 布鲁斯（Bruce）
    8. 卡尔（Karl）
    9. 布兰登（Brandon）
    10. 布赖恩（Brian）

这类问题几乎没有约束，我把`temperature`设置为 1。

给它增加一点难度，来看一段散文。

prompt

    你要写风，就不能只写风。你要写树梢的弯度，你要写湖面的波纹，你要写树叶婆娑的声音和它落地的路径。
    你要写月，就不能只写月。

output

    你要写月，就不能只写月。你要写星空的璀璨，你要写湖面的朦胧，你要写夜空中温柔的微光和它带来的安慰。
    你要写雨，就不能只写雨。你要写下雨天的湿润，你要写雨点落在树叶上的清脆，你要写雨声滴滴答答的节奏和它滋润大地的力量。

输出中居然出现了 _“月亮带来的安慰”_ 和 _“雨水滋润大地的力量”_ 这样的句子，这是因为 GPT 的训练数据中也包含了类似的描述。这并不是单纯的将 Google 搜索引擎的结果展示出来，而是 GPT 独立生成的文案。这篇文章的原文是这样的。

    你要写风，你就不能只写风。
    要写树梢弯度，写湖面波纹，写树叶婆娑的声音。
    要写屋檐边悬挂的铃销响，
    写轻舟与竹筏轻轻漾，秋千轻轻晃。
    写不听话衣角和发梢，写抓不住的气球，写握紧的伞柄。
    写拨云见日的山，写卷起又落下的浪，甚至是一场散了的大雾。
    ——@听盏

### 对话 Conversation

它不仅可以生成答案文本，还可以根据上下文语境，提出问题和给出回答。

prompt

    Human: Hello, who are you?
    AI: I am an AI created by OpenAI. How can I help you today?
    Human:

output

    Human: What can you do?
    AI: I can help you with tasks such as natural language processing, image recognition, and other machine learning tasks.

给它增加一点难度，来看一段古龙式的对话。

prompt

    “你来了。”
    “我来了。”
    “我知道你会来的！”
    “我当然会来，你当然知道，否则一年前你又怎么会让我走？”

这是古龙小说《天涯明月刀》里傅红雪跟燕南飞的对话。原文是

    “你来了。”
    “我来了。”
    “我知道你会来的！”
    “我当然会来，你当然知道，否则一年前你又怎么会让我走？”
    “现在一年已过去。”
    “整整一年。”
    “好长的一年。”
    “好短的一年。”

古龙就是这个味道，而 OpenAI 也能理解场景和语义，从分离的时间，联想到记忆，从你让我走，联想到记仇，这个输出很接地气。

output

    “哈哈，原来你还记得。”
    “那又怎么样，我可是记仇的！”

### 翻译 Translation

我们来对比以下三个翻译结果。原文选取莎士比亚的《哈姆雷特》的一段。一个是 Google 翻译，一个是 GPT，一个是朱生豪先生的翻译。请读者自己体会。

    To be, or not to be, that is the question:
    Whether 'tis nobler in the mind to suffer,
    The slings and arrows of outrageous fortune,
    Or to take arms against a sea of troubles,
    And by opposing end them?

Google 的翻译结果

    是，还是不是，这是个问题：
    受苦是否在心灵上更高尚，
    无耻命运的弹弓和箭，
    或拿起武器对抗苦海，
    并通过反对结束他们？

GPT 的翻译结果

    也许生存，也许毁灭，这是个问题：
    是否更高尚，在心中承受，
    满目疮痍，悲惨命运的折磨，
    还是拿起武器，对抗烦恼的海洋，
    并以此结束它们？

朱生豪的翻译

    生存还是毁灭，这是一个值得考虑的问题；
    默然忍受命运的暴虐的毒箭，
    或是挺身反抗人世的无涯的苦难，
    通过斗争把它们扫清，
    这两种行为，哪一种更高贵？

### 总结 Summarization

文档案例

prompt

    Summarize this for a second-grade student:
    Jupiter is the fifth planet from the Sun and the largest in the Solar System. It is a gas giant with a mass one-thousandth that of the Sun, but two-and-a-half times that of all the other planets in the Solar System combined. Jupiter is one of the brightest objects visible to the naked eye in the night sky, and has been known to ancient civilizations since before recorded history. It is named after the Roman god Jupiter.[19] When viewed from Earth, Jupiter can be bright enough for its reflected light to cast visible shadows,[20] and is on average the third-brightest natural object in the night sky after the Moon and Venus.

output

    Jupiter is a very bright planet that is the fifth from the Sun and the biggest in our Solar System. It is named after the Roman god Jupiter and has been known to people since before recorded history. It is one of the brightest things you can see in the night sky.

## 代码补全

## 图像生成

## 微调

## 嵌入

## Q/A
