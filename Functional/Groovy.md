# Groovy
- Groovy 具有的Java所没有的语言特性 ：
    - 函数字面值（闭包）
    - 对集合的一等支持
    - 对正则表达式的一等支持
    - 对XML处理的一等支持 （一等表示内置，不需要调用类库）
- Groovy处理XML和循环遍历集合的方式要比Java简洁
- Groovy性能：如果你需要注重性能，Groovy不是一个好的选择，Groovy的对象都派生于GroovyObject，Groovy的方法都不是直接调用的而是反射执行的。虽然有invokedynamic关键字进行优化
    - 一些重活还是调用Java类库好些，毕竟是互通互用的，调用groovyserv类库能提高性能
## 安装配置
- [安装sdkman](./Skills/usually_app.md#使用sdkman)
- `sdk install groovy`

- 新建文件 `println "Hello World!"` 然后 `groovy 文件`
    - 或者`groovy -e "println 'Hello World!'"`
    
## Groovy基础
> 作为一个脚本语言，和Python Ruby Smalltalk语法相似

- groovyc groovy 类似于 javac 和 java

### Groovy特性

#### 默认导入
`这一些导入是默认隐含在Groovy代码中`
```
import groovy.lang.*
import groovy.util.*
import java.lang.*
import java.io.*
import java.math.BigDecimal
import java.math.BigInteger
import java.net.*
import java.util.*
```
- 添加额外的Jar可以使用@Grab注解或者和Java一样加入到ClassPath中去

#### 数字处理
- Groovy默认浮点数使用BigDecimal，Java中BigDecimal构造器入参是字符串，Groovy是数值，底层转换了一下，看起来更自然
- 因为是脚本语言，可以在控制台直接运行。Groovy对BEDMAS是支持的 （括号，次方，除，乘，加，减）

#### 变量，动态和静态类型，作用域
`变量`
- 如果要让Groovy和Java互操作，Groovy也能使用静态类型 `def static `，因为他简化了类型重载和调度机制
- 注意 普通类状态：在类中不能出现没有类型的变量 至少要有def这个无类型，其他的和Java一致 `def private static name = "90"`

`Groovy里的动态类型和静态类型`
- 如果变量不在定义时声明类型，那么就是一个动态类型，反之则是静态类型，静态类型就和Java的语法是一致的了，类型不可变

`作用域（通常这是指脚本状态时的普通类就是和Java一致）`
- 绑定域：是脚本的全局作用域。就是在脚本顶层没有声明类型的变量
- 本地域：变量的作用域局限于声明他们的代码块。就是在顶层声明了类型或者在代码块里
    - [变量作用域学习代码](https://github.com/kuangcp/JavaBase/blob/master/src/main/groovy/com/learn/base/VariableScope.groovy)


#### 列表和映射语法
- Groovy将列表和映射结构当做语言中的一等类型，列表和映射在底层是`ArrayList` 和 `LinkedHashMap`实现的
    - 列表：`lists = ['2', 2, new Date()]` 其实这个和Python的语法差不多，同样的支持负索引
    - 映射：`maps = [Java:"2", A:2]` 声明Maps
    - [列表和映射的学习代码](https://github.com/kuangcp/JavaBase/blob/master/src/main/groovy/com/learn/base/LearnListAndMap.groovy)

************
### 与Java的差异

- 简化输出语句：`println()` `print()` `printf()`
- Groovy的省略语法:
    - `语句结束处的分号`： 结束的分号是可选的，除非一行多条语句
    - `返回语句 return`： 方法中返回对象可以不使用return，会默认返回最后一个表达式的计算结果
    - `方法参数两边的括号`： 如果方法里的方法调用至少有一个参数，且没有二义性，则可以省略括号 
    - `public访问限定符`： 默认是public修饰符, 只有private public protected
    - 这类设计是为了让源码更为简洁，快速原型设计时体现出优势 

`脚本文件和类文件的差异`
- 一个文件定义了文件同名class 类（默认加上public） 就不能当脚本运行
    - 因为作为一个脚本文件运行的时候，是创建一个与文件同名的类 并且 继承了 `groovy.lang.Script` 将你写的语句封装起来
    - 当你显式的定义了同名类 也会隐性继承于Groovy超类 `groovy.lang.GroovyObject` 类似与Java的Object，就会当一个正常的类文件，不能写脚本了
- 脚本文件中方法不能命名为 run，对于类是没有限制，和Java一样使用

`异常处理`
- Groovy不区分已检查异常和未检查异常。Groovy会忽略方法签名中的所有throws

`Groovy中的相等`
- Groovy把 == 当做equals方法，检查真实对象是否相等需要使用Groovy的内置函数 is。但是仍然可以使用 == 来判断 null
    - 两种方式对于基本类型是一样的， == 用于对象时就是比较内存了，equals是比较值

`内部类`
- Groovy支持内部类，但大多数情况下我们应该使用函数字面值（下面有更为详细的学习）替代它。

> [示例代码](https://github.com/kuangcp/JavaBase/blob/master/src/main/groovy/com/learn/base/DiverseFromJava.groovy)

*******
### Java不具备的Groovy特性
- GroovyBean，更简单的bean
- 用操作符`?.`实现null对象的安全访问
- 猫王操作符(Elvis operator)，更短的if/else结构
- Groovy字符串，更强的字符串抽象
- 函数字面值（闭包），函数当值传递
- 对正则表达式的本地支持
- 更简单的XML处理


`GroovyBean`
- 虽然很像JavaBean但是省略了显式声明的getset方法，提供了自动构造方法（采用映射来作为入参，很方便），允许使用`.`引用私有成员变量。修改默认行为显式重定义即可
- 封装性得以保留，语法更精简

`安全解引用 ?.`
- 如果对象是null，就什么都不做，理解为当做该次调用不存在。
- 函数也是支持这种安全解引用，所以Groovy的默认集合方法，例如max() 能自动处理好null引用

`猫王操作符`
- 可以把带有默认值的 if/else 结构写的极其短小，因为这个符号看起来很像猫王鼎盛时期梳的大背头。用猫王操作符不用检查null，也不用重复变量。

`增强型字符串`
- Groovy有一个GString类，他比String要灵活
- 特别注意，这个GString和String没有任何继承关系，不能将GString作为映射中的键，或者用来比较相等，这些使用的结果都是不可预料的
- 和Python一样的有个 三引号多行字符串

### Java调用Groovy
> [参考博客](http://www.tuicool.com/articles/i6raAv)



