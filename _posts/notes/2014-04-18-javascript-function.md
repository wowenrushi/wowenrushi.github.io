---
layout: post
title: Javascript语言基础 - 函数
categories: notes
---

函数不同于程序,函数总是通过return(以及调用构造函数的new操作符)返回一个值,如果没有指定return返回值,则函数默认会返回undefined值.函数调用的参数就是函数的arguments对象.

### 1.函数内部属性

{% highlight javascript linenos %}
'use strict';

// 在函数内部有两个特殊的对象: this和arguments.
// arguments对象有一个callee属性,arguments.callee指向该函数本身.
var a = {width: '200px'}
function fn() {

  // this始终指向函数调用对象
  console.log(this.width)
}

fn.call(a)    // => '200px'

function sum(a,b) {
  return arguments[0] + arguments[1]
}

// 仅仅是一个递归
function factorial(n) {
  if (n <= 1) return 1
  else return n * factorial(n-1)
}

// 通过arguments.callee方式另一种递归写法
// 注意: 在ES5的严格模式下是不允许访问函数的arguments对象的
function factorial(n) {
  if (n <= 1) return 1
  return n * arguments.callee(n-1)
}

// 又一种递归写法
function factorial(n) {
  for (var s=1; n>1; s*=n,n--) /*empty*/;
  return s
}

// 关于函数的call()和apply()方法
obj.call(this, arg1, arg2)
obj.apply(this, arguments)
{% endhighlight %}

### 2.函数作用域
{% highlight javascript linenos %}
'use strict';

{% endhighlight %}

### 3.匿名函数(anonymous function)

{% highlight javascript linenos %}
'use strict';

// 匿名函数有时候也叫做拉姆达(lambda)函数.
// 请注意以下两个常用的创建函数的方式

// 函数声明就是创建了一个带有名字的函数.
// 而函数表达式实际上就是创建了一个匿名函数,
// 然后把这个匿名函数赋值给变量`fn`.

// 函数声明
function fn() {}

// 函数表达式
var fn = function() {}

// 一个无法被任何对象调用的无意义匿名函数
function() {}

// 自调用匿名函数
(function() {})()


// 作为值的匿名函数


//
// 一个简单返回匿名函数的函数
//
function fn(x) {

  // 返回匿名函数
  return function(a) {
    var a = x
    console.log(a)
  }
}

// 调用fn返回匿名函数`function(a) {...}`
fn('Hi')   // => function(a) { var a = x; return a; }

// 调用fn调用后返回的匿名函数`function(a) {...}`
fn('Hi')() // => 'Hi'


//
// 把匿名函数当作返回值的例子
//
function comparison(property) {

  // 在这里把匿名函数作为值返回
  return function(obj1, obj2) {
    var val1 = obj1[property]
    var val2 = obj2[property]

    if (val1 < val2) return -1
    else if (val1 == val2) return 0
    return 1
  }
}

var person1 = {name:'yuwen', age:28}
var person2 = {name:'lee', age:36}
var arr = [person1, person2]

arr.sort(comparison('name'))  // => [{name:'lee',age:36},{name:'yuwen',age:28}]
arr.sort(comparison('age'))   // => [{name:'yuwen',age:28},{name:'lee',age:36}]
{% endhighlight %}


### 4.闭包
{% highlight javascript linenos %}
'use strict';

// 闭包 是指有权访问另一个函数作用域中变量的函数
// 创建闭包的常见方式就是在一个函数内部创建另一个函数
// 由于闭包会携带包含它的函数的作用域,因此会比其他函数占用更多的内存,
// 过度使用闭包可能会导致内存占用过多,所以只有在绝对必要时再使用闭包.

//
// 定义一个fn函数
//
function fn(name) {

  // 函数fn内的变量name
  // 通常来说,函数内的变量会在函数执行完毕后销毁.
  var name = 'Mozilla'

  // 内部函数f,闭包
  function f() {

    // 闭包f可以访问fn函数内的变量name
    console.log(name)
  }

  // 调用函数fn之后返回闭包:f函数
  return f
}

// 调用fn()函数,返回闭包f
var a = fn()        // => function f() { console.log(name) }

// 注意:  在这里, 闭包f()仍然携带fn()内定义的变量name
a()                 // => 'Mozilla'


//
// 仅仅是Play一个闭包
//
function add(n) {
  return function(x) {
    return n + x
  }
}

var a = add(10)     // => function(x) { return 10 + x }
var b = add(20)     // => function(x) { return 20 + x }

a(2)                // => 12
b(100)              // => 120


//
// 更详细的例子
//
function comparison(property) {
  return function(obj1, obj2) {

    // 在这个匿名函数内访问了函数外部的property变量
    //
    // 注意,仔细理解下面的注释:
    // 即使comparison函数调用之后返回了该匿名函数,
    // 并且在其他地方调用该返回的匿名函数,但它仍然可以访问comparision函数内的变量.
    // **这是因为该匿名函数的作用域链中包含comparision的变量**.
    var val1 = obj1[property]
    var val2 = obj2[property]
    
    if (val1 < val2) return -1
    else if (val1 == val2) return 0
    return 1 
  }
}

// 理解函数作用域
// 当某个函数被调用的时候,会创建一个执行环境以及相应的作用域链,
// 并把作用域链赋值给一个特殊的内部属性([[Scope]]),
// 然后,使用this,arguments和其他命名的参数的值来初始化函数的活动对象.
// 但在作用域链中,外部函数的活动对象始终处于第二位,以此往上推.

function compare(a, b) {
  return a < b ? -1 : a == b ? 0 : 1
}

// 这里在**全局环境**中调用了comapre函数,
// 创建了一个包含(this,arguments,a,b)的活动对象.
// - this      => window,
// - arguments => [1,2],
// - a         => 1,
// - b         => 2
// 
// 而此处全局执行环境中的变量(this, result, compare)也在compare执行环境的作用域中,
// 然他们则处于第二位.
// - this      => window,
// - result    => undefined,
// - compare   => function compare() {}
var result = compare(1,2)    // => -1


// 一般来说,当函数执行完毕后,局部的活动对象就会被销毁,
// 内存中仅保存全局作用域(全局执行环境的变量对象),
// 但闭包的情况又有所不同.

// 在这里当调用comparision时,它的*作用域链*被初始化为包含它的*活动对象*和*全局变量对象*.
// 这样,内部的匿名函数就可以访问它的*作用域*中的*活动对象*和*变量对象*,更为重要的是:
// 当comparision函数执行完毕后,它的*活动对象*也没有被自动销毁,因为匿名函数的作用域链仍然
// 在引用这个*活动对象*. 换句话说,当comparision执行完毕后,其*执行环境的作用域链*会被销毁,
// 但它的*活动对象*仍然存在内存中,直到匿名函数被销毁后.
var fn = comparision('name')                   // => function(obj1, obj2) {...}
var finalia = fn({name:'yuwen'},{name:'lee'})  // => 1

// 手动解除对匿名函数的引用,释放内存
finalia = null


//
// 一个实际例子
//
function makeSizer(size) {
  return function() {
    document.body.style.fontSize = size + 'px'
  }
}

var size12 = makeSizer(12)   // => 返回闭包
var size14 = makeSizer(14)   // => 返回闭包
var size16 = makeSizer(16)   // => 返回闭包

document.getElementById('size-12').onclick = size12   // => 回调闭包
document.getElementById('size-14').onclick = size14   // => 回调闭包
document.getElementById('size-16').onclick = size16   // => 回调闭包


//
// 用闭包模拟私有方法
//

// counter是一个自调用匿名函数
var counter = (function() {

  // 这里的变量也无法被外界访问
  var privateCounter = 0

  // 在这里changeBy函数无法被外界访问
  // 但是可以被counter内的increment,decrement,value三个闭包访问
  // 所以changeBy就其意义来说成为counter函数的私有方法了.
  function changeBy(val) {
    privateCounter += val
  }

  return {
    increment: function() {
      changeBy(1)
    },
    decrement: function() {
      changeBy(-1)
    },
    value: function() {
      return privateCounter
    }
  }
})()

counter.value()        // => 0
counter.increment()    // => counter.value() => 1
counter.increment()    // => counter.value() => 2
counter.value()        // => 2
counter.decrement()    // => counter.value() => 1
counter.value()        // => 1

{% endhighlight %}
