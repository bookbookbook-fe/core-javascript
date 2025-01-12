# this

### 1. 상황에 따라 달라지는 this

자바스크립트에서 this는 실행 컨텍스트가 생성될 때 함께 결정됨<br/>
실행 컨텍스트는 함수를 호출할 때 생성<br/>
➡️ <b>this는 함수를 호출할 때 결정됨</b>

**1. 1 전역 공간에서의 this**

- 전역 객체
- 브라우저 환경에서는 window, Node.js 환경에서는 global

- 전역 변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로 할당

```jsx
var a = 1;
console.log(a); // 1
console.log(window.a); // 1
console.log(this.a); // 1
```

➡️ 전역 공간에서는 var로 변수를 선언하는 대신 window의 프로퍼티에 직접 할당하더라도, var로 선언한 것과 똑같이 동작

**But!** '삭제' 명령어는 예외!

```jsx
var a = 1;
delete window.a; //false
```

```jsx
window.b = 3;
delete window.b; // true
```

- 처음부터 전역 객체의 프로퍼티로 할당? 삭제됨
- 전역변수로 선언한 경우? 삭제안됨

➡️ 전역변수를 추가하면, 자바스크립트 엔진이 이를 자동으로 전역객체의 프로퍼티로 할당. 추가적으로 해당 프로퍼티의 configurable 속성을 false로 정의

**1. 2 메서드로서 호출할 때, 그 메서드 내부에서의 this**<br/>

**함수 vs 메서드**

- 함수는 그 자체로 독립적인 기능을 수행
- 메서드는 자신을 호출한 대상 객체에 관한 동작 수행

> ✨ 이 둘의 차이는 독립성!

```jsx
var func = function x(x) {
  console.log(this, x);
};
func(1); // Window { ... } f

var obj = {
  method: func,
};
obj.method(2); // { method: f } 2
```

- 함수 앞에 점(.)이 있으면 메서드로 호출(대괄호 표기법도 마찬가지), 없으면 함수로서 호출

**메서드 내부에서의 this**<br/>

- this에는 호출 주체에 대한 정보가 담긴다. 메서드로 호출하는 경우 호출 주체는 함수명 앞의 객체<br/>
  - ex) obj.methodA();

**1. 3 메서드로서 호출할 때, 그 메서드 내부에서의 this**<br/>

**함수 내부에서의 this**<br/>

- 함수로서 호출할 경우에는 this가 지정되지 않음.<br/>
- 함수의 this는 전역객체를 가리킴

**메서드 내부함수에서의 this**<br/>

- '설계상의 오류'
- 하지만, 내부함수 역시 함수로 호출했는지 / 메서드로 호출했는지만 파악하면 this 값을 맞출 수 있음

**메서드 내부 함수에서의 this를 우회하는 방법**<br/>

- 호출 주체가 없을 때는 자동으로 전역객체를 바인딩하지 않고, 호출 당시 주변 환경의 this를 그대로 상속받아 사용할 수 있으면 좋겠음

  - 변수를 활용!

  ➡️ 상위 스코프의 this를 저장해서 내부 함수에서 활용

**this를 바인딩 하지 않는 함수**<br/>

- this를 바인딩하지 않는 **_화살표함수_** 도입

```jsx
var obj = {
  outer: function () {
    console.log(this); // { outer: f }
    var innerFunc = () => {
      console.log(this); //  { outer: f }
    };
    innerFunc();
  },
};

obj.outer();
```

**1. 4 콜백함수 호출 시, 그 함수 내부에서의 this**<br/>

- 함수 A의 제어권을 다른 함수 B에게 넘겨주는 경우, 함수 A를 콜백 함수라고 한다.
- 이때 함수 A는 함수 B의 내부 로직에 따라 실행, this 역시 함수 B 내부 로직에서 정한 규칙에 따라 값이 결정됨.
- 콜백함수도 함수이기 때문에, this가 전역객체를 참조함, 제어권을 받은 함수에서 콜백 함수에 this가 될 대상을 지정한 경우는 그 대상을 참조!

**1. 4 생성자 함수 내부에서의 this**<br/>
생성자 함수: 어떤 공통 성질을 지니는 객체들을 생성하는 데 사용하는 함수

- 생성자를 **클래스**, 클래스를 통해 만든 객체를 **인스턴스** 라고 한다.
- 자바스크립트는 함수에 생성자로서의 역할을 함께 부여함.
  - new 명령어와 함께 함수를 호출 -> 해당 함수가 생성자로서 동작하게 됨.
  - 어떤 함수가 생성자 함수로서 호출된 경우, 내부에서의 this는 새로 만들 구체적인 인스턴스 자신이 됨.

```jsx
var Cat = function (name, age) {
  this.bark = "야옹";
  this.name = name;
  this.age = age;
};

var choco = new Cat("초코", 7);
var nabi = new Catg("나비", 5);
console.log(choco, nabi);

// Cat { bark: '야옹', name: '초코', age: 7 };
// Cat { bark: '야옹', name: '나비', age: 5 };
```

# 명시적으로 this를 바인딩하는 방법

**2.1 call 메서드**<br/>

```jsx
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

- call 메서드: 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령어
- 함수를 그냥 실행하면 this는 전역객체를 참조, call 메서드를 이용하면 객체를 this로 지정할 수 있음.

```jsx
var func = function (a, b, c) {
  console.log(this, a, b, c);
};

func(1, 2, 3); // Window { ... } 1 2 3
func.call({ x: 1 }, 4, 5, 6); // { x: 1 } 4 5 6
```

**2.2 apply 메서드**<br/>

```jsx
Function.prototype.apply(thisArg[, argsArray])
```

- call 메서드와 기능적으로 동일

```jsx
var func = function (a, b, c) {
  console.log(this, a, b, c);
};

func.apply({ x: 1 }, [4, 5, 6]); // { x: 1 } 4 5 6
```

**2.3 call / apply 메서드의 활용**<br/>

- **유사배열객체에 배열 메서드를 적용**
- **생성자 내부에서 다른 생성자를 호출**
- **여러 인수를 묶어 하나의 배열로 전달하고 싶을때 - apply 활용**

**2.4 bind 메서드**<br/>

```jsx
Function.prototype.bind(thisArg[, arg1[, arg2[, ...]]])
```

- call과 비슷하지만, 즉시 호출하지 않고 넘겨받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드
- 다시 새로운 함수를 호출할 때, 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달했던 인수들의 뒤에 이어서 등록됨.

```jsx
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};

func(1, 2, 3, 4); // Window { ... } 1 2 3 4

var bindFunc1 = func.bind({ x: 1 });
bindFunc1(5, 6, 7, 8); // { x: 1 } 5 6 7 8

var bindFunc2 = func.bind({ x: 1 }, 4, 5);
bindFunc2(6, 7); //  { x: 1 } 4 5 6 7
```

**2.5 화살표 함수의 예외사항**<br/>

- 화살표 함수 내부에는 this가 아예 없으며, 접근하고자 하면 스코프체인상 가장 가까운 this에 접근하게 됨.

**2.6 별도의 인자로 this를 받는 경우(콜백 함수 내에서의 this)**<br/>

- 콜백함수를 인자로 받는 메서드 중 일부는 추가로 this로 지정할 객체를 인자로 지정할 수 있음

  - 이러한 메서드의 thisArg 값을 지정하면 콜백함수 내부에서 this 값을 원하는대로 변경 가능
  - 이런 형태는 내부 요소에 대해 같은 동작을 반복 수행해야 하는 **_배열 메서드_** 에 많이 있음

  > forEach, map, filter, some, every, find ... 등등
