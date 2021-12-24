# Observable

## 1) Observable == Swift's Sequence
- RxSwift의 Observable과 Swift의 Sequence는 동등하다.
- RxSwift는 Sequence와 비교해서 elements를 **비동기로 받을 수 있다는 차이**가 있다.
- Observable.subscribe는 Sequence.makeIterator와 동등하다.
- elements를 받기위해 iterator의 next() 대신 Observer(callback)이 subscribe 메서드에 들어간다.

## 2) error or completed
- Sequence는 0개 이상의 elements를 가지고 있으며 error 또는 completed 이벤트를 받으면 더이상 element가 안나온다.
- Observable은 Observer인 callback을 가진다.
- Observable은 completed 또는 error 이벤트가 발생되면 모든 리소스를 해제한다.
- 이때 리소스를 즉시해제하기 위해서 DisposeBag이나 takeUntil을 사용해야한다.
- DisposeBag이 더 선호되는 편이라고 함.

---

# Dispose

## 1) dispose()
- dispose()를 직접 사용하는 경우는 거의 없다.
- 하지만 만약 사용하게 될 경우 같은 scheduer에서 사용하는것이 좋다.

## 2) DisposeBag
- DisposeBag이 해제가 되면 가지고 있는 모든 disposable들을 dispose한다.
- 따로 dispose 메소드가 없으므로 모든것을 즉시제거하려면 self.disposeBag = DisposeBag()으로 새로 만든다.
- takeUnitl을 사용하는 방법도 있음. 쓸일없을듯ㅋㅋㅋ

---

# Implicit Observable guarantees

- elemets를 만드는 thread와 상관없이 observer.on 메소드가 끝나기 전까지 다음 elements를 전달하지 않는다.

---

# Observable 만들기

## 1) 메소드 만들어서 return
- observable을 반환하는 메소드를 만들면 sequence generation 및 side effect가 없다.
- Observable.create를 사용해서 만들고 반환한다.

```swift
func myJust<E>(_ element: E) -> Observable<E> {
    return Observable.create { observer in
        observer.on(.next(element))
        observer.on(.completed)
        return Disposables.create()
    }
}

myJust(0)
    .subscribe(onNext: { n in
        print(n)
    }).disposed(by: disposeBag)

// 0 is printed
```

- Sequence의 경우 elements는 disposable이 반환되기 전에 생성되고 사라진다.
- 그래서 무슨 disposable이 반환되든지 상관없이 elements의 생성과정은 방해받지 않는다.
- 이 경우 disposable은 NopDisposlable의 싱글턴 인스턴스다.

```swift
func myFrom<E>(_ sequence: [E]) -> Observable<E> {
    return Observable.create { observer in
        for element in sequence {
            observer.on(.next(element))
        }
        observer.on(.completed)
        return Disposables.create()
    }
}

let stringCounter = myFrom(["first", "second"])

// first time subscribe
stringCounter
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag) // first, seconde 출력

// second time subscribe
stringCounter
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag) // 또 first, seconde 출력
```

---

# Stateless
- 0.1초마다 돌아가는 타이머 Observable을 만들고 1초 후에 dispose 시킨다.
```swift
func myInterval(_ interval: DispatchTimeInterval) -> Observable<Int> {
    return Observable.create { observer in
        print("subscribed")
        // global queue에서 돌아가는 timer
        let timer = DispatchSource.makeTimerSource(queue: DispatchQueue.global())
        // timer는 interval의 간격으로 돌아감
        timer.schedule(deadline: DispatchTime.now() + interval, repeating: interval)

        // dispose될때 timer를 cancle시키는 반환될 disposable
        let cancel = Disposables.create {
            print("Disposed")
            timer.cancel()
        }

        var next = 0
        timer.setEventHandler {
            // dispose 되기 전까지 timer가 돌아감
            if cancel.isDisposed {
                return
            }
            // next를 보내고 1 더함
            observer.on(.next(next))
            next += 1
        }
        timer.resume()
        return cancel
    }
}

// interval이 0.1인 Observable
let counter = myInterval(.milliseconds(100))

let subscription = counter
    .subscribe(onNext: {
        print($0)
    })

// 1초뒤에 cancel이 dispose 되니까 next는 10번 돈다
Thread.sleep(forTimeInterval: 1)
subscription.dispose()

// 0,1,2,3,4,5,6,7,8,9 출력됨.
```
- subscribe한 **Observer가 두개**라면?!

```swift
let subscription1 = counter
    .subscribe(onNext: {
        print("sub1: ", $0)
    })

let subscription2 = counter
    .subscribe(onNext: {
        print("sub2: ", $0)
    })

Thread.sleep(forTimeInterval: 1)
subscription1.dispose()

Thread.sleep(forTimeInterval: 1)
subscription2.dispose()

// 시작 ~ 1초까지 각자 0~9 출력, 1초~2초 부터는 subscription2만 10~19 출력
```
- subscriber는 각자 **독립된** elements sequence를 가진다!
- 문서에서는 Operator들은 기본적으로 **stateless**라고 한다.
- 각각의 Operator들은 이전의 동작과 독립적으로 동작한다.

---

# Sharing subscription
- 1개의 구독의 이벤트를 여러 Observer가 공유하는 방법
- 새로운 subscriber가 observing하기 전 과거 elements를 어떻게 다룰지(replay)와 언제 공유할지(refCount)가 필요하다.
- 이거를 replay(1).refCount(), 합쳐서 share(replay: 1)로 사용한다. Observable에 붙임.

```swift
let counter = myInterval(.milliseconds(100)).share(replay: 1)

// 위와 똑같이 sub1, sub2를 돌리면
// subscribed와 Disposed가 1번 출력된다!!
```