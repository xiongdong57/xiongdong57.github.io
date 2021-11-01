+++
title = "Pandas Tricks"
date = 2021-10-31
draft = false

[taxonomies]
Categories = ["Programming"]
Tags = ["Python", "Pandas"]
+++

工作中偶尔会用到Pandas处理数据，Pandas功能丰富，也有很多不同的使用方式。以下纪录一些自己觉得比较有意思的用法。
<!-- more -->

## Method Chain

先看一段样例代码
```
# above have createde a dataframe
df.drop_duplicates(subset=['col1'], keep='first')

df2 = pd.merge(df, df_category, on='code', how='left')
df3 = pd.merge(df2, df_saleType, on='code', how='left')
df4 = pd.merge(df3, product_info, on='code', how='left')

df4.rename(columns={'old_name': 'new_name'})

df4['category'] = df4['category'].apply(clean_category)
df4['saleType'] = df4['saleType'].apply(clean_saleType)
df4['price'] = df4['price'].apply(clean_price)
# there are another many lines likes this
```
这段代码中，进行了清除重复行、合并不同的DataFrame，对DataFrame的列进行清理等。这些都是数据处理中经常会遇到的任务。
这里的问题是，当这种处理逻辑比较多的时候，会出现几十行面条一样的代码，可读性很差，也难以维护（尤其在字段名称有中文的时候）。
这时候可以使用`pandas.DataFrame.method chain`来优化处理。

Pandas DataFrame 的 method chain，可以把多个方法放在一起。上述代码可以改写为：
```
def clean_saleType(df):
    # do some cleaning
    return df

df = (df.rename(columns={'old_name': 'new_name'})
        .drop_duplicates(subset=['col1'], keep='first')
        .merge(df_category, on='code', how='left')
        .merge(df_saleType, on='code', how='left')
        .merge(product_info, on='code', how='left')
        .assign(cagegory=x['category'].apply(clean_category),
                price=x['price'].apply(clean_price))
        .pipe(clean_saleType)
        )
```

上述Method chain 由以下部分组成： 
- `DataFrame.drop_duplicates()` 等Pandas自带的，默认返回一个DataFrame的API
- `DataFrame.assign(**kwargs)` 在DataFrame中添加列，列的值根据传入的kwargs进行计算
- `DataFrame.pipe(func, *args, **kwargs)` 将用户自定义的方法引入到Method chain中

### DataFrame.assign

`DataFrame.assign(**kwargs)` 作为`df['key'] = value`的替代，用来添加/替换列，且直接返回一个新的DataFrame对象，默认不会修改原有的DataFrame。
比如
```
df_copy = df.copy()
df['key1'] = value1
df['key2'] = value2

# alternative implement
df = df.assign(key1=value1, key2=value2)
```

另外，pandas的很多方法可以接收类似key=func的参数，assgin也一样。可以使用类似于
`df = df.assign(col1_demeaned=lambda x: x.col1 - x.col1.mean())`这样的写法。

### DataFrame.pipe

`df.pipe(func)`等价于`func(df)`，但是`df.pipe(func)`的优势在于，当连续调用多个方法的时候，可以使用类似 pipeline的写法，提高可读性。比如：
```
dfa = func1(df, arg1=v1)
dfb = func2(dfb, v2, arg3=v3)
dfc = func3(dfb, arg4=v4)

# alternative implement
df = (df.pipe(func1, arg1=v1)
        .pipe(func2, v2, arg3=v3)
        .pipe(func3, arg4=v4))
```

同时`DataFrame.pipe`也是直接返回一个新的DataFrame，默认不修改原有的DataFrame。

使用Method Chain可以将数据处理的逻辑整合到一起，提升可读性。但是，凡事过犹不及，Method Chain也可能被滥用。
不合理的代码组织，混乱的处理逻辑都可能导致可读性变差。同时过长的Method Chain也会导致Debug变得困难。

需要注意的是，对于Method Chain过长的情况，可以通过合并处理逻辑到一个Method，通过一个DataFrame.pipe调用来优化。

## TimeSeries

### TimeSeries Slice

```
dates = [datetime(2021, 1, 2), datetime(2021, 1, 5),
         datetime(2021, 1, 7), datetime(2021, 1, 8),
         datetime(2021, 1, 10), datetime(2021, 1, 12)]
ts = pd.Series(np.random.randn(6), index=dates)
```

Pandas 的 TimeSeriesIndex，可以传入字符串，比如`ts['2021/1/1']`或者`ts['20210102]`，甚至可以传入年或者年月的字符串进行筛选，比如`ts['2021']` 或者`ts['2021-01']`.Slice也完成可以直接用字符串，
比如`ts['20110101':'20210105']`  

也可以传入Datetime对象，比如`ts[datetime(2021, 1, 2)]`这样。

甚至还可以传入timestamp，灵活性非常高。

### TimeSeries Resample

TimeSeries的聚合可以直接使用`pandas.DataFrame.resample`。根据传入freq，比如Y或者YM等进行聚合。如`ts.resample('M', kind='period').mean()`

如果需要生成更低维度（频率）的数据，可以使用`DataFrame.asfreq(freq, method=None, how=None, normalize=False, fill_value=None)`.比如`ts.asfreq('4H').ffill()`