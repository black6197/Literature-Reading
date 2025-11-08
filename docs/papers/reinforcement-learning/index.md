# 强化学习论文笔记

这里整理了强化学习相关的论文阅读笔记，特别关注在足球分析和序列决策中的应用。

## 📖 已阅读

暂无

## 📝 核心概念

### Temporal Difference Learning
- **定义**: 通过自举（bootstrapping）学习价值函数
- **优势**: 不需要等到序列结束，可以在线学习
- **应用**: 评估球员动作的长期价值

### Credit Assignment Problem
- **问题**: 如何将最终结果归因到之前的动作
- **挑战**: 足球中进球稀疏，难以评估中间动作
- **解决思路**: 使用 TD Learning 传播价值

### Value Function
- **状态价值 V(s)**: 评估某个状态的好坏
- **动作价值 Q(s,a)**: 评估在某状态下采取某动作的价值
- **足球应用**: 评估球员动作对进球概率的贡献

## 🔗 相关资源

- [深度强化学习课程](http://rail.eecs.berkeley.edu/deeprlcourse/)
- [Spinning Up in Deep RL](https://spinningup.openai.com/)

## 📂 相关笔记

- [Transformer](../transformer/index.md) - 注意力机制可用于状态表示
- [足球分析](../soccer-analytics/index.md) - RL 在足球中的实际应用

---

*持续更新中...*
