# throttle, debounce

## throttle
- 정해진 시간 이후에 첫번째 아이템만 내보내진다.
- 버튼 터치를 여러번하는 것을 막는데 도움이 된다.
- Request를 터치때마다하는 낭비를 막을 수 있다.

```swift
var touchNumber = 0
resetButton.rx.tap
    .asDriver()
    .map { touchNumber += 1 }
    .throttle(.seconds(2))
    .drive { _ in
        print("\(touchNumber)")                
    }.disposed(by: disposeBag)

// 터치할때마다 touchNumber가 올라가는데
// 출력은 2초간격으로 됨 (클릭은 계속했음)

/*
1
13
24
35
45
*/
```

---

## debounce
- 가장 마지막 아이템을 정해진 시간 이후에 내보낸다.
- 변하는 값에 대응할때 모든 변경마다 대비할 필요가 없을때 사용하는게 좋아보인다.

```swift
searchBar.rx.text
    .orEmpty
    .debounce(.milliseconds(200), scheduler: MainScheduler.instance)
    .distinctUntilChanged()
    .subscribe { 
        print($0)
    }.disposed(by: disposeBag)

/*
next(ㅃ)
next(빠른 검색ㅇ)
next(빠른 검색어 변경에 모두 댕ㅇ)
next(빠른 검색어 변경에 모두 대)
next(빠른 검색어 변경에 모두 대대)
next(빠른 검색어 변경에 모두 대)
next(빠른 검색어 변경에 모두 대응할 필요가 없을때)
*/
```