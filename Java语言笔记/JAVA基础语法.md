## JRE与JDK

JRE： Java运行时环境，包含了java虚拟机，java基础类库。是使用java语言编写的程序运行所需要的软件环境，是提供给想运行java程序的用户使用的。
JDK：Java开发工具包，是程序员使用java语言编写java程序所需的开发工具包。JDK包含了JRE，同时还包含了编译java源码的编译器javac，还包含了很多java程序调试和分析的工具：jconsole，jvisualvm等工具软件，还包含了java程序编写所需的文档和demo例子程序。

## Java8 新增了非常多的特性，我们主要讨论以下几个

- **Lambda 表达式** − Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中。
- **方法引用** − 方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。
- **默认方法** − 默认方法就是一个在接口里面有了一个实现的方法。
- **新工具** − 新的编译工具，如：Nashorn引擎 jjs、 类依赖分析器jdeps。
- **Stream API** −新添加的Stream API（java.util.stream） 把真正的函数式编程风格引入到Java中。
- **Date Time API** − 加强对日期与时间的处理。
- **Optional 类** − Optional 类已经成为 Java 8 类库的一部分，用来解决空指针异常。



Lambda 表达式的结构
让我们了解一下 Lambda 表达式的结构。
一个 Lambda 表达式可以有零个或多个参数
参数的类型既可以明确声明，也可以根据上下文来推断。例如：(int a)与(a)效果相同
所有参数需包含在圆括号内，参数之间用逗号相隔。例如：(a, b) 或 (int a, int b) 或 (String a, int b, float c)
空圆括号代表参数集为空。例如：() -> 42
当只有一个参数，且其类型可推导时，圆括号（）可省略。例如：a -> return a*a
Lambda 表达式的主体可包含零条或多条语句
如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。匿名函数的返回类型与该主体表达式一致
如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空

Lambda 表达式与匿名类的区别
使用匿名类与 Lambda 表达式的一大区别在于关键词的使用。对于匿名类，关键词 this 解读为匿名类，而对于 Lambda 表达式，关键词 this 解读为写就 Lambda 的外部类。
Lambda 表达式与匿名类的另一不同在于两者的编译方法。Java 编译器编译 Lambda 表达式并将他们转化为类里面的私有函数，它使用 Java 7 中新加的 invokedynamic 指令动态绑定该



什么是函数式接口
在 Java 中，Marker（标记）类型的接口是一种没有方法或属性声明的接口，简单地说，marker 接口是空接口。相似地，函数式接口是只包含一个抽象方法声明的接口。
java.lang.Runnable 就是一种函数式接口，在 Runnable 接口中只声明了一个方法 void run()，相似地，ActionListener 接口也是一种函数式接口，我们使用匿名内部类来实例化函数式接口的对象，有了 Lambda 表达式，这一方式可以得到简化。
每个 Lambda 表达式都能隐式地赋值给函数式接口，例如，我们可以通过 Lambda 表达式创建 Runnable 接口的引用。





## 重载与重写

### 重载的规则

1. 在使用重载时只能通过**相同的方法名**、**不同的参数形式**实现。不同的参数类型可以是**不同的参数类型**，**不同的参数个数**，**不同的参数类型的顺序**；
2. 不能通过访问权限、返回类型、抛出的异常进行重载；
3. 方法的异常类型和数目不会对重载造成影响；

### 重写的规则

1. 重写方法的参数列表必须完全与被重写的方法的相同,否则不能称其为重写而是重载.
2. 重写方法的访问修饰符一定要大于被重写方法的访问修饰符（public>protected>default>private）。
3. 重写的方法的返回值类型必须和被重写的方法的返回一致；
4. 重写的方法所抛出的异常必须和被重写方法的所抛出的异常一致，或者是其子类；
5. 被重写的方法不能为private，否则在其子类中只是新定义了一个方法，并没有对其进行重写。
6. 静态方法不能被重写为非静态的方法（会编译出错）。





## JAVA中的接口

接口：是抽象方法的集合，接口通常以interface来声明。接口只描述所应该具备的方法，即只是	定义功能和行为规范，并没有具体实现，具体的实现由接口的实现类(相当于接口的子类)来完成。这样将功能的定义与实现分离，优化了程序设计。接口中的方法均为公共访问的抽象方法。接口中无法定义普通的成员变量。



### 接口和抽象类的区别

相同点:
1. 都位于继承的顶端,用于被其他类实现或继承;
2. 都不能直接实例化对象;
3. 都包含抽象方法,其子类都必须覆写这些抽象方法;

区别：
1. 接口不能有构造方法，抽象类可以有。因为构造方法是用来在对象初始化前对对象进行一些预处理的，提供了实例化一个具体东西的入口。而接口只是声明而已，不需要初始化。抽象类的构造方法，是用来被子类调用的。抽象类虽然是不能直接实例化，但实例化子类的时候，就会初始化父类，不管父类是不是抽象类都，都会调用父类的构造方法
2. 接口不能有方法体，抽象类可以有。
3. 接口不能有静态方法，抽象类可以有。
4. 在接口中凡是变量必须是public static final，而在抽象类中没有要求
5. 抽象类为部分方法提供实现,避免子类重复实现这些方法,提高代码重用性;接口只能包含抽象方法;
6. 一个类只能继承一个直接父类(可能是抽象类),却可以实现多个接口;(接口弥补了Java的单继承)
7. 抽象类是这个事物中应该具备的你内容, 继承体系是一种 is..a关系，接口是这个事物中的额外内容,继承体系是一种 like..a关系

二者的选用:
1. 优先选用接口,尽量少用抽象类;
2. 需要定义子类的行为,又要为子类提供共性功能时才选用抽象类;



### Java中接口的意义

1. 需要实现多态

2. 要实现的方法(功能)不是当前类族的必要(属性).

3. 要为不同类族的多个类实现同样的方法(功能).






## 关于Set中的两个toArray()方法理解

	Object[] toArray() // 返回一个包含 set 中所有元素的数组。
	<T> T[]	toArray(T[] a) //返回一个包含此 set 中所有元素的数组；返回数组的运行时类型是指定数组的类型。

对于Set而言，它只知道它内部保存的是Object，所以默认情况下，toArray只能是返回一个由这些Object构成的Object数组出来。但程序的作者或许更清楚其内部元素的更具体的类型，因此，HashSet类提供了toArray的另一个重载版本，允许用户指定一种比Object[]更具体的数组类型，方法是传递一个用户想要的数组类型的一个数组实例进去，多长都无所谓（因此我们常常使用一个0长度的，毕竟把类型带进去就OK了），于是，toArray内部就会按照你想要的这种类型，给构造一个数组出来。这样构造出来的数组，当然是很安全地被调用者转换回那个实际的类型。


报错：Implicit super constructor Person() is undefined. Must explicitly invoke another constructor

我们假设A是B的父类，B是A的子类。 
1、如果程序员没有给类A没有提供构造函数，则编译器会自动提供一个默认的无参数的构造函数，如果用户提供了自己的构造函数，则编译器就不在提供默认的无参数构造函数。 
2、子类B实例化时会自动调用父类A的默认构造函数，所以如果A的默认的无参数的构造函数为private，则编译器会报错，而如果A没有提供默认的无参数的构造函数，而提供了其他类型的构造函数，编译器同样会报错，因为B找不到A的默认无参数构造函数。所以，我们最好给父类A提供一个无参数的构造函数。 
3、或者在B的构造函数中显示的调用父类A的有参构造函数。super（parameter）

## JAVA中的异常体系

Throwable类是所有异常或错误的超类，它有两个子类：Error和Exception，分别表示错误和异常。其中异常Exception分为运行时异常(RuntimeException)和非运行时异常，也称之为不检查异常(Unchecked Exception)和检查异常(Checked Exception)。

Java的异常(包括Exception和Error)分为可查的异常（checked exceptions）和不可查的异常（unchecked exceptions）。
可查异常（编译器要求必须处置的异常）：正确的程序在运行中，很容易出现的、情理可容的异常状况。可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常状况，就必须采取某种方式进行处理。
除了RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。
不可查异常(编译器不要求强制处置的异常):包括运行时异常（RuntimeException与其子类）和错误（Error）。

如果使用throw在方法体中抛出可查异常比如：InterruptedException，则需要在方法头部声明方法可能抛出的异常类型。程序会在throw语句后立即终止，它后面的语句执行不到，然后在包含它的所有try块中（可能在上层调用函数中）从里向外寻找含有与其匹配的catch子句的try块。



5.运行时异常和非运行时异常

(1)运行时异常都是RuntimeException类及其子类异常，如NullPointerException、IndexOutOfBoundsException等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

当出现RuntimeException的时候，我们可以不处理。当出现这样的异常时，总是由虚拟机接管。比如：我们从来没有人去处理过NullPointerException异常，它就是运行时异常，并且这种异常还是最常见的异常之一。 
出现运行时异常后，如果没有捕获处理这个异常（即没有catch），系统会把异常一直往上层抛，一直到最上层，如果是多线程就由Thread.run()抛出，如果是单线程就被main()抛出。抛出之后，如果是线程，这个线程也就退出了。如果是主程序抛出的异常，那么这整个程序也就退出了。运行时异常是Exception的子类，也有一般异常的特点，是可以被catch块处理的。只不过往往我们不对他处理罢了。也就是说，你如果不对运行时异常进行处理，那么出现运行时异常之后，要么是线程中止，要么是主程序终止。 
如果不想终止，则必须捕获所有的运行时异常，决不让这个处理线程退出。队列里面出现异常数据了，正常的处理应该是把异常数据舍弃，然后记录日志。不应该由于异常数据而影响下面对正常数据的处理。


(2)非运行时异常是RuntimeException以外的异常，类型上都属于Exception类及其子类。如IOException、SQLException等以及用户自定义的Exception异常。对于这种异常，JAVA编译器强制要求我们必需对出现的这些异常进行catch并处理，否则程序就不能编译通过。所以，面对这种异常不管我们是否愿意，只能自己去写一大堆catch块去处理可能的异常。

## Java中的泛型（Generics）

Java容器能够容纳任何类型的对象，这一点表面上是通过泛型机制完成，Java泛型不是什么神奇的东西，只是编译器为我们提供的一个“语法糖”，泛型本身并不需要Java虚拟机的支持，只需要在编译阶段做一下简单的字符串替换即可。实质上Java的单继承机制才是保证这一特性的根本，因为所有的对象都是Object的子类，容器里只要能够存放Object对象就行了。
事实上，所有容器的内部存放的都是Object对象，泛型机制只是简化了编程，由编译器自动帮我们完成了强制类型转换而已。JDK 1.4以及之前版本不支持泛型，类型转换需要程序员显式完成。

## 内部类

### 静态内部类

只有内部类可以声明为static，外部类声明为static会报错。

##### 静态内部类 VS 普通内部类

1. 静态内部类只能访问外部类的静态成员变量和方法， 普通内部类可以访问外部类的任意成员变量和方法。
2. 静态内部类可以声明任意类型的成员变量和方法，普通内部类不能声明静态的成员变量，除非用final修饰。普通内部类不能声明静态方法。
3. 静态内部类可以单独初始化，普通内部类初始化时必须先初始化外部类。



### 匿名内部类

##### 匿名内部类访问外部类方法中的局部变量

   匿名内部类不能访问外部类方法中的局部变量，除非变量被声明为final类型

1. 这里所说的“匿名内部类”主要是指在其外部类的成员方法内定义，同时完成实例化的类，若其访问成员方法中的局部变量，局部变量必须要被final修饰。
2. 原因是编译程序实现上的困难：内部类对象的生命周期会超过局部变量的生命周期。局部变量的生命周期：当该方法被调用时，该方法中的局部变量在栈中被创建，当方法调用结束时，退栈，这些局部变量全部死亡。而内部类对象生命周期与其它类一样：自创建一个匿名内部类对象，系统为该对象分配内存，直到没有引用变量指向分配给该对象的内存，它才会死亡(被JVM垃圾回收)。所以完全可能出现的一种情况是：成员方法已调用结束，局部变量已死亡，但匿名内部类的对象仍然活着。
3. 如果匿名内部类的对象访问了同一个方法中的局部变量，就要求只要匿名内部类对象还活着，那么栈中的那些它要所访问的局部变量就不能“死亡”。
4. 解决方法：匿名内部类对象可以访问同一个方法中被定义为final类型的局部变量。定义为final后，编译程序的实现方法：对于匿名内部类对象要访问的所有final类型局部变量，都拷贝成为该对象中的一个数据成员。这样，即使栈中局部变量已死亡，但被定义为final类型的局部变量的值永远不变，因而匿名内部类对象在局部变量死亡后，照样可以访问final类型的局部变量，因为它自己拷贝了一份，且与原局部变量的值始终一致。



## Java中的反射机制

反射：指在程序运行时，根据类的全限定名动态的获取类的Class对象，并通过Class对象实现对该类的各种操作的过程。

其中Class对象是在该类的加载过程中的加载阶段，由类加载器创建的，且Class对象保存在堆中。

### 示例

```java
public static void main(String[] args){
    Class<People> clazz = (Class<People>) Class.forName("com.xu.People", false, 					ClassLoader.getSystemClassLoader()); 

    Constructor<People> ctor = clazz.getConstructor(String.class);
    People people = ctor.newInstance("zhang");
    System.out.println(people);
}
```



### 常用方法

```java
//使用给定的类加载器, 根据类的全限定名获取该类的class对象
public static Class<?> forName(String name, boolean initialize,ClassLoader loader);
//返回该Class对象所代表的类的所有公共构造方法
public Constructor<?>[] getConstructors();
//返回该Class对象所代表的类的指定参数类型的构造方法，后续可以调用该构造方法对象的newInstance()方法创建该类的实例
public Constructor<T> getConstructor(Class<?>... parameterTypes);
//返回该Class对象所代表的类的所有公共字段，包括从父类中继承的
public Field[] getFields()；
//返回该Class对象所代表的类的所有声明的字段，包括public,protected,default，private。不包括从父类中继承的
public Field[] getDeclaredFields();
//返回该Class对象所代表的类的所有公共方法，包括从父类中继承或接口中实现的
public Method[] getMethods();
//返回该Class对象所代表的类的所有声明的方法，包括public,protected,default，private。不包括从父类中继承的
public Method[] getDeclaredMethods();
//返回该Class对象所代表的类的指定名称及参数类型的特定方法
public Method getDeclaredMethod(String name, Class<?>... parameterTypes)
```





Class类和Class文件的区别与联系





## Object类

### hashCode()

HashCode的存在主要是用于查找的快捷性，如Hashtable，HashMap等

重写hashCode（）方法的基本规则：
- 在程序运行过程中，同一个对象多次调用hashCode（）方法应该返回相同的值。
- 当两个对象通过equals（）方法比较返回true时，则两个对象的hashCode（）方法返回相等的值。
- 对象用作equals（）方法比较标准的Field，都应该用来计算hashCode值。

hashCode（）方法的一般方式：
1. 把对象内每个有意义的Field计算出一个int类型的hashCode值：

    Boolean	hashCode=(f?0:1)
    整数类型(byte  short   int  char)	hashCode=(int)f
    long	hashCode=(int)(f^(f>>>32))
    float	hashCode=Float.floatToIntBits(f)
    double	long l = Double.doubleToLongBits(f);
    hashCode=(int)(l^(l>>>32))
    普通引用类型	hashCode=f.hashCode()

2.用第一步计算出来的多个hashCode值组合计算出一个hashCode值返回，为了避免直接相加产生偶然相等，可以通过为各个Field乘以任意一个质数后再相加。





### 常用API

```java
public final native Class<?> getClass();
//如果equals（）方法判断两个对象相等，这两个对象的hashCode()方法肯定返回相同值。
//如果equals（）方法判断两个对象不相等，这两个对象的hashCode()方法也可能返回相同的值。
public native int hashCode();
//用来判断两个对象是否相等。在Object类的默认实现中，采用（this == obj）判断两个对象是否相等。
//由于相等对象必须具有相同的hashCode，所以重写此方法时也需要重写hashCode()方法。
public boolean equals(Object obj);
//获得一个对象的拷贝。分为浅拷贝和深拷贝。
//1. 浅拷贝：对象中的基本数据类型进行值传递，对引用数据类型进行引用间的传递，不创建新对象。
//2. 深拷贝：对象中的基本数据类型进行值传递，对引用数据类型创建新的对象并复制其内容。
//Object类中的close()方法实现的是浅拷贝。
protected native Object clone();
//返回一个对象的字符串表示。Object类中默认是getClass().getName()+"@"+ Integer.toHexString(hashCode());
public String toString();
//唤醒正在此对象监视器上等待的单个(任意)线程。该方法需要在synchronized 中调用。
public final void notify();
//唤醒等待此对象监视器的所有线程
public final void notifyAll();
//将当前线程置于此方法所属对象的等待集中，即使当前线程等待，然后放弃当前线程在该对象上的所有的锁。
//直到另一个线程调用此对象的notify()方法或notifyAll()方法，或者已经过了指定的时间量。
//该方法需要在synchronized 中调用。
public final void wait();
public final void wait(long timeoutMillis);
```

### 关于wait() 和 notify() 方法的说明

线程也可以在没有被通知，中断或超时的情况下唤醒，即所谓的虚假唤醒。为了防止虚假唤醒，wait()操作应该放在循环中。

```java
synchronized (obj) {
    while (<condition does not hold>)
        obj.wait(timeout);
    ... // Perform action appropriate to condition
}
```

#### 使用wait()、notify() 的生产者-消费者模型

```java
private static final Integer EMPTY = 0;
private static final Integer FULL = 10;
private static final Object LOCK = new Object();
Queue queue = new ArrayDeque(10);

class Producer implements Runnable {
    @Override
    public void run() {
        while (true) {
            synchronized (LOCK) {
                while (queue.size() == FULL) {
                    LOCK.wait();      
                }
                queue.add(new Object());
                System.out.println("生产者生产，目前总共有" + queue.size());
                LOCK.notifyAll();
            }
        }
    }
}

class Consumer implements Runnable {
    @Override
    public void run() {
        while (true) {
            synchronized (LOCK) {
                while (queue.size() == EMPTY) {
                    LOCK.wait();   
                }
                queue.poll();
                System.out.println("消费者消费，目前总共有" + queue.size());
                LOCK.notifyAll();
            }
        }
    }
}

```



1. 为什么wait() 和 notify() 方法属于Object类？

   因为这一对方法阻塞时要释放占用的锁，而锁是任何对象都具有的，调用任意对象的 wait() 方法导致线程阻塞，并且该对象上的锁被释放。而调用 任意对象的notify()方法则导致因调用该对象的 wait() 方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。

2. 为什么wait() 和 notify() 方法却必须在 synchronized 方法或块中调用？

   只有在synchronized 方法或块中当前线程才占有锁，才有锁可以释放。同样的道理，调用这一对方法的对象上的锁必须为当前线程所拥有，这样才有锁可以释放。因此，这一对方法调用必须放置在这样的 synchronized 方法或块中，该方法或块的上锁对象就是调用这一对方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现 IllegalMonitorStateException 异常。

3. 关于 wait() 和 notify() 方法最后再说明两点：
   第一：调用 notify() 方法导致解除阻塞的线程是从因调用该对象的 wait() 方法而阻塞的线程中随
   机选取的，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问
   题。

   第二：除了 notify()，还有一个方法 notifyAll() 也可起到类似作用，唯一的区别在于，调用
   notifyAll() 方法将把因调用该对象的 wait() 方法而阻塞的所有线程一次性全部解除阻塞。当然，只有
   获得锁的那一个线程才能进入可执行状态。





## String、StringBuffer、StringBuilder

### String

String 是是线程安全的不可变字符串。因为每次对 String 类型变量的修改都是生成新的 String 对象，所以String 是线程安全的。

### StringBuffer

StringBuffer是线程安全的可变字符序列。StringBuffer 所有的方法都用 synchronized 修饰。StringBuffer继承自AbstractStringBuilder类，AbstractStringBuilder底层是byte数组。StringBuffer类的操作都是代理到父类的相应方法。

### StringBuilder

StringBuilder是非线程安全的。StringBuilder也继承自AbstractStringBuilder类，相应的操作都是代理到父类的相应方法。



## Thread类



