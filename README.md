# PHP7-New-Features

本文档只介绍PHP7相关的新特性以及功能修改等, 对PHP7的性能和源码结构不做分析.

# 目录

- [新增功能](#新增功能)
  - [标量类型和返回类型声明](#标量类型和返回类型声明)
  - [更多的Error变为Exception](#更多的Error变为Exception)
  - [AST](#ast)
  - [Native TLS](#native-tls)
  - [其他新特性](#其他新特性)
- [弃用功能](#弃用功能)
- [新增函数](#新增函数)
- [修改函数](#修改函数)
- [非兼容性修改](#非兼容性修改)
  - [错误处理机制修改](#错误处理机制修改)
  - [字符串处理机制修改](#字符串处理机制修改)
  - [整型处理机制修改](#整型处理机制修改)
  - [参数处理机制修改](#参数处理机制修改)
  - [foreach修改](#foreach修改)
  - [list()修改](#list修改)
  - [变量处理机制修改](#变量处理机制修改)
  - [其他语言层面的修改](#其他语言层面的修改)
- [其他修改](#其他修改)
- [参考](#参考)

# 新增功能

## 标量类型和返回类型声明 
PHP语言一个非常重要的特点就是"弱类型", 它让PHP的程序变得非常容易编写, 新手接触PHP能够快速上手, 不过, 它也伴随着一些争议. 

支持变量类型的定义, 可以说是革新性质的变化, PHP开始以可选的方式支持类型定义.

除此之外, 还引入了一个开关指令declare(strict_type=1); 当这个指令一旦开启, 将会强制当前文件下的程序遵循严格的函数传参类型和返回类型.
```php
// 可以在程序中这样写, 但这还是会做隐式转化
function add(int $a, int $b): int {
    return $a + $b;
}
```
```php
// 下面不会再做隐式转化
declare(strict_types=1);
function add(int $a, int $b): int {
    return $a + $b;
}
var_dump(add(1, 2));
var_dump(add(1.5, 2.6));
// 如果没有开启declare(strict_types=1); 那么第二条打印语句会打印浮点型数据. 
```
如果不开启strict_type, PHP将会尝试帮你转换成要求的类型, 而开启之后, 会改变PHP就不再做类型转换, 类型不匹配就会抛出错误. 

然而, PHP7有了这个玩意并不代表就是强类型了, 只是在传参和返回值上不允许隐式类型转化了. 在使用内置函数的时候还是会转化, 并且在参数声明的时候也不是强类型:
```php
// 这里还是没有做到"强类型"
$name = 'kevin';
$name = 18; 
```

参考: http://hansionxu.blog.163.com/blog/static/241698109201522451148440/


## 更多的Error变为Exception

PHP7实现了一个全局的throwable接口, 原来的Exception和部分Error都实现了这个接口, 以接口的方式定义了异常的继承结构. 于是, PHP7中更多的Error变为可捕获的Exception返回给开发者, 如果不进行捕获则为Error, 如果捕获就变为一个可在程序内处理的Exception. 
这些可被捕获的Error通常都是不会对程序造成致命伤害的Error, 例如函数不存在.

PHP7进一步方便开发者处理, 让开发者对程序的掌控能力更强. 因为在默认情况下, Error会直接导致程序中断, 而PHP7则提供捕获并且处理的能力, 让程序继续执行下去, 为程序员提供更灵活的选择.

例如, 执行一个我们不确定是否存在的函数, PHP5兼容的做法是在函数被调用之前追加的判断function_exist, 而PHP7则支持捕获Exception的处理方式.
```php
try {
    no_exists_func();
} catch (Error $e) {
    echo "Exception: " . $e->getMessage();
}
// 上面程序会输出: Exception: Call to undefined function no_exists_func()
```

## AST

AST: Abstract Syntax Tree, 抽象语法树

AST在PHP编译过程作为一个中间件的角色, 替换原来直接从解释器吐出opcode的方式, 让解释器(parser)和编译器(compliler)解耦, 可以减少一些Hack代码, 同时, 让实现更容易理解和可维护.

    PHP5 : PHP代码 -> Parser语法解析 -> OPCODE -> 执行
    PHP7 : PHP代码 -> Parser语法解析 -> AST -> OPCODE -> 执行

参考: https://wiki.php.net/rfc/abstract_syntax_tree

## Native TLS

Native Thread local storage, 原生线程本地存储

PHP在多线程模式下(例如, Web服务器Apache的woker和event模式, 就是多线程), 需要解决"线程安全"的问题, 因为线程是共享进程的内存空间的, 所以每个线程本身需要通过某种方式构建私有的空间来保存自己的私有数据, 避免和其他线程相互污染. 

而PHP5采用的方式, 就是维护一个全局大数组, 为每一个线程分配一份独立的存储空间, 线程通过各自拥有的key值来访问这个全局数据组.

而这个独有的key值在PHP5中需要传递给每一个需要用到全局变量的函数, PHP7认为这种传递的方式并不友好, 并且存在一些问题. 因而, 尝试采用一个全局的线程特定变量来保存这个key值. 

参考: https://wiki.php.net/rfc/native-tls

## 其他新特性

（1） Int64支持，统一不同平台下的整型长度，字符串和文件上传都支持大于2GB。

（2） 统一变量语法（Uniform variable syntax）。

（3） foreach表现行为一致（Consistently foreach behaviors）

（4） 新的操作符 <=>, ??

（5） Unicode字符格式支持（\u{xxxxx}）

（6） 匿名类支持（Anonymous Class）

# 弃用功能
* PHP4风格的构造函数将被弃用;(和类名同名的方法视为构造方法, 这是PHP4的语法.)
* 静态调用非静态方法将被弃用;
* capture_session_meta选项将被弃用, 可以调用stream_get_meta_data()获得;

# 新增函数
* GMP(The GNU MP Bignum Library)模块新增了gmp_random_seed()函数;
* PCRE增加了preg_replace_callback_array方法;
* 增加了intdiv()函数;
* 增加了error_clear_last()函数来重置错误状态;
* 增加了ZipArchive::setComapressionIndex()和ZipArchive::setCompressionName()来设置压缩方法;
* 增加了deflate_init(), deflate_add(), inflate_init(), inflate_add();

# 修改函数
* parse_ini_file()和parse_ini_string()的scanner_mode参数增加了INI_SCANNER_TYPED选项;
* unserialize()增加了第二个参数, 可以用来指定接受的类列表;
* proc_open()打开的最大限制之前是写死的16, 现在这个限制被移除了, 最大数量取决于PHP可用的内存;
* array_column() The function now supports an array of objects as well as two-dimensional arrays
* stream_context_create() windows下面可以接收array("pipe" => array("blocking" => <boolean>))参数;
* dirname()增加了可选项$levels, 可以用来指定目录的层级. dirname(dirname($foo)) => dirname($foo, 2);
* debug_zval_dump()打印的时候, 使用int代替long, 使用float代替double;

# 非兼容性修改

## 错误处理机制修改
1. 现在有两个异常类: Exception and Error <br>
这两个类都实现了一个新的接口: Throwable.

2. 一些致命错误和可恢复致命错误改为抛出Error对象 <br>
有一些致命错误和可恢复致命错误现在改为报出Error对象. Error对象是和Exception独立的,它们无法被常规的try/catch扑获. <br>
对于这些已经转为异常的可恢复致命错误, 已经无法通过error handler静默的忽略掉. 尤其是无法忽略类型暗示错误.

3. 语法错误会抛出一个ParseError对象 <br>
语法错误会抛出一个ParseError对象, 该对象继承自Error对象. 之前处理eval()的时候, 对于潜在可能错误的代码除了检查返回值或者error_get_last()之外, 还应该捕获ParseError对象.

4. 内部对象的构造方法如果失败的时候总会抛出异常 <br>
内部对象的构造方法如果失败的时候总会报出异常. 之前的有一些构造方法会返回NULL或者一个无法使用的对象.

5. 一些E_STRICT错误的级别调整了

6. 参考资料 <br>
https://wiki.php.net/rfc/engine_exceptions_for_php7 <br>
https://wiki.php.net/rfc/throwable-interface <br>
https://wiki.php.net/rfc/internal_constructor_behaviour <br>
https://wiki.php.net/rfc/reclassify_e_strict

## 字符串处理机制修改

1. 含有十六进制字符的字符串不再视为数字 <br>
含有十六进制字符的字符串不再视为数字, 也不再区别对待. 比如下面的代码: <br>
```php
var_dump("0x123" == "291");     // bool(false)     (previously true)
var_dump(is_numeric("0x123"));  // bool(false)     (previously true)
var_dump("0xe" + "0x1");        // int(0)          (previously 16)
var_dump(substr("foo", "0x1")); // string(3) "foo" (previously "oo")
// Notice: A non well formed numeric value encountered
``` 
<br>
可以使用filter_var函数来检查一个字符串是否包含十六进制字符或者是否可以转成一个整型 <br>
```php
$str = "0xffff";
$int = filter_var($str, FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX);
if (false === $int) {
    throw new Exception("Invalid integer!");
}
var_dump($int); // int(65535)
```

2. \u{后面如果包含非法字符会报错 <br>
双引号和heredocs语法里面增加了unicode 码点转义语法, “\u{”后面必须是utf-8字符. 如果是非utf-8字符, 会报错: <br>
```php
$str = "\u{xyz}"; // Fatal error: Invalid UTF-8 codepoint escape sequence
// 可以通过对第一个\进行转义来避免这种错误
 $str = "\\u{xyz}"; // Works fine
// “\u”后面如果没有{, 则没有影响:
 $str = "\u202e"; // Works fine
```
 
3. 参考资料: <br>
https://wiki.php.net/rfc/remove_hex_support_in_numeric_strings <br>
https://wiki.php.net/rfc/unicode_escape

## 整型处理机制修改

1. 无效八进制数字会报编译错误 <br>
无效的八进制数字(包含大于7的数字)会报编译错误, 比如下面的代码会报错: <br>
```php
$i = 0781; // 8 is not a valid octal digit!
// 老版本的PHP会把无效的数字忽略
```

2. 位移负的位置会产生异常 <br>
```php
 var_dump(1 >> -1);
 // ArithmeticError: Bit shift by negative number
```

3. 左位移如果超出位数返回0 <br>
```php
var_dump(1 << 64); // int(0)
// 老版本的PHP运行结果和cpu架构有关系. 比如x86会返回1
```

4. 右位移超出会返回0或者-1 <br>
```php
var_dump(1 >> 64);  // int(0)
var_dump(-1 >> 64); // int(-1)
``` 

5. 参考 <br>
https://wiki.php.net/rfc/integer_semantics

## 参数处理机制修改

1. 重复参数命名不再支持 <br>
```php
// 重复的参数命名不再支持, 比如下面的代码执行的时候会报错:
public function foo($a, $b, $unused, $unused) {
          // ...
}
```

2. func_get_arg和func_get_args()调整 <br>
```php
// func_get_arg()和func_get_args()这两个方法返回参数当前的值, 而不是传入时的值, 当前的值有可能会被修改   
function foo($x) 
{
    $x++;
    var_dump(func_get_arg(0));
}
foo(1);
//上面的代码会打印2,而不是1. 如果想打印原始的值, 调用的顺序调整下即可
```

3. 同样在打印异常回溯信息的时候也是显示修改后的值 <br>
```php
function foo($x) 
{
    $x = 42;
    throw new Exception;
}
foo("string");
// PHP7的运行结果:
Stack trace:
#0 file.php(4): foo(42)
#1 {main}
// PHP5的运行结果:
Stack trace:
#0 file.php(4): foo('string')
#1 {main}
```
<br>
这个调整不会影响代码的行为, 不过在调试的时候需要注意这个变化.
其他和参数有关的函数都是同样的调整，比如debug_backtrace().

4. 参考: <br>
https://wiki.php.net/phpng

## foreach修改

1. foreach()循环对数组内部指针不再起作用 <br>
```php
$array = [0, 1, 2];
foreach ($array as &$val) 
{
    var_dump(current($array));
}
```
<br>
PHP7运行的结果会打印三次int(0), 也就是说数组的内部指针并没有改变. <br>
之前运行的结果会打印int(1), int(2)和bool(false) <br>

2. 按照值进行循环的时候, foreach是对该数组的拷贝操作 <br>
foreach按照值进行循环的时候(by-value), foreach是对该数组的一个拷贝进行操作. 这样在循环过程中对数组做的修改是不会影响循环行为的. <br>
```php
$array = [0, 1, 2];
$ref =& $array; // Necessary to trigger the old behavior
foreach ($array as $val) {
    var_dump($val);
    unset($array[1]);
}
//上面的代码虽然在循环中把数组的第二个元素unset掉,但PHP7还是会把三个元素打印出来:(0 1 2)
//之前老版本的PHP会把1跳过, 只打印(0 2).
```

3. 按照引用进行循环的时候, 对数组的修改会影响循环 <br>

如果在循环的时候是引用的方式, 对数组的修改会影响循环行为. 不过PHP7版本优化了很多场景下面位置的维护. 比如在循环时往数组中追加元素.<br>
```php
$array = [0];
foreach ($array as &$val) {
    var_dump($val);
    $array[1] = 1;
}
// 上面的代码中追加的元素也会参与循环,这样PHP7会打印"int(0) int(1)",老版本只会打印"int(0)"
```

4. 对简单对象plain (non-Traversable)的循环 <br>
对简单对象的循环, 不管是按照值循环还是按引用循环, 和按照引用对数组循环的行为是一样的. 不过对位置的管理会更加精确. 

5. 对迭代对象(Traversable objects)对象行为和之前一致 

6. 参考 <br>
https://wiki.php.net/rfc/php7_foreach

## list()修改

1. list()不再按照相反的顺序赋值 <br>
```php
list($array[], $array[], $array[]) = [1, 2, 3];
var_dump($array);
// 上面的代码会返回一个数组:$array == [1, 2, 3] 而不是之前的 [3, 2, 1]
// 注意: 只是赋值的顺序发生变化, 赋的值还是和原来一样的
list($a, $b, $c) = [1, 2, 3];
 // $a = 1; $b = 2; $c = 3; 
// 和原来的行为还是一样的
```

2. 空的list()赋值不再允许 <br>
```php
list() = $a;
list(,,) = $a;
list($x, list(), $y) = $a;
// 上面的这些代码运行起来会报错了
```

3. list()不在支持字符串拆分功能 <br>
```php
$string = "xy";
list($x, $y) = $string;
//这段代码最终的结果是: $x == null and $y == null (不会有提示)
//PHP5运行的结果是: $x == "x" and $y == "y".
```

4. 除此之外, list()现在也适用于数组对象 <br>
```php
list($a, $b) = (object) new ArrayObject([0, 1]);
// PHP7结果: $a == 0 and $b == 1.
// PHP5结果: $a == null and $b == null.
```

5. 参考资料 <br>
https://wiki.php.net/rfc/abstract_syntax_tree#changes_to_list <br>
https://wiki.php.net/rfc/fix_list_behavior_inconsistency

## 变量处理机制修改

1. 间接变量, 属性和方法引用都按照从左到右的顺序进行解释: <br>
```php
 $$foo['bar']['baz'] // interpreted as ($$foo)['bar']['baz']
 $foo->$bar['baz']   // interpreted as ($foo->$bar)['baz']
 $foo->$bar['baz']() // interpreted as ($foo->$bar)['baz']()
 Foo::$bar['baz']()  // interpreted as (Foo::$bar)['baz']()
```
<br>
 如果想改变解释的顺序, 可以使用大括号: <br>
```php
${$foo['bar']['baz']}
$foo->{$bar['baz']}
$foo->{$bar['baz']}()
Foo::{$bar['baz']}()
```

2. global关键字现在只能引用简单变量  <br>
```php
global $$foo->bar;    // 这种写法不支持
global ${$foo->bar};  // 需用大括号来达到效果
```

3. 用括号把变量或者函数括起来没有用了 <br>
```php
function getArray() { return [1, 2, 3]; }
$last = array_pop(getArray());
// Strict Standards: Only variables should be passed by reference
$last = array_pop((getArray()));
// Strict Standards: Only variables should be passed by reference
// 注意第二句的调用, 是用圆括号包了起来, 但还是报这个严格错误. 之前版本的PHP是不会报这个错误的.
```

4. 引用赋值时自动创建的数组元素或者对象属性顺序和以前不同了 <br>
```php
$array = [];
$array["a"] =& $array["b"];
$array["b"] = 1;
var_dump($array);
// PHP7产生的数组: ["a" => 1, "b" => 1]
// PHP5产生的数组: ["b" => 1, "a" => 1]
```
5. 参考资料 <br>
https://wiki.php.net/rfc/uniform_variable_syntax <br>
https://wiki.php.net/rfc/abstract_syntax_tree

## 其他语言层面的修改

1. 在非兼容$this语境中以静态方式调用非静态方法将不再支持 <br>
在非兼容$this语境中以静态方式调用非静态方法将不再支持. 在这种场景下面, $this不会被定义, 但调用还可以调用, 但会有一个警告提示:<br>
```php
class A {
  public function test() { var_dump($this); }
}
// Note: Does NOT extend A
class B {
  public function callNonStaticMethodOfA() { A::test(); }
}
(new B)->callNonStaticMethodOfA();
// Deprecated: Non-static method A::test() should not be called statically
// Notice: Undefined variable $this
NULL
// 注意这种情况适用于在非兼容语境中调用. 上面代码的例子中class B和class A没有关系, 所以调用的时候$this是没有定义的.
// 但如果class B是从class A继承的话,该调用是合法的.
```

2. 下面的这些保留字不能用作类名, 接口名和trait名.<br>
```php
bool
int
float
string
null
false
true
```
<br>
下面这些关键字已经被留作将来使用, 目前可以使用, 但不建议<br>
```php
resourceobject
mixed
numeric
```

3. yield语法调整<br>
在表达式里面使用yield语法结构的时候, 不再需要括号了. 它现在是一个右关联的操作符, 优先级介于"print"和"=>"操作符. 在某些场景下面行为和之前会不一致.<br>
```php
echo yield -1;
echo (yield) - 1;  // 之前的语法解释行为
echo yield (-1);   // 现在的语法解释行为
yield $foo or die;
yield ($foo or die);  // 之前的语法解释行为
(yield $foo) or die;  // 现在的语法解释行为
// 可以通过括号来避免歧义
```
<br>
备注: 关于yield, 大家可以参考鸟哥的这篇文章: http://www.laruence.com/2012/08/30/2738.html 

4. 其他的一些调整. <br>
```php
移除了ASP格式的支持和脚本语法的支持: <% 和 <script language=php> 
Removed support for assigning the result of new by reference. (很晦涩, 大家有合适的翻译希望告诉我)
移除了在非兼容$this语境中对非静态方法的作用域调用. 参考资料:https://wiki.php.net/rfc/incompat_ctx. http://www.laruence.com/2012/06/14/2628.html
ini文件里面不再支持#开头的注释, 使用";". 
$HTTP_RAW_POST_DATA 变量被移除, 使用php://input来代替. https://wiki.php.net/rfc/remove_alternative_php_tags
```

# 其他修改
* NaN和Infinity转为整型的时候, 始终为0;
* Instead of being undefined and platform-dependent, NaN and Infinity will always be zero when cast to integer;
* Calling a method on a non-object现在会抛出一个可以扑获的错误, 不再是致命错误;
* zend_parse_parameters的错误信息始终使用integer, float, 不再使用long和double;
* ignore_user_abort设为真的话, 输出缓存会继续工作, 即使链接中断;
* Zend扩展接口使用zend_extension.op_array_persist_calc()和zend_extensions.op_array_persist();
* 引入了zend_internal_function.reserved[]数组;

# 参考
* PHP7升级声明 
[https://github.com/php/php-src/blob/php-7.0.0alpha1/UPGRADING](https://github.com/php/php-src/blob/php-7.0.0alpha1/UPGRADING)

* 官方给出的新特性介绍 
[http://php.net/manual/en/migration70.new-features.php](http://php.net/manual/en/migration70.new-features.php)
