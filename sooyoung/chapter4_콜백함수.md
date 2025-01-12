# 콜백함수



## 1. 콜백함수란?

콜백함수는 다른 코드의 인자로 넘겨주는 함수를 말합니다. 이때 중요한 점은 함수를 넘겨줄 때 제어권도 함께 위임한다는 것입니다. 쉽게 말해서, "이 작업이 끝나면 이 함수를 실행해줘"라고 부탁하는 것과 비슷합니다.

### 간단한 예시로 이해하기

실생활의 예시로 설명하자면:
- 알람 설정: "8시가 되면 알람을 울려주세요" (시계에 제어권 위임)
- 택배 배송: "물건이 도착하면 문자를 보내주세요" (택배회사에 알림 제어권 위임)
- 식당 예약: "자리가 나면 연락주세요" (식당에 연락 제어권 위임)

## 2. 제어권 이해하기

자바스크립트에서 제어권이 어떻게 동작하는지 자세히 살펴보겠습니다.

### 2.1 setInterval 예제

```javascript
var count = 0;
var cbFunc = function() {
  console.log(count);
  if (++count > 4) clearInterval(timer);
};
var timer = setInterval(cbFunc, 300);

// 실행 결과
// 0  (0.3초)
// 1  (0.6초)
// 2  (0.9초)
// 3  (1.2초)
// 4  (1.5초)
```

이 예제에서 `setInterval`의 동작 방식을 자세히 살펴보겠습니다:

1. 호출 시점: setInterval이 결정
2. 호출 주기: 300ms마다
3. 실행 횟수: 조건(count > 4)을 만족할 때까지
4. 종료 시점: clearInterval 호출 시

### 2.2 setInterval의 구조

```javascript
var intervalID = scope.setInterval(func, delay[, param1, param2, ...]);
```

여기서 알아야 할 점들:
- scope: Window 객체 또는 Worker의 인스턴스
- func: 주기적으로 실행할 콜백함수
- delay: 실행 간격 (밀리초 단위)
- param1, param2: 선택적 매개변수

## 3. 콜백함수의 매개변수 활용

### 3.1 Array.prototype.map 예제

```javascript
var newArr = [10, 20, 30].map(function(currentValue, index) {
  console.log(currentValue, index);
  return currentValue + 5;
});
console.log(newArr);

// 실행 결과
// 10 0
// 20 1
// 30 2
// [15, 25, 35]
```

map 메서드의 특징:
- 배열의 각 요소를 순회하며 콜백함수 실행
- 콜백함수의 반환값으로 새로운 배열 생성
- 원본 배열은 변경되지 않음

### 3.2 콜백함수의 매개변수 순서

map 메서드의 콜백함수는 다음 매개변수들을 순서대로 받습니다:
1. currentValue: 현재 처리중인 배열의 요소
2. index: 현재 처리중인 요소의 인덱스
3. array: map을 호출한 원본 배열

## 4. this 바인딩 이해하기

콜백함수에서 this는 특별한 주의가 필요한 부분입니다.

### 4.1 기본적인 this 바인딩

```javascript
setTimeout(function() {
  console.log(this); // Window {...}
}, 300);

[1, 2, 3].forEach(function(x) {
  console.log(this); // Window {...}
});
```

### 4.2 이벤트 리스너에서의 this

```javascript
document.body.innerHTML += '<button id="myButton">클릭</button>';
document.getElementById('myButton').addEventListener('click', function(e) {
  console.log(this); // <button id="myButton">클릭</button>
  console.log(e); // MouseEvent {...}
});
```

### 4.3 bind를 활용한 this 제어

```javascript
var obj = {
  name: 'My Object',
  print: function() {
    console.log(this.name);
  }
};

setTimeout(obj.print.bind(obj), 300); // 'My Object'
```

## 5. 콜백 지옥과 해결 방안

### 5.1 콜백 지옥의 예시

```javascript
setTimeout(
  function(name) {
    var coffeeList = name;
    console.log(coffeeList);

    setTimeout(
      function(name) {
        coffeeList += ', ' + name;
        console.log(coffeeList);

        setTimeout(
          function(name) {
            coffeeList += ', ' + name;
            console.log(coffeeList);
          },
          500,
          '카페모카'
        );
      },
      500,
      '아메리카노'
    );
  },
  500,
  '에스프레소'
);
```

### 5.2 Promise를 활용한 해결

```javascript
const getCoffee = (name) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(name);
    }, 500);
  });
};

getCoffee('에스프레소')
  .then(res => {
    return getCoffee('아메리카노').then(res2 => `${res}, ${res2}`);
  })
  .then(res => {
    return getCoffee('카페모카').then(res2 => `${res}, ${res2}`);
  })
  .then(res => {
    console.log(res); // 에스프레소, 아메리카노, 카페모카
  });
```

### 5.3 async/await을 활용한 해결

```javascript
const getCoffee = (name) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(name);
    }, 500);
  });
};

async function orderCoffee() {
  const coffee1 = await getCoffee('에스프레소');
  const coffee2 = await getCoffee('아메리카노');
  const coffee3 = await getCoffee('카페모카');
  return `${coffee1}, ${coffee2}, ${coffee3}`;
}

orderCoffee().then(result => {
  console.log(result); // 에스프레소, 아메리카노, 카페모카
});
```

## 6. 실제 개발에서의 활용 사례

### 6.1 API 호출

```javascript
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

### 6.2 이벤트 처리

```javascript
const handleClick = (event) => {
  console.log('버튼이 클릭되었습니다!');
  console.log('클릭된 좌표:', event.clientX, event.clientY);
};

document.getElementById('myButton').addEventListener('click', handleClick);
```

## 7. 콜백함수 작성 시 주의사항

1. 에러 처리
    - 콜백함수 내부에서 발생하는 에러를 적절히 처리해야 합니다.
    - try-catch 구문을 활용하세요.

2. 메모리 누수 방지
    - 이벤트 리스너 등록 시 필요없어진 시점에 반드시 제거해야 합니다.
    - 클로저 사용 시 메모리 점유에 주의해야 합니다.

3. 비동기 처리
    - 콜백함수가 언제 호출될지 예측하기 어려울 수 있습니다.
    - 적절한 상태 관리가 필요합니다.

