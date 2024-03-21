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

## enum
- 런타임에 객체로 반환되는 값. 실제 객체로 존재

### const enum
- 
