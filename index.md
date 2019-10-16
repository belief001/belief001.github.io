---
layout: default
---

# python3.7 datetime


```python

 import time
 import datetime

 # get current time 
 now_date_time = datetime.datetime.now()  

 # format datetime by format string
 now_str = now_date_time.strftime("%Y/%m/%d %H:%M:%S")

 # change date string to datetime
 str_2_datetime = datetime.datetime.strptime("2019-10-16 12:55:66", "%Y/%m/%d %H:%M:%S")

 # get current timestamp
 now_timestamp=time.time()

 # change date string to timestamp
 str_2_timestamp = time.mktime(time.strptime("2019-10-16 12:55:66", "%Y/%m/%d %H:%M:%S"))

 # timestamp to datetime
 timestamp_2_datetime = datetime.datetime.fromtimestamp(str_2_timestamp)

 # utc to string local date
 utc_date = datetime.datetime.strptime(utc_str ,"%Y-%m-%dT%H:%M:%SZ")
 local_date = utc_date + datetime.timedelta(hours=8)
 local_date_str = datetime.datetime.strftime(local_date, fmt)

```