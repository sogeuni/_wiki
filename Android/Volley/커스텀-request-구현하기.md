# Volley 커스텀 request 구현하기

https://developer.android.com/training/volley/request-custom.html

이 강좌에서는 Volley에서 기본적으로 지원하지않는 커스텀 request 타입을 구현하는 방법에 대해 설명합니다.

## 커스텀 request 작성하기

대부분의 request는 toolbox에 이미 구현되어 있습니다. string, image 혹은 JSON형태의 response를 받는다면 커스텀 request를 구현할 필요가 없습니다.

커스텀 request를 구현하려면 다음과 같은 작업이 필요합니다:

* **Request<T>** 클래스를 상속받는 클래스를 생성하세요. `<T>`는 request에 대해서 파싱된 response의 타입을 나타냅니다. 예를 들어, 파싱된 response가 string이라면 커스텀 request는 `Request<String>`을 상송받습니다. Volley toolbox의 `StringRequest`와 `ImageRequest` 클래스를 참고하세요.
* **parseNetworkResponse()**와 **deliverResponse()** 추상 메소드를 구현하세요. 이어서 좀 더 설명하겠습니다.

### parseNetworkResponse

Response는 전달을 위해 주어진 타입(string, image, JSON 등과 같은)으로 파싱하여 캡술화됩니다. 다음은 `parseNetworkResponse()`의 구현 예제입니다:

```java
@Override
protected Response<T> parseNetworkResponse(
        NetworkResponse response) {
    try {
        String json = new String(response.data,
        HttpHeaderParser.parseCharset(response.headers));
    return Response.success(gson.fromJson(json, clazz),
    HttpHeaderParser.parseCacheHeaders(response));
    }
    // handle errors
...
}
```

구현시에는 다음 사항에 유의하세요:

* `parseNetworkResponse()`는 인자로 `NetworkResponse`를 가집니다, `NetworkResponse`는 byte[] 형태의 페이로드와 HTTP 상태코드와 응답 헤더를 가지고 있습니다.
* 구현시에는 `Response<T>`를 반드시 리턴해야 합니다. 이것은 정형화된 response 오브젝트와 캐시 메타데이터 또는 파싱 실패의 경우와 같은 에러를 포함하고 있습니다.

만약 프로토콜이 표준적이지 않은 캐시형태를 가진다면 `Cache.Entry`를 따로 만들수 있지만, 대부분의 request는 다음과 같이 사용하면 됩니다:

```java
return Response.success(myDecodedObject,
        HttpHeaderParser.parseCacheHeaders(response));
```

Volley는 워커 스레드로 부터 `parseNetworkResponse()`를 호출합니다. 이것은 JPEG를 Bitmap으로 디코딩하는 것과 같이 부하가 많이 걸리는 작업이 UI 스레드를 멈추지 않도록 합니다.

### deliverResponse

Volley는 `parseNetworkResponse()`의 응답을 다시 메인 스레드로 전달합니다. 대부분의 request는 다음과 같은 콜백 인터페이스를 호출합니다:

```java
protected void deliverResponse(T response) {
        listener.onResponse(response);
}
```

### 예제: GsonRequest

[Gson](http://code.google.com/p/google-gson/)은 reflection을 이용하여 JSON을 Java 객체로 변환해주는 라이브러리 입니다. JSON의 키와 같은 이름을 가진 Java 객체를 정의한 후 Gson에 그것의 클래스 객체를 전달하면 Gson은 자동으로 필드를 채웁니다. 다음의 예제는 파싱을 위해 Gson을 사용하는 Volley request의 예제입니다:

```java
public class GsonRequest<T> extends Request<T> {
    private final Gson gson = new Gson();
    private final Class<T> clazz;
    private final Map<String, String> headers;
    private final Listener<T> listener;

    /**
     * GET request를 생성하고 JSON으로 부터 파싱된 결과를 리턴
     *
     * @param url request의 URL
     * @param clazz Gson reflection을 위한 관련 클래스 객체
     * @param headers request 헤더 맵
     */
    public GsonRequest(String url, Class<T> clazz, Map<String, String> headers,
            Listener<T> listener, ErrorListener errorListener) {
        super(Method.GET, url, errorListener);
        this.clazz = clazz;
        this.headers = headers;
        this.listener = listener;
    }

    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        return headers != null ? headers : super.getHeaders();
    }

    @Override
    protected void deliverResponse(T response) {
        listener.onResponse(response);
    }

    @Override
    protected Response<T> parseNetworkResponse(NetworkResponse response) {
        try {
            String json = new String(
                    response.data,
                    HttpHeaderParser.parseCharset(response.headers));
            return Response.success(
                    gson.fromJson(json, clazz),
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        }
    }
}
```

Volley는 즉시 사용 가능한 `JsonArrayRequest`와 `JsonArrayObject` 클래스를 제공합니다. [[표준 request 만들기]]를 참고하세요.