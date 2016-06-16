# 안드로이드 WebView 설정을 위한 WebSettings 옵션 정리

안드로이드 WebView의 설정을 관리하기 위해서는 **WebSettings** 객체를 사용합니다. WebView가 생성되었을 때 WebSettings는 기본값을 가지며 이 기본값들은 getter 호출로 가져올 수 있습니다. `WebView.getSettings()`로 얻어오는 WebSettings 객체는 WebView의 생명주기와 묶여있기 때문에 WebView가 소멸된 이후의 WebSettings 의 메소드를 호출한다면 **IllegalStateException**이 발생됩니다.

이제 WebView의 설정을 변경할 수 있는 WebSettings의 메소드들에 대해 알아보도록 하겠습니다.[^ver]

## 보안관련 설정

### setAllowContentAccess

```java
void setAllowContentAccess (boolean allow)
```

WebView에서 content URL을 접근할지의 여부를 결정합니다. 이 값이 true 인경우 WebView는 시스템에 설치된 content provider로부터 컨텐츠를 로드할 수 있습니다. 기본값은 **true** 입니다.

### setAllowFileAccess

```java
void setAllowFileAccess (boolean allow)
```

WebView의 파일접근을 허가할지의 여부를 결정합니다. 파일접근은 기본적으로 활성화 되어 있습니다. 단 이것은 파일 시스템의 엑세스 권한만을 의미하며, 이 값이 비활성화 되어 있더라도 asset과 resources 파일은 `file:///android_asset` 과 `file:///android_res` 의 형태로 접근이 가능하다는 것에 유의하시기 바랍니다.

### setAllowFileAccessFromFileURLs

```java
void setAllowFileAccessFromFileURLs (boolean flag)
```

file 스킴의 URL의 컨텍스트 내에서 자바스크립트에 접근할 수 있는지를 결정합니다. 보안을 위해서는 이 설정을 비활성화하여야 합니다. 

[getAllowUniversalAccessFromFileURLs()](#getAllowUniversalAccessFromFileURLs) 값이 **true** 일 경우 이 값은 무시됩니다. 또한 이 설정은 file 스킴 리소스로 자바스크립트에 접근할 때만 효과가 나타납니다. 예를들어 이미지 HTML 엘리먼트와 같은 다른 리소스에 접근할때는 효과가 나타나지 않습니다. ICE_CREAM_SANDWICH 이전의 장치에서 same domain policy[^same-origin_policy] 위반을 막기 위해서는 명시적으로 이 값을 false로 설정하여야 합니다.

[ICE_CREAM_SANDWICH_MR1](http://developer.android.com/intl/ko/reference/android/os/Build.VERSION_CODES.html#ICE_CREAM_SANDWICH_MR1) 이하의 API 레벨에서는 기본값이 **true** 이며 [JELLY_BEAN](http://developer.android.com/intl/ko/reference/android/os/Build.VERSION_CODES.html#JELLY_BEAN) 이상은 **false** 입니다.

### setAllowUniversalAccessFromFileURLs

```java
void setAllowUniversalAccessFromFileURLs (boolean flag)
```

위의 [setAllowFileAccessFromFileURLs](#setAllowFileAccessFromFileURLs) 와 비슷하지만 다른 도메인의 경우에도 허용하는 점이 다릅니다. 역시나 ICE_CREAM_SANDWICH_MR1 이하에서는 **true**로 설정되어 있으며, JELLY_BEAN 이후는 **false**로 설정되어 있습니다.

### setBlockNetworkImage

```java
void setBlockNetworkImage (boolean flag)
```

네트워크를 통한(http와 https URI 스킴을 통해 접근하는) 이미지 리소스를 로딩할지의 여부를 결정합니다.(**true**일 경우 이미지로딩 금지)

[getLoadsImagesAutomatically()](#getLoadsImagesAutomatically)의 결과값이 **true**일 경우 이 설정은 효과가 없습니다. 또한 [setBlockNetworkLoads(boolean)](#setBlockNetworkLoads)을 사용하여 모든 네트워크 로드를 막았을 경우에는 이 값이 **false**일지라도 네트워크 이미지는 로딩되지 않습니다. 이 값이 **true**에서 **false**로 바뀌면 WebView의 컨텐츠의 네트워크 이미지는 자동으로 로딩됩니다.

이 설정의 기본값은 **false**(네트워크 이미지 사용가능)입니다.

### setBlockNetworkLoads

```java
void setBlockNetworkLoads (boolean flag)
```

네트워크를 통한 이미지를 포함한 모든 리소스의 로딩여부를 결정합니다. 이 값이 **true**에서 **false**로 바뀌면 [setBlockNetworkImage](#setBlockNetworkImage) 과는 다르게 `reload()`가 호출될 때 까지 리소스가 로딩되지 않습니다. 어플리케이션이 **INTERNET** 퍼미션을 가지지 않는 경우 이 값을 **false**로 설정하면 `SecurityException`이 발생됩니다.

이 설정의 기본값은 **INTERNET** 퍼미션이 설정되어 있는 경우 **false**, 아닌 경우에는 **true** 입니다.

### setMixedContentMode

```java
void setMixedContentMode (int mode)
```

secure orgin이 insecure origin의 리소스를 로드할 때의 WebView의 동작방식을 설정합니다. 기본적으로 KITKAT 이전의 단말은 `MIXED_CONTENT_ALWAYS_ALLOW` 로 동작하며, LOLLIPOP 부터는 `MIXED_CONTENT_NEVER_ALLOW`가 기본값입니다. 가장 적절하고 안전한 모드는 `MIXED_CONTENT_NEVER_ALLOW` 이며 `MIXED_CONTENT_ALWAYS_ALLOW`는 권장하지 않습니다.

설정할 수 있는 모드들은 다음과 같습니다.

* **MIXED_CONTENT_NEVER_ALLOW**: secure origin에서 insecure origin의 컨텐츠를 로드하는 것이 허용되지 않습니다. 가장 안전한 모드입니다.
* **MIXED_CONTENT_ALWAYS_ALLOW**: secure origin이 어떤 origin(insecure origin 일지라도) 에서든 컨텐츠를 로드하는 것을 허용합니다. 이것은 최소한의 보안 모드이므로 가능하면 이 모드를 사용하지 않는 것이 좋습니다.
* **MIXED_CONTENT_COMPATIBILITY_MODE**: 최근의 일반적인 웹 브라우저의 모드와 호환되도록 동작하는 모드입니다. 특정 insecure 컨텐츠는 허용되며, 다른 타입의 컨텐츠는 블록됩니다. 어떤 컨텐츠를 허용하고 어떤 컨텐츠를 막을지는 릴리즈 시 마다 변경될 수 있으며 명시적으로 정의되어 있지 않습니다.

## Javascript 관련 설정

### setJavaScriptEnabled

```java
void setJavaScriptEnabled (boolean flag)
```

자바스크립트 실행여부를 결정합니다. 기본값은 **false**입니다.

### setJavaScriptCanOpenWindowsAutomatically

```java
void setJavaScriptCanOpenWindowsAutomatically (boolean flag)
```

자바스크립트 함수 `window.open()`을 허용할지를 결정합니다. 기본값은 **false**입니다.

## 데이터 저장 관련 설정

### setAppCacheEnabled

```java
void setAppCacheEnabled (boolean flag)
```

어플리케이션 캐시 API의 활성화 여부를 결정합니다. 기본값은 **false** 이며, 어플리케이션 캐시를 활성화 하기 위해서는 [setAppCachePath(String)](#setAppCachePath) 로 적절한 database 경로도 설정하여야 합니다.

### setAppCachePath

```java
void setAppCachePath (String appCachePath)
```

어플리케이션 캐시파일의 경로를 설정합니다. 어플리케이션 캐시 API를 활성화하려면 이 메소드를 통해 어플리케이션이 저장할 수 있는 경로를 반드시 지정하여야 합니다.

이 메소드는 한번만 호출하여야 하며 이후의 호출은 무시됩니다.

### setCacheMode

```java
void setCacheMode (int mode)
```

캐시가 사용되는 방법을 재정의 합니다. 캐시가 사용되는 방법은 탐색타입에 따라 결정됩니다. 일반 페이지 로드의 경우 캐시를 체크하고 필요한 컨텐츠를 재확인합니다. 뒤로 이동하는 경우 컨텐츠를 다시 검증하는 대신 캐시에서 읽어들입니다. 이 메소드는 클라이언트가 다음 한가지의 방식으로 데이터를 로딩하는 방식을 결정합니다.

기본값은 **LOAD_DEFAULT**입니다.

* **LOAD_DEFAULT**: 기본 캐시 사용모드입니다. 캐시가 만료되지 않았다면 캐시에서 리소스를 읽고 그렇지 않다면 네트워크에서 로드합니다.
* **LOAD_CACHE_ELSE_NETWORK**: 캐시에 저장된 내용이 만료되었더라도 캐시에서 리소스를 읽습니다. 저장된 내용이 없을 경우 네트워크에서 로드합니다.
* **LOAD_NO_CACHE**: 캐시를 사용하지 않고 네트워크에서 리소스를 로드합니다.
* **LOAD_CACHE_ONLY***: 네트워크를 사용하지 않고 캐시에서 데이터를 로드합니다.

### setDatabaseEnabled

```java
void setDatabaseEnabled (boolean flag)
```

데이터베이스 API를 활성화할지를 결정하며 기본값은 **false** 입니다. 이 설정은 프로세스내의 모든 WebView에 전역으로 적용됩니다. 프로세스내에서 만들어진 WebView가 페이지를 로드하기 전에 이 설정을 수정하여야 하며 이후의 변경은 무시됩니다.

### setDomStorageEnabled

```java
void setDomStorageEnabled (boolean flag)
```

HTML5의 DOM 스토리지 API의 사용여부를 결정합니다. 기본값은 **false**입니다.

### setSaveFormData

```java
void setSaveFormData (boolean save)
```

form 데이터를 저장할 수 있도록 설정합니다. 기본값은 **true**입니다.

## 컨텐츠 표시 관련 설정

### setLoadsImagesAutomatically

```java
void setLoadsImagesAutomatically (boolean flag)
```

WebView가 이미지를 자동으로 로드할지를 결정합니다. 이 메소드는 데이터 URI 스킴을 사용하는 내장된 이미지를 포함합니다. [setBlockNetworkImage(boolean)](#setBlockNetworkImage)을 사용하면 네트워크 URI를 사용하는 이미지만 컨트롤 합니다. 이 값이 **false**에서 **true**로 바뀌면 WebView는 리프레시 없이 자동으로 모든 이미지를 로드합니다. 기본값은 **true**입니다.

### setLayoutAlgorithm

```java
void setLayoutAlgorithm (WebSettings.LayoutAlgorithm l)
```

기본 레이아웃 알고리즘을 설정합니다. 기본값은 **NARROW_COLUMNS** 이며 다음중 하나의 값을 설정합니다.

* **NORMAL**: 렌더링을 변경하지 않습니다. 호환성을 위해 추천하는 옵션입니다.
* **SINGLE_COLUMN**: view 너비만큼의 하나의 컬럼으로 모든 컨텐츠를 이동합니다.
* **NARROW_COLUMNS**: 가능한한 모든 컬럼을 화면너비보다 넓게 하지 않습니다. KITKAT이전의 API 레벨에서만 사용합니다.
* **TEXT_AUTOSIZING** : Overview 모드의 wide-viewport 레이아웃에서 휴리스틱한 방법에 의거 글자를 키웁니다. 이 모드를 사용할 때에는 [setSupportZoom(boolean)](#setSupportZoom)으로 zoom을 활성화하는 것을 추천합니다. KITKAT부터 지원합니다.

### setLoadWithOverviewMode

```java
void setLoadWithOverviewMode (boolean overview)
```

화면 너비에 맞도록 컨턴츠를 축소하는 오버뷰 모드로 페이지를 열지를 결정합니다. WebView의 폭보다 컨텐츠의 폭이 넓은경우 이 설정의 영향을 받습니다. 기본값은 **false**입니다.

### setOffscreenPreRaster

```java
void setOffscreenPreRaster (boolean enabled)
```

WebView가 윈도우에 붙어있지만 오프스크린일 때 화면을 그릴지를 설정합니다. 이 설정을 켜면 오프스크린에서 온스크린으로 애니메이션 될 때 화면이 껌벅이는 것(rendering artifacts)을 막을 수 있습니다. 이 모드에서는 오프스크린 WebView는 메모리를 더 많이 사용하게 되며 따라서 이 값은 **false** 입니다.

메모리 사용을 제한하기 위해 다음의 지침을 따르세요.

* WebView의 크기은 단말의 화면크기보다 크지 않아야 합니다.
* 이 모드를 사용하는 WebView의 갯수를 최소한으로 유지하기 바랍니다.

### setSupportMultipleWindows

```java
void setSupportMultipleWindows (boolean support)
```

WebView이 multiple window 지원여부를 설정합니다. 이값이 **true**라면 어플리케이션에 `onCreateWindow(WebView, boolean, boolean, Message)`를 반드시 구현하여야 합니다. 기본값은 **false** 입니다.

### setUseWideViewPort

```java
void setUseWideViewPort (boolean use)
```

WebView에서 "viewport" HTML 메타태그 혹은 wide viewport를 지원할지를 결정합니다. 이 값이 **false**로 설정된 경우 레이아웃의 폭은 장치에 독립적인(CSS) 픽셀의 WebView 컨트롤의 폭으로 설정됩니다. 만약 이값이 true이고 페이지에 viewport 메타가 있다면 폭은 태그에 의해 결정됩니다. 만약 이 값이 **true**면서 페이지가 태그를 포함하고 있지 않거나 너비를 정의하지 않은 경우에는 wide viewport가 사용될 것입니다.

### setSupportZoom

```java
void setSupportZoom (boolean support)
```

온스크린 줌 컨트롤과 제스처를 이용하여 줌을 가능하게 할 지 결정합니다. 특정 줌 메커니즘을 사용할 지의 여부는 [setBuiltInZoomControls(boolean)](#setBuiltInZoomControls)으로 설정할 수 있으며 `zoomIn()`과 `zoomOut()`을 사용하는 데에는 영향을 주지 않습니다. 기본값은 **true** 입니다.

### setBuiltInZoomControls

```java
void setBuiltInZoomControls (boolean enabled)
```

빌트인된 줌 메커니즘을 사용할지를 결정합니다. 빌트인 줌 메커니즘은 WebView의 컨텐츠위에 표시되는 온스크린 줌 컨트롤과 핀치 제스처를 통한 줌 컨트롤을 포함합니다. 온스크린 컨트롤을 보여지게 할지는 [setDisplayZoomControls(boolean)](#setDisplayZoomControls)로 설정합니다. 이 설정의 기본값은 **false** 입니다.

빌트인 메커니즘는 현재 지원되는 줌 메커니즘만 가지므로 이 옵션은 항상 활성화하는 것을 추천합니다.

### setDisplayZoomControls

```java
void setDisplayZoomControls (boolean enabled)
```

빌트인 줌 메커니즘을 사용할 때 온스크린 줌 컨트롤을 보여줄지를 결정합니다. 기본값은 **true**입니다.

### setTextZoom

```java
void setTextZoom (int textZoom)
```

페이지의 텍스트 확대비율을 퍼센트로 표현합니다. 기본값은 100입니다.

## Geolocation 관련 설정

### setGeolocationEnabled

```java
void setGeolocationEnabled (boolean flag)
```

Geolocation 활성화 여부를 설정합니다. 기본값은 **true**입니다.

WebView내의 페이지에서 Geolocation API를 사용하기 위해서는 다음과 같은 조건을 만족해야 합니다.

어플리케이션은 장치의 location 정보에 접근하기 위한 권한을 가지고 있어야 합니다. (**ACCESS_COARSE_LOCATION**, **ACCESS_FINE_LOCATION**)
어플리케이션은 `onGeolocationPermissionsShowPrompt(String, GeolocationPermissions.Callback)`을 구현하여야 합니다.

### setGeolocationDatabasePath

```java
void setGeolocationDatabasePath (String databasePath)
```

Geolocation 데이터베이스가 저장될 경로를 지정합니다.

## Font 관련 설정

### setCursiveFontFamily

```java
void setCursiveFontFamily (String font)
```
cursive 폰트 패밀리 네임을 지정합니다. 기본값은 “cursive”입니다.

### setFantasyFontFamily

```java
void setFantasyFontFamily (String font)
```

fantasy 폰트 패밀리 네임을 지정합니다. 기본값은 “fantasy”입니다.

### setFixedFontFamily

```java
void setFixedFontFamily (String font)
```

fixed 폰트 패밀리 네임을 지정합니다. 기본값은 “monospace”입니다.

### setSansSerifFontFamily

```java
void setSansSerifFontFamily (String font)
```

sans-serif 폰트 패밀리 네임을 지정합니다. 기본값은 “sans-serif”입니다.

### setSerifFontFamily

```java
void setSerifFontFamily (String font)
```

serif 폰트 패일리 네임을 지정합니다. 기본값은 “sans-serif”입니다.

### setStandardFontFamily

```java
void setStandardFontFamily (String font)
```

standard 폰트 패밀리 네임을 지정합니다. 기본값은 “sans-serif”입니다.

### setDefaultFixedFontSize

```java
void setDefaultFixedFontSize (int size)
```

fixed 폰트의 크기를 설정합니다 기본값은 16이며 입력값의 범위는 1에서 72사이의 값 입니다.

### setDefaultFontSize

```java
void setDefaultFontSize (int size)
```

기본 폰트의 크기를 설정합니다. 역시 기본값은 16이며 1에서 72사이의 값을 가질 수 있습니다.

### setMinimumFontSize

```java
void setMinimumFontSize (int size)
```

폰트의 최소 크기를 설정합니다. 기본값을 8이며 1에서 72사이의 값을 지정할 수 있습니다.

### setMinimumLogicalFontSize

```java
void setMinimumLogicalFontSize (int size)
```

폰트의 logical 최소 크기를 설정합니다. 기본값은 8이며 1에서 72사이의 값을 지정할 수 있습니다.

## 기타 설정

### setDefaultTextEncodingName

```java
void setDefaultTextEncodingName (String encoding)
```

html 페이지를 디코딩할 때 사용하는 텍스트 인코딩을 설정합니다. 기본값은 “UTF-8” 입니다.

### setMediaPlaybackRequiresUserGesture

```java
void setMediaPlaybackRequiresUserGesture (boolean require)
```

미디어를 플레이하기 위해 사용자의 제스처가 필요한지를 결정합니다. 기본값은 **true**입니다.

### setNeedInitialFocus

```java
void setNeedInitialFocus (boolean flag)
```

requstFocus가 호출될 때 포커스를 가지는 노드를 설정할 필요가 있는지를 설정합니다(?). 기본값은 **true**입니다.

### setUserAgentString

```java
void setUserAgentString (String ua)
```

WebView의 user-agent 문자열을 설정합니다. 만약 문자열이 비었거나 null 일 경우 시스템의 기본값이 사용됩니다. KITKAT부터 페이지를 로딩하는 중에 user-agent를 변경하면 WebView 는 로딩 초기화를 한번 더 하게 됩니다.

## Deprecated 된 설정들

### ~~setAppCacheMaxSize~~

```java
void setAppCacheMaxSize (long appCacheMaxSize)
```

> 이 메소드는 API 레벨 18부터 deprecated 되었습니다. 이후부터는 할당량을 자동으로 관리합니다.

~~어플리케이션 캐시의 최대 크기를 지정합니다. 입력된 크기는 데이터베이스가 지원하는 가장 가까운 값으로 반올림되므로 hard limit이 아닌 가이드입니다. 현재 데이터베이스의 크기보다 작게 설정하여도 데이터베이스를 trim하지 않습니다. 기본값은 MAX_VALUE(2^63-1)이며 이값을 바꾸는 것을 권장합니다.~~

### setDatabasePath

```java
void setDatabasePath (String databasePath)
```

<blockquote>이 메소드는 API 레벨 19부터 deprecated 되었습니다. 19버전부터는 크로미움 WebView가 사용되므로 크로미움 엔진이 경로를 관리하게 되며 이 메소드를 호출해도 아무런 영향이 없습니다.</blockquote>
<strike>데이터베이스의 경로를 지정합니다. 어플리케이션이 write 권한이 있는 패스를 지정해야 하며 한번만 호출해야 합니다. 이후의 호출은 무시됩니다.</strike>

### setDefaultZoom

```java
void setDefaultZoom (WebSettings.ZoomDensity zoom)
```

<blockquote>이 메소드는 API 레벨 19부터 deprecated 되었습니다. 이 메소드는 더이상 지원되지 않습니다.</blockquote>
<strike>페이지의 기본 zoom density를 설정합니다. 이 메소드는 UI 스레드로 부터 불려야 하며 기본값은 MEDIUM입니다.</strike>

### setEnableSmoothTransition

```java
void setEnableSmoothTransition (boolean enable)
```

<blockquote>이 메소드는 API 레벨 17부터 deprecated 되었습니다. 이 메소드는 현재 사용되지 않습니다. 추후에는 없어질 것입니다.</blockquote>
<strike>패닝이나 줌동작시 부드러운 트랜지션을 할지를 결정합니다. 기본값은 false입니다.</strike>

### setLightTouchEnabled

```java
void setLightTouchEnabled (boolean enabled)
```

<blockquote>이 메소드는 API 레벨 18부터 deprecated 되었습니다. JELLY_BEAN부터 이 설정은 사용되지 않으며 아무런 효과도 나타나지 않습니다.</blockquote>
<strike>mouseover를 활성화하기 위해 light touch의 사용을 가능하게 합니다.</strike>

### setPluginState

```java
void setPluginState (WebSettings.PluginState state)
```

<blockquote>이 메소드는 API 레벨 18부터 deprecated 되었습니다. 향후 플러그인은 지원되지 않습니다.</blockquote>
<strike>WebView에서 플러그인을 사용할지를 결정합니다. ON, OFF, ON_DEMAND 값을 지정할 수 있으며 온디맨드의 경우 플러그인영역에 placeholder가 나타나며 클릭스 플러그인이 활성화 되는 것입니다. 기본값은 OFF입니다.</strike>

### setRenderPriority

```java
void setRenderPriority (WebSettings.RenderPriority priority)
```

<blockquote>이 메소드는 API 레벨 18부터 deprecated 되었습니다. 스레드의 우선순위를 조정하는 것은 권장하지 않으며, 향후에는 지원하지 않을 것입니다.</blockquote>
<strike>렌더 스레드의 우선순위를 결정합니다. 다른 설정과는 달리 하나의 프로세스당 한번만 호출 가능합니다. 기본값은 NORMAL입니다.</strike>

### setSavePassword

```java
void setSavePassword (boolean save)
```

<blockquote>이 메소드는 API 레벨 18부터 deprecated 되었습니다. 이후 버전부터는 WebView에서 password를 저장하는 것은 지원하지 않습니다.</blockquote>
<strike>WebView가 password를 저장할지를 설정합니다. 기본값은 true 입니다.</strike>

### setTextSize

```java
void setTextSize (WebSettings.TextSize t)
```

<blockquote>이 메소드는 API 레벨 14부터 deprecated 되었습니다. [[#setTextZoom|setTextZoom(int)]]를 사용하세요.</blockquote>
<strike>페이지의 텍스트 크기를 설정합니다. 기본값은 NORMAL입니다.</strike>

[^ver]: 이 글의 WebSetting 메소드들은 **API 레벨 23**을 기준으로 합니다.
[^same-origin_policy]: 동일 출처 정책(same-origin policy)은 한 출처(origin)에서 로드된 문서나 스크립트가 다른 출처 자원과 상호작용하지 못하도록 제약하는 것