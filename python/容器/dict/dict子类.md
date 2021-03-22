![image-20201122145737331](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201122145737331.png)

如果一个类要继承dict
继承UserDict不要继承dict,dict有可能不会调用父类的方法



Defaultdict() 这个dict里面__missing__的魔法函数