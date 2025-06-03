---
date: 2025-05-18
tags:
  - NLP
  - EAE
---


**与2025年BEMEAE paper不同在于，这篇文章关注如何提取出argument，即解决EAE task ([[Event Argument Extraction (EAE) Task]] ) 的方法。而那篇文章更多的是对EAE效果的评估，evaluation，即评估提取出来后的arguments 是否正确，优化ESM方法。**

# 主要内容

This paper presents multiple question generation strategies for document-level event argument extraction.
提出了一个新颖的框架，通过**自动生成问题**（question generation, QG）来改进**文档级事件参数抽取**（document-level event argument extraction, EAE）任务。

处理目标既包含 uncontextualized questions 也包含 contextualized questions.

>EAE: is about identifying entities participating in events and specifying their role (e.g., the giver, recipient, and thing given in a giving event). Transforming natural language into structured event knowledge benefits many down-stream tasks such as machine reading comprehension.

现状/limitation: Inter-sentential arguments are more challenging and have received less attention.

文档级 EAE 难点：events与arguments常跨句分布，传统方法依赖模板或局限在句内。

本文提出了一个新的策略：

利用多种策略自动生成问句，**将事件参数抽取转化为问答任务**，并比较不同问句对抽取性能
的影响。该方法使得模型更容易跨句推理、适应不同文档结构，同时降低人工设计模板的负担。

We approach the problem as a question answering task by querying a transformer model.

**Main contributions:**
- introduce several strategies to generate uncontextualized and contextualized questions. 用一个弱监督模型，无需人为定义prompt
- 实验结果显示结合uncontextualized and contextualized questions is beneficial with RAMS.
- 作者的方法可以effective with out-of-domain corpora, event triggers and roles. In particular, the question generation component transfers seamlessly to other corpora. can be easily adapted to any event-argument extraction benchmark—we do not have any benchmark specific component. The only requirement is a list of argument roles an event may have.

## Background

 Initially, datasets focused on extracting arguments within the same sentence as the event.
  起初，事件参数抽取的数据集主要**只关注在与事件触发词位于同一句子中的参数**。
  
- 在早期的数据集（如 ACE 2005）中，**模型只需要从同一句话中抽取参数**。
- 举例：
    `The US army attacked the city at dawn.                   
                 ↑ trigger: "attacked"`
    
    模型只需要从这一句中抽取参数：
    - `agent` → "The US army"
    - `target` → "the city"
    - `time` → "at dawn"
    
- 如果参数出现在**别的句子**，则被忽略或不标注。

随着任务复杂化：
- 新的数据集（如 **RAMS**、**WikiEvents**）开始标注跨句参数
- 现在的任务是 **document-level EAE（文档级参数抽取）**
 
 即：模型需要从整篇文档中找到与事件相关的参数，而不仅限于事件所在句。

---

The current landscape of argument extraction is dominated by prompt-based methods and question-answering techniques.
目前事件参数抽取（argument extraction）领域的主流方法，是基于 Prompt 的方法 和 问答（QA）技术。

---
### 1. Prompt-based methods

利用大语言模型（如 GPT）配合自然语言提示，引导模型完成参数抽取
- 示例 Prompt：
    `"Who is the agent in the attack event in the following text?"`
     Context: The rebel forces launched a surprise offensive at dawn.
- 模型直接输出参数，如："The rebel forces"。
---
### 2. Question-answering techniques

把抽取任务转化为标准的问答格式（像 SQuAD 数据集那样）：
    `Question: Who carried out the attack?   Context: The rebel forces launched a surprise offensive at dawn.   Answer: The rebel forces`
    
- 模型从 context 中预测出一个文本片段（span）作为答案。

these approaches heavily rely on rigid templates, neglecting the valuable context provided by the document.


## Methods & Experiment

### Datasets

All examples in the paper are from RAMS, but we also applied our optimal question-answering strategy to WikiEvents to demonstrate its adaptability.

==补充知识：==
在 NLP 和机器学习中，**“gold” 或 “gold standard”** 的意思是：

> 📌 **人工标注的、被认为是“绝对正确”的标准答案或数据**，通常用于监督训练或评估。

| 类型                | 含义                                         |
| ----------------- | ------------------------------------------ |
| **Gold Answer**   | 人工标注的、正确的事件参数（如：`"oil"`）                   |
| **Gold Question** | 如果存在的话，是由专家编写的、专门用于问这个参数的自然语言问题（但 RAMS 没有） |

作者在本文中提到将RAMS数据集中的event-argument structures转化为question和answer的形式。
然而，该数据集的 **gold event-argument annotations**中包含了**gold answers（即真实的参数文本）**，但并没有同时提供**gold questions（即问这个参数的标准问题）**。
尽管这些答案是已知的，但原始数据中**没有配套的问题**，所以作者设计了多种策略来自动生成这些问题，进而将抽取任务转化为问答任务。

因此：**Propose five question generation strategies divided into two categories:** 
uncontextualized and contextualized. Unlike uncontextualized questions, contextualized questions are grounded on the event and document of interest.

提出了五种问题生成策略，并将其分为两类：**无上下文问题（uncontextualized questions）** 和 **有上下文问题（contextualized questions）**。  
与 uncontextualized 问题不同，contextualized 问题是基于具体事件和文档上下文生成的，更加语义相关和语境贴合。

>即生成两种问题。

---
#### Uncontextualized Questions

Uncontextualized questions only have access to the event trigger (e.g., importing in Figure 1) and role (e.g., artifact).
只依赖两个元素： 
- **事件触发词**（如：`importing`）
 - **参数角色**（如：`artifact`）     
 - 
**不会使用文本上下文或文档内容**，因此：
- **同一个事件+角色始终生成相同的问题**
- 不考虑这个参数是否真实出现在文本中
- 如果不存在，则返回 **"No answer"**


**Uncontextualized 的两种生成方式：**
##### 1. Template-Based Questions
**核心思想：**
> 使用固定格式（模板）构造问句，generate wh-questions for each argument role and event

[Wh-word] is the [argument role] of event [event trigger]?
Wh-word is selected from the following: what, where, who and how
例子：
Q1: What is the artifact of the event importing?

- 模板人工设计
- 不需要实际文本或上下文参与
- 即使该参数在文中根本不存在，问题也会照常生成

生成条件：
- 只需要知道事件可能有哪些 argument role
- 与 ACE、RAMS、WikiEvents 等已有数据集格式兼容


##### 2. Prompt-Based Questions
**核心思想：**
> 不再用人工写模板，而是调用 **GPT-4 等大语言模型**，通过 zero-shot 或 few-shot prompting 方式自动生成问题。

当出现新argument role的时候，用template-base的方式生成question就不方便了，prompt-base的方式就比较好。

流程：
- 使用 GPT-4
- 输入 prompt 结构 + 示例，让模型生成更自然的 wh-questions
- 采用了**zero-shot**和**few-shot**的方式（这算两种策略）。其中，few-shot 模式中，加入了 **10 个随机例子**，enables in-context learning. 这些例子是，These questions are designed to ask for specific roles without being tied to any specific event document.

比template方式更加灵活，适应新角色的能力强。但仍然不使用上下文，模型只基于trigger + role 生成问题。不知道实际语境。


且无文本的问题又是会生成一些不自然或语法不正确的问题（如下例所示）。X is a placeholder for the event trigger.
Template-Based: 
**Where is the place of event [X]?**      结构生硬，不符合英语母语表达习惯
Zero-shot Prompt:
**Where did the event [X] take place?**      结构正常，但可能缺乏具体语境
Few-shot Prompt:
**What is the place or location related to the event [X]?**      表达啰嗦，接近自然但仍显模板感

与其设计复杂规则来处理正确的时态或优化措辞，我们更倾向于使用**有上下文问题（Contextualized Questions）**。

---
#### Contextualized Questions

无上下文问题**完全不利用事件所属文档中的其他内容**，包括：
- 文本上下文
- 其他参数
- 段落逻辑与语义线索
这导致问句往往缺乏自然性与语义针对性。

为了解决上述问题，本文提出使用 **Contextualized Questions（有上下文问题）**。通过一种 weakly supervised method to generate contextualized questions. 

尝试了以下两种方式来collect data to train models for question generation.

##### 1. SQuAD-Based Questions

leveraging SQuAD (Rajpurkar et al., 2016), a question answering dataset not related to event argument extraction. (使用别人收集好的数据集)

SQuAD (document, questions, and answers) as training data.

训练流程：
feeding the model document-answer pairs so that it learns to generate questions.

但是：
作者发现：While these questions are contextualized, and as we shall see it is beneficial to train with them, we found that they are often not grounded on the event of interest or contain misunderstandings. For example, the answer to Q4 from Figure 1, (What did Russia destroy 500 trucks with?) is not oil (oil is the artifact of importing).

==**insights:**== 是不是因为训练的时候没有feed role or event？没有给模型QA所对应的role，导致模型没有学习到这个问题与答案对，是针对什么argument role，和什么event的，所以生成的问题找不到目标。它不知道要生成的问题是关于什么role的，或者事件。就会随机的在document里找关键词来生成问题。

##### 2. Weak Supervised from LLMs

Weak supervision from questions automatically obtained by prompting LLMs.

流程：
To generate more sound contextualized questions, we adopted a targeted approach.

a).   从 RAMS 数据集中随机取 500 个事件样本（约训练集的 7%），对每个样本，提供：
- 文本（document）
- 事件触发词（trigger）
- 参数角色（role）
- 
使用精心设计的 prompt 让 **GPT-4** 生成five wh-questions based on input prompting.
最终大概生成了9000个questions asking for the arguments of events. 
每个样本不止一个argument role，每个事件可能有多个角色（如 agent, place, artifact, destination 等），问题数目是：每个样本事件 x 每个角色 x 每个角色的问题数 (5个)

b).  然后，train a T5 model to generate questions using weak supervision and the 9,000 questions (output) paired with the documents, event triggers and their roles (input).

这样子训练得到的模型可以generate role-specific questions不仅仅是对RAMS数据集包括WikiEvent数据集都行。（这其实就解决了我上面提出的insights）

这样的模型生成的问题，These questions capture both argument and non-argument cues from the document, facilitating the generation of nuanced and semantically diverse questions. Furthermore, using argument roles as input, as opposed to argument spans, facilitates event-grounded knowledge transfer to other datasets.

**弱监督**是指：没有完整人工标注，但借助规则、预训练模型或外部工具**自动产生标签或训练数据**。

> 这里的方法被称为 **弱监督**，是因为questions由 LLM 自动生成、答案已知但问题不是人工精标/或撰写，**属于部分可控、带噪的监督学习数据**。

由GPT生成的问题就作为label了，不是人工标注的，用于训练后面的T5模型。

---

## T5 Model
### 什么是 T5 模型？

> **T5（Text-To-Text Transfer Transformer）** 是 Google 于 2020 年提出的一种 **通用文本生成模型**，它将所有自然语言任务都统一建模为一个“文本到文本”的问题。

📚 论文标题：[_Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer_](https://arxiv.org/abs/1910.10683)

---
### 模型理念：一切皆“文本转文本”

| NLP 任务    | 输入形式                                                             | 输出形式                      |
| --------- | ---------------------------------------------------------------- | ------------------------- |
| 文本分类      | `classify: This movie was great!`                                | `positive`                |
| 问答任务      | `question: Who wrote Hamlet? context: Shakespeare wrote Hamlet.` | `Shakespeare`             |
| 翻译        | `translate English to German: Hello`                             | `Hallo`                   |
| 摘要        | `summarize: The report states...`                                | `The report concludes...` |
| 参数抽取（EAE） | `question: What is the artifact? context: ...`                   | `oil` or `no answer`      |

统一的格式 → 模型结构和训练流程都能共享，提高泛化能力

---
### 模型结构简要

- 基于 **Transformer 编码器-解码器架构**
- 输入：tokenized 文本序列
- 输出：tokenized 目标文本序列
- 类似于序列到序列（seq2seq）模型，但优化了：
    - 多任务训练能力
    - 预训练目标（使用 denoising 目标进行训练）

---
### 在本文中的应用（事件参数抽取）

在这篇事件参数抽取论文中，T5 被训练后，用于：生成 Contextualized Questions

|输入|输出|
|---|---|
|`context: [一段文档]` + `answer: [事件参数]`|`question: [问题]`|

训练过程：
- 使用 SQuAD 或 GPT-4 自动生成的数据 (document, answer, question)
- T5 学习根据上下文与答案反推问题

推理过程中：
- 给定 context + trigger + role，生成自然的、上下文相关的问题
- 让模型回答这些问题以找出参数位置（span）
---
### T5 的优点
- 强大的生成能力：适用于生成式问答、摘要、翻译等
- 支持多任务、多语言
- 可微调（fine-tune）适配各种任务

一句话总结:
> **T5 是一种将所有 NLP 任务统一为“文本转文本”形式的预训练模型**，在本文中它被用于自动生成 contextualized 问题，从而提升事件参数抽取的自然性和语义适应能力。

---

## 好词好句

1) allegation n. 陈诉，宣称，指控
2) smuggle  走私，偷运
3) artifact n. 人工制品，手工艺品
4) landscape n.风景，乡村风景画(风格)， 形势，格局
	The current landscape of argument extraction is dominated by...
	当前的参数抽取研究格局主要由……主导

5) rigid  adj.严格的，死板的，僵硬的
6) neglect   v. 忽视，疏忽，遗漏
7) avenue n. 大街，林荫道，途径
8) contextualize  v. 将....置于上下文中 (或背景)中研究
9) pinpoint  v 确定，准确的指出，精准定位  n. 针尖，精确位置  adj 精确的，精准的， 详尽的
10) seamlessly 无缝地，无空隙地
	the question generation component transfers seamlessly to other corpora
11) be grounded on...   基于....
		contextualized questions are grounded on the event and document of interest.
12) bypass n. 旁道 v. 越过，绕开
		we employ large language models to bypass this limitation.
13) sound n.声音，印象 v.听起来 adj. 明智的，透彻的，严厉的 adv. 充分地
		To generate more sound contextualized questions, we adopted a targeted approach.