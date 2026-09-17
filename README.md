# 파이썬으로 배우는 디자인 패턴 (Python Design Patterns)

이 저장소는 객체 지향 설계와 함수형 프로그래밍을 함께 다룹니다. 객체 지향 파트는 **Python**으로 GoF 디자인 패턴을 구현하고 설명하며, 함수형 파트는 **Scala**를 기준 언어로 기초부터 심화까지 학습한 뒤 각 개념을 실제 **Python에 어떻게 적용할 수 있는지와 그 표현 한계**를 비교합니다.

앞부분에서는 대표적인 GoF 디자인 패턴 23가지를 생성, 구조, 행위 유형별로 분류하고, 후반부에서는 함수형 사고, 데이터 모델링, 타입 추상화, 효과 제어와 함수형 설계 패턴으로 확장합니다.

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

## 함수형 프로그래밍 (Functional Programming)

후반부에서는 **Scala**를 기준 언어로 함수형 프로그래밍을 기초부터 심화까지 단계적으로 학습합니다. Scala의 강한 타입 시스템과 함수형 기능을 이용해 개념을 정확하게 설명한 뒤, 각 주제를 **Python에서는 어떻게 적용할 수 있는지**, 그리고 **Python의 언어 및 타입 시스템에서 무엇을 표현하기 어려운지** 함께 비교합니다.

> **학습 원칙**: 가상의 Python 문법을 만들지 않습니다. 먼저 Scala로 개념을 온전히 표현하고, 실제 Python으로 옮길 수 있는 부분만 구현합니다. 표현할 수 없는 성질은 `Python에서의 한계`로 명시적으로 다룹니다.

함수형 파트의 기본 학습 흐름은 다음과 같습니다.

`함수형 사고 → 함수와 데이터 변환 → 데이터 모델링 → 오류를 값으로 다루기 → 함수형 추상화 → 효과 제어 → 함수형 설계 패턴 → 고급 함수형 프로그래밍`

각 문서는 가능한 경우 다음 순서를 따릅니다.

1. 개념과 문제 상황
2. Scala에서의 표현
3. 설계 원리와 트레이드오프
4. Python에서 적용하기
5. Python에서의 표현 한계

### 1. 함수형 프로그래밍 기초 (Foundations)

함수형 프로그래밍의 출발점이 되는 사고방식과 언어 요소를 익힙니다.

* [**Expression-Oriented Programming**](functional/01-foundations/expression-oriented-programming.md): 문장보다 표현식을 중심으로 프로그램 구성
* [**Pure Functions**](functional/01-foundations/pure-functions.md): 동일 입력에 동일 출력을 보장하고 외부 상태 변경을 피하는 함수
* [**Immutability**](functional/01-foundations/immutability.md): 값을 변경하기보다 새로운 값을 만들어 상태 변화 표현
* [**Referential Transparency**](functional/01-foundations/referential-transparency.md): 표현식을 그 결과값으로 치환해도 의미가 변하지 않는 성질
* [**First-Class Functions**](functional/01-foundations/first-class-functions.md): 함수를 값처럼 저장하고 전달하고 반환
* [**Higher-Order Functions**](functional/01-foundations/higher-order-functions.md): 함수를 인자로 받거나 결과로 반환하는 함수
* [**Closures**](functional/01-foundations/closures.md): 함수와 함수가 캡처한 환경을 함께 다루기
* [**Recursion**](functional/01-foundations/recursion.md): 반복을 재귀적 정의로 표현하고 재귀의 비용과 한계 이해

### 2. 함수 조합과 데이터 변환 (Composition & Data Transformation)

작은 함수를 조합하여 데이터 변환 파이프라인을 만들고 명령형 반복을 선언적인 변환으로 바꾸는 방법을 배웁니다.

* [**Function Composition**](functional/02-composition/function-composition.md): 작은 함수를 연결해 더 큰 연산 구성
* [**Map**](functional/02-composition/map.md): 컬렉션의 각 값을 함수로 변환
* [**Filter**](functional/02-composition/filter.md): 조건을 만족하는 값만 선택
* [**Fold / Reduce**](functional/02-composition/fold.md): 데이터 구조를 하나의 결과로 축약
* [**FlatMap**](functional/02-composition/flat-map.md): 변환과 평탄화를 결합해 연속된 계산 구성
* [**Pipeline**](functional/02-composition/pipeline.md): 데이터를 연속된 함수 단계로 전달
* [**Partial Application**](functional/02-composition/partial-application.md): 일부 인자를 미리 적용해 새로운 함수 생성
* [**Currying**](functional/02-composition/currying.md): 다중 인자 함수를 단일 인자 함수의 연쇄로 변환

### 3. 함수형 데이터 모델링 (Functional Data Modeling)

행동보다 가능한 상태와 데이터의 형태를 먼저 정의하고, 잘못된 상태를 구조적으로 줄이는 방법을 배웁니다.

* [**Product Types**](functional/03-data-modeling/product-types.md): 여러 값을 동시에 가지는 데이터 구조 모델링
* [**Sum Types**](functional/03-data-modeling/sum-types.md): 여러 가능한 상태 중 하나를 타입으로 표현
* [**Algebraic Data Types (ADT)**](functional/03-data-modeling/algebraic-data-types.md): 합 타입과 곱 타입을 조합해 도메인 모델링
* [**Pattern Matching**](functional/03-data-modeling/pattern-matching.md): 데이터의 형태에 따라 계산을 분해하고 처리
* [**Smart Constructors**](functional/03-data-modeling/smart-constructors.md): 유효한 값만 생성되도록 생성 경로 제한
* [**Make Illegal States Unrepresentable**](functional/03-data-modeling/make-illegal-states-unrepresentable.md): 잘못된 상태 자체를 타입으로 표현하기 어렵게 설계

### 4. 오류를 값으로 다루기 (Error Handling as Values)

예외나 `null`에 의존하는 제어 흐름을 명시적인 데이터와 타입으로 바꾸는 방법을 배웁니다.

* [**Exceptions vs Values**](functional/04-error-handling/exceptions-vs-values.md): 예외 기반 오류 처리와 값 기반 오류 처리 비교
* [**Option**](functional/04-error-handling/option.md): 값의 부재를 명시적인 타입으로 표현
* [**Either**](functional/04-error-handling/either.md): 성공과 실패를 하나의 합 타입으로 모델링
* [**Try**](functional/04-error-handling/try.md): 예외가 발생할 수 있는 계산을 값으로 표현
* [**Validation**](functional/04-error-handling/validation.md): 독립적인 검증 결과를 조합
* [**Error Accumulation**](functional/04-error-handling/error-accumulation.md): fail-fast와 여러 오류 누적 방식의 차이 이해

### 5. 함수형 추상화 (Functional Abstractions)

여러 데이터 타입과 계산에 반복되는 구조를 타입 수준에서 일반화하는 방법을 배웁니다.

* [**Generic Functions**](functional/05-functional-abstractions/generic-functions.md): 구체 타입에 종속되지 않는 함수 작성
* [**Type Classes**](functional/05-functional-abstractions/type-classes.md): 타입과 동작의 구현을 분리하여 다형성 구성
* [**Higher-Kinded Types**](functional/05-functional-abstractions/higher-kinded-types.md): 타입 생성자를 추상화하는 고차 타입 이해
* [**Functor**](functional/05-functional-abstractions/functor.md): 컨텍스트의 구조를 유지하면서 내부 값 변환
* [**Applicative**](functional/05-functional-abstractions/applicative.md): 독립적인 컨텍스트 계산들을 결합
* [**Monad**](functional/05-functional-abstractions/monad.md): 앞선 계산 결과에 의존하는 연속 계산 구성
* [**Semigroup**](functional/05-functional-abstractions/semigroup.md): 결합 가능한 연산을 추상화
* [**Monoid**](functional/05-functional-abstractions/monoid.md): 항등원을 가진 결합 연산을 추상화

### 6. 효과와 의존성 (Effects & Dependencies)

상태 변경, 외부 입출력, 환경 의존성, 비동기 실행 같은 효과를 계산의 핵심 로직과 분리하여 다루는 방법을 배웁니다.

* [**Side Effects**](functional/06-effects/side-effects.md): 순수 계산과 외부 세계의 변화를 구분
* [**Reader**](functional/06-effects/reader.md): 환경과 의존성을 명시적인 계산 컨텍스트로 전달
* [**State**](functional/06-effects/state.md): 상태 변화를 입력과 출력 값으로 모델링
* [**Writer**](functional/06-effects/writer.md): 계산 결과와 로그나 부가 정보를 함께 누적
* [**IO**](functional/06-effects/io.md): 외부 부수효과를 지연된 계산으로 표현하고 조합
* [**Dependency Injection with Functions**](functional/06-effects/dependency-injection.md): 객체 컨테이너 대신 함수와 값으로 의존성 전달
* [**Asynchronous Effects**](functional/06-effects/asynchronous-effects.md): Future, Task, IO와 비동기 계산의 차이 이해

### 7. 함수형 설계 패턴 (Functional Design Patterns)

기초 개념과 추상화를 실제 소프트웨어 설계에 적용하고, 기존 객체지향 패턴이 함수형 스타일에서 어떻게 달라지는지 살펴봅니다.

* [**Functional Core, Imperative Shell**](functional/07-patterns/functional-core-imperative-shell.md): 순수한 핵심 로직과 효과적인 외곽 계층 분리
* [**Parse, Don't Validate**](functional/07-patterns/parse-dont-validate.md): 검증 후 원래 타입을 유지하기보다 유효한 타입으로 변환
* [**Railway-Oriented Programming**](functional/07-patterns/railway-oriented-programming.md): 성공과 실패 경로를 조합 가능한 계산 흐름으로 구성
* [**Dependency Rejection**](functional/07-patterns/dependency-rejection.md): 가능한 범위에서 의존성을 데이터와 순수 함수로 제거
* [**Functions as Strategies**](functional/07-patterns/functions-as-strategies.md): 전략 객체 대신 일급 함수로 행위를 주입
* [**Algebra & Interpreter**](functional/07-patterns/algebra-and-interpreter.md): 프로그램의 명세와 실행 방식을 분리
* [**Tagless Final**](functional/07-patterns/tagless-final.md): 고차 추상화를 이용해 프로그램 표현과 해석을 분리
* [**Free Monad**](functional/07-patterns/free-monad.md): 프로그램 구조를 데이터로 표현하고 실행을 나중에 해석
* [**Combinators & DSL**](functional/07-patterns/combinators-and-dsl.md): 작은 구성 요소를 조합해 도메인 특화 언어 구축

### 8. 고급 함수형 프로그래밍 (Advanced Functional Programming)

앞에서 배운 개념을 바탕으로 평가 전략, 재귀 추상화, 영속 자료구조, optics, 효과 시스템과 반응형 모델까지 확장합니다.

* [**Lazy Evaluation**](functional/08-advanced/lazy-evaluation.md): 값이 실제로 필요해질 때까지 계산 지연
* [**Memoization**](functional/08-advanced/memoization.md): 순수 함수의 결과를 캐시하여 반복 계산 제거
* [**Persistent Data Structures**](functional/08-advanced/persistent-data-structures.md): 이전 버전을 보존하면서 효율적으로 새로운 데이터 구조 생성
* [**Recursion Schemes**](functional/08-advanced/recursion-schemes.md): 재귀 구조와 실제 처리 로직을 분리
* [**Continuation-Passing Style (CPS)**](functional/08-advanced/continuation-passing-style.md): 이후 계산을 명시적인 함수로 전달하여 제어 흐름 표현
* [**Lens**](functional/08-advanced/lens.md): 불변 중첩 데이터의 특정 부분을 합성 가능한 방식으로 접근하고 갱신
* [**Prism**](functional/08-advanced/prism.md): 합 타입의 특정 경우에 초점을 맞춘 안전한 접근과 변환
* [**Optics**](functional/08-advanced/optics.md): Lens, Prism 등의 접근 추상화를 조합
* [**Effect Systems**](functional/08-advanced/effect-systems.md): 프로그램이 수행할 수 있는 효과를 타입과 추상화로 표현
* [**Functional Reactive Programming (FRP)**](functional/08-advanced/functional-reactive-programming.md): 시간에 따라 변화하는 값을 함수형 모델로 표현
