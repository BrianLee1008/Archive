# 선언형 UI

***본 문서는 AI가 작성한 문서가 아닙니다.***

Compose를 하루 10시간 이상 숨쉬듯이 사용하면서 Compose의 상위 카테고리인 선언현 UI(Declarative UI)도 깜빡깜빡 하는 것 같다.

이참에 정리해두자.

선언형 UI란 
```kotlin
어떤 상태일때 어떤 UI를 보여줄지 코드로 기술하는 패러다임
```

명령형 UI와 가장 크게 다른점은,
어떤 순서로 어떤 뷰를 만들고, 어떤 방법으로 어떤 뷰가 수정되는지 하나하나 명령을 해서 만드는 UI가 아니라,

"A State일때는 B UI를 보여준다. 그렇기 때문에 State가 바뀌면 UI를 새로 그린다." 를 선언적으로 작성한다는 점이다.

---

### XML - 명령형 View 시스템의 한계.

기존 Android Xml View 시스템에서는 앱이 복잡해질수록 구조적인 한계가 있었다.

1. UI 트리를 직접 수정해야한다. 그렇기 때문에 setText(), setVisibillity()와 같은 호출이 쌓이게 된다. 그러한 호출이 쌓이게 되면 순서상의 이슈가 생기게 되기 마련이다.
2. UI상태를 set하는 곳과 실제 UI가 분리되어있어 동기화 실패가 잦게 일어난다. 복잡한 환경에서는 UI update호출이 수십개가 되고, 어디선가는 분명 빼먹는 상황도 있게 된다.

```kotlin
// XML + 명령형 (동기화 버그 가능성)
var userName: String = ""
editText.setOnTextChangedListener { text ->
    userName = text
    updateUI() 
}
```

3. UI가 복잡해질수록 findViewById 혹은 ViewBinding으로 View를 참조해야하는 보일러플레이트가 늘어난다.
4. XML로 레이아웃을 짜고, Kotlin/Java로 동작을 붙히는 이중구조다. View가 복잡해질수록 XML과 Kotlin을 왔다갔다 하는 컨텍스트 스위칭 비용이 늘어난다.
5. ConstraintLayout같은 관계형 레이아웃의 등장으로 View 계층문제가 어느정도 해소되긴 했으나 그래도 View간 계층문제가 심각하다.

### Compose - 선언형 UI
XML기반의 Android View System에서 이러한 문제들이 발생하는것을 Compose에서는 UI = f(state) 공식으로 해결해 

동기화 버그 문제, UI 업데이트 호출 중복 문제, 보일러플레이트 문제, 레이아웃과 동작 코드가 이중으로 있는 문제등을 해결했다.

```kotlin
// Compose (동기화 버그 불가능)
var userName by remember { mutableStateOf("") }
TextField(
    value = userName,
    onValueChange = { userName = it }  // State 변경 → 자동으로 UI 갱신
)
Text("안녕, $userName!")  // userName State가 바뀌면 미리 선언 했던 Text가 항상 화면에 반영됨
```