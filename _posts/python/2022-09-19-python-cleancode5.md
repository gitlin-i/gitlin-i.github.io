---
title: "python clean code chapter5"
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

## 5. 데코레이터를 사용한 코드 개선

---

### 5-1. 파이썬의 데코레이터

함수와 메서드를 쉽게 수정하기 위해서,
예를 들어, original이라는 함수가 있고 그 기능을 약간 수정한 modifier라고 하는 함수가 있는 경우 다음과 같이 작성한다.
```
def original(...):
...

original = modifier(original)
```
함수에 변형을 할 때마다 modifier함수를 사용하여 함수를 호출한 다음 함수를 처음 정의한 것과 같은 이름으로 재할당해야 했다.

이는 혼란스럽고 오류가 발생하기 쉽고 번거롭다. 그래서 새로운 구문이 추가 되었다.

```
@modifier
def original(...):
...
```
데코레이터 이후에 나오는 것을 데코레이터의 첫 번째 파라미터로 하고 데코레이터의 결과 값을 반환하게 하는 **문법적 설탕**일 뿐이다.

>문법적 설탕(syntax sugar): 어떤 언어에서 동일한 기능이지만, 타이핑의 수고를 덜어주기 위해 또는 읽기 쉽게 하기 위해 다른 표현으로 코딩할 수 있게 해주는 기능

파이썬에서는 modifier를 데코레이터, original을 데코레이팅된 함수 또는 래핑된 객체라 한다.


* 데코레이터 디자인 패턴과 혼동하면 안된다.

---

#### 함수 데코레이터


```
def retry(operation):
    @wraps(operation)
    def wrapped(*arg,**kwargs):
        last_rasied = None
        RETRIES_LIMIT = 3
        for _ in range(RETRIES_LIMIT):
            try:
                return operation(*args,**kwargs)
            except:
                ...
        raise last_raised

    return wrapped

@retry
def run_operation(task):
    return task.run()
```

@retry는 실제로 파이썬에서 run_operation = retry(run_operation)을 실행하게 해주는 문법적 설탕일 뿐이다.

---

#### 클래스 데코레이터

함수에 적용한 것처럼 클래스에도 데코레이터를 사용헐 수 있다.  
유일할 차이점은 데코레이터 함수의 파라미터롤 함수가 아닌 클래스를 받는다는 점이다.

어떤 개발자들은 클래스 데코레이터가 복잡하고 가독성을 떨어뜨릴 수 있다고 말할 수 있다.  
클래스에서 정의한 속성과 메서드를 데코레이터 안에서 완전히 다른 용도로 변경할 수 있기 때문이다.

---

#### 중첩 함수의 데코레이터

```
RETRIES_LIMIT = 3
def with_retry(retries_limit=RETRIES_LIMIT, allowed_exceptions=None):
    allowed_exceptions or (ControlledException,)
    
    def retry(operation):
        @wraps(operation)
        def wrapped(*args, **kwargs):
            last_raised = None
            for _ in range(retries_limit):
                try:
                    return operation(*args,**kwargs)
                except:
                ...


            rasie lats_raised
        
    return wrapped
    
return retry


@with_retry(retries_limit =5)
def run_with_custom_retries_limit(task):
    return task.run()

```
데코레이터에 인자를 전달하기 위해 세 단계의 중첩된 함수가 필요.

---

#### 데코레이터 객체

```
class WithRetry:

    def __init__(self,retries_limit=RETRIES_LIMIT, allowed_exceptions=None):
        self.retries_limit = retries_limit
        self.allowed_exceptions = allowed_exceptions or (...)

    def __call__(self, operation):
        @wraps(operation)
        def wrapped(*args,**kwargs):
            last_raised = None
            ...

        return wrapped


@withRetry(retries_limit=5)
def run_with_custom_retries_limit(task):
    return task.run()
```

@ 연산 전에 전달된 파라미터를 사용해 데코레이터 객체를 생성한다.  
__init__메서드에 정해진 로직에 따라 초기화 후 @연산이 호출된다.   
데코레이터 객체는 다음 함수를 래핑 하여 __call__매직 메서드를 호출한다.

---

### 5-2.데코레이터 활용

---

+ 파라미터 변환
+ 코드 추적
+ 파라미터 유효성 검사
+ 재시도 로직 구현
+ 일부 반복 작업을 데코레이터로 이동하여 클래스 단순화


---

#### 흔한실수

```
def trace_decorator(function):
    #@wraps(function) <-생략하는 경우 문제 functools.wraps 사용
    def wrapped(*arg,**kwargs):
        ...
    return wrapped
```

----

#### 데코레이터 부작용 처리

데코레이터 함수가 되기 위해 필요한 조건은 가장 안쪽에 정의된 함수여야 한다는 것이다.  
그렇지 않으면 임포트에 문제가 발생할 수 있다.  
그럼에도 불구 하고 때로는 임포트 시에 실행하기 위해 이러한 부작용이 필요한 경우도 있고 반대의 경우도 있다.  

래핑된 함수 바깥에 추가 로직을 구현하는 것은 좋지 않다.  
결과를 의도하고 잘 사용하여야 한다.

---

#### 어느 곳에서나 동작하는 데코레이터 만들기

데코레이터를 만들 때 일반적으로 재사용을 고려하여 함수뿐 아니라 메서드에도 동작하길 바란다.

*args, **kwargs 서명을 사용하여 데코레이터를 정의하면 모든 경우에 사용할 수 있다.

그러나 다음 2가지 이유로 함수의 서명과 비슷하게 데코레이터를 정의하는 것이 좋다.

1. 원래의 함수 모양이 비슷하기 때문에 읽기가 쉽다.
2. 파라미터를 받어서 뭔가를 하려면 *args, **kwargs를 사용하는 것이 불편하다.

그러나 함수에서 사용하던 데코레이터를 메서드에서 사용하려고 하면 문제가 생긴다. 메서드는 첫 번째 인자로 self를 받기 때문이다.

디스크팁터 프로토콜을 구현한 데코레이터 객체를 만들어야 한다.
예제.
```
from functool import wraps
from types import MethodType

class inject_db_driver:

    def __init__(self, function):
        self.function = function
        wraps(self.function)(self)

    def __call__(self,dbstring):
        return self.function(DBDriver(dbstring))

    def __get__(self,instance,owner):
        if instance is None:
            return self
        return slef.__class__(MethodType(self.function,instance))
```
함수를 객체에 바인딩하고 데코레이터를 새로운 호출 가능 객체로 다시 생성한다.

---

### 5-3.데코레이터와 DRY원칙

데코레이터는 DRY원칙을 잘 따른다.

그러나 신중하게 설계되지 않은 데코레이터는 코드의 복잡성을 증가시킨다. 

결론적으로 다음과 같은 사항을 고려했을 경우만 데코레이터 사용을 권한다.

1. 처음부터 데코레이터를 만들지 않는다. 패턴이 생기고, 데코레이터에 대한 추상화가 명확해지면 그때 리팩토링을 한다.

2. 데코레이터가 적어도 3회 이상 필요한 경우에만 구현한다.

3. 데코레이터 코드를 최소한으로 유지한다.

---

