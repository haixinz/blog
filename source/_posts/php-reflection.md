---
title: PHP反射
date: 2016-12-29 09:44:32
tags: php
---


## 反射介绍

PHP 反射机制，对类、接口、函数、方法和扩展进行反向工程的能力。
分析类，接口，函数和方法的内部结构，方法和函数的参数，以及类的属性和方法。

反射中常用的几个类：


* ReflectionClass 解析类
* ReflectionProperty 类的属性的相关信息
* ReflectionMethod 类方法的有关信息
* ReflectionParameter 取回了函数或方法参数的相关信息
* ReflectionFunction 一个函数的相关信息

分析类：

``` php
class Student
{
    public $id;
    
    public $name;
    const MAX_AGE = 200;
    public static $likes = [];
    public function __construct($id, $name = 'li')
    {
        $this->id = $id;
        $this->name = $name;
    }
    public function study()
    {
        echo 'learning...';
    }
    private function _foo()
    {
        echo 'foo';
    }
    protected function bar($to, $from = 'zh')
    {
        echo 'bar';
    }
}


<!-- more -->


```
### ReflectionClass

``` php
$ref = new ReflectionClass('Student');

// 判断类是否可实例化
if ($ref->isInstantiable()) {
    echo '可实例化';
}

// 获取类构造函数
// 有返回 ReflectionMethod 对象，没有返回 NULL
$constructor = $ref->getConstructor();
print_r($constructor);

// 获取某个属性
if ($ref->hasProperty('id')) {
    $attr = $ref->getProperty('id');
    print_r($attr);
}

// 获取属性列表
$attributes = $ref->getProperties();
foreach ($attributes as $row) {
    // 这里的 $row 为 ReflectionProperty 的实例
    echo $row->getName() , "\n";
}

// 获取静态属性，返回数组
$static = $ref->getStaticProperties();
print_r($static);

// 获取某个常量
if ($ref->hasConstant('MAX_AGE')) {
    $const = $ref->getConstant('MAX_AGE');
    echo $const;
}

// 获取常量，返回数组
$constants = $ref->getConstants();
print_r($constants);

// 获取某个方法
if ($ref->hasMethod('bar')) {
    $method = $ref->getMethod('bar');
    print_r($method);
}

// 获取方法列表
$methods = $ref->getMethods();
foreach ($methods as $key => $value) {
    // 这里的 $row 为 ReflectionMethod 的实例
    echo $value->getName() . "\n";
}
```

### ReflectionProperty

``` php
if ($ref->hasProperty('name')) {
    $attr = $ref->getProperty('name');
    // 属性名称
    echo $attr->getName();
    // 类定义时属性为真，运行时添加的属性为假
    var_dump($attr->isDefault());
    // 判断属性访问权限
    var_dump($attr->isPrivate());
    var_dump($attr->isProtected());
    var_dump($attr->isPublic());
    // 判断属性是否为静态
    var_dump($attr->isStatic());
}
```

### RefleactionMethod & ReflectionParameter

``` php
if ($ref->hasMethod('bar')) {
    $method = $ref->getMethod('bar');
    echo $method->getName();
    //isAbstract 判断是否是抽象方法
    //isConstructor 判断是否是构造方法
    //isDestructor 判断是否是析构方法
    //isFinal 判断是否是 final 描述的方法
    //isPrivate 判断是否是 private 描述的方法
    //isProtected 判断是否是 protected 描述的方法
    //isPublic 判断是否是 public 描述的方法
    //isStatic 判断是否是 static 描述的方法
    
    // 获取参数列表
    $parameters = $method->getParameters();
    foreach ($parameters as $row) {
        // 这里的 $row 为 ReflectionParameter 实例
        echo $row->getName();
        echo $row->getClass();
        // 检查变量是否有默认值
        if ($row->isDefaultValueAvailable()) {
            echo $row->getDefaultValue();
        }
        // 获取变量类型
        if ($row->hasType()) {
            echo $row->getType();
        }
    }
}

```

### eflectionFunction & ReflectionParameter

``` php
$fun = new ReflectionFunction('demo');
echo $fun->getName();
$parameters = $fun->getParameters();
foreach ($parameters as $row) {
    // 这里的 $row 为 ReflectionParameter 实例
    echo $row->getName();
    echo $row->getClass();
    // 检查变量是否有默认值
    if ($row->isDefaultValueAvailable()) {
        echo $row->getDefaultValue();
    }
    // 获取变量类型
    if ($row->hasType()) {
        echo $row->getType();
    }
}
```
## 综合实例

下面用一个简单的示例：如果用反射实例化类。

file: Student.php

``` php
class Student
{
    public $id;
    
    public $name;
    public function __construct($id, $name)
    {
        $this->id = $id;
        $this->name = $name;
    }
    public function study()
    {
        echo 'learning.....';
    }
}
```

一般情况下，实例化类的时候，直接使用 new，但是我们现在不用这种方法，我们使用反射来实现。
file: index.php

``` php
require 'student.php';
function make($class, $vars = [])
{
    $ref = new ReflectionClass($class);
    // 检查类 Student 是否可实例化
    if ($ref->isInstantiable()) {
        // 获取构造函数
        $constructor = $ref->getConstructor();
        // 没有构造函数的话，直接实例化
        if (is_null($constructor)) {
            return new $class;
        }
        // 获取构造函数参数
        $params = $constructor->getParameters();
        $resolveParams = [];
        foreach ($params as $key => $value) {
            $name = $value->getName();
            if (isset($vars[$name])) {
                // 判断如果是传递的参数，直接使用传递参数
                $resolveParams[] = $vars[$name];
            } else {
                // 没有传递参数的话，检查是否有默认值，没有默认值的话，按照类名进行递归解析
                $default = $value->isDefaultValueAvailable() ? $value->getDefaultValue() : null;
                if (is_null($default)) {
                    if ($value->getClass()) {
                        $resolveParams[] = make($value->getClass()->name, $vars);
                    } else {
                        throw new Exception("{$name} 没有传值且没有默认值");
                    }
                } else {
                    $resolveParams[] = $default;
                }
            }
        }
        // 根据参数实例化
        return $ref->newInstanceArgs($resolveParams);
    } else {
        throw new Exception("类 {$class} 不存在!");
    }
}

```
### 情况一

``` php
try {
    $stu = make('Student', ['id' => 1]);
    print_r($stu);
    $stu->study();
} catch (Exception $e) {
    echo $e->getMessage();
}
```
### 情况二

``` php
try {
    $stu = make('Student', ['id' => 1, 'name' => 'li']);
    print_r($stu);
    $stu->study();
} catch (Exception $e) {
    echo $e->getMessage();
}
```

上面两种情况很明显第一种，缺少参数 name，无法实例化成功，第二种情况就可以实例化成功。

那么我们如果将类 Student 的构造函数修改为:

``` php
public function __construct($id, $name = 'zhang')
{
    $this->id = $id;
    $this->name = $name;
}
```

这样设置 name 有默认值的情况下，那么第一种情况也可以实例化成功。

### 情况三

如果在类的构造函数中有其他类为参数的情况下，那么也可以解析：

``` php
public function __construct($id, $name, Study $study)
{
    $this->id = $id;
    $this->name = $name;
    $this->study = $study;
}
```
那么这种情况下，在分析类的构造函数参数的时候，如果没有传递参数的话，就会递归调用 make 方法处理 Study 类，如果类存在的话，实例化。

file: study.php
``` php

// 我们这里不写构造函数，测试下没有构造函数的情况
class Study
{
    public function show()
    {
        echo 'show';
    }
}
```
将 Student 类的方法 study 修改为：

``` php
public function study()
{
    $this->name . ' ' . $this->study->show();
}

```
下面测试：

``` php
try {
    $stu = make('Student', ['id' => 1]);
    print_r($stu);
    $stu->study();
} catch (Exception $e) {
    echo $e->getMessage();
}
```

## PHP反射实际应用

* 自动生成文档
* 实现 MVC 架构
* 实现单元测试
* 配合 DI 容器解决依赖

### 自动生成文档

根据反射的分析类，接口，函数和方法的内部结构，方法和函数的参数，以及类的属性和方法，可以自动生成文档。

``` php
/**
 * 学生类
 *
 * 描述信息
 */
class Student
{
    const NORMAL = 1;
    const FORBIDDEN = 2;
    /**
     * 用户ID
     * @var 类型
     */
    public $id;
    /**
     * 获取id
     * @return int
     */
    public function getId()
    {
        return $this->id;
    }
    public function setId($id = 1)
    {
        $this->id = $id;
    }
}

$ref = new ReflectionClass('Student');
$doc = $ref->getDocComment();
echo $ref->getName() . ':' . getComment($ref) , "\n";
echo "属性列表：\n";
printf("%-15s%-10s%-40s\n", 'Name', 'Access', 'Comment');
$attr = $ref->getProperties();
foreach ($attr as $row) {
    printf("%-15s%-10s%-40s\n", $row->getName(), getAccess($row), getComment($row));
}
echo "常量列表：\n";
printf("%-15s%-10s\n", 'Name', 'Value');
$const = $ref->getConstants();
foreach ($const as $key => $val) {
    printf("%-15s%-10s\n", $key, $val);
}
echo "\n\n";
echo "方法列表\n";
printf("%-15s%-10s%-30s%-40s\n", 'Name', 'Access', 'Params', 'Comment');
$methods = $ref->getMethods();
foreach ($methods as $row) {
    printf("%-15s%-10s%-30s%-40s\n", $row->getName(), getAccess($row), getParams($row), getComment($row));
}
// 获取权限
function getAccess($method)
{
    if ($method->isPublic()) {
        return 'Public';
    }
    if ($method->isProtected()) {
        return 'Protected';
    }
    if ($method->isPrivate()) {
        return 'Private';
    }
}
// 获取方法参数信息
function getParams($method)
{
    $str = '';
    $parameters = $method->getParameters();
    foreach ($parameters as $row) {
        $str .= $row->getName() . ',';
        if ($row->isDefaultValueAvailable()) {
            $str .= "Default: {$row->getDefaultValue()}";
        }
    }
    return $str ? $str : '';
}
// 获取注释
function getComment($var)
{
    $comment = $var->getDocComment();
    // 简单的获取了第一行的信息，这里可以自行扩展
    preg_match('/\* (.*) *?/', $comment, $res);
    return isset($res[1]) ? $res[1] : '';
}

```
运行 php file.php 就可以看到相应的文档信息。

### 实现 MVC 架构

现在好多框架都是 MVC 的架构，根据路由信息定位 控制器($controller) 和方法($method) 的名称，之后使用反射实现自动调用。

``` php
$class = new ReflectionClass(ucfirst($controller) . 'Controller');
$controller = $class->newInstance();
if ($class->hasMethod($method)) {
    $method = $class->getMethod($method);
    $method->invokeArgs($controller, $arguments);
} else {
    throw new Exception("{$controller} controller method {$method} not exists!");
}
```
### 实现单元测试

一般情况下我们会对函数和类进行测试，判断其是否能够按我们预期返回结果，我们可以用反射实现一个简单通用的类测试用例。
``` php
class Calc
{
    public function plus($a, $b)
    {
        return $a + $b;
    }
    public function minus($a, $b)
    {
        return $a - $b;
    }
}
function testEqual($method, $assert, $data)
{
    $arr = explode('@', $method);
    $class = $arr[0];
    $method = $arr[1];
    $ref = new ReflectionClass($class);
    if ($ref->hasMethod($method)) {
        $method = $ref->getMethod($method);
        $res = $method->invokeArgs(new $class, $data);
        var_dump($res === $assert);
    }
}
testEqual('Calc@plus', 3, [1, 2]);
testEqual('Calc@minus', -1, [1, 2]);
```

这是类的测试方法，也可以利用反射实现函数的测试方法。
这里只是我简单写的一个测试用例，PHPUnit 单元测试框架很大程度上依赖了 Reflection 的特性，可以了解下。

### 配合 DI 容器解决依赖

Laravel 等许多框架都是使用 Reflection 解决依赖注入问题，具体可查看 Laravel 源码进行分析。
下面我们代码简单实现一个 DI 容器演示 Reflection 解决依赖注入问题。
``` php
class DI
{
    protected static $data = [];
    public function __set($k, $v)
    {
        self::$data[$k] = $v;
    }
    public function __get($k)
    {
        return $this->bulid(self::$data[$k]);
    }
    // 获取实例
    public function bulid($className)
    {
        // 如果是匿名函数，直接执行，并返回结果
        if ($className instanceof Closure) {
            return $className($this);
        }
        
        // 已经是实例化对象的话，直接返回
        if(is_object($className)) {
            return $className;
        }
        // 如果是类的话，使用反射加载
        $ref = new ReflectionClass($className);
        // 监测类是否可实例化
        if (!$ref->isInstantiable()) {
            throw new Exception('class' . $className . ' not find');
        }
        // 获取构造函数
        $construtor = $ref->getConstructor();
        // 无构造函数，直接实例化返回
        if (is_null($construtor)) {
            return new $className;
        }
        // 获取构造函数参数
        $params = $construtor->getParameters();
        // 解析构造函数
        $dependencies = $this->getDependecies($params);
        // 创建新实例
        return $ref->newInstanceArgs($dependencies);
    }
    // 分析参数，如果参数中出现依赖类，递归实例化
    public function getDependecies($params)
    {
        $data = [];
        foreach($params as $param)
        {
            $tmp = $param->getClass();
            if (is_null($tmp)) {
                $data[] = $this->setDefault($param);
            } else {
                $data[] = $this->bulid($tmp->name);
            }
        }
        return $data;
    }
    
    // 设置默认值
    public function setDefault($param)
    {
        if ($param->isDefaultValueAvailable()) {
            return $param->getDefaultValue();
        }
        throw new Exception('no default value!');
    }
}
class Demo
{
    public function __construct(Calc $calc)
    {
        echo $calc->plus(1, 2);
    }
}
$di = new DI();
$di->calc = 'Calc'; // 加载单元测试用例中 Calc 类
$di->demo = 'Demo';
$di->demo;
```

注意上面的 calc 和 demo 的顺序，不能颠倒，不然的话会报错，原因是由于 Demo 依赖 Calc，首先要定义依赖关系。
在 Demo 实例化的时候，会用到 Calc 类，也就是说 Demo 依赖于 Calc，但是在 $data 上面找不到的话，会抛出错误，所以首先要定义 $di->calc = 'Calc'。

Reflection 是一个非常 Cool 的功能，使用它，但不要滥用它。