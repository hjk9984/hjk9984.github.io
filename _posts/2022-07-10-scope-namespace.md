---
title: "[Python] Scope와 Namespace"
categories:
  - Blog
classes: wide
tags:
  - python
  - scope
  - namespace
  - LEGB
---

## Scope & Namespace
* **Namespace** : 객체에 매핑된 식별자(이름)으로 구성된 콜렉션. 파이썬에서는 딕셔너리로 정의되어 있다.
* **Scope** : 변수가 접근 가능한 영역(유효범위).

쉽게 말하자면 코드를 실행할 때 변수를 참고할 수 있는 집합을 **namespace**, 어떤 객체에 접근할 수 있는 현재 실행중인 코드 위치를 **scope**라고 할 수 있다.

[공식 문서](https://docs.python.org/3/tutorial/classes.html)에서의 정의는 아래와 같다.
> A **namespace** is a mapping from names to objects.
>
> A **scope** is a textual region of a Python program where a namespace is directly accessible. 

여러 글들을 보면 namespace와 scope를 설명하는 방식이 다른 걸 확인할 수 있다. scope는 컴퓨터 분야에서 통용되는 말이다보니 **프로그래밍 언어마다 적용되는 방식이 다르기 때문**이다. 이 포스팅은 python에 적용되는 방식을 살펴본다. 각 언어마다 어떻게 쓰이는 지 확인하려면 [이 글](https://en.wikipedia.org/wiki/Scope_(computer_science)#Python)을 참조하길 바란다.

<br>

### Namespace

프로그램을 실행하다보면 여러 네임스페이스가 생기기 마련이다. 아래의 예제를 살펴보자.
```python
x = 1     # global namespace {'x': 1}
print(x)  # 1
print(y)  # NameError

y = 2     # global namespace {'x': 1, 'y': 2}
print(y)  # 2

def foo():
  x = 3   # global namespace {'x': 1, 'y': 2}, local namespace {'x': 3}
```
객체에 식별자가 할당(x = 10)되면 look-up table(딕셔너리)인 namespace에 식별자와 해당 값이 `key-value` 형식으로 들어가게 된다. 할당 이후 namespace를 통해 객체를 참조할 수 있다. 그리고 다른 namespace에 같은 식별자로 정의된 변수(x)가 존재할 수 있다.

이렇게 식별자가 같은 변수(위 코드에서는 x)가 다른 namespace에 있을 경우 **[variable shadowing](https://en.wikipedia.org/wiki/Variable_shadowing)** 이라는 문제가 발생하고 유지보수가 안 좋은 영향을 끼친다. 그래서 보통 같은 변수명을 쓰는 것은 전혀 좋지 못한 습관이다.

<br>

### Scope
코드가 실행되면서 여러 namespace가 생기는 데, 어느 네임스페이스에 접근할 지를 도와주는 것이 **scope**이다. 

scope는 4가지로 나눠 볼 수 있다.
* **local scope**: 함수나 클래스의 블록 안에서 정의된 지역변수들을 포함하는 scope
* **enclosing scope**: non local scope로도 불리며, 어느 블록을 기준에서 블록을 감싸는 scope로 볼 수 있다.
* **global scope**: 함수나 클래스 외부의 전역 변수들을 포함하는 scope
* **built-in scope**: 빌트인 변수들을 포함하는 scope

이렇게 4가지로 분류된 scope는 위 순서대로 **local, enclosing, global, built-in scope 순서대로 변수를 탐색되고** 이를 **LEGB rule**이라고 한다. built-in scope까지 탐색하고 찾지 못하면 `NameError`가 발생한다.

공식문서에 따르면 코드를 실행 중일때 언제든지 namespace에 직접 액세스할 수 있는 **3개 혹은 4개의 중첩된 scope**가 있다고 한다. 이 말의 토대로 생각해보면 어느 함수 `tmp_func`를 호출했고 코드의 실행 시점이 `tmp_func`내부에 있을 때, local scope가 `tmp_func`내부를 가리킨다고 이해할 수 있다.

<br>

#### Example
```python
a = 10
b = 100

def foo():
    a = 20
    c = 200

    def hoo():
        a = 30
        print(f"a in hoo: {a}")   # local scope -> in hoo
        print(f"b in hoo: {b}")   # global scope
        print(f"c in hoo: {c}")   # enclosing scope  
    
    hoo()
    print(f"a in foo: {a}")        

foo()
print(f"a in global: {a}")    # global scope

# output:
# a in hoo: 30
# b in hoo: 100
# a in foo: 20
# a in global: 10
```
같은 scope내에서는 변수에 접근에 제약이 없지만, 다른 scope에서 선언된 변수에 접근할 때는 정해진 규칙을 지켜야 한다. 이 규칙에 따르면 **제일 안쪽의 scope부터 순서대로 탐색**을 해야 한다. 

예를 들어 함수 hoo 안의 변수 b는 local, enclosing(nonlocal) scope 순으로 탐색하지만 해당 scope에 선언되지 않았기 때문에 global scope에서 선언된 변수(100)에 접근한다. 하지만 함수 hoo 안의 변수 `a`의 경우 hoo 블록 안에서 선언되었기 때문에 local scope에서 바로 변수 `a`에 접근한다.

```python
a = 10
def foo1():
  print(a)    # read

def foo2():
  a = 20    #write
  print(f"a in foo2 : {a}")
  print(hex(id(a)))

foo1()
foo2()
print(f"a in global: {a}")
print(hex(id(a)))

# output:
# 10
# a in foo2 : 20
# 0x102bdc350
# a in global: 10
# 0x102bdc210
```
위의 예시에서 함수 **foo1 블록 안에서는 LEGB rule을 따라 local scope 거쳐 global scope에서 `a`에 접근한다(read)**. 이와 반대로 함수 foo2의 경우는 `a`라는 식별자에 20이라는 값(리터럴)을 할당(write)한다. foo2에서는 **local namespace에 `a`가 새로 포함되고 foo2의 local scope의 `a`에 접근**한다. 이후 마지막 줄에서 사용되는 변수는 global scope의 변수 a를 사용한다.

<br>

### global, nonlocal
위의 예시를 보면 다른 scope에 접근(read)은 가능해도 **다른 scope의 식별자에 할당(write)이 불가능**하다. 이를 가능케 하기 위해 `global`, `nonlocal`이라는 키워드를 사용하여 다른 scope의 식별자에 할당을 할 수 있다.

```python
a = 10
def foo1():
  print(a)    # read

def foo2():
  global a
  a = 20    # can write and read both
  print(f"a in foo2 : {a}")
  print(hex(id(a)))

foo1()
foo2()
print(f"a in global: {a}")
print(hex(id(a)))

# output:
# 10
# a in foo2 : 20
# 0x103454350
# a in global: 20
# 0x103454350
```
위의 변형된 코드를 보면 `foo2`에서 `global`을 이용하여 global scope의 `a`에 할당하는 것을 확인할 수 있다.

`nonlocal`의 경우 중첩된 함수의 내부함수에서 외부함수의 객체를 사용할 수 있게 해준다.
```python
def hoo():
  a = 20    #write

  def inner_hoo():
    nonlocal a
    a = 30
    print(f"a in inner_hoo : {a}")
    print(hex(id(a)))
    

  inner_hoo()
  print(f"a in hoo : {a}")
  print(hex(id(a)))

hoo()

# output:
# a in inner_hoo : 30
# 0x1026c0490
# a in hoo : 30
# 0x1026c0490
```

<br>

### Reference
* https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces

* https://medium.com/swlh/mastering-python-namespaces-and-scopes-7eba67aa3094

* https://realpython.com/python-scope-legb-rule/
