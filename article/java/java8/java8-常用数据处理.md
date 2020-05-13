# java8-常用数据处理

## 一、list转map
```
/**
 * List -> Map
 * 需要注意的是：
 * toMap 如果集合对象有重复的key，会报错Duplicate key ....
 *  apple1,apple12的id都为1。
 *  可以用 (k1,k2)->k1 来设置，如果有重复的key,则保留key1,舍弃key2
 */
Map<Integer, Apple> appleMap = appleList.stream().collect(Collectors.toMap(Apple::getId, a -> a,(k1,k2)->k1));
```

## 二、分组
```
//List 以ID分组 Map<Integer,List<Apple>>
Map<Integer, List<Apple>> groupBy = appleList.stream().collect(Collectors.groupingBy(Apple::getId));
```

## 三、求和
```
BigDecimal totalMoney = appleList.stream().map(Apple::getMoney).reduce(BigDecimal.ZERO, BigDecimal::add);
```

## 四、过滤
```
//过滤出符合条件的数据
List<Apple> filterList = appleList.stream().filter(a -> a.getName().equals("香蕉")).collect(Collectors.toList());
 
```

## 五、去重
```// 根据id去重
     List<Person> unique = appleList.stream().collect(
                collectingAndThen(
                        toCollection(() -> new TreeSet<>(comparingLong(Apple::getId))), ArrayList::new)
        );
```
## 六、最大值最小值
```
Optional<Dish> maxDish = Dish.menu.stream().
      collect(Collectors.maxBy(Comparator.comparing(Dish::getCalories)));
maxDish.ifPresent(System.out::println);
 
Optional<Dish> minDish = Dish.menu.stream().
      collect(Collectors.minBy(Comparator.comparing(Dish::getCalories)));
minDish.ifPresent(System.out::println);
```

## 常用函数
|工厂方法 |	返回类型| 	作用|
|-|-|-|
|toList 	| List<T> 	|把流中所有项目收集到一个 List|
|toSet 	    |Set<T> 	把流中所有项目收集到一个 Set，删除重复项|
|toCollection 	|Collection<T>| 	把流中所有项目收集到给定的供应源创建的集合menuStream.collect(toCollection(), ArrayList::new)|
|counting 	|Long |	计算流中元素的个数|
|sumInt 	 |Integer |	对流中项目的一个整数属性求和|
|averagingInt 	|Double |	计算流中项目 Integer 属性的平均值|
|summarizingInt 	|IntSummaryStatistics |	收集关于流中项目 Integer 属性的统计值，例如最大、最小、 总和与平均值|
|joining 	|String |	连接对流中每个项目调用 toString 方法所生成的字符串collect(joining(", "))|
|maxBy 	|Optional<T> |	一个包裹了流中按照给定比较器选出的最大元素的 Optional， 或如果流为空则为 Optional.empty()|
|minBy 	|Optional<T> 	|一个包裹了流中按照给定比较器选出的最小元素的 Optional， 或如果流为空则为 Optional.empty()|
|reducing 	|归约操作产生的类型 |	从一个作为累加器的初始值开始，利用 BinaryOperator 与流 中的元素逐个结合，从而将流归约为单个值累加int totalCalories = menuStream.collect(reducing(0, Dish::getCalories, Integer::sum));|
|collectingAndThen 	|转换函数返回的类型 |	包裹另一个收集器，对其结果应用转换函数int howManyDishes = menuStream.collect(collectingAndThen(toList(), List::size))|
|groupingBy 	|Map<K, List<T>> |	根据项目的一个属性的值对流中的项目作问组，并将属性值作 为结果 Map 的键|
|partitioningBy 	|Map<Boolean,List<T>>| 	根据对流中每个项目应用谓词的结果来对项目进行分区|
