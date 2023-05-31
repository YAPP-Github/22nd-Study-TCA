# TCA study
# The Composable Architecture Tutorials

### Essentials

## 1-1) Your first feature

- 기능의 행위와 로직을 구현하기 위해 ReducerProtocol를 준수하여 생성하는 방법을 알아보아요.

### ReducerProtocol

- Composable Architecture의 기능을 만들때 기초 단위
- system에 action이 전해질때 state가 변경되는 로직, 어떤 방식으로 외부 세계와 데이터를 교류하는지 등이 포함됩니다.
- 또한 SwiftUI View와 분리되어 코어 로직과 동작들을 구현할 수 있습니다. 따라서 재사용 및 테스트에 용이합니다.

### ReducerProtocol 만들기

- ComposableArchitecture를 import한다.
- ReducerProtocol을 채택한 CounterFeature 구조체를 구성한다. 그 안에는 State, Action, body 등이 들어간다.
  - State : 기능 비즈니스 로직에 사용될 상태
  - Action : 사용자가 기능에서 수행할 수 있는 action들이 정의됩니다. action이 수행될때 필요에 따라 state가 변경되고 변경된 state에 맞게 View를 랜더링할 수 있어요.
    - Tip : Action case명은 비즈니스 로직의 동작보다는 유저가 수행하는 행동에 기반하여 이름을 짓는게 최고에요.
- ReducerProtocol을 준수하기 위해, 마지막으로 action에 따른 비즈니스로직을 수행하는 reduce(into:action:)을 구현해줍니다.
  - reduce(into:action:) 내에서 사용되는 state는 inout parameter이므로, 원본을 참조하여 변경을 할 수 있어요.
  - reduce(into:action:) 내에서는 EffectTask<Action>을 반환해야해요. 이는, 비즈니스로직을 수행하고 다음 동작할 Action을 의미합니다. 만약 이후 더이상 취할 Action이 없으면 Empty Publisher로 정의되어있는 .none을 반환합니다.

### SwiftUI View와 앞서 만든 ReducerProtocol를 통합하기

- 일반적으로 ReducerProtocol과 View는 파일을 분리하는게 좋지만, 튜토리얼에서는 한 파일에 작성을 합니다.
- 앞서 만든 Feature에 대한 ReducerProtocol을 사용하기위해 StoreOf<CounterFeature>타입의 store를 선언합니다.
  - store로부터 state를 직접적으로 읽거나, store에서 action을 직접적으로 보낼 수는 없어요.
  - Tip : store는 상수로 선언됩니다. view에 의해 감지될 필요가 없어요.
- SwiftUI View는 일반적인 방식으로 구현할 수 있습니다. 그 View를 store를 parameter로 넘기는 WithViewStore로 감싸줍니다.
  - trailing closure 인자로 ObservableObject인 viewStore를 사용할 수 있습니다. store대신 viewStore를 통해서 state를 접근하거나 action을 보낼 수 있어요.
  - Tip : 기본적으로 View 내 모든 것을 감지할 수 있지만, WithViewStore의 observe 클로져를 통해 정말 필요한 영역만 감지를 하도록 할 수 도 있어요. 자세한 것은 [Performance](https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/performance/) 를 참고 할 수 있어요.
- TCA 방식으로도 PreviewProvider를 구현해서 Preview를 볼 수 있습니다.

### App과 통합하기

- 지금까지 Composable Architecture의 기능과 SwiftUI View를 통합해서 PreviewProvider를 정의해서 Preview로도 확인해볼 수 있었습니다.
- 이제 실제 App의 entry point에 TCA를 적용해봐요.
- MyApp: App { ... } 의 body에 최상위 뷰를 정의합니다. 앞서 만든 CounterView가 최상위 View라면, 이를 추가해주면 됩니다. 여기서 StoreOf<CounterFeature> 타입의 store도 초기상태를 정의합니다.
- Reducer들은 _printChanges(_:) 라는 메서드를 제공합니다. 이를 사용하면 해당 Reducer 내에서 발생하는 action과 action으로 state가 어떻게 변경되는지도 출력할 수 있습니다. 
  - 실제 어떤 Action에 대해 변경된 state가 없으면, (No state changes)가 함께 출력됨.



## 1-2) Adding side effects

- 기능과 바깥세계 간 소통하고, 데이터를 받는 방법을 알아보아요.

### 사이드 이펙트가 뭔가?

- 기능 개발에 사이드 이펙트는 가장 중요한 면입니다. 사이드 이펙트는 아래와 같은 외부 세계와의 소통을 할 수 있게 해줍니다.
  - API 요청
  - 파일 시스템과의 상호작용
  - 시간 기반의 비동기 작업
- 사이드 이펙트 없이는 사용자들을 위한 실질적 가치를 지닌 어떠한 것도 할 수 없어요.
- 1-1) Your first feature에서 만들었던 Reducer와 View를 이어서 봅시다.

### CounterFeature Reducer를 이어서 보자

- 앞서 다뤘듯이, CounterFeature Reducer의 State는 어떤 Action이 보내지냐에 따라 변경될 수 있습니다.
- 아래를 보면 어떤 Action이 동작하냐에 따라 count가 감소하거나 증가하는 것을 볼 수 있어요.
  - 각각의 Action은 이어서 동작한 Action이 없기에, .none을 반환하고 있습니다.

~~~swift
import ComposableArchitecture

struct CounterFeature: ReducerProtocol {
  // ...
  func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
    switch action {
    case .decrementButtonTapped:
      // decrementButtonTapped action이 동작하면, state의 count 값이 1 감소한다.
      state.count -= 1
      return .none

    case .incrementButtonTapped:
      // incrementButtonTapped action이 동작하면, state의 count 값이 1 증가한다.
      state.count += 1
      return .none
    }
  }
}
~~~



### Reducer 내에 API 요청 기능 구현하기

- TCA에서는 복잡한 사이드 이펙트로부터 State의 단순하고 순수한 변형을 분리합니다. 사이드이펙트에 대한 적절한 처리를 하기 위해 EffectTask를 사용할 수 있습니다. 이 개념은 다음 섹션에서 봐요.
- EffectTask는 run 이라는 기능으로 Store에 의해 비동기 단위 동작을 처리할 수 있도록 합니다. 여기에서 특정 API로부터 비동기로 데이터를 요청할 수 있습니다. 단, run의 sendable한 trailing closure에서는 inout parameter인 state를 캡쳐할 수 없습니다. 이는 사이드이펙트와 state의 변경 작업을 분리하기 위함입니다.
- 비동기 동작 실패에 대한 에러를 신경쓸 생각이 없다면, 모르겠지만, 에러처리까지 하고 싶다면, TaskResult 타입을 사용해서 성공, 실패에 대한 결과 처리를 할 수 있습니다. Result 대신 TaskResult를 Action case의 연관타입으로 사용되면 그 자체로 Equatable을 준수 할 수 있습니다. 

~~~swift
case .factButtonTapped:
      state.fact = nil
      state.isLoading = true
      return .run { [count = state.count] send in
				// EffectTask의 run을 사용하면 블럭 내에서 async 동작을 수행하고, system에 추가적인 Action을 보낼 수도 있어요.
        // ✅ Do async work in here, and send actions back into the system.
        let (data, _) = try await URLSession.shared
          .data(from: URL(string: "http://numbersapi.com/\(count)")!)
        let fact = String(decoding: data, as: UTF8.self)
        // 비동기 동작이 끝난 후, 아래와 같이 추가적으로 Action을 동작할 수 있습니다.
        await send(.factResponse(fact))
      }

    case let .factResponse(fact):
			// .factButtonTapped action 수행 후, .factResponse action이 수행되며 추가적인 state 변환이 진행됩니다.
      state.fact = fact
      state.isLoading = false
      return .none
~~~

ReducerProtocol을 채택한 Reducer 구조체를 적절하게 구성하고, View와 연결하면, 기존 SwiftUI 기능인 Preview 기능을 통해 View를 확인할 수 있어요.

### 타이머 관리, Managing a timer

- Network 요청은 가장 흔한 사이드 이펙트 중 하나이지만, 이는 한가지 종류에 불과해요. 그 외에 타이머 기능도 구현해봐요.
- run sendable trailing closure 내에 Task.sleep을 사용하면 비동기 동작 중, 특정 시간 대기를 할 수 잇습니다. 아래 코드는 무한 루프 내에서 1초마다 timerTick action이 동작합니다. 타이머와 같은 동작을 하지만, 아래 코드는 타이머를 종료할 수 없는 문제가 있습니다.

~~~swift
case .toggleTimerButtonTapped:
  state.isTimerRunning.toggle()
  return .run { send in
    while true {
      try await Task.sleep(for: .seconds(1))
      await send(.timerTick)
    }
  }
~~~

#### cancellable(id:cancelInFlight:), cancel(id:)로 타이머 종료하기

- 무한루프가 동작하는 run trailing closure 끝에 .cancellable(id:)를 사용하고, 이때 지정한 id를 통해 타이머를 종료하고자 하는 시점에 .cancel(id:)를 사용하면 타이머를 종료할 수 있습니다.

~~~swift
case .toggleTimerButtonTapped:
  state.isTimerRunning.toggle()
  if state.isTimerRunning {
    return .run { send in
      while true {
        try await Task.sleep(for: .seconds(1))
        await send(.timerTick)
      }
    }
    .cancellable(id: CancelID.timer)
  } else {
    return .cancel(id: CancelID.timer)
  }
}
~~~
  
<br>
<br>  


### SwiftUI and State Management: Part 1

- https://www.pointfree.co/collections/swiftui/state-management/ep65-swiftui-and-state-management-part-1
- notice : BindableObject -> ObservableObject로 rename 되었음

- SwiftUI는 Xcode 11 beta+, Swift 5.1+ 에서 사용 가능합니다.

#### Accounting App

- 숫자를 설정하고, 소수인지(소수라면 소수임을 알려주고 저장 or 삭제 가능) N번째 소수가 무엇인지 판별 가능
- 저장한 가장 좋아하는 소수 리스트 확인 기능

- PlaygroundPage.current.liveView에 SwiftUI View를 래핑한 UIHostingController를 넣으면 playground에서 View를 볼 수 있음
- NavigationView 블럭 안에 NavigationLink를 사용하면 버튼 역할 + 누르면 특정 뷰로 네비게이션 전환 할 수 있음

~~~swift
func foo() -> Int {
  fatalError()
}

// MARK: ContentView

struct ContentView: View {
  var body: some View {
    // body 계산 프로퍼티는 View를 준수하는 객체를 반환한다.
    NavigationView {
      List {
        // "Counter demo"를 눌렀을때 EmptyView()가 보여집니다.
        NavigationLink(destination: EmptyView()) {
         Text("Counter demo")
        }
        NavigationLink(destination: EmptyView()) {
         Text("Favorite primes")
        }
      }
    }
  }
}
~~~

- Button 은 안에 Label View를 설정할 수 있고, 클릭 시의 이벤트 클로져를 설정 가능

~~~swift
struct CounterView: View {
  var body: some View {
    VStack {
      HStack {
        Button(action: {
          // tap action에 대한 callback event 설정 가능
        }) {
          // 버튼에 대한 label view가 설정
          Text("-")
        }
        Text("0")
        Button(action: {}) {
          Text("+")
        }
      } //: HStack
      Button(action: {}) {
        Text("Is this prime?")
      }
      Button(action: {}) {
        Text("What is the 0th price?")
      }
    }
  }
}
~~~

- ObservableObject
  - ObservableObject는 AnyObject를 상속받고 있기 때문에, class만 채택하여 사용할 수 있어요.
  - BindingObject -> ObservableObject로 이름이 변경됨
  - ObjectBinding -> ObservedObject로 변경됨

~~~swift
import SwiftUI
import Combine

class AppState: ObservableObject {
    var count = 0 {
        willSet {
            // count 값이 설정 되면, publisher에서 이벤트를 방출할 수 있음
            objectWillChange.send()
        }
    }

    init() {}

    /// public var objectWillChange: ObservableObjectPublisher
    /// - ObservableObject protocol은 Failure타입이 Never인 ObservableObjectPublisher타입의 objectWillChange publisher가 정의되어 있습니다.
    var objectWillChange: ObservableObjectPublisher = .init()
}
~~~



### SwiftUI and State Management: Part 2

- State property wrapper로 선언한 flag 형태 변수로 모달 표출 여부를 정의할 수 있음

~~~swift
@State var isPrimeModalShown: Bool = false
~~~

- onDelete viewModifier로 삭제 이벤트 처리가 가능 

~~~swift
struct FavoritePrimeView: View {
    // AppState의 @Published 멤버가 변경되면 View는 랜더링 됨.
    // ObservedObject 멤버를 가진 FavoritePrimeView는 @MainActor가 적용됨
    @ObservedObject var state: AppState

    var body: some View {
        List {
            ForEach(self.state.favoritePrimes) { prime in
                Text("\(prime)")
            }
            .onDelete { indexSet in
                // 삭제 이벤트 처리
                for index in indexSet {
                    self.state.favoritePrimes.remove(at: index)
                }
            }
        }
        .navigationBarTitle("Favorite Primes")
    }
}
~~~



### SwiftUI and State Management: Part 3

- SwiftUI를 사용하면 선언형 프로그래밍으로 UI를 개발하게 된다.
- SwiftUI는 UIKit에 비해 매우 간결하게 UI를 구성할 수 있다.
- SwiftUI의 body는 계산프로퍼티로 되어있다. @ObservedObject, @StateObject 등의 프로퍼티 래퍼에 의해 View의 변화가 결정될 수 있다.
- UIKit의 경우, VC가 너무 큰 비즈니스 로직을 갖는 것을 알 수 있다, UIView, UIViewController가 동일한 이벤트를 받는 경우가 많다. 어디에서 비즈니스로직이 수행되고, 뷰가 업데이트되는지 등을 명확하게 구분하기 어렵다.
