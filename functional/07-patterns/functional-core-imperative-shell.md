# 44장. Functional Core, Imperative Shell

앞 부에서는 효과를 값 계산과 분리하는 여러 도구를 배웠다.
이제 그 도구들을 프로그램의 구조에 적용한다.
Functional Core, Imperative Shell은 핵심 판단을 순수한 계산에 두고 외부 읽기와 실행을 바깥
계층에서 조정하는 패턴이다.

이번 장은 여러 주문 행을 재고 예약 계획으로 바꾼다.
같은 상품이 여러 행에 있을 때 수량을 먼저 합쳐 검사해야 하는 실제적인 경계 사례를 포함한다.
계산의 성공, 저장 충돌, 부분 효과의 책임이 어느 계층에 있는지 명확히 한다.

---

## 1. 개념과 기본 구분

### 핵심과 바깥 계층

핵심 로직은 명시적인 입력값으로 판단하고 결과값을 반환한다.
바깥 계층은 필요한 외부 정보를 읽고 핵심에 전달하며 결과를 실제 시스템에 적용한다.
두 부분은 책임과 관측 가능한 효과의 범위가 다르다.

```text
Shell: 입력 수집과 외부 상태 읽기
Core:  값으로 판단하고 계획 생성
Shell: 계획 실행과 응답 전달
```

명령형 바깥 계층이 무조건 복잡하거나 품질이 낮다는 뜻은 아니다.
외부 실행의 순서와 실패를 명시적으로 다루는 중요한 역할을 가진다.
핵심을 순수하게 만들기 위해 필요한 실행 책임을 숨겨서는 안 된다.

### 핵심의 입력

핵심은 데이터베이스 연결 대신 필요한 재고·가격 스냅샷을 받는다.
시간에 따라 바뀌는 정책도 명시적인 값으로 전달할 수 있다.
재현에 필요한 입력이 무엇인지 시그니처에서 확인할 수 있다.

### 핵심의 출력

출력은 계산 결과나 오류, 실행할 명령·계획일 수 있다.
외부 시스템이 실제로 적용되었다는 증거가 아니라 판단의 결과다.
계획과 실행 영수증을 다른 타입으로 구분할 수 있다.

### 책임의 중복과 방어

핵심이 재고를 검사했어도 저장소는 실행 시 경쟁을 막기 위해 다시 조건을 확인해야 할 수 있다.
이는 같은 검증을 무의미하게 반복하는 것과 다르다.
스냅샷 판단과 원자적 저장 조건은 서로 다른 시점을 보호한다.

---

## 2. 명령형 스타일과 함수형 스타일

### 하나의 큰 처리 함수

```text
주문 행 순회
각 행마다 상품 조회
재고 검사 후 즉시 감소
다음 행 처리
총액 응답
```

중간 행에서 실패하면 앞의 재고 감소가 남을 수 있다.
같은 상품이 여러 행에 있을 때 각각만 검사하면 전체 요청 수량을 놓칠 수 있다.
외부 실행과 전체 주문 판단이 섞인 구조다.

### 전체 요청을 먼저 계산한다

```text
행별 수량을 상품 코드별로 합친다
전체 요청 수량을 스냅샷 재고와 비교한다
가격과 수량으로 전체 금액을 계산한다
모든 조건을 만족하면 하나의 계획을 반환한다
```

계산 중에는 재고를 실제로 바꾸지 않는다.
전체 계획이 유효할 때 바깥 계층이 저장을 요청한다.
실패한 계산에서 부분 저장이 발생하는 경로를 줄인다.

### 외부 실행을 숨기는 핵심

핵심 함수가 내부에서 전역 저장소를 읽거나 로그를 출력하면 경계가 다시 흐려진다.
함수 이름에 `pure`를 붙인다고 의존성이 사라지는 것은 아니다.
실제 입력과 호출되는 코드를 검토해야 한다.

### 과도하게 큰 스냅샷

전체 데이터베이스를 복사할 필요는 없다.
해당 주문 판단에 필요한 상품과 정책만 수집할 수 있다.
데이터 크기와 일관성, 조회 횟수를 함께 설계한다.

---

## 3. 왜 이 개념을 사용하는가?

### 핵심 규칙의 집중

수량 집계, 가격 계산, 오류 우선순위가 한 순수한 영역에서 보인다.
외부 연결 설정과 분리되어 업무 규칙을 검토하기 쉽다.
테스트는 작은 값으로 다양한 조건을 만들 수 있다.

### 실행 실패의 분리

계획을 만들 수 없다는 오류와 적용 시 버전 충돌을 구분한다.
호출자가 어떤 경우에 입력을 수정하고 어떤 경우에 재조회해야 하는지 결정할 수 있다.
실행 실패를 핵심의 정상 결과로 숨기지 않는다.

### 재현과 감사

요청과 사용한 스냅샷을 보관하면 같은 판단을 다시 계산할 수 있다.
현재 저장소를 조회해서 과거 판단을 재현하려는 방식보다 명확하다.
보관 비용과 개인정보 정책은 별도로 검토한다.

### 테스트 계층의 분리

핵심은 값과 법칙 중심의 테스트를 사용한다.
바깥 계층은 외부 호출 순서와 실패 시 행동을 검사한다.
실제 저장소 어댑터에는 원자성과 연결 실패의 통합 테스트가 필요하다.

### 점진적인 도입

기존 큰 함수에서 금액 계산이나 상태 판단부터 순수 함수로 추출할 수 있다.
프로그램 전체를 한 번에 새 프레임워크로 바꿀 필요는 없다.
명확한 입력·출력 경계를 하나씩 늘리는 방식으로 진행할 수 있다.

---

## 4. Scala에서의 표현

### Scala의 주문 핵심과 저장 포트

수량 집계에는 임의 정밀도 정수를 사용한다.
상품 코드 순으로 검사하여 오류 선택의 순서를 고정한다.
메모리 저장소는 단일 스레드 계약을 검사하는 모형이며 실제 분산 트랜잭션 구현이 아니다.

<!-- executable:scala -->
```scala
object Chapter44:
  final case class Line(sku: String, quantity: Int)
  final case class Request(orderId: String, lines: Vector[Line])
  final case class Product(unitPrice: BigInt, stock: BigInt)
  final case class Snapshot(products: Map[String, Product], version: Long)
  final case class Plan(orderId: String, expectedVersion: Long, amount: BigInt, reservations: Map[String, BigInt])
  final case class Receipt(orderId: String, amount: BigInt, version: Long)
  enum Error:
    case EmptyOrder
    case InvalidQuantity
    case UnknownSku(sku: String)
    case OutOfStock(sku: String)
    case Conflict

  def decide(request: Request, snapshot: Snapshot): Either[Error, Plan] =
    if request.lines.isEmpty then Left(Error.EmptyOrder)
    else if request.lines.exists(_.quantity <= 0) then Left(Error.InvalidQuantity)
    else
      val required = request.lines.foldLeft(Map.empty[String, BigInt]) { (acc, line) =>
        acc.updated(line.sku, acc.getOrElse(line.sku, BigInt(0)) + line.quantity)
      }
      val amount = required.toVector.sortBy(_._1).foldLeft[Either[Error, BigInt]](Right(BigInt(0))) {
        case (acc, (sku, quantity)) => acc.flatMap { total =>
          snapshot.products.get(sku) match
            case None => Left(Error.UnknownSku(sku))
            case Some(product) if product.stock < quantity => Left(Error.OutOfStock(sku))
            case Some(product) => Right(total + product.unitPrice * quantity)
        }
      }
      amount.map(total => Plan(request.orderId, snapshot.version, total, required))

  trait Repository:
    def readSnapshot(): Either[Error, Snapshot]
    def commit(plan: Plan): Either[Error, Receipt]

  def place(request: Request, repository: Repository): Either[Error, Receipt] =
    for
      snapshot <- repository.readSnapshot()
      plan <- decide(request, snapshot)
      receipt <- repository.commit(plan)
    yield receipt

  final class MemoryRepository(initial: Map[String, Product]) extends Repository:
    private var current = Snapshot(initial, 0)
    var reads = 0
    var writes = 0
    def inspect: Snapshot = current
    def readSnapshot(): Either[Error, Snapshot] =
      reads += 1
      Right(current)
    def commit(plan: Plan): Either[Error, Receipt] =
      writes += 1
      if plan.expectedVersion != current.version then Left(Error.Conflict)
      else
        val changed = plan.reservations.foldLeft(current.products) { case (products, (sku, quantity)) =>
          val old = products(sku)
          products.updated(sku, old.copy(stock = old.stock - quantity))
        }
        current = Snapshot(changed, current.version + 1)
        Right(Receipt(plan.orderId, plan.amount, current.version))

  def check(): Unit =
    val products = Map("A" -> Product(1000, 3), "B" -> Product(500, 2))
    val snapshot = Snapshot(products, 0)
    val request = Request("O-1", Vector(Line("A", 1), Line("A", 2), Line("B", 1)))
    val expected = Plan("O-1", 0, 3500, Map("A" -> BigInt(3), "B" -> BigInt(1)))
    assert(decide(request, snapshot) == Right(expected))
    assert(decide(request, snapshot) == decide(request, snapshot))
    assert(snapshot.products == products)
    assert(decide(Request("O-2", Vector(Line("A", 2), Line("A", 2))), snapshot) == Left(Error.OutOfStock("A")))
    assert(decide(Request("O-3", Vector.empty), snapshot) == Left(Error.EmptyOrder))
    val repository = new MemoryRepository(products)
    assert(place(request, repository) == Right(Receipt("O-1", 3500, 1)))
    assert(repository.reads == 1 && repository.writes == 1)
    assert(repository.inspect.products("A").stock == 0)
    assert(repository.inspect.products("B").stock == 1)
    assert(repository.commit(expected) == Left(Error.Conflict))
    val failedRepository = new MemoryRepository(products)
    assert(place(Request("bad", Vector(Line("A", 4))), failedRepository) == Left(Error.OutOfStock("A")))
    assert(failedRepository.writes == 0)
    assert(failedRepository.inspect == snapshot)
```

### 저장 포트의 신뢰 경계

메모리 모형의 `commit`은 핵심이 만든 계획만 전달된다는 내부 계약을 사용한다.
신뢰할 수 없는 외부 계획을 그대로 받는 공개 API가 아니다.
제품용 어댑터는 권한, 계획 유효성, 원자적 조건부 갱신을 해당 저장소에서 보장해야 한다.

### 요청 식별자와 멱등성

`orderId`를 결과에 보관하지만 중복 요청 제거를 구현한 것은 아니다.
같은 요청을 새 스냅샷으로 다시 실행하면 다른 계획이 만들어질 수 있다.
멱등성 키의 저장과 재사용 정책은 별도로 설계해야 한다.

---

## 5. 상태 변경보다 값 변환

### 경계에서 오가는 값

바깥 계층은 외부 데이터를 내부 스냅샷으로 변환한다.
핵심은 그 값에서 계획을 만들고 바깥 계층이 다시 외부 명령으로 적용한다.
포트의 세부 형식이 핵심으로 새어 들어오지 않도록 한다.

```mermaid
flowchart LR
    A["외부 요청과 저장소"] --> B["Shell: 읽기"]
    B --> C["Request와 Snapshot"]
    C --> D["Core: decide"]
    D --> E["Plan 또는 판단 오류"]
    E --> F["Shell: 조건부 저장"]
    F --> G["Receipt 또는 실행 오류"]
```

경계가 늘었다고 같은 데이터를 불필요하게 여러 번 복제해야 하는 것은 아니다.
도메인 의미와 외부 형식이 달라지는 지점에 적절한 값을 둔다.
이름만 다른 동일한 DTO를 계속 만드는 과도한 계층화는 피한다.

### 핵심의 불변식

한 상품의 여러 행은 합산된 수량으로 검사한다.
각 행이 재고 이하라는 사실만으로 전체 주문이 유효하지 않을 수 있다.
핵심은 개별 필드가 아니라 전체 판단에 필요한 관계를 다룬다.

### 외부 사실의 유효기간

스냅샷의 가격과 재고는 특정 시점의 사실이다.
핵심의 결과를 오래 보관하면 실행 시 유효성이 달라질 수 있다.
버전과 만료, 재판단 정책을 바깥 실행과 함께 설계한다.

---

## 6. 함수 합성과 데이터 흐름

### 함수 합성의 구조

바깥 계층은 읽기 결과, 판단 결과, 저장 결과를 의존적으로 연결한다.
각 단계가 실패하면 다음 단계의 실행 정책이 달라진다.
이 구조를 `Either`와 `flatMap`으로 표현할 수 있다.

```text
read -> decide -> commit
```

하지만 이 세 함수를 연결했다고 하나의 트랜잭션이 되는 것은 아니다.
순수한 중간 판단과 외부 읽기·쓰기 사이의 경쟁을 별도로 처리해야 한다.
연결의 타입과 저장 프로토콜의 보장을 구분한다.

### 핵심 내부의 조합

필터, fold, 오류 결과, 불변 데이터 모델을 함께 사용한다.
각 도구는 전체 주문 판단의 특정 부분을 담당한다.
패턴은 앞 장의 개념을 대체하는 새 문법이 아니라 그 배치 원칙이다.

### 여러 단계의 외부 상호작용

모든 프로그램이 읽기 한 번과 쓰기 한 번으로 끝나지는 않는다.
외부 응답에 따라 다음 순수 판단을 수행하는 여러 단계의 코어·셸 왕복이 가능하다.
한 거대한 순수 함수에 모든 흐름을 억지로 넣지 않는다.

---

## 7. 장점과 트레이드오프

### 장점과 트레이드오프

| 선택 | 이점 | 남는 책임 |
| --- | --- | --- |
| 순수 핵심 | 규칙 테스트와 재현 | 올바른 스냅샷 수집 |
| 계획 반환 | 부분 실행 감소 | 실제 적용의 실패 |
| 얇은 셸 | 효과 위치 명확 | 오류·취소·자원 정책 |
| 입력 집계 | 전체 관계 검사 | 큰 입력의 비용 |
| 명시적 버전 | 오래된 판단 감지 | 원자적인 비교와 저장 |

### 셸이 너무 얇다는 오해

바깥 계층에는 중요한 실행 정책이 남는다.
재시도와 자원 관리, 관측, 권한을 아무 의미 없는 배관으로 취급하지 않는다.
업무 규칙과 실행 규칙의 책임을 구분하면서 둘 다 검증해야 한다.

### 데이터 이동 비용

큰 스냅샷을 만들면 조회와 메모리 비용이 커질 수 있다.
필요한 데이터만 읽거나 단계별로 판단하는 구조를 검토한다.
순수성을 위해 현실적인 I/O 제약을 무시하지 않는다.

### 외부 원자성

계획 안에 여러 재고 변경이 있어도 저장소가 이를 하나로 적용하지 않으면 부분 변경이 생길 수 있다.
트랜잭션이나 적절한 원자적 명령을 사용하는 것은 어댑터의 책임이다.
메모리 모형의 성공을 실제 데이터베이스 보장으로 확대하지 않는다.

---

## 8. 상태와 부수효과의 경계

### 경쟁과 재판단

판단 이후 다른 요청이 재고를 바꾸면 계획 적용이 충돌할 수 있다.
다시 읽고 다시 판단할지 사용자에게 충돌을 알릴지 정책이 필요하다.
무한 재시도나 무조건 성공으로 바꾸는 처리는 피한다.

### 부분 외부 성공

재고 예약 후 결제, 알림 같은 추가 작업이 있으면 전체 원자성이 어려울 수 있다.
작업별 성공 상태와 보상, 멱등성을 명시적으로 설계한다.
순수 핵심 패턴이 분산 트랜잭션을 자동 해결하지는 않는다.

### 자원 수명

바깥 계층에서 연결과 파일을 획득하고 사용 범위가 끝나면 해제한다.
핵심에 자원 핸들을 넘기면 수명과 효과가 다시 섞일 수 있다.
가능하면 필요한 값만 전달한다.

### 관측과 개인정보

스냅샷과 계획을 남기는 로그는 재현에 도움이 되지만 민감정보를 포함할 수 있다.
필요한 식별자와 정책 버전만 보관하는 대안을 검토한다.
디버깅 편의와 정보 보관 위험을 함께 관리한다.

---

## 9. Python에서 적용하기

### Python의 핵심 함수와 셸

Python에서도 포트는 셸에서 사용하고 핵심에는 값만 전달할 수 있다.
스냅샷과 계획은 튜플 기반의 불변 레코드로 표현한다.
작은 입력을 위한 교육용 구현이며 조회 자료구조의 성능 최적화는 별도다.

<!-- executable:python -->
```python
from dataclasses import dataclass, replace
from enum import Enum
from typing import Protocol


@dataclass(frozen=True)
class Line:
    sku: str
    quantity: int


@dataclass(frozen=True)
class Product:
    sku: str
    unit_price: int
    stock: int


@dataclass(frozen=True)
class Snapshot:
    products: tuple[Product, ...]
    version: int


@dataclass(frozen=True)
class Plan:
    order_id: str
    expected_version: int
    amount: int
    reservations: tuple[tuple[str, int], ...]


@dataclass(frozen=True)
class Receipt:
    order_id: str
    amount: int
    version: int


class Error(Enum):
    EMPTY_ORDER = "empty_order"
    INVALID_QUANTITY = "invalid_quantity"
    UNKNOWN_SKU = "unknown_sku"
    OUT_OF_STOCK = "out_of_stock"
    CONFLICT = "conflict"


def decide(order_id: str, lines: tuple[Line, ...], snapshot: Snapshot) -> Plan | Error:
    if not lines:
        return Error.EMPTY_ORDER
    if any(type(line.quantity) is not int or line.quantity <= 0 for line in lines):
        return Error.INVALID_QUANTITY
    required: dict[str, int] = {}
    for line in lines:
        required[line.sku] = required.get(line.sku, 0) + line.quantity
    products = {product.sku: product for product in snapshot.products}
    amount = 0
    for sku, quantity in sorted(required.items()):
        product = products.get(sku)
        if product is None:
            return Error.UNKNOWN_SKU
        if product.stock < quantity:
            return Error.OUT_OF_STOCK
        amount += product.unit_price * quantity
    return Plan(order_id, snapshot.version, amount, tuple(sorted(required.items())))


class Repository(Protocol):
    def read_snapshot(self) -> Snapshot | Error:
        ...

    def commit(self, plan: Plan) -> Receipt | Error:
        ...


def place(order_id: str, lines: tuple[Line, ...], repository: Repository) -> Receipt | Error:
    snapshot = repository.read_snapshot()
    if isinstance(snapshot, Error):
        return snapshot
    plan = decide(order_id, lines, snapshot)
    return plan if isinstance(plan, Error) else repository.commit(plan)


class MemoryRepository:
    def __init__(self, products: tuple[Product, ...]) -> None:
        self.current = Snapshot(products, 0)
        self.reads = 0
        self.writes = 0

    def read_snapshot(self) -> Snapshot:
        self.reads += 1
        return self.current

    def commit(self, plan: Plan) -> Receipt | Error:
        self.writes += 1
        if plan.expected_version != self.current.version:
            return Error.CONFLICT
        required = dict(plan.reservations)
        changed = tuple(replace(product, stock=product.stock - required.get(product.sku, 0)) for product in self.current.products)
        self.current = Snapshot(changed, self.current.version + 1)
        return Receipt(plan.order_id, plan.amount, self.current.version)


def test_core_and_shell() -> None:
    products = (Product("A", 1000, 3), Product("B", 500, 2))
    snapshot = Snapshot(products, 0)
    lines = (Line("A", 1), Line("A", 2), Line("B", 1))
    expected = Plan("O-1", 0, 3500, (("A", 3), ("B", 1)))
    assert decide("O-1", lines, snapshot) == expected
    assert decide("O-1", lines, snapshot) == decide("O-1", lines, snapshot)
    assert decide("O-2", (Line("A", 2), Line("A", 2)), snapshot) is Error.OUT_OF_STOCK
    repository = MemoryRepository(products)
    assert place("O-1", lines, repository) == Receipt("O-1", 3500, 1)
    assert repository.reads == 1 and repository.writes == 1
    assert [product.stock for product in repository.current.products] == [0, 1]
    assert repository.commit(expected) is Error.CONFLICT
    failed = MemoryRepository(products)
    assert place("bad", (Line("A", 4),), failed) is Error.OUT_OF_STOCK
    assert failed.writes == 0 and failed.current == snapshot


if __name__ == "__main__":
    test_core_and_shell()
```

### 모형에서 생략한 것

요청 식별자의 형식과 사용자 권한, 외부 JSON의 구조는 이 예제의 앞쪽 경계에서 처리한다고
가정한다.
저장소 연결 장애와 분산 멱등성은 구현하지 않았다.
핵심·셸 분리의 실행 가능한 예제와 완성된 주문 서비스의 범위를 구분한다.

---

## 10. Python의 표현 한계

### 순수성의 강제 수준

Python에서는 핵심 함수가 전역 객체를 읽거나 바꾸는 것을 타입 주석이 자동 금지하지 않는다.
입력과 import 의존성을 검토하고 필요한 경우 별도 분석 도구를 사용할 수 있다.
구조적인 설계 의도와 언어가 강제하는 보장을 구분한다.

### 가변 데이터의 유출

스냅샷 안에 가변 딕셔너리나 ORM 객체를 넣으면 핵심에서 저장소 상태와 별칭을 공유할 수 있다.
필요한 필드만 단순한 도메인 값으로 변환하는 것이 도움이 된다.
얕은 frozen 래퍼 하나로 깊은 불변성이 완성되는 것은 아니다.

### 일반 함수의 이점

핵심은 복잡한 클래스 계층 없이 평범한 함수로 작성할 수 있다.
셸도 명시적인 분기문으로 오류와 호출 순서를 표현할 수 있다.
함수형 패턴을 사용하기 위해 모든 제어 흐름을 체이닝으로 바꿀 필요는 없다.

### 테스트의 경계

핵심 테스트가 많이 통과해도 실제 저장소가 계획을 원자적으로 적용하는지는 별도다.
포트 계약과 실제 구현을 함께 검증해야 한다.
테스트 대역의 단순한 성공을 운영 보장으로 오해하지 않는다.

---

## 11. 핵심 정리

### 핵심 결론

Functional Core, Imperative Shell은 판단과 실행의 책임을 나누는 패턴이다.
핵심은 명시적인 값에서 계획이나 오류를 계산하고 셸은 외부 작업을 조정한다.
전체 입력의 관계 검증과 실행 시점의 경쟁 검사는 서로 다른 책임이다.
원자성, 멱등성, 자원과 관측은 바깥 계층에서도 반드시 설계해야 한다.

### 연습 1: 중복 상품 행

재고가 3인 상품을 두 행에서 각각 2개씩 주문했다.
각 행만 검사하면 어떤 오류를 놓치는가?

**해설.** 전체 요청 수량은 4이므로 재고가 부족하다.
상품별 수량을 합친 뒤 전체 관계를 검사해야 한다.
개별 필드 검증과 주문 전체의 판단을 구분한다.

### 연습 2: 스냅샷의 시점

핵심에서 재고가 충분하다고 판단했으니 저장소에서는 경쟁 검사가 필요 없다는 주장을 평가하라.

**해설.** 판단 이후 다른 요청이 상태를 바꿀 수 있다.
실행 경계에서 원자적인 버전 확인과 적용이 필요할 수 있다.
순수한 판단은 특정 스냅샷에 대한 보장이다.

### 연습 3: 숨은 읽기

핵심 함수가 인자에 없는 전역 시계를 읽는다.
어떤 문제가 생기는가?

**해설.** 같은 명시적 입력에서도 결과가 달라질 수 있다.
재현에 필요한 기준 시각을 값으로 전달하는 편이 명확하다.
함수 이름보다 실제 의존성을 확인해야 한다.

### 연습 4: 얇은 셸

셸은 단순 배관이므로 테스트가 필요 없다는 주장에 반론하라.

**해설.** 셸은 호출 순서와 실패, 취소, 자원 수명을 관리한다.
부분 외부 성공과 재시도 같은 중요한 오류가 발생할 수 있다.
핵심과 다른 관점의 검증이 필요하다.

### 다음 장과 참고 자료

다음 장은 핵심에 들어오는 원시 입력을 유효한 도메인 값으로 바꾸는 경계를 다룬다.
검사 결과를 버리지 않고 타입과 데이터에 남기는 Parse, Don't Validate 원칙을 적용한다.

[Gary Bernhardt: Boundaries](https://www.destroyallsoftware.com/talks/boundaries)
[Scala 공식 문서: Pure Functions](https://docs.scala-lang.org/scala3/book/fp-pure-functions.html)
[Martin Fowler: Dependency Injection](https://martinfowler.com/articles/injection.html)
