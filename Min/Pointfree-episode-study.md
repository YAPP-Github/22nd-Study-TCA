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

