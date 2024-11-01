---
layout:     post
title:      对Excel处理的各种Python
subtitle:   Excel
date:       2024-11-01
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Python 
- Excel

---

## 前言

>最近在工作中遇到各种要处理Excel文件的问题, 所以每次都会自己写个python脚本来自动化处理各种Excel, 所以在此记录一下



## 案例


### 一,案例一

> 对比2个Excel的不同,将不同的地方标记成黄色,并生成2个Excel结果文件来展示黄色不同的地方.

**代码片段**

```
workbook1 = pxl.load_workbook(r"/Users/luochuan/Desktop/用Python对比2个Excel的不同/Resume_Louis.Luo_Test1.xlsx")
workbook2 = pxl.load_workbook(r"/Users/luochuan/Desktop/用Python对比2个Excel的不同/Resume_Louis.Luo_test2.xlsx")

......
workbook1_sheet1 = workbook1['sheet']
workbook2_sheet1 = workbook2['sheet']
for i in range(1, (max_row + 1)):
    for j in range(1, (max_column +1)):
        cell_1 = workbook1_sheet1.cell(i, j)
        cell_2 = workbook2_sheet1.cell(i, j)
        
        if cell_1.value != cell_2.value:
            cell_1.fill = PatternFill("solid", fgColor='FFFF00')
            cell_1.font = Font(color=colors.BLACK, bold=True)
            cell_2.fill = PatternFill("solid", fgColor='FFFF00')
            cell_2.font = Font(color=colors.BLACK, bold=True)

workbook1.save(r'result1.xlsx')
workbook2.save(r'result2.xlsx')
```

**效果显示:(已将2个excel的不同之处用黄色显示)**
![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2024/matchExcel_screen.png)

**详细代码及测试数据已上传到GitHub**[用Python对比2个Excel的不同](https://github.com/LuochuanAD/PythonProject/tree/main/%E7%94%A8Python%E5%AF%B9%E6%AF%942%E4%B8%AAExcel%E7%9A%84%E4%B8%8D%E5%90%8C) 



### 二,案例2
> ...

