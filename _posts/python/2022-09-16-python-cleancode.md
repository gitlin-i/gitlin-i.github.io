---
title: "python clean code chapter2"
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

## 2. 파이썬스러운 코드

프로그래밍에서 관용구(idiom)는 특정 작업을 수행하기 위해 코드를 작성하는 특별한 방법이다. 매번 동일한 구조를 반복하고 따르는 것이 일반적이다.


각 언어별로 작업을 처리하는 고유한 관용구를 가지고 있는데, 특히 파이썬에서는 **파이썬스럽다(Pythonic)** 고 한다.

---
### 2-1. 인덱스와 슬라이스

```
>>>my_number = [1,1,2,3,5,8,9,9,9,10]
>>>my_number[2:5]
[2,3,5]
```
Slice를 사용하여 특정구간의 요소를 구할 수 있다.
시작 인덱스의 값은 포함하고, 끝 인덱스의 값은 포함하지 않는다.

[시작 : 끝 : 간격]

각 값이 공백이면 None으로 간주. 값이 None 일 경우, 시작은 맨 처음부터, 끝은 맨 끝까지, 간격은 1로 바로 다음 값을 찾아간다.

음수의 경우 맨 끝값을 -1, 앞으로 갈 수록 -1을 더한다.

```
>>> interval = slice(1,7,2)
>>> my_number[interval]
[1,3,8]
```
slice는 파이썬 내장 객체로 직접 빌드하여 전달할 수도 있다.

---
### 2-2. 자체 시퀀스 생성

클래스가 표준 라이브러리 객체를 감싸는 래퍼인 경우 기본 객체에 가능한 많은 동작을 위임할 수 있다.  
즉, 클래스가 리스트의 래퍼인 경우 동일한 메서드를 호출하여 호환성을 유지할 수 있다.

```
class Items:
  def __init__(self, *values):
    self._values = list(values)

  def __len__(self):
    return len(self.values)

  def __getitem__(self,item):
    return self.values.__getitem__(item)

```

이 예제는 캡슐화 방식을 사용했다. 다른 방법으로 상속을 사용할 수도 있다.

---

### 2-3. 컨텍스트 관리자(Context Manager)

컨텍스트 관리자는 파이썬이 제공하는 유용한 기능이다. 이것이 특별히 유용한 이유는 패턴에 잘 대응되기 때문이다.  
이 패턴은 사실상 모든 코드에 적용될 수 있으며, 사전조건과 사후조건을 가지고 있다.

```
fd = open(filename)
try:
  process_file(fd)
finally:
  fd.close()
```
파일을 열면 누수를 막기위해 작업이 끝나면 적절히 닫히길 기대한다. 서비스나 소켓에 대한 연결을 열었을 때도 적절하게 닫히거나 임시 파일을 제거하는 작업 등을 해야한다.  
위와 같이 finally 안에 정리하는 것도 방법이지만,
파이썬스러운 방법으로 구현할 수도 있다.

```
with open(filename) as fd:
  process_file(fd)
```

with문(PEP-343)은 컨텍스트 관리자로 진입하게 한다.  
예외가 발생한 경우에도 블록이 완료되면 파일이 자동으로 닫힌다.  
with문은 __enter__와 __exit__두 개의 매직 메서드로 구성된다.  

with문은 __enter__메서드를 호출하고 이 메서드가 무엇을 반환하든 as 이후에 지정된 변수에 할당한다.  
또한, __enter__메서드가 특정한 값을 반환할 필요는 없으며, 반환하더라도 필요하지 않더라도 변수에 할당하지 않을 수도 있다.

이 라인이 실행되면 다른 파이썬 코드가 실행될 수 있는 새로운 컨텍스트로 진입한다. 해당 블록에 대한 마지막문장이 끝나면 컨텍스트는 종료되며 __exit__메서드를 호출함을 의미한다.


컨텍스트 관리자 블록 내에 예외 또는 오류가 있는 경우에도 __exit__메서드가 여전히 호출되므로 안전하게 실행하는데 편하다. __exit__메서드는 블록에서 예외가 발생한 경우 해당 예외를 파라미터로 받기 때문에 임의의 방법으로 처리할 수도 있다.

__exit__메서드의 반환값은 예외가 없다면 None이다.  
__exit__반환 값이 True면, 잠재적으로 발생한 예외를 호출자에게 전파하지 않고 멈춘다는 것을 뜻하니 주의하여야 한다.

---

### 2-4. 컨텍스트 관리자 구현

__enter__와 __exit__매직 메서드만 구현하면 해당 객체는 컨텍스트 관리자 프로토콜을 지원할 수 있다.

표준라이브러리 contextlib 모듈의 contextmanager **데코레이터**를 적용하면 해당 **함수**의 코드를 컨텍스트 관리자로 변환한다.  
이때, 함수는 **제너레이터**라는 특수한 함수의 형태여야 한다.

```
import contextlib

@contextlib.contextmanager
def db_handler():
  stop_database()
  yield
  start_database()

with db_handler():
  db_backup()
```
중요한 것은 yield문 앞을 __enter__메서드, 뒤를 __exit__메서드로 볼 수 있다.  
기존 함수를 리팩토링하기 쉬운 장점이 있다.

---
### 2-5 프로퍼티, 속성과 객체 메서드의 다른 타입들

public, private, protected 프로퍼티를 가지는 다른 언어들과는 다르게 파이썬 객체의 **모든 프로퍼티와 함수는 public**이다.

#### 파이썬에서의 밑줄

_<속성명> 으로 선언한 속성은 해당 클래스 내부에서만 사용되고 호출자는 이 속성에 접근하지 않아야한다. 객체의 인터페이스로 공개하는 용도가 아니라면 모든 멤버에 접두사로 하나의 밑줄을 사용한는 것이 좋다.

__<속성명> 이중 밑줄로 선언한 속성은 private 속성을 만들 수 있다는 오해가 있다.  
밑줄 두개를 사용하면 실제로 파이썬은 다른 이름을 만든다.


**이름 맹글링(name mangling)**
실제 코드의 속성을 다음과 같이 만든다.

**'_<클래스명>__<속성명>"**

그래서 __<속성명>으로 선언한 속성을 그대로 접근하려하면, 찾을 수 없다고 한다. 위와 같이 접근하려고 하면 접근이 된다.

이중 밑줄은 **파이썬스러운** 코드가 아니다. 속성을 private으로 정의하려는 경우 하나의 밑줄을 사용하고 파이썬스러운 관습을 지키도록 해야한다.

#### 프로퍼티

프로퍼티는 객체의 어떤 속성에 대한 접근을 제어하려는 경우 사용한다. 이렇게 하는 것 또한 파이썬스러운 코드이다.


자바와 같은 다른 프로그래밍 언어에서는 접근 메서드(getter,setter)를 만들지만 파이썬에서는 프로퍼티를 사용한다.

```
~~~
@property
def email(self):
  return self._email

@email.setter
def email(self,new_mail):
  self._email = new_email
~~~
```

getter에 해당하는 메서드를 @property
setter에 해당하는 메서드를 @property_name.setter
```
>>>user.email = 'abcd@a.com'
>>>user.email
'abcd@a.com'
```
이렇게 하면 기대하는 값이 명확하며 간단하다.

---

### 2-6. 이터러블 객체

반복가능한 객체. 리스트, 튜플, 세트 및 사전  
for를 이용한 값을 반복적으로 가져올 수 있다.

물론 위의 내장형 객체말고도, 자체 이터러블,이터레이터를 만들 수도 있다.   
**이터러블은 __iter__매직 메서드를 구현한 객체.**   
**이터레이터는 __next__매직 메서드를 구현한 객체**를 말한다.

for e in object: 형태로 객체를 반복할 수 있는지 확인하기 위해 파이썬은 고수준에서 다음 두 가지를 차례로 검사한다.

>객체가 __next__나 __iter__메서드 중 하나를 포함하는지 여부  
>객체가 시퀀스이고, __len__과 __getitem__를 모두 가졌는지 여부

for 루프를 반복하려고 할 때  
1. iter() 함수 호출
2. 해당 객체에 __iter__메서드가 있는지 확인
3. 있다면 __iter__메서드 실행, self를 반환하여 자신이 이터러블임을 나타냄.
4. 각 단계에 next() 함수 호출, __next__메서드에 위임  
(__next__메서드에서 요소를 어떻게 생산하고 반환할지를 결정.
StopIteration 예외가 발생할때까지)  

반복 프로토콜은 이터러블 객체가 이터레이터를 생성하고 이것을 사용해 반복한다.

이때, __iter__에 **제너레이터**를 사용할 수 있다.
이를 **컨테이너 이터러블**이라고 한다.

---

#### 시퀀스

시퀀스는 __len__과 __getitem__을 구현하고 첫 번째 인덱스 0부터 시작하여 포함된 요소를 한 번에 하나씩 차례로 가져올 수 있어야 한다.  
(__getitem__을 올바르게 구현하여야한다. 그렇지 않으면 작동이 되지 않는다.)

이터러블을 사용하면 메모리를 적게 사용하지만 n번째 요소를 얻기 위한 시간복잡도는 O(n)이다.

하지만 시퀀스로 구현하면 더 많은 메모리가 사용되지만 (모든 것을 한 번에 보관해야하므로) 특정요소를 가져오기 위한 인덱싱의 시간 복잡도는 O(1)이다.


---

### 2-7. 컨테이너 객체

컨테이너는 __contains__메서드를 구현한 객체.  
이 메서드는 in 키워드가 발견될 때 호출된다.  
일반적으로 Boolean값을 반환.

```
element in container
#이 코드는 다음과 같이 해석된다.
container.__contains__(element)
```
예제, 2차원 지도에서 특정 위치에 표시를 해야한다.
```
def mark_coordinate(grid,coord):
  if 0 <= coord.x < grid.width  and 0<= coord.y < grid.height:
    grid[coord] = MARKED
```
if문이 난해해보이고 의도가 무엇인지 이해하기 어렵고 직관적이지 못하다.  
무엇보다 매번 경계선 검사를 위해 if 문을 중복해서 호출한다.
```
class Boundaries:
  def __init__(self,width,height):
    self.width = width
    self.height = height
  def __contains__(self, coord):
    x,y = coord
    return 0 <= x <self.width and 0 <= y < self.height
class Grid:
  def __init__(self,width,height)
    self.width = width
    self.height = height
    self.limits = Boundaries(width,height)

  def __contains__(self, coord):
    return coord in self.limits
######
def mark_coordinate(grid,coord):
  if coord in grid:
    grid[coord] = MARKED
```

---

#### 객체의 동적인 속성

__getattr__매직 메서드를 사용해 객체에서 속성을 얻는 방법을 제어할 수 있다. 심지어 새로운 속성을 만들 수도 있다.
```
~~~
def __getattr__(self,attr):
  if attr.startwith("fallback_"):
    name = attr.replace("fallback_","")
    return f"[fallback resolved] {name}"
  raise AttributeError(f"{self.__class__.__name__}에는 {attr} 속성이 없음.)
```
__getattr__같은 동적 메서드를 구현할 때는 AttributeError를 발생시켜 주어야함

---

#### 호출형(callable) 객체 

__call__매직 메서드를 사용하면 객체를 일반 함수처럼 호출할 수 있다. 여기에 전달된 모든 파라미터는 __call__메서드에 그대로 전달된다.

이렇게 사용하는 주된 이점은 객체에는 **상태**가 있기 때문에 함수 호출 사이에 정보를 저장할 수 있다는 점이다.

이 메서드는 객체를 파라미터가 있는 함수처럼 혹은, 정보를 기억하는 함수처럼 사용할 경우 유용하다.

```
class CallCount:
  def __init__(self):
    self._counts = defaultdict(int)
  def __call__(self, argument):
    self._counts[argument] += 1
    return self._counts[argument]
####

>>> cc = CallCount()
>>> cc(1)
1
>>> cc(2)
2
>>> cc(1)
3
>>> cc('a')
1
```

### 2-8.파이썬에서 유의할 점

1. 변경 가능한 객체를 함수의 기본 인자(파라미터의 기본값)로 사용하면 안된다.
2. 내장(built-in) 타입 확장시 collections 모듈을 사용.





















