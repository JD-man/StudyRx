# Hot & Cold Observable

- Observable이 언제부터 item을 내보내는지에 대한 문제

## 1. Hot
- 생성하자마자 item을 내보낼 수 있는 Observable
- 따라서 나중에 subscribe한 Observer는 Sequence의 중간부터 시작한다.

---

## 2. Cold
- 한 Observer가 subscribe하기 전까지 기다린다.
- Observer는 처음부터 모든 Sequence를 볼 수 있다.

---

## 3. 차이점
- 주로 item을 내보내는데 Observer의 subscribe가 필요한지에 대한 내용이다.

### 1) Cold의 경우
```swift
var disposeBag = DisposeBag()

func myObservable(_ arr: [Int]) -> Observable<[Int]> {
    return Observable.create { observer in
        observer.on(.next(arr))
        observer.onCompleted()
        return Disposables.create()
    }
}

var coldObservable = myObservable([0,1,2])
```

- 위와같은 코드가 있을때 실행하면 아무일도 일어나지 않는다.
- 하지만 subscribe하고 실행하면 [0,1,2]를 사용한다.

```swift
coldObservable
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// [0,1,2] 출력됨
```
- cold는 Observer의 subscription이 있어야 리소스를 사용할 수 있다.

### 2) Hot의 경우
- subject, relay는 Hot Observable이므로 이걸 사용해서 확인해본다.
```swift
var hotRelay = ReplayRelay<Int>.create(bufferSize: 3)

hotRelay.accept(0)
hotRelay.accept(1)
hotRelay.accept(2)
```
- subscribe한 Observer가 없어도 지혼자 accept해서 값을 내보낼 수 있다.
- 여기서 subscribe한다면 ReplayRelay이므로 0,1,2를 사용한다.

```swift
var hotRelay = ReplayRelay<Int>.create(bufferSize: 3)

hotRelay.accept(0)
hotRelay.accept(1)
hotRelay.accept(2)

hotRelay
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
    
//0
//1
//2
```

- Hot은 Observer의 subscription이 있는지와 무관하게 item을 내보낸다.

### 3) share의 차이
- 문서의 설명으로는 다음과 같이 나와있다
- Hot : Sequence computation resources are usually shared between all of the subscribed observers.
- Cold : Sequence computation resources are usually allocated per subscribed observer.

### 4) share 차이의 의의
- Cold의 경우 Observer당 자원이 할당된다고 하는것을 보면 Observer당 하나의 스트림이 생겨나는 것으로 이해할 수 있다.
- Hot의 경우 자원을 공유한다고 하는것을 보면 Observer들은 하나의 스트림으로 연결되어 있다고 생각할 수 있다.
- 따라서 스트림 분기가 필요한 경우 subject같은 Hot Observable을 사용해 subscription을 공유하는 것이 좋아보인다.
- Hot과 Cold의 차이점 중 가장 중요한 부분이 아닐까 생각된다..

