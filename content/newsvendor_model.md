+++
title = "Newsvendor model"
date = 2021-11-15
draft = false

[taxonomies]
Categories = ["Supply Chain Management"]
Tags = ["Inventory Management"]
+++

在 EOQ 模型中，我们假定需求是长期的、固定的常量，且 lead time 为0，然后求解使得整体成本最低的最佳订购量。作为对比，在 Newsvendor model(或者single period model)，需求是随机的，lead time 大于销售周期（也就是说有且仅有一次订购机会）。在这些限制条件下，我们来评估如何确认最佳订购量和期望利润。

<!-- more -->


## 模型假设
- Demand: variable, random and continuous
- Lead Time: all inventory be ordered prior to the start of the time period, no replenished during the time period
- Dependence of Items: Independent
- Review Time: continuous
- Capacity/Resource: Unlimited
- Discounts: None
- Excess Demand: Lost orders
- Perishability: None
- Planning Horizon: Single Period
- Number of Items: One
- Form of Product: Single Stage

考虑卖报纸的商家，在每天开始之前需要确定当天订购的数量。如果实际需求大于订购数量，报纸会卖完，同时也会损失销售机会（因为原本可以卖更多）；如果需求小于订购数量，报纸会剩余，剩余的报纸只能报废掉（我们假定，过期报纸价值会大大降低）。那么最关键的就是如何确定最佳订购量，使期望利润最大化。

## 模型推导

先做如下定义：
- p：price
- c：cost
- \\(c_e\\)：excess cost
- \\(c_s\\): shortage cost
- Q: quantity
- P(x): probability of demand
- E[x]: expected value of demand
- g: salvage value
- B: shortage penalty
- E[Units Short]: expected units short
- E[Units Sold]: expected units sold
- E[US]: expected units short


### 最佳订购量 \\(Q^\ast\\)
在我们的利润方程可以表示为：
$$Profit = p(\min[x, Q]) - cQ$$

在需求(Demand)是连续分布函数时，我们有两种成本:
1. \\(c_eP[x \leq Q]\\) = expected excess cost \\(\space\space\\)   i.e. having too much product
2. \\(c_s(1-P[x \leq Q])\\) = expected shortage cost \\(\space\space\\)   i.e. having too little product

当 E[Excess Cost] < E[Shortage Cost] 时，我们会增加订购量Q，二者相等时的Q即为最佳订购量\\(Q^\ast\\)。
$$
c_eP[x \leq Q] = c_s(1-P[x \leq Q]) \newline
c_eP[x \leq Q] + c_sP[x \leq Q] = c_s \newline
P[x \leq Q] = \frac {c_s} {c_e + c_s} \newline
$$

上述公式中，右侧被称为Critical Ratio(CR)。也就是说，为了最大化利润，最佳订购量(\\(Q^\ast\\))覆盖的需求分布应该等于CR。

另，根据如下公式，CR也可以表示为利润除以售价
$$
CR = \frac {c_s} {c_e + c_s} = \frac {p - c} {c + p - c} = \frac {p - c} p
$$

当g/B存在时，直接影响的是\\(c_s\\)和\\(c_e\\)。这时，CR可以拓展为：
$$
c_s = p - c + B \newline
c_e = c - g \newline
CR = \frac {c_s} {c_e + c_s} = \frac {p - c + B} {p + B - g}
$$

在实际应用中，会发现真正困难的地方在于对需求分布，在缺乏相关数据时，市场或者销售人员有可能针对期望需求做出预测，但是需求的分布（\\(\mu-\sigma\\)）则几乎无法预测。所以，在Newvendor model中的预测，一定需求有数据支持（哪怕是使用 triangle distribution）。我们也要知道，预测是会有偏差的，所以使用 Newsvendor model 时也格外留意有预测分布替代需求分布时可能带来的偏差。

### Expected Profits

计算期望利润之前，我们要先了解期望销售数量和期望短缺数量。E[Units Short] 作为E[Units Short | Q]的简写，指的是在订购量为Q是，期望短缺数量。同样的，E[Units Sold]指的是在订购量为Q是，期望销售数量。这两者都和我们的订购数量Q有关。而E[Units Demand]是期望需求，和Q无关。下面来看这三者的关系。

对于需求分布函数\\(\f_x(x)\))，订购数量Q。当需求x低于Q时，销售的数量为需求数量x；当需求数量x高于Q时，销售数量为Q（因为我们只有Q的库存量）。可以表示为：
$$
E[Units Sold] = \int_{x=0}^{Q} x f_x(x)dx + Q\int_{x=Q}^{\infty} f_x(x)dx \newline
E[Units Short] = \int_{x=Q}^{\infty} (x-Q)f_x(x)dx \newline
E[Units Demand] = \int_{x=0}^{\infty} x f_x(x)dx
$$

根据上述公式，我们可以看到：
$$E[Units Demand] = E[Units Sold] + E[Units Short] $$

现在回到期望利润，首先利润和订购数量Q，需求x可以表述为：
$$
Profit(Q, x) = \begin{cases}
px - cQ & \text{if } {x \leq Q }
\\\pQ - cQ & \text{if } {x > Q}
\end{cases}
$$

对上述公式进行积分:
$$
E[Profit(Q, x)] = \int_{x=0}^{Q}(px - cQ)f_x(x_0) dx_0 + \int_{x=Q}^{\infty} (pQ - cQ)f_x(x_0) dx_0 \newline
E[Profit(Q, x)] = p\int_{0}^{\infty}(x)f_x(x_0)dx_0 - cQ - p\int_{Q}^{\infty}(x-Q)f_x(x_0)dx_0 \newline
E[Profit(Q, x)] = pE[x] - cQ - pE[UnitsShort]
$$

这里还有另外一种表述(两者结果一致):
$$
E[Profit(Q, x)] = pE[UnitsSold] - cQ \newline
E[Profit(Q, x)] = pE[x] - cQ - pE[UnitsShort]
$$

其中，E[x]可以根据需求分布计算，Q可以根据需求分布和CR计算，E[UnitsShort]（后续简写为E[US]）我们会讨论如何根据需求分布来计算。所以，我们最终的计算公式为：
> $$ E[Profit(Q, x)] = pE[x] - cQ - pE[US] $$

同样的，当存在g/B时，我们可以拓展上面的公式:
$$ 
P(Q) = \begin{cases}
px - cQ + g(Q-X) & \text{if } {x \leq Q }
\\\pQ - cQ - B(x-Q) & \text{if } {x > Q}
\end{cases} \newline
E[P(Q)] = (p-g)E[x] - (c-g)Q - (p-g+B)E[US] \newline
E[P(Q)] = p(E[x] - E[US]) - cQ - g(Q - (E[x] - E[US])) - BE[US]
$$

#### Unit Normal Loss Function

对于正态分布的需求，我们定义G(k):
$$ E[US] = \int_{x=Q}^{\infty} (x-Q)f_x(x)dx = \sigma G(\frac {Q-\mu} \sigma) = \sigma G(k) $$

其中：
$$ k = \frac {(Q-\mu)} \sigma \newline
G(k) = f_x(x_0) - k*Prob[x_0 \] $$

在Excel中，可以用如下公式计算: 
> G(k)=NORMDIST(k, 0, 1, 0)-k*(1-NORMSDIST(k)).

## summary

Newsvendor model 在时尚类产品中有很好的应用，主要原因在于：我们面对的时不确定的需求。供应链的核心挑战之一就是如何应对不确定性。在实际使用过程中，确定需求分布的参数是非常容易产生较大误差的地方。同时我们也要了解不同的需求分布参数对最终的最佳订购量\\(Q^\ast\\)和E[Profit(Q)]都会有很大的影响。