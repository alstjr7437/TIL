# Combine 실습 ([개념](../Combine.md))
- [Publisher & Subscriber](#publisher--subscriber) - [자료](https://developer.apple.com/documentation/combine/publisher)
    - [sink](#sink-실습)
    - [assign(to:)](#assignto--실습)
- [Subject](#subject) - [자료](https://developer.apple.com/documentation/combine/subject)
    - [passthroughsubject](#passthroughsubject)
    - [currentvaluesubject](#currentvaluesubject)
    - [사용 예제](#subject-사용-부분)
- [Subscription](#subscription) - [자료](https://developer.apple.com/documentation/combine/subscription)
- [Published](#published) - [자료](https://developer.apple.com/documentation/combine/published)
- [Foundation and Combine](#foundation-and-combine)
    - [URLSessionTask](#urlsessiontask-publisher)
    - [Notifications](#notifications)
    - [keypath-binding](#keypath-binding)
    - [Timer 만들기](#timer-만들기)
- [Scheduler](#scheduler) - [자료](https://developer.apple.com/documentation/combine/scheduler)
- [Operator](#opreator) - [공식 사이트](https://developer.apple.com/documentation/combine) 확인하기 너무 많음
    - [map](#map)
    - [filter](#filter)
    - [combinelatest](#combinelatest)
    - [removedup](#removedup)
    - [compactmap](#compactmap)
    - [ignoreoutput](#ignoreoutput)
    - [prefix](#prefix)
    - [여러가지 Operator](https://kioo.tistory.com/entry/Operator-Basics-%EC%97%B0%EC%82%B0%EC%9E%90-%EA%B8%B0%EC%B4%88)

<br><br><br><br>

# Publisher & Subscriber [⬆️](#combine-실습)

> Publisher는 데이터 배출 <br>
> Subscriber는 Publisher에게 데이터 요청 <br>
> .sink와 assign이 있음

**Just**는 오직 하나의 값만을 출력하고 끝나게 되는 가장 단순한 형태의 Publisher

```swift
let just = Just(1000)
let subscription = just.sink { value in
        print("Recevied Value: \(value)")
}
// Recevied Value: 1000
```

<br><br>

## sink 실습
> **.pulisher**로 Publisher 만들기 <br>
> **.sink**가 Subscriber로 받아서 출력 
```swift
let arrayPublisher = [1,3,5,7,9].publisher
let subscription2 = arrayPublisher.sink {value in
    print("Recevied Value: \(value)")
}
// Recevied Value: 1
// Recevied Value: 3
// Recevied Value: 5
// ..
```

<br> <br>

## assign(to: ) 실습
arrayPubsliher가 배출 될때 마다 property에 값 셋팅
```swift
class MyClass {
    var property: Int = 0{
        didSet{
            print("Did set property to \(property)")
        }
    }
}

let object = MyClass()
object.property = 3
// Did set property to 3

let subscription3 = arrayPublisher.assign(to: \.property, on: object)
// Did set property to 1
// Did set property to 3
// Did set property to 5
// ..

print("Final Value: \(object.property)")
// Final Value: 9
```

<br><br><br><br>

# Subject [⬆️](#combine-실습)
## PassthroughSubject
output type String, <br>
실패시 형태는 Never(실패를 안한다) 
```swift
let relay = PassthroughSubject<String, Never>()

relay.send("Initial text")

let subscription1 = relay.sink{ value in
    print("subscription1 received value:\(value)")
}

relay.send("Hello!")
relay.send("World!")
// subscription1 received value:Hello! 
// subscription1 received value:World!
```

<br><br>

## CurrentValueSubject
초기값이 필요함.
.value로 마지막 값을 확인 가능
```swift
let variable = CurrentValueSubject<String, Never>("")

variable.send("Initial text")
let subscription2 = variable.sink {value in
    print("subscription2 received value: \(value)")
}
// subscription2 received value: Initial text
// 위에 send로 초기를 지정하지 않으면 
// ubscription2 received value: 
// 가 출력이 됨

variable.send("More text")
// subscription2 received value: More text
print(variable.value)
// More text
```

<br><br>

## Subject 사용 부분
```swift
let publisher = ["Here", "we", "go"].publisher

publisher.subscribe(relay)
// subscription1 received value:Here
// subscription1 received value:we
// subscription1 received value:go
"""
아래와 같은 효과
relay.send("Here")
relay.send("we")
relay.send("go")
"""
publisher.subscribe(variable)
// subscription2 received value: Here
// subscription2 received value: we
// subscription2 received value: go
```

<br><br><br><br>

# Subscription [⬆️](#combine-실습)
구독권 같은 느낌
Subscriber가 publisher에게 붙음 
Subscription이라는 티켓을 줌
```swift
let subject = PassthroughSubject<String, Never>()

let subscription = subject
    .print("[Debug]")       // Debug를 넣어서 subscription 오면 출력함.
    .sink{ value in
        print("Subscriber received value \(value)")
}

subject.send("Hello")
subject.send("Hello again")
subject.send("Hello for the last time!")

// completion을 보내서 관계가 끝남
// finish를 해서 끝낼 수도 있고 Cancle을 해서 끊을 수도 있음
// subject.send(completion: .finished)
subscription.cancel()
subject.send("Hello ?? :(")
```

```
[Debug]: receive subscription: (PassthroughSubject)
[Debug]: request unlimited
[Debug]: receive value: (Hello)
Subscriber received value Hello
[Debug]: receive value: (Hello again)
Subscriber received value Hello again
[Debug]: receive value: (Hello for the last time!)
Subscriber received value Hello for the last time!
[Debug]: receive cancel
```

<br><br><br><br>

# Published [⬆️](#combine-실습)
@Published로 선언된 프로퍼티를 퍼블리셔로 만들어주는 것
```swift
final class SomeViewModel{
    // 1. @Published로 선언
    @Published var name: String = "Kim"
    var age: Int = 25
}

final class Label{
    var text: String = ""
}

let label = Label()
let vm = SomeViewModel()
print("text : \(label.text)")
// text : 

// $을 해서 넣어서 해주면 Publisher로 만들어줌(Subscriber할 수 있음)
// Label의 text를 Vm의 name으로 
vm.$name.assign(to: \.text, on: label)
print("text : \(label.text)")
// text : Kim

// Publisher로 Vm의 name을 수정하면 똑같이 수정됨.
vm.name = "Min"
print("text : \(label.text)")
// text : Min

vm.name = "Seok"
print("text : \(label.text)")
// text : Seok

label.text = "\(vm.age)"
print("text: \(label.text)")
// text : 25
```

<br><br><br><br>

# Foundation and Combine [⬆️](#combine-실습)
## URLSessionTask Publisher
URL에서 받은 데이터를 디코딩해서 원하는 데이터형을 변환 시켜주는 것
```swift
struct SomeDecodable: Decodable{ }

URLSession.shared.dataTaskPublisher(for: URL(string: "https://www.google.com")!)
    .map {data, response in
        return data
    }
    .decode(type: SomeDecodable.self, decoder: JSONDecoder())
```

<br><br>

## Notifications
Notifications에 실어서 데이터를 전송하는 방법도 있음
```swift
let center = NotificationCenter.default
let noti = Notification.Name("MyNoti")
let notiPublisher = center.publisher(for: noti, object: nil)
let subscription1 = notiPublisher
    .print("[Debug]")
    .sink { _ in
    print("Noti Received")
}
// [Debug]: receive subscription: (NotificationCenter Observer)
// [Debug]: request unlimited

center.post(name: noti, object: nil)
subscription1.cancel()
// [Debug]: receive value: (name = MyNoti, object = nil, userInfo = nil)
// Noti Received
// [Debug]: receive cancel
```

<br><br>

## KeyPath binding
UILabel에 Text라는 프로퍼티의 데이터를 assign할 수 있게
```swift
let ageLabel = UILabel()
print("text: \(ageLabel.text)")
// text: nil

// \.text가 keypass
Just(28)
    .map { "Age is \($0)"}
    .assign(to: \.text, on: ageLabel)
print("text: \(ageLabel.text)")
// text: Optional("Age is 28")
```

<br><br>

## Timer 만들기
autoconnect 를 이용하면 subscribe 되면 바로 시작함. <br>
timer pulisher 만들고 subscription이 connect하면 자동으로 connect 됨.
```swift
let timerPublisher = Timer.publish(every: 1, on: .main, in: .common)
    .autoconnect()

let subscription2 = timerPublisher.sink { time in
    print("time: \(time)")
}
// DispatchQueue 없으면 쭉 시간이 출력 됨.


// 5초뒤에 끊어버림
DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    subscription2.cancel()
}

// time: 2024-03-12 14:15:27 +0000
// time: 2024-03-12 14:15:28 +0000
// time: 2024-03-12 14:15:29 +0000
// ..
```

<br><br><br><br>

# Scheduler  [⬆️](#combine-실습)
custom number가 1 -> 메인 스레드 <br>
1제외 다른 것 -> 메인이 아닌 스레드(queue)<br>
서버나 데이터 처리 같은 작업 헤비한건 백그라운드에서 돌리고<br>
다 작업하고 UI 업데이트시 메인 스레드에서 받는다

아래 코드는 queue라는 다른 스레드를 만들어서 돌리고
밑에 DispatchQueue.main이라는 메인 스레드에서 돌리는 부분
```swift
let arrPubsliher = [1,2,3].publisher

let queue = DispatchQueue(label: "custom")

let subscrition = arrPubsliher
    .subscribe(on: queue)
    .map{ value in
        print("transform: \(value), thread: \(Thread.current)")
        return value
    }
    // transform: 1, thread: <NSThread: 0x600001711b80>{number = 2, name = (null)}
    // transform: 2, thread: <NSThread: 0x600001711b80>{number = 2, name = (null)}
    // transform: 3, thread: <NSThread: 0x600001711b80>{number = 2, name = (null)}
    .receive(on: DispatchQueue.main)
    .sink { value in
    print("Receive Value: \(value), thread: \(Thread.current)")
}

// Receive Value: 1, thread: <_NSMainThread: 0x60000170c000>{number = 1, name = main}
// Receive Value: 2, thread: <_NSMainThread: 0x60000170c000>{number = 1, name = main}
// Receive Value: 3, thread: <_NSMainThread: 0x60000170c000>{number = 1, name = main}
```

<br><br><br><br>

# Opreator [⬆️](#combine-실습)
> Publisher에게 온 데이터를 가공해서 Subscriber에게 보내는 역할!

## map
Map은 데이터를 가공해주는 역할 - 숫자 2배로 곱해서 출력
```swift
let numPublisher = PassthroughSubject<Int, Never>()
let subscription1 = numPublisher
//    .print("[Debug]")
    .map { $0 * 2 }
    .sink{value in
        print("Transformed Value: \(value)")
    }

numPublisher.send(10)
numPublisher.send(20)
numPublisher.send(30)
subscription1.cancel()

// Transformed Value: 20
// Transformed Value: 40
// Transformed Value: 60
```

<br><br>

## filter
필터 이용해서 a를 포함한거만 value 출력
```swift
let stringPublisher = PassthroughSubject<String, Never>()
let subscription2 = stringPublisher
//    .print("[Debug]")
    .filter{ $0.contains("a")}
    .sink{value in
        print("Filterd Value: \(value)")
    }

stringPublisher.send("abc")
stringPublisher.send("Kim")
stringPublisher.send("Min")
stringPublisher.send("Seok")
stringPublisher.send("age")
subscription2.cancel()

// Filterd Value: abc
// Filterd Value: age
```

<br><br>

## CombineLatest

### Basic CombineLatest

Publisher와 CombineLatest 선언 부분
```swift
let strPublisher = PassthroughSubject<String, Never>()
let numPubsliher = PassthroughSubject<Int, Never>()

Publishers.CombineLatest(strPublisher, numPubsliher).sink{ (str, num) in
    print("Received: \(str), \(num)")
}
```
|출력 부분||a,1||a,2||b,3|||c,3||c,5|
|---|---|---|---|---|---|---|---|---|---|---|---|
|strPublisher|a|||||b|||c|||
|numPubsliher||1||2||3|||||5|

위와 같이 되므로
아래와 같이 보내면 출력 되는게 없음.
```swift
strPublisher.send("a")
strPublisher.send("b")
strPublisher.send("c")

numPubsliher.send(1)
numPubsliher.send(2)
numPubsliher.send(3)
```

위의 표와 똑같이 보내면 이렇게 됨.
```swift
strPublisher.send("a")
numPubsliher.send(1)
// Received: a, 1
numPubsliher.send(2)
// Received: a, 2
strPublisher.send("b")
// Received: b, 2
numPubsliher.send(3)
// Received: b, 3
strPublisher.send("c")
// Received: c, 3
numPubsliher.send(5)
// Received: c, 5
numPubsliher.send(3)
// Received: c, 3
```
<br><br>

---
### Advanced CombineLatest(로그인)
응용해서 로그인과 비밀번호 테스트를 위한 코드
아이디가 있고 비밀번호 12자 이상
```swift
let usernamePublisher = PassthroughSubject<String, Never>()
let passwordPublisher = PassthroughSubject<String, Never>()

let validatedCrendentialsSubscription = usernamePublisher.combineLatest(passwordPublisher)
    .print("[Debug]")
    .map{ (username, password) -> Bool in
        return !username.isEmpty && !password.isEmpty && password.count > 12
    }
    .sink{ valid in
        print("Credential valid? : \(valid)")
    }

usernamePublisher.send("minseok")
passwordPublisher.send("hello")
// [Debug]: receive value: (("minseok", "hello"))
// Credential valid? : false

passwordPublisher.send("verystrongpassword")
// [Debug]: receive value: (("minseok", "verystrongpassword"))
// Credential valid? : true
```

<br><br>

## Merge
타입이 같아야 머지가 됨 아니면 위에 처럼 튜플형식으로
```swift
let publisher1 = [1,2,3,4,5].publisher
let publisher2 = [300,400,500].publisher
// 아래 부분은 String으로 타입이 같지 않아서 에러가 발생
// let publisher2 = ["300","400","500"].publisher

let mergePublisherSubscrition = Publishers.Merge(publisher1, publisher2)
//    .print("[Debug]")
    .sink { value in
        print("Merge: subscrition recevied value: \(value)")
    }

// Merge: subscrition recevied value: 1
// Merge: subscrition recevied value: 2
// Merge: subscrition recevied value: 3
// Merge: subscrition recevied value: 4
// Merge: subscrition recevied value: 5
// Merge: subscrition recevied value: 300
// Merge: subscrition recevied value: 400
// Merge: subscrition recevied value: 500
```

<br><br>

## removeDup
중복 제거하기
```swift
var subscriptions = Set<AnyCancellable>()

let words = "hey hey there! Mr Mr ?"
    // 공백으로 자르기
    .components(separatedBy: " ")
    .publisher
words
    // 중복 자르기
    .removeDuplicates()
    .sink{ value in
        print(value)
    }.store(in: &subscriptions)

// hey (hey 하나 없어짐)
// there!
// Mr (Mr 하나 없어짐 중복이라.)
// ?
```

<br><br>

## compactMap
해당으로 캐스팅이 가능한 녀석들만 출력하도록 Float("a")는 안되니까
```swift
var subscriptions = Set<AnyCancellable>()

let strings = ["a", "1.24", "3", "def", "45", "0.23"].publisher
strings
    // Float으로 캐스팅이 가능한 녀석들만 출력이 됨.
    .compactMap{ Float($0) }
    .sink{ value in
        print(value)
    }.store(in: &subscriptions)

// 1.24
// 3.0
// 45.0
// 0.23
```

<br><br>

## ignoreOutput
다 ignore해서 그냥 끝남
```swift
let number = (1...10_000).publisher
number
    // 그냥 다 무시하고 Completed 출력
    .ignoreOutput()
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)

// Completed with: finished
```

<br><br>

## prefix
prefix로 앞에 데이터 2개 받고 끝남
```swift
let tens = (1...10).publisher

tens
    // 2개 출력하고 끝남
    .prefix(2)
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)

// 1
// 2
// Completed with: finished
```

[⬆️](#combine-실습)