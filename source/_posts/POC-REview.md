---
title: TOC Review
date: 2021-11-11 11:08:38
tags:
---

# Theory of Computation  ReView

> 计算的要义就是在有限步内找到准确的解。
>
> 读书就是在识词，了解定义，尤其是形式化的那些。



## Chapter One 正则语言

Q: 为什么大多数计算模型是受限的

### 有穷自动机

$Definition1.1:$

$$M=(Q,\sum,\delta,q_0,F)$$

$Q$是状态集，$\sum$是字母表，$\delta:Q\times \sum \rightarrow Q$ ,$q_0$是起始状态, $F \subseteq Q$ 是接受状态集

特别的，$\delta$ 包含每一个状态与每一个字母对应的转化。

#### 语言

若A是机器M接受的全部字符串集，则称A是机器M的语言，记作L(M)=A,又称M识别A。一台机器可能接受若干字符串，但只会接受一个语言。

### 正则语言

$Definition 1.7$ 如果一个语言被一台有穷自动机识别，则称它是正则语言(Regular Language).

#### 计算

设$M=(Q,\sum,\delta,q_0,F)$是一台有穷自动机，$w=w_1w_2w_3w_4...w_n$是一个字符串并且其中任意$w_i$是字母表的成员，若存在Q中的一个状态序列$r_0r_1r_2...r_n$满足下述条件：

- $r_0=q_0$
- $\delta(r_i,w_{i+1})=r_{i+1},i=0,...,n-1$
- $r_n\in F$

则称M接受w。

#### 正则运算

$Definition1.10$ 设$A$和$B$是两个语言，我们可以定义得到正则运算**并(union)**，**连接(concatenation)**和**星号(star)**。

需要注意，正则运算的定义对象是语言即自动机。进一步证明了正则语言类在正则运算下的完备性，通过连接运算部分证明的困难引出了非确定性。

### 非确定型有穷自动机

$Definition 1.17$ 

$M=(Q,\Sigma,\delta,q_0,F)$

Q是状态集，$\sum$是字母表，$\delta:Q\times \Sigma _\epsilon \rightarrow P(Q),q_0$是起始状态,  $F \subseteq Q$是接受状态集

引入非确定型有穷自动机后，我们可以轻易证明正则语言类在正则运算下的封闭性。

#### DFA与NFA的等价性

$Theorem 1.19$ 考虑基于NFA $M$ 构建一个新的DFA $M'$, $M'$的状态集$Q' = P(Q)$，即可完成所有模拟。 相应的，我们构建出了$M'$。

- $E(R) =\{q|$从R出发沿着0个或多个$\epsilon$ 箭头可以到达q$\}$
- $Q' = P(Q)$
- $q_0'=E(\{q_0\})$
- $F'=\{R \in Q'|R$包含$Q$的一个接受状态 $\}$ 
- $\delta '(R,a) = \{q\in Q| \exists r\in R(q\in E(\delta(r,a))) \}$
- $\Sigma ' =\Sigma$

$Inference 1.20$ 一个语言是正则的，当且仅当有一台非确定型有穷自动机识别他。

### 正则表达式

$InductiveDefinition1.26$ 称R是一个正则表达式，若R是：

1. $a,a\in \Sigma$
2. $\epsilon$
3. $\varnothing$
4. $(R_1 \cup R_2 ),R_1R_2$都是正则表达式
5. $(R_1\circ R_2),R_1R_2$都是正则表达式
6. $(R_1)^\ast,R_1$是正则表达式



#### 正则表达式与NFA的等价性以及NFA向正则表达式的转化

$Theorem 1.28$ 一个语言是正则的，当且仅当可以用正则表达式描述它。

$Inference1.29$ 如果一个语言可以用正则表达式描述，则它是正则的。

该引理的证明是显然的，该引理的过程就是从正则表达式向NFA的转化。

$Inference 1.30$ 如果一个语言是正则的，则可以用正则表达式描述它。

该引理的过程就是从NFA向正则表达式转化的过程，详见Page:42。

### 非正则语言

$Theorem1.37PumpingTheorm$ 若A是一个正则语言，则存在一个数p(泵长度)使得，如果s是A中任一长度不小于p的字符串，那么s可以被分成三段，s=xyz，满足下述条件：

1.对每一个$i\ge 0,xy^iz \in A $

$2.|y|>0$

$3.|xy|\le p$

证明思路：其中p=自动机状态数，因为s长度不小于p，$s=r_0r_1..r_n$，若能计算，则计算历史为$q_0q_1...q_n$长度一定大于自动机状态数，根据$Pigeon holes'Theorem$，一定至少$q_i=q_j$。 

特别地，只能用泵引理证明语言不是正则的，若需证明语言是正则的，需找到相应的正则表达式。

## Chapter Two 上下文无关文法

### 上下文无关文法

$Definition2.1$ 上下文无关文法是一个四元组$(V,\Sigma,R,S)$,且：

1. $V$ 是一个有穷集合，称为变元集。
2. $\Sigma$是一个与$V$不相交的有穷集合，称为终结符集。
3. $R$是一个有穷规则集。
4. $S\in V$是起始变元。

#### 歧义性

$Definition 2.4$ 如果字符串$w$在上下文无关文法$G$中有两个或两个以上的最左派生，则称G歧义地产生某个字符串，则称G是**歧义**的。

某些上下文无关语言只能用歧义文法产生，称这样的语言为**固有歧义**的。 

#### Chomsky Normal Form

$ Definition 2.5$ 称一个上下文无关文法为Chomosky Normal Form，如果它的每一个规则具有如下形式:
$A \to BC, A \to a$


### 下推自动机

$Definition 2.8$ 六元组$(Q,\Sigma,\Gamma,\delta,q_0,F)$，其中$Q,\Sigma ,\Gamma ,F$都是有穷集合。

1. $Q$是状态集。
2. $\Sigma$是输入字母表。
3. $\Gamma$是栈字母表。
4. $\delta :Q\times \Sigma_\epsilon \times \Gamma_\epsilon \to \mathcal P (Q\times \Gamma_\epsilon)$是非确定性的转移函数。
5. $q_0 \in Q$是起始状态。
6. $F \subseteq Q$是接受状态集。

#### PDA 的计算过程

PDA接受输入$w=w_1w_2..w_p,w_i\in\Sigma_\epsilon$，若存在状态序列$r_0r_1..r_p,r_i\in Q$,和字符串序列$s_0s_1...s_p$ $s_i\in\Gamma^\ast$，满足下述条件:

1. $r_0 = q_0且s_0=\epsilon$，该条件表示$M$从起始状态和空栈开始。
2. 对于$i=0,1...p-1$有$(r_{i+1},b)\in \delta(r_i,w_{i+1},a)$其中$s_i=at,s_{i+1} =bt,a,b\in\Gamma_\epsilon,t\in\Gamma^\ast$ 
3. $r_m\in F$，该条件说明在输入结束时出现一个接收状态。

#### 非确定性PDA与CFL的等价性

1. $Inference 2.13$ 如果一个语言是上下文无关的，则存在一个下推自动机识别它。

$eg. S \to abSxy $，且S是起始变元，则对应产生的规则为$\delta (q,a,S)=\{(q_1,\epsilon)\},\delta(q_1,b,\epsilon)=\{(q_2,\epsilon)\},\delta(q_2,\epsilon,\epsilon)=\{(q_3,y)\},\\\delta(q_3,\epsilon,\epsilon)=\{(q_4,x)\},\delta(q_4,\epsilon,\epsilon)=\{(q,S)\}$  

2. $Inference 2.14$ 如果一个语言被一台下推自动机$M$所识别，则它能被CFG $L$派生。

使用$A_{pq}$变元代表从$M$的p状态及空栈，转移到q状态以及空栈所派生的字符串。则：

- $A_{pq}\to aA_{rs}b$, if $\delta(p,a,\epsilon)\to(r,u),\delta(s,b,u)\to(q,\epsilon)$
- $A_{pq}\to A_{pr}A_{rq}$
- $A_{pp}\to \epsilon$

### 非上下文无关语言

$Definition 2.19$ $PumpinglemmaforCFL$

如果$A$是上下文无关语言，则存在数p（pumping length），使得$A$中任何一个长度不小于p的字符串$s$都能被划分成5段$s=uvxyz$，使得：

1. 对每一个$i\ge 0,uv^ixy^iz\in A$
2. $|vy|>0$
3. $|vxy|\le p$

有一个显然的例子，$a^nb^nc^n \notin CFL$ 是因为使用泵引理进行分析，发现$v,y$都不能含有两种符号，不然会发生顺序错乱，但若仅含有一种符号，又不能保证三种字母的数量相同。

## Chapter Three 丘奇-图灵论题

$Definition 3.1$ 图灵机是一个7元组$(Q,\Sigma,\Gamma,\delta,q_0,q_{accept},q_{reject})$，其中$Q,\Sigma,\Gamma$都是有穷集合，且：

1. $Q$是状态集。
2. $\Sigma$是输入字母表，不包括特殊空白符号$\sqcup$
3. $\Gamma$是带子字母表，其中$\sqcup \in \Gamma,\Sigma \subseteq \Gamma$
4. $\delta: Q\times \Gamma \to Q \times \Gamma \times \{L,R \}$是转移函数
5. $q_0 \in Q$ 是起始状态
6. $q_{accept}\in Q$是接受状态
7. $q_{reject}\in Q$是拒绝状态

#### 格局 Configuration

显见，上文所定义的图灵机仅有三个自由度，分别是当前状态，当前带子内容和当前读写头位置。故此，这三者组合在一起称为图灵机的格局，对于状态$q$，和状态字母表$\Gamma$上的两个字符串$u和v$，以$uqv$代表如下格局：当前状态是q，当前带子内容是uv，读写头的当前位置是v的第一个符号。

对于任意输入，图灵机有三种可能的表现：接受，拒绝及循环，其中我们认为拒绝和循环都是不接受。若某图灵机对所有字符串要么接受，要么拒绝（有限时间内给出结果）则称该图灵机为判定器。

$Definition 3.2$如果一个语言能被某一图灵机识别，则称该语言是图灵可识别的(Turing-recognizable)。

$Definition3.3$如果一个语言能被某一图灵机判定，则称该语言是图灵可判定的(Turing-decidable)。



### 非确定型图灵机
#### 等价性

等价于确定型图灵机：我们可以使用一个确定性图灵机广度优先搜索的模拟一台非确定型图灵机。

已经证明：任何可以无限制地访问无限的存储器，且在一步中只能执行有限的工作量的计算模型的能力都是等价的。

### 算法的定义

> 希尔伯特第10问题：通过有限多次运算决定一个多项式方程有没有整数根。

$Formal Description:$ $D=\{p|p是有整数根的多项式 \}$ 是否是图灵可判定的。

$\to$  既然算法是用图灵机定义的，实则每个图灵机也对应一个被准确形式化描述的算法。

## Chapter Four 可判定性

### 不可判定性

$Theorem 4.9$ $A_{TM}$是不可判定的。

#### 不可识别性

$Inference 4.15$ 存在不能被任何图灵机识别的语言。

图灵机的个数是可数的，因为构成其的$Q,\Sigma,\Gamma$都是有穷集合。但要识别的语言是不可数的，字符串个数为$|\Sigma^\ast|$不可数，对应产生的语言为$L={w|w=01串，且为无限长度}$。

$Theorem 4.16$ 一个语言是可判定的，当且仅当它和它的补都是图灵可识别的。

1. 语言是可判定的$\to$补是图灵可识别的。

2. 它和它的补是图灵可识别的$\to$语言是可判定的。

## Chapter Five 可归约性

$Definition$ 如果A可归约到B，就可以用B的解来解A。说明解B的难度比解A更大。

$Inference$ 当A可归约到B时，如果B是可判定的，则A是可判定的。如果A是不可判定的，则B是不可判定的。

### 映射可归约性

#### 可计算函数

$Definition 5.12$ 如果存在一个判定器$M$，对于函数$f:\Sigma^\ast \to \Sigma^\ast$，使得在每一个输入$w$上$M$都接受，且只有$f(w)$留在带子上。

#### 映射可归约性

归约性的一种定义。

$Definition 5.15$ 语言$A$是映射可归约到语言$B$的，如果存在可计算函数$f:\Sigma ^ \ast \to \Sigma ^\ast$使得对每个$w$:$w\in A \Longleftrightarrow w\in B$。记作$A \le_m B$。

特别的，在用于证明非图灵可识别过程中，有一个典型手段。

$Inference 5.23$ 如果$A\le_m B$，且$A$不是图灵可识别的，则$B$也不是图灵可识别的。

而一个标志性的非图灵可识别的例子是$\bar{A_{TM}}$，而可知$A\le_m B \leftrightarrow \bar{A}\le_m \bar{B}$，所以为证明$B$非图灵可识别，可证$A\le_m \bar{B}$。

## Chapter Six 可计算性理论的高级专题

### 递归原理 

## Chapter Seven 时间复杂性

### 复杂性的度量

$Definition 7.2$ 设$f和g$是两个函数，$f，g:\mathrm{N\to R^+}$。称$f(n)=O(g(n))$，若存在正整数$c$和$n_0$，使得对所有$n\ge n_0$有：
$$
f(n)\le cg(n)
$$
当$f(n)=O(g(n))$时，称$g(n)为f(n)$的上界，或更准确的说，渐进上界。

$Definition 7.7$ 令$t:\mathrm{N\to R^+}$是一个函数，定义时间复杂性类$TIME(t(n))$为由$O(t(n))$时间的图灵机判定的所有语言的集合。

$Theorem 7.8$ 设$t(n)$是一个函数，$t(n)\ge n $，则每一个$t(n)$时间的多带图灵机都和某一个$O(t^2(n))$时间的单带图灵机等价，每一个$t(n)$时间的非确定性单带图灵机都与某一个$2^{O(t(n))}$时间的确定性单带图灵机等价。

### P类问题

$Definition 7.11$ P是确定型单带图灵机在多项式时间内可判定的语言集。换言之，$P=\bigcup_k(n^k)$。

其重要性在于：

- 对于所有与确定型单带图灵机多项式等价的计算模型来说，P是不变的。
- P大致对应于在计算机上实际可解的那一类问题。

### NP类问题

$Definition 7.15$ 语言A的验证机是一个算法V，这里 $A=\{w|对某个字符串c，V接受<w,c> \}$，多项式时间验证机在$w$的长的的多项式时间内运行。若语言A有一个多项式时间验证机，则称他为多项式可验证的。

$Definition 7.16$ NP是具有多项式时间验证机的语言类。

### NP-Complete

#### 多项式时间可归约性

$Definition 7.23$ 若存在多项式时间图灵机$M$，使得在任何输入$w$上，$M$停机时$f(w)$恰好在带子上，则称$f:\Sigma ^\ast \to \Sigma ^\ast$为多项式时间可计算函数。

$Definition 7.24$ 语言A称为多项式时间可归约到B，记为$A\le_p B$，若存在多项式时间可计算函数$f:\Sigma ^\ast \to \Sigma ^\ast$，对于每个$w$都有$w\in A \Leftrightarrow f(w)\in B$，函数$f$称为$A$到$B$的多项式时间归约。 

$ Definition 7.27$ 如果语言B满足下面两个条件，就称为NP完全的：

1. $ B\in NP$
2. $\forall A(A\in NP)\land(A \le_p B) $

#### 库克-列文定理

$Theorem 7.30$ SAT是NP完全的。

使用SAT可以模拟得到$A_{TM}$，其公式$\phi= \phi _{cell} \land \phi _{start} \land \phi _{move} \land \phi _{accept}$。

## Appendix 

### Turing Machine Encoding 

$ Encode_{Beginning} = 0^i10^j10^k10^m10^n10^r10^t$ where $ i=|Q|$, $j=|\Sigma|$,$k=|\Gamma|$,$q_m=q_{start},q_n=q_{accept},q_r=q_{reject},$ Blank Symbol $\sqcup$ is $b_t$ 

$Encode_{Transition} = \delta_111\delta_211...11\delta_n$. Where for each transition function: $\delta(q_i,a_j) = (q_k,b_m,D_n) = 0^i10^j10^k10^m10^n$

$D_n:L/R$ where $R(D_1):0^1=0,L(D_2):0^2=00$

$Encoding_{input}: a_ia_j..a_n\mapsto 0^i10^j1..10^n$

$Encoding_{total}= Encoding_{Beginning}111Encoding{Transition}111Encoding{input}$

