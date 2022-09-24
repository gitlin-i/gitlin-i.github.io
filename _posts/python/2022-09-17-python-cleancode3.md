---
title: "python clean code chapter3"
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
## 3. 좋은 코드의 일반적인 특징

좋은 소프트웨어는 좋은 디자인으로부터 나온다.  
코드가 디자인이고, 디자인이 코드다.

---
### 3-1. 계약에 의한 디자인

**계약에 의한 디자인(Design by Contract,DbC)** 는 관계자가 기대하는 바를 암묵적으로 코드에 삽입하는 대신 양측이 동의하는 계약을 먼저 한 다음, 계약을 어겼을 경우는 명시적으로 왜 계속할 수 없는지 예외를 발생시키는 것이다.

---
#### 계약

여기서 계약은 소프트웨어 컴포넘트 간의 통신 중에 반드시 지켜야 할 몇 가지 규칙을 강제하는 것이다.  
계약은 주로 사전조건, 사후조건을 명시하며 때로는 불변식과 부작용을 기술한다.

+ 사전조건
코드가 실행되기전에 체크해야 하는 것들. 유효성 체크를 통해 부작용을 최소화된다는 점을 고려할 때 유효성 검사를 많이 하는 것이 좋다. 이러한 작업은 **호출자**에게 부과된 임무이다.

+ 사후조건
사전 조건과 반대로 함수 반환 값의 유효성 검사가 수행된다.
사후조건 **검증**은 **호출자**가 이 컴포넌트에서 기대한 것을 제대로 받았는지 확인하기 위해 수행한다.

+ 불변식
힘수가 실행되는 동안 일정하게 유지되는 것. 로직에 문제가 없는지 확인하기 위한 것.

+ 부작용
선택적으로 코드의 부작용을 docstring에 언급하기도 한다.

이상적으로 이 모든 것들을 계약서의 일부로 문서화해야 하지만, 처음 2개인 사전조건과 사후조건만 저수준(코드)레벨에서 강제한다.

---

#### 사전조건(precondition)

사전조건은 클라이언트와 연관되어 있다.   
클라이언트는 코드를 실행하기 위해 사전에 약속한 조건을 준수해야 한다.

문제는 이 유효성 검사를 어디서 할지이다. 클라이언트가 함수를 호출하기 전에 모든 유효성 검사를 하도록 할 것인지, 함수가 자체적으로 로직을 실행하기 전에 검사하도록 할 것인지에 대한 문제다.

전자를 관용적인(tolerant) 접근방법.  
왜냐하면 함수가 어떤 값이라도 수용하기 때문이다.  

후자를 까다로운(demanding) 접근방법.

일반적으로 가장 안전하고 견고하며 널리 쓰이는 방법이 **까다로운 접근법**이다.  

어떤 방식을 택하든 **중복 제거 원칙**을 항상 마음속에 간직해야 한다. 사전조건 검증을 양쪽에서 하지 말고 오직 어느 한쪽에서만 해야 한다는 것이다.  
이는 DRY원칙과 관련있다.

---

#### 사후조건(postcondition)

사후조건은 컴포넌트와 연관되어 있다.   
컴포넌트는 클라이언트가 확인하고 강제할 수 있는 값을 보장해야 한다.

함수 또는 메서드가 적절한 속성으로 호출되었다면 (즉, 사전조건에 맞는다면) 사후조건은 특정속성이 보존되도록 보장해야한다.  
사전 조건이 맞고, 계약이 이루어졌으면 **사후조건 검증**에 통과하고 클라이언트는 반환객체를 아무 문제없이 사용할 수 있어야 한다.

---

#### 결론

디자인 원칙의 주된 가치는 문제가 있는 부분을 효과적으로 식별하는데 있다.  
계약을 정의함으로써 런타임시 오류가 발생했을 때 코드의 어떤 부분이 손상되었는지 계약이 파손되었는지 명확해진다.

물론 이러한 원칙을 따르면 추가 작업이 발생한다. 왜냐하면 핵심 로직뿐만이 아니라 계약을 작성해야하기 때문이다.  
그러나 이 방법을 통해 얻은 품질은 장기적으로 보상된다.  

함수에 제공된 파라미터의 올바른 데이터 타입만 검사하는 계약을 정의하는 것은 별로 의미가 없다.  
그렇게 하는 것은 파이썬을 정적 타입을 가진 언어로 만드는 것과 비슷하다.

**무엇을 기꺼이 검증할 것인지** 신중히 검토해봐야한다.
함수에 전달되는 객체의 속성과 반환 값을 검사하고 이들이 유지해야 하는 조건을 확인하는 등의 작업이 실질적인 가치가 있다.

----

### 3-2. 방어적 프로그래밍

방어적 프로그래밍은 **DbC**와는 다소 다른 접근 방식을 따른다.
방어적 프로그래밍의 주요 주제는 **예상할 수 있는 시나리오의 오류를 처리하는 방법**과 **발생하지 않아야 하는 오류를 처리하는 방법**에 대한 것이다.

전자는 에러 핸들링 프로시저.
후자는 어썰션.


이것은 DbC와 다른 철학을 가졌다는 의미는 아니고, 다른 디자인 원칙과 서로 보완 관계에 있을 수 있다는 것을 의미한다.

---
#### 에러 핸들링

에러 핸들링의 주요 목적은 예상되는 에러에 대해서 실행을 계속할 수 있을지 아니면 극복할 수 없는 오류여서 프로그램을 중단할지를 결정하는 것이다.

---

##### 값대체 

일부 시나리오에서 오류가 있어 소프트웨어가 잘못된 값을 생성하거나 전체가 종료될 위험이 있을 경우 결과값을 안전한 다른 값으로 대체할 수 있다.

그러나 값 대체가 항상 가능하지는 않다.  
견고성과 정확성간의 트레이드 오프이기 때문이다.  

일반적으로 누락된 파라미터를 기본값로 바꾸어도 큰 문제가 없다.
오류가 있는 데이터를 유사한 값으로 대체하는 것은 더 위험하며,일부 오류를 숨겨버릴 수 있다. 

---

##### 예외 처리

함수는 심각한 오류에 대해 명확하고 분명하게 알려줘서 적절하게 해결할 수 있도록 해야한다.  
예외적인 상황을 명확하게 알려주고 원래의 비즈니스 로직에 따라 흐름을 유지하는 것이 중요하다.


예외를 사용시

1. 프로그램의 흐름을 읽기 어려워진다.
2. 예외는 캡슐화를 약화시킨다.
3. 예외가 많을 수록 호출자가 호출하는 함수에 대해 많은것을 알아야만 한다.
4. 너무 많은 예외를 발생 시키면, 문맥에서 자유롭지 않다.

파이썬에서 예외와 관련된 몇 가지 권장사항

1. 올바른 수준의 추상화 단계에서 예외 처리
예외는 오직 한 가지 일을 하는 함수의 한 부분이어야한다.
캡슐화된 로직과 일치해야 한다.
2. Traceback 노출 금지
tracebakc은 매우 유용하고 많은 디버깅 정보
악의적인 사용자에게도 매우 유용한 정보.
3. 비어 있는 except 블록 지양
```
try:
  process()
except:
  pass
```
가장 악마(?)같은 패턴
절대 실패하지않는다. 에러조차 모르게 한다.
이 코드는 문제를 숨기고 유지보수를 더 어렵게 만든다.

대안
> 보다 구체적인 예외를 사용한다.
>(Exception 같이 광범위한 예외를 사용하면 안된다.)
>
>except 블록에서 실제 오류를 처리한다.

4. 원본 예외 포함
```
class InternalDataError(Exception):

  def process(data_dictionary,record_id):
    try:
      return data_dictionary[record_id]
    except KeyError as e:
      rasie InternalDataError("record not present") from e
```

raise e from o 구문

기본 예외를 사용자 정의 예외로 래핑

---

#### 어설션

절대로 일어나지 않아야 하는 상황. 이 상태가 된다는 건 소프트웨어에 결함이 있음을 의미.

어설션은 잘못된 시나리오에 도달할 경우 프로그램이 더 큰 피해를 입지 않도록 하는 것이다.
이러한 이유로 어설션을 비즈니스 로직과 섞거나 소프트웨어의 제어 흐름 메커니즘으로 사용해서는 안된다.

---

### 3-3. 관심사의 분리 

여러 수준에 적용되는 설계 원칙.  

책임이 다르면 컴포넌트, 계층 또는 모듈로 분리되어야 한다.  
프로그램의 각 부분은 기능의 일부분(관심사)에 대해서만 책임을 지며 나머지 부분에 대해서는 알 필요가 없다.

관심사를 분리하는 **목표**는 파급 효과를 최소화하여 유지보수성을 향상시키는 것.

---
#### 응집력(Cohesion)과 결합력(Coupling)

**응집력**: 객체가 작고 잘 정의된 목적을 가져야하며 가능한 작아야한다.

**결합력**: 두 개 이상의 객체가 서로 어떻게 의존하는지를 나타낸다. 너무 의존적이라면 다음과 같은 바람직하지 않은 결과를 가져온다.

+ 낮은 재사용성
+ 파급효과
+ 낮은 수준의 추상화

응집력은 높이고 결합력는 낮추어야 한다.

---
### 3-4. 개발 지침 약어

정식 학술 용어는 아님.

---
#### DRY/OAOO

+ DRY(Don't Repeat Yourself)

+ OAOO(Once and Only Once)

중복을 반드시 피하라.

코드 중복으로 발생하는 문제
1. 오류가 발생하기 쉽다.
2. 비용이 비싸다. (코드 수정시 드는 시간이 많이 듬.)
3. 신뢰성이 떨어진다.

---

#### YAGNI

+ YAGNI(You Ain't Gonna Need it)

과잉 엔지니어링을 하지 않기위해 솔루션 작성 시 계속 염두에 두어야하는 원칙

유지보수가 가능한 소프트웨어를 만드는 것은 미래의 요구 사항을 예측하는 것이 아니다.  
오직 현재의 요구 사항을 잘 해결하기 위한 소프트웨어를 작성하고 가능한 나중에 수정하기 쉽도록 작성하는 것이다.

굳이 필요 없는 추가 개발을 하지 말라.

---

#### KIS

+ KIS(Keep It Simple)

YAGNI와 유사. 선택한 솔루션이 문제에 적합한 최소의 솔루션인가를 묻는다.

디자인이 단순할수록 유지 관리가 쉽다.

---

#### EAFP/LBYL

+ EAFP(Easier to Ask Forgiveness than Permission)
+ LBYL(Look Before You Leap)

허락보다는 용서를 구하는 것이 쉽다.
도약하기 전에 살피라.

EAFP는 일단 코드를 실행하고 실제 동작하지 않을 경우에 대응한다는 뜻이다.

LBYL는 그 반대이다. 이름에서 알 수 있듯이 도약하기 전에 무엇을 사용하려고 하는지 확인하라는 뜻이다.

파이썬은 EAFP 방식으로 만들어졌으며, LBYL방식은 파이썬스러운 방식이 아니다.

**LBYL 원칙 보다 EAFP원칙이 더 바람직하다.**

LBYL
```
if os.path.exists(filename):
  with open(filename) as f:
    ...
```
EAFP
```
try:
  with open(filename) as f:
    ...
except FileNotFoundError as e:
  logger.error(e)
```
---

### 3-5. 컴포지션과 상속

객체 지향 소프트웨어를 디자인할 때 다형성, 상속, 캡슐화 같은 주요 개념을 어떻게 사용하여 문제를 해결할 것인지에 대한 논쟁이 있어왔다.

일반적으로 사용되는 개념은 상속일 것이다.  
상속은 강력한 개념이지만, 위험도 있다. 가장 주된 위험은 부모 클래스를 확장하여 새로운 클래스를 만들 때마다 부모와 **강력하게 결합된** 새로운 클래스가 생긴다는 점이다.

단순히 코드를 재사용하기 위해 상속을 하는 것은 좋지 않은 생각이다.

--- 
#### 상속이 좋은 경우

파생 클래스를 만드는것은 양날의 검이 될 수 있으므로 주의해야 한다.

만약 대부분의 메서드를 필요로 하지 않고 재정의하거나 대체해야만 한다면 다음과 같은 이유로 설계상의 실수라고 할 수 있다.

>상위 클래스는 잘 정의된 인터페이스 대신 막연한 정의와 너무 많은 책임을 가졌다.
>
>하위 클래스는 확정하려고 하는 상위 클래스의 적절한 세분화가 아니다.

상위 클래스의 기능을 그대로 물려받으면서 추가 기능을 더하려는 경우 또는 특정기능을 수정하려는 경우 상속하는 것이 좋다.

---
#### 파이썬의 다중상속

파이썬은 다중상속을 지원한다.

다중상속은 올바르게 사용될 때만 온전히 유효한 해결책이 될 수 있으므로 파이썬은 어댑터 패턴과 믹스인을 사용한다.

---
##### 메서드 결정순서

다중상속은 **다이아몬드 문제**. 손자 클래스의 부모로 부터 상속받은 공통 메서드를 사용할 때, 어느 메서드를 사용해야 할지 모호해지는 문제가 있다.

파이썬은 **C3 linearization** 또는 **MRO**라는 알고리즘을 사용하여 문제를 해결한다. 이 알고리즘은 메서드가 호출되는 방식을 정의한다.

구체적으로 클래스에게 결정순서를 물어볼 수도 있다.

---

#### 믹스인

믹스인은 코드를 재사용하기 위해 일반적인 행동을 캡슐화해놓은 기본 클래스이다.

```
calss BaseTokenizer:
  def __init__(self,str_token):
    self.str_token = str_token

  def __iter__(self):
    yield from self.str_token.split("-")

#믹스인 클래스
class UpperIterationMixin:
  def __iter__(self):
    return map(str.upper, super().__iter__())

class Tokenizer(UpperIterationMixin, BaseTokenizer):
  pass
```
믹스인 클래스는 그자체로는 유용하지 않다.  
보통은 다른 클래스와 함께 믹스인 클래스를 **다중 상속**하여 믹스인에 있는 메서드나 속성을 사용한다.

위의 새로운 Tokenizer클래스는 믹스인을 사용하기때문에 새로운 코드가 필요없다.
super() 호출을 통해 다음 클래스에 위임한다.

---

### 3-6. 함수와 메서드의 인자

파이썬의 첫 번째 규칙은 모든 인자가 값에 의한 전달(passed by value)이다.

**immutable** 타입 변수를 함수에 전달하면, 함수안에서 그 값을 활용하여 새로운 객체를 만들어 할당한다. 그러므로 원래 변수의 값을 수정하지 않는다.

**mutable** 타입 변수를 함수에 전달하면, 함수는 해당 변수의 참조를 보유하고 있는 변수를 통해 값을 수정하므로 함수외부에서도 값을 수정할 수 있다.

---
#### 가변인자

가변 인자를 사용하려면 해당 인자를 패킹할 변수의 이름 앞에 별표(*)를 사용한다.
```
>>> first, *middle, last = range(6)
>>> first
0
>>> middle
[1,2,3,4]
>>> last
5
```

비슷한 표기법으로 이중별표(**)를 키워드 인자에 사용할 수 있다.
```
def func_1(**kwargs):
  pass
```

사전에 이중별표를 사용하여 함수에 전달하면 파라미터 이름으로 키를 사용하고, 파라미터의 값으로 사전의 값을 사용한다.

반대로 이중별표로 시작하는 파라미터를 함수에 사용하면 반대현상이 벌어진다.

함수의 서명으로 *args와 **kwarges로 정의된 함수가 실제로 융통성 있고 적응력이 좋은 것은 사실이지만 단점은 서명을 잃어버린다는 것과 가독성을 거의 상실한다는 것이다.

위치인자든 키워드인자든 가변인자를 사용하면, 매우 좋은 docstring을 만들어 놓지 않는 한 나중에 해당 함수에 사용된 파라미터를 보고도 정확한 동작을 알 수가 없다.


---

### 3-7. 소프트웨어의 독립성

독립성과 관련된 흥미로운 품질 특성이 있다. 코드의 두 부분이 독립적이라는 것은 다른 부분에 영향을 주지 않고 변경할 수 있다는 것을 뜻한다.

이는 변경된 부분의 단위 테스트가 나머지 단위 테스트와도 독립적이라는 것을 뜻한다.  
이러한 가정하에 두 개의 테스트가 통과하면 전체 회귀 테스트를 하지 않고도 애플리케이션에 문제가 없다고 어느 정도 확신할 수 있다.

---

### 3-8.코드 구조

코드를 구조화하는 방법은 팀의 작업 효율성과 유지보수성에 영향을 미친다.

특히 여러 정의(클래스,함수,상수 등)가 들어있는 큰 파일을 만드는 것은 좋지 않다.

극단적으로 하나의 파일에 하나의 정의만 유지하하는 것은 아니지만, 좋은 코드라면 유사한 컴포넌트끼리 정리하여 구조화해야 한다.
