# TCA Study

### PropertyWrapper와 $ 의미

→ [https://ios-development.tistory.com/895](https://ios-development.tistory.com/895)

- PropertyWrapper 종류 (= SwiftUI에서 주로 사용되는)
    ```swift
    @State, @Binding, @Published, @StateObject, @ObservedObject, @EnvironmnetObject... 
    ```
    
- 사용 이유는?
    
    ⇒ ****보일러플레이트 코드 최소화 하자.****    
    -  [https://zeddios.tistory.com/1221](https://zeddios.tistory.com/1221)
    -  [https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/#Property-Wrappers](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/#Property-Wrappers)
    

**- wrappedValue**
```swift
@propertyWrapper
struct TwelveOrLess {
    private var number = 0
    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, 12) }
    }
}

    @TwelveOrLess var height: Int
    @TwelveOrLess var width: Int
```
- 초기 값 또는 값을 입력받아, “전처리” → wrappedValue
    - Ex) → @UpperCase var upperWord: String = value
        - value → @propertywrapper UpperCase {  .. } →  wrapedValue

**- ProjectedValue**
```swift
@frozen @propertyWrapper public struct State<Value> : DynamicProperty {
  ...
  public var wrappedValue: Value { get nonmutating set }
  public var projectedValue: Binding<Value> { get }
 }
 
struct PlayerView: View {
    var episode: Episode
    @State private var isPlaying: Bool = false
    var body: some View {
        VStack {
            Text(episode.title)
                .foregroundStyle(isPlaying ? .primary : .secondary)
            PlayButton(isPlaying: $isPlaying)
             }
         }
     } 
```
- propertyWrapper 내부의 projectedValue를 정의
- 선언된 변수 앞에 `$` 접두사로 접근하면 propertyWrapper 내부에서 정의한 값 반환
    - 접근 방식 “$” 키워드로 접근
        - @State → @Binding을 통해, SubView로 특정 property 전달 할
        - Ex) @State var isTapped: Bool = false
        - @propertyWrapper State<Value> { public var projectedValue: Binding<Value> { get } }
        - Alert( isPresent: Binding<bool>.. ) → Alert(isPresented: $isTapped)
