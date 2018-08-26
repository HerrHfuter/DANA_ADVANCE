
# 2016年8月上海摩拜单车数据分析

## 1、数据整理


```python
import pandas as pd
import numpy as np
from math import radians,cos,sin,asin,sqrt
import time
```


```python
df = pd.read_csv('mobike_shanghai_sample_updated.csv')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>orderid</th>
      <th>bikeid</th>
      <th>userid</th>
      <th>start_time</th>
      <th>start_location_x</th>
      <th>start_location_y</th>
      <th>end_time</th>
      <th>end_location_x</th>
      <th>end_location_y</th>
      <th>track</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78387</td>
      <td>158357</td>
      <td>10080</td>
      <td>2016-08-20 06:57</td>
      <td>121.348</td>
      <td>31.389</td>
      <td>2016-08-20 07:04</td>
      <td>121.357</td>
      <td>31.388</td>
      <td>121.347,31.392#121.348,31.389#121.349,31.390#1...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>891333</td>
      <td>92776</td>
      <td>6605</td>
      <td>2016-08-29 19:09</td>
      <td>121.508</td>
      <td>31.279</td>
      <td>2016-08-29 19:31</td>
      <td>121.489</td>
      <td>31.271</td>
      <td>121.489,31.270#121.489,31.271#121.490,31.270#1...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1106623</td>
      <td>152045</td>
      <td>8876</td>
      <td>2016-08-13 16:17</td>
      <td>121.383</td>
      <td>31.254</td>
      <td>2016-08-13 16:36</td>
      <td>121.405</td>
      <td>31.248</td>
      <td>121.381,31.251#121.382,31.251#121.382,31.252#1...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1389484</td>
      <td>196259</td>
      <td>10648</td>
      <td>2016-08-23 21:34</td>
      <td>121.484</td>
      <td>31.320</td>
      <td>2016-08-23 21:43</td>
      <td>121.471</td>
      <td>31.325</td>
      <td>121.471,31.325#121.472,31.325#121.473,31.324#1...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>188537</td>
      <td>78208</td>
      <td>11735</td>
      <td>2016-08-16 07:32</td>
      <td>121.407</td>
      <td>31.292</td>
      <td>2016-08-16 07:41</td>
      <td>121.418</td>
      <td>31.288</td>
      <td>121.407,31.291#121.407,31.292#121.408,31.291#1...</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 102361 entries, 0 to 102360
    Data columns (total 10 columns):
    orderid             102361 non-null int64
    bikeid              102361 non-null int64
    userid              102361 non-null int64
    start_time          102361 non-null object
    start_location_x    102361 non-null float64
    start_location_y    102361 non-null float64
    end_time            102361 non-null object
    end_location_x      102361 non-null float64
    end_location_y      102361 non-null float64
    track               102361 non-null object
    dtypes: float64(4), int64(3), object(3)
    memory usage: 6.6+ MB
    

通过观察，准备将数据集分成两部分，一部分包含了订单信息（原始数据中除去track列之外的其他信息），另一部分，包含了订单的轨迹路线信息（包含订单id，经度，维度以及该位置处于订单路径中的哪一阶段）。
### （1）订单信息表order_info


```python
#订单信息order_info
order_info = df.drop('track',axis=1)
order_info.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>orderid</th>
      <th>bikeid</th>
      <th>userid</th>
      <th>start_time</th>
      <th>start_location_x</th>
      <th>start_location_y</th>
      <th>end_time</th>
      <th>end_location_x</th>
      <th>end_location_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78387</td>
      <td>158357</td>
      <td>10080</td>
      <td>2016-08-20 06:57</td>
      <td>121.348</td>
      <td>31.389</td>
      <td>2016-08-20 07:04</td>
      <td>121.357</td>
      <td>31.388</td>
    </tr>
    <tr>
      <th>1</th>
      <td>891333</td>
      <td>92776</td>
      <td>6605</td>
      <td>2016-08-29 19:09</td>
      <td>121.508</td>
      <td>31.279</td>
      <td>2016-08-29 19:31</td>
      <td>121.489</td>
      <td>31.271</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1106623</td>
      <td>152045</td>
      <td>8876</td>
      <td>2016-08-13 16:17</td>
      <td>121.383</td>
      <td>31.254</td>
      <td>2016-08-13 16:36</td>
      <td>121.405</td>
      <td>31.248</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1389484</td>
      <td>196259</td>
      <td>10648</td>
      <td>2016-08-23 21:34</td>
      <td>121.484</td>
      <td>31.320</td>
      <td>2016-08-23 21:43</td>
      <td>121.471</td>
      <td>31.325</td>
    </tr>
    <tr>
      <th>4</th>
      <td>188537</td>
      <td>78208</td>
      <td>11735</td>
      <td>2016-08-16 07:32</td>
      <td>121.407</td>
      <td>31.292</td>
      <td>2016-08-16 07:41</td>
      <td>121.418</td>
      <td>31.288</td>
    </tr>
  </tbody>
</table>
</div>



添加骑行时间duration和骑行距离distance字段


```python
#计算结束时间和起始时间的差值，并将其转换成小时，保留2位小数
order_info['duration']= pd.to_datetime(order_info.end_time)-pd.to_datetime(order_info.start_time)
order_info['duration']= order_info.duration.apply(lambda x:round(x.total_seconds()/3600,2))
order_info.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>orderid</th>
      <th>bikeid</th>
      <th>userid</th>
      <th>start_time</th>
      <th>start_location_x</th>
      <th>start_location_y</th>
      <th>end_time</th>
      <th>end_location_x</th>
      <th>end_location_y</th>
      <th>duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78387</td>
      <td>158357</td>
      <td>10080</td>
      <td>2016-08-20 06:57</td>
      <td>121.348</td>
      <td>31.389</td>
      <td>2016-08-20 07:04</td>
      <td>121.357</td>
      <td>31.388</td>
      <td>0.12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>891333</td>
      <td>92776</td>
      <td>6605</td>
      <td>2016-08-29 19:09</td>
      <td>121.508</td>
      <td>31.279</td>
      <td>2016-08-29 19:31</td>
      <td>121.489</td>
      <td>31.271</td>
      <td>0.37</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1106623</td>
      <td>152045</td>
      <td>8876</td>
      <td>2016-08-13 16:17</td>
      <td>121.383</td>
      <td>31.254</td>
      <td>2016-08-13 16:36</td>
      <td>121.405</td>
      <td>31.248</td>
      <td>0.32</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1389484</td>
      <td>196259</td>
      <td>10648</td>
      <td>2016-08-23 21:34</td>
      <td>121.484</td>
      <td>31.320</td>
      <td>2016-08-23 21:43</td>
      <td>121.471</td>
      <td>31.325</td>
      <td>0.15</td>
    </tr>
    <tr>
      <th>4</th>
      <td>188537</td>
      <td>78208</td>
      <td>11735</td>
      <td>2016-08-16 07:32</td>
      <td>121.407</td>
      <td>31.292</td>
      <td>2016-08-16 07:41</td>
      <td>121.418</td>
      <td>31.288</td>
      <td>0.15</td>
    </tr>
  </tbody>
</table>
</div>




```python
#通过起始的经纬度信息，来计算骑行距离，单位按公里，保留两位小数
def haversine(lon1, lat1, lon2, lat2): # 经度1，纬度1，经度2，纬度2 （十进制度数）
    # 将十进制度数转化为弧度
    # math.degrees(x):为弧度转换为角度
    # math.radians(x):为角度转换为弧度
    lon1, lat1, lon2, lat2 = map(radians, [lon1, lat1, lon2, lat2])
    # haversine公式
    dlon = lon2 - lon1
    dlat = lat2 - lat1
    a = sin( dlat /2 ) **2 + cos(lat1) * cos(lat2) * sin( dlon /2 ) **2
    c = 2 * asin(sqrt(a))
    r = 6371 # 地球平均半径，单位为公里
    return round(c * r,2)
```


```python
order_info['distance']=order_info.apply(lambda x : haversine(x.start_location_x, x.start_location_y, x.end_location_x, x.end_location_y),axis=1)
order_info.sample(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>orderid</th>
      <th>bikeid</th>
      <th>userid</th>
      <th>start_time</th>
      <th>start_location_x</th>
      <th>start_location_y</th>
      <th>end_time</th>
      <th>end_location_x</th>
      <th>end_location_y</th>
      <th>duration</th>
      <th>distance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>64013</th>
      <td>474503</td>
      <td>88988</td>
      <td>11701</td>
      <td>2016-08-15 09:18</td>
      <td>121.463</td>
      <td>31.296</td>
      <td>2016-08-15 09:26</td>
      <td>121.458</td>
      <td>31.287</td>
      <td>0.13</td>
      <td>1.11</td>
    </tr>
    <tr>
      <th>28836</th>
      <td>1195551</td>
      <td>218407</td>
      <td>13579</td>
      <td>2016-08-30 14:31</td>
      <td>121.430</td>
      <td>31.221</td>
      <td>2016-08-30 14:58</td>
      <td>121.417</td>
      <td>31.202</td>
      <td>0.45</td>
      <td>2.45</td>
    </tr>
    <tr>
      <th>80807</th>
      <td>758346</td>
      <td>289107</td>
      <td>9228</td>
      <td>2016-08-25 17:25</td>
      <td>121.345</td>
      <td>31.272</td>
      <td>2016-08-25 17:27</td>
      <td>121.348</td>
      <td>31.270</td>
      <td>0.03</td>
      <td>0.36</td>
    </tr>
    <tr>
      <th>31809</th>
      <td>1053804</td>
      <td>145327</td>
      <td>17459</td>
      <td>2016-08-29 20:53</td>
      <td>121.413</td>
      <td>31.156</td>
      <td>2016-08-29 20:57</td>
      <td>121.416</td>
      <td>31.150</td>
      <td>0.07</td>
      <td>0.73</td>
    </tr>
    <tr>
      <th>45105</th>
      <td>1272583</td>
      <td>140311</td>
      <td>1464</td>
      <td>2016-08-27 17:47</td>
      <td>121.474</td>
      <td>31.283</td>
      <td>2016-08-27 17:53</td>
      <td>121.477</td>
      <td>31.289</td>
      <td>0.10</td>
      <td>0.73</td>
    </tr>
  </tbody>
</table>
</div>




```python
#保留文件
order_info.to_csv('order_info.csv',index=False)
```

### （2）订单路线表order_track


```python
order_track = df[['orderid','track']]
order_track.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>orderid</th>
      <th>track</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78387</td>
      <td>121.347,31.392#121.348,31.389#121.349,31.390#1...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>891333</td>
      <td>121.489,31.270#121.489,31.271#121.490,31.270#1...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1106623</td>
      <td>121.381,31.251#121.382,31.251#121.382,31.252#1...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1389484</td>
      <td>121.471,31.325#121.472,31.325#121.473,31.324#1...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>188537</td>
      <td>121.407,31.291#121.407,31.292#121.408,31.291#1...</td>
    </tr>
  </tbody>
</table>
</div>




```python
order_track.shape
```




    (102361, 2)




```python
# 由于电脑内存太小，直接melt会报错MemoryError，所以这里将原始数据拆分处理，每20000条进行一次处理得到每个订单的轨迹点
track_df = pd.DataFrame(columns=['orderid','position'])
for i in range(order_track.shape[0]//20000+1):
    order_track_sample = order_track.loc[i*20000:(i+1)*20000-1,:]
    mtest = order_track_sample.track.str.split('#',expand=True)
    mtest['orderid']=order_track_sample['orderid']
    mdf = pd.melt(mtest,id_vars=['orderid'],value_name='position').dropna()
    mdf.drop('variable',axis=1,inplace=True)
    track_df = track_df.append(mdf,ignore_index=True) 
```


```python
track_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>orderid</th>
      <th>position</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78387</td>
      <td>121.347,31.392</td>
    </tr>
    <tr>
      <th>1</th>
      <td>891333</td>
      <td>121.489,31.270</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1106623</td>
      <td>121.381,31.251</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1389484</td>
      <td>121.471,31.325</td>
    </tr>
    <tr>
      <th>4</th>
      <td>188537</td>
      <td>121.407,31.291</td>
    </tr>
  </tbody>
</table>
</div>




```python
#简单对得到的trace_df进行整理,将位置信息拆成经纬度
track_df['loc_x']=track_df.position.str.split(',',expand=True).iloc[:,0]
track_df['loc_y']=track_df.position.str.split(',',expand=True).iloc[:,1]
track_df.drop('position',axis=1,inplace=True)
```


```python
track_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>orderid</th>
      <th>loc_x</th>
      <th>loc_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78387</td>
      <td>121.347</td>
      <td>31.392</td>
    </tr>
    <tr>
      <th>1</th>
      <td>891333</td>
      <td>121.489</td>
      <td>31.270</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1106623</td>
      <td>121.381</td>
      <td>31.251</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1389484</td>
      <td>121.471</td>
      <td>31.325</td>
    </tr>
    <tr>
      <th>4</th>
      <td>188537</td>
      <td>121.407</td>
      <td>31.291</td>
    </tr>
  </tbody>
</table>
</div>




```python
track_df.dtypes
```




    orderid    object
    loc_x      object
    loc_y      object
    dtype: object




```python
#将track_df的数据类型与原数据的保持一致
track_df['orderid']=track_df.orderid.astype('int64')
track_df['loc_x']=track_df.loc_x.astype('float64')
#由于整合后的loc_y列中的数据有问题，存在'31.29\\'这种的不能直接转float,先处理一下再转换类型
track_df.loc_y=track_df.loc_y.str.extract('(\d+\.?\d+)',expand=False).astype('float64')
track_df.dtypes
```




    orderid      int64
    loc_x      float64
    loc_y      float64
    dtype: object



通过上述计算得到了每个订单的轨迹点，接下来添加路径点顺序列。根据起始位置的经纬度，来确定排序方式，对同一订单的路径点的经纬度进行排序，确定路径点的顺序即可。


```python
#逐个orderid进行排序生成轨迹点,....跑了7个小时。。。。有没有好的优化方法啊？
starttime = time.clock()
final_track = pd.DataFrame(columns=['trackid','orderid','loc_x','loc_y'])
idarray = track_df.orderid.unique()
for ids in idarray:
    tmp_df = track_df[track_df['orderid']==ids]
    tmp_organ = df[df['orderid']==ids]
    start_loc_x = tmp_organ.start_location_x.values[0]
    start_loc_y = tmp_organ.start_location_y.values[0]
    end_loc_x = tmp_organ.end_location_x.values[0]
    end_loc_y = tmp_organ.end_location_y.values[0]
    xflag = start_loc_x < end_loc_x
    yflag = start_loc_y < end_loc_y
    tmp_df = tmp_df.sort_values(by=['loc_x','loc_y'],ascending=[xflag,yflag]).reset_index(drop=True).reset_index()
    tmp_df.rename(columns={'index':'trackid'},inplace=True)
    tmp_df.trackid = tmp_df.trackid+2
    #将起始的经纬度信息添加进来
    mdict = {'trackid':[1,tmp_df.shape[0]+2],'orderid':[ids,ids],'loc_x':[start_loc_x,end_loc_x],'loc_y':[start_loc_y,end_loc_y]}
    se_df = pd.DataFrame(mdict)
    tmp_df = tmp_df.append(se_df,ignore_index=True)
    tmp_df = tmp_df.sort_values(by='trackid').reset_index(drop=True)
    final_track = final_track.append(tmp_df,ignore_index=True)
endtime = time.clock()
print(endtime-starttime)
```

    27082.707907075666
    


```python
final_track.sample(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>trackid</th>
      <th>orderid</th>
      <th>loc_x</th>
      <th>loc_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>353509</th>
      <td>38</td>
      <td>668328</td>
      <td>121.434</td>
      <td>31.212</td>
    </tr>
    <tr>
      <th>1697880</th>
      <td>18</td>
      <td>912082</td>
      <td>121.254</td>
      <td>31.373</td>
    </tr>
    <tr>
      <th>2514254</th>
      <td>36</td>
      <td>1318060</td>
      <td>121.392</td>
      <td>31.231</td>
    </tr>
    <tr>
      <th>718238</th>
      <td>5</td>
      <td>597354</td>
      <td>121.501</td>
      <td>31.262</td>
    </tr>
    <tr>
      <th>2406442</th>
      <td>6</td>
      <td>754919</td>
      <td>121.513</td>
      <td>31.303</td>
    </tr>
  </tbody>
</table>
</div>




```python
#########最终生成结果导出
final_track.to_csv('order_track.csv',index=False)
```

## 2、总结  
通过对原始数据的简单整理，得到了摩拜订单数据order_info.csv和摩拜订单轨迹数据order_track.info。结合Tableau对整理好的数据进行了简单的分析，创建了一个[摩拜的Tableau可视化故事](https://public.tableau.com/profile/heer5135#!/vizhome/mobike_v1/1),后面根据相关反馈，对可视化故事进行了调整，得到了[上海Mobike故事的最终版](https://public.tableau.com/profile/heer5135#!/vizhome/mobike_v2/1)。分析的出发点从订单的数量变化出发的。
- 首先，结合起始时间、地理位置（经纬度）和订单数量的统计，发现从8月1日至8月31日，订单的数量增长很快，并且订单的分布范围也更广。
- 其次，对时间进行进行细分，从两个方面观察订单数量的变化，从工作日的角度观察观察周末和工作日的平均订单数量差异；从一天的时间段的角度来说，订单数量有两次高峰期，所以这里对时间段进行了分组，把6点-9点分为早高峰，把17点-20点分为早高峰，发现这两个时间段的订单数量占了一天订单数量的一多半。
- 再次，对订单的骑行时间和骑行距离做了一个统计，发现约76%的订单的骑行时间在20分钟以内，约89%的订单的骑行距离在3公里以内。另外，还对每天不同时间段的平均骑行时间和骑行距离做了个统计，发现早高峰时的平均骑行距离和平均骑行时间最短，由于人们赶着去上班，所以骑行速度也就加快，而晚上下班后，不赶时间，更愿意骑慢一点。
- 最后，通过日期和时间段的选择，可以观察不同时间段的骑行轨迹，摩拜和人们的生活的联系也越来越紧密，越来越多的人选择摩拜作为短程交通的工具。

## 3、设计与反馈  
### （1）设计  
- 首先，通过观察发现数据中只包含了8月份的订单信息，所以考虑每一天的订单数量变化，从订单范围和订单数量两个角度去统计。
- 之后，将时间进一步细分，观察周一至周日24小时内的订单变化，发现订单数量呈双峰分布，在上下班的时间段出现订单的峰值，所以这里对时间段进行了分组，夜间（21点-5点），早高峰（6点-9点），日常时间（10点-16点），晚高峰时间（17点-20点）。发现了早晚高峰的订单数占了一天总订单数的一多半。此外还发现周末在9点-17点的平均订单数量，要高于周内的同期水平，说明人们愿意在周末使用摩拜，周末的平均订单数和周内基本持平。
- 由于还增加了骑行时间和骑行距离的统计字段，所以对订单的骑行时间和骑行距离进行分组统计，骑行时间分为20分钟以内、20分钟-1小时和1小时以上，骑行距离分为3公里以内、3-10公里和10公里以上。发现大部分的订单骑行时间都在20分钟之内，骑行距离在3公里以内。说明人们愿意选择摩拜进行短距离交通。
- 根据之前创建的时间段，来观察不同时间段的骑行距离和骑行时间。发现，日常时间和早高峰的平均骑行距离最短，早高峰时的平均骑行时间最短。夜间和晚高峰的骑行距离和骑行时间都较长。所以，上下班通勤摩拜的使用率很高，并且早高峰骑得快，赶着去上班，下班后时间则相对充裕，骑行速度自然就放慢了。
- 最后，由于轨迹数据点很多，所以利用时间进行筛选，观察不同日期每个时间段的路径信息。可以发现轨迹路径也呈现多样化，说明摩拜越来越融入人们的生活中。  

### （2）设计思路
- 利用起始经纬度信息来创建订单的地理分布图，加入时间信息，添加页面动态效果
- 折线图和颜色的结合，观察订单总数随时间的变化情况
- 利用突出显示表来展现每日订单的具体数量
- 考虑到周内和工作日人们生活方式的不同，利用工作日，观察每一天不同时间段的平均订单数量变化；再对时间段分组，结合堆积条形图，观察不同工作日的平均订单组成
- 对骑行时间和骑行距离分组，利用饼状图和环形图，观察不同骑行时间和骑行距离的订单数量
- 利用双轴散点图，观察不同时间段的订单的平均骑行时间和骑行距离
- 最后利用轨迹信息，结合筛选器，呈现不同时间段的订单地理轨迹信息图

### （3）反馈  
在[第一版故事](https://public.tableau.com/profile/heer5135#!/vizhome/mobike_v1/1)创建出来后，通过交流，得到了下列反馈意见。
- 故事的布局不合理，要考虑故事的连贯性；
- 每一页的细节进行优化，包括标题，图例，颜色等细节的调整
- 故事的描述不够精简

根据反馈意见，进行了调整，得到了故事的[最终版](https://public.tableau.com/profile/heer5135#!/vizhome/mobike_v2/1)。

# 4、参考资料  
- 根据两点经纬度计算直线距离（https://blog.csdn.net/vernice/article/details/46581361）
- 共享单车分析报告（https://www.jianshu.com/p/4394e39d62c8）
- 2017共享单车大数据报告（https://blog.csdn.net/qq_19600291/article/details/78953966）
- Tableau教程创建路径（https://onlinehelp.tableau.com/current/pro/desktop/zh-cn/maps_howto_origin_destination.html）
- Tableau其他相关教程
