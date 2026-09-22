# 5장. First-Class Functions

지금까지 함수는 입력값을 계산하는 도구였다.
이번 장에서는 함수 자체를 다른 값처럼 보관하고 전달한다.
할인 정책을 문자열 분기 안에 숨기는 대신 정책 함수를 선택해서 계산에 넘기는 방식으로 사고를
확장한다.

일급 함수는 함수형 프로그래밍만의 전유물이 아니다.
이벤트 처리, 정렬 기준, 테스트 대역, 라우팅에서도 같은 기반을 사용한다.
중요한 것은 람다 문법의 짧음이 아니라 동작을 값으로 취급하는 능력이다.

---

## 1. 개념과 기본 구분

### 일급이라는 뜻

어떤 언어 요소가 값처럼 변수에 저장되고 인자로 전달되며 결과로 반환될 수 있으면 일급으로
다룬다고 말한다.
함수가 이런 위치에 올 수 있으면 일급 함수다.
함수의 이름을 알고 직접 호출하는 기능만으로는 이 설계의 의미를 다 설명하지 못한다.

```text
동작을 정의한다
  -> 동작을 값으로 선택한다
  -> 선택한 값을 전달한다
  -> 전달받은 곳에서 호출한다
```

함수를 전달하는 시점과 실행하는 시점은 다르다.
`discount`를 전달하는 것과 `discount(amount)`의 결과를 전달하는 것을 구분해야 한다.
후자는 이미 실행된 계산의 결과다.

### 함수와 함수 호출 결과

```scala
val double: Int => Int = x => x * 2
val functionValue = double
val calculatedValue = double(10)
```

`functionValue`의 타입은 함수 타입이다.
`calculatedValue`의 타입은 정수다.
이 차이를 놓치면 콜백 등록 시점에 함수를 실행하는 버그가 생긴다.

### 메서드와 함수값

Scala의 `def`는 메서드를 정의하고 함수값은 `A => B` 타입으로 표현한다.
메서드를 함수가 필요한 위치에서 함수값으로 바꾸는 변환을 eta expansion이라고 부른다.
Python에서는 함수 정의가 호출 가능한 함수 객체를 바인딩한다.
언어별 모델은 다르지만 동작을 전달한다는 설계 목적은 비교할 수 있다.

### 일급 함수와 고차 함수

일급 함수는 함수를 값으로 다룰 수 있는 언어의 능력이다.
고차 함수는 실제로 함수를 인자로 받거나 반환하는 함수다.
다음 장은 그 능력을 사용하는 API의 설계를 다룬다.
이 장은 먼저 함수값의 저장, 선택, 호출을 분리해서 익힌다.

---

## 2. 명령형 스타일과 함수형 스타일

### 문자열 분기에 묶인 정책

할인 정책을 하나의 함수 안에서 문자열로 분기할 수 있다.
정책이 늘어날수록 계산과 선택의 책임이 같은 위치에 쌓인다.

```scala
def discounted(amount: Long, tier: String): Long =
  if tier == "VIP" then amount - amount / 10
  else if tier == "STAFF" then amount / 2
  else amount
```

이 코드가 항상 나쁜 것은 아니다.
정책이 적고 변하지 않으면 명확한 분기가 더 간단할 수 있다.
그러나 정책의 선택과 사용을 여러 장소에서 반복하면 중복과 결합이 생긴다.

### 동작을 값으로 분리

```scala
val vip: Long => Long = amount => amount - amount / 10
val staff: Long => Long = amount => amount / 2
val regular: Long => Long = amount => amount

val selected = vip
val payable = selected(10000)
```

할인 계산은 정책 이름을 알 필요가 없다.
선택 계층은 상황에 맞는 함수값을 고르고 계산 계층은 그것을 사용한다.
새 정책이 들어와도 선택과 계산의 변경 범위를 나눌 수 있다.

### 객체 전략과의 관계

전략 객체가 메서드 하나만 제공하고 별도 상태가 없다면 함수 하나로 표현할 수 있다.
반대로 여러 관련 연산과 수명 관리가 필요하면 객체나 레코드가 더 명확할 수 있다.
일급 함수를 배웠다고 모든 인터페이스를 함수로 바꿀 필요는 없다.
설계의 책임 크기와 계약을 기준으로 선택한다.

---

## 3. 왜 이 개념을 사용하는가?

### 선택과 실행의 분리

정책을 값으로 만들면 어떤 정책을 선택했는지 먼저 확인할 수 있다.
선택한 정책을 여러 주문에 적용할 수도 있다.
이때 정책 함수가 숨은 상태를 읽는다면 선택 당시와 실행 당시의 의미가 달라질 수 있다.
일급이라는 성질이 순수성을 보장하지는 않는다.

### 테스트 대역

실제 정책 대신 고정된 결과를 반환하는 함수를 전달하여 호출 경로를 검사할 수 있다.
다만 대역이 실제 정책의 계약을 지키는지도 확인해야 한다.
아무 값이나 반환하는 가짜 함수는 테스트를 통과시켜도 설계 오류를 숨길 수 있다.

### 중복 제거

여러 화면이나 서비스가 같은 정책 함수를 재사용할 수 있다.
공통된 것은 정책의 동작이며 문자열 분기 전체가 아닐 수 있다.
변하는 부분만 값으로 전달하면 재사용의 단위가 작아진다.

### 이름의 중요성

짧은 람다는 간단한 변환에 적절하다.
업무 규칙이 복잡하면 이름 있는 함수가 의도를 더 잘 전달한다.
함수값을 저장할 수 있다는 사실과 모든 함수를 익명으로 써야 한다는 주장은 다르다.

```text
좋은 이름: applyVipDiscount
모호한 이름: f, handler, process
```

이름은 호출 위치에서 정책의 의미를 설명해야 한다.
특히 같은 함수 타입을 가진 서로 다른 업무 규칙을 구별할 때 중요하다.

---

## 4. Scala에서의 표현

### 정책을 저장하고 선택하는 Scala 프로그램

다음 예제는 표준 라이브러리만 사용한다.
함수값은 불변 맵에 저장되며 알 수 없는 정책은 `None`으로 나타낸다.
문자열 입력을 조용히 기본 정책으로 처리하지 않는 선택이다.

<!-- executable:scala -->
```scala
object Chapter05:
  type Policy = Long => Long

  def regular(amount: Long): Long = amount
  def vip(amount: Long): Long = amount - amount / 10
  def staff(amount: Long): Long = amount / 2

  val policies: Map[String, Policy] = Map(
    "REGULAR" -> regular,
    "VIP" -> vip,
    "STAFF" -> staff
  )

  def choose(name: String): Option[Policy] =
    policies.get(name)

  def applyPolicy(amount: Long, policy: Policy): Long =
    require(amount >= 0)
    val result = policy(amount)
    require(result >= 0 && result <= amount)
    result

  def namedPolicy(
    name: String,
    policy: Policy
  ): (String, Policy) =
    (name, policy)

  def check(): Unit =
    val selected: Policy = vip
    val result: Long = selected(10000)
    assert(result == 9000)
    assert(applyPolicy(10000, regular) == 10000)
    assert(applyPolicy(10000, staff) == 5000)
    assert(choose("missing").isEmpty)
    assert(choose("VIP").map(p => p(10000)).contains(9000))

    val catalog = List(
      namedPolicy("regular", regular),
      namedPolicy("vip", vip),
      namedPolicy("staff", staff)
    )
    val amounts = catalog.map { case (_, policy) =>
      applyPolicy(10000, policy)
    }
    assert(amounts == List(10000, 9000, 5000))

    for amount <- List(0L, 1L, 9L, 10L, 999L, 10000L) do
      for (_, policy) <- catalog do
        val payable = applyPolicy(amount, policy)
        assert(payable >= 0 && payable <= amount)
```

### 함수 타입의 정보와 빈틈

`Long => Long`은 입력과 출력이 정수라는 사실을 나타낸다.
반환값이 원금 이하라는 업무 규칙까지 표현하지는 않는다.
그래서 예제의 호출 경계에서 결과 범위를 검사했다.

또한 함수 타입만으로 로그 출력, 전역 상태 변경, 예외 발생을 배제하지 못한다.
실제 정책의 계약은 타입, 구현, 테스트를 함께 읽어야 한다.
타입이 같다고 의미가 같은 함수는 아니다.

### 선택 실패의 표현

알 수 없는 정책 이름은 `Option[Policy]`다.
이 책의 순서상 `Option`을 아직 깊게 배우지 않았으므로 “있거나 없음”으로 이해하면 충분하다.
4부에서 부재를 합성하는 방법을 자세히 다룬다.

---

## 5. 상태 변경보다 값 변환

### 함수값은 계산의 선택을 보존한다

정책을 고른 뒤 함수값을 보관하면 선택 결과를 다음 단계로 전달할 수 있다.
정책 이름과 함수값을 함께 담은 레코드로 감사 정보를 유지할 수도 있다.
그러나 함수값 자체를 저장소에 직렬화할 수 있다고 가정해서는 안 된다.

```mermaid
flowchart LR
    A["정책 이름"] --> B["정책 선택"]
    B --> C["함수값"]
    C --> D["금액에 적용"]
    D --> E["결제 예정 금액"]
```

함수값을 전달하는 것은 원금 데이터를 변경하는 일이 아니다.
선택된 계산을 명시적인 입력으로 만드는 일이다.
이런 분리는 의존성 주입 장에서 일반화된다.

### 정책의 버전

오래된 견적을 재현하려면 정책 이름만으로는 부족할 수 있다.
같은 이름의 정책 구현이 바뀌기 때문이다.
필요한 경우 정책 버전과 입력 데이터를 함께 보관한다.
함수 객체의 메모리 주소를 업무상의 정책 식별자로 쓰는 것은 적절하지 않다.

### 환경을 가진 함수

할인율을 캡처한 함수는 동작과 환경을 함께 담을 수 있다.
환경이 불변 스냅샷이면 의미를 추적하기 쉽다.
환경이 가변 설정 객체이면 실행 시점의 값에 영향을 받는다.
클로저 장에서 이 차이를 자세히 살펴본다.

---

## 6. 함수 합성과 데이터 흐름

### 함수값을 다시 함수에 전달

함수값은 합성의 재료다.
하나의 정책을 선택한 뒤 표시 함수와 연결할 수 있다.
이때 두 함수의 입출력 타입이 이어져야 한다.

```scala
val discount: Long => Long = amount => amount - amount / 10
val render: Long => String = amount => s"$amount KRW"
val quoteLabel: Long => String = discount.andThen(render)
```

`discount`는 금액을 금액으로 바꾸고 `render`는 금액을 문자열로 바꾼다.
합성된 함수의 타입은 금액에서 문자열로 가는 타입이다.
함수를 값으로 다루지 못하면 이런 조립을 매번 새 래퍼 코드로 작성해야 한다.

### 목록의 원소로 보관

정책 함수를 목록에 담아 같은 입력에 각각 적용할 수 있다.
다만 이것은 정책들을 순서대로 누적 적용하는 것과 다르다.
각 정책이 최초 원금을 받는지 앞 정책의 결과를 받는지 명시해야 한다.

```text
비교:  정책 A(원금), 정책 B(원금)
누적:  정책 B(정책 A(원금))
```

두 구조는 세금과 할인처럼 순서가 중요한 업무에서 다른 결과를 낸다.
함수값의 컬렉션이라는 자료구조만 보고 업무 의미를 추측하지 않는다.

---

## 7. 장점과 트레이드오프

### 단순한 전략 표현

함수 하나로 충분한 변동점에서는 클래스 계층을 줄일 수 있다.
호출자가 필요한 동작을 직접 전달하여 테스트와 조합이 쉬워진다.
다만 함수 이름과 타입만으로 부족한 계약이 있다면 별도 데이터 모델을 함께 사용한다.

| 상황 | 함수값이 적합한 이유 | 다른 표현을 검토할 이유 |
| --- | --- | --- |
| 단일 할인 규칙 | 입력에서 출력으로 명확함 | 여러 관련 정책 메서드 |
| 정렬 기준 | 작은 키 계산 | 상태ful 비교기와 부수효과 |
| 테스트 대역 | 동작을 직접 교체 | 실제 계약과의 차이 |
| 이벤트 콜백 | 호출 시점 분리 | 구독 해제와 수명 관리 |
| 정책 카탈로그 | 선택과 실행 분리 | 직렬화와 버전 추적 |

### 디버깅 비용

익명 함수가 여러 층으로 감싸지면 호출 스택에서 의도를 찾기 어려울 수 있다.
핵심 업무 규칙은 이름 있는 함수로 정의하는 편이 좋다.
메타데이터가 필요하면 이름과 함수를 함께 담은 작은 레코드를 사용할 수 있다.

### 과도한 매개변수화

변하지 않는 모든 연산을 함수 인자로 만들면 호출자가 알아야 할 것이 늘어난다.
실제로 교체되거나 테스트 경계가 필요한 동작부터 분리한다.
동작을 값으로 표현할 수 있다는 능력은 모든 곳에서 사용해야 한다는 의무가 아니다.

---

## 8. 상태와 부수효과의 경계

### 등록과 실행 사이의 시간

이벤트 처리기에 함수를 등록하는 순간에는 함수가 실행되지 않을 수 있다.
나중에 호출될 때 필요한 환경이 여전히 유효한지 확인해야 한다.
닫힌 파일 핸들이나 종료된 세션을 캡처하면 호출 시점에 실패할 수 있다.

### 권한을 가진 함수

파일 쓰기 함수를 전달하면 받는 쪽은 그 동작을 실행할 능력을 얻는다.
함수값은 단순한 계산뿐 아니라 외부 효과를 수행하는 권한을 운반할 수 있다.
필요한 동작만 좁게 전달하면 의존성 범위를 줄이는 데 도움이 된다.
하지만 보안 격리가 자동으로 생기는 것은 아니다.

### 호출 횟수의 계약

함수를 받는 API는 그것을 몇 번 호출할 수 있는지 설명해야 한다.
정렬 키, 재시도 콜백, 필터 조건은 호출 횟수와 순서의 기대가 다를 수 있다.
효과가 있는 함수를 전달할 때 이 차이가 중요해진다.
고차 함수 장에서는 이 계약을 예제로 확인한다.

### 직렬화의 경계

함수값을 프로세스 밖으로 전송하거나 데이터베이스에 저장하는 일은 일반 데이터 저장과 다르다.
업무에서는 정책 식별자와 버전, 필요한 인자를 저장하고 실행 시 해석하는 설계가 더 적절할 수 있다.
프로그램을 데이터로 표현하는 방법은 후반부의 해석기 패턴으로 이어진다.

---

## 9. Python에서 적용하기

### Python 함수와 호출 가능한 객체

Python에서는 함수뿐 아니라 `__call__`을 정의한 객체도 호출할 수 있다.
아래 예제는 이름 있는 함수, 람다, 호출 가능한 객체를 같은 계약으로 사용한다.
객체를 사용한 예는 함수형 설계가 객체를 금지하지 않는다는 점도 보여 준다.

<!-- executable:python -->
```python
from collections.abc import Callable
from dataclasses import dataclass

Policy = Callable[[int], int]


def regular(amount: int) -> int:
    return amount


def vip(amount: int) -> int:
    return amount - amount // 10


@dataclass(frozen=True)
class FixedDiscount:
    reduction: int

    def __call__(self, amount: int) -> int:
        return max(0, amount - self.reduction)


POLICIES: dict[str, Policy] = {
    "REGULAR": regular,
    "VIP": vip,
    "STAFF": lambda amount: amount // 2,
}


def choose(name: str) -> Policy | None:
    return POLICIES.get(name)


def apply_policy(amount: int, policy: Policy) -> int:
    if amount < 0:
        raise ValueError("negative amount")
    result = policy(amount)
    if not 0 <= result <= amount:
        raise ValueError("policy violated amount contract")
    return result


def test_function_and_result() -> None:
    selected = vip
    calculated = vip(10_000)
    assert callable(selected)
    assert selected(10_000) == 9000
    assert calculated == 9000
    assert not callable(calculated)


def test_catalog() -> None:
    selected = choose("VIP")
    assert selected is not None
    assert apply_policy(10_000, selected) == 9000
    assert choose("missing") is None
    assert apply_policy(10_000, regular) == 10_000
    assert apply_policy(10_000, POLICIES["STAFF"]) == 5000


def test_callable_object() -> None:
    policy = FixedDiscount(700)
    assert apply_policy(1000, policy) == 300
    assert apply_policy(300, policy) == 0
    assert policy.reduction == 700


def test_policy_contract() -> None:
    for amount in (0, 1, 9, 10, 999, 10_000):
        for policy in POLICIES.values():
            result = apply_policy(amount, policy)
            assert 0 <= result <= amount
    try:
        apply_policy(100, lambda amount: amount + 1)
    except ValueError:
        pass
    else:
        raise AssertionError("invalid policy result accepted")


if __name__ == "__main__":
    test_function_and_result()
    test_catalog()
    test_callable_object()
    test_policy_contract()
```

### 호출 가능한 객체를 쓴 이유

고정 할인액은 정책의 환경이다.
frozen 데이터 클래스에 그 값을 보관하면 설정을 읽기 쉽다.
동일한 동작을 클로저로도 표현할 수 있으며 어느 쪽이 더 명확한지는 필요한 메타데이터에 달려 있다.

이 예제의 정책 딕셔너리는 기술적으로 가변이다.
모듈 외부에서 정책 등록을 허용할지 여부는 별도의 공개 API 계약으로 정해야 한다.
동적 등록이 필요 없으면 읽기 전용 매핑으로 노출하는 방법도 있다.

---

## 10. Python의 표현 한계

### `Callable`이 표현하지 않는 것

`Callable[[int], int]`는 호출 형태를 설명한다.
반환값의 범위, 순수성, 호출 비용, 재진입 가능성까지 보장하지 않는다.
따라서 예제는 업무 계약을 런타임 검사와 테스트로 보완했다.

### 함수의 동일성

두 함수가 모든 입력에서 같은 결과를 내는지 일반적으로 자동 비교할 수 없다.
함수 객체의 `==`나 `is`를 정책의 의미상 동등성 검사로 사용하지 않는다.
필요하면 정책 식별자와 버전 같은 별도 데이터를 정의한다.

### 정적 검사와 실행

타입 주석은 런타임에 호출 규약을 자동 검증하지 않는다.
잘못된 인자 개수나 잘못된 결과 타입은 실행 시점에 드러날 수 있다.
검사 도구를 사용하더라도 외부 입력과 동적 등록 경계는 별도로 검토해야 한다.

### 직렬화의 제한

일반적인 람다와 클로저를 어디서나 안전하게 직렬화할 수 있다고 가정하지 않는다.
특정 라이브러리의 동작은 배포 환경과 코드 버전에 의존할 수 있다.
장기 보관할 업무 데이터는 실행 가능한 코드 객체보다 명시적인 정책 데이터를 우선 검토한다.

---

## 11. 핵심 정리

### 핵심 결론

함수를 저장하고 전달하고 반환할 수 있으면 동작을 값으로 다룰 수 있다.
함수값을 전달하는 것과 실행 결과를 전달하는 것은 다르다.
일급 함수는 순수성이나 직렬화 가능성을 자동 보장하지 않는다.
작은 전략은 함수로, 여러 책임과 수명은 더 명시적인 구조로 표현할 수 있다.

### 연습 1: 괄호 하나의 차이

콜백 등록 함수에 `on_click(save())`와 `on_click(save)`를 전달하는 차이를 설명하라.

**해설.** 첫 표현식은 등록 전에 `save`를 실행하고 그 결과를 넘긴다.
두 번째는 함수값을 넘겨 나중에 실행하게 한다.
등록 API가 기대하는 타입과 실행 시점을 확인해야 한다.

### 연습 2: 정책의 계약

`int -> int` 타입이 같은 두 정책 가운데 하나는 원금을 늘린다.
타입만으로 그 오류를 잡지 못하는 이유와 보완 방법을 설명하라.

**해설.** 함수 타입은 값의 범위 관계까지 표현하지 않는다.
정책 결과의 범위를 검사하거나 더 강한 도메인 타입과 법칙 테스트를 사용한다.
어떤 정책이 할인이고 어떤 정책이 세금인지 이름과 계약도 구분한다.

### 연습 3: 함수와 객체

할인 계산 하나만 있는 전략과 준비·실행·정리 세 동작이 있는 전략을 비교하라.
각각 어떤 표현이 더 자연스러운가?

**해설.** 단일 계산은 함수 하나로 충분할 수 있다.
세 동작이 같은 자원을 공유하면 객체나 자원 범위를 가진 구조가 계약을 더 잘 드러낸다.
함수형 설계는 객체 사용 여부보다 책임과 효과의 경계를 중시한다.

### 연습 4: 정책 보관

오늘 선택한 정책을 1년 뒤 다시 실행해야 한다.
함수 객체만 저장하는 설계의 문제를 설명하라.

**해설.** 코드와 환경, 직렬화 방식, 정책 버전이 바뀔 수 있다.
입력과 정책 식별자, 버전, 필요한 규칙 데이터를 보관하는 설계를 검토한다.
재현 요구사항에 따라 해당 버전의 실행 의미도 보존해야 한다.

### 다음 장으로

다음 장은 함수를 받는 쪽의 API를 설계한다.
콜백을 언제, 몇 번, 어떤 순서로 호출하는지가 핵심 계약이 된다.
일급 함수가 재료라면 고차 함수는 그 재료를 조립하는 도구다.

### 참고 자료

[Scala 공식 문서: Function Variables](https://docs.scala-lang.org/scala3/book/fun-function-variables.html)
[Scala 공식 문서: Higher-Order Functions](https://docs.scala-lang.org/scala3/book/fun-hofs.html)
[Python 공식 문서: typing](https://docs.python.org/3.14/library/typing.html)
