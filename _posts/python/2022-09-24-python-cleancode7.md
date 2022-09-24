---
title: "python clean code chapter7"
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
## 7. 제너레이터 사용하기

제너레이터는 전통적인 언어와 파이썬을 구분 짓는 특징적인 기능.  
메모리를 적게 사용하는 반복을 위한 방법.

제너레이터의 주요 목적은 **메모리를 절약**하는 것.


---

### 7-1. 제너레이터 만들기

```
def  _load_purchases(filename):
  purchases = []
  with open(filename) as f:
    for line in f:
      *_, price_raw = line.partition(",")
      purchases.append(float(price_raw))

  return purchases
```
이 코드는 정상적인 결과를 반환한다. 파일에서 모든 정보를 읽어서 리스트에 저장한다. 파일에 많은 데이터가 있다면 로드하는데 시간이 오래 걸리고, 메인 메모리에 담지 못할 만큼 큰 데이터일 수도 있다.

그런데 한 번에 하나의 데이터만을 사용하고 있다면, 굳이 파일의 모든 데이터를 한 번에 모두 읽어 메모리에 보관해야 할 이유가 있을까?

해결책은 제너레이터를 만드는 것이다. 전체를 보관하는 대신 그때그때 값을 가져오는 것이다. 
```
def  _load_purchases(filename):
  with open(filename) as f:
    for line in f:
      *_, price_raw = line.partition(",")
      yield float(price_raw)
```
파이썬에서는 **yield 키워드**를 사용하면 제너레이터가 된다.

모든 제너레이터 객체는 **이터러블**이다.

---
#### 컴프리헨션(comprehension)

컴프리헨션에 의해 정의될 수 있는 리스트나 세트,사전 처럼
제너레이터도 표현식으로 정의될 수 있다.

이터러블 연산이 가능한 함수에 직접 전달할 수도 있다.
```
>>> [x**2 for x in range(10)]
[0,1,4,9,16,...]
>>> (x**2 for x in range(10))
<generator ...>
>>> sum(x**2 for x in range(10))
285
```
---

### 7-2. 이상적인 반복

---
#### 관용적인 반복 코드

eunmerate() 이터러블을 입력받아 인덱스 번호와 원본의 원소를 튜플 형태로 변환하여 eunmerate 객체를 반환.


for 루프를 사용하기 위해서는 __iter__메서드 작성이 필요.
(이러터블 객체)
__next__메서드 작성시 객체는 이터레이터  

next() 내장 함수를 이용하여 이터러블 요소를 반환.
next() 함수에 두번째 파라미터로 기본값 설정 가능  
(반복가능한 요소가 없을 경우 기본값 반환)

제너레이터는 무한루프로부터 완벽하게 안전.

---
#### itertools

+ 여러번 반복
itertools.tee(iterable, n) 기존 iterable을 n개의 iterable로 변환. (itertool 사용권장)

+ 중첩 루프
다음은 피해야 할 코드이다.
```
def search_neseted_bad(array, desired_value):
  coords = None
  for i, row in enumerate(array):
    for j, cell in enumerate(row):
      if cell == desired_value:
        coords = (i,j)
        break
    if coords is not None:
      break
  if coords is None:
  raise ValueError('{} not found'.format(desired_value))      
  return coords
```
다음은 종료 플래그를 사용하지 않은 보다 간단하고 컴팩트한 형태의 예이다.

```
def _iterate_array2d(array2d):
  for i,row in enmerate(array2d):
    for j,cell in enumerate(row):
      yield (i,j), cell

def search_nested(array,desired_value):
  try:
    coords = next(
      coord for (coord,cell) in _iterate_array2d(array)
      if cell == desired_value
    )

  except StopIteration:
    raise ValueError('{} not found'.format(desired_value))

  return coords
  ```
중첩을 풀어 1차원 루프로 만들면 더 깔끔하다.

---

#### 이터러블이 가능한 시퀀스 객체

__iter__매직 메서드를 구현한 객체는 for 루프에서 사용할 수 있다. 그러나 __iter__메서드가 없을 경우 객체가 **시퀀스**인지 확인한다. 시퀀스 또한 반복가능하다.

>시퀀스(Sequence) 객체: __getitem__과 __len__매직메서드를 구현한 경우(IndexError 예외가 발생할 때까지 순서대로 값을 제공)

다만 객체가 __iter__를 구현하지 않았을 경우를 대비한 방법임을 주의하라.

객체가 시퀀스여서 우연히 반복이 가능할 수 있지만, 기본적으로 반복을 위한 객체를 디자인할 때는 __iter__메서드를 구현하여 정식 이터러블 객체를 만들어야 한다.

---

### 7-3. 코루틴(coroutine)

제너레이터를 코루틴으로 활용할 수도 있다.

>코루틴: 함수나 메서드 같은 서브루틴이 메인루틴과 종속관계를 가진 것과 다르게, **메인 루틴과 대등한 관계**로 협력하는 모습에서 코루틴이라 한다.
(COoperative ROUTINE, COROUTINE)

---
#### close()

이 메서드를 호출하면 제너레이터에서 GeneratorExit 예외가 발생한다.   
이 예외를 따로 처리하지 않으면 제너레이터가 값 생성을 중지하고 반복이 중지된다.

---
#### throw()

이 메서드는 현재 제너레이터가 중단된 위치에서 예외를 던진다.  
제너레이터가 예외를 처리했으면 해당 except절에 있는 코드가 호출되고, 예외를 처리하지 않았으면 예외가 호출자에게 전파된다.

---

#### send()

이 메서드는 제너레이터와 코루틴을 **구분하는 기준**이다.  
send()메서드를 사용했다는 것은 yield키워드가 할당 구문의 오른쪽에 나오게 되고 인자 값을 받아서 다른 곳에 할당할 수 있음을 뜻한다.
```
recieve = yield produced
```

```
dd
```

