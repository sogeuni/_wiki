# Volley 표준 request 만들기

https://developer.android.com/training/volley/request.html

이번 강좌에서는 Volley에서 기본적으로 지원하는 request 타입을 사용하는 방법에 대해 설명합니다:

* **StringRequest**는 특정 URL의 raw string형태의 응답을 받습니다. [[RequestQueue-설정하기]]의 예제를 참고하세요.
* **ImageRequest**는 특정 URL의 image를 응답으로 받습니다.
* **JsonObjectRequest**와 **JsonArrayRequest**(둘다 **JsonRequest**의 서브클래스)는 각각 특정 URL의 JSON 객체나 배열을 응답으로 받습니다.

예상되는 response가 이들 타입중 하나라면 사용자 정의 request를 구현할 필요가 없습니다. 이번 강좌에서는 이러한 표준 request 타입을 사용하는 방법에 대해 설명합니다. 사용자 정의 request를 구현하는 방법은 [[커스텀 request 구현하기]]를 참고하세요.

## 이미지 request하기

Volley는 이미지를 요청하기 위해 다음과 같은 클래스를 제공합니다. 이 클래스들은 각자 이미지를 처리하기 위한 탑레이어 클래스이며 각기 다른 레벨을 제공합니다.

* **ImageRequest** — 특정 URL에서 디코팅된 비트맵을 콜백으로 얻는 request. 리사이즈와 같은 편의 기능을 제공합니다. Volley의 스레드 스케쥴링이 비용이 많이 드는 이미지 작업(디코딩, 리사이징)을 자동으로 워커 스레드에서 동작하도록 보장해준다는 장점이 있습니다.
* **ImageLoader** — 원격 URL의 이미지를 로딩하거나 캐시하는 핼퍼 클래스. `ImageLoader`는 많은 수의 `ImageRequest`를 처리하는 조정자(orchestrator)입니다. 예를 들어, [ListView](https://developer.android.com/reference/android/widget/ListView.html)에서 여러개의 썸네일을 보여주는 경우 `ImageLoader`는 플리커링을 막기 위해 Volley 캐시 앞에다 메모리 캐시를 둡니다. 이것은 메인 스레드가 블로킹되거나 지연되지 않고 캐시 히트(cache hit)할 수 있게 합니다. 또한 `ImageLoader`는 이미지 response 핸들러를 병합하여 여러 응답을 동시에 처리하여 성능을 향상시킵니다.
* **NetworkImageView** — `ImageLoader` 내장된 네트워크 이미지를 표시하기 위한 `ImageView`의 효과적인 대체 view. `NetworkImageView`가 계층 구조에서 분리되면 처리되지 않은 request는 취소됩니다.

### ImageRequest 사용하기

다음은 `ImageRequest`를 사용하여 특정 URL의 이미지를 가져와 앱에 보여주는 예제입니다. 여기서는 싱글톤 클래스를 통해 `RequestQueue`에 접근합니다. ([[RequestQueue 설정하기]] 참조)

```java
ImageView mImageView;
String url = "http://i.imgur.com/7spzG.png";
mImageView = (ImageView) findViewById(R.id.myImage);
...

// 특정 URL의 이미지를 가져와 UI에 표시
ImageRequest request = new ImageRequest(url,
    new Response.Listener<Bitmap>() {
        @Override
        public void onResponse(Bitmap bitmap) {
            mImageView.setImageBitmap(bitmap);
        }
    }, 0, 0, null,
    new Response.ErrorListener() {
        public void onErrorResponse(VolleyError error) {
            mImageView.setImageResource(R.drawable.image_load_error);
        }
    });
// 싱글톤 클래스를 통해 RequestQueue에 접근
MySingleton.getInstance(this).addToRequestQueue(request);
```

### ImageLoader와 NetworkImageView 사용하기

`ListView` 등에서 여러 이미지를 디스플레이하는 것을 효과적으로 관리하기 위해 `ImageLoader`와 `NetworkImageView`를 함께 사용할 수 있습니다. 레이아웃 XML에 `ImageView`를 사용하는 것과 동일하게 `NetworkImageView`를 사용합니다.

```xml
<com.android.volley.toolbox.NetworkImageView
        android:id="@+id/networkImageView"
        android:layout_width="150dp"
        android:layout_height="170dp"
        android:layout_centerHorizontal="true" />
```

이미지를 표시하기 위해 다음처럼 `ImageLoader`만을 사용할 수 있습니다.

```java
ImageLoader mImageLoader;
ImageView mImageView;
// 로드할 이미지 URL
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
...
mImageView = (ImageView) findViewById(R.id.regularImageView);

// 싱글톤 클래스를 통해 ImageLoader를 가져옴
mImageLoader = MySingleton.getInstance(this).getImageLoader();
mImageLoader.get(IMAGE_URL, ImageLoader.getImageListener(mImageView,
         R.drawable.def_image, R.drawable.err_image));
```

하지만, `networkImageView`는 간단히 네트워크 이미지를 그릴 수 있습니다.

```java
ImageLoader mImageLoader;
NetworkImageView mNetworkImageView;
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
...

// 이미지를 표시할 NetworkImageView를 얻어옴
mNetworkImageView = (NetworkImageView) findViewById(R.id.networkImageView);

// 싱글톤 클래스를 통해 ImageLoader를 가져옴
mImageLoader = MySingleton.getInstance(this).getImageLoader();

// view에 로드할 이미지의 URL과 request를 처리할 ImageLoader를 지정
mNetworkImageView.setImageUrl(IMAGE_URL, mImageLoader);
```

위의 코드는 [[RequestQueue 설정하기|RequestQueue-설정하기#싱글톤-singleton-패턴-사용하기]]에서 설명한 싱글톤 클래스를 통해 `RequestQueue`와 `ImageLoader`에 접근합니다. 따라서, 앱이 실행되는 동안 하나의 인스턴스만 생성됩니다. `ImageLoader`의 비트맵 캐시는 activity 외부에 생성됩니다. 만약 activity에 `ImageLoader`가 생성되면 activity가 재생성될때 마다 `ImageLoader`도 새로 생성되어 플리커링이 발생할 수 있습니다.

#### LRU 캐시 예제

Volley toolbox는 `DiskBasedCache` 클래스를 통해 캐시를 구현합니다. 이 클래스는 하드디스크의 특정 디렉토리에 파일로 캐시를 저장합니다. 하지만 `ImageLoader`를 사용하려면 `ImageLoader.ImageCache` 인터페이스를 구현하여 메모리 LRU 캐시를 제공해야 합니다. 캐시를 싱글톤으로 구현하고자 한다면 [[RequestQueue 설정하기|RequestQueue-설정하기#싱글톤-singleton-패턴-사용하기]]를 참고하세요.

다음은 `LruBitmapCache`를 구현하는 예제입니다. [LruCache](https://developer.android.com/reference/android/support/v4/util/LruCache.html) 클래스를 상속받고 `ImageLoader.ImageCache` 인터페이스를 구현합니다.

```java
import android.graphics.Bitmap;
import android.support.v4.util.LruCache;
import android.util.DisplayMetrics;
import com.android.volley.toolbox.ImageLoader.ImageCache;

public class LruBitmapCache extends LruCache<String, Bitmap>
        implements ImageCache {

    public LruBitmapCache(int maxSize) {
        super(maxSize);
    }

    public LruBitmapCache(Context ctx) {
        this(getCacheSize(ctx));
    }

    @Override
    protected int sizeOf(String key, Bitmap value) {
        return value.getRowBytes() * value.getHeight();
    }

    @Override
    public Bitmap getBitmap(String url) {
        return get(url);
    }

    @Override
    public void putBitmap(String url, Bitmap bitmap) {
        put(url, bitmap);
    }

    // 화면에 표시될 이미지의 크기의 3배 크기의 캐시 사이즈를 리턴
    public static int getCacheSize(Context ctx) {
        final DisplayMetrics displayMetrics = ctx.getResources().
                getDisplayMetrics();
        final int screenWidth = displayMetrics.widthPixels;
        final int screenHeight = displayMetrics.heightPixels;
        // 4 bytes per pixel
        final int screenBytes = screenWidth * screenHeight * 4;

        return screenBytes * 3;
    }
}
```

이제 `ImageLoader`에서 캐시를 사용하도록 하여 초기화합니다.

```java
RequestQueue mRequestQueue; // assume this exists.
ImageLoader mImageLoader = new ImageLoader(mRequestQueue, new LruBitmapCache(
            LruBitmapCache.getCacheSize()));
```

## JSON request하기

Volley는 JSON request를 위해 다음과 같은 클래스를 제공합니다:

* **JsonArrayRequest** — 특정 URL에서 response의 body로 [JSONArray](https://developer.android.com/reference/org/json/JSONArray.html)를 받음
* **JsonObjectRequest** — 특정 URL에서 response의 body로 [JSONObject](https://developer.android.com/reference/org/json/JSONObject.html)를 받음. request의 body에 JSONObject를 실어 전달할 수 있음

두 클래스 모두 `JsonRequest` 클래스의 서브클래스입니다. 다음의 기본 패턴으로 각 request를 사용할 수 있습니다. 다음은 JSON 피드를 가져와 UI에 텍스트 형태로 보여주는 예제입니다:

```java
TextView mTxtDisplay;
ImageView mImageView;
mTxtDisplay = (TextView) findViewById(R.id.txtDisplay);
String url = "http://my-json-feed";

JsonObjectRequest jsObjRequest = new JsonObjectRequest
        (Request.Method.GET, url, null, new Response.Listener<JSONObject>() {

    @Override
    public void onResponse(JSONObject response) {
        mTxtDisplay.setText("Response: " + response.toString());
    }
}, new Response.ErrorListener() {

    @Override
    public void onErrorResponse(VolleyError error) {
        // TODO Auto-generated method stub

    }
});

// 싱글톤 클래스를 통해 RequestQueue에 접근
MySingleton.getInstance(this).addToRequestQueue(jsObjRequest);
```

[Gson](http://code.google.com/p/google-gson/)을 이용한 커스텀 JSON request를 구현하는 예제는 [[커스텀 request 구현하기]]를 참고하세요.