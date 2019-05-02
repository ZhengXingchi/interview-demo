## THANKYOU
如果帮助到您，请star以给作者以鼓励，谢谢!!!

## JavaScript实现继承的方式
参考[JavaScript实现继承的方式](https://juejin.im/post/59e09676f265da430629cad2)
[JavaScript六种继承方式详解](http://caibaojian.com/6-javascript-prototype.html)



1.类式继承
```js
function Animal(age){
  this.age=18
  this.type=['pig','cat']
}
Animal.prototype.greet=function(sound){
    console.log(sound)
}
function Dog(){
    this.name='dog'
}
Dog.prototype=new Animal(18)
```

缺点
```
第一个是引用缺陷：
var dog1 = new Dog();
dog1.type.push('dog');
var dog2 = new Dog();
console.log(dog2.type);  // ["dog", "cat", "dog"]

第二个是我们无法为不同的实例初始化继承来的属性，我们可以修改一下上面的例子：
function Animal(age) {
  this.age = age;
  this.type=['pig','cat']
}
Dog.prototype = new Animal(18);
console.log(dog.age); // 18
console.log(do2.age); // 18
```

2.构造函数继承
```
// 声明父类

function Animal(age) {

  this.name = 'animal';

  this.type = ['pig','cat'];

  this.age = age;

}



// 添加共有方法

Animal.prototype.greet = function(sound) {

  console.log(sound);

}

// 声明子类

function Dog(age) {

  Animal.apply(this, arguments);

}

var dog = new Dog(18);
var dog2 = new Dog(20);

dog.type.push('dog');
console.log(dog.age);  // 18
console.log(dog.type);  // ["pig", "cat", "dog"]

console.log(dog2.type);  // ["pig", "cat"]
console.log(dog2.age);  // 20

 
 
```

缺点
```
但是，构造函数继承也是有缺陷的，那就是我们无法获取到父类的共有方法，也就是通过原型prototype绑定的方法：
dog.greet();  // Uncaught TypeError: dog.greet is not a function
```


3. 组合继承
```
// 声明父类   
function Animal(age) {    
  this.name = 'animal';    
  this.type = ['pig','cat'];    
  this.age = age;   
}     

// 添加共有方法  
Animal.prototype.greet = function(sound) {    
  console.log(sound);   
}     

// 声明子类   
function Dog(age) { 
  // 构造函数继承    
  Animal.apply(this, arguments);   
}   

// 类式继承
Dog.prototype = new Animal();   

var dog = new Dog(18);   
var dog2 = new Dog(20);     
dog.type.push('dog');   
console.log(dog.age); // 18
console.log(dog.type);  // ["pig", "cat", "dog"]
dog.greet('汪汪');  // "汪汪"
console.log(dog2.type); // ["pig", "cat"]
console.log(dog2.age);  // 20
```

缺点
```
这种组合继承也是有点小缺陷的，那就是它调用了两次父类的构造函数。
```


4.寄生组合式继承
```
function Animal(age) {
  this.age = age;
  this.name = 'animal';
  this.type = ['pig', 'cat'];
}

Animal.prototype.greet = function(sound) {
  console.log(sound);
}

function Dog(age) {
  Animal.apply(this, arguments);
  this.name = 'dog';
}



/* 注意下面两行 */
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.getName = function() {
  console.log(this.name);
}

var dog = new Dog(18);   
var dog2 = new Dog(20);     

dog.type.push('dog');   
console.log(dog.age);   // 18
console.log(dog.type);   // ["pig", "cat", "dog"]
console.log(dog2.type);  // ["pig", "cat"]
console.log(dog2.age);  // 20
dog.greet('汪汪');  //  "汪汪"
```

5.extends继承
```
class Animal {   
  constructor(age) {   
    this.age = age;   
  }   
  greet(sound) {   
    console.log(sound);   
  }  
}   

class Dog extends Animal {   
  constructor(age) {   
    super(age);   
    this.age = age;   

  }  

}   

let dog = new Dog('黑色');  
dog.greet('汪汪');  // "汪汪"
console.log(dog.color); // "黑色"
```

子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象。

ES5的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。

在子类的构造函数中，只有调用super之后，才可以使用this关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有super方法才能返回父类实例。


## instanceof用法
参考[JavaScript instanceof 运算符深入剖析](https://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/index.html)
1. 首先需要了解原型链的姿势
![原型链](./img/prototype.png "原型链")

2. 其次了解instanceof的运行机制
```
function instance_of(L, R) {//L 表示左表达式，R 表示右表达式
 var O = R.prototype;// 取 R 的显示原型
 L = L.__proto__;// 取 L 的隐式原型
 while (true) { 
   if (L === null) 
     return false; 
   if (O === L)// 这里重点：当 O 严格等于 L 时，返回 true 
     return true; 
   L = L.__proto__; 
 } 
}

```

3. 测验题
Function instanceof Function
```
L=FunctionL.__proto__=Function.prototype
O=FunctionR.prototype=Function.prototype
// 第一次判断
O == L 
// 返回 true
```

Object instanceof Object
```
L=ObjectL.__proto__=Function.prototype
O=ObjectR.prototype=Object.prototype

// 第一次判断
O !== L 

// 进行第二次判断
L=ObjectL.__proto__.__proto__=Function.prototype.__proto__=Object.prototype
// 返回 true
```

Foo instanceof Foo
```
L=FooL.__proto__=Function.prototype
O=FooR.prototype=Foo.prototype
// 第一次判断
O !== L 

// 进行第二次判断
L=FooL.__proto__.__proto__=Function.prototype.__proto__=Object.prototype
// 第二次判断
O !== L 

// 进行第三次判断
L=FooL.__proto__.__proto__.__proto__=Function.prototype.__proto__.__proto__=Object.prototype.__proto__=null
// 第三次判断
L==null//根据instanceof机制返回false 
```



