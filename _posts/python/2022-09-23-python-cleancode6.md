---
title: "python clean code chapter6"
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

## 6. 디스크립터로 더 멋진 객체 만들기

데이터 디스크립터, 비데이터 디스크립터 2가지 유형이 있다.

---

### 6-1. 디스크립터

복잡하진 않으나 세부구현시 주의사항이 많다.

+ 최소 2 개의 클래스가 필요하다.
    * 클라이언트 클래스: 디스크립터 구현 기능을 활용할 도메인 모델

    * 디스크립터 클래스: 디스크립터 로직의 구현체

디스크립터 클래스는 다음 매직 매서드 중에 최소 한 개 이상을 포함해야 한다.(python ver3.6기준)
```
__get__
__set__
__delete__
__set_name__
```

다음과 같은 네이밍 컨벤션을 사용한다.

+ ClientClass 클래스 속성으로 descriptor를 갖는다.
+ DescriptorClass
+ client
+ descriptor

여기서의 클래스 속성은 **클래스 본문**에 정의한 속성이다.  
__init__내부에서 정의하지 않음을 주의.(내부에서 정의하면 인스턴스 속성이라 한다.)  
여러 객체가 값을 공유하는 특성이 있다.  


---
#### __get__메서드

```
__get__(self,instance,owner)
```
instance 파라미터는 디스크립터를 호출한 객체를 의미한다.
디스크립터가 행동을 취하려는 객체

owner 파라미터는 인스턴스의 클래스를 의미한다. 

```
class Descriptorclass:
    def __get__(self,instance,owner):
        if instance is None:
            return f"{self.__class__.__name__}.{owner.__name__}"
        return f"value for {instance}"

class ClientClass:

    descriptor = DescriptorClass()
```

ClientClass 에서 직접호출시 네임스페이스와 클래스이름 출력
```
>>>ClientClass.descriptor
'DescriptorClass.ClientClass
#클래스 속성으로 descriptor를 호출
#이때, ClientClass 인스턴스에서 호출 한게 아니므로 instance는 None 이다.

>>>ClientClass().descriptor
'value for <instance ...>'
#인스턴스를 생성하고 descriptor를 호출
```

---

#### __set__메서드

디스크립터 속성에 값을 할당하려고 할 때 호출된다. 
반드시 __set__메서드를 구현해야만 활성화 된다.
```
client.descriptor = 'value'
```
__set__메서드를 구현하지 않으면, 'value'는 descriptor 자체를 덮어 쓴다.

```
def __set__(self,instance,value):
    instance.__dict__[self._name] = value

>>> client.descriptor = 42
>>>client.descriptor
42
#__set__메서드를 구현하지 않았다면 descriptor가 42가 된다.
구현했다면 _name 의 값이 42고, _name값을 보는 것이다.
```
프로퍼티 자리에 놓일 수 있는 것은 디스크립터로 추상화 가능하며, __set__메서드가  @property.setter가 하던 일을 대신 한다.

---

#### __delete__메서드

```
__delete__(self, instance)

#다음과 같은 형태로 호출
>>> del client.descriptor
```

---

#### __set_name__메서드

```
def __set_name__(self, owner, name):
    self.name = name
```
기존 디스크립터는 속성을 처리하려면 디스크립터에 명시적으로 속성을 전달해야 함.

__set_name__메서드를 사용하면 디스크립터에 속성명을 명시적으로 전달하지 않아도 처리가 가능.


---

### 6-2. 디스크립터 유형

__set__이나 __delete__메서드를 구현 했다면 **데이터 디스크립터**.  
그렇지 않고 __get__만을 구현한 디스크립터를 **비데이터 디스크립터**.

---

#### 비데이터 디스크립터

__get__만을 구현한 디스크립터

```
class NonDataDescriptor:
    def __get__(self,instance,owner):
        if instance is None:
            return self
        return 42
class ClientClass:
    descriptor = NonDataDescriptor()

client = ClientClass()
#descriptor 속성은 인스턴스가 아니라 클래스 안에 있음
#client 객체의 사전을 조회하면 그값은 비어있음

vars(client)
{}

client.descriptor
#여기서 .descriptor 속성을 조회하면
#client.__dict__에서 descriptor 라는 키를 찾지 못함
#찾지 못하고 마침내 클래스에서 디스크립터를 찾게 됨.
#그래서 __get__메서드 결과를 반환
#속성의 __dict__에서 키를 찾아보고, 없으면 디스크립터를 찾게됨

client.descriptor = 99
vars(client)
{'descriptor':99}

#속성에 다른 값을 설정하면 인스턴스의 사전이 변경됨
#이제 __dict__는 비어있지 않음.
```

----

#### 데이터 디스크립터

__set__나 __delete__메서드를 구현한 디스크립터
```

class DataDescriptor:
    def __get__(self,instance,owner):
        if instance is None:
            return self
        return 42
    def __set__(self,instance,value):
        instance.__dict__['descriptor'] = value

class ClientClass:
    descriptor = DataDescriptor()

client = ClientClass()
client.descriptor
42

client.descriptor = 99
client.descriptor
42

vars(client)
{'descriptor':99}

#__set__()메서드가 호출되면서 객체의 사전에 값을 설정
#속성을 조회하면 __dict__에서 조회하는 대신 클래스의 descriptor를 먼저 조회

```
비데이터 디스크립터와 데이터 스크립터의 중요한 차이
+ 비데이터 디스크립터: 인스턴스의 __dict__우선순위가 (descriptor클래스 보다) 더 높다.

+ 데이터 디스크립터: descriptor클래스의 우선순위가 (인스턴스의 __dict__보다) 더 높다.

그렇기에 우선순위가 높은 메서드가 먼저 호출

디스크립터의 __set__메서드에서 setattr()이나 할당 표현식을 직접 사용하면 안된다. 무한루프가 발생한다.

__set__메서드와 setattr()이 서로 호출하기때문

---
### 6-3. 디스크립터 실전

만약, 실질적인 코드 반복의 증거가 없거나 복잡성의 대가가 명확하지 않다면 굳이 디스크립터를 사용할 필요가 없다.

디스크립터의 코드가 다소 복잡한 것은 사실이다. 반면 클라이언트 클래스코드는 상당히 간단해진다. 따라서 이 **디스크립터를 여러번 사용한다면,** 충분히 가치가 있다.

디스크립터는 비즈니스로직을 포함하지 않아 독립적이다.
비즈니스 로직의 구현보다는 라이브러리, 프레임워크, 또는 내부 API를 정의하는데 적합하다.

예제.
```
class HistoryTracedAttribute:
    def __init__(self, trace_attribute_name) -> None:
        self.trace_attribute_name = trace_attribute_name
        self._name = None

    def __set_name__(self, owner, name):
        self._name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self._name]

    def __set__(self, instance, value):
        self._track_change_in_value_for_instance(instance, value)
        instance.__dict__[self._name] = value

    def _track_change_in_value_for_instance(self, instance, value):
        self._set_default(instance)
        if self._needs_to_track_change(instance, value):
            instance.__dict__[self.trace_attribute_name].append(value)

    def _needs_to_track_change(self, instance, value) -> bool:
        try:
            current_value = instance.__dict__[self._name]
        except KeyError:
            return True
        return value != current_value

    def _set_default(self, instance):
        instance.__dict__.setdefault(self.trace_attribute_name, [])

class Traveller:

    current_city = HistoryTracedAttribute("cities_visited")

    def __init__(self, name, current_city):
        self.name = name
        self.current_city = current_city
```
---
#### 전역 상태 공유

```
class SharedDataDescriptor:
    def __init__(self, initial_value):
        self.value = initial_value

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.value

    def __set__(self, instance, value):
        self.value = value

class ClientClass:
    descriptor = SharedDataDescriptor("첫 번째 값")

>>> client1 = ClientClass()
>>> client1.descriptor
'첫 번째 값'
>>> client2 = ClientClass()
>>> client2.descriptor
'첫 번째 값'
>>> client2.descriptor = "client2를 위한 값"
>>> client2.descriptor
'client2를 위한 값'
>>> client1.descriptor
'client2를 위한 값'
```
디스크립터 객체에 데이터를 보관하면 모든 객체가 동일한 값에 접근할 수 있다.  
인스턴스의 값을 수정하면 같은 클래스의 다른 모든 인스턴스에서도 값이 수정된다.

이는 ClientClass.descriptor가 고유하기 때문이다.

만약 원하는 상황이 아니라면 각 인스턴스의 __dict__사전에 값을 설정하고 검색해야 한다.

---
#### 약한 참조

인스턴스 __dict__를 사용하지 않으려는 경우 또 다른 대안은 디스크립터 객체가 직접 내부 매핑을 통해 각 인스턴스의 값을 보관하고 반환하는 방법이 있다.  
(이때, 내부 매핑시 사전을 사용해서는 안된다. 가비지 컬렉션 불가)

이를 해결하기 위해서는 사전은 weaker.WeakKeyDictionary를 사용

이러면 단점은 인스턴스 객체는 더 이상 속성을 보유하지 않고, 디스크립터가 속성을 보유하게 된다.  
또한 __hash__메서드로 해시 가능해야 사용가능 하다.

----

### 6-4.디스크립터 분석

파이썬이 어떻게 디스크립터를 사용하는지 알아보자.

---

#### 함수와 메서드

```
class myClass:
    def method(self, ...):
        self.x = 1
#이것은 다음의 정의와 같다.

class myClass: pass

def method(myClass_instance, ...):
    myClass_instance.x = 1

method(myClass())
```

메서드는 객체를 수정하는 또 다른 함수일 뿐이며, 객체 안에서 정의되었기 때문에 객체에 바인딩되어 있다고 말한다.

```
instance = myClass()
instance.method(...)
#이것은 다음과 같다.
instance = myClass()
myClass.method(instance,...)
```
파이썬 추가 예제.

```
class Method:
    def __init__(self,name):
        self.name = name

    def __call__(self, instance, arg1, arg2):
    print(f'{self.name}: {instance} 호출됨. 인자는 {arg1}와 {arg2}입니다.')

class Myclass:
    method = Method('internal Call')


instance = Myclass()
#1
Method('External Call')(intance,'first','second')
#2
instance.method('first','second')
```
첫번째 호출은 동작하지만, 두번째 호출은 에러가 발생한다.   
Method.__call__기준으로 파라미터 위치가 1칸씩 밀려서 arg2에 값이 전달되지 않았다.

이를 해결하려면 디스크립터를 변경해야한다.

```
from types import MethodType

class Method:
    def __init__(self,name):
        self.name = name

    def __call__(self, instance, arg1, arg2):
    print(f'{self.name}: {instance} 호출됨. 인자는 {arg1}와 {arg2}입니다.')

    def __get__(self,instance,owner):
        if instance is None:
            return self

        return MethodType(self, instance)
```

types 모듈의 MethodType을 사용하여 함수를 메서드로 변환하는 것이다.

method는 클래스 속성으로 정의된 객체이고 __get__메서드가 있기 때문에 __get__메서드가 호출된다. 그리고 __get__메서드가 하는 일은 **함수를 메서드로 변환**하는 것이다.

즉, 함수를 작업하려는 객체의 인스턴스에 바인딩한다.

---

#### 메서드를 위한 빌트인 데코레이터

@property, @classmethod, @staticmethod 데코레이터는 디스크립터다.

메서드를 인스턴스가 아닌 클래스에서 직접 호출하면 디스크립터 자체를 반환한다.

@classmethod를 사용하면 디스크립터의 __get__함수가 메서드를 인스턴스에서 호출하든 클래스에서 직접 호출하든 상관없이, 데코레이팅 함수에 첫 번째 파라미터로 메서드를 소유한 클래스를 넘겨준다.

@staticmethod를 사용하면 정의한 파라미터 이외의 파라미터를 넘기지 않도록한다.

---

#### 슬롯

클래스에 __slot__속성을 정의하면 클래스가 기대하는 특정 속성만 정의하고 다른 것은 제한할 수 있다.

__slot__에 정의되지 않은 속성을 동적으로 추가하려고 할 경우 AttributeError가 발생한다.

이 속성을 정의하면 클래스는 정적으로 되고, __dict__속성을 갖지 않는다.

```
class Coordinate2D:

    __slots__ = ('lat','long')

    def __init__(self,lat,long):
        self.lat =lat
        self.long = long

    ...
```

파이썬의 동적인 특성을 없애기에 사용에 주의.  
메모리를 덜 사용하게 된다는 장점이 있다.





