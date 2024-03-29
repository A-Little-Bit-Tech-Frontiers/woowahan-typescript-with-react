## unknown
- 어떤 타입이든 unknown 타입에 할당 가능
- unknown 타입은 any 타입 외에 다른 타입으로 할당 불가능
- 데이터 구조를 파악하기 힘들 때 any 타입 대신 unknown 타입 활용

## never
- 값을 반환할 수 없는 타입
  - 에러를 던지는 경우
  - 무한히 함수가 실행되는 경우
 
### useState에서 빈 배열을 default Value로 주었을 때 값이 never[]인 이유
`const [data, setData] = useState([]);`

이렇게 빈 배열만 useState의 초기값을 넣었을 때 data의 타입은 `never[]`, setData의 타입은 `React.Dispatch<React.SetStateAction<never[]>>`가 된다.

빈 배열 []는 배열 내에 요소가 없기 때문에 어떤 타입의 요소도 들어갈 수 없다고 판단한다. 따라서 어떠한 값도 들어갈 수 없는 never[]를 반환하게 되는 것이다.

## 인덱스드 엑세스 타입
- 다른 타입의 특정 속성이 가지는 타입을 조회

### Router의 타입 조회
`NextRouter['push']`와 같이 useRouter를 직접 호출해서 사용하지 않고 push 등을 받으려 할 때 push 함수에 대해 따로 타입을 import 해올 수 없으니 NextRouter에 인덱스트 엑세스 타입을 활용한다.

## readonly
- 특정 속성이나 배열이 수정 불가능하다는 것을 명시
- 불변성을 강제하여 프로그램의 안정성과 예측 가능성을 높임

### readonly 사용
- react props / state
  - react의 props와 state는 불변성을 가져야만 한다.
  - 따라서 react 내부적으로 readonly로 표기하고 있기에 직접 적어줄 필요 없음
- as const
  - 객체에 as const를 붙이면 모든 속성이 readonly가 된다.
