# 系统相关

## 2.Runtime类与Java9的ProcessHandle

1. Runtime类代表Java程序的运行环境，每个Java程序都有一个与之对应Runtime实例，应用程序通过该对象与其运行时环境相连。应用程序不能创建自己的Runtime实例，但可以通过getRuntime()方法获取与之关联的Runtime对象。

2. 与System相似的是，Runtime类也提供了gc()方法和runFinalization方法来通知系统进行垃圾回收，清理系统资源，并提供了load(String filename)和loadLibrary(String libname)方法来加载文件和动态链接库。

3. Runtime类代表Java程序的运行环境，可以访问JVM的相关信息，如处理器数量、内存信息等

   ```Java
   public class userInteraction {
       public static void main(String[] args){
           var rt=Runtime.getRuntime();
           System.out.println("处理器数量"+rt.availableProcessors());
           System.out.println("空闲内存数"+rt.freeMemory());
           System.out.println("总内存数"+rt.totalMemory());
           System.out.println("可用最大内存数"+rt.maxMemory());
       }
   }
   ```

4. Runtime类可以单独启动一个进程来运行操作系统的命令

   ```Java
   public class userInteraction {
       public static void main(String[] args) throws Exception{
          var rt=Runtime.getRuntime();
          rt.exec("notepad");
       }
   }
   ```

5. 

   


## 3.常用类

### 1.Object类

1. boolean equals(Object obj)

2. protected void finalize():当系统中没有引用变量引用到该对象时，垃圾回收器调用此方法来清理该对象的资源。

3. Class<?>getClass():返回该对象运行时的类

4. int hashCode():返回该对象的hashCode值，在默认情况下，Object类的hashCode()方法根据该对象的地址来进行计算（即与System.ideatityHashCode(Object x)方法计算的结果相同）。但很多类都重写了Obect类的hashCode()方法，不再根据地址来计算其hashCode()值

5. String toString():返回该对象的字符串表示。

6. 除此之外Object类还提供了wait(),notify(),notifyAll()几个方法，通过这几个方法可以控制线程的暂停和运行。

7. Java还提供了一个protected的修饰的clone方法，该方法用于帮助其他对象来实现“自我克隆”，所谓“自我克隆”就是得到一个当前对象的副本，而且二者之间完全隔离。由于Object类提供的clone()方法使用了protected修饰，因此该方法只能被子类重写或调用。

   自定义类实现"克隆"的步骤如下

   1. 自定义实现Cloneable接口。这是一个标记性的接口，实现该接口的对象可以实现“自我克隆”,接口里没有定义任何的方法
   
   2. 自定义类实现自己clone方法
   
   3. 实现clone()方法时通过super.clone()：**调用Object实现的clone()**方法来得到该对象的副本，并返回该副本。
   
      ```Java
      class Address{
        String detail;
        public Address(String detail){
          this.detail=detail;
        }
      }
      class User implements Cloneable{//标记性接口
        int age;
        Address address;
        public User(int age){
          this.age=age;
          address=new Address("兰州大学");
        }
        public User clone() throws CloneNotSupportedException//实现克隆方法
        {
          return (User)super.clone();//调用Object实现的clone()方法来得到该对象的副本，并返回该副本。
        }
      }
      public class userInteraction {
          public static void main(String[] args) throws Exception{
            var u1=new User(29);
            var u2=u1.clone();
            System.out.println(u1==u2);//false
            System.out.println(u1.address==u2.address);//true
          }
      }
      ```

Object类提供的clone机制支对象里各实例变量进行“简单复制”，如果实例变量的类型为引用类型，Object的Clone机制也只是简单地复制这个引用变量，这**样原有对象的引用类型的实例变量与克隆对象的引用类型的实例变量依然指向内存中的同一个实例，所以上面程序中最后一行输出true。第一个false输出的原因是u1对象的引用和u2对象的引用不同，而他们之中的实例变量完全是复制过来的。**

Object类提供的clone()方法不仅能简单地处理“复制”对象的问题，而且这种“自我克隆”机制十分高效。比如clone一个包含100个元素的Int[]数组，用系统默认的clone方法比静态copy方法快近2倍

需要指出的是，object类的clone方法虽然简单，易用，但它只是一种“浅克隆”，它只是克隆该对象的所有成员变量值，不会对引用类型的成员变量值所引用的对象进行克隆。如果开发者需要对对象进行“深克隆”，则需要开发者自己进行递归克隆，保证所有引用类型的成员变量值所引用的对象都被复制了。

## 4.操作对象的Object工具类 

1. Java7新增了一个Object工具类，它提供了一些工具方法来操作对象，这些工具方法大多是“空指针”安全的。比如你不确定引用变量是否为Null，如果贸然用该变量的toString()方法，则可能引发NullPointerException异常；如果使用Objects类提供的toString(Object o )方法，就不会引发空指针异常，当o为Null时，程序将返回 一个"null"字符串

2. ```Java
   public class userInteraction {
     static userInteraction obj;
     public static void main(String[] args) throws Exception{
   
         System.out.println(Objects.hashCode(obj));//0
         System.out.println(Objects.toString(obj));//null
         System.out.println(Objects.requireNonNull(obj,"obj参数不能是null!"));
       }
   }
   ```

上面的程序示范了Objects提供的requireNonNull()方法，当传入的参数不为null时，该方法返回参数本身，否则将会引发NullPointerException异常。该方法主要用来对方法形参进行输入校验，例如

```Java
public class userInteraction {
  static userInteraction obj;
  public static void main(String[] args) throws Exception{
    userInteraction a=Objects.requireNonNull(obj,"参数不能为null");
    }
}
```

## 5.Java9改进的String、StringBuffer 和StringBuilder类

1. String类时不可变类，即一旦一个String对象被创建以后，包含在这个对象中的字符序列时不可改变的，直至这个对象被销毁。

2. StringBuffer对象则代表一个字符序列可变的字符串，当一个StringBuffer被创建以后，通过StringBuffer提供的append(),insert(),reverse(),setCharAt(),setLength()等方法可以改变这个字符串对象的字符序列。一旦通过StringBuffer生成了最终想要的字符串，就可以调用它的toString()方法将其转换为一个String对象。

3. StringBuilder和StringBuffer基本相似，两个类的构造器和方法也基本相同。不同的是，StringBuffer是线程安全的，而StringBuilder则没有实现线程安全的功能，所以性能略高。因此在通常的情况下，如果要创建一个内容可变的字符串对象，则应该有限考虑十一哦你给StringBuilder类。

4. String，StringBulider，StringBuffer都实现了CharSequence接口，因此CharSequence可认为是一个字符串的协议接口。

   **有关String的**

   String():创建一个包含0个字符穿序列的String对象（并不是返回Null）