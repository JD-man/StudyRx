# concat, catch

## concat
- 여러개의 observable을 합성한다.
- 합성시의 **순서가 보장**되어서 element를 방출한다.
- 이전 시퀀스의 complete 이벤트가 발생해야 계속 진행된다.

```swift
Observable.concat([
    Observable.just(Mutation.setLoadingStatus(status: true)),
    getBeers(page: currentState.nextPage)
        .map { Mutation.addBeersItem(items: $0.0, nextPage: $0.1) },
    Observable.just(Mutation.setLoadingStatus(status: false)),
    ])
```

- 이 Observable concat은 세개의 Mutation을 차례대로 방출한다.
- 주의할점은 반드시 다음 시퀀스를 위해서 이전 시퀀스가 completed 되어야한다.
- 따라서 Observable을 직접 만들었다면 next와 동시에 completed를 해줘야한다.

---

## catch
- RxSwift에서 에러 핸들링하는 Operation 중 하나.
- 받은 에러를 방출한 뒤에 **completed** 된다.
- **do(onError:)** 사용시에는 concat의 이후 시퀀스가 진행되지 않는다!
- 그래서 catch는 concat과 사용할때 에러처리에 좋다.

```swift
Observable.concat([
    Observable.just(Mutation.setLoading(status: true)),
    useCase.executeSearchRepository(q: text, page: 1)
            .asObservable()
            // catch 사용으로 에러처리
            .catch {
                print($0)
                return Observable.empty()
            }.map { Mutation.setRepoItems(model: $0, q: text) },
    Observable.just(Mutation.setLoading(status: false))
```

---
## 결론
- concat 사용시에는 completed를 꼭 시켜줘야한다.
- catch는 에러방출후 completed 되므로 concat과 사용하기 좋다.