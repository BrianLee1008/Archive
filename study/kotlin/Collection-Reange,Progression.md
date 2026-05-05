# Collection - Range, Progression


- 공통점: 정수와 문자 범위를 수학적 등차수열 표현
- 차이점: 
  - Range는 말 그대로 시작과 끝이 정해져 있는 범위를 의미. 
  - Progression은 시작과 끝, 그리고 간격(step)을 가짐. 

| 구분 | Range (범위) | Progression (수열) |
| :--- | :--- | :--- |
| **정의** | 두 지점 사이의 구간 | 일정한 간격을 가진 값들의 나열 |
| **간격 (step)** | 항상 **1**로 고정 | **사용자 지정** 가능 (2, 3, -1 등) |
| **방향** | 항상 증가하는 방향만 존재 | 증가(`..`)와 감소(`downTo`) 모두 가능 |
| **주 목적** | 구간 포함 여부 (`in`) 확인 | 반복문(`for`) 등에서의 순회 |
| **상속 구조** | `IntRange`는 `IntProgression`을 상속함 | 최상위 개념 (수열 그 자체) |

```kotlin
Range

1..10           // IntRange: 1 이상 10 이하
1..<10          // IntRange: 1 이상 10 미만
'a'..'z'        // CharRange: 문자 범위
```

```kotlin
Prograssion

// first: 가장 첫번째 요소를 반환
// last: 가장 마지막 요소를 반환
// 0이 아닌 Step: 특정 범위와 간격을 지정 해서 반환

10 downTo 1     // IntProgression: 10 부터 1까지 Step -1
1..10 step 2    // IntProgression: 1, 3, 5, 7, 9

val range = 1..9 step 3

range.toList() // [1, 4, 7]
range.last() // 7 -> step 3로 반환된 요소의 마지막
```
