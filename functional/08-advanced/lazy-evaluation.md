# 53장. Lazy Evaluation

함수를 조립하는 시점과 실제 계산하는 시점은 항상 같지 않다.
지연 평가는 결과가 필요할 때 계산을 수행하는 전략이다.
하지만 계산을 늦춘다는 사실만으로 한 번만 실행되거나 결과가 영구히 저장되는 것은 아니다.

이번 장은 엄격한 인자, 이름에 의한 인자 전달, Scala의 lazy val과 LazyList를 구분한다.
Python에서는 함수로 감싼 지연 계산, 결과를 보관하는 작은 Lazy 모형, 일회성 생성자를 비교한다.
평가 시점과 재실행, 공유, 자원 수명을 서로 다른 축으로 읽는 것이 목표다.

---

## 1. 개념과 기본 구분

### 엄격한 평가

일반적인 값 인자는 함수를 호출하기 전에 계산될 수 있다.
함수는 이미 얻은 값을 여러 번 사용할 수 있다.
Scala와 Python의 평범한 함수 호출을 이해하는 기본 출발점이다.

```text
strict(f())
  먼저 f를 실행
  그 결과를 strict에 전달
```

함수 본문이 인자를 사용하지 않더라도 인자 계산이 이미 일어날 수 있다.
불필요한 비용과 효과를 피하려면 평가를 미루는 별도의 표현이 필요하다.
문법상 인자를 전달했다는 사실과 실제 계산 시점을 구분한다.

### 이름에 의한 전달

Scala의 by-name 인자는 인자 표현식을 필요할 때 평가한다.
본문에서 여러 번 사용하면 여러 번 평가될 수 있다.
Python에서는 호출할 함수를 명시적으로 전달하여 비슷한 재실행 계약을 표현할 수 있다.

### 필요할 때 계산하고 공유

lazy val은 처음 필요한 시점에 계산한 성공값을 이후 사용에서 공유한다.
단순한 thunk의 반복 호출과 다르다.
실패한 초기화와 여러 스레드의 접근은 해당 언어와 구현의 구체적인 계약을 확인해야 한다.

### 지연 자료구조

LazyList는 필요한 원소를 계산하면서 그 구조의 계산 결과를 보관할 수 있다.
Iterator는 보통 소비 위치를 앞으로 이동하는 일회성 순회다.
둘 다 모든 원소를 처음부터 계산하지 않을 수 있지만 재사용과 메모리의 의미는 다르다.

---

## 2. 명령형 스타일과 함수형 스타일

### 사용하지 않을 계산까지 실행

```text
비싼 기본값을 먼저 계산한다
설정에 값이 있으면 기존 값을 사용한다
없으면 기본값을 사용한다
```

기본값이 실제로 필요하지 않았어도 비용이나 외부 효과가 발생할 수 있다.
필요한 분기에서만 실행하도록 계산을 지연할 수 있다.
이미 계산한 값을 함수에 넣는 것과 아직 실행하지 않은 작업을 함수에 넣는 것은 다르다.

### thunk 전달

```text
fallback = () => expensive()
필요할 때 fallback()
```

호출을 미루었지만 여러 번 호출하면 비싼 계산도 여러 번 실행된다.
한 번 얻은 결과를 공유하려면 별도의 저장이 필요하다.
이 차이를 모르고 by-name 인자에 비싼 효과를 넣으면 호출 횟수를 놓칠 수 있다.

### 지역 lazy 값

함수 안에서 by-name 인자를 지역 lazy 값에 연결하면 필요한 경우 한 번 계산하여 공유할 수 있다.
아예 사용하지 않으면 계산하지 않는다.
그 공유의 범위는 해당 함수 호출에서 생성한 값의 수명에 달려 있다.

### 무한 입력의 유한 소비

자연수처럼 끝이 없는 논리적인 수열도 앞의 몇 개만 계산하는 방식으로 다룰 수 있다.
하지만 원하는 원소를 찾지 못하는 필터는 끝없이 탐색할 수 있다.
지연된 무한 자료구조와 항상 끝나는 계산은 같은 개념이 아니다.

---

## 3. 왜 이 개념을 사용하는가?

### 필요 없는 계산의 생략

실제로 요구되지 않는 분기의 비용과 효과를 피할 수 있다.
조건부 기본값과 큰 자료구조의 접두사 처리에 유용하다.
무엇이 결과를 강제로 계산하는 연산인지 API 계약을 확인해야 한다.

### 단계적인 생산

전체 입력을 메모리에 모두 만들지 않고 필요한 부분을 생성할 수 있다.
대규모 입력이나 끝이 정해지지 않은 수열의 처리에 도움이 된다.
중간 결과를 보관하는 구조인지 버리는 구조인지에 따라 메모리 사용은 달라진다.

### 공유의 명시

성공한 계산을 한 번만 수행하고 여러 소비자가 같은 값을 사용할 수 있다.
비용 절감과 함께 값의 별칭 공유가 생길 수 있다.
반환값이 가변이면 한 소비자의 변경이 다른 소비자에게 보일 수 있다.

### 실행 시점의 테스트

카운터와 기록을 사용해 생성 시점과 사용 시점의 차이를 확인할 수 있다.
값만 맞는지 검사하는 것보다 효과의 실행 횟수를 정확히 파악할 수 있다.
동시 실행의 안전성은 단일 스레드 테스트와 별도로 검증한다.

### 표현과 실행의 분리

IO나 Free에서 배운 작업 설명과 실행의 구분을 평가 전략에서도 다시 볼 수 있다.
다만 LazyList와 IO가 같은 오류·취소·자원 계약을 가진다는 뜻은 아니다.
공통 아이디어와 구체적인 실행 보장을 구분한다.

---

## 4. Scala에서의 표현

### Scala의 평가 횟수 비교

아래 코드는 인자 표현식이 언제 몇 번 실행되는지 기록한다.
시간 측정에 기대지 않고 호출 횟수와 반환값으로 계약을 확인한다.
LazyList와 Iterator는 소비 이후의 재사용 동작을 따로 검사한다.

<!-- executable:scala -->
```scala
object Chapter53:
  def strictTwice(value: Int): Int = value + value
  def namedTwice(value: => Int): Int = value + value
  def sharedTwice(value: => Int): Int =
    lazy val shared = value
    shared + shared
  def ignore(value: => Int): Int = 0

  def check(): Unit =
    var calls = 0
    def next(): Int =
      calls += 1
      calls
    assert(strictTwice(next()) == 2)
    assert(calls == 1)
    calls = 0
    assert(namedTwice(next()) == 3)
    assert(calls == 2)
    calls = 0
    assert(sharedTwice(next()) == 2)
    assert(calls == 1)
    calls = 0
    assert(ignore(next()) == 0)
    assert(calls == 0)

    lazy val value = next()
    assert(calls == 0)
    assert(value == 1 && value == 1)
    assert(calls == 1)

    var attempts = 0
    lazy val retried: Int =
      attempts += 1
      if attempts == 1 then throw new IllegalStateException("first attempt")
      7
    var failed = false
    try { val observed = retried; assert(observed == 7) }
    catch case _: IllegalStateException => failed = true
    assert(failed && attempts == 1)
    assert(retried == 7 && retried == 7)
    assert(attempts == 2)

    var produced = Vector.empty[Int]
    val stream = LazyList.from(1).map { number =>
      produced = produced :+ number
      number * 2
    }
    assert(produced.isEmpty)
    assert(stream.take(3).toList == List(2, 4, 6))
    assert(produced == Vector(1, 2, 3))
    assert(stream.take(2).toList == List(2, 4))
    assert(produced == Vector(1, 2, 3))
    assert(stream.take(5).toList == List(2, 4, 6, 8, 10))
    assert(produced == Vector(1, 2, 3, 4, 5))

    val iterator = (1 to 3).iterator
    assert(iterator.take(2).toList == List(1, 2))
    assert(iterator.toList == List(3))
    assert(iterator.toList.isEmpty)

    var setting = 10
    val readLater = () => setting
    val captured = setting
    val readSnapshot = () => captured
    setting = 20
    assert(readLater() == 20)
    assert(readSnapshot() == 10)
```

### 실패한 초기화

이 Scala 예제에서는 lazy 값의 첫 초기화가 실패한 뒤 다음 접근에서 다시 시도한다.
성공한 값은 이후 접근에서 재사용한다.
캐시된 실패를 반환하는 별도의 Lazy 구현도 가능하므로 모든 지연 계산이 같은 정책이라고 가정하지
않는다.

### 효과가 있는 지연 수열

원소 생산의 기록은 평가 횟수를 보여 주기 위한 관측 장치다.
실제 설계에서 원소 생산이 외부 요청을 수행한다면 캐시와 재사용이 업무 의미를 바꿀 수 있다.
지연과 효과의 결합을 단순한 성능 최적화로만 취급하지 않는다.

---

## 5. 상태 변경보다 값 변환

### 지연은 스냅샷이 아니다

클로저가 가변 설정을 캡처하면 나중 실행 시점의 설정을 읽을 수 있다.
생성 당시 값을 사용하려면 그 값을 별도로 보관해야 한다.
평가를 미루는 것과 입력을 고정하는 것을 구분한다.

```mermaid
flowchart LR
    A["계산 설명 생성"] --> B["아직 평가하지 않음"]
    B --> C["처음 값이 필요함"]
    C --> D["실제 계산"]
    D --> E["성공값 공유 또는 재실행 정책"]
```

그림의 마지막 단계는 모든 지연 구조에 동일하지 않다.
thunk는 재실행할 수 있고 lazy 값은 성공값을 보관할 수 있다.
Iterator는 소비한 값을 원래 위치로 돌려주지 않을 수 있다.

### 값의 공유

lazy 계산이 리스트나 가변 객체를 반환하면 이후 접근도 같은 객체를 받을 수 있다.
계산 횟수의 절약과 공유 변경의 위험이 함께 생긴다.
불변 결과나 명시적인 복사 정책을 검토한다.

### 공간 누수

LazyList의 시작점을 계속 보관하면 이미 계산한 접두사가 계속 도달 가능한 상태로 남을 수 있다.
지연된 계산이 항상 적은 메모리를 사용한다는 주장은 틀릴 수 있다.
소비 방식과 참조 수명에 따라 자료구조를 선택한다.

---

## 6. 함수 합성과 데이터 흐름

### 지연된 합성

`map`과 `filter`가 실제 입력 소비 때 실행되는지 확인해야 한다.
같은 연산 이름이라도 엄격한 컬렉션과 지연된 컬렉션에서 실행 시점이 다를 수 있다.
예외도 값을 요구하는 위치에서 뒤늦게 발생할 수 있다.

```text
설명 조립 -> 일부 소비 -> 필요한 계산만 수행
```

### 단락 평가

앞 결과만으로 답을 정할 수 있는 계산은 뒤의 입력을 읽지 않을 수 있다.
하지만 구현이 전체 입력을 먼저 만들면 이런 이득을 얻지 못한다.
연산 자체의 의미와 입력 자료구조의 평가 전략을 함께 확인한다.

### 생산성과 종료

무한 수열에서 각 다음 원소를 유한한 단계 안에 만들 수 있는지 확인한다.
지연 구조가 만들어졌다는 사실만으로 원소를 계속 얻을 수 있는 것은 아니다.
원하는 값이 존재하지 않는 검색은 끝나지 않을 수 있다.

### 메모이제이션과의 연결

지연 계산의 결과를 보관하는 것은 메모이제이션의 특수한 형태로 볼 수 있다.
일반적인 메모이제이션은 여러 입력 키와 결과를 관리한다.
다음 장에서 캐시 키와 용량, 실패와 공유의 정책을 더 자세히 다룬다.

---

## 7. 장점과 트레이드오프

### 장점과 트레이드오프

| 구조 | 평가 시점 | 재사용의 의미 |
| --- | --- | --- |
| 엄격한 값 인자 | 호출 전 계산 | 이미 얻은 값 사용 |
| by-name 또는 thunk | 사용할 때 | 여러 번 실행 가능 |
| lazy 값 | 처음 필요할 때 | 성공값 공유 |
| LazyList | 필요한 부분 소비 때 | 계산한 구조 보관 가능 |
| Iterator | 다음 원소를 요구할 때 | 소비 위치가 이동 |

### 오류 위치의 이동

계산을 만든 함수가 끝난 뒤 실제 오류가 발생할 수 있다.
그 시점에 필요한 진단 정보와 자원이 여전히 존재하는지 확인한다.
지연은 오류를 없애는 것이 아니라 관측되는 위치를 바꿀 수 있다.

### 비용의 예측

값에 처음 접근하는 코드가 예상하지 못한 긴 계산을 수행할 수 있다.
성능이 중요한 경계에서 무엇이 평가를 강제하는지 문서화한다.
지연된 초기화 비용을 단순한 필드 읽기 비용으로 가정하지 않는다.

### 필요한 만큼의 지연

작고 확실한 계산까지 전부 thunk로 감싸면 코드와 호출 비용이 늘 수 있다.
사용하지 않는 분기나 큰 입력처럼 실제 이득이 있는 지점을 선택한다.
지연을 전역적인 목표가 아니라 평가 전략의 도구로 사용한다.

---

## 8. 상태와 부수효과의 경계

### 자원 수명

파일을 열어 생성자를 만든 뒤 파일을 닫고 나중에 소비하면 읽기가 실패할 수 있다.
생산과 소비가 자원 범위 안에서 일어나도록 설계한다.
지연된 값의 수명과 그 값이 사용하는 자원의 수명을 맞춘다.

### 취소와 정리

생성자 소비를 중단하면 정리 코드가 언제 실행되는지 확인해야 한다.
명시적인 닫기나 범위 관리가 필요한 경우가 있다.
더 이상 값을 요구하지 않는 것과 외부 작업을 취소한 것은 다르다.

### 동시 초기화

여러 스레드가 같은 지연값을 처음 읽는 상황은 구현의 동기화 계약을 따른다.
단일 스레드의 작은 Lazy 래퍼가 자동으로 안전한 초기화를 제공하지 않는다.
한 번 실행 보장과 결과 저장의 일관성도 구분해야 한다.

### 민감정보와 보관

성공값을 공유하면 비밀값이나 큰 데이터가 오래 남을 수 있다.
필요한 보관 기간과 참조 수명을 관리한다.
재사용 비용 절감과 정보 보관 위험을 함께 평가한다.

---

## 9. Python에서 적용하기

### Python의 지연과 성공값 공유

다음 Lazy 모형은 성공값만 보관하고 예외가 발생하면 다음 접근에서 다시 시도한다.
준비되지 않은 상태와 실제 결과 `None`을 다른 타입으로 표현한다.
이 구현은 단일 스레드 교육용이며 동기화와 재진입 검사를 제공하지 않는다.

<!-- executable:python -->
```python
from collections.abc import Callable, Iterator
from dataclasses import dataclass
from itertools import islice
from typing import Generic, TypeVar

A = TypeVar("A")
B = TypeVar("B")


@dataclass(frozen=True)
class Empty:
    pass


@dataclass(frozen=True)
class Ready(Generic[A]):
    value: A


class Lazy(Generic[A]):
    def __init__(self, compute: Callable[[], A]) -> None:
        self._compute = compute
        self._state: Empty | Ready[A] = Empty()

    def get(self) -> A:
        if isinstance(self._state, Ready):
            return self._state.value
        value = self._compute()
        self._state = Ready(value)
        return value

    def map(self, function: Callable[[A], B]) -> "Lazy[B]":
        return Lazy(lambda: function(self.get()))


def test_evaluation_and_sharing() -> None:
    calls: list[int] = []

    def next_value() -> int:
        value = len(calls) + 1
        calls.append(value)
        return value

    thunk = next_value
    assert calls == []
    assert thunk() + thunk() == 3
    assert calls == [1, 2]
    calls.clear()
    shared = Lazy(next_value)
    mapped = shared.map(lambda value: value * 2)
    assert calls == []
    assert mapped.get() == 2
    assert mapped.get() == 2
    assert shared.get() == 1
    assert calls == [1]
    none_calls: list[str] = []

    def none_value() -> None:
        none_calls.append("called")
        return None

    none = Lazy(none_value)
    assert none.get() is None and none.get() is None
    assert none_calls == ["called"]


def test_retry_policy() -> None:
    attempts = 0

    def compute() -> int:
        nonlocal attempts
        attempts += 1
        if attempts == 1:
            raise ValueError("first attempt")
        return 7

    value = Lazy(compute)
    try:
        value.get()
    except ValueError:
        pass
    else:
        raise AssertionError("first attempt unexpectedly succeeded")
    assert value.get() == 7 and value.get() == 7
    assert attempts == 2


def test_generator_is_one_shot() -> None:
    produced: list[int] = []

    def numbers() -> Iterator[int]:
        for value in range(1, 6):
            produced.append(value)
            yield value * 2

    stream = numbers()
    assert produced == []
    assert list(islice(stream, 3)) == [2, 4, 6]
    assert produced == [1, 2, 3]
    assert list(islice(stream, 2)) == [8, 10]
    assert list(stream) == []
    assert produced == [1, 2, 3, 4, 5]
    events: list[str] = []

    def resource_scope() -> Iterator[int]:
        try:
            yield 1
            yield 2
        finally:
            events.append("closed")

    resource = resource_scope()
    assert next(resource) == 1
    resource.close()
    assert events == ["closed"]


if __name__ == "__main__":
    test_evaluation_and_sharing()
    test_retry_policy()
    test_generator_is_one_shot()
```

### 생성자와 LazyList

같은 Python 생성자 객체의 앞 세 값을 읽은 뒤 다음 두 값을 읽으면 남은 값이 나온다.
처음 두 값이 다시 나오는 Scala LazyList의 재사용 예제와 다르다.
생성자를 새로 만드는 것과 이미 계산한 접두사를 공유하는 것도 다른 계약이다.

---

## 10. Python의 표현 한계

### 예외 정책의 선택

이 Lazy 모형은 실패를 저장하지 않는다.
실패도 결과값으로 만들어 저장하는 다른 구현이 가능하다.
어느 정책이 필요한지 외부 효과와 재시도의 의미를 기준으로 정한다.

### 재진입

계산 중 자기 자신의 `get`을 다시 부르면 무한 재귀가 발생할 수 있다.
제품용 구현에는 초기화 중 상태와 순환 의존성 검사가 필요할 수 있다.
두 상태만 가진 예제의 보장 범위를 넘어서지 않는다.

### 동시성

Python 예제에는 잠금이 없다.
여러 실행이 동시에 접근하면 계산이 여러 번 수행될 수 있다.
인터프리터의 내부 동작에 기대어 한 번 실행을 보장했다고 주장하지 않는다.

### 깊은 불변성

Ready에 들어간 값이 리스트라면 같은 리스트가 여러 접근에서 공유된다.
래퍼가 있다고 결과 객체가 불변이 되는 것은 아니다.
가변 결과의 공유와 복사 정책을 별도로 정한다.

---

## 11. 핵심 정리

### 핵심 결론

지연 평가는 결과가 필요한 시점까지 계산을 미루는 전략이다.
by-name과 thunk의 재실행, lazy 값의 공유, Iterator의 일회성 소비를 구분해야 한다.
입력 캡처와 예외 재시도, 공간 사용은 평가 시점과 별도의 계약이다.
자원 수명과 동시 초기화는 실제 구현에 맞게 설계해야 한다.

### 연습 1: 두 번 사용한 인자

by-name 인자를 함수 안에서 두 번 사용했다.
그 계산은 반드시 한 번만 실행되는가?

**해설.** 아니다. 표현식을 사용할 때마다 실행될 수 있다.
한 번 얻은 성공값을 공유하려면 지역 lazy 값 같은 별도 구조를 사용할 수 있다.
지연과 메모이제이션을 구분한다.

### 연습 2: 생성자의 재사용

같은 생성자 객체에서 앞 세 값을 읽은 뒤 앞 두 값을 다시 읽고 싶다.
단순히 두 번 더 `next`하면 되는가?

**해설.** 보통 다음 소비 위치의 값이 나온다.
재실행 가능한 생성 함수나 결과 보관이 필요하다.
일회성 순회와 접두사 공유 자료구조를 구분한다.

### 연습 3: 지연된 설정

클로저를 만든 뒤 설정 객체를 변경했다.
나중 계산이 생성 당시 설정을 반드시 사용하는가?

**해설.** 실제로 무엇을 캡처했는지에 달려 있다.
가변 객체를 참조하면 실행 시점의 값을 볼 수 있다.
생성 당시 사실이 필요하면 명시적인 스냅샷을 보관한다.

### 연습 4: 파일 수명

열린 파일을 사용하는 생성자를 반환하고 파일을 닫았다.
어떤 문제가 생길 수 있는가?

**해설.** 실제 읽기가 나중 소비 시점에 발생하여 닫힌 파일을 사용할 수 있다.
생산과 소비를 적절한 자원 범위에 두어야 한다.
계산 설명의 수명과 자원의 수명을 함께 설계한다.

### 다음 장과 참고 자료

다음 장은 여러 입력에 대한 계산 결과를 재사용하는 메모이제이션을 다룬다.
캐시 키와 보관 범위, 가변 결과 공유를 별도의 정책으로 정리한다.

[Scala 공식 문서: Lazy Vals Initialization](https://docs.scala-lang.org/scala3/reference/changed-features/lazy-vals-init.html)
[Scala API: LazyList](https://www.scala-lang.org/api/current/scala/collection/immutable/LazyList.html)
[Python 공식 문서: Generator Expressions](https://docs.python.org/3.14/reference/expressions.html#generator-expressions)
