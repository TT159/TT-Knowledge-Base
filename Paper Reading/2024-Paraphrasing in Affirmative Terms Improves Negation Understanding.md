---
date: 2025-05-21
tags:
  - NLP
  - Negation
---

# 主要内容

In this paper, we experiment with seamless strategies that incorporate affirmative interpretations (i.e., paraphrases without negation) to make models more robust against negation.


**作者的目标**是：把带有否定的句子，用**不带否定词的句子**进行改写（paraphrase），以增强自然语言理解系统在遇到否定时的鲁棒性。

用一个不包含否定的改写来补充包含否定的输入。

For example:
I don't like this movie. <=> This movie is boring.

Affirmative interpretations are obtained automatically

==提出得到不含否定的否定句子的改写句的方法，然后再将这种affirmative term作为辅助信息，用CondaQA等QA benchmarks进行评估，看是否这种affirmative terms或者说这种改写信息能否提高模型对否定的理解能力。==

## Background

Negation is a fundamental linguistic phenomenon present in all human languages (Horn, 1989). Language models underperform in various natural language understanding (NLU) tasks when the input includes negation.

BERT fails to distinguish between negated and non-negated cloze questions. 

有人analyze negation in eight popular corpora for six Nature Language Understanding (NLU) tasks. 

Conclude that (a) NLU corpora have few negations compared to general-purpose texts and (b) the few negations in them are often unimportant. (a)与通用文本相比，NLU语料库中的否定词很少；(b)其中的少数否定词往往不重要

CondaQA is the largest benchmark (14,182 question-answer pairs from Wikipedia) requiring reasoning over the implications of negations.


**Related Work**
The first work on affirmative interpretations was by Sarabi et al. (2019). Hossain et al. (2022b) present AFIN, a corpus of ≈3,000 sentences with negations and their affirmative interpretations.

Sarabi 等人（2019）首次提出了肯定性解释的研究；Hossain 等人（2022b）构建了一个包含约 3,000 个否定句及其肯定性改写的数据集 AFIN。但这两项工作**仅限于生成**肯定性解释，**并未在具体的自然语言处理任务中评估其实际效果**。

Extrinsic evaluation 指的是**在具体应用场景中评估某个方法的效果**，比如看看是否真的提高了问答系统的准确率、情感分析的鲁棒性等。

More recently, Hossain and Blanco (2022) present Large-AFIN, over 153,000 pairs of sentences with negation and their affirmative interpretations obtained from parallel corpora via backtranslation.
这些句子是通过 **平行语料库 + 回译（backtranslation）** [[NLP Concepts]] 的方式自动获得的。


==而**本文提出了一种新的方法**，可以**不依赖平行语料库或机器翻译系统**，来生成肯定性解释。==


## Contributions

1) 提出strategies to generate and incorporate affirmative interpretations
2) experimental results showing that doing so yields better results.


## Methods

An affirmative interpretation generator is a system that takes a sentence with negation as its input and outputs an affirmative interpretation.

- input: a sentence with negation
- output: an affirmative interpretation

这个任务与paraphrase generation是相似的，不过多了一个限制条件，the output must **not** contain negation.

作者提出了2种方法来generate affirmative interpretations

### First approach
 
 an off-the-shelf T5 (Raffel et al., 2020) fine-tuned by Hossain and Blanco (2022) with Large-AFIN (Section 1) to generate affirmative interpretations. 

将这个模型称为T5-HB。

作者使用了一个现成的 **T5 模型**，这个 T5 是 **Hossain 和 Blanco 在 2022 年使用 Large-AFIN 数据集**（包含 15 万个带否定的句子和其肯定性解释对）**微调过的**。

### Second approach

bypasses the need for a large collection of pairs of sentences with negation and their affirmative interpretations. 
第二种方法**不需要大量的“否定句 ↔ 肯定解释”配对数据pair**，也就是不依赖 Large-AFIN 这种平行语料。

It is based on the work by Vorobev and Kuznetsov (2023), who fine-tuned T5 on a paraphrase dataset obtained with ChatGPT (419,197 sentences and five paraphrases per sentence). 

这方法基于 Vorobev 和 Kuznetsov（2023）的工作。
他们用 ChatGPT 自动生成了一个 paraphrase 数据集（约 42 万句子，每个句子生成 5 个同义改写），然后用这个数据集对 T5 进行微调。这个模型是被训练来生成paraphrases的，并不是训练来生成affirmative interpretations的

这个模型被称为T5-CG。

We obtain affirmative interpretations with T5-CG by generating five paraphrases and selecting the first one that does not contain negation.
使用方法是：**让它生成 5 个 paraphrases，然后从中选出第一个不含否定词的句子**，作为肯定性解释。

**怎么用 T5-CG 得到 affirmative interpretation 的？**
作者采用了一个**启发式方法（heuristic trick）**：

**步骤如下：**

1. 给 T5-CG 一个输入句，比如：
     _I don’t like this movie._

2. T5-CG 会生成 **5 个同义句**，比如：
    - I dislike this movie.
    - I’m not a fan of this movie.
    - I don’t enjoy this film.
    - This movie isn’t enjoyable.
    - This film doesn’t appeal to me.

3. 然后，**从中筛选第一个不含否定的句子**（没有 “not”, "don’t", "isn’t", "dislike", "lack" 等表达 negation 的词），比如：
    - 如果这些都带 negation，就跳过；
    - 如果它生成了 "This movie is boring"，那就是一个符合条件的 affirmative interpretation。

4. 作者称这种生成方式为 **ACG（Affirmative interpretation from ChatGPT-based model）**。

---

|模型|训练目标|是否要求去除否定|输出句中可能包含否定？|如何获得 affirmative|
|---|---|---|---|---|
|T5-HB|专门训练生成“无否定”的肯定性解释|✅ 是|❌ 不应该有否定|直接输出|
|T5-CG|训练生成 paraphrase（不限制否定）|❌ 否|✅ 有可能有否定|从中筛选出第一个没有 negation 的句子|

---

### Negation Cues

We use all negation cues in CondaQA to identify negation cues in our experiments. 

CondaQA contains over 200 unique cues, including single words (e.g., inaction, unassisted, unknown), affixal negations (e.g., dislike, unmyelinated, unconnected, inadequate, impartial), and multiword expressions (e.g., a lack of, in the absence of, no longer, not at all, rather than).

**作者使用了 CondaQA 数据集中收集的所有“否定提示词（negation cues）”来识别否定表达**。

CondaQA 数据集**包含了 200 多种不同类型的否定提示词**，包括单词形式，词缀否定，多词短语，不同词性的否定词等等。

CondaQA 是一个 **问答（QA）数据集**。

> **CondaQA: A Contrastive Dataset for Understanding the Impact of Negation in QA**

它是由 **Abhilasha Ravichander**, **Matt Gardner**, 和 **Ana Marasović**  在 **EMNLP 2022** 上发表的论文，专门用于研究“否定语义如何影响问答模型的理解和推理能力”。"CONDAQA: A Contrastive Reading Comprehension Dataset for Reasoning about Negation"

- **任务类型**：阅读理解（Reading Comprehension）
- **重点研究内容**：模型是否能正确推理包含**否定（negation）**的文本
- **数据结构**：构造**成对对比样本**（contrastive pairs），每对中只有一个因素不同 —— 是否包含 negation，用以分析模型对“否定”理解的敏感性和鲁棒性。

---

## Experiment

We use RoBERTa-Large (Liu et al., 2019) as the base model. In addition to experimenting with the original inputs for a task (e.g., passage and question from CondaQA), we **couple** the original input with one affirmative interpretation of the sentence with negation (if any; no change otherwise). Affirmative interpretations are **concatenated** to the original input after the \<sep\> special token. Our approach is the same regardless of the type of negation.
作者在使用 **RoBERTa-Large** 模型进行自然语言理解任务（如 CondaQA）时，为了增强模型对**否定句的理解能力**，引入了一个额外策略：**将“肯定解释”（affirmative interpretation）作为辅助输入**。

做了增强处理：
- **如果输入中包含否定句**，就会取出其中的一句否定句，并使用一个**对应的肯定解释**；
- 然后将这个肯定解释**附加到原始输入上**；
- **如果没有否定句**，则不做任何修改。

无论是什么形式的否定（单词、前缀、词组、词汇化否定等），他们都使用同样的策略来处理：
	如果句子中含有否定结构，就找出对它生成的肯定解释并拼接进模型输入。

 However, not all of the improvements are statistically significant. The only statistically significant improvements are with (1) the scope edit type when trained with P+Q+A_CG or A_CG and tested with P+Q+ACG, and (2) the affirmative edit type when trained with P+Q+A_HB and tested with P+Q+A_HB


## Limitations & Future work

1) Not the only way to incorporate or generate affirmative interpretations. For example, one might be able to use LLMs such as GPT-4 or Llama to generate affirmative interpretations. 比如直接用LLM来生成这种肯定性解释呢

2) Another interesting direction is to investigate the effect of affirmative interpretations on other NLU tasks, such as sentiment analysis or text classification. 尝试其他任务

3) To investigate the effect of affirmative interpretations on other languages, especially those with different word order or negation structures. 尝试英语之外的语言

**None of the corpora that we work with include information about the scope and focus of negation.**  
我们使用的所有语料库都**没有包含关于否定的“范围（scope）”和“焦点（focus）”的信息**。

- **scope of negation（否定范围）**：指的是一个否定词影响到句子的哪一部分。  
- **focus of negation（否定焦点）**：指的是**否定句中被特别强调的部分**，也就是对比的“焦点”。

    例如：
    > I didn’t eat the cake `[on the table]`.（否定影响的范围是“吃桌上的蛋糕”）

可能的意思是：
- **我没有吃**“桌子上的蛋糕”（可能吃的是冰箱里的蛋糕）  
     否定范围是：`[eat the cake on the table]`（整个动宾短语）

	  
    > I didn’t eat `[the cake]` on the table.（这里的 scope 更窄，只是否定蛋糕）

可能是：
- 我没吃**蛋糕**，但我吃了**桌上的面包**  
     否定范围是：`[the cake]`（否定的焦点是“蛋糕”）

英文中，有时光靠语法形式无法唯一判断 scope，**语调、重音或上下文**会影响听者如何理解否定影响到哪里。同一句话在不同语境下可以有不同的 scope。

所以，如果没有明确标注 scope 的语料，就很难训练模型准确理解否定句的完整含义。


> **Therefore, we do not have any insight into the relation between affirmative interpretations and the scope and focus of a negation.**  
> 因此，我们**无法分析“肯定性解释”与否定的“范围”和“焦点”之间的关系**。


这句话其实是说明他们的研究**有一个限制**：  
他们虽然能生成“肯定性解释”，但由于原始数据中**没有标注清楚每个否定词到底否定的是哪个成分（scope），或强调的是哪个词（focus）**，所以他们**不能进一步分析这种结构信息是否影响了改写的质量或方式**。

---

## Affirmative interpretations

肯定性解释。The definition of affirmative interpretation is a paraphrase (i.e., rewording that preserves meaning) not containing negation.

目标是让模型在面对否定表达时，也能理解其积极版本，从而提升鲁棒性和语义推理能力。

> **Paraphrases vs. Affirmative Interpretations**

- “paraphrase” 是保留原始语义的**同义改写**，但不一定去掉否定。
- “affirmative interpretation” 是指**既是 paraphrase 又不含否定词**的表达。**不带否定词**的同义改写

这种**不含否定词的改写版本**为“affirmative interpretation”（肯定性解释）。
举例说明：“I am not sad”（我不难过） → “I am just ok”（我还行）或 “I am happy”（我很开心）。


自动生成的 paraphrase 不一定都符合“affirmative interpretation”的定义：
1. 有些还保留了否定词；
2. 有些语义改变了，不能算真正的同义改写。

比如：
**I stayed home today** is **not** a true paraphrase of **I didn’t go shopping today**

- “I stayed home today”（我今天呆在家里）并不是 “I didn’t go shopping today”（我今天没去购物） 的严格同义改写（不是 paraphrase），因为它没有完全保留原句的语义；

- 但它也**不是矛盾**，因为在语境中，“呆在家”可以**暗示**“没有购物”，对某些问答任务仍然有用。
  比如问：“我今天去购物了吗？” 答“我在家”也可以被视为合理推断。

---

## 好词好句

affirmative 
	adj. 肯定的；同意的 n. 肯定；同意
interpretation
	n. c. 解释；表演；演绎；理解
Crucially
	adv. 至关重要地，关键地
cloze
	adj. 完形的，填充测验法的 填充测验法 （测验阅读理解能力的方法）
	cloze questions
violate 
	v. 违反；侵犯，打搅；亵渎
paraphrase
	n. 释义，意译；演释曲 vt. 改述 vi. 改述；意译
plausible
	adj. 貌似真实的；貌似有理的；花言巧语的；有眉有眼
contradict
	v. 反驳，否认…的真实性；与…发生矛盾，与…抵触
couple 
	n. 两个；夫妻；一对儿；几个 v. 加上；结合
	Unlike these works, we couple original inputs containing negation with affirmative interpretations.
extrinsic
	adj. 非本质的；外在的；外来的；外部的
	they do not provide extrinsic evaluations.
complement
	n. 补足物；补语；足额，编制名额 v. 补充；衬托
	complement inputs that contain negation with a paraphrase that does not contain negation.