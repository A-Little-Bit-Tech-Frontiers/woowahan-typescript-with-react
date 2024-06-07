# API 요청

## 컴포넌트의 반환 타입

### 1. ReactElement

- **정의**: ReactElement는 React 요소를 나타내는 인터페이스이며 React 요소는 React 컴포넌트의 가장 작은 단위로, 화면에 렌더링 될 수 있는 것을 의미함.

- **사용**: ReactElement는 주로 React 컴포넌트의 반환 타입으로 사용됨. 예를 들어, 함수형 컴포넌트나 클래스 컴포넌트에서 render 메서드는 ReactElement를 반환함.

- **예시:**

```tsx
const MyComponent: React.FC = (): React.ReactElement => {
  return <div>Hello World</div>;
};
```

### 2. ReactNode

- **정의**: ReactNode는 ReactElement보다 더 넓은 범위의 타입으로, ReactNode는 ReactElement, 문자열, 숫자, 배열, fragment, boolean 등을 포함할 수 있음. 즉, React 컴포넌트가 반환할 수 있는 거의 모든 것을 포함함.

- **사용**: ReactElement는 주로 React 컴포넌트의 반환 타입으로 사용됨. 예를 들어, 함수형 컴포넌트나 클래스 컴포넌트에서 render 메서드는 ReactElement를 반환함.

- **예시:**

```tsx
const MyComponent: React.FC = (): React.ReactNode => {
  return <div>{someCondition ? "Text" : <OtherComponent />}</div>;
};

type ReactFragment = {} | Iterable<ReactNode>;
type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

### 3. JSX.Element

- **정의**: JSX.Element는 TypeScript가 JSX를 사용할 때 정의하는 타입임 이는 기본적으로 ReactElement와 동일하지만, JSX 문법을 사용하는 경우에 한해 적용됨.

- **사용**: 주로 JSX 문법을 사용하는 함수나 컴포넌트의 반환 타입으로 사용됨.

- **예시:**

```tsx
const MyComponent = (): JSX.Element => {
  return <div>Hello World</div>;
};

declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

## DetailedHTMLProps와 ComponentPropsWithoutRef

HTML 태그의 속성 타입을 활용하는 대표적인 2가지 방법은 리액트의 DetailedHTMLProps와 ComponentPropsWithoutRef 타입을 활용하는 것.

- DetailedHTMLProps는 특정 HTML 요소의 모든 속성(기본 HTML 속성 및 React 추가 속성)을 포함하는 타입을 정의하는 데 사용됨. 이를 통해 특정 HTML 요소의 타입 안전성을 보장할 수 있음.

- ComponentPropsWithoutRef는 ref 속성을 포함하지 않는 컴포넌트의 타입을 정의할 때 사용됨. 이는 주로 forwardRef를 사용하지 않는 컴포넌트에 적합.

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

type ButtonProps = {
  onClick?: NativeButtonProps["onClick"];
};

type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
type ButtonProps = {
  onClick?: NativeButtonType["onClick"];
};
```
