# this

## this의 이해

다른 객체지향 언어에서 this는 클래스로 생성한 인스턴스 객체를 의미합니다. 클래스에서만 사용할 수 있기 때문에 혼란의 여지가 적습니다. 하지만 자바스크립트에서의 this는 어디서든 사용할 수 있으며, 상황에 따라 this가 가리키는 대상이 달라질 수 있어 혼란을 줄 수 있습니다.

함수와 객체(메서드)의 구분이 느슨한 자바스크립트에서 this는 실질적으로 이 둘을 구분하는 거의 유일한 기능입니다.

## 상황에 따라 달라지는 this

자바스크립트에서 this는 기본적으로 실행 컨텍스트가 생성될 때 함께 결정됩니다. 실행 컨텍스트는 함수를 호출할 때 생성되므로, 바꿔 말하면 this는 함수를 호출할 때 결정됩니다.

### 1. 전역 공간에서의 this

전역 공간에서의 this는 전역 객체를 가리킵니다. 브라우저 환경에서는 window, Node.js 환경에서는 global 객체를 가리킵니다.

```javascript
// 브라우저 환경
console.log(this === window); // true

var a = 1;
console.log(window.a); // 1
console.log(this.a); // 1
```

전역공간에서 선언한 변수 a는 window 객체의 프로퍼티로 접근할 수 있습니다. 전역공간에서 this.a로 접근해도 window.a와 같은 결과를 얻습니다. 이는 자바스크립트의 모든 변수가 실은 특정 객체의 프로퍼티로서 동작하기 때문입니다.

### 2. 메서드로서 호출할 때 그 메서드 내부에서의 this

함수와 메서드의 차이점은 다음과 같습니다:
- 함수는 객체에 속해 있는 것이 아니라 독립적으로 존재
- 메서드는 객체에 속해 있는 함수를 의미

메서드 내부에서의 this는 그 메서드를 소유하고 있는 객체를 가리킵니다.

```javascript
var obj = {
    a: 1,
    b: function() {
        console.log(this === obj); // true
    }
};

obj.b();
```

### 3. 메서드의 내부함수에서의 this를 우회하는 방법

대표적인 방법은 변수를 활용하는 것입니다.

```javascript
var obj = {
    outer: function() {
        console.log(this); // { outer: [Function: outer] }
        var inner = function() {
            console.log(this); // window
        };
        inner();
        
        var self = this;
        var inner2 = function() {
            console.log(self); // { outer: [Function: outer] }
        };
        inner2();
    }
};
obj.outer();
```

### 4. this를 바인딩하지 않는 함수

ES6에서 도입된 화살표 함수는 함수를 선언할 때 this를 바인딩하지 않습니다. 화살표 함수 내부에서 this를 참조하면 상위 스코프의 this를 그대로 참조합니다.

```javascript
var obj = {
    a: 1,
    b: function() {
        console.log(this === obj); // true
        var inner = () => {
            console.log(this === obj); // true
        };
        inner();
    }
};
obj.b();
```

### 5. 생성자 함수에서의 this

생성자 함수는 객체를 생성하는 함수를 의미합니다. 생성자 함수 내부에서의 this는 생성자 함수가 생성할 인스턴스를 가리킵니다.

```javascript
var Cat = function(name, age) {
    this.bark = '야옹';
    this.name = name;
    this.age = age;
};

var choco = new Cat('초코', 7);
var nabi = new Cat('나비', 5);
console.log(choco, nabi);
// Cat { bark: '야옹', name: '초코', age: 7 } Cat { bark: '야옹', name: '나비', age: 5 }
```

## this를 명시적으로 바인딩하는 방법

### call, apply, bind 메서드

이 메서드들은 모두 this를 명시적으로 바인딩하는 방법입니다. 함수의 내부에서 this를 어떤 객체로 바인딩할지 결정할 수 있습니다.

```javascript
var func = function(a, b, c) {
    console.log(this, a, b, c);
};

func(1, 2, 3); // window 1 2 3
func.call({ x: 1 }, 4, 5, 6); // { x: 1 } 4 5 6
func.apply({ x: 1 }, [4, 5, 6]); // { x: 1 } 4 5 6

var bound = func.bind({ x: 1 });
bound(7, 8, 9); // { x: 1 } 7 8 9
```

### 화살표 함수의 활용

화살표 함수는 상위 스코프의 this를 그대로 활용할 수 있어, 콜백 함수에서 기존 this를 그대로 활용할 때 유용합니다.

```javascript
var obj = {
    a: 1,
    b: function() {
        setTimeout(() => {
            console.log(this === obj); // true
        }, 100);
    }
};
obj.b();
```