# 데이터 타입의 종류

JS의 데이터 타입은 `기본형(원시형)`과 `참조형`으로 나뉜다.

## 기본형

- 숫자, 문자열, boolean, null, undefined, symbol(ES6)
- 기본형은 할당이나 연산 시 복제된다.
  - 값이 담긴 주솟값을 바로 복제

## 참조형

- 객체, 배열, 함수, 날짜, 정규표현식, ES6(Map, WeakMap, Set, WeakSet) 등
- 참조형은 값을 참조함
  - 값이 담긴 주솟값들로 이뤄진 묶음을 가리키는 주소값을 복제한다.

# 식별자와 변수

## 변수

- 변할 수 있는 수(변할 수 있는 데이터)
  - 변경 가능한 데이터가 담길 수 있는 공간

## 식별자

- 어떤 데이터를 식별하는 데 사용하는 이름(변수명)

# 변수 선언과 데이터 할당

## 변수 선언

```js
var a; // 변수 a 선언
a = "abc"; // 변수 a에 데이터 할당

var a = "abc"; // 변수 선언과 할당을 한 문장으로 표현
```

선언과 할당을 위의 1,2번째 줄과 같이 두 문장으로 나눠 하든 4번째 줄처럼 한 줄로 쓰든 결과는 동일

- a 변수에 직접 문자열 abc를 저장하는게 아님
  1. 데이터(abc)를 저장하기 위한 메모리 공간 확보
  2. 데이터(abc) 저장
  3. 데이터(abc)가 저장된 주소를 변수(a)에 저장
  - 이러한 복잡한 과정을 거치는 이유
    - JS는 숫자형 데이터에 대해 64비트의 공간을 확보한다.(문자열은 정해진 규격 X)
    - 만일, 미리 확보된 공간 내에서만 데이터 변환이 가능하다면, 변환한 데이터를 다시 저장하기 위해서는 확보된 공간을 변환된 데이터 크기에 맞게 늘리는 작업이 선행 되어야함
      - ex) 1,2,3,4 이렇게 줄 서있는데 2와 3 사이에 8이 들어가려면 3과 4를 한칸씩 뒤로 미루고 2뒤에 8배치하는 등 연산이 많아짐
    - 이를 해결하기 위해 변수에 새로운 값을 할당할때는 다른 주소에 데이터를 저장한 뒤, 그 주소를 변수에 할당
      - ex) abc데이터 대신 abcdef를 할당
        1. 데이터(abcdef)를 저장하기 위해 메모리 공간 확보
        2. 데이터(abcdef)를 저장
        3. 기존 데이터(abc)가 저장된 주소를 가지고 있는 변수 a에 신규 데이터(abcdef)가 저장된 주소를 재할당

# 변수와 상수

- 변수와 상수를 구분 짓는 것은 변수 영역의 메모리
  - 데이터 할당이 이뤄진 공간에 다른 데이터를 재할당 할 수 있는지 여부
- 불변성 여부를 구분 짓는 것은 데이터 영역의 메모리
  - 기본형 데이터(숫자, 문자열, boolean, null, undefined, Symbol)은 모두 불변형

```js
var a = "abc";
a = a + "def";

var b = 5;
var c = 5;
b = 7;
```

- 문자열은 불변값이기에 1번줄에서 abc를 저장하고 2번줄에서 abcdef를 할당할 때 abc가 저장된 데이터영역 메모리에 abcdef를 재할당 하는 것이 아닌 별도의 메모리 공간을 확보 후 그곳에 abcdef를 저장
  - 이 과정 중 데이터 영역 메모리에 abcdef가 할당된 곳이 있는지 확인
    - 있다면 기존 저장된 abcdef의 메모리 주소를 변수에 할당
    - 없다면 별도의 메모리 공간 확보 후 그곳에 abcdef를 저장 후 이 주소를 변수에 할당

# 가변값

- 불변형 데이터와 차이는 `객체의 변수(프로퍼티) 영역`이 별도로 존재

```js
var obj1 = {
  a: 1,
  b: "bbb",
};
obj1.a = 2;
```

1. obj1의 a 프로퍼티에 숫자 2를 할당
   1. 데이터 영역에서 숫자 2를 검색
   2. 검색 결과가 없으면, 별도의 데이터 영역에 2를 저장
   3. 2가 저장된 주소를 obj1의 a의 주소에 저장
      - 이 과정 중 obj1이 바라보고 있는 주소는 변하지 않음
        - obj1 안에 있는 a가 저장하고 있는 데이터 영역의 주소만 변경됨

# 깊은 복사와 얕은 복사

## 얕은 복사

- 객체의 최상위 속성(객체의 주소값)만 복사한다.

```js
const original = {
  name: "John",
  address: {
    city: "Seoul",
    country: "Korea",
  },
};
// 1) Object.assign()을 사용한 얕은 복사
const shallowCopy1 = Object.assign({}, original);

// 2) 스프레드 연산자를 사용한 얕은 복사
const shallowCopy2 = { ...original };

// 내부 객체 수정
shallowCopy1.address.city = "Busan";

console.log(original.address.city); // 'Busan' -> 얕은 복사라서 원본도 변경됨
console.log(shallowCopy1.address.city); // 'Busan'
```

## 깊은 복사

- 객체의 모든 중첩 구조를 재귀적으로 새롭게 복사하여 원본과 복사본이 완전히 독립적으로 동작

```js
const original = {
  name: "John",
  address: {
    city: "Seoul",
    country: "Korea",
  },
};

// JSON.stringify + JSON.parse를 사용한 깊은 복사
const deepCopy = JSON.parse(JSON.stringify(original));

// 내부 객체 수정
deepCopy.address.city = "Busan";

console.log(original.address.city); // 'Seoul' -> 깊은 복사라서 원본은 변경되지 않음
console.log(deepCopy.address.city); // 'Busan'
```

- `JSON.parse(JSON.stringify(original))`이 코드는 함수, 정규식, undefined 등 JSON으로 변환할 수 없는 값을 제대로 복사하지 못하고, 순환 참조가 있을 경우 에러가 발생할 수 있음
  - 순환 참조 = 서로가 서로를 참조하는 구조

# undefined와 null

## undefined

1. 사용자가 명시적으로 지정하는 경우
   - 명시적으로 undefined를 지정해야 할 때는 undefined 대신 null 사용 권장
2. JS 엔진이 값이 존재하지 않을 때 자동으로 부여하는 경우
   1. 값을 대입하지 않은 변수 즉, 데이터 영역의 메모리 주소를 지정하지 않은 식별자에 접근
   2. 객체 내부의 존재하지 않는 프로퍼티에 접근
   3. return 문이 없거나 호출되지 않는 함수의 실행

### undefined와 배열

```js
var arr1 = [];
arr1.length = 3;
console.log(arr1); // [empty × 3]

var arr2 = new Array(3); // [empty × 3]
console.log(arr2);

var arr3 = [undefined, undefined, undefined]; // [undefined, undefined, undefined]
console.log(arr3);
```

- 비어있는 요소(empty)와 undefined를 할당한 요소는 출력 결과가 다름
  - 즉, forEach, map, filter, reduce 등 여러 순회 메서드에서 결과가 다름
    - empty는 순회와 관련된 많은 배열 메서드들의 순회 대상에서 제외됨
    - undefined는 일반적인 방식대로 배열의 모든 요소를 순회해서 결과 출력

## null

- undefined와 마찬가지로 명시적으로 값이 없음을 나타냄
- `typeof null이 object를 반환하는 것만 주의하면 됨`
  - JS 버그
  - typeof null을 이요해서 값을 비교해야 할 경우 동등 연산자(==)로 비교가 아닌 일치 연산자(===)를 사용해야 정확한 비교 가능
