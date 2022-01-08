# zip, merge

## Zip
- 여러개의 Observable이 뱉은 아이템을 튜플로 합쳐준다.
- combineLatest와 다른점은 모든 Observable이 아이템을 뱉어야 실행된다는 점이다.
- 그래서 Observable들 중에서 최소아이템 개수만큼 결과가 나온다.

```swift
var disposeBag = DisposeBag()

let first = Observable.from([1,2,3,4,5])
let second = Observable.from([6,7,8,9,10,11])

Observable
    .zip(first, second)
    .subscribe(onNext: {
        print("Zip result = tuple \($0)")
        print("")
        print("$0.0 = \($0.0), $0.1 = \($0.1)")
        print("================================")
        print("")
    }).disposed(by: disposeBag)
/*
Zip result = tuple (1, 6)

$0.0 = 1, $0.1 = 6
================================

Zip result = tuple (2, 7)

$0.0 = 2, $0.1 = 7
================================

Zip result = tuple (3, 8)

$0.0 = 3, $0.1 = 8
================================

Zip result = tuple (4, 9)

$0.0 = 4, $0.1 = 9
================================

Zip result = tuple (5, 10)

$0.0 = 5, $0.1 = 10
================================
*/
```

---

## Merge
- 여러개의 Observable을 하나로 합친 효과가 생긴다.
- 각각의 Observable이 방출한 아이템이 순서대로 나온다.

```swift
// first는 1초마다 방출
let first = Observable<Int>
    .interval(.seconds(1), scheduler: MainScheduler.instance)
    .map { _ in return "first" }

// second는 3초마다 방출
let second = Observable<Int>
    .interval(.seconds(3), scheduler: MainScheduler.instance)
    .map { _ in return "second" }

Observable
    .merge(first, second)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 3초마다 first와 second가 같이 방출된다.
/*
first
first
first
second
first
first
first
second
first
first
first
second
first
first
first
second
*/
```

---