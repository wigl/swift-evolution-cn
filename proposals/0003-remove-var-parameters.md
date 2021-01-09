# 从函数参数中移除`var`

* 提议: [SE-0003](0003-remove-var-parameters.md)
* 开发者: [David Farler](https://github.com/bitjammer)
* Review Manager: [Joe Pamer](https://github.com/jopamer)
* 状态: **已实现 (Swift 3)**
* 决策记录: [Rationale](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/008145.html)
* Implementation: [apple/swift@8a5ed40](https://github.com/apple/swift/commit/8a5ed405bf1f92ec3fc97fa46e52528d2e8d67d9)

## 注意

该提议经历了一些重大改动（相比最初的提议）。文档最后面有历史信息以及更改的原因。

## 引言

函数参数标记为`inout`(相比`var`）有一些语法上的歧义。这两个关键词都提供了copy过来的可变的局域变量，但是当参数标记为`inout`后，值会自动回写。

函数参数默认为不可变：

```swift
func foo(i: Int) {
  i += 1 // 错误
}

func foo(var i: Int) {
  i += 1 // OK, 但是调用方不能获取到该变化
}
```
上面的例子，copy过来的*局部*变量`x`是可变的，但是不会重回写到原始的入参，所以调用方永远不会获取到变化。为了使值类型可以回写，必须把参数标记为`inout`：

```swift
func doSomethingWithVar(var i: Int) {
  i = 2 // 局部变量修改了，但是调用方传入的Int值不会被修改。
}

func doSomethingWithInout(inout i: Int) {
  i = 2 // 会修改调用方传入的Int值。
}

var x = 1
print(x) // 1

doSomethingWithVar(x)
print(x) // 1

doSomethingWithInout(&x)
print(x) // 2
```

## 动机
使用`var`标记函数参数作用很有限，使一行代码最优化但是和`inout`造成歧义[<sup>1</sup>](#refer-anchor-1)，为了强调值是唯一的copy以及相比`inout`不会回写，在这里，我们应该不允许使用`var`

总的来说，促使修改的原因是：

- 在函数参数中，`var`和`inout`经常造成歧义。

- `var` 造成值类型有引用类型的歧义。

- 函数参数不像*if-*,
*while-*, *guard-*, *for-in-*, 和 *case* 这些场景下的模式匹配（refutable patterns）[<sup>2</sup>](#refer-anchor-2)。

## 方案

对语法分析而言，这个改动很严格/狭隘/不重要（trivial）。语法分析会在Swift 2.2显示警告，但是将在Swift 3显示为错误。

## 对现有代码的影响

由于只是简单的移除了`var`，可以立刻声明一个临时的变量，用来覆盖掉不可变的copy。例如：

```swift
func foo(i: Int) {
  var i = i
}
```

然而，覆盖并不是一个理想的解决方式，而且可能是一个反面模式（anti-pattern）。我们希望Swift用户重新思考（但也不是完全必须）他们使用了这样方法的代码。

## 其他方案

该提议最初是要移除所有的`var`模式匹配绑定，以及函数参数。

[Original SE-0003 Proposal](https://github.com/apple/swift-evolution/blob/8cd734260bc60d6d49dbfb48de5632e63bf200cc/proposals/0003-remove-var-parameters-patterns.md)

从模式绑定中移除`var`被重新思考，因为Swift 2模式匹配已经使用了`var`，突变修改可能造成负担。你可以在 swift-evolution的邮件列表查看讨论[<sup>3</sup>](#refer-anchor-3)：

[Initial Discussion of Reconsideration](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160118/007326.html)

最后的结论也发送到了swift-evolution列表，你可以在这里查看：
[Note on Revision of the Proposal](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/008145.html)

<div id="refer-anchor-1"></div>
[1]: 翻译的有问题。（Using var annotations on function parameters have limited utility, optimizing for a line of code at the cost of confusion with inout, which has the semantics most people expect. ）

<div id="refer-anchor-2"></div>
[2]: Function parameters are not refutable patterns like in if-, while-, guard-, for-in-, and case statements. 没完全懂。

<div id="refer-anchor-3"></div>
[3]: 翻译不准确，模式匹配没有移除的原因？
