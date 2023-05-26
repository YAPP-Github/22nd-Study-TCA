TCA Study
[Part1]
Property Wrapper를 통해, 뷰 렌더링 및 상태 관리를 수행.
[@State, @Binding, @ObservedObject, @EnvironmentObject, @Published, @StateObject]
var body: some View {
    // @State 프로퍼티래퍼가 적용된 상태관리변수 "count"
    @State var count: Int = 0
     ...

    Text("\(self.count)")

    Button(action: { self.count -= 1 }) {
      Text("-")
    }
    Button(action: { self.count += 1 }) {
      Text("+")
    }
}
@ObservedObject
deprecated: [BindableObject, @ObjectBinding] ⇒ [Observable, @ObservedObject]
View에서 @State로 상태 관리 시, view 전환 즉 화면 이동 시(dismiss) 지역변수 value 소멸.
ex) counter 값 증가 후, 메인 화면 이동 → 카운터화면 이동 [값 소멸]
// BindableObject
class AppState: Observable{
    @Published count = 0 
    }
}

struct ContentView: View{
 @ObservedObject var appState = AppState()
 NavigationLink(destination: CounterView(state: self.state))
}

struct CounterView: View{
    // @State count = 0 -> 상위뷰로 이동 시, 값이 소실.
    @ObservedObject var appState: AppState
}
@Binding
단일 @State 변수 사용 시, @Binding 프로퍼티래퍼를 사용하여, 뷰 이동 시 값 보존 및 읽기 쓰기 가능

상위 뷰에서 “@State” 프로퍼티 선언 후, 하위 뷰에 “Binding” 하여 값을 사용하는 방식.

struct ContentView: View{
 @State var count: Int = 0
 NavigationLink(destination: CounterView(state: self.$count))
}

struct CounterView: View{
    @Binding var count: Int
    
}

[Part2]
List
강의에서 ForEach에 “self.state.favoritePrime”를 그냥 넣어서 출력했는데, 사실 해당 타입이 Identifiable 프로토콜을 채택한 경우에만 id 파라미터를 생략할 수 있고, 그 외에는 'ForEach' requires that 'Int' conform to 'Identifiable' 해당 프로토콜을 채택해야함.

List {
      ForEach(self.state.favoritePrimes, id: \.self) { prime in
        Text("\(prime)")
      }
      .onDelete { indexSet in
        for index in indexSet {
          let prime = self.state.favoritePrimes[index]
          self.state.favoritePrimes.remove(at: index)
          self.state.activityFeed.append(.init(timestamp: Date(), type: .removedFavoritePrime(prime)))
        }
      }
    }

[Part3]
번거로운 영구 상태 API(Cumbersome persistent state API)
SwiftUI로 복잡하거나 규모있는 앱을 개발할 경우, 새로운 features가 추가되는 상황에서 어떤 가이드 라인이나 모델링을 명확하게 apple에서 제시해주지 않기 때문에 만들어야한다!

예시) logged-in user을 추가할 때 생기는 확장성. 어떤 명확한 구분없이, 새롭게 기능이 추가될때마다 AppState class에 단순히 추가하는 방식.

class AppState: ObservableObject {
  @Published var count = 0
  @Published var favoritePrimes: [Int] = []
  @Published var loggedInUser: User? = nil //< 
  @Published var activityFeed: [Activity] = [] //<

  struct Activity {
    let timestamp: Date
    let type: ActivityType

    enum ActivityType {
      case addedFavoritePrime(Int)
      case removedFavoritePrime(Int)
    }
  }

  struct User {
    let id: Int
    let name: String
    let bio: String
  }
}
+) TCA에대한 이론적 배경인 것 같다…! Apple이 아직 확장 가능한 방식으로 글로벌 앱 상태를 진정으로 모델링하기 위한 솔루션을 제공하지 않았기 때문에 자체 솔루션을 마련해야 한다


뿔뿔이 흩어진 상태 변이(Scattered state mutation)
View 곳곳에서 mutations이 일어나고 있다. (= not exactly clear how we should organize our mutations.) 각각의 Button의 액션에 따라 상태변화가 이루어짐. 해당 로직들이 정돈되지 못하고 다양하게 적용되고 있다. local 및 global 상태 변화의 구분없이 사용. 즉 상태변화가 어떻게 이루어지는지 한눈에 파악하기 매우 어렵다.

[1] Button Action block에서 mutating State 적용
@ObservedObject var state: AppState
@State var isPrimeModalShown: Bool = false
@State var alertNthPrime: PrimeAlert?
@State var isNthPrimeButtonDisabled = false

Button(action: { self.state.count -= 1 }) 
Button(action: { self.state.count += 1 }) 
Button(action: { self.isPrimeModalShown = true }) 
Button(action: self.nthPrimeButtonAction)
  .sheet(isPresented: self.$isPrimeModalShown) 
  .alert(item: self.$alertNthPrime) 
[2] View에 method 정의
func nthPrimeButtonAction() {
  self.isNthPrimeButtonDisabled = true
  nthPrime(self.state.count) { prime in
    self.alertNthPrime = prime
    self.isNthPrimeButtonDisabled = false
  }
}
[3] class 내에 method를 생성하여 적용
extension AppState {
  func addFavoritePrime() {
    self.favoritePrimes.append(self.count)
    self.activityFeed.append(Activity(timestamp: Date(), type: .addedFavoritePrime(self.count)))
  }

  func removeFavoritePrime(_ prime: Int) {
    self.favoritePrimes.removeAll(where: { $0 == prime })
    self.activityFeed.append(Activity(timestamp: Date(), type: .removedFavoritePrime(prime)))
  }

  func removeFavoritePrime() {
    self.removeFavoritePrime(self.count)
  }

  func removeFavoritePrimes(at indexSet: IndexSet) {
    for index in indexSet {
      self.removeFavoritePrime(self.favoritePrimes[index])
    }
  }
}
3가지 방법 :

Button Action block에서 mutating State 적용
해당 View에 method로 생성하여 적용
class 내에 method를 생성하여 적용
→ Apple은 SwiftUI에서 이 문제를 해결하는 방법에 대한 지침을 제공하지 않는다.


사이드 이펙트에 대한 논의X(No story for side effects)
→ 방법론적인 부분에서, 사이드 이펙트 처리를 어떻게 해야할지 애플이 말해주지 않았다.(TCA도입배경느낌..)

본 강의에서 가장 맞는 방법으로 구현했는데 실제 이것이 옳은 방법인지 확신할 수 없다.

+) combine을 활용한다면 해당 작업처리에 도움이 될 수 있다.

func nthPrimeButtonAction() {
  self.isNthPrimeButtonDisabled = true
  nthPrime(self.state.count) { prime in
    self.alertNthPrime = prime
    self.isNthPrimeButtonDisabled = false
  }
}

상태 관리 구성이 가능하지 않다(State management isn't composable)
Large states를 small state로 분해하여 모듈화하기에 쉬운 방법을 제공해주지 않는다.

struct FavoritePrimes: View {
  @ObjectBinding var state: AppState
  
  extension AppState {
  var favoritePrimesState: FavoritePrimesState {
    get {
      FavoritePrimesState(
        favoritePrimes: self.favoritePrimes,
        activityFeed: self.activityFeed
      )
    }
    set {
      self.favoritePrimes = newValue.favoritePrimes
      self.activityFeed = newValue.activityFeed
    }
  }
}
즉, AppState의 Large states를 FavoritPrimes View에서 요구하는 특정 state로 분해과정이 요구.

하지만, SwiftUI에서는 어떻게 이루어지는지 알려진 정보가 부족.


SwiftUI는 테스트 할 수 없다.(SwiftUI isn’t testable)
state and mutations이 현재 예시 앱에서 View내부에 얽혀있기 때문에 테스트하기 어렵고, 애플이 SwiftUI View를 테스트할 도구나 가이드라인을 주지 않았다.
