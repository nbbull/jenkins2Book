## 一、Groovy简介

Groovy是用于Java虚拟机的一种敏捷的动态语言，它是一种成熟的面向对象编程语言，既可以用于面向对象编程，又可以用作纯粹的脚本语言。使用该种语言不必编写过多的代码，同时又具有闭包和动态语言中的其他特性。

Groovy是JVM的一个替代语言（替代是指可以用 Groovy 在Java平台上进行 Java 编程），使用方式基本与使用 Java代码的方式相同，该语言特别适合与Spring的动态语言支持一起使用，设计时充分考虑了Java集成，这使 Groovy 与 Java 代码的互操作很容易。（注意：不是指Groovy替代java，而是指Groovy和java很好的结合编程。

#### 1.1 基本特点

1、构建在强大的Java语言之上 并 添加了从Python，Ruby和Smalltalk等语言中学到的 诸多特征，例如动态类型转换、闭包和元编程（metaprogramming）支持。

2、为Java开发者提供了 现代最流行的编程语言特性，而且学习成本很低（几乎为零）。

3、支持DSL（Domain Specific Languages领域定义语言）和其它简洁的语法，让代码变得易于阅读和维护。

4、受检查类型异常\(Checked Exception\)也可以不用捕获。

5、Groovy拥有处理原生类型，面向对象以及一个Ant DSL，使得创建Shell Scripts变得非常简单。

6、在开发Web，GUI，数据库或控制台程序时 通过 减少框架性代码 大大提高了开发者的效率。

7、支持单元测试和模拟（对象），可以 简化测试。

8、无缝集成 所有已经存在的 Java对象和类库。

9、直接编译成Java字节码，这样可以在任何使用Java的地方 使用Groovy。

10、支持函数式编程，不需要main函数。

11、一些新的运算符。

12、默认导入常用的包。

13、断言不支持jvm的-ea参数进行开关。

14、支持对对象进行布尔求值。

15、类不支持default作用域，且默认作用域为public。

16、groovy中基本类型也是对象，可以直接调用对象的方法。

#### 1.2 动态类型

类型对于变量，属性，方法，闭包的参数以及方法的返回类型都是可有可无的，都是在给变量赋值的时候才决定它的类型， 不同的类型会在后面用到，任何类型都可以被使用,即使是基本类型 \(通过自动包装（autoboxing）\). 当需要时，很多类型之间的转换都会自动发生，比如在这些类型之间的转换: 字符串（String），基本类型\(如int\) 和类型的包装类 \(如Integer\)之间，可以把不同的基本类型添加到同一数组（collections）中。

### 1.3 闭包

闭包就是可以使用参数的代码片段，每个闭包会被编译成继承groovy.lang.Closure类的类，这个类有一个叫call方法，通过该方法可以传递参数并调用这个闭包.它们可以访问并修改在闭包创建的范围内的变量，在闭包内创建的变量在闭包被调用的范围内同样可以被引用， 闭包可以保存在变量中并被作为参数传递到方法中。

#### 1.4 语法

Groovy 语法与Java 语言的语法很相似，虽然 Groovy 的语法源于Smalltalk和Ruby这类语言的理念，但是可以将它想像成 Java 语言的一种更加简单、表达能力更强的变体。（在这点上，Ruby与 Groovy 不同，因为它的语法与 Java 语法差异很大。）

许多 Java 开发人员喜欢 Groovy 代码和 Java 代码的相似性。从学习的角度看，如果知道如何编写 Java 代码，那就已经了解 Groovy 了。Groovy 和 Java 语言的主要区别是：完成同样的任务所需的 Groovy 代码比 Java 代码更少。

#### 1.5 类

Groovy类和java类一样，完全可以用标准java bean的语法定义一个Groovy类。但作为另一种语言，可以使用更Groovy的方式定义类，这样的好处是，可以少写一半以上的javabean代码。

（1）不需public修饰符

如前面所言，Groovy的默认访问修饰符就是public，如果Groovy类成员需要public修饰，则根本不用写它。

（2）不需要类型说明

同样前面也说过，Groovy也不关心变量和方法参数的具体类型。

（3）不需要getter/setter方法

在很多ide（如eclipse）早就可以为程序员自动产生getter/setter方法了，在Groovy中，不需要getter/setter方法--所有类成员（如果是默认的public）根本不用通过getter/setter方法引用它们（当然，如果一定要通过getter/setter方法访问成员属性，Groovy也提供了它们）。

（4）不需要构造函数

不再需要程序员声明任何构造函数，因为实际上只需要两个构造函数（1个不带参数的默认构造函数，1个只带一个map参数的构造函数--由于是map类型，通过这个参数可以构造对象时任意初始化它的成员变量）。

（5）不需要return

Groovy中，方法不需要return来返回值。

（6）不需要（）

Groovy中方法调用可以省略（）（构造函数除外）。

## 二、Groovy基础使用

#### 2.1 Groovy和Java对比

* Groovy 的松散的 Java 语法允许省略分号和修改符。
* 除非另行指定，Groovy 的所有内容都为 `public`。
* Groovy 允许定义简单脚本，同时无需定义正规的 `class`对象。
* Groovy 在普通的常用 Java 对象上增加了一些独特的方法和快捷方式，使得它们更容易使用。
* Groovy 语法还允许省略变量类型。
* **关于闭包:**
  可以将_闭包_ 想像为一个代码块，可以现在定义，以后再执行。可以使用这些强大的构造做许多漂亮的事，不过最著名的是简化迭代。使用 Groovy 之后，就有可能再也不需要编写 `Iterator`实例了。
* **动态的 Groovy:** 从技术上讲，Groovy 可能是您最近听说过的类型最松散的动态语言之一。从这个角度讲，Groovy 与 Java 语言的区别很大，Java 语言是一种固定类型语言。在 Groovy 中，类型是可选的，所以您不必输入 `String myStr = "Hello";` 来声明 `String` 变量。可以直接使用def进行不指定类型定义，类似于js中的var。

* **与Java互用：**用 Groovy 编写的任何内容都可以编译成标准的 Java 类文件并在 Java 代码中重用。类似地，用标准 Java 代码编写的内容也可以在 Groovy 中重用。

### 2.2 实例演示Java和Groovy的区别

**用 Java 编写的 Hello World**

用 Java 编写的典型的 Hello World 示例如下所示：

```java
public class HelloWorld {
  public static void main(String[] args) {    
    System.out.println("Hello World!");
  }
}
```

**编译和运行 Java 示例**

在这个简单的 `HelloWorld` 类中，我省略了包，而且向控制台输出的时候没有使用任何多余的编码约定。下一步是用 `javac` 编译这个类，如下所示：

```
c:>javac HelloWorld.java
```

最后，运行经过编译的类：

```
c:>java HelloWorld
```

迄今为止还不错 — 很久以前就会编这么基础的代码了，所以这里只是回顾一下。下面，请看用 Groovy 编码的相同过程。

**用 Groovy 编写的 Hello World**

就像前面提到过的，Groovy 支持松散的 Java 语法 — 例如，不需要为打印 “Hello World!” 这样的简单操作定义类。

而且，Groovy 使日常的编码活动变得更容易，例如，Groovy 允许输入 `println`，而无需输入 `System.out.println`。当您输入 `println`时，Groovy 会非常聪明地知道您指的是 `System.out`。

所以，用 Groovy 编写 Hello World 程序就如下面这样简单：

```groovy
println "Hello World!"
```

请注意，在这段代码周围没有类结构，而且也没有方法结构！我还使用 `println` 代替了 `System.out.println`。

**运行 Groovy 示例**

假设我将代码保存在文件/home/amosli/developsoft/language/groovy/test/Hello.groovy 内，只要输入以下代码就能运行这个示例：

```bash
amosli@amosli-ThinkPad:~/developsoft/language/groovy/groovy-2.3.6/bin$ ./groovy../../test/Hello.groovy

Hello World!
```

如果已经配置了groovy的环境变量，那么只需要输入以下命令即可：

```bash
root@amosli-ThinkPad:/home/amosli/developsoft/language/groovy/test#groovy Hello.groovy 
Hello World!
```

在控制台上输出 “Hello World!” 所需的工作就这么多。

**更快捷的方式**

```bash
amosli@amosli-ThinkPad:~/developsoft/language/groovy/groovy-2.3.6/bin$./groovy -e "println 'helloworld '"

helloworld
```

如果已经配置了groovy的环境变量，那么只需要输入以下命令即可：

```bash
root@amosli-ThinkPad:/home/amosli/developsoft/language/groovy/test#groovy -e "println 'helloworld '"
helloworld
```

这会生成相同的结果，而且甚至**无需定义任何文件**！

### 2.3 Groovy 是没有类型的 Java 代码

很可能将 Groovy 当成是没有规则的 Java 代码。但实际上，Groovy 只是规则少一些。这一节的重点是使用 Groovy 编写 Java 应用程序时可以不用考虑的一个 Java 编程的具体方面：类型定义。

**为什么要有类型定义？**

在 Java 中，如果要声明一个 `String` 变量，则必须输入：

```java
String value = "Hello World";
```

但是，如果仔细想想，就会看出，等号右侧的字符已经表明 `value` 的类型是 `String`。所以，Groovy 允许省略 `value` 前面的 `String` 类型变量，并用 `def` 代替。

```groovy
def value = "Hello World"
```

实际上，Groovy 会根据对象的值来判断它的类型。

**运行程序！**

将 HelloWorld.groovy 文件中的代码编辑成下面这样：

```java
String message = "Hello World"

println message.class    //class java.lang.String
```

### 2.4 通过 Groovy 进行循环

方式1：

这里可以定义i为int 或者 def ，或者不定义其类型

```groovy
for(i = 0; i <5; i++){
  println i
 }
```

方式2：

使用in进行循环，其中..表示“到”，**0..5**表示0到5，**类似于0&lt;=5;这里循环6次；**

```groovy
for
(i in 0..5){
  println i
 }
```

可以使用0..&lt;5进行限定，类似于0&lt;5,循环5次;

### 2.5 Groovy中的集合

#### 1\)Groovy  中的List

```groovy
def range = 0..4
println range.class
assert range instanceof List
```

请注意，`assert` 命令用来证明范围是 `java.util.List` 的实例。接着运行这个代码，证实该范围现在是类型 `List` 的集合。

Groovy 的语法:

```groovy
def coll = ["Groovy", "Java", "Ruby"]
assert  coll instanceof Collection
assert coll instanceof ArrayList
```

你将会注意到，coll对象看起来很像 Java 语言中的数组。实际上，它是一个 Collection。要在普通的 Java 代码中得到集合的相同实例，必须执行以下操作：

```groovy
Collection<String> coll = new ArrayList<String>();
coll.add("Groovy");
coll.add("Java");
coll.add("Ruby");
```

在 Java 代码中，必须使用 `add()` 方法向 `ArrayList` 实例添加项。

**而Groovy中则提供了3种方法：**

```groovy
coll.add("Python")
coll << "Smalltalk"
coll[5] = "Perl"
```

##### 查找元素:

如果需要从集合中得到某个特定项，可以通过像上面那样的位置参数获取项。例如，如果想得到第二个项 “Java”，可以编写下面这样的代码（请记住集合和数组都是从 0开始）：

```groovy
assert coll[1] == "Java"
```

Groovy 还允许在集合中增加或去掉集合，如下所示：

```groovy
def numbers = [1,2,3,4]
assert numbers + 5 == [1,2,3,4,5]
assert numbers - [2,3] == [1,4]
```

**Groovy中的特殊方法：**

```groovy
def numbers = [1,2,3,4]
assert numbers.join(",") == "1,2,3,4" 
assert [1,2,3,4,3].count(3) == 2
```

**join\(\) 和 count\(\)**只是在任何项List上都可以调用的众多方便方法中的两个。分布操作符（spread operator） 是个特别方便的工具，使用这个工具不用在集合上迭代，就能够调用集合的每个项上的方法。

假设有一个 String 列表，现在想将列表中的项目全部变成大写，可以编写以下代码：  
`assert ["JAVA", "GROOVY"] ==["Java", "Groovy"]*.toUpperCase()`  
请注意 \*. 标记。对于以上列表中的每个值，都会调用 toUpperCase\(\)，生成的集合中每个 String 实例都是大写的。

#### 2）Groovy中的Map

Java 语言中的映射是名称-值对的集合。所以，要用 Java 代码创建典型的映射，必须像下面这样操作：

```java
Map<String, String> map = new HashMap<String, String>();
map.put("name", "Andy");
map.put("VPN-#","45");
```

Groovy 使得处理映射的操作像处理列表一样简单 — 例如，可以用 Groovy 将上面的 Java 映射写成

```groovy
def hash = [name:"Andy", "VPN-#":45]
```

请注意，Groovy 映射中的键不必是 `String`。在这个示例中，`name` 看起来像一个变量，但是在幕后，Groovy 会将它变成 `String`。

验证hash格式：

```groovy
assert hash.getClass() == java.util.LinkedHashMap
```

Groovy 中Hash的Set/Get

```groovy
//方法1
hash.put("id", 23)
assert hash.get("name") == "Andy"
//方法2
hash.dob = "01/29/76"
//. 符号还可以用来获取项。例如，使用以下方法可以获取 dob 的值：
assert hash.dob == "01/29/76" 
//方法3
assert hash["name"] == "Andy"
hash["gender"] = "male"
assert hash.gender == "male"
assert hash["gender"] == "male"
```

请注意，在使用 `[]` 语法从映射获取项时，必须将项作为 `String` 引用。

### 2.6 Groovy 中的闭包

Java 的 `Iterator` 实例，用它在集合上迭代，就像下面这样：

```groovy
def acoll = ["Groovy", "Java", "Ruby"]
for(Iterator iter = acoll.iterator(); iter.hasNext();){
 println iter.next()
}
```

请注意，`each` 直接在 `acoll` 实例内调用，而 `acoll` 实例的类型是 `ArrayList`。在 `each` 调用之后，引入了一种新的语法 —`{`，然后是一些代码，然后是 `}`。由 `{}` 包围起来的代码块就是闭包。

```groovy
def acoll = ["Groovy", "Java", "Ruby"]       
acoll.each{
 println it
}
```

闭包中的 `it` 变量是一个关键字，指向被调用的外部集合的每个值 — 它是默认值，可以用传递给闭包的参数覆盖它。

下面的代码执行同样的操作，但使用自己的项变量：

```groovy
def acoll = ["Groovy", "Java", "Ruby"]
acoll.each{ value ->
 println value
}
```

在这个示例中，用 value 代替了 Groovy 的默认 it。

```groovy
def hash = [name:"Andy", "VPN-#":45]
hash.each{ key, value ->
 println "${key} : ${value}"
}
```

请注意，闭包还允许使用多个参数 — 在这个示例中，上面的代码包含两个参数（`key` 和 `value`）。

** 请记住，凡是集合或一系列的内容，都可以使用下面这样的代码进行迭代。**

```groovy
> "amosli".each{
println it.toUpperCase();
}
A
M
O
S
L
I
```

```groovy
def excite = {
word->return "this is ${word} "
};
```

这段代码是名为 excite 的闭包。这个闭包接受一个参数（名为 word），返回的 String 是 word 变量加两个感叹号。

请注意在 String 实例中替换 的用法。在 String 中使用 ${value}语法将告诉 Groovy 替换 String 中的某个变量的值。可以将这个语法当成 return word + "!!" 的快捷方式。

```groovy
//可以通过两种方法调用闭包：直接调用或者通过 call() 方法调用。
excite("Java");
excite.call("Groovy")
```

输出：this is Groovy

### 2.7 Groovy 中的类

新建一个类song:

```groovy
class Song {
 def name
 def artist
 def genre
}
```

```groovy
class SongExample {
static void main(args) {
  def sng = new Song(name:"Le Freak",artist:"Chic", genre:"Disco")
 }
}
```

Groovy 自动提供一个构造函数，构造函数接受一个名称-值对的映射，这些名称-值对与类的属性相对应。这是 Groovy 的一项开箱即用的功能 — 用于类中定义的任何属性，Groovy 允许将存储了大量值的映射传给构造函数。映射的这种用法很有意义，例如，您不用初始化对象的每个属性。

也可以添加下面这样的代码：

```groovy
def sng2 = new Song(name:"Kung Fu Fighting", genre:"Disco")
```

也可以像下面这样直接操纵类的属性：

```groovy
def sng3 = new Song()
sng3.name = "Funkytown"
sng3.artist = "Lipps Inc."
sng3.setGenre("Disco") 

assert sng3.getArtist() == "Lipps Inc."
```

在 `Song` 类中，添加以下代码：

```groovy
String toString(){
"${name}, ${artist}, ${genre}"
}
```

### ** 2.8 Groovy中的单元测试**

```groovy
   @Test
    public void test(){
        def sng2 = new Song(name:"Kung Fu Fighting", genre:"Disco")
        println sng2.getArtist();
    }
```

在Intellij 中只需要加入@Test注解就可以使用JUnit 测试

**加个?可以防止空指针的错误：**

```groovy
def getArtist(){       
 artist?.toUpperCase();
    }
```



