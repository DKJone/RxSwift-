# RxSwift-文档翻译
对RxSwift 官方playground的翻译，playGround基于2016年-12月-1日版本

重要提示：使用Rx.playground
---
1.   打开Rx.xcworkspace.
2. 编译 RxSwift-macOS 项目 (Product → Build)
3. 在项目导航栏你打开RX playground
4. 打开调试窗口(View → Debug Area → Show Debug Area)

序章：介绍
---
为什么我们要使用RxSwift？
我们写的绝大多数代码都包含了界面元素的事件响应。当用户操作控件时，我们需要写一个@IBAction的处理句柄去响应用户事件。我们需要订阅通知去观测何时键盘位置发生改变。当URL sessions返回一个数据时我们需要提供一个可执行的闭包。我们利用KVO去观测变量的变化。这些各种各样的机制促使我们的代码产生了不必要的复杂。有什么能比只使用一种机制去处理所有请求或响应更好的呢？Rx就是这样一种机制。

RxSwift是官方的Reactiv Extension（一款同时支持多种语言平台）的实现。

第一章：概念
---
任何一个Observable的实例都是一个队列
一个Observable队列和Swift的SequenceType相比它的核心优势就在于它依然可以接收异步元素，这是RxSwift的核心所在。其他的所有都是建立在这基础之上的。
- 一个Observable (`ObservableType`)等价于一个 `SequenceType`
- `ObservableType.subscribe(_:)`方法等价于`SequenceType.generate()`
- `ObservableType.subscribe(_:)`需要一个观察者(`ObserverType`)作为参数，他将自动订阅由Observable发出的事件队列，而不是手动的用`Next()`方法订阅回调。

如果 一个`Observable`发出一个`next`事件(`Event.next(Element)`),它人可以继续发出更多的事件。但是如果它发出了一个错误事件(`Event.error(ErrorType)`)或者一个完成事件(`Event.completed`)，他讲不再能够发送更多的事件给订阅者。

这样介绍队列的语法更简洁
```
next* (error | completed)?
```
用图表可以更形象的解释
```
1-->1-->2-->3-->4-->5-->6-->|----> // "|" =正常停止
--a--b--c--d--e--f--X----> // "X" =错误时停止
--tap--tap----------tap--> // "|" =永远不停止，例如按钮的点击事件队列
```
```
graph LR
1-->2;2-->3;3-->4;4-->|-|正常停止 ;4-->|-|错误停止  
button-->tap
tap-->tap.....tap....tap;tap.....tap....tap-->|永不停止|...
```

### Observables and observers (aka subscribers)
可观测的和观测者(也叫作订阅者)
---
可订阅对象(Observables)不会执行他们的订阅闭包，除非他们有一个订阅者。例如下面这个例子，他的闭永远不会执行因为他没有一个订阅者
```swift
example("Observable with no subscribers") {
_ = Observable<String>.create { observerOfString -> Disposable in
print("This will never be printed")
observerOfString.on(.next("😬"))
observerOfString.on(.completed)
return Disposables.create()
}
}
```
在下面这个例子中，闭包会在被订阅(`subscribe(_:)`)时执行
```swift
example("Observable with subscriber") {
_ = Observable<String>.create { observerOfString in
print("Observable created")
observerOfString.on(.next("😉"))
observerOfString.on(.completed)
return Disposables.create()
}
.subscribe { event in
print(event)
}
}
```
提示：不要关心`Observables`是怎么创建的，我们将在之后介绍
提示：`subscribe(_:)`返回一个`Disposable`实例代表一次性资源比如一个订阅。他在之前的简单例子中被忽略了，但是它常常正确的处理了。这意味着将它放入内容一个` DisposeBag`实例中。在此后的例子中我们将包含适当的处理，因为实践出真知！

你可已在这里获取更多
- [Disposing section](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md#disposing)
- [Getting Started](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md)


第二章：创建和订阅可订阅者
---
有下面几种创建和订阅` Observable`队列的方式
#### 1. never:绝对不要创建一个永不结束且不发送任何事件的队列

```swift
example("never") {
let disposeBag = DisposeBag()
let neverSequence = Observable<String>.never()

let neverSequenceSubscription = neverSequence
.subscribe { _ in
print("This will never be printed")
}

neverSequenceSubscription.addDisposableTo(disposeBag)
}
```
#### 2. empty:创建一个空的`Observable`只会发送一个完成事件 
```swift
example("empty") {
let disposeBag = DisposeBag()

Observable<Int>.empty()
.subscribe { event in
print(event)
}
.addDisposableTo(disposeBag)
}
```
提示：这个例子也包括创建和订阅一个`Observable`队列
#### 3. just:创建一个只有单一信号元素的`OBservable`队列
```swift
example("just") {
let disposeBag = DisposeBag()

Observable.just("🔴")
.subscribe { event in
print(event)
}
.addDisposableTo(disposeBag)
}
```
#### 4. of:创建一个带有固定元素个数的`Observable`队列
```swift
example("of") {
let disposeBag = DisposeBag()

Observable.of("🐶", "🐱", "🐭", "🐹")
.subscribe(onNext: { element in
print(element)
})
.addDisposableTo(disposeBag)
}
```
#### 提示：
这个例子也包括了使用`subscribe(onNext:)`简便方法，和`subscribe(_:)`订阅所有时间句柄不同(next,error,completeed),`subscribes(onNext:)`订阅一个元素的除了完成和错误(Error、Completed)的其他事件而且只产生下一个事件元素，当然还有`subscribe(onCompleted:)`和`subscribe(onError:) `只订阅而对应的事件.也有一个`subscribe(onNext:onError:onCompleted:onDisposed:)`方法，可以允许你响应一个或者多个类型的事件，包括由于某种原因使这个订阅终止和正常处理。
例如
```swift
someObservable.subscribe(
onNext: { print("Element:", $0) },
onError: { print("Error:", $0) },
onCompleted: { print("Completed") },
onDisposed: { print("Disposed") }
)
```
#### 5. from:由一个`SequenceType`创建一个`Observable`队列，例如`Array, Dictionary, Set`
```swift
example("from") {
let disposeBag = DisposeBag()

Observable.from(["🐶", "🐱", "🐭", "🐹"])
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)
}
```
#### 提示：这个示例也示范了使用默认的声明`$0`去替代明确的声明

#### 6. create:创建一个自定义的`Observable`队列
```swift
example("create") {
let disposeBag = DisposeBag()

let myJust = { (element: String) -> Observable<String> in
return Observable.create { observer in
observer.on(.next(element))
observer.on(.completed)
return Disposables.create()
}
}

myJust("🔴")
.subscribe { print($0) }
.addDisposableTo(disposeBag)
}
```
#### 7. range：创建一段连续区间的整数的`Observable`队列
```swift
example("range") {
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 10)
.subscribe { print($0) }
.addDisposableTo(disposeBag)
}
```
#### 8. `repeatElement`:创建一个无线发送元素的`Observable`队列
```swift
example("repeatElement") {
let disposeBag = DisposeBag()

Observable.repeatElement("🔴")
.take(3)
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)
}
```
#### 提示：这个示例还展示了使用`take`操作符去返回一个指定数量元素的队列

#### 9.generate：创建一个只要提供的条件成立就持续生成值的队列
```swift
example("generate") {
let disposeBag = DisposeBag()

Observable.generate(
initialState: 0,
condition: { $0 < 3 },
iterate: { $0 + 1 }
)
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)
}
```
#### 10. deferred：为所有订阅者创建一个新的`Observable`队列
```swift
example("deferred") {
let disposeBag = DisposeBag()
var count = 1

let deferredSequence = Observable<String>.deferred {
print("Creating \(count)")
count += 1

return Observable.create { observer in
print("Emitting...")
observer.onNext("🐶")
observer.onNext("🐱")
observer.onNext("🐵")
return Disposables.create()
}
}

deferredSequence
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)

deferredSequence
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)
}
```
#### 11.error:创建一个不发送任何元素并且立即停止的错误`Observable`队列
```swift
example("error") {
let disposeBag = DisposeBag()

Observable<Int>.error(TestError.test)
.subscribe { print($0) }
.addDisposableTo(disposeBag)
}
```
#### 12. doOn:为所有发出和接受的事件添加一个附加的操作
```swift
example("doOn") {
let disposeBag = DisposeBag()

Observable.of("🍎", "🍐", "🍊", "🍋")
.do(onNext: { print("Intercepted:", $0) },
onError: { print("Intercepted error:", $0) }, 
onCompleted: { print("Completed") })
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)
}
```
#### 提示：当然也会有`doOnNext(_:), doOnError(_:), doOnCompleted(_:)`这些方法去拦截特定的事件，`doOn(onNext:onError:onCompleted:)`拦截一个或多个的事件信号

第三章：利用分类工作(编码)
----
一个分类是获取Rx的观测者和可观察属性(`Observable`)的桥梁和代理。因为是观察者，所以它可以订阅一个或者多个可观察对象(`Observable`)。因为是可观察对象(`Observable`)，它可以通过元素观察和重发他们，也可以发送新的元素。
```swift
extension ObservableType {

/**
为id添加观察者，并打印所有发出的事件
- parameter id: 订阅者的id.
*/
func addObserver(_ id: String) -> Disposable {
return subscribe { print("Subscription:", id, "Event:", $0) }
}

}

func writeSequenceToConsole<O: ObservableType>(name: String, sequence: O) -> Disposable {
return sequence.subscribe { event in
print("Subscription: \(name), event: \(event)")
}
}
```
#### 1. PublishSubject:在订阅后向他的观察者广播事件
```swift
example("PublishSubject") {
let disposeBag = DisposeBag()
let subject = PublishSubject<String>()

subject.addObserver("1").addDisposableTo(disposeBag)
subject.onNext("🐶")
subject.onNext("🐱")

subject.addObserver("2").addDisposableTo(disposeBag)
subject.onNext("🅰️")
subject.onNext("🅱️")
```
#### 提示：这个示例还是用了`onNext(_:)`简便方法，等价于使用`on(.next(_:))`,让用户使用订阅元素的下一个事件。也有`onError(_:) 和onCompleted()`简便方法分别等价于`on(.error(_:)) 和   on(.completed)`
#### 2. ReplaySubject：广播新事件给所有订阅者，并指定新事件的之前的缓存大小。
```swift
example("ReplaySubject") {
let disposeBag = DisposeBag()
let subject = ReplaySubject<String>.create(bufferSize: 1)

subject.addObserver("1").addDisposableTo(disposeBag)
subject.onNext("🐶")
subject.onNext("🐱")

subject.addObserver("2").addDisposableTo(disposeBag)
subject.onNext("🅰️")
subject.onNext("🅱️")
}
```
#### 3. BehaviorSubject广播新的事件给订阅者，并发送最近的(或者初始值)给行的而订阅者
```swift
example("BehaviorSubject") {
let disposeBag = DisposeBag()
let subject = BehaviorSubject(value: "🔴")

subject.addObserver("1").addDisposableTo(disposeBag)
subject.onNext("🐶")
subject.onNext("🐱")

subject.addObserver("2").addDisposableTo(disposeBag)
subject.onNext("🅰️")
subject.onNext("🅱️")

subject.addObserver("3").addDisposableTo(disposeBag)
subject.onNext("🍐")
subject.onNext("🍊")
}
```
#### 提示：注意这些之前的例子中都遗漏了什么？完成事件！`PublishSubject, ReplaySubject,BehaviorSubject`当他们即将被处理时，不能自动发出完成事件。
#### 4.Variable覆盖`BehaviorSubject`所以它将发送最近(或初始)的值给新的订阅者，并维持最近值得状态。`Variable`将不会发送错误事件，然而他会在销毁前发送完成事件和结束
```swift
example("Variable") {
let disposeBag = DisposeBag()
let variable = Variable("🔴")

variable.asObservable().addObserver("1").addDisposableTo(disposeBag)
variable.value = "🐶"
variable.value = "🐱"

variable.asObservable().addObserver("2").addDisposableTo(disposeBag)
variable.value = "🅰️"
variable.value = "🅱️"
}
```
#### 提示：一个`Variable`实例使用`asObservable`方法，访问它的原始队列，`Variables`没有实现`on`操作符(如`onNext(_:)`),但是作为替代，提供了一个`value`属性可以用作获取最近的值，也可以设置一个新的值，设置新值也会添加这个值到原始的`BehaviorSubject`队列。

第四章 组合运算符
----
操作符可以绑定多个`Observable`为一个`Observable`信号

#### 1. StarWith发送指定元素的在队列发出之前(在队列最前方插入)
```swift
example("startWith") {
let disposeBag = DisposeBag()

Observable.of("🐶", "🐱", "🐭", "🐹")
.startWith("1️⃣")
.startWith("2️⃣")
.startWith("3️⃣", "🅰️", "🅱️")
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)
}
```
提示：如例所示如，`startWidth`可以连接成一个后进先出队列，所有继承`StartWidth`的元素都被添加到之前`StartWidth`元素之前
#### 2. merge 从源头合并多个`Observable`元素为一个信号队列，并且发送原`Observable`队列的事件
```swift
example("merge") {
let disposeBag = DisposeBag()

let subject1 = PublishSubject<String>()
let subject2 = PublishSubject<String>()

Observable.of(subject1, subject2)
.merge()
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)

subject1.onNext("🅰️")

subject1.onNext("🅱️")

subject2.onNext("①")

subject2.onNext("②")

subject1.onNext("🆎")

subject2.onNext("③")
}
``` 
#### 3.zip
绑定最多达8个`Observable`队列源为一个信号源，并发送按原始队列对应序号绑定后的元素，直到每个原始队列在该序号上都有元素。
```
graph LR
1-->2;2-->3;3-->4;4-->5;5-->....  
A-->B;B-->C;C-->D;D-->...
```
zip后

```
graph LR
1A-->2B;2B-->3C;3C-->4D;4D-->...
```
```swift
example("zip") {
let disposeBag = DisposeBag()

let stringSubject = PublishSubject<String>()
let intSubject = PublishSubject<Int>()

Observable.zip(stringSubject, intSubject) { stringElement, intElement in
"\(stringElement) \(intElement)"
}
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)

stringSubject.onNext("🅰️")
stringSubject.onNext("🅱️")

intSubject.onNext(1)

intSubject.onNext(2)

stringSubject.onNext("🆎")
intSubject.onNext(3)
}
```


#### 4. combineLatest
绑定最多达8个`Observable`队列为一个新的信号队列，并绑定每个原始队列的最新的一个元素在一起为一个信号，在每个原始队列添加元素师都会发送一个新的绑定元素。

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/combinelatest.png)
```swift
example("combineLatest") {
let disposeBag = DisposeBag()

let stringSubject = PublishSubject<String>()
let intSubject = PublishSubject<Int>()

Observable.combineLatest(stringSubject, intSubject) { stringElement, intElement in
"\(stringElement) \(intElement)"
}
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)

stringSubject.onNext("🅰️")

stringSubject.onNext("🅱️")
intSubject.onNext(1)

intSubject.onNext(2)

stringSubject.onNext("🆎")
}
```
#### 提示：`combinLatest`基于数组的扩展要求原队列元素类型相同。元素按原队列序号依次添加111222333....

#### 5. switchLatest 转换`Observable`队列发送的元素，并发送内部队列里最近的值
```swift
example("switchLatest") {
let disposeBag = DisposeBag()

let subject1 = BehaviorSubject(value: "⚽️")
let subject2 = BehaviorSubject(value: "🍎")

let variable = Variable(subject1)

variable.asObservable()
.switchLatest()
.subscribe(onNext: { print($0) })
.addDisposableTo(disposeBag)

subject1.onNext("🏈")
subject1.onNext("🏀")

variable.value = subject2

subject1.onNext("⚾️")

subject2.onNext("🍐")
}

```
#### 提示：在这个例子中，在设置`variable.value=subject2`后添加⚾️到`subject1`不会产生任何影响，因为只有最近的内部`Observable`队列`subject2`才会发送元素

第五章 Transforming Operators 转换操作符
---
