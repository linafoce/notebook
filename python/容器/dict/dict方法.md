## 深拷贝，浅拷贝

```python
a = {
    "hobby1":{"1":"lala"},
    "hobby2":"2"
}

import copy

#浅拷贝会改变原来的值
newDict = a.copy()
##深拷贝值不会变
deepDict = copy.deepcopy(a.copy())
newDict['hobby1']['1'] = "haha"

print(a.items())
print(newDict.items())
print(deepDict.items())

​```
dict_items([('hobby1', {'1': 'haha'}), ('hobby2', '2')])
dict_items([('hobby1', {'1': 'haha'}), ('hobby2', '2')])
dict_items([('hobby1', {'1': 'lala'}), ('hobby2', '2')])
​```


#dict.fromKeys
new_list = ['test1','test2']
newDict = dict.fromkeys(new_list,'aa')

print(newDict)


#get方法可以防止key不存在的wenti 
emptyDict = newDict.get("bobby",{})
print(emptyDict)

#如果有就get，没有就set
value = newDict.setdefault('test1','bb')
print(value)
print(newDict)
```

