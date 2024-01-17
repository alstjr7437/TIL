# SymbolRoller 프로젝트
# 사용 기술
- StoryBoard 
    - StackView : 세로(전체 부분) 연습
    - Auto Layout 설정

- ViewController
    - list 통한 데이터 셋팅
    - 버튼 동작 : 버튼 클릭시 이미지 및 라벨 수정

<img src = "image-1.png" width = "50%">

1. UI 만들기
    - StackView - Vertical
        - ImgView
        - Label
        - Btn
2. 버튼 동작 넣기
    - Reload 버튼 클릭시
    - 이미지 변경
    - Label 변경

<br><br><br>

# UI 만들기(StoryBoard)
1. Stack View 만들기
2. Stack View 안에 ImgView, Label, Btn 순서대로 넣기
3. Stack View Auto Layout 설정하기
4. Label, Btn에 Height 설정하기

## 최종 결과 StoryBoard
<img src = "image.png" width = "50%">

<br><br><br>

# ViewController 작업 (버튼 동작 넣기)
하는 방법 : Storyboard에서 오른쪽 상단 ![Alt text](image-2.png) 클릭하여 Assistant 클릭<br>
ViewController가 나오면 해당 부분에 필요한 UI ctrl 눌러서 들고오기<br>
imgView와 Label은 그냥 들고오고 Btn은 IBOutlet과 Action으로 들고오기

## 1. 각 UI들 들고오기
```Swift
@IBOutlet weak var imageView: UIImageView!
@IBOutlet weak var label: UILabel!
@IBOutlet weak var btn: UIButton!
```

## 2. 바껴야할 Symbol들 List 만들어 놓기
```Swift
let symbols: [String] = ["sun.min", "moon", "cloud", "wind", "snowflake"]
```

## 3. 처음 Load시 만들어질 이미지 viewDidLoad에 넣기
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    print(reload(), "로 생성")

    //button 생성시 색깔 변경
    btn.tintColor = UIColor.systemPink
}
```

## 4. 이미지가 바뀌는 동작을 하는 함수 만들기
```Swift
func reload() -> String {
    let symbol = symbols.randomElement()!
    imageView.image = UIImage(systemName: symbol)
    label.text = symbol
    
    return symbol
}
```

## 5. 버튼 누르면 동작할 함수 만들기
```Swift
@IBAction func btnTap(_ sender: Any) {
    print(reload(),"바꿈")
}
```

<br><br><br>

# 💡 알아간 부분
1. 공부 첫 UI 작업
2. 버튼 동작
3. viewDidLoad를 통한 처음 설정

# [전체 코드](https://github.com/alstjr7437/IosFirstStudy/tree/main/SybolRoller/SymbolRoller)
[StoryBoard](https://github.com/alstjr7437/IosFirstStudy/blob/main/SybolRoller/SymbolRoller/Base.lproj/Main.storyboard)<br>
[ViewController](https://github.com/alstjr7437/IosFirstStudy/blob/main/SybolRoller/SymbolRoller/SymbolRollerViewController.swift)