# AppleFramework - Combine 실습2 ([개념](../Combine.md), [원래 코드](../../../Uikit/Study/AppleFramework/AppleFramework.md))
- [메인 리스트 뷰](#메인-리스트-뷰-업데이트-⬆️)
    - [기존 viewDidLoad](#원래의-viewdidload)
    - Presebtation, layout [함수화](#presentation-layout-함수화-하기)
    - data [함수화](#data를-함수화)
    - [combine 변수 만들기](#combine을-위해-사용할-subscriptions와-subject-만들기)
    - [bine 이용하기](#bind-함수를-통해-combine으로)
    - [정리](#정리)
- [디테일 리스트 뷰](#디테일-리스트-뷰-업데이트-⬆️)
    - [수정 전](#수정-전-전체-코드)
    - [수정 후](#수정-후-코드)
- [💡생각](#생각)


<br><br><br><br><br><br><br>

# 메인 리스트 뷰 업데이트 [⬆️](#appleframework---combine-실습2-개념-원래-코드)
DiffableDatasource의 data, presentation, layout 등을 함수화하고 옮겨줌

<br><br><br>

## 원래의 ViewDidLoad
```swift
// Data, Presentation, Layout
@IBOutlet weak var collectionView: UICollectionView!
    
typealias Item = AppleFramework
enum Section {
    case main
}

var dataSource: UICollectionViewDiffableDataSource<Section, Item>!

let list: [AppleFramework] = AppleFramework.list

override func viewDidLoad() {
    super.viewDidLoad()
    
    // presentation
    dataSource = UICollectionViewDiffableDataSource<Section, Item>(collectionView: collectionView, cellProvider: { collectionView, indexPath, item in
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "FrameworkCell", for: indexPath) as? FrameworkCell else {
            return nil
        }
        cell.configure(item)
        return cell
    })
    
    // data
    var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
    snapshot.appendSections([.main])
    snapshot.appendItems(list, toSection: .main)
    dataSource.apply(snapshot)
    
    // layer
    collectionView.collectionViewLayout = layout()
    
    collectionView.delegate = self
}
```

<br><br><br>

## Presentation, Layout 함수화 하기
```swift
// CollectionView Presentation, Layout 설정주는 것
private func configureCollectionView(){
    // presentation
    dataSource = UICollectionViewDiffableDataSource<Section, Item>(collectionView: collectionView, cellProvider: { collectionView, indexPath, item in
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "FrameworkCell", for: indexPath) as? FrameworkCell else {
            return nil
        }
        cell.configure(item)
        return cell
    })
    // layer
    collectionView.collectionViewLayout = layout()
    
    collectionView.delegate = self
}
```

<br><br><br>

## Data를 함수화
```swift
private func applySectionItems(_ items: [Item], to section: Section = .main){
    // data
    var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
    snapshot.appendSections([section])
    snapshot.appendItems(items, toSection: section)
    dataSource.apply(snapshot)
}
```

<br><br><br>

## Combine을 위해 사용할 subscriptions와 Subject 만들기
didSelect는 bind에서 input(사용자 인풋 받기)<br>
items는 bind에서 output(data,state 변경에 따른 UI 업데이트 부분)

currentValue -> 초기값 있는
Passthrough -> 초기값 없는

```swift
import Combine

let didSelect = PassthroughSubject<AppleFramework, Never>()
let items = CurrentValueSubject<[AppleFramework], Never>(AppleFramework.list)
var subscriptions = Set<AnyCancellable>()
```

<br><br><br>

## bind 함수를 통해 Combine으로
Collection View Data를 설정해주는 부분

didSelect는 아이템 클릭시 처리<br>
items는 세팅 되었을때 컬렉션뷰 업데이트
```swift
private func bind(){
    // input : 사용자 인풋을 받아서, 처리해야할 것
    // - item 선택 되었을때 처리
    didSelect
        .receive(on: RunLoop.main)
        .sink { [unowned self] framework in
        let sb = UIStoryboard(name: "Detail", bundle: nil)
        let vc = sb.instantiateViewController(withIdentifier: "FrameworkDetailViewController") as! FrameworkDetailViewController
            vc.framework.send(framework)
        self.present(vc, animated: true)
    }.store(in: &subscriptions)
    
    // output : data, state 변경에 따라서, UI 업데이트 할 것
    // - items 세팅이 되었을때 컬렉션뷰를 업데이트
    items
        .receive(on: RunLoop.main)
        .sink { [unowned self] list in
            self.applySectionItems(list)
    }.store(in: &subscriptions)
    
}

private func applySectionItems(_ items: [Item], to section: Section = .main){
    // data
    var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
    snapshot.appendSections([section])
    snapshot.appendItems(items, toSection: section)
    dataSource.apply(snapshot)
}
```

<br><br><br>

## 정리
달라진 부분 : 데이터 처리 부분을 다르게 해줌, viewDidLoad에 보기 불편한 부분이 없어짐

Presentation, Layout의 부분을 함수화 시키고 바꿔줌<br>
Data부분을 Combine으로 변경을 하고<br>
input 로직과 output로직을 보기 쉽게 변경함.

최종 viewDidLoad
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // Presentation, Layout 설정
    configureCollectionView()
    // Data 설정
    bind()   
}
```

<br><br><br><br><br><br><br><br>

# 디테일 리스트 뷰 업데이트 [⬆️](#appleframework---combine-실습2-개념-원래-코드)
위와 똑같이 Presentation, Layout, Data부분을 수정

<br><br><br>

## 수정 전 전체 코드
```swift
import UIKit
import SafariServices

class FrameworkDetailViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var descriptionLabel: UILabel!
    
    var framework: AppleFramework = AppleFramework(name: "Unknown", imageName: "", urlString: "", description: "")
    
    override func viewDidLoad() {
        super.viewDidLoad()
        updateUI()
    }
    
    func updateUI() {
        imageView.image = UIImage(named: framework.imageName)
        titleLabel.text = framework.name
        descriptionLabel.text = framework.description
    }
    
    
    @IBAction func learnMoreTapped(_ sender: Any) {
        
        guard let url = URL(string: framework.urlString) else {
            return
        }
        
        let safari = SFSafariViewController(url: url)
        
        present(safari, animated: true)
    }
}
```

<br><br><br>

## 수정 후 코드
### 설명
> buttontapped : input(버튼 클릭)   -> 초기값 없음 <br>
> framework : output(Data 세팅시 UI 업데이트)   -> 초기값 있음

- bind()에다가 동작 로직을 넣었음
    - 버튼 동작시 함수를 가져와서 input()의 publisher에 넣기
    - UI 업데이트 함수를 가져와서 output()의 publisher에 넣기


```swift
import UIKit
import SafariServices
import Combine

class FrameworkDetailViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var descriptionLabel: UILabel!
    
    
    var subscritions = Set<AnyCancellable>()
    let buttonTapped = PassthroughSubject<AppleFramework, Never>()
    let framework = CurrentValueSubject<AppleFramework, Never>(AppleFramework(name: "Unknown", imageName: "", urlString: "", description: ""))
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        bind()
    }
    
    private func bind(){
        // input : Button 클릭
        // framework -> url -> safari -> present
        buttonTapped
            .compactMap { URL(string: $0.urlString)}
            .sink { [unowned self] url in
                let safari = SFSafariViewController(url: url)
                self.present(safari, animated: true)
            }.store(in: &subscritions)
        
        // output : Data 세팅 될때 UI 업데이트
        framework
            .receive(on: RunLoop.main)
            .sink { [unowned self] framework in
                self.imageView.image = UIImage(named: framework.imageName)
                self.titleLabel.text = framework.name
                self.descriptionLabel.text = framework.description
            }.store(in: &subscritions)
    }
    
    
    @IBAction func learnMoreTapped(_ sender: Any) {
        buttonTapped.send(framework.value)
        
    }
     
}
```

<br><br><br><br><br><br><br>

# 생각
Combine을 이용해서 **비동기 처리 데이터**를 움직이는 부분을 실습을 **많이 할줄** 알았는데

생각보다 DiffableDatasource에서 데이터가 이동할때 라던가 그런 **로직에서의 부분이 더욱 잘보이는 실습**이었던 것 같다.
