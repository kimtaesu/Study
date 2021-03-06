# Chap02 - 시퀀스 An array of sequences

파이썬에서 제공하는 다양한 시퀀스를 이해하면 코드를 새로 구현할 필요가 없으며, 시퀀스의 공통 인터페이스를 따라 기존 혹은 향후에 구현될 시퀀스 자료형을 적절히 지원하고 활용할 수 있게 API를 정의할 수 있다.



## 2.1 내장 시퀀스 개요

파이썬은 C로 구현된 다음과 같은 시퀀스들을 제공한다.

- ***컨테이너 시퀀스*** : 서로 다른 자료형의 항목들을 담을 수 있는 `list, tuple, collections.deque` 형태
- ***균일 시퀀스*** : 하나의 자료형만 담을 수 있는 `str, bytes, memoryview, array.array` 형태

컨테이너 시퀀스(container sequence)는 객체에 대한 참조를 담고 있으며 객체는 어떠한 자료형도 될 수 있다. 하지만, 균일 시컨스(flat sequence)는 객체에 대한 참조 대신 자신의 메모리 공간에 각 항목의 값을 직접 담는다. 따라서, 균일 시퀀스가 메모리를 더 적게 사용하지만, 문자, 바이트, 숫자 등 기본적인 자료형만 저장할 수 있다. <br />

시퀀스는 다음과 같이 가변성에 따라 분류할 수도 있다.

- ***가변 시퀀스*** : `list, bytearray, array.array, collections.deque, memoryview` 등
- ***불변 시퀀스*** : `tuple, str, bytes` 등



## 2.2 리스트 컴프리헨션과 제너레이터 표현식

리스트 컴프리헨션(listcomp, List Comprehension)이나 제너레이터 표현식(genexp, Generator Expression)을 사용하면 시퀀스를 간단히 생성할 수 있다. 



### 2.2.1 리스트 컴프리헨션과 가독성

아래의 [예제 2-1]과 [예제 2-2]를 보도록 하자. 

```python
# 예제 2-1 : 문자열에서 유니코드 코드 포인트 리스트 만들기 ver01 
>>> symbols = '$¢£¥€¤'
>>> codes = []
>>> for symbol in symbols:
    	codes.append(ord(symbol))
    
>>> codes
[36, 162, 163, 165, 8364, 164]


# 예제 2-2 : 문자열에서 유니코드 코드 포인트 리스트 만들기 ver02
>>> symbols = '$¢£¥€¤'
>>> codes = [ord(symbol) for symbol in symbols]
>>> codes
[36, 162, 163, 165, 8364, 164]
```

[예제 2-1]은 `for` 루프를 이용해서 `codes` 리스트를 만들고, [예제 2-2]는 `listcomp`를 이용해서 `codes`리스트를 만든다. 파이썬에 익숙한 사람들은 리스트 컴프리헨션 방법이 가독성이 더 좋다는 것을 알 수 있다. (실제로 저도 그렇네요ㅎㅎ) <br />

하지만, 그렇다고 해서 리스트 컴프리헨션을 남용해서는 안된다. 리스트 컴프리헨션을 사용할 경우 구문이 두 줄이상 넘어가는 경우나 다중 `for`문을 사용하는 경우 [예제 2-1]처럼 코드를 분할해서 사용하는 것이 오히려 더 낫다. 



### 2.2.2 리스트 컴프리헨션과 `map()/filter()` 비교

위의 [예제 2-1]과 [예제 2-2]를 `map()`과 `filter()`를 사용해서 [예제 2-3]처럼 구현할 수 있다. 

```python
# [예제 2-3] - map/filter 로 만든 codes 리스트
>>> symbols = '$¢£¥€¤'
>>> codes = list(filter(lambda c: c > 1, map(ord, symbols)))
>>> codes
[36, 162, 163, 165, 8364, 164]
```

하지만, 가독성은 [예제 2-2]보다 떨어진다. 



### 2.2.3 데카르트 곱

이번에는 리스트 컴프리헨션을 이용해 데카르트 곱을 구현해 보자. 데카르트 곱(곱집합)에 대해서는 [wikipedia](https://ko.wikipedia.org/wiki/%EA%B3%B1%EC%A7%91%ED%95%A9)를 참고하면 된다. 아래의 그림은 데카르트 곱에 대한 예제이다.

![](./images/descartes.PNG)

다음 [예제 2-4] 와 같이 두 가지 색상과 세 가지 크기의 티셔츠에 대해 데카르트 곱을 통해 리스트를 만들게 되면 총 6가지 항목이 만들어 진다.

```python
# [예제 2-4] 리스트 컴프리헨션을 이용한 데카르트 곱
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> tshirts = [(color, size) for color in colors 
                             for size in sizes]
>>> tshirts
[('black', 'S'),
 ('black', 'M'),
 ('black', 'L'),
 ('white', 'S'),
 ('white', 'M'),
 ('white', 'L')]
```



### 2.2.4 제너레이터 표현식

튜플(tuple), 배열(array) 등의 시퀀스를 초기화 하기위해서는 리스트 컴프리헨션을 사용할 수도 있지만, 리스트를 한번에 통째로 만들지 않고, 반복자 프로토콜(iterator protocol)을 이용하여 항목을 하나씩 생성하는 제너레이터(generator) 표현식을 사용하면 메모리를 더 적게 사용한다. 

제너레이터 표현식은 리스트 컴프리헨션과 동일한 구문을 사용하지만, 대괄호`[]`가 아닌 괄호`()`를 사용한다.

[예제 2-5]는 제너레이터 표현식을 이용해 튜플을 생성하는 코드이다.

```python
# [예제 2-5] - 제너레이터 표현식을 이용한 튜플, 배열 초기화
>>> symbols = '$¢£¥€¤'
>>> tuple(ord(symbol) for symbol in symbols)
(36, 162, 163, 165, 8364, 164)

>>> import array
>>> array.array('I', (ord(symbol) for symbol in symbols))
array('I', [36, 162, 163, 165, 8364, 164])
```



아래의 [예제 2-6]은 데카르트 곱을 제너레이터 표현식을 사용해 [예제 2-4]를 구현한 코드이다. [예제 2-4]와 달리 [예제 2-6]은 생성된 리스트의 항목을 메모리에 할당하지 않는다. 제너레이터 표현식은 한번에 한 항목을 생성할 수 있도록 설정되어 있기 때문에 [예제 2-6]과 같이 `for` 루프에 데이터를 전달한다. 따라서, 항목이 많은 리스트를 생성할 때 생길 수 있는 메모리 부족 문제를 해결할 수 있다.

```python
# [예제 2-6] - 제너레이터 표현식을 이용한 데카르트 곱
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> for tshirt in ('%s %s' % (c, s) for c in colors for s in sizes):
    	print(tshirt)

black S
black M
black L
white S
white M
white L
```



## 2.3 튜플은 단순한 불변 리스트가 아니다

튜플은 단순히 *'불변 리스트'* 로 사용할 수 있지만, 필드명이 없는 *레코드* 로 사용할 수도 있다.

### 2.3.1 레코드로서의 튜플

튜플은 레코드로서 사용될 수 있다. 튜플의 각 항목은 레코드의 필드 하나를 의미하며 항목의 위치가 의미를 결정한다. 

튜플을 단순히 '불변 리스트'로 생각한다면 경우에 따라 항목의 크기와 순서가 중요할 수도 있고 그렇지 않을 수도 있다.  하지만, 튜플을 필드의 집합으로 사용하는 경우에는 항목 수가 고정되어 있고 항목의 순서가 중요하다.

[예제 2-7]은 튜플을 레코드로 사용하는 경우를 보여준다. 

```python
# [예제 2-7] : 레코드로 사용된 튜플
lax_coordinates = (33.9425, -118.408056)  # LA 국제공항의 위도와 경도
city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)  # 도쿄에 대한 데이터(지명, 연도, 인구수, 인구 변화율, 면적)
traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567'), ('ESP', 'XDA206856')]  # (국가코드, 여권 번호) 튜플 리스트

for passport in sorted(traveler_ids):  # 리스트를 반복할 때 passport 변수가 각 튜플에 바인딩된다.
    print('%s/%s' % passport)  # 퍼센트(%) 포맷 연산자는 튜플을 이해하고 각 항목을 하나의 필드로 처리

''' 출력결과
BRA/CE342567
ESP/XDA206856
USA/31195855
'''

for country, _ in traveler_ids:  # 언패킹을 이용한 각 항목 가져오기 '_'는 dummy variable
    print(country)
    
'''출력결과
USA
BRA
ESP
'''
```



### 2.3.2 튜플 언패킹(Unpacking)

[예제 2-7]에서 `city, year, pop, chg, area ` 변수에 `('Tokyo', 2003, 32450, 0.66, 8014)`를 각각 할당했다. 이러한 방법이 바로 *튜플 언패킹(tuple unpacking)* 이다. 언패킹은 반복 가능한 객체라면 어느 객체든 적용할 수 있다. 

튜플 언패킹은 **병렬 할당**(parallel assignment)을 할 때 주로 사용한다. 아래의 코드는 튜플을 변수에 병렬할당하는 예제이다. 

```python
>>> lax_coordinates = (33.9425, -118.408056)
>>> latitude, longitude = lax_coordinates  # 튜플 언패킹
>>> print(latitude)
33.9425
>>> print(longitude)
-118.408056
```



튜플 언패킹을 이용하면 임시 변수를 사용하지 않고도 두 변수의 값을 서로 교환할 수 있다.

```python
>>> a, b = (1, 2)
>>> print('a =', a, 'b =', b)
a = 1 b = 2

# 두 변수 교환하기
>>> b, a = a, b
>>> print('a =', a, 'b =', b)
a = 2 b = 1
```



또한, 다음과 같이 함수를 호출할 때 인수 앞에 `*` 을 붙여 튜플을 언패킹 할 수 있다.

```python
>>> divmod(20, 8)
(2, 4)

>>> t= (20, 8)
>>> divmod(*t)
(2, 4)

>>> quotient, remainder = divmod(*t)
>>> quotient, remainder
(2, 4)
```



아래의 예제 코드는 `os.path.split()` 함수를 이용하여 파일에서 경로명(`path`) 과 파일명(`filename`)을 가져오는 코드이다.

```python
>>> import os

>>> path, filename = os.path.split('./fluent/python/chap02/sequences.py')
>>> print(path)  # 경로
./fluent/python/chap02
>>> print(filename)  # 파일명
sequences.py
```



#### 초과 항목을 잡기위한 `*` 사용하기

파이썬 3에서는 `*`를 이용해 아래와 같이 병렬 할당에도 사용할 수 있다.  언패킹을 하고난 뒤 나머지 값들을  `*`를 사용하여 할당해 줄 수 있다. 이때 `*`를 적용한 변수는 리스트`[]` 형태로 반환된다.

```python
>>> a, b, *rest = (1, 2, 3, 4, 5)
>>> print(a, b, rest)
1 2 [3, 4, 5]

>>> a, b, *rest = range(5)
>>> print(a, b, rest)
0 1 [2, 3, 4]

>>> a, b, *rest = range(3)
>>> print(a, b, rest)
0 1 [2]

>>> a, b, *rest = range(2)
>>> print(a, b, rest)
0 1 []
```

 

병렬 할당의 경우 `*`는 단 하나의 변수에만 적용할 수 있다. 대신 `*` 위치는 어떠한 곳에도 상관이 없다. 

```python
>>> *head, b, c = range(5)
>>> print(start, b, c)
[0, 1, 2] 3 4

>>> a, *body, c = range(5)
>>> print(a, body, c)
0 [1, 2, 3] 4
```



### 2.3.3 내포된 튜플 언패킹 - Nested tuple unpacking

튜플은 `(a, b, (c, d))` 처럼 튜플 안에 튜플이 내포된(nested) 형태로 되어있을 수도 있다. 파이썬은 내포된 튜플의 경우도 변수 할당만 제대로 해주면 무리없이 언패킹이 가능하다. [예제 2-8]을 보도록 하자.

```python
# [예제 2-8]: longitude에 접근하기 위해 내포된 튜플 언패킹하기

metro_areas = [
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)), 
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)), 
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)), 
    ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]

print('{:15} | {:^9} | {:^9}'.format('', 'lat.', 'long.'))
fmt = '{:15} | {:9.4f} | {:9.4f}'
for name, cc, pop, (latitude, longitude) in metro_areas:
    print(fmt.format(name, latitude, longitude))
    
'''출력결과
                |   lat.    |   long.  
Tokyo           |   35.6897 |  139.6917
Delhi NCR       |   28.6139 |   77.2089
Mexico City     |   19.4333 |  -99.1333
New York-Newark |   40.8086 |  -74.0204
Sao Paulo       |  -23.5478 |  -46.6358
'''
```



지금까지 살펴본 예를 통해 튜플은 아주 편리하게 사용할 수 있다는 것을 알 수 있다. 그러나 레코드로 사용하기에는 부족한 점이 있다. 때로는 필드에 이름을 붙여야할 경우도 있다. 이를 위해 `collections` 모듈의 `namedtuple()` 함수가 고안되었다.



### 2.3.4 Named tuples

`collections.namedtuple()` 함수는 필드명과 클래스명을 추가한 튜플의 서브클래스를 생성하는 팩토리 함수이다.  `collections.namedtuple(typename, field_names, verbose=False, rename=False)`을 입력값으로 받으며, *field_names* 를 통해 `namedtuple()`의 키 즉, 필드명(fieldname)을 정의할 수 있다.

[예제 2-9]는 `namedtuple()` 을 정의해 도시에 대한 정보를 담고 있는 객체를 만드는 예시이다.

```python
>>> from collections import namedtuple

>>> City = namedtuple('City', 'name country population coordinates')
>>> seoul = City('Seoul', 'KR', 10.204, (37.5668237, 126.9779504))
>>> print(seoul)
City(name='Seoul', country='KR', population=10.204, coordinates=(37.5668237, 126.9779504))

>>> print(seoul.population)
10.204

>>> print(seoul.coordinates)
(37.5668237, 126.9779504)

>>> print(seoul[1])
KR
```



`namedtuple()`은 튜플에서 상속받은 속성 외에 몇가지 속송들을 더 가지고 있다. 

- `_fields` : 클래스의 필드명을 담고 있는 튜플을 반환한다.
- `_make()` : 반복형 객체로 부터 namedtuple을 만든다. 
- `_asdict()` : namedtuple 에서 만들어진 `collections.OrderedDict` 객체를 반환한다. 

[예제 2-10]은 `_fields` 클래스 속성, `_make(iterable)` 클래스 메소드, `_asdict()` 객체 메소드를 보여주는 예제이다.

```python
# [예제 2-10] : namedtuple의 속성과 메소드

>>> print(City._fields)
('name', 'country', 'population', 'coordinates')

>>> LatLong = namedtuple('LatLong', 'lat long')
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, LatLong(28.613889, 77.208889))
>>> delhi = City._make(delhi_data)
>>> print(delhi._asdict())
OrderedDict([('name', 'Delhi NCR'), ('country', 'IN'), ('population', 21.935), ('coordinates', LatLong(lat=28.613889, long=77.208889))])

>>> for key, value in delhi._asdict().items():
        print(key + ':', value

name: Delhi NCR
country: IN
population: 21.935
coordinates: LatLong(lat=28.613889, long=77.208889)
```



### 2.3.5 불변 리스트로서의 튜플

튜플을 불변 리스트로 사용할 때, 튜플과 리스트가 얼마나 비슷한지 알고 있으면 도움이 된다.

| 메소드                     | 리스트 | 튜플 | 설명                                                         |
| -------------------------- | ------ | ---- | ------------------------------------------------------------ |
| `s.__add__(s2)`            | O      | O    | `s + s2`: 리스트를 연결한다.                                 |
| `s.__iadd__(s2)`           | O      |      | `s += s2` : 리스트를 연결하고 `s`에 저장한다.                |
| `s.append(e)`              | O      |      | 제일 뒤에 원소를 하나 추가한다.                              |
| `s.clear()`                | O      |      | 모든 항목을 삭제한다.                                        |
| `s.__contains__(e)`        | O      | O    | `e in s`                                                     |
| `s.copy()`                 | O      |      | 리스트를 복사한다.                                           |
| `s.count(e)`               | O      | O    | `e`가 나타난 횟수를 계산한다.                                |
| `s.__delitem__(p)`         | O      |      | `p` 위치의 원소를 삭제한다.                                  |
| `s.extend(it)`             | O      |      | 반복형 `it` 안에 있는 원소를 추가한다.                       |
| `s.__getitem__(p)`         | O      | O    | `s[p]`: `p` 위치의 원소를 가져온다.                          |
| `s.__getnewargs__()`       |        | O    | `pickle` 을 이용해서 최적화된 직렬화를 지원한다.             |
| `s.index(e)`               | O      | O    | `s` 안에서 `e` 가 처음 나타나는 위치를 찾는다.               |
| `s.insert(p, e)`           | O      |      | `p` 위치에 있는 원소 앞에 `e` 원소를 삽입한다.               |
| `s.__iter__()`             | O      | O    | 반복자를 가져온다.                                           |
| `s.__len__()`              | O      | O    | `len(s)` : 항목 개수를 구한다.                               |
| `s.__mul__(n)`             | O      | O    | `s * n` : 문자열을 반복한다.                                 |
| `s.__imul__(n)`            | O      |      | `s *= n` : 문자열을 반복하여 `s` 에 저장한다.                |
| `s.__rmul__(n)`            | O      | O    | `n * s` : 역순 반복 추가 메소드                              |
| `s.pop([p])`               | O      |      | 마지막 항목이나 `p` 위치의 항목을 제거하고 반환한다.         |
| `s.remove(e)`              | O      |      | `e` 값을 가진 첫 번째 항목을 제거한다.                       |
| `s.reverse()`              | O      |      | 항목을 역순으로 정렬한 후 `s` 에 저장한다.                   |
| `s.__reversec__()`         | O      |      | 마지막에서 첫 번째 항목까지 반복하는 반복자를 반환한다.      |
| `s.__setitem__(p, e)`      | O      |      | `s[p] = e` : `e` 를 `p` 위치에 저장하고, 기존항목을 덮어쓴다. |
| `s.sort([key], [reverse])` | O      |      | 선택적인 `key` 와 `reverse`에 따라 항목을 정렬하고 `s` 에 저장한다. |

