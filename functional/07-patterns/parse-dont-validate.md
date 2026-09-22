# 45장. Parse, Don't Validate

![섞인 수확물에서 알곡만 새로운 상태로 가려내는 키](../../assets/images/fp/parse-dont-validate.png)

원시 문자열이 올바른 수량인지 검사한 뒤 같은 문자열을 그대로 다음 함수에 넘기면 무엇을 알게
되었는지 사라질 수 있다.
뒤의 함수는 다시 형식과 범위를 검사하거나 앞의 검사를 암묵적으로 믿어야 한다.
Parse, Don't Validate는 검사에서 얻은 정보를 더 정밀한 데이터와 타입으로 보존하자는 설계
원칙이다.

이번 장은 상품 코드, 수량, 단가 문자열을 유효한 주문 행으로 바꾼다.
파싱은 단순히 문자열을 숫자로 읽는 작업보다 넓은 의미로 사용된다.
다만 외부 상태의 변화와 권한처럼 계속 확인해야 하는 사실까지 한 번의 파싱으로 영구 보장할 수는
없다.

---

## 1. 개념과 기본 구분

### 검사 결과를 값에 남긴다

검증 함수가 `Boolean`만 반환하면 원래 입력은 여전히 넓은 타입이다.
파서는 실패 정보 또는 더 제한된 유효값을 반환한다.
성공 이후의 함수는 그 유효값이 가진 계약을 사용할 수 있다.

```text
validate: Raw -> Boolean
parse:    Raw -> Either[Error, ValidatedValue]
```

이 구분은 모든 검증 함수를 금지하자는 주장이 아니다.
검사로 얻은 정보를 버리지 않고 이후 코드가 사용할 수 있도록 표현하자는 방향이다.
구체적인 경계와 실패 정책은 프로그램의 요구에 따라 정한다.

### 정보의 증가

문자열에서 숫자로, 일반 숫자에서 양수 수량으로, 임의 목록에서 비어 있지 않은 주문으로 변환할 수
있다.
각 단계가 어떤 불변식을 추가했는지 타입과 생성 경계에 남긴다.
이전 장의 스마트 생성자와 잘못된 상태를 줄이는 데이터 모델링이 연결된다.

### 파싱과 정규화

앞뒤 공백 제거와 대소문자 변경 같은 정규화는 정보를 바꿀 수 있다.
허용하는 입력 범위와 저장할 표준 형태를 명시해야 한다.
사용자가 준 원문이 필요하면 별도 진단 데이터로 보관할 수 있다.

### 시간에 따라 바뀌는 사실

수량이 양수라는 구조적 사실과 현재 재고가 충분하다는 외부 사실은 다르다.
전자는 값의 생성 경계에서 보존하기 쉽다.
후자는 실행 시점의 상태가 바뀌면 다시 판단해야 할 수 있다.

---

## 2. 명령형 스타일과 함수형 스타일

### 확인한 뒤 원시값을 다시 전달

```text
if isPositiveInteger(rawQuantity):
    calculate(rawQuantity)
```

`calculate`가 여전히 문자열을 받으면 숫자 변환과 범위 검사를 다시 해야 할 수 있다.
앞의 검사를 잊은 호출자도 같은 함수를 부를 수 있다.
검사가 코드의 순서에만 의존하고 함수 계약에는 나타나지 않는다.

### 유효한 타입으로 변환

```text
parseQuantity(rawQuantity)
  실패 -> 구체적 입력 오류
  성공 -> Quantity
```

계산 함수는 `Quantity`를 받는다.
정상적인 생성 경로를 통해 값이 만들어졌다면 양수 범위를 다시 검사할 필요가 줄어든다.
타입의 생성 통제와 외부 역직렬화 경계가 그 보장의 일부다.

### 숫자 변환만으로 충분하지 않다

`"0"`을 정수 0으로 읽는 것은 기술적으로 성공할 수 있다.
하지만 주문 수량의 규칙에서는 실패일 수 있다.
문법 파싱과 도메인 범위 정제의 단계를 함께 설계한다.

### 기본값으로 실패를 숨기지 않는다

잘못된 수량을 1로 바꾸면 입력 오류가 정상 주문으로 바뀔 수 있다.
기본값이 의도된 정책인지 오류 은폐인지 구분한다.
실패 이유를 결과로 남기는 것이 호출자의 선택을 돕는다.

---

## 3. 왜 이 개념을 사용하는가?

### 내부 코드의 단순화

계산 함수가 유효한 값만 받도록 하면 반복적인 방어 코드가 줄어든다.
업무 계산의 전제조건을 타입에서 읽을 수 있다.
모든 계층이 같은 원시 문자열을 해석하는 문제를 피한다.

### 오류 위치의 명확성

입력 경계에서 필드와 오류 코드를 함께 반환한다.
화면이나 API 응답이 어느 값을 수정해야 하는지 알 수 있다.
도메인 계산 중 뒤늦게 일반적인 숫자 예외가 발생하는 상황을 줄인다.

### 모델의 확장성

Quantity와 UnitPrice를 다른 타입으로 두면 인자 순서를 바꾸는 실수를 줄일 수 있다.
원시 정수 두 개보다 의미가 분명하다.
타입을 늘리는 비용과 실제 오류 예방의 이득을 함께 고려한다.

### 신뢰 경계의 문서화

어디에서 외부 데이터를 내부 값으로 바꾸는지 명확해진다.
저장된 데이터와 캐시, 메시지 큐에서 읽는 데이터도 같은 경계 검토가 필요하다.
한 번 파싱했다는 과거 사실을 모든 미래 입력의 신뢰 근거로 사용하지 않는다.

### 테스트의 집중

허용 문법과 범위를 경계값으로 검사한다.
성공값을 받는 계산은 도메인 연산 자체에 집중해 테스트한다.
두 테스트 계층이 어떤 전제조건을 공유하는지 명시한다.

---

## 4. Scala에서의 표현

### Scala의 생성이 제한된 값

이 예제는 원시 폼 필드 세 개를 받는다.
숫자는 ASCII 숫자만 허용하며 입력 길이를 먼저 제한한다.
성공한 값은 별도의 클래스에 담아 계산 함수가 원시 문자열을 다시 해석하지 않게 한다.

<!-- executable:scala -->
```scala
object Chapter45:
  final case class ParseError(field: String, code: String)
  final case class RawLine(sku: String, quantity: String, unitPrice: String)

  final class Sku private (val value: String)
  object Sku:
    def parse(raw: String): Either[ParseError, Sku] =
      if raw.matches("[A-Z0-9][A-Z0-9-]{0,31}") then Right(new Sku(raw))
      else Left(ParseError("sku", "invalid_format"))

  final class Quantity private (val value: Int)
  object Quantity:
    def parse(raw: String): Either[ParseError, Quantity] =
      if raw.isEmpty || raw.length > 7 || !raw.forall(ch => ch >= '0' && ch <= '9') then
        Left(ParseError("quantity", "invalid_format"))
      else
        val value = raw.toInt
        if value >= 1 && value <= 1000000 then Right(new Quantity(value))
        else Left(ParseError("quantity", "out_of_range"))

  final class UnitPrice private (val value: BigInt)
  object UnitPrice:
    def parse(raw: String): Either[ParseError, UnitPrice] =
      if raw.isEmpty || raw.length > 13 || !raw.forall(ch => ch >= '0' && ch <= '9') then
        Left(ParseError("unitPrice", "invalid_format"))
      else
        val value = BigInt(raw)
        if value <= BigInt("1000000000000") then Right(new UnitPrice(value))
        else Left(ParseError("unitPrice", "out_of_range"))

  final case class Line(sku: Sku, quantity: Quantity, unitPrice: UnitPrice)

  def parseLine(raw: RawLine): Either[ParseError, Line] =
    for
      sku <- Sku.parse(raw.sku)
      quantity <- Quantity.parse(raw.quantity)
      price <- UnitPrice.parse(raw.unitPrice)
    yield Line(sku, quantity, price)

  def amount(line: Line): BigInt = line.unitPrice.value * line.quantity.value

  def check(): Unit =
    val valid = parseLine(RawLine("A-1", "2", "1000"))
    assert(valid.map(amount) == Right(BigInt(2000)))
    assert(valid.map(_.sku.value) == Right("A-1"))
    assert(parseLine(RawLine("bad", "2", "1000")) == Left(ParseError("sku", "invalid_format")))
    assert(parseLine(RawLine("A", "0", "1000")) == Left(ParseError("quantity", "out_of_range")))
    assert(parseLine(RawLine("A", "1000001", "1000")) == Left(ParseError("quantity", "out_of_range")))
    assert(parseLine(RawLine("A", "2.0", "1000")) == Left(ParseError("quantity", "invalid_format")))
    assert(parseLine(RawLine("A", "２", "1000")) == Left(ParseError("quantity", "invalid_format")))
    assert(parseLine(RawLine("A", "2", "-1")) == Left(ParseError("unitPrice", "invalid_format")))
    assert(parseLine(RawLine("A", "2", "1000000000001")) == Left(ParseError("unitPrice", "out_of_range")))
    assert(parseLine(RawLine("A", "0002", "0")).map(amount) == Right(BigInt(0)))
    assert(Quantity.parse("1000000").map(_.value) == Right(1000000))
    assert(Quantity.parse("9" * 10000) == Left(ParseError("quantity", "invalid_format")))
    val maximum = parseLine(RawLine("A", "1000000", "1000000000000"))
    assert(maximum.map(amount) == Right(BigInt("1000000000000000000")))
```

### 문법과 범위의 분리

수량 문자열은 길이를 제한한 뒤 정수로 변환하므로 예제 범위에서 변환 오버플로를 피한다.
그다음 도메인의 상한을 검사한다.
기술적인 파싱 성공과 도메인 유효성을 단계별로 확인한다.

### 정규화 정책

`"0002"`는 숫자 2로 받아들이므로 원래의 앞자리 0은 유효값에 남지 않는다.
공백이나 전각 숫자는 이 예제에서 허용하지 않는다.
이는 선택한 입력 계약이며 모든 서비스에 동일한 문법을 강제하는 규칙이 아니다.

---

## 5. 상태 변경보다 값 변환

### 넓은 값에서 좁은 값으로

입력 경계는 가능한 값의 범위를 줄인다.
원시 문자열은 임의의 내용을 가질 수 있지만 Quantity는 정해진 수량 범위를 표현한다.
성공값을 사용하는 함수는 그 정제된 계약을 사용한다.

```mermaid
flowchart LR
    A["원시 폼 문자열"] --> B["문법과 길이 검사"]
    B --> C["숫자 변환과 범위 검사"]
    C --> D["Sku · Quantity · UnitPrice"]
    D --> E["유효한 Line"]
    E --> F["반복 파싱 없는 금액 계산"]
```

이 흐름에서 실패가 발생하면 더 좁은 값을 만들지 않는다.
유효하지 않은 원시값을 정상 타입의 내부 필드로 조용히 넣지 않는다.
성공·실패를 결과 타입으로 분리한다.

### 유효값의 수명

불변 Quantity가 만들어진 뒤 값 자체가 바뀌지 않으면 구조적 범위 조건을 유지할 수 있다.
하지만 외부 규칙의 버전이 바뀌면 허용 상한의 의미가 달라질 수 있다.
정책 버전과 값의 해석 범위를 필요한 경우 기록한다.

### 직렬화 이후의 경계

도메인 값을 저장하거나 전송했다가 다시 읽으면 새로운 입력 경계가 생긴다.
과거에 유효했더라도 현재 스키마와 무결성을 다시 확인해야 할 수 있다.
타입 이름을 JSON 필드에 넣는 것만으로 생성 규칙이 보장되지는 않는다.

---

## 6. 함수 합성과 데이터 흐름

### 작은 파서의 합성

각 필드의 파서는 원시값에서 오류 또는 유효값을 만든다.
의존적인 연결은 `Either`로, 독립 오류의 누적은 Validation으로 조합할 수 있다.
이 원칙이 반드시 첫 오류만 반환해야 한다는 뜻은 아니다.

```text
RawSku      -> Either[Error, Sku]
RawQuantity -> Either[Error, Quantity]
RawPrice    -> Either[Error, UnitPrice]
```

### 검증 결과를 버리지 않는다

`isValid`를 호출한 뒤 원시값을 계속 전달하는 대신 파싱 결과를 사용한다.
이렇게 하면 검사를 우회한 일반 호출 경로를 줄일 수 있다.
생성자 접근 통제와 모듈 경계가 실제 보장을 뒷받침해야 한다.

### 도메인 관계의 파싱

개별 필드가 유효해도 시작 시각이 종료 시각보다 늦을 수 있다.
여러 유효값을 받아 관계를 검사한 새로운 도메인 값을 만들 수 있다.
파싱과 정제는 한 번의 숫자 변환에만 한정되지 않는다.

### 외부 사실의 확인

상품 코드의 형식이 유효하다는 사실과 그 상품이 존재한다는 사실은 다르다.
존재·권한·재고는 필요한 시점의 외부 정보를 사용해 판단한다.
값 정제와 실행 시점의 확인을 구분한다.

---

## 7. 장점과 트레이드오프

### 장점과 트레이드오프

| 선택 | 이점 | 주의점 |
| --- | --- | --- |
| 정제된 타입 | 전제조건 가시화 | 타입과 어댑터 증가 |
| 생성 경계 통제 | 잘못된 값의 유입 감소 | 우회 경로 검토 |
| 필드별 오류 | 수정 위치 명확 | 오류 정책의 일관성 |
| 정규화 | 내부 표현 통일 | 원문 정보 손실 |
| 파서 합성 | 재사용과 단계 분리 | 의존성·누적 정책 |

### 타입의 과도한 세분화

모든 임시 정수에 새 클래스를 만들면 코드의 탐색 비용이 커질 수 있다.
의미 혼동과 유효성 오류가 실제로 발생하는 경계부터 정제한다.
기계적인 래퍼 증가보다 불변식의 명확성이 중요하다.

### 변경되는 정책

수량 상한이 고객 등급이나 날짜에 따라 달라지면 하나의 전역 Quantity 타입만으로 설명하기 어려울
수 있다.
기본 구조 조건과 요청별 정책 판단을 분리한다.
정제된 값이 무엇을 보장하는지 구체적으로 적는다.

### 입력 길이와 비용

숫자 형식이 맞더라도 매우 긴 문자열은 큰 자원을 사용할 수 있다.
변환 전에 길이와 전체 입력 크기를 제한한다.
유효성은 논리 조건뿐 아니라 처리 가능한 자원 범위도 고려해야 한다.

---

## 8. 상태와 부수효과의 경계

### 신뢰 경계

폼, JSON, 메시지 큐, 데이터베이스 행은 각각 외부 입력 경계일 수 있다.
어디에서 어떤 스키마와 불변식을 확인하는지 정한다.
내부 타입을 그대로 역직렬화하는 도구가 생성자를 우회하는지도 확인해야 한다.

### 권한과 인증

유효한 상품 코드와 수량을 가졌다고 사용자가 구매할 권한이 생기는 것은 아니다.
인증 상태와 권한은 별도의 입력과 실행 정책이다.
파싱 성공을 보안 승인과 동일시하지 않는다.

### 재고와 동시성

양수 수량의 유효성은 안정적일 수 있지만 현재 재고는 변할 수 있다.
실제 예약 시 원자적인 확인이 필요하다.
한 번 파싱했다는 이유로 모든 이후 검사를 제거하지 않는다.

### 오류 공개

파싱 실패의 원문에 개인정보나 토큰이 들어갈 수 있다.
오류 경로와 코드만으로 충분한 경우 원문 전체를 로그에 남기지 않는다.
사용자 안내와 내부 진단을 분리한다.

---

## 9. Python에서 적용하기

### Python의 유효값과 폼 경계

Python에서는 타입 주석만으로 생성자를 제한할 수 없으므로 정상 생성 경로에서도 불변식을 확인한다.
아래 구현은 폼의 필드가 정확히 세 개의 문자열인지 확인한 뒤 각 값을 파싱한다.
생성자 검사는 외부 객체 조작을 막는 보안 격리 장치가 아니다.

<!-- executable:python -->
```python
from dataclasses import dataclass
import re


@dataclass(frozen=True)
class ParseError:
    field: str
    code: str


@dataclass(frozen=True)
class Sku:
    value: str

    def __post_init__(self) -> None:
        if type(self.value) is not str or re.fullmatch(r"[A-Z0-9][A-Z0-9-]{0,31}", self.value) is None:
            raise ValueError("invalid SKU")


@dataclass(frozen=True)
class Quantity:
    value: int

    def __post_init__(self) -> None:
        if type(self.value) is not int or not 1 <= self.value <= 1_000_000:
            raise ValueError("invalid quantity")


@dataclass(frozen=True)
class UnitPrice:
    value: int

    def __post_init__(self) -> None:
        if type(self.value) is not int or not 0 <= self.value <= 1_000_000_000_000:
            raise ValueError("invalid price")


@dataclass(frozen=True)
class Line:
    sku: Sku
    quantity: Quantity
    unit_price: UnitPrice


def parse_digits(raw: str, field: str, max_length: int, minimum: int, maximum: int) -> int | ParseError:
    if not 1 <= len(raw) <= max_length or any(character < "0" or character > "9" for character in raw):
        return ParseError(field, "invalid_format")
    value = int(raw)
    return value if minimum <= value <= maximum else ParseError(field, "out_of_range")


def parse_line(fields: dict[str, object]) -> Line | ParseError:
    expected = {"sku", "quantity", "unitPrice"}
    if set(fields) != expected:
        return ParseError("line", "invalid_fields")
    if any(type(fields[key]) is not str for key in expected):
        return ParseError("line", "expected_strings")
    sku, raw_quantity, raw_price = fields["sku"], fields["quantity"], fields["unitPrice"]
    assert isinstance(sku, str) and isinstance(raw_quantity, str) and isinstance(raw_price, str)
    if re.fullmatch(r"[A-Z0-9][A-Z0-9-]{0,31}", sku) is None:
        return ParseError("sku", "invalid_format")
    quantity = parse_digits(raw_quantity, "quantity", 7, 1, 1_000_000)
    if isinstance(quantity, ParseError):
        return quantity
    price = parse_digits(raw_price, "unitPrice", 13, 0, 1_000_000_000_000)
    if isinstance(price, ParseError):
        return price
    return Line(Sku(sku), Quantity(quantity), UnitPrice(price))


def amount(line: Line) -> int:
    return line.quantity.value * line.unit_price.value


def test_parse_and_use() -> None:
    valid = parse_line({"sku": "A-1", "quantity": "2", "unitPrice": "1000"})
    assert isinstance(valid, Line) and amount(valid) == 2000
    assert parse_line({"sku": "bad", "quantity": "2", "unitPrice": "1000"}) == ParseError("sku", "invalid_format")
    for raw in ("0", "1000001"):
        assert parse_line({"sku": "A", "quantity": raw, "unitPrice": "1000"}) == ParseError("quantity", "out_of_range")
    for raw in ("2.0", "２", "9" * 10_000):
        assert parse_line({"sku": "A", "quantity": raw, "unitPrice": "1000"}) == ParseError("quantity", "invalid_format")
    assert parse_line({"sku": "A", "quantity": True, "unitPrice": "1000"}) == ParseError("line", "expected_strings")
    assert parse_line({"sku": "A"}) == ParseError("line", "invalid_fields")
    normalized = parse_line({"sku": "A", "quantity": "0002", "unitPrice": "0"})
    assert isinstance(normalized, Line) and normalized.quantity.value == 2 and amount(normalized) == 0
    maximum = parse_line({"sku": "A", "quantity": "1000000", "unitPrice": "1000000000000"})
    assert isinstance(maximum, Line) and amount(maximum) == 1_000_000_000_000_000_000
    try:
        Quantity(0)
    except ValueError:
        pass
    else:
        raise AssertionError("invalid quantity constructed")


if __name__ == "__main__":
    test_parse_and_use()
```

### 내부 단언의 위치

필드 타입을 명시적으로 검사한 뒤 타입 좁힘을 보조하는 단언을 사용한다.
사용자 입력 검증 자체를 `assert`에만 맡긴 것이 아니다.
최적화 옵션으로 단언이 제거되어도 앞의 정상 검증 경로가 남아야 한다.

---

## 10. Python의 표현 한계

### 생성자 통제의 차이

Python의 관례적인 비공개 이름과 frozen 클래스는 강력한 보안 경계가 아니다.
직접적인 객체 조작이나 특수한 역직렬화가 일반 생성 규칙을 우회할 수 있다.
지원하는 호출 경로와 신뢰 경계를 명시한다.

### 타입 주석의 실행 의미

`Line` 필드에 Quantity라고 적어도 실행기가 모든 호출에서 자동으로 확인하지 않는다.
내부 코드는 정적 검사와 정상 생성 경로를 전제로 할 수 있다.
외부 입력을 받는 경계에서는 실제 검사가 필요하다.

### `bool`과 정수

Python에서는 불리언과 정수의 관계 때문에 넓은 정수 검사가 의도보다 많은 값을 허용할 수 있다.
예제는 수량과 단가에서 정확한 정수 타입을 요구한다.
도메인 계약에 맞는 검사를 선택한다.

### 재파싱의 필요

파일과 메시지에서 다시 읽은 데이터는 새 신뢰 경계를 통과한다.
이미 과거에 유효한 객체였다는 이유로 현재 입력을 무조건 신뢰하지 않는다.
스키마 버전과 저장 무결성을 확인한다.

---

## 11. 핵심 정리

### 핵심 결론

Parse, Don't Validate는 검사에서 얻은 정보를 유효한 값과 타입에 보존하자는 원칙이다.
원시 입력을 계속 전달하는 대신 정제된 값을 계산에 사용한다.
파싱 문법과 정규화, 범위, 오류 정책을 명시해야 한다.
외부 상태와 권한의 변화까지 한 번의 파싱으로 영구 보장할 수는 없다.

### 연습 1: 불리언 검증

양수 여부를 검사한 뒤 원래 문자열을 계산 함수에 넘긴다.
어떤 정보가 함수 계약에 남지 않는가?

**해설.** 문자열이 숫자로 읽히고 양수 범위를 만족한다는 사실이 타입에 남지 않는다.
유효한 Quantity를 반환하고 계산 함수가 그것을 받게 할 수 있다.
검사 결과를 값에 보존하는 것이 핵심이다.

### 연습 2: 정규화

앞자리 0을 허용하면서 정수로 변환했다.
원래 문자열을 그대로 복원할 수 있는가?

**해설.** 정수값만으로 앞자리 0의 개수를 알 수 없다.
정규화가 정보를 잃는다는 사실을 계약에 포함해야 한다.
원문이 필요하면 별도의 진단 데이터로 보관한다.

### 연습 3: 재고 검증

Quantity가 유효하므로 실제 예약 시 재고를 다시 확인하지 않아도 되는가?

**해설.** 양수 범위와 현재 재고의 충분함은 다른 사실이다.
외부 상태는 바뀔 수 있으므로 실행 시점의 원자적인 판단이 필요하다.
구조적 유효성과 동적인 사실을 구분한다.

### 연습 4: 모든 오류

Parse, Don't Validate를 따르면 반드시 첫 오류만 반환해야 하는가?

**해설.** 아니다. 유효값을 만드는 목표와 오류 누적 정책은 별개다.
독립 필드 파서를 Validation으로 결합하여 여러 오류를 보존할 수 있다.
의존 관계와 사용자 경험에 맞는 조합을 선택한다.

### 다음 장과 참고 자료

다음 장은 성공과 실패 경로를 명시적으로 연결하는 Railway Oriented Programming을 다룬다.
파싱, 도메인 판단, 외부 실행의 실패를 어떻게 이어 갈지 설계한다.

[Alexis King: Parse, Don't Validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
[Cats 공식 문서: Validated](https://typelevel.org/cats/datatypes/validated.html)
[Python 공식 문서: dataclasses](https://docs.python.org/3.14/library/dataclasses.html)
