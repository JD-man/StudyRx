# orEmpty, distinctUntilChanged

## orEmpty
- ControlProperty에서 사용하는 자료형의 옵셔널을 해제해준다.

```swift
// orEmpty 사용전
nameTextField.rx.text            
    .bind {
        print($0)
    }.disposed(by: disposeBag)
/*
Optional("")
Optional("")
Optional("S")
Optional("Sd")
Optional("Sda")
Optional("Sdad")
Optional("Sdada")
Optional("Sdadas")
*/

// orEmpty 사용후
nameTextField.rx.text
    .orEmpty
    .bind {
        print($0)
    }.disposed(by: disposeBag)
/*


A
As
Asd
Asda
Asdas
Asdasd
*/

```

---

## distinctUntilChanged
- 이전에 들어온값과 같은값은 무시한다. 중복방지

```swift
// Asd 입력후 Asd 복사해서 전체붙여넣기함. (Asd만 계속 입력)
nameTextField.rx.text
    .orEmpty
    .distinctUntilChanged()
    .bind {
        print($0)
    }.disposed(by: disposeBag)

/*
A
As
Asd
이후로 print 실행 안됨.
*/

```