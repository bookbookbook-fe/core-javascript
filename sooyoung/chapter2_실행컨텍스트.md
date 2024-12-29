# 실행컨텍스트

## 실행 컨텍스트란?

실행 컨텍스트(Execution Context)는 자바스크립트 코드가 실행되는 환경을 추상화한 객체입니다. 이는 자바스크립트의 동적 특성을 가장 잘 보여주는 개념으로, 코드 실행에 필요한 환경 정보들을 모아둔 객체라고 할 수 있습니다.

실행 컨텍스트는 다음과 같은 특징적인 동작을 수행합니다:
- 변수 호이스팅
- 외부 환경 정보 구성
- this 값 설정

### 실행 컨텍스트가 생성되는 경우

실행 컨텍스트는 다음 세 가지 경우에 생성됩니다:
1. 전역 공간
2. 함수 실행
3. eval 함수 실행

## 실행 컨텍스트의 작동 방식

### 콜 스택과 실행 순서

실행 컨텍스트는 콜 스택(Call Stack)에 순차적으로 쌓이며 실행됩니다. 예를 들어 다음 코드를 보겠습니다:

```javascript
var a = 1;
function outer() {
  function inner() {
    console.log(a);
    var a = 3;
  }
  inner();
  console.log(a);
}
outer();
console.log(a);
```

실행 순서는 다음과 같습니다:

1. 전역 컨텍스트 생성 및 콜 스택 추가
2. outer 함수 호출 시 outer 컨텍스트 생성 및 스택 추가
3. inner 함수 호출 시 inner 컨텍스트 생성 및 스택 추가
4. inner 함수 종료 시 inner 컨텍스트 제거
5. outer 함수 종료 시 outer 컨텍스트 제거
6. 전역 컨텍스트 종료 및 제거

## 실행 컨텍스트의 구성 요소

실행 컨텍스트는 다음 세 가지 주요 구성 요소를 가집니다:

### 1. VariableEnvironment

- 현재 컨텍스트 내의 식별자 정보
- 외부 환경 정보
- 선언 시점의 LexicalEnvironment의 스냅샷
- 변경 사항이 반영되지 않음

### 2. LexicalEnvironment

- 초기에는 VariableEnvironment와 동일
- 변경 사항이 실시간으로 반영됨
- environmentRecord와 outerEnvironmentReference로 구성

### 3. ThisBinding

- this 식별자가 참조하는 객체 정보

## 호이스팅 (Hoisting)

호이스팅은 자바스크립트 엔진이 코드를 실행하기 전에 변수와 함수 선언을 메모리에 저장하는 과정을 설명하는 개념입니다.

### 호이스팅의 예시

```javascript
function a(x) {
    console.log(x);
    var x;
    console.log(x);
    var x = 2;
    console.log(x);
}
a(1);
```

이 코드는 내부적으로 다음과 같이 동작합니다:

```javascript
function a(x) {
    var x;  // 호이스팅
    x = 1;  // 매개변수 할당
    console.log(x);  // 1
    console.log(x);  // 1
    x = 2;
    console.log(x);  // 2
}
```

## 함수 선언 방식의 차이

### 함수 선언문
- function 키워드와 함수명이 필수
- 호이스팅 됨
```javascript
function sum(a, b) {
    return a + b;
}
```

### 함수 표현식
- 변수에 함수를 할당
- 변수 호이스팅 규칙을 따름
```javascript
var multiply = function(a, b) {
    return a * b;
};
```

## 스코프와 스코프 체인

### 스코프
- 변수의 유효 범위
- 함수에 의해서만 생성됨
- 내부에서 외부로의 접근은 가능하나, 외부에서 내부로의 접근은 불가능

### 스코프 체인
- outerEnvironmentReference를 통해 구현
- 식별자 검색을 안에서 바깥으로 순차적으로 진행
- 가장 먼저 발견된 식별자에만 접근 가능
- 전역 스코프가 체인의 마지막에 위치

### 변수의 구분
1. 전역 변수
    - 전역 스코프에서 선언된 변수
    - 어디서든 접근 가능

2. 지역 변수
    - 함수 내부에서 선언된 변수
    - 해당 함수 내부에서만 접근 가능