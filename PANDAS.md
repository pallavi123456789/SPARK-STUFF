| Operation                                        | Spark / PySpark                                                                                                                                                               | Pandas                                                                                                                                           |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Create DataFrame**                             | `df = spark.createDataFrame([(1,'A',100),(2,'B',200)], ['id','name','amount'])`                                                                                               | `df = pd.DataFrame({'id':[1,2],'name':['A','B'],'amount':[100,200]})`                                                                            |
| **Select columns**                               | `df.select('id','amount')`                                                                                                                                                    | `df[['id','amount']]`                                                                                                                            |
| **Filter / WHERE**                               | `df.filter(df.amount > 150)`                                                                                                                                                  | `df[df['amount'] > 150]` or `df.query("amount > 150")`                                                                                           |
| **Filter multiple conditions**                   | `df.filter((df.amount > 150) & (df.id != 1))`                                                                                                                                 | `df[(df['amount'] > 150) & (df['id'] != 1)]`                                                                                                     |
| **Add column with condition**                    | `df.withColumn('tier', F.when(df.amount>150,'High').otherwise('Low'))`                                                                                                        | `df['tier'] = np.where(df['amount']>150,'High','Low')`                                                                                           |
| **Add multi-level tier column**                  | `df.withColumn('tier', F.when(df.amount>300,'High').when(df.amount>150,'Medium').otherwise('Low'))`                                                                           | `python df['tier'] = np.select([df['amount']>300,(df['amount']>150)&(df['amount']<=300),df['amount']<=150], ['High','Medium','Low']) `           |
| **GroupBy & Aggregate**                          | `df.groupBy('category').agg(F.sum('amount1').alias('total'), F.avg('amount2').alias('avg'))`                                                                                  | `df.groupby('category').agg(total=('amount1','sum'), avg=('amount2','mean')).reset_index()`                                                      |
| **Join**                                         | `df1.join(df2, df1.user_id==df2.userid, 'left')`                                                                                                                              | `df1.merge(df2, left_on='user_id', right_on='userid', how='left')`                                                                               |
| **Join + filter in one line**                    | `df1.join(df2,'user_id').where(df2.amount>100)`                                                                                                                               | `df1.merge(df2, on='user_id').query("amount > 100")`                                                                                             |
| **Select only certain columns from both tables** | `df1.select('id','amount').join(df2.select('userid','name'),'id')`                                                                                                            | `df1[['user_id','amount']].merge(df2[['userid','name']], left_on='user_id', right_on='userid')`                                                  |
| **Row number / window function**                 | `from pyspark.sql import Window as W from pyspark.sql import functions as F w = W.partitionBy('user_id').orderBy('date') df.withColumn('row_number', F.row_number().over(w))` | `df['row_number'] = df.sort_values(['user_id','date']).groupby('user_id').cumcount() + 1`                                                        |
| **Rank / Dense Rank**                            | `df.withColumn('rank', F.rank().over(w))` <br> `df.withColumn('dense_rank', F.dense_rank().over(w))`                                                                          | `df['rank'] = df.groupby('user_id')['amount'].rank(method='min')` <br> `df['dense_rank'] = df.groupby('user_id')['amount'].rank(method='dense')` |
| **Lag / Lead**                                   | `df.withColumn('prev_amount', F.lag('amount',1).over(w))` <br> `df.withColumn('next_amount', F.lead('amount',1).over(w))`                                                     | `df['prev_amount'] = df.groupby('user_id')['amount'].shift(1)` <br> `df['next_amount'] = df.groupby('user_id')['amount'].shift(-1)`              |
| **Month-over-Month % change**                    | `df.withColumn('MoM_change', (F.col('amount')-F.lag('amount',1).over(w))/F.lag('amount',1).over(w) * 100)`                                                                    | `df['MoM_change'] = df.groupby('user_id')['amount'].pct_change() * 100`                                                                          |
| **Drop column**                                  | `df.drop('userid')`                                                                                                                                                           | `df.drop(columns=['userid'])`                                                                                                                    |

```
import pandas as pd
import numpy as np

# Raw Sales Data (Note: Amount is a string, which is a common 'messy' data issue)
sales_data = {
    'sale_id': [101, 102, 103, 104, 105, 106],
    'user_id': [1, 2, 1, 3, 2 ,4],
    'amount': ['150.50', '200', '50.25', '300.10', '120', '500'],
    'amount_2': ['150.50', '200', '50.25', '300.10', '120', '500'],
    'date': ['2024-01-01', '2024-01-01', '2024-01-02', '', '2024-01-03','2024-01-03'],
    'name':[101, 102, 103, 104, '', 106]
}

# User Data
user_data = {
    'userid': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie']
}

sd = pd.DataFrame(sales_data) 
ud = pd.DataFrame(user_data)
print (sd)
print (ud)

sd['amount']=pd.to_numeric(sd['amount'])
sd['amount_2']=pd.to_numeric(sd['amount_2'])
sd['date']=pd.to_datetime(sd['date'])
sd=sd.rename(columns={"amount": "float_amount" ,"date":"date_time" , "name":"uid"})
print (sd)
print (ud)

# --- TRANSFORMATION: JOIN ---
df_stg = (
pd.merge(sd[['sale_id','user_id','float_amount','date_time','amount_2']], ud, left_on="user_id", right_on="userid", how="left").query("~name.isna()").drop(columns=["userid"])
).rename(columns={"float_amount": "amount_new" ,"date_time":"date_new"})

print(df_stg)

# --- TRANSFORMATION: CASE STATEMENT ---
# If amount > 150, 'High', else 'Low'
df_stg['tier']=np.where(
    df_stg['amount_new']<=60 , 'Low' ,
    np.where((df_stg['amount_new']>50) & (df_stg['amount_new']<80) , 'Med' ,'High')
    )
print(df_stg)  

# --- TRANSFORMATION: FILTER ---
df_stg = df_stg[df_stg['date_new'].isna()==False]

print(df_stg)  

# --- TRANSFORMATION: GROUP BY ---

summary = df_stg.groupby("name")["amount_new"].agg(["sum" , "mean"]).reset_index()
print(summary)  

summary = df_stg.groupby("name").agg({
    'amount_new': 'sum',   # sum amount1
    'amount_2': 'mean'   # average amount2
}).reset_index()
print(summary)  

# Step 4: Pivot for the Final Report

report = df_stg.pivot_table(index='name', columns='date_new', values='amount_new', aggfunc='sum').fillna(0).reset_index()

print (report)



```
