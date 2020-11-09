# 1.集合介绍

1. Java 集合大致分为Set，List,Queue和Map四种体系
   1. Set代表不重复，无序的集合
   2. List代表有序，重复的集合
   3. Map代表映射
   4. Queue代表一种队列的集合

2. 所有的集合类都位于java.util包下，后来为了支持线程安全机制，Java5还在java.util.concurrent包下提供了一些多线程集合类

3. Java集合与数组的区别

   数组中可以保存集合基本数据类型，也可以保存对象的引用（习惯上称为对象），而集合只能保存对象的引用（习惯上称为对象）

4. Java的集合类主要由两个接口派生出来：Collection和Map类，Collection和Map是Java集合框架的根接口，这两个接口又包含了一些子接口或实现类![image-20201109153857809](./pic/image-20201109153857809.png)

上图中国Set和List接口是Collection接口派生出的两个子接口，他们分别代表无序集合和有序的集合，Queue是Java提供的队列实现，类似于List

5. Map保存的每一项数据都是key-value对，也就是由key和value两个值组成。Map里面的key是不可重复的，key用于标识集合里的每项数据,Set集合无法记住添加这个元素的顺序，所以Set里的元素不能重复（既然没有顺序，就不能重复否则无法识别这个元素），List非常像一个数组，它可以记住每次添加元素的顺序，且List长度是可变的。
6. 如果访问List集合中的元素，可以直接根据元素的索引来访问；如果访问Map集合中的元素，可以根据每项元素的key来访问其value值；如果访问Set集合中的元素，则只能根据元素的本身来访问

![image-20201109155717198](./pic/image-20201109155717198.png)

# 2. Java11增强的Collection和Iterator接口

1. Collection接口是List，Set和Iterator接口
   1. boolean add(Object o):该方法用于向结合中添加一个元素。如果集合对象被添加操作改变了，则返回true。
   2. boolean addAll(Collection c)：该方法把集合c中的所有元素添加到指定的集合里。如果集合对象被添加操作改变了，则返回true。
   3. void clear():清除集合里的所有元素，将集合长度变为0.
   4. boolean contains(Object o)返回集合里是否包含指定的元素
   5. boolean containsAll(Collection c):返回结合里是否包含集合c里所有的元素
   6. boolean isEmpty():返回集合是否为空。当集合长度为0时返回true,否则返回false
   7. Iterator iterator() ：返回一个Iterator对象，用于遍历集合里的元素
   8. boolean remove(Object o)删除集合中的指定元素o,当集合中包含了一个或者多个元素o的时候，该方法只删除第一个符合条件的元素，该方法将返回true
   9. boolean removeAll(Collection c）：从集合中删除集合c里包含的元素，如果删除一个或者一个以上的元素，则该方法返回true
   
   10. boolean retainAll(Collection c):从集合中删除集合c里不包含的元素（相当于把调用该方法的集合变成该集合和结合c的交集），如果该操作改变了调用该方法的集合，则该方法返回true
   11. int size():该方法返回集合里元素的个数
   12. Object[] toArray():该方法把集合转换成一个数组，所有的集合元素变成对应的数组元素

```Java
var c=new ArrayList();
c.add("孙悟空");
c.add(6);
System.out.println("c集合的元素个数为："+c.size());
c.remove((Integer)6);
System.out.println("c集合的元素个数为："+c.size());
System.out.println("是否包含孙悟空"+c.contains("孙悟空"));
System.out.println("是否包含6"+c.contains(6));
c.add("java ee企业应用实践");
System.out.println(c);
System.out.println("===================");
var books=new HashSet();
books.add("java ee企业应用实践");
books.add("疯狂javaee讲义");
System.out.println("是否包含books中所有的元素"+c.containsAll(books));
c.removeAll(books);
System.out.println(c);
c.clear();
books.retainAll(c);
System.out.println("books元素的集合"+books);
```

传统的模式下将一个对象丢进集合，集合会忘记这个对象的类型，也就是说全部当做Object类型，但是jdk1.5后就改进了，可以使用泛型来限制集合里元素的类型。

Java11为Collection新增了一个toArray(IntFunction)方法，使用该方法的主要目的就是利用泛型，但对于传统的toArray而言，不管Collection本身是否使用泛型，toArray()的返回值总是Object[]；但是新增的toArray不同，当使用Collection使用泛型的时候，toArray可以返回特定的类型数组

```Java
 //该Collection使用了泛型，指定它的集合元素是String
var StrColl= List.of("Java","Kotlin","Swift","Python");
//toArray()方法参数是一个lambda表达式，代表IntFunction对象
 //toArray()方法的返回值类型是String[] 而不是Object[]
String[] aa=StrColl.toArray(String[]::new);
System.out.println(Arrays.toString(aa));
```

由于使用该方法的主要目的就是利用泛型，因此toArray(IntFunction)方法参数通常就是它要返回的数组类型后面加双引号和new(构造器引用)

## 2.使用Lambda表达式遍历集合

1. Java8为Iterable接口新增了一个forEach(Consumer action)默认方法，该方法所需要的参数的类型是一个函数式的接口，而Iterable接口是Collection接口的父接口，因此Collection集合也可以直接调用该方法。当程序调用Iterable的forEach(Consumer action)遍历集合元素的时候，程序会依次将结合元素传给Consumer的accep(T t)方法（该接口中唯一的抽象方法）。正因为Consumer是函数式的接口，因此可以使用lambda表达式来遍历集合元素。

   ```Java
   var books=new HashSet();
   books.add("轻量级JavaEE企业应用实践");
   books.add("疯狂Java讲义");
   books.add("疯狂Android讲义");
   books.forEach(obj->System.out.println("迭代集合元素："+obj));
   ```

上面程序中粗体字代码调用了Iterable的forEach()默认方法来遍历集合元素，传给该方法的参数是Lambda表达式，该Lambda表达式的目标类型是Consumer。forEach()方法会自动的将结合元素逐个的传给Lambda表达式的形参，这样Lambda表达式就可以遍历集合元素了.