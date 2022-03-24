# withUnretained

- Swift 6.0부터 나왔다.
- 약한 참조를 위해서 사용한다.

---

## 사용 전
```swift
searchTableView.rx.contentOffset    
    .filter { [weak self] in
        guard let self = self else { return false }
        guard self.searchTableView.contentSize.height > 0 else { return false }
        return $0.y + self.searchTableView.frame.height > self.searchTableView.contentSize.height - 100
    } // ...
```

- 꼴보기싫은 [weak self]와 self의 옵셔널 바인딩이 있다.

---

## 사용 후

```swift
searchTableView.rx.contentOffset
    .withUnretained(self)
    .filter {
        guard $0.searchTableView.contentSize.height > 0 else { return false }
        return $1.y + $0.searchTableView.frame.height > $0.searchTableView.contentSize.height - 100
    } // ...
```

- withUnretained를 사용하면 튜플이 클로저로 들어온다.
- 첫번째값은 self, 두번째값은 위의 경우 contentOffset이 들어온다.

---