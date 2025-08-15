---
date: 2025-06-09
tags:
  - Negation
  - LLM
---
# Main Content

==Explore LLM's ability to perform **CoT-style reasoning robustly with a focus on negation**. ==

The authors designed a series of controlled experiments, specifically **syllogism** reasoning tasks, to evaluate the reasoning stability of 31 mainstream LLMs when they contain negation words.

Given a chain of the reasoning process, we expect the LLM to derive a logically valid conclusion even when the problem involves lexical negation.

The results of multi-difficulty controlled experiments revealed that LLMs with CoT-style prompting struggled to address negation

检验LLM在CoT推理下，对negation的推理能力的强健性。

利用的是三段论的方式来设计任务，并进行了4个控制变量实验。一共有3种任务，sports task, occupation tasks, weight transition task.

这些三段论的任务拓展为了4种不同的level的控制变量实验，BASE, FIC, FICNEG, FICNEG-O

**这篇论文确实创建了一个新 benchmark**，具有如下特点：

数据来源与构造：
- **SPORTS Task** 基于 BIG-Bench 数据，采样 1000 个例子；
- **OCCUPATION、WEIGHT TRANSITION** 是手动合成的；
- **四种受控设置**（每个任务都适用）

他们做的是 **zero-shot 或 few-shot 的推理评估**，没有进行模型训练或微调。
模型处理方式：
- 使用 OpenAI API（GPT-3.5, GPT-4）
- 使用 Huggingface 运行其他模型（LLaMA, OPT, BLOOM 等）
- 测试每个模型在不同设置下的 yes/no 判断
- 模型输出后，只根据其“回答 yes 或 no”的概率来判断准确率


**Models Tested**
- GPT-3 (text-davinci-003), PaLM, FLAN-T5, LLaMA, GPT-J.
- Settings: zero-shot and few-shot with CoT prompting.


## Background

- Language models (LMs) have shown impressive performance on many logical reasoning tasks when prompted with step-by-step (chain-of-thought, CoT) explanations.
- However, lexical negation (e.g., “not,” “never,” “no one”) remains a persistent challenge.
- Traditional evaluation often fails to disentangle **reasoning quality** from **answer correctness**, especially when negation is involved.


## Methods

The authors propose a **three-part framework** to assess LM reasoning under negation:
#### Step 1: Syllogism Generation

- Syllogisms generated using predefined logical templates (e.g., some/no/all A are B).
- Four types of lexical negation introduced: **no, not, never, nobody**.

#### Step 2: CoT Prompting & Response Collection

- Models are prompted with CoT-style questions (e.g., “Let’s think step by step”).
- Example:
    > _Premise 1: Some dogs are pets. 
    > Premise 2: No pets are fish. 
    > Conclusion: No dogs are fish. 
    > Is this conclusion valid? Let’s think step by step..._

#### Step 3: Reasoning Step Analysis

- To evaluate the LLMs, the accuracy of each model’s binary answers was measured.

- to quantify the output bias, we also calculated a no-ratio, i.e., how many out of 1,000 instances the models answered no


## Additional Knowledge

### controlled experiment

==其实就是以前说的控制变量实验==

**受控实验（controlled experiment）** 是一种：
> **有意识地控制部分变量、只改变一个目标变量，以观察因果关系** 的实验设计方式。

换句话说：
- 你固定（控制）其他不相关的因素；
- **只改变你关心的那一个因素**，以便观察它对结果的影响。

在 NLP 或 LLM 实验中，controlled setting 的目的是：

> **剥离语言模型已有的知识、偏好或模式，只评估其某项能力**（如推理、理解否定、处理结构变化等）。

比如这篇论文中，作者使用“**虚构的角色和活动**” (fictional entities) 来控制背景知识干扰：

| 设置       | 是否控制背景知识                              | 是否含否定    | 目标                 |
| -------- | ------------------------------------- | -------- | ------------------ |
| BASE     | 使用真实人物，如 Messi                        | 否        | 检测 CoT 能力          |
| FIC      | 替换为虚构人物、虚构活动                          | 否        | 消除事实性影响            |
| FICNEG   | 在 FIC 基础上引入否定词（implausible）           | 是        | 单独测试对“否定词”的理解能力    |
| FICNEG-O | 示例为肯定（plausible），目标问题为否定（implausible） | （仅目标含否定） | 检测泛化能力与 prompt 依赖性 |

这就是非常典型的 **controlled settings**，通过“只改变是否有否定词”来**考察模型对 lexical negation 的鲁棒性**。


### syllogism

The base format of the syllogism is as follows:
Premise 1: PERSON is a SPORT player
Premise 2: ACTION happens in the SPORT
Conclusion: PERSON does ACTION

- 前提1：A 是某种角色（如球员）
- 前提2：某个动作出现在这种角色对应的活动中
- 结论：A 做了这个动作
        
- 模型需判断“这个结论是否可信（plausible）”或“是否不可信（implausible）”




## good words

probe 
	adj. 探索的；寻根究底的；寻求真理的；（问题）深入锐利的 v. 探索，用探针（或探测器等）探查，探测；盘问；（用试探性袭击等）侦察（敌情）；用尖物刺穿（物件）
	from a probing perspective 从探索的角度
Building on this 
	在此基础上
syllogism /ˈsiləˌjiz(ə)m/ 
	n. 三段论
premise /ˈpreməs/ 
	n. 前提
derive /dəˈrīv/  
	v. （使）起源于，来自；获得
elucidate /əˈlo͞osəˌdāt/  
	vt. 阐明，解释
	Section 2 elaborates on the controlled task settings to elucidate the abilities of the models.
elaborate  
	vi. 详尽说明；变得复杂 vt. 详细制定；详尽阐述；加工；尽心竭力地做 adj. 复杂的；详尽的；精心制作的