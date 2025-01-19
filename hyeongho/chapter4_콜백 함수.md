# 콜백 함수

- 다른 코드의 인자로 넘겨주는 함수
- 제어권과 관련이 있다.
- 다른 코드(함수 또는 메서드)에게 인자로 넘겨줌으로써 그 제어권도 함꼐 위임한 함수이다.

## 콜백 함수의 this

- 콜백 함수도 함수이기에 기본적으로 전역 객체를 참조한다.
- 제어권을 념겨받은 코드에서는 콜백 함수에 별도로 this가 될 대상을 지정하면 그 대상을 참조하게 된다.

# 콜백 함수가 함수인 이유

```js
var obj = {
  vals: [1, 2, 3],
  logValues: function (v, i) {
    console.log(this, v, i);
  },
};
obj.logValues(1, 2);
[4, 5, 6].forEach(obj.logValues);
```

- obj.logValues는 점을 이용하여 호출하였기에 메서드로 호출
  - 즉, this는 obj를 가리키고, 인자로 넘어온 1,2 가 출력된다.
- forEach에서는 함수의 콜백으로 전달됨
  - obj를 this로 하는 메서드를 그대로 전달한 것이 아닌 obj.logValues가 가리키는 함수만 전달한 것이다.
    - 즉, 별도로 this를 지정하지 않았기에 this는 전역객체를 바라보게 된다.

# 콜백 지옥과 비동기 제어

## 콜백 지옥

- 콜백 함수를 익명 함수로 전달하는 과정이 반복되어 코드의 들여쓰기 수준이 깊어지는 현상

### 콜백 지옥을 방지하기 위한 방법

1. 기명함수로 전환

- 변수를 최상단으로 끌어올리기 때문에 외부에 노출되는 문제가 있음
  - 즉시 실행 함수를 이용해서 해결 가능

2. Promise이용

- Promise의 인자로 넘겨주는 콜백 함수는 호출할 때 바로 실행되지만 그 내부에 resolve 또는 reject 함수를 호출하는 구문이 있을 경우 둘 중 하나가 실행되기 전까지 then 또는 catch 구문으로 넘어가지 않는다.
  - 즉, resolve혹은 reject를 호출하는 방법으로 비동기 작업의 동기적 표현이 가능해진다.

```js
var addCoffee = function (name) {
  return function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var newName = prevName ? prevName + ", " + name : name;
        console.log(newName);
        resolve(newName);
      }, 500);
    });
  };
};
addCoffee("에스프레소")()
  .then(addCoffee("아메리카노"))
  .then(addCoffee("카페모카"))
  .then(addCoffee("카페라떼"));
```

3. Generator

```js
var addCoffee = function (prevName, name) {
  setTimeout(function () {
    coffeeMaker.next(prevName ? prevName + ", " + name : name);
  }, 500);
};
var coffeeGenerator = function* () {
  // Generator
  var espresso = yield addCoffee("", "에스프레소");
  console.log(espresso);
  var americano = yield addCoffee(espresso, "아메리카노");
  console.log(americano);
  var mocha = yield addCoffee(americano, "카페모카");
  console.log(mocha);
  var latte = yield addCoffee(mocha, "카페라떼");
  console.log(latte);
};
var coffeeMaker = coffeeGenerator();
coffeeMaker.next();
```

- function뒤에 \*이 붙은게 Generator함수이다.
  - 제네레이터 함수를 실행하면 Iterator가 반환된다.
    - Iterator는 next라는 메서드를 가지고 있다.
      - 이 next 메서드를 호출하면 Generator 함수 내부에서 가장 번저 등장하는 yield에서 함수의 실행을 멈춘다.
      - 즉, 비동기 작업이 완료되는 시점마다 next 메서드를 호출해준다면 Generator 함수 내부의 소수가 위에서 아래로 순차적으로 진행된다.

4. async/await

- 비동기 작업을 수행하고자 하는 함수 앞에 async를 표기하고, 함수 내부에서 실질적인 비동기 작업이 필요한 위치마다 await를 표기한다.
  - await뒤의 내용을 Promise로 자동으로 전환하고, 해당 내용이 resolve된 이후 다음으로 진행한다.
- 즉, Promise then과 유사한 효과를 얻을 수 있다.

```js
var addCoffee = function (name) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(name);
    }, 500);
  });
};
var coffeeMaker = async function () {
  var coffeeList = "";
  var _addCoffee = async function (name) {
    coffeeList += (coffeeList ? "," : "") + (await addCoffee(name));
  };
  await _addCoffee("에스프레소");
  console.log(coffeeList);
  await _addCoffee("아메리카노");
  console.log(coffeeList);
  await _addCoffee("카페모카");
  console.log(coffeeList);
  await _addCoffee("카페라뗴");
  console.log(coffeeList);
};
coffeeMaker();
```
