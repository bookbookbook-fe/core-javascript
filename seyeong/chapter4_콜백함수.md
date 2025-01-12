# 콜백 함수란?

- 콜백함수는 다른 함수에게 파라미터로 전달되는 함수를 의미
- 다른 코드(함수 또는 메서드)에게 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수.

# 1. 제어권

콜백함수의 가장 중요한 특징 중 하나는 실행 시점에 대한 제어권이 함수를 전달받은 코드에게 있다는 점!

## 1-1. 호출 시점

```javascript
var count = 0;
var cbFufnc = function () {
  // 콜백함수
  console.log(count);
  if (++count > 4) clearInterval(timer);
};
var timer = setInterval(cbFunc, 300);

// -- 실행 결과 --
// 0 (0.3초)
// 1 (0.6초)
// 2 (0.9초)
// 3 (1.2초)
// 4 (1.5초)
```

- setInterval에 전달한 첫번째 인자인 cbFunc함수가 바로 콜백함수이고, 0.3초마다 자동으로 실행
- 콜백 함수 내부에서 count 값을 출력하고, count를 1초 증가시키고 그 값이 4보다 크면 반복 실행을 종료
- setInterval이라는 '다른코드'에 첫번째 인자로 cbFunc 함수를 넘겨주자, 제어권을 넘겨받은 setInterval이 스로 판단에 따라 적절한 시점(0.3초)마다 이 익명함수를 실행함.
- 이처럼, 콜백함수의 제어권을 넘겨받은 코드는 콜백 함수 호출 시점에 대한 제어권을 가짐

| code                      | 호출 주체   | 제어권      |
| ------------------------- | ----------- | ----------- |
| cbFunc();                 | 사용자      | 사용자      |
| setInterval(cbFunc, 300); | setInterval | setInterval |

## 1-2. 인자

콜백함수의 또 다른 중요한 특징은 매개변수의 제어권도 함수를 전달받은 코드에게 있다는 점!

**콜백 함수 예제 (map)**

```javascript
var newArr = [10, 20, 30].map(function (currentValue, index) {
  console.log(currentValue, index);
  return currentValue + 5;
});
console.log(newArr);

// -- 실행 결과 --
// 10 0
// 20 1
// 30 2
// [15, 25, 35]
```

위에서 사용한 map 함수는 아래와 같이 콜백 함수로 구성되어 있음.

```javascript
Array.prototype.map(callback[, thisArg]}
callback: function(currentValue, index, array)
```

- map 메서드에 정의된 규칙에는 콜백 함수의 인자로 넘어올 값들 및 그 순서도 포함되어있음.
- 콜백 함수를 호출하는 주체가 사용자가 아닌 Map 메서드 이므로, map 메서드가 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지가 전적으로 map메서드에게 달림
- 이처럼, 콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지에 대한 제어권을 가짐

## 1-3. this

콜백함수도 함수이기 때문에, 기본적으로 this가 전역객체를 참조하지만, 제어권을 넘겨받을 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조

```javascript
setTimeout(function () {
  console.log(this);
}, 300); // 1. Window { ... }

[1, 2, 3, 4, 5].forEach(function (x) {
  console.log(this); // 2. Window { ... }
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector("#a").addEventListener("click", function (e) {
  console.log(this, e); // 3. <button id="a">클릭</button>
}); //  MouseEvent { isTrusted: true, ... }
```

1. setTimeout은 내부에서 콜백 함수를 호출할 때 call 메서드의 첫 번째 인자에 전역 객체를 넘기기 때문에, 콜백 함수 내부에서의 this가 전역 객체를 가리킴
2. forEach는 '별도의 인자로 this를 받는 경우'에 해당된다. 하지만, 별도의 인자로 this를 넘져주지 않았기 때문에 전역 객체를 가리킴
3. addEventListener는 내부에서 콜백 함수를 호출할 때 call 메서드의 첫번째 인자에 addEventListener 메서더의 this를 그래도 넘기도록 정의됨

# 2. 콜백 함수는 함수다

- 콜백 함수는 함수다
- 즉, 콜백 함수로 어떤 객체의 메서드를 전달하더라고 그 메서드는 메서드가 아닌 '함수'로서 호출된다.

```javascript
var obj = {
  vals: [1, 2, 3],
  logValues: function (v, i) {
    console.log(this, v, i);
  },
};
obj.logValues(1, 2); // 1. {vals: [1, 2, 3], logValues: f} 1 2
[4, 5, 6].forEach(obj.logValues); // 2. Window  {...} 4 0
```

1. obj 객체의 logValues는 메서드로 정의됨. 따라서 (1) 에서는 이 메서드의 이름앞에 점이 있으니 메서드로서 호출. 이때, this는 obj를 가리키고, 인자로 넘어온 1, 2가 출력됨
2. 이 메서드를 forEach 함수의 콜백 함수로서 전달함. obj를 this로 하는 메서드를 그대로 전달한게 아니라, obj.logValues가 가리키는 함수만 전달. 즉, forEach에 의해 콜백이 함수로서 호출이 되고, 별도로 this를 지정하지 않았으므로 함수 내부에서의 this는 전역객체를 바라봄

=> 즉, 객체의 메서드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없다.

# 3. 콜백 함수 내부의 this에 다른 값 바인딩하기

위에서 "객체의 메서드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없다." 라고 했는데, 그럼에도 콜백 함수 내부의 this가 객체를 바라보게하고 싶다면?

=> 전통적으로는 this를 다른 변수에 담아, 콜백 함수로 활용할 함수에서는 this 대신 그 변수(OTHER)를 사용하게 하고, 이를 클로저로 만드는 방식을 씀

**콜백 함수 내부의 this에 다른값을 바인딩(전통적인 방식):**

```javascript
var obj1 = {
  name: "obj1",
  func: function () {
    var self = this;
    return function () {
      console.log(self.name);
    };
  },
};
var callback = obj1.func(); // 1.
setTimeout(callback, 1000); // 2. obj1
```

- obj1.func 메서드 내부에서 self 변수에 this를 담고, 익명함수를 반환
- (1) 에서 obj1.func()를 호출하면 앞서 선언한 내부함수가 반환되어 callback 변수에 담김.
- (2)에서 이 callback을 setTimeout함수에 인자로 전달하면 1초 뒤 callback이 실행되면서 'obj1'을 출력
  하지만 위 코드는 너무 번거롭고 비효율적

**콜백 함수 내부에서 this를 사용하지 않을 경우:**

```javascript
var obj1 = {
  name: "obj1",
  func: function () {
    console.log(obj1.name);
  },
};
setTimeout(obj1.func, 1000);
```

**ES5의 bind 메서드를 활용해서 바인딩 할 수도 있음:**

```javascript
var obj1 = {
  name: "obj1",
  func: function () {
    console.log(this.name);
  },
};
setTimeout(obj1.func.bind(obj1), 1000);
```

# 4. 콜백 지옥과 비동기 제어

**콜백 지옥**

- 콜백 함수를 익명 함수 형태로 중첩해서 사용할 때 발생하는 코드 구조의 문제를 의미
- 이는 자바스크립트에서 흔히 발생하는 현상으로, 코드의 들여쓰기가 과도하게 깊어져 가독성과 유지보수성을 떨어뜨림.
- 특히 이벤트 처리나 서버 통신과 같은 비동기 작업을 처리할 때 자주 발생

**비동기(aynchronous)**

- `동기`의 반대말. 현재 실행 중인 코드가 완료되기를 기다리지 않고 다음 코드를 즉시 실행하는 방식
- 종류
  - 타이머 함수 (setTimeout): 특정 시간이 경과한 후 함수 실행
  - 이벤트 핸들러 (addEventListener): 사용자의 행동(클릭 등)에 반응하여 함수 실행
  - AJAX 통신 (XMLHttpRequest): 서버와의 통신 후 응답에 따라 함수 실행

=> 이러한 비동기 작업들은 Promise, async/await 등의 최신 자바스크립트 기능을 활용하여 더 깔끔하게 처리할 수 있음

**콜백 지옥 예시**

```javascript
setTimeout(
  function (name) {
    var coffeeList = name;
    console.log(coffeeList);

    setTimeout(
      function (name) {
        coffeeList += ", " + name;
        console.log(coffeeList);

        setTimeout(
          function (name) {
            coffeeList += ", " + name;
            console.log(coffeeList);
            setTimeout(
              function (name) {
                coffeeList += ", " + name;
                console.log(coffeeList);
              },
              500,
              "에스프레소"
            );
          },
          500,
          "카페라떼"
        );
      },
      500,
      "카페모카"
    );
  },
  500,
  "아메리카노"
);

// 결과:
// 아메리카노
// 아메리카노, 카페모카
// 아메리카노, 카페모카, 카페라떼
// 아메리카노, 카페모카, 카페라떼, 에스프레소
```

위 코드는 0.5초 주기마다 커피목록을 수집하고 출력함. 각 콜백은 커피 이름을 전달해 목록에 커피명을 추가!
위 콜백 함수들의 가족성을 개선하려면 현재 익명함수들을 기명함수로 전환하는 방법도 있음

**콜백 지옥 해결 1 - 기명함수로 전환**

```javascript
var coffeeList = "";

var addAmericano = function (name) {
  coffeeList += name;
  console.log(coffeeList);
  setTimeout(addMocha, 500, "카페모카");
};

var addMocha = function (name) {
  coffeeList += ", " + name;
  console.log(coffeeList);
  setTimeout(addLatte, 500, "카페라떼");
};

var addLatte = function (name) {
  coffeeList += ", " + name;
  console.log(coffeeList);
  setTimeout(addEspresso, 500, "에스프레소");
};

var addEspresso = function (name) {
  coffeeList += ", " + name;
  console.log(coffeeList);
};

setTimeout(addAmericano, 500, "아메리카노");
```

위 방식 코드는 가독성을 높이고, 함수 선언과 호출만 구분할 수 있다면 위에서부터 아래로 순서대로 읽어내려가는 데 어려움이 없으나 최선이라고 할 수는 없음. 일회성 함수를 전부 변수에 할당하는 것이 비효율적임

자바스크립트는 십수 년간 비동기적인 작업을 동기적으로, 혹은 동기적인 것처럼 보이게끔 처리해주는 장치를 마련함.
ES6에서는`Promise`, `Generator` 등이 도입됐고, ES2017에서는 `async/await` 가 도입됨.
