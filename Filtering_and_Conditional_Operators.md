
[上一页](/Transforming_Operators.md) - [返回目录](/README.md)

# 第五章 过滤
选择性的发送原`Observable`队列的元素。
## `filter`
只发送原`Observable`队列中符合条件的元素。 [更多信息](http://reactivex.io/documentation/operators/filter.html)
![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/filter.png)
```swift
example("filter") {
    let disposeBag = DisposeBag()
    
    Observable.of(
        "🐱", "🐰", "🐶",
        "🐸", "🐱", "🐰",
        "🐹", "🐸", "🐱")
        .filter {
            $0 == "🐱"
        }
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----


## `distinctUntilChanged`
 阻止同一`Observable`队列多次发送相同元素。 [更多信息](http://reactivex.io/documentation/operators/distinct.html)
![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/distinct.png)
```swift
example("distinctUntilChanged") {
    let disposeBag = DisposeBag()
    
    Observable.of("🐱", "🐷", "🐱", "🐱", "🐱", "🐵", "🐱")
        .distinctUntilChanged()
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----

## `elementAt`
 只发`Observable`队列送指定位置上的元素。[更多信息](http://reactivex.io/documentation/operators/elementat.html)
 ![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/elementat.png)
```swift
example("elementAt") {
    let disposeBag = DisposeBag()
    
    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .elementAt(3)
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----

## `single`
 发送`Observabel`队列中的第一个满足条件的元素，如果没满足条件的元素这会发送一个错误(`error`)。
 ```swift
example("single") {
    let disposeBag = DisposeBag()
    
    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .single()
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}

example("single with conditions") {
    let disposeBag = DisposeBag()
    
    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .single { $0 == "🐸" }
        .subscribe { print($0) }
        .addDisposableTo(disposeBag)
    
    Observable.of("🐱", "🐰", "🐶", "🐱", "🐰", "🐶")
        .single { $0 == "🐰" }
        .subscribe { print($0) }
        .addDisposableTo(disposeBag)
    
    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .single { $0 == "🔵" }
        .subscribe { print($0) }
        .addDisposableTo(disposeBag)
}
```
 
----

## `take`
 发送`Observable`队列d的前n个元素。 [更多信息](http://reactivex.io/documentation/operators/take.html)
 ![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/take.png)
 ```swift
example("take") {
    let disposeBag = DisposeBag()
    
    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .take(3)
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----

## `takeLast`
 takeLast发送`Observable`队列d的最后n个元素。 [更多信息](http://reactivex.io/documentation/operators/takelast.html)
 ![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/takelast.png)
 ```swift
example("takeLast") {
    let disposeBag = DisposeBag()
    
    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .takeLast(3)
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----

## `takeWhile`
 发送满足条件的元素。[更多信息](http://reactivex.io/documentation/operators/takewhile.html)
 ![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/takewhile.png)
 ```swift
example("takeWhile") {
    let disposeBag = DisposeBag()
    
    Observable.of(1, 2, 3, 4, 5, 6)
        .takeWhile { $0 < 4 }
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----

## `takeUntil`
 选择一个参考队列在该队列发送元素前发送本队列的元素。 [更多信息](http://reactivex.io/documentation/operators/takeuntil.html)
 ![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/takeuntil.png)
 ```swift
example("takeUntil") {
    let disposeBag = DisposeBag()
    
    let sourceSequence = PublishSubject<String>()
    let referenceSequence = PublishSubject<String>()
    
    sourceSequence
        .takeUntil(referenceSequence)
        .subscribe { print($0) }
        .addDisposableTo(disposeBag)
    
    sourceSequence.onNext("🐱")
    sourceSequence.onNext("🐰")
    sourceSequence.onNext("🐶")
    
    referenceSequence.onNext("🔴")
    
    sourceSequence.onNext("🐸")
    sourceSequence.onNext("🐷")
    sourceSequence.onNext("🐵")
}
```
 
----

## `skip`
 跳过前n个元素，发送之后的元素. [更多信息](http://reactivex.io/documentation/operators/skip.html)
 ![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/skip.png)
 ```swift
example("skip") {
    let disposeBag = DisposeBag()
    
    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .skip(2)
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----

## `skipWhile`
 跳过满足条件的元素，发送之后的元素. [更多信息](http://reactivex.io/documentation/operators/skipwhile.html)
 ![](http://reactivex.io/documentation/operators/images/skipWhile.c.png)
 ```swift
example("skipWhile") {
    let disposeBag = DisposeBag()
    
    Observable.of(1, 2, 3, 4, 5, 6)
        .skipWhile { $0 < 4 }
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----

## `skipWhileWithIndex`
skipWhileWithIndex跳过条件成立之前的元素，发送满足条件之后的元素，闭包发送每个元素的`index`
 ```swift
example("skipWhileWithIndex") {
    let disposeBag = DisposeBag()
    
    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .skipWhileWithIndex { element, index in
            index < 3
        }
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
}
```
 
----

## `skipUntil`
跳过参考队列发送元素前本队列发送的元素。[更多信息](http://reactivex.io/documentation/operators/skipuntil.html)
 ![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/skipuntil.png)
 ```swift
example("skipUntil") {
    let disposeBag = DisposeBag()
    
    let sourceSequence = PublishSubject<String>()
    let referenceSequence = PublishSubject<String>()
    
    sourceSequence
        .skipUntil(referenceSequence)
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
    
    sourceSequence.onNext("🐱")
    sourceSequence.onNext("🐰")
    sourceSequence.onNext("🐶")
    
    referenceSequence.onNext("🔴")
    
    sourceSequence.onNext("🐸")
    sourceSequence.onNext("🐷")
    sourceSequence.onNext("🐵")
}
```
[下一章](/Mathematical_and_Aggregate_Operators.md) - [返回目录](/README.md)
