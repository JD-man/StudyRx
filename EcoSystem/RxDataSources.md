# RxDataSources

## 목차
1. 사용이유
2. 이전
3. 사용방법
4. 애니메이션

---

## 1. 사용이유
- TableView 또는 CollectionView를 사용할때 복잡한 데이터셋을 다루고 싶을때 기본방식으로는 힘들다.
- 셀이 변경될때의 애니메이션이 필요하거나 다양한 섹션을 다루고 싶을때 사용하면 좋다.

---

## 2. 이전

```swift
var beforeTestData = ["first", "second", "third"]
var beforeRelay = PublishRelay<[String]>()

func before() {
    beforeRelay.asDriver(onErrorJustReturn: [""])
        .drive(tableView.rx.items(cellIdentifier: "TestTableViewCell", cellType: TestTableViewCell.self)) {
        row, element, cell in
        cell.label.text = element
    }.disposed(by: disposeBag)
    
    tableView.rx.itemSelected
        .asDriver()
        .drive { [weak self] _ in
            self?.beforeTestData.append("추가데이터")
            self?.beforeRelay.accept(self!.beforeTestData)
        }.disposed(by: disposeBag)
    
    beforeRelay.accept(beforeTestData)
}
```
<table align="center">
<tr>
<td align="center"> Result </td>
</tr>
<tr>
<td>
<p align="center">
<video src= https://user-images.githubusercontent.com/62129500/148647591-93f8c41b-2405-4a26-8175-e7f7d5eabf18.mp4 width = 70%>
</p>
</td>
</tr>
</table>

- 섹션을 따로 나누기도 어렵고 애니메이션이 없어서 노잼이다..

---

## 3. 사용방법
### 1. 사용할 데이터와 섹션 구조체를 만든다.
```swift
struct CellModel {
    var text: String
}

struct SectionOfCellModel: SectionModelType {
    typealias Item = CellModel
    
    var items: [CellModel]
    var headerTitle: String
    init(original: SectionOfCellModel, items: [CellModel]) {
        self = original
        self.items = items
    }
}
```
- 사용할 데이터는 타이틀로 사용할 headerTitle과 cell에 들어있는 label에 사용할 text 두개다.
- 섹션으로 사용할 모델은 SectionModelType 프로토콜을 채택한다.
- Item은 사용할 데이터의 타입이다.

### 2. RxTableViewSectionedReloadDataSource를 만든다.
```swift
let dataSource = RxTableViewSectionedReloadDataSource<SectionOfCellModel> { dataSource, tableView, indexPath, item in
    let cell = tableView.dequeueReusableCell(withIdentifier: "TestTableViewCell", for: indexPath) as! TestTableViewCell
    cell.label.text = item.text
    return cell
}

// 섹션 타이틀을 사용하고 싶다면 요거 추가
dataSource.titleForHeaderInSection = { dataSource, index in
    return dataSource.sectionModels[index].headerTitle
}
```
- RxTableViewSectionedReloadDataSource를 작성하고 (configuration:) 작성하고 엔터치면 4개의 매개변수를 작성하게끔 되어있다.

### 3. section data 만들고 bind

```swift
var afterSectionData = [
    SectionOfCellModel(headerTitle: "First Section", items: ["0", "1", "2"].map {
        CellModel(text: "First \($0)")
    }),
    SectionOfCellModel(headerTitle: "Second Section", items: ["0","1"].map {
        CellModel(text: "Second \($0)")
    }),
    SectionOfCellModel(headerTitle: "Third Section", items: ["0","1","2","3","4"].map {
        CellModel(text: "Third \($0)")
    })
]

afterRelay
    .asDriver(onErrorJustReturn: [])
    .drive(tableView.rx.items(dataSource: dataSource))
    .disposed(by: disposeBag)

afterRelay.accept(afterSectionData)

```
- section data는 SectionOfCellModel 구조체다!. 이거의 items로 CellModel 배열을 만들어야함.
- section data 만들고 tableView.rx.items(dataSource:)를 사용해 bind

### 4. 터치시 데이터 추가하기

```swift
let newCellData = CellModel(text: "추가데이터")
        
tableView.rx.itemSelected
    .asDriver()
    .drive { [weak self] _ in
        let idx = Int.random(in: 0 ..< self!.afterSectionData.count)
        self?.afterSectionData[idx].items.append(newCellData)
        self?.afterRelay.accept(self!.afterSectionData)
    }.disposed(by: disposeBag)
```
- 랜덤으로 섹션별 새로운 데이터를 추가한다. 섹션으로 나눠서 작업돼서 좋다.

<table align="center">
<tr>
<td align="center"> Result </td>
</tr>
<tr>
<td>
<p align="center">
<video src= https://user-images.githubusercontent.com/62129500/148647627-9f7f49a6-c6cc-428b-8396-cdc585b51e93.mp4 width = 70%>
</p>
</td>
</tr>
</table>

- 그래도 애니메이션은 없다!!??

---

## 4. 애니메이션
- 애니메이션이 있어야 유잼이기 때문에 꼭 해야한다.
- 3가지 변경사항이 있다.

### 변경사항 1: Data Model IdentifiableType, Equatable 프로토콜 채택
```swift
struct CellModel: IdentifiableType, Equatable {
    typealias Identity = String
    var identity: String
    var text: String
}
```
- IdentifiableType 프로토콜을 채택하면서 identity를 설정해줘야한다.
- 만약 새로 생성된 데이터의 identity가 같다면 fatalError가 나온다.

### 변경사항 2: Section Model에 AnimatableSectionModelType 프로토콜 채택
```swift
struct SectionOfCellModel {
    var headerTitle: String
    var items: [CellModel]    
}

extension SectionOfCellModel: AnimatableSectionModelType {
    var identity: String {
        return headerTitle
    }
    
    typealias Identity = String
    
    typealias Item = CellModel
    init(original: SectionOfCellModel, items: [CellModel]) {
        self = original
        self.items = items
    }
}

```
- AnimatableSectionModelType 프로토콜을 채택하면서 identity를 설정해줘야한다.
- 만약 새로 생성된 섹션의 identity가 같다면 fatalError가 나온다.

### 변경사항 3: dataSource 변경
```swift
let dataSource = RxTableViewSectionedAnimatedDataSource<SectionOfCellModel> { dataSource, tableView, indexPath, item in
    let cell = tableView.dequeueReusableCell(withIdentifier: "TestTableViewCell", for: indexPath) as! TestTableViewCell
    cell.label.text = item.text
    return cell
}
```
- RxTableViewSectionedAnimatedDataSource를 사용하도록 변경

### 변경사항 4: Model 생성 방식 변경
```swift
var newID = 100
tableView.rx.itemSelected
    .asDriver()
    .drive { [weak self] _ in
        let idx = Int.random(in: 0 ..< self!.afterSectionData.count)
        let newCellData = CellModel(identity: "\(newID)", text: "추가데이터")
        self?.afterSectionData[idx].items.append(newCellData)
        self?.afterRelay.accept(self!.afterSectionData)
        newID += 1
    }.disposed(by: disposeBag)
```
- identity가 중복되지 않게 새로운 데이터를 만든다.

<table align="center">
<tr>
<td align="center"> Result </td>
</tr>
<tr>
<td>
<p align="center">
<video src= https://user-images.githubusercontent.com/62129500/148647649-b695435a-bbfe-44b3-9bb2-f009c9ce88e5.mp4 width = 70%>
</p>
</td>
</tr>
</table>

- 이맛이지
