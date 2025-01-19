# 콜백 함수

### 1. 콜백 함수란?

콜백함수는 다른 코드의 인자로 넘겨주는 함수<br/>
➡️ <b>🔥제어권🔥</b>도 함께 위임한 함수<br/>
➡️ 콜백함수를 위임받은 코드는 이 콜백함수를 적절한 시점에 실행할 것

### 2. 제어권

**2. 1 호출 시점**

```jsx
var intervalID = scope.setInterval(func, delay[, params1, params2, ...]);
```

- scope에는 window객체 또는 Worker의 인스턴스가 들어올 수 있다.
- 매개변수로 func, delay 값은 반드시 전달해야한다.
- 세번째 매개변수부터는 선택적
- func에 넘겨준 함수는 매 delay(ms)마다 실행되며, 그 결과 어떠한 값도 리턴하지 않는다.
- setInterval을 실행하면 반복적으로 실행되는 내용 자체를 특정할 수 있는 고유한 ID값이 반환된다.
  - 반복 실행되는 중간에 종료(clearInterval)할 수 있게 하기 위해

**2. 2 인자**

```jsx
var newArr = [10, 20, 30].map(function (currentValue, index) {
  console.log(newValue, index);
  return currentValue * 2;
});
```

```jsx
Array.prototype.map(callback[, thisArg])
callback: function(currentValue, index, array)
```

- map 메서드는 첫 번째 인자로 callback 함수를 받고, 생략 가능한 두 번째 인자로 콜백함수 내부에서 this로 인식한 대상을 특정할 수 있다.
  - thisArg를 생략할 경우, 전역객체 바인딩

➡️ 콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수를 호출할 때, 인자에 <b>어떤 값들을 어떤 순서로</b> 넘길 것인지에 대한 제어권을 가진다.

**2. 3 this**

> 콜백 함수도 함수이기 때문에 기본적으로는 this가 전역객체를 참조하지만,<br/> 제어권을 넘겨받을 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조하게 된다.

```jsx
setTimeout(function () {
  console.log(this);
}, 300); // Window { ... }

[1, 2, 3, 4, 5].forEach(function (x) {
  console.log(this); // Window { ... }
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector("#a").addEventListener("click", function (e) {
  console.log(this, e); // <button id="a">클릭</button>
  // MouseEvent { isTrusted : true, ...}
});
```

### 3. 콜백 함수는 함수다

```jsx
var obj = {
  vals: [1, 2, 3],
  logValues: function (v, i) {
    console.log(this, v, i);
  },
};

obj.logValues(1, 2); // {vals : [1,2,3], logValues: f} 1 2
[4, 5, 6].forEach(obj.logValues); // Window { ... } 4 0
// Window { ... } 5 1
// Window { ... } 6 2
```

- 메서드로 호출한 경우, this는 obj를 가리킨다.
- 메서드를 forEach 함수의 콜백 함수로 전달한 경우(obj.logValues가 가리키는 함수만 전달),<br/> forEach에 의해 콜백함수가 함수로서 호출되고 별도로 this를 지정하는 인자가 없으므로 함수내부에서 this는 전역객체

### 4. 콜백 함수 내부의 this에 다른 값 바인딩하기

```jsx
var obj2 = {
  name: "obj1",
  func: function () {
    console.log(this.name);
  },
};
setTimeout(obj1.func.bind(obj1), 100);

var obj2 = { name: "obj2" };
setTimeout(obj1.func.bind(obj2), 1500);
```

### 5. 콜백 지옥과 비동기 제어권

- 콜백 지옥은 콜백함수를 익명 함수로 전달하는 과정이 반복되어 코드의 들여쓰기가 깊어지는 현상

  - 주로 이벤트 처리나 서버 통신과 같이 비동기 작업을 수행하기 위해 자주 발생<br/>

  ➡️ 가독성이 떨어지고, 코드 수정도 어렵다.

- 비동기적인 코드는 현재 실행 중인 코드의 완료 여부와 무관하게 즉시 다음 코드로 넘어간다.<br/>
  ➡️ <b>별도의 요청, 실행 대기, 보류</b> 등

> Promise, Generator, Async/await . .
