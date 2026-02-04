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
