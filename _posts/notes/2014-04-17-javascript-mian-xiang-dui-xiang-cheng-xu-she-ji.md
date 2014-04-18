---
layout: post
title: Javascript语言基础 - 面向对象程序设计
categories: notes
---

面向对象语言都有类的概念,通过类可以创建任意多个具有相同属性和方法的对象.
但在Javascript中是通过prototype原型模式来模拟类的继承.

#### 1).工厂模式

{% highlight javascript linenos %}
'use strict';

function createBook(title, author) {
  var book = new Object
  book.title = title
  book.author = author
  // 方法体定义在createBook函数内部
  // 故每次用createBook创建一个对象的时候,bookMethod方法也被重复创建
  book.bookMethod = function() {
    console.log(this.title)
  }
  return book
}

var book1 = createBook('Javascript: The Definitive Guide', 'David Flanagan')
var book2 = createBook('Node.js in Action', 'Mike Cantelon')

book1.bookMethod()    // => 'Javascript: The Definitive Guide'
book2.bookMethod()    // => 'Node.js in Action'

// Book实例访问的是不同的函数
book1.bookMethod() === book2.bookMethod()   // => false
{% endhighlight %}

#### 2).构造函数模式

{% highlight javascript linenos %}
'use strict';

function Book(title, author) {
  this.title = title
  this.author = author
  // 这里把方法体写到了构造函数的外面
  // 所以Book实例访问的方法都是共享同一个函数
  this.bookMethod = bookMethod
}

// 然而这种方法却是在全局作用域中定义的,假如构造函数有多个方法,就必须定义多个全局函数了
// 另外,虽然属于全局定义的函数,事实上该方法最终只能被Book的实例调用
function bookMethod() {
  console.log(this.title)
}

var book1 = new Book('Javascript: The Definitive Guide', 'David Flanagan')
var book2 = new Book('Node.js in Action', 'Mike Cantelon')
book1.bookMethod()    // => 'Javascript: The Definitive Guide'
book2.bookMethod()    // => 'Node.js in Action'

book1.bookMethod === book2.bookMethod   // => true
{% endhighlight %}

#### 3).原型模式
{% highlight javascript linenos %}
'use strict';

function Book() {}
Book.prototype.title = 'Javascript: The Definitive Guide'
Book.prototype.author = 'David Flanagan'
Book.prototype.bookMethod = function() {
  console.log(this.title) 
}

var book1 = new Book
var book2 = new Book
book1.bookMethod === book2.bookMethod   // => true


//
// 简单的写法
//
function Book() {}

// 把原型对象Book.prototype用一个完全新的对象覆盖
Book.prototype = {
  // 在初始化的原型对象中'只有'一个constructor属性
  // 把constructor属性重新设置为构造函数Book本身
  constructor: Book,
  title: 'Javascript: The Definitive Guide',
  author: 'David Flanagan',
  bookMethod: function() {
    console.log(this.title)
  }
}

var book = new Book
book.bookMethod()      // => 'Javascript: The Definitive Guide'


//
// 注意: 这里的问题
//
function Person() {}

// instance.__proto__  // => Person.prototype
var instance = new Person

// Person.prototype    // => {constructor: Person, personMethod: function() {...}}
Person.prototype = {
  constructor: Person,
  personMethod: function() {
    console.log('This works!')
  }
}

instance.personMethod()  // => TypeError: undefined is not a function
instance.__proto__        // => Person.prototype(最原始的原型对象)
Person.prototype          // => {constructor: Person,personMethod() {...}} (新对象)
instance.__proto__  === Person.prototype  // => false


//
// 组合构造函数模式和原型模式
//
function Book(title, author) {
  this.title = title
  this.author = author
}

Book.prototype = {
  constructor: Book,
  bookMethod: function() {
    console.log(this.title)
  }
}


//
// 动态原型模式
//
function Book(title, author) {
  this.title = title
  this.author = author
  if (typeof bookMethod != 'function')
    Book.prototype.bookMethod = function() { console.log(this.title) }
}


//
// 理解原型对象
//
function Book() {}

// 构造函数Book的prototype属性等于(原型对象`Book.prototype`)
Book.prototype === (Book.prototype)  // => true

// (原型对象`Book.prototype`)的constructor属性等于构造函数Book
Book.prototype.constructor === Book  // => true

var book1 = new Book
var book2 = new Book

// 检测原型对象的方式
Book.prototype.isPrototypeOf(book1)  // => true
Book.prototype.isPrototypeOf(book2)  // => true

// 每个实例对象内部都包含一个__proto__属性
// 指向原型对象`Book.prototype`
book1.__proto__                      // => Book.prototype
book2.__proto__                      // => Book.prototype
book1.__proto__.constructor          // => function Book() {}
book2.__proto__.constructor          // => function Book() {}

Book.prototype.constructor           // => function Book() {}
Book.prototype.constructor.prototype // => Book.prototype


//
// 理解原型链
//
function Super() {}
function Sub() {}

Sub.prototype = new Super

// 等于:
var superinstance = new Super
superinstance.__proto__               // => Super.prototype
Sub.prototype = superinstance         // 注意: 这里把Sub的整个原型对象替换掉了

// 通过superinstance实例的`__proto__`属性找到原型对象
Sub.prototype.__proto__               // => Super.prototype
Sub.prototype                         // => Super.prototype

// Sub.prototype的构造函数被改写为Super()
Sub.prototype.constructor             // => function Super() {}

// 另外一点需要注意:
// 在实例中重写继承自原型对象中的属性和方法,只会覆盖来自原型对象中的值,
// 即使把实例中的属性重新设置为null,也不可以恢复访问原型中对应的属性和方法,
// 然而通过`delete`操作符删除实例中的属性,就可以重新访问原型中对应的属性和方法.

{% endhighlight %}

#### 4). 寄生构造函数模式 

{% highlight javascript linenos %}
'use strict';

// 这种模式的基本思想是创建一个函数,用该函数封装创建对象的代码,
// 最后又返回这个对象.
// 
// 由于构造函数在不返回值的时候,默认会返回新的实例对象.
// 而在这里我们在构造函数的末尾特别指定新的返回值,重写了默认返回的实例
function Book(title, author) {

  // 初始化一个新的对象
  var obj = new Object

  // 为对象添加属性和方法
  obj.title = title
  obj.author = author
  obj.objMethod = function() {}

  // 最后指定返回这个对象
  return obj
}


//
// 实际例子: 创建一个特殊的数组构造函数
//
function SpecialArray() {
  
  // 先创建一个原生的数组
  var arr = new Array

  // 用原生数组的push方法把传入的所有参数(arguments)逐个添加到数组arr中,
  // 这里等于是用原生数组的方法来实现SpecialArray构造函数的数组实例初始化.
  arr.push.apply(arr, arguments)

  // 为arr创建一个新方法
  arr.toPipedString = function() {
    return this.join('|')
  }

  // 需要注意的是,在这里返回的arr对象重写了构造函数默认返回的实例对象,
  // 所以该返回对象arr与构造函数和构造函数的原型对象之间没有任何关系.
  // 所以通过`instanceof`操作符是无法确定arr的对象类型的.
  return arr
}

var colors = new SpecialArray('red','black','yellow')
colors.toPipedString()  // => 'red|black|yellow'
{% endhighlight %}

#### 5).稳妥构造函数模式 **?**

{% highlight javascript linenos %}
'use strict';

function Person(name, age) {

  // 创建最终要返回的对象
  var person = new Object

  person.hi = function() {

    // 注意这里不是`this.name`
    console.log(name)
  }

  // 返回这个对象
  return person
}

// 这里不是用new方法来创建的
var p = Person('Yuwen Song', 28)
p.hi()   // => 'Yuwen Song'
{% endhighlight %}

#### 6).借用构造函数(伪造对象或经典继承)

{% highlight javascript linenos %}
'use strict';

// 创建超类构造函数
function Super() {

  // 初始化属性colors
  this.colors = ['red','yellow','blue']
}

function Sub() {

  // 借调超类的构造函数
  // 继承Super类
  //
  // 实际上是在创建新Sub实例的时候调用了Super构造函数,
  // 这样一来,就会在新的'Sub实例上执行'Super构造函数中定义的所有对象初始化代码,
  // 结果,Sub的每个实例都会具有自己的colors副本了.
  Super.call(this)
}

var instance1 = new Sub
instance1.colors.push('black')
instance1.colors                   // => ['red','yellow','blue','black']

var instance2 = new Sub
instance2.colors                   // => ['red','yellow','blue']
{% endhighlight %}

#### 7).组合继承(经典伪继承)
{% highlight javascript linenos %}
'use strict';

// 经典伪继承指的是将原型链和借用构造函数的技术组合到一起.
// 其背后的思路是使用原型链实现对原型属性和方法的继承,又通过借用构造函数来实现对实例属性的继承.
// 这样一来,既通过在原型上定义方法实现了函数复用,又能够保证每个实例都有它自己的属性.

// 定义Super类构造函数
function Super(name, age) {
  this.name = name
  this.age = age
  this.skill = ['eating','playing']
}

var super1 = new Super('yuwen', 28)
super1             // => {name:'yuwen',age:28,skill:[..]}

// 这里的Super原型对象中只有默认的constructor和__proto__属性
super1.__proto__   // => Super.prototype => Super {constructor:Super,__proto__:Object}

// Super原型对象的构造函数Super内的定义
super1.__proto__.constructor
// => funtion Super(name,age) {this.name = name, this.age = age, this.skill = ['eating','playing']}

Super.prototype.isPrototypeOf(super1)   // => true

Super.prototype.printName = function() {
  console.log(this.name)
}

super1                  // => {name:'yuwen',age:28,skill:Array(2),printName:function}
super1.printName()      // => 'yuwen'

// 在这里的Super原型对象中多出来了printName方法
super1.__proto__        // => Super {constructor:Super,printName:function,__proto__:Object}

var super2 = new Super('lorenzo', 34)
super2                  // => {name:'lorenzo',age:34,skill:Array[2],printName:function}
super2.printName()      // => 'lorenzo'

// 定义Sub子类
function Sub(name, age, gender) {

  // 借调Super构造函数
  Super.call(this, name, age)

  // 定义Sub类私有属性
  this.gender = gender
}

var sub1 = new Sub('lee', 36, 'male')
sub1                     // => {name:'lee',age:36,skill:Array[2],gender:'male'}

// 由于Sub构造函数内仅仅是借调了Super类的构造函数
// 所以创建的Sub实例的原型仍然是默认的Sub.prototype
sub1.__proto__           // => Sub {constructor:Sub,__proto__:Object}


// 注意:  定义Sub.prototype属性指向Super新实例
// 这里实现了继承Super原型,然后仅仅是截断Sub构造函数与默认的Sub原型对象之间的关系.
Sub.prototype = new Super

Sub                      // => function Sub(name,age,gender) {...}

// 注意:这里Sub.prototype指向Super类的(new Super)实例
Sub.prototype            // => (new Super) {name:undefined,age:undefined,skill:Array[2],__proto__:Super}
Sub.prototype.__proto__  // => Super {constructor:Super,printName:{},__proto__:Object}

var sub2 = new Sub('wong',28,'male')
sub2                     // => {name:'wong',age:28,skill:[...],gender:'male',printName:{}}

// 注意:这里Sub实例sub2的__proto__指向的是(new Super)这个Super类的实例
sub2.__proto__           // => (new Super) {name:undefined,age:undefined,skill:Array[2],printName:function,__proto__:Super}

// 注意:
// 定义Sub.prototype的printAge方法,实际上是在Super类的(new Super)实例上添加的方法(并非Super原型中)
Sub.prototype.printAge = function () {
  console.log(this.age)
}

Sub.prototype            // => (new Super) {name:undefined,age:undefined,skill:Array[2],printName:function,printAge:function,__proto__:Super}
Super.prototype          // => Super {constructor:Super,printName:function,__proto__:Object}
{% endhighlight %}

#### 8).原型式继承
{% highlight javascript linenos %}
// 定义init函数
function init(obj) {

  // 定义一个Lang构造函数
  function Lang() {}

  // 把构造函数Lang的prototype指向参数obj对象
  Lang.prototype = obj

  // 返回一个新的Lang实例
  // 这个实例从Lang.prototype中(obj对象)继承所有的属性和方法
  return new Lang
}

var base = {language: 'javascript', isFun: true}

// 从base对象中复制属性
var instance = init(base)
instance    // => {language: 'javascript', isFun: true} 

//
// 实际等价与:
//
function Lang() {}
var obj = {language: 'javascript', isFun: true}
Lang.prototype = obj

var instance = new Lang

// 注意: 这里Lang的实例instance.__proto__指向的正是对象obj
instance.__proto__   // => obj => Object {language: 'javascript', isFun: true}
{% endhighlight %}

#### 9).寄生组合式继承
{% highlight javascript linenos %}
function __extend(child, parent) {

  // 把复制parent的原型对象保存在proto变量中
  var _parentProto = parent.prototype,
      proto = function(_parentProto) {
        function Base() {}
        Base.prototype = _parentProto
        return new Base
      }

  // 设置parent原型对象副本的constructor为child
  proto.constructor = child

  // 把child的原型设置为变量proto中的原型
  child.prototype = proto
}
{% endhighlight %}
