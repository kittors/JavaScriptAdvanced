# JavaScript原型与原型链详细总结



## 一、原型

1.原型本质上是一个对象，存在于每一个==非箭头函数==中，所以每一个==非箭头函数==都有一个属性==prototype==指向==原型对象==



![函数fun](https://i.imgur.com/M6wiRzH.png)



展开详解：

（1）箭头函数的原型属性是undefined，只有非箭头函数才有原型对象

箭头函数没有自己的this和prototype

```javascript
const arrowFunction = () => {
  // 箭头函数没有自己的 this 或 prototype
};

console.log(arrowFunction.prototype); // 输出 "undefined"
```

（2）一个构造函数的原型初始时为一个空对象{}，通过构造函数实例化的实例对象将继承构造函数的原型方法

```javascript
//创建一个构造函数
function RegularFunction() {
  this.someProperty = 42;
}
//给构造函数的原型创建一个sayHello的方法（函数）
RegularFunction.prototype.sayHello = function() {
  console.log("Hello from RegularFunction!");
};
//创建实例对象 regularObj
const regularObj = new RegularFunction();
//实例对象将继承其构造函数的原型方法
```

2.原型对象、构造函数、实例对象三者之间的关系

- 构造函数中==prototype==属性指向==原型对象==

- 原型对象中的==constructor==属性指向==构造函数==

- 实例对象中的==\__proto__==属性指向构造函数的原型，也就是==原型对象==

  

  ![构造函数和原型以及实例对象的关系](https://i.imgur.com/rrF2P1s.png)

注意⚠️：

`prototype` 和 `__proto__` 是 JavaScript 中与原型（prototype）相关的两个属性，但它们具有不同的作用和用途。

**prototype**：

- `prototype` 是函数对象（构造函数）特有的属性，而不是每个对象都具有的属性。
- `prototype` 用于定义构造函数所创建对象的原型，它是一个对象。
- 当你创建一个新对象实例时（通过构造函数和 `new` 关键字），这个新对象的 `__proto__` 属性会指向构造函数的 `prototype` 属性。
- `prototype` 属性通常用于添加方法和属性，以便这些方法和属性可以在构造函数创建的对象实例之间共享。

示例：

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function() {
  console.log(`Hello, my name is ${this.name}`);
};

const person = new Person("Alice");
person.sayHello(); // 输出 "Hello, my name is Alice"

```

**proto**：

- `__proto__` 是每个 JavaScript 对象都具有的属性，它指向该对象的原型（prototype）。
- `__proto__` 属性用于在对象之间创建原型链，允许对象继承原型上的属性和方法。
- `__proto__` 属性是在运行时动态连接到对象的原型，可以用于查找继承的属性和方法。

示例：

```javascript
const person = { name: "Alice" };
console.log(person.__proto__); // 输出对象的原型

const personPrototype = { sayHello: function() { console.log(`Hello, my name is ${this.name}`); } };
person.__proto__ = personPrototype;

person.sayHello(); // 输出 "Hello, my name is Alice"

```

总结：

- `prototype` 属性是构造函数特有的，用于定义构造函数创建的对象实例的原型。
- `__proto__` 属性是每个对象都有的，用于指向对象的原型，创建原型链以实现继承。不建议直接操作 `__proto__`，而应该使用更现代的 `Object.create()` 或 `Object.setPrototypeOf()` 方法来设置对象的原型。



3.所有非空类型数据都有原型对象,因为从本质上他们都是通过对应的==构造函数==构建出来的，所以它们都具有`__proto__`属性,指向==构造函数==的==原型对象==

4.要判断某个值其原型对象，只需要确认该值是通过哪个构造函数构建的即可，只要确定构造函数，那么该值的`__proto__`必然指向构造函数的`prototype`

## 二、原型链

1.原型链：根据上文，所有的非空数据，都可以通过`__proto__`指向原型对象，同时如果原型对象非空，那么必然同样会有`__proto__`指向它自己的`原型对象`，如此一层层的往上追溯，以此类推，就形成一整条链路，一直到某个原型对象为null，才到达最后一个链路的最后环节，原型对象之间连路关系被称为原型链（`prototype chain`）

![原型链](https://i.imgur.com/vHOxJW0.png)

注意⚠️：其中构造函数`Object`是javaScript自带的构造函数，用来构造javaScript对象，其中创建的`Person`构造函数的`prototype`属性也是javaScript对象，所以也是由`Object` `new`出来的。

`Function`也是javaScript自带的构造函数，用来构造javaScript的函数对象

示例：

```javascript
// console.log(Function.prototype)
// console.log(Function.__proto__)

// console.log(Function.prototype.__proto__)
// console.log(Object.prototype.__proto__)

const Person = Function("name","this.name = name")//new 可以不写

console.log(Person) // [Function: anonymous]
Person.prototype.helloWorld = function () {
	console.log("helloWorld,"+this.name)
}

const person1 = new Person('kittors')
person1.helloWorld() //helloWorld,kittors

// const Person.prototype = new Object()

// console.log(Person.prototype)
```

2.原型链最后会到Object.prototype,因为原型对象，本质上就是个对象，由Object进行创建，其`__proto__`指向Object.prototype，同时约定`Object.prototype.__proto__`等于null，所有原型链的终点都是null结束

示例：

```javascript
const flag = Object.prototype.__proto__  === null
console.log(flag);//true
```

![EkPLjP4](https://i.imgur.com/EkPLjP4.png)

3.作用：

- 实现继承：JS中继承主要就是通过原型、原型链实现的
- 为某一类数据设置共享属性、方法，将大大节省内存
- 查找属性：当我们试图访问对象属性时，它会先在当前对象上进行搜寻，搜寻没有结果时会继续搜寻该对象的原型对象，以及对象的原型对象的原型对象，依次层层向上搜索，直到找到一个名字匹配的属性或者达到原型链顶端。

示例：

```javascript
//创建一个构造函数 
function Animal(name) {
	this.name = name
}


//在Animal的原型上添加一个方法eat
Animal.prototype.eat = function () {
 console.log(`${this.name} is eating.`);
}

//创建一个更加具体的构造函数Cat。它继承Animal
function Cat(name,breed) {
	Animal.call(this,name)
	this.breed = breed
}

//设置Cat的原型为Animal的实例对象，形成原型链
Cat.prototype = new Animal()

//在Cat的原型上添加一个新的方法
Cat.prototype.meow = function (){
	console.log(`${this.name} says meow!`);
}

//具体化一个mycat实例

const mycat = new Cat("小喵","白猫");

//访问对象的属性
console.log(mycat.name);  // "小喵"
console.log(mycat.breed); // "白猫"

//调用原型链上的方法
mycat.eat(); //"小喵 is eating."
mycat.meow(); //"小喵 says meow!"
```

4.补充说明：`__proto__`并不是`ECMAScript`语法规范的标准，它只是大部分的浏览器厂商实现或者支持的一个属性，通过该属性方便我们访问、修改原型对象，从`ECMAScript6`开始，可以通过`Object.getPrototypeOf()`和`Object.setPrototypeof`来访问和修改原型对象

5.总结：每个==构造函数==都有一个==原型对象==，==原型==有一个属性（constructor）指向==构造函数==，而实例有一个内部指针指向原型；如果原型是另一个类型的实例呢？那就意味着这个原型本身有一个内部指针指向另一个原型，相应地另一个原型也有一个指针指向另一个构造函数。（可以看我画的图和提供的代码实例理解）

比如构建了一个叫动物的构造函数，再构建一个叫猫的更加具体的构造函数。并且用这个猫的构造函数继承了动物这个构造函数的this指向。动物的构造函数的原型有一个方法叫吃，猫的原型对象是由动物这个构造函数构建的，其实这样猫和动物就形成了原型链了。我们再回到这个问题 如果原型是另一个类型的实例呢？

猫的原型是动物的实例，那也就意味着猫的原型指向了动物的原型，而动物的原型指向了另一个构造函数（这里可以理解为Function js中创建构造函数的构造函数，我们举的例子在生物界你可以理解为是原核生命）



参考文章：https://juejin.cn/post/7262358127344746556 感谢作者[@墨渊君](https://github.com/MoYuanJun)的分享



