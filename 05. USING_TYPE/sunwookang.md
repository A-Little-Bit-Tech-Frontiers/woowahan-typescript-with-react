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

