# SymbolRoller 프로젝트
# 사용 기술
- StoryBoard 
    - StackView : 세로(전체 부분) 가로(요일 부분) 연습
    - Auto Layout 설정

- ViewController
    - list 통한 데이터 셋팅
    - 버튼 동작 : 버튼 클릭시 이미지 및 라벨 수정

<img src = "image-1.png" width = "50%">

1. UI 만들기
    - StackView - Vertical
        - Label
        - ImgView
        - Label
        - StackView - Horizontal
            - Label
            - ImgView
            - Label
        - Btn
2. 버튼 동작 넣기
    - Change City 버튼 클릭시
    - Label 변경
    - ImgView 변경

<br><br><br>

# UI 만들기(StoryBoard)
1. Stack View - Vertical(전체) 만들기
2. Stack View 안에 Label, ImgView, Label, StackView - Horizontal(주차), Btn 순서대로 넣기
3. Auto Layout 설정하기
4. StackView - Horizontal 만들기 - Label, ImgView, Label 순서대로 넣기 (5개)
5. Auto Layout 설정하기


## 최종 결과 StoryBoard!
<img src = "image.png" width = "50%">

<br><br><br>

# ViewController 작성(버튼 동작 넣기)
## 1. 각 UI들 들고오기
```Swift
@IBOutlet weak var cityLabel: UILabel!
@IBOutlet weak var weatherImgView: UIImageView!
@IBOutlet weak var temperatureLabel: UILabel!
```

## 2. 바껴야할 Symbol들 List 만들어 놓기
```Swift
let cities = ["Seoul", "Tokyo", "LA", "Seattle"]
let weathers = ["cloud.fill", "sun.max.fill", "wind", "cloud.sun.rain.fill"]  
```

## 3. 버튼 누르면 동작할 함수 만들기
```Swift
@IBAction func changeButtonTapped(_ sender: Any) {
    cityLabel.text = cities.randomElement()

    // list 이용해서 이미지 변경해주기
    // .withRenderingMode를 통해서 이미지 색상 및 모드 설정 
    let imageName = weathers.randomElement()
    weatherImgView.image = UIImage(systemName: imageName!)?.withRenderingMode(.alwaysOriginal)
    
    //random 이용해서 10°C에서 30°C로 만들기
    let randomTemp = Int.random(in: 10..<30)
    temperatureLabel.text = "\(randomTemp)°C"

    print("도시, 온도, 날씨 이미지 변경하기!")
}
```

<br><br><br>

# 💡 알아간 부분
1. 각 상위 또는 본인의 AutoLayout 설정하기
2. StackView안의 StackView

# [전체 코드](https://github.com/alstjr7437/IosFirstStudy/tree/main/SimpleWeather/SimpleWeather)
[StoryBoard](https://github.com/alstjr7437/IosFirstStudy/blob/main/SimpleWeather/SimpleWeather/Base.lproj/Main.storyboard)<br>
[ViewController](https://github.com/alstjr7437/IosFirstStudy/blob/main/SimpleWeather/SimpleWeather/WeatherViewController.swift)