# 闭包

## 什么是闭包？

- 《高级程序设计》的定义:
  闭包是指有权访问**另一个函数作用域中的变量**的**函数**。

- 《维基百科》的定义：
  链接地址:  
   [维基百科：闭包](<https://zh.wikipedia.org/wiki/%E9%97%AD%E5%8C%85_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)>)

  闭包（英语：Closure），又称词法闭包（Lexical Closure）或函数闭包（function closures），是在支持头等函数的编程语言中实现词法绑定的一种技术。闭包在实现上是一个结构体，它**存储了一个函数**（通常是其入口地址）和**一个关联的环境**（相当于一个符号查找表）-->包含非本函数的局部变量所在的执行环境。

- MDN 的描述：  
  [MDN web docs-闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

      函数和对其周围状态（lexical environment，词法环境）的引用捆绑在一起构成闭包（closure）。也就是说，**闭包可以让你从内部函数访问外部函数作用域。** **在 JavaScript 中，每当函数被创建（指的是闭包函数被创建，通常是包含闭包的外层函数执行的时候），就会在函数生成时生成闭包。**

## 闭包的由来?为什么叫闭包？

闭包的由来：

- 彼得·兰丁（Peter Landin）在 1964 年将术语“闭包”定义为一种包含环境成分和控制成分的实体，用于在他的 SECD 机器上对表达式求值。 Joel Moses 认为是 Landin 发明了“闭包”这一术语，用来指代某些其开放绑定（自由变量）已经由其语法环境完成闭合（或者绑定）的 lambda 表达式，从而形成了闭合的表达式，或称闭包。这一用法后来于 1975 年被 Sussman 和 Steele 在定义 Scheme 语言的时候予以采纳。并广为流传。  

为什么叫闭包？

- 个人理解：封闭和包含。每个函数都有自己封闭的函数作用域，但在该封闭作用域内包含引用了其他函数的变量，所以这就是闭包-封闭（自身作用域）和包含（引用变量所在的变量对象）。通常情况下，一个函数要引用其他函数的变量，该函数为其引用变量函数的子函数，这样才能正常访问其他函数的变量。

## 闭包的表现形式

如果函数 A 内定义了函数 B，那么如果 B 存在自由变量，且这些自由变量（在函数外部定义但在函数内被引用）没有在编译过程 B 中被优化掉，那么将产生闭包。  
自由变量：一个变量，既不是函数参数，也不是函数的本地变量，则称为 自由变量。


个人理解：

- 一个函数 A 引用了另一个函数 B 的局部变量，函数 A 在被调用时肯定会形成闭包。闭包在外部函数 A 执行时就已经产生（重要）。

疑问：为什么 B 函数没有执行，闭包就已经产生，这时知道 B 中是否使用了 A 函数中的变量吗？  
回答：答案是可以知道的，因为 js 有预解析机制。

示例：

```javascript
function createComparisonFunction(propertyName) {
    return function (object1, object2) {
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];
        debugger;
        if (value1 < value2) {
            return -1;
        } else if (value1 > value2) {
            return 1;
        } else {
            return 0;
        }
    }
}

var compareNames = createComparisonFunction("name"); // createComparisonFunction调用后闭包已经形成，闭包Clourse被包含在变量compareNames中

var result = compareNames({
    name: "Jackson Yee"
}, {
    name: "易烊千玺"
});

compareNames = null; // 解除对匿名函数的引用，以便释放内存
```

例子中匿名函数之所以还能访问这个变量，是因为内部函数的作用链中包含 createComparisonFunction()的作用域。  
某个函数被调用时，会创建一个执行环境（execution context）及相应的作用链。然后，使用 arguments 和其他命名参数的值来初始化函数的活动对象（activation object）。但在作用域链中，外部函数的活动对象始终处于第二位，外部函数的外部函数的活动处于第三位，......直至作为作用域终点的全局执行环境。
当 createComparisonFunction()函数返回后，其执行环境的作用域链会被销毁，但它的活动对象仍然会留在内存中；直到匿名函数被销毁后，createComparisonFunction()的活动对象才会被销毁。  
创建的比较函数被保存在变量 compareNames 中。而通过将 compareNames 设置为等于 null 解除该函数的引用，就等于通知垃圾回收历程将其清除。

![闭包-作用域链图片解析](https://github.com/guitar99776/JSCore/blob/develop/images/closure-sopce-chain.jpg)

## 闭包与变量

作用域链的配置机制会引出一个值得注意的副作用，即闭包**只能取得包含函数中任何变量（即该函数引用其它函数的变量）的最后一个值**。因为闭包保存的是整个变量对象（引用变量函数的作用域），而不是某个特殊的变量。

重点：只会取得引用变量的最后一个值、闭包保存的是整个变量对象而不是某个特殊变量。

示例：

```javascript
function createFunctions(){
    var result=[];

    for(var i=0;i<10;i++){
        result[i]=function(){
            return i;
        }
    }

    return result;
}
```

> 扩展：变量对象、执行环境、作用域链

    变量对象：执行环境中定义的所有的变量和函数都会保存在一个对象中，保存的这个对象就叫变量对象。

    执行环境：函数在执行时形成的环境。每个函数都有自己的执行环境，每个执行环境都有一个与之关联的变量对象。全局执行环境是最外围的执行环境。

    作用域链：当代码在一个环境中执行时，会创建变量对象的作用域链。作用域链本质上是一个 指向变量对象的指针列表 ，它 引用但不实际包含变量对象 。作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在的环境的变量对象。如果这个环境是函数，则将其活动对象作为变量对象。全局执行环境的变量对象始终是作用域链中最后一个对象。

## 常见使用闭包的场景

- 模仿块级作用域
  ``` javascript
  (function(){
      //块级作用域
  })()
  ```
- 保护私有变量  
  什么是私有变量?

  - 任何在函数中定义的变量，都可以认为是私有变量，因为不能在函数的外部访问这些变量。私有变量包括函数的参数、局部变量和函数内部定义的其它函数。

  示例：

  ```javascript
  function MyObject(){
      // 私有变量和私有函数
      var privateVariable = 10;

      function privateFunction(){
          return false;
      }
      // 特权方法
      this.publicMethod = function (){
          privateVariable++;
          return privateFunction();
      }
  }
  ```

  我们把有权访问私有变量和私有函数的公有方法称为**特权方法**。利用私有和特权成员，可以隐藏那些不应该被直接修改的数据。

- 模块模式
  - 什么是模块模式?  
    XXXXXXXXXXXXX

## 闭包导致的问题

1.有内存泄漏的**风险**：

- 什么是内存泄露？

  - 百度百科的定义：
    内存泄漏（Mermery Lake）是指程序中动态分配由于某种原因程序未释放或无法释放，**造成系统的浪费**，导致程序运行的速度减慢甚至崩溃等严重后果。

  注意：并不会所有的闭包都会发生内存泄漏。只有**大量**引用的变量对象无法释放的情况下才会发生内存泄漏（重点，只有很多变量对象无法释放的情况下，才会造成内存泄露），比如：

  ```javascript
  // 示例一：（会发生内存泄漏）
  function A(){
      let name="易烊千玺";
      return functionn(){
          console.log(name);
          return name;
      }
  }
  let stack=[];

  let result=A();
  
  for(let i=0;i<1000;i++){
    stack.push(result);  // 此处会发生内存泄露，因为包含闭包的变量result被多次使用
  }
  ```
    ```
    // 示例二：（不会发生内存泄漏）
    function A(){
        let name="易烊千玺";
        return functionn(){
            console.log(name);
            return name;
        }
    }

    let result=A();

    for(let i=0;i<1000;i++){
        result();  // 不会发生内存泄露，result虽然会被执行多次，但每当一个result函数执行完，当前那个执行的函数所占用的空间就被释放了。
    }

    ```

解决内存泄漏的方式：

- 将被保存的变量对象设置为 null，比如将示例一中的变量 result 使用完之后手动置空。result=null;

疑问：如何查看内存泄漏呢？

