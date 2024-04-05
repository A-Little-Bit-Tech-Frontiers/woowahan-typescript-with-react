## 타입 확장하기

### 유니온 타입

```js
type MyUnion = A | B;
```

A와 B의 유니온 타입인 MyUnion은 타입 A와 B의 합집합이다. 합집합으로 해석하면 집합 A의 모든 원소는 집합 MyUnion의 원소이며, 집합 B의 모든 원소 역시 집합 MyUnion의 원소라는 뜻이다. 즉 A 타입과 B 타입의 모든 값이 MyUnion 타입의 값이 됨
-> 유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근할 수 있음(교집합이 아닌 합집합이기 때문)

```js
interface CookingStep {
  orderId: string;
  price: number;
}

interface DeliveryStep {
  orderId: string;
  time: number;
  distance: string;
}

function getDeliveryDistance(step: Cooking | DeliveryStep) {
  return step.distance;
  // Property 'distance' does not exist on type 'CookingStep | DeliveryStep'
  // Property 'distance' does not exitst on type 'CookingStep'
}
```

## 타입 좁히기 - 타입 가드

자바스크립트 연산자를 활용한 타입 가드는 typeof, instanceof, in과 같은 연산자를 사용해서 제어문으로 특정 타입 값을 가질 수밖에 없는 상황을 유도

자바스크립트 연산자를 사용하는 이유는 런타임에 유효한 타입 가드를 만들기 위해 사용

### 원시 타입을 추론할 때: typeof 연산자 활용

### 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용

A instanceof B 형태로 사용하며 A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어가며, instanceof는 A의 프로토타입 체인에 생성자 B가 존재하는지를 검사해서 true, false로 반환.

```js
export function convertToRange(selected?: Date | Range): Range | undefined {
  return selected instanceof Date
    ? { start: selected, end: selected }
    : selected;
}

const onKeyDown = (event: React.KeyboardEvent) => {
  if (event.target instanceof HTMLInputElement && event.key === "Enter") {
    // 이 분기에서는 event.target의 타입이 HTMLInputElement이며
    // event.key가 'Enter'이다
    event.targEt.blur();
    onCTAButtonClick(event);
  }
};
```

### 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용

in 연산자는 객체에 속성이 있는지 확인한 다음에 true 또는 false를 반환

```js
const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
  if ("cookieKY" in props) return <NoticeDialogWithCookie {...props} />;
  return <NoticeDialogBase {...props} />;
};
```

### is 연산자로 사용자 정의 타입 가드를 만들어 활용하기

사용자 정의 타입 가드는 반환 타입이 타입 명제(type predicates)인 함수를 정의하여 사용할 수 있음. 타입 명제는 A is B 형식으로 작성하면 되는데 여기서 A는 매개변수 이름이고 B는 타입

참/거짓의 진릿값을 반환하면서 반환타입을 타입 명제로 지정하게 되면 반환 값이 참일 때 A 매개변수의 타입을 B 타입으로 취급하게 된다.

> \*타입 명제(type predicates) : 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수

```js
const isDestinationCode = (x: string): x is DestinationCode =>
  destinationCodeList.includes(x);
```

isDestinationCode의 반환 값 타이핑을 x is DestinationCode가 아닌 boolean으로 한다면 타입스크립트는 아래의 코드에서 includes 함수를 타입으로 추론할 수 없기에 에러가 발생

```js

const getAvailableDestinationNameList = async (): Promise<DestinationName[]> => {
  const data = await AxiosRequest<string[]>(“get”, “.../destinations”);
  const destinationNames: DestinationName[] = [];
  data?.forEach((str) => {
  if (isDestinationCode(str)) { // str이 destinationCodeList의 원소가 맞는지 체크.
    destinationNames.push(DestinationNameSet[str]); // 맞다면 DestinationNames 배열에 push.
    /*
    isDestinationCode의 반환 값에 is를 사용하지 않고 boolean이라고 한다면 다음 에러가
    발생한다
    - Element implicitly has an ‘any’ type because expression of type ‘string’
    can’t be used to index type ‘Record<”MESSAGE_PLATFORM” | “COUPON_PLATFORM” | “BRAZE”,  // string[] 타입인 str을 DestinationName[]에 push할 수 없다는 에러
    “통합메시지플랫폼” | “쿠폰대장간” | “braze”>’
    */
    }
  });
  return destinationNames;
};
```

### 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

태그된 유니온으로도 불리는 식별할 수 있는 유니온은 타입 좁히기에 널리 사용되는 방식

```js
type TextError = {
  errorCode: string,
  errorMessage: string,
};
type ToastError = {
  errorCode: string,
  errorMessage: string,
  toastShowDuration: number, // 토스트를 띄워줄 시간
};
type AlertError = {
  errorCode: string,
  errorMessage: string,
  onConfirm: () => void, // 얼럿 창의 확인 버튼을 누른 뒤 액션
};
```

```js
const errorArr: ErrorFeedbackType[] = [
  // ...
  {
  errorCode: “999”,
  errorMessage: “잘못된 에러”,
  toastShowDuration: 3000,
  onConfirm: () => {},
  }, // expected error
];
```

위의 코드에서는 에러를 예상하지만 자바스크립트의 “덕 타이핑 언어” 특징으로 인해 별도의 타입 에러를 뱉지 않는 것을 확인할 수 있음. 이런 상황에서 타입 에러가 발생하지 않는다면 앞으로의 개발에서 의미를 알 수 없는 에러 객체가 생겨날 위험성이 커짐.

### 식별할 수 있는 유니온

각 타입이 비슷한 구조를 가지지만 서로 호환되지 않도록 만들어주기 위해서는 타입들이 서로 포함 관계를 가지지 않도록 정의해야 함. 이때 바로 식별할 수 있는 유니온을 활용할 수 있음.

**식별할 수 있는 유니온** : 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자를 달아주어 포함 관계를 제거 하는 것.

아래 코드는 errorType을 추가한 코드

```js
type TextError = {
  errorType: “TEXT”;
  errorCode: string;
  errorMessage: string;
};
type ToastError = {
  errorType: “TOAST”;
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number;
}
type AlertError = {
  errorType: “ALERT”;
  errorCode: string;
  errorMessage: string;
  onConfirm: () = > void;
};
```

아까와 달리 에러가 발생함

```js
type ErrorFeedbackType = TextError | ToastError | AlertError;

const errorArr: ErrorFeedbackType[] = [
  { errorType: “TEXT”, errorCode: “100”, errorMessage: “텍스트 에러” },
  {
    errorType: “TOAST”,
    errorCode: “200”,
    errorMessage: “토스트 에러”,
    toastShowDuration: 3000,
  },
  {
    errorType: “ALERT”,
    errorCode: “300”,
    errorMessage: “얼럿 에러”,
    onConfirm: () => {},
  },
  {
    errorType: “TEXT”,
    errorCode: “999”,
    errorMessage: “잘못된 에러”,
    toastShowDuration: 3000, // Object literal may only specify known properties, and ‘toastShowDuration’ does not exist in type ‘TextError’
    onConfirm: () => {},
  },
  {
    errorType: “TOAST”,
    errorCode: “210”,
    errorMessage: “토스트 에러”,
    onConfirm: () => {}, // Object literal may only specify known properties, and ‘onConfirm’ does not exist in type ‘ToastError’
  },
  {
    errorType: “ALERT”,
    errorCode: “310”,
    errorMessage: “얼럿 에러”,
    toastShowDuration: 5000, // Object literal may only specify known properties, and ‘toastShowDuration’ does not exist in type ‘AlertError’
  },
];
```

### Exhaustiveness Checking으로 정확한 타입 분기 유지하기

Exhaustiveness Checking은 모든 케이스에 대해 철저하게 타입을 검사하는 것을 말하며 타입 좁히기에 사용되는 패러다임 중 하나. 모든 케이스에 대해 분기 처리해야만 유지보수 측면에서 안전하다고 생각되는 경우 Exhaustiveness Checking을 통해 모든 케이스에 대한 타입 검사를 강제할 수 있음.

-> 개발자의 실수를 방지할 수 있으며 엄격하게 타입을 검사할 수 있음

```js
type ProductPrice = “10000” | “20000” | “5000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  if (productPrice === “5000”) return “배민상품권 5천 원”; // 조건 추가 필요
  else {
    return “배민상품권”;
  }
};
```

```js
type ProductPrice = “10000” | “20000” | “5000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  // if (productPrice === “5000”) return “배민상품권 5천 원”;
  else {
    exhaustiveCheck(productPrice); // Error: Argument of type ‘string’ is not assignable to parameter of type ‘never’
    return “배민상품권”;
  }
};

const exhaustiveCheck = (param: never) => {
  throw new Error(“type error!”);
};
```

첫번째 예시코드에 있던 productPrice가 “5000”일 때의 분기 처리가 주석된 상태로 exhaustiveCheck(productPrice);에서 에러를 뱉고 있는데 5000이라는 값에 대한 분기 처리를 하지 않아서 (철저하게 검사하지 않았기 때문에) 발생한 것을 볼 수 있음. 이렇게 모든 케이스에 대한 타입 분기 처리를 해주지 않았을 때, 컴파일타임 에러가 발생하게 하는 것을 Exhaustiveness Checking이라고 함

exhaustiveCheck 함수를 자세히 보면, 이 함수는 매개변수를 never 타입으로 선언하고 있음. 즉, 매개변수로 그 어떤 값도 받을 수 없으며 만일 값이 들어온다면 에러를 내뱉음. 이 함수를 타입 처리 조건문의 마지막 else 문에 사용하면 앞의 조건문에서 모든 타입에 대한 분기 처리를 강제할 수 있음.

이렇게 Exhaustiveness Checking을 활용하면 예상치 못한 런타임 에러를 방지하거나 요구사항이 변경되었을 때 생길 수 있는 위험성을 줄일 수 있음. 타입에 대한 철저한 분기 처리가 필요하다면 Exhaustiveness Checking 패턴을 활용하는 것도 하나의 방법.
