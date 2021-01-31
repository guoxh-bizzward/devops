# effective java 3rd

### 1.考虑使用静态工厂方法替代构造方法

一个类允许客户端获取其实例的方传统方式是 **提供一个构造方法**,其实还有另一种技术应该成为每个程序员工具箱的一部分. **一个类可以提供一个公共静态工厂方法,它只是一个返回类实例的静态方法**.

```java
/**
* Boolean的一个简单例子,此方法将boolean基本类型转换为Boolean对象引用
*/
public static Boolean valueOf(boolean b){
    return b?Boolean.TRUE:Boolean.FALSE;
}
```

注意:静态工厂方法与设计模式中的工厂方法模式不同.本条目中描述的静态工厂方法在设计模式中没有直接的等价.

* **静态工厂方法的一个优点是,不像构造方法,它们是有名字的.** 如果构造方法的参数本身并不描述被返回的对象,则具有精心选择名称的静态工厂更易于使用,并且生成的客户端更易于阅读.

  ```
  //构造方法 表示一个可能为素数的BigInteger
  BigInteger(int,int,Random)
  //静态工厂方法 JDK1.4添加
  BigInteger.probablePrime 
  ```

  

* **与构造方法不同,它们不需要每次调用时都创建一个新对象.**

* **与构造方法不同,它们可以返回其返回类型的任何子类型对象**,这为你在选择返回对象的类时提供了很大的灵活性. 这种灵活性的一个应用是API可以返回对象而不需要公开它的类.以这种方式隐藏实现类会使API非常紧凑.这种技术适用于基于接口的框架,其中接口为静态工厂方法提供自然返回类型

  

* **返回对象的类可以根据输入参数的不同而不同**,声明的返回类型的任何子类都是允许的.返回对象的类也可以随每次发布而不同

* **在编写包含该方法的类时,返回对象的类不需要存在**. 这种灵活的静态工厂方法构成了服务提供者框架的基础,比如java书库连接API(jdbc).

* **只提供静态工厂方法的主要限制是 没有公共或受保护构造方法的类不能被子类化**

* **静态工厂的第二个缺点是 程序员很难找到它们**

  下面是一些静态工厂方法的常用名称:

  * from -- A类型转化方法,它接受单个参数并返回此类型的相应实例,例如 Date  d = Date.from(instant)

  * of -- 一个聚合方法,接受多个参数并返回该类型的实例,并把它们合并在一起,例如

    ` Set<Rank> faceCards = EumSet.of(JACK,QUEEN,KING) `

  * valueof -- from和to更为详细的替代方式,例如 

    `BigInteger prime = BigInteger.valueof(Integer.MAX_VALUE)`

  * instance 或者 getInstance

  * create 或者 createInstance

  * getType

  * newType

  * type -- getType和newType 简洁的替代方式


### 2.当构造方法参数过多时使用builder模式

静态工厂和构造方法都有一个限制:它们不能很好的扩展到很多可选参数的情景.传统上,程序员使用可伸缩(telescoping construtor)构造方法模式.

```java
public class NutritionFacts {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings,0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories,0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}

```



通常情况下,这个构造方法的调用需要许多你不想设置的参数,但是你不得不为它们传递一个值.随着参数数量的增加,它很快就会失控.

简而言之,可伸缩构造方法模式是有效的,但是当很多参数的时候,很难编写客户端代码,而且很难读懂它们.读者并不知道这些值是什么意思,并且必须仔细的计算参数才能找到答案.一长串相同类型的参数可能会导致一些细微的bug,如果客户端意外的翻转了两个这样的参数,编译器无法发现,但是程序在运行的过程中会出现错误的行为.

当构造方法中存在许多可选参数的时候,另一个选择是javaBeans模式,在这种模式中,调用一个无参的构造函数来创建对象,然后调用setter方法来设置每个必须的参数和可选参数.

```java
public class NutritionFacts2 {
    private  int servingSize = -1;  // (mL)            required
    private  int servings = -1 ;     // (per container) required
    private  int calories = 0;     // (per serving)   optional
    private  int fat = 0;          // (g/serving)     optional
    private  int sodium = 0 ;       // (mg/serving)    optional
    private  int carbohydrate = 0; // (g/serving)     optional

    public NutritionFacts2(){

    }

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}

```



这种模式没有可伸缩构造方法模式的缺点,但是有点冗长,但是创建实例很容易,而且容易阅读所生成的代码.

JavaBeans模式本身也有严重的缺陷.由于构造方法在多次调用中被分割,所以在构造过程中JavaBean可能处于不一致的状态.该类没有通过检查构造参数的有效性来执行一致性的选项,在不一致的状态下尝试使用对象可能会导致与包含bug的代码大相径庭的错误,因此很难调试.

一个相关的缺点是,Javabean模式排除了让类不可变的可能性,并且需要在程序员的部分增加工作以确保线程安全.

第三个选择,它结合了可伸缩构造方法模式的安全性和javabean模式的可读性,他是Builder模式的一种形式. 客户端不直接调用所需的对象,而是调用构造方法(或静态工厂),并使用所有必须的参数,并获得一个builder对象,然后,客户端调用builder对象的setter相似方法来设置每个可选参数.最后客户端调用一个无参的build方法来生成对象,该对象通常是不可变的.Builder通常是它所构建的类的一个静态成员类.

```java
public class NutritionFacts3 {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional

    public static class Builder{
        private int servingSize;
        private int servings;

        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize,int servings){
            this.servingSize = servingSize;
            this.servings = servings;
        }
        public Builder calories(int val) {
            this.calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts3 build() {
            return new NutritionFacts3(this);
        }
    }
    private NutritionFacts3(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```



NutritionFacts3类是不可变的,所有的参数默认值都在一个地方. builder的setter方法返回builder本身,这样调用就可以被链接起来,从而生成一个流程的API,调用方式如下图

```java
NutritionFacts3 nutritionFacts3 = new NutritionFacts3.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();
```

这个代码很容易编写,更重要的是易于阅读

Builder非常适合类层次结构.使用平行层次的builder,每个嵌套在相应的类中.抽象类有抽象的builder,具体的类有具体的builder.例如,考虑代表各种披萨饼的根层次结构的抽象类:

``` java
import java.util.EnumSet;
import java.util.Objects;
import java.util.Set;

public abstract class Pizza {
    public enum Topping {HAM,MUSHROOM,ONION,PEPER,SAUSAGE}
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>>{
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping){
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza builder();

        protected  abstract T self();

    }
    Pizza(Builder<?> builder){
        toppings = builder.toppings.clone();
    }
}
```

请注意:Pizza.builder是一个带有递归类型参数(recursive type parameter )的泛型类型.这与抽象的self方法一起,允许方法链在子类中正常工作,而不需要强制转换. Java缺乏自我类型的这种变通解决方法被称为 模拟自我类型(simulated self-type)的习惯用法.

子类:

```java
public class NyPizza extends Pizza {
    public enum Size {SMALL,MEDIUM,LARGE}
    private final Size size;
    public static class Builder extends Pizza.Builder<Builder>{
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }


        @Override
        NyPizza builder() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder>{
        boolean sauceInside = false;

        public Builder sauceInside(){
            sauceInside = true;
            return this;
        }

        @Override
        Calzone builder() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        this.sauceInside = builder.sauceInside;
    }
}
```

请注意:每个子类builder中的build方法被声明为返回正确的子类: NyPizza.Builder的build方法返回NyPizza,而Calzone.Builder中的build方法返回Calzone.这种技术,其一个子类的方法被声明为返回再超类中声明的返回类型的子类型,成为协变返回类型(convariant return typing). 它允许客户端使用这些builder,而不需要强制转换.



builder对构造方法的一个微小优势是 builder可以有多个可变参数,因为每个参数都是再它自己的方法中指定的,或者 builder可以将传递给多个调用的参数聚合到单个属性中,如前面的addTopping方法所演示的那样

builder模式非常灵活,单个builder可以重复使用来构建多个对象.builder参数可以在构建方法的调用之间进行调整,以改变创建的对象. builder可以在创建对象时自动填充一些属性,例如每次创建对象时增加的序列号.

builder模式也有缺点.为了创建对象,首先必须创建他的builder.有可能在性能关键的情况下出现问题.

builder模式比伸缩构造方法模式更冗长,因此只有在有足够多的参数时才使用它,比如四个或者更多的参数,同时 最好是一开始就创建一个builder

总而言之,当设计类的构造方法或静态工厂的参数超过几个时,builder模式是一个不错的选择,特别是如果许多参数是可选或者相同类型的 客户端代码比使用伸缩构造方法更容易读写,而且builder模式比Javabeans更安全.

### 3.使用私有构造方法或枚举实现Singleton属性

单例是一个仅实例化一次的类.单例对象通常表示无状态对象,如函数或者一个本质上唯一的系统组件.让一个类沉稳给单例会使测试它的客户变得困难,因为除非实现一个作为它类型的接口,否则不可能用一个模拟实现替代单例.

有两种常见的方法来实现单例. 两者都是基于保持构造方法私有和导出公共静态成员以提供对唯一实例的访问.

在第一种方法中,成员是final修饰的属性

```java
/**
* 公共属性方法的主要优点是API明确表示该类是一个单例: 公共静态属性是final的,所有它总是包含相同的对象引用.
* 第二个好处是它更简单
*/
public class Singleton{
    public static final Singelton instance = new Singleton();
    private Singleton(){...}
    
    public void leaveTheBuilding(){...}
}
```



第二种方法中,公共成员是一个静态的工厂方法

```java
/**
*静态工厂方法的优点是  可以灵活的改变你的想法,无论该类是否为单例均无需更改其API
*第二个好处是 如果你的应用程序需要它,可以编写一个泛型单例工厂(generic singleton factory)
*第三个好处是 方法引用可以用supplier(Java 函数式编程),例如 Singleton::instance 等同于 Supplier<Singleton>
*/
public class Singleton{
    private static final Singelton INSTANCE = new Singleton();
    private Singleton(){...}
    public static Singelton getInstance(){ return INSTANCE;}
    public void leaveTheBuilding(){...}
}
```



但是要注意的是, 以上两种方法均可以通过`AccessibleObject.setAccessible`方法,以反射调用私有构造方法.如果需要防御此攻击,请修改构造函数,使其请求创建第二个实例时抛出异常.



创建一个使用这两种方法的单例类,仅仅将implement Serializable 添加到声明中是不够用的.为了维护单例的保证,声明所有的实例属性为 transient,并提供一个readResolve方法,否则,每当序列化实例被反序列时,就会创建一个新的实例

```java
private Object readResovle(){
    return INSTANCE;
}
```



实现一个单例的第三种方法就是声明单一元素的枚举类.

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```



这种方法类似于公共属性方法,但更简洁,提供了免费的序列化机智,并提供了针对多个实例化的坚固保证,即使在复杂的序列化或反射攻击的情况下.

单一元素枚举类通常是实现单例的最佳方式.注意,如果单例必须继承Enum以外的父类,那么就不能使用这种方法



### 4.使用私有构造方法执行非实例化

场景: 工具类,如 java.lang.Math,java.util.Arrays,java.util.Collections等

**试图通过创建抽象类来强制执行非实例化是行不通的**,该类可以被子类化,子类可以被实例化.此外,它误导用户认为该类是为继承而设计的.

只有当类不包含显式构造方法时,才会生成一个默认构造方法,**因此可以通过包含一个私有构造方法来实现类的非实例化**

```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

这种习惯的一个副作用是 阻止了类的子类化,所有的构造方法都必须显示或者隐式的调用父类的构造方法,而子类则没有可访问的父类构造方法.



### 5.使用依赖注入取代硬连接资源

 静态实用类和单例对于那些行为被底层资源参数化的类来说是不合适的

### 6.避免创建不必要的类

在每次需要时重用一个对象而不是创建一个新的相同功能对象通常是恰当的.重用更快更流行.如果对象是不可变条目,他总是可以被重用.

```java
//不应该这样做的极端例子
String s = new String("hello");
//改进后的版本
String s = "hello";
```



* **通过使用静态工厂方法(static factory methods),可以避免创建不需要的对象.** 例如,工厂方法 Boolean.valueOf(String)比构造方法Boolean(String)更可取,后者在Java9中被弃用.

  构造方法每次调用时都必须创建一个新对象,而工厂方法永远不需要这样做,在实践中也不需要.除了重用不可变对象,如果知道它们不会被修改,还可以重用可变对象

* **重用一些创建代价昂贵的对象**

  ```java
  /**
   *确定一个字符串是否是一个有效的罗马数字
   *
   */
  static boolean isRomanNumeral(String s) {
      return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
              + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
  }

  /**
   * 改版后
   */
  public class RomanNumerals {
      private static final Pattern ROMAN = Pattern.compile(
              "^(?=.)M*(C[MD]|D?C{0,3})"
              + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

      static boolean isRomanNumeral(String s) {
          return ROMAN.matcher(s).matches();
      }
  }
  ```

  这个实现的问题在于它依赖于String.matches方法.虽然String.matches是检查字符串是否与正则表达式匹配的最简单方法,但它不适合在性能临界的情况下重复使用.问题是在它的内部为正则表达式创建了一个Pattern实例,并且只使用一次,之后它就有资格进行垃圾收集.创建Pattern实例的代价是昂贵的,因为它需要将正则表达式编译成有限状态机(finite state machine)

  为了提高性能,作为类初始化的一部分,将正则表达式显式编译成一个Pattern实例(不可变),缓存它,并在`isRomanNumeral`方法的每个调用中重复使用相同的实例

* **自动装箱(autoboxing). 优先使用基本类型而不是装箱的基本类型,也要注意无意识的自动装箱.** 它允许程序员混用基本类型和包装的基本类型,根据需要自动装箱和拆箱.自动装箱模糊不清,但不会消除基本类型和装箱基本类型之间的区别.有微妙的语义区别和不那么细微的性能差异

  ```java
  /**
   * 结果是正确的,因为sum被定义为Long,运行结果要比实际慢很多.这意味着程序构造了大约2的31次方不必要的Long实例
   *
   */
  private static long sum(){
      Long sum = 0L;
      for(long i=0;i<=Integer.MAX_VALUE;i++){
          sum += i;
      }
      return sum;
  }
  ```

  

### 7.消除过期的对象引用

考虑以下简单的堆栈实现：

``` java
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * Ensure space for at least one more element, roughly
     * doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

这个程序没有什么明显的错误（但是对于泛型版本，请参阅条目 29）。 你可以对它进行详尽的测试，它都会成功地通过每一项测试，但有一个潜在的问题。 笼统地说，程序有一个“内存泄漏”，由于垃圾回收器的活动的增加，或内存占用的增加，静默地表现为性能下降。 在极端的情况下，这样的内存泄漏可能会导致磁盘分页（ disk paging），甚至导致内存溢出（OutOfMemoryError）的失败，但是这样的故障相对较少。

那么哪里发生了内存泄漏？ 如果一个栈增长后收缩，那么从栈弹出的对象不会被垃圾收集，即使使用栈的程序不再引用这些对象。 这是因为栈维护对这些对象的过期引用（ obsolete references）。 过期引用简单来说就是永远不会解除的引用。 在这种情况下，元素数组“活动部分（active portion）”之外的任何引用都是过期的。 活动部分是由索引下标小于size的元素组成。

垃圾收集语言中的内存泄漏（更适当地称为无意的对象保留 unintentional object retentions）是隐蔽的。 如果无意中保留了对象引用，那么不仅这个对象排除在垃圾回收之外，而且该对象引用的任何对象也是如此。 即使只有少数对象引用被无意地保留下来，也可以阻止垃圾回收机制对许多对象的回收，这对性能产生很大的影响。

这类问题的解决方法很简单：一旦对象引用过期，将它们设置为 null。 在我们的`Stack`类的情景下，只要从栈中弹出，元素的引用就设置为过期。 `pop`方法的修正版本如下所示：

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

取消过期引用的另一个好处是，如果它们随后被错误地引用，程序立即抛出`NullPointerException`异常，而不是悄悄地做继续做错误的事情。尽可能快地发现程序中的错误是有好处的。

当程序员第一次被这个问题困扰时，他们可能会在程序结束后立即清空所有对象引用。这既不是必要的，也不是可取的；它不必要地搞乱了程序。**清空对象引用应该是例外而不是规范**。消除过期引用的最好方法是让包含引用的变量超出范围。如果在最近的作用域范围内定义每个变量(条目 57)，这种自然就会出现这种情况。

那么什么时候应该清空一个引用呢？`Stack`类的哪个方面使它容易受到内存泄漏的影响？简单地说，它管理自己的内存。存储池（storage pool）由`elements`数组的元素组成(对象引用单元，而不是对象本身)。数组中活动部分的元素(如前面定义的)被分配，其余的元素都是空闲的。垃圾收集器没有办法知道这些；对于垃圾收集器来说，`elements`数组中的所有对象引用都同样有效。只有程序员知道数组的非活动部分不重要。程序员可以向垃圾收集器传达这样一个事实，一旦数组中的元素变成非活动的一部分，就可以手动清空这些元素的引用。

一般来说，**当一个类自己管理内存时，程序员应该警惕内存泄漏问题**。 每当一个元素被释放时，元素中包含的任何对象引用都应该被清除。

**另一个常见的内存泄漏来源是缓存**。一旦将对象引用放入缓存中，很容易忘记它的存在，并且在它变得无关紧要之后，仍然保留在缓存中。对于这个问题有几种解决方案。如果你正好想实现了一个缓存：只要在缓存之外存在对某个项（entry）的键（key）引用，那么这项就是明确有关联的，就可以用`WeakHashMap`来表示缓存；这些项在过期之后自动删除。记住，只有当缓存中某个项的生命周期是由外部引用到键（key）而不是值（value）决定时，`WeakHashMap`才有用。

更常见的情况是，缓存项有用的生命周期不太明确，随着时间的推移一些项变得越来越没有价值。在这种情况下，缓存应该偶尔清理掉已经废弃的项。这可以通过一个后台线程(也许是`ScheduledThreadPoolExecutor`)或将新的项添加到缓存时顺便清理。`LinkedHashMap`类使用它的`removeEldestEntry`方法实现了后一种方案。对于更复杂的缓存，可能直接需要使用`java.lang.ref`。

第三个常见的内存泄漏来源是监听器和其他回调。如果你实现了一个API，其客户端注册回调，但是没有显式地撤销注册回调，除非采取一些操作，否则它们将会累积。确保回调是垃圾收集的一种方法是只存储弱引用（weak references），例如，仅将它们保存在`WeakHashMap`的键（key）中。

因为内存泄漏通常不会表现为明显的故障，所以它们可能会在系统中保持多年。 通常仅在仔细的代码检查或借助堆分析器（ heap profiler）的调试工具才会被发现。 因此，学习如何预见这些问题，并防止这些问题发生，是非常值得的。

 ### 8.避免使用Finalizer和Cleaner机制

让你的类实现AutoCloseable接口,并要求客户在不再需要时调用每个实例的close方法,通常使用 try-with-resource确保终止,即使面对有异常抛出情况.

### 9. 使用try-with-resource语句替代 try-catch-finally语句

``` java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
           new FileReader(path))) {
       return br.readLine();
    }
}

static void copy(String src, String dst) throws IOException {
    try (InputStream   in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}

static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(
           new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}

```

在处理必须关闭的资源时,使用try-with-resource语句代替try-finally语句.生成的代码更简洁,更清晰,并且生成的异常更有用.try-with-resource语句在编写必须关闭资源的代码时会更容易,也不会出错,而使用try-finally语句实际上是不可能的

### 10.重写equals方法时遵循通用约定

重写equals方法看似简单,但是有很多方式会导致重写出错.避免此问题的最简单方法时不覆盖equals方法,在这种情况下 类的每个实例只与自身相等.如果满足以下条件,则可以不用重写equals方法:

* 每个类的实例都是固定唯一的

* 类不需要提供一个"逻辑相等(logical equality)" 的测试功能

* 父类已经重写了equals方法,则父类行为完全适合于该子类.

* 类是私有的或者包级私有的,可以确定它的equals方法永远不会调用.如果非常厌恶风险,可以重写equals方法,以确保不会被意外调用

  ```java
  @Override
  public boolean equals(Object o){
      throw new AssertionError();
  }
  ```

需要重写equals方法的情况:

如果一个类包含一个逻辑相等(logic equality)的概念,此概念有别于对象标识(Object identity),而父类还没有重写过equals方法.这通常在值类(value classes)的情况.值类是一个表示值的类,例如Integer或者String类,程序员使用equals方法比较值对象的引用,期望发生他们在逻辑上是否相等,而不是引用相同的对象.重写equals方法不仅可以满足程序员的期望,它还支持重写过的equals的实例作为Map的key,或者Set里面的元素,以满足预期和期望的行为

一种不需要重写equals方法的值类是使用实例控制(instance control)的类,以确保每个值至多存在一个对象.枚举类型属于这个类别.对于这些类,逻辑相等和对象标识是一样的,所以Object的equals方法是等价的.

重写equals方法遵守的通用约定:

* 自反性:对于任何非空引用x,x.equals(x)必须返回true
* 对称性:对于任何非空引用x和y,如果当且仅当y.equals(x)返回true时,x.equals(y)必须返回true
* 传递性:
* 一致性: x.equals(y)的多次调用必须始终返回true或者false
* 对于任何非空引用x,x.equals(null)必须返回false

编写高质量equals方法的配方:

* 使用 == 运算符检查参数是否为该对象的引用.如果是,返回true. 这只是一种性能优化,但是如果这种比较可能很昂贵的话,那就值得去做
* 使用 instanceof 运算符来检查参数是否具有正确的类型.如果不是,则返回false.通常,正确的类型是equals方法所在的类.有时候,该类实现了一些接口,如果类实现了一个接口,该接口可以改进equals约定以允许实现接口的类进行比较,那么使用接口.集合接口(如Set,List,Map和Map.Entry)具有此特性
* 参数转换成正确的类型.因为转换操作在instanceof中已经处理过了,所以他肯定会成功
* 对于类中的每个"重要"的属性,请检查该参数属性是否与该对象对应的属性相匹配.如果所有这些测试成功,返回true,否则返回false.如果步骤2中的类型是一个接口,那么必须通过接口方法访问参数的属性,如果类型是类,则可以直接访问属性,这取决于属性的访问权限

对于类型非float或double的基本类型,使用 == 运算符进行比较;

对于对象引用属性,递归的调用equals方法;

对于float基本类型的属性,使用静态的Float.compare(float,float)方法;

对于double基本类型的属性,使用Double.compare(double,double)方法.

由于存在 Float.NaN,-0.0f,+0.0f和类似的double类型的值,所以需要对float和double属性进行特殊的处理;

虽然你可以使用静态方法Float.equals和Double.equals方法对float和double的基本类型的属性进行比较,这回导致每次比较时发生自动装箱,引发非常差的性能.

对于数组属性,请将这些准则应用于每个元素,如果数组属性中的每个元素都很重要,请使用其中一个重载的Arrays.equals方法



某些对象引用的属性可能合法的包含null.为了避免出现NullPointerException异常,请使用静态方法Object.equals(Object,Object)检查这些属性是否相等



equals方法的性能可能受属性比较顺序的影响.为了获得最佳性能,你应该首先比较最可能不同的属性,开销比较小的属性,或者是最好两者都满足

不要比较不属于对象逻辑状态的属性,例如用于同步操作的lock属性;

不需要比较可以从"重要属性"计算出来的派生属性,但是这样做可以提高equals方法的性能

提醒:

1. 当重写equals方法时,同时也要重写hashCode方法
2. 不要让equals方法试图太聪明
3. 在equals方法声明中,不要将参数Object替换成其他类型

```xml
<dependency>
    <groupId>com.google.auto.value</groupId>
    <artifactId>auto-value-annotations</artifactId>
    <version>1.6.1</version>
</dependency>
<dependency>
    <groupId>com.google.auto.value</groupId>
    <artifactId>auto-value</artifactId>
    <version>1.6.1</version>
</dependency>
```


### 11.重写equals方法时同时也要重写hashCode方法



### 12.始终重写toString方法



### 13.谨慎的重写clone方法



### 14.考虑实现comparable接口

无论何时实现具有合理排序的值类,都应该让类实现Comparable接口,以便在基于比较的集合中轻松对其实例进行排序,搜索和使用.

比较CompareTo方法的实现中的字段值时,请避免使用"<"和">"运算符. 相反,使用包装类中的静态compare方法或者comparator接口中的构建方法

### 15.使类和成员的可访问性最小化

一个设计良好的组件隐藏了它的所有实现细节,干净的将它的API与它的实现分离开来.然后,组件只通过他们的API通信,并且对彼此的内部工作一无所知.这一概念,被成为信息隐藏或者封装,是软件设计的基本原则.

如果一个包级私有顶级类或者接口只被一个类使用,那么可以考虑将这个类作为使用它的唯一类的私有静态嵌套类.这将它的可访问性从包级的所有类减少到使用它的一个类.

### 16. 在公共类中使用访问方法而不是公共属性



### 17.最小化可变性

要使一个类不可变,请遵循以下五条原则

1. 不要提供修改对象状态的方法;
2. 确保这个类不能为继承;防止子类话通常是使用final修饰类;还有一种方式是构造方法私有或者包私有,然后通过提供静态工厂的方式对外访问;
3. 把所有的属性设置为final
4. 把所有的属性设置为private
5. 确保对任何可变组件的互斥访问



```java
/**
 * 当BigInteger和BigDecimal被写入时,不可变类必须时有效的final,因此他们的所有方法都有可能重写,
 * 不幸的是,在保持向后兼容性的同时,这一事实无法矫正.如果你编写一个安全性取决于来自不受信任的客户端的
 * BigDecimal或者BigDecimal参数的不变类时,则必须检查该参数是"真实的"BigInteger还是BigDecimal,
 * 而不应该是不受信任的子类的示例.如果是后者,则必须假设可能是可变的情况下保护性copy
 * @param val
 * @return
 */
public static BigInteger safeInstance(BigInteger val){
    return val.getClass()==BigInteger.class ? val : new BigInteger(val.toString());
}
```



关于序列化应该加一个警告,如果你选择不可变类实现Serializable方法,并且它包含一个或者多个引用可变对象的属性,则必须提供显式的readObject或readResolve方法,或者使用ObjectOutputStream.writeUnshared 和 ObjectInputStream.readUnshared方法,即默认的序列化形式也是可以接受的.否则攻击者可能会创建一个可变的实例.



总而言之,坚决不要为每个属性编写一个get方法后再编写一个对象的set方法.**除非有充分的理由使类成为可变类,否则类应该不可变的.**

对于一些类来说,不变性是不切实际的.**如果一个类不能设计为不可变类,那么也要尽可能的限制它的可变性.** 减少状态可以存在的状态数量,可以更容易的分析对象,以及降低出错的可能性.因此,除非有足够多的理由把属性设置为非final的情况下,否则应该每个属性都设置为final. **除非有充分的理由不这样做,否则应该把每个属性声明为私有final的.**

**构造方法应该创建完全初始化的对象,并建立所有的不变性**

### 18.组合优于继承

继承是强大的,但是它是有问题的,因为它违反封装.只有在子类和父类之间存在真正的子类型继承关系时才适用.即使如此,如果子类与父类不在同一个包中,并且父类不是为了继承而设计的,继承可能会导致脆弱性.为了避免这种脆弱性,使用合成和转发代替继承,特别是如果存在一个合适的接口来实现包装类.包装类不仅比子类更健壮,而且更强大.

### 19.如果使用继承则设计,并文档说明,否则不应该使用



### 20.接口优于抽象类

**现有的类可以很容易的进行改进来实现一个新的接口**

**接口是定义混合类型(mixin)的理想选择** 一般来说,mixin是一个类,除了它的"主类型"之外,还可以声明它提供一些可选的行为.例如,comparable是一个类型接口,它允许一个类声明它的实例相对于其他可相互比较的对象是有序的.这样的接口被成为 类型,因为它允许可选功能被"混合"到类型的主要功能.抽象类不能用于定义混合类,这是因为他们不能被加载到现有的类中:一个类不能有多个父类,并且在类层次结构中没有合理的位置来插入一个类型.

**接口允许构建非层级类型的框架**

**接口通过包装类模式确保安全的,强大的功能增加成为可能**

抽象的骨架实现类

### 21.为后代设计接口



### 22.接口仅用来定义类型

当类实现接口时,该接口作为一种类型(type),可以用来引用类的实例.因此,一个类实现了一个接口,因此表明客户端可以如何处理类的实例.为其他目的定义接口是不合适的.

**一种失败的接口就是所谓的常量接口(constant interface)**.这样的接口不包含任何方法,只包含静态的final属性.

**常量接口模式是对接口的槽糕使用** .类在内部使用的一些常量,完全属于实现细节.实现一个常量接口会导致这个实现细节泄漏到类的导出API中.对类的用户来说,类实现一个常量接口是没有意义的.事实上,它甚至使他们感到困惑.更糟糕的是,它代表了一种承诺:如果在将来的版本中修改了类,不在需要使用常量.那么它仍然必须实现接口,以保证二进制兼容性.如果一个非final类实现了常量接口,那么它的所有的子类的命名空间都会被接口中的常量所污染.

如果想要导出常量,有几个合理的选择方案.如果常量与现有的类或者接口紧密相关,则应将其添加到该类或者接口中.如果常量最好被看作枚举类型的成员,则应该使用枚举类型导出他们.否则,你应该用一个不可实例化的工具类来导出常量.

通常,使用工具类要求客户端使用类名来限制常量名.**如果大量使用实用工具类导出的常量,则通过使用静态导入来限定具有类名的常量.**

接口只能用于定义类型,它们不应该用于导出常量.

### 23.优先使用类层次而不是标签类

标签类:所有的功能实现都在一个类中,通过switch的形式进行选择;

类层次:定义抽象类和抽象方法,然后通过子类重写实现

### 24.优先考虑静态成员类

静态成员类,非静态成员类,匿名类,局部类.

在语法上,静态成员类与非静态成员类之间的唯一区别是静态成员类在其声明中具有static修饰符.尽管语法类似,但这两种嵌套是非常不同的.非静态成员类的每个实例都隐含地与包含的类的宿主实例相关联.在非静态成员类的实例方法中,可以调用宿主实例上的方法,或者使用限定的构造获得对宿主实例的引用.如果嵌套类的实例可以与其宿主类的实例隔离存在,那么嵌套类必须是静态成员类:不可能在没有宿主实例的情况下创建非静态成员类的实例.

非静态成员类的一个常见用法是定义一个adapter,它允许将外部类的实例视为某个不相关的实例.

**如果你声明了一个不需要访问宿主实例的成员类,总是把static修饰符放在它的声明中,使它成为一个静态成员类,而不是非静态的成员类.** 如果你忽略了这个修饰符,每个实例都会有一个隐藏的外部引用给它的宿主实例.如前所述,存储这个引用需要占用时间和空间.更严重的是,并且会导致即使宿主类满足垃圾回收的条件时却仍然驻留在内存中.由此产生的内存泄漏可能是灾难性的.由于引用是不可见的,所以通常是难以检测的.

在将lambda表达式添加到java之前,匿名类是创建小方法对象和处理对象的首选方法,但lambda表达式现在是首选.匿名类的另一个常见用途是实现静态工厂方法.

### 25.将源文件限制为单个顶级类



### 26.不要使用原始类型

如果你使用原始类型,则会丧失泛型的所有安全性和表达上的优势.



### 27.消除非检查警告

尽可能的消除每一个未经检查的警告;

如果你不能消除警告,但你可以证明引发警告的代码是类型安全的,那么(并且只能这样)用@SuppressWarning("unchecked")注解来抑制警告;

始终在尽可能最小的范围内使用SuppessWarning注解;

每当使用@SuppessWarning注解时,请添加注释,说明为什么是安全的.



### 28.列表优于数组



### 29.优先考虑泛型



### 30.优先使用泛型方法