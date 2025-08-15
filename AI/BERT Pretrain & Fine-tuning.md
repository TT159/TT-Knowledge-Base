---
date: 2025-06-17
tags:
  - LLM
---

# 通俗理解

预训练像闭卷考试，使模型学习大量的**通用**知识。但不够专精。

微调呢，还是闭卷考试，但是就像考前去特意的针对性的学了一个补习班，针对性的补充，训练了某一个领域的知识。
- 掌握行业属于。深入理解特定行业的专业术语。
- 语境敏感度提升。更好的捕捉和理解特定专业对话语境的差别和意图。有些词在不同专业语境下意义不同。
- 专业知识整合。

RAG，给模型一个外挂知识库，像是一个开卷考试。在模型回答问题时，可以去知识库里去查阅资料。
- 即时信息检索。
- 拓展性强。无需对模型进行更改或者重训练。
- 满足动态需求。
但也有劣势，
- 非常依赖检索回来的信息的质量。之前是通过模型本身的能力来解决。现在模型还需要根据从外部知识库检索回来的信息进行综合回答，但是模型无法知道检索到的信息是真正相关还是不相关。如果知识库里就没有包含与问题相关的知识，那么怎么检索，得到的信息都是错的/不相关的，而模型的回答会收到这检索到的信息的影响。
- 检索会需要额外的计算。如果应用需要非常快的响应速度，可能不合适。

# 微调基本流程

1. 首先得选一个微调的对象，即准备一个预训练模型。

2. 需要准备一个用于微调的数据集。

3. 迭代训练过程。

4. 定期评估性能。使用验证集来评估模型的性能。

5. 优化训练策略。例如，适时调整超参数或改进数据预处理方法等。

6. 模型保存。


- 任务导向。选择与目标任务最匹配的预训练模型，确保模型的初始能力贴合项目需求。
- 模型规模。考虑模型大小于计算资源，平衡性能与硬件之间的关系。
- 领域相关性。优先选择在相似领域预训练的模型。提高微调的效率和效果。


Hugging Face Dataset Hub
Kaggle 竞赛数据
学术论文
Github上的easy dataset 项目


# Bert 预训练 & 微调

**B站视频“BERT带你见证预训练和微调的奇迹”**
**Youtube上有一个不错的视频：https://www.youtube.com/watch?v=gh0hewYkjgo&t=1175s**

BERT from scratch 预训练BERT，资源需求极高。BERT-base 的预训练使用了 **16 张 TPU，训练 4 天**

**预训练**BERT，主要是训练两个任务/方法：
- 完形填空，填空完成句子。**MLM任务**，masked language model 学习文本序列的逻辑和结构
	e.g.: 日照 - 炉生 - 烟
	
- 看两个句子是不是一个句子对。即判断一句话是否是另一个句子的next sentence. 
	**NSP Next sentence prediction 任务** 学习文本中句子和段落的关系。
	e.g.: 判断，遥看瀑布挂前川， 是不是上方那句话的next sentence
	sentence 1: I am going outside.
	sentence 2: I will be back after 6.
	Next sentence?: YES

	sentence 1: I am going outside.
	sentence 2: You know nothing about John.
	Next sentence?: NO

在这样预训练后，可以使模型获得一个通用的全局的文本理解能力，从而去完成一些下游的任务。
==主要就是填空的能力。==

Q: 如果使用这个原始的BERT模型进行一些问答任务，效果会好吗？
A: 效果应该是不怎么好的。因为原始的BERT只是进行了MLM, 和 NSP的训练，并没有进行过QA的训练。所以我们需要针对任务对BERT进行微调。比如NER (命名实体识别任务数据集)，SQuAD (问答任务数据集)，还有MNLI等。不同任务，逐个fine tune。


原始的BERT有很多个类型，即提供了很多不同的任务头，供我们去微调。任务头是一个供我们微调去适应下游任务的适配器（相当于一个接口）。

![[BERT微调.png]]


如今，想要测试一个model的能力，通常会用多个任务来进行检测。BERT可以分化，微调去做各种各样的能力，所以通常不会只用一个任务来进行评估，而是看其在多个不同任务上的表现的平均。

## GLUE (General Language Understanding Evaluation)

**GLUE**（General Language Understanding Evaluation）是一个用于评估自然语言理解（**NLU**）能力的**基准任务集合（benchmark suite）**，旨在衡量通用语言模型在多种语言任务上的表现。由 **纽约大学** 和 **华盛顿大学** 的研究者在 2018 年提出。

> GLUE 被广泛用于评估像 BERT、RoBERTa、GPT 等语言模型的综合能力，是 NLP 领域的重要标准之一。

GLUE 包含 **9 个不同的语言理解任务**，涵盖了分类、句子匹配、语义相似性判断等多种语言能力：

|任务名称|简要说明|任务类型|
|---|---|---|
|**MNLI** (Multi-Genre Natural Language Inference)|判断两个句子之间是“蕴涵”、“矛盾”或“中立”关系|三分类|
|**QNLI** (Question Natural Language Inference)|判断句子是否回答了问题|二分类|
|**QQP** (Quora Question Pairs)|判断两个问题是否等价|二分类|
|**SST-2** (Stanford Sentiment Treebank)|句子的情感极性（正面/负面）|二分类|
|**CoLA** (Corpus of Linguistic Acceptability)|判断句子是否语法合理|二分类|
|**STS-B** (Semantic Textual Similarity Benchmark)|两个句子的相似度打分（0~5）|回归|
|**MRPC** (Microsoft Research Paraphrase Corpus)|判断两个句子是否为复述关系|二分类|
|**RTE** (Recognizing Textual Entailment)|判断句子对是否为蕴涵关系|二分类|
|**WNLI** (Winograd NLI)|推理代词指代关系，极具挑战性|二分类（但不用于评分）|

- 每个子任务根据其性质使用 **准确率（Accuracy）**、**F1 分数** 或 **相关系数（Pearson/Spearman）** 等指标评估。
- **GLUE 评分** 是多个任务结果的加权平均分，用于衡量模型整体语言理解能力。

- 提供了多任务统一评测平台，使得模型可以在不同类型任务中展现“通用性”。
- **SuperGLUE** 于 2019 年发布，是对 GLUE 的**更难版本**，应对模型在原 GLUE 上“满分化”的趋势。
- 包含更加复杂的推理、多跳问题等任务，进一步挑战模型的理解能力。


## SQuAD (Stanford QA Dataset)

**SQuAD（Stanford Question Answering Dataset）** 是由斯坦福大学创建的一个著名的 **机器阅读理解（Machine Reading Comprehension, MRC）** 数据集，用于训练和评估自然语言处理模型在阅读理解任务上的能力。

- **名称**：SQuAD（Stanford Question Answering Dataset）
- **任务类型**：问答（Question Answering, QA）
- **数据来源**：英文维基百科（Wikipedia）文章
- **发布机构**：斯坦福大学（Stanford University）

SQuAD 数据集由多个样本组成，每个样本包含以下字段：

{
  "title": "Apollo 11", // 一篇维基百科文章的标题
  "paragraphs": [
    {
      "context": ""Apollo 11 was the spaceflight that first landed humans on the Moon. Commander Neil Armstrong and lunar module pilot Buzz Aldrin formed the American crew that landed the Apollo Lunar Module Eagle on July 20, 1969.", // 模型必须从中找到问题的答案
      "qas": [  // 这一段文本下的所有问题
        {
          "id": "q1", // 问题编号，唯一ID，用于评估对齐
          "question": "Who was the commander of Apollo 11?", // 需要模型回答的问题
          "answers": [  // 答案数组
            {
              "text": "Neil Armstrong", // 正确答案
              "answer_start": 59 // 答案字符串在context中出现的起始字符位置
            }
          ],
          "is_impossible": false // 是否无法从 context 找到答案（SQuAD 2.0）
        }
      ]
    }
  ]
}

**is_impossible** 是 `false`，表示问题有答案，答案能在 context 中找到。
**答案位置**是 `59`，即 `"Neil Armstrong"` 在原文中的起始字符索引是第 59 个字符。

### SQuAD 1.1
- **发布时间**：2016年
- **数据规模**：
    - 536篇维基百科文章
    - 100,000+ 问题-答案对

- **特点**：
    - 所有问题都有**确切答案**，答案都是上下文中一个连续的**子串**。
    - 没有“无解”问题。

### SQuAD 2.0
- **发布时间**：2018年
- **新增内容**：
    - 50,000+ 无答案问题（no-answer questions），增强了模型判断是否存在答案的能力。

- **任务目标**：
    - 模型必须决定是否问题可以从 context 中找到答案。
    - 如果找不到，应返回“无答案”。

一个问题 + 一个文本段。其答案一定存在文本段里，（除非找不到），所以其实是一个文本抽取的任务，只要在文本段里找到答案词就成功了，而不是那种自己去生成答案。

==从大段文本里抽取出答案词，作为答案。==

### 模型评估方式

SQuAD 通常使用以下指标来评估模型：

- **Exact Match (EM)**：预测答案与真实答案完全一致的比例。
    
- **F1 Score**：预测答案与真实答案之间的词重合程度（精确率与召回率的调和均值）。


SQuAD 是 **最广泛使用的阅读理解数据集之一**，被用于测试各类 NLP 模型（如 BERT、RoBERTa、ALBERT 等）。


## 微调BERT 完成问答任务

这里是代码项目链接：
https://github.com/huangjia2019/chatgpt-briefing?tab=readme-ov-file

1. 准备问题和问答类型任务

2. 下载含有问答任务头的原始版BERT

3. 加载SQuAD特征数据集

4. 用SQuAQ微调BERT

5. 用微调后的BERT做推理，回答问题。