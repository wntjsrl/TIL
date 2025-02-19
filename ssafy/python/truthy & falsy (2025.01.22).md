## 단축 평가, Truthy & Falsy

## 단축 평가

- 논리 연산 결과가 확실할 때 나머지 평가를 건너뛰는 기능
- 단축 평가를 사용하면 우리는 조금 더 파이썬스러운 코드를 작성할 수 있음

```python
# and 연산자
# 첫 번째 조건이 False이면 두 번째 조건은 평가되지 않음
result = False and print("이 문장은 출력되지 않습니다.") # 출력이 이루어 지지 않음음
print(result)                                           # False가 출력됨됨

# 두 번째 조건이 True이면 두 번째 조건은 평가됨
# 함수 입장에서는 평가된다는 건 함수가 실행된다는 것
result = True and print("이 문장은 출력됩니다.")         # 이 문장은 출력
print(result)                                           # None 내장 함수는 반환 값이 없음

# or 연산자
# 첫 번째 조건이 True여서 or 뒤에 조건은 평가가 이루어 지지 않음
result = True or print("이 문장은 출력되지 않습니다.")   # 출력이 이루어 지지 않음
print(result)                                           # True가 출력됨

# 두 번째 조건이 False여서 or 뒤에 두 번째 조건은 평가됨
# 함수 입장에서는 평가된다는 건 함수가 실행된다는 것
result = False or print("이 문장은 출력됩니다.")         # 이 문장은 출력
print(result)                                           # None 내장 함수는 반환 값이 없음
```

---

## Truthy / Falsy

```python
# Truthy / Falsy
프로그래밍 언어에서 ‘Truthy’ / ‘Falsy’ 개념은 컨텍스트 내에서
값들이 어떻게 평가되는지 설명하는 것이 중요

# Truthy
컨텍스트 내에서 True로 평가되는 값

# Falsy
컨텍스트 내에서 False로 평가되는 값

# Falsy로 평가되는 값들
1. 빈 시퀀스 & 컬렉션
 - 빈 리스트   '[]'
 - 빈 튜플     '()'
 - 빈 딕셔너리 '{}'
 - 빈 집합     'set()'
 - 빈 문자열   '""'
 - 빈 range    'range(0)'
 
2. 숫자
 - 정수
 - 부동소수
 - 복소수
 
3. 상수
 - None
 - False

# Truthy로 평가되는 값들
 - Falsy로 명시되지 않은 대부분의 값들이 Truthy
 - 비어있지 않은 시퀀스 컬렉션 등등
 - 0이 아닌 모든 숫자
```

```python
# Truthy로 평가가 되는 것
# 이 코드에서 items 리스트가 비어있지 않기 때문에, Truthy로 평가됨
items = [1, 2, 3]
if items:
	print("리스트에 항목이 있습니다.")
else:
	print("리스트에 항목이 없습니다.")

# Truthy / Falsy 여부 확인하기
# bool() 내장 함수로 할 수 있음
print(bool([]))
print(bool(set()))
print(bool(""))
print(bool(0))

# 빈 컨테이너를 체크한 것!
# (if len(items) > 0:) vs (if items:)로 하는 게 좀 더 파이써닉하지 않나?
```

```python
# Falsy / Truthy와 단축 평가의 실제 활용처

# 리스트의 첫 번쨰 요소가 존재하는지 확인
my_list = [1, 2, 3]
first_item = my_list and my_list[0]
print(first_item) # 1

empty_list = []
first_item = empty_list and empty_list[0] # 기본값 설정
print(first_item) # []

# 문자열이 비었는지 확인
name = "" or "Guest"
print(name) # Guest
```
