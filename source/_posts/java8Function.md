---
title: Map/Reduce 及Java8的Function和Stream
date: 2019-11-13 14:04:56
tags:
- Show All
- idea
header-img: "Demo.png"
---

#### 概览
主要介绍Map/Reduce的提出和意义 及其在Java中的运用。结合Java8的Lambda表达式、函数式编程、Stream流式操作、Fork/Join框架来阐述其操作。

#### Map/Reduce的概念
Map/Reduce起源于Google关于大数据的三篇论文之一的“MapReduce: Simplified Data Processing on Large Clusters”，用于海量数据分布在不同机器上的情况下的并行运算的一种编程模型。
Map意为映射，Reduce意为归约。

数学上讲，Map/Reduce用于集合操作，假设有一个集合list=[X1,X2,X3,,,Xn]。
Map即为对list集合中的每一个元素进行Y=f(X)的操作后变为一个新的集合list2=[Y1,Y2,Y3,,,Yn]。
Reduce意为执行g(g(g(g(X1),X2),X3)...Xn)这样一个函数后的的结果值。

大白话讲，对集合中的每一个元素执行同一个操作，就是Map；对集合中的所有元素进行归一操作，就是Reduce。

###### tips：这里的Map/Reduce概念的阐述有点狭隘和不准确，目的是以Java代码的角度理解为主。


#### Java中的Lambda表达式
Java8中开始引入Lambda表达式语法，方便进行函数式编程开发。
eg:
```java
public interface People {
    String getName();
}
public interface ManyPeople {
    void view(People people);
}
```
```java
People person1 = () -> "a"; // 1
People person2 = () -> { return "b"; }; // 2

ManyPeople manyPeople1 = people -> people.getName(); // 3
ManyPeople manyPeople2 = (People people) -> people.getName(); // 4

BinaryOperator<Integer> maxBy = (Integer a, Integer b) -> a > b ? a : b; // 5
BinaryOperator<Integer> minBy = (a, b) -> a < b ? a : b; // 6

UnaryOperator<People> identify = t -> t; // 7
```
小结：
1，Lambda表达式可以看做是匿名内部类的语法糖；
2，目标接口只能有一个未实现的方法；（默认方法不影响）
3，-> 后的返回值可以是简单的值，也可以是复杂的代码块； eg: 1,2
4，接口无参的话，-> 前需要()； eg: 1,2
5，接口有参的话，-> 前的()内标出方法参数； eg: 4,6
6，接口只有一个参数的话，-> 前的()可以省略； eg: 3,7
7，-> 前的()内的接口的参数的类型可以不写； eg： 6
8，具有泛型类型推断的功能； eg：5，6

#### 方法的引用
Java8开始，可以以函数式的方式引用方法。
```java
// Fucntion接口的方法名叫做apply，但是依然可以引用。（极大的方便）
Function<String, Integer> converter = Integer::valueOf; // 类的方法
还可以是：
Integer::new; // 构造方法
list::add; // 某个对象的方法
Integer::sum; // 静态方法
```
方法的引用天然支持类型推断。

#### Java中的函数式
主要是java.util.function包里的接口 和 java.util.stream包里的接口及实现类。
java.util.function包下的主要接口：
```java
public interface Consumer<T> { // Stream.forEach()
    void accept(T t);
}
public interface Function<T, R> { // Stream.map()
    R apply(T t);
}
public interface Predicate<T> { // Stream.filter()
    boolean test(T t);
}
public interface Supplier<T> { // Stream.collect()
    T get();
}
```
该报下Bi开头表示两元素类型的函数接口。比如
```java
public interface BiConsumer<T, U>{
    void accept(T t, U u);
}
```
该报下其他如DoubleXxx、IntXxx、ObjXxx、LongXxx是特定的类型的函数接口。

java.util.stream包下的主要接口：
Stream接口的主要方法：
```java
// 过滤
Stream<T> filter(Predicate<? super T> predicate);
// 映射
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
// 平铺映射
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
// 根据equals方法去重
Stream<T> distinct();
// 排序
Stream<T> sorted(Comparator<? super T> comparator);
// 过程中遍历
Stream<T> peek(Consumer<? super T> action);
// 从后面去掉一部分元素
Stream<T> limit(long maxSize);
// 从前面去掉一部分元素
Stream<T> skip(long n);
// 遍历
void forEach(Consumer<? super T> action);
// 转数组
<A> A[] toArray(IntFunction<A[]> generator);
// 最小值
Optional<T> min(Comparator<? super T> comparator);
// 最大值
Optional<T> max(Comparator<? super T> comparator);
// 元素个数
long count();
// 只要有一个匹配就返回true
boolean anyMatch(Predicate<? super T> predicate);
// 所有都匹配则返回true
boolean allMatch(Predicate<? super T> predicate);
// 没有一个匹配则返回true
boolean noneMatch(Predicate<? super T> predicate);
// 返回第一个元素
Optional<T> findFirst();
// 返回任意一个元素，并行是可能不同
Optional<T> findAny();
// Reduce操作，返回值与元素值类型相同，提供初始值和reduce函数
T reduce(T identity, BinaryOperator<T> accumulator);
// Reduce操作，返回值与元素值类型相同，无初始值，提供reduce函数
Optional<T> reduce(BinaryOperator<T> accumulator);
// Reduce操作，返回值与元素值类型不同，提供初始值、reduce函数、并行时的归一函数
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator,BinaryOperator<U> combiner);
// Recuce操作，利用Java封装的形式，提供初始值函数、reduce函数、并行时的归一函数
<R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner);
// Recuce操作，利用Java封装的常用的Collector接口，建议从Collectors工具类返回。
<R, A> R collect(Collector<? super T, A, R> collector);
```

#### 惰性执行
对stream的操作分为两种：中间操作(intermediate) 和 终级操作(terminal)。
中间操作在stream中不会立即执行，只有等到用户真正需要结果的时候才执行，调用中间操作只会生成一个标记该操作的新stream对象而已，称为惰性执行(lazy)。
在对于一个 Stream 进行多次中间操作 (Intermediate 操作)，每次都对 Stream 的每个元素进行转换，而且是执行多次，这样时间复杂度就是 N（转换次数）个 for 循环里把所有操作都做掉的总和吗？其实不是这样的，转换操作都是 lazy 的，多个转换操作只会在 Terminal 操作的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream 里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在 Terminal 操作的时候循环 Stream 对应的集合，然后对每个元素执行所有的函数。
```java
中间操作：
concat() distinct() filter() flatMap() limit() map() peek()
skip() sorted() parallel() sequential() unordered()
终极操作：
allMatch() anyMatch() collect() count() findAny() findFirst()
forEach() forEachOrdered() max() min() noneMatch() reduce() toArray()
```

#### 用 Collectors 来进行 reduce 操作
对stram对象执行reduce操作，调用
```java
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator,BinaryOperator<U> combiner)
```
这个方法，需要传递3个参数。第一个参数是初始值，第二参数是reduce函数，第三个参数是并行运行是需要的合并函数。而对于大部分转List、Set、Map等集合的操作，都可以被封装。于是collect()就诞生了。
```java
<R, A> R collect(Collector<? super T, A, R> collector)
```
Collectors工具类中定义了很多常用的工具方法，返回特定的collector对象，用于Stream.collect()方法。常见的有：
```java
// 转List
Collector<T, ?, List<T>> toList();
// 转Set
Collector<T, ?, Set<T>> toSet();
// 字符串join
Collector<CharSequence, ?, String> joining();
// 字符串join
Collector<CharSequence, ?, String> joining(CharSequence delimiter);
// 字符串join
Collector<CharSequence, ?, String> joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix);
// 映射收集函数，经常与groupingBy()共用
Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper, Collector<? super U, A, R> downstream);
// 计数
<T> Collector<T, ?, Long> counting();
// 分组，转Map，value是集合
<T, K> Collector<T, ?, Map<K, List<T>>> groupingBy(Function<? super T, ? extends K> classifier);
// 分组，转Map，Value自定义的收集函数
<T, K, A, D> Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier, Collector<? super T, A, D> downstream);
// 转Map
<T, K, U>Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper);
// 转Map，二分操作
<T> Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate);
```

#### 并行执行
利用Java7引入的Fork/Join框架，可以利用多核cpu并行执行运算。
list.parallelStream()，即返回一个可以并行执行的stream对象，其他操作不用动。
Collectors.groupingByConcurrent()也可以并行形式的收集，提高运算效率。
对于中间操作(Intermediate)，如果是无状态的API，是可以完全并行的。如果是有状态的API，即使是并行API实际也不能完全并行（or根本就不能并行？）。
有状态的中间操作包括：distinct() sorted() limit() skip()

#### 流水线操作pipeline
集合中的元素执行完所有的函数后，下一个元素再去执行所有的函数。
从数学上讲，Map就会变为 x -> h(g(f(x)))... 这样的一个复合函数操作，一个元素执行完复合函数，下一个元素才执行。
本质上是通过java.util.stream.Sink这个接口的实现类来操作的。

#### 集合接口中新增的默认方法
- Iterable接口中的新增
```java
default void forEach(Consumer<? super T> action); // 等价于Stream.forEach(action)
```
- Collection接口中的新增
```java
default Stream<E> stream(); // 返回stream对象
default Stream<E> parallelStream(); // 返回可并行的stream对象
default boolean removeIf(Predicate<? super E> filter); // Stream.filter(predicate)
```
- List中的新增
```java
default void replaceAll(UnaryOperator<E> operator); // 等价于Stream.map(mapper)
default void sort(Comparator<? super E> c); // 等价于Stream.sorted(comparator)
```

- Map接口中的新增
```java
default V getOrDefault(Object key, V defaultValue); // 如果key不存在则返回给定的默认value
default void forEach(BiConsumer action); // 等价于Stream.forEach(action)
default void replaceAll(BiFunction function); // 等价于Stream.map(mapper)
default V putIfAbsent(K key, V value); // 如果key对应的value不存在或为null则存储k-v键值对。
default boolean remove(Object key, Object value); // 存在key且值为value，则移除
default V replace(K key, V value); // 如果key存在则替换，有点Stream.map(mapper)的意思
default V merge(K key, V value,BiFunction remappingFunction); // 并行Stream归一时的操作
```

#### 函数式注解
使用@FunctionalInterface注解在接口上，则该接口只能有一个未实现的接口。否则编译报错。（该注解仅作用于编译器）

#### 备注
- Lambda表达式可以看做是匿名内部类的语法糖，但与匿名内部类不同；匿名内部类的this指向内部类对象，如果要指向外部类对象的话，得是外部类类名.this。
- 如果是::进行方法引用的话，如果接口满足@FunctionalInterface注解，那么方法名可以不同。即使该接口没有被@FunctionalInterface注解。
- 被@FunctionalInterface注解的接口只能有一个待实现的接口，特殊情况下可以不止一个，多出来的其他接口可以是Object对象本身实现的。比如boolean queals()方法。参考Comparator接口。


#### 问答
- 什么时候使用Map/Reduce这套东西呢？针对集合的操作就可以考虑。
- 是否全是Stream而不再用原来那套？以可读性、优雅性考虑，哪个更优用哪个。

#### 附录1.demo
```java
public class TestFunction {

    public static void main(String[] args) {

        //testLazy();
        
        //testPipeLine();
        
        //testParallel();
        
        //testReduce();
        
        testCollectors();
        
        //testFlatMap();
        
    }
    
    
    
    
    public static void testLambda() {
        // 匿名内部类的写法
        Person person1 = new Person() {
            @Override
            public String getName() {
                return "a";
            }
        };
        // lambda的写法，接口无参
        Person person2 = () -> "a";
        Person person3 = () -> { return "b"; };

        // 接口一个参数
        Function<String, String> lowerCase = (String s) -> s.toLowerCase();
        // 具备类型推断的功能
        Function<String, String> upperCase = s -> s.toUpperCase();
        
        // 接口多个参数
        BinaryOperator<Integer> maxBy = (Integer a, Integer b) -> a > b ? a : b;
        
        // 自身
        Function<Object, Object> identity = Function.identity(); // t -> t
    }
    
    public static void testMethodReference() {
        Function<String, Integer> converter = Integer::valueOf; // 类的普通方法
        Supplier<String> generator = String::new; // 类的静态方法
        Developer devloper = new Developer();
        Function<String, String> echoAddress = devloper::echoAddress; // 对象的方法
        Function<Integer, Long> squareFunction = Developer::getSquare; // 类的静态方法
    }
    
    public static void testLazy() {
        List<String> list = Arrays.asList("You", "are", "so", "clever", "!");
        
        list.stream().map(s -> s + "1").peek(System.out::println);
        
        list.stream().map(s -> s + "2").forEach(System.out::println);
        
    }
    
    public static void testPipeLine() {
        List<String> list = Arrays.asList("You", "are", "so", "clever", "!");
        list.stream().map(s -> s + "1").peek(System.out::println)
                .map(s -> s + "2").peek(System.out::println)
                .map(s -> s + "3").peek(System.out::println)
                .map(s -> s + "4").forEach(System.out::println);
    }
    
    public static void testParallel() {
        int numberSize = 20000000;
        
        List<Long> numberList = LongStream.range(0, numberSize).mapToObj(Long::valueOf).collect(Collectors.toList());
        
        
        System.out.println("start...");
        long nanoTime1 = System.nanoTime();
        long result = numberList.stream().map(num -> num*11).map(num -> num*13).map(num -> num*17).mapToLong(Long::longValue).sum();
        
        long nanoTime2 = System.nanoTime();
        long result2 = numberList.parallelStream().map(num -> num*11).map(num -> num*13).map(num -> num*17).mapToLong(Long::longValue).sum();
        long nanoTime3 = System.nanoTime();
        
        
        
        System.out.println(nanoTime2 - nanoTime1);
        System.out.println(nanoTime3 - nanoTime2);
    }
    
    public static void testReduce() {
        
        Stream<Long> longStream = LongStream.range(0, 100).mapToObj(Long::valueOf);
        
        
        List<Long> longList = longStream.collect(Collectors.toList());
        
        // BiFunction<Long,String,String> a  = (x,y) -> String.valueOf(x) + y;
        // BinaryOperator<String> b = (x,y) -> x + y;
        
        String reduceString = longList.stream().reduce("", (x,y) -> String.valueOf(x) + "," + y, (x,y) -> x + y);
        System.out.println(reduceString);
        
        // 使用过的stream再次使用会报错
        // java.lang.IllegalStateException: stream has already been operated upon or closed
        // long count = longStream.count();
        // System.out.println(count);
        
        // parallelStream(),可以并行，但依然保持有序
        String reduceString2 = longList.parallelStream().reduce("", (x,y) -> String.valueOf(x) + "," + y, (x,y) -> x + y);
        System.out.println(reduceString2);
        System.out.println(reduceString.equals(reduceString2));
        
        // 去掉初始的逗号
        System.out.println(IntStream.range(0, 100).mapToObj(String::valueOf).reduce((x,y) -> x + "," + y).get());
        System.out.println(IntStream.range(0, 100).mapToObj(String::valueOf).collect(Collectors.joining(",")));
        
        // 转Map
        BiFunction<Map<String, String>,String,Map<String, String>> a  = (x,y) ->  {x.put(y, y); return x;};
        BinaryOperator<Map<String, String>> b = (x,y) -> {x.putAll(y); return x;};
        System.out.println(IntStream.range(0, 100).mapToObj(String::valueOf).reduce(new HashMap<>(), a, b));
        
    }
    
    public static void testCollectors() {
        
        int maxDeptId = 5;
        int maxUserId = 100;
        
        // 构造UserList，dpId随机，uId从1到100
        List<User> userList = IntStream.rangeClosed(1, maxUserId)
                .mapToObj(uId -> new User(uId, generateRandom(maxDeptId))).collect(Collectors.toList());
        
        // 按部门Id归类，Map<Integer, List<User>>
        Map<Integer, List<User>> dpId2UserList = userList.stream().collect(Collectors.groupingBy(User::getDpId));
        System.out.println(dpId2UserList);
        
        // 按部门Id归类，Map<Integer, List<Integer>>
        Map<Integer, List<Integer>> dpId2UserIdList = userList.stream()
                .collect(Collectors.groupingBy(User::getDpId, Collectors.mapping(User::getuId, Collectors.toList())));
        System.out.println(dpId2UserIdList);
        
        // 计算每个部门有多少人，Map<Integer, Long>
        Map<Integer, Long> deptId2Count = userList.stream().collect(Collectors.groupingBy(User::getDpId, Collectors.counting()));
        System.out.println(deptId2Count);
        
        
        // List<User> 转 Map<Integer, String>, eg: {stuId,stuName}这样子的map
        Map<Integer, Integer> uId2DeptId = userList.stream().collect(Collectors.toMap(User::getuId, User::getDpId));
        System.out.println(uId2DeptId);
        
        
        // List<User>排序，按deptId从小到大，uId从大到小排序
        
        Comparator<User> comparator = (User user1, User user2) -> {
            if (user1.getDpId() < user2.getDpId()) {
                return -1;
            } else if (user1.getDpId() > user2.getDpId()) {
                return 1;
            } else {
                if(user1.getuId() < user2.getuId()) {
                    return 1;
                }else if (user1.getuId() > user2.getuId()) {
                    return -1;
                }else {
                    return 0;
                }
            }
        };
        List<User> userListSorted = userList.stream().sorted(comparator).collect(Collectors.toList());
        System.out.println(userListSorted);
        
    }
    
    
    public static void testFlatMap() {
        
        List<Integer> classIdList1 = IntStream.range(138001, 138015).mapToObj(Integer::valueOf).collect(Collectors.toList());
        Grade grade1 = new Grade(1380, classIdList1);
        
        List<Integer> classIdList2 = IntStream.range(138010, 138020).mapToObj(Integer::valueOf).collect(Collectors.toList());
        Grade grade2 = new Grade(1382, classIdList2);
        
        Arrays.asList(grade1, grade2).stream().flatMap(grade -> grade.getClassIdList().stream()).forEach(System.out::println);
        
    }
    
    // 生成一个随机数，下限是1，上限是upper
    private static int generateRandom(int upper) {
        return (int)Math.ceil(Math.random() * upper);
    }
    
}
```