## pointfree 링크

[Announcing SwitchStore for the Composable Architecture](https://www.pointfree.co/blog/posts/59-announcing-switchstore-for-the-composable-architecture)

### Why

- enum state에 모델링된 동작들을 Enum의 각 case에 대한 동작으로 세분화 하기 위해서 SwitchStore를 출시

### When

- 상태에 따라 화면의 View, Reducer의 구성을 다르게 해줄때 사용합니다
- 예를들어 로그인을 했을때 안했을때에서 화면이 다르게 구성이 될때

### How

1. enum 형태의 state
    
    ```swift
    enum AppState {
      case loggedIn(LoggedInState)
      case loggedOut(LoggedOutState)
    }
    ```
    <br>

1. body에 Reducer를 추가 
    
    IfCaseLet을 이용해 Reducer를 생성하거나 아래처럼 Scope를 생성해도 된다
    
    → IfCaseLet을 쓰면 state가 없는경우 reducer가 생성 안되는줄 알았는데 그건 아니었음.. 
    
    ```swift
    Reduce { state, action in
          switch action {
          case .login:
            return .none
          case .newGame:
            return .none
          }
        }
        .ifCaseLet(/State.login, action: /Action.login) {
          Login()
        }
        .ifCaseLet(/State.newGame, action: /Action.newGame) {
          NewGame()
        }
    ```
    
    ```swift
    Scope(state: /State.main, action: /Action.main) {
        GroupMainReducer()
     }
    ```
    <br>

1. View단에는 SwitchStore와 CaseLet을 이용하여 view를 생성
    
    ```swift
    SwitchStore(self.store) {
          CaseLet(state: /TicTacToe.State.login, action: TicTacToe.Action.login) { store in
            NavigationView {
              LoginView(store: store)
            }
            .navigationViewStyle(.stack)
          }
          CaseLet(state: /TicTacToe.State.newGame, action: TicTacToe.Action.newGame) { store in
            NavigationView {
              NewGameView(store: store)
            }
            .navigationViewStyle(.stack)
          }
        }
    
    ```
    

→ SwitchStore의 내부구현을 살펴보면 상위 Reducer의 store를 ObservableObject class로 감싸서

enviromentObject로 만들어줍니다 (다른 뷰에서도 해당 객체를 공유 가능)

`CaseLet`내에서는 `IFLetStore`를 이용하여 store로 scope를 생성하게 된다

```swift
self.content
      .environmentObject(StoreObservableObject(store: self.store))
```

```swift
IfLetStore(
      self.store.wrappedValue.scope(
        state: self.toCaseState,
        action: self.fromCaseAction
      ),
      then: self.content
    )
```
<br>

1. 상위 리듀서에서 원하는 코드에 맞는 case state를 선택하여 내부 state를 생성해준다
    a. action을 통해 state를 생성할수도 있고
    
    ```swift
    case .login(.twoFactor(.twoFactorResponse(.success))):
            state = .newGame(NewGame.State())
            return .none
    ```
    
    b.  init에서 설정할수도 있고
    
    ```swift
    public init(roleType: RoleType) {
          switch roleType {
          case .leader:
            self = .leader(LeaderMenuListReducer.State())
          case .worker:
            self = .worker(WorkerMenuListReducer.State())
          }
        }
    ```
    
    c.  static 변수로 초기 initial state를 설정하는 방법도 있다 (TCACoordinator)
    
    ```swift
    static let initialState = Self(routeIDs: [.root(.step1, embedInNavigationView: true)])
    ```
    
<br>

### Reference

- TCA tictactoe example
- TCA Coordinator
