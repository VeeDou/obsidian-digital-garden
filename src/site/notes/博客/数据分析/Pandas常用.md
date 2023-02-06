---
{"dg-publish":true,"permalink":"/博客/数据分析/Pandas常用/"}
---

---
### 1. 初始化
```python
import pandas as pd import numpy as np
# 1.1 通过“列表”初始化：列表可以是：等长的numpy多维数组、等长的多维的list 
dates = pd.date_range('20190728', periods=3)
df1 = pd.DataFrame(np.random.randn(6,3),index=list('abcdef'),columns=['col1','col3','col88'])
```python
#1.2 通过“列表”组成的字典初始化：等长的numpy多维数组、等长的多维的list 
data = {'A':{'20190728':1,'20190729':1,'20190730':1}, 
'B':{'20190728':2,'20190729':2,'20190730':2},
 'C':{'20190728':3,'20190729':3,'20190730':3},
 'D':{'20190728':4,'20190729':4,'20190730':4} } 
#通过 columns指定获取数据的列和顺序（显式列索引 > 数据列索引） 
df2 = pd.DataFrame(data,columns = ['D','C','B']) 

#1.3 读取文件test.xlsx 中 sheet名为测试数据 的数据，数据中有列名 header 设置为真。 
data = pd.read_excel('test.xlsx',sheet_name = '测试数据',header = True)
```
### 2. 切片
**数据切片3种方式**：通过label（行/列名）、位置（数字坐标）、布尔运算
1. label：行标签    df.loc[]
2. 数字坐标：df.iloc[0:2,0:2]   int类型 行列标签（默认顺序，从0开始） 
3.  布尔运算：对列进行条件判断
示例如下：
```python
#1. 获取index为 a 到index为c的三行数据：
df1.loc['a':'c']
#2. 获取顺序第0行和第1行的两行数据 ：
df1.iloc[0:2]
#3. 通过位置切片,行和列都切割 : 
df1.iloc[0:2,0:2] 
#4. 过布尔运算切片,筛选出col3值小于0的数据：
df1[df1.col3<0] 
#5. 通过布尔运算切片 筛选出col3值小于0的数据,只取指定列：
df1[[df1.col3<0\|df1.col3<0]][['col3','col88'\|'col3','col88']]
```
### 3. 合并
5种数据合并方式：append、join、merge、concat、combine、update
1. **append**：官方建议用concat 代替，concat功能更全（详情见第4）
2. **join** ：功能同SQL的join操作，**基于索引**横向拼接
	指定列需要设为索引 
	df.join(other.set_index('key'), on='key')
3. **merge**：功能同SQL的join操作，**基于指定列横向拼接**
	df1.merge(df2, left_on='lkey', right_on='rkey',suffixes=('_left', '_right'))
4. **concat**：兼顾横向和纵向，**可合并多个**，merge只能合并2个
	纵向：pd.concat([x,y]) ##默认0，纵向拼接index，可以代替append
	横向：pd.concat([x,y],axis=1)，设置为1，水平拼接column
4. **combine**：以特定函数合并
	DataFrame.combine(_other_, _func_, _overwrite=True_)
	例：df1.combine(df2, np.minimum) 相同列取最小值  函数可以lambda
5. **update**：更新某些列数据，直接以后者替换前者
	 df9.update(df10)
	 1. 空值不更新
	 2. 保留原数据长度：只更新原数据有的index/column

### 4. 聚合
聚合：groupby、apply、agg、lambda、map
1. **groupby** 和 **agg**
```python
#1. 分组
df.groupby('col1')
#2.分组后基本函数聚合 
1. df.groupby('col1')['col2'].sum()
2. df.groupby('col1')['col2'].count()
#3. 多列分组和聚合
df.groupby(['co1','col2'])['col3','col4'].agg(['sum','mean'])
```
2. **应用函数**
```python
#1. 基本写法 
lambda x:1 if x>10 else 2
#2.不分组应用
new_ads_add.apply(lambda x : batch_process.get_dynamic_adcreative_spec(x), axis=1)
#3.分组应用
df.groupby('col1').apply(ambda x : (x.last_week_sales - x.last_month_sales/4).mean() , axis=1)  # 对每一组应用自定义函数
#4. 在agg中应用：对col1应用自定义函数
df.groupby(['co1','col2'])['col3','col4'].agg(['sum',lambda x : (x.last_week_sales - x.last_month_sales/4])  
```
3. 两种应用：apply和applymap
	1. `apply()`函数主要用于对DataFrame中的**某一column或row中的元素**执行相同的函数操作
		1. df["C1"].apply(lambda x:x+1) 对column 为 c1的进行操作
		2. df.loc[1].apply(lambda x:x+1)  index为1的行进行操作
	2. `applymap()`函数用于对DataFrame中的**每一个元素**执行相同的函数操作。 
		1. df.applymap(lambda x:x+1)
		2.  df.apply(lambda x:x+1)

5. 排序输出
    df.groupby('col1')['col2'].mean().sort_values(ascending = True)
   
6. 排序
    ```python
    df['rank'] = df.groupby('col1')['col2'].rank('method' = 'dense','ascending' = False)
    ```

6. 唯一值
```python
##取唯一值 + 聚合后字段名
df.groupby('col1').agg(new_col_name = ("col2",'unique'))
```
6. map
```python
#1. python 的map 函数 map(fun, iter) 
map(lambda x:1 if x>10 else 2,[10,20,-1,5,8,2])  
#2.Series的map 函数：有两种映射方式，字典和函数
 # 基础函数映射：*对列进行函数操作*
df['data1'].map(lambda x : "%.3f"%x)
# 基于字典映射 ：以字典的形式进行替换
df['key1'].map({'a':'c',"b":"d"}) #  用map把key1的a改成c，b改成d
# 组合groupby+map：在分组中对列进行函数操作
df.groupby([df[‘日期].map(lambda x : x.isocalendar()[1]),"小组)"]).agg({['消耗金额']:np.sum,"客户id":pd.Series.nunique}) 
```
[[map函数映射.png]]
[[map字典映射.png]]
### 5. 行列转置
```python
#行转列  
#1.pivot 
	df.pivot(index=None, columns=None, values=None)
	#去重以免引起报错
	df1.drop_duplicates().pivot(index='name',columns='subject',values='score')
#2.pivot_table 数据透视表
	# value 为值；index为聚合列；columns为转置列，aggfun为聚合函数，可以是字符也可以是字典
	pivot_table(data, values=None, index=None, columns=None, aggfunc='mean')
	pd.pivot_table(df1,index='name',columns='subject',values='score',aggfunc={'score':'max'})
# 列转行 stack、melt
# 数据
f = pd.DataFrame({'姓名': ['A','B','C'],
                  '英语':[90,60,70],
                  '数学':[80,98,80],
                  '语文':[85,90,75]})
#1. 先重命名index，再重名name,最后重置inde
tmp=df.set_index(['姓名']).stack() 
tmp2=tmp.rename_axis(index=['姓名','科目']) 
tmp2.name='分数' 
tmp2.reset_index()
#2. 直接重置index，接着重命名列
tmp=df.set_index(['姓名']).stack().reset_index() tmp.columns=['姓名','科目','分数']
#3 melt 指定id、值对应变量、新变量名、值名
### id列 理解为 不需要做列转行处理的字段
# value_vars 列转行的字段
# var_name 新字段名称
# _value_name = 新字段对应值字段的名称 
DataFrame.melt(_id_vars=None_, _value_vars=None_, _var_name=None_, _value_name='value')
#未指定 值对应变量，默认剩下的变量都拉进去？
tmp=pd.melt(df,id_vars='姓名',var_name='科目',value_name='分数')
```
---
创建时间:2023-02-05 22:56
更多参考资料：
1. [数据分析学习笔记 - 知乎 (zhihu.com)](https://www.zhihu.com/column/c_1102570753626591232)
2. [ Python实现行转列？！超简单，赶快get起来_严小样儿的博客-CSDN博客](https://blog.csdn.net/Eric_data/article/details/104645738)
