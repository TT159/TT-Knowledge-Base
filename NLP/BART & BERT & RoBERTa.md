---
date: 2025-05-19
tags:
  - NLP
  - Transformer
---
## 📌 BART 是什么模型？

> **BART（Bidirectional and Auto-Regressive Transformer）** 是由 Facebook AI 提出的一种 **文本生成模型**，结合了 **BERT 的编码器** 和 **GPT 的解码器** 架构。

📚 论文：[_BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension_](https://arxiv.org/abs/1910.13461)

---

## 🏗️ BART 的结构：Encoder-Decoder 架构

|模块|类似于哪种模型|功能|
|---|---|---|
|Encoder|BERT|双向上下文理解（bidirectional）|
|Decoder|GPT|自回归生成（auto-regressive）|

因此，BART 同时具备：

- **理解输入的能力（如 BERT）**
- **生成文本的能力（如 GPT）**
---

## 🔧 BART 的预训练方式

BART 采用 **文本去噪自编码器（denoising autoencoder）** 进行预训练：

1. 对原始文本进行扰动（如：遮蔽、打乱顺序、删除单词）
2. 模型学习从被破坏的输入恢复出原始文本

这种方式使得 BART 能很好地用于：

- **文本生成**
- **摘要（summarization）**
- **问答（QA）**
- **机器翻译**
- **事件参数抽取（EAE）中的问答式抽取任务**

---

## 🔍 BART 与 BERT 的区别对比表

| 特性      | **BERT**          | **BART**                      |
| ------- | ----------------- | ----------------------------- |
| 架构类型    | 编码器（Encoder-only） | 编码器 + 解码器（Encoder-Decoder）    |
| 上下文感知   | 双向（Bidirectional） | 编码器双向，解码器单向                   |
| 预训练目标   | 掩码语言建模（MLM）       | 文本重构（denoising autoencoding）  |
| 是否能生成文本 | ❌ 否（只用于理解任务）      | ✅ 是（用于生成类任务）                  |
| 适合的任务类型 | 分类、抽取、命名实体识别等     | 摘要、问答、翻译、生成式抽取等               |
| 在本文中的用途 | 没有用到              | 用于生成式问答（从问题中预测 argument span） |


---

## ✅ 使用 BART 的例子

> [[2024-Generating Uncontextualized and Contextualized Questions for Document-Level Event Argument Extraction]] 文章中，将事件参数抽取转化为问答任务（输入问题 + 文本，输出 span），**所以使用 BART 这类可以“理解+生成”的模型更加合适**。

它可以处理输入格式为：
`Question: What is the artifact of importing? Context: They are importing not only oil but wheat...`

生成：
`Answer: oil`

---

## ✅ 总结

> **BART 是一种编码-解码结构的文本生成模型，融合了 BERT 的理解能力与 GPT 的生成能力，非常适合用于问答、摘要等生成式任务。** 与只用于理解的 BERT 不同，BART 可用于本文的问答式事件参数抽取任务。

> **BERT 是一个“理解型”的模型**，主要将输入文本转换成高维的向量表示（如 768 维 embedding），适用于分类、抽取等任务；

> **BART 不仅具备 BERT 的编码理解能力，还结合了 GPT 的解码生成机制**，可以用于文本生成任务，如问答、摘要、翻译等。**BERT 是文本转向量的“理解器”，而 BART 是理解器 + 生成器的结合体**。BART 不仅能提取语义表示，还能直接生成文本输出。


| 维度            | **BERT**                         | **BART**                             |
| ------------- | -------------------------------- | ------------------------------------ |
| 📚 主要用途       | 文本理解 → 向量表示（embedding）           | 文本理解 + 文本生成                          |
| 📦 输入 → 输出格式  | 文本 → 向量（例如 `[CLS]` token, 768 维） | 文本 → 文本（seq2seq：输入问题和上下文，输出答案）       |
| 🧠 架构         | **Encoder-only**（双向编码）           | **Encoder-Decoder**（编码器理解输入，解码器生成输出） |
| 🏗️ 与 GPT 的联系 | ❌ 没有                             | ✅ Decoder 部分类似 GPT：左到右生成文本           |
| 💡 是否可直接生成文本  | ❌ 不行                             | ✅ 可以（例如问答、摘要）                        |
| 🎯 应用举例       | 文本分类、NER、句子相似度、QA（extractive）    | 生成式问答、文本摘要、机器翻译、EAE 中生成参数 span       |

[BERT]
Input  →  Encoder →  Embedding（理解型输出）

[BART]
Input  →  Encoder →  Decoder → Generated Text（理解 + 生成）


## Bert里的NSP任务

“**Next Sentence Prediction (NSP)**” 是 BERT 模型在预训练阶段使用的一个 **辅助任务（auxiliary task）**，它的目的是帮助模型学习句子级别的关系，即判断两个句子之间是否是上下文连续的。

### NSP 任务的定义

**任务目标：**  
给定一对句子 A 和 B，模型需要判断：
- **B 是否是 A 的下一句**（即上下文中连续出现的句子）。

**二分类任务：**
- `IsNext`：B 是 A 的下一句。
- `NotNext`：B 不是 A 的下一句。

### 在 BERT 中的实际操作

在 BERT 的预训练中，每对句子对的构建如下：
- 50% 的概率选择一对实际的连续句子（IsNext）。
- 50% 的概率随机选择另一段文本中的句子作为 B (NotNext)。

例子：
- A: "I went to the store to buy some groceries."  
  B: "I found some fresh apples there." → `IsNext`

- A: "I went to the store to buy some groceries."  
  B: "The Eiffel Tower is located in Paris." → `NotNext`

### 输入表示形式

BERT 对句子对的输入格式如下：
[CLS] Sentence A [SEP] Sentence B [SEP]
模型使用 `[CLS]` token 的输出 embedding 来做 NSP 分类（即二分类的 softmax）。

在 BERT 中，[CLS] 和 [SEP] 是两种特殊的 token（标记），它们不是自然语言里的词，而是 专门用于 BERT 模型结构和任务的功能性符号。

`[CLS]` 是什么？ 
全称：Classification token  
位置：总是在输入序列的最开头。 
作用：
放在输入序列的**最前面**，并不表示自然语言的内容，只是告诉 BERT：
> “这是整个输入的起始，我想要你基于整个输入做一些分类任务。”

在 BERT 中，[CLS] 的输出 embedding（隐藏向量）被用作整个输入序列的代表。
CLS的输出embedding指的是：经过 BERT 模型处理之后，**`[CLS]` 位置对应的那个输出向量（embedding）**。并不是这整句话的 embedding，也不是其他词的 embedding，而是 `[CLS]` 这个 token 自己的位置的 embedding。

==但重要的是==：**这个 embedding 被认为编码了整个输入序列的意思**，所以后面的分类器会用它来做判断。提供整个输入的语义总结，用于分类任务。
在分类任务（如情感分析、NSP）中，BERT 会基于 [CLS] 的输出做最终的预测。

例子：
比如我们要做 **情感分类**，判断这句话是正面还是负面：
> "I love this movie!"

BERT 实际输入是：
[CLS] I love this movie ! [SEP]
每个词都会被切成 token，并有自己的位置，每一个token都是一个768维的向量

[CLS] → 一个768维的向量，比如：[0.2, -0.3, ..., 0.1]
经过BERT训练，这个向量是整个句子的“语义表示”，就像整个句子的总结

最终分类怎么做？
我们会在 BERT 后面接一个分类器，比如一个全连接层（linear layer）+ softmax：

> logits = classifier(output_from_CLS)

输出：
- Positive 情绪？Negative 情绪？Neutral 情绪？
- 就是靠 `[CLS]` 的那个 embedding 来判断的。

#### 为什么 `[CLS]` 能表示整个句子的意思？

这是 BERT 训练出来的效果。BERT 的 Transformer 层会让 `[CLS]` 这个位置可以“关注”（self-attention）到所有其他 token。所以它最后的 embedding 实际上融合了**整个输入的信息**。

`[CLS]` : 是一个特殊的 token，放在最前面，用来做“句子级别”的分类任务
`[CLS] 的输出embedding` ：是 BERT 对这个 token 位置输出的向量，融合了全句信息

`[SEP]` 是什么？
全称：**Separator token**
位置：用于分隔句子。分隔两个句子；表示句子结束
作用：
把两个句子 A 和 B 分开。
如果只有一个句子，也要加一个 `[SEP]`，表示句子结束。


### NSP 的作用

NSP 帮助模型学习：
- 句子之间的**上下文连接**。
- 改进像 QA（问答）和 NLI（自然语言推理）这类任务中的**跨句推理能力**。

不过，后来研究发现 NSP 并不是那么有效（如 RoBERTa 直接去掉 NSP 仍然表现更好），但它仍然是 BERT 最初训练中的一个重要组成部分。



## RoBERTa

RoBERTa: A Robustly Optimized BERT Pretraining Approach

它是由 Facebook AI 在 2019 年提出的，核心目标是：**让 BERT 更强更稳**。虽然它名字像是“BERT 的妹妹”，但本质上它是对 BERT 的一些关键训练策略进行了优化。

核心思路：
不改架构，只改训练方式。（Same architecture, better pretraining）  
它还是用 BERT 的 Encoder， Transformer 层，但靠更好的训练策略和更多数据，让模型性能更强。

结果：
在多个 NLP 任务（GLUE、SQuAD、RACE、MNLI）中，RoBERTa 都比原始 BERT 表现更好，甚至一度刷新了多个基准测试的记录。


BERT: 开创性地引入 Masked Language Model 和 NSP 任务
RoBERTa: 去掉 NSP，更大数据、更久训练、更强性能

| 改进点                                  | BERT 原始设置                    | RoBERTa 改进做法                          | 原因                         |
| ------------------------------------ | ---------------------------- | ------------------------------------- | -------------------------- |
| 1️⃣ NSP 任务（Next Sentence Prediction） | 有                            | ❌ **取消 NSP 任务**                       | 实验证明 NSP 没有显著提升效果，反而可能干扰训练 |
| 2️⃣ 输入长度                             | 最多 512 个 token               | ✅ 更灵活地使用长输入                           | 利用更多上下文                    |
| 3️⃣ 训练数据                             | 16GB（Wikipedia + BookCorpus） | ✅ 使用 **更多数据**，比如 CommonCrawl，共计 160GB | 训练更稳健，泛化更强                 |
| 4️⃣ 训练轮数                             | 1M steps                     | ✅ 增加训练步数（10x）                         | 训练更充分                      |
| 5️⃣ 动态 mask                          | ❌ 固定 mask（预处理时决定哪些词被 mask）   | ✅ **动态 mask**（每次训练重新随机 mask）          | 避免模型“记住” mask 位置，提高泛化能力    |
| 6️⃣ batch size & 学习率                 | 普通设定                         | ✅ 使用 **大 batch size、大学习率**            | 提高训练效率与性能                  |