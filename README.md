# 파이썬으로 배우는 디자인 패턴 (Python Design Patterns)

이 저장소는 객체 지향 프로그래밍의 핵심 디자인 패턴(Design Patterns)을 **파이썬(Python) 예제 코드**로 구현하고 설명하는 공간입니다.

효율적이고 확장성 있는 소프트웨어를 설계하기 위한 대표적인 디자인 패턴 23가지를 생성, 구조, 행위 유형별로 분류하여 제공합니다.

---

## 목차

### 읽기 전에

클래스 다이어그램을 알지 못한다면 다음 문서부터 참고하세요.

* [클래스 다이어그램 (Class Diagram)](object/class-diagram.md)

이 문서는 단순한 패턴 암기를 넘어 **왜 이 패턴이 필요한가**부터 **현대적 관점에서의 깊이 있는 해석**까지 단계별로 다룹니다.

이 문서의 일부분에서는 객체 지향 언어를 이용하여 프로그램을 설계할 때 지켜야 할 원칙을 설명합니다. 만약 다음 단어를 알지 못한다면 다음 문서를 참고하세요.

* 다섯 약어 SRP, OCP, LSP, ISP, DIP의 본래 의미
* YAGNI, DRY, ... 등 약어
* [객체 지향 설계 원칙](object/oop-design-principle.md)

### 목적별 추천 읽기 코스

* **초급 코스 (빠른 개념 잡기)**
  * `1. 패턴이 없을 때 발생하는 문제점`
  * `2. 패턴으로 해결하기`
  * `6. 파이썬 예제 코드`
  * 코드가 만드는 문제 상황을 먼저 살펴본 뒤, 이를 어떻게 해결하는지 예제 코드로 확인하세요.

* **중급 코스 (실무 설계 및 판단)**
  * `3. 장점, 단점 및 트레이드오프` 
  * `4. 파이썬 오픈소스 예시`
  * 트레이드오프를 체크하여 실제 프로젝트 도입 여부를 결정하세요.

* **고급 코스 (아키텍처 및 패러다임 재해석)**
  * `부록 (Appendix): 현대적 타입 시스템과 관점의 재해석`
  * 전통적인 OOP 디자인 패턴을 깊이 있게 재해석합니다. *(내용이 다소 까다로우니 마음의 준비를 하고 읽으시는 것을 권장합니다!)*

### 1. 생성 패턴 (Creational Patterns)

객체 생성 메커니즘을 다루며, 상황에 맞는 적절한 객체를 생성하도록 돕습니다.

* [싱글턴 (Singleton)](object/creational/singleton.md)
* [팩토리 메서드 (Factory Method)](object/creational/factory-method.md)
* [추상 팩토리 (Abstract Factory)](object/creational/abstract-factory.md)
* [빌더 (Builder)](object/creational/builder.md)
* [프로토타입 (Prototype)](object/creational/prototype.md)

### 2. 구조 패턴 (Structural Patterns)

클래스나 객체를 조합해 더 큰 구조를 만드는 방법을 다룹니다.

* [어댑터 (Adapter)](object/structural/adapter.md)
* [브리지 (Bridge)](object/structural/bridge.md)
* [컴포지트 (Composite)](object/structural/composite.md)
* [데코레이터 (Decorator)](object/structural/decorator.md)
* [파사드 (Facade)](object/structural/facade.md)
* [플라이웨이트 (Flyweight)](object/structural/flyweight.md)
* [프록시 (Proxy)](object/structural/proxy.md)

### 3. 행위 패턴 (Behavioral Patterns)

객체 간의 책임 분배와 알고리즘, 상호작용을 효율적으로 정의합니다.

* [책임 연쇄 (Chain of Responsibility)](object/behavioral/chain-of-responsibility.md)
* [커맨드 (Command)](object/behavioral/command.md)
* [인터프리터 (Interpreter)](object/behavioral/interpreter.md)
* [이터레이터 (Iterator)](object/behavioral/iterator.md)
* [미디에이터 (Mediator)](object/behavioral/mediator.md)
* [메멘토 (Memento)](object/behavioral/memento.md)
* [옵저버 (Observer)](object/behavioral/observer.md)
* [스테이트 (State)](object/behavioral/state.md)
* [전략 (Strategy)](object/behavioral/strategy.md)
* [템플릿 메서드 (Template Method)](object/behavioral/template-method.md)
* [방문자 (Visitor)](object/behavioral/visitor.md)

---

## 함수형 디자인 패턴 (Functional Design Patterns)

후반부에서는 파이썬을 활용한 **함수형 디자인 패턴**을 다룹니다. 불변성, 순수 함수, 타입 시스템 및 효과 제어를 바탕으로 프로그램을 더 안전하고 간결하게 작성하는 기법을 배웁니다.

> **안내**: 본 문서에서 다루는 일부 함수형 패턴은 파이썬 언어 자체에서 직접 지원하지 않습니다. 이해를 돕기 위해 파이썬에 특정 함수형 문법이 포함되어 있다고 전제한 **가상의 언어 스펙**을 사용하여 작성되었습니다.

### 1. 함수 구성 (Function Composition)

작은 단위의 함수들을 조합하여 복잡한 연산 흐름을 만들어냅니다.

* [**Function Composition**](functional/function-composition/function-composition.md): 작은 함수를 연결해 큰 연산 구성
* [**Pipeline / Pipe**](functional/function-composition/pipeline-pipe.md): 데이터를 함수 체인으로 순차 전달
* [**Partial Application**](functional/function-composition/partial-application.md): 함수 인자의 일부를 미리 고정하여 재사용
* [**Currying**](functional/function-composition/currying.md): 다중 인자 함수를 단일 인자 함수들의 연쇄로 변환
* [**Higher-Order Function**](functional/function-composition/higher-order-function.md): 함수를 인자로 받거나 결과로 반환

### 2. 데이터 처리 (Data Processing)

컬렉션과 데이터 스트림을 선언적으로 변환하고 축약합니다.

* [**Map**](functional/data-processing/map.md): 각 원소를 지정한 함수로 변환
* [**Filter**](functional/data-processing/filter.md): 조건에 맞는 원소만 선별
* [**Fold / Reduce**](functional/data-processing/fold-reduce.md): 컬렉션을 하나의 값으로 축약
* [**Scan**](functional/data-processing/scan.md): Fold의 중간 연산 과정 결과를 유지하며 출력
* [**Zip**](functional/data-processing/zip.md): 여러 컬렉션을 동일 인덱스끼리 대응시켜 결합
* [**Unfold**](functional/data-processing/unfold.md): 초기 상태에서 출발해 순차적으로 데이터 구조 생성

### 3. 재귀 (Recursion)

반복문 대신 재귀적 구조를 안전하고 명확하게 다룹니다.

* [**Tail Recursion**](functional/recursion/tail-recursion.md): 스택 오버플로우 없이 반복을 꼬리 재귀로 표현
* [**Recursion Schemes**](functional/recursion/recursion-schemes.md): 재귀 구조 자체와 실제 데이터 처리 로직을 분리

### 4. 타입 및 오류 처리 (Type & Error Handling)

Null 예외나 실행 오류를 타입 시스템 내에서 안전하게 값으로 다룹니다.

* [**Option / Maybe**](functional/type-error-handling/option-maybe.md): 값의 부재(Null/None)를 타입으로 명시
* [**Either / Result**](functional/type-error-handling/either-result.md): 성공과 실패 결과를 명시적인 값으로 표현
* [**Validation**](functional/type-error-handling/validation.md): 여러 연산 과정의 오류를 한 번에 누적하여 검증

### 5. 효과 제어 (Effect Management)

부수효과(Side Effect)나 외부 환경, 상태 변경을 순수 함수형 틀 안에서 관리합니다.

* [**Functor**](functional/effect-management/functor.md): 컨텍스트 내부의 값을 안전하게 변환
* [**Applicative**](functional/effect-management/applicative.md): 컨텍스트 안의 함수와 값을 결합하여 연산
* [**Monad**](functional/effect-management/monad.md): 컨텍스트를 유지하면서 순차 연산을 체이닝
* [**Reader**](functional/effect-management/reader.md): 환경이나 의존성을 명시적으로 전달
* [**Writer**](functional/effect-management/writer.md): 계산 과정과 함께 로그/부가정보를 누적
* [**State**](functional/effect-management/state.md): 상태 변경을 명시적인 값 반환으로 모델링
* [**IO**](functional/effect-management/io.md): 외부 부수효과를 연산 단위로 분리하여 제어

### 6. 비동기 (Asynchrony)

* [**Future / Promise / Task**](functional/asynchrony/future-promise-task.md): 비동기 연산과 미래의 계산을 값으로 표현

### 7. 불변성 (Immutability)

원본 데이터를 변경하지 않고 안전하게 관리합니다.

* [**Immutable Data**](functional/immutability/immutable-data.md): 데이터를 직접 수정하지 않고 새로운 값 생성
* [**Persistent Data Structure**](functional/immutability/persistent-data-structure.md): 기존 구조를 보존하면서 변경된 새 구조 생성

### 8. 도메인 모델링 (Domain Modeling)

타입 수준에서 불가능한 상태를 방지하여 견고한 모델을 만듭니다.

* [**Algebraic Data Type (ADT)**](functional/domain-modeling/algebraic-data-type.md): 합 타입(Sum)과 곱 타입(Product)으로 상태 모델링
* [**Smart Constructor**](functional/domain-modeling/smart-constructor.md): 잘못된 상태의 객체 생성을 원천 차단
* [**Make Illegal States Unrepresentable**](functional/domain-modeling/make-illegal-states-unrepresentable.md): 잘못된 상태 자체를 타입으로 표현 불가능하게 설계

### 9. 의존성 관리 (Dependency Management)

* [**Dependency Injection via Functions**](functional/dependency-management/dependency-injection-via-functions.md): 클래스 대신 함수 인자로 의존성 직접 전달

### 10. 제어 흐름 (Control Flow)

* [**Continuation / CPS**](functional/control-flow/continuation-cps.md): 이후 실행할 연산 전체를 함수로 넘겨 제어

### 11. 지연 계산 및 최적화 (Lazy Evaluation & Optimization)

* [**Lazy Evaluation**](functional/lazy-evaluation-optimization/lazy-evaluation.md): 실제 필요한 시점까지 연산을 지연
* [**Memoization**](functional/lazy-evaluation-optimization/memoization.md): 동일한 함수 호출 결과를 캐싱하여 재사용

### 12. 이벤트 기반 처리 (Event-Driven)

* [**Event Stream / FRP**](functional/event-driven/event-stream-frp.md): 시간에 따라 변화하는 데이터 흐름을 함수형으로 모델링

### 13. 추상화 및 기타 (Abstraction & DSL)

* [**Lens / Prism / Optics**](functional/abstraction-dsl/lens-prism-optics.md): 불변 중첩 데이터 구조의 안전한 조회 및 수정
* [**Interpreter Pattern / Tagless Final**](functional/abstraction-dsl/interpreter-pattern-tagless-final.md): 프로그램의 정의(Syntax)와 실행 방식(Semantics)을 분리
* [**Combinator**](functional/abstraction-dsl/combinator.md): 작은 구성 요소를 조합하여 도메인 특화 언어(DSL) 구축
