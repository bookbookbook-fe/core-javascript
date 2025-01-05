# 실행 컨텍스트

실행할 코드에 제공할 환경 정보들을 모아놓은 객체

### 실행 컨텍스트란?

스택 : 출입구가 하나! 데이터 a,b,c,d를 순서대로 저장했다면 d,c,b,a 순서로 꺼낼 수 밖에 없음<br/>
큐 : 양쪽이 모두 열려 있음! a,b,c,d를 저장했다면 꺼낼 때도 a,b,c,d 순서로 꺼낼 수 밖에 없음

> ✨ 컨텍스트를 구성하고(**= 함수를 실행**) 이를 콜스택에 쌓아 올렸다가, 가장 위에 있는 컨텍스트와 관련 있는 코드들을 실행하며 전체 코드의 환경과 순서 보장!

한 실행 컨텍스트가 콜 스택 맨 위에 쌓이는 순간, 곧 현재 실행할 코드에 관여하게 되는 시점

- 기존의 컨텍스트는 새로 쌓인 컨텍스트보다 아래에 위치하기 때문
- 실행 컨텍스트가 활성화될 때, 자바스크립트 엔진은 해당 컨텍스트에 관련된 코드들을 실행하는데 필요한 환경 정보들을 수집 -> 실행 컨텍스트 객체에 저장

**VariableEnvironment** : 현재 컨텍스트 내의 식별자들에 대한 정보 + 외부 환경정보. 선언 시점의 LexicalEnvironment의 스냅샷, 변경 사항 반영 x<br/>
**LexicalEnvironment** : 처음에는 VariableEnvironment와 같지만 변경 사항이 실시간으로 반영<br/>
**ThisBinding** : this 식별자가 바라봐야 할 대상 객체

### VariableEnvironment

- VariableEnvironment에 담기는 내용은 LexicalEnvironment와 같지만 최초 실행 시의 스냅샷 유지하는 점이 다름
- 실행 컨텍스트 생성 시, VariableEnvironment에 정보를 먼저 담고, 이를 그대로 복사하여 LexicalEnvironment를 만들고 이후 LexicalEnvironment를 주로 활용

- VariableEnvironment & LexicalEnvironment의 내부는 environmentRecord와 outer-EnvironmentReference로 구성

### LexicalEnvironment

**environmentRecord와 호이스팅**

environmentRecord에는 현재 컨텍스트와 관련된 코드의 식별자 정보들이 저장됨
컨텍스트 내부 전체를 처음부터 끝까지 훑으며 **_순서대로_** 수집

**호이스팅 규칙**

environmentRecord는 현재 실행될 컨텍스트의 대성 코드 내에 어떤 식별자들이 있는지에만 관심 -> 각 식별자에 어떤 값이 할당되는 것에는 관심 x

> ✨ 변수를 호이스팅할 때, 변수명만 끌어올리고 할당 과정은 원래 자리에 남겨둔다. 매개변수도 마찬가지!

**함수 선언문 & 함수 표현식**

```jsx
function a() {} // 함수 선언문. a는 변수명
a();

var b = function () {}; // 익명 함수 표현식. b는 함수명
b();

var c = function d() {}; // 기명 함수 표현식. 변수명은 c, 함수명은 d
c();
d(); // 에러
```

> 함수 선언문은 전체를 호이스팅. 함수 표현식은 변수 선언부만 호이스팅
> => 함수도 하나의 값으로 취급 가능

**스코프, 스코프 체인, outerEnvironmentReference**

스코프 : 식별자에 대한 유효범위 <br/>
스코프 체인 : 식별자의 유효범위를 안에서부터 바깥으로 차례로 검색해나가는 것<br/>
=> 이를 가능하게 하는 것이 LexicalEnvironment의 두번째 수집자료인 outerEnvironmentReference

- **스코프 체인** <br/>
  여러 스코프에서 동일한 식별자를 선언한 경우, 무조건 스코프 체인상에서 가장 먼저 발견된 식별자에만 접근 가능 <br/>
  But, 스코프 체인상에 있는 변수라고 해서 무조건 접근 가능한 것 아님!

  **this**<br/>

- 실행 컨텍스트의 thisBinding에는 this로 지정된 객체가 저장됨 <br/>
- 실행 컨텍스트 활성화 당시, this가 지정되지 않은 경우 전역 객체가 저장됨<br/>
- 그 밖에는 함수 호출 방법에 따라 저장되는 대상이 다름
