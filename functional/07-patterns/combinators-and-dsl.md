# 52장. Combinators & DSL

![작은 쐐기돌을 조합해 만든 정원 아치](../../assets/images/fp/combinators-and-dsl.png)

고차 함수는 작은 동작을 받아 더 큰 동작을 만들 수 있다.
이 원리를 입력 해석에 적용하면 작은 파서를 조합하여 하나의 언어를 만들 수 있다.
조합기와 DSL은 앞서 배운 함수값, 합성, 결과 타입, 재귀를 하나의 실용적인 문제로 묶는다.

이번 장의 언어는 `A:2,B:3`처럼 상품 코드와 수량을 나열하는 작은 주문 형식이다.
공백과 뒤의 불필요한 문자를 어떻게 처리하는지, 실패 위치와 되돌리기의 의미를 명시한다.
범용 파서 라이브러리 전체를 구현하지 않으며 좌재귀와 무제한 입력을 지원한다고 주장하지 않는다.

---

## 1. 개념과 기본 구분

### 조합기

조합기는 이미 정의된 계산을 받아 새로운 계산을 만드는 함수다.
파서 조합기는 파서를 받아 순차 연결, 선택, 반복 같은 새로운 파서를 만든다.
각 조합기의 타입뿐 아니라 입력 소비와 실패의 의미가 중요하다.

```text
Parser[A] ≈ (입력 문자열, 시작 위치) -> ParseResult[A]
```

성공에는 결과값과 다음 입력 위치가 들어 있다.
실패에는 오류 위치와 기대한 요소가 들어 있다.
입력 위치를 숨은 전역 변수로 관리하지 않고 명시적인 값으로 전달한다.

### DSL

DSL은 특정 문제 영역을 표현하기 위한 작은 언어다.
이번 장의 문자열 형식은 주문 행 나열이라는 제한된 목적을 가진다.
내부 DSL은 호스트 언어의 함수와 연산을 사용하고, 외부 DSL은 별도의 문자열이나 파일 문법을 가질
수 있다.

### 순차 연결과 선택

순차 연결은 첫 파서가 끝난 위치에서 다음 파서를 시작한다.
선택은 첫 파서가 실패했을 때 다른 문법을 시도한다.
실패 전에 소비한 입력을 되돌릴지 여부는 구현마다 다른 계약이다.

### 반복의 진행 조건

반복 안의 파서가 성공해도 입력 위치가 전진하지 않으면 같은 성공을 무한히 반복할 수 있다.
반복 조합기는 이 상황을 금지하거나 명시적으로 처리해야 한다.
파서가 성공했다는 사실만으로 전체 반복이 진행하는 것은 아니다.

---

## 2. 명령형 스타일과 함수형 스타일

### 문자열을 수동으로 나눈다

```text
쉼표로 행을 나눈다
콜론으로 각 행을 나눈다
상품 코드와 수량을 검사한다
```

작은 고정 형식에는 이런 구현이 더 간단할 수 있다.
하지만 선택·중첩·인용·오류 위치 같은 요구가 늘어나면 분할과 검사가 흩어질 수 있다.
파서 조합기는 반복되는 입력 처리 규칙을 이름 있는 연산으로 묶는다.

### 작은 문법의 정의

```text
order    = item ("," item)*
item     = sku ":" quantity
sku      = 대문자·숫자로 시작하는 제한된 코드
quantity = 1 이상 1000000 이하의 ASCII 숫자
```

예제는 공백을 자동으로 건너뛰지 않는다.
빈 주문도 허용하지 않는다.
허용 문법을 코드와 테스트에 같은 의미로 반영해야 한다.

### 전체 입력 확인

앞부분 `A:2`만 읽고 뒤의 `junk`를 무시하면 잘못된 입력을 성공으로 받을 수 있다.
최상위 파서는 성공한 위치가 입력 끝인지 확인한다.
접두사 파싱과 전체 문서 파싱을 구분한다.

### 구체적인 실패 정보

단순한 `False` 대신 어디에서 어떤 요소를 기대했는지 반환한다.
화면이나 API 경계는 그 정보를 사용자 안내로 바꿀 수 있다.
내부 문법 이름을 외부에 그대로 공개할지 여부는 별도의 표시 정책이다.

---

## 3. 왜 이 개념을 사용하는가?

### 문법의 조립

상품 코드 파서와 수량 파서를 각각 테스트할 수 있다.
그 둘을 연결한 행 파서를 다시 반복하여 주문 파서를 만든다.
작은 계약에서 큰 문법으로 확장하는 구조가 드러난다.

### 오류의 일관성

모든 파서는 같은 성공·실패 구조를 반환한다.
선택에서 더 멀리 진행한 실패를 보존하거나 같은 위치의 기대값을 합칠 수 있다.
오류 결합 정책은 실제 문법의 사용 경험에 영향을 준다.

### 입력 상태의 명시

현재 위치가 인자와 결과에 포함되므로 파서 간의 상태 전달을 추적하기 쉽다.
외부에서 입력 문자열을 변경하지 않는다면 같은 입력과 위치에서 같은 결과를 계산할 수 있다.
파서 콜백에 숨은 효과가 들어가는 경우는 별도로 검토한다.

### 의미값의 생성

문법을 읽은 결과를 단순한 문자열 조각이 아니라 `Item` 같은 내부 값으로 만든다.
뒤의 계산이 동일한 숫자 해석을 반복할 필요가 줄어든다.
Parse, Don't Validate의 원칙이 파서 조합에도 연결된다.

### 제한의 문서화

최대 입력 길이, 코드 길이, 숫자 길이를 명시한다.
문법상 맞더라도 처리 비용이 지나치게 큰 입력을 제한할 수 있다.
작은 DSL의 표현력을 의도적으로 좁히는 것은 유효한 설계 선택이다.

---

## 4. Scala에서의 표현

### Scala의 작은 파서 조합기

선택은 첫 분기의 실패에서 원래 위치로 돌아간다.
반복은 일반적인 실패를 만나면 해당 반복을 끝내지만, 성공했는데 위치가 전진하지 않으면 명시적인
오류를 반환한다.
이 단순한 정책은 소비 후 실패를 구분하는 고급 파서 라이브러리와 같지 않다.

<!-- executable:scala -->
```scala
object Chapter52:
  enum Parsed[+A]:
    case Ok(value: A, next: Int)
    case Bad(position: Int, expected: Set[String])

  final case class Parser[A](parse: (String, Int) => Parsed[A]):
    def map[B](f: A => B): Parser[B] = flatMap(a => Parser.pure(f(a)))
    def flatMap[B](f: A => Parser[B]): Parser[B] = Parser { (input, start) =>
      parse(input, start) match
        case Parsed.Ok(value, next) => f(value).parse(input, next)
        case Parsed.Bad(position, expected) => Parsed.Bad(position, expected)
    }
    def orElse(other: () => Parser[A]): Parser[A] = Parser { (input, start) =>
      parse(input, start) match
        case success @ Parsed.Ok(_, _) => success
        case Parsed.Bad(firstPosition, firstExpected) =>
          other().parse(input, start) match
            case success @ Parsed.Ok(_, _) => success
            case Parsed.Bad(secondPosition, secondExpected) =>
              if firstPosition > secondPosition then Parsed.Bad(firstPosition, firstExpected)
              else if secondPosition > firstPosition then Parsed.Bad(secondPosition, secondExpected)
              else Parsed.Bad(firstPosition, firstExpected ++ secondExpected)
    }
    def many: Parser[Vector[A]] = Parser { (input, start) =>
      val output = Vector.newBuilder[A]
      var position = start
      var running = true
      var invalidProgress = false
      while running do
        parse(input, position) match
          case Parsed.Ok(value, next) =>
            if next <= position || next > input.length then
              invalidProgress = true
              running = false
            else
              output += value
              position = next
          case Parsed.Bad(_, _) => running = false
      if invalidProgress then Parsed.Bad(position, Set("forward input progress"))
      else Parsed.Ok(output.result(), position)
    }

  object Parser:
    def pure[A](value: A): Parser[A] = Parser((_, start) => Parsed.Ok(value, start))
    def fail[A](expected: String): Parser[A] = Parser((_, start) => Parsed.Bad(start, Set(expected)))
    def char(expected: Char): Parser[Char] = Parser { (input, start) =>
      if start < input.length && input(start) == expected then Parsed.Ok(expected, start + 1)
      else Parsed.Bad(start, Set(expected.toString))
    }
    def token(label: String, limit: Int)(accept: Char => Boolean): Parser[String] = Parser { (input, start) =>
      var end = start
      while end < input.length && end - start < limit && accept(input(end)) do end += 1
      if end == start then Parsed.Bad(start, Set(label))
      else if end < input.length && accept(input(end)) then Parsed.Bad(end, Set(s"$label within length limit"))
      else Parsed.Ok(input.substring(start, end), end)
    }

  final case class Item(sku: String, quantity: Int)
  val sku: Parser[String] = Parser.token("SKU", 32)(ch =>
    (ch >= 'A' && ch <= 'Z') || (ch >= '0' && ch <= '9') || ch == '-').flatMap { text =>
      if text.head == '-' then Parser.fail("SKU beginning with a letter or digit") else Parser.pure(text)
    }
  val quantity: Parser[Int] = Parser.token("quantity", 7)(ch => ch >= '0' && ch <= '9').flatMap { text =>
    val value = text.toInt
    if value >= 1 && value <= 1000000 then Parser.pure(value) else Parser.fail("quantity in 1..1000000")
  }
  val item: Parser[Item] = for
    code <- sku
    _ <- Parser.char(':')
    count <- quantity
  yield Item(code, count)
  val order: Parser[Vector[Item]] = for
    first <- item
    rest <- Parser.char(',').flatMap(_ => item).many
  yield first +: rest

  def parseAll[A](parser: Parser[A], input: String): Parsed[A] =
    if input.length > 4096 then Parsed.Bad(4096, Set("input of at most 4096 characters"))
    else parser.parse(input, 0) match
      case Parsed.Ok(value, next) if next == input.length => Parsed.Ok(value, next)
      case Parsed.Ok(_, next) => Parsed.Bad(next, Set("end of input"))
      case Parsed.Bad(position, expected) => Parsed.Bad(position, expected)

  def isBad[A](result: Parsed[A]): Boolean = result match
    case Parsed.Bad(_, _) => true
    case Parsed.Ok(_, _) => false

  def check(): Unit =
    assert(parseAll(order, "A:2,B:3") == Parsed.Ok(Vector(Item("A", 2), Item("B", 3)), 7))
    assert(parseAll(order, "A-1:0002") == Parsed.Ok(Vector(Item("A-1", 2)), 8))
    assert(parseAll(order, "A:1000000") == Parsed.Ok(Vector(Item("A", 1000000)), 9))
    for invalid <- List("", "A:0", "A:1000001", "A:2,", "A:2junk", "A: 2", "-A:2", "A:２") do
      assert(isBad(parseAll(order, invalid)))
    assert(parseAll(Parser.pure(1).many, "") == Parsed.Bad(0, Set("forward input progress")))
    val choice = Parser.char('A').orElse(() => Parser.char('B'))
    assert(parseAll(choice, "B") == Parsed.Ok('B', 1))
    assert(parseAll(choice, "C") == Parsed.Bad(0, Set("A", "B")))
    val ab = Parser.char('A').flatMap(_ => Parser.char('B'))
    val ac = Parser.char('A').flatMap(_ => Parser.char('C'))
    assert(parseAll(ab.orElse(() => ac), "AC") == Parsed.Ok('C', 2))
    assert(isBad(parseAll(order, "A" * 5000)))
```

### 반복 실패의 진단 한계

`A:2,`에서 두 번째 행 파서가 실패하면 반복은 쉼표 이전 위치로 돌아간다.
최상위 검사가 남은 쉼표를 발견하여 전체 입력을 거부한다.
이 구현은 실패한 두 번째 행의 상세 원인을 최종 오류에 보존하지 않으므로, 더 좋은 진단에는 소비
여부와 확정 실패를 구분하는 설계가 필요하다.

### 내부 파서의 전제조건

기본 파서의 시작 위치는 올바른 조합기가 만든 유효한 입력 위치라고 가정한다.
외부 API는 `parseAll`을 통해 0에서 시작한다.
임의의 음수 위치를 직접 전달하는 공개 파싱 인터페이스까지 구현한 것은 아니다.

---

## 5. 상태 변경보다 값 변환

### 상태를 값으로 넘긴다

파서가 반환하는 다음 위치는 뒤 파서의 입력이 된다.
실패 결과는 값으로 남고 선택 조합기는 필요한 위치에서 다시 시도한다.
State와 오류 결과의 조합을 구체적인 입력 해석 문제에서 볼 수 있다.

```mermaid
flowchart LR
    A["문자열과 위치 0"] --> B["SKU 파서"]
    B --> C["콜론 파서"]
    C --> D["수량 파서"]
    D --> E["Item과 다음 위치"]
    E --> F["쉼표와 다음 Item 반복"]
    F --> G["입력 끝 확인"]
```

위치는 문법의 실행 상태이며 외부 문자열 자체를 변경하지 않는다.
파서가 원본을 잘라서 계속 복사하는 대신 위치를 전달하면 불필요한 복사를 줄일 수 있다.
반환할 토큰 문자열의 생성 비용은 여전히 존재한다.

### 의미값의 범위

성공한 Item의 수량은 파서가 정한 범위를 만족한다.
그러나 상품이 실제 카탈로그에 존재하는지와 현재 재고가 충분한지는 별도다.
문법 해석과 도메인 판단, 외부 실행의 경계를 유지한다.

### 외부 언어의 버전

문법에 공백이나 인용 문자를 추가하면 기존 입력의 해석이 달라질 수 있다.
오래 저장된 문자열을 다시 읽는 요구가 있는지 확인한다.
작은 DSL에도 버전과 호환성 정책이 필요할 수 있다.

---

## 6. 함수 합성과 데이터 흐름

### `map`과 `flatMap`

`map`은 읽은 의미값을 변환한다.
`flatMap`은 앞 결과에 따라 다음 파서를 선택하고 새 위치에서 실행한다.
항상 같은 다음 문법을 사용하는 순차 연결도 이 구조로 표현할 수 있다.

```text
Parser[A] + (A -> B)         -> Parser[B]
Parser[A] + (A -> Parser[B]) -> Parser[B]
```

### 선택의 법칙과 오류

선택 순서를 바꾸면 먼저 성공하는 분기가 바뀔 수 있다.
같은 문자열을 여러 문법이 받아들이면 선택 순서가 언어의 의미가 된다.
단순한 집합 합집합처럼 교환적으로 취급하지 않는다.

### 좌재귀

어떤 파서가 입력을 소비하기 전에 자기 자신을 다시 호출하면 종료하지 않을 수 있다.
`many`의 진행 검사는 성공한 반복의 문제를 막을 뿐 모든 재귀 문법을 해결하지 않는다.
좌재귀 제거, 다른 파싱 알고리즘, 전용 라이브러리의 지원을 검토해야 한다.

### 언어의 크기

인용과 탈출, 우선순위, 재귀 중첩이 늘어나면 작은 예제보다 더 정교한 문법과 진단이 필요하다.
몇 개의 조합기가 있다는 사실만으로 범용 프로그래밍 언어 파서가 완성되는 것은 아니다.
지원하는 문법과 실패 정책을 명시적으로 제한한다.

---

## 7. 장점과 트레이드오프

### 장점과 트레이드오프

| 구성 | 이점 | 주의점 |
| --- | --- | --- |
| 작은 파서 조합 | 문법별 재사용 | 조합의 평가 순서 |
| 명시적인 위치 | 오류와 되돌리기 | 위치 계약의 유지 |
| 전체 입력 검사 | 뒤의 쓰레기 입력 거부 | 접두사 파싱과 구분 |
| 항상 되돌리는 선택 | 간단한 대안 시도 | 과도한 재시도 비용 |
| 반복 진행 검사 | 빈 성공의 무한 반복 방지 | 모든 재귀 종료를 보장하지 않음 |

### 되돌리기 비용

중복된 접두사가 많은 문법은 같은 입력을 여러 번 읽을 수 있다.
선택을 정리하거나 특정 지점 이후 실패를 확정하면 비용과 진단을 개선할 수 있다.
입력 길이 제한만으로 모든 복잡도 문제가 사라지는 것은 아니다.

### 오류 메시지의 품질

더 먼 위치의 실패가 항상 가장 이해하기 쉬운 오류인 것은 아니다.
도메인 문법에 이름을 붙이고 기대값을 사람이 이해할 표현으로 바꿀 수 있다.
기술적인 실패 위치와 사용자 안내의 수준을 구분한다.

### 단순 분할의 대안

형식이 정말 단순하고 바뀌지 않으면 명시적인 문자열 분할과 검증이 더 읽기 쉬울 수 있다.
조합기를 도입하기 전에 문법의 확장과 진단 요구를 확인한다.
DSL이라는 이름을 붙이기 위해 과도한 프레임워크를 만들지 않는다.

---

## 8. 상태와 부수효과의 경계

### 신뢰하지 않는 입력

길이와 중첩 깊이, 토큰 개수를 제한할 수 있어야 한다.
예제는 전체 길이와 토큰 길이를 제한하지만 일반적인 무제한 중첩 문법을 구현하지 않는다.
외부 언어는 입력 검증과 자원 제한을 함께 가져야 한다.

### 콜백의 효과

파서의 `map`에서 데이터베이스에 저장하면 되돌리기 과정에서 같은 효과가 반복될 수 있다.
문법을 읽는 동안은 가능한 한 의미값만 만들고 실행은 뒤의 경계에서 수행한다.
파싱 성공과 실제 명령 적용을 구분한다.

### 오류와 예외

문법 불일치는 정상적인 실패값이다.
조합기 내부의 버그와 콜백 예외는 자동으로 문법 실패로 바꾸지 않는다.
광범위한 포착은 잘못된 문법 구현을 숨길 수 있다.

### 코드 실행과 DSL

주문 문자열을 파싱하는 것은 임의의 호스트 언어 코드를 실행하는 것과 다르다.
`eval`로 입력을 실행하여 파싱을 대신하지 않는다.
허용한 문법과 명시적인 의미값의 경계를 유지한다.

---

## 9. Python에서 적용하기

### Python의 동일한 작은 문법

Python에서도 입력 위치를 전달하는 함수 래퍼로 조합기를 작성할 수 있다.
반복에는 명시적인 루프를 사용하고 입력을 소비하지 않는 성공을 검사한다.
타입 주석은 조합의 의도를 설명하지만 외부에서 만든 임의 파서의 올바름을 자동 보장하지 않는다.

<!-- executable:python -->
```python
from __future__ import annotations
from collections.abc import Callable
from dataclasses import dataclass
from typing import Generic, TypeVar

A = TypeVar("A")
B = TypeVar("B")


@dataclass(frozen=True)
class Ok(Generic[A]):
    value: A
    next_position: int


@dataclass(frozen=True)
class Bad:
    position: int
    expected: frozenset[str]


@dataclass(frozen=True)
class Parser(Generic[A]):
    parse: Callable[[str, int], Ok[A] | Bad]

    def map(self, function: Callable[[A], B]) -> Parser[B]:
        return self.flat_map(lambda value: pure(function(value)))

    def flat_map(self, function: Callable[[A], Parser[B]]) -> Parser[B]:
        def run(text: str, start: int) -> Ok[B] | Bad:
            result = self.parse(text, start)
            return result if isinstance(result, Bad) else function(result.value).parse(text, result.next_position)
        return Parser(run)

    def or_else(self, other: Callable[[], Parser[A]]) -> Parser[A]:
        def run(text: str, start: int) -> Ok[A] | Bad:
            first = self.parse(text, start)
            if isinstance(first, Ok):
                return first
            second = other().parse(text, start)
            if isinstance(second, Ok):
                return second
            if first.position != second.position:
                return first if first.position > second.position else second
            return Bad(first.position, first.expected | second.expected)
        return Parser(run)

    def many(self) -> Parser[tuple[A, ...]]:
        def run(text: str, start: int) -> Ok[tuple[A, ...]] | Bad:
            values: list[A] = []
            position = start
            while True:
                result = self.parse(text, position)
                if isinstance(result, Bad):
                    return Ok(tuple(values), position)
                if result.next_position <= position or result.next_position > len(text):
                    return Bad(position, frozenset({"forward input progress"}))
                values.append(result.value)
                position = result.next_position
        return Parser(run)


def pure(value: A) -> Parser[A]:
    return Parser(lambda text, start: Ok(value, start))


def fail(label: str) -> Parser[A]:
    return Parser(lambda text, start: Bad(start, frozenset({label})))


def char(expected: str) -> Parser[str]:
    return Parser(lambda text, start: Ok(expected, start + 1) if start < len(text) and text[start] == expected else Bad(start, frozenset({expected})))


def token(label: str, limit: int, accept: Callable[[str], bool]) -> Parser[str]:
    def run(text: str, start: int) -> Ok[str] | Bad:
        end = start
        while end < len(text) and end - start < limit and accept(text[end]):
            end += 1
        if end == start:
            return Bad(start, frozenset({label}))
        if end < len(text) and accept(text[end]):
            return Bad(end, frozenset({f"{label} within length limit"}))
        return Ok(text[start:end], end)
    return Parser(run)


@dataclass(frozen=True)
class Item:
    sku: str
    quantity: int


sku = token("SKU", 32, lambda ch: "A" <= ch <= "Z" or "0" <= ch <= "9" or ch == "-").flat_map(
    lambda value: fail("SKU beginning with a letter or digit") if value.startswith("-") else pure(value))
quantity = token("quantity", 7, lambda ch: "0" <= ch <= "9").flat_map(
    lambda value: pure(int(value)) if 1 <= int(value) <= 1_000_000 else fail("quantity in 1..1000000"))
item = sku.flat_map(lambda code: char(":").flat_map(lambda _: quantity.map(lambda count: Item(code, count))))
order = item.flat_map(lambda first: char(",").flat_map(lambda _: item).many().map(lambda rest: (first,) + rest))


def parse_all(parser: Parser[A], text: str) -> Ok[A] | Bad:
    if len(text) > 4096:
        return Bad(4096, frozenset({"input of at most 4096 characters"}))
    result = parser.parse(text, 0)
    if isinstance(result, Ok) and result.next_position != len(text):
        return Bad(result.next_position, frozenset({"end of input"}))
    return result


def test_dsl() -> None:
    assert parse_all(order, "A:2,B:3") == Ok((Item("A", 2), Item("B", 3)), 7)
    assert parse_all(order, "A-1:0002") == Ok((Item("A-1", 2),), 8)
    assert parse_all(order, "A:1000000") == Ok((Item("A", 1_000_000),), 9)
    for invalid in ("", "A:0", "A:1000001", "A:2,", "A:2junk", "A: 2", "-A:2", "A:２"):
        assert isinstance(parse_all(order, invalid), Bad)
    assert parse_all(pure(1).many(), "") == Bad(0, frozenset({"forward input progress"}))
    choice = char("A").or_else(lambda: char("B"))
    assert parse_all(choice, "B") == Ok("B", 1)
    assert parse_all(choice, "C") == Bad(0, frozenset({"A", "B"}))
    ab = char("A").flat_map(lambda _: char("B"))
    ac = char("A").flat_map(lambda _: char("C"))
    assert parse_all(ab.or_else(lambda: ac), "AC") == Ok("C", 2)
    assert isinstance(parse_all(order, "A" * 5000), Bad)


if __name__ == "__main__":
    test_dsl()
```

### 되돌리기와 반복의 실제 관측

선택 테스트는 `AB`를 시도하다 실패한 뒤 원래 위치에서 `AC`를 받아들이는지 확인한다.
반복 테스트는 빈 입력에서 성공하는 파서가 무한히 반복되지 않는지 확인한다.
문법의 정상 사례뿐 아니라 조합기 자체의 잘못된 사용을 검사한다.

---

## 10. Python의 표현 한계

### 재귀적인 문법 정의

Python에서 파서 객체를 서로 참조하는 재귀 문법은 생성 시점과 실행 시점을 구분해야 한다.
지연된 파서 참조가 필요할 수 있다.
좌재귀를 단순히 람다로 감쌌다고 종료 문제가 해결되는 것은 아니다.

### 타입의 강제 수준

외부에서 임의의 위치나 잘못된 결과 객체를 반환하는 파서를 만들 수 있다.
제네릭 주석이 입력 소비 법칙을 증명하지 않는다.
조합기의 공개 범위와 테스트, 계약을 명확히 한다.

### 성능

Python 함수 객체와 호출, 결과 래퍼에는 비용이 있다.
고정된 단순 형식에는 정규식이나 직접적인 루프가 더 적합할 수 있다.
가독성과 진단, 유지보수 요구를 실제 입력 규모와 함께 평가한다.

### 라이브러리와 예제의 차이

실제 파서 라이브러리는 더 풍부한 오류, 위치 정보, 지연 참조, 성능 정책을 제공할 수 있다.
이 예제는 기본 조합의 의미를 이해하기 위한 구현이다.
제품에서 필요한 모든 문법과 보안 요구를 충족했다고 확대하지 않는다.

---

## 11. 핵심 정리

### 핵심 결론

조합기는 작은 계산을 연결하여 더 큰 계산을 만드는 함수다.
파서 조합기에서는 의미값뿐 아니라 입력 위치와 실패 정책도 연결한다.
전체 입력 검사와 반복의 진행 조건을 명시해야 한다.
DSL의 문법, 의미, 오류와 자원 제한을 함께 설계한다.

### 연습 1: 전체 입력

`A:2junk`에서 앞의 `A:2`만 읽고 성공을 반환했다.
어떤 검사가 빠졌는가?

**해설.** 최상위 결과의 다음 위치가 입력 끝인지 확인해야 한다.
접두사 파싱의 성공을 전체 문서 파싱의 성공으로 바꾸지 않는다.
남은 문자의 의미를 계약으로 정한다.

### 연습 2: 빈 성공의 반복

항상 같은 값과 같은 위치를 반환하는 파서에 `many`를 적용했다.
무엇을 검사해야 하는가?

**해설.** 성공할 때 입력 위치가 실제로 전진하는지 확인해야 한다.
진행하지 않으면 오류로 처리하거나 반복을 금지한다.
성공 여부와 종료에 필요한 진전을 구분한다.

### 연습 3: 되돌리기와 저장

선택 분기 안의 파서가 성공값을 만드는 도중 외부 저장소에 썼다.
왜 위험한가?

**해설.** 뒤에서 실패하여 다른 분기를 시도해도 이미 수행한 쓰기는 남는다.
반복 시 같은 효과가 여러 번 발생할 수도 있다.
파싱에서는 의미값을 만들고 실제 실행은 별도 경계로 옮긴다.

### 연습 4: 좌재귀

`many`의 진행 검사만으로 모든 재귀 문법이 안전해지는가?

**해설.** 입력을 읽기 전에 자기 자신을 호출하는 좌재귀는 별도의 문제다.
문법을 바꾸거나 적절한 파싱 알고리즘을 사용해야 한다.
하나의 방어 조건을 모든 종료 보장으로 확대하지 않는다.

### 다음 부와 참고 자료

7부는 값과 타입, 효과를 프로그램 구조로 배치하는 여러 패턴을 비교했다.
8부는 평가 시점과 메모리 공유, 재귀의 구조, 제어 흐름, 데이터 접근과 시간에 따른 변화를 더 깊게
다룬다.

[Scala Parser Combinators 공식 저장소](https://github.com/scala/scala-parser-combinators)
[Scala 공식 문서: For Expressions](https://docs.scala-lang.org/scala3/book/control-structures.html)
[Python 공식 문서: 정규식](https://docs.python.org/3.14/library/re.html)
