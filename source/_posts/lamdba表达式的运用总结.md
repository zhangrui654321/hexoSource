# lambda表达式总结（集合操作）

## 1.集合的排序

```java
Comparator<Hero> comparator=new Comparator<Hero>() {
    @Override
    public int compare(Hero o1, Hero o2) {
        return o1.getHp()-o2.getHp();
    }
};
```

先写一个comparator,根据age排序

然后调用

```java
List<Hero> sortHeroList=heroList.stream().sorted(comparator).collect(Collectors.toList());
```

## 2.最大最小值的属性以及具体元素

```java
Hero maxAgeHero=heroList.stream().max(comparator).get();
```

## 3.去重

### 一般去重

```java
List<Hero> heroeDistinct=heroList.stream().distinct().collect(Collectors.toList());
```

## 4.求和

### 一般求和

```java
int sum=heroList.stream().mapToInt(Hero::getHp).sum();
```

大数类求和

```java
BigDecimal totalQuantity = bigDecimalList.stream().map(Hero::getHp).reduce(BigDecimal.ZERO,BigDecimal::add);
```

## 5.分组

根据某个属性分组并且转成Map<String,List<Hero>>

```java
Map<String,List<Hero>> groupMap=heroList.stream().collect(Collectors.groupingBy(Hero::getSex));
```

## 6.list转为map

```java
//list转为map,如果重复，则取k1的值
Map<Integer,Hero> heroMap=heroList.stream().collect(Collectors.toMap(Hero::getHp,a->a,(k1,k2)->k1));
```

## 7. 新的list

```java
//获取某个值转为新的list
List<Integer> heroAgeList=heroList.stream().map(a->a.getHp()).collect(Collectors.toList());
```

## 8.过滤

```java
List<Hero> heroList1=heroList.stream().filter(hero -> hero.getHp()<300&&hero.getHp()>100).collect(Collectors.toList());
```

## 总结

表达式主要分为中间过程和结束操作

### 零个或多个中间操作： 每个中间操作会返回一个流，如filter,中间操作时懒操作，不会真正遍历

​     中间操作有很多种，主要分为两种：

​     1.**对元素进行筛选**：filter（匹配），distinct（去除重复）,sorted（自然排序）

​                     sorted(Comparator)（指定排序），limit（保留），skip（忽略）

​     2.**转为其他形式的流**：mapToDouble（转为double的流）, map（转换为任意类型的流）

 

### 结束操作： 例如forEach，会返回非流结果，例如基本类型的值（int,float,double）、对象或者集合，或者在终端操作为forEach的情况下没有返回值。

​     结束操作时才真正进行遍历行为，前面的中间操作也在这个时候真正的执行。

​     常见的结束操作如下：

​      forEach()（遍历每个元素），toArray()（转换为数组），min(Comparator)（取最小的元素）

​      max(Comparator)（取最大的元素），count()（总数），findFirst()（第一个元素）.get(). 

以及Collectors.toList()转为list