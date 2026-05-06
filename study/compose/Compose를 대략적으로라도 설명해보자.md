# Compose를 대략적으로라도 설명해보자

State는 범위가 광범위 하니 좀 빼놓고 얘기하겠다.

## Compose란?
1. Android 개발을 위해서 Google이 만든 선언형 UI 툴킷.
2. Kotlin으로 작성되며 @Composable 함수를 조합해 UI를 구성.
3. State를 기반으로 UI를 노출하는 방식이며 기존 XML View System에서 가지고 있던 여러 고질적인 문제들을 해결함.
4. 현재는 구글의 권장하는 UI 작성 방식이며, XML View System을 빠르게 대체 중.

---

## 왜 필요했을까?
study/compose/선언형UI.md 문서 참조.

간단히 말하면 기존 XML View System도 잘 동작했으나 앱 UI가 점점 더 복잡해질수록 나타나는 몇가지 문제들이 있었음.

1. UI 요소의 상태와 실제 UI를 업데이트하는 부분으로 나누어져 있는데 -> UI 상태 업데이트나 UI 업데이트 하는곳이 수십곳으로 늘어날 수 있다. 
2. 당연히 코드는 점점 더 읽기 어려워지고 UI 동작이 복잡해질수록 상태 업데이트가 꼬일 가능성이 있다.
3. XML에서는 View간 계층 문제가 있었는데, UI가 복잡해질수록 계층도 더욱 두터워지고 복잡해진다.
4. viewBinding을 하더라도 View자체가 XML, Kotlin/Java로 이중화 되어있기 때문에 UI이 복잡해질 수록 View를 참조하는 보일러코드가 늘어난다

등등의 문제가 있었음.

Compose에서는 UI = f(state)로 해결

---

## 어떻게 작동하나?


### @Composable

@Composable 이 붙은 함수는 UI 자체를 반환하지 않는다. 대신 Composable runtime이 관리하는 UI tree에 노드를 방출(emit)하고,

Composable tree에서 계층으로 관리되고 있는 Composable들을 기반으로 UI를 렌더링 한다.

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "안녕 $name!")
}
```
일반 함수처럼 생겼지만 Compose compiler가 별도로 처리한다.

이런 Composable로 UI들은 Composable tree로 관리되는데, XML의 View계층과 개념은 같다 (코드로 선언되는게 다를뿐)

```kotlin
@Composable
fun App() {
    Box {
        Row {
            Text("이름")
            TextField()
        }
        Column {
            Button("저장") {}
            Text("상태 메세지")
        }
    }
}
```

```kotlin
Box
├── Row
│   ├── Text
│   └── TextField
└── Column
    ├── Button
    └── Text
```


### State - Recomposition

UI가 Recompose되는 유일한 방법은 State가 바뀌는 것이다. 

State는 mutableStateOf로 선언하고, State값이 변경되면 이 State 값을 참조하는 Composable들에게 상태가 변경되었으니 Recompose하라고 신호를 보낸다.
(그냥 일반 변수로 두면 Observe하지 않음).

* mutableStateOf가 반환하는 MutableState 객체는 Observable한 State를 담는 객체임. 결국, Compose가 UI를 다시 렌더링해야겠다고 판단하는 기준은 State<T>이기 때문에 어떤 식으로든 State가 변경되어야함

mutableStateOf가 State변경을 알리는 역할이라면 remember는 State 값을 기억하는 역할이다.

Composable 함수는 State가 바뀔때마다 완전히 새롭게 호출되기 때문에 remmeber 없이 함수 내부에 state를 가지고 있다면 함수가 재 실행될때마다 리셋되게 된다.

```kotlin
// 잘못된 예: 리컴포지션 때마다 count는 다시 0이 됨
@Composable
fun BadCounter() {
    var count = 0 // 일반 변수는 UI를 갱신하지 못함
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}

// 올바른 예: 상태를 기억하고, 변경 시 UI를 갱신함
@Composable
fun GoodCounter() {
    val count = remember { mutableStateOf(0) }
    Button(onClick = { count.value++ }) {
        Text("Count: ${count.value}")
    }
}
```

State가 바뀌고 Recompose가 일어나 Compose가 UI를 새로 렌더링하게 되면, Composable tree안에서는 State의 변경에 영향을 받는 Composable만 다시 그린다.

예를들어 TextField의 입력 상태가 바뀌면 App 전체가 아니라 Row 아랫부분만 recompose된다 (smart recomposition)
```kotlin
Box
├── Row
│   ├── Text // Text State 참조중이므로 영향!
│   ├── TextField // Text 변경! 영향!
│   └── Button // Text State 참조 안하므로 영향 x 
└── Column
    ├── Button
    └── Text
```
---

## Compose 장단점

### 장점
| 항목 | 내용 |
|---|---|
| 코드 감소 | XML 없이 Kotlin만으로 UI를 완성한다. 보일러플레이트가 크게 줄어든다. |
| 상태 일관성 | UI = f(state) 모델로 동기화 버그가 구조적으로 줄어든다. |
| 미리보기 | `@Preview`로 IDE 내에서 실시간 렌더링 확인이 가능하다. |
| 재사용성 | 함수 조합으로 컴포넌트를 작게 나누고 재사용하기 쉽다. |
| 애니메이션 API | `animate*AsState`, `AnimatedVisibility` 등 선언형 애니메이션 API가 내장되어 있다. |
| Kotlin 완전 활용 | 람다, 코루틴, 확장 함수 등 Kotlin의 모든 기능을 UI 코드에서 쓸 수 있다. |

### 단점
| 항목                  | 내용                                                                        |
|---------------------|---------------------------------------------------------------------------|
| 학습 곡선               | 선언형 UI 기본 개념, recomposition, stateHoisting등에 익숙치 않으면 개념 익히고 배우는데 시간 좀 걸린다. |
| 기존 XML기반 코드와 혼용 어려움 | ComposeView, AndroidView로 혼용이 가능하나 상태 관리 면에서는 복잡도 올라감                     |
| Recomposition비용     | remember, derivedStateOf등을 적절히 사용하지 않으면 불필요한 recomsition 비용 발생            |

---

## 비교 — View 시스템과 Compose

| 항목 | View 시스템 (XML + Kotlin) | Jetpack Compose |
|---|---|---|
| UI 정의 방식 | XML + 코드 분리 | Kotlin 함수 단일 파일 |
| UI 업데이트 | 뷰 참조 후 setter 직접 호출 | 상태 변경 → 자동 recomposition |
| 레이아웃 중첩 | 깊어질수록 측정 비용 증가 | 단일 패스 측정으로 설계됨 |
| 상태 관리 | 개발자가 동기화 직접 관리 | 상태가 UI를 구동 |
| 애니메이션 | `Animator`, `TransitionManager` | 선언형 animate API |
| 테스트 | Espresso, Robolectric | Compose Testing API (시맨틱 트리 기반) |
| 기존 생태계 | 매우 풍부 | 빠르게 성장 중 |

View 시스템과 Compose는 `ComposeView` / `AndroidView`를 통해 한 화면에서 공존할 수 있다. 기존 앱을 점진적으로 이전(migration)하는 것이 가능하다.
