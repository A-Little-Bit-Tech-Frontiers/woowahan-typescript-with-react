## 타입스크립트만의 독자적 타입 시스템

### unknown 타입

unknown 타입은 any 타입과 유사하게 모든 타입의 값이 할당될 수 있음 그러나 any를 제외한 다른 타입으로 선언된 변수에는 unknown 타입 값을 할당할 수 없음

unknown 타입은 어떤 타입이 할당되었는지 알 수 없음을 나타내기 때문에 unknown 타입으로 선언된 변수는 값을 가져오거나 내부 속성에 접근할 수 없음

```js
//할당하는 시점에서는 에러가 발생하지 않음
const unknownFunction: unknown = () => console.log("this is unknown type");

// 하지만 실행 시에는 에러가 발생; Error: Object is of type 'unknown'.ts (2571)
unknownFunction();
```

any 타입을 특정 타입으로 수정해야 하는 것을 깜빡하고 누락하면 어떤 값이든 전달될 수 있기 때문에 런타임에 예상치 못한 버그가 발생할 가능성이 높아지므로 이러한 상황을 보완하기 위해 등장한 unknown 타입

### never 타입

never 타입은 단어가 내포하고 있는 의미처럼 값을 반환할 수 없는 타입을 말함 -> 에러를 던지는 경우 또는 무한 루프(함수)가 실행되는 경우

never 타입은 모든 타입의 하위 타입으로 never 자신을 제외한 어떤 타입도 never 타입에 할당될 수 없다.
-> any 타입이라 할지라도 never 타입에 할당될 수 없음

따라서 타입스크렙트에서는 조건부 타입을 결정할 때 특정 조건을 만족하지 않는 경우에 엄격한 타입 검사 목적으로도 사용함

### enum 타입

enum 타입은 열거형이라고도 부르며 일종의 구조체를 만드는 타입 시스템. 타입스크립트는 명명한 각 멤버의 값을 스스로 추론하여 숫자 0부터 1씩 늘려가며 값을 할당함.

```js
enum ProgrammingLanguage {
    Typescript, // 0
    Javascript, // 1
    Java, // 2
    Python, // 3
    Kotlin, // 4
    Rust, // 5
    Go, // 6
}
```

아래와 같은 방식으로도 값을 늘려 간다

```js
enum ProgrammingLanguage {
    Typescript = 'Typescript',
    Javascript = 'Javascript',
    Java = 300,
    Python = 400,
    Kotlin, // 401
    Rust, // 402
    Go, // 403
}
```

이러한 enum은 아래와 같은 이점을 얻을 수 있다.

- 타입 안정성
- 명확한 의미 전달과 높은 응집력
- 가독성

이러한 enum에는 문제점이 존재한다. 숫자로만 이루어져 있거나 타입스크립트가 자동으로 추론한 열거형은 안전하지 않은 결과를 낳을 수 있다.

```js
// 역방향으로도 접근이 가능
ProgrammingLanguage[2];
```

이러한 동작을 막기 위해 `const enum`으로 열거형을 선언하는 방법이 있다. 이 방식은 역방향으로의 접근을 허용하지 않기 때문에 자바스크립트에서의 객체에 접근하는 것과 유사한 동작을 보장한다.

```js
ProgrammingLanguage[200]; // undefined를 출력하지만 별다른 에러를 발생시키지 않음

// 다음과 같이 선언하면 위와 같은 문제를 방지할 수 있음
const enum ProgrammingLanguage {
    // ...
}
```

이러한 `const enum`도 숫자 상수로 관리되는 열거형은 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못함. 반면 문자열 상수 방ㅇ식으로 선언한 열거형은 미리 선언하지 않은 멤버로 접근을 방지함

```js
const enum NUMBER {
    ONE = 1,
    TWO = 2,
}

const myNumber: NUMBER = 100; // NUMBER enum에서 100을 관리하고 있지 않지만 이는 에러를 발생시키지 않음

const enum STRING_NUMBER {
    ONE = "ONE",
    TWO = "TWO",
}

const myStringNumber: STRING_NUMBER = "THREE" // Error
```

이외에 타입 공간과 값 공간에서 모두 사용되기 때문에 타입스크립트 코드가 자바스크립트로 변환될 때 즉시 실행 함수(IIFE)형식으로 변환이 되며 이러한 경우 일부 번들러에서 트리쉐이킹 과정 중 즉시 실행 함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 경우가 발생할 수 있음

이러한 문제를 해결하기 위해서 앞서 언급했던 `const enum ` 또는 `as const assertion`을 사용해서 유니온 타입으로 열거형과 동일한 효과를 얻는 방법이 있음

## 타입 조합

### 교차 타입(Intersection)

교차 타입을 사용하면 여러 가지 타입을 결합하여 하나의 단일 타입으로 만들 수 있음.

```js
type ProductItem = {
  id: number,
  name: string,
  type: string,
  price: number,
  imageUrl: string,
  quantity: number,
};

type ProductItemWithDiscount = ProductItem & { discountAmount: number };
```

### 유니온 타입(Union)

유니온 타입은 타입 A 또는 타입 B 중 하나가 될 수 있는 타입을 말하며 A | B 로 표기

```js
type CardItem = {
  id: number,
  name: string,
  type: string,
  imageUrl: string,
};

type PromotionEventItem = ProductItem | CardItem;

const pritePromotionItem = (item: PromotionEventItem) => {
  console.log(item.name);

  console.log(item.quantity); // 컴파일 에러 발생
};
```

코드를 보면 item.quantity를 참조하면 컴파일 에러가 발생하는데 그 이유는 quantity 속성은 ProductItem 타입에만 존재하는 속성이기 때문이므로 유니온 타입을 사용해서 접근할 때는 공통된 속성만 참조가 가능하다는 점을 유의해야 함.

### 인덱스 시그니처(Index Signatures)

인덱스 시그니처는 특정 타입의 속성 이름은 알수 없지만 속성값의 타입을 알고 있을 때 사용

```js
interface IndexSignatureEx {
  [key: string]: number | boolean;
  length: number;
  name: string; // 에러 발생
}
```

인덱스 시그니처를 선언할 때 다른 속성을 추가로 명시하면 인덱스 시그니처에 포함되는 타입이어야 함

### 인덱스드 엑세스 타입(Indexed Access Types)

인덱스드 엑세스 타입은 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용

```js
type IndexedAccess = Example['a'];
type IndexedAccess2 = Example[keyof Example];
```

### 맵드 타입(Mapped Types)

다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법으로 인덱스 시그니처 문법을 사용해서 반복적인 타입 선언을 효과적으로 줄일 수 있음

```js
type Example = {
    a: number;
    b: string;
    c: boolean;
};

type Subset<T> = {
    [K in keyof T]?: T[K];
}
```

추가로 readonly, ? 수식어를 더하거나 제거할 수 있음

```js
type ReadOnlyEx = {
    readonly a: number;
    readonly b:
}

type CreateMutable<Type> = {
    -readonly [Property in keyof Type]: Type[Property];
}

type Concrete<Type> = {
    [Property in keyof Type]-?: Type[Perperty];
}
```

### 제네릭(Generic)

다양한 타입 간에 재사용성을 높이기 위해 사용

제네릭은 일반화된 데이터 타입을 의미하므로 함수나 클래스 등의 내부에서 제네릭을 사용할 때 어떤 타입이든 될 수 있음
-> 특정한 타입에서만 존재하는 멤버를 참조할 수 없음

```js
function exampleFun2<T>(arg: T): number {
  return arg.length; // 에러 발생: Property 'length' does not exist on Type 'T'
}
```

만약 특정 타입에서 존재하는 것을 사용하고 싶다면 내부에 특정 속성을 가진 타입만 받도록 제약(범위를 좁힘)을 걸어주면 가능함

```js
interface TypeWithLength {
    length: number;
}

function exampleFunc2<T extends TypeWithLength(ar: T): number> {
    return arg.length;
}
```
