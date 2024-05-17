## API 호출
- 여러 API 요청 정책이 변경될 수 있으니 비동기 호출 코드는 컴포넌트 영역에서 분리되어 처리되어야 한다.
- axios의 create method를 사용하여 api instance를 만들어 사용
<br/>

- api 응답 타입 지정
    - Resonse를 제네릭 타입으로 만들고 각 api에 맞춰서 사용
<br/>

> 좋은 컴포넌트는 변경될 이유가 하나뿐인 컴포넌트

- view model을 만들어 api 응답이 바뀌어도 ui가 깨지지 않게 개발 가능
<br/>

- superstruct 라이브러리를 사용해 런타임에서 응답 타입 검증
    - 인터페이스 정의와 자바스크립트 데이터의 유효성 검사를 쉽게
    - 개발자와 사용자에게 자세한 런타임 에러를 보여줌
    - zod로도 가능하다
        ```tsx
        // 사용자 정보 스키마 정의
        const UserSchema = z.object({
          id: z.number(),
          name: z.string(),
          email: z.string().email(),
          age: z.number().optional(),
        });
        
        // API 응답 스키마 정의
        const ApiResponseSchema = z.object({
          status: z.string(),
          data: UserSchema,
          error: z.string().optional(),
        });
        ```
        
        ```tsx
        async function fetchUserData(apiUrl) {
          try {
            // API 호출
            const response = await fetch(apiUrl);
            const responseData = await response.json();
        
            // 응답 데이터 유효성 검사
            ApiResponseSchema.parse(responseData);
        
            console.log('유효한 응답입니다:', responseData);
          } catch (error) {
            if (error instanceof z.ZodError) {
              // 유효성 검사 실패
              console.error('유효하지 않은 응답 데이터:', error.errors);
            } else {
              // 기타 오류 (예: 네트워크 오류)
              console.error('API 호출 오류:', error);
            }
          }
        }
        
        // 예제 API URL
        const apiUrl = 'https://api.example.com/user/1';
        fetchUserData(apiUrl);
        
        ```

## API 상태 관리하기
- 컴포넌트 내에서는 비동기 함수를 직접 호출하지 않고 api의 상태가 관리되어야 한다.
- redux, mobx
    - axios interceptor등을 통해 api 요청이 완료되면 상태를 저장
- react-query, useSwr
    - 훅
    - 폴링도 간단하게 사용 가능

## API 에러 핸들링
- 타입 가드 활용
    - 미리 셋팅해놓은 에러가 있을 땐 해당 에러를 보여주고 없으면 일반적인 에러
- axios interceptor
- 에러 바운더리

## API 모킹
- JSON 파일 불러오기
- NextApiHandler
