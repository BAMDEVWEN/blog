---
title: 'Call Apply Bind 的实现'
description: 'all Apply Bind 的实现'
pubDate: 'Nov 18 2024'
tags: ['js']

---


# Call Apply Bind

## 作用

- **改变函数内部 `this` 指向**

## 用法

### call

#### 语法

`func.call(thisArg, arg1, arg2, ...)`

#### 例子


```js
function greet(name) {
    console.log(`Hello, ${name}! I am ${this.name}.`);
}

const person = { name: 'Alice' };

greet.call(person, 'Bob'); 
// 输出：Hello, Bob! I am Alice.
```

#### 手动实现

```js
Function.prototype.MyCall=function(context,...arges){
    context=context===null ||context===undefined?globalThis:Object(context)
    var key=Symbol('temp');// 避免contex对象上已存在的任何其他属性发生冲突。
    Object.defineProperty(context,key,{
        enumerable:false,
        value:this,//this指向调用的this
    })
    var result=context[key](...arges);//调用 调用的函数
    delete context.fn;
    return result;
}
function greet(name){
    console.log(`${this.name}${name}`)
}
const li={name:'张三'}
greet.MyCall(li,'李四');
```



### apply

#### 语法

`func.apply(thisArg, [arg1, arg2, ...])`

#### 例子

```js
function greet(greeting, punctuation) {
    console.log(greeting + ', ' + this.name + punctuation);
}

const person = { name: 'Alice' };

greet.apply(person, ['Hello', '!']);  // 输出: Hello, Alice!

```

- 传递时是数据，接收是数据的解构

#### 手写实现

```js
Function.prototype.MyApply=function(context,arges){
    if(!Array.isArray(arges)){
         throw new Error('参数非数组')
    }
    context = context === null || context === undefined ? globalThis : Object(context);
    const key = Symbol('temp')
    Object.defineProperty(context, key, {
        enumerable: false,
        value: this
    })
    const result = context[key](...arges)
    return result;

}
```



### bind

#### 语法

`let boundFunc = func.bind(thisArg, arg1, arg2, ...)`

#### 例子

```js
function greet(greeting, punctuation) {
    console.log(greeting + ', ' + this.name + punctuation);
}

const person = { name: 'Alice' };

const boundGreet = greet.bind(person, 'Hello');  // 绑定了 this 为 person，并预设 'Hello'
boundGreet('!');  // 输出: Hello, Alice!

```

#### 使用场景

当你需要将 `this` 绑定到某个特定对象，并且稍后再调用这个函数时，可以使用 `bind()`

```js
const person = {
    name: 'Alice',
    greet: function() {
        console.log('Hello, ' + this.name);
    }
};

const greetPerson = person.greet.bind(person);
setTimeout(greetPerson, 1000);  // 1秒后输出: Hello, Alice

```

#### 手写bind

```js
Function.prototype.MyBind=function(context,...arges){
    context=context===null||context===undefined?globalThis:Object(context)
    const key=Symbol('temp');
    Object.defineProperty(context,key,{
        enumerable:false,
        value:this
    })
    return function(...newArges){
        const result=context[key](...arges)
        delete context[key]
        return result
    }
}
```



### 区别

- **`call(thisArg, arg1, arg2, ...)`**: 改变 `this`，并立即执行函数，参数逐个传递。

- **`apply(thisArg, [argsArray])`**: 改变 `this`，并立即执行函数，参数通过数组传递。

- **`bind(thisArg, arg1, arg2, ...)`**: 改变 `this`，返回一个新的函数，参数可以预设，但函数不会立即执行。
- 函数的参数接收，和普通函数一样
