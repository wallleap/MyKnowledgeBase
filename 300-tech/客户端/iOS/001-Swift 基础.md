---
title: 001-Swift 基础
date: 2024-08-21T01:57:26+08:00
updated: 2024-08-21T03:49:27+08:00
dg-publish: false
---

## 打印语句

打印“Hello, world”

```swift
print("Hello, world!")
```

## 变量声明

使用 `let` 来声明常量，使用 `var` 来声明变量

```swift
var myVariable = 42
myVariable = 50
let myConstant = 42
```

常量和变量名可以包含几乎所有的字符，包括 Unicode 字符

- 不能包含数学符号、箭头、保留的或非法的 Unicode 码、连线、制表符
- 不能以数字开头

## 数据类型

你不用明确地声明类型，编译器会自动推断其类型

在声明变量或常量的时候可以加上类型注解，用冒号分割

### 数值

- `Int` 整型：`42`、`0`、`-23`、`+12`
	- 提供了 8、16、32 和 64 位的有符号和无符号整数类型（`Int8` 有符号 `UInt8` 无符号，以此类推）
	- 不需要专门指定整数的长度，特殊整数类型 `Int`，它和当前平台的原生字长相同（在 32 位平台上，`Int` 和 `Int32` 长度相同，在 64 位平台上，`Int` 和 `Int64` 长度相同）
	- 特殊无符号整型 `UInt`，长度也和平台相关
- `Double` 和 `Float` 浮点型：`3.14159`、`0.1` 和 `-273.15`
	- `Double` 表示 64 位浮点数。当你需要存储很大或者很高精度的浮点数时请使用此类型（浮点数推断时会默认是这个）
	- `Float` 表示 32 位浮点数。精度要求不高的话可以使用此类型

```swift
let explicitDouble: Double = 70
```

可以通过 `min` 和 `max` 属性获取对应类型的最小最大值

```swift
UInt8.min  // 8位无符号整型
Int32.max  // 32位有符号整型
```

> **类型安全**的语言，会进行类型检查
> 声明变量或常量的时候如果没有显式指定类型，会进行**类型推断**

整数字面量可以被写作：

- 一个 *十进制* 数，没有前缀
- 一个 *二进制* 数，前缀是 `0b`
- 一个 *八进制* 数，前缀是 `0o`
- 一个 *十六进制* 数，前缀是 `0x`

```swift
let decimalInteger = 17
let binaryInteger = 0b10001       // 二进制的17
let octalInteger = 0o21           // 八进制的17
let hexadecimalInteger = 0x11     // 十六进制的17
```

浮点数需要加上特定的字母

```swift
let decimalDouble = 12.1875
let exponentDouble = 1.21875e1  // 10进制 → e
let hexadecimalDouble = 0xC.3p0  // 16进制 → p
```

整数和浮点数都可以添加额外的零并且包含下划线来增强可读性

```swift
let paddedDouble = 000123.456
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
```

> **类型别名**：可以使用 `typealias` 给现有类型定义另一个名字

```swift
typealias AudioSample = UInt16
var maxAmplitudeFound = AudioSample.min
// maxAmplitudeFound 现在是 0
```

### 字符串

- `String` 文本型

字符串字面量

- 单行 `""`
- 多行 `"""   """`

`String` 显示转换成字符串

```swift
let label = "The width is "
let width = 94
let widthLabel = label + String(width)
```

**字符串插值**：把值写到括号中，并且在括号之前写一个反斜杠（`\`），也可以转成字符串

```swift
let apples = 3
let oranges = 5
let appleSummary = "I have \(apples) apples."
let fruitSummary = "I have \(apples + oranges) pieces of fruit."
```

使用三个双引号（`"""`）来包含多行字符串内容，每行行首的缩进会被去除，只要和结尾引号的缩进相匹配，在行尾写一个反斜杠（`\`）作为续行符就不会换行

```swift
let quotation = """
I said "I have \(apples) apples."
And then I said "I have \(apples + oranges) pieces of fruit."
"""
```

特殊字符

- 转义字符 `\0`(空字符)、`\\`(反斜线)、(水平制表符)、(换行符)、(回车符)、`\"`(双引号)、`\'`(单引号)。
- Unicode 标量，写成 `\u{n}`(u 为小写)，其中 `n` 为任意一到八位十六进制数且可用的 Unicode 位码。

将字符串放在引号（`"`）中并用数字符号（`#`）括起来，字符串中的特殊字符将会被直接包含而非转义

### 布尔

- `Bool`：`true` 和 `false`

和其他语言不一样，进行逻辑判断的时候，并不会将其他类型转成 Bool 类型，而是会报错（控制语句中有示例），其他类型就只能进行值的比较运算返回 Bool

### 元组 tuples

把多个值组合成一个复合值

```swift
let http404Error = (404, "Not Found")
// http404Error 的类型是 (Int, String)，值是 (404, "Not Found")

// 可以通过下标进行访问
print("The status code is \(http404Error.0)")
// 输出“The status code is 404”
print("The status message is \(http404Error.1)")
// 输出“The status message is Not Found”
```

将一个元组的内容分解（decompose）成单独的常量和变量

```swift
let (statusCode, statusMessage) = http404Error
print("The status code is \(statusCode)")
// 输出“The status code is 404”
print("The status message is \(statusMessage)")
// 输出“The status message is Not Found”
```

分解的时候可以把要忽略的部分用下划线（`_`）标记

```swift
let (justTheStatusCode, _) = http404Error
print("The status code is \(justTheStatusCode)")
// 输出“The status code is 404”
```

在定义元组的时候给单个元素命名

```swift
let http200Status = (statusCode: 200, description: "OK")

// 这样可以通过名字来获取元素的值
print("The status code is \(http200Status.statusCode)")
// 输出“The status code is 200”
print("The status message is \(http200Status.description)")
// 输出“The status message is OK”
```

### 可选类型

正常**类型后加问号**，代表可能会没有值，没有值的时候自动复制 `nil`

```swift
var serverResponseCode: Int? = 404  // 一定要可选
serverResponseCode = nil  // 可以手动设置为 nil

var surveyAnswer: String?  // 会自动设置为 nil
```

判断的时候是通过比较运算符 `a != nil`

隐式解析可选类型

```swift
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // 需要感叹号来获取值

let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString  // 不需要感叹号
```

### 集合类型

- `Array`
- `Set`
- `Dictionary`

使用方括号 `[]` 来创建数组和字典，并使用下标或者键（key）来访问元素

```swift
var shoppingList = ["catfish", "water", "tulips", "blue paint"]
shoppingList[1] = "bottle of water"

var occupations = [
    "Malcolm": "Captain",
    "Kaylee": "Mechanic",
]
occupations["Jayne"] = "Public Relations"
shoppingList.append("blue paint")  // append 添加元素
```

使用初始化语法来创建一个空数组或者空字典。

```swift
let emptyArray: [String] = []
let emptyDictionary: [String: Float] = [:]
```

如果类型信息可以被推断出来，你可以用 `[]` 和 `[:]` 来创建空数组和空字典——比如，在给变量赋新值或者给函数传参数的时候。

```swift
shoppingList = []
occupations = [:]
```

## 注释

```swift
// 单行注释

/* 多
行
注
释
*/

// 注释能嵌套：
/* 这是第一个多行注释的开头
/* 这是第二个被嵌套的多行注释 */
这是第一个多行注释的结尾 */
```

## 运算符

### 赋值运算符

```swift
// 初始化或更新值
let b = 10
var a = 5
a = b
// a 现在等于 10

// 赋值的右边是一个多元组，它的元素可以马上被分解成多个常量或变量
let (x, y) = (1, 2)
// 现在 x 等于 1，y 等于 2
```

赋值操作并不返回任何值，`if x = y` 为无效语句，能帮你避免把 （ == ）错写成（ = ）这类错误的出现

### 算术运算符

```swift
1 + 2       // 等于 3
5 - 3       // 等于 2
2 * 3       // 等于 6
10.0 / 2.5  // 等于 4.0
```

默认情况下不允许在数值运算中出现溢出情况。但是你可以使用 Swift 的溢出运算符来实现溢出运算（如 `a &+ b`）

加法运算符也可用于 `String` 的拼接：

```swift
"hello, " + "world"  // 等于 "hello, world"
```

### 求余

在 Swift 中可以表达为：

```swift
9 % 4    // 等于 1

-9 % 4   // 等于 -1
```

在对负数 `b` 求余时，`b` 的符号会被忽略。这意味着 `a % b` 和 `a % -b` 的结果是相同的

### 一元运算符

```swift
let three = 3
let minusThree = -three       // minusThree 等于 -3
let plusThree = -minusThree   // plusThree 等于 3, 或 "负负3"

let minusSix = -6
let alsoMinusSix = +minusSix  // alsoMinusSix 等于 -6
```

### 组合赋值运算符

其他运算符和赋值运算（ = ）组合

```swift
var a = 1
a += 2
// a 现在是 3
```

表达式 `a += 2` 是 `a = a + 2` 的简写，一个组合加运算就是把加法运算和赋值运算组合成进一个运算符里，同时完成两个运算任务。

复合赋值运算没有返回值，`let b = a += 2` 这类代码是错误

### 比较运算符 Comparison Operators

```swift
1 == 1   // true, 因为 1 等于 1
2 != 1   // true, 因为 2 不等于 1
2 > 1    // true, 因为 2 大于 1
1 < 2    // true, 因为 1 小于2
1 >= 1   // true, 因为 1 大于等于 1
2 <= 1   // false, 因为 2 并不小于等于 1
```

恒等（ === ）和不恒等（ !== ）这两个比较符来判断两个对象是否引用同一个对象实例

如果两个元组的元素相同，且长度相同的话，元组就可以被比较。比较元组大小会按照从左到右、逐值比较的方式，直到发现有两个值不等时停止。如果所有的值都相等，那么这一对元组我们就称它们是相等的。

### 三元运算符

```swift
let contentHeight = 40
let hasHeader = true
let rowHeight = contentHeight + (hasHeader ? 50 : 20)
// rowHeight 现在是 90
```

### 空合运算符 Nil Coalescing Operator

*空合运算符*（`a ?? b`）将对可选类型 `a` 进行空判断，如果 `a` 包含一个值就进行解包，否则就返回一个默认值 `b`

表达式 `a` 必须是 Optional 类型。默认值 `b` 的类型必须要和 `a` 存储值的类型保持一致。

空合运算符是对以下代码的简短表达方法：

```swift
a != nil ? a! : b
```

### 区间运算符

- *闭区间运算符*（`a...b`）定义一个包含从 `a` 到 `b`（包括 `a` 和 `b`）的所有值的区间。`a` 的值不能超过 `b`
- *半开区间运算符*（`a..<b`）定义一个从 `a` 到 `b` 但不包括 `b` 的区间。 之所以称为*半开区间*，是因为该区间包含第一个值而不包括最后的值。
- 单侧区间：往一侧无限延伸的区间
	- `2...`
	- `...2`
	- `..<2`

### 逻辑运算符

- 逻辑非（`!a`）：对一个布尔值取反
- 逻辑与（`a && b`）：只有 `a` 和 `b` 的值都为 `true` 时，整个表达式的值才会是 `true`
- 逻辑或（`a || b`）：其中一个为 `true`，整个表达式就为 `true`

逻辑与和逻辑或都是「短路计算」，只有左边有一个不满足条件了就会立马输出结果，后面的表达式不会再进行计算

### 括号

在合适的地方使用括号来明确优先级

```swift
if (enteredDoorCode && passedRetinaScan) || hasDoorKey || knowsOverridePassword {
	print("Welcome!")
} else {
	print("ACCESS DENIED")
}
// 输出“Welcome!”
```

## 语句

Swift 并不强制要求你在每条语句的结尾处使用分号（`;`）
