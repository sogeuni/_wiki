# 발리(Volley)를 이용한 네트워크 데이터 전송

https://developer.android.com/training/volley/index.html

발리(Volley)는 안드로이드 앱에서 네트워킹을 쉽고 빠르게 구현하기 위한 HTTP 라이브러리입니다. Volley는 오픈 [AOSP](https://android.googlesource.com/platform/frameworks/volley)[^AOSP] 저장소를 통해 사용가능합니다.

[^AOSP]: [Android Open Source Project](https://source.android.com/)

Volley는 다음과 같은 이점을 제공합니다: 

* 네트워크 요청(request)의 자동 스케줄링
* 여러 개의 동시 네트워크 연결
* 표준 HTTP [캐시 일관성](https://ko.wikipedia.org/wiki/%EC%BA%90%EC%8B%9C_%EC%9D%BC%EA%B4%80%EC%84%B1)을 통한 투명 디스크 및 메모리 응답(response) 캐싱
* 요청에 대한 우선 순위 지원
* 요청 API 취소 가능. 단일 요청을 취소하거나 요청 블록 또는 범위를 설정하여 취소할 수 있습니다.
* 사용자 정의가 용이함. 예를 들어, 재시도와 백오프(backoff)[^backoff] 등.
* 네트워크로 부터 비동기적으로 들어오는 데이터를 UI에 제대로 채울 수 있도록 요청 순서대로 응답을 동기화함
* 디버깅 및 추적 도구를 제공

[^backoff]: 네트워크 상에서 데이터 송수신 중 충돌이 발생하여 전송이 실패한 경우 일정한 주기로 반복된 데이터 블록 전송을 하는 것

Volley는 구조화된 데이터의 검색 결과를 페이지에 붙이는 것과 같은 UI를 채우는데 사용되는 RPC 형태의 작업에 뛰어납니다. 어떤 프로토콜과도 쉽게 통합되며 문자열, 이미지, JSON과 같이 다양한 데이터를 전송 가능합니다. Volley는 당신이 필요로 하는 기능을 이미 내장하고 있기 때문에, 보일러플레이트 코드를 매번 작성해야 하는 수고를 덜고 앱의 특정 로직에 더 집중할 수 있도록 해줍니다.

Volley는 파싱하는 동안 모든 응답을 메모리에 가지고 있기 때문에, 대용량의 다운로드나 스트리밍 작업에는 적합하지 않습니다. 큰 다운로드 작업의 경우, [DownloadManager](https://developer.android.com/reference/android/app/DownloadManager.html)와 같은 대안을 사용하는 것이 좋습니다.

Volley 코어 라이브러리는 AOSP 저장소의 `frameworks/volley`에서 개발되고 있으며, Volley에서 사용하는 기본 요청 파이프라인 뿐만 아니라 일반적으로 적용가능한 유틸리티 모음인 **toolbox**를 포함합니다. 프로젝트에 volley를 추가하는 가장 쉬운 방법은 volley 저장소를 clone하고 그것을 라이브러리 프로젝트로 지정하는 것입니다:[^link]

[^link]: `build.gradle`의 dependencies에 다음을 추가하면 더 간단히 사용할 수 있습니다.
```
compile 'com.android.volley:volley:1.0.0'
```

0. 다음 명령을 이용하여 저장소를 clone 합니다:
```
git clone https://android.googlesource.com/platform/frameworks/volley
```
0. 다운받은 소스를 앱 프로젝트에 [안드로이드 라이브러리 모듈](https://developer.android.com/studio/projects/android-library.html)로 import 합니다.

## Lessons

[[간단한 요청 보내기]]

Volley의 기본 동작을 사용하여 간단한 요청을 보내는 방법과 요청을 취소하는 방법에 대해 알아 봅니다.

[[RequestQueue 설정하기]]

**RequestQueue**를 설정하는 방법과 앱이 종료될 때 까지 유지되는 **RequestQueue**를 만들기 위해서 싱글턴 패턴을 구현하는 방법에 대해 알아 봅니다.

[[표준 요청 만들기]]

Volley에서 기본적으로 제공하는 타입(raw string, image, JSON)을 사용하여 요청을 보내는 방법에 대해 알아 봅니다.

[[사용자 정의 요청 구현하기]]

사용자 정의 요청을 구현하는 방법에 대해 알아 봅니다.