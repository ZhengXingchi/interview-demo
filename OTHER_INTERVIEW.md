# 模块化
#### CommonJS模块规范

Node应用由模块组成，采用CommonJS模块规范。CommonJS规范规定，每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性（即module.exports）是对外的接口。加载某个模块，其实是加载该模块的module.exports属性。

`example.js`
```
var x = 5;
var addX = function (value) {
  return value + x;
};
module.exports.x = x;
module.exports.addX = addX;
```

`b.js`
```
var example = require('./example.js');
console.log(example.x); // 5
console.log(example.addX(1)); // 6
```


为了方便，Node为每个模块提供一个exports变量，指向module.exports。这等同在每个模块头部，有一行这样的命令。
`var exports = module.exports;`
于是我们可以直接在 exports 对象上添加方法，表示对外输出的接口，如同在module.exports上添加一样。注意，不能直接将exports变量指向一个值，因为这样等于切断了exports与module.exports的联系。


所有的exports收集到的属性和方法，都赋值给了Module.exports。当然，这有个前提，就是Module.exports本身不具备任何属性和方法。如果，Module.exports已经具备一些属性和方法，那么exports收集来的信息将被忽略。
修改rocker.js如下：
```
module.exports = 'ROCK IT!';
exports.name = function() {
    console.log('my name is cp');
}
```
再次引用执行rocker.js
```
var rocker = require('./rocker.js');
rocker.name();
//出现报错：rocker.name is not a function
```
rocker模块忽略了exports收集的name方法，返回了一个字符串“ROCK IT!”。由此可知，你的模块并不一定非得返回“实例化对象”。你的模块可以是任何合法的javascript对象--boolean, number, date, JSON, string, function, array等等。
你的模块可以是任何你设置给它的东西。如果你没有显式的给module.exports设置任何属性和方法，那么你的模块就是exports设置给module.exports的属性。

 









#### ES6模块规范
不同于CommonJS，ES6使用 export 和 import 来导出、导入模块。

```
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```
需要特别注意的是，export命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。
```
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
```

export default 命令
```
使用export default命令，为模块指定默认输出。
// export-default.js
export default function () {
  console.log('foo');
}
```

[CommonJS规范](http://javascript.ruanyifeng.com/nodejs/module.html)
[ES6 Module 的语法](http://es6.ruanyifeng.com/#docs/module)



