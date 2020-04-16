# SwiftUI TextField 글자 수 제한

```swift
class NicknameWithLimit: ObservableObject {
    @Published var text = "" {
        didSet {
            if text.count > characterLimit && oldValue.count <= characterLimit {
                text = oldValue
            }
        }
    }

    let characterLimit: Int

    init(limit: Int = 8) {
        characterLimit = limit
    }
}
```

```swift
@ObservedObject var nickname = NicknameWithLimit()

TextField("", text: $nickname.text)
```

## ObservableObject

클래스에 `ObservableObject` 프로토콜을 상속받으면 SwiftUI와 연동이 가능하다. `@Published` 어노테이션이 붙은 변수의 값이 변경될 때 마다 `View`에게 알려줘 다시 렌더링되게 해준다.