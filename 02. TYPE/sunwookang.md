## 타입 애너테이션 
- 타입을 명시적으로 선언해서 어떤 타입 값이 저장될 것인지 컴파일러에 직접 알려주는 것

## 구조적 서브타이핑
- 객체가 가지고 있는 속성을 바탕으로 타입을 구분하는 것
- 이름이 다르더라도 가진 속성이 동일하다면 서로 호환 가능한 타입

## Object.keys()의 타입을 확신할 수 없는 이유
```
interface Cube {
  width: number;
  height: number;
  depth: number;
}

function addLines(c: Cube) {
  for (const axis of Object.keys(c)){
    ...
  }
}
```
- c[axis]는 number일 것 같지만 c에는 어떤 속성이든 가질 수 있기에 에러가 발생
- Object.values()도 동일. any로 나오게 된다.

## enum
- 런타임에 객체로 반환되는 값. 실제 객체로 존재
- key-value의 관계를 양방향으로 선언

### const enum
- enum 앞에 const 식별자를 붙여 사용
- 컴파일 후 남지 따로 남지 않는다. -> 코드가 가벼워지고 Tree-shaking 가능
- inline이 되어버려 함수 사용에 대한 문제 발생 가능
- 요즘은 as const로 대체

### as const
- object의 value는 변할 수 있는 값
- 이 object를 enum처럼 변하지 않는 값으로 사용하고 싶을 때 사용

### 코드 예시

*enum 사용*
```
export enum PasswordResetModalEnum {
  INFO = 'info',
  CURRENT = 'current',
  FORM = 'form',
  COMPLETE = 'complete',
}

const PasswordResetModal = () => {
  const [currentStep, setCurrentStep] = useState<PasswordResetModalEnum>(
    PasswordResetModalEnum.INFO,
  );

  const decideComponentByStep = () => {
    switch (currentStep) {
      case PasswordResetModalEnum.INFO:
        return (
          <PasswordResetInfo
            onNextStep={() => setCurrentStep(PasswordResetModalEnum.CURRENT)}
          />
        );
      case PasswordResetModalEnum.CURRENT:
        return (
          <PasswordResetCurrent
            onNextStep={() => setCurrentStep(PasswordResetModalEnum.FORM)}
          />
        );
      case PasswordResetModalEnum.FORM:
        return (
          <PasswordResetForm
            onNextStep={() => setCurrentStep(PasswordResetModalEnum.COMPLETE)}
          />
        );
      case PasswordResetModalEnum.COMPLETE:
        return <PasswordResetComplete />;
    }
  };

  return <Modal>{decideComponentByStep()}</Modal>;
};
```

*객체에 as const 사용*
```
export const PasswordReset = {
  INFO: 'info',
  CURRENT: 'current',
  FORM: 'form',
  COMPLETE: 'complete',
} as const;

const PasswordResetModal = () => {
  const [currentStep, setCurrentStep] = useState<
    (typeof PasswordReset)[keyof typeof PasswordReset]
  >(PasswordReset.INFO);

  const decideComponentByStep = () => {
    switch (currentStep) {
      case PasswordReset.INFO:
        return (
          <PasswordResetInfo
            onNextStep={() => setCurrentStep(PasswordReset.CURRENT)}
          />
        );
      case PasswordReset.CURRENT:
        return (
          <PasswordResetCurrent
            onNextStep={() => setCurrentStep(PasswordReset.FORM)}
          />
        );
      case PasswordReset.FORM:
        return (
          <PasswordResetForm
            onNextStep={() => setCurrentStep(PasswordReset.COMPLETE)}
          />
        );
      case PasswordResetModalEnum.COMPLETE:
        return <PasswordResetComplete />;
    }
  };

  return <Modal>{decideComponentByStep()}</Modal>;
};
```

### 결론
- 코드의 명확성과 타입 안정성 -> enum 사용
- 프로젝트의 크기가 크고 런타임의 성능이 중요, JavaScript의 기본 기능 활용 -> 객체에 as const 사용
