# API 요청

## 에러 서브 클래싱 하기

서브클래싱을 사용하여 코드상에서 어떤 에러가 발생한것인지 바로 확인할 수 있으며 에러 인스턴스가 무엇인지에 따라 에러 처리 방식을 다르게 구현할 수 있음

```ts
class OrderHttpError extends Error {

  private readonly privateResponse: AxiosResponse<ErrorResponse : undfiend>

  constructor(message?: string, response?:AxiosResponse<ErrorResponse>){
    super(message);
    this.name = "OrderHttpError"
    this.privateResponse = response
  }

  get response(): AxiosResponse<ErrorResponse> | undfined {
    return this.privateResponse
  }
}

class NetworError extends Error{
    constructor(message: ""){
    super(message);
    this.name = "NetworkError"
  }
}

class UnauthorizedError extends Error{
    constructor(message?: string, response?:AxiosResponse<ErrorResponse>){
    super(message, response);
    this.name = "Unauthorized"
  }
}
```

```ts
// 주문 내역 가져오기
const fetchOrderHistory = async () => {
  try {
    const response = await apiRequester.get("/order-history");
    console.log("Order history data received:", response.data);
  } catch (error: unknown) {
    if (error instanceof AxiosError) {
      if (error.response) {
        const { status, data } = error.response;
        const { errorCode, errorMessage } = data;

        if (status === 401) {
          // UnauthorizedError
          throw new UnauthorizedError(errorMessage, error.response);
        } else {
          // OrderHttpError
          throw new OrderHttpError(errorMessage, error.response);
        }
      } else if (error.request) {
        // 요청 전송 후 응답이 없는 경우
        throw new NetworkError("No response received");
      } else {
        // 요청 전송 전에 에러가 발생한 경우
        throw new NetworkError("Request error");
      }
    } else {
      // Axios 에러가 아닌 경우
      throw error;
    }
  }
};
```
