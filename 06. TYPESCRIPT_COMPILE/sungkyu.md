# 자바스크립트의 런타임과 타입스크립트의 컴파일

## 타입스크립트의 컴파일

타입스크립트는 tsc라고 불리는 컴파일러를 통해 자바스크립트 코드로 변환되는데 사실 고수준 언어(타입스크립트)가 또 다른 고수준 언어(자바스크립트)로 변환되는 것이기 때문에 컴파일이 아닌 트랜스파일이라고 부르기도 함.

또한 이러한 변환 과정은 소스코드를 다른 소스코드로 변환하는 것이기에 타입스크립트 컴파일러를 소스 대 소스 컴파일러라고 지칭하기도 함.

타입스크립트 컴파일러는 소스코드를 해석하여 AST(최소 구문 트리)를 만들고 이후 타입확인을 거친 다음에 결과 코드를 생성함.

**프로그램이 실행되기까지의 과정**

1. 타입스크립트 소스코드를 타입스크립트 AST로 만든다. (tsc)
2. 타입 검사기가 AST를 확인하여 타입을 확인한다. (tsc)
3. 타입스크립트 AST를 자바스크립트 소스로 변환한다.(tsc)
4. 자바스크립트 소스코드를 자바스크립트 AST로 만든다. (런타임)
5. AST가 바이트 코드로 변환된다. (런타임)
6. 런타임에서 바이트 코드가 평가되어 프로그램이 실행된다. (런타임)

> AST(Abstract Syntax Tree) 컴파일러가 소스코드를 해석하는 과정에서 생성된 데이터 구조다. 컴파일러는 어휘적 분석과 구문 분석을 통해 소스코드를 노드 단위의 트리 구조로 구성.

이때 타입스크립트 소스코드의 타입은 1~2단계에서만 사용되고 3단계부터는 타입을 확인하지 않음. 즉, 개발자가 작성한 타입 정보는 단지 타입을 확인하는 데만 쓰이며, 최종적으로 만들어지는 프로그램에는 아무런 영향을 주지 않음.

타입스크립트는 컴파일타임에 타입을 검사하기 때문에 에러가 발생하면 프로그램이 실행되지 않음. 이러한 특징 때문에 타입스크립트를 컴파일타임에 에러를 발견할 수 있는 정적 타입 검사기라고 부름.

## 코드 변환기로서의 타입스크립트 컴파일러

타입스크립트 코드가 자바스크립트 코드로 변환되는 과정은 타입 검사와 독립적으로 동작하기 때문에 타입스크립트 컴파일러는 타입 검사를 수행한 후 코드 변환을 시작함. 이때, 타입스크립트 코드의 타이핑이 잘못되어 발생하는 에러는 자바스크립트 실행 과정에서 런타임 에러로 처리됨.

```ts
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    // ‘Rectangle’ only refers to a type, but is being used as a value here
    // Property ‘height’ does not exist on type ‘Shape’
    // Property ‘height’ does not exist on type ‘Square’
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

예제에서 Square와 Rectangle은 인터페이스로, 타입스크립트에서만 존재함. 이들은 자바스크립트 런타임에서는 실제 객체가 아니며, 따라서 instanceof 같은 런타임 체크에 사용될 수 없음.

즉, instanceof 체크는 런타임에 실행되지만 Rectangle은 타입이기 때문에 자바스크립트 런타임은 해당 코드를 이해하지 못함. 타입스크립트 코드가 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문이 제거되어 버리기 때문에 런타임에서는 타입을 사용할 수 없는 것

## 타입스크립트 컴파일러의 구조

### 타입스크립트 컴파일러의 실행 과정

![타입스크립트 컴파일러의 실행 과정](https://camo.githubusercontent.com/66352f3e7803276c4fb20209900ca294d1256f4c1206ce0b9818b9137016a79c/68747470733a2f2f76656c6f672e76656c63646e2e636f6d2f696d616765732f6f726f6461652f706f73742f62663730396436642d316336352d346233302d623835352d6232306431613433333037372f696d6167652e706e67)

### 프로그램

타입스크립트 컴파일러는 tsc 명령어로 실행되며, 컴파일러는 tsconfig.json에 명시된 컴파일 옵션을 기반으로 컴파일을 수행.

먼저 전체적인 컴파일 과정을 관리하는 프로그램 객체(인스턴스)가 생성되며. 이 프로그램 객체는 컴파일할 타입스크립트 소스 파일과 소스 파일 내에서 임포트된 파일을 불러오는데, 가장 최초로 불러온 파일을 기준으로 컴파일 과정이 시작됨.

### 스캐너(Scanner)

스캐너는 타입스크립트 소스코드를 작은 단위로 나누어 의미 있는 토큰으로 변환하는 작업을 수행.

```ts
const woowa = "bros";
```

위 코드는 스캐너에 의해 다음과 같이 분석할 수 있음.
![스캐너](https://camo.githubusercontent.com/75b2b8623ecaf89ced39193f37e23290bc406bc12d4d7aeec190b8fd8de67676/68747470733a2f2f76656c6f672e76656c63646e2e636f6d2f696d616765732f6f726f6461652f706f73742f65346362333135642d303235642d343739392d386232332d6163623564326531303632312f696d6167652e706e67)

### 파서(Parser)

스캐너가 소스 파일을 토큰으로 나눠주면 파서는 그 토큰 정보를 이용하여 AST를 생성함.

AST는 컴파일러가 동작하는 데 핵심 기반이 되는 자료 구조로, 소스코드의 구조를 트리 형태로 표현하며 AST의 최상위 노드는 타입스크립트 소스 파일이며, 최하위 노드는 파일의 끝 지점으로 구성됨.

스캐너가 어휘적 분석을 통해 토큰 단위로 소스코드를 나눈다면, 파서는 이렇게 생성된 토큰 목록을 활용하여 구문적 분석을 수행한다고 볼 수 있으며 이를 통해 코드의 실질적인 구조를 노드 단위의 트리 형태로 표현하는 것.

각각의 노드는 코드상의 위치, 구문 종류, 코드 내용과 같은 정보를 담고있음.

```ts
function normalFunction() {
  console.log("normalFunction");
}

normalFunction();
```

![파서](https://camo.githubusercontent.com/73f35b09f84b88246f5aa4bce324676a0534af6535198bd926d964db131c09ef/68747470733a2f2f76656c6f672e76656c63646e2e636f6d2f696d616765732f6f726f6461652f706f73742f39626537303631372d326132332d346630382d613132362d6564666231633065366461652f696d6167652e706e67)

### 바인더(Binder)

바인더의 주요 역할은 체커 단계에서 타입 검사를 할 수 있도록 기반을 마련하는 것, 바인더는 타입 검사를 위해 심볼이라는 데이터 구조를 생성하며 심볼은 이전 단계의 AST에서 선언된 타입의 노드 정보를 저장함.

심볼의 인터페이스 일부는 다음과 같이 구성됨.

```ts
export interface Symbol {
  flags: SymbolFlags; // Symbol flags

  escapedName: string; // Name of symbol

  declarations?: Declaration[]; // Declarations associated with this symbol

  // 이하 생략...
}
```

flag 필드는 AST에서 선언된 타입의 노드 정보를 저장하는 식별자로 심볼을 구분하는 식별자 목록은 다음과 같음.

```ts
// src/compiler/types.ts
export const enum SymbolFlags {
  None = 0,
  FunctionScopedVariable = 1 << 0, // Variable (var) or parameter
  BlockScopedVariable = 1 << 1, // A block-scoped variable (let or const)
  Property = 1 << 2, // Property or enum member
  EnumMember = 1 << 3, // Enum member
  Function = 1 << 4, // Function
  Class = 1 << 5, // Class
  Interface = 1 << 6, // Interface
  // ...
}
```

심볼 인터페이스의 declarations필드는 AST 노드의 배열 형태를 보임.

결과적으로 바인더는 심볼을 생성하고 해당 심볼과 그에 대응하는 AST 노드를 연결하는 역할을 수행함.

다음은 여러 가지 선언 요소에 대한 각각의 심볼 결과.

```ts
type SomeType = string | number;

interface SomeInterface {
  name: string;
  age?: number;
}

let foo: string = "LET";

const obj = {
  name: "이름",
  age: 10,
};
class MyClass {
  name;
  age;
  constructor(name: string, age?: number) {
    this.name = name;
    this.age = age ?? 0;
  }
}

const arrowFunction = () => {};

function normalFunction() {}

arrowFunction();

normalFunction();

const colin = new MyClass("colin");
```

**여러 선언 요소에 대한 심볼**
![바인더](https://camo.githubusercontent.com/985148765c0f7815ff67bfea98c4ca582609af2e18ca15c37698495364259530/68747470733a2f2f76656c6f672e76656c63646e2e636f6d2f696d616765732f6f726f6461652f706f73742f37326162636265622d626532312d343966342d613030372d3838363865333363656462372f696d6167652e706e67)

### 체커(Checker)와 이미터(Emitter)

### 체커(Checker)

체커는 파서가 생성한 AST와 바인더가 생성한 심볼을 활용하여 타입 검사를 수행하며 이 단계에서 체커의 소스 크기는 파서의 소스 크기보다 매우 크며, 전체 컴파일 과정에서 타입 검사가 차지하는 비중이 크다는 것을 짐작할 수 있음.

체커의 주요 역할은 AST의 노드를 탐색하면서 심볼 정보를 불러와 주어진 소스 파일에 대해 타입 검사를 진행하는 것.

checker.ts의 getDiagnostics()함수를 사용해서 타입을 검증하고 타입 에러에 대한 정보를 보여줄 에러 메시지를 저장함.

### 이미터(Emitter)

이미터는 타입스크립트 소스를 자바스크립트(js) 파일과 타입 선언 파일(d.ts)로 생성함.

이미터는 타입스크립트 소스 파일을 변환하는 과정에서 개발자가 설정한 타입스크립트 설정 파일을 읽어오고, 체커를 통해 코드에 대한 타입 검증 정보를 가져옴 그리고 emitter.ts 소스 파일 내부의 emitFiles()함수를 사용하여 타입스크립트 소스 변환을 진행.

1. tsc 명령어를 실행하여 프로그램 객체가 컴파일 과정을 시작
2. 스캐너는 소스 파일을 토큰 단위로 분리
3. 파서는 토큰을 이용하여 AST를 생성
4. 바인더는 AST의 각 노드에 대응하는 심볼을 생성, 심볼은 선언된 타입의 노드 정보를 담고 있음
5. 체커는 AST를 탐색하면서 심볼 정보를 활용하여 타입 검사를 수행
6. 타입 검사 결과 에러가 없다면 이미터를 사용해서 자바스크립트 소스 파일로 변환
