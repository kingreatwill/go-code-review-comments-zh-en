# Go Code Review Comments

This page collects common comments made during reviews of Go code, so
that a single detailed explanation can be referred to by shorthands.
This is a laundry list of common mistakes, not a comprehensive style guide.

You can view this as a supplement to [Effective Go](https://golang.org/doc/effective_go.html).

[英文原文](https://github.com/golang/go/wiki/CodeReviewComments)

# GO代码审查注释

该页面收集了在Go代码的审阅期间提出的常见注释，因此，速记可以引用一个详细的说明。 这是常见错误的清单，而不是全面的规范指南。

您可以将此作为[《Effective Go》中英双语版](https://github.com/bingohuang/effective-go-zh-en)的补充。

**Please [discuss changes](https://golang.org/issue/new?title=wiki%3A+CodeReviewComments+change&body=&labels=Documentation) before editing this page**, even _minor_ ones. Many people have opinions and this is not the place for edit wars.

**编辑此页面之前请先[讨论更改](https://golang.org/issue/new?title=wiki%3A+CodeReviewComments+change&body=&labels=Documentation)**。 许多人都有意见，这不是编辑争论的地方。

* [Gofmt](#gofmt)
* [Comment Sentences - 注释语句](#comment-sentences)
* [Contexts - 上下文](#contexts)
* [Copying - 拷贝](#copying)
* [Crypto Rand - 密码随机数](#crypto-rand)
* [Declaring Empty Slices - 声明空切片](#declaring-empty-slices)
* [Doc Comments - 文档注释](#doc-comments)
* [Don't Panic - 不要使用panic](#dont-panic)
* [Error Strings - 错误字符串](#error-strings)
* [Examples - 示例](#examples)
* [Goroutine Lifetimes - Goroutine生命周期](#goroutine-lifetimes)
* [Handle Errors - 处理错误](#handle-errors)
* [Imports - 导入](#imports)
* [Import Blank - 导入_](#import-blank)
* [Import Dot - 导入.](#import-dot)
* [In-Band Errors - 带内的错误](#in-band-errors)
* [Indent Error Flow - 缩进错误流](#indent-error-flow)
* [Initialisms - 首字母缩略词](#initialisms)
* [Interfaces - 接口](#interfaces)
* [Line Length - 单行代码长度](#line-length)
* [Mixed Caps - 组合名](#mixed-caps)
* [Named Result Parameters - 命名返回参数](#named-result-parameters)
* [Naked Returns](#naked-returns)
* [Package Comments - 包注释](#package-comments)
* [Package Names - 包名](#package-names)
* [Pass Values - 传值](#pass-values)
* [Receiver Names - 接收者名称](#receiver-names)
* [Receiver Type - 接收者类型](#receiver-type)
* [Synchronous Functions - 同步函数](#synchronous-functions)
* [Useful Test Failures - 有用的测试失败信息](#useful-test-failures)
* [Variable Names - 变量名](#variable-names)

## Gofmt

Run [gofmt](https://golang.org/cmd/gofmt/) on your code to automatically fix the majority of mechanical style issues. Almost all Go code in the wild uses `gofmt`. The rest of this document addresses non-mechanical style points.

An alternative is to use [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports), a superset of `gofmt` which additionally adds (and removes) import lines as necessary.

## Gofmt

你可以在你的代码中运行 [Gofmt](https://golang.org/cmd/gofmt/) 以解决大多数代码格式问题。几乎所有的代码都使用 `gofmt`。

另一种方法是使用 [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports)，它是 `gofmt`的超集，可根据需要添加（删除）行。
  

## Comment Sentences

See https://golang.org/doc/effective_go.html#commentary.  Comments documenting declarations should be full sentences, even if that seems a little redundant.  This approach makes them format well when extracted into godoc documentation.  Comments should begin with the name of the thing being described and end in a period:

```go
// Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

and so on.

## 注释语句

参考 https://github.com/bingohuang/effective-go-zh-en/blob/master/03_Commentary.md. 记录声明的注释应该是完整的句子，即使这看起来有点多余。 当提取到godoc文档中时，这种方法可以使它们格式化良好。 注释应以所描述事物的名称开头，并以句点结尾：
```go
// Request 表示运行命令的请求。
type Request struct { ...

// Encode 将req的JSON编码写入w。
func Encode(w io.Writer, req *Request) { ...
```
等等。

## Contexts

Values of the context.Context type carry security credentials,
tracing information, deadlines, and cancellation signals across API
and process boundaries. Go programs pass Contexts explicitly along
the entire function call chain from incoming RPCs and HTTP requests
to outgoing requests.

Most functions that use a Context should accept it as their first parameter:

```
func F(ctx context.Context, /* other arguments */) {}
```

A function that is never request-specific may use context.Background(),
but err on the side of passing a Context even if you think you don't need
to. The default case is to pass a Context; only use context.Background()
directly if you have a good reason why the alternative is a mistake.

Don't add a Context member to a struct type; instead add a ctx parameter
to each method on that type that needs to pass it along. The one exception
is for methods whose signature must match an interface in the standard library
or in a third party library.

Don't create custom Context types or use interfaces other than Context in
function signatures.

If you have application data to pass around, put it in a parameter,
in the receiver, in globals, or, if it truly belongs there, in a Context value.

Contexts are immutable, so it's fine to pass the same ctx to multiple
calls that share the same deadline, cancellation signal, credentials,
parent trace, etc.

## 上下文

上下文的值。上下文类型带有安全凭证，跨API跟踪信息，截止日期和取消信号和过程边界。 Go程序沿显式传递上下文来自传入的RPC和HTTP请求的整个函数调用链发出请求。

使用上下文的大多数函数都应将其作为第一个参数：
```
func F(ctx context.Context, /* 其它参数 */) {}
```
从不做特定请求的函数可以使用 context.Background()，但即使您认为不需要，也可以在传递Context时使用错误(error)。如果您有充分的理由认为替代方案是错误的，那么只能直接使用context.Background()。

不要将 Contexts 放入结构体，context应该作为第一个参数传入。 不过也有例外，函数签名(method signature)必须与标准库或者第三方库中的接口相匹配。

不要在函数签名中创建Context类型或者使用Context以外的接口。

如果您有应用程序数据要传递，请将其放在参数中，接收器中，全局变量中，或者如果它确实属于该参数，则在Context值中。使用context的Value相关方法只应该用于在程序和接口中传递的和请求相关的元数据，不要用它来传递一些可选的参数

上下文是不可变的，所以可以将相同的ctx传递给共享相同截止日期、取消信号、凭据、父跟踪等的多个调用。

## Copying

To avoid unexpected aliasing, be careful when copying a struct from another package.
For example, the bytes.Buffer type contains a `[]byte` slice. If you copy a `Buffer`,
the slice in the copy may alias the array in the original, causing subsequent method
calls to have surprising effects.

In general, do not copy a value of type `T` if its methods are associated with the
pointer type, `*T`.

## 拷贝

为了避免意外的别名，从另一个包复制结构时要小心。比如， `bytes.Buffer` 类型包含了 `[]byte` 切片类型，并且作为小字符串的优化，可被较小的字节数组引用。如果你拷贝了一个`Buffer`，拷贝中的切片可能会alias原始数组中的切片，从而导致后续的操作带来不可预测的结果。

通常来说，如果一个类型 `T`其方法与指针结构相关，那么请不要拷贝 `T`的值。

## Crypto Rand

Do not use package `math/rand` to generate keys, even throwaway ones.
Unseeded, the generator is completely predictable. Seeded with `time.Nanoseconds()`,
there are just a few bits of entropy. Instead, use `crypto/rand`'s Reader,
and if you need text, print to hexadecimal or base64:

``` go
import (
	"crypto/rand"
	// "encoding/base64"
	// "encoding/hex"
	"fmt"
)

func Key() string {
	buf := make([]byte, 16)
	_, err := rand.Read(buf)
	if err != nil {
		panic(err)  // out of randomness, should never happen
	}
	return fmt.Sprintf("%x", buf)
	// or hex.EncodeToString(buf)
	// or base64.StdEncoding.EncodeToString(buf)
}
```
## 密码随机数
请不要使用包 `math/rand` 来生成密钥，即使是一次性的。如果不提供种子，则密钥完全可以被预测到。就算用`time.Nanoseconds()`作为种子，也仅仅只有几个位上的差别。

使用`crypto/rand`'s Reader作为代替。并且如果你需要生成文本，打印成16进制或者base64类型即可。
``` go
import (
	"crypto/rand"
	// "encoding/base64"
	// "encoding/hex"
	"fmt"
)

func Key() string {
	buf := make([]byte, 16)
	_, err := rand.Read(buf)
	if err != nil {
		panic(err)  // 出于随机，永远都不会发生
	}
	return fmt.Sprintf("%x", buf)
	// or hex.EncodeToString(buf)
	// or base64.StdEncoding.EncodeToString(buf)
}
```


## Declaring Empty Slices

When declaring an empty slice, prefer

```go
var t []string
```

over 

```go
t := []string{}
```

The former declares a nil slice value, while the latter is non-nil but zero-length. They are functionally equivalent—their `len` and `cap` are both zero—but the nil slice is the preferred style.

Note that there are limited circumstances where a non-nil but zero-length slice is preferred, such as when encoding JSON objects (a `nil` slice encodes to `null`, while `[]string{}` encodes to the JSON array `[]`).

When designing interfaces, avoid making a distinction between a nil slice and a non-nil, zero-length slice, as this can lead to subtle programming errors.

For more discussion about nil in Go see Francesc Campoy's talk [Understanding Nil](https://www.youtube.com/watch?v=ynoY2xz-F8s).

## 声明空切片

当声明一个空切片时， 使用

```go
var t []string
```

而不是

```go
t := []string{}
```
前者声明了一个指向nil的切片，后者声明了一个 non-nil 且 len = 0的切片。虽然他们的功能是等价的：它们的 `len` 和`cap` 都是0，但nil切片应当作为首选方案。

请注意在有些情况下， non-nil 切片表现更好，比如当解码JSON对象时，nil切片解码为 `null`, []string{} 切片解码为 JSON 数组[]。

在设计接口时，应当避免因nil切片和non-nil切片带来的不同表现，因为这可能会导致细微的编码错误。

有关nil的更多讨论，参考 [Understanding Nil](https://www.youtube.com/watch?v=ynoY2xz-F8s).


## Doc Comments

All top-level, exported names should have doc comments, as should non-trivial unexported type or function declarations. See https://golang.org/doc/effective_go.html#commentary for more information about commentary conventions.

## 文档注释

所有顶级的导出名称都应具有文档注释，非平凡的未导出类型或函数声明也应具有文档注释。 有关注释约定的更多信息，请参见 https://github.com/bingohuang/effective-go-zh-en/blob/master/03_Commentary.md 。

## Don't Panic

See https://golang.org/doc/effective_go.html#errors. Don't use panic for normal error handling. Use error and multiple return values.

## 不要使用Panic

请参见 https://github.com/bingohuang/effective-go-zh-en/blob/master/15_Errors.md 。不要对常规错误处理使用panic。 使用错误和多个返回值。

## Error Strings

Error strings should not be capitalized (unless beginning with proper nouns or acronyms) or end with punctuation, since they are usually printed following other context. That is, use `fmt.Errorf("something bad")` not `fmt.Errorf("Something bad")`, so that `log.Printf("Reading %s: %v", filename, err)` formats without a spurious capital letter mid-message. This does not apply to logging, which is implicitly line-oriented and not combined inside other messages.

## 错误字符串

错误字符串不应大写（除非以专有名词或缩写开头）或标点符号结束，因为它们通常是在其他上下文之后打印的。 也就是说，使用`fmt.Errorf("something bad")`而不是`fmt.Errorf("Something bad")`，以便`log.Printf("Reading %s: %v", filename, err)`格式不包含中间的假大写字母 -信息。 这不适用于日志记录，后者是隐式面向行的，并且未在其他消息中合并。

## Examples

When adding a new package, include examples of intended usage: a runnable Example,
or a simple test demonstrating a complete call sequence.

Read more about [testable Example() functions](https://blog.golang.org/examples).

## 示例
添加新程序包时，请包括预期用法的示例：可运行的示例或演示完整调用序列的简单测试。

阅读更多[示例函数的用法](https://blog.golang.org/examples)

## Goroutine Lifetimes

When you spawn goroutines, make it clear when - or whether - they exit.

Goroutines can leak by blocking on channel sends or receives: the garbage collector
will not terminate a goroutine even if the channels it is blocked on are unreachable.

Even when goroutines do not leak, leaving them in-flight when they are no longer
needed can cause other subtle and hard-to-diagnose problems. Sends on closed channels
panic. Modifying still-in-use inputs "after the result isn't needed" can still lead
to data races. And leaving goroutines in-flight for arbitrarily long can lead to
unpredictable memory usage.

Try to keep concurrent code simple enough that goroutine lifetimes are obvious.
If that just isn't feasible, document when and why the goroutines exit.

## Goroutine生命周期
当你需要使用goruntines时，请确保它们什么时候/什么条件下退出。

Goroutines可能会因阻塞channel的send或者receives而泄露,即使被阻塞的通道无法访问，垃圾收集器也不会终止goroutine。

即使gorountine没有泄露，当不再需要它们时仍在继续运行会导致难以诊断的问题。即使在不需要结果后，修改正在使用的数据也会导致数据竞争。

尽量保证并发代码足够简单，使gorountine的生命周期更明显。如果难以做到，请记录gorountines退出的时间和原因。

## Handle Errors

See https://golang.org/doc/effective_go.html#errors. Do not discard errors using `_` variables. If a function returns an error, check it to make sure the function succeeded. Handle the error, return it, or, in truly exceptional situations, panic.

## 处理错误

请参见 https://github.com/bingohuang/effective-go-zh-en/blob/master/15_Errors.md  。不要使用_变量丢弃错误。 如果函数返回错误，请检查以确保函数成功。 处理错误，将其返回，或者在真正异常的情况下，应紧急处理。

## Imports

Avoid renaming imports except to avoid a name collision; good package names
should not require renaming. In the event of collision, prefer to rename the most
local or project-specific import.


Imports are organized in groups, with blank lines between them.
The standard library packages are always in the first group.

```go
package main

import (
	"fmt"
	"hash/adler32"
	"os"

	"appengine/foo"
	"appengine/user"

	"github.com/foo/bar"
	"rsc.io/goversion/version"
)
```

<a href="https://godoc.org/golang.org/x/tools/cmd/goimports">goimports</a> will do this for you.

## 导入

尽量避免导入包时的重命名，以避免名称冲突。如果发生名称冲突，尽量重命名本地或项目特定的包。

导入包按名称分组，用空行隔开。

标准库包应始终位于第一组。
```go
package main

import (
	"fmt"
	"hash/adler32"
	"os"

	"appengine/foo"
	"appengine/user"

	"github.com/foo/bar"
	"rsc.io/goversion/version"
)
```
可以使用 [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) 来规范包的排序。

## Import Blank

Packages that are imported only for their side effects (using the syntax `import _ "pkg"`) should only be imported in the main package of a program, or in tests that require them.

## 导入`_`
仅出于副作用而导入的软件包（使用语法import _“ pkg”）应仅在程序的主软件包或需要它们的测试中导入。

## Import Dot

The import . form can be useful in tests that, due to circular dependencies, cannot be made part of the package being tested:

```go
package foo_test

import (
	"bar/testutil" // also imports "foo"
	. "foo"
)
```

In this case, the test file cannot be in package foo because it uses bar/testutil, which imports foo.  So we use the 'import .' form to let the file pretend to be part of package foo even though it is not.  Except for this one case, do not use import . in your programs.  It makes the programs much harder to read because it is unclear whether a name like Quux is a top-level identifier in the current package or in an imported package.

## 导入`.`
使用 import . 来导入包在测试中很有用，但由于循环依赖性，它不能成为被测试包的一部分：
```go
package foo_test

import (
	"bar/testutil" // also imports "foo"
	. "foo"
)
```
在这种情况下，测试文件不能在包foo中，因为它使用了同样导入foo的包 bar/testutil，所以我们使用import .格式导入来让测试文件伪装成包foo的一部分，即使它并不是。除了这一情况，不要在你的程序中使用import .来导入包。它将使程序更难阅读，因为不清楚 Quux 这样的名称是在当前包中还是导入包中。

## In-Band Errors

In C and similar languages, it's common for functions to return values like -1 
or null to signal errors or missing results:

```go
// Lookup returns the value for key or "" if there is no mapping for key.
func Lookup(key string) string

// Failing to check a for an in-band error value can lead to bugs:
Parse(Lookup(key))  // returns "parse failure for value" instead of "no value for key"
```

Go's support for multiple return values provides a better solution.
Instead of requiring clients to check for an in-band error value, a function should return
an additional value to indicate whether its other return values are valid. This return
value may be an error, or a boolean when no explanation is needed.
It should be the final return value.

``` go
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

This prevents the caller from using the result incorrectly:

``` go
Parse(Lookup(key))  // compile-time error
```

And encourages more robust and readable code:

``` go
value, ok := Lookup(key)
if !ok {
	return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

This rule applies to exported functions but is also useful
for unexported functions.

Return values like nil, "", 0, and -1 are fine when they are
valid results for a function, that is, when the caller need not
handle them differently from other values.

Some standard library functions, like those in package "strings",
return in-band error values. This greatly simplifies string-manipulation
code at the cost of requiring more diligence from the programmer.
In general, Go code should return additional values for errors.

## 带内的错误
在C和类似语言中，函数通常返回-1或null之类的值以表示错误或结果丢失：

```go
// Lookup returns the value for key or "" if there is no mapping for key.
func Lookup(key string) string

// Failing to check a for an in-band error value can lead to bugs:
Parse(Lookup(key))  // returns "parse failure for value" instead of "no value for key"
```
Go语言对多个返回值的支持提供了更好的解决方案。 相比要求客户端特判返回值情况，函数应该在最后返回一个附加值以指示其他返回值是否有效。此返回值可能是一个error，或者在不需要解释时可以是bool。
``` go
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```
这还可以防止调用者错误地使用结果:
``` go
Parse(Lookup(key))  // compile-time error
```
鼓励编写更强大和可读性更强的代码:

``` go
value, ok := Lookup(key)
if !ok {
	return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```
此规则适用于导出的功能，但对于未导出的功能也很有用。

当返回值像nil，“”，0和-1都是函数的有效结果时，即调用方不需要与其他值进行不同的处理时，它们是好的。

某些标准库函数（如程序包“字符串”中的函数）返回带内错误值。 这大大简化了字符串处理代码，但需要程序员加倍努力。 通常，Go代码应返回错误的其他值。

## Indent Error Flow

Try to keep the normal code path at a minimal indentation, and indent the error handling, dealing with it first. This improves the readability of the code by permitting visually scanning the normal path quickly. For instance, don't write:

```go
if err != nil {
	// error handling
} else {
	// normal code
}
```

Instead, write:

```go
if err != nil {
	// error handling
	return // or continue, etc.
}
// normal code
```

If the `if` statement has an initialization statement, such as:

```go
if x, err := f(); err != nil {
	// error handling
	return
} else {
	// use x
}
```

then this may require moving the short variable declaration to its own line:

```go
x, err := f()
if err != nil {
	// error handling
	return
}
// use x
```

## 缩进错误流

尝试使普通代码路径保持最小缩进，并缩进错误处理，并首先对其进行处理。 通过允许在视觉上快速扫描正常路径，可以提高代码的可读性。 例如，不要写：
```go
if err != nil {
	// error handling
} else {
	// normal code
}
```

而应该写成：

```go
if err != nil {
	// error handling
	return // or continue, etc.
}
// normal code
```

如果if语句有初始化语句，比如：

```go
if x, err := f(); err != nil {
	// error handling
	return
} else {
	// use x
}
```

那么这可能需要将变量的声明另提一行：

```go
x, err := f()
if err != nil {
	// error handling
	return
}
// use x
```

## Initialisms

Words in names that are initialisms or acronyms (e.g. "URL" or "NATO") have a consistent case. For example, "URL" should appear as "URL" or "url" (as in "urlPony", or "URLPony"), never as "Url". As an example: ServeHTTP not ServeHttp. For identifiers with multiple initialized "words", use for example "xmlHTTPRequest" or "XMLHTTPRequest".

This rule also applies to "ID" when it is short for "identifier" (which is pretty much all cases when it's not the "id" as in "ego", "superego"), so write "appID" instead of "appId".

Code generated by the protocol buffer compiler is exempt from this rule. Human-written code is held to a higher standard than machine-written code.

## 首字母缩略词

名称中的缩写词或首字母缩写词（例如“ URL”或“ NATO”）具有一致的大小写。 例如，“ URL”应显示为“ URL”或“ url”（如在“ urlPony”或“ URLPony”中），而不应显示为“ Url”。 例如：ServeHTTP而不是ServeHttp。 对于具有多个已初始化“单词”的标识符，请使用“ xmlHTTPRequest”或“ XMLHTTPRequest”。

当“ ID”是“ identifier”的缩写时，该规则也适用于“ ID”（这在几乎所有情况下都不是“ ego”，“ superego”中的“ id”），因此请写上“ appID”而不是“ appId” ”。

协议缓冲区编译器生成的代码不受此规则限制。 人工编写的代码比机器编写的代码具有更高的标准。

## Interfaces

Go interfaces generally belong in the package that uses values of the
interface type, not the package that implements those values. The
implementing package should return concrete (usually pointer or struct)
types: that way, new methods can be added to implementations without
requiring extensive refactoring.

Do not define interfaces on the implementor side of an API "for mocking";
instead, design the API so that it can be tested using the public API of
the real implementation.

Do not define interfaces before they are used: without a realistic example
of usage, it is too difficult to see whether an interface is even necessary,
let alone what methods it ought to contain.

``` go
package consumer  // consumer.go

type Thinger interface { Thing() bool }

func Foo(t Thinger) string { … }
```

``` go
package consumer // consumer_test.go

type fakeThinger struct{ … }
func (t fakeThinger) Thing() bool { … }
…
if Foo(fakeThinger{…}) == "x" { … }
```

``` go
// DO NOT DO IT!!!
package producer

type Thinger interface { Thing() bool }

type defaultThinger struct{ … }
func (t defaultThinger) Thing() bool { … }

func NewThinger() Thinger { return defaultThinger{ … } }
```

Instead return a concrete type and let the consumer mock the producer implementation.
``` go
package producer

type Thinger struct{ … }
func (t Thinger) Thing() bool { … }

func NewThinger() Thinger { return Thinger{ … } }
```

## 接口

Go接口通常属于使用接口类型的值的包中，而不是实现这些值的包中。 实现包应返回具体的（通常是指针或结构）类型：这样，可以将新方法添加到实现中，而无需进行大量重构。

不要在“用于模拟”的API的实现方定义接口； 而是设计API，以便可以使用实际实现的公共API对其进行测试。

在使用接口之前，请不要定义接口：如果没有实际的用法示例，很难知道接口是否是必需的，更不用说接口应该包含什么方法了。

``` go
package consumer  // consumer.go

type Thinger interface { Thing() bool }

func Foo(t Thinger) string { … }
```

``` go
package consumer // consumer_test.go

type fakeThinger struct{ … }
func (t fakeThinger) Thing() bool { … }
…
if Foo(fakeThinger{…}) == "x" { … }
```

``` go
// DO NOT DO IT!!!
package producer

type Thinger interface { Thing() bool }

type defaultThinger struct{ … }
func (t defaultThinger) Thing() bool { … }

func NewThinger() Thinger { return defaultThinger{ … } }
```

而是返回一个具体的类型，并让消费者模拟生产者的实现。
``` go
package producer

type Thinger struct{ … }
func (t Thinger) Thing() bool { … }

func NewThinger() Thinger { return Thinger{ … } }
```


## Line Length

There is no rigid line length limit in Go code, but avoid uncomfortably long lines.
Similarly, don't add line breaks to keep lines short when they are more readable long--for example,
if they are repetitive.

Most of the time when people wrap lines "unnaturally" (in the middle of function calls or
function declarations, more or less, say, though some exceptions are around), the wrapping would be
unnecessary if they had a reasonable number of parameters and reasonably short variable names.
Long lines seem to go with long names, and getting rid of the long names helps a lot.

In other words, break lines because of the semantics of what you're writing (as a general rule)
and not because of the length of the line. If you find that this produces lines that are too long,
then change the names or the semantics and you'll probably get a good result.

This is, actually, exactly the same advice about how long a function should be. There's no rule
"never have a function more than N lines long", but there is definitely such a thing as too long
of a function, and of too stuttery tiny functions, and the solution is to change where the function
boundaries are, not to start counting lines.

## 单行代码长度

Go代码中没有严格的行长限制，但要避免长行。 同样，当可读性强时（例如重复），请不要添加换行符以使行短。

在大多数情况下，人们“不自然地”换行（在函数调用或函数声明的中间，或多或少，例如，尽管周围有一些例外情况），如果参数数量合理且合理，则不需要换行 简短的变量名。 长行似乎伴随长名，而摆脱长名会很有帮助。

换句话说，换行是因为您所写内容的语义（作为一般规则），而不是因为行长。 如果发现行太长，则更改名称或语义，可能会得到不错的结果。

实际上，对于功能应该持续多长时间，这是完全相同的建议。 没有规则“一个函数的长度不能超过N行”，但是肯定存在这样一个问题，即函数太长，微小的函数过于紧凑，解决方案是更改函数边界的位置，而不是改变函数边界。 开始数行。

## Mixed Caps

See https://golang.org/doc/effective_go.html#mixed-caps. This applies even when it breaks conventions in other languages. For example an unexported constant is `maxLength` not `MaxLength` or `MAX_LENGTH`.

Also see [Initialisms](https://github.com/golang/go/wiki/CodeReviewComments#initialisms).

## 组合名

参见 https://github.com/bingohuang/effective-go-zh-en/blob/master/04_Names.md#mixedcaps 。Go语言决定使用MixedCaps或mixedCaps来命名由多个单词组合的名称，而不是使用下划线来连接多个单词。即使它打破了其他语言的惯例。

例如，未导出的常量名为maxLength 而不是MaxLength 或 MAX_LENGTH。

参见 [首字母缩略词](#initialisms)。

## Named Result Parameters

Consider what it will look like in godoc.  Named result parameters like:

```go
func (n *Node) Parent1() (node *Node) {}
func (n *Node) Parent2() (node *Node, err error) {}
```

will stutter in godoc; better to use:

```go
func (n *Node) Parent1() *Node {}
func (n *Node) Parent2() (*Node, error) {}
```

On the other hand, if a function returns two or three parameters of the same type, 
or if the meaning of a result isn't clear from context, adding names may be useful
in some contexts. Don't name result parameters just to avoid declaring a var inside
the function; that trades off a minor implementation brevity at the cost of
unnecessary API verbosity.


```go
func (f *Foo) Location() (float64, float64, error)
```

is less clear than:

```go
// Location returns f's latitude and longitude.
// Negative values mean south and west, respectively.
func (f *Foo) Location() (lat, long float64, err error)
```

Naked returns are okay if the function is a handful of lines. Once it's a medium
sized function, be explicit with your return values. Corollary: it's not worth it
to name result parameters just because it enables you to use naked returns.
Clarity of docs is always more important than saving a line or two in your function.

Finally, in some cases you need to name a result parameter in order to change
it in a deferred closure. That is always OK.

## 命名返回参数

考虑下结果参数命名为下面这种情况时，godoc下的文档会是什么样子：
```go
func (n *Node) Parent1() (node *Node) {}
func (n *Node) Parent2() (node *Node, err error) {}
```

文档会念起来更拗口，推荐下面这种：

```go
func (n *Node) Parent1() *Node {}
func (n *Node) Parent2() (*Node, error) {}
```

另一方面，如果一个函数返回两个或三个相同类型的参数，或者如果从上下文中不清楚结果的含义，则在某些情况下添加名称可能很有用。 不要仅仅为了避免在函数内部声明var而命名结果参数； 牺牲了一些简短的实现方式，但付出了不必要的API冗长性。


```go
func (f *Foo) Location() (float64, float64, error)
```

更优的写法：

```go
// Location returns f's latitude and longitude.
// Negative values mean south and west, respectively.
func (f *Foo) Location() (lat, long float64, err error)
```

## Naked Returns

See [Named Result Parameters](#named-result-parameters).

## Naked Returns

参见 [命名返回参数](#named-result-parameters).

## Package Comments

Package comments, like all comments to be presented by godoc, must appear adjacent to the package clause, with no blank line.

```go
// Package math provides basic constants and mathematical functions.
package math
```

```go
/*
Package template implements data-driven templates for generating textual
output such as HTML.
....
*/
package template
```

For "package main" comments, other styles of comment are fine after the binary name (and it may be capitalized if it comes first), For example, for a `package main` in the directory `seedgen` you could write:

``` go
// Binary seedgen ...
package main
```
or
```go
// Command seedgen ...
package main
```
or
```go
// Program seedgen ...
package main
```
or
```go
// The seedgen command ...
package main
```
or
```go
// The seedgen program ...
package main
```
or
```go
// Seedgen ..
package main
```

These are examples, and sensible variants of these are acceptable.

Note that starting the sentence with a lower-case word is not among the
acceptable options for package comments, as these are publicly-visible and
should be written in proper English, including capitalizing the first word
of the sentence. When the binary name is the first word, capitalizing it is
required even though it does not strictly match the spelling of the
command-line invocation.

See https://golang.org/doc/effective_go.html#commentary for more information about commentary conventions.

## 包注释

与godoc提供的所有注释一样，包注释必须出现在Package子句的旁边，不能出现空行。

```go
// Package math provides basic constants and mathematical functions.
package math
```

```go
/*
Package template implements data-driven templates for generating textual
output such as HTML.
....
*/
package template
```

对于main包的注释，很多种注释格式都是可以接受的，比如在目录seedgen中的 package main包，注释可以这下写:

``` go
// Binary seedgen ...
package main
```
或
```go
// Command seedgen ...
package main
```
或
```go
// Program seedgen ...
package main
```
或
```go
// The seedgen command ...
package main
```
或
```go
// The seedgen program ...
package main
```
或
```go
// Seedgen ..
package main
```
这些是示例，并且这些的合理变体是可以接受的。

请注意，以小写单词开头的句子不属于打包注释的可接受选项，因为它们是公众可见的，应使用适当的英语书写，包括将句子的第一个单词大写。 当二进制名称是第一个单词时，即使它与命令行调用的拼写不严格匹配，也必须将其大写。

有关注释约定的更多信息请参见 https://github.com/bingohuang/effective-go-zh-en/blob/master/03_Commentary.md 。

## Package Names

All references to names in your package will be done using the package name, 
so you can omit that name from the identifiers. For example, if you are in package chubby, 
you don't need type ChubbyFile, which clients will write as `chubby.ChubbyFile`.
Instead, name the type `File`, which clients will write as `chubby.File`.
Avoid meaningless package names like util, common, misc, api, types, and interfaces. See http://golang.org/doc/effective_go.html#package-names and
http://blog.golang.org/package-names for more.

## 包名

包名应当简洁、清晰、意义深刻。按照惯例，包名应该由小写字母组成，不要使用下划线或混合大小写。

包中名称的所有引用都将会使用到包名，所以你可以省略包中名称的标识符。 例如，chubby.ChubbyFile，可以精简为chubby.File。

请注意避免使用无意义的包名称，如 util,common,misc,api,types和interfaces。

参见 http://blog.golang.org/package-names 和 https://github.com/bingohuang/effective-go-zh-en/blob/master/04_Names.md#package-names

## Pass Values

Don't pass pointers as function arguments just to save a few bytes.  If a function refers to its argument `x` only as `*x` throughout, then the argument shouldn't be a pointer.  Common instances of this include passing a pointer to a string (`*string`) or a pointer to an interface value (`*io.Reader`).  In both cases the value itself is a fixed size and can be passed directly.  This advice does not apply to large structs, or even small structs that might grow.

## 传值

不要仅仅为了节省几个字节而传指针给函数。如果函数仅仅将其参数"x"作为*x使用，那么参数就不应该是指针。

常见的传指针的情况有：传递一个string的指针、指向接口值(*io.Reader)的指针;这两种情况下，值本身都是固定的，可以直接传递。

对于大型数据结构，或者是小型的可能增长的结构，请考虑传指针。

## Receiver Names

The name of a method's receiver should be a reflection of its identity; often a one or two letter abbreviation of its type suffices (such as "c" or "cl" for "Client"). Don't use generic names such as "me", "this" or "self", identifiers typical of object-oriented languages that gives the method a special meaning. In Go, the receiver of a method is just another parameter and therefore, should be named accordingly. The name need not be as descriptive as that of a method argument, as its role is obvious and serves no documentary purpose. It can be very short as it will appear on almost every line of every method of the type; familiarity admits brevity. Be consistent, too: if you call the receiver "c" in one method, don't call it "cl" in another.

## 接收者名称

方法的接收者的名称应反映其身份。 通常，其类型的一个或两个字母缩写就足够了（例如，“ Client”使用“ c”或“ cl”）。 请勿使用诸如“我”，“此”或“自我”之类的通用名称，这些名称是使方法具有特殊含义的面向对象语言的典型标识符。 在Go中，方法的接收者只是另一个参数，因此应相应地命名。 该名称不必像方法参数那样具有描述性，因为它的作用是显而易见的，没有任何文档目的。 它可能很短，因为它将出现在该类型的每种方法的几乎每一行上； 熟悉承认简洁。 也要保持一致：如果在一种方法中将接收器称为“ c”，则在另一种方法中请勿将其称为“ cl”。

## Receiver Type

Choosing whether to use a value or pointer receiver on methods can be difficult, especially to new Go programmers.  If in doubt, use a pointer, but there are times when a value receiver makes sense, usually for reasons of efficiency, such as for small unchanging structs or values of basic type. Some useful guidelines:

  * If the receiver is a map, func or chan, don't use a pointer to them. If the receiver is a slice and the method doesn't reslice or reallocate the slice, don't use a pointer to it.
  * If the method needs to mutate the receiver, the receiver must be a pointer.
  * If the receiver is a struct that contains a sync.Mutex or similar synchronizing field, the receiver must be a pointer to avoid copying.
  * If the receiver is a large struct or array, a pointer receiver is more efficient.  How large is large?  Assume it's equivalent to passing all its elements as arguments to the method.  If that feels too large, it's also too large for the receiver.
  * Can function or methods, either concurrently or when called from this method, be mutating the receiver? A value type creates a copy of the receiver when the method is invoked, so outside updates will not be applied to this receiver. If changes must be visible in the original receiver, the receiver must be a pointer.
  * If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention more clear to the reader.
  * If the receiver is a small array or struct that is naturally a value type (for instance, something like the time.Time type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, a value receiver makes sense.  A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap. (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first.
  * Finally, when in doubt, use a pointer receiver.

## 接收者类型

选择是在方法上使用值接收器还是指针接收器可能很困难，尤其是对于新的Go程序员而言。 如有疑问，请使用指针，但是有时出于效率的原因（例如，小的不变结构或基本类型的值），值接收器才有意义。 一些有用的准则：

- 请不要使用指针，如果满足receiver为map，func或者chan，或receiver为切片并且该方法不会重新分配切片
- 请不要使用指针，如果receiver是较小数组或结构体，没有可变的字段和指针，或者是一个简单的基本类型，int或者string
- 请使用指针，如果方法需要改变receiver
- 请使用指针，以避免被拷贝，如果receiver包含了sync.Mutex 或类似同步字段的结构
- 请使用指针，以提升效率，如果receiver是较大的数组或结构体
- 请使用指针，如果需要在方法中改变receiver的值
- 请使用指针，如果receiver是结构体，数组或切片这种成员是一个指向可变内容的指针
- 请使用指针，如果不清楚该如何选择


## Synchronous Functions

Prefer synchronous functions - functions which return their results directly or finish any callbacks or channel ops before returning - over asynchronous ones.

Synchronous functions keep goroutines localized within a call, making it easier to reason about their lifetimes and avoid leaks and data races. They're also easier to test: the caller can pass an input and check the output without the need for polling or synchronization.

If callers need more concurrency, they can add it easily by calling the function from a separate goroutine. But it is quite difficult - sometimes impossible - to remove unnecessary concurrency at the caller side.

## 同步函数

首选同步函数-在异步函数上直接返回结果或在返回之前完成所有回调或通道操作的函数。

同步功能使goroutine在调用中保持本地状态，从而更容易推断其寿命，并避免泄漏和数据争用。 它们也更容易测试：调用方可以传递输入并检查输出，而无需轮询或同步。

如果调用者需要更多的并发性，则可以通过从单独的goroutine中调用该函数来轻松地添加它。 但是，在调用者端消除不必要的并发是非常困难的-有时是不可能的。

## Useful Test Failures

Tests should fail with helpful messages saying what was wrong, with what inputs, what was actually got, and what was expected.  It may be tempting to write a bunch of assertFoo helpers, but be sure your helpers produce useful error messages.  Assume that the person debugging your failing test is not you, and is not your team.  A typical Go test fails like:

```go
if got != tt.want {
	t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want) // or Fatalf, if test can't test anything more past this point
}
```

Note that the order here is actual != expected, and the message uses that order too. Some test frameworks encourage writing these backwards: 0 != x, "expected 0, got x", and so on. Go does not.

If that seems like a lot of typing, you may want to write a [[table-driven test|TableDrivenTests]].

Another common technique to disambiguate failing tests when using a test helper with different input is to wrap each caller with a different TestFoo function, so the test fails with that name:

```go
func TestSingleValue(t *testing.T) { testHelper(t, []int{80}) }
func TestNoValues(t *testing.T)    { testHelper(t, []int{}) }
```

In any case, the onus is on you to fail with a helpful message to whoever's debugging your code in the future.

## 有用的测试失败信息

测试应该失败，并显示有用的消息，指出错误所在，输入内容，实际得到的内容以及预期的内容。 编写一堆assertFoo帮助程序可能很诱人，但是请确保您的帮助程序会产生有用的错误消息。 假设调试失败的测试的人不是您，也不是您的团队。 典型的Go测试失败，例如：
```go
if got != tt.want {
	t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want) // or Fatalf, if test can't test anything more past this point
}
```
请注意，这里的顺序是实际的！=预期的，并且消息也使用该顺序。 一些测试框架鼓励向后编写这些代码：0！= x，“预期为0，得到x”，依此类推。 去不行。

如果这看起来很麻烦，那么您可能需要编写一个[表驱动的测试](https://github.com/golang/go/wiki/TableDrivenTests)。

在使用具有不同输入的测试助手时，消除失败测试歧义的另一种常用技术是用不同的TestFoo函数包装每个调用方，因此测试将以该名称失败：

```go
func TestSingleValue(t *testing.T) { testHelper(t, []int{80}) }
func TestNoValues(t *testing.T)    { testHelper(t, []int{}) }
```
在任何情况下，您都有责任失败，并会向将来调试您代码的人提供有用的信息。

## Variable Names

Variable names in Go should be short rather than long.  This is especially true for local variables with limited scope.  Prefer `c` to `lineCount`.  Prefer `i` to `sliceIndex`.

The basic rule: the further from its declaration that a name is used, the more descriptive the name must be. For a method receiver, one or two letters is sufficient. Common variables such as loop indices and readers can be a single letter (`i`, `r`). More unusual things and global variables need more descriptive names.

## 变量名

Go中的变量名应该短而不是长。 对于范围有限的局部变量尤其如此。 lineCount 用`c`更好。 sliceIndex 用`i`更好。

基本规则：名称声明越远，使用的名称就越具有描述性。 对于方法接收者，一个或两个字母就足够了。 诸如循环索引和读取器之类的常见变量可以是单个字母（i，r）。 更多不寻常的事物和全局变量需要更多描述性名称。
