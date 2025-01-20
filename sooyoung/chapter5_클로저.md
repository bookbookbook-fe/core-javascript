# 클로저 (Closure)

## 클로저의 의미 및 원리 이해

클로저는 함수형 프로그래밍에서 자주 등장하는 중요한 개념입니다. 클로저를 이해하기 위해서는 먼저 정의부터 살펴보겠습니다.

클로저란 함수 A 내부에서 선언된 변수 a를 참조하는 내부함수 B가 외부로 전달될 때, 함수 A의 실행이 끝났음에도 변수 a가 사라지지 않고 유지되는 현상을 말합니다.

MDN에서는 "클로저는 함수와 그 함수가 선언될 당시의 Lexical Environment의 상호관계에 의해 만들어지는 현상"이라고 정의하고 있습니다.

### Lexical Environment와 스코프

여기서 'Lexical Environment'는 실행 컨텍스트의 구성 요소 중 하나인 outerEnvironmentReference를 의미합니다. LexicalEnvironment의 구성 요소인 environmentRecord와 outerEnvironmentReference에 의해 스코프(변수의 유효범위)가 결정되고, 이를 통해 스코프 체인이 가능해집니다.

예를 들어, 컨텍스트 A에서 선언한 내부함수 B가 실행될 때는:
- B의 outerEnvironmentReference가 참조하는 A의 LexicalEnvironment에도 접근 가능
- A에서는 B의 변수에 접근할 수 없음
- B에서는 A의 변수에 접근 가능

단, 이는 내부함수 B가 외부 변수를 참조할 때만 해당됩니다. 만약 외부 변수를 참조하지 않는다면, B는 A의 LexicalEnvironment를 사용할 수 없습니다.

### 클로저의 예제

#### 1. 기본적인 클로저의 형태

```javascript
var outer = function () {
    var a = 1;
    var inner = function () {
        return ++a;
    };
    return inner;
};

var outer2 = outer(); // inner 함수 자체를 반환
console.log(outer2()); // 2
console.log(outer2()); // 3
```

이 예제에서 주목할 점:
1. inner 함수를 반환하면서 outer 함수의 실행은 종료됨
2. outer2 변수는 반환된 inner 함수를 참조
3. outer2를 호출하면 inner 함수가 실행되며, 이때 outer의 LexicalEnvironment를 참조해서 변수 a의 값을 증가시킴
4. 결과적으로 outer 함수의 실행은 종료되었지만 변수 a가 살아있는 상태로 값을 유지

## 클로저의 활용

### 1. 정보 은닉과 캡슐화

클로저를 활용하면 외부에서 직접 접근할 수 없는 private한 변수를 만들 수 있습니다.

```javascript
const makeCounter = function () {
    let count = 0; // private 변수
    return {
        increase: function () {
            return ++count;
        },
        decrease: function () {
            return --count;
        },
        getCount: function () {
            return count;
        }
    };
};

const counter = makeCounter();
console.log(counter.increase()); // 1
console.log(counter.decrease()); // 0
console.log(counter.getCount()); // 0
```

### 2. 이벤트 핸들러와 콜백 함수

```javascript
const fruits = ['apple', 'banana', 'peach'];
const $ul = document.createElement('ul');

fruits.forEach(function (fruit) {
    const $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click', function () {
        alert('선택한 과일: ' + fruit);
    });
    $ul.appendChild($li);
});

document.body.appendChild($ul);
```

### 3. 부분 적용 함수 (Partially Applied Function)

```javascript
const partial = function (fn, ...args) {
    return function (...restArgs) {
        return fn.apply(this, [...args, ...restArgs]);
    };
};

const add = function (a, b, c) {
    return a + b + c;
};

const add5 = partial(add, 5);
console.log(add5(10, 20)); // 35
```

## 클로저와 메모리 관리

클로저는 의도적으로 특정 변수를 메모리에 유지시키는 것이므로, 더 이상 필요하지 않은 경우에는 해당 변수를 참조하는 함수를 제거하는 것이 좋습니다.

```javascript
let outer = (function () {
    let a = 1;
    let inner = function () {
        return ++a;
    };
    return inner;
})();

console.log(outer());
console.log(outer());
outer = null; // 메모리 해제
```

클로저는 강력한 기능이지만, 남용하면 메모리 문제가 발생할 수 있으므로 필요한 경우에만 적절히 사용하는 것이 중요합니다.

### 4. 커링 함수 (Currying Function)

커링은 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수들의 체인으로 변환하는 기법입니다. 부분 적용 함수와 비슷해 보이지만, 다음과 같은 차이가 있습니다:

- 부분 적용 함수: 여러 개의 인자를 한 번에 전달 가능
- 커링 함수: 한 번에 하나의 인자만 전달하는 것이 원칙

#### 기본적인 커링 함수 예제

```javascript
const curry = function(func) {
    return function(a) {
        return function(b) {
            return func(a, b);
        }
    }
};

const getMaxWith10 = curry(Math.max)(10);
console.log(getMaxWith10(8));  // 10
console.log(getMaxWith10(25)); // 25
```

#### ES6 화살표 함수를 사용한 커링

```javascript
const curry = func => a => b => c => func(a, b, c);

// 실제 사용 예시
const sum = (a, b, c) => a + b + c;
const curriedSum = curry(sum);

console.log(curriedSum(1)(2)(3)); // 6
```

#### 실무에서의 커링 활용 예시

```javascript
// API 요청 URL 생성을 위한 커링 함수
const createRequestURL = baseURL => path => id => 
    `${baseURL}${path}/${id}`;

// 기본 URL 설정
const createGithubURL = createRequestURL('https://api.github.com');

// API 경로 설정
const getGithubUser = createGithubURL('/users');
const getGithubRepo = createGithubURL('/repos');

// 실제 URL 생성
console.log(getGithubUser('john')); // https://api.github.com/users/john
console.log(getGithubRepo('project')); // https://api.github.com/repos/project
```

커링의 장점:
1. 함수의 재사용성 향상
2. 부분적인 함수 실행 (지연 실행)
3. 함수 조합을 통한 새로운 함수 생성
4. 매개변수의 유연한 처리