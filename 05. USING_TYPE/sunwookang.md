## extends
`T extends U ? X : Y`
-> T를 U에 할당할 수 있으면 X 아니면 Y

generic에서 extends는 들어오는 타입에 제약을 걸어주는 것
`<T extends string>`
-> T는 string이어야 함

## infer
타입 추론
`T extends infer U ? X : Y`
T가 U로 추론이 되면 X 아니면 Y

```ts
type ElementType<T> = T extends (infer U)[] ? U : never;
```
T가 배열이라면 배열의 요소 타입 U 아니면 never

```ts
function getFirstElement<T extends any[]>(arr: T): T extends [infer U, ...any[]] ? U : never {
  return arr[0];
}
```
T는 배열. 하나 이상의 요소를 가진다면 첫번째 요소의 타입 U를 반환 아니면 never

## NonNullable
generic으로 들어온 타입이 null이나 undefined이면 never 아니면 타입 그대로

책에선
`type NonNullable<T> = T extends null | undefined ? never : T;`
이렇게 소개하였으나 실제론
`type NonNullable<T> = T & {};`
이렇게 생겼다.

{}는 아무런 특정 속성이 없는 객체를 의미한다. 모든 객체는 {}를 상속하므로 null과 undefineds는 걸러진다.

## PartialRecord
책에는 PartialRecord라는 유틸리티 타입을 만들어 사용하고 있다.
`type PartialRecord<K extends string, T> = Partial<Record<K, T>>;`
Record에 Partial을 씌워 만들어진 객체의 모든 속성을 선택적으로 만든다.
그냥 Record를 만들 때 key에 그냥 string을 넣고 그 객체를 사용할 때 실제로 만들어진 객체에 해당 키가 없더라도 에러가 발생하지 않는다.

```
const fundingAccountHistoryStatusObj: Record<string, StatusVariantType> = {
  APPROVED: 'approve',
  REJECTED: 'rejected',
  CREATED: 'created',
  PENDING: 'pending',
};

<Status variant={fundingAccountHistoryStatusObj[info.getValue()]}>
          {t(`fundingAccounts:${info.getValue().toLowerCase()}`)}
        </Status>
```
이런식으로 사용할 때 저 `info.getValue()`에 string의 어떤 값을 넣더라도 에러가 나지 않는다. `fundingAccountHistoryStatusObj['abcd']` 이렇게 abcd가 `fundingAccountHistoryStatusObj`에 없음에도 불구하고 에러가 나지 않는다. 이럴 때 PartialRecord를 사용할 수 있다.

```
const fundingAccountHistoryStatusObj: PartialRecord<string, StatusVariantType> =
  {
    APPROVED: 'approve',
    REJECTED: 'rejected',
    CREATED: 'created',
    PENDING: 'pending',
  };
```
이렇게 만들고 사용하면 `fundingAccountHistoryStatusObj[info.getValue()]` 이것이 undefined일 수도 있어서 `Type 'StatusVariantType | undefined' is not assignable to type 'StatusVariantType'.` 이런 에러가 뜨게 된다. 이렇게 에러가 발생하기에 문제를 막을 수 있다.

그럼 이렇게 PartialRecord를 쓰면 끝일까? 여기서 중요한 것은 그게 아닌 것 같다.
가능하면 Record key에 string말고 정해진 값을 넣자.

```
type FundingAccountHistoryStatusType =
  | 'APPROVED'
  | 'REJECTED'
  | 'CREATED'
  | 'PENDING';

const fundingAccountHistoryStatusObj: Record<FundingAccountHistoryStatusType, StatusVariantType> =
  {
    APPROVED: 'approve',
    REJECTED: 'rejected',
    CREATED: 'created',
    PENDING: 'pending',
  };
```
`FundingAccountHistoryStatusType`을 만들어서 사용하면 PartialRecord를 쓰지 않고도 더욱 안정성을 높게 만들 수 있다.
