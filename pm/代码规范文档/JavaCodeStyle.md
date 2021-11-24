#  Java代码规约
介绍：参考 Google Java Style ，并基于Spring Code Style

*	Google Java Style:
https://google.github.io/styleguide/javaguide.html 
*	Spring Code Style:
https://github.com/spring-projects/spring-framework/wiki/Code-Style
*	Java Programming Style Guidelines:
https://petroware.no/javastyle.html

## 一、前言
### 1.1	术语说明
略
### 1.2	指南说明
略

##	二、源文件基础
### 2.1	文件名
源文件以其最顶层的类名来命名，大小写敏感，文件扩展名为<font color="red" face="黑体">.java</font>  

### 2.2	文件编码：UTF-8
源文件编码格式必须使用UTF-8编码


### 2.3	特殊字符
#### 2.3.1		空白字符
除了行结束符序列，ASCII水平空格字符(0x20，即空格)是源文件中唯一允许出现的空白字符，这意味着：所有其它字符串中的空白字符都要进行转义。

#### 2.3.2		特殊字符转义序列	
对于具有特殊转义序列的任何字符(\b, \t, \n, \f, \r, \“, \‘及)，我们使用它的转义序列，而不是相应的八进制(比如\012)或Unicode(比如\u000a)转义。

#### 2.3.3非ASCII字符
对于剩余的非ASCII字符，是使用实际的Unicode字符(比如∞)，还是使用等价的Unicode转义符(比如\u221e)，取决于哪个能让代码更易于阅读和理解。

### 2.4	缩进
*	缩进使用制表符（tab）（不要使用空格）
*	统一使用Unix (LF), 不要使用 DOS (CRLF) 作为行尾
*	消除所有行尾空格
在Linux, Mac, 或者其他系统使用如下命令即可删除行尾空格: find . -type f -name "*.java" -exec perl -p -i -e "s/[ \t]$//g" {} \;

## 三、源文件结构
### 3.1	源文件按以下顺序包含以下内容：

*	License（许可或版权信息）
*	Package statement（package语句）
*	Import statements（import语句）
*	Exactly one top-level class


#### 3.1.1	License（下面的许可换成咱们自己的许可）：
如果一个文件包含许可证或版权信息，那么它应当被放在文件最前面。示例如下：

```
/*
 * Copyright 2002-2019 the original author or authors.
 *
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```
修改代码时要始终检查许可标题中的日期范围并修改至当前时间范围   
```
* Copyright 2002-2019 the original author or authors.
```

#### 3.1.2	Package statement
package语句不换行

#### 3.1.3	Import statements
import语句的结构如下:  

*	import java.*
*	import javax.*
*	blank line
*	import all other imports
*	blank line
*	import org.springframework.*
*	blank line
*	import static all other importt	

1.	import语句尽量不要使用通配符
2. import语句不换行
3. 所有的静态导入独立成组，组之间要有空白行
4.	不应在生产代码中使用静态导入。它们应该用在测试代码中，特别是像org.junit.Assert这样的导入


### 3.2	Java源文件组织
下面是java源文件的组织方式及顺序

1.	static fields（静态属性/字段）
2.	normal fields（非静态属性/字段）
3.	constructors（构造方法）
4.	(private) methods called from constructors（被构造函数调用的方法）
5. static factory methods（静态工厂方法）
6.	JavaBean properties (i.e., getters and setters)（bean属性的方法，例如：get/set方法等）
7.	method implementations coming from interfaces（实现接口的方法）
8.	private or protected templates that get called from method implementations coming from interfaces）
9.	other methods（其他方法）
10.	equals, hashCode, and toString（最后是被重写的equals,hashcode,或者toString 方法）

⚠️注意：私有（private）或受保护（proteed）的方法,应该紧跟在调用他们的方法下面。例如实现了三个接口方法A,B,C，每个接口方法调用一个私有方法a,b,c，他们的组织顺序应该是A,a,B,b,C,c。


## 四、格式化
### 4.1	大括号
使用大括号，即使是可选的。大括号与if, else, for, do, while语句一起使用，即使只有一条语句(或是空)，也应该把大括号写上。
### 4.2	非空代码块
对于非空块和块状结构，大括号遵循Kernighan和Ritchie风格 (Egyptian brackets):   
*	左大括号前不换行  
*	左大括号后换行  
*	右大括号前换行  	
*	如果右大括号是一个语句、函数体或类的终止，则右大括号后换行; 否则不换行。例如，如果右大括号后面是else或逗号，则不换行。  

示例:

```
return new MyClass() {
	@Override 
	public void method() {
		if (condition()) {
			something();
		}
		else {
			try {
				alternative();
			} 
			catch (ProblemException ex) {
				recover();
			}
		}
	}
};

```

### 4.3	空代码块
一个空的块状结构里什么也不包含，大括号可以简洁地写成{}，不需要换行。例外：如果它是一个多块语句的一部分(if/else 或 try/catch/finally) ，即使大括号内没内容，右大括号也要换行。

### 4.4	换行
术语说明：一般情况下，一行长代码为了避免超出列限制(80或100个字符)而被分为多行，我们称之为自动换行(line-wrapping)。

尽量保持每行代码小于80，最大不超过100个字符。

### 4.5	空白行
在以下元素之前添加2个空行：

*	static {} block	（静态代码块）
*	Fields				（属性声明或初始化）
*	Constructors		（构造函数）
*	Inner classes		（内部类）

方法的签名后添加1个空白行,示例如下：

```
@Override
protected Object invoke(FooBarOperationContext context, 
        AnotherSuperLongName name) {

    // code here
}
```

## 五、类声明
尽可能将类声明放在一行

## 六、命名约定
### 6.1	包名
包名全部小写，连续的单词只是简单地连接起来，不使用下划线。
### 6.2	类名
类名都以UpperCamelCase风格编写（俗称大驼峰风格）。

类名通常是名词或名词短语，接口名称有时可能是形容词或形容词短语。现在还没有特定的规则或行之有效的约定来命名注解类型。

测试类的命名以它要测试的类的名称开始，以Test结束。例如，HashTest或HashIntegrationTest。

抽象类以Abstract为前缀(可选)。

### 6.3	方法名
方法名都以lowerCamelCase风格编写（俗称小驼峰风格）。

方法名通常是动词或动词短语。

下划线可能出现在JUnit测试方法名称中用以分隔名称的逻辑组件。一个典型的模式是：test<MethodUnderTest>_<state>，例如testPop_emptyStack。 并不存在唯一正确的方式来命名测试方法。

### 6.4	常量名
常量名命名模式为CONSTANT_CASE，全部字母大写，用下划线分隔单词。那，到底什么算是一个常量？

每个常量都是一个静态final字段，但不是所有静态final字段都是常量。在决定一个字段是否是一个常量时， 考虑它是否真的感觉像是一个常量。例如，如果任何一个该实例的观测状态是可变的，则它几乎肯定不会是一个常量。 只是永远不打算改变对象一般是不够的，它要真的一直不变才能将它示为常量。

```
// Constants(常量)
private static final Object NULL_HOLDER = new NullHolder();
public static final int DEFAULT_PORT = -1;
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final Joiner COMMA_JOINER = Joiner.on(',');  // because Joiner is immutable
static final SomeMutableType[] EMPTY_ARRAY = {};
enum SomeEnum { ENUM_CONSTANT }
// Not constants（非常量）
private static final ThreadLocal<Executor> executorHolder = new ThreadLocal<Executor>();
private static final Set<String> internalAnnotationAttributes = new HashSet<String>();
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
```
这些名字通常是名词或名词短语。
### 6.5	非常量字段名
非常量字段名以lowerCamelCase(小驼峰)风格编写。

这些名字通常是名词或名词短语。
### 6.6	参数名
参数名以lowerCamelCase风格编写。

参数应该避免用单个字符命名。
### 6.7	局部变量名
局部变量名以lowerCamelCase(小驼峰)风格编写，比起其它类型的名称，局部变量名可以有更为宽松的缩写。

虽然缩写更宽松，但还是要避免用单字符进行命名，除了临时变量和循环变量。

即使局部变量是final和不可改变的，也不应该把它示为常量，自然也不能用常量的规则去命名它。
### 6.8	类型变量名
类型变量可用以下两种风格之一进行命名：

*	单个的大写字母，后面可以跟一个数字(如：E, T, X, T2)。
*	以类命名方式(5.2.2节)，后面加个大写的T(如：RequestT, FooBarT)。

## 七、驼峰式命名法(CamelCase)
驼峰式命名法分大驼峰式命名法(UpperCamelCase)和小驼峰式命名法(lowerCamelCase)。 有时，我们有不只一种合理的方式将一个英语词组转换成驼峰形式，如缩略语或不寻常的结构(例如”IPv6”或”iOS”)。Google指定了以下的转换方案。

1.	把短语转换为纯ASCII码，并且移除任何单引号。例如：”Müller’s algorithm”将变成”Muellers algorithm”。
2.	把这个结果切分成单词，在空格或其它标点符号(通常是连字符)处分割开。
	- 推荐：如果某个单词已经有了常用的驼峰表示形式，按它的组成将它分割开(如”AdWords”将分割成”ad words”)。 需要注意的是”iOS”并不是一个真正的驼峰表示形式，因此该推荐对它并不适用。
3.	现在将所有字母都小写(包括缩写)，然后将单词的第一个字母大写：
	- 每个单词的第一个字母都大写，来得到大驼峰式命名。
	- 除了第一个单词，每个单词的第一个字母都大写，来得到小驼峰式命名。
4.	最后将所有的单词连接起来得到一个标识符。
示例：

```
Prose form                Correct               Incorrect
------------------------------------------------------------------
"XML HTTP request"        XmlHttpRequest        XMLHTTPRequest
"new customer ID"         newCustomerId         newCustomerID
"inner stopwatch"         innerStopwatch        innerStopWatch
"supports IPv6 on iOS?"   supportsIpv6OnIos     supportsIPv6OnIOS
"YouTube importer"        YouTubeImporter
                          YoutubeImporter*
```
加星号处表示可以，但不推荐。
> Note：在英语中，某些带有连字符的单词形式不唯一。例如：”nonempty”和”non-empty”都是正确的，因此方法名checkNonempty和checkNonEmpty也都是正确的



## 八、编程实践

### 8.1	捕获的异常：不能忽视
除了下面的例子，对捕获的异常不做响应是极少正确的。(典型的响应方式是打印日志，或者如果它被认为是不可能的，则把它当作一个AssertionError重新抛出。)

如果它确实是不需要在catch块中做任何响应，需要做注释加以说明(如下面的例子)。

```
try {
  	int i = Integer.parseInt(response);
  	return handleNumericResponse(i);
} catch (NumberFormatException ok) {
  	// it's not numeric; that's fine, just continue
}
return handleTextResponse(response);
```
例外：在测试中，如果一个捕获的异常被命名为expected，则它可以被不加注释地忽略。下面是一种非常常见的情形，用以确保所测试的方法会抛出一个期望中的异常， 因此在这里就没有必要加注释。

```
try {
  	emptyStack.pop();
  	fail();
} catch (NoSuchElementException expected) {
}
```


###8.2	静态成员：使用类进行调用
使用类名调用静态的类成员，而不是具体某个对象或表达式。

```
Foo aFoo = ...;
Foo.aStaticMethod(); // good
aFoo.aStaticMethod(); // bad
somethingThatYieldsAFoo().aStaticMethod(); // very bad
```

### 8.3	三元运算符
将三元运算符包括在括号内，并且首先要保证非空条件。如下示例：

```
return (foo != null ? foo : "default");
```

### 8.4	非空检查
1.	如果未使用spring 框架可，使用使用自定义或者其他第三方工具类检查并抛出异常

2.	如果使用spring框架，可使用 org.springframework.util.Assert.notNull 的静态方法去检查方法的参数是否为null。格式化异常信息，首字母大写，且后面给跟着 "must not be null". 如下示例：
```
public void handle(Event event) {
    Assert.notNull(event, "Event must not be null");
    //...
}
```

### 8.5	@Override
只要是合法的，就把@Override注解给用上。

### 8.6	@since
对框架版本有要求的类中，需要加上@since注解



### 8.7	工具类
只是静态实用工具方法集合的类必须使用Utils后缀命名，必须具有私有默认构造函数，并且必须是抽象的。因为使类抽象并提供私有默认构造函数会阻止任何人实例化它。如下示例:

```
public abstract MyUtils {

    private MyUtils() {
        /* prevent instantiation */
    }

    // static utility methods
}
```


### 8.8	属性和方法引用

类属性的引用必须显示使用this。
类方法的引用永远都不要使用this。


## 九、测试
### 9.1	命名
测试类名必须要使用Test后缀。
### 9.2	断言
使用AssertJ断言。

### 9.3	Mocking
使用BDD Mockito模拟。

### 十、代码层命名
#### Service和Repository/Dao层方法命名规约

注意：Service层一定要明确方法用图；Repository层可以不用那么清晰，因为有动态sql；

|  操作(方法)  |  Service层命名规则（只规定前缀/主键/分页）  |           Repository/Dao层命名规则(只规定前缀/主键/分页；如不需要可不带实体名字)           |
| :----------: | :-----------------: | :---------------------------------------------: |
| 查询单个对象 | get前缀；示例：getUser、getUserById |              select(动态sql)、selectByPrimaryKey              |
| 查询列表/分页查询 | list前缀；示例：listUser、listUserWithPage | list、listWithRowbounds |
|  获取统计值  | count前缀 |                count(动态sql)                |
| 插入单个对象 |       add前缀；示例：addUser       |                   insert、insertSelective |
|   批量插入   |      addBatch前缀；示例：addBatchUser      |                   insertBatch                   |
|     修改     |      update前缀；示例：updateUserById      | updateByPrimaryKey、updateByPrimaryKeySelective |
|   批量修改   | updateBatch前缀；示例：updateBatchByIds |          updateBatchByPrimaryKey          |
|     删除单个对象     |      remove前缀；示例：removeUserById      |            deleteByPrimaryKey |
|   批量删除   | removeBatch前缀；示例：removeBatchUser、removeBatchUserByIds |          deleteBatch(动态sql)、deleteBatchByPrimaryKey          |
|   存在则修改,否则创建   |save前缀；示例：saveUser |          insertOrUpdate           |




#### 领域模型命名规约 
1） 数据对象Do/Entity：xxxDo/xxxEntity/xxx，xxx即为数据表名。
2） 数据传输对象：xxxDto，xxx为业务领域相关的名称。没有业务操作。单纯的data container。 
3） ViewModel展示对象：xxxVM，xxx一般为网页名称。会包含前端页面UI的参数 (前后端分离的项目一般用不到)
4)	ValueObject值对象：xxxVo/xxxEnum(如果是枚举值)。没有主键，没有业务操作的对象，用来替换数据库中的值为可视化值或者前端可以理解的值；
5） POJO是Do/Entity/Dto/Bo/Vo的统称，禁止命名成xxxPOJO。