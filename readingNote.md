# The Electric Fleet Size and Mix Vehicle Routing Problem with Time Windows and Recharging Stations
## abstract
* 问题：Electric Fleet Size and Mix Vehicle Routing Problem with Time Windows and Recharging Stations (E-FSMFTW) to model decisions to be made with regards to fleet composition and the actual vehicle routes including the choice of recharging times and locations.
* 方法：We solve this problem by means of branch-and-price（分支定价） as well as proposing a hybrid heuristic（混合启发式）, which combines an Adaptive Large Neighbourhood Search with an embedded local search and labeling procedure for intensification
## introduction
### contribution fo this article
MIP model(混合的整数规划模型——非凸)来数学化该问题，利用分支定价法来解决
we propose a metaheuristic approach(启发式方法) based on Adaptive Large Neighbourhood Search (ALNS)（自适应大邻域搜索） with embedded local search（嵌入式本地搜索） and labeling procedures(标签程序).
(核心是把问题变为MIP，对于节点较少的场景，使用分支定界等方法来求解；对于节点较多的场景，使用启发式算法来求解)
## Problem description and model
E-FSMFTW consists of finding admissible tours for vehicles of different types such that every customer is covered by exactly one tour while minimizing acquisition costs and the total distance travelled.每个客户都被正好覆盖一次（高能辐射场有的点需要被覆盖好几次）
Our formulation is based on Bräysy et al. (2008a) for the FSMVRPTW and Schneider et al. (2014) for the E-VRPTW. As for the E-VRPTW, a vehicle is also as- sumed to always recharge to full capacity every time it visits a recharging station.每次充电都充满（辐射场里面人可以回到起点，但是累积的受辐射量不会减少，不存在被“充电”）
###　Mixed integer programming model
E-FSMFTW consists of 
C：a set of customers 一系列客户（一系列只需单人完成的任务点）
F：a set of recharging stations一系列充电站（一系列需要多人完成的任务点，但是不涉及需要同时完成，或者需要先后的任务）
a depot node 一个仓储点（起点/终点）
k different vehicle types k种卡车类型（k种受辐照影响不同的员工）
N 是客户节点C和虚拟节点F'的集合，F'为了对多次访问充电站计数
种类k卡车最大装载量和当前装载量（k类人的任务能力和null）
种类k卡车最大能源总量和当前能源量（k类人的最大受辐射剂量和当前受辐射剂量）
单位距离消耗的能源量（单位距离受辐射量）
每单位能源量充电时间
service time of each node每个节点所需的服务时间（不同任务执行所需时间）
the actual start of service time实际开始服务时间（执行任务开始时间）
节点i的时间窗口（任务点i的时间窗口）
The distance and the travel time between two nodes i and j（任务点i和j之间的距离和行走时间）
* MIP model
* Set partitioning formulation(怎么翻译?集合划分公式？)
## Branch and price algorithm

### Labeling for the ESPPTWRS
In this work,the pricing problem is solved using a bi-directional labeling algorithm.本文中的定价问题使用双向标签算法解决。

# SHORTEST PATH PROBLEMS WITH RESOURCE CONSTRAINTS
## Introduction
column generation(also known as branch-and-price when embedded in a branch-and-bound framework)成功解决了大量的车辆路径和员工分配的问题
在大多数这样的应用中，列生产的主问题是带有边界限制的集合划分或集合覆盖。大多数的变量和车辆路径以及人员工作安排有关，路径和任务分配变量由一个或多个子问题组成，每个子问题都和一个SPPRC（shortest path problem with resource constraints）问题或其变体对应。
SPPRC为列生成成功解决这类问题贡献了以下三点：
1. 通过它的资源限制，它为模型化可行路径的复杂成本结构和大量的路径可行性规则构造了灵活的工具。
2. 它不具有整体性（最优值和线性松弛值之间可能存在正差），列生成方法比基于弧的公式的线性松弛获得的边界更加严格。
3. 已有一些能够有效解决至少是SPPRC的重要变体的算法。
REFs：resource extension functions —— A resource corresponds to a quantity, such as the time, the load picked-up by a vehicle, or the duration of a break in a work shift, that varies along a path according to functions. A REF is defined for every arc in the network and every resource considered. It provides a lower bound on the value that the corresponding resource can take at the head node of the corresponding arc, given the values taken by all the resources at its tail node.（REF是为网络中的每条边弧定义的并且每个资源都被考虑。它为值提供下界，相应的资源的值会在相应的边弧的头节点获取，被资源获取的值将提供给尾节点）
资源窗口：The resource constraints are given as intervals, called resource windows, which restrict the values that can be taken by the resources at every node along a path.
案例介绍SPPRC，SPPRC很类似于multi-criteria problem（多准则问题）
SPPTW：The two-resource SPPRC, better known as the shortest path problem with time windows.

## Classification of the SPPRCs
各种SPPRC的变体，就是经典最短路问题的拓展，只是将成本用多维的资源向量代替了，这些多维向量在路径过程中累积，在中间节点被限制。不同类型的SPPRC是由以下划分：
1. 资源累积的方式，指向不同资源可行路径的定义
2. 排除特殊路径（比如说：非基础路径）之外，还存在其他路径结构约束
3. 目标
4. 底层网络

### Resource feasible paths
资源限制可以由（最小化）资源消耗和资源间隔构成
REF：可以理解为从起始到中间某个点的累积资源消耗
### Path-structural constraints
路径结构约束可以为关于路径可行性的进一步需求建模，这是资源所不涉及的
ESPPRC（elementray SPPRC）与SPPRC区别在于，ESPPRC不允许一个节点被多次访问
precedence constraints and pairing constraints：Given two nodes i, j, a path P fulfills the (i, j)-pairing constraint if node i occurs as often as node j in P (possibly P contains none of them). （配对约束：两个节点i,j发生的频率一样）A path P fulfills the (i, j)-precedence constraint if P contains no subpath connecting j with i.（优先约束：没有从j连接i的子路径）
在分支定价背景下，每个节点和每个边都代表一系列任务，一个任务和主问题的分区约束关联。同理，一个任务可以是几个序列的一部分可以由几个节点和边弧表示。对于任何的路径都对应串联的任务序列。所有的路径结构约束也许都能由任务序列表示。
### Objectives and generic SPPRC formulation
SPPRC的目标由可行路径最后一个节点上的资源向量确定
SPPRC的更一般的公式是基于帕累托最优资源向量集的考虑

## Properties of T（P）
## Modeling issues
## Solution methods
dynamic programming(动态规划) which has been used extensively, Lagrangean relaxation（拉格朗日松弛）, constraint programming（约束规划）, and heuristics（启发式）
### Dynamic programming and labeling algorithms
动态规划：从起始点开始，通过一个一个的探索所有的可行方向来构建新的路径。它的效率取决于，识别和丢弃不能构建帕累托最优路径集或难以扩展成帕累托路径的能力。
为了提高效率，动态规划算法中的路径采用标号(label)编码。
共享同一前缀的路径用一条表示它们共同前缀的单链表示，使用树形数据结构来标注路径。除了编码路径，标签存储一个代表性的资源向量(resource vector)。


VRP介绍：
https://wiki.mbalib.com/wiki/%E8%BD%A6%E8%BE%86%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98
多目标：
https://blog.csdn.net/u011622208/article/details/64440843
遗传算法求解VRP：
https://zhuanlan.zhihu.com/p/125779424?utm_source=wechat_session

论文：
基于客户重要度的混合时间窗车辆路径问题研究



# The time-dependent pickup and delivery problem with time windows
本文研究带有时间依赖的取货和时间窗口发货问题，目的在于在操作灵活性的两个维度下优化运输公司的服务。首先，考虑运输公司最大化其利润来响应运输需求；其次，运输公司通过告诉司机何时上路来充分利用交通顺畅的时间。提出了一种基于分支定价的精确方法来解决该系列问题，其中的列是由定制的标记算法(tailored labeling algorithm)生成的。



# mine
## abstract
The increasing need for safe, inexpensive and quick search, combined with the development of unmanned equipments, has made search missions by a team of UAVs (unmanned aerial vehicles) in the spotlight. When avaliable information of search regions are lacking, UAVs make real-time action decisions according to surrounding environments they have perceived. Navigating in an unkonwn area safely is counted as the underlying work which can support the UAVs team for more complex tasks. However, frequent and massive interactions with neighbor UAVs and environments are inevitable for collision avoidance. The task allocation and collision avoid between UAVs are also hard to balance. In this paper, we proposed a method that UAVs can provide assistances to surrounding UAVs by spreading the status information of them. This kind of status information contains ambient infromation and perceptions of the task which are transfered to reward data for convenient and uniform distributions. In other words, individuals utilize inter-neighbor interactions to achieve the same high-level goal, as well as result in an intelligent independent multirobot system. 

with collective efforts.
## introduction
Cooperation and collision avoidance are of equal importance for the multirobot system. When a fleet of robotic agents navigates in a shared workspace, there arised another risk that robots may frequently block each other's ways. (Complete Algorithms for Cooperative Pathfinding Problems) Trevor Standley and Richard Korf 

Without details of searching area, a fleet of robotic agents cannot rely on an unified action plan. 




# how to write
## abstract
介绍做的是什么，要点在哪里，用什么方法，
## introduction
第一段：具体介绍自己做的是什么，以及背景；用到了什么方法，有什么作用，涉及到哪些领域。
第二段：将第一段提到的不同方法，分别展开，这种方法有什么具体作用（类似于把abstrace拓展，细化）
第三段：