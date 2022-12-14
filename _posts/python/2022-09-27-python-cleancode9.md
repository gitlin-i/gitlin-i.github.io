---
title: "python clean code chapter9"
categories:
  - python
tags:
  - clean-code
toc: true
toc_label: "Index"
toc_icon: ""
toc_sticky: true
---
# Python Clean Code

---

## 9. 일반적인 디자인 패턴

여기서는 디자인 패턴이 어떻게 클린 코드에 기여할 수 있는지 살펴보자.  
파이썬의 동적인 특성으로 인해 많은 디자인 패턴이 염두에 둔 정적인 언어들과는 구현상의 차이가 있다.  
파이썬에서 디자인패턴을 적용할 때는 주의해야 할 점이 있고 어떤 경우에는 파이썬스럽지 않은 코드를 만들게 될 수 있다.

---

### 9-1. 파이썬에 디자인 패턴 적용 시 고려사항

객체 지향 디자인 패턴은 다양한 시나리오에서 해결책의 하나로 제시되는 모델 중 하나이다.  
이것은 고차원적인 개념이기 때문에 특정 프로그래밍 언어에만 종속된 이야기는 아니다.

파이썬의 특성을 생각해보면 고전적인 디자인 패턴 중 일부는 실제로 필요하지 않다.  
즉, 이미 그러한 디자인 패턴을 내부적으로 구현하고 있기 때문이다.  
이터레이션 패턴을 직접 구현하려고 시도하는 것은 완전히 파이썬스럽지 않다.


---

### 9-2. 실전 속의 디자인 패턴

GoF의 23개 디자인 패턴 기준 참고.  
생성, 구조, 행동 3개의 대분류.
모든 것을 살펴보진 않고, 2가지만 염두에 두자.

1. 일부 패턴은 파이썬 내부에서 자체적으로 구현되어 있으므로 보이지 않은 채로 적절히 적용가능.

2. 모든 패턴이 똑같이 일반적인 것은 아님.

가장 발생빈도가 높은 일반적인 디자인 패턴을 보자.  
이러한 패턴은 보통 디자인 도중에 **출현**하게 된다.

애플리케이션의 솔루션에 **강제로 디자인 패턴을 적용해서는 안 되며**, 패턴이 출현할 때까지는 솔루션을 진화시키고 리팩토링하고 개선해야만 한다.

따라서 디자인 패턴은 발명 되는 것이 아니라 발견되는 것이다.

---
#### 생성(creational)패턴

객체를 인스턴스화 할 때의 복잡성을 최대한 추상화하기 위한 것.  

+ 팩토리
파이썬에서는 팩토리 패턴이 별로 필요하지 않다.

+ 싱글턴
싱글턴 패턴은 파이썬에 의해 완전히 추상화되지 않은 패턴이다.  
사실 대부분의 경우 이 패턴은 필요하지 않거나 **나쁜 선택**.  
객체 지향 소프트웨어를 위한 전역 변수의 한 형태이며 결국은 나쁜 습관이다. 또한 단위테스트가 어렵다. 어떤 객체에 의해서 언제든지 수정될 수 있다는 사실은 예측하기 어렵다는 뜻이고, 실제로 부작용이 큰 문제를 일으킬 수도 있다.
꼭 사용해야 한다면, 모듈을 사용하여 해결하는 것이 좋다.
모듈은 이미 싱글턴을 의미한다. 여러번 임포트하더라도 모듈은 항상 한 개만 로딩된다.

+ 공유 상태
싱글턴인지 아닌지에 상관없이 일반 객체처럼 많은 인스턴스를 만들 수 있어야 한다는 것.  
완전히 투명한 방법으로 정보를 동기화.

+ borg
공유상태(mono state)패턴으로 파이썬에서는 borg패턴이라고 부른다.  
같은 클래스의 모든 인스턴스가 모든 속성을 복제하는 객체를 만드는 것.

+ 빌더
객체의 복잡한 초기화를 추상화하는 흥미로운 패턴. 언어의 특수성에 의존하지 않음.  
필요로 하는 모든 객체를 직접 생성해주는 하나의 복잡한 객체를 만들어야 한다는 것.  
사용자가 필요로 하는 모든 보조 객체를 직접 생성하여 메인 객체에 전달하는 것이 아니라, 한 번에 모든 것을 처리해주는 추상화를 해야한다.

---
#### 구조(structural)패턴

인터페이스를 복잡하게 하지 않으면서도 기능을 확장하여 더 강력한 인터페이스 또는 객체를 만들어야 하는 상황에 유용.

+ 어댑터
래퍼(wrapper)라고도 함. 호환되지 않는 두 개 이상의 객체에 대한 인터페이스를 동시에 사용할 수 있게 한다.  
해당 객체를 수용할 수 있는 새로운 인터페이스를 개발.

구현시 2가지 방법
1. 클래스를 상속시킴.
상속은 **is a** 관계에 한정해서 적용하는 것이 바람직.

2. 컴포지션을 사용하는 것
불필요하게 솔루션을 복잡하게 만드는 것은 아닌지 주의.

+ 컴포지트
객체는 구조화된 트리 형태로 생각해보자. 기본 객체는 리프,컨테이너 객체는 중간 노드라 볼 수 있다. 클라이언트는 이 중에 아무거나 호출하여 결과를 얻고자 할 것이다.  
컴포지트 객체도 클라이언트처럼 동작한다.
아무 노드로 해당요청을 처리할 수 있을 때까지 계속 전달한다.

+ 데코레이터 패턴 (!= 파이썬의 데코레이터)
동일한 인터페이스를 가지고 여러 단계를 거쳐 결과를 향상(장식) 할 수도 있고 결합도 할 수 있는 또 다른 객체를 만드는 것. 이 객체들은 연결되어 있으며 각각의 객체는 본래 의도에 더해 새로운 기능이 추가 될 수 있다.  
이렇게 새로운 기능을 추가하는 단계가 데코레이션 단계.

+ 파사드(Facade)
객체 간 상호 작용을 단순화하려는 많은 상황에서 유용.  
파사드는 허브 또는 단일 참조점의 역할.  
각각의 객체에 대한 모든 연결을 만드는 대신에 파사드 역할을 하는 중간 객체를 만드는 것이다.  
클래스나 객체에 한정된 것이 아니라 패키지에도 적용됨.  
즉, 사용자에게 노출해야 하는 임포트 가능한 외부용 레이아웃과 직접 임포트해서는 안 되는 내부용 레이아웃을 구분.  
파이썬에서 디렉토리의 패키지를 빌드할 때는 __init__py를 나머지 파일들과 함께 둔다. 이것이 모듈의 루트로서 파사드와 같은 역할을 한다.

---
#### 행동(behavioral) 패턴

객체가 어떻게 협력해야하는지, 어떻게 통신해야하는지, 런타임 중에 인터페이스는 어떤 형태여야 하는지에 대한 문제를 해결하는 것을 목표로 함.

+ 책임 연쇄 
클라이언트로부터 받은 이벤트(혹은 어떠한 요청)를 해당 객체가 당장 처리할 수 있으면 처리하되, 처리가 불가능할 경우 이를 처리할 다른 객체에 연쇄적으로 전달한다.
처리순서에 유연함.

+ 템플릿 메서드
어떤 행위를 정의할 때 특정한 형태의 클래스 계층구조를 만드는 것.  
사용자는 하위 클래스를 만들고, 특정 private메서드를 오버라이드하기만 하면 하위 호환성이 유지되는 새로운 행동을 정의할 수 있다.

+ 커맨드
수행해야 할 작업을 요청한 순간부터 실제 실행 시까지 분리할 수 있는 기능. 또한 클라이언트가 발행한 원래 요청을 수신자와 분리할 수도 있음. 수신자는 다른 객체일 수 있다.  
특히 첫번째 기능의 흥미로운 점은 명령의 실제 실행과 실행 순서의 조절을 분리할 수 있다는 점이다.  

+ 상태 패턴
도메인 문제의 개념을 부수적인 가치에서 명시적인 객체로 전환시킬 수 있다.  
상태별로 작은 객체를 만들어 각각의 객체가 적은 책임을 갖게 하는 것이 좋다. 먼저 표현하고자 하는 각 종류의 상태를 객체로 만들고, 각 객체의 메서드에 전이로직을 작성한다.

---
### 9-3. NULL 객체 패턴

함수나 메서드는 일관된 타입을 반환해야 한다는 것.  
이것이 보장되면, 클라이언트는 다형성을 가진 메서드에서 반환되는 객체에 대해 null체크를 하지 않고도 바로 사용할 수 있다.  
예를 들어 사전타입의 반환을 기대할 때, 아무런 처리를 하지 않을 경우 None을 반환하기보다는 빈 사전 { }을 반환하자.  
이렇게 일관성이 있는 타입의 객체를 반환하도록 하자.  

이후 이 반환을 사용할 때 2가지 처리 방법.
1. 예외 발생
2. Unknown타입을 반환

어떠한 경우에도 None을 반환하면 안된다.

---

### 9-4. 디자인패턴 정리

디자인 패턴은 그 자체로 좋다거나 나쁜 것은 아니다.  
그 보다는 어떻게 구현하느냐의 문제이다.  
어떤 경우에는 실제로 디자인 패턴이 필요하지 않으며, 더 간단한 솔루션이 있을 수 있다.  
패턴이 맞이 않는 곳에 패턴을 강요하는 것은 오버엔지니어링으로 분명히 나쁜 것이지만 디자인 패턴 자체에 문제가 있다는 뜻은 아니다.

일반적으로 솔루션을 만들거나 추상화를 하기 전에 **3회 반복의 법칙**을 기억할 필요가 있다.

이미 해결책이 있는 문제에 대해 해결책을 찾느라 시간을 낭비하는 것은 나쁜 일이다. 하지만 해결하려고 하는 문제에 대한 해결책이 이미 있었다는 사실 자체가 나쁜 것이 될 수는 없다.

디자인 패턴을 적용한 경우 코드에서 이름을 표기해야 할까?

1. 코드가 의도한 대로 잘 동작하는 한 사용자는 해당 코드의 내부에 어떤 디자인 패턴이 적용되었는지 알 필요가 없다.

2. 디자인 패턴을 언급하면 명확한 의도를 드러내지 못하게 된다.

위의 2가지 이유로 권장되지 않는다.

다만 docstring에서 디자인 패턴을 언급하는 것은 문서화 작업의 일부로서 디자인 철학을 공유하는데 도움이 된다.
그러나 이것도 꼭 필요한 것은 아니다. 위의 1번의 이유.
