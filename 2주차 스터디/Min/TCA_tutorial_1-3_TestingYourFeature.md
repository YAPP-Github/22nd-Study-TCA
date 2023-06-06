# TCA study
# The Composable Architecture Tutorials

## 1-3) Testing Your Feature

- 튜토리얼 1-2까지 만들었던 Counter 앱에 테스트코드를 작성해봐요. 또한 Effect를 실행하고, 특정 입력에 대해 시스템이 어떻게 반응하는지 봅시다.

## Section 1

### Testing state changes, 상태 변화 테스트하기

TCA에서 기능을 테스트하기 위해서 필요한 것은 Reducer입니다. action을 받을때 어떻게 state가 변경되는지, 어떻게 Effect가 실행되는지, data로 Reducer가 어떻게 반응하는지를 볼 수 있습니다. 

TCA에 상태변화는 테스트하기 매우 쉬운 요소입니다. 테스트를 위해 해야하는 것은 단지 action, state가 전달 되었을때 어떻게 상태가 바뀌는지 확인하는 것 뿐입니다.

하지만 TestStore 덕분에 TCA에서 더 쉬운 절차를 밟을 수 있습니다. Test Store는 기능을 위한 테스트가능한 runtime으로 action들을 보냈을때 시스템 내에서 발생하는 모든것을 관찰할 수 있습니다. 이 동작 과정에서 간단한 assertion 코드를 작성할 수 있고, 만약 assertion이 실패하는 경우, failure message를 제공합니다.

#### Step1

- Counter Feature에 대한 테스트 코드를 작성해봅시다. CounterFeatureTests.swift 파일을 생성합니다.
- TCA의 테스팅 도구는 비동기적으로 동작하므로, async method를 사용합니다.

#### Step2

- 테스트 메서드에 TestStore를 생성합니다. initialState로 테스트하려는 실제 Reducer State, trailing closure에 CounterReature()라는 Reducer를 넣습니다.

#### Step3

- 앞서 생성한 store에서 send 메서드를 통해 특정 action을 실행할 수 있습니다.
  - TestStore에서 send를 실행 시, await 키워드를 붙혀서 비동기적으로 실행합니다.

#### Step4

- cmd+U로 테스트를 실행할 수 있고, action이 실행됐을때 State의 변화와 의도대로 실행되었는지를 확인 가능합니다.

```swift
import ComposableArchitecture
import XCTest

@MainActor
final class CounterFeatureTests: XCTestCase {
  func testCounter() async {
    let store = TestStore(initialState: CounterFeature.State()) {
      CounterFeature()
    }
  }
}
```

#### Step5

- TestStore의 send 메서드를 사용할때 trailing closure를 통해 변화가 기대되는 state의 값을 명시해줍니다. 보다 명확한 assertion을 위해서 += 같은 연산을 통해 기대되는 값을 모호하게 명시하는 것 보다는 절대값을 명시하는 것이 좋습니다. 
- 아래와 같이 만약 기대한 결과와 실제 결과가 일치하면 failure message는 발생하지 않습니다.
- 이렇게 테스트 결과를 확인하여 제대로 기능이 동작하는지 확인할 수 있습니다. 하지만, count를 1 증가하고, 1 감소하는 기능은 매우 단순한 기능으로, 실제로는 훨씬 복잡한 기능들이 많습니다. 이런 기능들을 TestStore를 사용하여 간단하게 테스트할 수 있습니다.

```swift
import ComposableArchitecture
import XCTest

@MainActor
final class CounterFeatureTests: XCTestCase {
  func testCounter() async {
    let store = TestStore(initialState: CounterFeature.State()) {
      CounterFeature()
    }

    await store.send(.incrementButtonTapped) {
      $0.count = 1 // count가 1 증가해서 1이 될거라고 예상
    }
    await store.send(.decrementButtonTapped) {
      $0.count = 0 // count가 0이 될거라고 예상
    }
  }
}
```



## Section 2

### Testing Effects, Effect 테스트하기

앞서 Reducer의 가장 중요한 책임들 중 하나를 테스트해봤습니다. action이 수행될 때 state를 어떻게 바뀌는지.
그 다음 중요한 책임은 store에 의해 반환되고 처리되는 Effect입니다. 사이드이펙트에 대한 테스트코드를 작성하는 것은 많은 작업이 필요합니다. 이는 외부 시스템에서 많은 Dependency들을 제어하는 경우가 전형적으로 많기 때문입니다. TCA에서는 이러한 Dependency들에 대한 테스트에 대해 테스트 친화적인 버전을 제공합니다. 이어서 타이머 기능 테스트를 시작해보아요.

#### Step 1

- 이번에도 TestStore를 사용해서 타이머 기능에 대한 테스트코드를 작성해봐요. 

#### Step 2

- 먼저, toggleTimerButtonTapped action이 실행되면, 타이머가 실행되어야 합니다. isTimerRunning이 true가 되는지 확인합니다. 

#### Step 3 ~ 4

- 주의할 점은 테스트 종료 전에 effect들이 계속 동작하면 안됩니다. 따라서, 테스트 종료 전에 타이머를 꺼주어야 합니다.

```swift
import ComposableArchitecture
import XCTest

@MainActor
final class CounterFeatureTests: XCTestCase {
  func testTimer() async {
    let store = TestStore(initialState: CounterFeature.State()) {
      CounterFeature()
    }

    await store.send(.toggleTimerButtonTapped) {
      $0.isTimerRunning = true
    }
    // 아래 처럼 테스트 종료 전에 타이머를 꺼서 테스트 종료 전에 모든 Effect 동작이 끝나도록 해야 함
    await store.send(.toggleTimerButtonTapped) {
      $0.isTimerRunning = false
    }
  }
}
```

#### Step 5

- 타이머가 실행된 이후, 주기적으로 timerTick action이 실행되는지를 확인하기 위해서는 store에서 [`receive(_:timeout:assert:file:line:)`](https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/teststore/receive(_:timeout:assert:file:line:)-1rwdd) 메서드를 사용할 수 있습니다. 이 경우에도 특정 action을 받았을때, 기대되는 state의 값이 설정되는지 assertion할 수 있습니다.
  - TestStore를 통해 action receive 메서드를 사용하기 위해서는 Equatable을 준수하는 Action 타입을 사용해야합니다.

```swift
await store.receive(.timerTick) {
    $0.count = 1
}
```

#### Step 6 ~ 7

- store receive 메서드를 통해 테스트할때, timeout parameter를 통해 특정 action이 올때까지 기다릴 수 있습니다. 
  - 가령 타이머가 1초 간격으로 A액션을 전달할때, timeout을 설정하지 않으면 failure message를 받을 수 있는데, timeout 설정이 안되면 1초를 기다리지 않고, A액션을 받았는지를 체크하기 때문입니다. 이때 timeout parameter를 지정해서 타이머 action이 동작하는지 정상적으로 체크할 수 있습니다.

#### Step 8

- CounterFeature.swift로 돌아가서 timer를 실행할 Dependency를 추가해봅니다. Reducer 내에 아래와 같이 사이드이펙트를 발생시키는 Dependency를 추가해서 사용할 수 있습니다. Task.sleep등의 시스템 기능을 별도 사용할 필요 없이 해당 Dependency에 의존해서 타이머를 동작시킵니다.

```swift
@Dependency(\.continuousClock) var clock
```

#### step 9 ~ 10

- 다시 CounterFeatureTests.swift로 돌아와서 제어가능한 clock을 제공하여 테스트코드를 작성해봅니다. 
- 테스트 코드 내에 앞서 Dependency인 TestClock을 만들어줍니다. 또한 TestStore를 생성할때 withDependencies 레이블을 가진 parameter에 사용할 dependency를 명시합니다. 이후 TestClock을 원하는 시점에 실행시켜서 테스트할 수 있습니다. 
  - TestClock 에 대한 코드는 오픈소스라이브러리인  [swift-clocks](https://github.com/pointfreeco/swift-clocks) 에서 확인할 수 있어요. 많은 유용한 clock 구현들을 제공합니다.
- 이로써 타이머 기능을 동작시켰을때 기능이 정상적으로 수행되는지 TestStore로 테스트하는 방법을 알아봤습니다. 하지만 타이머 기능에 대한 테스트만으로 데이터 로드나 네트워크 요청 등의 다른 사이드 이펙트들에 대한 테스트를 커버할 수 없습니다.



## Section 3

### 테스워크 요청 테스트, Testing network requests

네트워크 요청은 아마 가장 보편적인 사이드 이펙트 중 하나일 거에요. 왜냐하면 외부 서버에서 사용자 데이터를 쥐고 있는 경우가 많기 때문이죠. 테스트할때 네트워크 요청을 만드는 건 쉽지 않아요. 요청을 만드는게 느릴 수도 있고, 서버와의 네트워크 연결에 의존할 수 있고, 어떤 케이스의 데이터가 서버로부터 왔는지 예측할 방법이 없기 때문이에요.

네이티브한 방법으로 실제 행위들에 대한 테스트 코드를 작성해보고, 어떤 문제가 있는지 봅시다.

#### Step 1

- CounterFeatureTests.swift에 다시 테스트코드를 작성해봅니다. 이번에도 [`TestStore`](https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/teststore)를 사용합니다.
  - 테스트를 위해 실제 버튼을 텝했을때 네트워크 요청이 일어나는 과정을 흉내내봅니다.

#### Step 2

- 버튼 탭이 일어나고 factButtonTapped action이 수행되며, isLoading은 true가 되어야 하고, progress indicator가 보여집니다. 이후 시스템에 데이터가 수신될 겁니다.

#### Step 3

- step 2 까지 했을때 failure 가 발생합니다. 테스트 종료 전까지 네트워크 요청이 끝나지 않기 때문입니다. 

#### Step 4

- 이러한 문제를 해결하기 위해 실제 네트워크 응답을 받을 필요가 있습니다. timeout을 지정하고, 서버 응답을 기다려봅니다. 이때 예상되는 응답 요청을 trailing closure에 지정합니다.

```swift
import ComposableArchitecture
import XCTest

@MainActor
final class CounterFeatureTests: XCTestCase {
  func testNumberFact() async {
    let store = TestStore(initialState: CounterFeature.State()) {
      CounterFeature()
    }

    await store.send(.factButtonTapped) {
      $0.isLoading = true
    }
    await store.receive(.factResponse("???"), timeout: .seconds(1)) {
      $0.isLoading = false
      $0.fact = "???"
    }
  }
}
```

#### Step 5

- 만약 서버로부터 예상하지 못한 response를 받게 되면 fact는 "???"값이 아닌 다른 값을 받게 될 수도 있으며 이때 failure message를 받게 됩니다.
- 지금까지의 과정을 통해 알 수 있는 것은 이러한 행위에 대한 테스트에 대한 방법이 없다는 것입니다.
- 각각의 요청에 서버는 다른 fact를 내려줄 수 있습니다. 또한 서버로부터 받을 데이터를 예측할 수 있다고 손 쳐도, 이상적이지 않을 수 있는데, 이러한 테스트는 느리고, 불안정적일 수 있습니다. 외부 요인인 인터넷 연결상태나 서버의 상태에 따라 변수가 있기 때문입니다.

```swift
// ❌ A state change does not match expectation: …
//
//       CounterFeature.State(
//         count: 0,
//     −   fact: "???",
//     +   fact: "0 is the atomic number of the theoretical element tetraneutron.",
//         isLoading: false,
//         isTimerRunning: false
//       )
```



## Section 4

### Dependencies 제어, Controlling dependencies

지금까지 기능 코드에서 네트워크 요청과 같은 제어할 수 없는 Dependencies를 사용했을때의 문제점을 알아봤습니다. 이런 경우 테스트가 장시간 소요되거나 불안정하게 동작할 수 있었습니다.

이러한 이유들로 인해, 외부 시스템에 대한 Dependency들을 제어하는 것이 좋습니다. (더 자세한 내용은 [Dependencies](https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/dependencymanagement) 참고)
TCA에서는 앱 내에서 제어되거나 전파되는 Dependencies를 위한 도구 셋을 갖고 있습니다. 

#### Step 1

- NumberFactClient.swift 라는 파일을 새로 생성하고 ComposableArchitecture를 import 합니다. ComposableArchitecture를 import하면 기능 상 dependency 제어를 위해 필요한 도구들을 접근할 수 있습니다.

#### Step 2

- dependency를 제어하는 첫 시작은 dependency를 추상화하는 인터페이스를 모델링하는 것입니다. 이는 테스트에 사용이 됩니다.
  - 프로토콜이 dependency 인터페이스를 추상화하는 가장 유명한 방법이지만 유일한 방법은 아닙니다. TCA에서는 변경가능한 프로퍼티를 가진 struct를 사용하는 것을 선호합니다. 이 방식에 대한 더 자세한 내용은 [series of videos](https://www.pointfree.co/collections/dependencies) 를 참고하세요.

```swift
import ComposableArchitecture

struct NumberFactClient {
  var fetch: (Int) async throws -> String
}
```

#### Step 3

- 그 다음, Dependency를 library에 등록해야합니다. 먼저, DependencyKey 프로토콜을 채택해서 liveValue를 정의해야합니다. liveValue는 실제 앱과 시뮬레이터에서 기능이 동작할때 사용됩니다. 또한 실제 네트워크 요청을 만들기에 적절합니다.
  - 기술적으로 TCA의 dependency 관리 시스템은 [swift-dependencies](https://github.com/pointfreeco/swift-dependencies) 라는 또다른 라이브러리에서 제공됩니다. 

```swift
import ComposableArchitecture
import Foundation

struct NumberFactClient {
  var fetch: (Int) async throws -> String
}

extension NumberFactClient: DependencyKey {
  static let liveValue = Self(
    fetch: { number in
      let (data, _) = try await URLSession.shared
        .data(from: URL(string: "http://numbersapi.com/\(number)")!)
      return String(decoding: data, as: UTF8.self)
    }
  )
}
```



#### Step 4

- Dependency를 등록하는 두번째 단계로 DependencyValues에 getter, setter를 가진 계산프로퍼티를 추가하는 겁니다. 이걸 추가해야 @Dependency(\.numberFact)와 같이 Reducer에서 사용할 수 있게 됩니다.
  - dependency를 등록하는 것은 SwiftUI에서 default Value를 제공하기 위해 EnvironmentKey를 준수하여 Environment value를 등록하는 것, 계산 프로퍼티 제공을 위해 EnvironmentValues를 확장하는 것과 다릅니다. 
- 이렇게 하나의 dependency에 대한 제어가능한 인터페이스를 만들었습니다. 이제 dependency의 테스트 친화적인 버전을 사용해서 테스트를 해봅니다.

```swift
extension DependencyValues {
  var numberFact: NumberFactClient {
    get { self[NumberFactClient.self] }
    set { self[NumberFactClient.self] = newValue }
  }
}
```



#### Step 5

- CounterFeature.swift로 돌아와서 @Dependency propertywrapper 방식으로 새로운 dependency를 추가합니다.

#### Step 6

- 이어서 앞서 봤던 CounterFeatureTests.swift로 돌아가서 테스트를 진행합니다. 역시나 failure가 발생하지만 새로운 로그가 추가로 발생합니다. 네트워크 요청을 받았거나, 디스크에 무언가 작성했거나, analytics를 추적할때마다 알려주기 때문에 유용합니다.
- 현재는 실제 네트워크 요청을 받으며 테스트할게 아니므로, 테스트 가능하도록 이를 고쳐봅니다. 

```swift
// ❌ @Dependency(\.numberFact) has no test implementation, but was
//    accessed from a test context:
//
//   Location:
//     TCATest/ContentView.swift:70
//   Dependency:
//     NumberFactClient
//
// Dependencies registered with the library are not allowed to use
// their default, live implementations when run from tests.
```

- 이번에도 TestStore를 생성할때 withDependencies parameter에 numberFact의 fetch 메서드에 대한 클로져를 정의해줍니다. 이제 fetch를 통해 정수 값을 받으면 그에 대한 mock response로 특정 문구를 반환할 것입니다.
- 아래와 같이 numberFact client의 fetch 기능을 재정의해서 예측가능한 결과를 만들 수 있었고, 그에 대한 테스트가 가능해졌습니다.

```swift
import ComposableArchitecture
import XCTest

@MainActor
final class CounterFeatureTests: XCTestCase {
  func testNumberFact() async {
    let store = TestStore(initialState: CounterFeature.State()) {
      CounterFeature()
    } withDependencies: {
      $0.numberFact.fetch = { "\($0) is a good number." }
    }

    await store.send(.factButtonTapped) {
      $0.isLoading = true
    }
    await store.receive(.factResponse("0 is a good number.")) {
      $0.isLoading = false
      $0.fact = "0 is a good number."
    }
  }
}
```



- 지금까지 기능을 테스팅하는 방법을 알아봤습니다. 테스트를 위해 몇가지 선행단계가 있었지만 일단 그 단계를 밟고 나면, 어떤 기능에서든 이 Dependency를 테스트할 수 있습니다. 또한 테스트 코드 작성을 넘어서 dependency들을 제어할 때 많은 장점이 있는데, 예를들면 Xcode의 preview 기능을 사용하거나 사용자들을 위해 온보딩을 제공할때, 예측불가능한 상황을 배제하고 예측가능한 상태로 앱을 동작 시킬 수 있습니다.
