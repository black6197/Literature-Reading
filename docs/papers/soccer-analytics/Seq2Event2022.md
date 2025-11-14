# Seq2Event：基于Transformer的足球事件预测（2022） 

<div style="text-align: center;">
    <a href="https://dl.acm.org/doi/10.1145/3534678.3539138">
        <img width="2000" height="497" alt="image" src="https://github.com/user-attachments/assets/15a392bd-4a48-4672-83e3-b946230b1f34" />
    </a>
</div>


## 摘要

足球是一项特点为开放性和动态性的运动，球员的行动和角色根据团队战略在多个时间尺度上同时进行调整，并且具有高度的空间自由度。这种复杂性带来了分析上的挑战，迄今为止，解决方案主要是根据特定标准将比赛分解以分析特定问题。我们提出了一种更加整体的方法，在新型Seq2Event模型中利用Transformer或RNN组件，<span style="color: red;">该模型根据先前的比赛事件和上下文预测下一个比赛事件</span>。我们展示了使用通用的上下文感知模型创建指标作为可部署的实际应用，并演示了使用Seq2Event模型<span style="color: red;">开发poss-util指标的过程</span>。通过总结每次控球期间关键进攻事件（射门、传中）的期望值，我们的指标显示与流行的xG指标在多场比赛中具有较高相关性（r = 0.91，n = 190）。文章展示了poss-util在分析控球行为和比赛中的实际应用实例。此外，我们还讨论了该方法在具有更强序列性的运动中（如英式橄榄球）的潜在应用价值。

## 1 引言

在足球等团队性运动中，定量分析在教练过程中发挥着不可或缺的作用，并为战略和战术决策提供信息支持[**[9]**](#ref-9)[**[34]**](#ref-34)。近年来，随着这个价值数十亿美元的产业[**[6]**](#ref-6)中的职业队伍努力寻求额外的见解来支持和强化教练理念，定量分析的重要性和普及程度显著提高。这一发展加速的原因是比赛事件和跟踪数据的广泛性和可获取性，以及源自其他行业且在其中得到发展的大量研究和分析技术的整合[**[11]**](#ref-11)。

将传统技术与体育特定公式相结合，促成了几种新指标的开发和实施[**[10]**](#ref-10)[**[12]**](#ref-12)[**[20]**](#ref-20)，这些指标成功地提供了对比赛更深入的理解。然而，随着足球分析的日趋成熟，这类公式变得越来越难以发现，需要更复杂的技术来继续推进该领域的前沿发展[**[9]**](#ref-9)。机器学习(ML)技术的最新进展，加上数据的广泛可用性，提供了一个机会，使我们能够理解体育中的复杂性，远超传统统计技术的能力。

虽然在体育研究中已有使用**序列机器学习技术**的证据[**[15]**](#ref-15)[**[34]**](#ref-34)[**[35]**](#ref-35)，但大多数情况下使用的是**序列不变**方法，如**随机森林(RF)**、**多层感知器(MLP)**或**卷积神经网络(CNN)**，通过预先确定的数据聚合或时间数据的连接来分析比赛事件序列。本质上设计用于模拟序列数据的技术在**自然语言处理(NLP)**领域已经产生了近十年的最先进成果，其中**循环神经网络(RNN)**[**[13]**](#ref-13)[**[19]**](#ref-19)系列架构，特别是**长短期记忆(LSTM)**[**[17]**](#ref-17)和**门控循环单元(GRU)**[**[8]**](#ref-8)，以及**Transformer**[**[32]**](#ref-32)系列架构在前沿应用中处于领先地位。

在本文中，我们展示了当这些架构构建在适当的框架中时，可以用来解决体育分析问题。我们提出的新型Seq2Event模型作为一个实际示例，其中使用一系列比赛事件数据来预测下一个比赛事件。超参数搜索实验发现该模型优于基准统计方法，并在应用于顶级比赛的横截面时提供了关于最佳模型超参数的一些信息。预测分析表明模型做出了上下文感知的预测。

我们引入了<span style="color: red;">新型poss-util指标，该指标表达了任何给定控球的进攻利用率，它被公式化为Seq2Event模型中进攻概率的积分函数</span>。我们将这些方法应用于西甲联赛的球队，展示了该指标表征队伍进攻行为的能力。指标的直接开发（<span style="color: red;">poss-util是传中和射门概率的函数</span>）说明了通用模型（Seq2Event预测下一个事件的六个属性）用于加速指标开发和部署的潜力。

本文旨在通过以下方式推进体育分析的最新技术水平：

(1) 新型Seq2Event模型允许通过RNN或Transformer编码器组件学习比赛事件序列和上下文特征的表示，从中预测下一个比赛事件的动作和位置。

(2) 使用来自七个顶级比赛的138场比赛的真实数据，训练模型以探索和确定此类模型的最佳超参数。Elman-RNN、LSTM、GRU和Transformer变体都比基准自回归(AR)和马尔可夫模型产生更好的结果（比最佳基准改进>21%），其中LSTM和Transformer模型产生最佳预测性能（比最佳基准分别改进53%和46%）。

(3) 我们引入poss-util指标来表达每次控球的进攻利用率。基于事件，将上下文相关的预期进攻概率在每次控球的事件上进行积分，并乘以一个指示变量，该变量基于是否确实发生了进攻。

(4) 我们基于2017/18赛季西班牙甲级联赛的五支球队提供了一个案例研究，展示了将模型与指标结合使用在展示球队控球利用率方面的实用性。

本文的组织结构如下：第2节回顾了体育分析和机器学习(ML)领域的相关文献。第3节介绍了Seq2Event模型，第4节和第5节分别呈现了实证评估和案例研究应用的结果。第6节讨论了模型在足球行业以及其他职业体育中用于指标开发和决策的潜力。

## 2 背景

本节回顾了用于建模序列的技术相关文献，随后总结了用于识别足球队战略的指标。

### 2.1 序列建模

首先回顾传统统计方法以形成基准模型，然后回顾用于Seq2Event模型的机器学习技术。

**基准技术**：**自回归(AR)模型**是流行的**自回归积分移动平均(ARIMA)模型**家族[**[16]**](#ref-16)的一个组成部分，作为足球事件数据连续特征观测的简单基准模型。**马尔可夫模型**家族[**[14]**](#ref-14)常用于离散的马尔可夫数据，作为足球事件数据分类动作特征观测的简单基准模型。在此领域，由于数据是回顾性检查的，系统被视为自主的。尽管"踢足球"这一过程极其复杂，可能存在无法观察到的其他潜在参数，但观测是系统且稳健的，因此选择**基于转移概率矩阵**的简单马尔可夫链模型作为动作预测的基准模型。**模型阶数**（即模型回溯多远）是两种基准模型的超参数。

**机器学习技术**：**循环神经网络(RNN)**[**[19]**](#ref-19)[**[25]**](#ref-25)被引入作为学习序列的机器学习方法。其网络架构基于前馈深度神经网络，添加了循环连接，运行时接收输入并前向传播。某些输出单元值被输入到持久单元中，从而使网络能够记忆状态。**Elman-RNN**[**[13]**](#ref-13)被用作Seq2Event架构中最简单的机器学习模型组件。RNN应用于长序列曾受到**梯度消失**问题的限制，即长期趋势因误差噪声而丢失**。LSTM[**[17]**](#ref-17)通过引入恒定误差传送带(CEC)并通过带门的LSTM单元应用这一概念来解决这一问题**，从而避免了之前遇到的误差累积和信号丢失。LSTM在许多自然语言处理任务上取得了最先进的结果，但在这一领域被Transformer模型的使用所超越[**[32]**](#ref-32)[**[33]**](#ref-33)。**GRU**[**[8]**](#ref-8)单元被提出作为LSTM单元的更简单替代方案，包含的可学习参数减少了三分之一。然而，有观察表明，在更复杂的序列上，LSTM可能由于控制记忆对网络的暴露而提供更好的拟合[**17]**](#ref-7)。**Transformer[**[32]**](#ref-32)旨在不使用递归来解决序列学习问题，而是使用位置嵌入和注意力机制来学习数据的序列性质，采用编码器-解码器架构**。通过消除每个训练周期迭代数据的需要，这些模型可以比RNN等效模型快得多，只要较大的Transformer模型能够装入计算内存。完整模型由编码器堆栈后跟解码器堆栈组成，已被用于设定序列到序列自然语言处理任务的最先进结果，例如将英语序列作为输入、德语序列作为输出的英德翻译。在序列到事件的足球预测任务中，由于只预测单个事件而不是完整序列，因此不需要解码器堆栈，因此只使用Transformer编码器堆栈。历史上，LSTM和Transformer模型在足球中的使用主要集中在比赛视频处理任务上，如注释[**[30]**](#ref-30)和摘要[**[2]**](#ref-2)，尽管最近其他作者也开始研究团队行为预测[**[15]**](#ref-15)[**[35]**](#ref-35)。

<div style="text-align: center;">
    <img width="924" height="288" alt="image 1" src="https://github.com/user-attachments/assets/1bab47e8-70d1-4a04-8dd6-3591e417e5e0" />
</div>

### 2.2 识别足球队战略的指标

预期进球(xG)最初引入到冰球运动中[**[21]**](#ref-21)。该文指出冰球的一个局限性是与篮球等其他运动相比，得分率较低。文章指出，**进球的随机性和稀缺性限制了仅使用进球来正确判断球队当前表现和预测未来表现的能力。**得分稀疏性是足球共有的难题[**[9]**](#ref-9)。为了提供更连续的背景，xG模型允许估计一支球队根据关键指标（如射门、射偏、被封堵的射门、失误、争球和实际进球）表现应该得到的进球数。xG[**[21]**](#ref-21)使用**普通最小二乘(OLS)回归和岭回归**对这些指标的时间序列建模。xG首次在2016年被记录于足球中[**[12]**](#ref-12)，模型基于随机森林和Adaboost[**[12]**](#ref-12)。xG的一个局限性是它仅是即时射门动作的函数，不考虑产生机会但未导致可累积xG的重要比赛事件的情况。几个指标旨在解决这一问题，如**无球得分机会**[**[31]**](#ref-31)、**预期助攻(xA)**[**[23]**](#ref-23)、**预期进球链(xGChain)**[**[20]**](#ref-20)、**预期进球构建(xGBuildup)**[**[29]**](#ref-29)和**预期威胁(xT)**[**[29]**](#ref-29)指标。通过**估计行动价值估计概率(VAEP)指标**[**[10]**](#ref-10)采取了更全面的方法，虽然它仍然只是持球动作的函数，但估计了在不久的将来进球的概率。该指标已被证明是成功的，并简洁地说明了在比赛的进攻和防守方面的超额完成和不足完成。VAEP分数的产生以需要为进球和失球概率建立模型为基础，Decroos等人[**[10]**](#ref-10)使用**Catboost梯度提升模型**基于过去三个动作生成下一个进球或失球的概率。

估计团队战术的回报是Beal等人[**[3]**](#ref-3)的研究重点，在此基础上，决策制定的长期优化[**[4]**](#ref-4)。为团队定义了"流畅目标"，这是"目标变量"的函数，对应于代理规划视野中的不同点。进行马尔可夫链蒙特卡洛(MCMC)方法，以预测战略针对不同目标的未来表现，并用于指导代理选择最佳目标。更完整的综述请参见Beal等人[**[5]**](#ref-5)。

## 3 用于比赛事件预测的SEQ2EVENT模型

本节介绍我们用于比赛事件预测的新型Seq2Event模型的架构及相关损失函数。

### 3.1 损失函数（Loss Function）

定义一个能够表征误差的损失函数对于正确衡量模型的成功至关重要。虽然数据集中的所有特征作为模型输入都是相关的，但并非所有特征都能区分不同的比赛风格。例如，比分优势是一个重要的上下文输入，但变化不频繁，因此不是直接预测的兴趣点。**动作类型和持球动作导致的x、y空间位置被选为目标变量，因为它们是最基础的兴趣特征，并且在源数据中被直接观察和记录。**

<span style="color: red;">**交叉熵损失(CEL)**常用于自然语言处理中等效的多类分类任务，在此用于衡量下一个动作预测的误差。均方根误差(RMSE)损失被确定为适合 x、y坐标误差</span>，特别是因为这一指标在空间度量上直观对应欧几里得距离。应用未加权的总和会导致模型偏向于优先减少RMSE损失，因此最终损失函数呈现为加权总和，权重通过实验确定。结果损失函数如等式1所示：

<div style="text-align: center;">
    <img width="646" height="45" alt="image 2" src="https://github.com/user-attachments/assets/b42d3e95-8aae-4252-8685-1ac961699afc" />
</div>

事件动作类型存在类别不平衡，为解决这个问题，CEL本身按类别出现比例的倒数加权。这意味着，例如，"传球"的强真阳性不会产生像同等强度的"射门"真阳性那样低的损失，因为射门出现的频率较低(1.4%对比56%)。进球、控球权转换事件和比赛结束事件纯粹是上下文性的，不被视为这一任务的风格指标，因此在CEL函数中给予零权重。

### 3.2 Seq2Event模型架构

Seq2Event模型包括七个主要阶段，有RNN和Transformer变体，如**图1**所示。输入是从t-seqlen到t-1的比赛事件，输出是对t时刻事件的估计。

**阶段1：可学习嵌入（Learnable Embedding）**。对于这个任务，我们可以简单地将动作视为词语。然而，我们知道事件的上下文具有意义，并希望在模型中捕捉十个连续变量。使用嵌入层嵌入动作，使用密集层转换连续变量。超参数：

- seqlen：序列长度；模型回溯多远。
- act_embed_dim：动作嵌入维度。
- cont_embed_dim：连续特征密集层维度。

**阶段2：连接（Concatenation）**。我们需要向RNN/Transformer传递一个矩阵，该矩阵由顺序事件的向量组成，因此连接前一阶段的两个输出。在Transformer变体中，在此阶段还应用位置嵌入。

**阶段3：RNN/Transformer组件**。在此阶段运行RNN或Transformer编码器组件。RNN变体超参数：

- RNN类型：指定Elman-RNN、LSTM或GRU之一。
- 隐藏大小：每个RNN单元内的隐藏单元数量。
- 层数：堆叠的RNN单元数量。
- 丢弃率：如果层数>1，则训练期间随机忽略的最终层权重比例。
- 方向性：单向：此阶段是从输入序列开始到结束的有序输入的函数；或双向：此阶段是从开始到结束，然后从结束到开始的有序输入的函数。
- 激活函数：对于Elman RNN，在每次时间迭代更新隐藏状态之前应用的激活函数(例如ReLU)。

Transformer变体超参数：

- 前馈维度：类似于RNN隐藏大小。
- 头数：多头注意力组件中的头数。
- 层数：堆栈中并行层的数量。
- 丢弃率：类似于RNN丢弃率。

**阶段4：提取最终预测（Extraction of final prediction）**。RNN/Transformer组件对整个集合进行一步预测。然而，对于这个任务，我们只对代表下一个事件的集合的最终预测感兴趣。

**阶段5：带ReLU激活的密集层（Dense Layer with ReLU Activation）**。从阶段5开始，重点是将表示映射到动作和x、y输出。应用带有修正线性单元(ReLU)[**[22]**](#ref-22)激活函数的密集层。初始原型排除了这一层，但Javid等人[**[18]**](#ref-18)提到这种结构可能有助于学习，特别是当使用更大的隐藏单元大小时，因为当神经元输出低于x=0激活阈值时可能被忽略。应用于原型模型显示训练损失略有改善(约1%)。超参数：

- 输出维度：密集层维度。

**阶段6：密集层（Dense Layer）**。最终可学习层是将表示映射到最终输出的密集层。预测七种动作类型，尽管由于在损失函数中给予零权重，其中三种类型是冗余的，因此在未来版本中可以减少到四种动作类型。

**阶段7：分割（Splitting）**。长度为9的向量分为长度为7的动作logits向量(用于四个动作加上三个控球权改变字符)，和长度为2的x、y位置坐标向量。在训练下，等式1应用作这些输出的损失函数。

<div style="text-align: center;">
    <img width="1695" height="832" alt="image 3" src="https://github.com/user-attachments/assets/e4cc7bf9-88e8-40c7-9209-29c1a94c9ffa" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图1：Seq2Event模型架构。基于包含十个连续特征和一个分类动作特征的11 × seqlen源数据，预测下一个动作和位置。可学习组件以橙色和红色阴影标示。
</p>

**对于实际应用，使用了简化的特征工程动作：传球('p')、盘带('d')、传中('x')和射门('s')，如附录A中详述。对四个动作logits取softmax，产生下一个动作预测概率。图2**所示的模型输出示例中，每个单独预测都是基于前40个事件给出的。x、y图中的空隙是球队未处于控球状态的比赛时段，在动作序列中显示为'_'。模型不预测失误，因为它们在损失函数中给予零权重，这是有意为之，因为预期目的是分析进攻行为，其中失误预测相关性较低。然而，损失函数和/或模型的最终阶段可以轻松修改以用于其他可能包括失误预测的任务。

<div style="text-align: center;">
    <img width="1026" height="795" alt="image 4" src="https://github.com/user-attachments/assets/f01c5ac5-81f0-466e-a9f9-2cfa385bd843" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图2：Seq2Event模型预测（蓝色）的下一个事件位置和动作与实际情况（黑色）的对比，每个时间步都基于前40个事件的上下文。位置中的空隙（灰色）表示控球权转换。
</p>

## 4 实证评估

为评估我们的模型，使用了来自WyScout开放访问数据集[**[24]**](#ref-24)的比赛事件数据，涵盖了英国、法国、德国、意大利和西班牙男子顶级联赛的2017/18赛季，以及国际欧洲杯2016和世界杯2018比赛。在总共1,941场比赛中，选取了138场具有代表性的比赛样本，这些样本跨越了不同的成功程度和比赛。关于我们结果可重现性的更多细节可在**附录A**中找到。

### 4.1 实验1：超参数选择（Experiment 1: Hyperparameter Selection）

超参数搜索通过在所有参数的广泛值范围内进行初步实验来确定可能最优的值，然后在这些值上进行超参数网格搜索。总共拟合了145个不同的模型，使用如**图3**所示的数据流。所有模型按测试损失的性能如**图4**所示。

<div style="text-align: center;">
    <img width="817" height="589" alt="image 5" src="https://github.com/user-attachments/assets/e0322ce1-5708-453d-8d42-cfa1c8ab1507" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图3：WyScout数据被重新排列以形成用于Seq2Event建模的数据流。从按时间排序的源数据（上两行）开始，识别每支球队的控球时段（第三行）。每次控球的事件按球队聚合，并在控球之间添加转换指示符。
</p>

<div style="text-align: center;">
    <img width="807" height="370" alt="image 6" src="https://github.com/user-attachments/assets/f1b578d6-94d5-4795-b7eb-27b0164d8549" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图4：超参数搜索过程中所有训练模型的训练损失（蓝色）、验证损失（黄色）和测试损失（黑色）。
</p>

> 图4的解释：145种模型的超参搜索，x轴表示模型排名（基于测试loss），y轴表示模型的los，（也就是损失）。其中排名140到145是基线模型（Baseline models），排名120-140是Elman-RNN模型，最早期的RNN架构。右侧（排名1-100）为现代深度学习模型（LSTM、GRU、Transformer模型）。LSTM-22,21（排名1+2），最佳性能；Transformer-31（排名3）第三佳，但训练更快。
> 

[关于超参搜索](https://www.notion.so/266692a22bd880039f5bf4ad47b51bce?pvs=21)

从持球的x、y和时间数据中提取工程化的连续特征，以提供在足球分析中具有已知上下文重要性[**[1]**](#ref-1)[**[10]**](#ref-10)且在表示上相对独立的特征。最终的特征集包括x、y、T及它们各自在观测之间的变化量；观测之间覆盖的距离；与对方球门的角度和距离；以及比分优势（负值表示处于落后状态）。

通过将106种可能的动作类型映射到四种广义动作类型：传球、盘带、传中和射门，工程化了一个分类动作特征。减少操作基于三个原则：(1)类别支持必须足够充分，以便允许在中等规模的数据集上进行学习；(2)动作在足球领域的上下文中必须有足够明显的含义区别；(3)简化的编码必须产生具有合理顺序多样性的序列。

在给定一支球队之前的进攻动作来预测下一个进攻动作的任务中，每场比赛仅保留了目标球队的进攻动作，但添加了控球权变化标记。因此，可以在数据中进行指定长度和步长的观测，如**图3**所示。

我们拟合了12个基准自回归和马尔可夫链模型，阶数为1-5，基于x、y和所有十个连续变量。最佳基准模型是基于所有十个连续变量的一阶模型（测试损失0.704）。最差的Seq2Event模型优于此分数，这是一个具有17个头和隐藏单元大小4096的Transformer变体（测试损失0.548）。<span style="color: red;">表现最好的是一个LSTM变体，序列长度为100，1层，隐藏单元大小为8，单向顺序（测试损失0.332），其双向等效模型排名第二（测试损失0.344）。一个序列长度为40，1层，隐藏单元大小为8的Transformer变体排名第三（测试损失0.362）。</span>

模型的PyTorch实现[**[28]**](#ref-28)在Google Colab Pro硬件上运行，由Nvidia Tesla P100 GPU提供加速。一般来说，<span style="color: red;">在等效设置下，Transformer模型比RNN变体训练得更快。最佳LSTM模型训练时间为15.5小时。将序列长度减少到40可将模型训练时间减少到3.5至5.5小时，取决于其他超参数，但这仍比可比较的最佳Transformer模型要长得多，后者仅需1.4小时训练完成</span>。这验证了Transformer模型的已知优势之一，即该架构避免了对数据的迭代，因此只要整个源数据适合内存，拟合速度就会更快；而RNN模型，包括LSTM，必须在数据上按顺序操作，计算复杂度与源序列长度呈线性关系[**[26]**](#ref-26)。

LSTM模型按测试损失标准产生了最佳结果，并且优于具有等效配置的Transformer模型（测试损失0.332对比0.379），这表明它在学习长序列模式方面具有增强能力，这一点在其他领域的这种架构中也有所注意[**[7]**](#ref-7)[**[8]**](#ref-8)。GRU模型的可学习参数比LSTM少，这导致训练时间略快，但测试性能不如LSTM或Transformer模型。ElmanRNN模型是最早的RNN类型，通常在所有Seq2Event变体中提供最差的测试性能。

序列长度为5的模型、层数超过2的RNN模型和头数超过2的Transformer模型的表现均较差（最佳模型损失分别为0.423、0.440、0.384）。我们强调<span style="color: red;">LSTM对于未来研究特别有前景，尤其是涉及100或更多序列长度的研究，在这种情况下可以证明计算资源的合理性</span>。对于通过检查与"平均球队"的差异来识别团队战略的应用，回顾太多动作可能是不可取的。模型可能会捕捉到球队的"风格"，并开始通过预测"当前球队"会做什么的下一个事件而受益。在本文的下一节中，由于计算速度优越，使用了性能排名第三且序列长度为40的Transformer模型来生成结果。

### 4.2 实验2：动作预测概率分析（Experiment 2: Analysis of Action Prediction Probability）

对所有四种动作类型（传球、盘带、传中、射门）评估了实际动作发生情况与预测动作发生概率。这里重点关注射门预测，**图5**显示了相关摘要统计的空间分布。**图5(a)**表明射门预测大致与实际数据一致（与**图10**对比）。**图5(b)**表明，在预测射门的情况下，平均射门预测概率通常在接近对方球门时更高。左侧的高强度区域样本量非常低，不具有统计显著性。**图5(c)**展示了模型在给定位置上的射门预测概率多样性。即使在右侧高样本量区域，也可以观察到多样化的射门概率。类似的多样性可以在图9中观察到。这些是重要的观察结果，因为它们表明，尽管模型确实在我们根据经验空间分布预期的地方预测了较高的射门概率，但它们的生成既不完全是空间源的独立函数，也不是预测特征的独立函数。

[这部分解读](https://www.notion.so/266692a22bd880288cd5e855f528158e?pvs=21)

<div style="text-align: center;">
    <img width="649" height="769" alt="image 7" src="https://github.com/user-attachments/assets/0184d273-f619-4506-9611-dace2d488e95" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图5：射门预测统计的空间分布（n=5,666）。(a) 预测的下一个动作为射门；(b) 在预测射门的条件下射门概率的平均值 P(射门|预测射门)；(c) 在预测射门的条件下射门概率的标准差
</p>

该模型的多类混淆矩阵如**表1**所示。对于所有动作，模型平均正确地分配了最高概率，传球除外，其中模型预测盘带(P = 0.38)多于传球(P = 0.36)。这可能通过微调CEL部分损失函数中的权重来解决。考虑到射门(2.2%)、传中(3.9%)和盘带(10.2%)的实际类别发生率较低，结果是合理的。

<div style="text-align: center;">
    <img width="618" height="259" alt="image 8" src="https://github.com/user-attachments/assets/0795abb7-cccf-46c5-81fb-f98c51590e39" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
表1：多类混淆矩阵（平均概率）
</p>

### 4.3 实验3：控球利用分析（Experiment 3: Analysis of Possession Utilisation）

**攻击指标开发（Attack metric development）**：在预测的四种动作中，射门和传中可以被视为具有"攻击"意图。传中定义为从进攻侧翼传出的球，目标是对方球门前区域的队友；射门定义为朝对方球门尝试射门，意图进球。将射门和传中预测概率相加，得出"攻击"概率。进一步按控球权累加，给出每次控球期间累积的攻击期望权重的度量。

为了区分攻击性和非攻击性控球，当控球过程中没有攻击事件（射门或传中）发生时，累加的攻击概率乘以-1。不攻击的原因各不相同，这种方法可以获得一些见解。较弱的球队可能根本没有创造出攻击机会（从而导致累积期望的幅度较低，带负号）。相反，较强的球队可能一直在寻找开始攻击的最佳机会（在此过程中产生高幅度的累积期望，带负号）。此外，比赛状态（领先、落后、平局）和比赛时段等上下文因素也可能影响球队的攻击战略。

为了便于解释，对正负两组应用百分位数排名，为每次控球提供范围为[-1, 1]的指标，所得统计量称为poss-util。

**指标应用（Metric application）**：该指标应用于2017/18赛季西甲的选定球队：巴塞罗那（联赛冠军）、马德里竞技（第2名）、皇家马德里（第3名）、赫罗纳（第10名）和马拉加（第20名）。**图6(a)**显示了按球队划分的控球分布，可以看出poss-util正值较低，因为只有少数控球导致攻击（23%）。高幅度值，即接近-1或1的值，表示在控球期间积累了高概率的攻击，但只有正号的那些最终导致了攻击（射门或传中）。

<div style="text-align: center;">
    <img width="613" height="580" alt="image 9" src="https://github.com/user-attachments/assets/2b6f6eb1-7a61-40ce-81a7-bf78ba9dd226" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图6：使用poss-util分析团队行为。(a) 控球过程中poss-util的分布（n=23,951）；(b) 比赛中正值poss-util均值的分布（n=190）
</p>

[图6的解读](https://www.notion.so/6-266692a22bd88073bc80c64263325283?pvs=21)

巴塞罗那和皇家马德里值得注意的是，与其他球队相比，它们有着明显不同的分布。它们倾向于产生具有更高攻击期望的控球，这在高幅度正值和负值的更高密度中显示出来。马德里竞技、赫罗纳和马拉加彼此之间有相似的分布。马德里竞技与这两支球队的相似性也许令人惊讶，考虑到最终联赛排名的差异，从第二名到中游和垫底。然而，马德里竞技以不同寻常的防守风格著称，这对于如此成功的球队来说很不寻常。<sup style="color: red;">3</sup>他们场均失球最少（0.6），明显低于联赛平均水平（1.6）。相比之下，在进攻方面，他们的场均进球仅略低于平均水平（1.5对比1.6），这验证了我们的发现，即马德里竞技在进攻中与更普通的球队相似。赫罗纳场均进球1.3个，排名中游。马拉加场均仅进0.6球，尽管其poss-util分布与马德里竞技和赫罗纳相似，但作为垫底球队，这一指标突显了最后三分之一区域技术熟练度的重要性，以及球员执行特定技能（射门、传中）的高精度能力和/或这些位置的球员做出何时执行动作的最佳决策的能力。巴塞罗那和皇家马德里都贡献了联赛中最高的进球数（场均分别为2.61和2.47个进球）。

> <sup style="color: red;">3</sup>[https://bleacherreport.com/articles/2589852-analysing-atletico-madrids-defensivestructure-under-diego-simeone](https://bleacherreport.com/articles/2589852-analysing-atletico-madrids-defensivestructure-under-diego-simeone)

**使用poss-util预测进球（Using poss-util to predict goals）**：选择进球数作为比较指标，作为一个关键且明确定义的统计数据。分析了所有、仅正值和仅负值poss-util的总和、平均值和中位数的相关性（九个实验）。正值poss-util的中位数具有最高的皮尔逊相关性（r = 0.47）。正值poss-util的总和预计是最直观的，但仅有较弱的相关性（r = 0.17）。正值poss-util的平均值仅显示比中位数略低的相关性（r = 0.46），且更容易被体育分析师理解，因此是选择的方法。最后，使用每场比赛控球过程的正值poss-util均值（p ∈ M）对整个赛季的实际进球数进行线性变换，以得出预测进球数（ĝ），如等式2所示。

<div style="text-align: center;">
    <img width="595" height="70" alt="image 10" src="https://github.com/user-attachments/assets/64f109e1-9876-4ba1-a290-ef1df05414f1" />
</div>

**与实际进球和xG的验证（Validation against actual goals and xG）**：发现poss-util预测的进球数与进球数适中相关（跨比赛r = 0.46），<sup style="color: red;">4</sup>并且与xG强相关（r = 0.91）。xG与实际进球数的相关性略强于我们的指标（r = 0.57对比0.46）。**图7**进一步展示了两个指标之间的相似性，尽管xG表现略好（RMSE 1.29对比1.42）。**表2**显示，当按赛季平均聚合时，两个指标与实际值的相关性都非常高，且我们的指标表现略好（poss-util r = 0.98对比xG r = 0.97）。球队相对于xG的表现不足和超额表现的方面在我们的指标中也可见。

> <sup style="color: red;">4</sup>[https://understat.com/league/la_liga/2017](https://understat.com/league/la_liga/2017)

由于xG是一个用于目标得分预测的流行且可靠的模型[**[31]**](#ref-31)，预测性能的相似性验证了我们的指标。通过归纳，这也验证了由其导出的基础Seq2Event模型，该模型在更一般的下一个事件预测任务上进行了训练。

<div style="text-align: center;">
    <img width="654" height="306" alt="image 11" src="https://github.com/user-attachments/assets/ca34ab40-b5ec-4037-93d6-ecf4678d4179" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图7：xG和基于平均poss-util预测进球数的误差分布（比赛数n=190）。
</p>

<div style="text-align: center;">
    <img width="624" height="279" alt="image 12" src="https://github.com/user-attachments/assets/b7cca91b-d5a6-4ea5-a05b-e13446bc0cdf" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
表2：2017/18赛季西甲每场比赛平均进球数
</p>


## 5 模型在西甲联赛中的应用

在本节中，我们展示了Seq2Event模型如何作为一种团队剖析方法在实践中应用，通过使用球权效用(poss-util)指标来深化对比赛的理解，并获取关于比赛过程中进攻行为的额外见解。

**比赛时间线视图（Match timeline view）**：通过计算每次球权的指标，**图8**展示了两场比赛中球权效用的演变。第一张图显示了2018年4月7日巴塞罗那3-1战胜莱加内斯的比赛。总体而言，巴塞罗那的平均正向球权效用为0.61，对应2.5个预测进球数(与预期进球xG指标2.7相近)。如负区域中橙色10分钟滚动平均线所示，巴塞罗那在整场比赛中产生了大量高进攻潜力但未转化为进攻的球权(线接近-1)。当他们确实转化为进攻时，正区域的中位数可以看到很高，尽管如滚动平均线所示，进攻发生得不频繁。查看其他相关比赛数据，值得注意的是巴塞罗那在第26分钟和第31分钟进球后取得并保持了领先；而莱加内斯仅获得0.65的预期进球数。通过这种方式使用模型可以获得两个主要见解：巴塞罗那有很高的预测进球数，并且在这方面表现大致符合预期；尽管积累了大量高潜力球权，他们的进攻并不频繁(由蓝线表示)。

<div style="text-align: center;">
    <img width="660" height="646" alt="image 13" src="https://github.com/user-attachments/assets/4101c9a3-e51c-44b2-a814-b6d74f3548e5" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图8：球权效用(poss-util)随时间的演变。灰色点代表单次球权；阴影单元格提供点密度和幅度的视觉指示；蓝色和橙色线条分别表示10分钟正向和负向滚动平均线。垂直绿线表示进球，垂直红线表示失球。(a) 巴塞罗那的球权效用随时间变化（对阵莱加内斯，2018年4月7日，3-1获胜）；(b) 皇家马德里的球权效用随时间变化（对阵比利亚雷亚尔，2018年1月13日，0-1失利）
</p>

第二张图显示了2018年1月13日皇家马德里0-1负于比利亚雷亚尔的比赛。总体而言，皇马的平均正向球权效用为0.49，对应2.4个预测进球数(与预期进球xG指标2.35相近)。可以看到皇马在整场比赛中持续产生各种不同进攻潜力和转化为进攻的球权组合。比利亚雷亚尔在第86分钟打进唯一一球。

**球权概览（Possession overview）**：基于该指标选择感兴趣的球权，**图9(a)**显示了巴塞罗那球权效用最高的一次球权。更亮的颜色表示更高的进攻期望值，这次长时间的球权可以看到在对方三区边缘累积了多次中等进攻期望值，随后在对方左侧角球区和禁区内出现几次高进攻期望值。最终，他们做出了一次传中，虽然没有导致进球，但耐心的进攻建设随后的进攻获得了高指标分数。相比之下，第二张图显示了一次中等指标分数为0.39的球权。这是一次直接打法，最初预期是门将的长传球传中，随后是一次未成功的射门。

<div style="text-align: center;">
    <img width="573" height="838" alt="image 14" src="https://github.com/user-attachments/assets/53d80869-6c94-4cdc-a8b6-c46246e480b1" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图9：poss-util模型预测下一动作为进攻的概率，显示实际x、y坐标和实际动作与预测最可能动作，跨两次球权。颜色表示进攻概率；'S'和'F'分别表示球权开始和结束。动作解码参见**表3**。(a) 巴塞罗那的球权（对阵莱加内斯，2018年4月7日，3-1获胜）；(b) 皇家马德里的球权（对阵比利亚雷亚尔，2018年1月13日，0-1失利）
</p>

<div style="text-align: center;">
    <img width="552" height="783" alt="image 15" src="https://github.com/user-attachments/assets/44cf7fdf-2566-4bd8-8443-3b2b3db874bb" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
表3：WyScout事件到项目编码映射
</p>

## 6 讨论

**团队行为作为方言（Team Behaviour as Dialect）**：大多数体育中的事件通常是顺序发生的。模式出现，战略决策在多个时间尺度上做出。这样，运动可以使用RNN和Transformer组件进行建模和学习，有效地"学习体育语言"。与方言可以通过分析词汇与预期词汇的差异来检测的方式非常相似，不同的战术可以通过分析行动与预期行动的差异来检测。在我们的足球数据集中，平均每场比赛记录了1,401个事件，每个动作都可以被归类为一种学习到的战术。通过在宏观层面上分析战术的出现情况（按团队汇总和顺序分析），可以获得关于战术使用、战略和影响的见解。

**通用机器学习模型与特定统计模型（General Purpose ML Models vs Specific Statistical Models）**：投入精力训练更先进但通用的概率模型被强调为专业体育行业的潜在益处，有可能为团队提供多功能的建模能力，这些能力可以调整和适应以得出当代指标。随着体育的发展，这些可能被用来帮助确定规则变化或赛季结构的影响。我们训练的模型被赋予了一个通用任务：预测下一个比赛事件，该事件根据工程化的通用进攻特征进行参数化。在此基础上，可以轻松地对模型概率进行聚合和线性回归，从而生成一个有用的指标，我们已经证明该指标与一个流行的特定指标相关。从Seq2Event模型提供的其他概率来看，其他指标的制定应该是容易实现的，并且通过特别修改模型的最终层和事件流，这一通用原则可以用于检查进攻风格以外的任务。

### 6.1 未来工作

**进一步利用模型输出（Further Leveraging Model Outputs）**：已经进行了初步工作来利用预测的空间特征。对x位置的预测误差分析显示，在第5节分析的五支球队中，误差平均值没有显著差异。然而，误差的标准差与排名和进球数弱相关（2017/18西甲最终排名中，巴塞罗那到马拉加的标准差分别为0.157、0.176、0.164、0.190、0.193）。本质上，发现该模型能够比其他球队更精确地预测巴塞罗那和皇家马德里的下一个x位置。进一步的初步工作是对模型预测的动作对数和实际动作之间的差异进行聚类，识别出五种可以按球队计数并用于帮助识别球队行为的比赛风格。这是我们将在未来工作中与领域专家密切合作探索的内容。

**应用于其他运动**：其他比足球具有更强顺序性的运动也可以从这些技术和我们的模型中获益。例如，在橄榄球联盟中，从争抢、茅氏推进、边线球和争球中发生更明显的状态转换。在橄榄球联赛中，每次铲抢后比赛重置，球权限制为六次铲抢，然后转换球权。在美式足球中，跑、传或踢球是在四次进攻中未能前进十码后转换球权的。

## 7 结论

本文提出了应用顺序机器学习技术预测足球中下一个比赛事件数据的新型Seq2Event框架，并附带了寻找最佳超参数的实验结果，以及上下文事件预测和对预期进球(xG)的验证证据。在实际应用方面，我们已经证明了通用概率模型帮助快速原型开发指标的能力，作为证据，我们提出了球权效用(poss-util)指标并应用于西甲联赛。作为警示，我们注意到特定模型（如xG）可能会优于通用模型，但我们断言通用模型在专业体育分析行业中有一席之地。我们建议具有更强顺序性的运动可能特别受益于这个框架，并且还建议可以为其他任务调整初始层和最终层。

## A 可复现性

### A.1 数据准备

**WyScout开放获取数据集的使用**：本研究使用了比赛、赛事、团队和事件的JSON文件。由于本研究专注于团队表现，未使用球员文件，但值得注意的是，这些信息可随时整合到模型中，用于未来关于球员行为的研究。事件文件记录比赛事件，按21个"事件"类别和78个"子事件"类别分类，还有数百个可能的标签提供更多细节。标签类型在数据集中的应用范围差异很大，一些是特定的且只用于详细说明一种事件/子事件类型，而其他则可以更一般性地解释。

**动作特征工程（Action Feature Engineering）**：为了定义一个具有可接受维度的相关分类动作标签，必须对比赛事件属性进行简化编码。如果在更大的数据集上训练，这种简化可能不必要；事件、子事件和标签信息可以直接传递给嵌入层，从中模型可以学习这些信息的相关性。然而，对于本研究中使用的相对紧凑的数据集，认为有必要降低维度。首先进行定性分析，以筛选和分组适当表征进攻战术的比赛事件。参考<span style="color: red;">[**[9]**](#ref-9)[**[27]**](#ref-27)</span>进行的编码，并思考建模进攻风格的目标，初始编码如**表3**所示。这种编码提供了丰富的顺序多样性，当以这种方式编码数据时，为人类分析师提供了直观的画面。然而，类别支持在许多情况下不平衡或太弱，例如传球占事件的49.3%，而点球仅占事件的0.02%。因此，认为需要更粗略的编码。对于"最终"项目编码，事件被分为四类：传球('p')、盘带('d')、传中('x')和射门('s')。还增加了三种事件类型用于球权和比赛情境：进球('g')、球权结束('_')和比赛结束('@')。

**连续特征工程（Continuous Feature Engineering）**：源数据提供了事件位置坐标和比赛时间。从这些数据中，工程化了额外特征，通过提供已知方差源[9]的信息，以及通过提供不同时间尺度的上下文（例如，x、thetag和scrad随时间变化的量级不同），来帮助训练。这产生了十个归一化特征，空间分布如**图10**所示，定义如下：

- x, y ∈ [0, 1]：事件位置坐标。
- deltax, deltay ∈ [−1, 1]：与前一事件的x, y差异。
- s ∈ [0, √2]：与前一事件的距离。
- sg ∈ [0, √1.5]：到对方球门中心的距离。
- thetag ∈ [π/2, π]：从对方球门中心的角度。
- T ∈ [0, 1]：事件比赛时间。
- deltaT：自前一事件以来的时间（归一化）。
- scrad ∈ [−6, 6]：当前比分优势：领先（正）或落后（负）的进球数。

**定义建模集合（Defining Modelling Sets）**：对所有七个可用比赛的高/中/低排名球队进行抽样，以捕捉成功的代表性样本，如**表6**所示。使用50/6/44的训练/验证/测试比率，训练/验证和测试集的球队互不重叠。使用高测试权重是为了确保球队、赛季状态和联赛的代表性横截面。

<div style="text-align: center;">
    <img width="574" height="700" alt="image 16" src="https://github.com/user-attachments/assets/b9143535-c45c-4a02-be3d-0d3a6aa7e935" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图10：所有工程化源特征的空间分布情况(n=96,850)。对于动作特征，显示的是出现密度。对于连续特征，显示的是平均值。进攻方向从左至右。
</p>

### A.2 超参数搜索

**表4**概述了用于超参数搜索的方案。首先进行了广泛的手动搜索，之后对表中加粗显示的超参数进行了集中组合网格搜索。**表5**显示了前10个Seq2Event模型的超参数。然后使用第三好的模型（一个Transformer变体，已在**表6**所示数据上训练）预测整个赛季五支西甲球队的事件，结果在主论文中呈现。

<div style="text-align: center;">
    <img width="612" height="318" alt="image 17" src="https://github.com/user-attachments/assets/819fb26d-aeef-4488-946c-24942bd2bd08" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
表4：超参数搜索方案
</p>

<div style="text-align: center;">
    <img width="580" height="357" alt="image 18" src="https://github.com/user-attachments/assets/8d0675b5-54f1-4900-acd8-b3ff3a873b88" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
表5：按测试损失排序的模型：前10名加选定的基准模型
</p>

[表5的解释](https://www.notion.so/5-266692a22bd880b2849eeaf13ab67731?pvs=21)

<div style="text-align: center;">
    <img width="493" height="592" alt="image 19" src="https://github.com/user-attachments/assets/af383ea8-7c28-46aa-aeee-84a371f21244" />
    <img width="486" height="583" alt="image 20" src="https://github.com/user-attachments/assets/592f8518-19fc-4285-968d-d0e2ebdf2181" />
    <img width="483" height="433" alt="image 21" src="https://github.com/user-attachments/assets/0894e6cb-2880-4ef7-bff1-9546c750342e" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
表6：建模集合方案
</p>

### A.3 模型的PyTorch实现

建模使用PyTorch完成，我们共享了模型复现的代码[**[28]**](#ref-28)。提供了两个笔记本：一个数据准备和特征工程笔记本，以及一个建模笔记本。

# 参考文献
<a id="ref-1"></a>
[1] David Adams, Ryland Morgans, Joao Sacramento, Stuart Morgan, and Morgan D Williams. 2013. Successful short passing frequency of defenders differentiates between top and bottom four English Premier League teams. **后卫成功短传频率区分英超联赛前四名和后四名球队**. International Journal of Performance Analysis in Sport 13, 3 (2013), 653–668.

<a id="ref-2"></a>
[2] Rockson Agyeman, Rafiq Muhammad, and Gyu Sang Choi. 2019. Soccer video summarization using deep learning. **使用深度学习的足球视频摘要**. In 2019 IEEE Conference on Multimedia Information Processing and Retrieval (MIPR). IEEE, 270–273.

<a id="ref-3"></a>
[3] Ryan Beal, Georgios Chalkiadakis, Timothy J Norman, and Sarvapali D Ramchurn. 2020. Optimising Game Tactics for Football. **优化足球比赛战术**. In Proceedings of the 19th International Conference on Autonomous Agents and MultiAgent Systems. 141–149.

<a id="ref-4"></a>
[4] Ryan Beal, Georgios Chalkiadakis, Timothy J Norman, and Sarvapali D Ramchurn. 2021. Optimising Long-Term Outcomes using Real-World Fluent Objectives: An Application to Football. **使用现实世界流畅目标优化长期结果：足球应用**. In Proceedings of the 20th International Conference on Autonomous Agents and MultiAgent Systems. 196–204.

<a id="ref-5"></a>
[5] Ryan Beal, Timothy J Norman, and Sarvapali D Ramchurn. 2019. Artificial intelligence for team sports: a survey. **团队运动的人工智能：一项调查**. The Knowledge Engineering Review 34 (2019).

<a id="ref-6"></a>
[6] Daniel Berrar, Philippe Lopes, Jesse Davis, and Werner Dubitzky. 2019. Guest editorial: special issue on machine learning for soccer. **客座社论：足球机器学习特刊**. Machine Learning 108, 1 (2019), 1–7.

<a id="ref-7"></a>
[7] Roberto Cahuantzi, Xinye Chen, and Stefan Güttel. 2021. A comparison of LSTM and GRU networks for learning symbolic sequences. **LSTM和GRU网络在学习符号序列方面的比较**. (2021).

<a id="ref-8"></a>
[8] Junyoung Chung, Caglar Gulcehre, Kyunghyun Cho, and Yoshua Bengio. 2014. Empirical evaluation of gated recurrent neural networks on sequence modeling. **门控循环神经网络在序列建模中的实证评估**. In NIPS 2014 Workshop on Deep Learning, December 2014.

<a id="ref-9"></a>
[9] Tom Decroos. 2020. Soccer Analytics Meets Artificial Intelligence: Learning Value and Style from Soccer Event Stream Data. **足球分析遇上人工智能：从足球事件流数据中学习价值和风格**. Ph. D. Dissertation. KU Leuven.

<a id="ref-10"></a>
[10] Tom Decroos, Lotte Bransen, Jan Van Haaren, and Jesse Davis. 2019. Actions speak louder than goals: Valuing player actions in soccer. **行动胜于进球：评估足球中球员行动的价值**. In Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining. 1851–1861.

<a id="ref-11"></a>
[11] Deepmind. 2021. Advancing sports analytics through AI research. **通过AI研究推进体育分析**. Retrieved December 27, 2021 from [https://deepmind.com/blog/article/advancing-sportsanalytics-through-ai](https://deepmind.com/blog/article/advancing-sportsanalytics-through-ai)

<a id="ref-12"></a>
[12] H Eggels, R van Elk, and M Pechenizkiy. 2016. Expected goals in soccer: Explaining match results using predictive analytics. **足球中的预期进球：使用预测分析解释比赛结果**. In The machine learning and data mining for sports analytics workshop, Vol. 16.

<a id="ref-13"></a>
[13] Jeffrey L Elman. 1990. Finding structure in time. **在时间中寻找结构**. Cognitive science 14, 2 (1990), 179–211.

<a id="ref-14"></a>
[14] Eric Fosler-Lussier. 1998. Markov models and hidden Markov models: A brief tutorial. **马尔可夫模型和隐马尔可夫模型：简明教程**. International Computer Science Institute (1998).

<a id="ref-15"></a>
[15] Lucas Queiroz Gongora. 2021. Estimating Football Position from Context (Master's thesis). **从上下文估计足球位置（硕士论文）**. Master's thesis. KTH Royal Institute of Technology, Stockholm, Sweden.

<a id="ref-16"></a>
[16] Steven Craig Hillmer and George C Tiao. 1982. An ARIMA-model-based approach to seasonal adjustment. **基于ARIMA模型的季节性调整方法**. J. Amer. Statist. Assoc. 77, 377 (1982), 63–70.

<a id="ref-17"></a>
[17] Sepp Hochreiter and Jürgen Schmidhuber. 1997. Long short-term memory. **长短期记忆**. Neural computation 9, 8 (1997), 1735–1780.

<a id="ref-18"></a>
[18] Alireza M Javid, Sandipan Das, Mikael Skoglund, and Saikat Chatterjee. 2021. A relu dense layer to improve the performance of neural networks. **改善神经网络性能的ReLU密集层**. In ICASSP 2021-2021 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP). IEEE, 2810–2814.

<a id="ref-19"></a>
[19] Michael I Jordan. 1997. Serial order: A parallel distributed processing approach. **序列顺序：并行分布式处理方法**. In Advances in psychology. Vol. 121. Elsevier, 471–495.

<a id="ref-20"></a>
[20] Thom Lawrence. 2018. Introducing xGChain and xGBuildup. **介绍xGChain和xGBuildup**. Retrieved December 27, 2021 from [https://statsbomb.com/2018/08/introducing-xgchain-andxgbuildup/](https://statsbomb.com/2018/08/introducing-xgchain-andxgbuildup/)

<a id="ref-21"></a>
[21] Brian Macdonald. 2012. An expected goals model for evaluating NHL teams and players. **评估NHL球队和球员的预期进球模型**. In Proceedings of the 6th MIT Sloan Sports Analytics Conference.

<a id="ref-22"></a>
[22] Rajeev Murgai, Robert K Brayton, and Alberto Sangiovanni-Vincentelli. 1991. On clustering for minimum delay/ara. **关于最小延迟/面积的聚类**. In 1991 IEEE international conference on computer-aided design digest of technical papers. IEEE Computer Society, 6–7.

<a id="ref-23"></a>
[23] Opta. 2021. Opta Advanced Metrics. **Opta高级指标**. Retrieved December 27, 2021 from [https://www.statsperform.com/opta-analytics/](https://www.statsperform.com/opta-analytics/)

<a id="ref-24"></a>
[24] Luca Pappalardo, Paolo Cintia, Alessio Rossi, Emanuele Massucco, Paolo Ferragina, Dino Pedreschi, and Fosca Giannotti. 2019. A public data set of spatiotemporal match events in soccer competitions. **足球比赛时空事件的公开数据集**. Scientific data 6, 1 (2019), 1–15.

<a id="ref-25"></a>
[25] David E Rumelhart, Geoffrey E Hinton, and Ronald J Williams. 1986. Learning representations by back-propagating errors. **通过反向传播误差学习表示**. nature 323, 6088 (1986), 533–536.

<a id="ref-26"></a>
[26] Haşim Sak, Andrew Senior, and Françoise Beaufays. 2014. Long short-term memory based recurrent neural network architectures for large vocabulary speech recognition. **基于长短期记忆的大词汇量语音识别循环神经网络架构**. (2014).

<a id="ref-27"></a>
[27] Thomas Seidl, Aditya Cherukumudi, Andrew Hartnett, Peter Carr, and Patrick Lucey. 2018. Bhostgusters: Realtime interactive play sketching with synthesized NBA defenses. **Bhostgusters：使用合成NBA防守进行实时互动比赛草图**. In Proceedings of the 12th MIT Sloan Sports Analytics Conference.

<a id="ref-28"></a>
[28] Ian Simpson. 2021. Code Repository for Seq2Event PyTorch Implementation for Soccer. **足球Seq2Event PyTorch实现的代码库**. [https://github.com/statsonthecloud/Soccer-SEQ2Event](https://github.com/statsonthecloud/Soccer-SEQ2Event)

<a id="ref-29"></a>
[29] Karun Singh. 2021. Introducing Expected Threat (xT). **介绍预期威胁(xT)**. Retrieved December 27, 2021 from [https://karun.in/blog/expected-threat.html](https://karun.in/blog/expected-threat.html)

<a id="ref-30"></a>
[30] Danilo Sorano, Fabio Carrara, Paolo Cintia, Fabrizio Falchi, and Luca Pappalardo. 2020. Automatic Pass Annotation from Soccer Video Streams Based on Object Detection and LSTM. **基于对象检测和LSTM的足球视频流自动传球标注**. In Joint European Conference on Machine Learning and Knowledge Discovery in Databases. Springer, 475–490.

<a id="ref-31"></a>
[31] William Spearman. 2018. Beyond expected goals. **超越预期进球**. In Proceedings of the 12th MIT Sloan Sports Analytics Conference.

<a id="ref-32"></a>
[32] Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Łukasz Kaiser, and Illia Polosukhin. 2017. Attention is all you need. **注意力是你所需要的一切**. In Advances in neural information processing systems. 5998–6008.

<a id="ref-33"></a>
[33] Zhiwei Wang, Yao Ma, Zitao Liu, and Jiliang Tang. 2019. R-transformer: Recurrent neural network enhanced transformer. **R-transformer：增强型循环神经网络transformer**. (2019).

<a id="ref-34"></a>
[34] Neil Watson, Sharief Hendricks, Theodor Stewart, and Ian Durbach. 2021. Integrating machine learning and decision support in tactical decision-making in rugby union. **在橄榄球联盟战术决策中整合机器学习和决策支持**. Journal of the Operational Research Society 72, 10 (2021), 2274–2285.

<a id="ref-35"></a>
[35] Qiyun Zhang, Xuyun Zhang, Hongsheng Hu, Caizhong Li, Yinping Lin, and Rui Ma. 2021. Sports match prediction model for training and exercise using attention-based LSTM network. **使用基于注意力的LSTM网络的训练和练习体育比赛预测模型**. Digital Communications and Networks (2021).
