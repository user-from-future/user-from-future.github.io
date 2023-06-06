# 前言

**目录**

[前言](#%E5%89%8D%E8%A8%80)

[代码实现](#%C2%A0%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0)

[可视化分析](#%E5%8F%AF%E8%A7%86%E5%8C%96%E5%88%86%E6%9E%90)

---

今天我们看看自己所在的省份的人口人数，使用pandas并作可视化分析。

**环境使用**

- python 3.9
- pycharm

**模块使用**

- pandas

> Pandas 是基于NumPy的一种工具，该工具是为解决数据分析任务而创建的。Pandas 纳入了大量库和一些标准的数据模型，提供了高效地操作大型数据集所需的工具。pandas提供了大量能使我们快速便捷地处理数据的函数和方法。你很快就会发现，它是使Python成为强大而高效的数据分析环境的重要因素之一。

# 代码实现

我们以湖南省为例，我们可以在湖南省统计局官网上获取数据。

![](https://img-blog.csdnimg.cn/6ca348ed93d748d799d9ceaf74878474.png)

在里面我们可以看到各个地方的人口数据如下：

![](https://img-blog.csdnimg.cn/37398bbc8ffe45c6aca0676b5e344852.png)

有的人说了，我可以右击查看源代码或者自己一个一个敲下来。今天，我们不用这个方法，我们用pandas的方法获取数据。

```python
import pandas as pd

df = pd.read_html('http://tjj.hunan.gov.cn/hntj/tjfx/tjgb/rkpc/202105/t20210519_19050124.html')
```

运行得到结果如下：

![](https://img-blog.csdnimg.cn/7e97bf028b7541f2b6fb77425a59b705.png)

我们会发现这是一个列表，所以需要我们处理数据。

**数据清洗**

> 1.删除（remove）
> 
> 2.重命名列索引字段（rename）

```python
df = df[0]

df = df.rename(columns={0: '城市', 1: '人口数', 2: '城市比重', 3: '乡村比重'})

df = df.drop(df.index[[0, 1, 2]])

df = df.reset_index(drop=True)
```

运行结果如下：

![](https://img-blog.csdnimg.cn/6a7f2e97b2bc43628d7c4a643dcf600e.png)

# 可视化分析

下面我们做可视化分析：

首先导入模块

```python
from pyecharts.charts import Bar
```

这里我们要用到pyecharts模块。

> Pyecharts是一个用于生成Echarts图表的类库，可以与Python进行对接，方便在Python中直接使用数据生成图。Echarts是百度开源的一个数据可视化JS库，生成的图可视化效果非常棒，凭借着良好的交互性，精巧的图表设计，得到了众多开发者的认可。
> 
> Pyecharts分为v0.5.X和v1两个大版本，v0.5.X和v1间不兼容，v1是一个全新的版本。Pyecharts经过了半年的沉寂后，终于发布了新版本，新版本号将从v1.0.0开始，这是一个全新的、向下不兼容的Pyecharts版本，类似于Python 3与Python 2。不过，如果开发者以前接触过Pyecharts，新版本就很容易上手。

```python
c = (

    Bar()

    .add_xaxis(df['城市'].tolist())

    .add_yaxis('人口数',df['人口数'].tolist())

)
```

可视化数据输出

```python
 c.render('人口数.html')
```

运行结果展示：

![](https://img-blog.csdnimg.cn/fdf8c52603bb457693f50f9bf4569cee.png)

大家还可以对图表进行设置，看看自己省份的人口分布地图吧。

```python
from pyecharts.charts import Map
map_show = df.groupby('城市')['人口数'].sum()
a = (
    Map()
    .add('人口数',maptype='湖南',data_pair=[list(i) for i in zip(map_show.index,map_show.values)])

)
```

可视化地图输出

![](https://img-blog.csdnimg.cn/3c438ceaefe64ca9ae658a979f77a96e.png)

湖南人口分布可视化

到这里，大体的效果就做出来了。