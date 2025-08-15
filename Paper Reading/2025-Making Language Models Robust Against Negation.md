---
date: 2025-05-20
tags:
  - NLP
  - Negation
---
# Main

本文专注于Negation。Previous studies have shown that they struggle with negation in many natural language understanding tasks. In this work, we propose a **self-supervised method** to make language models more robust against negation.

**这篇文章不是构建benchmark，而是直接在9个已有的benchmarks上进行评估**。其构建了一些训练数据，即add negation & remove negation。然后再用这些数据去微调了大模型，然后去benchmark上评估。

==The goal of the paper is to improve LM robustness to negation==

场景/任务 tasks:
Next Sentence Polarity Prediction (NSPP)
given a sentence, the model predicts whether the next sentence will contain negation.

Next Sentence Prediction (NSP) 的变体 variation
In our version, we reverse the polarity of the second sentence to form negative samples rather than select random sentences from the document.
Define reversing polarity as adding (or removing) negation to a sentence that contains (or does not contain) negation.

Experiment：
作者发现/证明，在他们这些任务上pre-train以后的BERT & RoBERTa 比已有的那些方法要好，在9个negation-related benchmarks上验证。

同时，发现预训练后在CondaQA (一个需要在negation上进行推理的QA语料库)的性能也提高了。

Contribution:
1) 为pre-training LMs for negation，引入/创建了两种新颖的self-supervised tasks
2) 创建了为这些tasks使用的大型数据集，6.4M samples
3) 证明了进一步pre-training BERT和RoBERTa independently on these tasks improves performance on CondaQA 和其他8个negation-related benchmarks. 
   但是， 也发现，joint pre-training on both tasks, doesn't always improve performance. 

使用English Wikipedia corpus，使用spaCy generate the dependency tree for each sentence. 是自动化的生成数据的，并不是手工人工整数据集。 

Limitation
1）转化极性的否定仅包含not, n't, never 然而，CondaQA拥有超过200种独特的否定线索。
2）仅experiment with models pre-trained on 500K and 1M instances. 未来可以考虑在整个corpus上训练
3）all the corpora we work with are in English. We acknowledge that negation may be expressed differently in other languages and this work may not generalize to other languages. 考虑其他语言


==**总结**==：
那个转化极性部分挺繁琐的其实，涉及很多语言语法上的，比如有助动词如何添加negation,如何removing negation，时态，yet, at all, but等等，涉及挺多语法的东西，因为肯定不想手动逐个处理，所以最好就是通过一些语法规则，来自动化的构建数据。

---

## Background

Language models (LMs) have achieved outstanding performance across many natural language understanding tasks, but they often struggle with negation.

==Observation==: BERT (Devlin et al., 2019) often predicts the same token when negation is added to a sentence.

==假设 Assumption==: hypothesize that this behavior is due to the lack of negation modeling in pre-training.
Specifically, the model has not been exposed to instances where the addition (or removal) of negation influences meaning and coherence within a discourse.

==Method==:  Propose a self-supervised method to make LMs more robust against negation. Our approach is to further **pre-train** LMs on two tasks that involve negation.


## Method

- The authors propose two self-supervised pre-training tasks:
    
    1. **Next Sentence Polarity Prediction (NSPP)**: Given a sentence S1, predict whether the following sentence S2 contains negation.
        
    2. **Next Sentence Prediction with Reversed Polarity (NSP)**: Adapt the classic NSP task by generating negative examples by reversing the polarity of S2 (adding or removing negation).
        
- They develop automatic rules to add/remove negation cues like “not,” “n't,” and “never,” generating a large-scale training dataset (6.4 million sentence pairs).
    
- They fine-tune BERT and RoBERTa using these tasks.

## Two Tasks: NSPP & NSP Variation

### Next Sentence Polarity Prediction (NSPP) task

NSPP as the task of predicting the polarity of the next sentence given the current sentence. 
Given a pair of consecutive sentences, (S1, S2), the input to the model is only S1, and the output is a binary label indicating whether S2 includes any negation cues or not.

例子：
following pair of sentences:

S1: The weather report showed sunny skies.
S2: But it didn’t stay that way.

==Given only S1, the model should predict that the following sentence includes negation cues.==

一个单句输入、二分类输出的任务。输入S1，预测S2是否包含negation

---

### Next Sentence Prediction (NSP) variation task

**传统的BERT里的NSP任务**(参考文章: [[BART & BERT & RoBERTa]])是：判断两个句子是否是连续句。
NSP task is to predict whether two sentences are consecutive. 判断两个句子是否连续。利用的是Wikipedia的数据，use consecutive sentences as positive examples; chose a random sentence from the same article to replace the second sentence and create a negative example.

#### 与原始 BERT NSP 的主要区别

| 项目     | 原始 NSP     | 改进版 NSP（本文）        |
| ------ | ---------- | ------------------ |
| 负例构造   | ==用随机句子==  | 用**极性反转的下一句**      |
| 否定信息   | 不涉及        | 明确引入否定与其反义         |
| 训练目标   | 学习句子连续性    | 学习**否定影响语义的能力**    |
| 输入语义差异 | 差异不大，可能可忽略 | 差异明确，对模型提出更高语义推理挑战 |

提出了一个变体来提升negation的理解能力。原始 NSP 任务并不涉及**句义的变更**，模型可能学不到处理否定所需的语义推理能力。作者提出了一种**改进版 NSP**，用“极性反转”（polarity reversal）来构造更具挑战性的负例。

==原始NSP任务里，负例是随机换一个句子即可，使得其不是原句的next sentence. 而本文的变体，在构造样本时进行了更精巧的设计，即使用极性反转来替代原始NSP中的方式。==

##### 改进版 NSP 不改变模型结构，也不改变损失函数
- **模型结构：** 和原始 BERT 的 NSP 任务完全一样。输入两个句子，用 `[CLS]` 表示句对的整体表示，然后接一个分类头（softmax 或 linear + sigmoid）。
- **训练目标：** 二分类，判断 S2 是否为 S1 的真实后续句（Label ∈ {0, 1}）。
- **损失函数：** 交叉熵损失（CrossEntropy Loss），不变。

##### 唯一的改进在于：负例的构造方式

| 原始 NSP（BERT）        | 改进版 NSP（本文）             |
| ------------------- | ----------------------- |
| 负例是随机选的 S2′（来自其他段落） | 负例是从真实 S2 进行极性反转得到的 S2′ |
| 通常语义差异较大，模型容易学到     | 语义差异微妙（仅否定变化），对模型更具挑战性  |
| 不涉及否定建模             | 强化对“否定 vs 肯定”语义变化的理解    |

> 因此，**无需修改模型，仅需重新构造训练数据**（正例依旧是真实句对，负例换成了极性反转对），即可开展更有针对性的预训练。


所以你可以这样理解：

> ==**改进版 NSP = 原始 NSP + 更高质量的负例（极性反转）**  ==
> 
> → 不改网络结构，不改训练目标，只在数据构造上做“聪明的改动”。

---

#### 举个例子

原始对：
- S1: _“He thought it would be sunny.”_
- S2: _“It was not sunny that day.”_

极性反转后构建负例：
- S2′: _“It was sunny that day.”_

**输入：**
一个句子对（S1, S2'）
- `S1` 是原始句子；
- `S2'` 是通过**对真实下一句 `S2` 进行极性反转**（加或去否定）得到的。

**输出:**
一个**二分类任务**：
- **Label = 1**：如果 S2 是 S1 的真实后续句；
- **Label = 0**：如果 S2′ 是一个反转极性的“假”后续句。

换句话说:
- `(S1, S2)` → 正例 → Label = 1
- `(S1, S2′)` → 负例 → Label = 0

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

---

#### 极性反转（Polarity Reversal）细节

**规则驱动（rule-based）修改句子**，只处理含一个否定词的简单句，例如：

- 添加否定：
    - “I went to the store.” → “I didn’t go to the store.”

- 移除否定：
    - “He doesn’t like pizza.” → “He likes pizza.”

仅限于包含 `not`, `n’t`, `never` 等否定词，主语、时态、语态也会做语法调整（如“did not go”→“went”）。

> 作者验证这些规则在 96% 的情况下能正确构造极性反转句


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

---
## Contributions

- Introduced the novel NSPP task to enhance negation modeling.
    
- Proposed a variation of the NSP task using polarity reversal instead of random sentence sampling.
    
- Created a large-scale, automatically constructed dataset for these tasks.
    
- Demonstrated significant improvements (up to 9.1%) on the CondaQA benchmark and eight additional negation-related benchmarks.

## Experiments

- Evaluated models on CondaQA (a QA dataset requiring negation reasoning) and several NLI and NLU datasets (RTE, SNLI, MNLI, QNLI, WiC, WSC).
    
- Key findings:
    
    - All pre-trained models showed consistent improvements in accuracy and group consistency on CondaQA, especially RoBERTa models.
        
    - On NLI tasks with negation subsets, further pre-training improved performance compared to baseline models.
        
    - Joint training on NSPP and NSP improved RoBERTa-large but did not consistently outperform single-task training for other models.
        
    - Ablation studies showed that reversing sentence polarity was more effective than simply adding or removing negation.


## Limitations

- Only used English Wikipedia as the training corpus; multilingual generalization remains untested.
    
- The polarity reversal rules cover only basic negation forms (“not,” “n't,” “never”), leaving out more complex cases.
    
- Experiments **only focused on BERT and RoBERTa**, without testing larger or more recent LMs.
    
- Data size capped at 1 million sentence pairs, with no exploration of even larger datasets.
    
- Evaluation was limited to English datasets, so results may not generalize to other languages.

---
## Summary

- The paper introduces two self-supervised tasks (NSPP and polarity-reversed NSP) that significantly improve LM performance on negation tasks—achieving up to 9.1% improvement on CondaQA.
    
- NSP consistently outperformed NSPP; joint training helped RoBERTa-large but not others.
    
- The approach is simple and inference-efficient, making it practical for real-world deployment.
    
- Future work could explore multilingual adaptation, richer negation phenomena, and larger pre-trained models.

---

## Additional Knowledge

### 本文说的further pre-train的意思

本文说的是pre-train LMs 而不是说的fine tune。但是却又说从base version of BERT 和 large version of RoBERTa开始，并说这两者预训练过了。所以我一直有点困惑，它为什么不是微调，而被称为further pre-train。

>  We use base and large versions of BERT (Devlin et al., 2019) and RoBERTa (Liu et al., 2019) as our baseline models. We further pre-train the models on NSPP and NSP tasks individually and jointly. Since BERT and RoBERTa are already pre-trained on Wikipedia using masked language modeling, further masked language modeling during our pretraining is redundant. We use Transformers (Wolf et al., 2020) and PyTorch (Paszke et al., 2019) libraries.

首先要确定的是：
他们是从**已经预训练好**的 BERT 和 RoBERTa 模型开始的，也就是 Huggingface 提供的开源权重，比如：
- `"bert-base-uncased"`
- `"roberta-large"`

> **“We further pre-train the models on NSPP and NSP tasks individually and jointly.”**

关键词是 **“further pre-train”**：  
这不是微调（fine-tuning），而是在**原本的预训练基础上再次进行预训练**，但目标不是 MLM（Masked Language Modeling），而是他们自己提出的 NSPP 和极性反转版 NSP 任务。

==BERT的pre-train其实就是两个任务，一个是MLM, 一个是NSP。RoBERTa的预训练任务中还剔除了NSP任务。本文无需更改MLM能力，更多的是拓展NSP的能力，最重要的是，其并没有对具体的下游任务（如，QA, NER, NLI等）进行训练，所以不是微调，依旧处在对模型最基础的能力训练的步骤，所以被称为了further pre-train==

- 原版 BERT/RoBERTa 预训练任务是 MLM（掩码语言建模）；
- 作者认为再重复做 MLM 已经没用；
- 所以他们只用 NSPP 和 NSP 来进一步训练模型。

|对比|微调（Fine-tuning）|继续预训练（Further Pre-training）|
|---|---|---|
|数据|标注数据（QA、分类）|无标注的自监督数据|
|目标|优化下游任务表现|增强通用语言理解能力|
|示例|SQuAD、MNLI|NSPP、NSP（本文）|
|作用|适配特定任务|加强模型能力（如处理否定）|

作者没有从头训练 BERT，而是加载了 Huggingface 的模型，如 `bert-base-uncased`、`roberta-base`，然后再训练新的任务。

> **进行继续预训练（further pre-training）时，模型的**所有参数**都会被更新**，不像微调时通常只更新部分参数（如分类头）。

这是因为：
- **继续预训练**的目标是让模型在**编码句子时的表示能力发生改变**，这涉及整个 Transformer 编码器；
- 而 **微调**通常是对最后几层或加的任务头（classification head）进行调整，保持主干参数不动或只轻调。

|项目|微调 (Fine-tuning)|继续预训练 (Further Pre-training)|
|---|---|---|
|训练目的|适配某个下游任务|改善模型的语言理解能力|
|参数更新|通常只更新分类头 / 最后几层|**更新整个模型参数（从 embedding 到 encoder）**|
|输入数据|标注的任务数据（如问答）|自监督构造数据（如 NSPP, 改进NSP）|
|学习率|通常较高（如1e-5）|通常较低（如1e-6）|
|实现复杂度|简单|稍高，需要重新构造任务和训练流程|


🔁 **“进一步预训练” 的本质** 是：

- 保留了 BERT 在原始 MLM 任务上的预训练知识；
    
- 用新任务（NSP/NSPP）继续让模型学习“否定语义”的建模能力；
    
- 不直接学具体下游任务（比如分类、问答） → 所以这不是微调。

#### 模拟流程图示
Huggingface 预训练模型（bert-base-uncased）
          ↓
继续预训练（本文提出的 NSP / NSPP）
          ↓
得到了“更理解否定”的 BERT
          ↓
下游任务评估（CondaQA / RTE / SNLI 等）


### how to get the data in this paper

We begin by extracting all sentences from Wikipedia containing negation that are not the first sentence of a section, ==ensuring that each has a preceding sentence (S1) to provide context for the next sentence (S2).==

We also extract affirmative sentences (i.e., without negation cues) along with their preceding sentences (S1).

所以，我可以理解为，都是先从Wikipedia文本中提取出后面的句子S2（with or without negation），然后去找其的先行句S1？这样以防找到S1而没有后续的句子？

**“We only work with sentences that:**  
• include `not`, `n’t`, or `never` as negation cues;  
• the negation cue **modifies the main verb**;  
• are not questions;  
• contain exactly one negation cue.”

---

### how to categorize negation cues

- **第一种分类（Figure 4）**：按**词性/语法角色**分类（noun, adverb, verb, ...）→ 更细致；
    
- **第二种分类（Figure 2）**：按**构词结构/表达形式**分类（single-word, affixal, mwe）→ 更通用（general）；

Based on CondaQA, we can categorize negation cues into three classes: affixal, single_word, mwe (multiple word expression).

Goal: evaluate different negation cues and try to figure out which is the best useful one for negation understanding.

### how to get data


I am thinking that can we use the fictional data rather than extracting from some real corpus？

If we use real data, apparently, we can be benefit from other established negation datasets.
Roughly thinking, we can extract the sentences that contains the key words from these corpus and then reverse the polarity.
example: 
Affixal negation cue : unable
search unable -> reverse to able


If we use fictional data, then we need to think about how to **generate** targeted data.

1) extract data from other fictional corpus. just like extract targeted sentences from Wikipedia, we can simply search the key word (negation cues) and extract the sentences, though they are fictional or contain fictional entities.
   Though I don't know whether there is this kind of corpus or not.

2) self-generate fictional data. we can find some way to generate some fictional data.
	a. designing some templates and using LLMs to generate?
		- “[SUBJ] is **{ADJ}** to [VERB].” → “He is **able** to walk.” / “He is **unable** to walk.”
		- “The results were **{ADJ}**.” → “The results were **known** / **unknown**.”


---

Since we will categorize negation cues into three classes. I think that it's not practical or efficient to evaluate every negation cue in each class. Instead, we can select the majority negation cues in each class, that is, we can simply pick the Top 10 negation cues in Affixal and Top 5 negation cues in single_word class in order to ensure the distribution/portion is still the same as the entire distribution.

we can train them jointly or separately.   

Thus, we will need a list of each kind of negation cue category.
example:
Affixal Top k: unknown, unlike, unsuccessful, unable, unusual, illegal....
single_word Top k: not, none, lack, never, except, rarely, cannot, nobody....
mwe Top k: with the exception of, no longer....	

NLI任务，RTE任务

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