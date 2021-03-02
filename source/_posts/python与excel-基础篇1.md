---
title: python与excel-基础篇1
tags:
  - excel
  - python
categories: python
keywords: 'openpyxl, python'
description: 使用openpyxl对excel读写操作
abbrlink: df7e
---

>使用openpyxl对excel文件进行读写
>
>[官方指导文档](https://openpyxl.readthedocs.io/en/stable/tutorial.html)

## 准备

### excel基本术语



| 术语                    | 解释                                        |
| ----------------------- | ------------------------------------------- |
| Spreadsheet or Workbook | excel文件                                   |
| Worksheet or Sheet      | 表，一个workbook/spreadsheet可以有多个sheet |
| Column                  | 表格列A....Z....                            |
| Row                     | 表格行1....10....                           |
| Cell                    | 单元格A1....A2....                          |

### 安装openpyxl

```bash
pip install openpyxl -i https://pypi.douban.com/simple/
```

## 基本操作

### 新建workbook

```python
from openpyxl import Workbook
wb = Workbook()
```

### 加载已经存在的工作簿

```python
from openpyxl import load_workbook
wb = load_workbook('File Name')
```



### 选择sheet

默认Workbook创建时，会有一个sheet，通过`ws = wb.active` 选择并使用它

存在多个sheet时，可以通过`ws = wb[Sheet Name]` 选择所有操作的sheet，也可以通过以下操作选择

```python
# 获取当前所有sheetname
sheetlist = wb.sheetnames
# >> ['1st Sheet', '标签内容信息- Basic', '标签内容信息 -Standard', 'Readme', 'Add your sheets and content...']
ws = wb[sheetlist[2]] # 或者
ws.active = 2
ws = wb.active
# 查看当前sheet名字
ws.title
>> 标签内容信息- Basic
```

`ws = wb[Sheet Name]` 方法没有改变当前活跃的表格名

```python
wb.active
>> <Worksheet "标签内容信息- Basic">
ws = wb['标签内容信息 -Standard']
wb.active # 还是运来的单元格
>> <Worksheet "标签内容信息- Basic">
```

### 新建sheet

```python
# 默认插在最后位置
ws = wb.create_sheet('Sheet Name')
# 也可以在sheet name参数后增加一个参数，表示创建位置
ws = wb.create_sheet('Sheet Name', 0)
```

### 修改sheet tittle

```python
ws.title = New Title
```



### 写入单元格

```python
ws['A1'] = 'Name'
ws['B1'] = 'Age'
ws['C1'] = 'score'
ws['A2'] = '张三'
ws['B2'] = 18
ws['C3'] = 95
```

### 查看单元格内容

```python
ws['A2'].value
>> '张三'
```

### 保存文件/工作簿

```python
# 如果filename已经存在，会没有确认的修改这个文件
wb.save(filename)
```

### 关闭工作簿

```python
# 可能没有用？
wb.close()
```



## 参考

[1] [官方文档](https://openpyxl.readthedocs.io/en/stable/tutorial.html)

[2] [选择sheet](https://stackoverflow.com/questions/41556378/openpyxl-set-active-sheet/50117733)

[3] [realpython openpyxl guide](https://realpython.com/openpyxl-excel-spreadsheets-python/)

