---
layout:     post
title:      python可视化工具之vizro框架
subtitle:   vizro
date:       2024-4-16
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- python 

---

# 前言

>最近学到python可视化工具,推荐使用vizro框架.(听说vizro是麦肯锡公司出的,所以抱着好奇心测试一下)




## 一, 如何绘制箱子图和折线图


![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2024/vizro_image1.png)


```
from vizro import Vizro
import vizro.models as vm
import vizro.plotly.express as px
df = px.data.gapminder()
gapminder_data = (
        df.groupby(by=["continent", "year"]).
            agg({"lifeExp": "mean", "pop": "sum", "gdpPercap": "mean"}).reset_index()
    )

first_page = vm.Page(
    title="First Page",
    layout=vm.Layout(grid=[[0,0],[1,2],[1,2],[1,2]]),
    components=[
        vm.Card(
            text="""
        # 标题
        内容......
            """,
        ),
        vm.Graph(
            id="box",
            figure=px.box(gapminder_data, x="continent", y="lifeExp", color="continent",
                            labels={"lifeExp": "Life Expectancy", "continent":"Continent"},title='箱子图'),
        ),
        vm.Graph(
            id="line",
            figure=px.line(gapminder_data, x="year", y="pop", color="continent",
                    labels={"year": "Year", "continent": "Continent",
                    "pop":"GDP Per Capxxx"}, title='折线图'),
        ),


    ],
    controls=[vm.Filter(column="continent", targets=["box", "line"]),],
)
dashboard = vm.Dashboard(pages=[first_page])
Vizro().build(dashboard).run(debug=True)
```

## 二, 如何绘制点状图图和柱状图图


![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2024/vizro_image2.png)


```
vm.Graph(
            id="scatter_iris",
            figure=px.scatter(iris_data, x="sepal_width", y="sepal_length", color="species",
                color_discrete_map={"setosa": "#00b4ff", "versicolor": "#ff9222"},
                labels={"sepal_width": "Sepal Width", "sepal_length": "Sepal Length",
                        "species": "Species"},title='点状图',
            ),
        ),
        vm.Graph(
            id="hist_iris",
            figure=px.histogram(iris_data, x="sepal_width", color="species",
                color_discrete_map={"setosa": "#00b4ff", "versicolor": "#ff9222"},
                labels={"sepal_width": "Sepal Width", "count": "Count",
                        "species": "Species"},title='柱状图',
            ),
        ),
        
```

## 三, 如何加载原始数据和绘制气泡图

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2024/vizro_image3.png)

```
df_gapminder = px.data.gapminder().query("year == 2007")
fourth_page = vm.Page(
            title="Filter interaction",
            components=[
                vm.AgGrid(
                    figure=dash_ag_grid(data_frame=df_gapminder),
                    actions=[
                        vm.Action(function=filter_interaction(targets=["scatter_relation_2007"]))
                    ],
                ),
                vm.Graph(
                    id="scatter_relation_2007",
                    figure=px.scatter(
                        df_gapminder,
                        x="gdpPercap",
                        y="lifeExp",
                        size="pop",
                        color="continent",
                    ),
                ),
            ],
            controls=[vm.Filter(column='continent')]
)

```
## 四, 组件:
![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2024/vizro_image4.png)

```
controls=[
        vm.Parameter(
            targets=["scatter_iris.color_discrete_map.virginica",
                        "hist_iris.color_discrete_map.virginica"],
            selector=vm.Dropdown(
                options=["#ff5267", "#3949ab"], multi=False, value="#3949ab", title="颜色切换"),
            ),
        vm.Parameter(
            targets=["scatter_iris.opacity"],
            selector=vm.Slider(min=0, max=1, value=0.8, title="透明度"),
        ),
    ],
```

## Github项目链接:

[https://github.com/LuochuanAD/PythonProject/blob/main/vizroProject/vizroTest.py](https://github.com/LuochuanAD/PythonProject/blob/main/vizroProject/vizroTest.py "https://github.com/LuochuanAD/PythonProject/blob/main/vizroProject/vizroTest.py")

## 个人反思:

看了vizro官网的文档, 理解了麦肯锡的可视化图表,真的干货,就是有点太干了. 多个图表的连动,加载等等都是通过id这个核心属性来绑定的. 就是代码有太多的","号,这与python语法不太搭配.


