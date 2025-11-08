# Transformer 论文笔记

Transformer 架构和注意力机制相关论文整理。

## 📖 已阅读

暂无

## 📝 核心概念

### Self-Attention Mechanism
- **定义**: 通过注意力权重聚合序列信息
- **优势**: 并行化计算，捕捉长距离依赖
- **足球应用**: 建模控球序列中的动作依赖关系

### Position Encoding
- **问题**: Transformer 本身不包含位置信息
- **解决**: 添加位置编码表示序列顺序
- **足球应用**: 编码事件的时间顺序和场上位置

### Multi-Head Attention
- **机制**: 多个注意力头学习不同的表示子空间
- **好处**: 捕捉多种类型的依赖关系
- **应用**: 同时关注传球方向、防守压力等不同因素

## 🎯 在 xCV 中的应用

我的研究中使用 Transformer 来：

1. **编码控球序列** - 将一系列球员动作编码为向量表示
2. **上下文建模** - 结合场上位置、球员分布等上下文
3. **价值传播** - 通过注意力机制传播价值信号

## 🔗 相关资源

- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)
- [Attention Is All You Need (原论文)](https://arxiv.org/abs/1706.03762)

## 📂 相关笔记

- [强化学习](../reinforcement-learning/index.md) - 结合 Transformer 做价值估计
- [足球分析](../soccer-analytics/index.md) - 在事件序列中的应用

---

*持续更新中...*
