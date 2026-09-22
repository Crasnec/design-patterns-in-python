# 6장. Higher-Order Functions

![여러 작업 날을 받아 다른 도구가 되는 정원용 손잡이](../../assets/images/fp/higher-order-functions.png)

앞 장에서 함수를 값으로 전달하는 방법을 익혔다.
이번 장은 그 값을 받는 함수의 설계를 다룬다.
목록을 순회하는 규칙과 각 원소에 적용할 업무 규칙을 분리하면 같은 순회 구조를 여러 계산에
재사용할 수 있다.

고차 함수의 핵심은 람다를 인자로 넣는 문법이 아니다.
호출 시점, 호출 횟수, 순서, 실패 전파를 포함한 계약을 설계하는 일이다.
이 장에서는 주문 금액의 변환과 조건 검사를 통해 그 계약을 구체화한다.

---

## 1. 개념과 기본 구분

### 함수를 입력이나 출력으로 사용한다

함수를 인자로 받거나 결과로 반환하는 함수가 고차 함수다.
둘 중 하나만 만족해도 된다.
함수 안에서 다른 함수를 직접 호출한다는 사실만으로 고차 함수라고 부르지는 않는다.

```text
일반 함수:  A -> B
고차 함수:  (A -> B) -> C
고차 함수:  A -> (B -> C)
```

타입에서 괄호는 중요하다.
`A -> (B -> C)`는 `A`를 받은 뒤 함수를 반환한다.
`(A -> B) -> C`는 함수를 받아 결과를 만든다.
반환된 함수를 언제 실행할지는 그 함수를 받은 쪽의 책임이다.

### 순회와 정책

목록의 모든 금액에 세금을 더하는 코드와 할인하는 코드는 순회 구조가 같다.
다른 부분은 원소를 어떻게 바꾸는지다.
고차 함수는 그 변동점을 함수 인자로 분리한다.

```text
공통 구조: 원소를 순서대로 읽고 결과를 모은다
변동 정책: 원소 하나를 어떻게 계산하는가
```

### 함수 인자도 계약을 가진다

함수 타입은 입력과 출력의 모양을 나타낸다.
그러나 호출 횟수, 예외, 순수성, 계산 비용은 따로 설명해야 한다.
동일한 `A => B` 타입이라도 로그를 남기거나 상태를 변경할 수 있다.

### 소비자에서 작성자로

`map`이나 `filter`를 사용하는 것은 고차 함수의 소비자가 되는 일이다.
직접 그런 API를 작성하면 언제 추상화가 유용하고 언제 과한지 이해하기 쉽다.
2부에서는 각 연산의 법칙과 비용을 더 깊게 다룬다.

---

## 2. 명령형 스타일과 함수형 스타일

### 중복된 반복

다음 Scala 조각은 두 계산의 순회가 동일하다는 점을 보여 준다.
업무 규칙만 다르다.

```scala
val amounts = List(1000, 2000, 3000)

val discounted = List.newBuilder[Int]
for amount <- amounts do
  discounted += amount - amount / 10

val labels = List.newBuilder[String]
for amount <- amounts do
  labels += s"$amount KRW"
```

반복을 그대로 복사하면 입력 처리 방식이나 오류 정책이 여러 곳에 흩어진다.
반대로 모든 반복을 하나의 거대한 추상화에 넣으면 이해하기 어려워진다.
공통되는 순회 계약이 실제로 같은지 확인한 뒤 분리한다.

### 변환을 인자로 받기

```scala
def transform[A, B](values: List[A])(f: A => B): List[B] =
  val builder = List.newBuilder[B]
  for value <- values do
    builder += f(value)
  builder.result()
```

이 구현은 원소마다 함수를 한 번 호출하며 입력 순서를 유지한다.
예외가 발생하면 남은 원소는 처리하지 않는다.
빈 목록에는 함수를 호출하지 않고 빈 목록을 반환한다.
이런 문장이 API의 실제 계약이다.

### 지역 변경의 의미

빌더는 함수 내부에서 변경된다.
하지만 외부 입력을 수정하지 않고 빌더를 누출하지 않으면 반환 경계는 새 값의 계산으로 읽을 수
있다.
콜백이 순수할 때 전체 변환도 순수하게 사용할 수 있다.
지역 대입문과 외부 효과를 구분했던 앞 장의 원칙이 여기에도 적용된다.

---

## 3. 왜 이 개념을 사용하는가?

### 반복 구조의 일관성

순회 로직을 한 곳에 두면 빈 입력, 순서, 결과 수집 방식을 일관되게 유지할 수 있다.
호출자는 원소 하나의 업무 규칙에 집중한다.
다만 서로 다른 순회 계약을 무리하게 합치면 숨은 옵션이 늘어난다.

### 작은 정책의 테스트

원소 하나를 처리하는 함수는 컬렉션 전체보다 작은 입력 공간을 가진다.
정책 테스트와 순회 계약 테스트를 분리할 수 있다.
예를 들어 할인 경계는 정책에서, 호출 횟수는 변환 함수에서 검사한다.

### 함수 반환의 재사용

반복 횟수를 정하는 함수를 만들어 나중에 여러 입력에 적용할 수 있다.
이때 설정과 실행을 분리할 수 있다.
클로저는 그런 설정값을 보관하는 일반적인 구현 방식이다.

### 추상화의 이름

`process`처럼 모호한 이름보다 `transform`, `select`, `all`처럼 결과의 관계를 드러내는 이름이
좋다.
이름만으로 모든 계약이 정해지지는 않지만 독자가 기대할 방향을 제공한다.
같은 이름을 사용하면서 호출 순서나 실패 정책을 다르게 만들면 혼란이 생긴다.

### 재시도는 별개다

콜백을 인자로 받는다고 자동으로 재시도해도 되는 것은 아니다.
실패 후 다시 호출하면 외부 효과가 중복될 수 있다.
재시도는 일반적인 `map`의 숨은 구현 세부사항이 아니라 별도의 효과 계약이어야 한다.

---

## 4. Scala에서의 표현

### Scala의 고차 함수 구현

다음 예제는 변환, 선택, 단락 평가, 함수 반환을 함께 보여 준다.
각 연산의 타입과 호출 규칙을 비교하자.
표준 `map`을 다시 구현하는 목적은 제품용 컬렉션 라이브러리를 만드는 것이 아니라 계약을 드러내는
데 있다.

<!-- executable:scala -->
```scala
object Chapter06:
  def transform[A, B](values: List[A])(f: A => B): List[B] =
    val builder = List.newBuilder[B]
    for value <- values do
      builder += f(value)
    builder.result()

  def select[A](values: List[A])(predicate: A => Boolean): List[A] =
    val builder = List.newBuilder[A]
    for value <- values do
      if predicate(value) then builder += value
    builder.result()

  def all[A](values: List[A])(predicate: A => Boolean): Boolean =
    values.forall(predicate)

  def repeated[A](times: Int)(f: A => A): A => A =
    require(times >= 0)
    initial =>
      var current = initial
      for _ <- 0 until times do
        current = f(current)
      current

  def check(): Unit =
    val amounts = List(1000, 2000, 3000)
    val reduced = transform(amounts)(n => n - n / 10)
    assert(reduced == List(900, 1800, 2700))
    assert(amounts == List(1000, 2000, 3000))
    assert(transform(List.empty[Int])(_ + 1) == List.empty[Int])
    assert(select(amounts)(_ >= 2000) == List(2000, 3000))
    assert(all(amounts)(_ > 0))
    assert(all(List.empty[Int])(_ > 0))
    assert(repeated(3)((n: Int) => n + 2)(1) == 7)
    assert(repeated(0)((n: Int) => n + 2)(1) == 1)

    var calls = Vector.empty[Int]
    val mapped = transform(List(1, 2, 3)) { n =>
      calls = calls :+ n
      n * 10
    }
    assert(mapped == List(10, 20, 30))
    assert(calls == Vector(1, 2, 3))

    calls = Vector.empty
    val accepted = all(List(2, 4, 5, 6)) { n =>
      calls = calls :+ n
      n % 2 == 0
    }
    assert(!accepted)
    assert(calls == Vector(2, 4, 5))
```

### 테스트의 관측용 변경

`calls`는 호출 순서를 관측하기 위한 테스트 장치다.
그 콜백은 의도적으로 효과가 있다.
따라서 해당 테스트 실행 전체를 순수 계산이라고 부르지 않는다.
고차 함수가 콜백의 효과를 없애지 않는다는 사실도 동시에 보여 준다.

### 빈 입력의 의미

`all`은 빈 목록에서 참을 반환한다.
위반하는 원소가 없다는 전칭 조건의 의미다.
업무에서 최소 한 개의 품목이 필요하면 비어 있지 않다는 검사를 별도로 추가해야 한다.
논리적 항등값과 업무 유효성은 구분한다.

---

## 5. 상태 변경보다 값 변환

### 컬렉션 변환의 흐름

고차 함수는 입력을 어떤 관계의 출력으로 만드는지 드러낸다.
`transform`은 원소 수를 유지하고 타입을 바꿀 수 있다.
`select`는 원소 타입을 유지하고 일부 원소를 제거할 수 있다.

```mermaid
flowchart LR
    A["금액 목록"] --> B["select: 유효 금액"]
    B --> C["금액 목록"]
    C --> D["transform: 표시 문자열"]
    D --> E["문자열 목록"]
```

이 관계는 함수 이름과 타입을 함께 읽으면 드러난다.
일반적인 반복문은 이런 의도를 본문 전체에서 찾아야 할 수 있다.
그렇다고 모든 반복문보다 고차 함수가 항상 더 읽기 좋다는 뜻은 아니다.

### 결과를 무시하는 실수

불변 변환은 새 결과를 반환한다.
반환값을 버리면 입력이 변경되지 않아 아무 효과도 없는 것처럼 보일 수 있다.
`map`을 출력이나 저장만을 위해 사용하는 코드는 의도를 흐릴 수 있다.
효과를 수행하는 순회와 값을 만드는 변환을 구분한다.

### 한 번만 소비하는 입력

리스트와 반복자는 같은 계약이 아니다.
반복자는 소비된 원소를 다시 제공하지 않을 수 있다.
Python 예제에서는 입력을 한 번만 순회하는 계약을 명시한다.

---

## 6. 함수 합성과 데이터 흐름

### 고차 함수의 조합

변환 함수를 연속해서 호출하면 각 단계의 결과가 다음 입력이 된다.
순수한 함수라면 두 변환을 하나의 합성 함수로 묶는 법칙을 검토할 수 있다.
효과가 있으면 호출 순서와 시점이 달라질 수 있다.

```text
transform(transform(xs, f), g)

transform(xs, g ∘ f)
```

엄격한 목록에서 첫 형태는 모든 `f` 호출을 끝낸 뒤 `g`를 호출한다.
두 번째 형태는 원소마다 `f`와 `g`를 이어 호출한다.
반환값이 같더라도 로그 순서가 달라질 수 있다.
합성 법칙의 전제조건에 순수성이 등장하는 이유다.

### 함수 반환과 설정

`repeated(3)(f)`는 새 함수를 만든다.
그 함수를 실행해야 `f`를 세 번 적용한다.
설정 시점과 실행 시점의 구분은 부분 적용과 커링에서 다시 나타난다.

### 단락 평가의 합성

`all`은 첫 거짓에서 멈춘다.
비싼 검사를 뒤에 두면 호출 횟수를 줄일 수 있지만 효과 순서가 바뀔 수 있다.
조건들의 독립성과 실패 의미를 확인한 뒤 순서를 조정해야 한다.

---

## 7. 장점과 트레이드오프

### 이득

순회 계약을 재사용하면서 업무 규칙을 작은 함수로 나눌 수 있다.
타입을 통해 원소 타입 변화와 결과 형태를 드러낼 수 있다.
호출 횟수와 빈 입력 처리를 한 곳에서 테스트할 수 있다.

### 비용

함수 호출과 중간 컬렉션의 비용이 생길 수 있다.
작은 함수가 지나치게 흩어지면 전체 흐름을 찾기 어려워질 수 있다.
추상화가 실제 실행 순서를 가리면 효과가 있는 코드에서 오해가 생긴다.

| 연산 | 콜백 호출 | 결과 형태 | 주의점 |
| --- | --- | --- | --- |
| 변환 | 원소마다 한 번 | 같은 길이 | 중간 컬렉션 |
| 선택 | 원소마다 한 번 | 길이 감소 가능 | 거부 원인 손실 |
| 전체 조건 | 첫 거짓까지 | 불리언 | 빈 입력은 참 |
| 반복 적용 | 지정 횟수 | 단일 값 | 효과 중복 |
| 지연 변환 | 소비 시점 | 반복자 등 | 실행 시점 이동 |

### 과잉 일반화

아직 한 번만 쓰는 짧은 반복을 무조건 범용 프레임워크로 만들 필요는 없다.
실제로 공통되는 계약을 확인하고 이름 붙일 가치가 있을 때 추출한다.
고차 함수의 장점은 복잡한 이름을 늘리는 것이 아니라 반복되는 구조를 분리하는 데 있다.

### 라이브러리 계약을 확인한다

같은 이름의 함수라도 언어와 컬렉션 종류에 따라 엄격성이나 반환 타입이 다를 수 있다.
Scala의 엄격한 목록과 Python의 `map` 반복자를 동일하게 취급하지 않는다.
2부에서 각 연산을 언어별로 비교한다.

---

## 8. 상태와 부수효과의 경계

### 콜백의 효과는 사라지지 않는다

고차 함수가 내부에서 콜백을 호출하면 그 콜백의 효과도 실행된다.
`map(save)`는 저장 작업을 값 변환 문법으로 감쌌을 뿐 순수하게 만들지 않는다.
실행 시점과 실패 시 남는 부분 효과를 확인해야 한다.

### 부분 성공

네 번째 원소에서 예외가 나면 앞의 세 원소에 대한 저장은 이미 끝났을 수 있다.
결과 목록이 반환되지 않았다고 외부 효과까지 취소되는 것은 아니다.
트랜잭션이나 보상 정책은 고차 함수와 별개의 계층이다.

### 병렬 실행

콜백들을 병렬로 실행하는 고차 함수는 순서, 동시성 한도, 취소 정책을 설명해야 한다.
순차 `map`을 병렬 버전으로 바꾸는 것은 단순한 성능 옵션이 아닐 수 있다.
외부 서비스의 호출 제한과 부하도 계약에 포함된다.

### 자원 수명

지연 반복자가 파일을 캡처하면 소비가 끝날 때까지 파일이 필요할 수 있다.
함수를 반환하거나 반복자를 만드는 시점에 자원을 닫아 버리면 나중에 실패한다.
효과와 지연 평가를 함께 다룰 때는 자원 범위를 명시해야 한다.

---

## 9. Python에서 적용하기

### Python의 직접 구현

아래 함수들은 임의의 iterable을 한 번만 순회한다.
반환값은 새 리스트이므로 입력 반복자와 결과 목록의 수명이 분리된다.
타입 변수는 입력 원소와 출력 원소의 관계를 드러낸다.

<!-- executable:python -->
```python
from collections.abc import Callable, Iterable
from typing import TypeVar

A = TypeVar("A")
B = TypeVar("B")


def transform(values: Iterable[A], function: Callable[[A], B]) -> list[B]:
    result: list[B] = []
    for value in values:
        result.append(function(value))
    return result


def select(values: Iterable[A], predicate: Callable[[A], bool]) -> list[A]:
    result: list[A] = []
    for value in values:
        if predicate(value):
            result.append(value)
    return result


def all_values(values: Iterable[A], predicate: Callable[[A], bool]) -> bool:
    for value in values:
        if not predicate(value):
            return False
    return True


def repeated(times: int, function: Callable[[A], A]) -> Callable[[A], A]:
    if times < 0:
        raise ValueError("negative repetition count")

    def apply(initial: A) -> A:
        current = initial
        for _ in range(times):
            current = function(current)
        return current

    return apply


def test_shapes() -> None:
    values = [1000, 2000, 3000]
    assert transform(values, lambda n: n - n // 10) == [900, 1800, 2700]
    assert select(values, lambda n: n >= 2000) == [2000, 3000]
    assert values == [1000, 2000, 3000]
    assert transform([], str) == []
    assert all_values([], lambda _: False)
    assert repeated(3, lambda n: n + 2)(1) == 7
    assert repeated(0, lambda n: n + 2)(1) == 1


def test_call_order() -> None:
    calls: list[int] = []

    def record(value: int) -> int:
        calls.append(value)
        return value * 10

    assert transform([1, 2, 3], record) == [10, 20, 30]
    assert calls == [1, 2, 3]


def test_short_circuit() -> None:
    calls: list[int] = []

    def even(value: int) -> bool:
        calls.append(value)
        return value % 2 == 0

    assert not all_values([2, 4, 5, 6], even)
    assert calls == [2, 4, 5]


def test_single_pass() -> None:
    iterator = iter([1, 2, 3])
    assert transform(iterator, lambda n: n + 1) == [2, 3, 4]
    assert list(iterator) == []


if __name__ == "__main__":
    test_shapes()
    test_call_order()
    test_short_circuit()
    test_single_pass()
```

### 표준 기능과의 관계

실제 Python 코드에서는 리스트 컴프리헨션이나 `all`이 더 자연스러울 수 있다.
직접 구현은 호출 계약을 학습하기 위한 것이다.
표준 `map`은 지연 반복자를 반환하므로 여기의 엄격한 `transform`과 평가 시점이 다르다.

---

## 10. Python의 표현 한계

### 함수 타입의 제한

표준 타입 힌트는 “정확히 한 번 호출”이나 “효과 없음”을 표현하지 않는다.
콜백 호출 규칙은 문서와 테스트에 남겨야 한다.
타입 검사에 통과해도 호출 횟수에 의존하는 버그가 생길 수 있다.

### 입력의 반복 가능성

`Iterable`은 항상 여러 번 안전하게 순회할 수 있다는 뜻이 아니다.
반복자나 생성기는 소모될 수 있다.
입력을 여러 번 사용하는 API는 재순회 가능성을 요구하거나 명시적으로 물질화해야 한다.

### 타입 변수와 런타임

`A`, `B`는 정적 관계를 설명하며 런타임에 원소를 검사하지 않는다.
콜백이 기대와 다른 값을 반환해도 실행은 계속될 수 있다.
외부 데이터와 동적 플러그인 경계에서는 추가 검증이 필요하다.

### 추상화 비용의 자동 제거

Python이 모든 고차 함수 호출을 인라인하거나 중간 컬렉션을 자동 제거한다고 가정하지 않는다.
가독성을 유지한 구현을 먼저 만들고 실제 병목을 측정한다.
성능만을 이유로 검증되지 않은 복잡한 합성기를 도입하는 것은 피한다.

---

## 11. 핵심 정리

### 핵심 결론

고차 함수는 순회 구조와 원소별 정책을 분리한다.
콜백의 타입뿐 아니라 호출 시점, 횟수, 순서, 실패 정책이 계약이다.
지역 변경을 사용한 구현도 외부 입력을 보존하는 값 변환으로 사용할 수 있다.
콜백의 효과는 고차 함수 안에서도 그대로 효과다.

### 연습 1: 빈 입력

`all`이 빈 입력에서 참인 이유를 설명하고, 최소 한 품목이 필요한 주문 검사를 작성하라.

**해설.** 위반 원소가 없다는 전칭 조건이므로 참이다.
업무 검사는 `비어 있지 않음`과 `모든 품목이 유효함`을 함께 확인해야 한다.
논리 연산의 항등값을 업무 정책과 혼동하지 않는다.

### 연습 2: 호출 순서

`map(f).map(g)`와 `map(g ∘ f)`의 로그 순서를 비교하라.
두 입력 원소에 대해 호출 순서를 직접 써 보라.

**해설.** 엄격한 목록의 첫 형태는 `f1, f2, g1, g2`다.
합친 형태는 `f1, g1, f2, g2`다.
순수 함수의 값 법칙을 효과가 있는 콜백에 그대로 적용할 수 없다.

### 연습 3: 재시도

콜백이 예외를 던지면 자동으로 한 번 더 호출하는 `transform`을 설계했다.
저장 콜백을 전달했을 때 어떤 문제가 생길 수 있는가?

**해설.** 첫 호출이 저장 후 응답 과정에서 실패했다면 재호출이 중복 저장을 만들 수 있다.
재시도는 멱등성, 실패 구분, 횟수 제한을 가진 별도 효과 정책이어야 한다.
일반 변환의 숨은 기능으로 넣지 않는다.

### 연습 4: 한 번만 순회

생성기를 입력으로 받아 길이를 먼저 세고 다시 변환하는 구현의 문제를 설명하라.

**해설.** 길이를 세는 동안 생성기가 소모되어 두 번째 순회가 비어 있을 수 있다.
한 번의 순회로 처리하거나 필요한 경우 목록으로 명시적으로 모은다.
물질화의 메모리 비용도 함께 설명해야 한다.

### 다음 장과 참고 자료

다음 장은 반환된 함수가 설정값을 어떻게 보관하는지 설명한다.
그 환경이 변경 가능한지에 따라 같은 고차 함수도 다른 동작을 보일 수 있다.

[Scala 공식 문서: Write Your Own map Method](https://docs.scala-lang.org/scala3/book/fun-write-map-function.html)
[Python 공식 문서: Built-in Functions](https://docs.python.org/3.14/library/functions.html)
