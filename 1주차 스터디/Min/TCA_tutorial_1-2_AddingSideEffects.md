# TCA study
# The Composable Architecture Tutorials

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
