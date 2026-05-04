# Collection-Sequence

## - 1 What: 이게 무엇인가? 핵심적이고 근본적인 정의
- Collection
  - 데이터 집합
- Kotlin Collection
  - mutable, immutable로 명시적인 분리가 가능한 데이터 집합
- Kotlin Sequence
  - 연산이 있을때 중간에 새로운 Collection을 만드는게 아니라 연산을 지연 실행하는 Collection

## - 5 Why's: 여러 관점의 질문들

### 1. 왜 필요했나? 탄생 배경 및 문제 의식
Collection
```agsl
  - 근본적인 이유는 여러 데이터를 하나의 단위로 다루고 싶다는 필요성(학생 명단, 주문 목록, 검색결과 등등)
  - List, Set, Map 자료구조는 서로 다르지만 원소 순회, 크기 반환, 필터링과 같은 동작은 비슷하기 때문에 추상화에 대한 필요성
```

Sequence
```agsl
  - Collection은 기본적으로 eager evaluation임. 모든 원소가 메모리에 이미 위치한 상태로 연산을 하는 즉시 새로운 Collection을 반환함
  - 하나의 Collection에서 여러 연산이 체이닝되면 그만큼 원소에 접근해서 연산을 수행하는 작업도 원소 만큼 늘어날 것이고, 매번 Collection을 새로 만들게 됨
  - 이런 불필요한 연산을 매번 하지 않고 지연해서 한번에 하기 위한 필요성
  - 연산때마다 새로운 Collection을 만들지 않기 위한 필요성
```


### 2. 어떻게 작동하나? 내부 동작 매커니즘

Collection은 각 연산이 즉시(eagerly) 실행되고, 각 단계마다 중간 컬렉션이 생성된다:

```kotlin
listOf(1, 2, 3, 4, 5)
  .filter { it > 2 }        // [3, 4, 5] 생성
  .map { it * 2 }           // [6, 8, 10] 생성
  .take(2)                  // [6, 8] 반환
```

반면 Sequence는 연산이 지연되고, 터미널 연산(최종 결과를 반환하는 연산)이 호출될 때만 처리가 시작된다:

```kotlin
listOf(1, 2, 3, 4, 5).asSequence()
  .filter { it > 2 }
  .map { it * 2 }
  .take(2)                  // 여기서 처리 시작: 1은 건너뛰고, 2는 건너뛰고, 
                            // 3을 처리(filter → map) → 1개 수집
                            // 4를 처리(filter → map) → 2개 수집 → 종료
```


### 3. 트레이드오프가 뭔가? 장단점 고려

- Collection(Eager)의 장단점
```
장점:
- 모든 요소가 메모리에 상주하므로 임의 접근 가능
- `sorted()`, `distinct()` 등 상태 유지 연산이 효율적
- 여러 번 순회 가능
- 작은 데이터셋에는 오버헤드가 적다

단점:
- 각 연산 단계마다 중간 컬렉션 생성
- 조기 종료 조건(`take()`, `first()`)이 있어도 전체 처리
- 메모리 사용량이 많다 (특히 대용량 데이터)
```
- Sequence의 장단점
```
장점:
- 중간 컬렉션 미생성으로 메모리 효율적
- 조기 종료 조건(short-circuit)이 있으면 조기 종료로 성능 향상
- 무한 시퀀스 지원 (예: `generateSequence(1) { it + 1 }`로 무한 자연수 시퀀스 생성)
- 연산 체인에서 Lazy evaluation의 이점 (다단계 필터링/매핑)

단점:
- 1회 소비 후 재사용 불가 (생성 후 한 번만 순회)
- 상태 유지 연산에서는 오버헤드 커짐
- 작은 데이터에는 Lazy 오버헤드가 이득보다 크다
```


### 4. 무엇과 다른가? 유사 개념과 비교, 구분
Kotlin Sequence vs Java Stream

| 항목 | Kotlin Sequence | Java Stream |
|---|---|---|
| 평가 방식 | Lazy | Lazy |
| 병렬 처리 | 미지원 (sequential만) | `parallelStream()`으로 지원 |
| 타입 안전성 | 완전한 Kotlin 타입 | 플랫폼 타입 포함 가능 |
| Autoboxing | 일부 발생 | `IntStream` 등 원시 타입 특화로 회피 가능 |
| 재사용 | 1회 소비 후 재사용 불가 | 1회 소비 후 재사용 불가 |
| API 풍부도 | Extension function 기반, 더 풍부 | 상대적으로 제한적 |

Kotlin Collection vs Java Collection
```
Java는 모든 컬렉션이 mutable이고 쓰기 연산에 최적화되어 있다. Kotlin은 기본이 read-only 인터페이스이므로 불변성을 강조하고, 공변성을 활용해 타입 안전성을 높였다.
또한 Kotlin은 컬렉션 위에 Sequence 레이어를 얹어 lazy evaluation이 필요한 경우를 분리하고, Java Stream과 유사하게 1회 소비만 가능하지만 더 간결한 API를 제공한다 [(비교 자료)](https://proandroiddev.com/java-streams-vs-kotlin-sequences-c9ae080abfdc).
```

### 5. 언제 써야하나? 사용 시점 판단 기준

Collection(eager) 사용 시점
```
- **소규모 데이터**: 수십~수백 개. Lazy 오버헤드보다 eager 실행이 빠르다.
- **상태 유지 연산**: `sorted()`, `distinct()`, `groupBy()` 등. 전체 데이터를 먼저 수집해야 한다.
- **다중 순회**: 같은 컬렉션을 여러 번 순회한다.
- **단일 연산**: map, filter 등 한 번만 처리한다.
```
Sequence 사용 시점
```
- **다단계 연산**: filter → map → take 등 3단계 이상 체인이 있고, 각 단계가 중간 컬렉션을 생성할 때.
- **조기 종료 조건**: `take()`, `first()`, `any()`, `find()` 등이 포함되어 조기 종료가 가능할 때.
- **대용량 데이터**: 수만 개 이상. 메모리 효율이 중요할 때.
- **무한 시퀀스**: `generateSequence()` 등으로 무한 데이터를 다룰 때.
```
성능 판단 기준
```
공식 문서는 "대·소 데이터 모두에 맹목적으로 `asSequence()`를 적용하지 말라"고 권고한다. 정확한 판단을 위해서는 실제 데이터와 환경으로 직접 벤치마크하는 것이 최선이다 [(성능 가이드)](https://kt.academy/article/ek-sequence).

일반적으로:
- **Sequence가 2배 이상 빠른 경우**: 다단계 필터링 + 단락 조건, 대용량 데이터
- **성능 차이 미미한 경우**: 소규모 데이터, 단일 연산
- **Collection이 빠른 경우**: `sorted()` 같은 상태 유지 연산

실제 프로젝트에서는 가독성 우선으로 eager를 기본으로 두고, 성능 문제가 발생하면 해당 부분만 `asSequence()`로 전환하는 방식을 추천한다.
```
Collection vs Sequence 성능 비교 — 중단 조건의 효과

```kotlin
val numbers = (1..10000).toList()

println("=== Collection (Eager) ===")
var collectionOps = 0
val cResult = numbers
    .filter { collectionOps++; it > 5000 }
    .map { collectionOps++; it * 2 }
    .take(3)
println("Operations: $collectionOps")  // 20,000

println("=== Sequence (Lazy) ===")
var sequenceOps = 0
val sResult = numbers.asSequence()
    .filter { sequenceOps++; it > 5000 }
    .map { sequenceOps++; it * 2 }
    .take(3)
    .toList()
println("Operations: $sequenceOps")  // ~9
```

Collection은 `filter`가 10,000개를 모두 처리하고 → `map`이 필터링된 5,000개를 모두 처리한 뒤 → `take(3)`이 첫 3개를 취한다(총 20,000 연산). 반면 Sequence는 역순으로 계산해서 `take(3)`을 만족하는 데 필요한 요소만 처리하므로 약 2000배 적은 연산만 수행한다. 이것이 "조기 종료 조건이 있을 때 Sequence가 유리"한 이유다.


## - 개인적인 의견 (optional)
- 


