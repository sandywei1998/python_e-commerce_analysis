# 电商数据分析SOP
> 适用：电商订单数据集 pandas 数据清洗、可视化标准作业流程

## 一、读取数据
### （一）启动Jupyter Notebook
命令行输入，cmd终端以管理员运行：
```cmd
jupyter notebook --notebook-dir="文件存放路径"
```
### （二）引入pandas库、matploylib、seaborn并且运行生效
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```
### （三）读取原始数据表
```python
original_data = pd.read_csv("你的数据源文件名.csv")
```
## 二、评估数据
### （一）评估数据整齐度
sample方法
```python
original_data.sample(10)
```
### （二）评估数据干净度
使用info方法和describe方法
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
> 目标：基于清洗完成的 cleaned_data 做多维度可视化，挖掘业务特征，复核清洗质量；
> 规范：**一个单元格绘制一张图**，单独调试，单个报错不会影响其他图表。
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
## 六、营收归因分析（进阶分析）
> 说明：本数据集无广告渠道、点击日志，无法做营销触点归因；可开展业务营收归因，拆解收入来源。
> 分析目标：识别营收贡献主体（商品/客户类型/国家），定位业务增长支点。

### （一）商品营收归因
```python
# 1. 按商品名称汇总销售额
product_sales = cleaned_data.groupby("Description")["SalesAmount"].sum().sort_values(ascending=False)

# 2. 单品销售额占总销售额比例 + 累计占比
product_df = product_sales.reset_index()
product_df["percent"] = product_df["SalesAmount"] / product_df["SalesAmount"].sum()
product_df["cum_percent"] = product_df["percent"].cumsum()

# 查看结果
product_df.head(10)
```
### （二）客户类型归因
```python
# 获取每位客户的首次下单时间
customer_first_order = cleaned_data.groupby("CustomerID")["InvoiceDate"].min().reset_index()
customer_first_order.columns = ["CustomerID","FirstInvoiceDate"]

# 合并首单时间至主数据集
data_with_first = pd.merge(cleaned_data, customer_first_order, on = "CustomerID")

# 标记新客、老客：首次下单为新客，非首次为老客
data_with_first["CustomerType"] = data_with_first.apply(
    lambda x: "New Customer" if x["InvoiceDate"] == x["FirstInvoiceDate"] else "Repeat Customer", axis = 1
)

# 按客户类型汇总营收、计算占比
cust_type_attribution = data_with_first.groupby("CustomerType")["SalesAmount"].sum()

cust_type_attribution / cust_type_attribution.sum()
```
### （三）区域市场营收归因
```python
# 按国家区域汇总营收并计算占比
country_attribution = cleaned_data.groupby("Country")["SalesAmount"].sum().sort_values(ascending = False)
country_attribution / country_attribution.sum()
```
## 七、客户转化漏斗分析
> 说明：本数据集仅包含订单交易记录，无网站访问、浏览、加购行为，无法构建流量漏斗；基于客户订单行为，构建【首单→复购→高价值客户】交易复购漏斗，评估客户留存转化能力。

### 分析目标
统计客户从首次下单，到多次复购的逐级转化情况，识别客户流失节点，评估客户粘性，为客户运营策略提供依据。

### 代码实现
```python
import pandas as pd
import matplotlib.pyplot as plt

# 1. 统计每个客户的订单总次数
customer_order_count = cleaned_data.groupby("CustomerID")["InvoiceNo"].nunique().reset_index()
customer_order_count.columns = ["CustomerID", "order_times"]

# 2. 按下单次数划分客户阶段
def customer_level(times):
    if times == 1:
        return "首单客户"
    elif times ==2:
        return "2次复购客户"
    else:
        return "3次及以上高价值客户"

customer_order_count["customer_level"] = customer_order_count["order_times"].apply(customer_level)

# 3. 统计各阶段客户数量，构造漏斗表
funnel_df = customer_order_count["customer_level"].value_counts().reset_index()
funnel_df.columns = ["stage", "user_count"]
# 固定漏斗顺序
stage_order = ["首单客户", "2次复购客户", "3次及以上高价值客户"]
funnel_df["stage"] = pd.Categorical(funnel_df["stage"], categories=stage_order, ordered=True)
funnel_df = funnel_df.sort_values("stage")

# 4. 计算整体转化率、逐级转化率
funnel_df["conversion_rate"] = funnel_df["user_count"] / funnel_df["user_count"].iloc[0]
funnel_df["step_rate"] = funnel_df["user_count"] / funnel_df["user_count"].shift(1)

funnel_df
