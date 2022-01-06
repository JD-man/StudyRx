## Input, Output 1

- 목차
1. Protocol
2. ViewModel
3. Transform

---

## 1. Protocol

- 기본적으로 뷰모델에 입력과 출력에 대한 구조체를 만든다.
- 따라서 프로토콜에는 associatedtype으로 타입을 만들어둔다.
- 입력을 출력으로 변환시키기 위한 함수와 disposeBag도 만들어둔다.

```swift
protocol CommonViewModel {
    associatedtype Input
    associatedtype Output
    
    var disposeBag: DisposeBag { get set }
    func transform(input: Input) -> Output
}
```

---

## 2. ViewModel
- 입력 : nameTextField를 통한 ControlProperty, button을 통한 ControlEvent
- 출력 : 유효성검사 결과 Bool, 유효성조건 문구 String, 뷰전환 ControlEvent

```swift
class ValidationViewModel: CommonViewModel {
    struct Input {
        let text: ControlProperty<String?>
        let tap: ControlEvent<Void>
    }
    
    struct Output {
        let validStatus: Observable<Bool>
        let validText: BehaviorRelay<String>
        let sceneTransition: ControlEvent<Void>
    }
    
    var validText = BehaviorRelay<String>(value: "최소 8자 이상 필요합니다")
    var disposeBag = DisposeBag()
    

    // 입력을 출력으로 변환시켜준다.
    // Observable로 변환시켜주므로 뷰에서 이 출력을 가지고 바인딩한다.
    // tap의 경우 별다른 변환이 필요없으므로 입력의 tap을 그대로 사용한다.
    func transform(input: Input) -> Output {
        let resultText = input.text
            .orEmpty
            .map { $0.count >= 8 }
            .share(replay: 1, scope: .whileConnected)
        
        return Output(validStatus: resultText,
                      validText: validText,
                      sceneTransition: input.tap)
    }
}
```

---

## 3. View
- 입력과 출력을 만들고 출력과 뷰를 바인딩한다.

```swift
let input = ValidationViewModel.Input(text: nameTextField.rx.text,
                                      tap: button.rx.tap)
let output = viewModel.transform(input: input)

output.validStatus
    .bind(to: button.rx.isEnabled, nameValidationLabel.rx.isHidden)
    .disposed(by: disposeBag)

output.validText
    .bind(to: nameValidationLabel.rx.text)
    .disposed(by: disposeBag)

output.sceneTransition
    .bind {
        self.present(ReactiveViewController(), animated: true, completion: nil)
    }.disposed(by: disposeBag)
```

---

### 위 예제를 통해 생각하면
- 뷰로부터 발생할 수 있는 이벤트들을 모아서 뷰모델의 비즈니스 로직을 사용해 출력으로 변환한다.
- 출력과 뷰는 바인딩돼야하니까 Observable이어야한다.

### 그렇다면?!
- RxSwift Github에 있는 문서보면 연산자를 만드는건 Observable 만드는거나 다름없다는 식의 말을했다,
- 그러면 뷰모델에 있는 비즈니스 로직은 커스텀 연산자를 만드는 것이 아닐까 하는 생각이든다.
- 이미 있는 연산자를 많이 쓰겠지만 리스폰스를 받아온다던지 이런거는 없으니까...
- 연산자를 만들일이 생각보다 많아보인다...