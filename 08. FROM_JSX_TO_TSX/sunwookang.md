## 컴포넌트 타입

### 함수 컴포넌트

- React.FC
    - 함수형 컴포넌트를 타입으로 정의
- React.VFC
    - Props에 children을 포함하지 않는 함수형 컴포넌트
- : JSX.Element
    - 위 두 타입의 반환타입
    - Props의 타입을 정의할 순 없음

```tsx
interface Props {}

// 1번 방식
const index = (props: Props) => {
  return <div>index</div>;
};

// 2번 방식
const index: React.FC<Props> = (props) => {
  return <div>index</div>;
};
```
<br/>

- 두 방식 중에선 요즘은 1번 방식으로 많이 사용
- React팀에선 2번 방식을 권장하지 않음
    - 제너릭 타입 매개변수와의 호환성이 제한적
    - 어차피 추론됨. 명시적으로 굳이 쓸 필요가 없음
 
<br/>

- ReactElement
    - React 컴포넌트나 DOM 요소를 표현하는 객체
    - JSX로 작성된 코드가 변환된 결과
- JSX.Element
    - JSX를 사용하여 생성된 ReactElement의 타입을 나타냄
- ReactNode
    - 컴포넌트의 children으로 허용할 수 있는 모든 타입을 표현
    - ReactElement, 문자열, 숫자, null, undefined 모두 가능

<br/>

- ReactElement와 JSX.Element의 차이
    - 실제로 존재하는 구체적인 객체가 아니라 타입스크립트에서 타입 정의로 사용
    - 동일하게 동작하지만, 이를 TypeScript 코드에서 구체적으로 타입으로 지정할 때 JSX.Element 사용

<br/>

- DetailedHTMLProps
    - HTMLAttributes와 HTMLProps를 결합하여 더 구체적이고 세부적인 HTML 태그의 props를 정의
    - `DetailedHTMLProps<HTMLAttributes<HTMLButtonElement>, HTMLButtonElement>`
- ComponentWithoutRef
    - ref를 사용하지 않는 컴포넌트의 props 타입을 정의
- HTMLProps
    - HTMLAttributes 를 확장하며, ref 속성도 포함
- HTMLAttributes
    - 특정 HTML 태그에 대해 기본적인 HTML 속성(이벤트 핸들러 포함)을 정의
- GPT 추천 가이드라인
    - ![image](https://github.com/NiNinanana/woowahan-typescript-with-react/assets/74632731/99f072a5-97ab-46b7-8ce2-cb14505ff09a)

    

## 리액트 컴포넌트

### 제네릭 컴포넌트

```tsx
interface SelectProps<OptionType extends Record<string, string>> {
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  options,
  onChange,
  selectedOption,
}: SelectProps<OptionType>) => {};
```

사용할 때

- <Select<Fruit> … />
- 넘겨주는 props의 타입으로 자동 추론
