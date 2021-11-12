+++
title = "EOQ Model"
date = 2021-11-09
draft = false

[taxonomies]
Categories = ["Supply Chain Management"]
Tags = ["Inventory Management"]
+++

库存管理是供应链管理的核心之一。持有库存有很多原因，包括：1. 平衡需求和供给；2. 降低运营成本；3. 缓冲需求和制造方面的不确定性。在现实世界中，产品从生产、运输到商家、商家再配送给客户的过程中往往需要相当长的时间。商家通常会持有一定水平的库存量，当客户有需求的时候，可以第一时间交付，当商家持有库存不足时，由于无法满足需求，可能会失去销售机会。但是，持有过多的库存会有较高的库存持有成本甚至会影响到企业现金流，而且库存也存在过期、变质、损坏的风险。

库存管理本质上是确定合适的产品在合适的位置以合适的形式的数量。战略决策包括产品层面和网络设计的影响。战术决策包括部署和确定携带什么物品，以什么形式（原材料、在制品、成品等），以及在哪里。最后，运营决策决定了这些库存的补货政策（何时和多少）。我们寻求能使这些总成本最小化的补货策略。补货策略本质上需要说明两件事：要订购的数量，以及何时订购。

不同的补货策略模型对应不同的需求和供应的假设（需求是常量还是变量，确定还是随机等）。这里我们首先从介绍 EOQ Model。
<!-- more -->

## EOQ Model介绍
### EOQ Model假设

- Demand: constant, known and continuous
- Lead Time: Instantaneous
- Dependence of Items: Independent
- Review Time: continuous
- Capacity/Resource: Unlimited
- Discounts: None
- Excess Demand: None
- Perishability: None
- Planning Horizon: Infinite
- Number of Items: One
- Form of Product: Single Stage

使用EOQ Model一定要注意模型的初始假设是否满足，尤其是均一、确定且长期保持稳定不变的需求，lead time 为0，以及整批货物一起收到。

### EOQ Model 库存管理策略

- 每隔 \\(T^\ast \\) 的时间预定 \\(Q^\ast \\) 
- 当库存低于0时，预定 \\(Q^\ast \\)

### EOQ Model 推导
先做如下定义：
- D: 平均需求量
- c: 每个产品的采购单价
- \\(c_t\\)：订单执行成本（和订单次数有关，比如固定的清关费用等）
- \\(c_e\\)：库存持有成本（通常高于资金成本）
- \\(c_s\\)：缺货成本
- Q: 每次订购量
- T: 每次订购间隔
- N=1/T: 单位时间订购次数
- TC(Q): 总成本
- TRC(Q): 总相关成本
- \\(Q^\ast\\): 总成本最低的最佳订购量

总成本可以表示为：TC = Purchase + Order + Holding + Shortage。在EOQ 模型的假设下可以可以改写为：
 $$ TC(Q) = cD + c_t\frac D Q + c_e\frac Q 2+ c_sE[UnitsShort] $$

其中，采购成本 cD 和短缺成本 \\(c_sE[UnitsShort]\\) 和单次订购数量无关，所以问题可以简化为求如下等式的最小值：
 $$ TRC(Q) = c_t\frac D Q + c_e\frac Q 2 $$

求解上述表达式一次导数为0时，即为 \\(Q^\ast\\)：
 $$ \frac {dTRC(Q)} {dQ} =  -c_t\frac D {Q^2} + \frac {c_e} 2 = 0 \newline
 \frac {c_e} 2 = c_t\frac D {Q^2} \newline
 Q^\ast = \sqrt{\frac {2c_t D} {c_e}} $$

求解二次导数，也会发现恒大于0，所以一次导数为0时，即为最小值。
 $$ \frac {d^2TRC(Q)} {d^2Q} = \frac {2c_tD} {Q^3} > 0 $$

求解 \\(T^\ast\\):
 $$ T^\ast = \frac {Q^\ast}{D} = \sqrt{\frac {2 c_t D} {c_e}}{\frac 1 D} = \sqrt{\frac {2 c_t} {c_e D}} $$

求解\\(TRC(Q^\ast)\\):
 $$ TRC(Q^\ast) = c_t\frac D {Q^\ast} + c_e\frac {Q^\ast} 2 = \sqrt {2 c_t c_e D} \newline
 TC(Q^\ast) = cD + \sqrt {2 c_t c_e D} $$

## EOQ Model Analysis
### Sensitivity to Order Quantity

 $$ \frac {TRC(Q)} {TRC(Q^\ast)} = \frac {c_t\frac D Q + c_e\frac Q 2} {\sqrt {2 c_t c_e D}} \newline
 = \frac {c_tD} {Q \sqrt {2 c_t c_e D}} + \frac {c_eQ} {2\sqrt {2 c_t c_e D}} \newline
 = \frac {\sqrt{c_tD}} {Q \sqrt{2c_e}} + \frac {\sqrt{c_e}Q} {2\sqrt {2 c_t D}} \newline
 = (\frac {\sqrt2} {\sqrt2})\frac {\sqrt{c_tD}} {Q \sqrt{2c_e}} + \frac {\sqrt{c_e}Q} {2\sqrt {2 c_t D}} \newline
 = \frac {\sqrt{2c_tD}} {2Q \sqrt{c_e}} + \frac {\sqrt{c_e}Q} {2\sqrt {2 c_t D}} \newline
 = (\frac 1 2)(\frac {\sqrt{2c_tD}} {Q \sqrt{c_e}} + \frac {\sqrt{c_e}Q} {\sqrt {2 c_t D}}) \newline
 = (\frac 1 2)(\frac {Q^\ast} Q + \frac Q {Q^\ast}) $$

根据上面的公式，计算使用不同的Q对总的TRC的影响:

| \\(\frac Q {Q^\ast}\\) | \\(\frac {TRC(Q)} {TRC(Q^\ast)}\\) |
|------------------------|------------------------------------|
| 300%                   | 167%                               |
| 200%                   | 125%                               |
| 150%                   | 108%                               |
| 50%                    | 125%                               |

从上述表格中可以看出，EOQ 模型对订购量Q相当稳定，即使Q比最优的\\(Q^\ast\\)翻倍或者减少50%，TRC也只增加25%。

### Sensitivity to Demand

既然EOQ 模型对订购量Q的变化非常稳定，那么如果考虑最初的假设，我们使用的时需求（D），如果我们使用预测（F）来替代对TRC会有多大的影响呢？尤其是考虑到预测往往会和实际需求有偏差。

 $$ {Q ^ \ast}_F = \sqrt{\frac {2c_t D_F} {c_e}} \newline
 {Q ^ \ast}_A = \sqrt{\frac {2c_t D_A} {c_e}} \newline
 \frac {{Q ^ \ast}_F} {{Q ^ \ast}_A} = \sqrt{ \frac {D_F} {D_A} } \newline
 \frac {TRC(Q)} {TRC(Q^\ast)} = (\frac 1 2)(\frac {Q^\ast} Q + \frac Q {Q^\ast})  \newline
 \frac {TRC({Q ^ \ast}_F)} {TRC({Q ^ \ast}_A)} = (\frac 1 2)(\frac {{Q ^ \ast}_A} {Q ^ \ast}_F + \frac {{Q ^ \ast}_F} {{Q ^ \ast}_A}) \newline
 = (\frac 1 2)(\sqrt { \frac {D_A} {D_F} } + \sqrt { \frac {D_F} {D_A} }) $$

对比上述公式和EOQ对Q的稳定程度，可以看出即使预测的偏差在50%或者200%之间，TRC的变化也会非常小（1.06%）。但是这里一定要留意，EOQ 模型比较的是TRC对预测非常稳健（且需求是长期且稳定的的假设不变），并不是说任何时候都可以适用，尤其是当TRC不是优先考虑的因素时。

### Sensitivity to Order Cycle Time

根据同样的方程，我们可以得出：
 \\[\frac {TRC(T)} {TRC(T^\ast)} = (\frac 1 2)(\frac T {T^\ast} + \frac {T^\ast} {T}) \\]

和订购量Q一样，EOQ 模型对间隔订购时间（T）没有严格的要求。使用不同的T，对最终的TRC影响不大。

### summary

通过上述分析可以看到，EOQ模型有非常严格的假设要求，但是仍然有非常广泛的用途。当存在长期且稳定的需求时，EOQ模型都是一个很好的开始，可以根据实际情况调整Q。EOQ模型平衡的是库存持有成本和订购成本，当有其它的考虑因素时，我们可以知道即使Q偏离\\(Q^\ast\\)，那么也可以预估TRC的变化幅度。另外一点需要注意的是TRC只是TC的一部分，那么TC的变化幅度则会更小。

EOQ模型对采购量（Q），订购周期（T），预测误差（F）等的变化都非常稳健。同样的，对于模型设定的参数，\\(c_t\\)、\\(c_e\\)等的误差，结果也是非常稳定的。

> EOQ model is a good start point, but not always a good end point.

## EOQ Model Extension

EOQ模型有非常严格的约束条件，下面会分别到了Lead Time 不为0对订购数量和订购时间的影响，以及存在折扣时EOQ模型如何变化。

### Extension with non-zero lead time
当Lead Time 不为零时，EOQ 模型的总成本中，TC = Purchase + Order + Holding + Shortage, 其中，Purchase、Order、Shortage cost不会变化，而 Holding cost会变化。首先来看总库存（在库+在途）的变化:  
- cycle time = \\(T^\ast = \frac {Q^\ast} D \\)
- average pipeline inventory = \\({Q^\ast}(\frac L {T^\ast}) + 0(1 - \frac L {T^\ast}) = \frac {Q^\ast L} {T^\ast} = \frac {Q^\ast L} {\frac {Q^\ast} D} = DL \\)

根据商务条款的不同，在途库存有可能会需要纳入我们的考虑（比如EXW），这里假设在途库存的 Holding cost也由我们承担。那么新的TC方程可以表示为：
 $$ TC(Q) = cD + c_t\frac D Q + c_e(\frac Q 2 + DL)+ c_sE[UnitsShort] $$

根据前面的推导，可以得出，Lead Time(L) > 0时，最佳订购量\\(Q^\ast\\)不会发生变化。真正会影响的时我们补货的时间点，为了避免缺货，L>0时的补货策略更新为：
- 当库存低于DL时，预定 \\(Q^\ast \\)
- 每隔 \\(T^\ast \\) 的时间预定 \\(Q^\ast \\)

### Extension with discount rate

这里我们分别讨论如下三种形式的折扣：
- 当订购数量达到\\(Q_1\\)时，所有产品都享受折扣
- 只有超过\\(Q_1\\)的部分才享受折扣
- 一次性折扣（没有数量限制）

#### All Units Discount

这种情况下，采购单价c和TRC方程可以表示为:  
$$
c = \begin{cases}
c_0 & \text{if } {0 \le Q<Q_1}
\\\c_1 & \text{if } {Q_1 \le Q}
\end{cases}
$$

$$
TRC = \begin{cases}
Dc_0 + \frac {c_tD} Q + \frac {c_0hQ} 2 & \text{   }  {0 \le Q<Q_1}
\\\Dc_1 + \frac {c_tD} Q + \frac {c_1hQ} 2 & \text{   }  {Q_1 \le Q}
\end{cases}
$$

求解过程如下：
1. 求解\\(Q_{c0}^\ast\\)和\\(Q_{c1}^\ast\\)
2. 如果\\(Q_{c1}^\ast\\) > \\(Q_1\\)，那么选择\\(Q_{c1}^\ast\\)，否则进入3
3. 求解\\(TRC(Q_{c0}^\ast)\\)和\\(TRC(Q_1)-c_1\\), 如果前者更小，那么选择\\(Q_{c0}^\ast\\)，否则选择\\(Q_1\\)

#### Incremental Discount

只有超过\\(Q_1\\)的部分才享受折扣。当订购数量低于\\(Q_1\\)时，采购单价保持\\(c_0\\)不变，当订购数量达到\\(Q_1\\)时，低于\\(Q_1\\)的部分采购单价仍为\\(c_0\\)，高于\\(Q_1\\)的部分采购单价为\\(c_1\\)。这种情况下，当订购数量高于\\(Q_1\\)时，可以等价为订购数量的采购单价都为\\(c_1\\)，再加上每次订购会产生固定成本\\(F = (c_0 - c_1)Q_1\\)。那么，
$$ Q_1^\ast = \sqrt{\frac {2(c_t + F_i) D} {c_e}} $$

求解过程如下：
1. 求解\\(TRC(Q_1^\ast)\\) 和 \\(Q_1^\ast\\)
2. 如果 \\(Q_1^\ast > Q_1\\), 比较\\(TRC(Q_1^\ast)\\)和\\(TRC(Q^\ast)\\)，选择较小的那个

## summary

EOQ 模型的简洁和稳健容易导致误用，尤其是没有充分考虑模型要求的前置假设（稳定、均一和已知的需求）而盲目套用。