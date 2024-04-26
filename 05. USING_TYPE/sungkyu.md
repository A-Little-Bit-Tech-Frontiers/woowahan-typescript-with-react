## 5.1 조건부 타입

타입스크립트의 조건부 타입은 자바스크립트의 삼항 연산자와 동일한 형태를 가짐.

```
Condition ? A : B;
// Condition이 true일 때 A 타입
// Condition이 false일 때 B 타입
```

조건부 타입을 활용하면 얻을 수 있는 **장점**

- **중복되는 타입 코드를 제거** 가능
- 상황에 따라 적절한 타입을 얻을 수 있기에 **더욱 정확한 타입 추론**이 가능

### extends와 제네릭을 활용한 조건부 타입

타입스크립트에서 다양한 상황에서 활용되는 **`extends`**

```
T extends U ? X : Y
// 타입 T를 U에 할당할 수 있으면 X 타입
// 타입 T를 U에 할당할 수 없으면 Y 타입
```

```tsx
interface Bank {
  financialCode: string;
  fullName: string; // Card와의 차이
}
interface Card {
  financialCode: string;
  appCardType?: string; // Bank와의 차이
}
type PayMethod<T> = T extends "card" ? Card : Bank; // 제네릭 타입으로 extends를 사용한 조건부 타입
type CardPayMethodType = PayMethod<"card">; // 제네릭 매개변수 = "card" : Card 타입
type BankPayMethodType = PayMethod<"bank">; // 제네릭 매개변수 != "card" : Bank 타입
```

**조건부 타입을 사용하지 않았을 때의 문제점**

```tsx
type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;

export const useGetRegisteredList = (
  type: "card" | "appcard" | "bank"
): UseQueryResult<PayMethodType[]> => {
  const url = `baeminpay/codes/${type === "appcard" ? "card" : type}`;
  const fetcher = fetcherFactory<PayMethodType[]>({
    onSuccess: (res) => {
      const usablePocketList =
        res?.filter(
          (pocket: PocketInfo<Card> | PocketInfo<Bank>) =>
            pocket?.useType === "USE"
        ) ?? [];
      return usablePocketList;
    },
  });
  const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);

  return result;
};
```

`Card`와 `Bank` 를 명확히 구분하는 로직이 없음

사용자가 인자로 "card"를 전달했을 때 함수가 반환하는 타입이 `PayMethodInfo<Card>[]`였으면 좋겠지만, 타입 설정이 유니온(`|`)으로만 되어있기 때문에 구체적으로 추론할 수 없음.

즉, `useGetRegisteredList`는 인자로 넣는 타입에 알맞은 타입을 반환하지 못하는 함수

**extends 조건부 타입을 활용하여 개선하기**

extends **조건부 타입**을 활용하여 하나의 API 함수에서 타입에 따라 정확한 반환 타입을 추론하게 만들 수 있음. 또한 extends를 **제네릭의 확장자**로 활용해서 "card", "appcard", "bank" 외 다른 값이 인자로 들어오는 경우도 방어.

```tsx
// before
type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;

// after
type PayMethodType<T extends "card" | "appcard" | "bank"> = T extends
  | "card"
  | "appcard"
  ? Card
  : Bank;
// PayMethodType의 제네릭으로 받은 값이 "card" 또는 "appcard"면 PayMethodInfo<Card> 타입을 반환
// PayMethodType의 제네릭으로 받은 값이 이외의 값이면 PayMethodInfo<Bank> 타입을 반환
```

새롭게 정의한 `PayMethodType`에 제네릭 값을 넣어주기 위해 `useGetRegisteredList` 함수 인자의 타입을 넣음.

```tsx
// before
export const useGetRegisteredList = (
  type: "card" | "appcard" | "bank"
): UseQueryResult<PayMethodType[]> => {
  /* ... */
  const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
  return result;
};

// after
export const useGetRegisteredList = <T extends "card" | "appcard" | "bank">(
  type: T
): UseQueryResult<PayMethodType<T>[]> => {
  /* ... */
  const result = useCommonQuery<PayMethodType<T>[]>(url, undefined, fetcher);
  return result;
};
```

이렇게 조건부 타입을 활용함으로써

- 인자로 "card" 또는 "appcard"를 받으면 `PayMethodInfo<Card>`를 반환
- 인자로 "bank"를 받으면 `PayMethodInfo<Bank>`를 반환

이에 따라 불필요한 타입 가드와 불필요한 타입 단언을 하지 않아도 됨.

**infer를 활용해서 타입 추론하기**

extends를 사용할 때 **`infer`** 키워드 사용하며 extends로 조건을 서술하고 infer로 타입을 추론

```tsx
type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;
// Promise<infer K> : Promise의 반환 값을 추론해 해당 값의 타입을 K라고 지정

const promises = [Promise.resolve("Mark"), Promise.resolve(38)];
type Expected = UnpackPromise<typeof promises>; // string | number
```

### 커스텀 유틸리티 타입 PickOne 구현하기

해당 커스텀 유틸리티 타입을 만들기 위해 작은 단위 타입인 One과 ExcludeOne 타입을 각각 구현한 뒤, 두 타입을 활용해 하나의 타입 PickOne을 표현.

```js
// One<T> : 제네릭 타입 T의 1개 키는 값을 가짐
type One<T> = { [P in keyof T]: Record<P, T[P]> }[keyof T];

// ExcludeOne<T> : 제네릭 타입 T의 나머지 키는 옵셔널한 undefined 값을 가짐
type ExcludeOne<T> = {
  [P in keyof T]: Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];

// PickOne<T> = One<T> + ExcludeOne<T>
type PickOne<T> = One<T> & ExcludeOne<T>;
```

작은 단위부터 단계별로 구현해 만든 타입 PickOne을 이용해 정확한 타입을 추론하도록 할 수 있음.

```js
type Card = {
  card: string;
};
type Account = {
  account: string;
};

// 커스텀 유틸리티 타입 PickOne
type PickOne<T> = {
  [P in keyof T]: Record<P, T[P]> &
    Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];

// CardOrAccount가 Card의 속성이나 Account의 속성 중 하나만 가질 수 있게 정의
type CardOrAccount = PickOne<Card & Account>;

function withdraw(type: CardOrAccount) {
  /* ... */
}

withdraw({ card: "hyundai", account: "hana" }); // ERROR
withdraw({ card: "hyndai" }); // ok
withdraw({ card: "hyundai", account: undefined }); // ok
withdraw({ account: "hana" }); // ok
withdraw({ card: undefined, account: "hana" }); // ok
withdraw({ card: undefined, account: undefined }); // ERROR
```

### NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

NonNullable 타입

```js
type NonNullable<T> = T extends null | undefined ? never : T;
```

- 제네릭으로 받는 T가 null 또는 undefined일 때 never 또는 T를 반환하는 타입
- null이나 undefined가 아닌 경우를 제외하기 위해 사용

NonNullable 함수

```js
function NonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}
```

- 매개변수(value)가 null 또는 undefined일 때 false를 반환하는 함수
- 반환값이 true라면 null과 undefined가 아닌 다른 타입으로 타입 가드.

Promise.all을 사용할 때 NonNullable를 적용한 예시

```js
const shopList = [
  { shopNo: 100, category: "chicken" },
  { shopNo: 101, category: "pizza" },
  { shopNo: 102, category: "noodle" },
];

class AdCampaignAPI {
  static async operating(shopNo: number): Promise<AdCampaign[]> {
    try {
      return await fetch(`/ad/shopNumber=${shopNo}`);
    } catch (error) {
      return null;
    }
  }
}

const shopAdCampaignList = await Promise.all(
  shopList.map((shop) => AdCampaignAPI.operating(shop.shopNo))
);
```

AdCampaignAPI.operating 함수는 error시 null을 반환하도록 되어있기 때문에 shopAdCampaignList의 타입은 Array<AdCampaign[] | null>로 추론됨. 이렇게 되면 순회할 때마다 고차 함수 내 콜백 함수에서 if문을 사용한 타입 가드를 반복하게 되는 문제가 생길 수 있음.

아래와 같이 단순하게 필터링을 해도 null이 필터링 되지는 않음.

```js
const shopAds = shopAdCampaignList.filter((shop) => !!shop);
// shopAds의 타입 : Array<AdCampaign[] | null>
```

다음과 같이 NonNullable를 이용해 null을 필터링해야만 Array<AdCampaign[]>로 추론하게 만들 수 있음.

```js
// showAdCampaignList가 null이 될 수 있는 경우를 방어하기 위해 NonNullable 사용
const shopAds = shopAdCampaignList.filter(NonNullable);
// shopAds는 필터링을 통해 null이나 undefined가 아닌 값을 가진 배열이 됨
// shopAds의 타입 : Array<AdCampaign[]>
```
