---
date: 2025-05-19
tags:
  - ML
---
## 什么叫“弱监督”？监督在哪里？

### 📘 基本定义：什么是弱监督（Weak Supervision）？

> **弱监督**是指：没有完整人工标注，但借助规则、预训练模型或外部工具**自动产生标签或训练数据**

---

在[[2024-Generating Uncontextualized and Contextualized Questions for Document-Level Event Argument Extraction]]中的“弱监督”来自哪里？

他们用：

- **GPT-4**
- 输入：
    - 文档（document）
    - 事件触发词（trigger）
    - 参数角色（role）
    - 事件参数的答案（argument）

让 GPT-4 生成问题问这个参数 → 得到 (question, answer) 对

所以监督信号就是：

- **答案是已知的 gold argument**（从 RAMS 数据集来的）
    
- 模型学到的任务是：从文档 + 答案生成问题
    

---

### ✅ 为什么是弱而不是强？

| 对比维度      | 强监督       | 弱监督（本文情况）            |
| --------- | --------- | -------------------- |
| 问题由人工写？   | ✅ 是       | ❌ 否（由 GPT-4 自动生成）    |
| 标签是否完全可信？ | ✅ 高质量人工标注 | ⚠️ 有噪声（不一定全对）        |
| 用于监督任务？   | ✅ 用于训练    | ✅ 但 supervision 来源弱化 |

---
