---
title: 'Java集合'
tags: ["疯狂Java讲义第三版阅读笔记","javaSE"]
date: 2019-06-07
draft: false
---

**介绍:** 这是疯狂`Java`讲义第三版阅读笔记.

# 1.`Collection`和`Iterator`接口

## 1.1`Iterator`

`Iterator`接口也是`Java`集合框架的成员,但它与`Collection`系列,`Map`系列的集合不一样:`Collection`系列集合,`Map`系列集合主要用于装其他对象,而`Iterator`则主要用于遍历(即迭代访问)`Collection`集合中的元素,`Iterator`对象也被称为迭代器.

**示例代码:**

```java
	@Test
    public void test5() {
        List<String> list = new ArrayList<>(Arrays.asList("Java", "Python", "Go", "Swift", "JavaScript"));
        System.out.println("list未迭代之前->" + list);
        //第一次迭代
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String next = iterator.next();
            //修改next值,不会影响集合元素本身
            next = "Test";
        }
        System.out.println("第一次迭代->" + list);

        //第二次迭代
        //迭代器已进行过一次遍历,需要对迭代器重新赋值
        iterator = list.iterator();
        while (iterator.hasNext()) {
            String next = iterator.next();
            //通过iterator删除集合中的Java元素
            if (next.equals("Java")) {
                iterator.remove();
            }
        }
        System.out.println("第二次迭代->" + list);

        //第三次迭代
        //迭代器已进行过一次遍历,需要对迭代器重新赋值
        iterator = list.iterator();
        while (iterator.hasNext()) {
            String next = iterator.next();
            //在迭代过程中,对集合本身进行修改,会导致ConcurrentModificationException异常
            if (next.equals("Python")) {
                list.remove(next);
            }
        }
        System.out.println("第三次迭代->" + list);
    }
```

**控制台输出:**

```
list未迭代之前->[Java, Python, Go, Swift, JavaScript]
第一次迭代->[Java, Python, Go, Swift, JavaScript]
第二次迭代->[Python, Go, Swift, JavaScript]

java.util.ConcurrentModificationException
```

在第一次迭代中,对迭代变量`next`赋值,但当再次输出`list`集合时,回发现集合里的元素没有任何改变.这就可以得出一个结论:当使用`Iterator`对集合元素进行迭代时,`Iterator`并不是把集合元素本身传给了迭代变量,而是把集合元素的值传给了迭代变量,所以修改迭代变量的值对集合元素本身没有任何影响.

在第二次和第三次迭代中说明了在使用`Iterator`迭代器访问`Collection`集合元素时,`Collection`集合里的元素不能被改变,只有通过`Iterator`的`remove()`方法删除上一次`next()`方法返回的集合元素才可以:否则将会引发`java.util.ConcurrentModificationExcepiton`异常.

同样,当使用`foreach`循环迭代访问集合元素时,该集合也不能被改变.

## 1.2`Predicate`

使用`Java`8新增的`Predicate`关键字来操作集合.(也可直接通过`Stream`来完成其操作)

```java
	@Test
    public void test7() {
        List<String> books = new ArrayList<>(Arrays.asList("Java编程思想", "并发编程", "深入理解Java虚拟机", "Head First Java设计模式", "Spring实战", "图解HTTP"));
        //获取books集合中包含Java字符串的元素个数
        System.out.println(calTotal(books, str -> ((String) str).contains("Java")));
        //获取books集合中元素长度大于8的元素个数
        System.out.println(calTotal(books, str -> ((String) str).length() > 8));

        System.out.println("原List->" + books);
        //删除大于8的元素
        books.removeIf(str -> str.length() > 8);
        System.out.println("操作过后List->" + books);
    }

    private int calTotal(Collection list, java.util.function.Predicate predicate) {
        int total = 0;
        for (Object object : list) {
            if (predicate.test(object)) {
                total++;
            }
        }
        return total;
    }
```

**控制台输出:**

```
3
原List->[Java编程思想, 并发编程, 深入理解Java虚拟机, Head First Java设计模式, Spring实战, 图解HTTP]
操作过后List->[Java编程思想, 并发编程, Spring实战, 图解HTTP]
```

# 2.`Set`集合

`Set`集合无序不可重复.

## 2.1`HashSet`

`HashSet`是Set接口的典型实现,大多数使用`Set`集合时就是使用这个实现类.`HashSet`按`Hash`算法来存储集合中的元素,因此具有很好的存取和查找性能.

`HashSet`具有以下特点:

* 不能保证元素的排列顺序,顺序可能与添加顺序不同;
* `HashSet`不是线程安全的;
* 集合元素值可以是`null`;

**`hashCode()`与`HashSet`之间的关系**

`hash`(也被翻译为哈希,散列)算法的功能是,它能保证快速查找被检索的对象,`hash`算法的价值在于速度.当需要查询集合中某个元素时,`hash`算法可以直接根据该元素的`hashCode`值计算出该元素的存储位置,从而快速定位该元素.



**`HashSet`集合判断两个元素相等的标准是两个对象通过`equal()`方法比较是否相等,并且两个对象的`hashCode()`方法返回值也相等.**

如果两个对象通过`equals()`方法比较返回`true`,但这两个对象的`hashCode()`方法返回不同的`hashCode`值时,这将导致`HashSet`会把这两个对象保存在`Hash`表的不同位置,从而使两个对象都可以添加成功,这就与`Set`集合的规则冲突了.

`HashSet`中每个能存储元素的"槽位"(`slot`)通常称为"桶"(`bucket`),如果有多个元素的`hashCode`值相同,但它们通过`equals()`方法比较返回`false`,就需要在一个"桶"里放多个元素,这样会导致性能下降.

**测试代码:**

```java
	class A {
        @Override
        public int hashCode() {
            return 1;
        }
    }

    class B {
        @Override
        public boolean equals(Object obj) {
            return true;
        }
    }

    class C {
        @Override
        public int hashCode() {
            return 10;
        }

        @Override
        public boolean equals(Object obj) {
            return true;
        }
    }


    @Test
    public void test38() {
        Set set = new HashSet();
        set.add(new A());
        set.add(new A());
        set.add(new B());
        set.add(new B());
        set.add(new C());
        set.add(new C());
        System.out.println(set);
    }
```

**控制台输出:**

```
[PersonalTest$A@1, PersonalTest$A@1, PersonalTest$B@5e8c92f4, PersonalTest$C@a, PersonalTest$B@61e4705b]
```

`Set`集合中只有`C`对象是唯一的,因为它的`hashCode()`和`equals(Object obj)`返回均相同.

**当向`HashSet`中添加可变对象时,必须十分小心.如果修改`HashSet`集合中的对象,有可能导致该对象与集合中的其他对象相等,从而导致`HashSet`无法准确访问该对象.**

**示例代码:**

```java
	class R {
        private int count;

        public void setCount(int count) {
            this.count = count;
        }

        public R(int count) {
            this.count = count;
        }

        @Override
        public boolean equals(Object object) {
            if (this == object) return true;
            if (object == null || getClass() != object.getClass()) return false;

            R r = (R) object;

            return count == r.count;
        }

        @Override
        public int hashCode() {
            return count;
        }

        @Override
        public String toString() {
            return "R{" +
                    "count=" + count +
                    '}';
        }
    }

    @Test
    public void test8() {
        Set<R> set = new HashSet<>();
        set.add(new R(1));
        set.add(new R(2));
        set.add(new R(3));
        set.add(new R(4));
        System.out.println(set);

        //更改其中一个元素为R(2),此处中改变的是R(1)
        Iterator<R> iterator = set.iterator();
        R r1 = iterator.next();
        r1.setCount(2);

        //此时set拥有了重复的元素
        System.out.println(set);

        //删除R(2),根据R(2)的hashCode值,删除R(2),而不是替换R(1)的R(2)
        set.remove(new R(2));

        //此时剩下的是 替换R(1)的R(2),R(3),R(4)
        System.out.println(set);

        //根据R(1)的hashCode值,找到当前set中的R(2),但由于equals()返回false(count值不同),所以输出false
        System.out.println(set.contains(new R(1)));
        //R(2)的hashCode值在之前的remove(new R(2))中已经被移除,所以输出false
        System.out.println(set.contains(new R(2)));
    }
```

**控制台输出:**

```
[R{count=1}, R{count=2}, R{count=3}, R{count=4}]
[R{count=2}, R{count=2}, R{count=3}, R{count=4}]
[R{count=2}, R{count=3}, R{count=4}]
false
false
```

## 2.2`LinkedHashSet`

`LinkedHashSet`也是根据元素的`hashCode`值来决定元素的存储位置,但它同时使用链表维护元素的次序.因此它是有序,不可重复的.但是性能略低于`HashSet`.但是它在迭代访问`Set`里的全部元素时将有很好的性能,因为它以链表来维护内部顺序.
**示例代码:**

```java
	@Test
    public void test39() {
        LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>();
        linkedHashSet.add("Java");
        linkedHashSet.add("Python");
        linkedHashSet.add("Go");
        linkedHashSet.add("Swift");
        System.out.println(linkedHashSet);
        linkedHashSet.remove("Java");
        linkedHashSet.add("Java");
        //可以看出Java 字符串 从首跑到了尾
        System.out.println(linkedHashSet);
    }
```

**控制台输出:**

```
[Java, Python, Go, Swift]
[Python, Go, Swift, Java]
```

## 2.3`TreeSet`

`TreeSet`有序,不可重复.`TreeSet`并不是根据元素的插入顺序进行排序的,而是根据元素实际值的大小来进行排序的.`TreeSet`采用红黑树的数据结构来存储集合元素.

### 2.3.1自然排序

自然排序需要确保`TreeSet`中的对象实现了`Comparable`接口.

当两个对象通过`compare(Object obj)`方法返回0时,这两对象的`equals()`方法返回`true`.此时`TreeSet`不会把第二个元素添加到集合中.

```java
	@Test
    public void test40() {
        TreeSet<Integer> treeSet = new TreeSet<>();
        treeSet.add(5);
        treeSet.add(-3);
        treeSet.add(7);
        treeSet.add(-2);
        System.out.println(treeSet);
        //输出集合中的第一个元素
        System.out.println(treeSet.first());
        //输出集合中的最后一个元素
        System.out.println(treeSet.last());
        //返回小于-2的子集,不包含-2
        System.out.println(treeSet.headSet(-2));
        //返回大于等于5的子集,包含5
        System.out.println(treeSet.tailSet(5));
        //返回大于等于-2,小于5的子集
        System.out.println(treeSet.subSet(-2, 5));
    }
```

**控制台输出:**

```
[-3, -2, 5, 7]
-3
7
[-3]
[5, 7]
[-2]
```

### 2.3.2定制排序

定制排序时不再要求`Set`元素实现`Comparable`接口.

```java
	class T {
        private int count;

        public T(int count) {
            this.count = count;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            T t = (T) o;
            return count == t.count;
        }

        @Override
        public int hashCode() {
            return Objects.hash(count);
        }

        public int getCount() {
            return count;
        }

        @Override
        public String toString() {
            return "T{" +
                    "count=" + count +
                    '}';
        }
    }

    @Test
    public void test48() {
        //使TreeSet按照从大到小的顺序排列.
        TreeSet<T> treeSet = new TreeSet<T>((o1, o2) -> {
            return Integer.compare(o2.getCount(), o1.getCount());
        });
        treeSet.add(new T(3));
        treeSet.add(new T(-1));
        treeSet.add(new T(2));
        treeSet.add(new T(6));
        System.out.println(treeSet);
    }
```

**控制台输出:**

```
[T{count=6}, T{count=3}, T{count=2}, T{count=-1}]
```

## 2.4`EnumSet`

`EnumSet`是一个专为枚举类设计的集合类,`EnumSet`中的所有元素都必须是指定枚举类型的枚举值.`EnumSet`的集合元素也是有序的,它以枚举值在`Enum`类内的定义顺序来决定集合元素的顺序.

`EnumSet`在内部以位向量的形式存储,这种存储非常紧凑,高效.因此`EnumSet`对象占用内存很小,而且运行效率很好.`EnumSet`集合不允许加入`null`元素,否则将会抛出`NullPointerException`异常.

**示例代码:**

```java
	enum Season {
        SPRING, SUMMER, FALL, WINTER
    }

    @Test
    public void test42() {
        //创建一个EnumSet枚举类,集合元素时Season的全部枚举值
        EnumSet<Season> es1 = EnumSet.allOf(Season.class);
        System.out.println(es1);
        for (Season season : es1) {
            System.out.println(season.ordinal());
        }

        //创建一个空的EnumSet,指定其集合元素是Season类的枚举值
        EnumSet<Season> es2 = EnumSet.noneOf(Season.class);
        es2.add(Season.SUMMER);
        es2.add(Season.WINTER);
        System.out.println(es2);

        //以指定枚举值创建EnumSet集合
        EnumSet<Season> es3 = EnumSet.of(Season.SPRING, Season.FALL);
        System.out.println(es3);
    }
```

**控制台输出:**

```
[SPRING, SUMMER, FALL, WINTER]
0
1
2
3
[SUMMER, WINTER]
[SPRING, FALL]
```

## 2.5各`Set`实现类的性能分析

`HashSet`和`TreeSet`是`Set`的典型实现.`HashSet`的性能比`TreeSet`要好,因为`TreeSet`需要额外的红黑树算法来维护集合的次序.所以如果需要`Set`有序时,使用`TreeSet`,否则使用`HashSet`.

`HashSet`还有一个子类`LinkedHashSet`,对于普通的插入,删除操作,`LinkedHashSet`比`HashSet`要略微慢一点,这是由于维护链表所带来的额外开销造成的,但由于有了链表,遍历`LinkedHashSet`会更快.

`EnumSet`是所有`Set`实现类中性能最好的,但它只能保存同一个枚举类的枚举值作为集合元素.

`HashSet`,`LinkedHashSet`,`TreeSet`和`EnumSet`都是线程不安全的,通常可以通过`Collections`工具类的`synchronizedSortedSet`方法来包装`Set`集合.

# 3.`List`集合

`List`集合代表一个元素有序,可重复的集合.`List`集合允许使用重复元素,默认按照元素的添加顺序设置元素的索引,可以通过索引来访问指定位置的集合元素.

## 3.1`List`根据`equals`判断元素是否存在

**示例代码:**

```java
	class L {
        public L() {
        }

        @Override
        public boolean equals(Object obj) {
            return true;
        }
    }

    @Test
    public void test43() {
        List list = new ArrayList();
        list.add("AA");
        list.add("BB");
        list.add("CC");
        System.out.println(list);
        //因为L对象equals一直返回true,所以删除此时处于第一个对象的"AA"
        list.remove(new L());
        System.out.println(list);
        //删除此时处于第一个对象的"BB"
        list.remove(new L());
        System.out.println(list);
    }
```

**控制台输出:**

```
[AA, BB, CC]
[BB, CC]
[CC]
```



## 3.2`sort`和`replaceAll`方法

**示例代码:**

```java
	@Test
    public void test44() {
        List<String> books = new ArrayList<>(Arrays.asList("AA", "BBBBBBB", "C", "DDDD", "EEE", "FFFFF"));
        System.out.println(books);
        books.sort((o1, o2) -> o1.length() - o2.length());
        System.out.println("after sorted->" + books);
        books.replaceAll(ele -> String.valueOf(ele.length()));
        System.out.println("after replaced->" + books);
    }
```

**控制台输出:**

```
[AA, BBBBBBB, C, DDDD, EEE, FFFFF]
after sorted->[C, AA, EEE, DDDD, FFFFF, BBBBBBB]
after replaced->[1, 2, 3, 4, 5, 7]
```

## 3.3`ArrayList`和`Vector`实现类

`ArrayList`和`Vector`作为`List`类的两个典型实现，完全支持前面介绍的`List`接口的全部功能。

`ArrayList`和`Vector`类都是基于数组实现的`List`类，所以`ArrayList`和`Vector`类封装了一个动态的、允许再分配`Object[]`数组。`ArrayList`或`Vector`对象使用`initialCapacity`参数来设置该数组的长度，当向`ArrayList`或`Vector`中添加元素超出该数组的长度时，它们的`initialCapacity`会自动增加。
对于通常的编程场景，程序员无需关心`ArrayList`或`Vector`的`initialCapacity`。但如果向`ArrayList`或`Vector`集合中添加大量元素时，可使用`ensureCapacity（int minCapacity）`方法一次性地增加`initialCapacity`。这可以减少重分配的次数，从而提高性能。
如果开始就知道`ArrayList`或`Vector`集合需要保存多少个元素，则可以在创建它们时就指定`initialCapacity`大小。如果创建空的`ArrayList`或`Vector`集合时不指定`initialCapacity`参数，则`Object[]`数组的长度默认为10。除此之外，`ArrayList`和`Vector`还提供了如下两个方法来重新分配Object[]数组。

* `void ensureCapacity(int minCapacity)`：将`ArrayList`或`Vector`集合的`Object[]`数组长度增加`minCapacity`。
* `void trimToSize()`：调整`ArrayList`或`Vector`集合的`Object[]`数组长度为当前元素的个数。程序可调用该方法来减少`ArrayList`或`Vector`集合对象占用的存储空间。

`ArrayList`和`Vector`在用法上几乎完全相同，但由于`Vector`是一个古老的集合，实际上`Vector`具有很多缺点，通常尽量少用`Vector`实现类。
除此之外，`ArrayList`和`Vector`的显著区别是：`ArrayList`是线程不安全的，当多个线程访问同一个`ArrayList`集合时，如果有超过一个线程修改了`ArrayList`集合，则程序必须手动保证该集合的同步性；但`Vector`集合则是线程安全的，无需程序保证该集合的同步性,所以`Vector`的性能比`ArrayList`的性能要低。实际上，即使需要保证`List`集合线程安全，也同样不推荐使用`Vector`实现类。后面会介绍一个`Collections`工具类，它可以将一个`ArrayList`变成线程安全的。
`Vector`还提供了一个`Stack`子类，它用于模拟“栈”这种数据结构，“栈”通常是指“后进先出”（`LIFO`）的容器。最后“`push`”进栈的元素，将最先被“`pop`”出栈。与`Java`中的其他集合一样，进栈出栈的都是`Object`，因此从栈中取出元素后必须进行类型转换，除非你只是使用`Object`具有的操作。所以`Stack`类里提供了如下几个方法。

* `Object peek()`：返回“栈”的第一个元素，但并不将该元素“`pop`”出栈。
* `Object pop()`：返回“栈”的第一个元素，并将该元素“`pop`”出栈。
* `void push(Object item)`：将一个元素“`push`”进栈，最后一个进“栈”的元素总是位于“栈”顶。

需要指出的是,由于`Stack`继承了`Vector`,因此它也是一个非常古老的`Java`集合类,它同样是线程安全,性能较差的.因此应该尽量少用`Stack`类.如果程序需要使用**栈**这种数据结构,则可以考虑使用后面将要介绍的`ArrayDeque`

## 3.4固定长度的`List`

`Arrays.asList`是一个固定长度的`List`集合,程序只能遍历访问该集合里的元素,不可增加,删除该集合里的元素.

**示例代码:**

```java
    @Test
    public void test46() {
        //固定长度的List,对于该List的任何操作都会触发UnsupportedOperationException异常
        List<String> list1 = Arrays.asList("A", "B", "C", "D", "E", "F");
        //比如添加就会触发异常
        //list1.add("AA");

        //将list1放入ArrayList()中,既初始化了集合,也使集合变得可变
        List<String> list2 = new ArrayList<>(list1);
        list2.add("G");
        System.out.println(list2);
    }
```

# 4.`Queue`集合

`Queue`用于模拟队列这种数据结构,队列通常是指**先进先出**(`FIFO`)的容器.队列的头部保存队列中存放最久的元素,队列的尾部保存在队列中存放时间最短的元素.新元素插入到队列的尾部,访问元素会返回队列头部的元素.

`Queue`接口有一个`PriorityQueue`实现类.除此之外,`Queue`还有一个`Deque`接口,`Deque`代表一个**双端队列**,双端队列可以同时从两端来添加,删除元素,因此`Deque`的实现类既可当成队列使用,也可以当成栈使用.`Java`为`Deque`提供了`ArrayDeque`和`LinkedList`两个实现类.

## 4.1`PriorityQueue`实现类

`PriorityQueue`保存队列元素的顺序不是按照加入队列的顺序,而是按队列元素的大小进行重新排序.因此当调用`peek`方法或`poll`方法去除队列中的元素时,并不是取出最先进入队列的元素,而是取出最小的元素.

**示例代码:**

```java
	@Test
    public void test47() {
        PriorityQueue<Integer> priorityQueue = new PriorityQueue<>();
        priorityQueue.add(3);
        priorityQueue.add(-4);
        priorityQueue.add(10);
        priorityQueue.add(-10);
        //直接输出priorityQueue集合时,可能看到该队列没有完全按照从小到大的顺序输出,这是受到PriorityQueue的toString方法的影响
        System.out.println(priorityQueue);
        //正确的输出priorityQueue对象的方法
        int size = priorityQueue.size();
        for (int i = 0; i < size; i++) {
            System.out.println(priorityQueue.poll());
        }
    }
```

**控制台输出:**

```
[-10, -4, 10, 3]
-10
-4
3
10
```

`PriorityQueue`不允许插入`null`元素.`PriorityQueue`的元素有两种排序:自然排序和定制排序,参照2.3节的`TreeSet`.

## 4.2`Deque`接口与`ArrayDeque`实现类

`Deque`接口是`Queue`接口的子接口,它代表一个双端队列.`Deque`接口提供了一个典型的实现类`ArrayDeque`,从该名称可以看出,它是一个基于数组实现的双端队列,创建`Deque`时同样可指定一个`numElements`参数,该参数用于指定`Object[]`数组的长度:如果不指定`numElements`参数,`Deque`底层数组的长度为16.它与`ArrayList`的实现机制基本相似.

**将`ArrayDeque`当成栈来使用**

```java
	@Test
    public void test49() {
        ArrayDeque stack = new ArrayDeque();
        //将一个元素push进该双端队列的栈顶
        stack.push("Java");
        stack.push("Ios");
        stack.push("Python");
        //输出stack
        System.out.println(stack);
        //获取栈顶元素但不删除
        System.out.println(stack.peek());
        System.out.println(stack);
        //获取并删除栈顶元素
        System.out.println(stack.poll());
        System.out.println(stack);
    }
```

**控制台输出:**

```
[Python, Ios, Java]
Python
[Python, Ios, Java]
Python
[Ios, Java]
```

**将`ArrayDeque`当做队列使用**

```java
	@Test
    public void test50() {
        ArrayDeque queue = new ArrayDeque();
        //将元素放入deque的尾部
        queue.offer("Java");
        queue.offer("Ios");
        queue.offer("Python");
        System.out.println(queue);
        System.out.println(queue.peek());
        System.out.println(queue);
        System.out.println(queue.poll());
        System.out.println(queue);
    }
```

**控制台输出:**

```
[Java, Ios, Python]
Java
[Java, Ios, Python]
Java
[Ios, Python]
```

## 4.3`LinkedList`实现类

`LinkedList`类是`List`接口的实现类,这意味着它是一个`List`集合,可以根据索引来随即访问集合中的元素.除此之外,`LinkedList`还实现了`Deque`接口,可以被当成双端队列来使用,因此既可以被当成**栈**来使用,也可以当成**队列**使用.

**示例代码:**

```java
	@Test
    public void test51() {
        LinkedList books = new LinkedList();
        //放到队列的尾部
        books.offer("Java");
        //放到栈的顶部
        books.push("Ios");
        //放到队列的头部,相当于栈的顶部
        books.offerFirst("Python");
        //作为集合的用法,
        for (int i = 0; i < books.size(); i++) {
            System.out.println(books.get(i));
        }
        System.out.println("================");
        System.out.println(books);
        //访问并不删除栈顶的元素   Python
        System.out.println(books.peekFirst());
        System.out.println(books);
        //访问并不删除队列最后的一个元素   Java
        System.out.println(books.peekLast());
        System.out.println(books);
        //弹出栈顶的元素   Python
        System.out.println(books.pop());
        System.out.println(books);
        //弹出队列尾部的元素 Java
        System.out.println(books.pollLast());
        System.out.println(books);
    }
```

**控制台输出:**

```
Python
Ios
Java
================
[Python, Ios, Java]
Python
[Python, Ios, Java]
Java
[Python, Ios, Java]
Python
[Ios, Java]
Java
[Ios]
```

`LinkedList`与`ArrayList`,`ArrayDeque`的实现机制完全不同,`ArrayList`,`ArrayDeque`内部以数组的形式来保存集合中的元素,因此随机访问集合元素时有较好的性能:而`LinkedList`内部以链表的形式来保存集合中的元素,因此随即访问集合元素时性能较差,但在插入,删除元素时性能比较出色.

## 4.4总结

* 如果需要遍历`List`集合元素,对于`ArrayList`,`Vector`集合,应该使用随机访问方法(`get`)来遍历集合元素,这样性能更好.对于`LinkedList`集合,则应该采用迭代器`Iterator`来遍历集合元素.
* 如果需要经常执行插入,删除操作来改变包含大量数据的`List`集合的大小,可考虑使用`LinkedList`集合.使用`ArrayList`,`Vector`集合可能需要经常重新分配内部数组的大小,效果可能较差.
* `Vector`,`Stack`均线程安全,但不建议使用.若需要保证线程安全,可考虑使用`Collections`将集合包装成线程安全的集合.

# 5.`Map`集合

`Map`用于保存具有映射关系的数据,因此`Map`集合里保存着两组值,一组值用于保存`Map`里的`key`,另外一组用于保存`Map`里的`value`,`key`和`value`都可以是任何引用类型的数据.`Map`的`key`不允许重复,即同一个`Map`对象的任何两个`key`通过`equals()`方法比较总是返回`false`.

`key`和`value`之间存在单向一对一关系,即通过指定的`key`,总能找到唯一的,确定的`value`.从`Map`中取出数据时,只要给出指定的`key`,就可以取出对应的`value`.

## 5.1`Map`中的基本方法

**示例方法:**

```java
	@Test
    public void test9() {
        Map<String, Integer> map = new HashMap<>();
        map.put("Java", 1);
        map.put("Go", 5);
        map.put("Python", 3);
        map.put("Swift", 4);
        //put方法,在key不存在时为新增,key存在时为替换
        map.put("Go", 2);
        System.out.println(map);
        //仅当key存在时,才会进行value替换
        map.replace("Php", 5);
        map.replace("Java", 0);
        System.out.println(map);
        //在key存在的情况下,会进行value与参数的计算,将得到的新值替换原value
        map.merge("Go", 4, (oldVal, param) -> oldVal + param);
        //在key不存在的情况下,会将新增key-value,key为输入的key,value为输入的参数
        map.merge("Php", 7, (oldVal, param) -> oldVal + param);
        System.out.println(map);
        //当key对应的value为null或空时,会新增此key-value
        map.computeIfAbsent("Java", (key) -> key.length());
        map.computeIfAbsent("Java Script", (key) -> key.length());
        System.out.println(map);
        //当key对应的value存在时(不为空,不为null),会将计算后的值替换value
        map.computeIfPresent("Java", (key, value) -> key.length() * (value + 1));
        map.computeIfPresent("C++", (key, value) -> Integer.valueOf(key) * value);
        System.out.println(map);
        //获取map中value的集合
        Collection<Integer> values = map.values();
        System.out.println(values);
    }
```

**控制台输出:**

```
{Java=1, Go=2, Swift=4, Python=3}
{Java=0, Go=2, Swift=4, Python=3}
{Java=0, Go=6, Php=7, Swift=4, Python=3}
{Java=0, Java Script=11, Go=6, Php=7, Swift=4, Python=3}
{Java=4, Java Script=11, Go=6, Php=7, Swift=4, Python=3}
[4, 11, 6, 7, 4, 3]
```

## 5.2`HashMap`和`Hashtable`

`HashMap`和`Hashtable`都是`Map`接口的典型实现类,它们之间的关系完全类似于`ArrayList`和`Vector`的关系.

`Java8`改进了`HashMap`的实现,使用`HashMap`存在`key`冲突时依然具有较好的性能.

`HashMap`和`Hashtable`存在两点典型区别

`Hashtable`是一个线程安全的`Map`实现,但`HashMap`是线程不安全的实现,所以`HashMap`比`Hashtable`的性能高一点;但如果有多个线程访问同一个`Map`对象时,不建议使用`Hashtable`,可通过`Collections`工具类把`HashMap`变成线程安全的.

`Hashtable`不允许使用`null`作为`key`和`value`,如果试图把`null`值放进`Hashtable`中,将会引发`NullPointerException`异常,但`HashMap`可以使用`null`作为`key`或`value`.

**示例代码:**

```java
	@Test
    public void test10() {
        //HashMap允许key-value值为null
        HashMap map = new HashMap();
        map.put(null, null);
        map.put("a", null);
        map.put(null, null);
        System.out.println(map);
        Hashtable hashtable = new Hashtable();
        //Hashtable不允许key,value为null,否则触发NullPointerException
        //hashtable.put(null, "a");
        //hashtable.put("a", null);
        System.out.println(hashtable);
    }
```

**控制台输出:**

```
{null=null, a=null}
{}
```

当使用自定义类作为`HashMap`,`Hashtable`的`key`时,如果重写该类的`equals(Object o)`和`hashCode()`方法,则应该保证两个方法的判断标准一致.它们对`key`的要求与`HashSet`对集合元素的要求完全相同.

与`HashSet`类似的是,尽量不要使用可变对象作为`HashMap`,`Hashtable`的`key`,如果确实需要使用可变对象作为`HashMap`,`Hashtable`的`key`,则尽量不要在程序中修改作为`key`的可变对象.

## 5.3`LinkedHashMap`实现类

`HashSet`有一个`LinkedHashSet`子类,`HashMap`也有一个`LinkedHashMap`子类.`LinkedHashMap`也使用双向链表来维护`key-value`对的次序(其实只需要考虑`key`的次序),该链表负责维护`Map`的迭代顺序,所以在迭代访问`Map`里的全部元素时有更好的性能,迭代顺序与`key-value`对的插入顺序保持一致.由于需要维护顺序,因此在整体性能上要低于`HashMap`.

**示例代码:**

```java
	@Test
    public void test11() {
        //按照元素的插入顺序进行保存
        LinkedHashMap<String, String> linkedHashMap = new LinkedHashMap<>();
        linkedHashMap.put("Java", "First");
        linkedHashMap.put("Python", "Second");
        linkedHashMap.put("Go", "Third");
        //使用Java8语法对元素进行遍历
        linkedHashMap.forEach((key, value) -> System.out.println(key + "-->" + value));
    }
```

**控制台输出:**

```
Java-->First
Python-->Second
Go-->Third
```

## 5.4`SortedMap`接口和`TreeMap`实现类

`Set`接口派生出`Sorted`子接口,`SortedSet`接口有一个`TreeSet`实现类.`Map`接口派生出`SortedMap`子接口,`SortedMap`接口也有一个`TreeMap`实现类.

`TreeMap`就是一个红黑树数据结构,每个`key-value`对即作为红黑树的一个节点.`TreeMap`存储`key-value`对(节点)时,需要根据`key`对节点进行排序.`TreeMap`可以保证所有的`key-value`对处于有序状态.

`TreeMap`的`key`如果是自定义类,那么需要重写`hashCode()`和`equals()`方法.如果是自然排序,还需要实现`Comparable`接口,而且要确保`compareTo()`和`equals()`方法相同.

**示例代码:**

```java
	@Test
    public void test12() {
        //排序Map
        TreeMap<Q, String> treeMap = new TreeMap<>();
        treeMap.put(new Q(5), "Java");
        treeMap.put(new Q(2), "Go");
        treeMap.put(new Q(-4), "Python");
        treeMap.put(new Q(-8), "Java Script");
        System.out.println(treeMap);
        System.out.println("最小的key->" + treeMap.firstKey());
        System.out.println("最大的key->" + treeMap.lastKey());
        System.out.println("比Q(-4)大的最小的key" + treeMap.higherKey(new Q(-4)));
        System.out.println("比Q(-4)小的最大的key" + treeMap.lowerKey(new Q(-4)));
        System.out.println("大于等于Q(-4),小于Q(2)"+treeMap.subMap(new Q(-4),new Q(2)));
    }
```

**控制台输出:**

```
{Q{count=-8}=Java Script, Q{count=-4}=Python, Q{count=2}=Go, Q{count=5}=Java}
最小的key->Q{count=-8}
最大的key->Q{count=5}
比Q(-4)大的最小的keyQ{count=2}
比Q(-4)小的最大的keyQ{count=-8}
大于等于Q(-4),小于Q(2){Q{count=-4}=Python}
```

## 5.5`WeakHashMap`实现类

`WeakHashMap`与`HashMap`的用法基本相似.与`HashMap`的区别在于,`HashMap`的`key`保留了对实际对象的强引用,这意味着只要该`HashMap`对象不被销毁,该`HashMap`的所有`key`所引用的对象就不会被垃圾回收,`HashMap`也不会自动删除这些`key`所对应的`key-value`对:但`WeakHashMap`的`key`只保留了对实际对象的弱引用,这意味着如果`WeakHashMap`对象的`key`所引用的对象没有被其他强引用变量所引用,则这些`key`所引用的对象可能被垃圾回收,`WeakHashMap`也可能自动删除这些`key`所对应的`key-value`对.

`WeakedHashMap`中的每个`key`对象只持有对实际对象的弱引用,因此,当垃圾回收了该`key`所对应的实际对象之后,`WeakHashMap`会自动删除该`key`对应的`key-value`对.
**示例代码:**

```java
	@Test
    public void test13() {
        WeakHashMap<String, String> weakHashMap = new WeakHashMap<>();
        //new String生成的对象都是匿名字符串对象,没有其它强引用
        weakHashMap.put(new String("Java"), "1");
        weakHashMap.put(new String("Python"), "2");
        weakHashMap.put(new String("Go"), "3");
        //"java script"是一个系统缓存的字符串直接量,系统会直接保留对该字符串对象的强引用,所以垃圾回收时不会回收它
        weakHashMap.put("java script", "4");
        System.out.println(weakHashMap);
        System.gc();
        System.runFinalization();
        System.out.println("after GC weakHashMap->" + weakHashMap);
        System.out.println("==============================");
        //HashMap保留了对实际对象的强引用,所以只要HashMap对象没有被销毁,该对象的所有key引用的对象就不会被回收.
        HashMap<String, String> hashMap = new HashMap<>();
        hashMap.put(new String("Java"), "1");
        hashMap.put(new String("Python"), "2");
        hashMap.put(new String("Go"), "3");
        hashMap.put("java script", "4");
        System.out.println(hashMap);
        System.gc();
        System.runFinalization();
        System.out.println("after GC hashMap->" + hashMap);
    }
```

**控制台输出:**

```
{Java=1, Python=2, Go=3, java script=4}
after GC weakHashMap->{java script=4}
==============================
{Java=1, Go=3, java script=4, Python=2}
after GC hashMap->{Java=1, Go=3, java script=4, Python=2}
```

**注意:**如果需要使用`WeakHashMap`的`key`来保留对象的弱引用,则不要让该`key`所引用的对象具有任何强引用,否则将失去使用`WeakHashMap`的意义.

## 5.6`IdentityHashMap`实现类

`IdentityHashMap`类的实现机制与`HashMap`基本相似,但它在处理两个`key`相等时比较独特,当且仅当`key1`==`key2`时,`IdentityHashMap`才认为两个`key`相等.而`HashMap`则是根据`equals`是否返回`true`,以及它们的`hashCode()`值相等进行判断根据.

**示例代码:**

```java
    @Test
    public void test14() {
        //key值根据是否==作为判断依据
        IdentityHashMap<String, String> identityHashMap = new IdentityHashMap<>();
        //new String()生成新的对象,即使放入的字符串相同,但生成的字符串对象地址不同,所以==不同
        identityHashMap.put(new String("AA"), "11");
        identityHashMap.put(new String("AA"), "22");
        //"BB"为系统字符串常量,指向地址相同
        identityHashMap.put("BB", "33");
        identityHashMap.put("BB", "44");
        System.out.println("identityHashMap" + identityHashMap);
        //根据equals()和hashCode()进行判断
        HashMap<String, String> hashMap = new HashMap<>();
        hashMap.put(new String("AA"), "11");
        hashMap.put(new String("AA"), "22");
        hashMap.put("BB", "33");
        hashMap.put("BB", "44");
        System.out.println("hashMap" + hashMap);
    }
```

**控制台输出:**

```
identityHashMap{AA=11, BB=44, AA=22}
hashMap{AA=22, BB=44}
```

## 5.7`EnumMap`实现类

`EnumMap`是一个与枚举类一起使用的`Map`实现,`EnumMap`中的所有`key`都必须是单个枚举类的枚举值.创建`EnumMap`时必须显式或隐式指定它所对应的枚举类.`EnumMap`具有如下特征:

* `EnumMap`在内部以数组形式保存,所以这种实现方式非常紧凑,高效.
* `EnumMap`根据`key`的自然顺序(即枚举值在枚举类中的定义顺序)来维护`key-value`对的顺序.
* `EnumMap`不允许使用`null`作为`key`,但允许使用`null`作为`value`.

**示例代码:**

```java
	enum Time{
        SECOND,MINUTE,HOURS,QUARTER;
    }

    @Test
    public void test15() {
        //初始化enumMap并制定对应的Enum
        EnumMap enumMap = new EnumMap(Time.class);
        enumMap.put(Time.HOURS, "小时");
        enumMap.put(Time.QUARTER, "刻度");
        enumMap.put(Time.SECOND, "秒");
        //按照枚举中定义的顺序排序
        System.out.println(enumMap);
    }
```

**控制台输出:**

```
{SECOND=秒, HOURS=小时, QUARTER=刻度}
```

## 5.8各`Map`实现类的性能分析

`HashMap`和`Hashtable`的实现机制几乎一样,但由于`Hashtable`是一个古老的,线程安全的集合,因此`HashMap`通常比`Hashtable`要快.

`TreeMap`中的`key-value`对总是处于有序状态,无需专门进行排序操作.由于其底层采用红黑树来管理`key-value`对(红黑树的每个节点就是一个`key-value`对).因此`TreeMap`通常比`HashMap`,`Hashtable`要慢(尤其是在插入,删除`key-value`对时更慢)

`LinkedHashMap`比`HashMap`慢一点,因为它需要维护链表来保持`Map`中`key-value`时的添加顺序.

`IdentityHashMap`性能没有特别出色之处,只是判断`key`的标准是`==`.

`EnumMap`的性能最好,但它只能使用同一个枚举类的枚举值作为`key`.

`HashSet`,`HashMap`和`Hashtable`默认的负载极限是`0.75`

## 5.9同步控制

`Collections`类中提供了多个`synchronizedXxx()`方法,该方法将指定集合包装成线程同步的集合,从而解决多线程并发访问集合时的线程安全问题.

**示例代码:**

```java
    @Test
    public void test16() {
        List<Object> list = Collections.synchronizedList(new ArrayList<>());
        Set<Object> objects = Collections.synchronizedSet(new HashSet<>());
        Map<Object, Object> hashMap = Collections.synchronizedMap(new HashMap<>());
    }
```