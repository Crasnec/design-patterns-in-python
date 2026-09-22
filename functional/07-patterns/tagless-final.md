# 50장. Tagless Final

![같은 종을 손잡이나 물레방아로 울리는 두 해석 방식](../../assets/images/fp/tagless-final.png)

앞 장에서는 계산식을 ADT로 만들고 패턴 매칭으로 해석했다.
Tagless Final은 프로그램을 명시적인 문법 트리 대신 연산 인터페이스에 대해 작성하는 다른 표현
방식이다.
프로그램은 구체적인 결과 표현을 모르고, 전달된 해석기가 각 연산의 의미를 제공한다.

이번 장은 같은 가격 계산을 오류 가능 평가와 문자열 표시로 해석한다.
Scala의 타입 생성자 매개변수와 서로 다른 결과 타입의 관계를 사용한다.
Python에서는 단일 값 영역에 한정한 실용적인 대응을 보여 주고 같은 고차 타입 표현력이라고
주장하지 않는다.

---

## 1. 개념과 기본 구분

### 인터페이스에 대해 쓰는 프로그램

연산 인터페이스는 프로그램이 사용할 수 있는 언어를 정의한다.
프로그램은 그 인터페이스의 메서드만 사용한다.
구체적인 평가값이나 AST 노드의 생성자를 직접 선택하지 않는다.

```text
program: 모든 해석 표현 F에 대해 Pricing[F] -> F[Amount]
```

여기서 `F`는 결과의 표현을 결정한다.
평가에서는 오류 결과, 표시에서는 타입이 붙은 문서값이 될 수 있다.
프로그램의 소스 구조는 같은 연산 인터페이스를 사용한다.

### Tagless의 의미

명시적인 문법 태그를 패턴 매칭하는 방식 대신 타입이 있는 연산 인터페이스로 프로그램을 표현한다.
이름이 런타임의 모든 태그와 분기, 할당을 제거한다는 뜻은 아니다.
오류 결과나 해석기 내부는 여전히 자신의 데이터 구조를 사용할 수 있다.

### Final의 의미

이 용어는 프로그램을 연산의 해석 인터페이스로 표현하는 스타일과 관련된다.
Scala의 `final` 키워드를 모든 클래스에 붙이라는 뜻이 아니다.
명시적인 문법을 먼저 만드는 표현과 비교하여 읽는다.

### 타입별 표현

`F[Amount]`와 `F[Boolean]`을 구분하면 금액과 조건을 잘못 섞는 표현을 줄일 수 있다.
해석기가 그 타입 관계를 보존해야 한다.
값의 업무 범위와 외부 효과까지 타입 모양만으로 모두 검증되는 것은 아니다.

---

## 2. 명령형 스타일과 함수형 스타일

### 명시적인 AST의 생성

```text
Add(Scale(Price("A"), 2), Scale(Price("B"), 3))
```

이 표현은 프로그램 데이터로 남고 해석기는 각 노드를 분해한다.
새 해석기를 추가하기 쉽지만 새 노드가 생기면 기존 패턴 매칭을 검토해야 한다.
앞 장에서 배운 표현 방식이다.

### 연산 인터페이스의 사용

```text
algebra.add(
  algebra.scale(algebra.price("A"), 2),
  algebra.scale(algebra.price("B"), 3)
)
```

프로그램은 연산을 요청하고 해석기가 결과 표현을 만든다.
그 결과가 금액인지 문서인지 AST인지 프로그램이 직접 정하지 않는다.
해석기를 바꾸어 같은 프로그램을 다른 의미로 실행할 수 있다.

### 인터페이스도 언어다

메서드의 입력과 출력은 허용되는 계산을 제한한다.
가격 결과를 더하거나 조건을 선택하는 연산이 타입으로 연결된다.
단순히 모든 함수에 제네릭 매개변수를 붙였다고 같은 설계가 되는 것은 아니다.

### 분석의 다른 경로

기본 표현에 명시적인 AST가 없으면 프로그램을 임의로 분해하기 어렵다.
필요하면 해석기 하나가 AST나 의존성 요약을 만들도록 할 수 있다.
Tagless Final이 구조 분석을 절대 불가능하게 만든다는 뜻은 아니다.

---

## 3. 왜 이 개념을 사용하는가?

### 같은 프로그램의 여러 해석

업무 표현식을 한 번 작성하고 평가와 표시를 분리할 수 있다.
테스트용 해석기나 설명용 해석기를 추가할 수 있다.
각 해석기가 제공하는 의미와 전제조건을 확인해야 한다.

### 타입 관계의 보존

금액 연산과 조건 연산을 구분하는 인터페이스를 사용할 수 있다.
잘못된 연결이 프로그램의 타입 검사에서 드러날 수 있다.
외부 입력에서 언어를 파싱하는 문제는 별도다.

### 확장 방식의 선택

작은 연산 인터페이스들을 조합하여 필요한 능력을 요구할 수 있다.
모든 프로그램이 거대한 공통 인터페이스를 받을 필요는 없다.
새 연산과 새 해석기 중 어느 방향의 변화가 많은지 고려한다.

### 직접적인 해석

해석기가 값을 바로 계산할 수 있어 명시적인 AST 생성이 불필요한 경우가 있다.
하지만 메서드 호출과 함수, 결과 래퍼의 비용이 자동으로 모두 사라지는 것은 아니다.
실제 구현과 최적화 결과를 확인해야 한다.

### 효과 추상화와의 관계

애플리케이션에서는 `F[A]` 형태의 효과를 반환하는 포트에도 같은 스타일을 사용할 수 있다.
그 경우에도 오류, 취소, 자원의 의미는 구체적인 해석기 계약이다.
인터페이스 일반화와 실행 보장을 구분한다.

---

## 4. Scala에서의 표현

### Scala의 타입이 있는 연산 인터페이스

가격 계산 외에 비교와 조건 선택을 넣어 `F[BigInt]`와 `F[Boolean]`의 차이를 보여 준다.
조건의 두 분기는 함수로 전달하여 평가 해석기가 필요한 분기만 실행할 수 있게 한다.
표시 해석기는 양쪽 분기의 문서를 만들어 전체 식을 보여 준다.

<!-- executable:scala -->
```scala
object Chapter50:
  enum Error:
    case MissingPrice(sku: String)
    case InvalidQuantity
    case InvalidRate
  type Result[A] = Either[Error, A]
  final case class Document[A](text: String)

  trait Pricing[F[_]]:
    def literal(amount: BigInt): F[BigInt]
    def price(sku: String): F[BigInt]
    def add(left: F[BigInt], right: F[BigInt]): F[BigInt]
    def scale(value: F[BigInt], quantity: Int): F[BigInt]
    def discount(value: F[BigInt], bps: Int): F[BigInt]
    def lessThan(left: F[BigInt], right: F[BigInt]): F[Boolean]
    def choose[A](condition: F[Boolean], whenTrue: () => F[A], whenFalse: () => F[A]): F[A]

  def program[F[_]](using P: Pricing[F]): F[BigInt] =
    val items = P.add(P.scale(P.price("A"), 2), P.scale(P.price("B"), 3))
    P.discount(items, 1000)

  final class Evaluate(prices: Map[String, BigInt]) extends Pricing[Result]:
    def literal(amount: BigInt): Result[BigInt] = Right(amount)
    def price(sku: String): Result[BigInt] = prices.get(sku).toRight(Error.MissingPrice(sku))
    def add(left: Result[BigInt], right: Result[BigInt]): Result[BigInt] =
      for a <- left; b <- right yield a + b
    def scale(value: Result[BigInt], quantity: Int): Result[BigInt] =
      if quantity < 0 then Left(Error.InvalidQuantity) else value.map(_ * quantity)
    def discount(value: Result[BigInt], bps: Int): Result[BigInt] =
      if bps < 0 || bps > 10000 then Left(Error.InvalidRate)
      else value.map(amount => amount - amount * bps / 10000)
    def lessThan(left: Result[BigInt], right: Result[BigInt]): Result[Boolean] =
      for a <- left; b <- right yield a < b
    def choose[A](condition: Result[Boolean], whenTrue: () => Result[A], whenFalse: () => Result[A]): Result[A] =
      condition.flatMap(value => if value then whenTrue() else whenFalse())

  object Display extends Pricing[Document]:
    def literal(amount: BigInt): Document[BigInt] = Document(amount.toString)
    def price(sku: String): Document[BigInt] = Document(s"price($sku)")
    def add(left: Document[BigInt], right: Document[BigInt]): Document[BigInt] = Document(s"(${left.text} + ${right.text})")
    def scale(value: Document[BigInt], quantity: Int): Document[BigInt] = Document(s"(${value.text} * $quantity)")
    def discount(value: Document[BigInt], bps: Int): Document[BigInt] = Document(s"discount(${value.text}, ${bps}bps)")
    def lessThan(left: Document[BigInt], right: Document[BigInt]): Document[Boolean] = Document(s"(${left.text} < ${right.text})")
    def choose[A](condition: Document[Boolean], whenTrue: () => Document[A], whenFalse: () => Document[A]): Document[A] =
      Document(s"if ${condition.text} then ${whenTrue().text} else ${whenFalse().text}")

  def check(): Unit =
    val evaluation = new Evaluate(Map("A" -> BigInt(1000), "B" -> BigInt(500)))
    assert(program[Result](using evaluation) == Right(BigInt(3150)))
    assert(program[Result](using new Evaluate(Map("A" -> BigInt(1000)))) == Left(Error.MissingPrice("B")))
    assert(program[Document](using Display).text == "discount(((price(A) * 2) + (price(B) * 3)), 1000bps)")
    val condition = evaluation.lessThan(evaluation.literal(1), evaluation.literal(2))
    assert(condition == Right(true))
    assert(evaluation.choose(condition, () => evaluation.literal(7), () => evaluation.price("missing")) == Right(BigInt(7)))
    val failedCondition: Result[Boolean] = Left(Error.MissingPrice("condition"))
    var branches = 0
    val branch: () => Result[Int] = () => { branches += 1; Right(1) }
    assert(evaluation.choose(failedCondition, branch, branch) == Left(Error.MissingPrice("condition")))
    assert(branches == 0)
    val shown = Display.choose(Display.lessThan(Display.literal(1), Display.literal(2)),
      () => Display.literal(7), () => Display.literal(9))
    assert(shown.text == "if (1 < 2) then 7 else 9")
```

### 추상 프로그램의 타입 검사

`program`은 구체적인 Result나 Document를 모르고 `Pricing[F]`만 사용한다.
금액을 더하는 자리에 `F[Boolean]`을 넣는 코드는 같은 계약으로 연결되지 않는다.
표시 결과가 문자열이라고 추상 프로그램의 금액과 조건 구분까지 사라지는 것은 아니다.

### 평가 전략의 명시

일반 메서드 인자는 엄격하게 평가될 수 있다.
오류 결과의 `add`에 전달하기 전에 양쪽 결과를 이미 만들 수 있다는 점을 확인해야 한다.
조건 선택에는 분기를 함수로 전달하여 필요한 실행 정책을 명시했다.

---

## 5. 상태 변경보다 값 변환

### 해석 표현의 선택

프로그램은 해석기를 받아 결과 표현을 만든다.
명시적인 AST가 중간에 반드시 존재하는 것은 아니다.
필요하면 AST를 만드는 해석기를 추가하는 방식으로 구조를 다시 얻을 수 있다.

```mermaid
flowchart LR
    A["Pricing 인터페이스로 작성한 프로그램"] --> B["Evaluate 해석기"]
    A --> C["Display 해석기"]
    B --> D["오류 가능 금액"]
    C --> E["타입이 붙은 문서"]
```

각 해석기가 프로그램을 한 번에 실행하는지 지연된 설명을 만드는지 확인한다.
`F`라는 이름만으로 실행 시점이 결정되지 않는다.
표현의 다형성과 효과의 실행 의미를 구분한다.

### 데이터로 다시 표현하기

연산 메서드가 AST 노드를 반환하도록 하는 해석기를 만들 수 있다.
그러면 Tagless Final 프로그램을 명시적인 문법 구조로 재해석할 수 있다.
분석 요구가 있는 경우 표현 방식을 혼합할 수도 있다.

### 숨은 효과의 위험

추상 프로그램 안에서 임의의 외부 출력이나 전역 변경을 수행하면 해석기 교체의 의미가 흐려진다.
허용한 연산 인터페이스만 사용하는 규칙을 실제로 지켜야 한다.
일반적인 Scala 타입 매개변수가 모든 숨은 효과를 자동 금지하지는 않는다.

---

## 6. 함수 합성과 데이터 흐름

### 타입 클래스와의 관계

연산 인터페이스를 문맥 인자로 전달하는 구조는 타입 클래스와 연결된다.
하지만 여기서는 그 인터페이스로 하나의 프로그램 언어를 표현한다.
일반적인 제네릭 유틸리티와 언어 해석 설계의 목적을 구분한다.

```text
연산 계약 + 다형적인 프로그램 + 여러 해석
```

### 작은 대수의 조합

가격 읽기와 로그 기록, 오류 처리 같은 계약을 나누어 필요한 것만 요구할 수 있다.
프로그램의 능력 범위가 더 명확해질 수 있다.
너무 많은 작은 인터페이스가 오히려 조립을 어렵게 만드는지도 확인한다.

### 프로그램 확장

새 연산을 요구하는 프로그램은 확장된 인터페이스를 받을 수 있다.
기존 프로그램이 새 연산을 사용하지 않으면 기존 계약으로 남을 수 있다.
실제 언어와 라이브러리의 인터페이스 조합 방식을 검토해야 한다.

### Free와의 비교 예고

Free는 연산과 다음 계산을 명시적인 프로그램 구조로 보관하는 다른 방향이다.
Tagless Final은 해석 인터페이스에 대해 직접 프로그램을 작성한다.
두 방식 모두 만능이 아니며 분석·실행·확장 요구에 따라 선택한다.

---

## 7. 장점과 트레이드오프

### 장점과 트레이드오프

| 선택 | 이점 | 주의점 |
| --- | --- | --- |
| 다형적인 프로그램 | 해석 교체 | 추상 타입의 학습 비용 |
| 타입별 표현 | 잘못된 연결 감소 | 해석기 내부의 계약 |
| 직접 해석 | AST 생성을 생략 가능 | 비용이 자동 소거되지는 않음 |
| 인터페이스 확장 | 능력별 조합 | 인터페이스 복잡성 |
| 구조 재해석 | 필요할 때 AST 생성 | 추가 해석기 구현 |

### 읽기 어려운 타입

여러 효과와 대수 제약이 겹치면 시그니처가 길어질 수 있다.
도메인 이름과 작은 프로그램으로 분리하여 필요한 능력을 설명한다.
추상화를 줄여도 같은 목적을 달성할 수 있는지 검토한다.

### 관측과 디버깅

명시적인 프로그램 트리가 없으면 임의의 노드를 중간에 검사하기 어려울 수 있다.
기록 해석기나 문서 해석기를 사용하여 필요한 관측을 추가할 수 있다.
모든 해석이 자동으로 같은 진단 정보를 제공하는 것은 아니다.

### 비용의 검증

타입 매개변수의 존재만으로 모든 간접 호출과 래퍼가 제거되는 것은 아니다.
해석기가 만드는 값과 함수의 실제 할당을 확인해야 한다.
성능 주장은 구체적인 구현과 측정에 근거해야 한다.

---

## 8. 상태와 부수효과의 경계

### 효과를 반환하는 포트

실제 서비스에서는 `lookup: Id -> F[Product]` 같은 포트를 사용할 수 있다.
구체적인 `F`가 실패와 비동기 실행을 담당할 수 있다.
하지만 취소와 자원 수명은 포트와 실행기의 별도 계약으로 확인한다.

### 해석기 교체의 조건

테스트 해석기와 실제 해석기가 같은 메서드를 제공해도 실패·시간·일관성 의미가 다를 수 있다.
대체가 유효한 범위를 명시해야 한다.
공통 인터페이스가 모든 실행 관측의 동일성을 보장하지는 않는다.

### 권한의 범위

프로그램에 필요한 연산 인터페이스만 제공하면 코드 수준의 능력을 좁힐 수 있다.
해석기가 가진 실제 외부 권한은 별도로 관리해야 한다.
인터페이스 다형성과 보안 격리를 혼동하지 않는다.

### 실행과 조립의 혼합

어떤 해석기는 연산 호출 시 바로 효과를 수행할 수 있다.
다른 해석기는 나중 실행할 값을 만들 수 있다.
프로그램 조립 시점에 무엇이 발생하는지 구체적인 계약을 확인해야 한다.

---

## 9. Python에서 적용하기

### Python의 단일 값 영역 대안

표준 Python 타입 체계에서 임의의 `F[A]` 관계를 Scala와 같은 방식으로 직접 표현하지는 않는다.
아래 예제는 모든 연산 결과가 하나의 표현 타입 `R`을 쓰는 가격 언어로 범위를 제한한다.
금액과 불리언을 구분하는 Scala 예제 전체의 직접적인 고차 타입 대응은 아니다.

<!-- executable:python -->
```python
from dataclasses import dataclass
from typing import Protocol, TypeVar

R = TypeVar("R")


class Pricing(Protocol[R]):
    def literal(self, amount: int) -> R:
        ...

    def price(self, sku: str) -> R:
        ...

    def add(self, left: R, right: R) -> R:
        ...

    def scale(self, value: R, quantity: int) -> R:
        ...

    def discount(self, value: R, bps: int) -> R:
        ...


def program(algebra: Pricing[R]) -> R:
    items = algebra.add(algebra.scale(algebra.price("A"), 2), algebra.scale(algebra.price("B"), 3))
    return algebra.discount(items, 1000)


@dataclass(frozen=True)
class Error:
    code: str
    sku: str = ""


Result = int | Error


class Evaluate:
    def __init__(self, prices: dict[str, int]) -> None:
        self._prices = dict(prices)

    def literal(self, amount: int) -> Result:
        return amount

    def price(self, sku: str) -> Result:
        return self._prices.get(sku, Error("missing_price", sku))

    def add(self, left: Result, right: Result) -> Result:
        if isinstance(left, Error):
            return left
        return right if isinstance(right, Error) else left + right

    def scale(self, value: Result, quantity: int) -> Result:
        if quantity < 0:
            return Error("invalid_quantity")
        return value if isinstance(value, Error) else value * quantity

    def discount(self, value: Result, bps: int) -> Result:
        if not 0 <= bps <= 10_000:
            return Error("invalid_rate")
        return value if isinstance(value, Error) else value - value * bps // 10_000


class Display:
    def literal(self, amount: int) -> str:
        return str(amount)

    def price(self, sku: str) -> str:
        return f"price({sku})"

    def add(self, left: str, right: str) -> str:
        return f"({left} + {right})"

    def scale(self, value: str, quantity: int) -> str:
        return f"({value} * {quantity})"

    def discount(self, value: str, bps: int) -> str:
        return f"discount({value}, {bps}bps)"


def test_final_encoding() -> None:
    evaluated = program(Evaluate({"A": 1000, "B": 500}))
    displayed = program(Display())
    assert evaluated == 3150
    assert displayed == "discount(((price(A) * 2) + (price(B) * 3)), 1000bps)"
    assert program(Evaluate({"A": 1000})) == Error("missing_price", "B")
    evaluation = Evaluate({})
    assert evaluation.scale(1, -1) == Error("invalid_quantity")
    assert evaluation.discount(1, 10_001) == Error("invalid_rate")
    assert evaluation.add(Error("left"), Error("right")) == Error("left")
    assert Display().add(Display().literal(1), Display().literal(2)) == "(1 + 2)"


if __name__ == "__main__":
    test_final_encoding()
```

### 표현 타입의 차이

평가 해석기는 정수 또는 오류를 반환하고 표시 해석기는 문자열을 반환한다.
프로그램은 하나의 연산 인터페이스만 사용한다.
이 범위에서는 일반 제네릭 프로토콜로 유용한 재사용을 얻을 수 있다.

---

## 10. Python의 표현 한계

### 고차 타입의 한계

하나의 `R`을 추상화하는 것과 임의의 원소 타입에 대해 `F[A]`를 추상화하는 것은 다르다.
불리언과 금액, 문자열 계산을 같은 생성자 아래 정밀하게 연결하려면 더 복잡한 인코딩이 필요할 수
있다.
예제의 단순한 대응을 전체 Scala 표현력과 같다고 설명하지 않는다.

### 구조적 프로토콜

프로토콜은 필요한 메서드와 타입 관계를 설명한다.
실행 시 구현의 법칙과 효과를 자동 확인하지 않는다.
정적 검사와 실제 해석기 테스트가 필요하다.

### 프로그램의 자유도

Python 함수는 인터페이스 연산 외에 임의의 전역 작업을 수행할 수 있다.
그런 작업이 들어가면 해석기 교체만으로 프로그램의 모든 의미를 바꿀 수 없다.
연산 계약에 대한 프로그램이라는 설계 규칙을 유지해야 한다.

### 비용과 가독성

동적 메서드 호출과 래퍼를 사용하면 직접 계산보다 비용과 코드가 늘 수 있다.
여러 해석과 확장의 이득이 있는지 판단한다.
추상적인 이름보다 실제 사용 사례가 중요하다.

---

## 11. 핵심 정리

### 핵심 결론

Tagless Final은 연산 인터페이스에 대해 프로그램을 작성하고 해석기로 의미를 선택하는 방식이다.
명시적인 AST를 항상 만들지 않아도 여러 해석을 제공할 수 있다.
타입별 표현 관계와 실행 시점, 숨은 효과의 범위를 명확히 해야 한다.
Python의 단일 표현 타입 대안과 Scala의 고차 타입 표현력을 구분한다.

### 연습 1: 이름의 오해

Tagless Final이므로 모든 런타임 태그와 할당이 없어진다는 주장은 맞는가?

**해설.** 아니다. 표현 스타일의 이름을 전체 실행 비용 보장으로 해석해서는 안 된다.
오류 결과와 문서, 해석기 내부는 자신의 구조를 사용할 수 있다.
실제 할당과 호출 비용은 구현을 확인해야 한다.

### 연습 2: AST 분석

Tagless Final 프로그램은 구조를 분석할 수 없다는 주장을 평가하라.

**해설.** 기본 표현에 직접 분해할 AST가 없을 수 있다.
하지만 AST나 요약을 만드는 해석기를 추가할 수 있다.
표현의 기본 형태와 가능한 다른 해석을 구분한다.

### 연습 3: 조건의 평가

조건 선택 메서드에 양쪽 계산 결과를 엄격한 인자로 전달했다.
어떤 실행 문제가 생길 수 있는가?

**해설.** 메서드를 호출하기 전에 양쪽 계산이 이미 수행될 수 있다.
필요한 분기만 실행하려면 지연 인자나 명시적인 함수 전달이 필요하다.
인터페이스에 평가 전략을 드러낸다.

### 연습 4: Python의 대응

`Pricing[R]`가 `Pricing[F[_]]`의 모든 타입 관계를 그대로 표현하는가?

**해설.** 예제는 단일 값 영역의 표현 타입만 추상화한다.
`F[Boolean]`과 `F[Amount]` 같은 가족 전체의 관계를 직접 표현한 것은 아니다.
대안의 유용성과 한계를 함께 설명해야 한다.

### 다음 장과 참고 자료

다음 장은 연산과 다음 계산을 프로그램 구조로 보관하는 Free Monad를 다룬다.
프로그램 조립과 해석을 분리하면서 스택 사용과 분석의 한계도 확인한다.

[Oleg Kiselyov: Tagless-Final Style](https://okmij.org/ftp/tagless-final/)
[Scala 공식 문서: Type Lambdas](https://docs.scala-lang.org/scala3/reference/new-types/type-lambdas.html)
[Python 타입 명세: Generics](https://typing.python.org/en/latest/spec/generics.html)
