# decode

- JSONDecoder를 사용해서 decode 해준다. RxSwift 6.0부터 즉시지원
- 와... 할말이업네. 이런 호사를 누려도 되는걸까?????
- 데이터 받아오는걸 Generic으로 만들었었는데 이걸쓰면 그럴필요가 없어지는걸까???? 열심히 공부했는데..

```swift
// URLSession Data Task (Lotto Data)
func useURLSession() -> Observable<Data> {
    return Observable.create { value in
        let url = URL(string: self.lottoURL)
        let task = URLSession.shared.dataTask(with: url!) { data, response, error in
            guard error == nil else {
                value.onError(ExampleError.fail)
                return
            }
            if let data = data {
                print("datatask resume")
                value.onNext(data)
            }
            value.onCompleted()
        }
        task.resume()
        return Disposables.create() {
            task.cancel()
        }
    }
}

// binding
useURLSession()
    .decode(type: Lotto.self, decoder: JSONDecoder())
    .bind {
        print($0)
    }.disposed(by: disposeBag)


// 결과: Codable 구조체로 출력됨!!
/*
 Lotto(totSellamnt: 88625160000, returnValue: "success", drwNoDate: "2020-03-21", firstWinamnt: 1684582212, drwtNo6: 28, drwtNo4: 21, firstPrzwnerCo: 13, drwtNo5: 22, bnusNo: 45, firstAccumamnt: 21899568756, drwNo: 903, drwtNo2: 15, drwtNo3: 16, drwtNo1: 2)
*/
```