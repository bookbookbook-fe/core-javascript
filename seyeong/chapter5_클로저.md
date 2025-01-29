# 1. 클로저의 의미 및 원리 이해

- 클로저는 여러 함수형 프로그래밍 언어에서 등장하는 보편적인 특성
- 클로저란 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상
  - 그리고 "외부로 전달"이란 꼭 return만을 의미하는 것은 아니다. return이 없더라도 setInterval이나 eventListener 같은 경우에도 외부로 함수를 전달하므로 클로저 현상이 나타날 수 있다.
- 어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상 = 외부 함수의 LexicalEnvironment가 **가비지 컬렉팅되지 않는 현상**

- 즉, 어떤 컨텍스트 A에서 선언한 내부 함수 B의 실행 컨텍스트가 활성화된 시점에는 B의 outerEnvironmentReference가 참조하는 대상인 A의 LexicalEnvironment에도 접근 가능
  - A에서는 B에서 선언한 변수에 접근 불가
  - 반대로 B에서는 A에서 선언한 변수 접근 가능

단, 내부함수 B에서 외부 변수 A를 참조하는 경우에만 해당됨. 외부 변수를 참조하지 않으면 B가 A의 LexicalEnvironment를 사용할 수 없음!

**클로저의 다양한 정의**

> - 자신을 내포하는 함수의 컨텍스트에 접근할 수 있는 함수
> - 함수가 특정 스코프에 접근할 수 있도록 의도적으로 그 스코프에서 정의하는 것
> - 함수를 선언할 때 만들어지는 유효범위가 사라진 후에도 호출할 수 있는 함수
> - 이미 생명 주기상 끝난 외부 함수의 변수를 참조하는 함수
> - 자유변수가 있는 함수와 자유변수를 알 수 있는 환경의 결합
> - 로컬 변수를 참조하고 있는 함수 내의 함수
> - 자신이 생성될 때의 스코프에서 알 수 있었던 변수들 중 언젠가 자신이 실행될 때 사용할 변수들만을 기억하여 유지시키는 함수

**외부 함수의 변수를 참조하는 내부 함수 (1)**

```javascript
var outer = function () {
  var a = 1;
  var inner = function () {
    // inner 함수에는 a 변수가 없으므로, outerEnvironmentReference에서 outer 변수의 a 참조
    console.log(++a); // 2
  };
  inner();
};

outer();

// 일반적인 함수 및 내부 함수에서의 동작과 차이 없음
// outer의 LexicalEnvironment에 속하는 변수가 모두 가비지 컬렉팅 대상에 포함됨.
```

**외부 함수의 변수를 참조하는 내부 함수 (2)**

```javascript
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a; // 2
  };
  return inner(); // inner 함수 실행 결과를 리턴함.
  // 즉, outer 함수의 실행 컨텍스트가 종료된 시점에는 a 변수를 참조하는 대상이 없어짐
  // a, inner 변수 값들은 언젠간 가비지 컬렉터에의해 소멸
};

var outer2 = outer();
connsole.log(outer2); // 2

// 일반적인 함수 및 내부 함수에서의 동작과 차이 없음
// outer의 LexicalEnvironment에 속하는 변수가 모두 가비지 컬렉팅 대상에 포함됨.
```

**외부 함수의 변수를 참조하는 내부 함수 (3)**

```javascript
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a; // 2
  };
  return inner; // (1) inner 함수 실행 결과가 아닌, 'inner 함수 자체'를 리턴함.
};

var outer2 = outer(); // (2)
connsole.log(outer2); // (3) 2
connsole.log(outer2); // (4) 3

// 위 두 예시와는 달리, 변수 a가 가비지 컬렉팅 대상에서 제외됨.
```

**=> inner 함수 실행시점에는 outer 함수 실행이 종료되었는데, 어떻게 outer 함수의 LexicalEnvironment에 접근하는가?**

- 이는 가비지 컬렉트 동작 방식 때문
- 어떤 값을 참조하는 변수가 하나라도 있다면, 그 값은 수집 대상에 포함시키지 않는다.
- 위 예시에서, outer 함수는 실행 종료 시점에 inner 함수를 반환한다. 외부 함수인 outer의 실행이 종료 되더라도, 내부 함수인 inner함수의 outerEnvironmentReference 언젠가 outer2를 실행함으로써 호출될 가능성 있음.
- 따라서, outer 함수의 lexicalEnvironment는 수집대상에서 제외.

> 클로저란, 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우, A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상

# 2. 클로저와 메모리 관리

**메모리 누수란?**

- 개발자의 의도와 달리 어떤 값의 참조 카운트가 0이 되지 않아 가비지 컬렉팅의 수거 대상이 되지 않는 경우

클로저를 이용하면 메모리 누구가 된다고 클로저 사용을 지양해야한다고 주장하는 사람들이 있다.
하지만 메모리 소모는 클로저의 본질적인 특성일 뿐, 개발자가 클로저를 사용해 의도적으로 참조 카운트를 0이 되지 않게 설계한 경우는 ‘누수'라고 볼 수 없다.

**클로저의 메모리 관리 방법**
클로저는 어떤 필요에 의해 의도적으로 함수의 지역변수를 메모리를 소모하도록 함으로써 발생하는데, 그 필요성이 사라진 지점에는 더는 메모리를 소모하지 않게 해주면 된다. 즉, 참조 카운트를 0으로 만들어 주면 됨!!
=> 즉, **식별자에 참조형이 아닌 기본형 데이터 (null혹은 undefined)를 할당**하여 참조 카운트를 0으로 만들어 메모리를 소모하지 않게 해준다.

# 3. 클로저 활용 사례

## 3-1. 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

### 3-1-1. 콜백함수를 내부함수로 선언, 외부 변수 직접 참조

```javascript
const fruits = ["apple", "banana", "peach"];
const $ul = document.createElement("ul"); // (공통 코드)

fruits.forEach(function (fruit) {
  // (A) forEach 콜백
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", function () {
    // (B) 클릭 이벤트 핸들러
    alert("your choice is " + fruit); // fruit 외부 변수 참조
  });

  $ul.appendChild($li);
});

document.body.appendChild($ul);
```

- (A): fruits의 개수만큼 실행, 그때마다 새로운 실행 컨텍스트 활성화
- (B): (A)의 실행 종료 여부와 무관하게 클릭 이벤트에 의해 실행됨. outerEnvironmentReference가 (A)의 LexicalEnvironment를 참조. 따라서 (B)함수가 참조할 예정인 변수 fruit에 대해서는 (A)가 종료된 후에도 가비지 컬렉터대상에서 제외되고, 계속 참조 가능

### 3-1-2. bind 활용 - 인자 전달

- (B) 함수가 다른 곳에서도 쓰일 경우 반복을 줄이기 위해 (B) 함수를 외부로 분리

```javascript
var alertFruit = function (fruit) {
  // (1)
  alert("your choice is " + fruit);
};

fruits.forEach(function (fruit) {
  const $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruit.bind(null, fruit)); // (2)
  $ul.appendChlid($li);
});

document.body.appendChild($ul);
```

- (1) 공통 함수로 쓰고자 콜백 함수를 외부로 꺼냄
- (2) addEventListener는 콜백 함수를 호출할 때 첫 번째 인자에 '이벤트 객체'를 주입하기 때문에 bind메서드로 인자를 전달
  - 다만 이렇게 하면 이벤트 객체가 인자로 넘어오는 순서가 바뀌고, 함수 내부에서의 this가 원래와 달라지게 된다. 이런 변경사항이 발생하지 않게 하려면 bind 메서드가 아닌 ‘다른 방식’으로 풀어야한다.
  - ‘다른 방식’이란 고차함수를 활용하는 것. 함수형 프로그래밍에서 자주 쓰임

### 3-1-3. 고차함수 활용 - 클로저 적극 활용

```javascript
const alertFruitBuilder = function (fruit) {
  // 외부 함수
  return function () {
    // 내부 함수 (1)
    alert("your choice is " + fruit); // 외부 변수 참조
  };
};

fruits.forEach(function (fruit) {
  const $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruitBuilder(fruit));
  2;
  $ul.appendChild($li);
});
```

- (1) alertFruitBuilder라는 함수 내부에서는 다시 익명함수를 반환하는데, 이 익명함수가 바로 기존의 alertFruit함수다.
- (2) 이벤트 핸들러에서 alertFruitBuilder 함수를 실행하면서 fruit 값을 인자로 전달한다.
  이 함수의 실행 결과가 다시 함수가 되며, 이렇게 반환된 함수를 리스너에 콜백 함수로써 전달함.
- 이후에 클릭 이벤트가 발생하면 이 함수의 실행 컨텍스트가 열리면서 alertFruitBuilder의 인자로 넘어온 fruit를 outerEnvironmentReference에 의해 참조할 수 있게 됨. 즉, alertFruitBuilder의 실행 결과로 반환된 함수에는 클로저가 존재

## 3-2. 접근 권한 제어(정보 은닉)

**정보 은닉(information hiding)이란?**
어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈간의 결합도를 낮추고 유연성을 높이고자 하는 현대 프로그래밍 언어의 중요한 개념

접근 권한에는 public, private, protected의 세 종류

- public : 외부에서 접근 가능
- private : 내부에서만 사용, 외부에 노출 안되는 것

클로저를 이용하면 함수 차원에서 public한 값과 private한 값을 구분하는 것이 가능

```javascript
const outer = function () {
  let a = 1;
  const inner = function () {
    return ++a;
  };
  return inner; // (1)
};

const outer2 = outer();
console.log(outer2());
console.log(outer2());
```

- (1) outer 함수를 종료할 때, inner 함수를 반환함으로써 outer 함수의 지역 변수인 a의 값을 외부에서도 읽을 수 있음
- 즉, return을 활용해 (클로저) 외부 스코프에서 함수 내부의 변수들 중 선택적으로 일부의 변수에 대한 접근 권한을 부여 가능.

**[closure: 닫혀있음, 폐쇄성, 완결성]**

- outer 함수는 외부(전역 스코프)로부터 철저하게 격리된 닫힌 공간
- 외부에서는 outer 함수를 실행할 수 있지만, outer 함수 내부에는 어떠한 개입도 할 수 없음.
- 외부에서는 오직 outer 함수가 return한 정보에만 접근할 수 있음. 즉, return 값이 외부에 정보를 제공하는 유일한 수단
- 따라서, 외부에 제공하고자 하는 정보들을 모아서 return하고, 내부에서만 사용할 정보들은 return 하지 않는 것으로 접근 권한 제어가 가능.
- return한 변수들은 public member가 되고, 그렇지 않으면 private member가 되는 것.

**클로저를 활용해 접근권한을 제어하는 방법**

1. 함수에서 지역변수 및 내부함수 등을 생성
2. 외부에 접근권한을 주고자 하는 대상들로 구성된 참조형 데이터(대상이 여럿일 때는 객체나 배열, 하나일 때는 함수)를 return
3. return 한 변수들은 공개 멤버가 되고, 그렇지 않은 변수들은 비공개 멤버가 된다.

## 3-3. 부분 적용 함수

**부분 적용 함수(partially applied function)란?**
n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있도록 하는 함수

this를 바인딩해야 하는 점을 제외하면 bind메서드의 실행 결과가 바로 부분 적용 함수!

**실무에서의 예시, 디바운스**

- 짧은 시간 동안 동일한 이벤트가 많이 발생할 경우 이를 전부 처리하지 않고 처음 또는 마지막에 발생한 이벤트에 대해 한 번만 처리하는 것으로, 프론트엔드 성능 최적화에 큰 도움을 주는 기능 중 하나

```javascript
const debounce = function (eventName, func, wait) {
  let timeoutId = null;
  return function (event) {
    const self = this;
    console.log(eventName, "event 발생");
    clearTimeout(timeoutId); // (2)
    timeoutId = setTimeout(func.bind(self, event), wait); // (1)
  };
};

const moveHandler = function (e) {
  console.log("move event 처리");
  console.dir(e); // MouseEvent 객체
};

const wheelHandler = function (e) {
  console.log("wheel event 처리");
};

//이벤트가 발생하면 debounce 함수가 실행되고, 내부함수를 리턴, 반환된 내부함수가 이벤트 핸들러가 된다.
document.body.addEventListener("mouse", debounce("move", moveHandler, 500));
document.body.addEventListener("mouse", debounce("move", moveHandler, 700));
```

- (1) 최초 이벤트가 발생하면 (1)코드에 의해 timeout의 대기열에 'wait 시간 뒤에 func를 실행할 것'이라는 내용이 담긴다.
- 그런데 wait 시간이 경과하기 이전에 다시 동일한 event가 발생하면 이번에는 (2)에 의해 앞서 저장했던 대기열을 초기화하고, 다시 (1)번째 줄에서 새로운 대기열을 등록한다. 결국 각 이벤트가 바로 이전 이벤트로부터 wait 시간 이내에 발생하는 한 마지막에 발생한 이벤트만이 초기화되지 않고 무사히 실행될 것이다.
- 참고로, 해당 예제의 디바운스 함수에서 클로저로 처리죄는 변수에는 eventName, func, wait, timeoutId가 있다.
- 결과: 실제 이벤트 발생(wait시간 내)은 여러번이지만 이벤트 발생에 따른 콜백함수 실행은 한 번씩 된다.

## 3-4. 커링 함수

**커링 함수란?**
여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출 될 수 있게 체인 형태로 구성한 것

**부분 적용 함수와의 차이**

- 부분적용함수: 여러 개의 인자를 전달 가능, 실행결과 재 실행시 원본 함수 무조건 실행
- 커링함수: 한 번에 하나의 인자만 전달하는 것이 원칙. 중간 과정상의 함수를 실행한 결과는 그 다음 인자를 받기 위해 대기. 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않음

**커링 함수 예시 1**

```javascript
const curry5 = function (func) {
  return function (a) {
    return function (b) {
      return function (c) {
        return function (d) {
          return function (e) {
            return func(a, b, c, d, e);
          };
        };
      };
    };
  };
};

const getMax = curry5(Math.max);
console.log(getMax(1)(2)(3)(4)(5)); // 11
```

- 커링 함수는 인자가 많아질수록 가독성이 떨어진다는 단점이 있어서, 아래와 같이 ES6의 화살표 함수를 사용하면 한줄에 보기 쉽게 표기 가능

```javascript
// ES6 화살표함수
const curry5 = (func) => (a) => (b) => (c) => (d) => (e) => func(a, b, c, d, e);
```

화살표 함수로 커링을 작성하면 값이 순차적으로 전달되어 최종적으로 func이 호출되는 흐름을 한눈에 파악할 수 있어서 이해하기가 훨씬 쉽다!

그리고 커링은 모든 인자가 준비될 때까지 실행을 미룰 수 있는데, 이를 함수형 프로그래밍에서는 지연실행(lazy execution)이라고 한다. 실행 시점을 제어해야 하거나, 비슷한 인자를 받는 함수에서 일부 값만 자주 바뀌는 경우에 커링이 특히 유용하다!

**커링 함수 예시 2**

```javascript
// ES6 (서버에 요청할 주소의 기본 url, path값, id 값, 실제 서버에 정보 요청(fetch) )
const getInformation = (baseUrl) => (path) => (id) =>
  fetch(baseUrl + path + "/" + id);

//url 전달
const ImgUrl = "http://imageAddress.com/";
const getImg = getInformation(ImgUrl);

//path전달
const getEmoticon = getImg("emoticon");
const getIcon = getImg("icon");

//실제요청 (마지막 id만전달) -> 원본함수실행
const emoticon1 = getEmoticon(100);
const emoticon2 = getEmoticon(102);
const icon1 = getIcon(205);
const icon2 = getIcon(234);
const icon3 = getIcon(265);
```

fetch로 API를 호출할 때, baseUrl은 고정되어 있고 path나 id만 다양하게 변하는 경우가 많다. 이런 상황에서 커링 함수를 활용하면, baseUrl을 미리 저장해두고 변화되는 값만 매개변수로 받아 처리할 수 있다. 이렇게 하면 코드 재사용성이 높아지고 가독성도 개선됨!

**커링 함수 예시 3: Redux의 미들웨어의 커링 함수**

```javascript
// Redux Middleware 'Logger'
const loger = store => next => action => {
  console.log('dispatching', action);
  console.log('next state', store.getState());
  return next(action);
}

// Redux Middleware 'thunk'
const thunk = store => next => action => {
  return typeof action === 'function'
  ? action(dispatch, store.getState)
  : next(action);
```

- 위 두 미들웨어는 공통적으로 store, next, action 순서대로 인자를 받는다.
- 이 중 store는 프로젝트 내에서 한 번 생성된 이후로는 바뀌지 않는 속성이고, dispatch의 의미를 가지는 next 역시 마찬가지지만, action의 경우는 매번 달라진다.
- 그러니까 store, next 값이 결정되면 logger, thunk에 store, next를 미리 넘겨서 반환된 함수를 저장시켜놓고, 이후에 action만 받아서 처리할 수 있게끔 한 것이다.
