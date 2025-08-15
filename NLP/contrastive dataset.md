---
date: 2025-06-09
---
## 构造对比数据集检测模型一致性

**通过构造对比样本，可以用来检测模型的一致性（consistency）**，即模型在输入非常相似的情况下是否能保持预测的稳定性。

the contrastive nature of CONDAQA[[2022-CondaQA-A Contrastive Reading Comprehension Dataset for Reasoning about Negation]] enables us to measure the consistency of models—i.e., the extent to which models make correct predictions on closely-related inputs.


The dataset should feature contrastive groups: passages that are closely-related, but that may admit different answers to questions, in order to reduce models’ reliance on potential spurious cues in the data and to enable more robust evaluation
数据集应包含对比组：紧密相关的段落，但可能对问题给出不同的答案，以减少模型对数据中潜在虚假线索的依赖，并实现更稳健的评估

### 1️⃣ 什么是“the contrastive nature”？

- **CONDAQA** 是一个对比型数据集，里面的每一对问题/样本都非常相似，只是在否定或细微差别上有所变化。
    
- 举个例子：
    
    - “Which countries border Mexico?”（问题A）
        
    - “Which countries do not border Mexico?”（问题B）
        
- 这两个问题在语言结构和实体上非常接近，但意义刚好相反。
    

---

### 2️⃣ 什么是“consistency”？

- **Consistency**（一致性）在这里是指模型在处理相似（但略有不同）的问题时，能否根据问题本身的差异，做出逻辑一致的回答。
    
- 例如：
    
    - 对问题A模型回答了“Arizona, New Mexico, Texas, California”。
        
    - 对问题B模型应该回答除去这些州的其他州（即非接壤州），而不是依然把“Arizona”等州也算进去。
        

---

### 3️⃣ 为什么“对比性质”可以检测一致性？

- 因为“对比性质”可以让我们：
    
    - **控制变量**：问题A和问题B除了关键逻辑点（比如“border” vs. “not border”）之外，其他都一样。
        
    - 这样我们就可以判断模型是否真的理解了问题中的**否定信息**，或者只是把相似的输入模式记忆化（rote memorization）。
        
- 如果模型对两个问题都回答了“Arizona, New Mexico, Texas, California”，那就说明它**没有很好地分辨问题的差异**，一致性就差。
    

---

### 4️⃣ 直观解释：

👉 “对比样本”相当于一个“同一组问题中的小实验”，让我们把模型的推理暴露在非常相似的条件下。  
👉 如果模型在两个问题上答案逻辑一致，就说明它的推理稳定、可靠；如果前后矛盾，就说明它的一致性不好。