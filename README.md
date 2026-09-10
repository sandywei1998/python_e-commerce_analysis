# 数据分析清洗SOP
> 适用：电商订单数据集 pandas 数据清洗标准作业流程

## 一、读取数据
### （一）启动Jupyter Notebook
指令（命令行输入，cmd终端以管理员运行）：
```cmd
jupyter notebook --notebook-dir="文件存放路径"
```
### （二）引入pandas库、matploylib、seaborn并且运行生效
指令
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```
### （三）读取原始数据表
## 二、评估数据
### （一）评估数据整齐度
指令：sample方法
### （二）评估数据干净度
指令：使用info方法和describe方法
## 三、清理数据
建立数据副本，根据评估结果，在副本上清理数据
## 四、保存清理后的数据
把文件保存为csv文件
