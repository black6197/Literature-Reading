# 足球分析论文笔记

足球数据分析和机器学习应用相关论文整理。

## 📖 已阅读

暂无

## 📝 核心方法

### Expected Goals (xG)
- **定义**: 预测射门转化为进球的概率
- **特点**: 只关注射门事件
- **局限**: 无法评估非射门动作的价值

### VAEP (Valuing Actions by Estimating Probabilities)
- **核心思想**: 评估每个动作对进球/失球概率的影响
- **优势**: 可以评估所有类型的动作
- **方法**: ΔP(score) - ΔP(concede)
- **局限**: 使用手工设计的奖励，基于事件级分析

### Expected Possession Value (EPV)
- **定义**: 评估控球权在场上不同位置的价值
- **特点**: 连续空间建模
- **应用**: 评估传球和移动的价值

## 🎯 我的研究：xCV 框架

**Expected Contribution Value** 旨在解决现有方法的局限：

### 关键创新

1. **链式分析 vs 事件级分析**
   - VAEP: 分析单个动作 → xCV: 分析完整控球链
   - 好处: 更好地解决信用分配问题

2. **学习型奖励 vs 手工奖励**
   - VAEP: 手工设计 ΔP 奖励 → xCV: 神经网络学习终端价值
   - 好处: 更客观，数据驱动

3. **上下文感知**
   - 结合 360° 追踪数据
   - 考虑防守压力、空间控制等

### 技术架构

```
输入: 控球序列 + 360° 追踪数据
      ↓
   Transformer 编码
      ↓
   TD Learning 价值传播
      ↓
输出: 每个动作的贡献价值
```

## 📊 相关数据集

- **StatsBomb Open Data** - 免费的比赛事件数据
- **Wyscout** - 大规模足球数据
- **Second Spectrum** - 360° 追踪数据（需授权）

## 🔗 相关资源

- [StatsBomb 数据文档](https://github.com/statsbomb/open-data)
- [Friends of Tracking](https://www.youtube.com/channel/UCUBFJYcag8j2rm_9HkrrA7w) - 足球分析教程

## 📂 相关笔记

- [强化学习](../reinforcement-learning/index.md) - TD Learning 核心方法
- [Transformer](../transformer/index.md) - 序列编码技术

---

*持续更新中...*
