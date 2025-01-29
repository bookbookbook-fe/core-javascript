# 클래스

## 클래스와 인스턴스의 개념

클래스는 추상적인 개념이며, 인스턴스는 이 클래스의 속성을 지니는 실존하는 개체입니다. 프로그래밍에서 클래스는 특정 개체들의 공통된 속성과 기능을 정의한 추상적인 틀이라고 할 수 있습니다.

### 클래스의 특징

1. 클래스 간에는 상하관계가 존재합니다
    - 상위 클래스(superclass)와 하위 클래스(subclass)로 구분
    - 하위 클래스는 상위 클래스의 속성을 상속받음

2. 프로그래밍에서의 클래스
    - 현실세계: 개체가 먼저 존재하고 이를 분류하기 위해 클래스 도입
    - 프로그래밍: 클래스를 먼저 정의하고 이를 바탕으로 인스턴스 생성

## 자바스크립트의 클래스

자바스크립트에서는 프로토타입을 통해 클래스와 유사한 동작을 구현합니다.

### 클래스의 기본 구조

```javascript
var Rectangle = function(width, height) {
    this.width = width;
    this.height = height;
};

// 프로토타입 메서드
Rectangle.prototype.getArea = function() {
    return this.width * this.height;
};

// 스태틱 메서드
Rectangle.isRectangle = function(instance) {
    return instance instanceof Rectangle && 
           instance.width > 0 && 
           instance.height > 0;
};

var rect1 = new Rectangle(3, 4);
console.log(rect1.getArea()); // 12
console.log(Rectangle.isRectangle(rect1)); // true
```

### 메서드의 종류

1. 스태틱 메서드 (static method)
    - 인스턴스에서 직접 접근 불가
    - 생성자 함수를 통해서만 호출 가능

2. 프로토타입 메서드 (prototype method)
    - 인스턴스에서 직접 호출 가능
    - 모든 인스턴스가 공유하는 메서드

## 클래스 상속

### 기본 구현 방법

```javascript
var Grade = function() {
    var args = Array.prototype.slice.call(arguments);
    for(var i = 0; i < args.length; i++) {
        this[i] = args[i];
    }
    this.length = args.length;
};

var g = new Grade(100, 80);
```

### 상속 구현 방식

1. 인스턴스 생성 후 프로퍼티 제거 방식

```javascript
var extendClass1 = function(SuperClass, SubClass, subMethods) {
    SubClass.prototype = new SuperClass();
    for(var prop in SubClass.prototype) {
        if(SubClass.prototype.hasOwnProperty(prop)) {
            delete SubClass.prototype[prop];
        }
    }
    SubClass.prototype.constructor = SubClass;
    if(subMethods) {
        for(var method in subMethods) {
            SubClass.prototype[method] = subMethods[method];
        }
    }
    Object.freeze(SubClass.prototype);
    return SubClass;
};
```

2. 빈 함수를 활용한 방식

```javascript
var extendClass2 = (function() {
    var Bridge = function() {};
    return function(SuperClass, SubClass, subMethods) {
        Bridge.prototype = SuperClass.prototype;
        SubClass.prototype = new Bridge();
        SubClass.prototype.constructor = SubClass;
        if(subMethods) {
            for(var method in subMethods) {
                SubClass.prototype[method] = subMethods[method];
            }
        }
        Object.freeze(SubClass.prototype);
        return SubClass;
    };
})();
```

3. Object.create를 활용한 방식

```javascript
Square.prototype = Object.create(Rectangle.prototype);
Square.prototype.constructor = Square;
Object.freeze(Square.prototype);
```

## ES6의 클래스

ES6에서는 class 키워드를 통해 클래스를 정의할 수 있습니다.

### ES6 클래스 문법

```javascript
class ES6 {
    constructor(name) {
        this.name = name;
    }
    
    static staticMethod() {
        return this.name + 'staticMethod';
    }
    
    method() {
        return this.name + 'method';
    }
}

const es6Instance = new ES6('es6');
console.log(ES6.staticMethod());    // es6 staticMethod
console.log(es6Instance.method());   // es6 method
```

### ES6의 클래스 상속

```javascript
class Rectangle {
    constructor(width, height) {
        this.width = width;
        this.height = height;
    }
    
    getArea() {
        return this.width * this.height;
    }
}

class Square extends Rectangle {
    constructor(width) {
        super(width, width);
    }
    
    getArea() {
        console.log('size is: ', super.getArea());
    }
}
```