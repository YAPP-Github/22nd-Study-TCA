# TCA 2주차

## ReducerProtocol

- ### reduce
  - 주어진 action에 따라 feature state를 변경 및 관리하는 메서드
  - 비동기적으로 실행되는 Effect return 및 데이터 처리
  ```swift
  func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
    switch action {
    case .decrementButtonTapped:
      state.count -= 1
      return .none

    case .incrementButtonTapped:
      state.count += 1
      return .none
    }
  }
  ```
  
- ### 리듀서 정의 방법
  - ReducerProtocol 채택 후, 두 가지 방법으로 구현 가능
  - 두 요구 사항을 모두 구현하는 경우 Store에서 reduce(into:action:)만 호출!
  ``` swift
  func reduce(into state: inout State, action: Action) -> EffectTask<Action> // [1]
  
  var body: some ReducerProtocol<State, Action>  // [2]
  
   // 하나 또는 두 개 이상의 reduce를 결합할 때, 사용한다..?
  var body: some ReducerProtocol<State, Action> {
    Reduce { state, action in
      // extra logic
    }
    Activity()
    Profile()
    Settings()
  }
  
  ```
<br>

## Store

- ### Scoping
  - Store에서 가장 중요한 메서드, Store -> 하위뷰로 전달 시, 전체 도메인에서 하위뷰에 해당하는 State, action으로 범위 한정
  - 주로 RootView와 같이 최상위 View에서 활용도가 높다.
    Ex) TabView에서 full app domain store<AppFeature>에서 sub domain Store로 변형
    ```swift
    // <AppFeature>
      struct State { var activity: Activity.State  ... }
      enum Action {  case activity(Activity.Action) ... }
  
      struct AppView: View {
      let store: StoreOf<AppFeature>

      var body: some View {
       TabView {
      ActivityView(
        store: self.store.scope(state: \.activity, action: AppFeature.Action.activity)
      )
      .tabItem { Text("Activity") }
      ....
      }
    ```


 - ### Thread safety
   Store class는 Thread safe 하지 않다. action이 스토어로 보내지고, reducer가 현재 상태를 처리하는 과정이 multiple
   Store가 Thread safe 하지 않은 이유는 Action이 Store로 보내질 때, 리듀서가 현재 상태에서 실행되기 때문

<br>
  
## ViewStore
- State 변화와 Action을 관찰할 수 있는 object. View[SwiftUI views, UIView or UIViewController]에서 주로 사용된다. 

- ### ViewStore 이슈
  - WithViewStore 사용 시, compile-time 이슈를 겪을 수 있다.
  ``` swift
  //[1] 클로저에 구체적인 타입 명시
  WithViewStore(self.store, observe: { $0 }) { (viewStore: ViewStoreOf<Feature>) in
    // A large, complex view inside here...
  }
  
  //[2] ObservedObject PropertyWrapper 사용
  let store: StoreOf<Feature>
  @ObservedObject var viewStore: ViewStoreOf<Feature>

  init(store: StoreOf<Feature>) {
    self.store = store
    self.viewStore = ViewStore(self.store, observe: { $0 })
  }
  ```
  - withViewStore 사용 시, observe too much state 이슈.
  observe: {$0} 표기 시, 모든 State변화를 감지.
  ```swift
  // 기존 사용, observe: {$0} => 모든 State 변화 감지 => re-compute
  WithViewStore(self.store, observe: { $0 }) { viewStore in 
  // that causes self.store.state to change. 
  }
  
  // 감지가 필요한 특정 State만 감지할 수 있도록 명시.
  WithViewStore(self.store, observe: \.selectedTab) { viewStore in
  TabView(selection: viewStore.binding(send: AppFeature.Action.tabSelected)) {
    // ...
    }
  }
  ```


  



