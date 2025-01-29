# 프로토타입

## 프로토타입의 개념

자바스크립트는 프로토타입 기반 언어입니다. 클래스 기반 언어들이 상속을 통해 객체 지향적 프로그래밍을 구현하는 것과 달리, 자바스크립트는 어떤 객체를 원형(프로토타입)으로 삼고 이를 복제(참조)함으로써 상속과 유사한 효과를 얻습니다.

## 프로토타입의 구조

### Constructor, Prototype, Instance의 관계

생성자 함수(Constructor)를 new 연산자와 함께 호출하면 다음과 같은 과정이 발생합니다:

1. Constructor에 정의된 내용을 바탕으로 새로운 인스턴스가 생성됨
2. 이 인스턴스에는 `__proto__`라는 프로퍼티가 자동으로 부여됨
3. 이 `__proto__` 프로퍼티는 Constructor의 prototype이라는 프로퍼티를 참조함

```javascript
let Person = function(name) {
    this.name = name;
};
Person.prototype.getName = function() {
    return this.name;
};

let suzi = new Person('Suzi');
console.log(Person.prototype === suzi.__proto__); // true
```

## 프로토타입 체인

### 프로토타입 체인의 개념

프로토타입 체인은 어떤 데이터의 `__proto__` 프로퍼티 내부에 다시 `__proto__` 프로퍼티가 연쇄적으로 이어진 것을 의미합니다. 프로토타입 체이닝은 이 체인을 따라가며 검색을 수행하는 것을 말합니다.

```javascript
let arr = [1, 2];
arr.push(3);               // Array.prototype의 메서드 사용
arr.hasOwnProperty(2);     // Object.prototype의 메서드 사용
```

### 메서드 오버라이드

인스턴스가 프로토타입과 동일한 이름의 프로퍼티나 메서드를 가지게 되면 메서드 오버라이드가 발생합니다.

```javascript
let Person = function(name) {
    this.name = name;
};
Person.prototype.getName = function() {
    return this.name;
};

let iu = new Person('지금');
iu.getName = function() {
    return '바로 ' + this.name;
};
console.log(iu.getName()); // '바로 지금'
```

## constructor 프로퍼티

모든 프로토타입은 constructor 프로퍼티를 가지며, 이는 생성자 함수를 가리킵니다:

```javascript
let arr = [1, 2];
console.log(Array.prototype.constructor === Array);      // true
console.log(arr.__proto__.constructor === Array);       // true
console.log(arr.constructor === Array);                 // true
```

## Object.create()

Object.create()를 사용하면 프로토타입 체인 관계를 직접 설정할 수 있습니다:

```javascript
let _proto = Object.create(null);
_proto.getValue = function(key) {
    return this[key];
};

let obj = Object.create(_proto);
obj.a = 1;
console.log(obj.getValue('a')); // 1
```

## 다중 프로토타입 체인

프로토타입 체인은 여러 단계로 이어질 수 있습니다. 이를 활용하면 다른 언어의 클래스 상속과 유사한 구조를 만들 수 있습니다:

```javascript
let Grade = function() {
    let args = Array.prototype.slice.call(arguments);
    for(let i = 0; i < args.length; i++) {
        this[i] = args[i];
    }
    this.length = args.length;
};

Grade.prototype = [];  // Array의 인스턴스를 프로토타입으로 지정
let g = new Grade(100, 80);

g.push(90);
console.log(g);       // Grade(3) [100, 80, 90]
```

## 객체 전용 메서드의 구현

모든 데이터 타입이 프로토타입 체이닝을 통해 Object.prototype의 메서드에 접근할 수 있기 때문에, 객체에서만 사용할 메서드는 Object.prototype이 아닌 Object의 스태틱 메서드로 구현됩니다:

```javascript
// 잘못된 구현 예시
Object.prototype.getEntries = function() {
    let res = [];
    for (let prop in this) {
        if (this.hasOwnProperty(prop)) {
            res.push([prop, this[prop]]);
        }
    }
    return res;
};

// 올바른 구현 예시 (ES6)
Object.entries = obj => {
    return Object.keys(obj).map(key => [key, obj[key]]);
};
```