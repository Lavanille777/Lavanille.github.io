---
layout: post
title: 集合对象
category: Objective-C
---

> 只是回顾一下基础。关于本篇中的哈希存储，会有一篇单独的文介绍。

## NSArray

不可变数组，只能存储OC对象，基础类型可以装箱后存储。

### 创建方法

实例方法:

```
NSArray * array = [[NSArray alloc] initWithObjects:@"one",@"2",@"2.3",@"a",nil];
```

类方法:

```
NSArray * array1 = [NSArray arrayWithObjects:@"one",@"2",@"2.3",@"a",nil];
```

字面量方法:

```
NSArray * array3 = @[@"one#",@"23",@"2.3",@"a"];
```

复制方法(不同于copy,使用了新的地址):

```
NSArray * sorteArr = [[NSArray alloc]initWithArray:arr];
```

> 注意一个小问题，在初始化数组时，nil是数组的结束符，因此我们希望在数组中加入一个空元素时应该使用 [NSNull null] 方法，数组中的空元素是NSNull类型。

### 查询方法

从数组中通过索引获取某个元素:

```
Person * p = [array4 objectAtIndex:0];
//或是
Person * p = array4[0];
```

通过元素获取它的索引:

```
NSUInteger index = [arr indexOfObject:str];
```

获取数组元素数量:

```
NSUInteger count = [array4 count];
```

判断数组是否包含某个元素:

```
[array4 containsObject:p4]
```

### 排序方法

```

//传入一个CF函数，返回值大于0则交换元素，否则不交换
NSArray * array2 = [array sortedArrayUsingSelect:sel];
SEL sel = @selector(compare:);  //a-b-c-d-f
SEL sel2 = @selector(isGreatThan:); //a-b-c-d-f
SEL sel3 = @selector(isLessThan:);  //f-d-c-b-a
//因为compare:只在左边大于右边时返回值为1，因此效果等同于isGreatThan:，返回数组升序。


//如果希望自定义排序函数可以使用block方式，依然是根据返回值交换元素

NSArray * sortedArr = [arr sortedArrayUsingComparator:^NSComparisonResult(id obj1, id obj2) {
        //自定义函数，obj1代表左边对象
    }];
    
//使用排序描述符排序
//key代表按对象的某个属性排序，self代表按对象自身排序，ascending为YES代表升序。

    NSSortDescriptor *des = [[NSSortDescriptor alloc] initWithKey:@"self" ascending:YES];
    
    NSArray *newArray = [array sortedArrayUsingDescriptors:@[des]];

```

## NSMutableArray

可变数组,NSArray的子类。

### 创建方法

大致同于NSArray，但NSMutableArray不能用字面量方法初始化。


### 增删方法

```
//添加元素
[muArr addObject:@"one"];
//添加其他数组的元素
[muArr addObjectsFromArray:array];
//在指定位置插入元素
[muArr insetObject:@"a" atIndex:1];
//删除元素  会通过对象地址删除数组中所有的用同一个地址的对象
[muArr removeObject:@"one"];
//通过索引方式删除对象(索引值不能数组越界)
[muArr removeObjectAtIndex:0];
//删除所有元素
[muArr removeAllObjects];
//交换数组元素
[muArr exchangeObjectAtIndex:i withObjectAtIndex:j]
```


## NSDictionary

不可变字典，本质是键值对数组，采用哈希存储结构，因此键必须是可哈希的OC对象（不可变的，例如字符串，数值等等）并且要是唯一的(重复的键的键值对不会保存)，值可以使任意类型的OC对象，可以重复。

### 创建方法

实例方法：

```
NSDictionary * dic = [[NSDictionary alloc] initWithObjectsAndKeys:@"one",@"1",@"two",@"2",nil];
```

字面量方法：

```
NSDicitonary * dic2 = @{@"dic":dic,@"num":num,@"array":array};
```

与数组类似，它也有类方法和复制方法来初始化，不再赘述。
        
### 查询方法

获取字典的长度(键的个数):

```
NSUInteger count = [dic2 count];
```

通过key取值:

```
NSString * arr = [dic3 objectForKey:@"array"];
```

简写:

```
NSDictionary * dic4 = dic3[@"dic"];
```

forin遍历字典（只能得到key）:

```
for (id key in dic) {
	NSLog(@"%@", key);
}
```

## NSMutableDictionary

可变字典，NSDictionary的子类。

### 创建方法

同不可变字典，不能使用字面量方法。

### 增删方法

```
//向字典中插入数据
[muDic2 setObject:@"two" forKey:@"2"];

//删除数据
[muDic2 removeObjectForKey:@"2"];

//全部删除
[muDic removeAllObjects];
```

## NSSet

不可变集合，哈希存储结构的数组，无序，去重。

NSSet只能使用描述符排序，并且返回的是NSArray。

其他使用方法与数组基本一致，不再赘述。


## 相互转换

```
//数组-->集合
NSSet * set = [[NSSet alloc] initWithArray:array];

//集合-->数组
NSArray *array = [set  allObjects];

//字典-->数组
NSDictionary * dic = @{@"1":@"two",@"2":@"kk"};
NSArray * keysArr = [dic allKeys];
NSArray * valuaesArr = [dic allValues];

//字符串-->数组  (按字符分割)
NSString * str = @"I am in shanghai";
NSArray * strArr = [str componentsSeparatedByString:@" "];
    
//数组-->字符串  (添加字符拼接)
NSString * str1 = [strArr componentsJoinedByString:@"-"];
```

## 本篇到此为止，希望这对你有帮助，如果有错误或是有需要补充的地方，望告知。

