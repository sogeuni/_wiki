# 안드로이드 WebView를 이용한 웹앱 만들기

원문: http://developer.android.com/guide/webapps/webview.html

안드로이드에서 웹 어플리케이션(혹은 단순한 웹페이지)을 클라이언트 어플리케이션으로 제공하기를 원한다면 [WebView](http://developer.android.com/reference/android/webkit/WebView.html)를 사용할 수 있습니다. `WebView` 클래스는 안드로이드 View 클래스의 확장으로 Activity 레이아웃의 일부에 웹 페이지를 표시할 수 있도록 합니다. WebView는 네비게이션 컨트롤이나 주소창과 같은 웹 브라우저의 모든 기능을 포함하고 있지는 않습니다. WebView가 하는 기본적인 일은 웹 페이지를 보여주는 것입니다.

WebView를 사용하는 것이 유용한 일반적인 시나리오는 최종 사용자 계약 또는 사용자 설명서 등과 같이 어플리케이션에서 업데이트가 필요할 수 있는 정보를 제공하는 경우 입니다. 안드로이드 어플리케이션 내에서 WebView를 가지는 Activity를 만들 수 있고, 그런다음 온라인에 호스팅된 문서를 표시할 수 있습니다.

WebView가 유용한 또 다른 시나리오는 이메일과 같이 데이터를 제공하기 위해 항상 인터넷 연결이 필요한 경우입니다. 이러한 경우는 네트워크 요청의 결과를 파싱하여 안드로이드 레이아웃에 그리는 것보다 데이터를 가지는 전체 웹 페이지를 보여주는 것이 효율적일 수 있습니다. 대신에 WebView에 로딩될 웹페이지는 안드로이드 단말에 적합하게 디자인 할 필요가 있습니다.

## 어플리케이션에 WebView 추가하기

어플리케이션에 WebView를 추가하기 위해서는 간단하게 activity 레이아웃에 `<WebView>` 엘리먼트를 추가하면 됩니다. 예를 들어, 화면에 가득차는 WebView는 다음과 같이 정의합니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<WebView  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/webview"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
/>
```

WebView에 웹페이지를 로딩하기 위해서는 다음과 같이 `loadUrl()`을 사용합니다.

```java
WebView myWebView = (WebView) findViewById(R.id.webview);
myWebView.loadUrl("http://www.example.com");
```

이 작업을 하기전에 어플리케이션은 인터넷에 접속할 수 있어야 합니다. 인터넷에 접속하기 위해서 다음과 같이 manifest 파일에서 **INTERNET** 퍼미션을 요청합니다.

```xml
<manifest ... >
    <uses-permission android:name="android.permission.INTERNET" />
    ...
</manifest>
```

WebView에 웹페이지를 표시하기 위해 필요한 기본적인 동작은 이게 전부입니다.

## WebView에서 JavaScript 사용하기

WebView에서 JavaScript를 사용하려면 자바스크립트를 활성화하여야 합니다. 자바스크립트가 활성화되면 어플리케이션 코드(네이티브 코드)와 자바스크립트 코드 사이의 인터페이스를 만들 수도 있습니다.

### JavaScript 활성화하기

WebView는 기본적으로 자바스크립트의 사용이 비활성화 되어 있으며 WebView에 연결된 [WebSettings](http://developer.android.com/reference/android/webkit/WebSettings.html)를 통해서 활성화 할 수 있습니다. `getSettings()`를 통해 WebSettings를 가져올 수 있고 [[안드로이드_WebView_설정을_위한_WebSettings_옵션_정리#setJavaScriptEnabled|setJavaScriptEnabled()]]로 자바스크립트를 활성화 할 수 있습니다.

```java
WebView myWebView = (WebView) findViewById(R.id.webview);
WebSettings webSettings = myWebView.getSettings();
webSettings.setJavaScriptEnabled(true);
```

WebSettings는 WebView의 다양한 설정을 제공합니다. 예를 들어, 안드로이드 어플리케이션에서 WebView를 위해 특별히 디자인된 웹어플리케이션을 개발한다면 [[안드로이드_WebView_설정을_위한_WebSettings_옵션_정리#setUserAgentString|setUserAgentString()]]을 통해 커스텀 유저 에이전트를 정의할 수 있고, 그런 다음 엡페이지가 실제로 안드로이드 어플리케이션에서 호출되었는지 확인하기 위해 웹페이지에서 커스텀 유저 에이전트를 조회할 수 있습니다.

### 안드로이드 코드와 자바스크립트 코드의 연결

WebView를 이용하여 안드로이드에 특화된 웹 어플리케이션을 개발할 때, 안드로이드 코드와 자바스크립트 코드를 연결하는 인터페이스를 만들수 있습니다. 예를 들어, 자바스크립트 코드에서 자바스크립트의 ```alert()```를 호출하는 대신에 안드로이드 코드의 ```Dialog```를 호출할 수 있습니다.

자바스크립트와 안드로이드 코드사이의 연결을 위해서는 ```addJavascriptInterface()```를 호출하여, 자바스크립트에 연결되는 인스턴스의 클래스와 클래스에 연결하기위한 자바스크립트 인터페이스 이름을 넘겨줍니다.

다음 예제와 같이 안드로이드 어플리케이션에서 다음과 같은 클래스를 포함할 수 있습니다.

```java
public class WebAppInterface {
    Context mContext;

    /** Instantiate the interface and set the context */
    WebAppInterface(Context c) {
        mContext = c;
    }

    /** Show a toast from the web page */
    @JavascriptInterface
    public void showToast(String toast) {
        Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show();
    }
}
```

> **주의**: `targetSdkVersion`이 17 이상이라면 자바스크립트에서 사용하기를 원하는 메소드에는 **반드시 @JavascriptInterface 어노테이션을 붙여야 합니다**(또한 메소드는 반드시 public이어야 함). 만약 이 어노테이션을 붙이지 않는 경우, 안드로이드 4.2 이상에서 돌아가는 웹페이지에서는 이 메소드에 접근할 수 없습니다.

위의 예제에서 `WebAppInterface` 클래스는 웹페이지에서 `showToast()` 메소드를 통해 Toast메시지를 출력할 수 있도록 합니다.

다음의 예제처럼 WebView의 addJavascriptInterface()를 사용하여 클래스를 연결할 때 인터페이스의 이름을 '''Android'''로 지정할 수 있습니다.

```java
WebView webView = (WebView) findViewById(R.id.webview);
webView.addJavascriptInterface(new WebAppInterface(this), "Android");
```

이것은 WebView에서 자바스크립트를 실행하는 동안 Android라 불리는 객체를 생성하게 되고, 이 시점에서 웹어플리케이션은 WebAppInterface 클래스에 접근할 수 있게 됩니다. 다음 예제는 버튼 클릭 시 새로운 인터페이스를 통해 toast 메시지를 생성하는 HTML과 자바스크립트입니다.

```javascript
<input type="button" value="Say hello" onClick="showAndroidToast('Hello Android!')" />

<script type="text/javascript">
    function showAndroidToast(toast) {
        Android.showToast(toast);
    }
</script>
```

자바스크립트에서 Android 인터페이스를 초기화할 필요는 없습니다. WebView가 자동으로 웹페이지에서 사용가능하도록 만들어 주기 때문에 버튼을 클릭하면 showAndroidToast() 함수는 WebAppInterface.showToast() 메소드를 호출하는 Android 인터페이스를 사용합니다.

> **참고:** 자바스크립트에 연결된 객체는 그것이 생성된 스레드와 자바스크립트가 실행되는 스레드와는 다른 스레드에서 동작합니다.

> **주의:** addJavascriptInterface()를 사용하는 것은 안드로이드 어플리케이션의 자바스크립트 사용을 허용하는 것입니다. 이것은 아주 유용하긴 하지만, 위험한 보안 이슈를 가지기도 하는 기능입니다. WebView에서 사용하는 HTML을 믿지 못하는 경우(예를 들어, 모르는 사람이나 프로세스가 제공하는 HTML) 공격자가 임의의 클라이언트 코드를 실행할 수 있는 HTML을 삽입하여 실행할 수도 있습니다. 따라서 WebView에 표시되는 HTML과 자바스크립트를 모두 당신이 작성하지 않았다면 addJavascriptInterface()를 사용하지 마세요. 또한 사용자가 WebView내에서 당신의 웹페이지가 아닌 다른 페이지를 탐색하지 못하도록 하세요(대신에 외부링크에 대해서 기본브라우져가 실행되도록 하세요. 기본적으로 URL링크는 사용자의 브라우져로 열리게 되어 있으므로 다음 단락에서 설명하는 페이지 네비게이션을 처리할 때는 주의를 기울이시기 바랍니다).

## 페이지 네비게이션 다루기

WebView내의 웹페이지의 링크를 클릭할 때의 기본적인 동작은, 해당 URL을 처리할 수 있는 안드로이드 어플리케이션이 실행되는 것입니다. 일반적으로, 기본 웹브라우저가 목적 URL을 열고 로딩하게 됩니다만, WebView의 동작을 오버라이드해서 WebView에서 링크를 열 수도 있습니다. 그런 다음 WebView에서 관리되는 히스토리를 통해 사용자가 뒤로 혹은 앞으로 탐색할 수 있도록 합니다.

사용자가 클릭한 링크를 열기 위해 `setWebViewClient()`를 사용하여 다음과 같이 WebView를 위한 WebViewClient를 간단히 구현합니다.

```java
WebView myWebView = (WebView) findViewById(R.id.webview);
myWebView.setWebViewClient(new WebViewClient());
```

이것으로 클릭한 모든 링크는 WebView에서 로드됩니다.

만약 링크가 로드될 때 세부 제어를 하기를 원한다면 아래와 같이 자신의 WebViewClient를 만들고 `shouldOverrideUrlLoading()` 메소드를 오버라이드 할 수 있습니다.

```java
private class MyWebViewClient extends WebViewClient {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if (Uri.parse(url).getHost().equals("www.l2r2.net")) {
            // This is my web site, so do not override; let my WebView load the page
            return false;
        }
        // Otherwise, the link is not for a page on my site, so launch another Activity that handles URLs
        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
        startActivity(intent);
        return true;
    }
}
```

그런다음 WebView를 위햔 새로운 WebViewClient의 인스턴스를 생성합니다.

```java
WebView myWebView = (WebView) findViewById(R.id.webview);
myWebView.setWebViewClient(new MyWebViewClient());
```

이제 사용자가 링크를 클릭하면 시스템은 URL 호스트가 특정 도메인과 일치하는지를 확인하는 `shouldOverrideUrlLoading()`을 호출합니다. 만약 일치한다면 URL 로딩을 오버로드 하지않으며 false를 리턴합니다(WebView가 URL을 로딩합니다). 만약 일치하지 않는다면 URL 처리를 위한 기본 Activity를 실행하는 Intent를 생성합니다(기본 웹브라우져가 확인).

### 웹페이지 히스토리 탐색하기

WebView가 URL 로딩을 오버라이드하면 자동적으로 WebView는 방문한 웹페이지의 히스토리를 관리하게 됩니다. `goBack()`과 `goForward()` 메소드를 통해 히스토리에서 전후로 탐색이 가능하게 됩니다.

Activity에서 장치의 **Back** 버튼을 사용하여 뒤로 탐색하게 하려면 다음과 같이 하면 됩니다.

```java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    // Check if the key event was the Back button and if there's history
    if ((keyCode == KeyEvent.KEYCODE_BACK) && myWebView.canGoBack()) {
        myWebView.goBack();
        return true;
    }
    // If it wasn't the Back key or there's no web page history, bubble up to the default
    // system behavior (probably exit the activity)
    return super.onKeyDown(keyCode, event);
}
```

사용자가 방문한 웹페이지가 히스토리에 남아 있을 경우 `canGoBack()` 메소드는 true를 리턴합니다. 마찬가지로 forward 히스토리가 있는지 체크하기 위해 `canGoForward()` 메소드를 사용할 수 있습니다. 만약 이러한 체크를 하지 않는다면 히스토리의 끝에 도착한 경우 `goBack()` 또는 `goForward()` 메소드는 아무런 동작도 수행하지 않습니다.
