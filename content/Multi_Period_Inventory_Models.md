+++
title = "Multi Period Inventory Models"
date = 2021-11-21
draft = false

[taxonomies]
Categories = ["Supply Chain Management"]
Tags = ["Inventory Management", "satety stock"]
+++

前面的文章介绍了 EOQ Model 和 Single Period Model，分别对应稳定的需求和随机的需求且仅有一次补货机会。现实世界，往往都在这两个极端情况的中间地带，根据不同的行业会有不同的偏向。这里介绍更加一般的情况。

<!-- more -->
## 库存管理评估体系
在库存管理中，我们的目标可以分为两类，一类是从服务水平上评估，另一类是从总成本上评估。二者都和安全库存和设定有关。

从服务水平角度出发，通常使用的指标有 Cycle Service Level(CSL) 和 Item Fill Rate(IFR)。CSL 指的是每个周期内的服务水平（类似于100个周期内有几个周期的库存低于需求）；IFR 指的是每个周期内的需求数量满足比率。通常 IFR 会比 CSL 高，原因是：即使一个周期内库存低于需求（此时 CSL 为0），但是仍有一部分需求被满足（此时 IFR 大于 0 小于1）。

$$ CSL = 1- P[Stockout] = 1- P[X > s] = P[X \le s] \newline
    IFR = 1 - \frac {E[US]} Q = 1 - \frac {\sigma_{DL} G(k)} Q \newline
    G(k) = \frac Q {\sigma_{DL}}(1 - IFR)
$$

其中，k 是安全库存系数。根据上述公式，我们可以通过 k 计算对应的服务水平，也可以根据服务水平计算出 k 的值。

从成本的角度出发，可以使用 Cost of Stock Out Event(CSOE) 和 Cost of Item Short(CIS)，这两者需要针对缺货成本给出明确的数值定义（B1 for Stockout Event，\\(c_s\\) for Stockout per Item ）才能计算。

在 CSOE 中，我们仍然可以通过成本公式来求解 k：
$$ TC = cD + c_t(\frac D Q) + c_e(\frac Q 2 + k\sigma_{DL}) + B_1(\frac D Q)P[X \ge s] $$

当上述公式取最小值时：
$$ k = \sqrt {2 \ln \frac {B_1D} {c_e \sigma_{DL} Q \sqrt {2 \pi}}} $$

在 CIS 中，
$$ TC = cD + c_t(\frac D Q) + c_e(\frac Q 2 + k\sigma_{DL}) + c_s \sigma_{DL} G_u(k)(\frac D Q) $$

当上述公式取最小值时：
$$ P[Stockout] = P[x \ge k] = \frac {Qc_e} {Dc_s} $$

整体而言，CSL、IFR、COSE 和 CIS，四者都和 k 关联。所以，其中一项指标确定，都可以通过 k 来计算另外三项，也就是说四者是互相关联的。

## 安全库存(Safety Stock)

无论采用何种评估体系，最终都会涉及到安全库存。安全库存的计算公式如下：
$$ Safety \space Stock = k\sigma_{DL}$$

其中，k 是安全库存系数，可以根据需求分布和服务水平来确定。\\(\\sigma_{DL}\\) 的定义是：在 lead time 内需求的标准差。但是在实际应用中，我们通常是用预测来替代需求（请留意，二者是不同的）。从这个层面来讲，\\(\\sigma_{DL}\\) 应该是在 lead time 内需求和预测的差值的标准差，也就是RMSE。原因是：如果预测就是真实需求，那么就不应该有安全库存（因为每个时间点的预测都是实际需求）；即使预测未来的不同时间点有变化，那么应该算作 seasonality。在没有 RMSE 的情况下，我们可以选择用预测的在 lead time 内的标准差来替代。

## 库存管理策略介绍

常见的库存管理策略可以分为如下几类，其中 EOQ Model 和 Single Period Model(Newsvendor model) 分别都在另外的文章有专门介绍，这里不再展开。

### EOQ
- Order \\(Q^\ast\\) every \\(T^\ast\\) periods
- Order \\(Q^\ast\\) every when \\(IP = \mu_{DL}\\)

### Single Period
- Order \\(Q^\ast\\) at start of period where \\(P[x \le Q] = CR\\)

### Base Stock Policy
基本策略如下：
- a one-for-one replacement
- Order what was demand when it was demanded in the quantity of it was demanded

在这种策略下，如果我销售了4个，那么我就预定4个（适用于 7-11 门店补货这类补货周期较短的业务模式）。这个策略对应的参数如下：
- Optimal Base Stock: \\(S^\ast = \mu_{DL} + k_{LOS}\sigma_{DL} \\)
- Level of Service(LOS): \\(LOS = P[\mu_{DL} \le S^\ast = CR] = \frac {c_s} {c_s + c_e}\\)

### Continuous Review Policy(s, Q)
基本策略如下：
- Order \\(Q^\ast\\) when \\(IP \le s\\)

其中，s 又称为再补货点，当库存低于 s 的时候，预定 \\(Q^\ast\\)。其中的 \\(Q^\ast\\) 可以通过 EOQ model 求解。这种模式偏向于需求相对稳定的产品，同时假定 Review Period 为0。

这个策略对应的参数如下：
- Reorder Point: \\(s = \mu_{DL} + k\sigma_{DL} \\)
- Order Quantity: 通过 EOQ moddel 求解

### Periodic Review Policy(R, S)
- Order up to S every R time periods

Periodic Review Policy(R, S) 和 Continuous Review Policy(s, Q) 的区别在于：前者添加了 Review time（R），更加符合实际情况。

这个策略对应的参数如下：
- Order UP to Point: \\(S = \mu_{DL + R} + k\sigma_{DL + R} \\)

## SKU 分类

上述补货策略针对的都是单一 SKU，然而现实世界中，我们需要管理的 SKU 可能会有很多。这种情况下，将相似类别的产品进行分类，为每个类别设置不同的补货策略可以节省时间和精力。

从经验上来说，按照销量，销售额或者利润额来进行降序排列，约10%的产品贡献50%的销量，50%的产品贡献约80-90%的销量，剩余的产品都是长尾。这种现象被称为 [power law](https://en.wikipedia.org/wiki/Power_law)，在天文、气象、人口、心理学、生物学、经济学中都有类似的现象。

我们可以据此将产品分为ABC类，最多的精力放在A类产品上，频繁关注和Review，策略应该是尽可能准确细致；B类产品通过设定规则（CSL、安全库存等）来进行管理，上述的补货策略主要针对的是这类产品；C类产品尽可能少放精力，策略约简单越好（即使牺牲部分准确度）。