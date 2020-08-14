闭包示例：

```javascript
function createIncrement(i) {
  let value = 0

  function increment() {
    value += i
    console.log(value)
    const message = `Current value is ${value}`
    return function logValue() {
      console.log(message)
    }
  }
  return increment
}

const inc = createIncrement(10)

const log = inc() // 10

inc() // 20
inc() // 30

log() // "Current value is 10"
```

这是一个典型的闭包的例子，本篇文章将从函数的执行与内存空间占用的角度去解释函数执行后的打印结果。

# 内容

## 1.预编译

- 1.1 js 执行的三个步骤
- 1.2 预编译的两个现象
- 1.3 预编译的四个步骤

## 2.函数相关

- 2.1 引用类型
- 2.2 函数的执行与内存空间

## 3.函数执行与闭包

# 预编译

js 运行三部曲：  
1.词法分析  
2.预编译  
3.解释执行

js 的特点：单线程的解释性语言，解释性语言通俗来说就是翻译一句执行一句

词法分析：

- 语法分析或者语义分析，**通篇扫描一遍看是否存在低级的语法错误**，但不会执行，通篇扫描的过程就是语法分析的过程。

预编译：

- 前提：

  - 暗示全局任何变量，如果未经声明就赋值，此变量为全局对象所有。
  - 一切声明的全局变量，全是 window 的属性。

- 预编译的两个现象：

  - 函数声明 **整体提升**
  - 变量 **声明提升**（不会变量值，只是提升声明），因此在任何位置都可以调用函数，但若要使用变量的值，需等执行到赋值语句才可以。

- 预编译的四个步骤：
  - 1.创建 AO（active object）对象
  - 2.找形参和变量声明，将变量和形参名作为 AO 属性名，值为 undefined
  - 3.将实参和形参统一
  - 4.在函数体里面找到函数声明，值赋予函数体

```javascript
function fn(a) {
  console.log(a) // function a(){}

  var a = 123

  console.log(a) // 123

  function a() {}

  console.log(a) // 123
}
fn(1)
```

注意：预编译完成之后才会开始解释执行，也就说执行 console.log(a); 时预编译的四个步骤已经执行完成。

解释执行：

- 解释一行，执行一行

## 函数

- 引用类型
- 函数的执行与内存空间

### 1.引用类型

- 什么是类型？  
   引用类型是指那些可能由 **多个值** 构成的 **对象** 。

- 引用类型的特点：  
   ![引用类型的复制]('./images/引用类型.jpg')

  1、引用类型的变量名是一个保存引用地址的指针，不会与某个对象绑定。函数属于引用类型中的一种，那么函数名实际上也是一个指向函数对象的指针，不会与某个函数绑定。  
   不会与某个对象绑定：指的是引用类型的对象名、数组名、函数名和其它变量是一样的，都是保存在栈内存中的变量，对象名称本身不是对象，只是一个指针。

  2、引用类型复制时，会将栈内存中的引用地址（对象名）复制一份放到为新变量分配的空间中，复制后的新变量仍然和旧变量指向堆内存中的同一个对象。

  3、当复制保存着的变量时，操作的是对象的引用。但在对 对象进行操作（增、删、改）时，操作的是实际的对象。

### 2.函数的执行和内存空间

函数的[[scopes]]：

```
function compare(value1, value2) {
    console.log(value1,value2);
}

console.dir(compare);
```

打印结果如图：  
![函数创建]('./images/函数创建.png')

根据打印结果可以看到，函数创建之后，函数作用域链[[scopes]]已经存在，只是此时作用链中只有全局的变量对象。

[[scopes]]：保存某个函数作用域链的对象。根据引用类型的特点，作用域链本质上是一个指向变量的 **指针列表** ，它只引用但不实际包含变量对象。

[[scopes]]不能根据对象访问属性的方式直接访问，通过 compare.[[scopes]] 形式进行访问会直接报错。

函数不能直接访问高亮属性，可访问非高亮属性  
![函数属性访问]('./images/函数属性访问.png')

**函数的执行和内存空间**:  
 对上个例子中 compare 函数进行调用 compare(1,2) ,会经历如下步骤：

    1.为函数创建一个执行环境

    2.复制函数[[scopes]]（函数创建时的scopes）属性中的对象构建起执行环境的作用域链。

    3.创建函数活动对象并推入执行环境作用域的前端

    4.执行代码

    5.销毁执行环境和活动对象（闭包情况下活动对象可能仍被引用没被销毁）

![函数执行]('./images/函数执行.png')

需要刻在脑门上的一句话：函数可以有多个变量名，且它们指向同一个地址，但是函数执行的时候，每次执行创建的执行环境、内存空间、变量对象都是独立的，而不互相等的（对象本身也不可能相等）。

# 函数执行与闭包

```javascript
function createIncrement(i) {
    let value = 0;

    function increment() {
        value += i;
        console.log(value)
        const message = `Current value is ${value}`
        return function logValue() {
            console.log(message)
        }
    }
    return increment
}

const inc = createIncrement(10);
//createIncrement()执行步骤：
//1.创建一个执行环境（在内存中开辟新的内存空间）

//2.复制函数 createIncrement 创建的时候的[[scopes]]中的对象，并构建起作用域链
// ƒ increment()
// [[Scopes]]: Scopes[1]
//  0:Global

//3.创建函数createIncrement 活动对象并推入执行环境作用域的前端
// ƒ increment()
// [[Scopes]]: Scopes[0]
//  0:createIncrement的活动对象
//  1:Global

//4.执行代码，形成闭包，并把执行后的结果赋值给变量inc
// ƒ increment()
// [[Scopes]]: Scopes[2]
//  0: Closure (createIncrement) --->包含了自身increment函数 和 createIncrement 的变量对象
//     i: 10
//     value: 0
//  1:Global

//5.执行结束，因为 inc 变量中保存了闭包，闭包中引用了createIncrement的变量对象。

const log = inc() ; // 10
//inc()执行步骤：
//1.创建一个执行环境（在内存中开辟新的内存空间）

//2.复制函数函数 inc 创建的时候的[[scopes]]中的对象，并构建起作用域链
// ƒ increment()
// [[Scopes]]: Scopes[2]
//  0: Closure (createIncrement) --->包含了自身increment函数 和 createIncrement 的变量对象
//     i: 10
//     value: 0

//  1:Global

//3.创建函数createIncrement 活动对象并推入执行环境作用域的前端
// ƒ increment()
// [[Scopes]]: Scopes[2]
//  0: Closure (createIncrement) --->包含了自身increment函数 和 createIncrement 的变量对象
//     i: 10
//     value: 0

//  1:Global

inc() // 20  函数名inc指向的函数对象被调用，重新执行一遍inc()的执行步骤，会修改内存中inc函数对象中的值，因为[[scopes]]中引用的对象是一致的？？？吗？
inc() // 30

log() // "Current value is 10"

```

为什么最后 执行 log() 时，打印结果数字是 10，而不是 30？

- 因为 log 和 inc 之间并没有
