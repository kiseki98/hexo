---
title: Java基础
date: 2022/7/16 20:16:25
tags:
- Java
- JavaSE
categories:
- [Java, JavaSE]
description: Java基础
---

# JAVA语言基础

计算机能处理数据最小的单位是位 `bit` 而非 `byte`

## JAVA的基本特性

### 包

1. `JAVA` 中的包是为了解决命名冲突问题
2. 把功能相似或相关的类或接口组织在同一个包中，方便类的查找和使用
3. 基本类型不是对象

### 编码

`JAVA` 采用 `Unicode` 编码，中英文字符都是2字节

### 特性

1. `Java` 致力于检查程序在编译和运行时的错误
2. `Java` 虚拟机实现了跨平台接口
3. `Java ` 实现真数组（内存连续的数组），避免数据覆盖的可能；假数组：如`Python`的`list`
4. 类型检查帮助检查出许多开发早期出现的错误
5. `Java` 自己操纵内存减少了内存出错的可能性

### 标识符

1. 如程序员自己有权命名的：类名，方法名，变量名，常量名
2. 规则：不以数字开头，` a,1,$,_ ` 四种符号组成，关键字不能作为标志符

### 关键字

![关键字](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/Image.jpg)

### 变量

1. 编译期常量：使用 `static final` 修饰的常量
2. 类变量：`static` 修饰，类变量 `A` 属于类，对于不同实例的 `A` 属性相同的！
3. `JAVA` 局部变量可以和成员变量名称相同，访问成员变量要使用 `this` 来来访问，类变量使用类名调用
4. 局部变量：声明要赋初始值，默认值需要在使用前赋值，不能用 `static` 修饰

### 数据类型（F、D表示浮点，L表示Long）

1. 整型（默认0）：一个整数超过 `int` 的范围，需要加 `L`，`byte(1)`，`short(2)`，`int(4)`，`long(8)`
2. 浮点型（默认值0）：`float(4)`，`double(8)`，都是近似值
3. 布尔型（默认`false`）：`boolean(1)`
4. 字符型（默认`\u000`）：`char(2)`，存储的是 `Unicode` 码，可以赋值字符，也可以赋值字符对应的码（65对应 `A` ），还可以赋值 `Unicode` 如：`\u0639` 这样的 `Java` 的 `char` 类型，通常以 `UTF-16 Big Endian` 的方式保存一个字符。
5. `null`：不能赋值给基本类型，只能赋值给引用类型！
6. `JAVA` 对于 `long` 和 `duoble` 类型读取不是原子操作(一次读取32位，使用 `volatile` 修饰可解决)

### 进制

1. 十进制：默认缺省的方式
2. 二进制：以 `0B` 开头
3. 八进制：以 `0` 开头
4. 十六进制：以 `0X` 开头

### 类型转换

1. 包装类不存在自动类型转换 `byte < short(char) < int < long < float < double` ，浮点型不管占几个字节，都比整型大
2. 整数大小没超过 `byte` ，`short`，`char ` 范围的可以直接赋值给 `byte`，`short`，`char` 变量，不然需要类型转换
3. 多种类型混合计算先转换为容量最大的类型

### 运算符

1. 算术运算符：`+,-,*,/,%`（只有整形才能取余）,`++,--`
   1. `byte,char,short`，进行 `+,-,\,*,/` 的时候都会转成 `int` 型进行运算 `System.out.println(18 - '0')最终输入-30`。`final` 变量参与报错(不可以类型转换)
   2. 但是`+=,-=,\*=\,/=`不会转成 `int` 后运算！（计算结果强行转换为强转为对应类型），而是直接运算。`final` 变量参与不报错
2. 逻辑运算符：`&,|,!,^`（逻辑异或两边 `true` 和 `false` 表示真）
   1. `<` 和 `>` 优先级大于 `&&`，而 `&&` 大于 `||`
   2. `b = x > 50 && y > 60 || x > 50 && y < -60 || x < -50 && y > 60 || x < -50 && y < -60`
3. 位运算符：`&` 按位与 `|` 按位或 `～ ` 取反 `^ ` 异或
   1. `<<` 表示左移位 ，`>>` 表示带符号右移位
   2. `>>>` 表示无符号右移，没有 `<<<` 运算符，
4. 位异运算符：`^`， 两个数转为二进制，然后从高位开始比较，如果相同则为 0，不相同则为 1
5. 短路：`&& ||`，能判断结果就不在继续比较了
6. 三元运算符 ：`sex ? 男 : 女`，`sex  ` 为 `true` 则为男
   1. 两边如果是数值，会自动类型提升，`true ? new Integer(1) ：new Double(2)`，最终是 `1.0`
   2. 从右往左看！三元运算符时！

# 循环控制

## 选择结构

```java
public class TEST {
    public static void main(String[] args) {
        if (true) System.out.println("if只有一行代码可以不写{}代码块");
        // switch，case击穿，必须加break
        int i = 1;
        switch (i) {
            case 0:
                System.out.println("这是0");
                break;
            case 1:
                System.out.println("这是1");
                break;
            case 2: case 3:
                System.out.println("这是case合并，满足2或者3即可");
                break;
            default:
                System.out.println("这是默认");
                break;
        }
    }
}
```

## 循环结构

```java
public class TEST {
    public static void main(String[] args) {
        /**
         * 所有循环可以取别名，通过continue或者break操作指定循环
         * break：终止循环，后面加上循环的别名退出指定循环（多重循环）
         * continue：跳过，进入下次循环，配合别名可以指定进入哪个循环的下次循环（多重循环）
         */
        for1:for (System.out.println("初始化表达式只执行一次"); true; System.out.println("更新表达式")) {
            System.out.println("布尔表达式只能为Boolean");
        }
    }
}
```

# 方法

组成：修饰符列表+返回值类型+方法名+（形参列表）+{方法体}，修饰符列表可以缺省

方法名：合法的标识符

方法的重载：同名不同参，返回值无关（和修饰符列表也无关），根据实际传入的参数类型（与实际指向的对象类型无关）

方法的递归：方法自己调用自己，可能导致栈内存溢出

![访问修饰符](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/Image.jpg)

# 面向对象

## 特点

1. 优点：耦合度低，扩展能力强，组件复用性强
2. **OOA**：面向对象分析，**OOP**：面向对象编程，**OOD**：面向对象设计
3. 面向对象五大基本原则：
   1. **单一职责原则**（SRP）：一个类，最好只做一件事，只有一个引起它的变化。单一职责原则可以看做是低耦合、高内聚在面向对象原则上的引申
   2. **开放封闭原则**（OCP）：对扩展开放，对修改封闭的
   3. **Liskov替换原则**(LSP）：子类必须能够替换其基类
   4. **依赖倒置原则**（DIP）：依赖于抽象。具体而言就是高层模块不依赖于底层模块，二者都同依赖于抽象；抽象不依赖于具体，具体依赖于抽象。
   5. **接口隔离原则**（ISP）：使用多个小的专门的接口，而不要使用一个大的总接口

## 类

### 基本概念

1. **静态变量**存储在**方法区**，**局部变量**存储在**栈帧**中
2. `this`：表示当前对象（调用方法/属性的对象），方法中的 `this` 就是调用该方法的对象，`static` 方法不能使用 `this`
3. `super`：
   1. 用来访问父类被覆盖的非私有成员变量！
   2. 用来调用父类中被重写的方法
   3. 用来调用父类的构造函数

### 泛型

1. 如果泛型里的是一个类，那么尖括号里的就是一个点，泛型里的所有点(即使是**继承**关系)之间互相赋值都是错，如 `List<A>,List<B>,List<Object>`
2. 如果泛型里面带有问号，那么代表一个范围 `<? extends A>` 代表小于等于A的范围（子类），`<? super A>` 代表大于等于As的范围，<?>代表全部范围
3. 泛型小范围只能赋值给大范围，如果某点（视为小范围）包含在某个范围里，那么可以赋值
4. `List<?>` 和 `List` 是相等的，都代表最大范围

### 创建对象方式（5种）

1. `new`：使用构造方法，有参、无参都可以
2. 反射：通过 `class` 获取构造方法，`Object obj = clazz.newInstance();Object obj = clazz.getConstructor().newInstance();` 前者默认构造，后者可以有参
3. 反序列化：使用 `ObjectInputStream` 的 `readObject` 方法： `Object obj = ObjectInputStream.readObject()`。
4. `clone`：调用任何对象的 `clone` 方法，创建出一个全新的对象

## 类的分类

![Image](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/Image-1613804956477.png)

## 面向对象特征

### 构造方法

1. 构造方法属于类的方法
2. 构造方法不存在继承，`this()` 和 `super()` 不能同时出现，父类构造一定执行
3. 子类构造方法中默认调用父类无参构造方法，`super()`，同时父类的代码块{}也会执行
4. 没有默认构造方法需要显式调用有参构造方法，`super(参数)`
5. 构造方法可以私有化 `private` 修饰，构造方法中可以通过 `this(参数)` 来调用其他构造方法，但是{}不会再次执行

### new实例化过程

1. `new` 过程步骤：父类先于子类，属性初始化优先构造方法
2. **类加载：->初始化**
3. 父类静态域和静态代码块：`static{}`（  `Java ` 虚拟机加载类时，就会执行该块代码，故只执行一次）
4. 子类静态域和静态代码块：`static{}`（  `Java` 虚拟机加载类时，就会执行该块代码，故只执行一次）
5. **初始化：顺序**
6. 父类属性对象初始化
7. 父类普通代码块：{}（每次 `new` 每次执行）
8. 父类构造函数（每次new,每次执行）
9. 子类属性对象初始化
10. 子类普通代码块：{}（每次 `new` 每次执行）
11. 子类构造函数（每次 `new` 每次执行）

```java
public class TEST {
    public static void main(String[] args) {
        System.out.println(new B().getValue()); // 22 34 17
    }

    static class A {
        protected int value;

        public A(int v) {
            setValue(v); // 实际是调用子类的setValue(),子类重写方法，实际调用子类的方法(子类调用父类构造,调用子类setValue())
        }

        public void setValue(int v) {
            this.value = v;
        }

        public int getValue() {
            try {
                value++;
                return value;
            } finally {
                this.setValue(value); // 实际是调用子类的setValue()
                System.out.println(value);
            }
        }
    }

    static class B extends A {
        public B() {
            super(5); // 经过这里value -> 10,调用了子类的setValue();子类调用父类构造
            setValue(getValue() - 3); // 参数是8,value ->Return 11 -> 22 ->16 -> Return 17 -> 34输出是22 -> 34 -> 17
        }
        
        public void setValue(int value) {
            super.setValue(2 * value);
        }
    }
}

```

### 封装

1. 把对象的属性和操作结合为一个独立的整体，并隐藏内部细节
2. 使用 `private` 修饰对外提供简单的操作入口
3. `set` 和 `get`

### 继承（接口可以多继承）

1. 作用：代码复用，有了继承才有了方法覆盖和多态机制
2. 继承的数据：构造方法不继承。语言角度：私有属性不继承，官方文档这样说明的；内存角度：私有可以继承，但是没有权限访问
3. 方法覆盖：
   1. 访问权限可以更高，不能更低
   2. 抛出异常不能更多，可以更少，父类方法没抛出异常，子类也不能抛出异常
   3. 静态方法可以继承，但不能在重写

### 多态

1. 向上转型：子类转换为父类
2. 向下转型：父类转换为子类，必须先有向上转型才能有向下转型
3. 作用：降低耦合度，提高程序扩展力，父类型引用指向子类型对象，面向抽象编程，不面向具体
4. 多态分类：
   1. 对象多态：同一个对象具有不同状态，通过对象转型实现
   2. 行为多态：相同的语句执行不同的操作
   3. 编译时多态（静态绑定）：通过方法重载实现，编译阶段确定执行的代码
   4. 运行时多态（动态绑定）：运行阶段才能确定执行那个对象的方法
5. 实例过程多态：
   1. 子类实例化，首先调用父类默认构造方法，父类构造方法调用其他方法（如果该方法在子类中重写，实际调用的时子类方法）
   2. 父类方法通过 `this` 调用父类方法（实例子类时，子类重写该方法则调用是子类的方法）

### 抽象

### final 关键字

1. `final` 修饰的变量编译时视为常量
2. 修饰变量：
   1. 必须给修饰的变量赋初始值或者（代码块{}或者构造方法中赋值也是可以的！不可覆盖）
   2. `final` 修饰的字段不能重新赋值
   3. `final` 修饰的局部变量一旦初始化完成就不能重新赋值了
   4. `final ` 修饰引用表示引用不可修改，但是引用指向的对象可以修改
3. 修饰方法形参，方法内不能修改和定义
4. 修饰方法：不能被重写，可以被继承
5. 修饰类：该类不能被继承

### `static`关键字

1. `static` 方法和属性被可以继承
2. `static` 方法只能访问类中静态属性
3. 可以通过对象访问(不建议)类的静态属性和方法，前提是方法和属性的访问权限不是`private`
4. 类成员可以通过构造方法和 `set` 和 `get` 设置或者修改值 

## Object 类

1. 一个类没有继承任何类，默认继承`Object`类
2. `A instanceof B` 判断的 `A` 是否可以转换为 `B`，`A` 看的是实际类型不是静态类型
3. 方法：
   1. `clone`：创建并返回对象副本，需要实现 `Cloneable` 接口（只是一个标志）
   2. `hashcode`：计算对象或值的 `hash` 值
   3. `getClass`：调用当前方法的对象的类名
   4. `toString`：
   5. `wait`：阻塞线程
   6. `notify`：唤醒线程
   7. `equals`：先比较类型再比较值或对象是否相等
   8. `finalize`：垃圾回收器确定不存在对象引用后调用该方法（优先级极低，如果对象在等待清理队列中又被调用，则不会调用该方法）。 这个方法一个对象只能执行一次，只能在第一次进入被回收的队列，而且对象所属于的类重写了 `finalize` 方法才会被执行。第二次进入回收队列的时候，不会再执行其 `finalize` 方法，而是直接被二次标记，在下一次GC的时候被GC。

## 接口（无构造方法）

1. 面向对象设计时，每个类的职责应该单一，不要再一个类中引入过多的接口
2. 功能的封装，通过接口扩展类的方法
3. 有方法的定义，没有方法的实现，类实现接口要重写抽象方法（类没有实现接口所有方法需要abstract修饰）
4. 方法默认都是 `public abstract` 修饰的！字段默认 `public static final` 修饰
5. `JDK1.8` 开始接口可以定义：`static` 方法和 `default` 方法(接口调用)，需要实现方法，但是实现接口的类可以重写也可以不重写方法

## 抽象类-有构造方法(不可以实例化)

1. 抽象类：抽象类方法没有修饰符，默认为 `default`
2. 含有抽象（`abstract`修饰）方法的类必须声明为抽象类。抽象类可以没有抽象方法
3. 抽象类不能实例化对象，`abstract`（需要继承）不能和 `final`（不允许继承）一起使用。
4. 子类继承抽象类，需要重写所有抽象方法，如果没有重写所有抽象方法，子类也要被 `abstract` 修饰

# 常用API

## String

`String||StringBuilder||StringBuffer(线程安全)`都是 `final` 修饰的类。`StringBuilder `初始大小无参16，有参是 `str+16`，扩容 `2n+2`

速度比较：`StringBuilder > StringBuffer > String`

```java
// String 底层是 final char[] 数组
String[] arrays = {"as", "ds", "as"};
String str = String.format("你好%s", "张三");// 基本所有类型都可以用%s替换
str = str.trim(); // 去掉空格
int length = str.length(); // string的长度
str = str.toUpperCase().toLowerCase();// 字符串变为大小写
String concat = str.concat("组合新的字符串");
String intern = str.intern(); // 尝试把string放入字符串常量池，存在返回原来的地址，不存在则放入现地址
// 转换类
String str = String.copyValueOf(charArrays);// char数组替换成字符串abb
String[] split = str.split(","); // 分割字符串为数组
char[] array = str.toCharArray(); // 变为char数组
String str = String.join(",", charArrays);// 数组或者集合分割成字符串as,ds,as
// String的index相关类
String subStr = str.substring(int start, int end); // 截取子串
char c = str.charAt(1);
int codeNum = str.codePointAt(1); // 1索引的char的码值
int indexOf = str.indexOf("第一个匹配的索引");
int lastIndexOf = str.lastIndexOf("最后一个匹配的索引");
//正则相关类
String replace = str.replace("正则表达式", "符合正则变为何字符串"); // replaceAll(),replaceFirst()
boolean regionMatches = str.regionMatches(boolean ignoreCase,int toffset, String other,int ooffset,int len); // 比较区间内是否匹配
// 比较类
boolean matches = str.matches("是否匹配正则表达式");
boolean contains = str.contains("包含字符串");
boolean ignoreCase = str.equalsIgnoreCase("忽视大小写比较字符串");
boolean startsWith = str.startsWith("前缀");
boolean endsWith = str.endsWith("后缀");
int compareTo = str.compareTo("比较的字符串，返回不同字符的码值"); // str.compareToIgnoreCase("忽视大小写比较");
```

## 容器

1. `Array`：
   1. 基本类型数组定义会赋默认值（基本类型的默认值）
   2. 数组命名是名称和`[]`可以互换位置，多维数组在`Java`中是合法的`int [10][][]`，只要第一个中有数字，二维数组也是
   3. 数组的`length`属性是值数组的长度，不是指数组中元素个数
2. `Collection`：
   1. `Collection`基本方法：`add(obj)`添加，`clear()`清空，`remove( index/obj )`，`size()`元素个数，`iterator()`迭代器
   2. 数组的复制速度：`System.arraycopy > clone > Arrays.copyOf > for`
3. `List`：
   1. 特点：有序，数据元素可重复，提供针对索引操作的Api
   2. List方法：`list.set(index, obj);list.get(index);list.sort("比较器");`
   3. 实现类：元素要大于容量才扩容
      1. `ArrayList`：初始容量10，1.5倍扩容，底层是数组，指定索引插入元素，原索引元素和之后的元素统一后移！
      2. `Vector`：初始容量10，2倍扩容，线程安全，底层是数组( `Stack` 继承 `Vector` 同样是线程安全的)
      3. `LinkedList`：底层是双向链表，删除效率高，访问慢
4. `Set`：
   1. 特点：元素不可重复，无序，默认大小16
   2. 实现类：
      1. `HashSet`：底层是 `HashMap`，通过`equals()`方法比较是否是一个元素
      2. `TreeSet`：实现了 `SortSet` 接口，可以对集合中的元素自然排序，构造方法中提供比较器（优先），或者实现~接口，通过比较器比较是否为同一个元素，`o1-o2`表示小在前大在后
      3. `CopyOnWriteArraySet`：线程安全。无序
      4. `ConcurrentSkipListSet`：线程安全，有序
5. `Map`：
   1. `Map` 基本方法：`put(k,v)` 添加，`remove(k)` 删除指定元素，`replace(k,v)` 替换，`entrySet()` 返回所有 `Entry` 集合，`ketSet()` 遍历
   2. 实现类：
      1. `HashMap`：底层是哈希表，初始容量16（可指定），负载因子0.75，扩容 `2n`，自动调整大小为2的指数幂，k-v可以为 `null`
      2. `HashTable`：底层是哈希表，默认初始容量11（可指定），负载因子0.75，扩容`2n+1`，是线程安全，k-v不可以为`null`
      3. `Propertie`：键值对都是 `String`，用来设置读取系统设置（继承 `HashTable`，线程安全的）。`properties.load(inputStream)`
      4. `TreeMap`：实现 `SortMap` 接口，使用 `CompareTo` 根据 `key` 自然排序，排序根据红黑树原理。`SortMap` 要求键必须是可比较的。
      5. `ConcurrentHashMap`：线程安全
   3. `HashMap`：
      1. 底层结构：哈希表是数组，数组的每个元素是单向链表，数组元素存储的是单向链表的第一个元素
      2. `Hash`冲突：不同的键根据 `Hash` 函数计算得到的 `Hash` 值可能相等，`HashMap` 采用链表法解决 `Hash` 冲突
      3. `put`原理：`HashMap` 的初始化在 `put` 的时候执行
         1. 添加元素时根据 `key` 的 `Hash` 码计算得到数组下标i
         2. 访问该下标 `i`，如果为`table[i]`为`null`，创建节点在i位置（节点存储元素的值）
         3. 如果 `i` 位置不为空，遍历i处单向链表的节点，如果找到 `key` 值相同节点，就替换原有的值
         4. 如果所有节点都不匹配，新建节点插入单向链表尾部
      4. `get`原理：
         1. 根据 `key` 的 `Hash` 码计算得到数组下标i，如果 `table[i]` 为 `null`，返回 `null` 
         2. 如果 `i` 位置不为空，遍历单向链表，查找是否匹配 `key` ，没有返回 `null` ，有返回对应 `value`

```java
// List和Set的构造方法可以传参，达到List去重的效果
Set set = new HashSet(list);
List list = new ArrayList(set);
// Collections的Api
Collections.reverse(list); // 反转集合[1, 0, -1] -> [-1, 0, 1]
Collections.synchronizedSortedSet(); // 将Set变为线程安全的
Collections.synchronizedCollection();// 将Collection变为线程安全的
Collections.synchronizedList(); // 将List变为线程安全的
Collections.synchronizedMap(); // 将Map变为线程安全的
Collections.sort(list, new Comparator<Map>() {
    @Override
    public int compare(Map o1, Map o2) {
        Object score1 = o1.get("score");
        Object score2 = o2.get("score");
        return score2 - score1;
    }
});
```

## Math

```java
Math.ceil(a); // 向上取整
Math.floor(a); //向下取整
Math.random(a); //取[0,1)随机数
Math.abs(a); // 取绝对值
Math.max(a,b); // 取最大值
Math.min(a,b); // 取最小值
Math.round(a); // 四舍五入
Math.pow(a,b); // a的b次幂
```

## Date/Calender

```java
// 线程安全的时间类java.time包下面
LocalDate now = LocalDate.now();
LocalDateTime localDateTime = LocalDateTime.now();
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");
// Date可以指定时间戳创建
Date date = new Date(1313131321231L);
date.after(afterDate); // 比较date是否晚于afterDate
date.befort(beforeDate); //比较date是否晚于beforeDate
// Calender可以设置字段的值，月份比设置的值大1
Calendar instance = Calendar.getInstance();
instance.setTime(date); // 设定指定时间
instance.set(Calendar.YEAR, 2019);
instance.set(Calendar.MONTH, 7); // 月份实际要加一
instance.set(Calendar.DATE, 12);
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
String format = sdf.format(instance.getTime()); // 2019-08-12
System.out.println(sdf.format(instance.getTime()));
```

## 包装类

```java
// 包装类和基本类型比较（==），两边有一个基本类型比较的是值（自动拆箱），都是包装类就比引用
int num1 = 127;
Integer num2 = 127; // 相当于调用Integer.valueOf(127)这个方法
Integer num3 = Integer.valueOf(127); // Integer.valueOf(456),参数值不在[-128,127]范围相当于new对象，范围内从常量池取对象
Integer num4 = new Integer(127);
System.out.println(num1 == num2); // 117 true
System.out.println(num1 == num3); // 117 true
System.out.println(num1 == num4); // 117 true
System.out.println(num2 == num3); // 117 true    118 false
System.out.println(num2 == num4); // 117 false
System.out.println(num3 == num4); // 117 false
```

## JDK8+新特性

```java
// JDK8 Optional
List orElse = Optional.ofNullable(list).orElse(new ArrayList()); // 避免非空判断
```

# 异常

1. 异常分类：
   1. 编译时异常（受检异常）
   2. 没有继承 `RuntimeException` 的异常就是编译时异常，必须预处理 `throw` 或者 `try-catch`
   3. `throws`：出现异常直接抛出异常，代码停止执行
   4. `try-catch`：`try` 代码块出现异常，跳到 `catch` 代码块，`try` 之外的代码继续执行，程序不中断，提高程序的健壮性
2. 运行时异常（非受检异常）：
   1. 继承 `RuntimeException` 的异常就是运行时异常（非受检异常）
   2. 直接继承 ` Exception`  就是受检异常 `try-catch` 捕获的异常，可以交给其他方法处理！如：`catch{ throw new Exception("抛出异常") }`抛出处理
3. `try-catch` 中的 `return` 语句返回值：
   1. 与 `catch` 和 `finally` 有关，`try `和 `catch` 返回值会暂存，`finally `有 `return` 会覆盖，
   2. 如`return` `func()`；`func()` 会执行，但是返回值被暂存起来
   3. `finally` 先于 `throw` 和 `return` 执行！

# 反射

1. 概念：根据字节文件反射类信息（字段，方法），`int `和 `Integer `的 `class` 不是同一个东西
2. 反射步骤：
   1. 创建 `Class` 对象：`Class c=obj.getClass() || class.forName() || Student.class`这三种方式
   2. 反射信息：
      1. 注解：获取注解信息
      2. 类：
         1. 获取修饰符：`clazz.getModifiers()`
         2. 获取类名：`clazz.getName()`
         3. 获取`Calss`类型：`clazz.getType()`
         4. 获取父类：`clazz.getSuperclass()`
         5. 获取接口：`clazz.getInterfaces()`数组
      3. 获取字段：
         1. 获取指定`public`字段 ：`clazz.getField( name )`
         2. 获取所有`public`字段：`clazz.getFields()`
         3. 获取所有字段：`clazz.getDeclaredField( name )`
         4. 获取所有字段：`clazz.getDeclaredFields()`
      4. 获取方法：根据参数列表获取方法
         1. 获取指定`public`方法：`clazz.getMethod( name，参数列表类型 )`
         2. 获取所有`public`方法：`clazz.getMethods()`
         3. 获取指定方法：`clazz.getDeclaredMethod( name，参数列表类型 )`
         4. 获取所有方法：`clazz.getDeclaredMethods()`
      5. 字段：
         1. 获取修饰符：`field.getModifiers()`
         2. 获取注解：`field.getAnnotations()`
      6. 方法：
         1. 获取修饰符：`method.getModifiers()`
         2. 获取方法名：`method.getName()`
         3. 获取返回值：`method.getResultType()`
         4. 获取参数列表：`method.getParamTypes()`
      7. **方法调用**：`method.invoke(真实对象,方法参数)`
3. 创建对象：
   1. `clazz.getConstructor().newInstance()`，可以参数指定使用哪个构造方法
   2. `clazz.newInstance()`，调用默认构造方法
4. 反射细节：
   1. 反射会动态创建额外的对象，比如每个成员方法只有一个 `Method` 对象作为 `root`，他不会直接暴露给用户。调用时会返回一个 `Method` 的包装类
   2. 反射带来的效率问题主要是动态解析类，`JVM` 没法对反射代码优化。
   3. 反射会降低效率，可以设置禁止安全检查，来提高反射的运行速度； 反射常见的作用有：动态加载类、动态获取类的信息（属性、方法、构造器）；

# 注解

1. 定义：`public @interface MyAnno{}`
2. 三大内置注解
   1. `@Override`：表明子类中覆盖了超类中的某个方法，如果写错了覆盖形式，编译器会报错
   2. `@Deprecated`： 表明不希望别人在以后使用这个类，方法，变量等等
   3. `@Suppresswarnings`：达到抑制编译器产生警告的目的，如： `@SuppressWarnings(“deprecation”)`的功能是屏蔽不赞同（就是过时废弃的意思）使用的类和方法的警告
3. 元注解：
   1. `@Documented`：表示注解是否保留到`Api`文档中
   2. `@Retention(RetentionPolicy.RUNTIME)`：注解保留时长：`SOURCE`（源文件）,`CLASS`（字节码）,`RUNTIME`（运行时）
   3. `@Target(ElementType.ANNOTATION_TYPE)`：表示可以修饰哪些成员（类，字段，方法）
   4. `@Inherited`：子类会继承父类使用的注解中被 `@Inherited` 修饰的修饰；子接口不会继承父接口中的任何注解，即时使用的注解被 `@Inherited` 修饰
   5. 其他元注解：
      1. `@Repeatable`：如果一个注解需要再同一个地方上使用多次，这个注解需要被**元注解** `@Repeatable` 修饰
      2. `@Native`：使用 `@Native` 注解修饰成员变量，则表示这个变量可以被本地代码引用，常常被代码生成工具使用。对于 `@Native` 注解不常使用

# IO流

## IO流的基本概念

1. `IO`流：都是实现 `Closeable` 接口
2. 定义：定义：流是有起点有终点的有序字节序列（ `01` 序列），链接程序与数据间的通道
3. 国际化：`ResourceBundle` 能够依据 `Local` 的不同，选择性的读取与 `Local` 对应后缀的 `properties` 文件，以达到国际化的目的。
4. 原理：节点输入流，读取文件有游标，每次读取文件游标向后移（多次读取），直到文件读取完毕。
5. 分类：
   1. 输入/输出流：从外部读取数据（输入），写入数据到外部（输出）
   2. 字节/字符流：以字节为单位处理流中的数据字节流 `Stream` 结尾，以字符为单位处理流中的数据（字符流）`Reader/Writer`结尾
   3. 节点/处理流：直接操作数据源（节点流）；不直接操作数据源，对其他流进行包装（处理流/包装流）
   4. 缓冲流：都是处理流，缓冲区的存在是为了读写的提高效率（方法名基本都一样）

## 序列化和反序列化

1. 对象序列化/反序列化 `ObjectOutPutStream/ObjectInputStream`：**输入/出流，处理流，字节流**
2. 定义：对象序列化就是把一个对象转换为 `01` 二进制序列，反序列化就是把 `01` 二进制序列转换为对象
3. 不能序列化的属性：
   1. `Java` 在序列化时不会实例化 `static` 变量和 `transient` 修饰的变量，
   2. 因为 `static` 代表类的成员不属于对象， `transient` 代表对象的临时数据，被声明这两种类型的数据成员不能被序列化。
4. 实现前提：
   1. 对象的类要实现 `Serializable` 接口，便于对象在网络上传输或者保存到磁盘
   2. 系统默认赋值的 `SerialVersionUID` 在类发生变化时自己也会变化，反序列化旧的文件时就会失败

```java
// 所有流都需要手动在finally里关闭，try with resource可以不手动关闭流
int available = fis.available(); // 输入流可读取字节
byte[] bytes = new byte[1024];
// 文件输入输出流
FileInputStream fis = new FileInputStream("输入节流文件路径"); // fis.close();fos.flush();
FileOutputStream fos = new FileOutputStream("输出文件路径"); // fos.close();
int len = 0; // 输入流读取字节长度
fos.write(bytes, 0, bytes.length);
while ((len = fis.read(bytes ,0 , bytes.length)) > 0) {
    fos.write(bytes, 0, len);
}
// 数据输入输出流
DataInputStream dis = new DataInputStream(fis); // 构造方法参数是文件输入流;dis.close();
DataOutputStream dos = new DataOutputStream(fos); // 构造方法参数是输出流,dos.close();dos.flush();
while ((len = dis.read(bytes, 0, bytes.length)) > 0) {
    dos.write(bytes, 0, len);
}
// 打印输出流
PrintStream ps = new PrintStream("文件路径"); // 参数也可以是fos，输出流,ps.close();ps.flush();
System.setOut(ps); // 设置控制台输出的内容通过PrintStream输出到文件里面
System.out.println("控制台信息->文件");
// 对象输入参数出流
ObjectInputStream ois = new ObjectInputStream(fis); // 参数是fis输入流;ois.close();
ObjectOutputStream oos = new ObjectOutputStream(fos); // 参数是fos输出流;oos.flush();oos.close();oos.writeObject(obj);
oos.writeObject(obj); // 序列化对象
Object object = ois.readObject(); // 反系列化对象
// 字符流
char[] chars = new char[1024];
FileReader fr = new FileReader("文件路径");
FileWriter fw = new FileWriter("文件路径");
int read = 0;
while ((read = fr.read(chars, 0, chars.length)) > 0) {
    fr.skip(10L); // 跳过前10个
    fw.write(chars, 0, read); // 输出字符
    fw.append("追加的字符串");
}
File file = new File("文件路径");
boolean mkdir = file.mkdir(); // 创建文件夹
boolean exists = file.exists(); // 是否存在
boolean delete = file.delete(); // 删除
boolean newFile = file.createNewFile(); // 创建新文件
boolean canRead = file.canRead(); // 是否可读
boolean canWrite = file.canWrite(); // 是否可写
boolean setWritable = file.setWritable(false); // 设置是否可写
```

# 解决Hash冲突的四种方法

1. 开放地址法
2. 再哈希法
3. 链地址法
4. 公共溢出法

# SQL相关

1. 注册驱动：`Class.forName(驱动名)`
2. 获取连接：`DriverManager.getConnection(url，user ，password)`
3. 获取执行`SQL`对象：创建`Statement`是不传参的，`PreparedStatement`是需要传入`sql`语句
4. 处理结果集：`ResultSet`封装查询的结果集
5. 关闭资源：`statement`，`connection`等
6. 开启事务：通过`connection`的`setAutoCommit()；commit();`来开启事务和提交事务