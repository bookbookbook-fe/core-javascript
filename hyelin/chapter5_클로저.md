# 클로저

### 1. 클로저의 의미 및 원래 이해

> 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상

### 2. 클로저와 메모리 관리

메모리 누수의 위험으로 클로저 사용을 조심? 지양?<br/>
➡️ '메모리 소모'에 대한 관리법을 잘 파악해서 적용하자! <br/>
➡️ 클로저는 필요에 의해 의도적으로 함수의 지역변수를 메모리를 소모하도록 함으로써 발생! 그렇다면 그 필요성이 사라졌을때는 더는 메모리를 소모하지 않게 해주면 된다.

### 3. 클로저 활용 사례

**3.1 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때**

```jsx
var fruits = ["apple", "banana", "peach"];
var $ul = document.createElement("ul");

var alertFruit = function (fruit) {
  alert("");
};
fruits.forEach(function (fruit) {
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruit);
  $ul.appendChild($li);
});

document.body.appendChild($ul);
alertFruit(fruits[1]);
```

**3.2 접근 권한 제어(정보 은닉)**

- 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈간의 결합도를 낮추고 유연성을 높이고자 하는 개념
  - public, private, protected

**3.3 부분 적용 함수**

- n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 n-m개의 인자를 넘기면 원래 함수의 실행 결과를 얻을 수 있게 하는 함수

```jsx
var add = function () {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};

var addPartial = add.bind(null, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10));
```

**3.4 커링 함수**

- 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 것
- 한번에 하나의 인자만 전달하는 것이 원칙
- 중간 과정상의 함수를 실행한 결과는 그 다음 인자를 받기 위해 대기만 할 뿐, 마지막 인자가 전달되기 전까지는 원본 함수 실행되지 않음

```jsx
var curry = function (func) {
  return function (a) {
    return function (b) {
      return func(a, b);
    };
  };
};

var getMaxWith10 = curry3(Math.max)(10);
```
