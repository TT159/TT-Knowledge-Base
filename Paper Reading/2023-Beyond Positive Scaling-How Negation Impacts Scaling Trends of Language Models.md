---
date: 2025-06-12
tags:
  - Negation
  - LLM
---

# Main Content

大型语言模型（LLMs）通常被认为在规模扩大（参数、计算量、训练数据等）时表现提升，即所谓的 **positive scaling trend**。然而，研究者观察到，在包含 **否定(negation)** 成分的任务中，这种趋势可能会“失效”，甚至出现 **逆向（inverse）** 或  **U 形（先降后升）** 的 scaling 模式。

本文聚焦于这种现象，构建了一个新数据集 **NeQA**（Negation Question Answering），并在多个模型家族与提示（prompting）方法上测试其性能，揭示了否定任务中的 scaling 并非单调正向。

**Positive scaling**: performance improves as models are scaled up in terms of size, compute, or data.

Observation:  Introduce NeQA, a dataset consisting of questions with negation in which language models **do not** exhibit straightforward positive scaling. 

We show that this task can exhibit inverse scaling, U-shaped scaling, or positive scaling, and the three scaling trends shift in this order as we use more powerful prompting methods or model families.  

==Specifically, as we use more powerful prompting methods or model families, NeQA exhibits a gradation from inverse scaling to U-shape to positive scaling.==

 We **hypothesize** that solving NeQA depends on two subtasks: **question answering (task 1)** and **negation understanding (task 2)**.

task 1 has **linearly positive scaling**, while task 2 has **sigmoid-shaped scaling** with an emergent transition point, where the transition point is influenced by the prompt method and model family.
Combining these two scaling trends yields the final scaling trend observed in NeQA.

NeQA, a new task of answering multiple-choice questions containing negation words, constructed by transforming questions from OBQA (Mihaylov et al., 2018) and NegatedLAMA (Kassner and Schütze, 2020).

 We conduct experiments on this task using 4 language model families and 3 prompting methods


## Dataset: NeQA

源于 **NegatedLAMA** 和 **OBQA**，构造了包含否定表达的 **多项选择题**，每题有两个选项，一个正确、一个错误。
涵盖 **6种不同的否定类型**：如动词否定、情态否定、连接词否定、前缀否定（如 un-）、否定提示词等。
总共 **1718道题**，数据来源包括 ConceptNet、GoogleRE、SQuAD、TREx、OBQA。

We believe that this dataset serves as a valuable **benchmark** for assessing the ability of language models to process negation.

**First source: NegatedLAMA:**
The NegatedLAMA dataset includes negated questions from four subsets: ConceptNet, GoogleRE, SQuAD, and TREx.

Each example of the dataset consists of a negated question and two answer choices, one correct and one incorrect. 
An example of NeQA looks like: 
(question “Child does not want?”, correct choice “marriage”, incorrect choice “love”)

Each question in NegatedLAMA is associated with **a negated question, an answer, and a misprimed question.** 

We turn it into a multiple choice question by setting the negated question as the question, and setting the wrong answer in the misprimed question in conjunction with the correct answer as the two choices.

**Second source: OBQA**
To be able to analyze the impact of different negation types, we also created additional data by applying **diverse rules** to transform questions in OBQA (Mihaylov et al., 2018) into negated ones.

Defined 6 types of negation transformations.
 action verb negation (e.g., “cause” → “does not/- doesn’t cause”), linking verb negation (e.g., “is” “is not/isn’t”), modal verb negation (e.g., “can” → “can not/can’t”) .....

**non-negated questions**
Out of the 1718 questions, we define a set of 944 questions from ConceptNet, TREx, and a subset of OBQA that exhibit clear positive scaling on the corresponding original (non-negated) questions.


## Experiments

4 LLM families:
- GPT-3
- GPT-3 Text Series
- Cohere
- Jurassic


3 different prompting methods:
- Zero-shot
- Zero-shot with hint (Kojima et al., 2022)
- Few-shot with Chain-of-Thought (CoT) (Weiet al., 2022c)
![[NeQA different_prompts.png]]

Metric: accuracy of the model predictions


## Results

Evaluation reveals that the scaling trends of language models on the NeQA task vary depending on the **prompting method** and **model family** used.

The scaling trends of **all** language model families can be altered by different prompts.

 The overarching scaling trend of language models for NeQA is U-shaped, and if the model is weak (i.e., weaker prompt or model family), the left part of “U”/inverse slope is observed; if the model is strong, the right part of “U”/positive slope is observed.


作者将 NeQA 拆分为两个子任务:

| 子任务编号  | 描述                    | Scaling 模式      |
| ------ | --------------------- | --------------- |
| Task 1 | **常规问答**：回答非否定版本的问题   | 线性正向 scaling    |
| Task 2 | **否定理解**：识别并理解否定表达的含义 | sigmoid 曲线，有突变点 |

NeQA 实际上是这两个任务的组合：你必须先能回答原问题（Task 1），再理解否定反转含义（Task 2），最后才能答对否定版本的问题。

### 组合后为什么是 U 形 scaling？

- 在模型小或提示弱时：
    - Task 1 正确，但 Task 2 不激活 → 模型无法区分原问题和否定问题 → 输出一样答案 → 得分下降（因为答案标签被反转了）
    - → **表现出 inverse scaling**

- 在模型大或提示强时：
    - Task 2 被“激活” → 模型开始理解否定 → 能正确反转答案 → **得分回升**
    - → **表现出正向 scaling**

- 中间阶段：
    - Task 1 越来越强，但 Task 2 尚未激活 → 输出仍是错误的（甚至更“自信地错”）
    - → **形成 U 型的中段谷底**


> 作者称这种“否定理解”能力的突变点为 **“emergent transition point”**，其位置取决于提示强度和模型家族。

## Task Decomposition Analysis Experiments

We decomposed the NeQA task into two subtasks: task 1 is to answer the original non-negated questions, and task 2 is to “understand negation”.

2 LLM families:
- GPT-3
- GPT-3 Text Series

2 prompting methods:
- Zero-shot
- Zero-shot with hint

![[NeQA Task Decomposition.png]]

Our experiments showed that task 1 scales mostly linearly in a positive direction, whereas task 2 scales like a sigmoid shape with an emergent transition point, analogous to the Grokking curve (Power et al., 2022).

Before this transition point, models do not “understand negation”.

Interestingly, we found that the transition point can be moved earlier with stronger prompting methods or model families.


## Fine-tuning Simulation

**Fine-tuning Simulation 是有做的**，但它不是直接 fine-tune 在 NeQA 上，而是通过 **改造 SST-2 情感分类数据集**，来模拟与 NeQA 类似的任务结构，从而验证 negation 理解的 scaling 现象是否具有通用性。


we conduct experiments using synthetic data and small-size language models to simulate and analyze the language model learning process.

Dataset: SST-2 (synthetic)
Model: GPT-2
Variable: 
- different sizes of GPT-2
- numbers of epochs
- negation ratio

The author uses SST-2 (sentiment classification dataset) and artificially adds "negation" to simulate negation. -> synthetic data

As the number of training rounds or the proportion of negative samples increases, the scaling of Task 2 changes from inverse to U and then to positive.

这个实验虽然不直接使用 NeQA 数据，但它：

- **模拟了与 NeQA 相似的结构**（原始问题 + 翻转理解）
    
- 佐证了 “task decomposition + sigmoid scaling of negation” 的解释是普适的；
    
- 显示了训练数据中的 **negation 样本比例** 和 **训练量** 也是 scaling 的决定因素，不止模型规模或提示强度。

Scaling is inverse when the negation ratio is low;
After increasing training or negation data, the curve begins to become U-shaped, and even further turns positive.


Fine-tuning Simulation 是论文的一个关键补充分析，它通过 SST-2 构造的合成任务，验证了 NeQA 的 scaling trends 可以通过训练过程中的 negation 处理能力演化来解释。







## Additional knowledge

### NegatedLAMA dataset

The NegatedLAMA dataset includes negated questions from four subsets: ConceptNet, GoogleRE, SQuAD, and TREx. Each subset comprises multiple files that represent different question distributions, such as questions about different entity relations. **Each question** is associated with **a negated question, an answer, and a misprimed question** (i.e., a wrong answer followed by the question). For instance, when “Child wants?” is the original question, “Child does not want?” can be its associated negated question, “love” can be the answer, and “Marriage? Child wants?” can be its misprimed question.

每个问题都与一个否定问题、一个答案以及一个误导性问题。