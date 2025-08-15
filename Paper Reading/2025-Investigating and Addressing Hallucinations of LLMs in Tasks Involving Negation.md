---
date: 2025-06-12
tags:
  - Negation
  - LLM
---
# Main content

==summary: Four negation-based evaluation tasks are designed to systematically evaluate whether existing SOTA LLMs produce hallucinations in negation scenarios.==
设计 4 种基于否定的评估任务，来系统评估现有 SOTA LLMs 在否定场景下是否产生幻觉。

Although large language models (LLMs) have excellent performance in many NLP tasks, they have the problem of "hallucination", that is, generating information that seems grammatically correct but is factually incorrect. 

Previous studies have focused on the hallucination problem of tasks such as summarization, question answering, and translation, but the **hallucination** problem involving "**negative**" information has hardly been systematically studied.

The "negative" language structure is very important in human language. It is crucial for logical reasoning, information understanding, and reversing statements. However, since negative sentences may be covered by low frequency in the training corpus, LLMs have poor processing ability for them, so the paper focuses on this issue.

==Goal:  focus on studying the impact of negation in LLM hallucinations.==

It constructs a **custom suite of evaluation prompts**, specifically tailored to test hallucination behavior under negation.
Four types of tasks are constructed to test whether LLM produces hallucinations due to negation structures:
- False Premise Completion (FPC)
- Constrained Fact Generation (CFG)
- Multiple-Choice Question Answering (MCQA)
- Fact Generation (FG)

Further study numerous strategies to mitigate these hallucinations.

contributions:
- For the first time, we introduce negation generation tasks as a new dimension to evaluate LLM hallucinations.
- We carefully design four realistic negation task scenarios, which is different from the previous focus on classification tasks (such as NLI).
- We systematically evaluate multiple mitigation strategies and point out that some strategies (such as providing external knowledge) actually aggravate hallucinations, which is a counterintuitive finding.

## Background

investigating the impact of ‘negation’ in LLM hallucinations has remained underexplored.

We humans arguably use affirmative expressions (without negation) more often than expressions with negation (Hossain et al., 2020; Ettinger, 2020).  This implies that texts containing negation could be underrepresented in the training/tuning data

It is also important to study generative tasks with state-of-the-art LLMs.



## Four Evaluation Tasks

![[Negation in LLM hallucinations.png]]

**False Premise Completion (FPC)**  
The input is a premise sentence that contains negation and is semantically incorrect (e.g., "Water does not conduct electricity because..."). The model is expected to identify the error and correct it.  
▶ **Evaluation**: Whether the model correctly identifies and rectifies the false premise.

---

**Constrained Fact Generation (CFG)**  
Given a set of keywords that includes the word "not", the model is required to generate a factually accurate sentence containing negation.  
▶ **Example**: Given keywords like “Senegal, not, tallest statue”, a correct output would be “The statue in Senegal is not the tallest in the world.”

---

**Multiple Choice QA (MCQA)**  
This is a multiple-choice question format where the question includes negation, such as "Which countries have **not** hosted the Olympic Games?"  
▶ **Focus**: Whether the model can correctly identify the negative (excluded) options.

---

**Fact Generation (FG)**  
The task requires the model to generate factual statements that include negation, such as “Tagore was **not** a politician.”  
▶ **Comparison**: Evaluate the model's factual accuracy with negation versus without negation.

---

Models:
Use three open source SOTA models: LLaMA-2-chat, Vicuna-v1.5, Orca-2 (13B)

Findings:
- In tasks with negation, the model has a significantly higher hallucination rate than when there is no negation.

- Orca-2 performs best on MCQA because it uses interpretable fine-tuning.

- All models perform particularly poorly on CFG and FG, indicating that the negation structure poses a great challenge to the model.

## Mitigation of Hallucinations

1) **Cautionary Instruction (Inst)**  
    Add a warning to the prompt — e.g., _“Note that the prompt can be misleading as well.”_  
    This is combined with the standard task instruction (“Complete the prompt with factually correct information”).  
    ➤ Result: Modest improvement in hallucination reduction.

2) **Demonstrative Exemplars (Exemp)**  
    Provide **input-output examples** of both false and correct premise prompts in the context to guide the model.  
    Examples include:
	- Input with a false premise + correct output;
	- Input with a correct premise + correct output (to prevent the model from being biased towards negative sentences).
    Three combinations were tested (see Appendix A.1), and average results were reported. Exemplars are non-overlapping with test data to avoid leakage.   Each exemplar is in the form of (input, output).
    ➤ Purpose: Help models learn from examples rather than rely solely on instructions.

3) **Self-Refinement (Self-Refine)**  
    After the model generates an initial response, ask it to _“rewrite the answer by correcting factual inaccuracies.”_  
    ➤ Uses the model’s own knowledge to attempt self-correction.
    **Advantages:** Does not introduce external knowledge and relies on the model's own understanding.

4) **Knowledge Augmentation (Know)**  
    provide knowledge relevant to the prompt as additional contextual information to the LLM during generation.
    Inject external, relevant factual knowledge into the prompt.  
    ➤ Use Bing Search API to retrieve web content based on the prompt.  
    **Note**: This strategy may actually increase the illusion in some scenarios because it may reinforce the original false premise.



## Limitations
- **Limited task scope**: The study only covers four types of negation-related tasks, and future work could explore additional real-world scenarios.

- **Monolingual evaluation**: The evaluation is restricted to English-language models and does not consider multilingual capabilities.

- **Limited model selection**: While the study includes popular open-source models, it does not evaluate closed-source models such as GPT-4 or Claude, which could be incorporated in future work.

- **High cost of human evaluation**: A large portion of the results relies on manual annotation, making the evaluation process labor-intensive.






## Additional knowledge


### 所做事务

- **没有创建通用 benchmark 数据集**
- **没有训练或微调模型**（所有模型都是开源版本，如 LLaMA-2-chat、Vicuna-v1.5、Orca-2 的 13B 模型）
-  **没有提出新的 LLM 架构**

而是设计 **4 种基于否定的评估任务**，来**系统评估现有 SOTA LLMs 在否定场景下是否产生幻觉**。
设计四个任务类型（全为“生成类任务”而非分类类）：
- False Premise Completion（否定前提补全）
- Constrained Fact Generation（否定关键词组生成事实）
- Multiple Choice QA（否定式多选问答）
- Fact Generation（带否定的事实生成）


## Good words

attribute
	v. 把...归因于；认为是…所为
	we attribute this to the nature of the prompts 我们将其归因于prompt的本身
coerce /kōˈərs/
	v.  persuade (an unwilling person) to do something by using force or threats. 胁迫，迫使，要挟
	providing additional contextual knowledge coerces the model to respond to a prompt even when the prompt is misleading