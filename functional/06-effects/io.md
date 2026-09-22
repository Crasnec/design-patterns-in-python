# 41장. IO

Reader, State, Writer는 환경과 상태, 로그를 값의 구조로 표현했다.
실제 프로그램은 파일을 열고 네트워크를 호출하는 외부 작업도 수행해야 한다.
IO는 그런 작업을 지금 실행하는 것과 나중에 실행할 계산으로 표현하는 것을 구분하는 데 사용된다.

이번 장은 지연된 동기 계산을 담는 작은 IO 모형을 직접 구현한다.
이 모형은 실행 시점을 이해하기 위한 것이며 제품용 효과 실행기를 대체하지 않는다.
비동기 스케줄링, 스택 안전성, 취소와 자원 관리의 보장 범위를 분명히 나누어 설명한다.

---

## 1. 개념과 기본 구분

### 실행할 계산을 값으로 표현한다

일반적인 효과 호출은 호출한 시점에 외부 작업을 수행할 수 있다.
지연된 IO는 작업의 설명을 값으로 만들고 명시적인 실행 경계에서 수행할 수 있다.
어떤 IO 구현인지에 따라 실제 표현과 실행 기능은 달라진다.

```text
효과 호출:     read() -> 실제 읽기와 결과
지연된 설명:   delay(() => read()) -> IO[Result]
명시적 실행:   IO[Result] -> 실제 읽기와 결과
```

지연은 효과가 사라졌다는 뜻이 아니다.
실행 시 외부 상태를 읽고 바꾸며 실패할 수 있다.
효과의 설명과 효과의 수행을 다른 단계로 다루는 것이다.

### `pure`와 `delay`

`pure`는 이미 가진 값을 IO 결과로 넣는 연산이다.
`delay`는 아직 실행하지 않은 계산을 보관하는 연산이다.
엄격하게 평가되는 인자를 `pure`에 전달하면 인자 계산이 먼저 실행될 수 있다.

### 결과 보관과 작업 보관

이미 읽은 값을 보관하면 다시 사용해도 읽기를 반복하지 않는다.
읽기 작업을 보관하면 실행할 때마다 새 읽기가 발생할 수 있다.
지연, 재실행, 메모이제이션은 서로 다른 성질이다.

### IO와 순수성

효과를 기술하는 값을 조립하는 부분은 순수하게 설계할 수 있다.
그 값을 해석하여 실행하는 부분은 실제 효과를 수행한다.
함수 하나를 감쌌다는 이유로 전체 프로그램이 부수효과가 없어진다고 말하지 않는다.

---

## 2. 명령형 스타일과 함수형 스타일

### 즉시 실행하는 코드

```text
fileText = readFile()
result = parse(fileText)
```

파일 읽기는 그 줄에 도달했을 때 실행된다.
함수를 정의하는 것과 호출하는 것을 구분해야 한다.
실행 결과를 변수에 넣으면 이후에는 이미 읽은 값을 사용할 수 있다.

### 계산을 조립한다

```text
program = delay(readFile).map(parse)
```

IO의 `map`이 지연 조합을 제공한다면 이 줄은 아직 파일을 읽지 않는다.
실제 실행 경계에서 읽기와 파싱이 이어진다.
구체적인 구현이 이런 계약을 제공하는지 확인해야 한다.

### 잘못된 지연

```text
pure(readFile())
```

엄격한 인자 평가에서는 `readFile()`이 먼저 실행된다.
그 후 읽은 결과가 IO에 들어간다.
이것을 지연된 파일 읽기라고 설명하면 실행 시점과 실패 경계가 틀어진다.

### 이미 실행한 결과의 반복 사용

`Try`에 읽기를 넣으면 그 시점에 실행된 성공·실패 결과가 만들어질 수 있다.
지연된 IO는 실행할 때 성공·실패가 결정될 수 있다.
둘의 `map`과 `flatMap` 모양이 비슷해도 실행 의미를 혼동하지 않는다.

---

## 3. 왜 이 개념을 사용하는가?

### 실행 경계의 명시

업무 흐름을 먼저 조립하고 실제 실행은 프로그램의 바깥 경계에 모을 수 있다.
호출자가 언제 외부 작업이 시작되는지 이해하기 쉬워진다.
중간 함수에서 실행기를 반복 호출하면 이 이점이 줄어든다.

### 자원과 실패의 조합

효과 설명 안에 획득·사용·해제 범위를 둘 수 있다.
실패와 취소에서도 정리되도록 설계할 수 있지만 실제 실행기의 계약이 필요하다.
작은 thunk 래퍼가 이런 보장을 자동으로 제공하는 것은 아니다.

### 테스트 가능한 실행 시점

카운터나 메모리 출력 모형으로 조립할 때와 실행할 때의 호출 횟수를 검사한다.
재실행이 예상대로 새 효과를 수행하는지도 확인한다.
실제 외부 어댑터의 실패 모드는 별도 통합 테스트가 필요하다.

### 오류 처리의 위치

실행 시 예외를 결과로 포착하는 어댑터를 만들 수 있다.
무엇을 예상 오류로 번역하고 무엇을 전파할지 구분한다.
종료와 취소 신호를 무조건 정상 실패값으로 바꾸지 않는다.

### 실행 모델의 분리

업무 계산의 조합과 스레드·스케줄러·취소 처리를 분리할 수 있다.
그러나 실행기가 제공하는 보장을 충분히 이해해야 한다.
라이브러리 선택은 인터페이스 이름이 아니라 실제 실행 계약을 기준으로 한다.

---

## 4. Scala에서의 표현

### Scala의 동기 IO 모형

아래 구현은 함수를 보관했다가 호출하는 작은 모형이다.
Scala 표준 `Using`은 자원 범위를 설명하기 위해 별도로 사용한다.
이 IO 클래스 자체에는 비동기 실행, 취소, 스택 안전한 해석 기능이 없다.

<!-- executable:scala -->
```scala
object Chapter41:
  import scala.util.Using
  import scala.util.control.NonFatal

  final class IO[A] private (thunk: () => A):
    def unsafeRunSync(): A = thunk()
    def map[B](f: A => B): IO[B] = IO.delay(f(thunk()))
    def flatMap[B](f: A => IO[B]): IO[B] = IO.delay(f(thunk()).unsafeRunSync())
    def attempt: IO[Either[Throwable, A]] = IO.delay {
      try Right(thunk())
      catch case NonFatal(error) => Left(error)
    }

  object IO:
    def pure[A](value: A): IO[A] = new IO(() => value)
    def delay[A](body: => A): IO[A] = new IO(() => body)

  final class MemoryResource extends AutoCloseable:
    var closed = false
    def read(): String =
      require(!closed)
      "data"
    def close(): Unit = closed = true

  def check(): Unit =
    var calls = Vector.empty[String]
    def source(): Int =
      calls = calls :+ "source"
      7
    val delayed = IO.delay(source())
    val program = delayed.map(_ + 1).flatMap(value => IO.pure(value * 2))
    assert(calls.isEmpty)
    assert(program.unsafeRunSync() == 16)
    assert(calls == Vector("source"))
    assert(program.unsafeRunSync() == 16)
    assert(calls == Vector("source", "source"))

    val alreadyComputed = IO.pure(source())
    assert(calls.size == 3)
    assert(alreadyComputed.unsafeRunSync() == 7)
    assert(alreadyComputed.unsafeRunSync() == 7)
    assert(calls.size == 3)

    var failures = 0
    val failing = IO.delay[Int] {
      failures += 1
      throw new IllegalArgumentException("invalid input")
    }.attempt
    assert(failures == 0)
    assert(failing.unsafeRunSync().left.exists(_.isInstanceOf[IllegalArgumentException]))
    assert(failures == 1)

    var resources = Vector.empty[MemoryResource]
    val read = IO.delay {
      val resource = new MemoryResource
      resources = resources :+ resource
      Using.resource(resource)(_.read())
    }
    assert(resources.isEmpty)
    assert(read.unsafeRunSync() == "data")
    assert(read.unsafeRunSync() == "data")
    assert(resources.size == 2 && resources.forall(_.closed))
    val failingResource = new MemoryResource
    val failedRead = IO.delay {
      Using.resource(failingResource)(_ => throw new IllegalStateException("read failed"))
    }.attempt
    assert(failedRead.unsafeRunSync().isLeft)
    assert(failingResource.closed)

    var interrupted = false
    try IO.delay(throw new InterruptedException("stop")).attempt.unsafeRunSync()
    catch case _: InterruptedException => interrupted = true
    assert(interrupted)
```

### `attempt`의 실행 시점

실패할 IO를 만들고 `attempt`로 감싸도 실패 계산은 아직 실행되지 않는다.
실행할 때 포착 대상 예외가 왼쪽 값으로 바뀐다.
이 결과를 도메인 오류로 번역하는 정책은 별도로 둘 수 있다.

### 자원의 생성 위치

반복 실행할 읽기 작업은 실행마다 새 자원을 획득한다.
이미 닫힌 자원 하나를 지연 계산 밖에서 만들어 재사용하면 문제가 될 수 있다.
실패 정리 테스트의 단일 자원은 한 번만 실행하는 모형이라는 범위를 가진다.

---

## 5. 상태 변경보다 값 변환

### 조립 단계와 실행 단계

조립 단계에서는 외부 작업을 어떤 순서로 이어 갈지 표현한다.
실행 단계에서는 그 작업이 실제로 일어나고 실패가 발생할 수 있다.
실행기를 호출하는 위치를 분명히 해야 두 단계의 차이를 유지할 수 있다.

```mermaid
flowchart LR
    A["작업 설명 생성"] --> B["map과 flatMap으로 조립"]
    B --> C["실행 경계"]
    C --> D["외부 작업"]
    D --> E["결과 또는 실패"]
```

작업 설명 안의 함수가 가변 변수를 캡처하면 실행 시점의 값을 읽을 수 있다.
설명 생성 시점의 스냅샷을 원하면 명시적인 값을 캡처해야 한다.
지연 계산과 클로저 캡처의 의미를 함께 검토한다.

### 결과의 공유

같은 IO 값을 두 번 사용한다고 같은 실행 결과가 자동 공유되는 것은 아니다.
공유나 캐시가 필요하면 명시적인 연산과 수명 정책이 필요하다.
실제 라이브러리의 메모이제이션 기능도 실패와 취소의 처리 의미를 확인해야 한다.

### 값 안의 자원

IO가 열린 파일 핸들을 반환하면 호출자가 그 수명을 관리해야 할 수 있다.
자원 범위를 벗어난 핸들을 반환하지 않도록 설계하는 것이 중요하다.
지연 실행과 자원 소유권은 서로 다른 계약이다.

---

## 6. 함수 합성과 데이터 흐름

### `map`과 `flatMap`

일반 결과 변환은 `map`으로 조립한다.
다음 효과가 앞 결과에 의존하면 `flatMap`으로 조립한다.
이 연결이 즉시 실행되는지 나중 실행 구조를 만드는지는 구체적인 IO 계약이다.

```text
IO[A] + (A -> B)     -> IO[B]
IO[A] + (A -> IO[B]) -> IO[B]
```

### 법칙의 관측 기준

효과를 가진 계산의 동등성은 객체 주소를 비교해서 판단하지 않는다.
실행 결과와 필요한 효과 관측이 같은지 의미를 정의해야 한다.
실행 시간을 포함한 모든 성능 특성이 같다는 뜻은 아니다.

### 재시도

같은 작업 설명을 다시 실행하는 것이 재시도의 한 형태다.
외부 작업이 이미 일부 성공했다면 반복 실행이 중복 효과를 만들 수 있다.
재시도 횟수와 지연뿐 아니라 멱등성과 부분 성공 확인이 필요하다.

### 오류 번역의 계층

기술 예외를 포착하는 것과 업무 오류를 만드는 것을 구분한다.
예상하지 못한 버그를 정상적인 입력 오류로 바꾸면 운영 진단이 어려워진다.
작은 외부 호출 경계에서 필요한 실패만 번역하는 원칙을 유지한다.

---

## 7. 장점과 트레이드오프

### 장점과 트레이드오프

| 구조 | 이점 | 주의점 |
| --- | --- | --- |
| 지연된 작업 설명 | 실행 시점 명확 | 캡처한 환경의 변화 |
| 결과와 작업 구분 | 재실행 정책 가시화 | 캐시와 혼동 |
| 효과 조합 | 순서와 의존성 표현 | 실행기의 보장 필요 |
| 실행 시 오류 포착 | 실패 경계 정리 | 버그·취소의 은폐 위험 |
| 자원 범위 | 정리 정책 명시 | 실행 밖으로 자원 유출 |

### 작은 모형의 한계

예제는 중첩된 함수 호출로 `flatMap`을 실행한다.
수십만 단계의 연결에서 스택 안전성을 보장하지 않는다.
취소와 비동기 작업을 다루는 실행기도 구현하지 않았다.
이런 기능이 필요하면 검증된 실행기와 자원 추상화의 계약을 사용해야 한다.

### 래핑의 남용

이미 순수한 계산까지 매 단계마다 외부 효과처럼 취급할 필요는 없다.
순수한 함수는 일반 값 변환으로 유지하고 실제 효과 경계에서 조립할 수 있다.
모든 함수의 반환 타입을 복잡하게 만드는 것이 목적이 아니다.

### 디버깅과 추적

지연된 실행은 오류 발생 위치와 작업을 조립한 위치를 떨어뜨릴 수 있다.
실행기의 추적 정보와 관측 도구가 중요해진다.
예제 thunk 모형은 제품용 진단 기능까지 제공하지 않는다.

---

## 8. 상태와 부수효과의 경계

### 자원 획득·사용·해제

외부 자원은 정상 완료뿐 아니라 실패와 취소에서도 정리되어야 한다.
획득 실패와 해제 실패가 함께 있을 때 어떤 오류를 보고할지도 중요하다.
직접 작성한 단순 `finally`가 원래 오류를 덮어쓰지 않는지 검토해야 한다.

### 취소

동기 함수 래퍼에는 실행 중 작업을 안전하게 중단하는 기능이 없다.
스레드를 멈추거나 네트워크 요청을 취소하려면 구체적인 실행 프로토콜이 필요하다.
취소 가능한 IO라는 표현은 실제 라이브러리의 보장 범위를 확인한 뒤 사용한다.

### 원자성과 외부 상태

여러 IO를 연결해도 자동으로 하나의 데이터베이스 트랜잭션이 되지는 않는다.
한 작업이 성공한 뒤 다음 작업이 실패할 수 있다.
원자성, 보상, 멱등성은 해당 외부 시스템의 계약과 함께 설계한다.

### 실행 권한

작업 설명에 파일 경로나 토큰이 들어갈 수 있다.
실행 전후의 공개와 보관 범위를 관리해야 한다.
효과를 값으로 만들었다는 사실이 민감정보를 무해하게 만드는 것은 아니다.

---

## 9. Python에서 적용하기

### Python의 지연된 동기 계산

Python에서는 호출 가능한 객체를 보관하여 같은 실행 구분을 표현할 수 있다.
아래 `IO`는 교육용 동기 모형이며 코루틴 실행기나 취소 기능을 제공하지 않는다.
실행 시 예외를 포착하되 제어 신호를 무조건 삼키지 않는다.

<!-- executable:python -->
```python
from collections.abc import Callable
from dataclasses import dataclass
from io import StringIO
from typing import Generic, TypeVar

A = TypeVar("A")
B = TypeVar("B")


@dataclass(frozen=True)
class Success(Generic[A]):
    value: A


@dataclass(frozen=True)
class Failure:
    error: Exception


@dataclass(frozen=True)
class IO(Generic[A]):
    thunk: Callable[[], A]

    def unsafe_run_sync(self) -> A:
        return self.thunk()

    def map(self, function: Callable[[A], B]) -> "IO[B]":
        return IO(lambda: function(self.thunk()))

    def flat_map(self, function: Callable[[A], "IO[B]"]) -> "IO[B]":
        return IO(lambda: function(self.thunk()).unsafe_run_sync())

    def attempt(self) -> "IO[Success[A] | Failure]":
        def run() -> Success[A] | Failure:
            try:
                return Success(self.thunk())
            except Exception as error:
                return Failure(error)
        return IO(run)


def pure(value: A) -> IO[A]:
    return IO(lambda: value)


def delay(function: Callable[[], A]) -> IO[A]:
    return IO(function)


def test_evaluation() -> None:
    calls: list[str] = []

    def source() -> int:
        calls.append("source")
        return 7

    program = delay(source).map(lambda value: value + 1).flat_map(lambda value: pure(value * 2))
    assert calls == []
    assert program.unsafe_run_sync() == 16
    assert program.unsafe_run_sync() == 16
    assert calls == ["source", "source"]
    completed = pure(source())
    assert len(calls) == 3
    assert completed.unsafe_run_sync() == 7
    assert completed.unsafe_run_sync() == 7
    assert len(calls) == 3


def test_failure_and_resources() -> None:
    resources: list[StringIO] = []

    def read() -> str:
        resource = StringIO("data")
        resources.append(resource)
        with resource:
            return resource.read()

    program = delay(read)
    assert resources == []
    assert program.unsafe_run_sync() == "data"
    assert program.unsafe_run_sync() == "data"
    assert len(resources) == 2 and all(resource.closed for resource in resources)

    def fail() -> int:
        with StringIO("data") as resource:
            resources.append(resource)
            raise ValueError("read failed")

    captured = delay(fail).attempt()
    assert len(resources) == 2
    result = captured.unsafe_run_sync()
    assert isinstance(result, Failure) and isinstance(result.error, ValueError)
    assert len(resources) == 3 and resources[-1].closed

    def stop() -> int:
        raise KeyboardInterrupt("test signal")

    try:
        delay(stop).attempt().unsafe_run_sync()
    except KeyboardInterrupt:
        pass
    else:
        raise AssertionError("control signal was swallowed")


if __name__ == "__main__":
    test_evaluation()
    test_failure_and_resources()
```

### 코루틴을 반환하는 경우

이 모형에 코루틴 객체를 반환하는 함수를 넣으면 그 객체를 얻을 뿐 내부 비동기 작업을 실행한 것은
아닐 수 있다.
실제 비동기 실행은 `await`와 이벤트 루프의 계약을 따른다.
동기 thunk와 비동기 실행기를 같은 추상화라고 가정하지 않는다.

---

## 10. Python의 표현 한계

### 표준 IO 타입과의 차이

Python 표준 라이브러리의 `IO` 관련 타입 이름은 파일 객체의 입출력 인터페이스를 뜻하는 경우가
있다.
이 장의 교육용 효과 래퍼와 같은 클래스가 아니다.
이름이 겹칠 때 모듈과 실제 연산을 확인한다.

### 타입 주석의 보장

`Callable[[], A]`는 호출 결과의 타입을 설명하지만 효과 종류를 제한하지 않는다.
파일 삭제나 네트워크 호출도 숨겨 들어갈 수 있다.
실제 권한과 허용 작업은 별도 인터페이스와 실행 경계에서 관리한다.

### 취소와 스택

작은 람다 래퍼는 안전한 취소, 스레드 중단, 긴 연결의 스택 안전성을 제공하지 않는다.
이 기능들이 필요하면 실행 구조를 더 설계해야 한다.
교육용 코드가 실행된다는 사실을 제품용 런타임의 완성으로 확대하지 않는다.

### 캐시와 재실행

같은 IO 객체를 다시 호출하면 저장된 함수를 다시 실행한다.
원하는 정책이 한 번 실행한 결과의 공유라면 명시적인 캐시와 수명 관리가 필요하다.
외부 상태가 변할 때 캐시가 여전히 유효한지도 확인한다.

---

## 11. 핵심 정리

### 핵심 결론

IO는 외부 작업의 설명과 실제 실행을 분리하는 데 사용된다.
`pure`의 값 주입과 `delay`의 계산 지연을 구분해야 한다.
지연된 작업의 재실행은 이미 계산한 결과의 반복 사용과 다르다.
자원, 취소, 스택 안전성, 원자성은 실제 실행기의 별도 계약이다.

### 연습 1: `pure`의 오해

`pure(read_file())`를 지연된 읽기라고 설명했다.
어떤 실행 시점이 잘못되었는가?

**해설.** 엄격한 인자 평가에서는 읽기가 먼저 실행된다.
`pure`는 이미 얻은 값을 보관한다.
아직 실행하지 않은 함수를 지연 연산에 전달해야 한다.

### 연습 2: 두 번 실행

같은 IO 객체를 두 번 실행했더니 카운터가 두 번 증가했다.
이것은 반드시 구현 버그인가?

**해설.** 작업 설명을 재실행하는 계약이라면 정상이다.
자동 메모이제이션을 가정해서는 안 된다.
한 번 실행한 결과를 공유하려는 요구는 별도로 표현한다.

### 연습 3: 자원 획득 위치

지연 계산 밖에서 연 파일을 IO 실행에서 사용하고 닫았다.
두 번째 실행이 실패할 수 있는 이유를 설명하라.

**해설.** 이미 닫힌 같은 자원을 재사용할 수 있기 때문이다.
반복 가능한 작업은 실행마다 필요한 자원을 적절히 획득해야 한다.
자원 수명과 지연 계산의 수명을 맞춘다.

### 연습 4: 실행기의 보장

`map`과 `flatMap`을 구현했으니 비동기 취소와 스택 안전성도 확보되었다는 주장에 반론하라.

**해설.** 조합 연산의 존재만으로 실행기 기능이 생기지는 않는다.
동기 함수 중첩은 그런 보장을 제공하지 않을 수 있다.
필요한 실행 계약을 실제 구현과 테스트로 확인해야 한다.

### 다음 장과 참고 자료

다음 장은 실제 외부 기능을 어떤 인터페이스로 받아 계산과 조립할지 의존성 주입 관점에서 다룬다.
효과의 설명뿐 아니라 효과를 수행할 능력의 전달을 설계한다.

[Cats Effect 공식 문서: Introduction](https://typelevel.org/cats-effect/docs/getting-started)
[Cats Effect 공식 문서: Resource](https://typelevel.org/cats-effect/docs/std/resource)
[Scala API: Using](https://www.scala-lang.org/api/current/scala/util/Using$.html)
[Python 공식 문서: contextlib](https://docs.python.org/3.14/library/contextlib.html)
