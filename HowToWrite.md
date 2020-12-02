
# how to write （conference）
## abstract
介绍做的是什么，要点在哪里，用什么方法，
## introduction
第一段：具体介绍自己做的是什么，以及背景；用到了什么方法，有什么作用，涉及到哪些领域。
第二段：将第一段提到的不同方法，分别展开，这种方法有什么具体作用（类似于把abstrace拓展，细化）
第三段：相关研究领域论述
第四段：我们工作不同在哪里
第五段：文章结构

## method
第一段：简要介绍该方法是怎么样的实现的
第二段：以一个例子介绍该方法，先将样例场景详细介绍
第三段及之后：开始讲其中的公式以及作用，对应样例场景图，以方便理解

## experiment




# mine
## abstract
The increasing need for safe, inexpensive and quick search, combined with the development of unmanned equipments, has made search missions by a team of UAVs (unmanned aerial vehicles) in the spotlight. When avaliable information of search regions are lacking, UAVs make real-time action decisions according to surrounding environments they have perceived. Navigating in an unkonwn area safely is counted as the underlying work which can support the UAVs team for more complex tasks. However, frequent and massive interactions with neighbor UAVs and environments are inevitable for collision avoidance. The task allocation and collision avoid between UAVs are also hard to balance. In this paper, we proposed a method that UAVs can provide assistances to surrounding UAVs by spreading the status information of them. This kind of status information contains ambient infromation and perceptions of the task which are transfered to reward data for convenient and uniform distributions. In other words, individuals utilize inter-neighbor interactions to achieve the same high-level goal, as well as result in an intelligent independent multirobot system. 
* 有问题，不要用multi-robot根据数量，还是要用swarm更好
* 本文与传统的方法不同的是，传统的很多文章只是利用邻居机器人的相对位置或相对速度来保持队伍形状或一致性，而并没有利用同伴的信息来避障。
* 之前的文章禁止了机器人的后退或左右变向（只能通过转向来改变运动方向），因为限制了传感器的感知方向（前向）。之后如果改成360度的感知，机器人的后腿和变向方式也会改变。
with collective efforts.
## introduction
Cooperation and collision avoidance are of equal importance for the multirobot system. When a fleet of robotic agents navigates in a shared workspace, there arised another risk that robots may frequently block each other's ways. (Complete Algorithms for Cooperative Pathfinding Problems) Trevor Standley and Richard Korf

Without a previous knowledge of the searching area, a fleet of robotic agents cannot rely on an unified action plan. 


# how to write (journal)
## abstract
## introduction
1. 两句话概况本文的目的
2. 由于机器人只能进行local感知，所以我们需要local control rule。这个local control rule要self-organization, fault-tolerance and self-repair
3. 本文的目标会遇到什么问题，使用的方法，及其相关研究领域，文章用这个方法做什么（用了几个方法写几个部分）
4. 本文的方法和目标
## 借鉴的别人的方法介绍（在本文的使用领域）
## 铺垫自己的方法（前面写的别人的方法，在这里可以做对比，看哪种方法好）
1. 整体的结构
2. 实验设计
3. 方法的详细介绍
4. 实验性能标准
5. 实验结果
6. 实验阐述
## 介绍自己的方法（ 前面的预训练结束了，这里开始讲本文提出的框架了）
还是要先讲背景和相关研究
1. 介绍方法
2. 介绍实验设计
3. 实验结果
