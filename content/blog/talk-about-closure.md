+++
date = "2016-04-18T19:26:22-07:00"
draft = false
title = "说说es5中的闭包"
Tags = ["es5","javascript"]
+++
- **匿名函数**：下面是最常见的一种**函数表达式**的书写形式
```
var functionName = function(arg0, arg1, arg2){
    //函数体
    }
```

即：创建一个函数，并将其复制给变量`functionName`，这种情况下创建的函数叫做匿名函数(anonymous function)，**因为**`function`**后面没有标识符**，其`name`属性是空字符串。

> 函数表达式使用前必须先赋值，与函数声明的**函数声明提升**不一致,关键是理解函数声明和函数表达式的区别

<!--more-->
___

- **闭包**：有权访问另一个函数作用域中的变量的函数。创建闭包常用**在一个函数内部创建另一个函数**。

> 闭包就是在全局作用域链中，某个位置想要访问函数只能从这一级一层层**向上**为优先级访问，而**不能访问该位置下创建的函数**。
一般来讲函数执行完毕后，局部活动对象就会被销毁，内存中仅仅保留全局作用域。但闭包不同。

- 在一个函数内部定义一个函数会将包含的函数（即外部函数）的活动对象添加到他的作用域链中。就是说，某一个函数内部定义的匿名函数的作用域链中，实际上会包含外部函数（即定义这个匿名函数的函数）的活动对象----->所以匿名函数从定义它的函数中返回后，匿名函数的作用域链被初始化为包含在定义它的函数的活动对象和全局变量对象------>匿名函数就可以访问在定义它的函数中的所有变量-------->更重要的是，定义他的函数在执行完毕后其活动对象不会被销毁（**因为匿名函数的作用域链仍在引用这个活动对象**），即 即使定义它的函数返回后，其执行环境的作用域链被销毁了，但活动对象仍在内存中---------->因此，**直到匿名函数被销毁后，定义它的函数的活动对象才会被销毁**
![closure](https://c1.staticflickr.com/1/579/33457488141_24d2d58c5e_b.jpg)
> 解除对匿名函数的引用（来释放内存）
    (定义匿名函数的函数名)`functionName = null`;
    解除函数的引用，即通知垃圾回收例程将其清除
    随着匿名函数作用域链被销毁，其他作用域（除了全局）也可安全销毁

- 闭包会携带它函数的作用域，因此会比其他函数占用更多内存，**过度使用闭包会导致内存占用过多**


- 注意这种作用域链机制带来的一个**副作用：闭包只能取得包含函数中的任何变量的最后一个值。**
> （因为   闭包保存的是整个变量对象而不是某个特殊变量）

```
<!DOCTYPE html>
<html>
    <head>
        <title>Closure Example</title>
    </head>
    <body>
        <script type="text/javascript">

//下面这个函数
            function createFunctions(){
                var result = new Array();
                
                
                for (var i=0; i < 10; i++){
                    result[i] = function(){
                        return i;
                    };
                }
                
                return result;
            }
//上面这个函数

            var funcs = createFunctions();
            
            //every function outputs 10
            for (var i=0; i < funcs.length; i++){
                document.write(funcs[i]() + "<br />");
            }

        </script>
     
    </body>
</html>
```

上面这个函数理应返回一个函数数组，每个函数都返回自己索引值（0~9）。**但是！**因为每个函数的作用域链都保存着`createFunctions()`函数的活动对象，所以**他们引用的是同一个变量**`i`。**当**`createFunctions()`**函数返回后，得到变量**`i`**的值为10。**此时，**每个函数都引用着保存变量**`i`**的同一个变量对象，所以每个函数内部的**`i`**值都为10。**

> 下面通过创建另一个匿名函数强制让闭包行为符合预期

```
<!DOCTYPE html>
<html>
    <head>
        <title>Closure Example 2</title>
    </head>
    <body>
        <script type="text/javascript">

//下面这个函数        
            function createFunctions(){
                var result = new Array();
                
                for (var i=0; i < 10; i++){
                    result[i] = function(num){
                        return function(){      //这部分
                            return num;
                        };
                    }(i);
                }
                
                return result;
            }
//上面这个函数
            
            var funcs = createFunctions();
            
            //every function outputs 10
            for (var i=0; i < funcs.length; i++){
                document.write(funcs[i]() + "<br />");
            }

        </script>
     
    </body>
</html>
```

这里并没有直接把闭包赋值给数组，而是**通过定义一个匿名函数**，并**将立即执行该匿名函数的结果赋值给数组**。匿名函数中的参数`num`，就是最终函数要返回的值。
调用每个匿名函数时传入变量`i`，因为**参数是按值传递，所以会将变量i的当前值复制给参数**`num`。而在**这个匿名函数内部，又创建了一个访问**`num`**的闭包**。因此，`result`数组中**每个函数都有各自**`num`**的副本**，便能返回各自不同的数值。


