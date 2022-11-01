# 在Cairo中编程

## 你的第一个函数

让我们从下面这个计算数组中所有元素和的Cairo函数开始: 

```
// Computes the sum of the memory elements at addresses:
//   arr + 0, arr + 1, ..., arr + (size - 1).
func array_sum(arr: felt*, size) -> felt {
    if (size == 0) {
        return 0;
    }

    // size is not zero.
    let sum_of_rest = array_sum(arr=arr + 1, size=size - 1);
    return arr[0] + sum_of_rest;
}

```

前两行为注释, 它们会被编译器略过. Cairo 中的注释由 `//` 起始一直到行末.

第一行非注释的代码 `func array_sum(arr: felt*, size) -> felt {` 定义了一个名为 `array_sum` 的函数, 它接受两个参数: `arr` 和 `size`. 其中 `arr` 指向一个元素为 `size` 的数组. 而 `arr` 的类型 `felt*` 是一个指针 ( 要了解更多 `felt` 的信息, 查看[此处](https://www.cairo-lang.org/docs/hello_cairo/intro.html#field-element)  ). 这个函数声明了它将返回一个 field 元素 (`field`) . 函数的作用域到 `}` 符号为止.  ( 不过 `}` 并不是代表函数在此处返回 -- 请明确使用 return 语句返回函数值 ).

下一行是 `if (size == 0) {` 当 `size` 为 0 时, Cairo 将会执行 if 语句内声明的内容, 否则直接来到语句结束的地方.

当 `size == 0` 时, 表明此时数组中没有元素, 所以我们可以直接将 0 作为总和的结果返回. 与其它编程语言一样，return 语句会立刻结束当前函数的执行并将控制权返回给调用函数.

现在到有趣的部分了.  这行 `let sum_of_rest = array_sum(arr=arr + 1, size=size - 1);` ( 只会在 `size != 0` 的时候执行 )  递归调用了 `array_sum()` , 函数调用会从第二个元素开始 (因为 `arr` 指向数组的第一个 memory cell , 那么 `arr + 1` 会指向第二个. 此外请注意, 我们还需要再传递一个参数 `size - 1` ) , 并且会将结果储存在名为 `sum_of_rest` 的变量中. 另外, 尽管在上述代码中 `arr=` 和 `size=` 不是必须的, 但是考虑到代码可读性, 我们还是建议你写出来.  

函数内部的那些返回值会有一些限制（比如，它们可能会由于一些诸如跳转或函数调用的原因而被取消）. 稍后, 我们将看到如何通过使用局部变量来克服这个问题. 你可以通过阅读 [Revoked references](https://www.cairo-lang.org/docs/how_cairo_works/consts.html#revoked-references) 来进一步了解.

最终, 我们返回了通过将第一个元素与其它元素的和相加得到的总和. 你可以使用 `arr[0]` 或 `[arr]` 来获取数组中的第一个值 ( [...] 是解引用运算, 因此使用 `[arr]` 可以获取到储存在内存中 `arr` 地址处的值 ). 



## 函数练习

编写一个可以计算出数组中所有偶数项乘积的函数`(arr[0] * arr[2] * ...)`. 你可以假设数组长度是偶数. ( 你可能需要在阅读完 [使用 array_sum() 函数](#使用 array_sum()) 后, 才能开始运行这个函数. )

## 具有强大语法糖的低级语言

Cairo 并不属于高级语言. 我们更喜欢将其称之为: 一个可以通过一些强大的语法糖来编写可维护代码的低级语言. Cairo 的优点是你可以编写出非常高效的代码 ( 你可以用 Cairo 语言编写出几乎所有能 Cairo 机上运行的东西 ). 而主要缺点是, 有时有些常见的错误, 你需要清楚的知道自己在做什么才能避开它们. 不过不用担心, 本文档将指引你通过那些微妙的地方.

## 递归而不是循环

你也许注意到了, 我们在上面的代码中使用了递归, 而不是你可能期待的循环. 这么写的主要原因是 Cairo 中的内存是不可变的 - 一旦你在一个 memory cell 中写入了值, 那这个 cell 将无法再被改变. 这就像是那些纯函数式语言中的不可变对象, 在那里我们需要用递归代替循环, 出于相同的原因, 在这里我们也这么做.   

换种说法, 我们将会注意到在 Cairo 中循环结构 ( 在某些场景下 ) 是可行的, 但是它们是被限制的 ( 例如, 你不能在循环中调用函数 ) 并且它实现起来会更加复杂. 它们唯一的优点是它们往往比递归高效那么一点点.

## 断言

这是断言: 

```
assert <expr0> = <expr1>;
```

我们之后会用到, 它可以让我们做到两件事: 检验两个值是否相同 ( 是否是你预期的 ), 以及为一个 memory cell 分配一个值. 例如，`assert [ptr] = 0;` 会将地址 `ptr` 处的 memory cell 的值设置为 0 ( 如果它还未被赋值的话 ) . 这和 Cairo 内存不可变的特性有关: 而如果之前设置了值, 那么它将作为断言语句. 另一方面, 如果左侧的值 ( 在某些简单的情况下也可以是右侧 ) 尚未设置, Cairo 会将其作为赋值语句执行, 从而导致断言通过.

那么如果我之前已经设置了 `[ptr]` 的值, 我该如何更改它呢? 既然断言语句主要是用作断言而不是赋值的话, 是否还有其他方法更改值呢? 答案是不能更改 —— Cairo 内存是不可变的, 这意味着一旦一个值被写入一个 memory cell , 它就无法再被改变.

你可以在 [ 内存模型 ( Memory model )](https://www.cairo-lang.org/docs/how_cairo_works/cairo_intro.html#memory-model) 处获得更多信息. 

## 写一个 main() 函数

在编写可以调用 array_sum() 的 main() 函数之前, 让我们先从更简单的开始说起: 

```
%builtins output

from starkware.cairo.common.serialize import serialize_word

func main{output_ptr: felt*}() {
    serialize_word(1234);
    serialize_word(4321);
    return ();
}
```

在这里我们可以看到一些新组件: 

1. **main() 函数**: `main()` 函数是 Cairo 程序的起点.

2. **内置指令与 output 内置函数**: 指令 `%builtins output` 告诉 Cairo 编译器我们的程序将使用 Cairo 中的 "output" 内置函数. 你可以在[此处](https://www.cairo-lang.org/docs/how_cairo_works/builtins.html#builtins)了解有关内置函数的一些信息. 现在让我们先回到正题. 

   output 函数可以让我们与外界进行通信. 你可以将其比作 Python 中的 `print()` 或 C++ 里的 `std::cout` . 与所有内置函数一样, 在 Cairo 中使用它们没有什么要特别注意的 —— 与内置函数的通信是通过读/写内存值来完成的.

   output 函数的使用相当简单: 通过 `%builtins` 声明 output , 然后将 `main()` 的签名转换为 `main{output_ptr: feel*}()`. 语法 `{output_ptr: feel*}` 声明了一个“隐式参数”, 这意味着它在幕后添加了相应的参数和返回值. 有关隐式参数的更多信息可以在 [此处](https://www.cairo-lang.org/docs/how_cairo_works/builtins.html#implicit-arguments) 找到.

   这个参数指向程序输出时应写入的内存段的*开头*. 然后程序应该 *返回* 一个标记在输出 *结束* 处的指针. 在 Cairo 中我们遵守这样一个约定: 一个内存段的结尾总是指向最后一个被写入单元后面的那一个 memory cell. 实际上, 这正是 Cairo 所期望的返回值.

3. **serialize_word() 函数**: 为了将值 `x` 传入 output, 我们可以使用库函数 `serialize_word(x)`.  `serialize_word` 获得一个参数 ( 也就是我们要写入的值 ) 和一个隐式参数 `output_ptr` ( 这意味着在幕后它也返回一个值 ) . 事实上这个函数所做的事情非常简单: 它将 `x` 写入 `output_ptr` 指向的 memory cell ( 也就是 `[output_ptr]` ) 并返回 `output_ptr + 1`. 现在隐式参数机制启动: 在第一次调用 `serialize_word()` 时, Cairo 编译器通过 `output_ptr` 的值作为隐式参数. 而在第二次调用中, 它使用的隐式参数为第一次调用返回的值.

4. **导入语句**: 这行 `from starkware.cairo.common.serialize import serialize_word` 将指示编译器去编译文件 `starkware/cairo/common/serialize.cairo`, 并且导出定义好的标识符 `serialize_word` . 你可以使用 `... import serialize_word as foo`  来声明一个别名, 这样你就能在当前模块中使用别名来替代 `serialize_word` . 您可以在[此处](https://www.cairo-lang.org/docs/how_cairo_works/imports.html#imports)了解有关导入语句的更多信息. 

## 运行代码

将上一节的代码 ( 有 `main()` 函数的 ) 保存为 `array_sum.cairo` 文件 ( 稍后我们会将其改写为调用 `array_sum()` 的函数 ) , 并使用以下命令编译并运行: 

```
cairo-compile array_sum.cairo --output array_sum_compiled.json

cairo-run --program=array_sum_compiled.json \
    --print_output --layout=small
```

你将会看到: 

```
Program output:
  1234
  4321
```

`--layout` 参数是必需的, 因为我们要使用 output 内置函数, 它在普通布局中是不可用的 ( 请参阅 [布局](https://www.cairo-lang.org/docs/how_cairo_works/builtins.html#layouts) ) .

## 原始类型 - field element ( felt )

在 Cairo 中, 当您不指定变量/参数的类型时, 那么它的类型就是 **field element** ( 用关键字 `felt` 表示 ). 在 Cairo 的概念里, 当我们说 “a field element” , 我们想要表达的是

```
%builtins output

from starkware.cairo.common.serialize import serialize_word

func main{output_ptr: felt*}() {
    serialize_word(6 / 3);
    serialize_word(7 / 3);
    return ();
}
```





## 使用 array_sum()

现在, 让我们来写一个调用了 `array_sum()` 的 `main()` 函数. 但是首先, 我们需要先为数组分配空间. 通过使用库函数 `alloc()` 我们可以轻易的做到这一点: 

```
%builtins output

from starkware.cairo.common.alloc import alloc
from starkware.cairo.common.serialize import serialize_word

func array_sum(arr: felt*, size) -> felt {
    // ...
}

func main{output_ptr: felt*}() {
    const ARRAY_SIZE = 3;

    // Allocate an array.
    let (ptr) = alloc();

    // Populate some values in the array.
    assert [ptr] = 9;
    assert [ptr + 1] = 16;
    assert [ptr + 2] = 25;

    // Call array_sum to compute the sum of the elements.
    let sum = array_sum(arr=ptr, size=ARRAY_SIZE);

    // Write the sum to the program output.
    serialize_word(sum);

    return ();
}
```



这里有几个新的东西: 

1. **内存分配**: 我们使用标准库中的函数 `alloc()` 来分配一个新的内存段. 实际上, 只有在程序终止时内存分配的位置才会被确定下来, 这样可以避免指定分配内存的大小.
2. **常量**: 在 Cairo 中常量是通过 `const CONST_NAME = <expr>;` 来定义的, 其中 `<expr>` 必须是一个在编译期间已知的整数 ( 准确的说, 是一个 field 元素 ) . 

编译并运行代码 ( 别忘了将本章开头的 `array_sum()` 函数复制进来 ) . 我们应该可以看到: 

```
Program output:
  50
```



## 练习

现在你可以尝试使用上面的 `main()` 函数运行[你的函数](#函数练习). 别忘了将元素的数量调整为偶数.

