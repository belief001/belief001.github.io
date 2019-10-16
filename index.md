---
layout: post
---

# python3.7 datetime


```python

 import time
 import datetime

 # 获取当前时间
 now_date_time = datetime.datetime.now()  # 格式为2019-02-06 11:47:36.406318

 # 格式化
 now_str = now_date_time.strftime("%Y/%m/%d %H:%M:%S")

 # 字符串日期转datetime
 str_2_datetime = datetime.datetime.strptime("2019-10-16 12:55:66", "%Y/%m/%d %H:%M:%S")

 # 获取当前时间时间戳 
 now_timestamp=time.time()

 # 字符串日期转时间戳
 str_2_timestamp = time.mktime(time.strptime("2019-10-16 12:55:66", "%Y/%m/%d %H:%M:%S"))

 # 时间戳转datetime格式
 timestamp_2_datetime = datetime.datetime.fromtimestamp(str_2_timestamp)

 # UTC日期转本地时间， 中国时区 +8
 utc_date = datetime.datetime.strptime(utc_str ,"%Y-%m-%dT%H:%M:%SZ")
 local_date = utc_date + datetime.timedelta(hours=8)
 local_date_str = datetime.datetime.strftime(local_date, fmt)

```