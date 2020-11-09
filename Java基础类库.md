# 系统相关

## 2.Runtime类与Java9的ProcessHandle

1. Runtime类代表Java程序的运行环境，每个Java程序都有一个与之对应Runtime实例，应用程序通过该对象与其运行时环境相连。应用程序不能创建自己的Runtime实例，但可以通过getRuntime()方法获取与之关联的Runtime对象。

2. 与System相似的是，Runtime类也提供了**gc()方法和runFinalization**方法来通知系统进行垃圾回收，清理系统资源，并提供了load(String filename)和loadLibrary(String libname)方法来加载文件和动态链接库。

3. **Runtime类代表Java程序的运行环境**，可以访问JVM的相关信息，如处理器数量、内存信息等

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

4. int hashCode():返回该对象的hashCode值，在默认情况下，Object类的hashCode()方法根据该对象的地址来进行计算（即与System.identityHashCode(Object x)方法计算的结果相同）。但很多类都重写了Obect类的hashCode()方法，不再根据地址来计算其hashCode()值

5. String toString():返回该对象的字符串表示。

6. 除此之外Object类还提供了**wait(),notify(),notifyAll()**几个方法，**通过这几个方法可以控制线程的暂停和运行**。

7. Java还提供了一个protected的修饰的clone方法，该方法用于帮助其他对象来实现“自我克隆”，所谓“自我克隆”就是得到一个当前对象的副本，而且二者之间完全隔离。由于Object类提供的clone()方法使用了protected修饰，因此该方法只能被子类重写或调用。

   自定义类实现"克隆"的步骤如下

   1. **自定义实现Cloneable接口。这是一个标记性的接口**，实现该接口的对象可以实现“自我克隆”,接口里没有定义任何的方法
   
   2. 自定义类实现自己clone方法
   
   3. **实现clone()方法时通过super.clone()**：**调用Object实现的clone()**方法来得到该对象的副本，并返回该副本。
   
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

## 4.操作对象的Objects工具类 

1. Java7新增了一个Object工具类，它提供了一些工具方法来操作对象，**这些工具方法大多是“空指针”安全的。比如你不确定引用变量是否为Null，如果贸然用该变量的toString()方法，则可能引发NullPointerException异常；如果使用Objects类提供的toString(Object o )方法**，就不会引发空指针异常，当o为Null时，程序将返回 一个"null"字符串

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

上面的程序示范了Objects提供的**requireNonNull()**方法，当传入的参数不为null时，该方法返回参数本身，否则将会引发NullPointerException异常。该方法主要用来对方法形参进行输入校验，例如

```Java
public class userInteraction {
  static userInteraction obj;//因为没有赋值，所以是空的
  public static void main(String[] args) throws Exception{
    userInteraction a=Objects.requireNonNull(obj,"参数不能为null");
    }
}
```

## 5.Java9改进的String、StringBuffer 和StringBuilder类

1. **String类是不可变类**，即一旦一个String对象被创建以后，包含在这个对象中的字符序列是不可改变的，直至这个对象被销毁。

2. **StringBuffer对象则代表一个字符序列可变的字符串**，当一个StringBuffer被创建以后，通过StringBuffer提供的**append(),insert(),reverse(),setCharAt(),setLength()**等方法可以改变这个字符串对象的字符序列。**一旦通过StringBuffer生成了最终想要的字符串，就可以调用它的toString()方法将其转换为一个String对象。**

3. StringBuilder和StringBuffer基本相似，两个类的构造器和方法也基本相同。不同的是，**StringBuffer是线程安全的，而StringBuilder则没有实现线程安全的功能，所以性能略高**。因此在通常的情况下，如果要创建一个内容可变的字符串对象，则应该有限考虑十一哦你给StringBuilder类。

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

   * **static String copyValue(char[] data,int offset,int count)**:将char数组的子数组中的元素连缀成字符串，与String(char[] value,int offset,int count)构造器的功能相同

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

   * **String substring(int beginIndex):获取从beginIndex位置开始到结束的字符串**

   * **String substring(int beginIndex,int endIndex):获取从beginIndex位置开始到endIndex位置的子字符串**

   * **char[] toCharArray():将该String对象转换成char数组**

   * **String toLowerCase():**将字符串转换成小写

   * **String ToUpperCase()**：将字符串转换成大写

   * **static String valueOf(X x)**:一系列用于将解百纳类型值转换为String对象的方法。
   
   * ```Java
     String a=new String("字符串");
     StringBuffer sb=new StringBuffer("可变字符串");
     StringBuilder sd=new StringBuilder("可变字符串");
     System.out.println(a);
     System.out.println(sb);
     System.out.println(sd);
     System.out.println(new String(new char[]{'i','a','m'}));
     System.out.println(String.copyValueOf(new char[]{'i','a','m'}));
     System.out.println("aabb".replace("b","c"));
     System.out.println("aabbcc".replaceFirst("b","c"));
     System.out.println("aabbccdd".toCharArray());
     ```

## 6.Math类

1. Java提供了基本的+,-,*,/等基本算数的运算符，但对于更复杂的数学运算，例如，三角函数，对数运算，指数运算无能为力
2. Math类是一个工具类，它的构造器被定义为private的，因此无法创建Math类的对象；Math中所有方法都是类方法，可以直接通过类名来调用。Math类除了体哦ing大量的静态类之外还提供了两个类变量：PI和E即Π和e

## 7.ThreaLocalRandom与Random

1. Random类专门用于生成一个伪随机数，它有两个构造器；一个构造器使用默认的种子（以当前的时间作为种子），另一个构造器需要程序员显式传入一个long型整数的种子。

2. ThreadLocalRandom类是Java7新增的一个类，它是Random的增强版。**在并发访问的环境下，使用ThreadLocalRandom来代替Random可以减少多线程资源的竞争，最终保证系统具有更好的线程安全性。**

3. **ThreadLocalRandom类的用法与Random类的用法基本相似，它提供了一个静态的current()方法来获取ThreadLocalRandom对象**，获取该对象之后即可调用各种nextXxx()方法来获取伪随机数了。

   ```Java
   ThreadLocalRandom rd =ThreadLocalRandom.current();
   System.out.println(rd.nextFloat());
   ```

4. **ThreadLocalRandom与Random都比Math的Random方**法提供了更多的方式来生成各种伪随机数，可以生成浮点类型的伪随机数，也可以生成整数类型的伪随机数，还可以指定生成随机数的范围。

   ```Java
   var rand=new Random();
   System.out.println(""+rand.nextBoolean()+","+rand.nextDouble()+","+rand.nextFloat()+",");
   var buffer=new byte[16];
   rand.nextBytes(buffer);
   System.out.println(Arrays.toString(buffer));
   ```

5. Random使用一个48位的种子，如果这两个类的两个实例是用同一个种子创建的，对它们以同样的顺序调用方法，则它们会产生相同的数字序列。

   ```Java
   Random rd=new Random(System.currentTimeMillis());
           System.out.println(rd.nextFloat());
   
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

**只要两个Random对象的种子相同，而且方法的调用顺序也相同**，它们就会产生相同的数字序列。从而证明了Random产生的数字并不是真正的随机而是一种种伪随机。

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

   1. **float,double两种基本浮点类型时已经指出，这两个基本类型的浮点数容易引起精度的丢失**。为了能够精确表示，计算浮点数，Java提供了BigDecimal类，该类提供了大量的构造器用于创建BigDecimal对象，包括把所有的基本数值型变量转换成BigDecimal对象，也包括利用数字字符串、数字字符数组来创建BigDecimal对象

   2. 查看BigDecimal类的Bigdecimal(double val)构造器的详细说明时，可以看到不推荐使用该构造器。主要时因为使用该构造器时有一定的不可预知性，**当程序使用new BigDecimal(0.1)来创建一个BigDecimal对象时，它的值并不是0.1，它实际上等于一个近似0.1的数**。**这是因为0.1无法准确的表示为double浮点数，所以传入BigDeciaml构造器的值不会正好等于0.1**

      如果使用BigDecimal(String val)构造器的结果时可预知的---写入new BigDecimal("0.1")将创建一个BigDecimal，**它正好等于预期的0.1**，因此通常建议优先使用基于String的构造器。

      如果必须使用double浮点数作为BigDecimal构造器的参数时，不要直接将该double浮点数作为构造器参数创建BigDecimal对象，而是应该通过BigDecimal.valueOf(double value)静态方法来创建BigDecimal对象。这样创建的为0.1

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

   4. **如果程序中要求对double浮点数进行加，减，乘，除基本运算，则需要先将double类型数值包装成BigDecimal对象，调用BigDecimal对象的方法执行运算后再将结果转换成double型变量。**这是比较繁琐的过程，可以考虑以BigDecimal为基础定义一个Arith工具类

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

## 8.Java8的日期时间类

1. Java提供了Date类来处理日期，时间，Date类从JDK1.0起就开始存在了，但正因为它历史悠久，所以它的大部分构造器，方法都已经过时了，不再推荐使用：但剩下了2个构造器和对象的几个方法

   1. Date():生成一个代表当前日期的Date对象，该构造器在底层调用System.currentTimeMillis()获得Long型整数作为日期参数。

   2. Date(long date):根据指定的Long型整数来生成一个Date对象。该构造器的参数表示创建的Date对象和GMT1970年1月1日00：00：00之间的时间差，以毫秒作为单位

      **Date对象的方法：**

   1. boolean after(Date when):测试该日期是否在指定日期when之后

   2. boolean before(Date wehen):测试该日期是否在指定日期when之前

   3. long getTime()返回该时间对应的long型整数，即从GMT1970-01-01 00：00：00到该Date对象之间的时间差，以毫秒作为计时单位。

      ```Java
      var d1=new Date();
      //获取当前时间之后的100ms时间
      var d2=new Date(System.currentTimeMillis()+100);
      System.out.println(d2);
      System.out.println(d1.compareTo(d2));
      System.out.println(d1.before(d2));
      ```

2. 总体来说Date是一个设计相当糟糕的类，因此Java官方推荐尽量少用Date构造器和方法，如果需要对时间进行运算或者获取相关的信息推荐使用Calendar工具类

## 9.Calendar

1. **Calender类本身是一个抽象类**，它是所有日历类的模板，并提供了一些所有日历通用的方法，但它本身并不能直接实例化，程序知道能创建Clendar子类的实例，Java本身提供了一个GregorianCalendar类，一个代表格里高利日历的子类，它代表了通常所说的公历。
2. 也可以创建自己的Calendar子类，然后将它作为Calendar对象使用（这就是多态）。
3. **Calendar类是一个抽象类**，所以不能是哟i给你构造器来进行创建Calendar对象。但它提供了几个静态getInstance()方法来获取Calendar对象，这些方法根据TimeZone,Locale类来和洛区特定的Calendar，如果不指定TimeZone,Locale，则使用默认的TImeZone，Local来创建Calendar。

```Java
var calendar= Calendar.getInstance();
var date=calendar.getTime();
var calendar2=Calendar.getInstance();
calendar2.setTime(date);
System.out.println(calendar2.getTime());
```

4. Calendar 类提供了大量访问，修改日期时间的方法

   1. void add(int field,int amount):根据日历的规则，为给定的日历字段添加或减去指定的时间量

   2. int get(int field):返回指定日历字段值

   3. int getActualMaximum(int field):返回指定日历字段可能拥有的最大值，例如月，最大值为11

   4. int getActualMinimum(int field)：返回只当日历字段可能拥有的最小值。例如月最小值为0

   5. void roll(int field,int amount)：与add方法类似，区别在于加上amount后超过了该字段所能表示的最大范围时，也不会向上一个字段进位

   6. void set(int field,int value):将给定的日历字段设置为给定的值。

   7. void set(int year,int month,int date):设置Calendar对象的年，月，日三个字段值。

   8. void set(int year,int month,int date,int hourOfDay,int minute,int second):设置Calendar对象的年、月，日，时，分，秒6个字段的值

      上面的很多方法都需要一个int 类型的field参数，field时Calendar类的类变量，如Calendar.YEAR、Calendar.MONTH等分别代表了年，月，日，小时，分钟，秒等字段。需要指出是Calendar.MONTH字段代表月份，月份的起始值不是1，而是0，所以要设置8月时，用7而不是8.

      ```Java
      var c=Calendar.getInstance();
      //取出年
        System.out.println(c.get(Calendar.YEAR));
        System.out.println(c.get(Calendar.MONTH));
        System.out.println(c.get(Calendar.DATE));
      
        c.set(2003,10,23,12,32,23);
        c.add(Calendar.YEAR,-1);//Calendar的年前推1年
        System.out.println(c.getTime());
        c.roll(Calendar.MONTH,-8);
        System.out.println(c.getTime());
      ```

   9. add与roll的区别

      add会发生进位

      ```Java
      var call=Calendar.getInstance();
      call.set(2003,7,23,0,0,0);
      call.add(MONTH,6);//2003-8-23=>2004-2-23
      call.set(2003,7,31,0,0,0);
      cal.add(MONTH,6);//2003-8-23=>2003-2-23
      ```

      roll不会发生进位

      ```Java
      var cal3=Calendar.getInstance();
      cal3.set(2003,7,31,0,0,0);
      cal3.roll(MOTH,6);//2003-8-23=>2003-2-23
      ```

   10. 设置Calendar的容错性

       ```Java
       Calendar cal=Calendar.getInstance();
       cal.set(Calendar.MONTH,13);//这里设置13会导致进位的发生Year会+1
       System.out.println(cal.getTime());
       
       //关闭容错性会导致程序出现异常
       cal.setLenient(false);
       cal.set(Calendar.MONTH,13);
       System.out.println(cal.getTime());
       ```

   Calendar 有两种解释日历字段的模式：lenient模式和non-lenient模式。当Calendar处于lenient模式时，每个时间字段可接受超出它的允许的范围值；当Calendar处于non-lenient模式时，如果为某个时间字段设置的值超出了它允许的取值范围，将会抛出异常。

   10. set方法延迟修改

       set(f,value)方法将日历字段f更改为value,此外它还设置了一个内部成员变量，以指示日历字段f已经被修改。尽管日历字段f时立即被更改的，但该Calendar所代表的时间不会立即修改，直到下次调用**get(),getTIme(),getTImeInMillis(),add()或roll()时才会重新计算日历的时间（多次set不会）。**这被称为set()方法的延迟修改，**采用延迟修改的优势是多次调用set()不会触发多次不必要的计算（需要计算出一个代表实际时间的Long型整数）**

       ```JAVA
       Calendar cal=Calendar.getInstance();
       cal.set(2003,7,31);//2003.8.31
       //这会将日期设置为9.31,但31日并不存在
       //如果立即修改，系统将会把cal自动调整到10.1
       cal.set(Calendar.MONTH,8);
       //下面输出10月1日
       System.out.println(cal.getTime());//1。如果没有注释掉会自动调整到10.1
    cal.set(Calendar.DATE,5);
       .System.out.println(cal.getTime());//2
       ```
   
   如果将1处注释掉则2处是9.5而当1处没有注释掉则就是10,5(注意一件事就是月份是从0开始的)

## 10.新的日期时间包

Java8专门新增了一个java.time包，该包下包含了如下常用的类

1. Clock：该类用于获取指定时区的当前日期，时间。该类可取代System类的currentTimeMillis()方法，而且提供了更多的方法来获取当前的时间，日期。该类提供了大量的静态方法来获取Clock对象。

   ```Java
   //获取当前的Clock
   Clock clock=Clock.systemUTC();
   //通过Clock获取当前的时间
   System.out.println("当前时刻为："+clock.instant());//当前时刻为：2020-10-29T01:24:15.431919700Z
   ```

2. Duration：该类代表持续时间。该类可以非常方便的获取一段时间。

   ```Java
   
         //获取当前的Clock
         Clock clock=Clock.systemUTC();
   //通过Clock获取当前的时间
         System.out.println("当前时刻为："+clock.instant());
         var d=Duration.ofSeconds(6000);
         System.out.println("6000秒相当于"+d.toMinutes()+"分");
         System.out.println("6000秒相当于"+d.toHours()+"小时");
         System.out.println("6000秒相当于"+d.toDays()+"天");
         var clock2=Clock.offset(clock,d);
         System.out.println("当前时刻加6000秒"+clock2.instant());
   ```

3. **Instant:代表一个具体的时刻，可以精确到纳秒。该类提供了静态的now()方法来获取当前的时刻，也提供了静态的now(Clock clock)方法来获取clock对应的时间**，除此之外还提供了一系列minusXxx()方法在当前的时刻基础上减去一段时间，也提供了plusXxx()方法在当前时刻基础上加上一段时间。

   ```Java
   var instant=Instant.now();
   //获取当前的时间
   System.out.println(instant);
   //instanta添加6000秒返回新的Instant
   var instant2=instant.plusSeconds(6000);
   System.out.println(instant2);
   //根据字符串解析Instant对象
   var instant3=Instant.parse("2014-02-23T10:12:35.3412z");
   System.out.println(instant3);
   //在instant3的基础上增加5小时4分钟
   var instance4=instant3.plus(Duration.ofHours(5).plusMinutes(4));
   System.out.println(instance4);
   //instant4的5天以前的时刻
   var instant5=instance4.minus(Duration.ofDays(5));
   System.out.println(instant5);
   ```

4. **LocalDate:该类代表不带时区的日期**，例如2007-12-03。该类提供了静态的now方法来获取当前的日期，也提供了静态的now(Clock clock)方法来获取clock对应的日期。除此之外，它还提供了minusXxx()方法在当前年份基础上减去几年，几月，几周，或者几日，也提供了plusXxx()方法在当前的年份基础上加上几年，几月，几周，或者几日。

   ```Java
   var localDate=LocalDate.now();
   System.out.println(localDate);
   //获得2014年的第146天
   localDate=LocalDate.ofYearDay(2014,146);
   System.out.println(localDate);
   //设置时间为2014、5、21
   localDate=LocalDate.of(2014,Month.MAY,21);
   System.out.println(localDate);
   ```

5. localDateTime:该类不带时区的日期，时间同上

   ```Java
   var localdatetime=LocalDateTime.now();
   var futer=localdatetime.plusHours(25).plusMinutes(3);
   System.out.println("当前日期，时间25h3min之后"+futer);
   ```

6. MothDay:该类仅代表几月几日，例如04-12.该类提供了静态的now()方法来获取当前的日期，也提供了静态的now(Clock clock)方法来获取clock对应的月，日。

7. Year：该类仅代表年，例如2014。该类提供了静态的now()方法来获取当前的年份，也提供了静态的now(Clock clock)方法来获取。除此之外，它还提供了minusYears()方法在当前年份基础上减去几年，也提供了plusYears()方法在当前年份基础上加上几年

8. YearMonth:该类仅代表年月，例如2014-04.该类提供了静态的now()......也提供了now(Clock clock).....除此之外........同上

9. ZonedDateTIme:该类代表一个时区化的日期，时间。

10. ZoneId:该类代表一个时区

11. DayOfWeek:这是一个枚举类，定义了周日到周六的枚举值

12. Month：这也是一个简单的枚举类，定义了一月到十二月的枚举值。

    ```Java
    var year=Year.now();
    System.out.println("当前的年份"+year);
    year=year.plusYears(5);
    System.out.println("当前年份再加5年"+year);
    //根据指定月份获取YearMonth
    var ym=year.atMonth(10);
    System.out.println("year年10月"+ym);
    //当前年月再加5年，减3个月
    ym=ym.plusYears(5).minusMonths(3);
    System.out.println("year年10月再加5年，减3个月"+ym);
    var md=MonthDay.now();
    System.out.println("当前月日:"+md);
    //设置为5月23
    var md2= md.with(Month.MAY).withDayOfMonth(23);
    System.out.println("5月23日为"+md2);
    ```

## 11.正则表达式

### 1.介绍

1. 正则表达式可以对字符串进行查找，提取，分割，替换等操作

   1. boolean matches(String reges,String replacement)：将该字符串中所有匹配regex的字符串替换提成replacement

3. String replaceFirst(String regex,String replacement):将该字符串中第一个匹配regex的子字符串替换成replacement

4. String[] split(String regex):以regex作为分隔符，把该字符串分割成多个子串

5. 除此之外Java还提供了Pattern和Matcher两个类专门用于提供正则表达式支持

6. 正则表达式就是一个用于匹配字符串的模板，可以匹配一批字符串，所以创建正则表达式就是创建一个特殊的字符串

   x:字符x

   \0mnn:八进制0mnn所表示的字符

   \xhh：十六进制0xhh所表示的字符

   \uhhhh:十六进制0xhhh所表示的Unicode字符

   \t:制表符('\u009')

   \n:换行符('\u00A')

   \r:回车符('\u00D')

   \f:换页符('\u00C')

   \a:报警符('\u007')

   \e:Escape符('\u001B')

   \cx:x对应的控制符，

   ==================================

   "\u0041\\\\"匹配A\

   "\u00616\t"匹配a<制表符>

   "\\\?\\\\ ]"匹配?]

   上面两个反斜杠相当于一个

7. 通配符

   * '.'可以匹配任何字符

   * \d 可以匹配0-9所有的数字

   * \D可以匹配非数字

   * \s匹配所有的空白字符，包括空格，制表符，回车，换页符，换行符等

   * \S所有非空白字符

   * \w所有单词字符，包括0-9所有数字，26个英文字母和下划线(_)

   * \W匹配所有非单词字符

     记忆：d的意思是digit代表数字;s是space的意思代表空白;w是word的意思，代表单词。d,s,w的大写形式正好与小写相反

8. 方括号表达式

   * 表示枚举[]：例如[abc]，表示a,b,c其中任意一个字符：[gz],表示g,z其中任意一个字符

   * 表示范围-：例如[a-f],表示a-f 范围内的任意字符；[\\\u004-\\\u0056],表示十六进制字符\\u0041到\\u0056范围内的字符
   
   * 表示求否^:例如\[^abc\]表示非a,b,c的任意字符；\[a-f\]表示不是a-f范围内的任意字符
   
   * 表示“与”运算：&&,例如\[a-z&&\[def\]]表示求a-z和\[def\]的交集.表示d,e,f
   
     \[a-z&&\[^bc\]],a-z范围内所有字符，除b和c之外，即[ad-z]
   
     \[a-z&&\[^m-p\]],a-z范围内的所有字符，除m-p范围之外的字符,即[a-lq-z]
   
   * 表示并运算：并运算与前面的枚举类似。例如\[a-d\[m-p\]\]表示[a-dm-p]

8. 正则表达式还支持圆括号表达式，用于将多个表达式组成一个子表达式，圆括号中可以使用或运算符。例如正则表达式((public)|(protected)|(private))用于匹配Java的三个访问控制符其中之一。

9. 正则表达式支持数量表示符有以下集中模式
   1. Greedy(贪婪模式):数量表示符默认采用贪婪模式，除非另有表示。贪婪模式的表达式会一直匹配下去，知道无法匹配为止-。如果你发现表达式匹配的结果与预期不符，很有可能是因为----你以为表达式只会匹配前面的几个字符，而实际上它是贪婪模式，所以会一直匹配下去。

   2. Reluctant(勉强模式)：用问号?表示，它只会匹配最少的字符，也称为最小匹配模式

   3. Possessive(占有模式):用加号+表示，目前只有Java支持占

   4. 有模式，通常比较少用

      ![image-20201103144231426](./pic/image-20201103144231426.png)

### 2.使用正则表达式

1. 一旦在程序中定义了正则表达式，就可以使用Pattern和Matcher来使用正则表达式

   **Pattern对象是正则表达式编译后在内存中的表达形式**，因此，**正则表达式字符串必须先被编译为Pattern对象**，然后再**利用该Pattern对象创建对应的Matcher对象**。执行匹配所涉及的状态保留在Matcher对象中，**多个Matcher对象可以共享一个Pattern对象**。

   ```Java
   //将一个字符串编译成Pattern对象
   Pattern P =Pattern.compile("ab");
   //使用Pattern对象创建Matcher对象
   Matcher m=P.matcher("aaaaab");
   System.out.println(m.matches());//true
   ```

   上面定义的Pattern对象可以重复多次使用。如果某个正则表达式仅需一次使用，则可直接使用Pattern类的静态matches方法，此方法自动把指定字符串编译成匿名的Pattern对象，并执行匹配

   ```Java
   boolean b =Pattern.matches("a*b","aaaab");
   System.out.println(b);
   ```

   但采用上面这条语句每次都要重新编译

   2. Pattern是不可变类，可供多个并发线程安全使用

   3. Matcher类提供了如下的方法

      * find():返回目标字符串中是否包含与Pattern匹配的子串
      * group():返回上一次与Pattern匹配的子串
      * start():返回上一次与Pattern匹配的子串在目标中的开始位置
      * end():返回上一次与Pattern匹配的子串在目标字符串中的位置加1
      * lookingAt():返回目标字符串前面的部分与Pattern是否匹配
      * matches():返回整个目标字符串与Pattern是否匹配
      * reset()：将现有的Matcher对象应用于一个新的字符序列

   4. **CharSequence接口代表一个字符串序列，**可以是各种形式的字符串。

   5. 通过Matcher类的find()和group方法可以从目标字符串中依次取出特定的子串（匹配正则表达式的子串），例如互联网爬虫，它们可以自动从网页中识别出所有的电话号码。

      ```Java
       var str="我想求购一本《疯狂Java讲义》，尽快联系我13500006666"+"交朋友，电话号码是15611125565";
          Matcher m=Pattern.compile("((13\\d)|(15\\d))\\d{8}").matcher(str);
          while (m.find()){
              System.out.println(m.group());
          }
      }
      ```

      **find()方法依次查找字符串中与Pattern匹配的子串，一旦找到对应的子串，下次调用find()方法将接着查找。**

      通过程序运行结果可以看出，使用正则表达式可以提取网页上的电话号码，也可以提取网址。如果程序再进一步，可以从网页上提取超链接信息，再根据超链接打开其他网页，然后在其他网页上重复这个过程就可以实现简单的网络爬虫了。

   6. find()方法还可以传入一个int类型的参数，带int参数的find()方法将从该Int索引处向下搜索。start()和end()方法主要用于确定子串在目标中的位置。

   7. matches()和lookingAt()方法有点相似，只是matches()方法要求整个字符串和Pattern完全匹配时才返回true,而lookingAt()只要字符串以Pattern开头就会返回true。reset()方法将现有的Matcher对象应用于新的字符串序列

      ```Java
   public static void main(String[] args){
         String[] mails={
           "kongyeeku@163.com",
           "kongyeeku@gmail.com",
           "ligang@crazyit.com",
           "wanwa@abc.xx"
         };
         var mailRegEx="\\w{3,20}@\\w+\\.(com|gov|net|cn|org)";
         var mailpattern=Pattern.compile(mailRegEx);
         Matcher matcher=null;
         for (var mail:mails){
             if(matcher==null){
                 matcher=mailpattern.matcher(mail);
             }else{
                 matcher.reset(mail);
             }
             String result=mail+(matcher.matches() ? "shi" :"bushi");
             System.out.println(result);
         }
      }
      ```
      
      
      
   8. String类里也提供了matches()方法，该方法返回该字符串是否匹配指定的正则表达式
   
      ```Java
      System.out.println("kongyeeku@fdfd.com".matches("\\w{3,20}@\\w+\\.(com|gov|net|cn|org)"));
      ```
   
   9. 还可以利用正则表达式对目标字符串进行分割，查找，替换等操作
   
      ```Java
      String[] msgs= {
      		"Java has regular expressions in 1.4",
      		"regular expressions now expressing in Java",
      		"Java represses oracular expressions"
      	};
      	var p=Pattern.compile("\\bre\\w*");
      	Matcher matcher=null;
      	for (var i=0;i<msgs.length;i++) {
      		if(matcher==null) {
      			matcher=p.matcher(msgs[i]);
      		}else {
      			matcher.reset(msgs[i]);
      		}
      		System.out.println(matcher.replaceAll("哈哈"));
      		
      	}
      ```
   
      上述程序将以re开头的单词全部替换，Matcher类还提供了一个replaceFirst()，该方法只替换第一个匹配的子串。
   
      String类也提供了replaceAll(),replaceFirst(),split()方法。
   

## 12.变量处理和方法处理

### 1.  Java9增强的MethodHandle

1. MethodHandle为Java增加了方法引用功能，方法引用的概念有点类似于C的“函数指针”。**这种方法引用是一种轻量级的引用方式，它不会检查方法的访问权限，也不会管方法所属的类，实例方法或静态方法，MethodHandle就是简单代表特定的方法，并可通过MethodHandle来调用。**

2. 为了使用MethodHandle,还涉及如下几个类

   1. MethodHandles:MethodHandle的工厂类，它提供了一系列静态的方法用于获取MethodHandle

   2. MethodHandles.Lookup：Lookup静态内部类也是MethodHandle,VarHandle的工厂类，专门用于获取MethodHandle和VarHandle.

   3. MethodType:代表一个方法类型。MethodType根据方法的形参，返回值类型来确定方法的类型 

      ```Java
      public class resgs {
      private static void hello() {
      	System.out.println("Hello world");
      }
      private String hello(String name) {
      	System.out.println("执行带参数的hello"+name);
      	return name+",您好";
      }
      public static void main(String[] args) throws Throwable{
      	var type=MethodType.methodType(void.class);
          //使用MethodHandles.Lookup的findStatic获取方法
      	var mtd=MethodHandles.lookup().findStatic(resgs.class, "hello", type);
      	mtd.invoke();
      	var mtd2=MethodHandles.lookup().findVirtual(resgs.class, 
                                                      "hello",
                                                      
                                                     MethodType.methodType(String.class,String.class));//指定获取返回值为String,形参为String的方法类型 
          //通过MethodHandle执行方法，传入主调对象和形参
      	System.out.println(mtd2.invoke(new resgs(),"孙悟空"));
      	
      }
      }
      ```

      从上面可以看出MethodHandle可以让Java动态的调用某个方法。

   ## 2. Java9增加的VarHandle

   VarHandle只要用于动态操作**数组的元素**或者**对象的成员变量**。VarHandle与MethodHandle非常相似，它也需要通过MethodHandles来获取实例。

   ```Java
   public class resgs {
   public static void main(String[] args) throws Throwable{
   	var sa=new String[] {"Java","Kotlin","Go"};
   	var avh=MethodHandles.arrayElementVarHandle(String[].class);
   	//比较设置：如果第三个元素是Go,则该元素被设为Lua
   	var r=avh.compareAndSet(sa,2,"Go","Lua");
   	System.out.println(r);
   	System.out.println(Arrays.toString(sa));
   	//获取数组的第二个元素
   	System.out.println(avh.get(sa,1));
   	
   	//用findVarHandle方法获取User类中名为name,类型为String的实例变量
   	var vh1=MethodHandles.lookup().findVarHandle(User.class, "name", String.class);
   	var user=new User();
   	System.out.println(vh1.get(user));//输出为null
   	vh1.set(user,"孙悟空");
   	System.out.println(user.name);
   
   }
   }
   class User{
   	String name;
   	static int MAX_AGE;
   }
   ```

## 13.Java 11改进国际化与格式化

1. Java国际化主要通过如下三个类完成

   * java.util.ResourceBundle:用于加载国家，语言资源包。
   * java.util.Locale:用于封装特定的国家/区域，语言环境。
   * java.text.MessageFormat:用于格式化带占位符的字符串

2. 为了实现程序的国际化，必须先提供程序所需要的资源文件。资源文件的内容是很多的key-value形式，其中key是程序使用的部分，而value则是程序界面显示字符串。

   资源文件明明可以有如下三种形式：

   * baseName_language_country.properties
   * baseName_language.properties
   * baseName.properies

   其中baseName是资源文件基本名称,用户可以随意的指定；而language和country都不可以随意的变化，必须是Java所支持的语言和国家

3. Java支持的语言和国家

   ```Java
   public static void main(String[] args) {
   
   	Locale[] localeList=Locale.getAvailableLocales();
   	for(var i=0;i<localeList.length;i++) {
   		System.out.println(localeList[i].getDisplayCountry()+"="+localeList[i].getCountry()+"="
   	+localeList[i].getDisplayLanguage()+localeList[i].getLanguage());
   		
   	}
   
   
   }
   
   ```

   

4. 完成国际化

   1. 创建第一个文件：mess_zh_CN.properties

      ```
      hello=你好!
      ```

   2. 创建第二个文件:mess_en_US.properties

      ```
      hello=Welcome You!
      ```

   3. ```Java
      public static void main(String[] args) {
      
      	var myLocale=Locale.getDefault(Locale.Category.FORMAT);
      	var bundle=ResourceBundle.getBundle("mess",myLocale);
      	System.out.println(bundle.getString("hello"));
      }
      ```

   4. Java程序国际化的关键是ResourceBundle，它有一个静态的方法:getBundle(String baseName,Locale locale)，该方法将根据Locale加载资源文件，而Locale封装了一个国家，语言。

   5. 不同国家，语言环境的资源文件的baseName是相同的，即baseName为mess的资源文件有很多个，不同国家，语言环境对应不同的资源文件。

      

      ```Java
      var bundle=ResourceBundle.getBundle("mess",myLocale);
      ```

      上述代码会加载baseNmae为mess的系列资源文件之一，到底加载哪一个资源文件，则取决于myLocale

   6. 使用MessageFormat处理包含占位符的字符串

      1. 提供一个myMess_en_US.properties文件，该文件的内容如下：

         ```
         msg=Hello,{0}!Today is {1}
         ```

      2. 提供一个myMess_zh_CN.properties文件，该文件的内容如下

         ```
         msg=你好,{0}!今天是{1}
         ```

      3. 档程序直接使用ResourceBundle的getString()方法取出Msg对应的字符串时，在简体中文环境下 你好,{0}!今天是{1} 这显然不是需要的结果，程序还需要为{0}和{1}两个占位符赋值。此时需要使用MessageFormat类,该类包含一个有用的静态方法

      format(String pattern,Object...values):返回后面的多个参数值填充前面的Pattern字符串，其中借助于上面的MessageFormat类的帮助，将国际化程序修改为如下的形式public static void main(String[] args) {

      ```Java
      Locale currentLocal=null;
      if(args.length==2) {
      	currentLocal=new Locale(args[0],args[1]);
      }else {
      	currentLocal=Locale.getDefault(Locale.Category.FORMAT);
      }
      var bundle=ResourceBundle.getBundle("myMess",currentLocal);
      var msg=bundle.getString("msg");
      System.out.println(MessageFormat.format(msg, "yeeku",new Date()));
      ```

   7. Java允许使用类文件来代替资源文件，必须满足以下条件

      * 该类的名字必须是baseName_language_country，这与属性文件命名相似

      * 该类必须继承ListResourceBundle，并重写getContents()方法，该方法返回Object数组，该数组的每一项都是key-value对

        ```Java
        package chapter07;
        import java.util.ListResourceBundle;
        
        public class myMess_zh_CN extends ListResourceBundle{
        
        	
        	private final Object myData[][]= {
        			{
        				"msg","{0} hello today is {1}"
        			}	
        	};
        	@Override
        	protected Object[][] getContents() {
        		return myData;
        	}
        }
        
        ```

        

        ```Java
        public static void main(String[] args) {
        
        	myMess_zh_CN s=new myMess_zh_CN();
        	Locale currentLocal=null;
        	if(args.length==2) {
        		currentLocal=new Locale(args[0],args[1]);
        	}else {
        		currentLocal=Locale.getDefault(Locale.Category.FORMAT);
        	}
        	//注意此处的的包名称，此类在chapter07的包下边
        	var bundle=ResourceBundle.getBundle("chapter07.myMess",currentLocal);
        	var msg=bundle.getString("msg");
        	System.out.println(MessageFormat.format(msg, "yeeku",new Date()));
        }
        
        ```

      * 如果资源文件和java类文件同事存在他们的加载顺序如下

        1. baseName_zh_CN.class
        2. baseName_zhu_CN.properties
        3. baseName_zh.class
        4. baseName_zh.properties
        5. baseName.class
        6. baseName.properties

        如果以上的文件都不存在则会抛出异常。

## 14. Java9新增的日志API（这一节有问题，代码错误）

1. 这套日志API的用法非常的简单，只要两步就行

   1. 调用System类的getLogger(String name)方法获取System.Logger对象

   2. System.Logger对象的log()方法输出日志。该方法的第一个参数用于指定日志级别

      为了与传统的Java.util.logging日志级别，主流日志框架的级别兼容，Java9定义了如下级别![image-20201105112110036](./pic/image-20201105112110036.png)

      该日志级别是一个非常有用的东西，在开发阶段调试程序的时候，可能需要大量输出调试信息，在发布软件时候，又希望关掉这些调试信息。此时就可以通过日志来实现，只要将系统日志级别调高，所有低于该级别的日志信息都会被自动关闭。

## 15. 使用NumberFormat格式化数字

1. MessageFormat是抽象类Fromat的子类，Format抽象类还有两个子类:NumberFormat和DateFormat，它们分别用以实现数值，日期的格式化。NumberFormat,DateForamt,它们分别用以实现数值，日期的格式化。NumberFormat,DateFormat可以将数值，日期转换成字符串，也可以将字符穿转换成数值，日期。

2. NumberFormat和DateFormat都包含了format()和parse()方法，其中format()用于将数值，日期格式化成字符串，parse()用于将字符串解析成数值，日期。

3. NumberFormat是一个抽象的类，所以无法通过它的构造器来创建NumberFormat对象，它提供了如下几个方法得到NumberFormat对象

   1. getCurrentInstance()：返回默认Locale的货币格式器。也可以在调用该方法的时候传入指定的Locale获取指定的Locale的货币器

   2. getIntegerInstance():返回默认Locale的整数格式器

   3. getNumberInstance():返回默认的Locale的通用数值格式器。也可以在调用该方法时传入指定的Locale，则获取指定Locale的通用数值格式器

   4. getPercentInstance():返回默认Locale的百分数格式器。也可以在调用方法的时候传入指定Locale，则获取指定Locale的百分数格式器。

      一旦取得了NumberFormat对象之后，就可以调用它的format()方法来格式化数值，包括整数和浮点数。
   
      ```Java
      //需要被转化的数字
      	var db=123400.567;
      	//创建不同的Locale
      	Locale[] locales = {
      			Locale.CHINA,Locale.JAPAN,Locale.GERMAN,Locale.US
      	};
      	var nf=new NumberFormat[12];
      	//为上面的四个Locale创建12个NumberFormat对象
      	//每个Locale分别有通用的数值格式器，比分数格式器，货币格式器
      	for (var i=0;i<locales.length;i++){
      		nf[i*3]=NumberFormat.getNumberInstance(locales[i]);
      		nf[i*3+1]=NumberFormat.getPercentInstance(locales[i]);
      		nf[i*3+2]=NumberFormat.getCurrencyInstance(locales[i]);
      	}
      	for(var i=0;i<locales.length;i++) {
      		var tip= i==0 ? "---中国的格式---":i==1? "---日本的格式---":i==2? "---德国的格式---":"---美国的格式---";
      		System.out.println(tip);
      		System.out.println("数值格式"+nf[i*3].format(db));
      		System.out.println("百分数"+nf[i*3+1].format(db));
      		System.out.println("货币格式"+nf[i*3+2].format(db));
      	}
      ```

## 16 .使用DateFormat格式化日期，时间

1. 余与NumberFormat相似的是，DateFormat也是一个抽象的类，它提供了如下几个类方法用于获取DateFormat对象

   1. getDateInstance():返回一个日期格式器，它格式化后的字符串只有日期，没有时间。该方法可以传入多个参数，用于指定日期样式和Locale等参数；如果不指定这些参数，则使用默认参数。
   2. getTimeInstance():返回一个时间格式器，它格式化后的字符串只有时间，没有日期。该方法可以传入多个参数，用于指定时间样式和Locale等参数；如果不指定这些参数，则使用默认参数。
   3. getDateTimeInstance():返回一个日期，时间格式器，它格式化后的字符串既有日期，也有时间。该方法可以传入多个参数，用于指定日期样式，时间样式和Locale等参数；如果不指定这些参数，则使用默认参数。

   ```Java
   
   	var dt=new Date();
   	Locale[] locales={Locale.CHINA,Locale.US};
   	var df =new DateFormat[16];
   	for(var i=0;i<locales.length;i++) {
   		df[i*8]=DateFormat.getDateInstance(DateFormat.SHORT,locales[i]);
   		df[i*8+1]=DateFormat.getDateInstance(DateFormat.MEDIUM,locales[i]);
   		df[i*8+2]=DateFormat.getDateInstance(DateFormat.LONG,locales[i]);
   		df[i*8+3]=DateFormat.getDateInstance(DateFormat.FULL,locales[i]);
   		df[i*8+4]=DateFormat.getTimeInstance(DateFormat.SHORT,locales[i]);
   		df[i*8+5]=DateFormat.getTimeInstance(DateFormat.MEDIUM,locales[i]);
   		df[i*8+6]=DateFormat.getTimeInstance(DateFormat.LONG,locales[i]);
   		df[i*8+7]=DateFormat.getTimeInstance(DateFormat.FULL,locales[i]);
   	}
   	for (var i=0;i<locales.length;i++) {
   		var tip= i==0?"---中国的日期格式---":"---美国的日期格式---";
   		System.out.println(tip);
   		System.out.println("SHORT格式的日期格式"+df[i*8].format(dt));
   		System.out.println("MEDIUM格式的日期格式"+df[i*8+1].format(dt));
   		System.out.println("LONG格式的日期格式"+df[i*8+2].format(dt));
   		System.out.println("FULL格式的日期格式"+df[i*8+3].format(dt));
   		System.out.println("SHORT格式的时间格式"+df[i*8+4].format(dt));
   		System.out.println("MEDIUM格式的时间格式"+df[i*8+5].format(dt));
   		System.out.println("LONG格式的时间格式"+df[i*8+6].format(dt));
   		System.out.println("FULL格式的时间格式"+df[i*8+7].format(dt));
   	}
   ```

   ```
   输出为
   ---中国的日期格式---
   SHORT格式的日期格式2020/11/5
   MEDIUM格式的日期格式2020年11月5日
   LONG格式的日期格式2020年11月5日
   FULL格式的日期格式2020年11月5日星期四
   SHORT格式的时间格式下午4:37
   MEDIUM格式的时间格式下午4:37:38
   LONG格式的时间格式CST 下午4:37:38
   FULL格式的时间格式中国标准时间 下午4:37:38
   ---美国的日期格式---
   SHORT格式的日期格式11/5/20
   MEDIUM格式的日期格式Nov 5, 2020
   LONG格式的日期格式November 5, 2020
   FULL格式的日期格式Thursday, November 5, 2020
   SHORT格式的时间格式4:37 PM
   MEDIUM格式的时间格式4:37:38 PM
   LONG格式的时间格式4:37:38 PM CST
   FULL格式的时间格式4:37:38 PM China Standard Time
   ```

4. DateFormat的parse()方法可以把一个字符串解析成Date对象，但它要求被解析的字符串必须符合日期的字符串的要求，否则可能会抛出ParseException

```Java
var str1="2020/11/5";
 var str2="2020年11月05日";
 System.out.println(DateFormat.getDateInstance().parse(str2));//Thu Nov 05 00:00:00 CST 2020
 System.out.println(DateFormat.getDateInstance(DateFormat.SHORT).parse(str1));//Thu Nov 05 00:00:00 CST 2020
 System.out.println(DateFormat.getDateInstance().parse(str1));//触发异常
```

5. **SimpleDateFormat格式化日期**！！！

   1. SimpleDateFormat是DateFormat的子类，它可以非常灵活的格式化Date，也可以解析各种格式的日期字符串。创建SimpleDateFormat对象时需要传入一个Pattern字符串，整个pattern不是正则表达式，而是一个日期模板字符串

      ```Java
      var d=new Date();
        var sdf1=new SimpleDateFormat("Gyyyy年中的第D天");
        var dateStr=sdf1.format(d);
        System.out.println(dateStr);
        var str="14###3##21";
        var sdf2=new SimpleDateFormat("yy###M##dd");
        System.out.println(sdf2.parse(str));
      ```

      

## 17. java8新增的日期，时间格式器

Java8新增的日期，时间API里不仅包含了Instant，LocalDate,LocalDateTime,LocalTime等代表日期，时间的类，而且在java.time.format包下提供了一个DateTimeFormatter格式器，该类相当于前面介绍的DateFormat和SimpleDateFromat的合体，功能非常的强大。

与DateFormat，SimpleDateFormat类似，DateTimeFormatter不仅可以将日期，时间对象格式化成字符串，也可以将特定格式的字符串解析成日期，时间对象。

获取DateTimeFormatter对象有如下三种方式

* 直接使用静态常量创建DateTimeFormatter格式器。DateTimeFormatterf类中包含了大量形如ISO_LOCAL_DATE、ISO_LOCAL_TIMEI,ISO_LOCAL_DATE_TIME等静态常量，这些静态常量本身就是DateTimeFormatter实例。
* 使用代表不同风格的枚举值来创建DateTimeFormatter格式器。在FormatStyle枚举类中定义了FULL,LONG,MEDIUM,SHORT四个枚举值，他们代表日期，时间格式的不同风格

### 17.1使用DateTimeForMatter完成格式化

使用DateTimeFormatter将日期，时间（LocalDate，LocalDateTime,LocalTime等实例）格式化为字符串，可以通过如下两种方式。

* 调用LocalDate,LocalDateTime,LocalTime等日期，时间对象的format(DateTimeFormatter formatter)方法执行格式化。

* 上面两种方式的功能相同，用法也基本相似。

  ```Java
  var formatters = new DateTimeFormatter[] {
          // 直接使用常量创建DateTimeFormatter格式器
          DateTimeFormatter.ISO_LOCAL_DATE,
          DateTimeFormatter.ISO_LOCAL_TIME,
          DateTimeFormatter.ISO_LOCAL_DATE_TIME,
          // 使用本地化的不同风格来创建DateTimeFormatter格式器
          DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL, FormatStyle.MEDIUM),
          DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG),
          // 根据模式字符串来创建DateTimeFormatter格式器
          DateTimeFormatter.ofPattern("Gyyyy%%MMM%%dd HH:mm:ss")
  };
  var date = LocalDateTime.now();
  // 依次使用不同的格式器对LocalDateTime进行格式化
  for (var i = 0; i < formatters.length; i++)
  {
    // 下面两行代码的作用相同
    System.out.println(date.format(formatters[i]));
    System.out.println(formatters[i].format(date));
  }
  ```

### 17.2使用DateTiemFormatter解析字符串

为了使用DateTimeFormatter将指定格式的字符串解析成日期，时间对象（LocalDate,LocalDateTime,LocalTeime等实例）可通过日期，时间对象提供的parse(CharSequece text,DateTimeFormatter formatter)方法进行解析。

```Java
var str1="2014==04==12 01时06分09秒";
var formatter1=DateTimeFormatter.ofPattern("yyyy==MM==dd HH时mm分ss秒");
var dt1=LocalDateTime.parse(str1,formatter1);
System.out.println(dt1);
```

 

