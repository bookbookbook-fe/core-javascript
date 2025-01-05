# 1. 실행 컨텍스트

- 실행할 코드에 제공할 환경 정보들을 모아놓은 객체
  - 즉, 실행 컨텍스트는 실행할 코드에 제공할 환경 정보들을 모아놓은 객체
    - 동일한 환경에 있는 코드들을 실행할 때 필요한 환경 정보들을 모아 컨텍스트를 구성하고, 이를 콜 스택에 쌓아올렸다가, 가장 위에 쌓여있는 컨텍스트와 관련있는 코드들을 실행하는 식으로 순서를 보장한다.
- JS는 실행 컨텍스트가 활성화되는 시점에 변수를 호이스팅, 외부 환경 종보 구성, this 값을 설정하는 등의 동작 수행

```js
// -------------------------- (1)
var a = 1;
function outer() {
  function inner() {
    console.log(a); // undefined;
    var a = 3;
  }
  inner(); // -------------------------- (2)
  console.log(a); // 1
}
outer(); // -------------------------- (3)
console.log(a); // 3
```

1. JS 코드를 실행하는 순간(1) 전역 컨텍스트가 콜 스택에 담긴다.
2. (3)에서 outer 함수를 호출하면 outer에 대한 환경 정보를 수집한 후 outer 실행 컨텍스트를 생성한 후, 콜 스택에 담는다.
   - 콜 스택의 맨 위에 outer 실행 컨텍스트가 놓여 있기에 전역 컨텍스트의 실행을 중단하고 outer 실행 컨텍스트와 관련된 코드, 즉 outer 함수 내부의 코드들을 순차로 실행한다.
3. (2)에서 inner 함수의 실행 컨텍스트가 콜 스태그이 가장 위에 담기면 outer 실행 컨텍스트의 실행을 중단하고 inner 실행 컨텍스트와 관련된 코드 즉, inner 함수 내부의 코드들을 순차로 실행한다.
4. inner 함수 내부에서 a 변수에 3을 할당한다.
   - inner 함수의 실행이 종료가되어 inner 실행 컨텍스트가 콜 스택에서 제거된다.
     - inner 아래에 있던 outer 실행 컨텍스트가 콜 스택의 맨 위에 존재하게 되므로 중단했던 (2)의 다음 줄부터 이어서 실행한다.
5. a를 출력하고 나면 outer 실행 컨텍스트가 콜 스택에서 제거되고 전역 컨텍스트를 이어서 실행한다.
6. a를 출력하고 나면 콜 스택에는 아무것도 남지 않은 상태로 종료된다.

## 즉, 실행 컨텍스트가 콜 스택의 맨 위에 쌓이는 순간이 현재 실행할 코드에 관여하게 되는 시점이다.

### 활성화된 실행 컨텍스트가 수집하는 정보

1. VariableEnvironment

- 현재 컨텍스트 내의 식별자들에 대한 정보 + 외부 환경 정보, 선언 시점의 LexicalEnvironment의 스냅샷으로 변경 사항은 반영되지 않는다.

2. LexicalEnvironment

- 처음에는 VariableEnvironment 와 동일하지만, 변경 사항이 실시간으로 반영된다.

3. ThisBinding

- this 식별자가 바라봐야 할 대상 객체

# 2. VariableEnvironment

- VariableEnvironment와 LexicalEnvironment에 담기는 내용은 같지만 최초 실행 시의 스냅샷을 유지한다는 점이 다르다.
  - 실행 컨텍스트를 생성할 때 VariableEnvironment에 정보를 먼저 담고, 이를 그대로 복사하여 LexicalEnvironment를 만들고 이후에는 LexicalEnvironment를 주로 사용한다.
- VariableEnvironment와 LexicalEnvironment의 내부에는 environmentRecord와 outerEnvironmentReference로 구성되어 있다.
  - 초기화 과정 중에는 완전히 동일하고 코드 진행에 따라 서로 달라지는 부분이 생긴다.

# 3. LexicalEnvironment

- 컨텍스트를 구성하는 환경 정보들을 모아놓은 것

## environmentRecord

- 현재 컨텍스트와 관련된 코드의 식별자 정보(매개변수의 이름, 함수 선언, 변수명 등)들이 저장된다.
  - 컨텍스트 내부 전체를 처음부터 끝까지 훑어나가며 순서대로 수집한다.
    - 이 과정이 호이스팅이다.

호이스팅 발생 전 함수

```js
function a() {
  var x = 1; // 수집 대상 1(매개변수 선언)
  console.log(x); // (1)
  var x; // 수집 대상 2(변수 선언)
  console.log(x); // (2)
  var x = 2; // 수집 대상 3(변수 선언)
  console.log(x); // (3)
}
a(1);
```

호이스팅 발생 후 함수

```js
function a() {
  var x; // 수집 대상 1의 변수 선언 부분
  var x; // 수집 대상 2의 변수 선언 부분
  var x; // 수집 대상 3의 변수 선언 부분

  x = 1; // 수집 대상 1의 할당 부분
  console.log(x); // (1)
  console.log(x); // (2)
  x = 2; // 수집 대상 3의 할당 부분
  console.log(x); // 3
}
a(1);
```

### 함수 선언문과 함수 표현식

- 함수 선언문
  - function의 정의부만 존재하고 별도의 할당 명령이 없다.
- 함수 표현식
  - 정의한 function을 별도의 변수에 할당하는 것을 말한다.
  - 함수명을 정의한 함수 표현식을 기명 함수 표현식, 정의하지 않은 것을 익명 함수 표현식이라고 부른다.

```js
function a() {} // 함수 선언문 함수명 a 가 곧 변수명

var b = function () {}; // 익명 함수 표현식 변수명 b가 곧함수명

var c = function d() {}; // 기명 함수 표현식, 변수명은 c, 함수명은 d
c(); // 실행 OK
d(); // Error
```

호이스팅 전

```js
console.log(sum(1, 2));
console.log(multiply(3, 4));

function sum(a, b) {
  // 함수 선언문
  return a + b;
}

var multiply = function (a, b) {
  // 함수 표현식
  return a * b;
};
```

호이스팅 후

```js
var sum = function sum(a, b) {
  // 함수 선언문은 전체를 호이스팅한다.
  return a + b;
};
var multiplyl; // 함수 표현식은 선언부만 호이스팅한다.

console.log(sum(1, 2));
console.log(multiply(3, 4));

multiply = function (a, b) {
  // 변수의 할당부는 원래 자리에 남겨둔다.
  return a * b;
};
```

## outerEnvironmentReference

- `outerEnvironmentReference는 현재 호출된 함수가 선언될 당시의 LexicalEnvironment를 참조한다.`
  - 선언 이란, 콜 스택 상에서 어떤 실행 컨텍스트가 활성화된 상태
  - Linked list 형태를 띈다.
- 각 outerEnvironmentReference는 자신이 선언된 시점의 LexicalEnvironment만 참조하고 있다.
  - 가장 가까운 요소부터 차례대로만 접근할 수 있고, 다른 순서로 접근하는 것은 불가능하다.
    - 즉, 여러 스코프에서 동일한 식별자를 선언한 경우에는 무조건 `스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근이 가능`

### 스코프 체인

```js
var a = 1;
var outer = function () {
  var inner = function () {
    console.log(a);
    var a = 3;
  };
  inner();
  console.log(a);
};
outer();
console.log(a);
```

1. 전역 컨텍스트 활성화

- `전역 컨텍스트의 environmentRecord에 {a, outer} 식별자를 저장`
- 전역 컨텍스트는 선언 시점이 없기에, 전역 컨텍스트의 outerEnvironmentReference에는 아무것도 담기지 않는다.

2. 전역 스코프에 있는 변수 a에 1을, outer에 함수를 할당
3. outer()함수를 호출

- 전역 컨텍스트는 일시 중단, outer 실행 컨텍스트가 활성화

4. outer 실행 컨텍스트의 environmentRecord에 {inner} 식별자 저장.

- outerEnvironmentReference에는 함수가 선언될 당시의 LexicalEnvironment 즉, 전역 컨텍스트의 LexicalEnvironment를 참조복사한다.

5. outer 스코프에 있는 변수 inner에 함수를 할당
6. inner 함수 호출

- outer 실행 컨텍스트는 일시 중지가 되고, inner 실행 컨텍스트 활성화

7. inner 실행 컨텍스트의 environmentRecord에 {a} 식별자 저장.

- outerEnvironmentReference에는 inner 함수가 선언될 당시의 LexicalEnvironment가 담긴다.
- inner 함수는 outer 함수 내부에서 선언됐으므로 outer 함수의 LexicalEnvironment를 참조 복사한다.

8. 식별자 a에 접근

- 현재 활성화된 inner 컨텍스트의 environmentRecord에서 a를 검색한다.
- a가 있지만 아직 값이 없음(undefined 출력)

9. inner 스코프 안의 a변수에 3할당.
10. inner 함수 실행 종료

- inner 실행 컨텍스트가 콜 스택에서 제거되고, outer 실행 컨텍스트가 다시 활성화

11. 식별자 a에 접근

- 활성화된 실행 컨텍스트의 LexicalEnvironment에 접근
  - 첫 요소의 environmentRecord에 a가 있는지 찾아보고 없다면 outerEnvironemntReference에 있는 environmentRecord로 넘어가는 식으로 검색 지속
    - 예제에서는 두 번째(전역 LexicalEnvironment)에 a가 선언되어 있으니 a에 저장된 1을 반환한다.

12. outer 함수 실행 종료

- outer 실행 컨텍스트가 콜 스택에서 제거되고, 전역 컨텍스트 다시 활성화

13. a 식별자 접근

- 현재 활성화 상태인 전역 컨텍스트의 environmentRecord에서 a를 검색

#### 전체 윤곽

- 전역 컨텍스트 => outer 컨텍스트 => inner 컨텍스트 순으로 규모가 작아지는 반면, 스코프 체인을 타고 접근 가능한 변수의 수는 증가한다.

- 스코프 체인 상에 있는 변수라고 해서 무조건 접근 가능한 것은 아님.
  - a를 전역과 inner에서 선언했기에, inner함수 내부에서 a에 접근하면 무조건 가장 가까운 inner의 a에만 접근 가능
    - 스코프 체인은 가장 가까운 식별자를 찾고 찾으면 검색을 더이상 진행 하지 않음
  - 이러한 과정을 `변수 은닉화`라고 한다.

### 전역 변수와 지역변수

- 전역 변수
  - 전역 스코프에서 선언한 a와 outer
  - `즉, 전역 공간에서 선언한 변수`
- 지역 변수
  - 지역 스코프(outer 함수 내부)에서 선언한 inner와 inner 함수 내부에서 선언한 a
  - `즉, 함수 내부에서 선언한 변수`

## this

- 실행 컨텍스트의 thisBinding에는 this로 지정된 객체가 저장된다.
- 실행 컨텍스트 활성화 당시 this가 지정되지 않는 경우 this에는 전역 객체가 저장된다.
