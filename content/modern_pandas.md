+++
title = "Pandas tricks"
date = 2021-10-31
draft = false

[taxonomies]
Tags = ["Programming"]
Categories = ["Python"]
+++

工作中处理数据，经常会用到Pandas。Pandas功能丰富，也有很多不同的API。以下纪录一些自己觉得比较有意思的用法。
<!-- more -->

## Method Chaining

先看一段代码
```
    # above have createde a dataframe
    df.drop_duplicates(subset=['col1'], keep='first')

    df = pd.merge(df, df_category, on='code', how='left')
    df = pd.merge(df, df_saleType, on='code', how='left')
    df = pd.merge(df, product_info, on='code', how='left')

    df.rename(columns={'old_name': 'new_name'})

    df['category'] = df['category'].apply(clean_category)
    df['saleType'] = df['saleType'].apply(clean_saleType)
    df['price'] = df['price'].apply(clean_price)
    # there are another many lines likes this
```
这段代码中，进行了清除重复行、合并不同的dataframe，对dataframe的列进行清理，都是数据处理中经常会遇到的任务。
问题是，当这种处理非常多的时候，会出现几十行面条一样的代码，可读性很差，也难以维护（尤其在字段名称有中文的时候）。
这时候可以使用`pandas.DataFrame.method chaining`来优化处理。

Pandas DataFrame 支持 Method chaining，可以把多个方法放在一起，比如下面这段代码：
```
    def clean_saleType(df):
        # do some cleaning
        return df

    df = (df.rename(columns={'old_name': 'new_name'})
          .drop_duplicates(subset=['col1'], keep='first')
          .merge(df_category, on='code', how='left')
          .merge(df_saleType, on='code', how='left')
          .merge(product_info, on='code', how='left')
          .assign(cagegory=lambda x: x['category'].apply(clean_category))
          .pipe(clean_saleType)
          )
```

上述Method chaining 由以下部分组成： 
- `DataFrame.drop_duplicates()` 等Pandas 自带的API（默认返回DataFrame即可）
- `DataFrame.assign(**kwargs)` 在DataFrame中添加列，列的值根据传入的col=Method进行计算
- `DataFrame.pipe(func, *args, **kwargs)` 将用户自定义的方式引入到Method chaining中

需要说明的是，`DataFrame.assign`和`DataFrame.pipe`都不会修改原有的DataFrame，而是返回一个新的DataFrame。
这一点在调试过程中会有帮忙。

对比而言，使用 Method Chaining 可以将数据处理的逻辑整合到一起，提升可读性。但是，凡事过犹不及，Method Chaining 也可能被滥用。
不合理的代码组织，混乱的处理逻辑都可能导致可读性变差。同时过长的 Method Chaining 也会导致 Debug 变得困难。需要注意的是，对于
Method Chaining 过于长的情况，可以通过合并处理逻辑到一个Method，通过一个DataFrame.pipe调用来优化。