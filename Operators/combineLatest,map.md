# combineLatest, map

## combineLatest
- 여러 Observable을 묶어서 같이 사용한다.
- 이들중 하나의 Observable이 값을 내보내면 나머지 다른 Observable들이 마지막으로 내보낸 값도 같이 내보내진다.

```swift
Observable.combineLatest(first.rx.text.orEmpty, second.rx.text.orEmpty) { textValue1, textValue2 -> String in
        print("Input: (\(textValue1) + \(textValue2)) / 2")
        return String(((Double(textValue1) ?? 0.0) + (Double(textValue2) ?? 0.0)) / 2)
    }
    .bind {
        print($0)
    }
.disposed(by: disposeBag)

/*
Input: (3 + 2) / 2
Output: 2.5
*/
```

---

## map
- 내보내진 item에 function을 적용해서 변환시킨다.

```swift
// 위에거에서 map 사용
Observable.combineLatest(first.rx.text.orEmpty, second.rx.text.orEmpty)
    .map {
        String(((Double($0) ?? 0.0) + (Double($1) ?? 0.0)) / 2)
    }
    .bind {
        print("Output: \($0)")
    }
    .disposed(by: disposeBag)
```