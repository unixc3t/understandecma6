###  块级绑定


从传统来讲变量声明方式已经成为js编程中一件棘手的事.在大多数类C语言中变量在声明的地方被创建 ,然而在js中不是这样,JS中你的变量真实被创建取决于你如何声明他们.Eamcscript6提供选项是控制范围更容易, 这一章演示为什么传统的变量声明容易误解,和ecmac6中提供了块级绑定并提供一些最佳实践如何使用他们

###### 变量声明和提升

函数里的变量使用var声明,无论他们出现在哪里,被视为在函数顶部声明(如果声明在函数外面就是在全局作用域),这叫做hoisting(提升)

一个示范证明提升做了什么,看下面函数定义

    function getValue(condition) {
        if (condition) {
          var value = "blue";
            // other code
          return value;
        } else {
          // value exists here with a value of undefined
          return null;
        }
        // value exists here with a value of undefined
    }

如果你不熟悉js,你或许期望变量仅在if condition执行结果为true时被创建, 但事实上, 变量被创建不论在哪, 在幕后,js引擎改变了getValue函数像下面这样

    function getValue(condition) {
        var value;
        if (condition) {
            value = "blue";
          // other code
            return value;
        } else {
                return null;
        }
    }

这个value变量声明被提升到函数的顶部, 初始化部分在原来地方,这意味着value变量仍然可以被else分支访问, 如果从else分支访问,便利哪个仅仅有一个undefined值, 因为还没有在else语句快初始化

对于一些新手需要适应变量提升 ,并且不理解这种行为导致BUG, 对于这种情况 ecma6引入了块级别作用域给开发者,更好的控制变量声明周期


###### 块级声明

块级声明,声明绑定在一个给定的块作用域 外面无法访问.block scopes 也叫做词法作用域 ,block scope被创建在下面两个地方
一个函数里
一个块里 (通过{}创建)

block scope在很多C类语言里都有使用, 在ecma6引入块级声明目的是提供对js一样的灵活性

###### let Declarations

let声明语法和 var语法一样, 你可以使用let替换var去声明一个变量,限制这个变量的作用域在当前代码快,(这有一些细微不同在 The Temporal Dead zone 那节详细讲解) let 声明不会提升到封闭代码快顶部 最好是在代码快里使用let声明 这样他们就适用于整个代码块 看下面例子

    function getValue(condition) {
      if (condition) {
          let value = "blue";
          // other code
          return value;
      } else {
          // value doesn't exist here
          return null;
      }
          // value doesn't exist here
      }

这个getValue函数行为更像你期望的其他C类语言那样 ,因为变量value被let声明 替代了var, 这个let声明不会导致提升到函数顶部, 这个变量的值也不允许在if语句块之外访问 if condition执行结果为false,那么value永远不会被声明或者初始化

###### 不要重复声明

如果一个标识符在一个作用域内已经被定义了,再使用let声明这个标识符在同样的作用域会抛出错误,
看下面例子

         var count = 30;

         // thorw an errors
         let count  = 40;

在这个例子里,count被声明了2次,一次使用 var,一次使用let,因为let不会重新定义一个已经存在的标识符,
在同一个作用域里. let声明会抛出一个错误.
相反,如果let创建一个新的变量使用同样的名字在它自己的作用域就不会抛出错误.看下面演示:

        var count = 30;

        if (contition) {
            //doesn't thorw an error
            let count = 40;

            // more code
        }

这个 let声明没有抛出一个错误,因为他创建了一个新的变量叫做count 在if语句块里,
替代了刚才var创建count的语句块,在if块内覆盖了全局的count,阻止访问全局count,
直到运行到if语句块外

###### 常量声明

在Ecma6中你可以使用const声明语法定义绑定, 使用const绑定声明被认为是常量,意味着他的值一旦设置
就不会改变,因为这个原因每个const绑定必须在声明的时候初始化 看下面例子

    // valid constant
    const maxItems = 30;
    // syntax error: missing initialization
    const name;

这个maxItems在绑定的时候初始化,所以这个const声明工作没有问题,然而这个name绑定会引起一个
语法错误,因为没有在声明的时候初始化


###### Constants vs. let Declarations

常量声明和let声明一样都是块级声明,这意味着常量不会被允许访问超过所声明的块级作用域,并且声明也不会被提升.看下面事例

    if (condition) {
        const maxItems = 5;
        // more code
    }
    // maxItems isn't accessible here

在这个代码里,常量maxItems被声明在一个if语句块里,在这个语句块执行完,maxItems不允许在块外面访问.

另一点类似let的是,一个常量声明使用在同一作用域里已经声明的变量会抛出一个错误,这个和是否变量被使用var(全局作用域或者函数作用域)或者let(块级作用域)无关,看下面例子

    var message = "Hello!";
    let age = 25;
    // each of these throws an error
    const message = "Goodbye!";
    const age = 30;

如果这两个常量单独声明是有效的,但是在这个例子里,前面已经有了var和let声明,所有这的语法声明是错误的.

虽然有这些相似之处,这里还是有一个重要的不同在let和const之间 ,尝试分配一个值给一个已经声明过得常量将会抛出一个错误,不管是在严格模式还是非严格模式

    const maxItems = 5;
    // throws an error
    maxItems = 6;

在尝试分配给一个声明过得常量之后,大多数语言不会存储这个新分配的值,但是这个值可以存放一个被修改的对象
