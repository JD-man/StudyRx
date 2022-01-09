# flatMap, flatMapLatest

- [참고블로그](https://rhammer.tistory.com/300)

## 1. flatMap

### 사용
- Observable을 다른 Observable로 변환한다.
- flatMap은 element로 들어온 모든 Observable을 관찰한다!!

```swift
// Subject를 가지고 있는 구조체
struct Student {
    var scoreSubject: BehaviorSubject<Int>
}

let disposeBag = DisposeBag()

let ryan = Student(scoreSubject: BehaviorSubject(value: 80))
let charlotte = Student(scoreSubject: BehaviorSubject(value: 90))

// Student를 가지는 Subject
let studentSubject = PublishSubject<Student>()

// flatMap을 사용해 Student의 
studentSubject
    .flatMap {
        $0.scoreSubject
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)


student.onNext(ryan)
ryan.scoreSubject.onNext(50)

student.onNext(charlotte)
charlotte.scoreSubject.onNext(60)

ryan.scoreSubject.onNext(100)
charlotte.scoreSubject.onNext(110)

// Student 구조체 scoreSubject를 onNext만 보내도 값이 출력된다.
```

- studentSubject로 들어온 Student 구조체의 scoreSubject를 subscribe하는거라고 생각하면 편하다.
- studentSubject로 들어온 이전의 element들 모두를 관찰하고 있다!!

### Map과의 차이?
```swift
studentSubject
    .map {
        try! $0.scoreSubject.value()
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)


studentSubject.onNext(ryan)
ryan.scoreSubject.onNext(50)
studentSubject.onNext(ryan)

studentSubject.onNext(charlotte)
charlotte.scoreSubject.onNext(60)
studentSubject.onNext(charlotte)
```

- Map은 item 자체를 반환하기 때문에 scoreSubject를 subscribe하고 있는 효과는 없다.
- 만약 Student의 성적을 변경했다면 다시 onNext로 보내줘야한다.


---

## 2. flatMapLatest

```swift
studentSubject
    .flatMapLatest {
        $0.scoreSubject
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

studentSubject.onNext(ryan)
ryan.scoreSubject.onNext(50)

studentSubject.onNext(charlotte)
charlotte.scoreSubject.onNext(60)

// charlotte이 studentSubject로 넘어온 마지막 element이므로 ryan은 무시됨.
ryan.scoreSubject.onNext(100) // 출력 X
charlotte.scoreSubject.onNext(110) // 110 출력
```

- flatMapLatest는 마지막으로 들어온 element에만 flatMap이 적용된다.