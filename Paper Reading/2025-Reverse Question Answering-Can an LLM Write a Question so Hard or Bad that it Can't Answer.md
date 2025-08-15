

# Main

Reverse question answering (RQA): for an input answer, give a question with that answer.

Versus QA, LLMs are much less accurate in RQA for numerical answers, but slightly more accurate in RQA for textual answers

QA and RQA are often tested separately, but we test them jointly.

 chaining RQA and QA forms a consistency check for LLM reasoning.
QA(RQA(a)) ≈ a

RQA errors correlate with question difficulty
RQA 的错误（模型在生成问题时失败）与问题的难度正相关，也就是说——问题越难，模型越容易在 RQA 阶段出错。

inversely correlate with answer frequencies in the Dolma corpus
这些 RQA 错误与 Dolma语料库（一个大型训练语料库）中答案的出现频率呈反比关系，也就是——答案在预训练语料中越罕见，模型越容易在 RQA 中出错。

Four domain inputs based on answer type: number, number + text, easy fact, hard fact

Evaluation metric: 
DSPY-optimized GPT-4o for easy/hard entities.
a rule-based method for numerical entities.

## Background

- Traditional question answering (QA) focuses on answering questions given a context, using deductive reasoning.
    
- Reverse question answering (RQA) flips this, asking the model to generate a question given an answer, requiring abductive reasoning.
    
- RQA is important for tasks like exam question generation and search query reformulation.

## Motivation

- Understanding the interplay between QA and RQA helps assess LLMs’ reasoning consistency.
    
- The authors aim to analyze how well LLMs can generate valid questions given answers (RQA) and then answer their own questions (QA).
    
- They investigate whether RQA failures are due to knowledge gaps or reasoning errors.
    

## Method

- Collected 3,443 trivia QA pairs across four answer types: Number, Number+Text, Easy Fact, and Hard Fact.
    
- Evaluated 16 LLMs (e.g., GPT-4, LLaMA-3, Claude, Mistral) on both QA and RQA tasks using zero-shot prompting.
    
- Designed a consistency check: QA(RQA(a)) ≈ a — can the LLM answer its own question?
    
- Analyzed factors influencing RQA errors: answer frequency in pretraining data (Dolma corpus) and question complexity.
    
- Conducted human annotation to validate model outputs.

## Contributions

- Systematically evaluated 16 LLMs on both QA and RQA tasks.
    
- Found LLMs often fail RQA, especially with numerical answers; these failures correlate with question difficulty and rare answers in pretraining data.
    
- Demonstrated that even when RQA fails, LLMs can often answer their own invalid questions, indicating reasoning inconsistencies.
    
- Suggested that controlling question complexity and using difficulty scores might improve RQA performance.

## Experiments

- Numerical RQA proved especially difficult; accuracy was much lower than in QA.
    
- In textual domains, RQA sometimes performed better than QA.
    
- Consistency checks showed models could often answer their own invalid questions, revealing logical inconsistencies.
    
- Analyzed question types in RQA failures: complex, multi-step questions caused more errors.
    
- Prompt engineering techniques (e.g., chain-of-thought) did not substantially improve numerical RQA performance.
    

## Limitations

发现LLMs are sensitive to prompt formats

- Focused on English trivia data; results may not generalize to other languages or domains.
    
- Limited to zero-shot prompting without few-shot or fine-tuning.
    
- Dataset may not cover all possible question styles or answer types.
    

## Summary

- The paper highlights that LLMs struggle with abductive reasoning in RQA, particularly with numerical answers.
    
- Many models exhibit logical inconsistencies: they can answer their own invalid questions.
    
- The authors provide a benchmark and insights for improving LLM consistency in QA and RQA, offering guidance for future work in model calibration and prompt design.

---


## Additional

> **RQA—inferring just one of many valid questions—is abductive (Abe, 1998), while QA—inferring an answer from question premises—is deductive.**

翻译过来就是：

- 在 RQA 任务中，模型需要从给定答案反推一个可能的问题（答案的“解释”），这是溯因推理，因为同一个答案可以有很多不同的问题。
    
- 而在 QA 任务中，模型需要从问题的前提推理到唯一的答案，这是演绎推理，因为问题本身包含了足够的信息，可以逻辑推演到答案。

|任务|过程|逻辑推理类型|
|---|---|---|
|**QA**|从问题推理到答案|**演绎推理**（从前提到结论）|
|**RQA**|从答案推理可能的问题|**溯因推理**（从结果到解释）|


- **演绎推理（deductive reasoning）**：从一般到特殊的推理，通常有一个清晰的前提，结论必然成立。比如：
    
    - 前提：所有人都是凡人。
    - 前提：苏格拉底是人。
    
    - 结论：苏格拉底是凡人。
    - 这里的结论是唯一的、必然的。
        
- **溯因推理（abductive reasoning）**：通常是从一个观察（或结果）去猜测最合理的解释，属于“倒推”。它不保证结论唯一正确。比如：
    
    - 观察：地上有水。
    
    - 可能的解释：刚下雨了，或者有人洒水了。
    - 你需要选择“最合理的解释”作为推测。


**Dolma:**

Dolma是一个非常大的预训练语料库（据论文介绍，有3万亿tokens），包含了模型训练时用到的文本数据。Dolma中的**答案频率**就是看“模型过去在训练中见过多少次这个答案”。

- 频率高：模型很熟悉，常见词、常见数字
- 频率低：模型不熟悉，罕见词、罕见数字