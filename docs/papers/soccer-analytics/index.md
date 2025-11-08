# 行动胜于进球：重视足球中的球员行动 Actions Speak Louder than Goals: Valuing Player Actions in Soccer - 2019 

<div style="text-align: center;">
    <a href="https://dl.acm.org/doi/abs/10.1145/3292500.3330758
        
        ">
        <img width="2000" height="497" alt="image" src="https://github.com/user-attachments/assets/779f1864-92ad-492e-ac6e-b76de4e32ed1" />
    </a>
</div>

## 摘要

评估足球运动员在比赛中执行的个人行动的影响是球员招募过程中的关键环节。不幸的是，大多数传统指标在解决这一任务时存在不足，它们要么仅关注射门和进球等罕见行动，要么未能考虑行动发生的上下文。本文引入了(1)一种描述球员在场上个人行动的新语言，以及(2)一个基于行动对比赛结果影响的评估框架，同时考虑了行动发生的上下文。通过汇总足球运动员的行动价值，可以量化他们对球队的进攻和防守贡献总和。我们展示了我们的方法如何考虑传统球员评估指标所忽略的相关上下文信息，并提出了一些与欧洲顶级赛事2016/2017和2017/2018赛季中球探和比赛风格特征相关的应用案例。

## 1 引言

足球运动员的行动如何影响其团队在比赛中的表现？这个问题与足球俱乐部内的多项任务相关，如球员引进、球员评估和球探工作。这对媒体和增强球迷参与度也很重要，因为球迷最喜欢的莫过于比较球员并争论为何他们最喜爱的球员比其他人更优秀。然而，客观量化足球运动员在比赛中执行的个人行动影响的任务至今仍基本未被探索。使这项任务复杂化的是足球比赛进球少且动态性强的特点。虽然大多数行动不会直接影响比分，但它们通常会产生重要的长期效果。例如，从一侧边路到另一侧的长传可能不会立即导致进球，但可以打开空间，为几个行动之后的进球机会做准备。

现有的足球行动价值评估方法主要存在三个重要局限性。首先，这些方法在很大程度上忽略了进球和射门以外的行动，因为迄今为止的大多数研究都集中在射门尝试的期望值概念上[**[1]**](#ref-1)[**[5]**](#ref-5)[**[20]**](#ref-20)[**[21]**](#ref-21)。其次，现有方法倾向于为每个行动分配固定价值，而不考虑执行该行动的环境。例如，许多基于传球的指标对待在毫无压力下防守三区的防守队员之间的传球和在对手重压下进攻三区的攻击队员之间的传球是相似的。第三，大多数方法仅考虑即时效果，而未能考虑行动在后续产生的效果。

为了填补客观量化球员表现的空白，本文提出了一个评估足球比赛中行动的新型数据驱动框架。与大多数现有工作不同，它考虑了所有类型的行动（如传球、传中、盘带、过人和射门），并考虑了每个行动发生的环境以及它们可能产生的长期效果。直观来说，行动价值反映了该行动对比分的预期影响。即，价值为+0.05的行动预期会为执行该行动的球队贡献0.05个进球，而价值为-0.05的行动预期会为对手产生0.05个进球。我们的方法符合日益增长的体育数据分析数据科学研究路线（例如[**[9]**](#ref-9)[**[19]**](#ref-19)[**[24]**](#ref-24)[**[27]**](#ref-27)）。

总而言之，本文做出以下五项贡献：
(1) 一种表示球员行动的语言；
(2) 一个评估球员行动和基于其对比赛影响评分球员的框架；
(3) 一个预测比赛任何时刻短期进球和失球概率的模型；
(4) 一些展示我们最有趣结果和见解的应用案例；
(5) 一个Python包<sup style="color: red;">1</sup>，它(a)将现有事件流数据转换为我们的语言，(b)实现我们的框架，以及(c)构建一个估计进球和失球概率的模型。

> <sup style="color: red;">1</sup> [https://github.com/ML-KULeuven/socceraction](https://github.com/ML-KULeuven/socceraction)
> 

## 2 SPADL：描述球员行动的语言

关于足球比赛的两种主要数据来源可用于评估行动：(1)事件流数据和(2)光学跟踪数据。事件流数据注释了比赛中发生的特定事件（如传球、射门和牌）的时间和位置。光学跟踪数据通过比赛中的光学跟踪系统以高频率记录球员和球的位置。多家不同公司（如Opta、Wyscout、STATS、Second Spectrum、SciSports和StatsBomb）生成这两种类型数据中的一种或两种。由于光学跟踪系统成本高昂，跟踪数据仅在富裕的联赛或俱乐部中可用，而事件流数据更广泛且廉价可得。此外，光学跟踪数据通常不在联赛间共享。因此，本文专注于事件流数据。然而，本文的贡献也可以通过一些小的扩展应用于完整的跟踪数据。从数据科学角度来看，一个关键挑战是事件流数据的性质使分析复杂化。我们首先描述当前可用的事件流数据带来的数据科学挑战，然后展示我们提出的SPADL语言如何解决这些挑战。

### 2.1 当前事件流数据带来的五个数据科学挑战

第一个挑战是事件流数据服务于多种不同目标（如向广播公司、报纸或足球俱乐部报告信息），这意味着数据不一定设计为便于数据分析。一些重要信息可能缺失（如Wyscout不记录射门的精确结束位置）。一些记录的信息可能与数据分析无关，实际上可能通过增加预处理步骤的复杂性而阻碍分析（如Wyscout将两名球员间的对抗记录为两个单独事件）。

第二个挑战是每个事件流数据供应商使用其独特的术语和定义来描述比赛中发生的事件。因此，用于分析数据的软件必须针对特定供应商定制，不经修改无法用于分析来自另一供应商的数据。

第三个挑战是供应商当前的事件流格式通常与其以前的格式保持向后兼容。一些供应商已提供数据超过十年，无法更改最初的次优设计选择。此外，供应商所注释的内容已经演变，现在包括额外事件和更详细信息。例如，Opta有四种不同的射门事件类型，取决于其结果，这使查询射门特征变得极其繁琐。

第四个挑战是大多数供应商为每种事件类型提供可选信息片段。例如，对于犯规，Opta通常会详细说明所犯犯规的确切类型。虽然有时有用，但这种动态信息使应用自动分析工具变得极其困难。

最后一个挑战是大多数机器学习算法需要固定长度的特征向量，无法处理来自例如可选信息片段偶尔存在产生的可变大小向量。因此，分析师通常必须编写复杂的事件预处理器，提取与其分析相关的特征。开发这些预处理器需要大量编程工作和对事件流格式的深入了解，但最终结果是一个专为一个特定供应商当前事件流格式量身定制的一次性脚本。

## 2.2 语言描述

基于领域知识和足球专家的反馈，我们提出SPADL（足球运动员行动描述语言）作为将现有事件流格式统一为共同词汇的尝试，以便后续数据分析。它设计为人类可解释、简单且完整，以准确定义和描述场上行动。人类可解释性允许推理场上发生的情况，并验证行动价值是否与足球专家的直觉相符。简单性降低了自动处理语言时出错的可能性。完整性使得能够表达分析行动及其完整背景所需的所有信息。

为解决各种事件流格式带来的挑战，并造福数据科学社区，我们发布了一个Python包，可自动将事件流转换为SPADL。我们的包目前支持Opta、Wyscout和StatsBomb提供的事件流。

SPADL是描述球员行动的语言，与商业供应商描述事件的格式不同。区别在于行动是事件的子集，需要球员执行行动。例如，传球事件是一个行动，而表示比赛结束的事件不是行动。我们将一场比赛表示为一系列持球行动[a1, a2, ..., am]，其中m是比赛中发生的行动总数。每个行动是九个属性的元组：

StartTime：行动的开始时间，
EndTime：行动的结束时间，
StartLoc：行动开始的(x, y)位置，
EndLoc：行动结束的(x, y)位置，
Player：执行行动的球员，
Team：球员的队伍，
ActionType：行动类型（如传球、射门、盘带），
BodyPart：球员用于行动的身体部位，
Result：行动的结果（如成功或失败）。

注意，与所有其他事件流格式不同，我们始终为每个行动存储相同的九个属性。排除可选信息片段使我们能够更容易地应用自动分析工具。

我们区分21种可能的行动类型，包括传球、角球传中、盘带、掷界外球、铲球、射门、点球、解围和守门员扑救等。这些行动类型是与领域专家合作设计的，旨在可解释且足够具体以准确描述场上发生的情况，同时又足够通用，使得类似行动具有相同类型。所有可能行动类型的列表在**附录A.1**中。

我们考虑最多四种不同的身体部位和最多六种可能的结果。可能的身体部位是脚、头、其他和无。两种最常见的结果是成功或失败，表明行动是否达到了预期结果。例如，传球到达队友或铲球成功夺回球权。其他四种可能的结果是越位传球导致越位判罚、乌龙球、黄牌和红牌。

## 3 VAEP：评估球员行动的框架

**本节介绍VAEP（通过估计概率评估行动价值）框架，用于评估足球运动员执行的行动。首先，我们展示如何使用进球和失球概率来计算客观行动价值。接下来，我们展示如何将一组行动价值转换为代表球员对其团队总进攻和防守贡献的球员评分。**

### 3.1 将进球和失球概率转换为行动价值

广义而言，足球比赛中的大多数行动都是为了(1)增加进球机会，或(2)减少失球机会。鉴于大多数行动的影响在时间上是有限的，评估行动效果的一种方法是计算它如何改变近期进球和失球的机会。我们分开处理行动对进球和失球的影响，因为这些影响可能在性质上不对称且依赖上下文。

假设对于每个比赛状态Si = [a1, ..., ai]，我们可以获得主队h和客队v在不久的将来进球和失球的概率。让Pscores(Si, h)和Pconcedes(Si, h)分别表示主队h在不久的将来进球和失球的概率。类似地，让Pscores(Si, v)和Pconcedes(Si, v)分别表示客队v在不久的将来进球和失球的概率。

对一个队伍的行动进行评估，需要评估由于行动ai将比赛从状态Si-1转变为状态Si而导致的进球和失球概率的变化。对于队伍x（x可以是主队h或客队v）进球概率的变化可以计算为：

![image.png](image%201.png)

如果行动增加了队伍x进球的概率，这个变化将是正值。我们称这个变化ΔPscores(ai, x)为行动ai对队伍x的进攻价值。类似地，队伍x失球概率的变化可以计算为：

![image.png](image%202.png)

如果行动增加了队伍x失球的概率，这个变化将是正值。然而，所有行动应该始终旨在降低失球的概率。这就是为什么我们称这个变化的否定-ΔPconcedes(ai, x)为行动ai对队伍x的防守价值。

我们结合公式1和2推导出行动的总VAEP价值。

定义1（VAEP价值）。行动的总VAEP价值是该行动的进攻价值和防守价值的总和。

![image.png](image%203.png)

鉴于我们通常对执行行动的球员所在队伍的行动价值感兴趣，我们使用V(ai)表示V(ai, xi)，其中xi是执行行动ai的球员所在的队伍。

VAEP框架提供了一种评估行动的简单方法，独立于用于描述行动的表示。该框架的优势在于它以自然的方式将主观的行动评估任务转化为预测未来事件可能性的客观任务。

## 3.2 将行动价值转换为球员评分

我们的方法为每个单独的行动分配价值。我们可以将单个行动价值聚合为多个时间粒度以及沿着几个不同维度的球员评分。球员评分可以针对任何给定的时间框架，其中最自然的包括比赛内的时间窗口、整场比赛或整个赛季。无论时间框架如何，我们以相同的方式计算球员评分。由于在场上花费更多时间提供了更多贡献机会，我们计算每90分钟比赛时间的球员评分。给定时间框架T和球员p，我们计算球员的评分为：

![image.png](image%204.png)

其中ATp是球员p在时间框架T内执行的行动集合，V(ai)按照定义1计算，m是球员在T期间比赛的分钟数。这个球员评分捕捉了球员每90分钟为其队伍贡献的平均净进球差。

此外，不是对所有行动求和，球员评分可以按行动类型计算。这允许构建球员档案，可能有助于识别不同的比赛风格。一般而言，球员评分可以沿着不同维度计算，取决于用例。

# 4 估计进球和失球概率

本节描述我们估计VAEP框架所需的进球和失球概率的方法。让goal(h)表示主队h进的球，goal(v)表示客队v进的球。那么我们的任务可以定义为：

给定：比赛状态Si = [a1, ..., ai]；
估计：主队h和客队v在不久的将来进球和失球的概率，我们表示为：
Pscores(Si, h) = P(goal(h) ∈ Fik|Si)
Pconcedes(Si, h) = P(goal(v) ∈ Fik|Si)
Pscores(Si, v) = P(goal(v) ∈ Fik|Si)
Pconcedes(Si, v) = P(goal(h) ∈ Fik|Si)
其中Fik = [ai+1, ..., ai+k]是跟随行动ai的k个行动序列，k是用户定义的参数。

因为Pconceding(Si, h) = Pscoring(Si, v)且Pscoring(Si, h) = Pconceding(Si, v)，我们只需要估计一个队伍的进球和失球概率，就可以免费获得另一个队伍的概率。我们利用这一点，只估计在比赛状态Si中拥有球权的队伍的进球和失球概率。因此，我们的任务简化为两个具有相同输入但不同标签的二元概率分类问题。

给定：比赛状态Si，其中xi是在Si期间拥有球权的队伍；
估计：(1) Pscores(Si, xi)和(2) Pconcedes(Si, xi)。

对于这两个二元分类问题，我们训练概率分类器来估计概率。原则上，任何预测概率的机器学习模型（如逻辑回归、随机森林或神经网络）都可以用于解决这些任务。然而，一个重要标准是概率估计应该是经过良好校准的[**[22]**](#ref-22)。我们使用**CatBoost[**[26]**](#ref-26)**，并在5.6.3节中经验性地证明这一选择。

**应用标准机器学习算法需要将描述整个比赛的行动序列[a1, a2, ..., am]转换为特征向量格式的样本**。因此，为每个比赛状态Si构建一个训练样本。现在我们描述如何为每个比赛状态计算标签和特征。

## 4.1 构建标签

对于估计Pscores(Si, xi)的第一个分类问题，如果在行动ai之后拥有球权的队伍在随后的k个行动中进球，我们给比赛状态Si分配一个正标签（= 1），在所有其他情况下分配一个负标签（= 0）。类似地，对于估计Pconcedes(Si, xi)的第二个分类问题，如果在行动ai之后拥有球权的队伍在随后的k个行动中失球，我们给比赛状态Si分配一个正标签（= 1），在所有其他情况下分配一个负标签（= 0）。

在这两个二元分类问题中，k是用户定义的参数，表示我们向前看多远来确定行动的效果。**在本文中，我们基于领域知识和初步实验选择k = 10**。

## 4.2 构建特征

**对于每个样本，我们不是基于整个当前比赛状态Si = [a1, ..., ai]定义特征，而是只考虑前三个行动[ai-2, ai-1, ai]**。以这种方式近似比赛状态提供了几个优势。首先，大多数机器学习技术需要样本由固定数量的特征描述。将具有不同数量行动（因此不同信息量）的比赛状态转换为这种格式必然会导致信息损失。其次，考虑一个小窗口集中注意力在当前上下文的最相关方面。要考虑的行动数量是方法的参数，经验上发现三个行动效果很好。从这三个行动中，我们定义影响近期进球概率的特征。基于SPADL表示，我们考虑三类特征。

1. SPADL特征。对于三个行动中的每一个，我们基于SPADL表示中明确包含的信息定义一组分类和实值特征。我们考虑行动类型和结果的分类特征，以及执行行动的球员使用的身体部位。类似地，我们考虑行动起始和结束位置的(x, y)坐标，以及自比赛开始以来经过的时间的实值特征。
2. 复杂特征。复杂特征结合了一个行动内和连续行动之间的信息。在每个行动内，这些特征包括(1)行动起始和结束位置到球门的距离和角度，以及(2)行动中在x和y方向上覆盖的距离。在两个连续行动之间，我们计算它们之间的距离和经过的时间，以及球权是否改变。这些特征提供了关于当前比赛速度的一些直觉。
3. 比赛上下文特征。比赛上下文特征是(1)在行动ai之后拥有球权的队伍在比赛中进球数，(2)在行动ai之后防守队伍在比赛中进球数，以及(3)在行动ai之后的进球差。我们包括这些特征是因为球队通常会根据当前比分调整比赛风格（例如，领先1-0的球队比落后0-1的球队更倾向于防守）。

# 5 实验

评估我们的框架具有挑战性，因为不存在客观的行动价值或球员评分真值。因此，我们的实验解决三个主要问题：(1)提供关于我们的框架如何运行并与其他指标比较的直觉，(2)展示围绕球员引进和特征化的应用案例，以及(3)评估我们的几个设计决策。

我们将分析重点放在英国、西班牙、德国、意大利、法国、荷兰和比利时顶级联赛的Wyscout数据上。我们将VAEP框架应用于2012/2013至2017/2018赛季进行的11565场比赛。我们只考虑联赛比赛，因此忽略所有友谊赛、杯赛和欧洲比赛。

**我们使用CatBoost算法和第4节详述的特征集训练两个分类模型，以产生进球和失球概率、行动价值和球员评分。我们在2012/2013至2015/2016赛季的数据上训练第一个模型，为2016/2017赛季产生结果。类似地，我们在2012/2013至2016/2017赛季的数据上训练第二个模型，为2017/2018赛季产生结果。**

## 5.1 行动价值背后的直觉

**图1**展示了我们的框架如何工作，通过可视化2017年12月23日巴塞罗那客场对阵皇家马德里比赛第93分钟的进球前的行动及其相应价值。

<div style="text-align: center;">
    <img width="535" height="757" alt="图1" src="https://github.com/user-attachments/assets/d21bcf05-a25b-4e68-9673-bb8259258e4a" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
图1：2017年12月23日巴塞罗那3-0战胜皇家马德里比赛中最后一粒进球前的进攻过程。
</p>

这次进攻包括六个行动，始于塞尔吉奥·布斯克茨向右边路传球(1)，获得一个中性行动价值0.00，因为它既没有改善也没有恶化局势。随后梅西回传给布斯克茨的传球(2)因为将球向后移动到比之前不太有利的位置而被罚以-0.01的行动价值。布斯克茨给梅西的出色直塞球(3)最终将球移近球门，获得+0.01的行动价值。梅西接球并过掉一名皇马防守队员进入禁区(4)，因为显著提高了进球几率从0.03到0.08而获得+0.05的行动价值。

梅西的下一个行动展示了他的天才，将球向后传离拥挤的小禁区(5)。我们的框架给这次传球+0.09的价值，因为它将进球几率从0.08提高到0.17。这个行动显示了我们框架的力量，它奖励梅西将球移离对手的球门。以纯数据驱动的方式，我们的框架识别出这个行动在当时环境下是一个好选择。据我们所知，没有其他使用事件流数据评估行动的方法会奖励梅西这个行动。最后，阿莱克斯·比达尔射门得分(6)。因为将0.17的进球机会转化为进球，我们的框架奖励比达尔+0.83的行动价值。如果比达尔射失，他将被罚以-0.17的行动价值。

## 5.2 将我们的VAEP球员评分与传统球员表现指标比较

目前，球员的进攻贡献通常通过计算进球和助攻来量化，因为这些事件直接影响比分。2因此，我们将我们的VAEP球员评分与以下三个基线指标进行比较：每90分钟进球数、每90分钟助攻数和每90分钟进球+助攻数。我们通过产生2017/2018英超赛季每个指标的前10名列表来研究这些指标识别顶级球员的能力，如**表1**所示。每90分钟进球数前10名由专注于完成而非创造得分机会的前锋组成。类似地，每90分钟助攻数前10名主要由专门为队友创造机会的中场组成。此外，每90分钟进球+助攻数的排名旨在在这两种原型之间取得平衡。

<img width="1207" height="3072" alt="image 6" src="https://github.com/user-attachments/assets/7a7ee26c-7bb9-434a-926c-cfb777ce88db" />
表1：2017/2018英超赛季至少踢满900分钟的前10名球员，按照(g)进球数、(a)助攻数、(g+a)进球+助攻数和(vaep)我们的VAEP球员评分排名。Rm表示该球员在305名球员中按指标m的排名。市场价值表示根据Transfermarkt.de网站2019年2月1日的球员市场价值。(a) 按每90分钟进球数(g/90)排名的前10名球员；(b) 按每90分钟助攻数(a/90)排名的前10名球员；(c) 按每90分钟进球+助攻数(g+a/90)排名的前10名球员；(d) 按我们的VAEP球员评分排名的前10名球员

然而，我们的框架也识别出在这些传统指标上排名不高的有影响力球员。首先，我们的VAEP框架产生的前10名列表包括凯文·德布劳内（曼城）、阿扎尔（切尔西）和里亚德·马赫雷斯（莱斯特城）。虽然他们被认为是英超球星，但他们没有出现在任何传统的前10名中。其次，我们前10名列表中球员的市场总价值（11.1亿欧元）明显高于进球（8.62亿欧元）、助攻（7.6亿欧元）和进球+助攻（9.47亿欧元）的前10名球员。

这些观察结果表明，我们的VAEP框架比传统球员表现指标更好地捕捉球员对其队伍表现的贡献。

## 5.3 识别有前途的年轻球员和小联赛人才

英格兰和西班牙联赛是最强大和最富有的联赛。因此，年轻球员很难获得上场时间，这迫使俱乐部从法国、荷兰和比利时等较小的联赛签下有前途的年轻人。通常，英格兰和西班牙俱乐部从这些联赛而非直接竞争对手那里获取有前途的年轻人更容易且尤其更便宜。因此，我们单独研究了2017/2018赛季排名最高的年轻人才（即1997年1月1日后出生且至少踢了900分钟的球员）在英格兰和西班牙联赛（**表2a**），以及法国、荷兰和比利时联赛（**表2b**）中的情况。

<img width="781" height="745" alt="image 7" src="https://github.com/user-attachments/assets/684ba3af-847a-4a23-acf6-9508770757aa" />
表2：在2017/2018赛季按照我们的VAEP球员评分排名的1997年1月1日后出生的前5名球员，分别在(a)更具竞争力的英格兰和西班牙联赛，以及(b)较小的法国、荷兰和比利时联赛。(a) 英格兰和西班牙联赛中的年轻人才。；(b) 法国、荷兰和比利时联赛中的年轻人才。

马库斯·拉什福德，3在2019年1月与皇家马德里1.1亿欧元的转会联系，以及乌斯曼·登贝莱，在2017年8月以1.2亿欧元的转会费加盟巴塞罗那，是**表2a**中最著名的球员。相比之下，排名第四但知名度较低的乔恩乔·肯尼的估计市场价值远低于这两位球员，原因有二。首先，肯尼是一名防守型球员，这类球员通常被俱乐部和球迷估值较低。其次，肯尼效力于中游俱乐部埃弗顿，那里只有少数世界级球员。尽管如此，我们的球员评分表明他的估值应远高于目前500万欧元的估计市场价值。

> 2 [https://www.squawka.com/en/news/every-player-with-10-goals-and-10-assists-ineuropes-top-five-leagues-this-season-ranked-by-contribution-per-90/1031863](https://www.squawka.com/en/news/every-player-with-10-goals-and-10-assists-ineuropes-top-five-leagues-this-season-ranked-by-contribution-per-90/1031863)  
3 [https://www.thesun.co.uk/sport/football/8318008/real-madrid-marcus-rashfordtransfer-man-utd/](https://www.thesun.co.uk/sport/football/8318008/real-madrid-marcus-rashfordtransfer-man-utd/)
> 

戴维·内雷斯位居**表2b**榜首。2017年夏天，当阿贾克斯以1500万欧元引进这位边锋时，他成为荷兰联赛第四昂贵的转入球员。他现在是顶级俱乐部利物浦、切尔西和阿森纳的转会目标，这些俱乐部都希望在2019年夏天签下他。排名第二的梅森·芒特从切尔西租借到荷兰球队维特斯一个赛季，并获得了维特斯年度最佳球员奖。排名第四的基利安·姆巴佩在2018年世界杯上获得了最佳年轻球员奖，而马尔科姆（2018年夏天，转会费4100万欧元）和弗伦基·德容（2019年夏天，转会费7500万欧元）都已与巴塞罗那签约。
**表2a**和**表2b**展示了我们框架作为球探有用工具的能力。只要提供所需的事件流数据，我们的框架可以为世界上每个联赛（例如，二级联赛或北美、南美和亚洲的联赛）生成排名。

## 5.4 特征化比赛风格

俱乐部在招募过程中越来越考虑球员风格，以识别最适合其团队首选比赛风格的球员（例如，短传和高位防守与长传和防守性打法）。目前，球探通常被要求用肉眼判断比赛风格。然而，这些球探的时间往往是限制资源，这使得考虑全部候选补强球员变得困难。因此，评估球员执行不同类型行动能力的指标可以帮助选择值得额外关注的相关球员集合。

使用我们的VAEP框架，解决这一任务简化为计算每90分钟每种行动类型的球员评分。

作为具体应用案例，考虑巴塞罗那2017年夏天试图通过引进多特蒙德的乌斯曼·登贝莱和利物浦的菲利普·库蒂尼奥来弥补内马尔离队的损失。**图2a**比较了登贝莱、库蒂尼奥和内马尔四种行动类型的每90分钟总评分。根据我们的指标，登贝莱和库蒂尼奥的传球都获得比内马尔更高的价值，而内马尔是更优秀的盘带者。从风格角度看，这一细分表明登贝莱和库蒂尼奥都是合理的目标，因为没有多少球员能接近复制内马尔标志性的盘带技巧。登贝莱和库蒂尼奥都是不错的盘带者，并且是比内马尔更好的传球手。此外，登贝莱在传中方面优于内马尔，而库蒂尼奥在射门方面优于他。

类似地，皇家马德里在2018年夏天失去了他们历史最佳射手克里斯蒂亚诺·罗纳尔多。这家陷入困境的俱乐部似乎迫切需要一位合适的替代者。曼联的马库斯·拉什福德和切尔西的阿扎尔都与转会到皇马有联系。然而，**图2b**显示，两人都无法接近复制罗纳尔多令人难以置信的终结技术。此外，罗纳尔多展示的每90分钟总射门价值高于拉什福德和阿扎尔的总和。虽然阿扎尔在各方面都优于拉什福德，但拉什福德在风格上更接近罗纳尔多，因为两人在传球和盘带方面评分相似。如果皇家马德里想坚持他们目前的比赛风格，我们的分析表明21岁的拉什福德会是更好的选择。然而，如果他们的目标是立即增强球队实力，那么28岁的阿扎尔将是首选，因为无论其特定比赛风格如何，他都是一名更好的球员。

![图2：不同类型行动每90分钟的总贡献概览：(a) 2016/2017赛季内马尔、乌斯曼·登贝莱和菲利普·库蒂尼奥的比较，以及(b) 2017/2018赛季克里斯蒂亚诺·罗纳尔多、马库斯·拉什福德和阿扎尔的比较。(a) 内马尔替代球员的比赛风格；(b) 罗纳尔多替代球员的比赛风格](image%208.png)

图2：不同类型行动每90分钟的总贡献概览：(a) 2016/2017赛季内马尔、乌斯曼·登贝莱和菲利普·库蒂尼奥的比较，以及(b) 2017/2018赛季克里斯蒂亚诺·罗纳尔多、马库斯·拉什福德和阿扎尔的比较。(a) 内马尔替代球员的比赛风格；(b) 罗纳尔多替代球员的比赛风格

## 5.5 权衡行动质量和数量

行动质量和数量之间存在自然张力。如果一名球员执行大量行动，那么每个行动拥有高价值就更难。**图3a**显示了在2017/2018赛季西班牙和英格兰联赛中至少踢满900分钟的球员平均每90分钟执行的行动数量（数量）和这些行动的平均价值（质量）。灰色虚线等值线显示了排名第一的梅西与其他人在VAEP评分上的差距。等值线是弯曲的，因为球员的评分是通过将每个行动的平均价值（x轴）和平均行动数量（y轴）相乘而获得的。如等值线和更传统的统计数据所示，4梅西显然是独一无二的。

> 4 [https://fivethirtyeight.com/features/lionel-messi-is-impossible/](https://fivethirtyeight.com/features/lionel-messi-is-impossible/)
> 

放大**图3a**，**图3b**显示了2017/2018英超赛季的前10名球员。前锋哈里·凯恩和穆罕默德·萨拉赫执行的行动数量相对较少，但他们的行动平均价值很高。中场球员凯文·德布劳内和保罗·博格巴执行更多行动，尽管每个行动的平均价值较低。菲利普·库蒂尼奥、阿扎尔、里亚德·马赫雷斯、安东尼·马夏尔、拉希姆·斯特林和孙兴慜介于这两种原型之间，在行动质量和数量之间找到了一个甜蜜点。

同样，**图3c**显示了西班牙联赛的前10名球员。我们观察到与英格兰联赛相同的原型。前锋克里斯蒂亚诺·罗纳尔多、安托万·格列兹曼、加雷斯·贝尔、埃尼斯·巴尔迪、伊亚戈·阿斯帕斯和塞德里克·巴坎布执行少量高价值行动。皇家马德里中场托尼·克罗斯和伊斯科执行更多价值较低的行动。菲利普·库蒂尼奥在2018年1月从利物浦转会至巴塞罗那后同时出现在两张图中，再次在行动质量和数量之间找到了甜蜜点。梅西在同时拥有高行动质量和数量方面是一个异常值。

![图3：2017/2018赛季在西班牙或英格兰联赛中至少踢满900分钟的球员散点图。这些图对比了球员每90分钟执行的平均行动数量与其行动的平均价值。如(a)和(c)中的灰色虚线等值线所示，梅西明显是独一无二的。(a) 所有球员；(b) 英格兰联赛前10名球员；(c) 西班牙联赛前10名球员](image%209.png)

图3：2017/2018赛季在西班牙或英格兰联赛中至少踢满900分钟的球员散点图。这些图对比了球员每90分钟执行的平均行动数量与其行动的平均价值。如(a)和(c)中的灰色虚线等值线所示，梅西明显是独一无二的。(a) 所有球员；(b) 英格兰联赛前10名球员；(c) 西班牙联赛前10名球员

## 5.6 评估设计选择

数据科学中的一个常见挑战是评估系统性能。虽然定义诸如为行动分配价值等高级任务可能很容易，但评估解决方案的一个被低估的方面是通常不存在真值，因此无法使用准确率、精确率和召回率等标准评估指标。因此，评估系统的唯一方法是评估其组成部分。在我们的案例中，我们通过评估可获得真值标签的底层进球和失球概率来评估行动价值。

### 5.6.1 评估方法

为了生成进球和失球概率，我们使用第4节描述的特征通过CatBoost算法训练分类模型。我们通过将我们的方法性能与使用不同特征集或不同算法的替代方法进行比较来评估这些设计选择。

与我们的主要方法类似，我们为每种替代方法训练两个分类模型：我们在2012/2013至2015/2016赛季的数据上训练第一个模型，为2016/2017赛季产生结果，并在2012/2013至2016/2017赛季的数据上训练第二个模型，为2017/2018赛季产生结果。

## 5.6.2 特征集的选择

大多数分析足球事件数据的现有模型只使用位置和行动类型数据[1, 8]。为了评估我们全面特征集（第4节详述）的有用性，我们将其与四个基线特征集进行比较：无特征、5位置、行动类型和位置+行动类型（**表3**）。

> 5 *The no features baseline always predicts the mean class probability, i.e., if our data  set contains 1.5% positive examples, we always predict 0.015.*
> 

我们使用CatBoost算法评估每个特征集。对于估计Pscores和Pconcedes，我们的特征集在两个评估指标上都优于基线特征集。这一结果表明，我们的特征捕捉了基线特征集中缺失的重要比赛状态上下文。

![表3：使用Brier分数和ROC AUC评估的不同设计选择对得分和让步概率的影响。对于Brier分数，数值越低越好；而对于ROC AUC，数值越高越好。](image%2010.png)

表3：使用Brier分数和ROC AUC评估的不同设计选择对得分和让步概率的影响。对于Brier分数，数值越低越好；而对于ROC AUC，数值越高越好。

## 5.6.3 学习算法的选择

数据科学项目中最流行的学习算法选择是逻辑回归[**[25]**](#ref-25)、随机森林[**[25]**](#ref-25)，以及最近的XGBoost[**[7]**](#ref-7)和CatBoost[**[26]**](#ref-26)。梯度提升方法在具有异构特征、噪声数据和复杂依赖关系的各种学习问题中有着成功的记录。表3比较了使用第4节特征的四种学习算法的性能。CatBoost在所有情况下表现最佳，XGBoost紧随其后。这一微弱优势可归因于CatBoost相比XGBoost更简单的独热编码，对分类特征的智能处理。

## 5.7 剩余挑战的讨论

我们VAEP框架的一个局限性是我们只评估持球行动。也就是说，该模型只评估带球行动，而防守常常更多关于通过巧妙的站位和预判来阻止对手获得球权。

另一个挑战是很难准确地跨联赛比较球员，因为在次级联赛（如法国、荷兰和比利时）比在更强的联赛（如英格兰和西班牙）更容易执行高价值行动。这在5.3节中可以清楚地观察到，次级联赛的年轻才俊获得的评分高于英格兰和西班牙联赛的球员。

同样，即使在同一联赛中比较不同俱乐部的球员也可能很困难，因为在拥有强大队友的顶级俱乐部比在拥有较弱队友的中游俱乐部更容易执行有价值的行动。

在现实世界中部署我们框架的最后一个挑战是建立对评分的信任，因为传统球探不熟悉我们评估足球运动员的方式。此外，我们的评分比每90分钟进球数等传统指标稍微不那么直观，这使得分析能力较弱的球探更难理解我们的评分到底衡量什么。

# 6 相关工作

虽然评估足球中的球员行动是一项重要任务，但由于足球的动态和低进球性质带来的挑战，它几乎未被探索。Nørstebø等人[**[23]**](#ref-23)、Bransen等人[**[2]**](#ref-2)和Fernández等人[**[11]**](#ref-11)对足球，Routley和Schulte[**[27]**](#ref-27)以及Liu和Schulte[**[19]**](#ref-19)对冰球，以及Cervone等人[**[6]**](#ref-6)对篮球的方法最接近我们的框架。这些方法中的大多数通过将比赛建模为马尔可夫博弈[**[18]**](#ref-18)来解决评估个人行动的任务。

与Nørstebø等人[**[23]**](#ref-23)和Routley与Schulte[**[27]**](#ref-27)将球场划分为固定数量区域的方法不同，我们的方法模拟每个行动的精确位置。与Cervone等人[**[6]**](#ref-6)仅评估三种持球行动的方法不同，我们的方法考虑比赛中任何相关的持球行动。然而，我们对球员行动、比赛状态和行动价值的定义与这些工作以及早期关于足球[**[16]**](#ref-16)[**[28]**](#ref-28)、美式足球[**[13]**](#ref-13)和棒球[**[29]**](#ref-29)的研究使用的定义相似。

关于足球的大多数相关工作要么专注于有限数量的球员行动类型，如传球和射门，要么未能考虑行动发生的环境。Decroos等人[**[10]**](#ref-10)、Knutson[**[17]**](#ref-17)和Gregory[**[14]**](#ref-14)解决了评估导致射门尝试的行动的任务，而Bransen等人[**[4]**](#ref-4)、Bransen和Van Haaren[**[3]**](#ref-3)以及Gyarmati和Stanojevic[**[15]**](#ref-15)解决了评估个别传球的任务。前者通过仅考虑有限的上下文信息来简单地将价值分配给个别行动，而后者仅限于单一类型的行动。

此外，这项工作也与预期进球模型相关，后者估计射门尝试导致进球的概率[**[1]**](#ref-1)[**[5]**](#ref-5)[**[8]**](#ref-8)[**[20]**](#ref-20)[**[21]**](#ref-21)。在我们的VAEP框架中，计算射门尝试的预期进球价值归结为估计射门尝试前的比赛状态价值。

# 7 结论

本文介绍了SPADL，一种用于表示事件流数据的语言，其设计目标是促进数据分析，以及VAEP，一个为足球比赛中每个个体球员行动分配价值的框架。VAEP相对于大多数现有工作的优势在于它(1)评估所有行动类型（如传球、传中、盘带和射门），(2)基于比赛上下文进行评估，以及(3)推理行动对后续行动的可能影响。直观上，增加球队进球机会的球员行动获得正值，而减少球队进球机会的行动获得负值。

# A 附录：可重复性说明

## A.1 SPADL动作类型

**表4**提供了SPADL表示法中21种动作类型的概述及其描述。

![表4：SPADL中21种动作类型的概述及其描述。"成功？"列指定了动作被视为成功所需满足的条件，而"特殊"列列出了其他可能的结果值。](image%2011.png)

表4：SPADL中21种动作类型的概述及其描述。"成功？"列指定了动作被视为成功所需满足的条件，而"特殊"列列出了其他可能的结果值。

## A.2 数据描述

我们在英国、西班牙、德国、意大利、法国、荷兰和比利时顶级联赛的Wyscout数据上进行了实验。我们考虑了2012/2013至2017/2018赛季的11,565场比赛。将Wyscout数据转换为我们的SPADL表示后，每场比赛平均包含约1250个动作。如**图4**所示，我们数据集中最常见的动作类型是传球(64.63%)、盘带(8.69%)和拦截(5.01%)。

![图4：Wyscout数据转换为SPADL表示后各动作类型的频率。](image%2012.png)

图4：Wyscout数据转换为SPADL表示后各动作类型的频率。

## A.3 实验设置和实现

本文的所有实验均在Python中进行。我们评估了四种流行学习算法的性能：

- **逻辑回归**：我们使用了scikit-learn⁶ Python包中的实现。我们使用了L2正则化惩罚，并使用L-BFGS作为优化问题的求解器。
- **随机森林**：我们使用了scikit-learn⁷ Python包中的实现。我们使用40个并行线程训练了100棵树的森林。
- **XGBoost**：我们使用了官方Python实现⁸。我们使用40个并行线程训练了100棵最大深度为3的树，学习率为0.1。
- **CatBoost**：我们使用了Yandex的官方Python实现⁹。我们将所有参数设置为默认值，除了并行线程数，我们设置为40。

> ⁶ [https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html)  ⁷ [https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)  
⁸ [https://xgboost.ai](https://xgboost.ai/)  
⁹ [https://tech.yandex.com/catboost/](https://tech.yandex.com/catboost/)
> 

我们在运行Ubuntu 16.04、配备128GB RAM和两个Xeon(R) CPU E5-2630 v4 @ 2.20GHz类型CPU的计算服务器上训练和评估了所有模型，提供最多20个核心和40个线程。**表5**中提供了每个学习算法在每个任务上的运行时间。

![表5：每个任务的各学习算法运行时间。两个训练集，分别为2012/2013至2015/2016赛季和2012/13至2016/2017赛季，分别包含8,518,378和11,438,956个动作。两个评估集，分别为2016/2017赛季和2017/2018赛季，分别包含2,920,578和2,988,847个动作。](image%2013.png)

表5：每个任务的各学习算法运行时间。两个训练集，分别为2012/2013至2015/2016赛季和2012/13至2016/2017赛季，分别包含8,518,378和11,438,956个动作。两个评估集，分别为2016/2017赛季和2017/2018赛季，分别包含2,920,578和2,988,847个动作。

## 参考文献
<a id="ref-1"></a>
[1] Daniel Altman. 2015. Beyond Shots: A New Approach to Quantifying Scoring Opportunities. (2015). [http://northyardanalytics.com/Dan-Altman-NYA-OptaProForum-2015.pdf](http://northyardanalytics.com/Dan-Altman-NYA-OptaProForum-2015.pdf) OptaPro Analytics Forum.

<a id="ref-2"></a>
[2] Lotte Bransen, Pieter Robberechts, Jan Van Haaren, and Jesse Davis. 2019. Choke or Shine? Quantifying Soccer Players' Abilities to Perform Under Mental Pressure. In MIT Sloan Sports Analytics Conference.

<a id="ref-3"></a>
[3] Lotte Bransen and Jan Van Haaren. 2018. Measuring Football Players' On-the-Ball Contributions from Passes During Games. In ECML/PKDD 2018 Workshop on Machine Learning and Data Mining for Sports Analytics.

<a id="ref-4"></a>
[4] Lotte Bransen, Jan Van Haaren, and Michel van de Velden. 2019. Measuring Soccer Players' Contributions to Chance Creation by Valuing Their Passes. Journal of Quantitative Analysis in Sports (2019).

<a id="ref-5"></a>
[5] Michael Caley. 2015. Premier League Projections and New Expected Goals. (2015). [https://cartilagefreecaptain.sbnation.com/2015/10/19/9295905/premierleague-projections-and-new-expected-goals](https://cartilagefreecaptain.sbnation.com/2015/10/19/9295905/premierleague-projections-and-new-expected-goals) Cartilage Free Captain.

<a id="ref-6"></a>
[6] Dan Cervone, Alexander D'Amour, Luke Bornn, and Kirk Goldsberry. 2014. POINTWISE: Predicting Points and Valuing Decisions in Real Time with NBA Optical Tracking Data. In MIT Sloan Sports Analytics Conference.

<a id="ref-7"></a>
[7] Tianqi Chen and Carlos Guestrin. 2016. XGBoost: A Scalable Tree Boosting System. In Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining. ACM, 785–794.

<a id="ref-8"></a>
[8] Tom Decroos, Vladimir Dzyuba, Jan Van Haaren, and Jesse Davis. 2017. Predicting Soccer Highlights from Spatio-Temporal Match Event Streams. In Proceedings of the Thirty-First AAAI Conference on Artificial Intelligence. 1302–1308.

<a id="ref-9"></a>
[9] Tom Decroos, Jan Van Haaren, and Jesse Davis. 2018. Automatic Discovery of Tactics in Spatio-Temporal Soccer Match Data. In Proceedings of the 24th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining. ACM.

<a id="ref-10"></a>
[10] Tom Decroos, Jan Van Haaren, Vladimir Dzyuba, and Jesse Davis. 2017. STARSS: A Spatio-temporal Action Rating System for Soccer. In ECML/PKDD 2017 Workshop on Machine Learning and Data Mining for Sports Analytics.

<a id="ref-11"></a>
[11] Javier Fernández, Luke Bornn, and Dan Cervone. 2019. Decomposing the Immeasurable Sport: A Deep Learning Expected Possession Value Framework for Soccer. In MIT Sloan Sports Analytics Conference.

<a id="ref-12"></a>
[12] César Ferri, José Hernández-Orallo, and R. Modroiu. 2009. An Experimental Comparison of Performance Measures for Classification. Pattern Recognition Letters 30, 1 (2009), 27–38.

<a id="ref-13"></a>
[13] Keith Goldner. 2012. A Markov Model of Football: Using Stochastic Processes to Model a Football Drive. Journal of Quantitative Analysis in Sports 8, 1 (2012).

<a id="ref-14"></a>
[14] Sam Gregory. 2017. How We Assign Credit in Football. (2017). [http://www.optasportspro.com/about/optapro-blog/posts/2017/blog-howwe-assign-credit-in-football/](http://www.optasportspro.com/about/optapro-blog/posts/2017/blog-howwe-assign-credit-in-football/) OptaPro Blog.

<a id="ref-15"></a>
[15] László Gyarmati and Rade Stanojevic. 2016. QPass: A Merit-based Evaluation of Soccer Passes. In KDD 2016 Workshop on Large-Scale Sports Analytics.

<a id="ref-16"></a>
[16] Nobuyoshi Hirotsu, Michael Wright, et al. 2002. Using a Markov Process Model of an Association Football Match to Determine the Optimal Timing of Substitution and Tactical Decisions. Journal of the Operational Research Society 53, 1 (2002).

<a id="ref-17"></a>
[17] Ted Knutson. 2017. Introducing xGChain. (2017). [http://www.statsbombservices.com/introducing-xgchain](http://www.statsbombservices.com/introducing-xgchain) StatsBomb IQ Services.

<a id="ref-18"></a>
[18] Michael Littman. 1994. Markov Games as a Framework for Multi-Agent Reinforcement Learning. In Proceedings of the International Conference on Machine Learning.

<a id="ref-19"></a>
[19] Guiliang Liu and Oliver Schulte. 2018. Deep Reinforcement Learning in Ice Hockey for Context-Aware Player Evaluation. In Proceedings of the Twenty-Seventh International Joint Conference on Artificial Intelligence. 3442–3448.

<a id="ref-20"></a>
[20] Patrick Lucey, Alina Bialkowski, Mathew Monfort, Peter Carr, and Iain Matthews. 2014. Quality vs. Quantity: Improved Shot Prediction in Soccer Using Strategic Features from Spatiotemporal Data. In MIT Sloan Sports Analytics Conference.

<a id="ref-21"></a>
[21] Nils Mackay. [n.d.]. Predicting Goal Probabilities for Possessions in Football. Master's thesis. Vrije Universiteit Amsterdam.

<a id="ref-22"></a>
[22] Alexandru Niculescu-Mizil and Rich Caruana. 2005. Predicting Good Probabilities with Supervised Learning. In Proceedings of the Twenty-Second International Conference on Machine Learning. 625–632.

<a id="ref-23"></a>
[23] Olav Nørstebø, Vegard Rødseth Bjertnes, and Eirik Vabo. 2016. Valuing Individual Player Involvements in Norwegian Association Football. Master's thesis. Norwegian University of Science and Technology.

<a id="ref-24"></a>
[24] Luca Pappalardo, Paolo Cintia, et al. 2018. PlayeRank: data-driven performance evaluation and player ranking in soccer via a machine learning approach. arXiv preprint arXiv:1802.04987 (2018).

<a id="ref-25"></a>
[25] Fabian Pedregosa, Gaël Varoquaux, et al. 2011. scikit-learn: Machine Learning in Python. Journal of Machine Learning Research 12, Oct (2011), 2825–2830.

<a id="ref-26"></a>
[26] Liudmila Prokhorenkova, Gleb Gusev, Aleksandr Vorobev, Anna Veronika Dorogush, and Andrey Gulin. 2018. CatBoost: Unbiased Boosting with Categorical Features. In Advances in Neural Information Processing Systems. 6639–6649.

<a id="ref-27"></a>
[27] Kurt Routley and Oliver Schulte. 2015. A Markov Game Model for Valuing Player Actions in Ice Hockey. In Proceedings of the Thirty-First Conference on Uncertainty in Artificial Intelligence. 782–791.

<a id="ref-28"></a>
[28] Sarah Rudd. 2011. A Framework for Tactical Analysis and Individual Offensive Production Assessment in Soccer Using Markov Chains. In New England Symposium on Statistics in Sports. [http://nessis.org/nessis11/rudd.pdf](http://nessis.org/nessis11/rudd.pdf)

[29] Tom Tango, Mitchel Lichtman, and Andrew Dolphin. 2007. The Book: Playing the Percentages in Baseball. Potomac Books, Inc.
