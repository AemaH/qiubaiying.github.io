---
layout:     post                    # 使用的布局（不需要改）
title:      Reinforcement Learning               # 标题 
subtitle:   对于强化学习的一些总结 #副标题
date:       2018-09-21              # 时间
author:     ERAF                      # 作者
header-img: img/shiraishi_20.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 强化学习
---
> 照例开篇的碎碎念，好吧，也知道自己之前缺了多久没有更新，其实我真的也在写真的(认真脸) ，起码前一段时间我就从：Model-based和TD 还有IRL都下笔写了的「这不叫挖坑...」，后来发生一些事情 emmmm 于是也就没写完(长叹气) ；
> 算了 无论如何先从之前 已经总结好的一些当做一个新的开始吧（握拳）

这一次所坐的主题是强化学习中一些基本问题，包括基础的：model-based和model-free的概念、off-policy和on-policy的概念、基于值函数和基于策略梯度的、RL和SL的一些碎碎念（慎读）、RL中的分类、RL过程中predict和control的概念；

更多的还是一些想法，缺少了相关概念性的描述 之后有时间再添加吧；

内容有点多 可以参考下面的目录
<!-- TOC -->

- [model-free和model-based](#model-free和model-based)
- [蒙特卡洛、TD、动态规划](#蒙特卡洛td动态规划)
- [predict和control](#predict和control)
- [off-policy和on-policy](#off-policy和on-policy)
- [off-policy和on-policy的好与坏](#off-policy和on-policy的好与坏)
- [深度强化学习中的DQN里面的experience replay方法和A3C的asynchronous](#深度强化学习中的dqn里面的experience-replay方法和a3c的asynchronous)
- [基于值函数的方法和基于策略的：](#基于值函数的方法和基于策略的)
    - [基于值函数的方法和基于策略的交叉点的AC方法](#基于值函数的方法和基于策略的交叉点的ac方法)
    - [Actor - Critic的优点](#actor---critic的优点)
    - [Actor - Critic的总结](#actor---critic的总结)
- [DQN及其相关变形和A3C](#dqn及其相关变形和a3c)
- [DQN](#dqn)
- [DDQN](#ddqn)
    - [首先对于double qlearning的理解](#首先对于double-qlearning的理解)
- [Dueling DQN](#dueling-dqn)
- [Deep Recurrent Q-Network](#deep-recurrent-q-network)
- [A3C](#a3c)
- [异步（asynchronous）](#异步asynchronous)
- [actor critic](#actor-critic)
- [advantage](#advantage)

<!-- /TOC -->

## model-free和model-based

对于强化学习来说 最终目的当然还是得到最大的累计回报期望，从这个角度来说 基于马尔科夫决策过程的强化学习，还是为了生成一条表现足够优秀的马尔科夫链；在这条链上 包含两种状态转化：状态到动作、动作到状态，进而也就转移到强化学习的两个重要问题之上：策略与转移概率；

前者的策略毋庸置疑，无论是基于值函数的方法还是基于策略的方法 最终目的都是为了获得那么一个足够优秀的状态到动作的转移方法；而后者也就是强化学习另一个绕不开的一个问题就是model-based和model-free；对于他们的分辨很简单：就是转移概率是否知道，知道了当前状态和要选择的动作后，可能导致的后果能不能推断出来。从这种角度回头看model-free 就是一步步来试图学习最优的策略，经过多次迭代后 得到整个环境中最优的策略。「来自[What-is-the-difference-between-model-based-and-model-free-reinforcement-learning](https://www.quora.com/What-is-the-difference-between-model-based-and-model-free-reinforcement-learning)」「model-free更有着探索性」

但探索 也意味着花费更多的时间 在和未知的环境之间的交互上，从这个角度来说：model-based要更为轻松毕竟 有着环境模型作为参照，直白的知道当前所做的选择：action 会导致怎样的结果，进而推断之后的可能情况；

因而对于model-based的方法 更多但还是使用动态规划的方法来进行解决；具体来说 将一个问题转化为多个可重复利用和存储的优化的子问题。主要方法包括：策略迭代和值迭代。对于策略迭代的 可以划分为：策略评估和策略改善两个部分「可以联系后面model-free里面的predict和control来进行理解」策略评估代表着对于值函数的更新 试图找到一个描述策略更为准确的值函数，而策略更新就是基于优化后的值函数 再找到一个好的策略；进而反复迭代；「同时 真正执行的时候 往往是直接对于全部的状态进行直接更新值函数 这是因为有着一个概念就是：**时间不变性**：指的是随着MDP的过程的进行，最终不同时刻下的相同状态总会有着相同的价值；」

所以常说的model-free里面的TD和蒙特卡洛之类的方法做的就是策略评估的事情，进而再根据执行策略和优化策略是不是一样的分为off-policy和on-policy；

说完了策略迭代之后 就该说值迭代了，其实说是策略迭代和值迭代 其实和所说的基于策略和基于值函数的概念不同；model-based里面的这两者都是需要基于值函数来完成对于策略的选取的。不过区别在于**策略迭代的方法 严格遵守先全部状态的值函数更新 再更新策略 再更新值函数这样的模式**，可以看到在这迭代过程中 出现了多个策略；而值迭代的方法不同 其对于中间策略不那么在意，**值迭代方法中心在于值函数 也就是更新当前这个值函数 让其最优化「贪婪策略 最大化」 进而基于此就选取策略** 「就像是只是一轮的策略迭代」

以上就是对于model-based的大致介绍，其中一些细节推导 就放在之后吧；

尽管model-based 可以提前知道转移概率 用来推断之后的可能性 找到最优的马尔科夫链 很让人向往，但实际问题中 更多的依旧是model-free ，无法知道后续状态的可能性 也无从计算后续状态的值函数，因而状态值函数$v_{\pi}(s_t)=\sum_{a_t}\pi(a_t\|s_t)\sum_{s_{t+1}}p(s_{t+1}\|s_t,a_t)[r_{t+1}+v_{\pi}(s_{t+1})]$  该公式也就无从使用；

于是对于这一类markov链的状态值函数的求解 也就是一类多步学习预测问题「预测出来后续的状态情况才能做出对于当前状态值函数的判断」，常见的方法包括蒙特卡洛方法 时域差值学习；  

我们需要知道model-free的难题在哪里：没有了环境模型因而就无法评估当前策略的好坏；从上面给出的model-based的式子可以知道，其实本质上值函数还是被用来计算的还是期望。因而蒙特卡洛的方法就是基于经验平均来代替随机变量的期望「随机样本估计期望」；

首先是对于蒙特卡洛的首先进行介绍，简单的说就是基于经验（experience）进行学习，这个经验包括样本序列的状态（state）、动作（action）和奖励（reward）。得到若干样本的经验后，通过**平均所有样本的回报（return）**来解决强化学习的任务。

因而具体谈到蒙特卡洛的方法来进行策略评估的时候，使用的公式依旧是$v_{\pi}(s)=E_{\pi}[G_t\|S_t=s]\text{ 有}G_t=R_{t+1}+\gamma R_{t+2}+...$ 因而还是需要等到一次试验结束之后 基于历史数据才能完成对于值函数的更新；「有些像有监督学习，基于历史数据 进行预测」

但显然蒙特卡洛的方法效率低下。而TD的方法 可以一方面借助于蒙特卡洛这种与环境交互进而基于经验估计，同时另一方面就像是model-based里面的接住后续值函数估计当前值函数的方法；TD的方法，可以利用连续两个时刻预测值的差值来更新模型；所使用的公式也还是$E_{\pi}[R_{t+1}+\gamma v_{\pi}(s_{t+1})]$ 不过和model-based的不同之处就只是在于 这里的后续状态不是基于转移概率推导出来的 而是基于试验得到的；

再进一步对于TD算法进行分类的话，就涉及qlearning和sarsa这样的on-policy和off-policy的区别的问题了；还有表格式和值函数逼近式，但本质所用的也大致都是TD；进一步的还有单步预测和多步预测也就是：TD(0)、TD(λ)；

## 蒙特卡洛、TD、动态规划

通过上面的介绍 这里我们可以对于蒙特卡洛、动态规划和TD进行一波总结；「emmm 这里直接用动态规划不是那么准确，更适合的应该是策略迭代里面的策略评估吧」**相同点：**如果说三者的相同点 那么也只有都是用于进行值函数的描述与更新的了「predict」；如果说**两者的之间的相同点**：蒙特卡洛和TD算法隶属于model-free，而动态规划属于model-based；TD算法和蒙特卡洛的方法 因为都是基于model-free的方法，因而对于后续状态的获知 也都是基于试验的方法；而**TD算法和蒙特卡洛的方法当然也有不同之处**就是在于：TD算法不需要等到实验结束后 才能进行当前状态的值函数的计算与更新，而蒙特卡洛的方法需要试验交互 产生一整条的马尔科夫链「到最终状态」才能进行更新。这个不同之处 就是**TD算法和动态规划的策略评估类似之处**了，它们两个都能基于当前状态的下一步预测情况来得到对于当前状态的值函数的更新，而它们的不同之处 还是就回到model-free和model-based 这一点上来了，动态规划可以凭借已知转移概率就能推断出来后续的状态情况，而TD只能借助试验才能知道了「因而 在强化学习的实验中 设置多次episode 以遍历尽量多的状态」

还有就是蒙特卡洛方法和TD方法的一个区别，因为蒙特卡洛方法进行完整的采样来获取了长期的回报值，因而在价值估计上会有着更小的偏差，但是也正因为收集了完整的信息 所以价值的方差会更大「毕竟基于试验的采样得到 和真实的分布还是有差距 不充足的交互导致的较大方差」；而TD算法与其相反 因为只考虑了前一步的回报值 其他都是基于之前的估计值 因而估计具有偏差 但方差也较小；

## predict和control

其实这部分在上面讲完之后，也没什么可以说的意义了；拿动态规划里面的策略迭代来说 前者是策略评估 后者就是策略改善；前者就是为了后者服务的，这里直接复制一下之前写的内容；

对于常见的强化学习问题可以分为学习预测和学习控制两类问题，前者的求解为实现以优化决策为目的的后者提供了基础；

具体的来说话，对于强化学习来说 一个绕不开的问题就是关于：计算平稳行为策略markov决策过程的状态值函数的问题「策略评估」；换句话说就是 在一个确定性的行为策略下如何确定其值函数；毕竟啊 确定好一个policy之后，也就相当于确定了一个markov链，那么如何计算这个markov链上的状态值函数也就成为了一大问题；

这时候就需要分：model-based 和model-free两种情况下了，「关于这两者的定义区别，直白的的说就是转移概率知不知道，也就是选取某个action后 能否直接推导下一个state 而不需要借助environment」对于model-based的情况下，直接借助动态规划就好了 具体可以分为：策略迭代、值迭代、策略搜索等；「已知模型了 那么就知道了转移概率 就可以直白的计算值函数；具体参考深入浅出P36」而实际问题中 更多的依旧是model-free 「某一个状态的之后状态的转移概率未知 那么就无法根据贝尔曼方程来直白的计算，于是我们就需要先知道当前这条markov链」于是对于这一类markov链的状态值函数的求解 也就是一类多步学习预测问题，常见的方法包括蒙特卡洛方法 时域差值学习；

首先是对于蒙特卡洛的首先进行介绍，简单的说就是基于经验（experience）进行学习，这个经验包括样本序列的状态（state）、动作（action）和奖励（reward）。得到若干样本的经验后，通过**平均所有样本的回报（return）**来解决强化学习的任务。

但显然能看出蒙特卡洛的方法，尽管可以作为一种类似监督学习的方法「基于历史数据 进行预测」可以解决多步预测的问题，但是有着效率低下的特点；而时域差分的方法 利用连续两个时刻预测值的差值来更新模型；

如果继续往下看的话，包括Qlearning、SARSA等控制学习算法的基础 也都是在这里，进而我们常见的强化学习中所分类得到的表格式和值函数逼近式的，也正是因为这里的TD算法的分类而导致的；进一步的还有对于单步预测和多步预测还有着：TD(0)、TD(λ)；具体的可以翻阅Reinforcement Learning: An Introduction ；

>   顺带说一句：根据观测数据的时间特性 预测可以分为单步预测和多步预测「基于历史数据预测现在和未来的区别」；关于监督学习和时间差分的区别，**传统**监督学习是根据预测值和实际观测值的误差来修正预测模型，而TD是根据时间上连续两次预测之间的差值来修正预测误差；具体的来说 前者需要得到全部观测数据后 才能通过计算预测误差来修正预测模型，后者 只需要某两个时刻的预测值和局部观测数据来修正预测模型；所以可以实现在线学习 减少存储量和计算量 有着更高效的学习效率；同时 也可以看出 监督学习只能实现单步学习预测 也就是只能基于当前信息来对于当前时刻的输出进行预测，而TD可以实习多步预测；
>
>   时域差分算法 往往被看组学习控制算法 如sarsa和qlearning的一部分 就像多步预测被看做为多步控制的一部分一样；

大致介绍了关于学习预测的内容之后，紧接着可以介绍关于学习控制的问题；

在上面也提到过学习预测可以看做学习控制的一个子问题，原因也就是在于 常见的强化学习算法的目的毕竟还是为了优化决策求解出来一个合适的行动策略；而学习预测本质上依旧是为了给学习控制提供出来一个评判标准：评判出来当前的优化得到的策略是好亦或是坏；这句话也指明了学习控制本质的作用：求解策略，而学习预测本质目的：求解出来一个评价函数；

>   通过上面的总结 我们可以直白的说 大部分强化学习都还是为了解决序贯决策问题而诞生的，而解决的方式就是先建立markov模型，同样的动态规划方法也可以用于解决markov模型 于是可以一起被用来解决序贯决策问题，不过后者需要提前知道模型情况 也就是状态转移概率，于是用来解决model-based问题；其余的RL算法被用来求解model-free的；可以这么说动态规划的思想是无模型强化学习的起源；这些也正是之间的关系：序贯决策→→markov决策→→动态决策/RL；「其实从markov决策出发 进而引到RL会更为直观」

所以我们知道 强化学习和动态规划是求解markov决策问题的两大方法，而这里的求解 也就是上面说的学习控制；类似于动态规划中的策略迭代需要先有着对于策略评价方法的策略评价，强化学习中 需要先得到一种适合的用于评价策略的方法 这也正是学习预测了，进而才有着类似动态规划中策略迭代一样的学习控制方法；

## off-policy和on-policy

上面讲完了model-based和model-free，借机把predict这一类问题或者说这一步问题给解释清楚了，那么紧接着 我们就需要介绍一波control了；

这里我们就直接从强化学习中最常用的TD开始说起，时间差分方法包括on-policy的SARSA和off-policy的qlearning；如果要论两者的区别 其实不应该被归于control需要解决的问题，而是属于predict里面，因为 对于两者区别的划分众所周知：就是看选取动作的策略和进行评估的策略是不是同一个策略就是了；而一般不同之处都是在评估的时候，进行对于下一个状态值函数预测的时候，qlearning选用的直接是`next_state`的最大的那个Q动作值函数，而sarsa还是依策略的和之前选取`state`时候动作`action`一样的选取`next_state`的动作`action_`进而确定Q动作值函数；举个SARSA和qlearning的例子就是：前者基于某一个策略如ε-greedy进行选取action，后面需要值函数进行更新的时候 也还是ε-greedy策略。后者则不然 选取action的策略是基于ε-greedy，更新值函数的时候依旧是选取max的那个；

如果不从predict这个对于值函数更新的角度，而是从control这个获取策略的角度来看：两者对于强化学习的目标「获取最优state-action对的马尔科夫链」这样一个最优策略的获取思路有所不同；

on-policy的方法就是遵从交互性 也就是常说的agent和environment类交互的结果、下一步的真实行动进行价值估计，于是更新也就是完全按照交互序列来的，这也是和上面介绍TD算法时候所说的一样 ：通过下一时刻的价值来更新前一时刻。 如果代入动态规划中的策略迭代来说，**on-policy有针对性的来获取这样一个target policy，直白的按照交互顺序进行更新值函数 获取策略 这样的节奏，一步步朝着最优值函数进行前进 进而获取最优的策略**。但 这样可能会到来问题就是：局部最优的问题。往大了说 还是探索与利用的矛盾，毕竟model-free 无法对于整个环境模型有着很好地感知 只能基于当前的认知来选取动作；所以 在SARSA里面 常使用ε-greedy 在一定程度上解决了这个问题； 

上面也说过 「SARSA其实是很标准的TD的方法」，这句话的意思其实是站在model-based里面标准的是策略迭代的角度上说的；而off-policy就类似model-based里面的值迭代，off-policy或者说是qlearning 它并不是直接的采用交互序列数据来对于值函数进行更新，而是选取了来自其他的策略来替换本来的交互序列，可以看做是一种探索的角度。毕竟经过这种方式 能够获得子部分的最优性，在前面累积的最优化结果的基础上进行更新与优化，参照值迭代的话 其实有些像，都是选取最优的结果进行更新；

## off-policy和on-policy的好与坏

on-policy优点因为是单纯的依照交互数据来进行更新，所以直接了当，劣势是不一定找到最优策略。

而off-policy优势在于更强的通用性 保证了探索性，其强大是因为它确保了数据全面性，所有行为都能覆盖。「这里所说的就是在选取$s_{next}$对应action的时候需要选取max的一个 那么需要全部action的q值 于是覆盖了全部的行为，相比之下on-policy只是单纯的依照策略选取了某个action；」但劣势也很明显 就是每次对于状态的值函数的估计 都是过高的进行估计。同时毕竟是基于采样的方式的 所以会有些状态没有被采样到，这就产生的偏差 在后面的DDQN里面 会有这方面的思考与解决；

## 深度强化学习中的DQN里面的experience replay方法和A3C的asynchronous

显而易见的 前者的经验重放就是一种标准的off-policy的方法，毕竟在DQN里面使用的时候是在对估计Q(s,a)，estimate_Q(s, a) = r + lamda * maxQ(s2, a) ，如果按照on-policy理解的话 这个时候的应该是estimate_Q(s, a) = r + lamda * Q(s2, a2)  也就是说这个时候应该保存当时候的网络模型来进行输出对应的Q值情况，进而进行基于相应的值函数 在基于相同的如ε-greedy来选择情况；现在经验重播中只保存了$<s,a,r,s_{next}>$ 谈何on-policy呢。

同时上面也说了是直接保存的交互数据进行训练，这样观察数据往往波动很大且前后sample相互关联「机器学习也要求样本彼此独立同分布」 经验重播的方法很不适合on-policy；

但多线程的synchronous不一样；因为是多个agent在多个环境实例中并行且异步的执行和学习。 数据就不存在上面所说的数据关联性的问题。多个并行的actor可以有助于exploration。在不同线程上使用不同的探索策略，使得经验数据在时间上的相关性很小。这样不需要DQN中的experience replay也可以起到稳定学习过程的作用，意味着学习过程可以是on-policy的。 

## 基于值函数的方法和基于策略的：

上面说了这么多，其实还是没有脱离一个值函数。总结性的说一句：值函数近似的方法是通过描述构建起来一个评估函数：值函数「评估策略的好坏的」，进而优化这个值函数 使之达到最优，当其最优的时候，作为最能描述策略好坏的值函数，那么此时 采用**贪婪策略「确定性策略」或者ε-greedy「随机策略」** 在某状态 选取对应的action，也就是当前状态所选的最优动作， 整体来说 也就是最优策略了；

如果拿「优化和学习」的区别来看待这个问题，值函数方法里面就是借助优化和学习值函数，以此为评判标准来获取一个好的策略了。进而值函数分为状态值函数`V`和动作值函数`Q` ，一般来说 我们选取动作的时候 还是基于后者的值来选取的，这样就显然带来一个问题就是 如果是连续动作 不能对每个动作都给予一个Q值 因而在连续动作集合中就不能有很好的表现；

于是引入了直接的策略搜索的方法；我们先说一下关于两者实现的区别：值函数的方法提过很多遍了，是需要先构建好值函数 然后优化好值函数，才能得到策略；而策略搜索的方法不同 则是使用一个参数化的线性或者非线性函数用以描述策略 然后寻找最优的策略，是强化学习的目标：累计回报的期望 最大；

于是 策略搜索 顾名思义的也正是在策略空间直接进行搜索，不再借助值函数那样的间接完成对于最优策略的确定；于是也带来了收敛性快这样的好处；方法包括策略梯度 DDPG等

策略梯度的方法收敛性快，可以有效地处理连续动作集的问题「值函数方法无法确定一个对应max Q的action」，但同样的容易收敛到局部最小值、方差较大。「这里的方差较大的原因其实和蒙特卡洛方法类似 每次交互都会产生一整条轨迹 然后基于这个策略更新的时候 会导致回报估计的波动 因而会导致高方差」

接下来 就从似然函数的角度来推导一下策略梯度：

上面 我们说过无论是基于值函数还是基于策略搜索的方法，本质上的目的也都是强化学习的目的：让累计折扣回报最大化；对于一个马尔科夫过程 所产生的一组状态-动作序列 我们用$\tau $表示$s_0,a_0,s_1,a_1...s_H,a_H$

那么对于这么一组状态-动作对 或者说这一条轨迹的回报我们用 $R(\tau)=\sum^{H}_{t=0}R(s_t,a_t)$，而该轨迹出现的概率我们使用$P(\tau;\theta)$表示，那么显而易见的从强化学习的目标：最大化回报 这个目的出发，可以写作 $max_{\theta}U(\theta)=max_{\theta}E(\sum^{H}_{t=0}R(s_t,a_t))=max_{\theta}\sum_{\tau}P(\tau;\theta)R(\tau)$ 「从回报期望的形式 到写作与概率相乘的形式，之前似然函数的部分说过」

进而为了优化这个目标函数，常见的如GD 找到那个最快下降的方向，于是对其求导有：

$\nabla _{\theta } U( \theta ) =\nabla _{\theta }\sum\limits _{\tau } P( \tau ;\theta ) R( \tau ) $

$=\sum\limits _{\tau } \nabla _{\theta } P( \tau ;\theta ) R( \tau ) $

$=\sum\limits _{\tau }\frac{P( \tau ;\theta )}{P( \tau ;\theta )} \nabla _{\theta } P( \tau ;\theta ) R( \tau ) $

$=\sum\limits _{\tau } P( \tau ;\theta )\frac{\nabla _{\theta } P( \tau ;\theta ) R( \tau )}{P( \tau ;\theta )} $

 $=\sum\limits _{\tau } P( \tau ;\theta ) R( \tau ) \nabla _{\theta } logP( \tau ;\theta ) $

于是 梯度的计算转换为求解$ R( \tau ) \nabla _{\theta } logP( \tau ;\theta ) $的期望，此时可以利用蒙特卡洛法近似「经验平均」估算，即根据当前策略π采样得到m条轨迹 ，利用m条轨迹的经验平均逼近策略梯度 于是有：

$$\nabla _{\theta } U( \theta ) \approx \frac{1}{m}\sum\limits ^{m}_{i=0} R( \tau ) \nabla _{\theta } logP( \tau ;\theta ) $$

如此 我们就得到策略梯度的推导结果；如果直观的理解上面的公式的话，$\nabla _{\theta } logP( \tau ;\theta )$ 就是轨迹随着参数θ变化最陡的方向，而 $R( \tau )$ 代指了步长 影响了某轨迹出现的概率；

进一步的 由似然函数 损失函数中的log部分可以推导为只包含动作策略的 $log\pi_{\theta}(a;s)$ 「转移概率无θ可删去；具体步骤可以参考RL :introduction」，因包含策略的表示，根据需要有着多种方式「具体参考RL:introduction」本文我们仅通过实例来进行说明 所构建出来的损失函数；

然后 参考现在的策略梯度中loss function的表示是 $logP(\tau;\theta )R(\tau)$ ；我们使用轨迹回报直接表达整个序列的价值也是根据定义来的毫无错误。但毕竟是基于model-free的，数据的产生来自于试验中的交互，导致着交互并不能很好的表示轨迹真实的期望 就和蒙特卡洛方法类似，会产生较大的方差「agent和环境交互 每一时刻可能产生很多结果 进而产生的一条马尔科夫链可以有多种可能，采样的至少其中一种，所以和所有路径的回报期望还是差距很大的」

**如何解决策略梯度这种因为采样问题导致的高方差呢？**思路就是考虑之前的TD算法了，或者说是基于值函数的方法：构建出来一个独立的模型来估计模型的长期误差，而不是单纯的使用轨迹的真实分布「有点像是从蒙特卡洛到TD的区别」

进而产生了作为基于值函数方法和基于策略梯度方法的actor-critic方法；

### 基于值函数的方法和基于策略的交叉点的AC方法

actor是基于策略梯度的方法进行选取动作，而critic是基于值函数的方法来评价它，两者协作完成；

于是我们可用先从策略梯度的角度来理解：

-   普通的策略梯度中loss function的表示是 $logP(\tau;\theta )R(\tau)$ 前者代指的是方向,进而基于θ求偏导的话 毋庸置疑也就是让轨迹τ的概率变化最快的方向，或快速增加或快速减少，只是取决于正负号；后者的是$R(\tau)$ 作为一个标量 类似于前者的增幅 当其为正的时候，当这个值越大的时候 轨迹τ出现的概率$P(\tau;\theta)$ 在参数更新之后会越大「建立的函数是两者相乘 但更新的参数只是在前者这个描述出现概率的变量中出现」反之则越来越小。所以，策略梯度的方法会增大高回报轨迹对应的出现概率 而会降低低回报轨迹对应的出现概率；
-   从这个角度再来理解actor-critic，轨迹的回报$R(\tau)$ 可看做一个critic；用于评价参数θ更新后 该轨迹出现的概率是该增大还是减少 以及对应的幅度；对应的可以表示成：

![](https://ws1.sinaimg.cn/large/005A8OOUgy1fu4zxc8pctj30h90ev0vz.jpg)
    

```
>   对于他们的评价如下：
>
>   1--3：直接应用轨迹的回报累积回报，由此计算出来的策略梯度不存在偏差，但是由于需要累积多步的回报，因此方差会很大。
>
>   4—6: 利用动作值函数，优势函数和TD偏差代替累积回报，其优点是方差小，但是这三种方法中都用到了逼近方法，因此计算出来的策略梯度都存在偏差。这三种方法以牺牲偏差来换取小的方差。当Ψ_t取4—6时，为经典的AC方法。
```

于是一个广义的AC框架 就是：前面的 $\pi_{\theta}(a|s)$ 是actor作为策略函数；
后面的 $\varPsi _t​$ 是critic 评价函数 「换句话说 critic使用的就是各种策略评估方法」

### Actor - Critic的优点

（总的来说就是结合两种方式优点）：$\varPsi_t$

-   相比以值函数为中心的算法，Actor - Critic应用了策略梯度的做法，这能让它在连续动作或者高维动作空间中选取合适的动作, 而 Q-learning 做这件事会很困难甚至瘫痪。
-   相比单纯策略梯度，Actor - Critic应用了Q-learning或其他策略评估的做法，使得Actor Critic能进行单步更新而不是回合更新，比单纯的Policy Gradient的效率要高。最重要的还是不再使用采样得到的真实回报 降低了因为采样率导致的方差

因为AC的方法，是在之前的策略梯度的方法之上 将其中的用于描述轨迹累计reward的项 转化为了行为价值函数 $Q_{\omega}(s,a)$  同时这里的状态值函数，接下来是一个简单实现的算法 行为值函数采用线性表达 于是更更新的过程中：critic通过近似TD(0)来更新ω，actor借助策略梯度更新θ，整体描述有：

![](https://ws1.sinaimg.cn/large/005A8OOUly1fvcxnjry2qj30gf09xdga.jpg)

可以看到 正是个实时的在线学习；针对每一步 一方面基于策略选取动作a和a‘，这时候的策略选取不再需要ε-greedy来获得，而是直接借助参数θ得到；然后得到TD-error，先类似于策略梯度中那样更新actor的参数θ；再更新critic中的参数ω用于生成值函数；

### Actor - Critic的总结

「回头看整个AC架构，就是在策略梯度更新的基础上 将其中的的元被用于描述整个轨迹的累计reward的$R(\tau)$ 改写成动作值函数；前者的策略梯度本身就是对于策略本身的描述 于是借助于它 我们能基于状态得到action；然后改写成的动作值函数 可以得到对于该state-action的评价值；进而回头类似于策略梯度的更新 更新这个策略」

再回头对比想想看和之前两者的区别：

1.  和策略梯度的区别，很明显 关于策略梯度的算法表述里面 并没有涉及关于critic部分或者具体点说就是值函数更新的部分；参考RL:an introduction 里面所描述的那样P292中所描述的那样： 策略梯度方法里 状态值函数更多的是作为一种基准而非是critic；也就是说只是作为一种基准来判断哪个状态需要被更新「for state-function」**而事实上 为了实现Policy Gradient，不管我们是计算Q，还是V，都需要一个对应的网络，这就是Critic。换句话讲，我们只有在使用Policy Gradient时完全不使用Q，仅使用reward真实值来评价，才叫做Policy Gradient，要不然Policy Gradient就需要有Q网络或者V网络，就是Actor Critic。** **「对于其中采用什么值来当做$R(\tau)$ 参考这来理解https://zhuanlan.zhihu.com/p/26882898」**
2.  和值函数的方法 差别就更大了；首先对于策略的描述 就不是值函数方法里面借助 $argmax_a Q(s,a)$ 而是使用的策略梯度的方法；当然使用对于该策略的好坏的评价 但也是一样的值函数；

然后对于一整个actor-critic的训练过程如下：

![](https://ws1.sinaimg.cn/large/005A8OOUgy1fu4zxspcrzj309w09st8t.jpg)

如图 基于策略梯度的actor基于概率来对于某状态来选取动作action；而critic基于actor的行为判别行为的得分；actor进而基于该评价值来计算出来一个td error修改选择行为的概率「换句话说就是：actor的策略梯度的方法生成得到梯度的方向「也就说之前的$logP(\tau;\theta )$」，然后进行沿着方向进行梯度的增减；我们需要一个值来判断这一个增减的方向是否正确 于是需要critic来计算出来td error」，同时基于选取的action计算TD error来更新critic；

具体到网络里面：Actor和Critic各为一个网络，Actor输入是状态输出的是动作，loss就是$log_{prob}\times td_{error}$,(和策略梯度相对应，注意到这里的loss和Policy Gradient中的差不多，只是vt换成td_error，引导奖励值vt换了来源（Critic给的）而已)，Critic输入的是状态输出的是Q值，loss是square((r+gamma*Q_next) - Q_eval)也就是square(td_error)，也就是说这里更新critic对应Q-learning是一样的均方误差。 

网络的交互也和上图一致：agent每次状态1从actor中得到一个动作a1 和env类交互得到s2和即时奖励r。然后把s1s2 r输入critic网络，更新其中的参数ω并计算得到td_error；然后把a1，s1，td_error输入到actor网络更新其中的参数θ；「TD_error信号同时指导actor网络critic网络的更新 」

## DQN及其相关变形和A3C

emmmm 其实这部分还是应该放在值函数的方法里面的吧，不过这已经算是深度强化学习的范畴了单独出来也没什么问题；
> 这里的几种不带有实现代码 emmmm 后面还会出一个带有代码的对比版本；

## DQN

通过上面的介绍，我们对于off-policy这个概念有了一定的了解，进而对于qlearning也一定很容易就能看懂了。进而我们就先来看看DQN对于qlearning有哪些改变：

![](https://ws1.sinaimg.cn/large/005A8OOUly1fvwhxw05j4j30rr0kzdyh.jpg)

![](https://ws1.sinaimg.cn/large/005A8OOUly1fvwi4d1xkyj30s60huk23.jpg)

![](https://ws1.sinaimg.cn/large/005A8OOUly1fvwi5sut8bj30lq0e9jvo.jpg)

首先直接放一下2015版的DQN的伪代码、网络模型、算法流程图；

很显然DQN相比于qlearning主要有三处改变：

1.  首先DQN采用了深度卷积神经网络来进行的值函数逼近，尽管这也不是什么新鲜操作了；但加入了第二点 就完全不同了；
2.  加入了经验回放的操作来训练强化学习；首先我们要知道 如果直接借助强化学习交互产生的数据 本身是带有关联性的，而在神经网络或者直接说机器学习中 对于数据的基本要求就是独立同分布。因而这里引入了经验回放这个操作 来打破了数据间的关联性，具体操作就是：agent在与环境交互的时候 将交互数据存放在一个库里面，然后训练的时候 从中随机采样数据进行训练；
3.  加入了目标网络的概念来单独的处理TD算法中的TD偏差；我们都知道 强化学习在表格式的时候 直接使用$Q^*(s,a) = Q(s,a)+\alpha(r+\gamma \max_{a'}Q(s',a')-Q(s,a))$ 来对于值函数就直接更新了；但函数近似后不行，现在的值函数本质上就是个函数的形式如$Q=f(\theta;feature)$ ，那么对于它的优化与更新也更多的是采用诸如梯度下降等方法来对于权值θ进行更新，有：$TargetQ = r+\gamma \max_{a'}Q(s',a';\theta)$ 和 $L(\theta)=E[(TargetQ-Q(s,a;\theta))^2]$ 。**因而目标网络的改进就在于这里的$TargetQ $使用了一个单独的网络** ；「emmm 这么一说 本来qlearning里面就不是按照交互顺序 而是使用的最大值来得到的对于当前值函数的估计，现在又进一步打破关联性 使用另一个网络的输出来进行估计 进一步减少关联性」也就是说现在有着$TargetQ = r+\gamma \max_{a'}Q(s',a';\theta^-)$ ，对于这个target 网络的权值$\theta^-$ 和前面的计算Q值的主网络不同，主网络是每一步一更新，而target网络每隔一段时间一更新；「联系后面的DDQN的话 两者的区别其实也就在于：DDQN是在选取动作的时候基于一个网络 而评价是基于另一个网络；也就是$TargetQ=R+\gamma Q(S_{t+1},argmax_aQ(S_{t+1},a;\theta_t);\theta'_t)$ 可以看到区别就是对于targetQ构建的时候 next state的action和q值选取还是不一样的」

说完DQN 然后就是它的几种变形了；

## DDQN

对于DDQN，本质上是构造了两个Q网络，

Double DQN产生的直接原因是常规的DQN在给定状态下往往会高估具有潜力的行动。如果所有的行动总是被同样高估的，那么这个情况也不错，但是事实并非如此。你可以想象一个情形，次优的行动经常得到超过最优行动的Q值，此时agent将很难习得最优的策略。为了纠正这个错误，DDQN的作者使用了一个简单的技巧：利用主网络选择行动，目标网络来生成该行动的目标Q值，而不是在训练过程中计算目标Q值的同时选择最大Q值对应的行动。将行动选择从目标Q值生成逻辑中抽离出来后，网络高估行动的问题基本得到了解决，训练也更加快速和可信。 

参考<https://www.cnblogs.com/wangxiaocvpr/p/5620365.html>；

还有书的P103来看；

### 首先对于double qlearning的理解

>   emmm 首先碎碎念一波对于P101中的两个值函数更新公式里面的TD target用了两个不同的表示方法，实际上说的都是一个意思，只是看法不同 实际上在写代码获取reward的过程中，我们都是对env类输入action，于是获得reward next_state, is_done， info等等信息；于是这时候的reward我们可以看做 这样基于state选取action后的当前state的reward；也可以看做 和后面next_state对应的reward；看你对于reward认定的角度了，是认为和state对应「对应后面的」 还是和state-action对应「」；

然后就对于qlearning；都知道对于异策略的qlearning「行动策略是ε-greedy；目标策略是贪婪策略」来说，无论如何对于策略的判定 也就是q值 **值函数的更新都需要一个max操作**，于是导致了一个结果就是：估计的值函数是比实际的值函数要大的；

然后对于这个被过估计的程度，如果这个过估计的程度 是均匀的；也就是类似对于全部的值函数都是一样的扩大，那么对于：以找到最大 值函数 对应的action的贪婪策略 为最优策略的RL，对于最后结果是不影响的，毕竟它只是要策略也就是选中对的action 而不是管你的q值函数是不是正确；

但是事实上 这个过估计的程度并不是均匀的在实际问题中，于是导致过估计成为了影响策略决策的问题所在，导致选取的action并不是最优 导致选取的不是最优策略；

对于qlearning来说，其中的动作选择让最大的那个action  ；然后是动作评估 也就是基于选取出来的action，利用其值函数构造出来TD目标 ；

这时候对于基于值函数近似「比如神经网络」来说 用式子表达就是 ，此时无论是前期选取action还是后面构造TD目标 都是基于某一step的网络结构参数 ，换句话说：选取action时候 需要先判断对于这一状态的全部action对应的Q值 这时候的需要的网络结构的参数θ；评价action时候 构造TD目标 也需要对应当前参数θ的输出g「一个包含对应S_t+1全部action对应Q的数组」根据上面动作选择的action 选择那个对应该action的Q值 然后再对应构造D目标 ；

也就是说有：

而对于double qlearning来说，这里的动作选取和动作评价就是不同的Q值函数了 也就是其中对应的θ是不一样的 于是有：；可以看到这时候关于动作的选取依旧是 基于当前值函数网络的参数 是基于online weight  选取出来那个对应最大Q值的action  ；但是对于策略的评价 也就是这个action选取的评价 采取另一个weight  来输出一个Q值来构造D目标 ，可以让策略评价更为公正；

之后就是整体的带有经验回放的double DQN了，貌似文章里面没有说 是参照P103来写的：

## Dueling DQN

「只要评估动作的好坏的advantage 而不用在于关于状态的好坏的V」

「主要是对于DQN的结构的改进」本文的贡献点主要是在 DQN 网络结构上，将卷积神经网络提出的特征，分为两路走，即：the state value function 和 the state-dependent action advantage function.  **好处**：在对底层强化学习算法不发生改变的情况下 泛化学习过程中的action，换句话说 当有着很多相似值action的时候 有着更好的策略估计；

因而在构造网络的时候 dueling network 将后续的输出分为了两个分支，一条输出标量的关于状态的价值，另外一条输出关于动作的Advantage价值函数的值  ;  具体来说 就是：在评估Q (S,A)的时候也同时评估了跟动作无关的状态的价值函数V(S)和在状态下各个动作的相对价值函数A(S,A)的值 ；

相比于上面的DQN直接输出的是Q函数的值（action的维度 ）评估了各个action，对于大多数状态来说可能不必要评估全部的action；

the dueling architecture 可以学习到哪一个状态是有价值的 （which states are (or are not) valuable），而不必学习每一个 action 对每一个 state 的影响。这个对于那些 actions 不影响环境的状态下特别有效**「针对那些action对于环境状态影响不是那么明显的状态」**。 

图中是不同时间点下value和advantage所关注的图，可以看到在无论哪个时间点里面 value所关注的一直是路面或者说是其中的地平线horizon「也就是行进的路线」；而advantage不关注视觉的输入，只关注的是有没有车辆会导致发生碰撞，于是在上右图中 advantage没有关注的东西，在下右才会关注到车辆 因为会导致其发生碰撞；所以动作更受advantage的影响；

## Deep Recurrent Q-Network

「学习到时间依赖」「在split成value和advantage 之前再加上一个LSTM 」

-   **部分可观察性**：作为人类，接触的世界往往是多变且受限的；但我们依旧可以基于这些片面的知识来解决生活中的众多问题；但一般情况下 比如之前的那些agent在训练的时候 agent是被给予关于环境的全部信息「已知关于环境的全部信息」进而选取最优的动作；量化到问题中也就是：每一帧里面 Env类会给予agent需要的全部的信息 进而使其max累计回报；进一步说的话，就是有关observation和state的区别，后者会描述环境的状态情况，而前者只是来自于agent对于环境的观测到的状态情况；而这一观测情况 往往存在着对于所需信息理解上的受限；比如仅仅是一张乒乓球的运动照片 尽管可以确定球的位置信息 但是无法确定球的速度和方向信息；那么对于这种部分观测的世界情况的解决思路，就是建立一种基于时间的观测量；也就是说当某状态内的信息不足以解决做出足够适合的决策的话，那么就随着时间的累计到足够的信息进而做出决策；再回到刚刚那个乒乓球问题的话，单张的图片不足以确定有关运动方向速度等信息，但是两张的话就可以确定关于运动方向，一序列图片的话 可以确定球的运动速度信息；
-   **部分可观察在RL**：就像上面所说的那样：建立一种基于时间关系的observation集成 进而完成对于信息的整合；常用的方法 比如在经典的[Human-level control through deep reinforcement learning](https://www.nature.com/articles/nature14236)里面对于网络层的输入，并非是单独的一张图片，而是把四张图片做一个外部缓冲区合并 然后作为输入到神经网络「当然 文章里面对每一帧做了灰度处理后 合并输入 也就是batch∗width∗higth∗4batch∗width∗higth∗4 ，说是这样说其实更类似于抽样 每四帧抽样一次 对应的状态图片和action，但是和抽样不同的在于 把中间隔着的三帧数据也作为了状态那一帧的一部分作为状态进行输入了，当然action的话 还是状态帧的那个」当然 不代表这个方法就是好的，毕竟只是在那些简单的游戏中有着较好的表现；实际上 不一定就是正确的 存在一系列的问题 比如存储这些缓存的话需要很大的内存；同时 对于整个事件「某个需要作出决策的事件」来说 单单几帧的数据并不能进行代表 于是需要一种更为鲁棒的方法来对于这个事件进行描述；
-   **Recurrent Neural Network** ：但无论如何 agent都是需要基于时间集成的；于是借助能够学习时序依赖「temporal dependencies」的RNN就很适合；将RNN作为黑箱加入现有的DQN，于是将agent的有关环境的单帧数据输入，网络能够基于接收到的观测值的时序模式「temporal pattern」来改变输出；对于每一个time step其计算出来一个隐状态，而RNN的模块可以吧之前的隐状态反馈到自身 进而告诉网络 之前发生了什么 ；这种加入RNN的DQN 就是 Deep Recurrent Q-Networks (DRQN).「换句话说就是加入RNN来学习到时序模式」

这个时候对于DQN里面的experience buffer的修改；因为我们希望训练网络学到关于时序的关系，于是就不能随机选取经验缓存区里面的经验凑成batch 来用于训练；而是应该基于给定长度的来选取一段 也就是所谓的「draw traces of experience」；下面的 experience buffer 存储了确定的episodes，对episodes里面的随机批次抽取8steps「其实就是原本 对某个episode里面随机抽取需要个数的experience；改为了随机选一个起点 然后连续抽取设定的几个experience」 

## A3C

## 异步（asynchronous）

A3C的全名是Asynchronous Advantage Actor-Critic ，顾名思义 是异步的架构；并非像DQN那样仅单智能体与单环境交互，A3C可以产生多交互过程；如上图可知 A3C包括一个主网络 和多个工作的有着各自参数的agent，同时的和各自的环境进行交互；相比于单个的agent 这种方法更有效之处在于 因为来自各自独立的环境 于是采集得到的经验也是独立的 于是采集得到的经验也更多元化；

## actor critic

说完了异步（asynchronous） 然后说到actor-critic，相比于单纯的值迭代方法如：qlearning 和 策略迭代方法如：策略梯度的方法；AC有这两者的优势；在本实践之中，本网络可以既估计值函数V(s)V(s)「某确定的的状态有多好」而且还能输出策略π(s)π(s)「动作概率输出的集合」；参考上图知道 也就是在全连接层之后的不同输出；AC方法的关键之处在于：agent采用值估计「critic」来更新策略「actor」相比于传统的策略梯度的方法更有效；

## advantage

回头看一下关于之前策略梯度的完成，其中关于损失函数的构建是直接使用的reward的折扣累计来实现的；emmmm [关于这部分策略梯度损失函数两部分的含义可以参考这个网页 或者是那个A3C.md](https://zhuanlan.zhihu.com/p/26882898) 于是在之前的策略梯度中的网络是根据这个折扣reward得到的关于该action好与坏的评价来进行对该action的鼓励亦或是阻止；这里采用advantage来替代传统的折扣reward累计；不光是能让agent判断action是否是好 而且能让其判断比预期好多少；intuitively （直觉的） 这样可以允许算法去更针对那些预测不足的区域 具体的可以参考之前的 dueling q network的架构：其中关于 Advantage:A=Q(s,a)−V(s)Advantage:A=Q(s,a)−V(s);在A3C中并不能直接确定Q值**(?)** 于是这里采用折扣reward来作为Q(s,a)的估计 来生成关于advantage的估计：A=R−V(s)A=R−V(s)同时 本教程之中又进行了修改 使用的是另一种有着更低方差的版本：[Generalized Advantage Estimation.](https://arxiv.org/pdf/1506.02438.pdf)

经典的A3C算法 是在actor-critic的基础上 采用了并行的结构运行，即它不在利用单个线程，而是利用多个线程。每个线程相当于一个智能体在随机探索，多个智能体共同探索，并行计算策略梯度，维持一个总的更新量。 

由经验可知道：online的RL算法「更新策略和选取策略一致」在和DNN简单结合后会不稳定。主要原因是观察数据往往波动很大且前后sample相互关联。像Neural fitted Q iteration和TRPO方法通过将经验数据batch，或者像DQN中通过experience replay memory对之随机采样，这些方法有效解决了前面所说的两个问题，但是也将算法限定在了off-policy方法中。 文章中，通过创建多个agent，在多个环境实例中并行且异步的执行和学习。于是，通过这种方式，在DNN下，解锁了一大批online/offline的RL算法（如Sarsa, AC, Q-learning） ；

>   「对于单个agent进行样本采样 获取的样本很可能就是高度相关的；而 machine learning 学习的条件是：sample 满足独立同分布的性质。在 DQN 中，我们引入了 experience replay 来克服这个难题。但是，这样子就是 offline 的了，因为你是先 sampling，然后将其存储起来，然后再 update 你的参数。  」

文章中不只是包含一种算法，而是将one-step Sarsa, one-step Q-learning, n-step Q-learning和advantage AC扩展至多线程异步架构。 可以看到几种方法的不同： **「AC是on-policy的policy搜索方法，而Q-learning是off-policy value-based方法。这也体现了该框架的通用性。 」**

简单地说，每个线程都有agent运行在环境的拷贝中，每一步生成一个参数的梯度，多个线程的这些梯度累加起来，一定步数后一起更新共享参数。 

所主要针对的还是产生多个独立环境，有多个 agent 对网络进行 asynchronous update，这样带来了样本间的相关性较低的好处，因此 A3C 中也没有采用 Experience Replay 的机制；这样 A3C 便支持 online 的训练模式了 ；

具体点说的话 就是启动了多个训练环境，同时进行采样 然后直接使用各个环境采集得到的样本进行计算梯度 训练更新相关的参数「actor中的policy gradient和critic中的值函数」

实践的话要有两套体系, 可以看作中央大脑拥有 `global net` 和他的参数, 每位玩家有一个 `global net` 的副本 `local net`, 可以定时向 `global net` 推送更新, 然后定时从 `global net` 那获取综合版的更新. 