# 深度足球分析：学习用于评估足球运动员的动作值函数（2020 ）
<div style="text-align: center;">
    <a href="https://doi.org/10.1007/s10618-020-00705-9">
        <img width="2000" height="497" alt="image" src="https://github.com/user-attachments/assets/9dacc5c3-be45-4400-81d5-d465b8ef7c3c" />
    </a>
</div>

## 摘要

足球比赛场地广阔、参赛球员众多、球员换位有限且进球稀疏，可以说是所有主要团队运动项目中分析难度最大的项目。在本研究中,我们开发了一种新方法,用于从逐场比赛事件数据中评估各类足球动作。我们的方法利用深度强化学习(Deep Reinforcement Learning, DRL)模型来学习动作价值Q函数。据我们所知,这是首个基于深度强化学习方法、针对足球动作全集的动作价值函数。我们的神经网络架构通过两个堆叠的LSTM塔来拟合连续的比赛情境信号和一次进攻中的序列特征,主队和客队各使用一个独立的LSTM塔。为验证模型性能,我们展示了所学Q函数的时间和空间投影,并进行了校准实验以研究不同比赛情境下的数据拟合效果。我们提出的新颖足球进球影响指标(Goal Impact Metric, GIM)应用了所学Q函数的数值,通过汇总球员在一个赛季所有比赛中各项动作的影响值来衡量其整体表现。为解释这些影响值,我们构建了一个模仿回归树,以找出对这些数值影响最大的比赛特征。作为GIM指标的应用示例,我们进行了一项案例研究,对英格兰足球冠军联赛的球员进行排名。实证评估表明,GIM是一个时间稳定的指标,且其与足球成功标准衡量指标的相关性高于其他最先进足球指标的相关性。

## 1 引言：行动价值与球员评估  
体育统计的一项重要任务是球员评估，这有助于深入理解球员的表现[（Schumaker等，2010）](#ref-28)。绩效评估对球队管理和球迷参与至关重要。例如，梦幻联赛允许球迷根据球员的技术和表现来选拔或组建心仪的球队。随着高频追踪系统和目标检测算法的出现，职业体育中球员移动数据日益丰富，这为利用大规模机器学习建模复杂赛事动态并评估球员表现提供了更多可能。近年来已提出多种评估指标，最常用的方法是通过量化球员行动的价值进行评估（[McHale等，2012](#ref-19)；[Decroos等，2019](#ref-9)）。  

传统体育评估方法面临两大问题：（1）许多球员评估指标（如预期进球）仅关注对进球有直接影响的动作（例如射门），而忽略了具有显著长期效应的其他行为。这一局限在得分稀疏的赛事中更为突出，例如足球比赛常以零进球或一球告终。（2）传统方法倾向于为动作分配固定价值，忽略比赛情境差异。为应对这些问题，[Routley和Schulte（2015）](#ref-24)构建了冰球比赛的马尔可夫模型，通过计算每个动作的Q值来捕捉比赛情境。Q值用于估算在特定比赛情境下，执行某一动作后球队得分的概率。  

足球被公认为主要团队运动中分析难度最高的项目[（Bornn等，2018）](#ref-4)。其比赛情境比冰球更为复杂：场上球员更多（22人）、场地更大（长350英尺，宽150英尺）、比赛时间更长（90分钟），这些因素导致各队形成复杂的时空分布模式。本文应用深度强化学习（DRL）从足球赛事数据中学习动作价值Q函数，提出双塔堆叠LSTM结构分别捕捉主客队的动态特征。与强化学习中旨在学习最优策略的传统控制问题不同，我们在被动学习（同策略）设定下解决预测问题。基于习得的Q函数，我们提出两种衡量球员表现的指标并从理论上证明其一致性：  

首先，目标影响指标（GIM）通过聚合球员所有行动的影响进行排名，其中行动影响由该动作引发的连续Q值变化量定义。在与四种对比指标的实证比较中，GIM与多数标准成功度量指标相关性最高。基于赛季初赛样本的推广实验表明，GIM对赛季总进球和助攻数具有最佳预测能力。  

其次，作为动作价值法的替代方案，可将球员与随机或联赛平均水平球员进行比较[（如Cervone等，2014）](#ref-7)，通过对比球员上场与替换为基准球员时期望胜率差异进行评估。我们基于逐场数据引入Q值高于平均替代值的新指标。主要定理证明：球员的Q值高于平均替代值与其总行动影响值等价。这表明DRL框架统一了两种基础球员评估方法，平均替代法的合理性佐证了总行动价值指标（GIM）的有效性。  

为计算所有球员的动作价值，我们整合多联赛数据构建包含超450万行动事件的大型数据集。该数据集使模型能学习动作价值的通用估计。但由于特定联赛情境与通用足球比赛存在差异，需针对不同联赛调整评估标准。为解决跨联赛泛化与特定联赛专化之间的平衡问题，我们提出微调方法：以通用模型为初始参数，再使用特定联赛数据训练模型。以英格兰足球联赛（EFL）锦标赛数据为例，微调显著提升了模型拟合性能及该联赛球员评估效果。
  
本文主要贡献包括：
    
1. 首次构建面向足球逐场事件数据的神经马尔可夫博弈模型，利用深度强化学习估计情境感知Q函数；
    
2. 提出新颖的双塔神经网络架构，分别捕捉足球比赛中主客队的时空复杂性；
    
3. 提出微调方法，从跨联赛海量数据中学习通用动作价值模型，同时捕捉特定联赛统计规律（该方法在深度体育分析中属首创）；
    
4. 基于Q函数提出两种足球表现新指标：目标影响指标与Q值高于平均替代值。据我们所知，后者是首例面向足球逐场数据的替代基准指标，并证明二者数值等价，在强化学习框架下统一了球员评估的两种基础范式。


## 2 相关工作

### 2.1 足球运动员评估

[Albert等人 (2017)](#ref-1)的手册提供了若干关于球员评估的最新综述文章。正负值(Plus-Minus)是一种常用的仅基于进球的球员评估指标,它量化球员在场时对本队进球机会的影响。基础版本规定:当球员在场且本队进球时,该球员获得+1;若对方进球则为-1。一些近期研究对基础正负值指标进行了改进,通过根据预期获胜概率、比赛时间和比赛频率对进球进行重要性加权[(Schultze和Wellbrock 2018)](#ref-27),或运用机器学习和生存模型估计预期进球数和预期得分来评估球员的整体防守和进攻影响力[(Kharrat等人 2019)](#ref-14)。预期进球数(Expected Goals, XG)利用射门信息,通过给定射门特征(如射门角度)下进球的概率来量化射门价值。球员按其总预期进球数排名[(Ali 2011)](#ref-2)。许多近期研究采用类似方法研究传球而非射门,通过传球对预期得分机会的影响来量化球员传球质量。传球是足球比赛中最频繁的动作之一。[Brooks等人(2016)](#ref-6)将每次传球的价值衡量为导致成功射门的估计概率。[Bransen和Van Haaren(2018)](#ref-5)将其价值衡量为传球前后进球概率的差值。这些评分方法的缺陷在于仅评估单一类型的动作,未能对球员的整体表现建模。

若干近期研究通过评估球员的所有动作来为其评分。预期控球价值(Expected Possession Value, EPV)[(Cervone等人 2016)](#ref-8)通过估计一次控球的预期得分来评估篮球比赛中一次控球内的所有动作。遵循这一框架,<span style="color: red;">[Fernández等人(2019)](#ref-11)基于完整分辨率的时空数据构建了一个深度模型,计算比赛期间所有动作的EPV值。他们研究了不同比赛情况下单个足球运动员的动作影响。**他们的方法需要跟踪数据**,这假设所有球员完全可观测</span>。包括我们在内的许多其他逐场比赛数据集仅提供比赛情境的部分可观测性:它们只记录特定时刻控球球员的动作。针对控球动作数据,[Decroos等人(2019)](#ref-9)提出了VAEP(通过估计概率评估动作价值,Valuing Actions by Estimating Probabilities)框架,该框架基于足球运动员所有控球动作对比赛结果的影响进行评估。然而,他们的模型并未明确表示比赛环境,而是考虑了一组从近期比赛历史中人工提取的动作特征,以及某个动作是否会在固定数量的未来步骤内导致进球。

另一种评估球员的方法是量化其相对替代水平的价值(value-above-replacement, VAR)。最常见的VAR指标包括进球/胜场超越替代球员(Goals/Wins Above Replacement, GAR/WAR),通过估计目标球员在场与替代水平球员在场时球队得分/获胜机会的差异,来衡量球员对球队的贡献。本文中,我们将替代水平球员定义为统计意义上的联赛平均水平或随机球员。在其他研究中,替代水平代表球队以最低成本可获得的具备一般技能的球员。

### 2.2 强化学习在体育分析中的应用

强化学习(Reinforcement Learning, RL)对形式为s₀, a₀, r₁, s₁, a₁, ..., sₜ, aₜ, rₜ₊₁, sₜ₊₁, aₜ₊₁的事件数据建模:环境状态sₜ出现,选择动作aₜ,产生奖励rₜ₊₁和状态sₜ₊₁。在下一时间步,选择另一动作aₜ₊₁。数据通常被分解为形式为T{s, a, r', s', a'}的局部转移。强化学习已被应用于评估球员动作。<span style="color: red;">[Schulte等人(2017a)](#ref-25)应用冰球逐场比赛数据集构建了马尔可夫模型</span>,

<div style="text-align: center;">
    <img width="582" height="229" alt="image 1" src="https://github.com/user-attachments/assets/01afa743-0d1d-4bef-94af-21f51e100e82" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**图1** 本研究在研究领域中的定位树状图。一个重要因素是指标是否考虑所有动作或仅考虑其子集。我们的方法为所有控球动作赋值。加粗方法在我们的实验中得到评估,星号标记所提出的指标。
</p>

其中动作记录球员移动,状态捕捉比赛情境。他们通过预期得分影响(Scoring Impact, SI)来衡量球员表现。<span style="color: red;"> 不同比赛情境下球员动作的预期得分概率通过基于贝尔曼方程(Bellman equation)的Q函数建模,采用动态规划方法(Puterman和Patrick 2017)</span>:

<div style="text-align: center;">
    <img width="496" height="80" alt="image 2" src="https://github.com/user-attachments/assets/0350b170-258d-4786-96c2-64324e353628" />
</div>

该递推式使我们能够在当前情境s、a下估计Q值,前提是已有下一Q值和转移概率Pr的估计。<span style="color: red;"> [Schulte等人(2017a)](#ref-25)**对位置和时间坐标进行离散化**,并对由此产生的离散转移概率使用**极大似然估计**</span>。XThreat模型是一个针对足球的离散马尔可夫模型,将球场划分为192个区域,使用贝尔曼方程评估预期得分变化及由此产生的影响值[(Van Roy等人 2017)](#ref-34)。<span style="color: red;"> XThreat模型仅考虑传球和带球两种动作类型</span>。离散化导致信息损失和Q函数在时空上出现不理想的不连续性。这些不连续性阻碍了模型对状态空间未观测部分的泛化能力。

我们的研究采用无模型方法(model-free approach),无需显式估计转移概率和奖励概率即可学习Q值[(Sutton和Barto 2018)](#ref-30),而非在离散马尔可夫决策过程中显式建模转移。许多先前的无模型强化学习研究[(Mnih等人 2015)](#ref-20)将无模型学习与深度神经网络结合,以捕捉连续的动作和状态特征。这些研究主要关注连续流动游戏(如Atari游戏)的控制。然而,职业体育比赛中的真实智能体——球员——是评估对象,而非强化学习方法的控制对象。[Dick和Brefeld(2019)](#ref-10)应用无模型强化学习根据当前控球球队将球带近对方球门的机会来评估足球比赛状态的价值。他们假设有跟踪数据(指定每个时间步的位置和球的状态),而非像我们的模型那样使用事件数据。此外,他们未将所学价值函数应用于评估球员表现。

为评估球员表现,<span style="color: red;"> [Liu和Schulte(2018)](#ref-15)应用深度循环模型捕捉冰球比赛历史特征。他们的模型采用Sarsa时序差分学习方法计算Q值</span>,以衡量球员下一个进球的预期概率。我们的研究将[Liu和Schulte(2018)](#ref-15)的方法从冰球扩展到针对更复杂的欧洲足球运动设计的更复杂模型。我们展示了所得影响值可以通过模仿学习进行解释,并为所学影响值提供了理论依据。

## 3 数据集

体育分析使用多种不同格式的数据:技术统计数据(box score data),提供每位球员和每场比赛的动作总计数(如进球数);逐场比赛数据(play-by-play data),记录离散动作事件的日志,详细说明动作的各种属性(如动作类型、执行球员、时间和位置);以及跟踪数据(tracking data),以密集的时间间隔记录每位球员的位置(例如每个广播视频帧,或通过体育场摄像机以更高频率记录)。<span style="color: red;"> 本文中,我们使用Opta公司提供的F24足球比赛逐场数据集<sup style="color: red;">1</sup></span>。该数据集记录了2017-2018整个赛季多个足球联赛的比赛事件和球员动作的逐场信息,包括英格兰超级联赛、荷兰甲级联赛、英格兰足球冠军联赛、意大利甲级联赛、德国甲级联赛、西班牙甲级联赛、法国甲级联赛和德国乙级联赛。**表3**展示了数据集统计信息。

> <sup style="color: red;">1</sup>[https://www.optasports.com/](https://www.optasports.com/)

该数据集记录了控球球员的动作以及空间和时间情境特征。完整的特征集列于**表2**。**表1**列出了一系列描述主客队进球序列的事件。该数据集使用调整后的空间坐标。X坐标和Y坐标均调整至[0, +100]范围。调整后的足球场如**图2**所示,两队的进攻方向均从左到右。为调整坐标,当控球球队向左侧进攻时,我们对坐标进行反转,因此在这种情况下X调整后 = −rescale(X),Y调整后 = −rescale(Y)。调整后的坐标加快了训练期间的模型收敛速度,并改善了空间特征的模型拟合效果(第6.1节)。

<div style="text-align: center;">
    <img width="904" height="449" alt="微信图片_2025-11-04_160624_745" src="https://github.com/user-attachments/assets/0c291113-298d-4c6f-9307-15a2c35a579c" />
    <img width="553" height="188" alt="image 3" src="https://github.com/user-attachments/assets/404a7d69-36d0-4bbf-ba68-435434bf8c84" />
    <img width="561" height="319" alt="image 4" src="https://github.com/user-attachments/assets/df4f2b97-df61-4d74-9c37-f15c0883b21c" />
    <img width="554" height="223" alt="image 5" src="https://github.com/user-attachments/assets/927ffb6c-868d-4869-9584-360dd01accd8" />
</div>

## 4 建模比赛动态

本节介绍我们定义足球比赛马尔可夫模型的方法,以及在不同比赛情境下评估球员动作的Q函数。

### 4.1 体育比赛的马尔可夫博弈模型

[马尔可夫博弈模型（Markov Game Model）详解](https://www.notion.so/Markov-Game-Model-2a2692a22bd88088a55bca6fe57e875d?pvs=21)

<span style="color: red;"> 与[Liu和Schulte(2018)](#ref-15)类似,我们应用马尔可夫博弈框架(Markov Game Framework)来建模体育比赛的进程动态</span>。该模型的基本构成要素包括:

- 有两个智能体,主队(Home)和客队(Away),分别代表各自的球队。
- 动作aₜ表示控球球员的移动。<span style="color: red;"> 我们的模型使用独热表示(one-hot representation)的离散动作向量</span>。
- 观测值是一个特征向量xₜ,在离散时间步t指定**表2**所列特征的取值。我们使用完整序列sₜ ≡ (xₜ, aₜ₋₁, xₜ₋₁, ..., x₀)表示状态[Mnih等人 2015)](#ref-20)。
- 奖励rₜ是进球值gₜ的向量,指定哪支球队(主队、客队)进球。我们引入了额外的"双方均未进球"(Neither)指示符,用于表示比赛结束前双方均未进球的情况。为便于阅读,我们使用主队、客队、双方均未进球来表示进球值向量rₜ = [gₜ,主队, gₜ,客队, gₜ,双方均未进球]中的三元向量,其中gₜ,主队 = 1表示主队在时刻t进球(见**表1**)。

### 4.2 下一进球Q函数

[Next-Goal Q函数：核心价值评估](https://www.notion.so/Next-Goal-Q-2a2692a22bd880b78909c6a57c688ded?pvs=21)

已有多种价值函数用于评估球员动作。<span style="color: red;"> 一种选择是衡量动作是否提高了获胜机会[(Routley 2015)](#ref-23)</span>。更近期的研究关注<span style="color: red;"> 动作在得分或进球方面的更直接影响([Cervone等人 2016](#ref-8);[Schulte等人 2017b](#ref-26))</span>。对于足球,我们用下一进球Q函数来形式化这一思想,其定义如下。我们<span style="color: red;"> **将足球比赛划分为若干进球回合(goal-scoring episodes),使得每个回合1)从比赛开始或进球后立即开始,2)以进球或比赛结束终止。下一进球Q函数表示主队或客队在当前进球回合结束时进球(进球主队 = 1或进球客队 = 1)的概率,或双方均未进球(进球双方均未进球 = 1)的概率**</span>:

<div style="text-align: center;">
    <img width="441" height="34" alt="image 6" src="https://github.com/user-attachments/assets/6942bd08-abe7-4b90-bbaa-14bafa1b6673" />
</div>

其中team是主队、客队或双方均未进球之一的占位符。<span style="color: red;">该Q函数表示在体育比赛的当前进程动态下,某支球队打进下一个进球的概率([Schulte等人 2017a](#ref-25);[Routley和Schulte 2015](#ref-24))</span>。对于球员评估,相比于获胜概率,下一进球Q函数具有若干优势。

- 与最终比赛结果相比,Q值建模的是时间上相对较近的下一进球概率,因此更易于解释和理解。
- 提高球员所在球队打进下一球的概率同时捕捉了进攻和防守价值。例如,抢断等防守动作降低了对方球队打进下一球的概率,从而提高了球员本队打进下一球的概率。
- 下一进球奖励捕捉了教练对球员的期望。例如,教练希望球员专注于当前时刻防守对手进攻和创造下一次得分机会,而非思考比赛将如何结束。

## 5 学习Q值:模型架构与训练

[实际训练流程](https://www.notion.so/2a2692a22bd88098a0bfe8a8e23b5731?pvs=21)

本节介绍用于学习Q函数(Q球队(s, a))的神经网络架构和权重训练方法。

### 5.1 模型架构:基于神经网络的函数逼近

<span style="color: grey;">（解释：我们如何计算足球比赛的Q值呢，动态规划可以计算Q值，但是需要在离散状态条件下才能进行计算。）</span>

[关于动态规划计算Q值](https://www.notion.so/Q-2a3692a22bd880049cc3dffba4071b01?pvs=21)

我们讨论学习Q值的模型架构。给定离散状态空间,可以使用<span style="color: red;">动态规划</span><span style="color: grey;">（解释：计算家到学校的共有多少走法。传统的会重复计算，动态规划先算家到路口A有多少种走法，路口A到学校有多少种走法，最后加起来。把算过的结果存起来，下次直接用，避免重复计算。）</span><span style="color: red;">计算Q值([Schulte等人 2017b](#ref-26);[Van Roy等人 2017](#ref-34))</span>。但我们的足球模型包含源自连续时间戳和空间位置的连续观测特征。一个常见的解决方案是对<span style="color: red;">**时空索引进行离散化[(Gudmundsson和Horton 2017)](#ref-12)**</span><span style="color: grey;">（解释：把连续变量划分为离散变量，比如x为1-120，划分为防守、进攻区等）</span>。然而,由此产生的不连续性损害了状态值的精度,并影响预测准确性。本文中,我们开发了一种能够直接整合连续观测特征的神经网络方法。

为生成Q值,我们的模型采用<span style="color: red;">双塔设计(two-tower design)(Song等人 2017)</span>分别拟合主客队数据,并使用<span style="color: red;">循环神经网络捕捉比赛历史中的序列特征</span>。**图3**展示了我们的模型结构。该模型分别拟合主客队数据,因为根据领域知识,我们预期Q值会因球队是主场作战还是客场作战而不同(关于主场优势的讨论见Swartz和Arce 2014)。<span style="color: red;">每个塔使用堆叠LSTM捕捉比赛历史,堆叠LSTM是一种多层LSTM,其中较低层LSTM单元的输出用作较高层的输入。与单层LSTM相比,堆叠为序列的输入特征增加了抽象层次。这提高了模型在复杂比赛情境中的泛化能力。</span>

<span style="color: red;">比赛情境和动作(sₜ, aₜ)的完整比赛历史被总结在顶层LSTM的最后一个隐藏状态中</span>。我们的模型使用球队标识单元根据当前比赛中谁控球来选择主队塔或客队塔的隐藏状态。选定的隐藏状态值被发送到隐藏层,<span style="color: red;">其输出通过softmax函数归一化,并被视为我们对Q̂主队(s, a)、Q̂客队(s, a)和Q̂双方均未进球(s, a)的估计。</span>
<div style="text-align: center;">
    <img width="558" height="336" alt="image 7" src="https://github.com/user-attachments/assets/acdfd7d7-4c8d-48de-b03a-616a02e560a6" />
</div>

### 5.2 权重训练

我们使用<span style="color: red;">**时序差分(Temporal Difference, TD)预测方法Sarsa([Sutton和Barto 2018](#ref-30),第6.4章)训练双塔神经网络**,并应用动态控球LSTM在训练期间控制轨迹长度。我们的目标是学习一个函数来估计数据集中观测到的比赛动态的Q球队(s, a),据此评估球员表现</span>。训练细节如下。

**主客队塔权重训练** 在训练时间步t,如果主队/客队在时刻t控球,我们的模型将主队/客队塔的输出送入隐藏层。在一个训练步骤中,隐藏层为一次转移T{sₜ, aₜ, rₜ₊₁, sₜ₊₁, aₜ₊₁}内的两个连续动作和状态估计Q值。估计的Q值用于计算TD损失:

<div style="text-align: center;">
    <img width="504" height="49" alt="image 8" src="https://github.com/user-attachments/assets/2ce00280-6506-43a7-9699-8a64d74f0f67" />
</div>

我们使用带<span style="color: red;">反向传播的小批量梯度下降法寻找最小化该损失函数的神经模型权重</span>(**图3**)。由于对每次转移,误差信号仅发送到主队塔或客队塔之一,梯度流仅影响两个塔中的一个,因此它们的权重独立更新。这种独立性分离了主客队信号,有助于网络学习它们的影响。

**动态控球LSTM** <span style="color: red;">像足球这样的团队运动具有轮流攻防的特点,其中一队进攻而另一队防守;这样的一次轮换称为一次进攻(play)。当控球权从时刻t的球队传递到时刻t+1的对方球队时,一次进攻结束[(Liu和Schulte 2018)](#ref-15)</span>。在体育比赛中,一次进攻内的事件高度相关,但当一支球队失去控球权(即进攻结束)时,进攻方转为防守。<span style="color: red;">因此,连续进攻的动作之间的依赖性要弱得多。</span>

<span style="color: red;">轮流攻防的特点启发了确定轨迹长度tlₜ的自然方式,该长度控制LSTM在输入历史中从当前时刻向回传播误差信号的时间跨度。我们的模型不固定轨迹长度,而是动态计算,将tlₜ设置为从当前时刻t到当前进攻开始的时间步数(最大为10步),使LSTM能够将历史轨迹限制在一支球队的连续控球期内。**使用控球权转换来定义时序模型的回合在许多连续流动的体育项目中已被证明是成功的,尤其是篮球([Cervone等人 2016](#ref-8);[Gudmundsson和Horton 2017](#ref-12))。**</span>

**训练设置** 对于**图3**中的TTDP-LSTM模型,主客队塔均采用两层LSTM,其输出被送入具有三个输出节点的两个隐藏层。<span style="color: red;">LSTM隐藏状态和隐藏层的节点数均为256</span>。LSTM的最大轨迹长度为10(Hausknecht和Stone 2015)。训练期间,我们使用Adam优化器最小化损失函数L(θ),在整个数据集(包含超过450万条事件数据)上的初始通用学习率为10E-04,在特定联赛数据集上的微调初始学习率为10E-05。

**计算复杂度** <span style="color: red;">应用神经网络逼近函数,Sarsa预测算法通过反向传播更新神经网络权重来学习Q函数</span>。我们的模型为每支球队应用一个两层堆叠LSTM(轨迹长度为10)加一个嵌入层,以及两个隐藏层来生成Q值。密集层和LSTM单元的隐藏层(或状态)大小均设为256。<span style="color: red;">假设批次中有m个训练样本,输入空间维度为n,则完成一批神经网络训练的时间复杂度为O(mn)</span>。虽然每个训练步骤的成本与批次大小呈线性关系,但收敛所需的梯度步数取决于数据集和超参数设置,无法先验地限定。

## 6 模型验证:Q值

我们的案例研究通过时间和空间投影展示了所学Q函数。为验证模型性能,我们证明所学Q值具有良好的校准性,即它们能够很好地拟合在不同比赛情境下观测到的实证得分频率。

### 6.1 时间和空间投影的展示

**时间投影** 我们展示了跨比赛时间的动作和状态的估计Q值。**图4**显示了一个<span style="color: red;">价值指示器(value ticker)[(Cervone等人 2016)](#ref-8),表示从数据集中随机抽样的一场比赛中Q值的演变</span>。该图根据代表Q̂主队(s, a)、Q̂客队(s, a)和Q̂双方均未进球(s, a)的三个输出节点的值绘制,据此我们突出显示关键事件以展示Q函数的情境敏感性。我们观察到:(1)一支球队的高得分概率会降低其对手的得分概率。(2)比赛结束时,双方均不得分的概率显著上升。

<div style="text-align: center;">
    <img width="553" height="248" alt="image 9" src="https://github.com/user-attachments/assets/6eac37d9-786a-41e9-a1e6-42cea41876fd" />
</div>

**空间投影** 为研究球员位置对得分概率的影响,我们为整个足球场生成Q值。我们的神经模型可以从观测到的状态和动作泛化到观测赛季中未出现的状态和动作。我们模型的泛化能力使我们能够为在任何位置执行的任何动作估计Q值。**图5**显示了主队若干动作(包括射门、传球、传中和抢断)在可能比赛轨迹上的已学平滑Q函数曲面Q̂主队(s, a)。<span style="color: red;">我们选择这些动作是因为它们频繁出现且在先前研究中被研究过([Brooks等人 2016](#ref-6);[Van Haaren等人 2016](#ref-33))</span>。

<div style="text-align: center;">
    <img width="560" height="354" alt="image 10" src="https://github.com/user-attachments/assets/ae370e2f-5f67-4954-95a5-161d36c7e521" />
</div>

对于所选动作,我们观察到射门、传球和传中等进攻动作的Q值随着接近对方球门而增加。防守性抢断的价值随着接近本队球门而增加。从球门左侧的角度看似乎比从右侧略有优势。Q̂主队(s, 传球)和Q̂主队(s, 传中)的图表显示了相同现象。对第一个观察的解释是,当球员接近对方球门时,他们有更多机会得分。关于射门角度的第二个观察,对数据集的检查显示有多个从上角得分的进球(例如成功的香蕉球),但下角没有进球。左右不对称性也解释了为什么在左下角附近进行的防守性抢断更有价值(最后一图):抢断干扰了对手可能导致其上角成功射门的动作。

### 6.2 所学Q函数的校准质量

校准研究评估我们所学Q函数对不同离散比赛情境下观测到的下一进球得分频率的拟合程度。<span style="color: red;">我们定义离散比赛情境的方法是将连续状态空间划分为离散区间。为计算与每个区间相关的实证得分频率,我们根据最后一次观测中三个离散情境特征的值将观测到的状态分配到一个区间:Manpower (Short Handed (SH), Even Strength(ES), Power Play(PP)), Goal Differential (≤-3, -2, -1, 0, 1, 2, ≥3)和时段(1(上半场)、2(下半场))。区间总数为3 × 7 × 2 = 42</span>。这种划分有两个优势。<span style="color: red;">(1)这些情境特征被足球专家充分研究且重要[(Decroos等人 2019)](#ref-9),因此模型预测可以与领域知识进行对照验证</span>。(2)该划分涵盖广泛的比赛情境,每个区间汇总了大量比赛历史。如果我们的模型表现出系统性偏差,汇总应该会放大偏差,使其可被检测到。

<span style="color: red;">给定区间集合,其中每个区间A包含总计|A|个状态,每个区间的实证和估计得分概率定义如下:</span>

- **实证得分概率**:对于每个观测状态s,如果包含状态s的观测回合以球队team(team = 主队、客队)进球结束,或双方均未进球(team = 双方均未进球),则设置进球观测球队(s) = 1。那么Q实证球队(A) = 1/|A| ∑s∈A 进球观测球队(s)
- **估计得分概率**:我们应用TTDP-LSTM模型为每个观测序列估计Q值,并对结果估计求平均以计算估计得分概率:Q̂球队(A) = 1/|A| ∑s∈A Q̂球队(s, a)

我们将拟合效果评估为平均实证得分概率Q实证球队(A)与平均估计得分概率Q̂球队(A)之间的差异。我们在**表4**中展示结果,其中情境特征人数优势(Man.)、进球差(Goal.)和时段(P.)定义一个区间,|A|记录数据集中每个区间A中的动作数量。

估计的Q函数匹配若干已知现象:1)任一球队在第二时段再次进球的机会下降。2)明显的主场优势(Swartz和Arce 2014):比较主客队角色互换的两种比赛情境,主队的相对优势大于客队的相对优势。3)主队的人数优势意味着客队得分机会降低。

我们的结论如下。(1)模型拟合令人满意(即所有区间的平均MAE低于0.1),除了一些相对罕见的比赛情境。(例如,主队在上半场处于人数优势但落后的情境,其对应区间计数在300万个比赛状态中仅为876)。(2)我们的模型显著优于具有离散状态空间的马尔可夫模型。这显示了能够利用连续时空信息而不因离散化损失信息的函数逼近模型的优势。

<div style="text-align: center;">
    <img width="668" height="463" alt="image 11" src="https://github.com/user-attachments/assets/23b83988-6d31-4e6e-a4ae-a5557b5c9afc" />
</div>

## 7 基于Q值的球员评估指标

本节展示如何从Q函数导出球员评估指标。我们论文衡量球员表现的主要方法是为球员动作分配影响值<span style="color: red;">(两个连续Q值之间的差值)</span>。为理解神经网络何时会为球员动作分配高值,我们用状态-动作特征及相应影响值拟合了一个回归树。为我们的影响指标提供理论基础,本节引入另一个超越替代水平的Q值指标来评估球员动作。通过证明两个指标等价,我们表明Q值统一了球员评估的两种主要方法。

### 7.1 进球影响:从Q值导出动作价值

我们的Q函数概念为动作赋值提供了一个新颖的基于人工智能的定义。<span style="color: red;">与[Schulte等人(2017b)](#ref-26)和[Routley和Schulte(2015)](#ref-24)类似</span>,我们通过动作对球员所在球队预期总奖励的改变程度来衡量动作质量:球员行动前后预期总奖励的差值。某一时刻的得分机会衡量状态的价值,因此取决于整个球队先前的努力,而价值的变化直接衡量特定球员某个动作的影响。对于我们选择的下一进球作为奖励函数,我们将其称为进球影响。球员所有动作的总影响即为其进球影响指标(Goal Impact Metric, GIM)值。

以下方程展示了如何根据我们的TTDP-LSTM模型的Q值估计,为转移T{s, a, r', s', a'}计算动作影响。s', a'之前的预期未来总奖励为r' + Es',a'[Q球队(s', a')|s, a]。s', a'之后的预期未来总奖励为r' + Q球队(s', a')。因此:

<div style="text-align: center;">
    <img width="480" height="73" alt="image 12" src="https://github.com/user-attachments/assets/c4d19266-2ec4-445b-8e74-434d45815559" />
</div>

其中D表示我们的数据集,球队ᵢ表示球员i所在球队,n[s, a, s', a', pl' = i; D]是球员i在s, a之后在s'处执行动作a'的出现次数。贝尔曼方程(1)意味着Es',a'[Q球队(s', a')|s, a] = Q球队(s, a) − E[r'|s, a]。因此,给定预期奖励模型,期望可以从估计的Q值计算得出。在我们的数据中,进球被表示为一个单独的动作goal,之后不发生转移。这意味着对于每个转移T{s, a, r', s', a'},我们有a ≠ goal,r' = 0,因此E[r'|s, a] = 0。因此在这种表示下,影响**方程(5)**简化为球员行动前后Q值的差值。

### 7.2 通过模仿决策树理解影响值

影响值通过Q函数计算,该函数应用黑箱神经网络拟合状态-动作特征。为理解为什么某些动作在特定比赛情境下具有大影响,<span style="color: red;">我们应用模仿学习[(Ba和Caruana 2014)](#ref-3),训练一个透明的回归树(CART)来模仿深度模型的行为</span>。这项可解释性研究包括两个主要步骤。(1)我们将球员的状态和动作作为输入送入CART,通过监督学习拟合产生的影响值。在每个分裂节点,CART自动选择对子节点影响值贡献最大方差减少的特征。我们分裂直到其中一个子节点包含的样本少于80/90个(分别针对射门/传球)。(2)树学习后,我们通过求和应用该特征的分裂处的方差减少来计算特征的重要性[(Liu等人 2018)](#ref-16)。我们按重要性值对状态和动作特征排序。

表5和表6显示了射门和传球的前10个重要特征。图6和图7通过绘制其前三层展示了CART树的结构。射门和传球影响的树都将动作结果(标记动作成功或失败的二元特征)置于根节点,直观上这是最重要的动作特征之一。我们还发现射门影响随着球员接近球门而显著增加,这与我们在Q值空间投影中的发现一致。对于传球,其影响随比赛速度增加而增加。一个解释是快速传球防止了对手的潜在干扰。当比赛接近结束时,我们观察到虽然平均传球影响下降,但不同传球之间影响的方差显著增加。我们在图7中的CART准确定位了这一现象开始发生的时间(剩余时间(t-1) < 39.45)。另一个重要观察是,除了当前时刻t的特征外,历史特征(如X坐标(t-1))也被认为对预测当前动作的影响很重要。
<div style="text-align: center;">
    <img width="507" height="230" alt="image 13" src="https://github.com/user-attachments/assets/ee01c347-6d3f-49d7-971b-f4725886ce87" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**表5** 射门影响的特征影响力
</p>

<div style="text-align: center;">
    <img width="516" height="229" alt="image 14" src="https://github.com/user-attachments/assets/9a90a6d0-28bb-4b4d-acf0-98aad4936130" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**表6** 传球影响的特征影响力
</p>

<div style="text-align: center;">
    <img width="424" height="190" alt="image 15" src="https://github.com/user-attachments/assets/287a5f12-8a41-473e-b315-5364e7c4abe7" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**图6** 射门影响的回归树
</p>

### 7.3 超越平均替代水平的Q值

我们将进球影响指标与使用超越平均替代水平框架从Q函数导出的球员指标进行比较。同一球员表现排名可以使用两种根本不同的方法导出,这一事实支持了我们指标的概念基础。
<div style="text-align: center;">
    <img width="428" height="184" alt="image 16" src="https://github.com/user-attachments/assets/a2a79663-426c-475d-a0dc-2f9e7a77de80" />
</div>

<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**图7** 传球影响的回归树
</p>


QAAR指标将给定球员i下一个行动的预期未来总奖励与给定随机替代球员下一个行动的预期未来总奖励进行比较:

<div style="text-align: center;">
    <img width="478" height="77" alt="image 17" src="https://github.com/user-attachments/assets/69f9ff5d-d19c-4616-a19c-c36598cd1db7" />
</div>

其中n[s, a, pl' = i; D]是球员i在s, a之后执行动作的出现次数。QAAR指标可以通过使用转移概率的极大似然估计对数据集进行计算。QAAR和GIM分别是超越替代水平方法和动作价值方法的自然定义。我们的主要结果是它们等价:

**命题1** 对于我们逐场比赛数据集D中记录的每位球员i,其超越替代水平的Q值等于其进球影响指标:QAARᵢ(D) = GIMᵢ(D)。

完整证明见我们的"附录"。该方程表明,通过对整个赛季的球员影响求和(GIM),我们衡量其一般比赛技能超过同联赛平均球员(具有平均Q值的替代球员)的程度。因此,可以使用两种根本不同的方法从Q函数导出相同的球员排名方法。在下一节中,我们通过应用GIM对球员评分展示一些排名示例。

## 8 球员排名:案例研究

为展示GIM,我们讨论若干球员的排名结果。我们按2017-2018整个赛季的GIM对英格兰足球冠军联赛球员进行排名。我们的案例研究仅对一个联赛的球员排名,因为他们面对相同竞争水平,因此他们的贡献具有可比性。我们选择英格兰足球冠军联赛,该联赛在联赛层级中仅次于英超联赛,因为它在我们数据集中有大量球员,且相比英超联赛研究较少。

**微调** 不同联赛有其自身特点,包括竞争水平、赛季长度和季后赛安排。因此,我们应用微调技术以更好地适应英格兰足球冠军联赛比赛。

1. 训练一个通用模型,使用来自多个欧洲足球联赛的比赛评估欧洲足球的动作。
2. 以较小的学习率微调通用模型的初始权重值,仅使用英格兰足球冠军联赛比赛数据。

微调精炼了通用模型,提高了其捕捉球员行为的能力。与从头训练模型相比,微调显著减少了训练时间并防止过拟合。在以下评估中,我们描述用微调模型计算的GIM值,并展示所有动作的总体排名和特定动作排名。

### 8.1 所有动作评估

表7列出了所有动作GIM最高的10名球员。我们的排名包括进球和助攻最多的球员。我们在下一节进一步研究我们的指标与标准成功指标之间的正相关性。马泰伊·维德拉(Matej Vydra)在我们的2017-2018赛季排名中位居榜首。他在英格兰冠军联赛得分榜上占据主导地位,赢得了2017-2018金靴奖<sup style="color: red;">2</sup>。在下一赛季(2018-2019),英超球队伯恩利认可了维德拉的天赋,与德比郡签下了他三年的合同。另一个例子是汤姆·凯尔尼(Tom Cairney),整个赛季仅有5个进球和5次助攻,但在GIM评估中排名第6。尽管他在任何标准成功统计指标(进球、助攻)上都不领先,但他的影响是他所在球队赢得2017-2018英格兰足球冠军联赛季后赛成功的不可或缺因素。例如,他在温布利球场决赛中打进唯一进球,富勒姆1-0击败阿斯顿维拉,赢得晋级英超联赛的资格。汤姆·凯尔尼被提名为英格兰足球联赛冠军联赛赛季最佳球员奖<sup style="color: red;">3</sup>。
> <sup style="color: red;">2</sup>https://www.skysports.com/football/news/11688/11361634/
> <sup style="color: red;">3</sup>https://www.bbc.com/sport/football/43641225

### 8.2 特定动作评估

特定动作排名仅评估感兴趣动作的影响。我们分别按射门和传球计算英格兰足球冠军联赛球员的两个GIM排名。这些是足球中频繁出现且影响大的动作。表8和表9列出了前10名球员。仅从射门计算的GIM可以视为流行的预期进球数(XG)指标的替代方案。高影响的射门将显著增加进球概率,因此表8中的顶级球员也在进球得分中领先。例如,马泰伊·维德拉是得分影响最高的球员,他在2017-18赛季也在进球得分上占据主导地位。然而,传球影响与助攻次数之间的关系更为复杂。存在一些关联,因为助攻通常是高价值传球。另一方面,助攻次数是对传球能力的不完整衡量,因为它忽略了中场和防守区域的传球。相反,我们的排名为球员的所有传球提供了全面评估。例如,康纳·霍里汉担任中场,整个赛季仅有2次助攻。但他完成了许多有影响力的传球,被我们的指标评为前10名传球手。

<div style="text-align: center;">
    <img width="523" height="247" alt="image 18" src="https://github.com/user-attachments/assets/8242e255-29d9-4c2d-8620-1c346084fbb2" />
</div>
<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**表7** 2017-2018赛季英格兰足球冠军联赛球员影响分前10名
</p>

<div style="text-align: center;">
    <img width="513" height="228" alt="image 19" src="https://github.com/user-attachments/assets/9b4d483a-b812-49d8-9762-7939bcc12b72" />
</div>
<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**表8** 2017-2018英格兰足球冠军联赛射门影响最大的前10名球员
</p>

<div style="text-align: center;">
    <img width="510" height="228" alt="image 20" src="https://github.com/user-attachments/assets/c85a1456-8f05-4983-bc4c-18fb6d4979a3" />
</div>
<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**表9** 2017-2018英格兰足球冠军联赛传球影响最大的前10名球员
</p>

## 9 球员排名:实证评估

我们描述比较方法和评估方法论。与聚类和推荐问题类似,球员排名没有真实标准。为评估球员评估指标,我们遵循先前研究([Routley和Schulte 2015](#ref-24);[Liu和Schulte 2018](#ref-15)),计算其与直接衡量成功的统计数据的相关性。

### 9.1 比较球员评估指标

我们将GIM与基线球员评估指标进行比较,以显示以下优势:(1)建模比赛情境;(2)整合连续情境信号和历史;(3)分别处理主客队状态动作信号。我们的基线球员评估指标如下。

**基于进球的指标** (i)正负值(PlusMinus, PM)是一个常被研究的指标,衡量球员在场对其球队进球的影响程度(Macdonald 2011)。(ii)预期进球数(Expected Goal, XG)根据每次射门导致进球的机会对其加权。球员按其总预期进球射门排名。PM和XG都只考虑非常有限的比赛情境和动作类型。

接下来的三个基线为所有动作分配影响值,并根据球员的总动作影响评估球员。

**所有动作指标** (iii)通过估计概率评估动作价值(Valuing Actions by Estimating Probabilities, VAEP)[(Decroos等人 2019)](#ref-9)应用动作价值的差值来计算控球动作的影响。VAEP不是应用时序差分学习估计Q值,而是使用分类器<sup style="color: red;">4</sup>估计动作在接下来k步(窗口大小)内导致进球的概率。(iv)得分影响(Scoring Impact, SI)基于具有预离散化空间和时间特征(如x, y坐标和比赛时间)的马尔可夫模型[(Schulte等人 2017a)](#ref-25)。应用动态规划估计离散状态-动作空间的Q函数和影响值。(v)DP-LSTM是先前应用于估计冰球动作价值的神经网络架构。它应用循环模型捕捉比赛情境,并使用TD学习训练模型[(Liu和Schulte 2018)](#ref-15)。与我们的TTDP-LSTM的区别在于它合并了主客队塔,用单层网络拟合所有状态和动作。我们将产生的影响分数称为(M-GIM),意为"合并"。

针对英格兰足球冠军联赛球员的特定联赛研究评估了我们的微调GIM(FT-GIM)。仅使用英格兰足球冠军联赛数据从头训练一个单独模型比微调通用模型消耗更多计算资源。我们的实验记录了438万次梯度步骤从初始权重学习可靠模型,而微调仅需要81.8万次梯度步骤。

> <sup style="color: red;">4</sup>分类器使用神经网络实现,而非[Decroos等人(2019)](#ref-9)中的CatBoost,这是由于数据集规模所致。我们在局限性部分(第10.2节)进一步讨论我们的VAEP实现。

**显著性检验** 为评估GIM是否与其他球员评估指标显著不同,我们对所有球员进行配对t检验。零假设被拒绝,相应的p值为:正负值9.33E-2、XG 5.27E-281、SI 8.03E-218、VAEP 4.82E-14和M-GIM 1.02E-118。这表明GIM值与其他指标的值不同。

### 9.2 赛季总计:与标准成功指标的相关性

我们报告了2017-2018整个赛季球员排名指标与常用成功指标之间的相关性,并突出了我们GIM指标的全面性。所检验的成功指标包括进球、助攻、每场射门次数(SpG)、传球成功率(PS%)和每场关键传球次数(KeyP)。我们还研究了两个惩罚指标:获得黄牌(Yel)和获得红牌(Red)。表10显示了所有10个联赛球员的比较方法与成功/惩罚指标之间的相关性。除了总体研究外,表11显示了特定联赛评估的结果,我们仅比较英格兰足球冠军联赛球员的相关性。

**与其他方法相比,我们的GIM实现了非常好的相关性** 在积极成功指标中,GIM与5个成功指标中的4个(进球、助攻、SPG和KeyP)具有最高相关性,在另一个指标(PS%)上具有竞争力的结果。总体而言,基于Q函数的指标GIM、M-GIM和SI显示出与成功指标的最高相关性。XG仅是第四好的指标,因为它只考虑射门的预期值,没有校正导致射门的团队努力。VAEP与成功指标的相关性有限。这是因为他们的模型为所有动作分配相似的预期值,这转化为所有动作影响值都接近0。传统的正负值指标与几乎所有成功指标的相关性都很差。我们得出结论,提供细粒度预期动作价值估计的强化学习技术导致性能指标更好地匹配传统成功统计数据。

比较不同的强化学习方法,神经网络模型使GIM能够处理连续输入而无需预离散化。这防止了比赛情境信息的损失,解释了为什么GIM和M-GIM在大多数成功指标上都优于SI。与M-GIM相比,GIM的更高相关性也证明了分别建模主客队数据的价值。

对于反映球员负面贡献的获得惩罚次数的Yel和Red,只有我们基于GIM的指标(GIM、M-GIM)与两者都显示负相关。该模型正确识别出惩罚将显著降低得分概率,影响整体球员GIM。相比之下,其他指标关注可能导致进球的动作,这倾向于奖励招致更多惩罚的激进球员。

特定联赛研究证明了微调深度强化学习模型的益处。与所有10个联赛球员的相关性相比,冠军联赛球员的相关性普遍下降。传统的基于动作计数的指标(PM、XG)和基于影响的指标(VAEP、SI、GIM、M-GIM)都显示出下降,但对我们的GIM指标而言更为严重,当用通用模型评估冠军联赛球员时,其相关性下降近20%。微调解决了这个问题:FT-GIM指标实现了与惩罚计数(Yel和Red)更大的负相关。

<div style="text-align: center;">
    <img width="514" height="182" alt="image 21" src="https://github.com/user-attachments/assets/079c648c-337c-4b0f-9b4f-01f4e16608b5" />
</div>
<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**表10** 所有球员与标准成功指标的相关性
</p>

<div style="text-align: center;">
    <img width="514" height="206" alt="image 22" src="https://github.com/user-attachments/assets/170c31a4-cf95-425b-8039-f03f98948530" />
</div>
<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**表11** 英格兰足球冠军联赛球员与标准成功指标的相关性
</p>

### 9.3 逐轮相关性:从过去表现预测未来表现

这些结果通过逐轮相关性评估球员表现指标。体育赛季可以分为若干轮。在第n轮,球队或球员已完成赛季中的n场比赛。对于给定的表现指标,我们测量(i)其在前n轮计算的值与(ii)两个主要成功指标——助攻和进球——在整个赛季计算的值之间的相关性。这使我们能够评估不同指标多快获得对最终赛季总计的预测能力,从而可以从过去表现预测未来表现。良好的表现指标应该在早期赛季与球员的整体表现一致,这为球员及其球队提供交易或训练的证据。

<div style="text-align: center;">
    <img width="501" height="186" alt="image 23" src="https://github.com/user-attachments/assets/90c4d43d-d60a-446b-9a95-ef2f849e7b44" />
</div>
<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**图8** 所有球员逐轮指标与赛季总计的相关性
</p>

<div style="text-align: center;">
    <img width="505" height="193" alt="image 24" src="https://github.com/user-attachments/assets/c7e94a26-3992-4c51-9a8b-2da7fb1ef1b7" />
</div>
<p style="text-align: center; color: #666; font-size: 0.9em; margin-top: 8px;">
**图9** 英格兰足球冠军联赛球员逐轮指标与赛季总计的相关性
</p>


**图8**显示了所有10个联赛球员的逐轮相关性<sup style="color: red;">5</sup>。GIM的预测能力增长速度快于任何其他基线:在赛季前半程之前,它与助攻(左)和进球(右)的相关性均优于其他指标。M-GIM实现了第二高的相关性,在前5轮对助攻的相关性甚至高于GIM。然而,其预测能力在前10轮之后大幅下降。其余两个指标XG和SI仅显示出与助攻和进球的弱相关性。

我们下一个实验的问题是:微调是否有助于从过去表现预测球员的最终总表现?该实验聚焦于英格兰足球冠军联赛球员。图9显示了表现指标与英格兰足球冠军联赛球员总助攻和进球的逐轮相关性。我们得出以下观察。(1)与图8的所有球员设置相比,当限制在英格兰足球冠军联赛球员时,指标的相关性下降。这种下降对我们的GIM指标更为明显。原因是在一般球员群体上训练的神经网络不能很好地拟合英格兰足球冠军联赛球员的行为。(2)微调显著改善了GIM的相关性,尤其是其与助攻的相关性,其中FT-GIM的相关性在前10轮之后超过了其他指标。

> <sup style="color: red;">5</sup>在图8和图9中,我们省略了2017-2018赛季比赛少于40场的球队的球员。

## 10 讨论

本节中,我们讨论与进球稀疏性、模型收敛和我们方法的局限性相关的主题。

### 10.1 进球的稀疏性

评估足球运动员贡献的常见方法是计算他们对进球得分的影响。然而,足球比赛中进球很少见。这个问题类似于强化学习(RL)中的稀疏奖励问题。为解决进球稀疏性,许多先前的体育分析研究建议在球员评估中包含其他指标,如助攻、传球和惩罚。这类似于强化学习中的奖励塑造(reward shaping),它添加一些人工设计的间接奖励信号来加速训练收敛[(Ng等人 1999)](#ref-21)。奖励塑造包含更多信息,但提出了如何权衡间接奖励(如传球)与真实目标奖励(得分)相对重要性的难题。时序差分解决方案学习一个Q函数,将奖励(得分)信号传播到先前事件,并在相同的预期奖励尺度上为所有动作分配价值。

### 10.2 模型收敛

我们讨论TTDP-LSTM模型的收敛性。TTDP-LSTM通过在线策略时序差分(TD)方法Sarsa训练。先前研究已保证在线策略TD与线性函数逼近器的收敛性[(Tsitsiklis和Van Roy 1997)](#ref-32)。然而,本文中我们应用非线性神经网络函数逼近器。众所周知,当动作价值Q函数定义为具有无限前瞻的预期累积奖励时,在线策略TD与非线性函数逼近器在传统强化学习设置中常表现出不稳定的收敛:
<div style="text-align: center;">
    <img width="212" height="56" alt="image 25" src="https://github.com/user-attachments/assets/d9e6cded-5309-43cf-a1ca-7019826f8c35" />
</div>

这里α ∈ (0, 1)是折扣因子,r是奖励函数。为缓解TD方法的不稳定性,本研究中我们将前瞻限制在下一个进球(而非比赛结束),并移除折扣因子,因此Q(sₜ, aₜ) = E[r(sₜ, tₜ)],即下一个进球的预期得分概率。这是有效的,因为如第7.1节所讨论,除了进球发生时刻T外,奖励r(sₜ, tₜ) = 0。

### 10.3 局限性

我们展示本研究的一些局限性并讨论一些潜在解决方案。

**场上球员的部分可观测性** 在每个时间步,我们的数据集仅记录控球球员的位置和动作。非控球球员的位置未知。然而,其他球员的信息对得分概率有影响,尤其是对于像足球这样复杂的团队运动。为缓解这一问题,我们的TTDP-LSTM模型应用循环模型拟合比赛历史,并包含先前控球球员的信息。先前在强化学习中已观察到,整合动作历史在一定程度上补偿了部分可观测性,因为模型可以从过去信息推断缺失的当前信息([McCallum 1996](#ref-18);[Hausknecht和Stone 2015](#ref-13))。例如,当前球员位置可以在一定程度上从过去球员位置预测。尽管如此,由于部分可观测性,模型性能受限。未来工作的一个方向是构建多智能体强化学习框架,将完全可观测的跟踪数据与事件类别结合。一个可能的方法是将[Dick和Brefeld(2019)](#ref-10)的深度强化学习跟踪模型与我们基于事件的深度强化学习模型结合。

**大规模输入数据问题** 我们的数据集有超过400万个事件,包括球员的空间和时间特征。拟合整个数据需要大量计算资源。当我们包含比赛历史时,可扩展性挑战增加。因此,很难利用标准机器学习包(如决策树、随机森林或梯度提升),这些包通常假设整个数据可以放入单个工作内存批次。本研究中,我们构建了一个带有小批量梯度的神经网络。在未来工作中,我们将探索在线学习方法并评估其在大规模体育数据上的性能。除了提高可扩展性外,在线方法非常适合体育数据,因为球队希望在每轮后更新球员评估。

## 11 结论

本文研究了深度强化学习(DRL)用于学习职业足球分析的复杂时空动态。我们设计了一个神经网络架构,据我们所知,这是迄今为止在体育分析中部署的最复杂架构:一个堆叠双塔LSTM架构,主客队各有一个塔。该网络使用来自多个欧洲联赛的控球动作日志进行训练,总共包含超过450万个动作事件。训练后的神经网络提供了关于球队打进下一个进球的机会如何取决于比赛情境的丰富知识来源。基于所学动作价值,我们为足球运动员开发了一个新的情境感知表现指标GIM,考虑了他们的所有动作。在我们的实验中,在整个赛季计算的GIM显示出与大多数标准成功指标的最高相关性。从赛季比赛样本进行泛化,GIM是赛季总进球和助攻的最佳预测器。为改善特定联赛球员的评估结果,我们应用微调方法在跨联赛泛化和特定联赛专业化之间实现有效平衡。

未来工作的方向包括整合跟踪数据和开发在线深度强化学习方法。深度强化学习方法在棋盘游戏中取得了惊人成功。我们的结果表明,物理团队运动的分析是另一个非常有前景的应用领域。

**致谢** 本研究得到了加拿大自然科学与工程研究委员会战略项目资助以及NVIDIA公司GPU捐赠的支持。我们感谢来自Sportlogiq的Norm Ferns、Evin Keane和Bahar Pourbabee的有益讨论和评论。

## 附录A 命题1的证明

数据记录从一个状态-动作-球员三元组到另一个三元组的转移,可能产生非零奖励(体育情境中的得分或分数)。我们将这种转移发生的次数表示为

<div style="text-align: center;">
    <img width="141" height="29" alt="image 26" src="https://github.com/user-attachments/assets/b3c4a9a5-aec4-445c-a0ef-be95339bd8ac" />
</div>

其中'表示后继三元组。我们也将此表示法用于边际计数,例如

<div style="text-align: center;">
    <img width="291" height="43" alt="image 27" src="https://github.com/user-attachments/assets/c25832f5-8e97-4122-b6ef-9ff55e07d971" />
</div>

从论文中,我们有超越替代水平的Q值和GIM指标的以下方程:

<div style="text-align: center;">
    <img width="465" height="146" alt="image 28" src="https://github.com/user-attachments/assets/db44174f-8cdb-4b45-880d-30a7151f1605" />
</div>

现在我们有

<div style="text-align: center;">
    <img width="504" height="196" alt="image 29" src="https://github.com/user-attachments/assets/02913d74-1b11-42f2-821a-721878b0cb00" />
</div>

<div style="text-align: center;">
    <img width="438" height="93" alt="image 30" src="https://github.com/user-attachments/assets/d91e2d1d-ddf6-4f4f-b9aa-b5fd7a3285ba" />
</div>

步骤(9)成立是因为期望E[Q球队(s', a'|s, a)]仅依赖于s, a,而不依赖于s', a'。行(10)使用给定球员i下一个行动的预期Q值Q球队(s', a')的经验估计,从转移概率的极大似然估计计算:

<div style="text-align: center;">
    <img width="288" height="47" alt="image 31" src="https://github.com/user-attachments/assets/07f2df6a-1034-423e-88fc-1e4d8c389c4b" />
</div>

最终结论(11)应用了方程(7)。

##**参考文献**  
<a id="ref-1"></a>
Albert J, Glickman ME, Swartz TB, Koning RH (2017) Handbook of Statistical Methods and Analyses in Sports. CRC Press, Boca Raton 

<a id="ref-2"></a>
Ali A (2011) Measuring soccer skill performance: a review. Scand J Med Scin Sports 21(2):170–183 

<a id="ref-3"></a>
Ba J, Caruana R (2014) Do deep nets really need to be deep? In: Advances in Neural Information Processing Systems, pp 2654–2662 

<a id="ref-4"></a>
Bornn L, Cervone D, Fernandez J (2018) Soccer analytics: unravelling the complexity of “the beautiful game”. Significance 15(3):26–29 

<a id="ref-5"></a>
Bransen L, Van Haaren J (2018) Measuring football players’ on-the-ball contributions from passes during games. In: Machine Learning and Data Mining for Sports Analytics, Proceedings of the 5th International Workshop. Springer, pp 3–15 

<a id="ref-6"></a>
Brooks J, Kerr M, Guttag J (2016) Developing a data-driven player ranking in soccer using predictive model weights. In: Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining. ACM, pp 49–55 

<a id="ref-7"></a>
Cervone D, D’Amour A, Bornn L, Goldsberry K (2014) Pointwise: predicting points and valuing decisions in real time with NBA optical tracking data. In: Proceedings of the 8th Annual MIT Sloan Sports Analytics Conference, vol 28 

<a id="ref-8"></a>
Cervone D, D’Amour A, Bornn L, Goldsberry K (2016) A multiresolution stochastic process model for predicting basketball possession outcomes. J Am Stat Assoc 111(514):585–599 

<a id="ref-9"></a>
Decroos T, Bransen L, Haaren JV, Davis J (2019) Actions speak louder than goals: valuing player actions in soccer. In: Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining, KDD 2019, Anchorage, AK, USA, August 4–8, 2019, pp 1851–1861 

<a id="ref-10"></a>
Dick U, Brefeld U (2019) Learning to rate player positioning in soccer. Big Data 7(1):71–82 

<a id="ref-11"></a>
Fernández J, Barcelona F, Bornn L, Cervone D (2019) Decomposing the immeasurable sport: a deep learning expected possession value framework for soccer. In: Proceedings MIT Sloan Sports Analytics Conference 

<a id="ref-12"></a>
Gudmundsson J, Horton M (2017) Spatio-Temporal Analysis of Team Sports. ACM Comput Surv 50(2):22:1–22:34. [https://doi.org/10.1145/3054132](https://doi.org/10.1145/3054132) 

<a id="ref-13"></a>
Hausknecht MJ, Stone P (2015) Deep recurrent Q-learning for partially observable MDPS. In: Proceedings of the 2015 AAAI Fall Symposia, Arlington, Virginia, USA, November 12–14, 2015, pp 29–37. CoRR. arXiv:1507.06527 

<a id="ref-14"></a>
Kharrat T, McHale IG, Peña JL (2019) Plus-minus player ratings for soccer. Eur J Oper Res 283:726–736 

<a id="ref-15"></a>
Liu G, Schulte O (2018) Deep reinforcement learning in ice hockey for context-aware player evaluation. In: Proceedings of the 27th International Joint Conference on Artificial Intelligence, IJCAI-18, [ijcai.org](http://ijcai.org/), pp 3442–3448 

<a id="ref-16"></a>
Liu G, Zhu W, Schulte O (2018) Interpreting deep sports analytics: Valuing actions and players in the NHL. In: International workshop on machine learning and data mining for sports analytics. Springer, pp 69–81 

<a id="ref-17"></a>
Macdonald B (2011) A regression-based adjusted plus-minus statistic for NHL players. J Quant Anal Sports 7(3):29 

<a id="ref-18"></a>
McCallum A (1996) Learning to use selective attention and short-term memory in sequential tasks. In: From animals to animats 4: proceedings of the fourth international conference on simulation of adaptive behavior, vol 4. MIT Press, p 315 

<a id="ref-19"></a>
McHale IG, Scarf PA, Folker DE (2012) On the development of a soccer player performance rating system for the english premier league. Interfaces 42(4):339–351 

<a id="ref-20"></a>
Mnih V, Kavukcuoglu K, Silver D et al (2015) Human-level control through deep reinforcement learning. Nature 518(7540):529–533 

<a id="ref-21"></a>
Ng AY, Harada D, Russell S (1999) Policy invariance under reward transformations: theory and application to reward shaping. Proceedings of the 16th International Conference on Machine Learning (ICML 1999), Bled, Slovenia, pp. 278–287 Puterman 

<a id="ref-22"></a>
ML, Patrick J (2017) Dynamic programming. In: Encyclopedia of machine learning and data mining, pp 377–388 

<a id="ref-23"></a>
Routley K (2015) A markov game model for valuing player actions in ice hockey. Master’s thesis, Simon Fraser University 

<a id="ref-24"></a>
Routley K, Schulte O (2015) A markov game model for valuing player actions in ice hockey. In: Proceedings of the International Conference on Uncertainty in Artificial Intelligence (UAI), pp 782–791 

<a id="ref-25"></a>
Schulte O, Khademi M, Gholami S, Zhao Z, Javan M, Desaulniers P (2017a) A markov game model for valuing actions, locations, and team performance in ice hockey. Data Mining and Knowledge Discovery, pp 1–23 

<a id="ref-26"></a>
Schulte O, Zhao Z, Javan M, Desaulniers P (2017b) Apples-to-apples: clustering and ranking NHL players using location information and scoring impact. In: Proceedings MIT Sloan Sports Analytics Conference 

<a id="ref-27"></a>
Schultze SR, Wellbrock CM (2018) A weighted plus/minus metric for individual soccer player performance. J Sports Anal 4(2):121–131 

<a id="ref-28"></a>
Schumaker RP, Solieman OK, Chen H (2010) Research in sports statistics. Sports Data Mining, Integrated Series in Information Systems, vol 26. Springer, US, pp 29–44 

<a id="ref-29"></a>
Song Y, Xu M, Zhang S, Huo L (2017) Generalization tower network: A novel deep neural network architecture for multi-task learning. arXiv preprint arXiv:1710.10036 

<a id="ref-30"></a>
Sutton RS, Barto AG (2018) Reinforcement learning: an introduction. MIT Press, Cambridge 

<a id="ref-31"></a>
Swartz TB, Arce A (2014) New insights involving the home team advantage. Int J Sports Sci Coach 9(4):681–692 

<a id="ref-32"></a>
Tsitsiklis JN, Van Roy B (1997) Analysis of temporal-diffference learning with function approximation. In: Advances in Neural Information Processing Systems, pp 1075–1081 

<a id="ref-33"></a>
Van Haaren J, Van den Broeck G, Meert W, Davis J (2016) Lifted generative learning of markov logic networks. Mach Learn 103(1):27–55 

<a id="ref-34"></a>
Van Roy M, Robberechts P, Decroos T, Davis J (2017) Valuing on-the-ball actions in soccer: a critical comparison of XT and VAEP. In: Workshop on Team Sports AAAI 2020  


