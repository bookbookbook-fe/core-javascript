# 클로저

- `어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상`
  - 외부로의 전달은 return문에만 해당하는 것이 아니다.
    - 지역변수를 참조하는 내부함수를 외부에 전달하는 것도 포함

```js
var outer = function () {
  var a = 1;
  var inner = function () {
    console.log(++a);
  };
  inner();
};
outer();
```

- outer 함수에서 변수 a 선언, outer 내부함수인 inner 함수에서 a의 값을 1만큼 증가시킨 다음 출력
- inner 함수 내부에서는 a를 선언하지 않았기에, environmentRecord에서 값을 찾지 못하므로 outerEnvironmentReference에 지정된 상위 컨텍스트인 outer의 LexicalEnvironment에 접근하여 다시 a를 찾는다.
- 4번째 줄에서는 2가 출력된다.
- outer 함수의 실행 컨텍스트가 종료되면 LexicalEnvironment에 저장된 식별자들(a, inner)에 대한 참조를 지운다.
  - 각 주소에 저장돼 있던 값들은 자신을 참조하는 변수가 하나도 없게 되므로 가비지 컬렉터의 수집 대상이 된다.

`outer 함수의 실행 컨텍스트가 종료되기 이전, inner 함수의 실행 컨텍스트가 종료되어 있으며, 이후 별도로 inner함수를 호출할 수 없다.`

```js
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner;
};
var outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3
```

- 6번째 줄에서 inner 함수의 실행 결과가 아닌 inner 함수 자체를 반환
  - outer 함수의 실행 컨텍스트가 종료될 때(8번째 줄) outer2 변수는 outer의 실행 결과인 inner를 참조하게 된다.
  - 이후 9번째 줄에서 outer2를 호출하면 앞서 반환된 함수인 inner가 실행된다.
- inner 함수의 실행 컨텍스트의 environmentRecord에는 수집할 정보가 없다.
- outerEnvironmentReference에는 inner 함수가 선언된 위치의 LexicalEnvironment가 참조복사된다.
- inner 함수는 outer 함수 내부에서 선언됐으므로, outer 함수의 LexicalEnvironment가 담길 것이다.
  - `inner 함수가 실행될 시점에는 outer 함수는 이미 종료되었는데 어떻게 outer 함수의 LexicalEnvironment에 접근할 수 있는가?`
    - GC(garbage collector)의 동작 방식 때문

## GC의 동작방식

- GC는 어떤 값을 참조한느 변수가 하나라도 있다면, 그 값은 수집 대상에 포함하지 안흔다.
- 위 코드의 outer 함수는 실행 종료 시점에 inner를 반환한다.
  - 즉, 외부함수인 outer의 실행이 종료되더라도, 내부 함수인 inner 함수는 언젠가 outer2를 실행함으로써 호출될 가능성이 있게 된것이다.
    - `언젠가 inner 함수의 실행 컨텍스트가 활성화되면 outerEnvironmentReference가 outer 함수의 LexicalEnvironment를 필요로 할 것이기에 수집 대상에서 제외된다.`

# 클로저와 메모리 관리

- 클로저는 어떤 필요에 의해 의도적으로 함수의 지역변수를 메모리를 소모하도록 함으로써 발생한다.
  - 즉, 필요성이 사라진 시점에서는 메모리를 소모하지 않게 하면 된다.
    - 참조 카운트를 0으로 만들어 GC가 수거대상으로 인식하게 한다.
      - 식별자에 참조형이 아닌 기본형 데이터(보통 null or undefined)를 할당하여 참조 카운트를 0으로 만든다.

## 메모리 누수

- 개발자의 의도와 달리 `어떤 값의 참조 카운트가 0이 되지 않아 GC의 수거 대상이 되지 않는 상황`

## 클로저의 메모리 관리

1. return에 의한 클로저의 메모리 해제

```js
var outer = (function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner;
})();
console.log(outer());
console.log(outer());
outer = null; // outer 식별자의 inner 함수 참조를 끊는다.
```

2. setInterval에 의한 클로저의 메모리 해제

```js
(function () {
  var a = 0;
  var intervalId = null;
  var inner = function () {
    if (++a >= 10) {
      clearInterval(intervalId); // inner 식별자의 함수 참조를 끊는다.
      inner = null;
    }
    console.log(a);
  };
  intervalId = setInterval(inner, 1000);
})();
```

3. eventListener에 의한 클로저의 메모리 해제

```js
(function () {
  var count = 0;
  var button = document.createElement("button");
  button.innerText = "click";

  var clickHandler = function () {
    console.log(++count, `times clicked`);
    if (count >= 10) {
      button.removeEventListener("click", clickHandler);
      clickHandler = null; // clickHandler 식별자의 함수 참조를 끊는다.
    }
  };
});
```

# 클로저 활용 사례

1. 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

```js
var fruits = ["apple", "banana", "peach"];
var $ul = docuemnt.createElement("ul");

var alertFruitBuilder = function (fruit) {
  return function () {
    alert("your choice is " + fruit);
  };
};
fruits.forEach(function (fruit) {
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruitBuilder(fruit));
  $ul.appendChild($li);
});
```

- 12번째 줄에서 alertFruitBuilder 함수를 실행하며 fruit 값을 인자로 전달한다.
  - 이 함수의 실행 결과가 다시 함수가 되며, 이렇게 반환된 함수를 리스너에 콜백 함수로 전달한다.
  - 이후 언젠가 클릭 이벤트가 발생하면, 이 함수의 실행 컨텍스트가 열리면서 alertFruitBuildr의 인자로 넘어온 fruit를 outerEnvironmentReference에 의해 참조할 수 있다.
    - 즉, alertFruitbuilder의 실행 결과로 반환된 함수애는 클로저가 존재한다.

2. 접근 권한 제어(정보 은닉)

- `어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화하여 모듈간의 결합도를 낮추고 유연성을 높이`고자 하는 방법
- 접근 권한에는 public, private, protected 세 종류가 있다.
  - public => 외부에서 접근 가능
  - private => 내부에서만 접근 가능(외부에서 접근 불가)

`클로저로 변수를 보호한 자동차 객체`

```js
var createCar = function () {
  var fuel = Math.ceil(Math.random() * 10 + 10); // 연료
  var power = Math.ceil(Math.random() * 3 + 2); // 연비
  var moved = 0; // 총 이동거리

  return {
    get moved() {
      return moved;
    },
    run: function () {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / power;
      if (fuel < wasteFuel) {
        console.log("이동불가");
        return;
      }
      fuel -= wasteFuel;
      moved += km;
      console.log(`${km}km 이동 (총 ${moved}km), 남은 연료 : ${fuel}`);
    },
  };
};
var car = crateCar();
```

- fuel, power 변수는 비공개 맴버로 지정하여 외부에서의 접근 제한
- moved 변수는 getter만을 부여함으로써 읽기 전용 속성을 부여
  - 즉, 외부에서는 오직 run 메서드를 실행하는 것과 현재의 moved 값을 확인하는 두 가지 동작만 할 수 있다.

즉, 클로저를 활용하여 접근권한을 제어하는 방법

1. 함수에서 지역변수 및 내부함수 등을 생성한다.
2. 외부에서 접근권한을 주고자 하는 대상들로 구성된 참조형 데이터(대상이 여럿일 때는 객체 또는 배열, 하낭리 때는 함수)를 return 한다.

   - return한 변수들은 공개 멤버가 되고, 그렇지 않은 변수들은 비공개 멤버가 된다.

3. 부분 적용 함수

- n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 하는 함수

```js
Object.defineProperty(window, "_", {
  value: "EMPTY_SPACE",
  writable: false,
  configurable: false,
  enumerable: false,
});

var partial2 = function () {
  var originalPartialArgs = arguments;
  var func = originalPartialArgs[0];
  if (typeof func !== "function") {
    throw new Error("첫 번째 인자가 함수가 아닙니다.");
  }
  return function () {
    var partialArgs = Array.prototype.slice.call(originalpartialArgs, 1);
    var restArgs = Array.prototype.slice.call(originalpartialArgs, 1);
    for (var i = 0; i < partialArgs.length; i++) {
      if (partialArgs[i] === _) {
        partialArgs[i] = restArgs.shift();
      }
    }
    return func.apply(this, partialArgs.concat(restArgs));
  };
};

var add = function () {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};
var addPartial = partial2(add, 1, 2, _, 4, 5, _, _, 8, 9);
console.log(addPartial(3, 6, 7, 10)); // 55

var dog = {
  name: "강아지",
  greet: partial2(function (prefix, suffix) {
    return prefix + this.name + suffix;
  }, "왈왈, "),
};
dog.greet("배고파요!");
```

4. 커링 함수

- 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠 순차적으로 호출될 수 있게 체인 형태로 구성한 것
  - 커링 함수는 한 번에 하나의 인자만 전달하는 것을 원칙으로 한다.
  - 중간 과정상의 함수를 실행한 결과는 그다음 인자를 받기 위해 대기만 할 뿐, 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않는다.
  - 인자가 많아질수록 가독성이 떨어진다는 단점이 있다.

```js
var curry5 = function (func) {
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
var getMax = curry5(Math.max);
console.log(getMax(1)(2)(3)(4)(5));

// Arrow Function을 사용한 경우
var curry5 = (func) => (a) => (b) => (c) => (d) => (e) => func(a, b, c, d, e);
```

- 각 단계에서 받은 인자들은 모두 마지막 단계에서 참조될 것이기에, GC 대상이 되지 않아 메모리에 저장도었다가, 마지막 호출로 실행 컨텍스트가 종료된 후에야 GC의 수거 대상이 된다.

## 커링 함수가 유용한 경우

- 당장 필요한 정보만 받아서 전달하고, 또 필요한 정보가 들어오면 전달하는 식으로 하면 결국 마지막 인자가 넘어갈 때까지 함수 실행을 미루는 셈이 된다.
  - 이러한 동작을 `지연실행(lazy excution)`이라고 한다.
    - 원하는 시점까지 지연시켰다가 실행하는 것이 목적일떄 사용한다.
    - 또한, 프로젝트 내에서 자주 쓰이는 함수의 매개변수가 항상 비슷하고 일부만 바뀌는 경우에도 도입이 가능하다.

```js
var getInformation = function (baseUrl) {
  // 서버에 요청할 주소의 기본 URL
  return function (path) {
    // path 값
    return function (id) {
      // id 값
      return fetch(baseUrl + path + "/" + id); // 실제 서버에 정보를 요청
    };
  };
};

// ES6
var getInformation = (baseUrl) => (path) => (id) =>
  fetch(baseUrl + path + "/" + id);
```

- baseUrl은 몇 개로 고정되지만, 나머지 path나 id 값은 매우 많을 수 있따.
  - 이런 상황에서는 baeUrl부터 전부 기입하기 보단, 공통적인 요소는 먼저 기억시켜 두고 특정한 값(id)만으로 서버 요청을 수행하는 함수를 만들어두는 편이 개발 효율성이나 가독성 측면에서 좋다.
