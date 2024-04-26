## 타입스크립트의 컴파일

- tsc라고 불리는 컴파일러를 통해 자바스크립트 코드로 변환
- .ts 확장자 파일을 찾아내 컴파일한 후 .js 확장자 파일을 만들어냄
- tsc는 소스코드를 해석하여 AST(최소 구문 트리)를 만들고 타입 확인을 거친 후 코드 생성

### 컴파일 과정 → tsc

1. 타입스크립트 소스코드를 AST로 만든다.
2. 타입 검사기가 AST를 확인하여 타입을 확인한다.
3. AST를 자바스크립트 소스로 변환한다.

### 자바스크립트 컴파일 과정 → 런타임

1. 자바스크립트 소스코드를 AST로 만든다.
2. AST가 바이트 코드로 변환된다.
3. 바이트 코드가 평가되어 프로그램이 실행된다.

| 💡 AST

컴파일러가 소스코드를 해석하는 과정에서 생성된 데이터 구조. 어휘적 분석, 구문 분석을 통해 소스코드를 노드 단위의 트리 구조로 구성

## 타입스크립트 컴파일러

- 코드 검사기
    - 런타임에서 발생할 수 있는 문법 오류 등의 에러뿐만 아니라 타입 에러도 잡아냄
    - tsc binder를 사용하여 타입 검사
- 코드 변한기
    - 타입을 검사한 후 타입스크립트 코드를 런타임 환경에서 동작할 수 있도록 자바스크립트로 트랜스파일
    - 타입 정보가 제거됨

### 컴파일 이후 런타임에선 타임 검사 불가

```tsx
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

- interface는 컴파일 과정에만 존재하고 런타임엔 없음

```tsx
interface Square {
  type: 'square';
  width: number;
}

interface Rectangle {
  type: 'rectangle';
  width: number;
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape): number {
  switch (shape.type) {
    case 'rectangle':
      return shape.width * shape.height;
    case 'square':
      return shape.width * shape.width;
    default:
      return 0;
  }
}

```

- 태그 식별자 사용할 수 있다.

## 타입스크립트 컴파일러 구조

1. 프로그램
    1. tsc 명령어로 실행
    2. tsconfig.json에 명시된 컴파일 옵션을 기반으로 수행
2. 스캐너
    1. 소스 파일을 어휘적으로 분석하여 토큰 생성
    2. `const name = "sun";`
        1. const: constKeyword
        2. 띄어쓰기: whitespace triva
        3. name: identifier
        4. =: equalsToken
        5. ‘sun’: stringliteral
        6. ;: semicolon token
3. 파서
    1. 토큰 정보를 이용하여 AST 생성
    2. AST
        1. 컴파일러가 동작하는데 핵심 기반이 되는 자료 구조
        2. 소스코드의 구조를 트리형태로 표현
            1. 최상위 노드는 타입스크립트 소스 파일
            2. 최하위 노드는 파일의 끝 지점
        3. 구문적 분석 수행
            1. identifier, block, call expression…
4. 바인더
    1. 체커 단계에서 타입 검사를 할 수 있도록 기반 마련
    2. 타입 검사를 위해 심볼이라는 데이터 구조 생성
        1. 이전 단계의 AST에서 선언된 타입의 노드 정보 저장
5. 체커
    1. AST와 심볼을 활용하여 타입 검사 수행
6. 이미터
    1. 타입스크립트 소스를 자바스크립트 파일과 타입 선언 파일로 생성

## TSX 파일 컴파일
- 트랜스파일 중에 JSX 구문 처리
  - React.createElement 호출로 변환
