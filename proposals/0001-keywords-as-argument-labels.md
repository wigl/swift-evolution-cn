# 允许大部分关键词（keywords）可以作为参数名（argument labels）

* 提议: [SE-0001](0001-keywords-as-argument-labels.md)
* 开发者: [Doug Gregor](https://github.com/DougGregor)
* 状态: **已实现 (Swift 2.2)**
* Bug: [SR-344](https://bugs.swift.org/browse/SR-344)

## 引言

参数名用来描述函数参数的具体作用并提高可读性，是Swift函数接口中非常重要的一部分。有时用Swift中的关键词作为参数名更自然，如：`in`, `repear`或者`defer`等。为了更好的描述函数（接口），应该允许关键词可以作为参数名。

## 动机

对于一些函数，使用关键词作为参数名是最好的选择。比如从集合中找到某个值的索引，对于这个函数[^1]，最自然的命名就是`indexOf(_:in:)`

    indexOf(value, in: collection)

然而，`in`是一个关键词，所以实际上，必须使用反引号来转义`in`，就像下面：

    indexOf(value, `in`: collection)


所以当定义一个新API的时候，开发者不得不尝试使用其他非关键词作为参数名（如，上例中就要使用`within`），这不是很理想。另外，因为关键词不能作为函数参数名，Objective-C APIs映射到Swift时，也必须使用反引号来转义，比如：

    event.touchesMatching([.Began, .Moved], `in`: view)
    NSXPCInterface(`protocol`: SomeProtocolType.Protocol)

## 解决方案

允许所有关键词（除了 `inout`, `var`, 和 `let`）可以作为函数参数。有三个地方的语法受到影响：

* 表达式的调用，如上面的例子。这一块，我们没有语法上的歧义，因为表达式括号内的语法不会出现"<keyword> \`:\`"[^2]。到目前为止，这是最重要的一个例子。

* Function/subscript/initializer 的声明：除了上述三个（`inout`, `var`, 和 `let`）关键词外，没有语法上的歧义，因为关键词（作为参数名）总会带有标识符‘:’ 或者 ‘\_’。如：

```swift
func touchesMatching(phase: NSTouchPhase, in view: NSView?) -> Set<NSTouch>
```

当关键词用来描述参数时候，只有"inout", "let", 和 "var"还保留原有的含义。如果需要使用这三个关键词作为参数，还是需要转义：

```swift
func addParameter(name: String, `inout`: Bool)
```

* 函数类型：这其实比#2更容易，因为参数名总是带有标识符‘:’：[^3]

```swift
(NSTouchPhase, in: NSView?) -> Set<NSTouch>
(String, inout: Bool) -> Void
```

## 对已有代码的影响

这个改动是严格的增量操作（strictly additive），不会影响现有的代码：它只是使之前不能使用的错误代码得以正确使用，并且，不会改变代码的行为。

## 其他方案

最主要的替代方案就是，不做修改，保持现状：即使某个关键词作为参数名更合适，Swift APIs也需要避免该关键词；另外Objective-C APIs映射到Swift时候，关键词参数继续使用反引号转义或者重命名。这个方案会导致大量要映射的API（大概200个）需要重命名或者使用反引号转义。

第二个方案是，只针对`in`做处理。在Objective-C APIs映射到Swift时候，`in`是最常见的关键词参数，在一个简单的调研（Objective-C APIs的场景）中，`in`作为关键词参数的冲突占所有关键词的90%。另外，Swift语法中，`in`只有两个地方使用：循环和闭包，它和语法上下文严格相关。但是这个方案有点复杂（因为需要更多的分析的上下文关键字）并且不通用。


[^1]:这个函数在原文中，被描述为：module-scope function，不知道具体是什么意思。

[^2]:Here, we have no grammatic ambiguities, because " `:`" does not appear in any grammar production within a parenthesized expression list. 没有完全懂。

[^3]:函数类型Swift后续还做过修改，现阶段（Swift 5），这里下面两个例子，都无法编译通过。
