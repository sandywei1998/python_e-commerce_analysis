# 电商数据分析SOP
> 适用：电商订单数据集 pandas 数据清洗、可视化标准作业流程

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
指令
```python
original_data = pd.read_csv("你的数据源文件名.csv")
```
## 二、评估数据
### （一）评估数据整齐度
指令：sample方法
```python
original_data.sample(10)
```
### （二）评估数据干净度
指令：使用info方法和describe方法
```
original_data.info()
original_data.describe()
```
## 三、清理数据
建立数据副本，根据评估结果，在副本上清理数据，指令如下
```python
# 1. 创建副本，保护原始数据
cleaned_data = original_data.copy()

# 2. 删除Description为空的行
cleaned_data = cleaned_data.dropna(subset=["Description"])

# 3. 过滤数量大于0的有效订单
cleaned_data = cleaned_data[cleaned_data["Quantity"] > 0]

# 4. 转换日期字段类型
cleaned_data["InvoiceDate"] = pd.to_datetime(cleaned_data["InvoiceDate"])

# 5. CustomerID转为字符串
cleaned_data["CustomerID"] = cleaned_data["CustomerID"].astype(str)

# 6. CustomerID字符串截取
cleaned_data["CustomerID"] = cleaned_data["CustomerID"].str.slice(0,-2)

# 清洗后校验
cleaned_data.info()
```
## 四、保存清理后的数据
把文件保存为csv文件
```python
cleaned_data.to_csv("cleaned_ecom_data.csv", index=False)
```
## 五、可视化
目标：基于清洗完成的 cleaned_data 做多维度可视化，挖掘业务特征，复核清洗质量；
规范：**一个单元格绘制一张图**，单独调试，单个报错不会影响其他图表。
### （一）全局绘图参数一次性配置（仅运行 1 次，后续所有图表生效）
```python
# 设置图片清晰度、中文正常显示、负号正常显示
plt.rcParams['figure.dpi'] = 120
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# 衍生核心指标：新增销售额字段，销售额 = 购买数量 × 单品单价
cleaned_data["SalesAmount"] = cleaned_data["Quantity"] * cleaned_data["UnitPrice"]
```
### （二）时序分析：月度销售额趋势折线图
```python
monthly_sales = cleaned_data.resample('ME', on = 'InvoiceDate')["SalesAmount"].sum()
plt.figure(figsize = (10,4))
sns.lineplot(data = monthly_sales)
plt.title("月度总销售额变化趋势")
plt.xlabel("订单月份")
plt.ylabel("销售总金额")
plt.grid(alpha = 0.3)
plt.tight_layout()
plt.savefig("month_sales_trend.png")
plt.show()
```
### （三）市场结构分析：各国销售额柱状图
```python
country_sales = cleaned_data.groupby("Country")["SalesAmount"].sum().sort_values(ascending = False)
plt.figure(figsize = (11,4))
sns.barplot(x = country_sales.index, y = country_sales.values)
plt.title("各国销售额")
plt.xlabel("国家")
plt.ylabel("销售额")
plt.xticks(rotation = 45, ha = "right")
plt.tight_layout()
plt.savefig("country_sales.png")
plt.show()
```
### （四）分布分析：单次购买数量直方图
```python
plt.figure(figsize = (7,4))
sns.histplot(data = cleaned_data, x = "Quantity", bins = 40)
plt.title("单次订单商品购买数量分布")
plt.xlabel("购买数量")
plt.ylabel("记录条数")
plt.tight_layout()
plt.savefig("quantity_hist.png")
plt.show()
```
### （五）异常校验：购买数量箱线图
```python
plt.figure(figsize=(5,4))
sns.boxplot(data=cleaned_data, y="Quantity")
plt.title("购买数量箱线图（异常值校验）")
plt.tight_layout()
plt.show()
```
### （六）客户复购分析：客户下单频次直方图
```python
cust_order_count = cleaned_data.dropna(subset = ["CustomerID"]).groupby("CustomerID")["InvoiceNo"].nunique()
plt.figure(figsize = (7,4))
sns.histplot(cust_order_count, bins = 30)
plt.title("客户下单频次分布")
plt.xlabel("单个客户下单次数")
plt.ylabel("客户数量")
plt.tight_layout()
plt.savefig("customer_order_count.png")
plt.show()
```
