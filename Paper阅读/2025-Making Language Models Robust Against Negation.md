---
date: 2025-05-20
tags:
  - NLP
  - Negation
---
# 主要内容

本文专注于Negation。Previous studies have shown that they struggle with negation in many natural language understanding tasks. In this work, we propose a **self-supervised method** to make language models more robust against negation.

场景/任务 tasks:
Next Sentence Polarity Prediction (NSPP)
given a sentence, the model predicts whether the next sentence will contain negation.

Next Sentence Prediction (NSP) 的变体 variation
In our version, we reverse the polarity of the second sentence to form negative samples rather than select random sentences from the document.
Define reversing polarity as adding (or removing) negation to a sentence that contains (or does not contain) negation.

实验：
作者发现/证明，在他们这些任务上pre-train以后的BERT & RoBERTa 比已有的哪些方法要好，在9个negation-related benchmarks上验证。

同时，发现预训练后在CondaQA (一个需要在negation上进行推理的QA语料库)的性能也提高了。

Contribution:
1) 为pre-training LMs for negation，引入/创建了两种新颖的self-supervised tasks
2) 创建了为这些tasks使用的大型数据集，6.4M samples
3) 证明了进一步pre-training BERT和RoBERTa independently on these tasks improves performance on CondaQA 和其他8个negation-related benchmarks. 
   但是， 也发现，joint pre-training on both tasks, doesn't always improve performance. ==why?==

使用English Wikipedia corpus，使用spaCy generate the dependency tree for each sentence. 是自动化的生成数据的，并不是手工人工整数据集。 

Limitation
1）转化极性的否定仅包含not, n't, never 然而，CondaQA拥有超过200种独特的否定线索。
2）仅experiment with models pre-trained on 500K and 1M instances. 未来可以考虑在整个corpus上训练
3）all the corpora we work with are in English. We acknowledge that negation may be expressed differently in other languages and this work may not generalize to other languages. 考虑其他语言


==**总结**==：
那个转化极性部分挺繁琐的其实，涉及很多语言语法上的，比如有助动词如何添加negation,如何removing negation，时态，yet, at all, but等等，涉及挺多语法的东西，因为肯定不想手动逐个处理，所以最好就是通过一些语法规则，来自动化的构建数据。


## Background

==Observation==: BERT (Devlin et al., 2019) often predicts the same token when negation is added to a sentence.

==假设 Assumption==: hypothesize that this behavior is due to the lack of negation modeling in pre-training.
Specifically, the model has not been exposed to instances where the addition (or removal) of negation influences meaning and coherence within a discourse.

==Method==:  Propose a self-supervised method to make LMs more robust against negation. Our approach is to further pre-train LMs on two tasks that involve negation.


## Two Tasks: NSPP & NSP Variation

### Next Sentence Polarity Prediction (NSPP) task

NSPP as the task of predicting the polarity of the next sentence given the current sentence. 
Given a pair of consecutive sentences, (S1, S2), the input to the model is only S1, and the output is a binary label indicating whether S2 includes any negation cues or not.

例子：
following pair of sentences:

S1: The weather report showed sunny skies.
S2: But it didn’t stay that way.

Given only S1, the model should predict that the following sentence includes negation cues.

一个单句输入、二分类输出的任务。输入S1，预测S2是否包含negation

### Next Sentence Prediction (NSP) variation task

传统的BERT里的NSP任务[[BART & BERT & RoBERTa]]是：NSP task is to predict whether two sentences are consecutive. 判断两个句子是否连续。利用的是Wikipedia的数据，use consecutive sentences as positive examples; chose a random sentence from the same article to replace the second sentence and create a negative example.

提出了一个变体来提升negation的理解能力。
即，对一个连续的句子对，For a pair of consecutive sentences（S1, S2）， we create negative pair ( S1, S2' ) where S2' is obtained by reversing the polarity of S2, 也就是S2的反转极性后的句子，肯定变否定，否定变肯定。that is , if S2 includes negation cues, we remove them, and vice versa.

| 比较点    | 原始 BERT 的 NSP | 本文的 NSP 变体          |
| ------ | ------------- | ------------------- |
| 负例生成方式 | 随机抽取非相邻句子     | 使用 “反转极性” 的 S₂ 作为负例 |
| 是否有否定  | 无特定要求         | 负例通过添加或移除否定构造       |
| 目的     | 学习句子间上下文关系    | 更精细地控制句子语义差异        |
Input: (S1, S2), Label: Yes, S2 is the next sentence.
Input: ( S1, S2‘ ), Label: No, S2’ is not the next sentence.

添加否定并不是简单地加个 “not”，而是要根据动词的类型（是否为助动词、现在分词、过去式等）来灵活调整句法结构，以保证语法和语义的正确性。

当从句子中移除否定时，不仅需要去掉 “not” 或 “n’t”，还要调整整个句子的语法结构，确保语义和时态一致。

在这些极性修改中，**dependency parsing** 用于：
- 判断主语动词是否有 `aux` 或 `auxpass` 辅助结构；
- 判断 `but` 是否与主语动词为并列关系（sibling）；
- 辅助准确定位 negation cue 和动词之间的关系。

作者尝试过使用LLM来完成极性反转任务。
但是，模型总是会修改句子的其他部分，模型会修改其他部分以保持句子含义不变

作者希望 LLM 只完成一件事：
> ✅ **添加或删除否定词（negation cues）**，  
> ❌ **不要改动句子的其它部分**。

但 LLM（如 GPT-4、LLaMA-2）经常不听话，会自动改动句子其它部分，为了让句子在逻辑上**仍然合理或真实**

例如:
我们希望 LLM 的输出像这样：
- 原句：`The computer is working.`
- 添加否定后：`The computer is not working.`（只加了 "not"）
但 LLM 可能输出的是这样的：
- 原句：`The computer is working.`
- 模型输出：`The computer seems to be malfunctioning.` 电脑好像出故障了
改动了句子结构，只是为了让句子更“自然”或更“合理”。

为什么 LLM 会这样做？
作者认为，可能是因为：
1. **Wikipedia 句子是“事实性描述”**，比如科学陈述、历史事件等。
2. **LLMs（GPT-4, LLaMA）被训练要“忠于事实”**（truthfulness）。
3. 所以当 prompt 要求模型生成“与原句相反”的内容时，它**出于事实一致性考虑会抗拒或改写整个句子**。
例如：
The Earth does not revolve around the Sun.
GPT-4 可能拒绝这样输出，因为这是**违背常识**的说法。

作者希望 LLM 只在句子里加/删“not”，但模型为了**保持事实性和语义连贯**，经常自动改写其它部分。这种“过度智能”行为**违背了任务目标**。


## 好词好句

Negation
	n. 反面，对立面，否定，否认
long-standing
	adj. 长期存在的；存在已久的
polarity 
	n. [物]极性；[生]反向性；对立；[数]配极
off-the-shelf 
	adj.现成的，买来不用改就用的
insensitive 
	adj. 感觉迟钝的；（对某事物）无感觉的；（对变化）懵然不知的；不敏感的
coherence 
	n. 一致性，连贯性
discourse 
	n. 演讲，论述；论文；〈语〉语篇，话语 v. 论述
sophisticated  
	adj. 复杂的；精致的；富有经验的；深奥微妙的 v. 使变得世故；使迷惑；(sophisticate的过去分词形式)
	Future work may consider working with more sophisticated rules
streamline
	vt. 把…做成流线型；使现代化；组织；使简单化
	To streamline the process, we only work with sentences that include.....
auxiliary verb  助动词