# TCA study

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



# The Composable Architecture Tutorials

### Essentials

## Your first feature

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



