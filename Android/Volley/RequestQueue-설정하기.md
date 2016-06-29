# Volley RequestQueue 설정하기

https://developer.android.com/training/volley/requestqueue.html

[[이전 강좌|간단한-요청-보내기]]에서는 volley의 기본동작을 활용하는 `RequestQueue`를 쉽게 설정할 수 있는 `Volley.newRequestQueue`에 대해 알아보았습니다. 이번에는 사용자정의 속성을 지원하는 `RequestQueue`를 만드는 단계에 대해 설명합니다.

또한, 앱의 라이프타임동안 유지되는 `RequestQueue`를 만들기 위해 싱글톤을 사용하는 방법도 연습해보겠습니다.

## 네트워크와 캐시 설정하기

`RequestQueue`는 request 전송을 수행하기 위한 network job과 캐시를 다룰 cache job을 필요로 합니다. Volley boolbox에는 이러한 것들이 기본 구현되어 있습니다. `DiskBasedCache`는 메모리에 색인을 가지는 response당 하나의 파일을 캐시로 제공하며, `BasicNetwork`는 기본적인 HTTP client에 기반한 네트워크 전송을 제공합니다.

`BasicNetwork`는 Volley의 기본 네트워크 구현입니다. `BasicNetwork`는  is Volley's default network implementation. BasicNetwork는 앱이 네트워크에 연결하는 데 사용하는 HTTP 클라이언트를 초기화해야하며, 이것은 보통 `HttpURLConnection`입니다.

다음의 코드는 `RequestQueue`를 설정하는 단계를 보여줍니다:

```java
RequestQueue mRequestQueue;

// 캐시를 인스턴스화
Cache cache = new DiskBasedCache(getCacheDir(), 1024 * 1024); // 1MB cap

// HttpURLConnection을 HTTP 클라이언트로 사용하여 네트워크를 설정
Network network = new BasicNetwork(new HurlStack());

// 캐시와 네트워크를 가지고 RequestQueue를 인스턴스화
mRequestQueue = new RequestQueue(cache, network);

// 큐 시작
mRequestQueue.start();

String url ="http://www.example.com";

// request를 만들고 그에 따른 response를 처리하는 코드작성
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
        new Response.Listener<String>() {
    @Override
    public void onResponse(String response) {
        // Do something with the response
    }
},
    new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            // Handle error
    }
});

// RequestQueue에 request를 추가
mRequestQueue.add(stringRequest);

// ...
```

한 번만 요청하고 스레드 풀을 그만 사용하고자 한다면, `RequestQueue`를 어디서든지 생성하고 response나 error을 받은 이후에 `RequestQueue`의 `stop()`을 호출하면 됩니다. 하지만, 일반적으로 `RequestQueue`는 앱의 생명주기동안 계속해서 싱글톤으로 실행되도록 생성하며 이 방법은 다음 절에서 설명하겠습니다.

## 싱글톤(Singleton) 패턴 사용하기

앱이 네트워크를 지속적으로 사용한다면 앱이 살아있는 동안 싱글톤으로 `RequestQueue`를 유지하는 것이 가장 효율적입니다. 방법은 여러가지가 있겠지만, 권장되는 방법은 `RequestQueue` 및 기타 volley의 기능을 캡슐화하는 싱글톤 클래스를 구현하는 것입니다. [Application](https://developer.android.com/reference/android/app/Application.html)의 서브클래스의 `Application.onCreate()`에서 `RequestQueue`를 설정하는 방법도 있지만, static 싱글톤이 같은 기능을 가지면서 더 모듈화된 방식이므로 application에서 정의하는 방법은 권장하지 않습니다.

핵심 개념은 `RequestQueue`가 Activity context가 아닌 Application context를 가지고 인스턴스화해야 한다는 것입니다. `RequestQueue`는 activity가 다시 생성될 때(예를 들어, 사용자가 화면을 회전하는 경우)마다 생기는게 아니라 앱의 생명주기와 동일하기 때문입니다.

다음은 `ImageLoader` 기능과 `RequestQueue`를 제공하는 싱글톤 클래스의 예입니다:

```
public class MySingleton {
    private static MySingleton mInstance;
    private RequestQueue mRequestQueue;
    private ImageLoader mImageLoader;
    private static Context mCtx;

    private MySingleton(Context context) {
        mCtx = context;
        mRequestQueue = getRequestQueue();

        mImageLoader = new ImageLoader(mRequestQueue,
                new ImageLoader.ImageCache() {
            private final LruCache<String, Bitmap>
                    cache = new LruCache<String, Bitmap>(20);

            @Override
            public Bitmap getBitmap(String url) {
                return cache.get(url);
            }

            @Override
            public void putBitmap(String url, Bitmap bitmap) {
                cache.put(url, bitmap);
            }
        });
    }

    public static synchronized MySingleton getInstance(Context context) {
        if (mInstance == null) {
            mInstance = new MySingleton(context);
        }
        return mInstance;
    }

    public RequestQueue getRequestQueue() {
        if (mRequestQueue == null) {
            // getApplicationContext()가 핵심입니다. 
            // Activity나 BroadcastReceiver를 사용하면 leak이 생길 수 있습니다.
            mRequestQueue = Volley.newRequestQueue(mCtx.getApplicationContext());
        }
        return mRequestQueue;
    }

    public <T> void addToRequestQueue(Request<T> req) {
        getRequestQueue().add(req);
    }

    public ImageLoader getImageLoader() {
        return mImageLoader;
    }
}
```

다음은 싱글톤 클래스를 사용하여 `RequestQueue` 작업을 수행하는 예입니다:

```java
// RequestQueue를 가져옴
RequestQueue queue = MySingleton.getInstance(this.getApplicationContext()).
    getRequestQueue();

// ...

// RequestQueue에 request(여기서는 stringRequest)를 추가
MySingleton.getInstance(this).addToRequestQueue(stringRequest);
```