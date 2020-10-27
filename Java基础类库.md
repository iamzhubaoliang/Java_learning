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

1. String类是不可变类，即一旦一个String对象被创建以后，包含在这个对象中的字符序列是不可改变的，直至这个对象被销毁。

2. StringBuffer对象则代表一个字符序列可变的字符串，当一个StringBuffer被创建以后，通过StringBuffer提供的append(),insert(),reverse(),setCharAt(),setLength()等方法可以改变这个字符串对象的字符序列。一旦通过StringBuffer生成了最终想要的字符串，就可以调用它的toString()方法将其转换为一个String对象。

3. StringBuilder和StringBuffer基本相似，两个类的构造器和方法也基本相同。不同的是，StringBuffer是线程安全的，而StringBuilder则没有实现线程安全的功能，所以性能略高。因此在通常的情况下，如果要创建一个内容可变的字符串对象，则应该有限考虑十一哦你给StringBuilder类。

4. **String，StringBulider，StringBuffer都实现了CharSequence接口**，因此CharSequence可认为是一个字符串的协议接口。

   **有关String的**

   * String():创建一个包含0个字符串序列的String对象（并不是返回Null）

   * String(byte[] bytes,Charset charset):使用指定的字符集将指定的byte[]数组解码成一个新的String对象。

   * String(byte[] bytes,int offset,int length):使用平台的默认字符集将指定byte[]数组从offset开始，长度length的子数组解码成一个新的String对象

   * String(byte[] bytes,String charsetName):使用指定字符集将指定的byte[]数组解码成一个新的String对象

   * String(char[] value,int offset,int count):将指定的字符数组从offset开始，长度为count的字符元素连缀成字符串

   * **String(String original)**:根据字符串直接量床架一个String对象，也就是说，新创建的String对象是该参数字符串的副本

   * **String(StringBuffer buffer)**:根据StringBuffer对象来创建对应的String对象

   * **String(StringBuilderr builder):**根据StringBuilder对象来创建对应的String对象

     **操作字符串**

   * **char charAt(int index)**:获取字符串中指定位置的字符。其中，参数index指的是字符串序数，字符串的字符序数从0开始到length()-1

     ```Java
     var s="fkit.org";
     System.out.println(s.charAt(5));
     ```

   * **int compareTo(String anoatherString)**:比较两个字符串的大小。如果两个字符串的字符序列相等则返回0；不相等时，从两个字符串第0个字符串开始比较，**返回第一个不相等的字符差**，**另一种情况，较长字符串的前面部分恰巧时较短的字符串，则返回他们的长度差**

   * String concat(String str):将该String对象与str连接在一起。与Java提供的字符串连接运算符“+”的功能相同。

   * boolean contentEquals(StringBuffer sb):将该String对象与StringBuffer对象sb进行比较，当它们包含的字符序列相同时返回true。

   * **static String copyValueOf(char[] data)**:将字符数组连缀成字符串，与String(char[] content)构造器的功能相同。

   * **static StringcopyValue(char[] data,int offset,int count)**:将char数组的子数组中的元素连缀成字符串，与String(char[] value,int offset,int count)构造器的功能相同

   * boolean **endsWith(String suffix)**:返回该String对象是否以suffix结尾

   * boolean equals(Object anObject):将该字符数组与指定对象比较，如果二者包含的字符序列相等，则返回true；否则false

   * **boolean equalsIgnoreCase(String str)**:与前一个方法基本相似，只是忽略字符的大小写。

   * byte[] getBytes():将该String对象转换成byte数组

   * void getChars(int srcBegin,int srcEnd,char[] dst,int dsBegin):该方法将字符串从srcBegin开始，到srcEnd结束的字符复制到dst字符数组中，其中dstBegin为目标字符数组的起始位置

   * **int indexOf(int ch):**找出ch字符在该字符串中**第一次出现的位置**

   * **int indexOf(int ch,int fromIndex):**找出ch字符在该字符串中从fromIndex开始后第一次出现的位置

   * int indexOf(String str):找出str子字符串在该字符串中第一次出现的位置

   * int indexOf(String str,int fromIndex):找出str子字符串在该字符串中从fromIndex开始后第一次出现的位置

   * int indexOf(String str,int fromIndex):找出str字符串在该字符串中从fromIndex开始后第一次出现的位置

   * **int lasIndexOf(int ch):**找出ch字符在该字符串中**最后一次出现的位置**

   * **int lastIndexOf(int ch,in fromIndex):**找出ch字符在该字符串中从fromIndex开始后最后一次出现的位置

   * Int lastIndexOf(String str):找出str字符串在该字符出啊中最后一次出现的位置。

   * int lastIndexOf(String str,int fromIndex):找出str子字符串在该字符串中从fromIndex开始后最后一次出现的位置

   * int length():返回当前字符串长度

   * **String replace(char oldChar,char newChar)**:将字符串中的第一个oldChar替换成newChar。

   * **boolean startsWith(String prefix)**:该String对象是否以prefix开始

   * **boolean startsWith(String prefix,int toffset**):该对象从toffset位置算起，是否以prefix开始。

   * String substring(int beginIndex):获取从beginIndex位置开始到结束的字符串

   * String substring(int beginIndex,int endIndex):获取从beginIndex位置开始到endIndex位置的子字符串

   * char[] toCharArray():将该String对象转换成char数组

   * **String toLowerCase():**将字符串转换成小写

   * **String ToUpperCase()**：将字符串转换成大写

   * **static String valueOf(X x)**:一系列用于将解百纳类型值转换为String对象的方法。

## 6.Math类

1. Java提供了基本的+,-,*,/等基本算数的运算符，但对于更复杂的数学运算，例如，三角函数，对数运算，指数运算无能为力
2. Math类是一个工具类，它的构造器被定义为private的，因此无法创建Math类的对象；Math中所有方法都是类方法，可以直接通过类名来调用。Math类除了体哦ing大量的静态类之外还提供了两个类变量：PI和E即Π和e

## 7.ThreaLocalRandom与Random

1. Random类专门用于生成一个伪随机数，它有两个构造器；一个构造器使用默认的种子（以当前的时间作为种子），另一个构造器需要程序员显式传入一个long型整数的种子。

2. ThreadLocalRandom类是Java7新增的一个类，它是Random的增强版。在并发访问的环境下，使用ThreadLocalRandom来代替Random可以减少多线程资源的竞争，最终保证系统具有更好的线程安全性。

3. ThreadLocalRandom类的用法与Random类的用法基本相似，它提供了一个静态的current()方法来和洛区ThreadLocalRandom对象，获取该对象之后即可调用各种nextXxx()方法来获取伪随机数了。

4. ThreadLocalRandom与Random都比Math的Random方法提供了更多的方式来生成各种伪随机数，可以生成浮点类型的伪随机数，也可以生成整数类型的伪随机数，还可以指定生成随机数的范围。

   ```Java
   var rand=new Random();
   System.out.println(""+rand.nextBoolean()+","+rand.nextDouble()+","+rand.nextFloat()+",");
   var buffer=new byte[16];
   rand.nextBytes(buffer);
   System.out.println(Arrays.toString(buffer));
   ```

5. Random使用一个48位的种子，如果这两个类的两个实例是用同一个种子创建的，对它们以同样的顺序调用方法，则它们会产生相同的数字序列。

   ```Java
   var r1=new Random(50);
       System.out.println(""+r1.nextInt()+","+r1.nextFloat()+","+r1.nextGaussian());
   
       var r3=new Random(50);
       System.out.println(""+r3.nextInt()+","+r3.nextFloat()+","+r3.nextGaussian());
   	//高斯是产生均值为0，标准差为1的伪随机数
       var r2=new Random(50);
       System.out.println(""+r2.nextFloat()+","+r2.nextInt()+","+r2.nextGaussian());
   -1160871061,0.597892,1.4344888752894227
   -1160871061,0.597892,1.4344888752894227
   0.7297136,-1727040520,1.4344888752894227
   ```

只要两个Random对象的种子相同，而且方法的调用顺序也相同，它们就会产生相同的数字序列。从而证明了Random产生的数字并不是真正的随机而是一种种伪随机。

为了避免两个Random对象产生相同的数字序列，通常推荐使用当前的时间作为Random对象的种子

```Java
Random rand=new Random(System.currentTimeMillis());
```

6. 多线程环境下使用ThreadLocalRandom的方式与使用Random基本类似

   ```Java
   ThreadLocalRandom rand=ThreadLocalRandom.current();
   int val=rand.nextInt(4,20);
   double val1=rand.nextDouble();
   ```

7. BigDecimal类

   1. float,double两种基本浮点类型时已经指出，这两个基本类型的浮点数容易引起精度的丢失。为了能够精确表示，计算浮点数，Java提供了BigDecimal类，该类提供了大量的构造器用于创建BigDecimal对象，包括把所有的基本数值型变量转换成BigDecimal对象，也包括利用数字字符串、数字字符数组来创建BigDecimal对象

   2. 查看BigDecimal类的Bigdecimal(double val)构造器的详细说明时，可以看到不推荐使用该构造器。主要时因为使用该构造器时有一定的不可预知性，当程序使用new BigDecimal(0.1)来创建一个BigDecimal对象时，它的值并不是0.1，它实际上等于一个近似0.1的数。这是因为0.1无法准确的表示为double浮点数，所以传入BigDeciaml构造器的值不会正好等于0.1

      如果使用BigDecimal(String val)构造器的结果时可预知的---写入new BigDecimal("0.1")将创建一个BigDecimal，**它正好等于预期的0.1**，因此通常建议优先使用基于String的构造器。

      如果必须使用double浮点数作为BigDecimal构造器的参数时，不要直接将该double浮点数作为构造器参数创建BigDecimal对象，而是应该通过BigDecimal.valueOf(double value)静态方法来创建BigDecimal对象。

   3. BigDecimal类提供了add(),subtract(),multiply(),divide(),pow()等方法对精确浮点数进行常规算数运算

      ```Java
      var f1=new BigDecimal("0.05");
      var f2=BigDecimal.valueOf(0.01);
      var f3=new BigDecimal(0.05);
      
      System.out.println("使用String作为BigDecimal");
      System.out.println(f1.add(f2));
      System.out.println(f1.subtract(f2));
      System.out.println(f1.multiply(f2));
      System.out.println(f1.divide(f2));
      
      System.out.println("使用double作为BigDecimal");
      System.out.println(f3.add(f2));
      System.out.println(f3.subtract(f2));
      System.out.println(f3.multiply(f2));
      System.out.println(f3.divide(f2));
      
      输出
          使用String作为BigDecimal
      0.06
      0.04
      0.0005
      5
      使用double作为BigDecimal
      0.06000000000000000277555756156289135105907917022705078125
      0.04000000000000000277555756156289135105907917022705078125
      0.0005000000000000000277555756156289135105907917022705078125
      5.000000000000000277555756156289135105907917022705078125
      ```

   4. 如果程序中要求对double浮点数进行加，减，乘，除基本运算，则需要先将double类型数值包装成BigDecimal对象，调用BigDecimal对象的方法执行运算后再将结果转换成double型变量。这是比较繁琐的过程，可以考虑以BigDecimal为基础定义一个Arith工具类

      ```Java
      public class Arith {
        private static final int DEF_DIV_SCALE=10;
        public static double add(double v1,double v2){
          var b1= BigDecimal.valueOf(v1);
          var b2=BigDecimal.valueOf(v2);
          return b1.add(b2).doubleValue();
        }
        //除法与其他的运算不同，其他运算类似 add()
        public static double div(double v1,double v2){
          var b1=BigDecimal.valueOf(v1);
          var b2=BigDecimal.valueOf(v2);
          return b1.divide(b2,DEF_DIV_SCALE, RoundingMode.HALF_UP).doubleValue();
        }//提供相对精确的除法运算，当发生除不尽的情况时精确到小数点以后10位数字四舍五入。
        }
      ```