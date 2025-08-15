---
date: 2025-07-08
tags:
  - notes
---

# Paraphrase affirmative sentences with negation

本质就是利用否定的形式表示肯定的含义。例如双重否定表是肯定。

| **肯定句（S2）**                       | **含否定的同义改写（S3）**                                         |
| --------------------------------- | -------------------------------------------------------- |
| He spoke during the presentation. | He wasn’t silent during the presentation.                |
| She smiled at the joke.           | She didn’t keep a straight face when she heard the joke. |
| They accepted the offer.          | They didn’t turn down the offer.                         |
| I agree with your idea.           | I don’t disagree with your idea.                         |
| He attended the meeting.          | He didn’t skip the meeting.                              |
| She passed the exam.              | She didn’t fail the exam. _(⚠ 若允许 fail)_                 |
| We saw something interesting.     | We didn’t see nothing interesting. _(⚠ 风格非正式但语义对)_       |
| The room was clean.               | The room wasn’t dirty.                                   |
| She was happy with the results.   | She wasn’t unhappy with the results.                     |
| He’s familiar with this concept.  | He isn’t unfamiliar with this concept.                   |
| They finished the work early.     | They didn’t finish the work late.                        |
| I enjoy classical music.          | I don’t dislike classical music.                         |


给一个肯定句，用含有否定的形式改写。这个LLM做的还行，可以批量给出一些合适的句子。
Task: Given a sentence S1, which expresses the affirmative meaning. Paraphrase S1 using a form that includes a negation cue to obtain sentence S2. Here is an example:
S1: He's familiar with this concept.
S2: He isn't unfamiliar with this concept.

Paraphrase following sentence as required:
S1: He spoke during the presentation.
S2:



## 一、为什么这个任务特别容易出错、模型很难做对？

这个任务比一般的否定处理任务更难，主要原因有以下几点：

1. **缺乏明确的“否定锚点”**  
    如果输入句本身就含有否定词（如 _not_, _never_），模型可以直接做一些表面上的替换（删掉否定词、调整动词）。  
    但如果输入是肯定句，模型就需要“主动构造”一个带否定的表达，这需要**真正理解语义**，而不仅仅是改词顺序。
    
2. **训练数据中很少出现自然的双重否定表达**  
    日常语料中大量是简单的否定句，但像“不是沉默”这种双重否定式的肯定句非常少见，模型缺乏统计上习得这些结构的机会。
    
3. **极性、否定范围、语义保持等多重挑战**  
    模型需要：
    
    - 找对该否定哪个成分（动词？形容词？整句？）
        
    - 用一个否定去“抵消”另一个否定，得到原句意义
        
    - 其他成分不能改动  
        一旦否定词的作用范围处理不好，语义就可能翻转或变得模糊。
        
4. **语用倾向会干扰生成**  
    英语中更偏好直接用“肯定表达”来说“肯定事实”（例如：He spoke），而不是“他不是沉默”。  
    模型的语言流畅性倾向会让它回避这些罕见又拗口的双重否定结构。
    
5. **这个任务对评估容错率非常低**  
    只要出现一个多余的否定词，或者用了“refuse”“fail”这类自带否定语义的词，整个输出就会被判错。  
    这对模型来说是个非常严苛的任务。
    

**总的来说**：这个任务其实是对模型“是否真正理解否定”的一种更深层次的测试，**比从否定句生成肯定句的任务要更具挑战性**。

---

## ✅ 二、你给出的现象的地道英文表达的中文翻译

大多数现有研究主要集中在以下两类任务上：

1. **极性反转**：输入是一个带有否定词的句子，然后将其改写为一个肯定句；
    
2. **同义改写**：从一个否定句出发，对它进行同义改写（可以保留或移除否定词）。
    

换句话说，**这些任务的输入几乎总是一个否定句**。

然而，**几乎没有研究反过来处理这个过程**：  
以一个**肯定句作为输入**，要求模型（a）反转其极性，或者（b）对其进行同义改写，其中的表达可以包含否定，也可以不包含。

尤其是当任务要求模型**用含有否定词的形式去同义改写一个肯定句**时，模型表现得非常吃力、容易出错。