# Python

### 0. 들어가기 전에

파이썬을 통해 웹개발을 진행할 예정이며, 프로젝트 완료 전까지 파이썬의 기초 문법을 복습하고 알지 못했던 내용을 적어가고자 한다.

### 1. 자료형

파이썬은 데이터를 담을 수 있는 list, set, dict가 있다. 그 중 dict는 키와 값으로 이루어진 자료형이며, 웹개발에서 데이터 전달에 필수적인 json 문자열을 parsing할 때 쓰인다.

- **dict**는 **key**를 가져올 지 **value**를 가져올 지 아니면 **둘 다** 가져올 지에 따라 알맞는 함수를 쓰면된다.

  ```python
  keys = dictionary.keys();
  values = dictionary.values();
  items = dictionary.items();
  ```

- **set**은 중복값을 허용하지 않는 자료형이며 선언 시 반드시 함수를 사용한다. 중괄호로 선언 시 dict가 생성된다

  ```python
  s3 = set()
  print(type(s3))
  
  s4 = {}
  print(type(s4))
  ```

- 서로 다른 set으로 교집합, 합집합, 차집합을 쉽게 구할 수 있다.

  ```python
  # 교집합
  print(s1 & s2)
  print(s1.intersection(s2))
  # 합집합
  print(s1 | s2)
  print(s1.union(s2))
  # 차집합
  print(s1 - s2)
  print(s1.difference(s2))    # 대입 순서 주의!
  ```

### 2. random 함수와 sample

파이썬 내장함수로 random하게 값을 추출할 때 사용되는 모듈이다.

로또를 자동으로 발권한다고 할 때 손쉽게 사용할 수 있다.

```python
numbers = random.sample(range(1, 46), 6)
```

여기서 sample() 함수를 통해 범위와 추출할 수를 지정하고, 해당 범위 내에서 1번 발생하므로 중복이 발생하지 않는다.

