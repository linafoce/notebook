5.5 输入某年某月某日，判断这一天是这一年的第几天？(可以用 Python 标准 

库)(2018-3-30-lxy) 

```
1. import datetime 

2. def dayofyear(): 

3. year = input("请输入年份：") 

4. month = input("请输入月份：") 

5. day = input("请输入天：") 

6. date1 = datetime.date(year=int(year)，month=int(month)，day=int(day)) 

7. date2 = datetime.date(year=int(year)，month=1，day=1) 

8. return (date1 - date2 + 1).days 
```

5.6 打乱一个排好序的 list 对象 alist？(2018-3-30-lxy) 

1. import random 

2. random.shuffle(alist)



