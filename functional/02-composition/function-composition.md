# 9장. Function Composition

1부에서 함수가 값이며 작은 계산의 계약을 분리할 수 있다는 점을 배웠다.
2부의 첫 장은 그 작은 계산들을 연결한다.
함수 합성은 앞 계산의 출력을 뒤 계산의 입력으로 넘겨 하나의 새 함수를 만드는 연산이다.

주문 금액에 할인과 배송비를 적용하고 표시 문자열을 만드는 사례를 사용한다.
각 단계가 단순하더라도 순서가 바뀌면 업무 의미가 달라질 수 있다.
합성의 타입, 읽는 방향, 법칙, 오류 경계를 함께 이해하는 것이 목표다.

---

## 1. 개념과 기본 구분

### 합성의 정의

`f: A -> B`와 `g: B -> C`가 있으면 `g ∘ f: A -> C`를 만들 수 있다.
합성된 함수는 입력 `a`에 대해 `g(f(a))`를 계산한다.
중간 타입 `B`가 두 단계의 연결 지점이다.

```text
A ──f──> B ──g──> C

(g ∘ f)(a) = g(f(a))
```

수학 표기에서는 오른쪽 함수부터 적용한다.
코드를 왼쪽에서 오른쪽으로 읽는 파이프라인 표기와 방향이 다를 수 있다.
표기법을 외우기보다 실제로 어느 함수가 먼저 호출되는지 풀어 써 보자.

### 합성과 실행

함수를 합성하는 일은 새 함수를 만드는 일이다.
그 새 함수에 입력을 전달해야 실제 계산이 실행된다.
합성 시점에 외부 작업을 수행하는 특수한 API가 아니라면 두 시점을 구분할 수 있다.

### 타입이 연결된다는 뜻

앞 함수가 문자열을 반환하고 뒤 함수가 정수를 요구하면 그대로 연결할 수 없다.
변환 함수를 추가하거나 잘못된 설계를 수정해야 한다.
타입이 맞더라도 단위와 도메인 의미가 맞는지는 별도 문제다.

예를 들어 두 단계가 모두 정수여도 하나는 원화이고 다른 하나는 초 단위일 수 있다.
함수 합성은 의미상 맞지 않는 단위를 자동으로 구분하지 않는다.
명목적인 도메인 타입은 이런 오류를 줄이는 데 도움이 된다.

---

## 2. 명령형 스타일과 함수형 스타일

### 중간 변수를 갱신하는 코드

```scala
var payable = 10000L
payable = payable - payable / 10
payable = payable + 3000
val label = s"$payable KRW"
```

이 코드는 실행 순서를 읽으면 이해할 수 있다.
문제는 같은 규칙의 조합이 여러 곳에서 반복될 때다.
각 위치가 할인과 배송비의 순서를 다르게 구현할 수 있다.

### 단계를 이름 붙인다

```scala
def discount(amount: Long): Long = amount - amount / 10
def shipping(amount: Long): Long = amount + 3000
def render(amount: Long): String = s"$amount KRW"

val label = render(shipping(discount(10000)))
```

각 규칙을 따로 테스트할 수 있다.
하지만 중첩 호출은 단계가 많아지면 안쪽부터 읽어야 한다.
합성 함수에 이름을 붙이면 조합 자체도 재사용 가능한 규칙이 된다.

```scala
val quoteLabel: Long => String =
  (discount: Long => Long).andThen(shipping).andThen(render)
```

이 조각은 Scala의 메서드를 함수값으로 사용하는 의도를 드러낸다.
아래 완전한 예제에서는 명시적인 함수 타입을 사용하여 더 단순하게 작성한다.

### 순서가 업무 정책이다

배송비를 더한 뒤 할인하면 배송비도 할인 대상이 된다.
할인한 뒤 배송비를 더하면 상품 금액만 할인한다.
두 코드가 모두 타입 검사에 통과해도 같은 정책은 아니다.

---

## 3. 왜 이 개념을 사용하는가?

### 조합을 이름 붙이는 이유

작은 함수만 분리하면 호출자마다 조합 순서를 다시 결정해야 한다.
조합 자체에 이름을 붙이면 업무 흐름을 한 곳에서 정의할 수 있다.
예를 들어 `quoteWithShipping`은 할인과 배송비의 순서를 계약으로 가진다.

### 단계별 테스트와 전체 테스트

할인 함수의 경계값은 개별 테스트로 확인한다.
배송비의 적용 순서는 합성된 함수의 테스트로 확인한다.
둘을 모두 두어야 작은 함수들이 맞는데 전체 정책이 틀린 경우를 잡을 수 있다.

### 중간값의 의미

함수 합성은 중간값을 숨기는 것만이 목적이 아니다.
필요하면 중간 결과에 이름과 타입을 부여하여 경계를 더 선명하게 만든다.
매우 긴 한 줄보다 의미 있는 단계가 보이는 코드가 낫다.

### 변경의 범위

표시 형식이 바뀌면 마지막 함수만 교체할 수 있다.
할인 규칙이 바뀌면 해당 정책 함수를 교체할 수 있다.
그러나 정책 간 의존성이 생기면 단순한 교체로 충분하지 않을 수 있다.
합성 구조는 실제 데이터 의존성을 반영해야 한다.

### 추상화의 최소 단위

한 번만 쓰는 짧은 계산에 범용 합성 라이브러리를 도입할 필요는 없다.
언어의 기본 함수와 명시적 호출만으로 충분할 수 있다.
반복되는 조합과 테스트 경계가 생길 때 합성 함수를 도입한다.

---

## 4. Scala에서의 표현

### Scala의 `andThen`과 `compose`

아래 프로그램은 두 방향을 같은 입력으로 확인한다.
함수 타입을 명시하여 메서드와 함수값의 변환에 대한 혼동을 줄였다.
금액 예제는 작은 정수 범위에서 실행한다.

<!-- executable:scala -->
```scala
object Chapter09:
  def compose[A, B, C](g: B => C, f: A => B): A => C =
    value => g(f(value))

  def andThen[A, B, C](f: A => B, g: B => C): A => C =
    value => g(f(value))

  def identity[A](value: A): A = value

  val discount: Long => Long = amount => amount - amount / 10
  val shipping: Long => Long = amount => amount + 3000
  val render: Long => String = amount => s"$amount KRW"

  val quoteLabel: Long => String =
    discount.andThen(shipping).andThen(render)

  def check(): Unit =
    assert(quoteLabel(10000) == "12000 KRW")
    assert(discount.andThen(shipping)(10000) == 12000)
    assert(shipping.andThen(discount)(10000) == 11700)
    assert(shipping.compose(discount)(10000) == 12000)
    assert(discount.compose(shipping)(10000) == 11700)
    assert(compose(shipping, discount)(10000) == 12000)
    assert(andThen(discount, shipping)(10000) == 12000)

    val f: Int => Int = value => value + 1
    val g: Int => Int = value => value * 2
    val h: Int => String = value => s"value=$value"

    for value <- -20 to 20 do
      val left = f.andThen(g).andThen(h)(value)
      val right = f.andThen(g.andThen(h))(value)
      assert(left == right)
      assert(f.andThen(identity[Int])(value) == f(value))
      assert((identity[Int]: Int => Int).andThen(f)(value) == f(value))

    var events = Vector.empty[String]
    val first: Int => Int = value =>
      events = events :+ "first"
      value + 1
    val second: Int => Int = value =>
      events = events :+ "second"
      value * 2
    val combined = first.andThen(second)
    assert(events.isEmpty)
    assert(combined(3) == 8)
    assert(events == Vector("first", "second"))
```

### 결합법칙과 교환법칙을 구분한다

함수 합성의 결합법칙은 괄호를 바꾸어도 함수 적용 순서가 같다는 뜻이다.
`f` 다음 `g` 다음 `h`라는 순서는 두 식에서 유지된다.
교환법칙은 순서 자체를 바꾸는 주장인데 일반적인 함수 합성에는 성립하지 않는다.

### 효과가 있는 경우의 정밀한 해석

단순한 함수 합성 래퍼의 괄호만 바꾸면 효과가 있어도 호출 순서가 그대로일 수 있다.
따라서 “효과가 있으면 합성의 결합법칙이 무조건 깨진다”라고 말하지 않는다.
반면 함수 순서를 바꾸거나 호출을 제거·복제하는 변환은 효과에 영향을 줄 수 있다.
어떤 변환을 주장하는지 정확히 구분해야 한다.

---

## 5. 상태 변경보다 값 변환

### 데이터 흐름으로 읽기

각 단계의 결과가 다음 단계의 입력이다.
공유 변수의 현재 값을 추적하는 대신 값이 지나가는 경로를 추적한다.
중간값이 불변이면 이전 단계의 결과를 나중 단계가 바꾸지 않는 계약을 만들기 쉽다.

```mermaid
flowchart LR
    A["상품 금액"] --> B["할인 적용"]
    B --> C["할인 후 금액"]
    C --> D["배송비 추가"]
    D --> E["최종 금액"]
    E --> F["표시 문자열"]
```

이 그림은 저장이나 결제를 수행하지 않는다.
최종 금액을 계산하는 흐름과 그 금액으로 결제하는 효과는 다른 단계다.
도메인 모델에서 계산 결과와 실행 완료 상태를 구분해야 한다.

### 중간 타입을 강화한다

원금, 할인 후 금액, 청구 금액을 모두 같은 정수로 두면 순서 오류를 타입으로 잡기 어렵다.
필요하면 각 단계의 결과를 다른 레코드나 명목 타입으로 표현할 수 있다.
다만 타입을 지나치게 세분화하면 변환 코드가 늘어나므로 실제 오류 위험을 기준으로 선택한다.

### 갱신을 합성하는 경우

`Order => Order` 형태의 함수도 합성할 수 있다.
이때 앞 장의 불변 갱신과 마찬가지로 적용 순서가 중요하다.
같은 필드를 덮어쓰는 두 갱신은 보통 교환 가능하지 않다.

---

## 6. 함수 합성과 데이터 흐름

### 항등 함수

아무 변화 없이 입력을 반환하는 함수가 항등 함수다.
합성에서 빈 단계의 역할을 한다.
설정에 따라 선택적 변환을 넣을 때 항등 함수를 사용할 수 있다.

```text
id(x) = x

id ∘ f = f
f ∘ id = f
```

항등 함수가 불필요한 분기를 줄여 주는 경우도 있다.
하지만 이름 없는 항등 함수가 여러 층에 숨어 있으면 오히려 흐름이 복잡해질 수 있다.
구조를 단순하게 만드는 범위에서 사용한다.

### 결합법칙의 풀이

`h(g(f(x)))`는 괄호를 어느 합성 연산에 먼저 붙이든 같은 적용 순서를 가진다.
함수 합성의 결합법칙은 이 전개에서 이해할 수 있다.
법칙을 외우기보다 입력 하나에 대한 식으로 풀어 보자.

### 실패 가능성이 있는 단계

`A -> Option[B]` 다음에 `B -> Option[C]`를 일반 합성으로 직접 연결할 수는 없다.
중간값은 `B`가 아니라 `Option[B]`이기 때문이다.
부재를 처리하며 이어 주는 `flatMap`과 Kleisli 합성의 필요성이 여기서 나온다.
이 장에서는 타입의 불일치를 인식하는 것으로 충분하다.

---

## 7. 장점과 트레이드오프

### 장점과 트레이드오프

| 선택 | 장점 | 주의할 점 |
| --- | --- | --- |
| 작은 함수 분리 | 규칙별 테스트 | 과도한 탐색 비용 |
| 조합 함수 명명 | 업무 순서 고정 | 이름과 실제 순서의 일치 |
| 중간 타입 구분 | 잘못된 연결 감소 | 래핑과 변환 코드 |
| 항등 함수 사용 | 선택적 단계 단순화 | 불필요한 추상화 |
| 긴 파이프라인 | 전체 흐름 표현 | 중간 실패와 디버깅 |

### 비용 모델

함수 합성은 호출 래퍼를 만들 수 있다.
언어 구현이 일부 호출을 인라인할 수도 있지만 항상 제거된다고 보장할 수 없다.
중간 객체를 만드는 함수들은 합성 후에도 그 할당 비용을 가질 수 있다.

### 가독성의 경계

함수 이름만 읽어도 업무 흐름을 알 수 있으면 합성이 도움이 된다.
각 함수를 열어 봐야 타입과 단위를 알 수 있다면 명시적인 중간 변수가 더 낫기도 하다.
합성의 목표는 코드 길이보다 데이터 의존성을 선명하게 만드는 것이다.

### 디버깅

중간값을 관측하고 싶으면 명시적 단계로 풀어 쓸 수 있다.
관측용 로그를 넣으면 효과가 추가된다는 점을 인식한다.
테스트에서는 순수한 중간값을 직접 비교하는 방식이 더 간단할 수 있다.

---

## 8. 상태와 부수효과의 경계

### 계산 파이프라인과 효과 파이프라인

할인과 배송비 계산은 값 변환으로 설계할 수 있다.
가격 조회, 저장, 결제는 외부 효과를 수행한다.
모든 단계를 같은 함수 타입으로 감싸도 실패와 재시도 계약은 같아지지 않는다.

```text
조회 -> 계산 -> 저장 -> 결제
```

이 흐름에서 저장 후 결제가 실패하면 어떻게 할지 별도 정책이 필요하다.
일반 함수 합성은 트랜잭션이나 보상을 제공하지 않는다.
효과를 값으로 표현하고 해석하는 방법은 후반부에서 다룬다.

### 예외의 전파

앞 함수가 예외를 던지면 뒤 함수는 실행되지 않을 수 있다.
이것은 단순한 합성의 실행 규칙이며 모든 오류를 명시적으로 모델링했다는 뜻은 아니다.
업무상 예상되는 실패는 결과 타입에 드러내는 편이 호출자에게 더 많은 정보를 준다.

### 재시도 범위

전체 합성을 재시도하면 앞에서 성공한 효과도 다시 실행될 수 있다.
순수 계산만 재실행하는 것과 결제까지 재실행하는 것은 다르다.
재시도는 단계별 멱등성과 실패 구분을 바탕으로 설계한다.

### 지연과 자원

합성된 함수가 닫힌 자원을 캡처하면 나중 실행 시 실패할 수 있다.
합성 가능성과 자원 수명 안전성은 다른 성질이다.
IO와 자원 관리 장에서 이 경계를 더 명시적으로 다룬다.

---

## 9. Python에서 적용하기

### Python에서 타입을 보존하는 두 함수 합성

Python에는 모든 경우를 위한 표준 `compose` 함수가 없지만 작은 함수를 직접 작성할 수 있다.
아래 예제는 세 타입 변수를 사용하여 중간 타입의 관계를 나타낸다.
가변 길이의 임의 합성기를 무리하게 일반화하기보다 두 단계의 계약을 먼저 이해한다.

<!-- executable:python -->
```python
from collections.abc import Callable
from typing import TypeVar

A = TypeVar("A")
B = TypeVar("B")
C = TypeVar("C")


def compose(g: Callable[[B], C], f: Callable[[A], B]) -> Callable[[A], C]:
    def combined(value: A) -> C:
        return g(f(value))
    return combined


def and_then(f: Callable[[A], B], g: Callable[[B], C]) -> Callable[[A], C]:
    return compose(g, f)


def identity(value: A) -> A:
    return value


def discount(amount: int) -> int:
    return amount - amount // 10


def shipping(amount: int) -> int:
    return amount + 3000


def render(amount: int) -> str:
    return f"{amount} KRW"


def test_direction() -> None:
    quote_label = and_then(and_then(discount, shipping), render)
    assert quote_label(10_000) == "12000 KRW"
    assert compose(shipping, discount)(10_000) == 12_000
    assert compose(discount, shipping)(10_000) == 11_700


def test_laws() -> None:
    f = lambda value: value + 1
    g = lambda value: value * 2
    h = lambda value: f"value={value}"
    left = and_then(and_then(f, g), h)
    right = and_then(f, and_then(g, h))
    for value in range(-20, 21):
        assert left(value) == right(value)
        assert and_then(identity, f)(value) == f(value)
        assert and_then(f, identity)(value) == f(value)


def test_creation_and_execution() -> None:
    events: list[str] = []

    def first(value: int) -> int:
        events.append("first")
        return value + 1

    def second(value: int) -> int:
        events.append("second")
        return value * 2

    combined = and_then(first, second)
    assert events == []
    assert combined(3) == 8
    assert events == ["first", "second"]


if __name__ == "__main__":
    test_direction()
    test_laws()
    test_creation_and_execution()
```

### 명시적 중간값도 합성이다

실무에서는 `discounted = discount(amount)`처럼 중간값을 쓰는 편이 더 읽기 좋을 수 있다.
함수 합성의 사고방식은 특정 헬퍼 함수를 반드시 사용해야 한다는 규칙이 아니다.
입출력 계약을 연결하고 순서를 명확하게 유지하는 것이 핵심이다.

---

## 10. Python의 표현 한계

### 가변 길이 합성의 타입

임의 개수의 서로 다른 타입 단계들을 하나의 함수로 묶는 정확한 타입은 단순한 `Callable[...,
object]`보다 복잡하다.
타입 정보를 버리고 모든 값을 `Any`로 처리하면 연결 오류를 숨길 수 있다.
두 단계 합성이나 명시적 중간 함수를 사용하는 것이 더 나은 선택일 수 있다.

### 도메인 단위

정수라는 타입만으로 금액, 수량, 시간 단위를 구분하지 못한다.
필요하면 데이터 클래스나 명목적인 래퍼를 사용한다.
표면상 합성 가능하다는 이유로 업무 의미까지 맞다고 판단하지 않는다.

### 효과 검사

타입 힌트는 합성된 함수의 효과를 자동으로 추적하지 않는다.
각 단계의 구현과 계약을 확인해야 한다.
네트워크 호출이 숨어 있으면 평가 시점과 실패 정책이 달라질 수 있다.

### 최적화 보장

Python이 합성 래퍼를 항상 제거하거나 인라인한다고 가정하지 않는다.
측정이 필요한 경우 명시적 호출과 합성 헬퍼를 같은 입력으로 비교한다.
성능 결과는 특정 환경의 관측이며 의미상의 법칙을 대신하지 않는다.

---

## 11. 핵심 정리

### 핵심 결론

함수 합성은 중간 타입을 통해 작은 계산을 연결한다.
`andThen`과 `compose`는 읽는 방향을 구분해야 한다.
결합법칙은 괄호의 이동이고 교환법칙은 순서의 교체다.
일반 합성은 실패 컨텍스트나 트랜잭션을 자동 처리하지 않는다.

### 연습 1: 순서 계산

원금 10,000에 10% 할인과 배송비 3,000을 적용한다.
할인 후 배송비와 배송비 후 할인의 결과를 각각 구하라.

**해설.** 전자는 12,000이고 후자는 11,700이다.
배송비를 할인 대상에 포함하는지의 정책 차이다.
타입이 같아도 업무 순서가 다르면 결과가 달라진다.

### 연습 2: 결합법칙

`(h ∘ g) ∘ f`와 `h ∘ (g ∘ f)`를 입력 `x`에 적용한 식으로 풀어라.

**해설.** 둘 다 `h(g(f(x)))`가 된다.
함수 적용 순서는 `f`, `g`, `h`로 같다.
이 법칙은 `f`와 `g`의 순서를 바꿀 수 있다는 뜻이 아니다.

### 연습 3: 연결되지 않는 타입

`parse: String -> Option[Int]`와 `double: Int -> Int`를 그대로 합성할 수 없는 이유를 설명하라.

**해설.** 중간 결과는 `Int`가 아니라 `Option[Int]`다.
부재를 유지하면서 내부 값만 변환하려면 `map` 같은 연산이 필요하다.
실패 가능한 다음 계산까지 연결하려면 `flatMap`을 검토한다.

### 연습 4: 재시도

조회, 계산, 결제를 합친 함수를 통째로 재시도할 때 생길 수 있는 문제를 설명하라.

**해설.** 이미 성공한 결제가 다시 실행될 수 있다.
순수 계산의 재실행과 외부 효과의 재실행을 분리해야 한다.
단계별 멱등성과 실패 상태를 명시한다.

### 다음 장과 참고 자료

다음 장은 함수를 컬렉션 내부 값에 적용하는 `map`을 다룬다.
일반 함수 합성과 구조 안의 변환이 어떻게 연결되는지 살펴본다.

[Scala 공식 문서: Functions](https://docs.scala-lang.org/scala3/book/fun-intro.html)
[Python 공식 문서: typing](https://docs.python.org/3.14/library/typing.html)
