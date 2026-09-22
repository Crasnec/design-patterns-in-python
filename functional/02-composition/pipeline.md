# 14장. Pipeline

![여러 석회암 층과 연못을 지나 맑은 샘으로 나오는 빗물](../../assets/images/fp/pipeline.png)

작은 변환을 배웠다면 이제 업무 흐름 전체를 읽을 차례다.
파이프라인은 입력을 여러 명시적인 단계에 차례로 전달하는 구성 방식이다.
특별한 연산자 하나를 도입하는 것이 아니라 각 단계가 어떤 값을 받고 무엇을 만들어 내는지 드러내는
설계다.

이 장에서는 이미 형식 검사가 끝난 주문 스냅샷으로 판매 보고서를 만든다.
대상 선택, 품목 금액 계산, 합계 요약, 표시 데이터 생성까지 순수하게 구성한다.
데이터 조회와 실제 보고서 발행은 바깥 경계에 남겨 두어 계산과 효과를 구별한다.

---

## 1. 개념과 기본 구분

### 단계와 경계

파이프라인의 각 단계는 입력 계약과 출력 계약을 가진다.
앞 단계의 출력은 다음 단계가 기대하는 의미를 만족해야 한다.
단순히 함수들이 순서대로 호출된다는 사실만으로 좋은 파이프라인이 되는 것은 아니다.

```text
원본 스냅샷
  -> 대상 주문
  -> 품목 금액
  -> 요약
  -> 표시 데이터
```

각 단계에서 데이터의 형태가 바뀌면 이름과 타입으로 그 변화를 드러낸다.
어떤 단계가 원소를 제거하거나 오류 정보를 버리는지도 확인해야 한다.
파이프라인은 데이터가 흐르는 경로이면서 정보가 축소되는 경로이기도 하다.

### 함수 합성과의 관계

함수 합성은 두 함수의 연결을 새 함수로 만드는 연산이다.
파이프라인은 그런 연결을 업무 흐름으로 조직하는 설계 관점이다.
명시적인 중간 변수를 사용해도 파이프라인으로 읽을 수 있다.

### 선형 흐름의 범위

모든 업무가 한 줄의 선형 단계에 맞지는 않는다.
독립 계산의 분기, 조건부 실행, 반복, 보상이 필요할 수 있다.
그런 구조를 무리하게 하나의 가변 딕셔너리에 넣어 흉내 내면 의존성이 숨는다.

### 데이터 파이프라인과 효과 실행기

값을 변환하는 파이프라인과 외부 작업을 스케줄링하는 실행기는 다른 추상화다.
전자는 계산의 연결을 설명하고 후자는 동시성, 재시도, 자원, 취소 등을 관리할 수 있다.
같은 단어를 사용하더라도 제공하는 보장은 구분해야 한다.

---

## 2. 명령형 스타일과 함수형 스타일

### 하나의 큰 함수

보고서 함수가 조회, 필터링, 금액 계산, 문자열 생성, 파일 저장을 모두 수행할 수 있다.
그 경우 계산 규칙을 테스트하려고 파일 시스템과 데이터베이스까지 준비해야 한다.
어디서 실패했는지 찾기도 어려워진다.

```text
buildReport()
  데이터베이스 조회
  전역 날짜 읽기
  주문 상태 검사
  품목 합계 계산
  CSV 문자열 생성
  파일 저장
```

이 함수의 반환값만 보고 전체 계약을 알기 어렵다.
특히 조회 시점과 파일 저장 시점 사이의 실패가 섞인다.
작은 계산부터 분리하면 효과 경계를 더 명확하게 만들 수 있다.

### 값 단계를 추출한다

```scala
val selected = selectPaid(snapshot)
val charges = extractCharges(selected)
val summary = summarize(charges)
val report = render(summary)
```

중간값은 단순한 임시 변수 이상의 의미를 가진다.
각 이름이 업무 상태와 데이터 형태를 설명한다.
테스트에서는 필요한 중간 단계부터 시작할 수 있다.

### 거대한 문맥 객체의 함정

모든 단계가 같은 가변 `context`를 받아 필드를 채우는 설계는 의존성을 감출 수 있다.
어떤 필드가 언제 준비되는지 별도 규칙을 알아야 한다.
단계별 입력·출력 타입을 구분하면 순서와 전제조건을 더 쉽게 확인할 수 있다.

---

## 3. 왜 이 개념을 사용하는가?

### 단계별 실패 위치

입력이 잘못되었는지, 대상이 없는지, 계산이 실패했는지를 구분할 수 있다.
단계 이름은 진단 메시지와 테스트 이름에도 사용된다.
하지만 모든 예외를 문자열 하나로 바꾸면 원래 오류 정보가 손실될 수 있다.

### 업무 규칙의 변경

선택 정책과 표시 형식을 분리하면 각각의 변경 범위를 줄일 수 있다.
다만 선택 정책이 계산 결과에 의존하기 시작하면 흐름을 다시 설계해야 한다.
기존 단계 순서에 새 요구를 억지로 끼워 넣지 않는다.

### 재현 가능한 입력

조회한 스냅샷을 값으로 전달하면 같은 보고서를 다시 계산할 수 있다.
기준 날짜와 통화 같은 정책 입력도 명시적으로 포함한다.
현재 환경을 각 단계에서 다시 읽으면 파이프라인 안에서도 시점이 달라질 수 있다.

### 중간 산출물

중간 데이터를 관찰하면 오류 원인을 좁히기 쉽다.
그러나 전체 고객 정보나 결제 정보를 무조건 로그로 남겨서는 안 된다.
진단에 필요한 최소 정보와 식별자만 노출하는 정책을 정한다.

### 과도한 단계 분리

한 줄 계산마다 별도 모듈을 만들면 흐름을 따라가는 비용이 커진다.
의미가 바뀌는 경계, 반복되는 정책, 독립 테스트가 유용한 위치를 기준으로 나눈다.
함수 개수보다 계약의 명료함이 중요하다.

---

## 4. Scala에서의 표현

### Scala의 보고서 파이프라인

입력은 이미 생성된 불변 주문값이다.
이 예제는 결제 상태가 `Paid`인 주문의 품목 금액을 요약한다.
빈 보고서는 오류가 아니라 합계 0인 정상 결과로 취급한다.

<!-- executable:scala -->
```scala
object Chapter14:
  enum Status:
    case Draft, Paid, Cancelled

  final case class Item(quantity: Int, unitPrice: BigInt)
  final case class Order(id: String, status: Status, items: List[Item])
  final case class Charge(orderId: String, amount: BigInt)
  final case class Summary(lineCount: Int, total: BigInt)
  final case class Report(title: String, rows: List[String])

  def selectPaid(orders: List[Order]): List[Order] =
    orders.filter(_.status == Status.Paid)

  def extractCharges(orders: List[Order]): List[Charge] =
    orders.flatMap { order =>
      order.items.map { item =>
        Charge(order.id, item.unitPrice * item.quantity)
      }
    }

  def summarize(charges: List[Charge]): Summary =
    charges.foldLeft(Summary(0, 0)) { (acc, charge) =>
      Summary(acc.lineCount + 1, acc.total + charge.amount)
    }

  def render(summary: Summary): Report =
    Report("판매 요약", List(
      s"품목 행 수: ${summary.lineCount}",
      s"합계: ${summary.total} KRW"
    ))

  def buildReport(snapshot: List[Order]): Report =
    val selected = selectPaid(snapshot)
    val charges = extractCharges(selected)
    val summary = summarize(charges)
    render(summary)

  def check(): Unit =
    val snapshot = List(
      Order("O-1", Status.Paid, List(Item(2, 1000), Item(1, 500))),
      Order("O-2", Status.Draft, List(Item(9, 9000))),
      Order("O-3", Status.Paid, List(Item(3, 200))),
      Order("O-4", Status.Cancelled, Nil)
    )
    assert(selectPaid(snapshot).map(_.id) == List("O-1", "O-3"))
    assert(extractCharges(selectPaid(snapshot)).map(_.amount) == List[BigInt](2000, 500, 600))
    val report = buildReport(snapshot)
    assert(report.rows == List("품목 행 수: 3", "합계: 3100 KRW"))
    assert(buildReport(snapshot) == report)
    assert(snapshot.length == 4)
    assert(buildReport(Nil).rows == List("품목 행 수: 0", "합계: 0 KRW"))
```

### 데이터의 이름을 정확히 한다

`lineCount`는 주문 수가 아니라 품목 행 수다.
수량의 합도 아니다.
같은 숫자 타입으로 표현되더라도 서로 다른 업무 집계이므로 이름과 테스트로 구별한다.

### 입력 계약

수량과 단가가 유효하다는 가정은 이 예제의 입력 경계에 있다.
외부 문자열을 그대로 이런 레코드로 만들면 잘못된 값이 들어올 수 있다.
3부의 스마트 생성자와 4부의 오류 처리가 그 경계를 보강한다.

### 보고서와 발행의 구분

`Report`는 보고할 데이터다.
파일이 만들어졌거나 이메일이 전송되었다는 상태를 의미하지 않는다.
계산된 결과와 외부 효과의 성공을 같은 이름으로 묶지 않는다.

---

## 5. 상태 변경보다 값 변환

### 각 단계에서 남는 정보

선택 단계는 비대상 주문을 제거한다.
품목 추출 단계는 주문 구조를 품목별 금액 구조로 바꾼다.
요약 단계는 개별 금액을 합계로 압축한다.
표시 단계는 숫자를 문자열로 바꾼다.

```mermaid
flowchart LR
    A["List Order"] --> B["selectPaid"]
    B --> C["List Order"]
    C --> D["extractCharges"]
    D --> E["List Charge"]
    E --> F["summarize"]
    F --> G["Summary"]
    G --> H["render"]
    H --> I["Report"]
```

표시 문자열에서 정확한 숫자를 다시 파싱하여 다음 계산에 쓰는 설계는 피하는 편이 좋다.
표시는 마지막 경계에 두고 계산은 도메인 값으로 유지한다.
형식과 의미를 일찍 섞으면 국제화와 반올림 정책이 계산에 침투한다.

### 중간값의 보관 여부

모든 중간값을 영구 저장할 필요는 없다.
재현과 감사 요구에 필요한 입력 스냅샷이나 요약만 보관할 수 있다.
보관 정책은 개인정보, 비용, 운영 요구와 함께 결정한다.

### 분기와 합류

같은 선택 결과에서 매출 합계와 상품별 집계를 독립적으로 계산할 수 있다.
이때 파이프라인은 선형 목록이 아니라 작은 의존성 그래프가 된다.
독립 계산의 결합은 Applicative 장의 관점과 연결된다.

---

## 6. 함수 합성과 데이터 흐름

### 조합과 단계 이름

각 단계의 타입이 이어지면 합성 함수로 만들 수도 있다.
그러나 명시적 중간값 버전이 더 잘 읽히면 그대로 유지한다.
파이프라인은 특정 표기법보다 데이터 의존성의 명시다.

```text
buildReport
  = render ∘ summarize ∘ extractCharges ∘ selectPaid
```

이 식은 흐름의 개요를 보여 준다.
실제 코드에서는 각 단계의 정책과 오류 계약을 함께 읽어야 한다.
타입이 이어진다는 이유로 모든 업무 전제조건이 충족된 것은 아니다.

### 순서를 바꿀 수 있는가?

결제된 주문을 고르기 전에 모든 주문의 품목 금액을 계산할 수는 있다.
하지만 불필요한 계산이 늘고 잘못된 비대상 데이터에서 실패할 수도 있다.
순서 변경은 값, 오류, 비용의 세 관점을 함께 검토해야 한다.

### 합성의 경계 넓히기

일반 함수로 연결되지 않는 실패 결과가 들어오면 전용 연결 연산이 필요하다.
비동기 계산도 결과가 준비되는 시점과 취소 계약을 포함한다.
`pipe`라는 함수 하나로 이런 차이를 지워서는 안 된다.

### 테스트의 연결

단계별 테스트 외에 전체 파이프라인 테스트를 유지한다.
각 단계가 맞아도 잘못된 순서로 연결하면 업무 결과가 틀릴 수 있다.
전체 테스트는 조합의 계약을 확인한다.

---

## 7. 장점과 트레이드오프

### 장점과 비용

| 선택 | 이점 | 검토할 비용 |
| --- | --- | --- |
| 단계별 타입 | 의존성 명시 | 레코드와 변환 증가 |
| 입력 스냅샷 | 결과 재현 | 수집과 보관 비용 |
| 순수 계산 분리 | 빠른 규칙 테스트 | 경계 어댑터 코드 |
| 중간값 노출 | 디버깅 편의 | 민감정보와 메모리 |
| 지연 처리 | 큰 입력의 점진 소비 | 오류 시점과 자원 수명 |

### 여러 번의 순회

예제는 설명을 위해 중간 목록을 명시적으로 만든다.
큰 데이터에서 메모리와 순회 비용이 문제가 되면 한 번의 fold나 스트리밍 처리를 검토할 수 있다.
그때도 단계의 의미를 잃지 않도록 이름 있는 변환을 유지한다.

### 추상 실행기의 비용

모든 단계가 플러그인으로 등록되는 범용 파이프라인 엔진은 설정과 디버깅 비용을 만든다.
단순한 업무 흐름에는 평범한 함수 호출이 더 적절할 수 있다.
실제로 동적 조합이나 운영 관리가 필요한지 확인한 뒤 도입한다.

### 관측 도구

단계별 시간과 실패율을 측정하면 병목을 찾는 데 도움이 된다.
측정 코드가 계산의 반환값이나 재시도 정책을 바꾸지 않도록 경계를 분리한다.
관측 자체도 비용과 정보 노출을 가진 효과다.

---

## 8. 상태와 부수효과의 경계

### 조회, 계산, 발행

실제 프로그램은 외부에서 스냅샷을 읽고 보고서를 계산한 뒤 발행한다.
이 세 책임을 분리하면 실패 상태를 더 정확히 설명할 수 있다.
계산이 성공했지만 발행이 실패한 경우를 별도로 다뤄야 한다.

```text
readSnapshot -> buildReport -> publishReport
  외부 효과       순수 계산       외부 효과
```

발행을 재시도할 때 조회와 계산까지 다시 해야 하는지는 요구사항에 달려 있다.
동일한 보고서의 재발행인지 최신 데이터로 새 보고서를 만드는지 구분한다.
무심코 전체 파이프라인을 재실행하면 내용이 바뀔 수 있다.

### 부분 발행

여러 채널로 보고서를 보내다 중간에 실패할 수 있다.
일부 채널의 발행 성공과 나머지 실패를 보존해야 할 수 있다.
일반 함수 합성은 이 상태를 자동으로 관리하지 않는다.

### 자원과 취소

큰 입력을 파일에서 읽는 동안 취소되면 파일과 중간 출력 자원을 정리해야 한다.
순수한 요약 함수와 자원을 가진 읽기·쓰기 경계의 책임을 분리한다.
비동기 효과 장에서는 이런 실패와 정리를 구체적으로 다룬다.

---

## 9. Python에서 적용하기

### Python의 명시적 단계

Python에서는 중간값을 이름 붙인 일반 함수가 읽기 좋은 파이프라인이 될 수 있다.
아래 코드는 입력값을 불변 튜플로 받고 결과를 불변 레코드로 반환한다.
범용 `Any` 기반 실행기를 만들지 않고 각 단계의 타입을 유지한다.

<!-- executable:python -->
```python
from dataclasses import dataclass
from enum import Enum


class Status(Enum):
    DRAFT = "draft"
    PAID = "paid"
    CANCELLED = "cancelled"


@dataclass(frozen=True)
class Item:
    quantity: int
    unit_price: int


@dataclass(frozen=True)
class Order:
    order_id: str
    status: Status
    items: tuple[Item, ...]


@dataclass(frozen=True)
class Charge:
    order_id: str
    amount: int


@dataclass(frozen=True)
class Summary:
    line_count: int
    total: int


@dataclass(frozen=True)
class Report:
    title: str
    rows: tuple[str, ...]


def select_paid(orders: tuple[Order, ...]) -> tuple[Order, ...]:
    return tuple(order for order in orders if order.status is Status.PAID)


def extract_charges(orders: tuple[Order, ...]) -> tuple[Charge, ...]:
    return tuple(
        Charge(order.order_id, item.quantity * item.unit_price)
        for order in orders
        for item in order.items
    )


def summarize(charges: tuple[Charge, ...]) -> Summary:
    return Summary(len(charges), sum(charge.amount for charge in charges))


def render(summary: Summary) -> Report:
    return Report("판매 요약", (
        f"품목 행 수: {summary.line_count}",
        f"합계: {summary.total} KRW",
    ))


def build_report(snapshot: tuple[Order, ...]) -> Report:
    selected = select_paid(snapshot)
    charges = extract_charges(selected)
    summary = summarize(charges)
    return render(summary)


def test_pipeline() -> None:
    snapshot = (
        Order("O-1", Status.PAID, (Item(2, 1000), Item(1, 500))),
        Order("O-2", Status.DRAFT, (Item(9, 9000),)),
        Order("O-3", Status.PAID, (Item(3, 200),)),
        Order("O-4", Status.CANCELLED, ()),
    )
    assert tuple(order.order_id for order in select_paid(snapshot)) == ("O-1", "O-3")
    charges = extract_charges(select_paid(snapshot))
    assert tuple(charge.amount for charge in charges) == (2000, 500, 600)
    expected = Report("판매 요약", ("품목 행 수: 3", "합계: 3100 KRW"))
    assert build_report(snapshot) == expected
    assert build_report(snapshot) == build_report(snapshot)
    assert len(snapshot) == 4
    assert build_report(()) == Report("판매 요약", ("품목 행 수: 0", "합계: 0 KRW"))


if __name__ == "__main__":
    test_pipeline()
```

### 외부 어댑터의 위치

HTTP 요청이나 데이터베이스 모델을 이 함수에 직접 넣지 않는다.
경계에서 필요한 필드를 읽어 검증된 `Order` 값으로 바꾼다.
ORM 속성 접근의 지연 조회가 계산 안으로 숨어 들어오지 않도록 주의한다.

---

## 10. Python의 표현 한계

### 파이프 연산자를 흉내 낼 필요는 없다

Python의 기본 문법에 없는 연산자를 임의로 가정해서 설명하지 않는다.
명시적인 함수 호출과 중간값만으로 충분히 같은 설계 원리를 적용할 수 있다.
가독성을 위해 도입한 헬퍼가 타입 정보를 잃거나 디버깅을 어렵게 만들면 다시 검토한다.

### 타입 주석의 보장

튜플과 데이터 클래스의 타입 주석은 외부 입력을 자동으로 검증하지 않는다.
문자열 상태, 음수 수량, 잘못된 단가를 생성 경계에서 처리해야 한다.
파이프라인의 내부 가정과 외부의 신뢰 경계를 구분한다.

### 자동 스트리밍의 부재

중간 튜플을 만드는 코드는 엄격하게 데이터를 보관한다.
겉으로 파이프라인처럼 보인다고 자동으로 스트리밍 실행이 되는 것은 아니다.
생성기로 바꾸면 평가 시점과 반복 가능성도 함께 달라진다.

### 효과 추적

함수 타입만으로 조회, 저장, 취소 가능성이 드러나지 않을 수 있다.
효과가 있는 단계는 이름과 모듈 경계, 결과 타입으로 명시한다.
일반 함수들의 연결과 완전한 작업 실행 시스템을 혼동하지 않는다.

---

## 11. 핵심 정리

### 핵심 결론

파이프라인은 단계별 값과 계약을 연결하는 설계다.
명시적인 중간 변수도 좋은 파이프라인을 구성할 수 있다.
정보 손실, 오류, 비용, 효과 경계를 단계마다 확인해야 한다.
조회한 값의 계산과 실제 발행의 성공은 다른 상태다.

### 연습 1: 집계 이름

품목 행 수를 `order_count`라고 이름 붙였다.
어떤 오류가 생길 수 있으며 어떻게 수정할 수 있는가?

**해설.** 주문 하나에 여러 품목이 있으면 주문 수와 다르다.
수량의 합과도 다르므로 정확한 집계 의미를 이름과 타입에 드러낸다.
필요한 세 집계를 별도 필드로 정의할 수 있다.

### 연습 2: 전체 재시도

발행 실패 후 전체 파이프라인을 다시 실행하자 보고서 내용이 달라졌다.
어떤 입력이 바뀌었을 수 있는가?

**해설.** 다시 조회한 주문이나 기준 시각이 달라졌을 수 있다.
동일 보고서 재발행이 목적이면 계산된 보고서나 입력 스냅샷을 유지해야 한다.
최신 보고서 생성과 재발행의 요구를 구분한다.

### 연습 3: 가변 문맥

모든 단계가 같은 딕셔너리를 받아 필드를 추가하는 파이프라인의 위험을 설명하라.

**해설.** 필드가 준비되는 순서와 필수 조건이 타입에 드러나지 않는다.
단계별 입력·출력 레코드나 명시적인 매개변수로 의존성을 나타낼 수 있다.
공유 변경과 숨은 의존성을 줄이는 방향으로 설계한다.

### 연습 4: 메모리 최적화

중간 목록이 너무 커서 한 번의 fold로 합치려 한다.
무엇을 보존해야 하는가?

**해설.** 선택 조건, 원소 순서, 집계 의미, 실패 정책을 보존해야 한다.
효과가 있는 단계라면 호출 시점과 순서도 확인한다.
성능 개선 전후를 같은 입력과 경계값으로 비교한다.

### 다음 장과 참고 자료

다음 장은 일부 입력을 미리 정하여 파이프라인에 넣기 좋은 함수를 만드는 부분 적용을 다룬다.
함수의 설정 단계와 실행 단계를 분리하는 방법을 배운다.

[Scala 공식 문서: Functions](https://docs.scala-lang.org/scala3/book/fun-intro.html)
[Python 공식 문서: Functional Programming HOWTO](https://docs.python.org/3.14/howto/functional.html)
