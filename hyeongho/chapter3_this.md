# this

- this는 실행 컨텍스트가 생성될 때 함께 결정된다
  - 즉, `함수를 호출할 때 결정된다.`

## 전역 공간에서의 this

- 전역 공간에서의 this 는`전역 객체`를 가리킨다.
  - 브라우저에서는 window, Node.js에서는 global

### 전역 공간에서의 특별한 특징

- 전역변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로도 할당한다.
  - 즉, 변수이면서 개겣의 프로퍼티이다.

```js
var a = 1;
console.log(a); // 1
console.log(window.a); // 1
console.log(this.a); // 1
```

- `자바스크립트의 모든 변수는 특정 객체의 프로퍼티로 동작`하기 때문에 간으하다.
  - `전역변수를 선언하면 JS엔진은 이를 전역객체의 프로퍼티로 할당한다.`
    - 변수 a에 접근하고자 하면 스코프 체인에서 a를 검색하다 가장 마지막에 도달하는 전역 스코프의 LE(Lexical Environment) 즉, 전역객체에서 해당 프로퍼티 a를 발견하여 그 값을 반환한다.

## 메서드로서 호출할 때 메서드 내부에서의 this

```js
var func = function (x) {
  console.log(this, x);
};
func(1); // Window {...} 1

var obj = {
  method: func,
};
obj.method(2); // {method: f} 2
```

- 익명함수는 그대로인데, 이를 변수에 담아 호출한 경우와 obj 객체의 프로퍼티에 할당해서 호출한 경우의 this가 달라진다.
  - 함수와 메서드는 함수 앞에 점(.)의 유무로 구별한다.
    - 점이 없으면 함수, 점이 있으면 메서드(대괄호 표기법에 따른 경우도 포함)
      - 즉, `앞에 객체가 명시되어 있으면 메서드, 아니면 모두 함수이다.`
- 함수

  - 그 자체로 독립적인 기능을 수행한다.

- 메서드
  - 자신을 호출한 대상 객체에 관한 동작을 수행한다.

### 메서드 내부에서의 this

- this에는 호출한 주체에 대한 정보가 담긴다.
  - 즉, 앞에 명시된 객체가 this가 된다.

```js
var obj = {
  methodA: function () {
    console.log(this);
  },
  inner: {
    methodB: function () {
      console.log(this);
    },
  },
};

obj.methodA(); // {methodA: f, inner: {...}} (===obj)
obj["methodA"](); // {methodA: f, inner: {...}} (===obj)

obj.inner.methodB(); // {methodB: f} (=== obj.inner)
obj.inner["methodB"](); // {methodB: f} (=== obj.inner)
obj["inner"].methodB(); // {methodB: f} (=== obj.inner)
obj["inner"]["methodB"](); // {methodB: f} (=== obj.inner)
```

### 함수 내부에서의 this

- 함수로서 호출한 경우 this가 지정되지 않는다.
  - this가 지정되지 않을 경우 this는 전역 객체를 바라본다.
    - 즉, 함수에서의 this는 전역 객체를 가리킨다.

### 메서드 내부함수에서의 this

```js
var obj1 = {
  outer: function () {
    console.log(this);
    var innerFunc = function () {
      console.log(this);
    };
    innerFunc();

    var obj2 = {
      innerMethod: innerFunc,
    };
    obj2.innerMethod();
  },
};
obj1.outer();
```

1. 객체를 생성하는데, 이때 객체 내부에는 outer 프로퍼티가 있으며, 익명함수가 연결된다.
   - 생성한 객체를 변수 obj1에 할당한다.
   - 할당 후 obj1.outer 호출
2. obj1.outer 함수의 실행 컨텍스트 생성
   - 호이스팅, 스코프 체인 정보 수집, this 바인딩 발생
     - obj1 객체가 명시되어 있으므로 메서드로 호출 즉, this에는 obj1이 바인딩된다.
   - obj1 객체 정보 출력
3. 호이스팅된 변수 innerFunc는 outer 스코프 내부에서만 접근할 수 있는 지역변수이다.
   - 이 지역변수에 익명함수를 할당한다.
   - innerFunc 호출
4. innerFunc 함수의 실행 컨텍스트가 생성된다.
   - 호이스팅, 스코프 체인 정보 수집, this 바인딩 진행
     - 함수로서 호출한 것이므로 this가 바인딩되지 않는다.
       - 즉, 스코프 체인상의 최상위 객체인 전역객체가 바인딩된다.
   - 전역객체 정보(window)출력
5. 호이스팅된 변수 obj2 역시 outer 스코프 내에서만 접근할 수 있는 지역변수
   - 객체를 다시 할당한다.
     - 객체에는 innerMethod라는 프로퍼티가 있고, 앞서 정의된 변수 innerFunc와 연결된 익명함수가 연결된다.
   - obj2.innerMethod를 호출
6. obj2.innerMethod 함수의 실행 컨텍스트가 생성된다.
   - innerMethod의 앞에 점이 있으니 메서드로 호출한 것이다.
     - 즉, this에는 객체 obj2가 바인딩된다.
   - 객체 obj2 정보가 출력된다.

- 즉, this 바인딩에서는 `함수를 실행하는 당시의 환경은 중요하지 않고, 함수를 호출하는 구문 앞에 점 또는 대괄호 표기가 있는지 없는지가 중요(메서드인지 함수인지)`하다.

#### 메서드의 내부 함수에서 this를 우회하는 방법

1. 변수 활용

```js
var obj = {
  outer: function () {
    console.log(this); // (1) {outer: f}
    var innerFunc1 = function () {
      console.log(this); // (2) Window {...}
    };
    innerFunc1();

    var self = this;
    var innerFunc2 = function () {
      console.log(self); // (3) {outer: f}
    };
    innerFunc2();
  },
};
obj.outer();
```

- innerFunc1 내부에서 this는 전역객체를 가리킨다.
- outer 스코프에서 self 변수에 this를 저장한 상태에서 호출한 innerFunc2의 경우 객체 obj가 출력된다.
  - 상위 스코프의 this를 저장해서 내부함수에서 활용하는 방법

2. this를 바인딩하지 않는 화살표 함수 사용

- 화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지게 되어, 상위 스코프의 this를 그대로 활용할 수 있다.
  - 이 방법을 사용하면 위에서 설명한 변수를 사용하는 방법이 불필요해진다.

```js
var obj = {
  outer: function () {
    console.log(this); // (1) {outer: f}
    var innerFunc = () => {
      console.log(this); // (2) {outer: f}
    };
    innerFunc();
  },
};
obj.outer();
```

3. call, apply 메서드를 활용 하는 방법

- [명시적으로 this를 바인딩하는 방법](#명시적으로-this를-바인딩하는-방법)

### 콜백 함수 호출 시 그 함수 내부에서의 this

- 콜백 함수
  - 함수 A의 제어권을 다른 함수(또는 메서드) B에게 넘겨주는 경우 함수 A를 콜백 함수라고 한다.
    - 함수 A는 함수 B의 내부 로직에 따라 실행되며, this 역시 함수 B 내부 로직에서 정한 규칙에 따라 값이 결정된다.
      - 콜백 함수도 함수이기에, `기본적으로 this가 전역객체를 참조`한다.
      - `제어권을 받은 함수에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우 그 대상을 참조`하게 된다.

### 생성자 함수 내부에서의 this

- 생성자 = 클래스
  - 인스턴스를 만들기 위한 구체적인 틀
- 인스턴스
  - 클래스를 통해 만든 객체

JS에서는 new 명령어와 함꼐 함수를 호출하면 해당 함수가 생성자로서 동작한다.

- `함수가 생성자 함수로 호출된 경우 내부에서는 this는 곧 새로 만들 구체적인 인스턴스 자신이 된다.`
  - 생성자 함수를 호출하면, 생성자의 prototype 프로퍼티를 참조하는 `__proto__`라는 프로퍼티가 있는 객체(인스턴스)를 만들고, 미리 준비된 공통 속성 및 개성을 해당 객체(this)에 부여한다.

```js
var Cat = function (name, age) {
  this.bark = "야옹";
  this.name = name;
  this.age = age;
};

var choco = new Cat("초코", 7);
var nabi = new Cat("나비", 5);

console.log(choco, nabi);
```

- 결과를 출력하면 각각 Cat 클래스의 인스턴스 객체(choco, nabi)가 출력된다.

## 명시적으로 this를 바인딩하는 방법

1. call 메서드 이용

- 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령이다.
  - 메서드의 첫 번째 인자를 this로 바인딩하고, 이후 인자들을 호출할 함수의 매개변수로 한다.

2. apply 메서드 이용

- call 메서드와 기능적으로 완전히 동일하다.
  - call 메서드는 나머지 인자들을 호출할 함수의 매개변수로 지정한다.
  - apply 메서드는 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다.

3. bind 메서드

- call과 비슷하지만, 즉시 호출하지 않고 넘겨받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드이다.
