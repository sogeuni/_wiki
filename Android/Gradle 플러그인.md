# 안드로이드 Gradle 플러그인

## 소개

**build.gradle** 파일의 전체 옵션은 [DSL reference](http://google.github.io/android-gradle-dsl/current/)를 참고하세요.

**새로운 빌드 시스템의 목표**

새로운 빌드 시스템의 목표는 다음과 같습니다.

* 코드와 리소스의 재사용을 용이하게 한다.
* 하나의 어플리케이션으로 여러 apk를 발행하거나 다른 flavor를 적용하기 쉽게 한다.
* 빌드 프로세스의 구성, 확장 및 수정을 용이하게 한다.
* IDE와의 좋은 통합

### 왜 Gradle인가?

Gradle은 고급 빌드 시스템일 뿐만 아니라 플러그인을 통해 빌드로직을 커스텀할 수 있는 고급 빌드 툴킷이기도 합니다. 우리가 Gradle을 선택하게된 특징의 일부가 여기 있습니다.

* 빌드 로직을 서술하는데 사용하는 Groovy 기반의 도메인 특화 언어(DSL)
* Groovy 기반의 빌드 파일은 커스텀 로직을 적용하기 위해 DSL을 통한 엘리먼트 선언과 코드를 사용하는 방식을 혼합해서 사용할 수 있다.
* Maven (또는 Ivy)를 통해 기본 제공되는 의존성 관리
* 매우 유연함. 좋은 예제들이 있지만 그 방식만을 강요하지 않음
* 플러그인은 빌드 파일에서 사용하기 위해 자신의 DSL과 API를 노출할 수 있음
* IDE 통합을 위한 좋은 tooling API들을 가짐

### 요구사항
* Gradle 2.2
* SDK과 Build Tools 19.0.0. 특정 기능은 더 최근의 버전이 필요할 수 있음

## 기본 프로젝트 셋업

Gradle 프로젝트는 root 폴더에 위치한 build.gradle 파일에 빌드과정을 서술합니다. (빌드 시스템 자체에 대한 개요는 [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide_single.html)를 참고하세요.)

### 간단한 빌드 파일

다음의 build.gradle 는 가장 간단한 안드로이드 프로젝트 입니다.

```groovy
buildscript {
  repositories { 
    jcenter()
  }

  dependencies {
    classpath 'com.android.tools.build:gradle:1.3.1'
  }
}

apply plugin: 'com.android.application'

android {
  compileSdkVersion 23
  buildToolsVersion "23.1.0"
}
```

이 안드로이드 빌드 파일에는 3개의 주요영역이 있습니다.

`buildscript { ...}`는 빌드를 구동하는 코드를 구성합니다. 이 경우 jCenter 저장소를 사용하는 것으로 선언하고 Maven 상의 바이너리 classpath를 의존성으로 가집니다. 이 바이너리는 Gradle을 사용하기 위한 안드로이드 플러그인을 포함하는 라이브러리며 버전은 1.3.1 입니다. 

> **참고:** 이 것은 프로젝트가 아닌 빌드를 실행하는 코드에만 영향을 미칩니다. 프로젝트에서 필요로 하는 저장소와 의존성은 따로 선언해야 하며 이것은 나중에 다시 설명합니다.

그런 다음, `com.android.application` 플러그인이 적용됩니다. 이것은 안드로이드 어플리케이션을 빌드하기 위한 플러그인입니다.

마지막으로, `android { ...}` 는 안드로이드 빌드를 위한 모든 매개변수를 설정합니다. 이 것은 안드로이드 DSL의 시작점입니다. 기본적으로 컴파일 대상(`compileSdkVersion`)과 빌드툴의 버전(`buildtoolsVersion`)만 필요합니다. 컴파일 대상은 이전 빌드 시스템의 project.properties 파일의 `target` 속성과 동일합니다. 새로운 속성은 int 값의 api 레벨이나 기존의 `target` 속성에서 사용하던 것과 같은 string 값을 할당할 수 있습니다.
 
> **중요:** `com.android.application` 플러그인 만을 적용해야 합니다. `java` 플러그인을 적용하면 빌드 오류가 발생합니다.

> **참고:** 또한 기존 SDK에서 처럼 `sdk.dir` 속성을 사용하여 SDK의 위치를​​ 설정하는 ''local.properties'' 파일이 필요할 수 있습니다. 아니면 `ANDROID_HOME` 환경 변수를 설정해도 됩니다. 두 방법의 차이점은 없으며 선호하는 방법을 사용하면 됩니다. 다음은 ''local.properties'' 파일의 예입니다.

```linux-config
sdk.dir=/path/to/Android/Sdk
```

### 프로젝트 구조

기본 프로젝트는 "source sets"라 불리는 두개의 컴포넌트로 시작하는데 하나는 메인소스 코드이며 하나는 테스트 코드입니다. 

* src/main/
* src/androidTest/

각 디렉토리내에는 소스 컴포넌트에 대한 서브디렉토리가 존재합니다. 자바 소스코드와 자바 리소스 디렉토리는 다음과 같습니다.

* java/
* resources/

​안드로이드 플러그인을 위한 추가 파일과 폴더는 다음과 같습니다.

* AndroidManifest.xml
* res/
* assets/
* aidl/
* rs/
* jni/
* jniLibs/

따라서 메인 소스셋의 모든 *.java 파일은 `src/main/java` 에 존재하며 메인 매니페스트 파일은 `src/main/AndroidManifest.xml` 이 됩니다.

> **참고:** `src/androidTest/AndroidManifest.xml` 파일은 자동으로 생성되므로 필요하지 않습니다.

#### 프로젝트 구조 설정

기본 프로젝트 구조가 적절하지 않다면 수정이 가능합니다. 순수 자바 프로젝트에서의 방법은 [gradle 문서](https://docs.gradle.org/current/userguide/java_plugin.html#N12394)를 참고하세요.

안드로이드 플러그인도 비슷한 구문을 사용하지만 자신만의 ''sourceSets''를 사용하기 때문에 이것은 `android` 블록안에 지정되어야 합니다. 다음은 이클립스에서 사용하던 이전 프로젝트의 구조를 지정하고 `androidTest`의 ''sourceSet''을 tests 폴더로 다시 매핑하는 예제입니다.

```groovy
android {
  sourceSets {
    main {
      manifest.srcFile 'AndroidManifest.xml'
      java.srcDirs = ['src']
      resources.srcDirs = ['src']
      aidl.srcDirs = ['src']
      renderscript.srcDirs = ['src']
      res.srcDirs = ['res']
      assets.srcDirs = ['assets']
    }

    androidTest.setRoot('tests')
  }
}
```

> **참고:** ​예전 구조는 모든 소스파일(Java, AIDL, RenderScript)이 같은 폴더에 있었기 때문에 모두 `src`로 매핑됩니다.
> **참고:** `setRoot()` 는 모든 ''sourceSet'' (서브폴더를 포함)을 새로운 폴더로 move 합니다. 예제에서 `src/androidTest/*`는 `tests/*`로 이동됩니다. 이것은 안드로이드만의 기능이며 자바 ''sourceSets''에서는 동작하지 않습니다.

### 빌드 Task

#### 일반 Task

플러그인이 적용된 빌드파일은 자동으로 빌드 task의 집합을 생성합니다. 자바플러그인과 안드로이드 플러그인 모두 이작업을 수행합니다. Task에 대한 규칙은 다음과 같습니다.

* **assemble**<br>프로젝트의 결과물을 만드는 task
* **check**<br>​모든 검사를 수행하는 task
* **build**<br>이 task는 **assemble**과 **check**를 둘 다 수행합니다.
* **clean**<br> 이 task는 프로젝트의 결과물을 삭제합니다.

**assemble**, **check**, **build** task는 실제로는 아무일도 하지 않으며 실제로 동작하는 task들을 추가하기 위한 "anchor" task 입니다.

이렇게 하면 프로젝트의 타입이나 어떤 플러그인을 쓰는지 상관하지 않고 동일한 task를 호출할 수 있습니다. 예를 들어, ​

''firebugs'' 플러그인을 적용하려면 새로운 task를 만들고 **check** task가 그것에 의존하도록 하면 **check** task가 호출될때 마다 호출이 될 것입니다.

다음의 커맨드로 실행중인 high level task의 목록을 가져올 수 있습니다.
```
gradle tasks
```

전체 리스트와 의존관계를 보려면 다음과 같이 합니다.
```
gradle tasks --all
```

> **참고:** Gradle은 task의 선언된 input과 output을 자동으로 모니터링 합니다.

프로젝트 변경없이 **build** task를 두 번 실행하면 Gradle은 모든 task를 UP-TO-DATE로 간주하고 아무 작업도 하지 않습니다. 이 것은 task간의 불필요한 빌드 동작을 요청하지 않아야 task가 잘 동작함을 의미합니다.

#### Java 프로젝트 task

다음은 **Java**플러그인에서 생성하는 두개의 중요한 task 이며, main acchor task와 의존관계를 가집니다.

* **assemble**
** **jar**<br>이 task는 결과물을 생성합니다.
* **check**
** **test**<br>이 task는 test를 수행합니다.

**jar** task는 직간접적으로 다른 task에 의존적입니다. 예를 들어 **classes** 는 자바 코드를 컴파일 합니다.  테스트는 **testClasses** task로 컴파일 됩니다.

일반적으로  **assemble** 이나 **check** 이외의 task들은 호출하지 않습니다. 자바 플러그인의 모든 task 의 상세설명은 [이 곳](https://docs.gradle.org/current/userguide/java_plugin.html)을 참고하세요.

#### 안드로이드 task

안드로이드 플러그인은 다른 플러그인과의 호환성을 유지하기위해 동일한 규칙을 사용하며 anchor task를 추가합니다.

* **assemble**<br>프로젝트의 결과물(들)을 만들어내는 task입니다.
* **check**<br>모든 검사를 수행하는 task입니다.
* **connectedCheck**<br>연결된 장치나 에뮬레이터를 체크합니다.
* **deviceCheck**<br>원격 장치에 연결하기 위한 API를 사용하여 체크를합니다. 이 것은 CI 서버에서 사용합니다.
* **build**<br>**assemble**과 **check**를 수행합니다.
* **clean**<br>프로젝트의 결과물을 청소합니다.

새로운 anchor task는 연결된 장치 없이도 일반적인 check를 하기위해 필요합니다.  **build''는 **deviceCheck**나 **connectedCheck**에 의존적이지 않음에 유의하세요.

안드로이드 프로젝트는 적어도 두개의 결과물(debug APK과 release APK)을 가집니다. 각각은 빌드를 편하게 하기위해 anchor task를 각각 따로 가집니다.

* **assemble**
  * **assembleDebug**
  * **assembleRelease**

각각은 APK를 빌드하기 위해 필요한 여러단계의 각각 다른 task에 의존적입니다. **assemble** task는 양쪽 모두에 의존적이며 따라서 두개의 APK를 빌드합니다.

**Tip:** Gradle은 커맨드라인 상에서 task명에 대한 camel case형태의 단축키를 지원합니다. 예를 들어
```
gradle aR
```
은 'aR'과 일치하는 task가 없다면
```
gradle assembleRelease
```
과 같습니다.

Check anchor task는 자신의 의존성을 가집니다.

* **check**
  * **lint**
* **connectedCheck**
  * **connectedAndroidTest**
* **deviceCheck**
  * 이것은 다른 플러그인이 test 확장을 구현할때 만든 task에 의존합니다.

마지막으로, 플러그인은 모든 빌드 타입(**debug**, **release**, **test**)​에 대해 설치와 삭제를 위한 task를 생성합니다.

* **installDebug**
* **installRelease**
* **uninstallAll**
 * **uninstallDebug**
 * **uninstallRelease**
 * **uninstallDebugAndroidTest**

### 기본 빌드 사용자지정

안드로이드 플러그인은 빌드 시스템의 거의 대부분을 수정할 수 있는 광범위한 DSL을 제공합니다.

#### 매니페스트 항목

DSL을 통해 다음과 같은 대부분의 manifest 항목을 설정할 수 있습니다.

* minSdkVersion
* targetSdkVersion
* versionCode
* versionName
* applicationId (packageName 보다 유용함. 사세한 사항은 [ApplicationId vs. PackageName](http://tools.android.com/tech-docs/new-build-system/applicationid-vs-packagename)을 참고)
* testApplicationId (test APK에서 사용)
* testInstrumentationRunner

예제:
```groovy
android {
  compileSdkVersion 23
  buildToolsVersion "23.0.1"

  defaultConfig { 
    versionCode 12
    versionName "2.0"
    minSdkVersion 16
    targetSdkVersion 23
  }
}
```

[Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/)를 문서에서 전체 빌드 속성을 확인할 수 있습니다.

매니페스트 속성을 빌드파일 쓰는 것은 값을 동적으로 선택할 수 있다는 장점이 있습니다. 예를 들어, 버전명을 파일이나 다른 ​커스텀 로직으로 부터 읽어 들일 수 있습니다.

```groovy
def computeVersionName() {
  ...
}

android {
  compileSdkVersion 23
  buildToolsVersion "23.0.1"

  defaultConfig {
    versionCode 12 
    versionName computeVersionName()
    minSdkVersion 16
    targetSdkVersion 23
  }
}
```

> **참고:** 주어진 영역내의 게터와 동일한 이름의 함수명을 사용하지 마세요. 예를 들어, **​defaultConfig { ...}**에서 **getVersionName()** 을 호출하면 자동으로 함수대신 **defaultConfig.getVersionName()**을 사용할 것입니다.

#### 빌드 타입

기본적으로 안드로이드 플러그인은 자동으로 어플리케이션의 debug와 release 버전의 빌드를 위한 프로젝트를 설정합니다. 둘의 큰 차이는 secure device (개발장치가 아닌) 상에서 디버깅이 가능한 지의 여부와 APK의 서명방식입니다. Debug 버전은 알려진 이름/비밀번호를 사용하여(빌드 중에 프롬프트가 뜨는것을 막기위해) 자동으로 생성된 키/인증서를 가지고 서명을 합니다. Release는 빌드 중에 서명을 하지 않고 이후에 서명합니다.

이러한 설정은 **BuildType**이라 불리는 오브젝트를 통해 이루어집니다. 기본적으로 **debug**와 **release** 2개의 인스턴스가 생성됩니다. 안드로이드 플러그인은 두개의 인스턴스를 수정하는 것 뿐만아니라 다른 ''빌드 타입''을 생성하는 것도 허용합니다. 이 것은 **buildTypes** DSL 컨테이너에서 이루어 집니다.

```groovy
android {
  buildTypes {
    debug {
      applicationIdSuffix ".debug"
    }

    jnidebug {
      initWith(buildTypes.debug)
      applicationIdSuffix ".jnidebug"
      jniDebuggable true
    }
  }
}
```

위의 코드는 다음을 수행합니다.

* **debug** 빌드 타입을 설정
 * 패키지명을 `<app applicationId>.debug` 로 설정하여 동일한 장치에 ''debug''와 ''release''버전을 함께 설치할 수 있게 합니다.
* **jnidebug''라는 ''빌드 타입''을 만들고 **debug** 빌드 타입의 속성을 복사합니다.
* **jnidebug** 빌드시 JNI 컴포넌트의 디버깅을 허용하고, ​패키지명에 다른 접미사를 추가합니다.

새로운 ''빌드 타입''을 추가하는 것은 **buildTypes** 컨테이너에 **initWith()** 혹은 클로저를 사용하여 새로운 엘리먼트를 추가하는 것처럼나 간단합니다.  빌드 타입을 설정할 수 있는 모든 속성은 [DSL Reference](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html)를 참고하세요.

추가로, 빌드 타입은 코드나 리소스를 추가하기 위해 사용할 수 있습니다. 각각의 빌드타입에 대해 매칭되는 ''sourceSet''이 생성되며 기본값은 `src/<buildtypename>/` 입니다. 예를 들어 debug APK를 빌드하는데 사용되는 리소스는 `src/debug/java` 디렉토리를 사용합니다. 이 것은 ''빌드 타입''의 이름이 ''main''이나 ''androidTest''가 되어서는 안되며 유일한 이름이어야 한다는 뜻입니다.

다른 소스셋 처럼 빌드 타입별 소스셋의 위치도 변경가능 합니다.
```groovy
android {
    sourceSets.jnidebug.setRoot('foo/jnidebug')
} 
```

또한, 각 ''빌드 타입''마다 **assembleDebug**와 같이 새로운 `assemble<BuildTypeName>` task가 생성됩니다. 앞에서 언급한 것처럼 **assembleDebug**와 **assembleRelease** task는 **debug**와 **release** 빌드타입이 미리 생성되어 있기 때문에 자동으로 생성됩니다. 이러한 규칙에 따라, 위의 ''build.gradle''에서는 **assembleJnidebug** task가 생성되고, **assemble** task가 **assembleDebug**와 **assembleRelease**에 의존성을 가지는 것과 같은 방식으로 **assembleJnidebug** task의 의존성을 가지게 됩니다.

> **Tip:** `gradle aJ`로 **assembleJnidebug** task를 실행한다는 것을 기억하세요.

사용가능한 사례:

* 디버그 모드만 권한을 주며, 릴리즈 모드는 주지 않음
* 디버깅 사용자 정의
* 디버깅 모드의 리소스를 다르게 정의함 (예를 들어 리소스 값이 인증서와 연동되는 경우)

''BuildType''의 코드/리소스는 다음과 같은 방법으로 사용됨

* manifest는 앱의 manifest로 머지됨
* code는 다른 소스 폴더처럼 동작
* resource는 main resource를 오버레이하며 기존값을 대치함

#### 서명 설정

어플리케이션 서명은 다음값들을 필요로 합니다. (APK 서명에 대한 자세한 정보는 [Signing Your Application](http://developer.android.com/tools/publishing/app-signing.html)을 참고하세요.)

* A keystore
* A keystore password
* A key alias name
* A key password
* The store type

기본적으로 **debug** 설정에서는 password를 알고 있는 debug keystore와 기본 key를 사용합니다.  debug keystore는 `$HOME/.android/debug.keystore`에 위치하며 없을경우 생성됩니다. **debug** 빌드 타입은 자동으로 **debug** ''SigningConfig''를 사용하도록 설정되어 있습니다.

**signingConfigs** DSL 컨테이너를 통해 내장된 기본설정을 수정하거나 새로운 설정을 생성할 수 있습니다. 
```groovy
android {
  signingConfigs {
    debug {
      storeFile file("debug.keystore")
    }

    myConfig {
      storeFile file("other.keystore")
      storePassword "android"
      keyAlias "androiddebugkey"
      keyPassword "android"
    }
  }

  buildTypes {
    foo {
      signingConfig signingConfigs.myConfig
    }
  }
}
```

위의 코드는 프로젝트 루트에 있는 debug keystore의 위치를 변경합니다. ​위의 경우 **debug** 빌드 타입의 keystore의 위치를 변경하였습니다. 또한, 새로운 Signing Config를 생성하여 새로운 빌드 타입에 적용하였습니다.

> **참고:** 기본 위치의 debug keystore만 자동으로 생성됩니다. debug keystore의 위치가 변경될 경우 생성되지 않습니다. 새로운 이름으로 ''SigningConfig''를 생성하며 기본 debug keystore 위치를 사용하면 keystore는 자동으로 생성됩니다. 달리 말하면 keystore의 위치는 설정의 이름과 연관되어 있지는 않습니다.

> **참고:** 일반적으로 keystore의 위치는 프로젝트의 루트에 상대적이지만 절대경로를 사용할 수도 있습니다. 하지만 추천하지는 않습니다.

> **참고:** [링크의 방법](http://stackoverflow.com/questions/18328730/how-to-create-a-release-signed-apk-file-using-gradle)을 통해 콘솔이나 환경 변수를 통해 password 값을 얻을 수도 있습니다.

## 의존성, 안드로이드 라이브러리 및 멀티-프로젝트 설정

Gradle 프로젝트는 다른 컴포넌트에 의존성을 가질 수 있습니다. 컴포넌트는 외부 바이너리 패키지나 다른 gradle 프로젝트가 될 수 있습니다.

### 바이너리 패키지에 대한 의존성

#### 로컬 패키지

외부 jar 라이브러리의 의존성을 설정하려면 **compile** 설정에 의존성을 추가합니다. 다음 코드는 `libs` 디렉토리 내의 모든 jar에 대한 의존성을 추가하는 예제입니다.

```groovy
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
}

android {
  ...
}
```

> **참고:** **dependencies** DSL 엘리먼트는 표준 Gradle API 이며 **android** 엘리먼트 내에 포함되지 않습니다.

**compile** 설정은 메인 어플리케이션을 컴파일하는데 사용됩니다. 그 안의 모든 것은 컴파일 classpath에 포함되며 최종 APK로 패키징됩니다. 의존성을 추가할 수 있는 설정들은 다음과 같습니다.

* **compile**: 메인 어플리케이션
* **androidTestCompile**: 테스트 어플리케이션
* **debugCompile**: 디버그 빌드 타입
* **releaseCompile**: 릴리즈 빌드 타입

관련된 빌드타입이 없는 경우 APK를 빌드할 수 없기 때문에, APK는 항상 두개(이상)의 설정을 가집니다. **compile**과 <buildtype>Compile. 빌드타입을 생성하면 그 이름을 바탕으로 새로운 설정이 생성됩니다. 이러한 방식은 디버그 버전에서 사용자 정의 라이브러리를 사용할 필요가 있거나 동일한 라이브러리의 다른 버전에 의존하는 경우에 유용합니다. (버전 충돌을 처리하는 방법에 대해 [Gradle 문서](https://docs.gradle.org/current/userguide/dependency_management.html#sub:version_conflicts)를 참고하세요)

#### 원격 바이너리

Gradle은 Maven이나 Ivy 저장소의 원격 바이너리를 가져오는 것을 지원합니다. 먼저 저장소를 리스트에 추가한 다음 Maven이나 Ivy에 선언된 방법대로 바이너리의 의존성을 선언합니다.

```groovy
repositories {
   jcenter()
}

dependencies {
  compile 'com.google.guava:guava:18.0'
}

android {
  ...
}
```

> **참고:** **jcenter()** 는 저장소의 URL을 지정하는 바로가기입니다. gradle은 원격 및 로컬 저장소를 모두 지원합니다.
**참고:** Gradle은 모든 의존성을 가져옵니다. 가져온 의존성이 그 자신의 의존성을 가지고 있다면 그것도 당겨옵니다.

의존성을 설정하는데 대한 더 자세한 정보는 [Gradle 사용자 가이드](http://gradle.org/docs/current/userguide/artifact_dependencies_tutorial.html)와 [DSL 문서](http://gradle.org/docs/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html)를 참고하세요.

### 멀티 프로젝트 설정

Gradle 프로젝트는 멀티 프로젝트 설정을 이용하여 다른 gradle 프로젝트에 의존할 수 도 있습니다. 멀티 프로젝트 설정은 일반적으로 주어진 루트 프로젝트의 모든 서브폴더를 프로젝트로 갖습니다. 예를들어 다음과 같은 구조에서
```
MyProject/
  + app/
  + libraries/
    + lib1/
    + lib2/
```
우리는 3개의 프로젝트를 인식할 수 있습니다. Gradle은 다음의 이름으로 프로젝트를 참조할 것입니다.

```
 :app
 :libraries:lib1
 :libraries:lib2
```

각각의 프로젝트는 자신의 빌드 방법을 선언한 ''build.gradle'' 파일을 가집니다. 또한,  프로젝트의 루트에 ''setting.gradle'' 파일이 있을 것입니다. 따라서 다음의 구조와 같습니다. 

```text
 MyProject/
 | settings.gradle
 + app/
    | build.gradle
 + libraries/
    + lib1/
       | build.gradle
    + lib2/
       | build.gradle
```

setting.gradle의 내용은 매우 간단합니다. 그것은 실제 Gradle 프로젝트의 폴더를 정의합니다.
```
include ':app', ':libraries:lib1', ':libraries:lib2'
```

**:app** 프로젝트는 라이브러리에 의존할 가능성이 있으며, 이것은 다음과 같이 의존성을 선언하는 것으로 이루어 집니다.

```groovy
dependencies {
   compile project(':libraries:lib1')
}
```

멀티 프로젝트 설정에 대한 일반적인 정보는 [이 글](http://gradle.org/docs/current/userguide/multi_project_builds.html)을 참고하세요.

### 라이브러리 프로젝트

위의 멀티-프로젝트 설정에서 **:libraries:lib1**과 **libraries:lib2**는 자바 프로젝트이며 **:app** 안드로이드 프로젝트는 그들의 ''jar''파일을 사용할 것입니다. 하지만 안드로이드 API를 사용하거나 안드로이드 스타일의 리소스를 사용하려면 이 라이브러리들은 일반적인 자바 프로젝트가 아닌 안드로이드 라이브러리 프로젝트가 되어야 합니다.

#### 라이브러리 프로젝트 만들기

라이브러리 프로젝트는 보통의 안드로이드 프로젝트와 몇가지만 다를뿐 거의 동일합니다. 라이브러리를 빌드하려면 어플리케이션을 빌드할 때와 다른 플러그인을 사용합니다. 내부적으로 두개의 플러그인은 대부분의 소스코드를 공유하며 동일한 `com.android.tools.build.gradle` jar 파일에 의해 제공됩니다.

```groovy
buildscript {
  repositories {
    jcenter()
  }

  dependencies {
    classpath 'com.android.tools.build:gradle:1.3.1'
  }
}

apply plugin: 'com.android.library'

android {
  compileSdkVersion 23
  buildToolsVersion "23.0.1"
}
```

위의 코드는 API 23으로 컴파일하는 라이브러리 프로젝트 입니다. sourceSet, 빌드타입, 의존성은 어플리케이션 프로젝트와 동일하게 설정할 수 있습니다.

#### 프로젝트와 라이브러리 프로젝트의 차이점

라이브러리 프로젝트의 결과물은 [.aar 패키지](http://tools.android.com/tech-docs/new-build-system/aar-format)입니다. 이것은 컴파일 코드(jar 파일과 .so파일)와 리소스(manifest, res, assets)의 조합입니다. 라이브러리 프로젝트는 어플리케이션으로 부터 독립적으로 라이브러리를 테스트하기위해 테스트 apk를 생성할 수도 있습니다. 동일한 anchor tesk를 사용(**assembleDebug**, **assembleRelease**)하므로 프로젝트를 빌드하는 명령의 차이가 없습니다. 라이브러리는 어플리케이션과 동일하게 동작합니다. 라이브러리는 여러 빌드타입과 product flavor(아래에서 설명)을 가지며 하나이상의 aar을 생성할 수도 있습니다. 빌드 타입의 대부분은 라이브러리 프로젝트에서는 적용되지 않음을 주의하세요. 하지만 라이브러리의 컨텐츠를 변경하는 ''sourceSet''를 사용할 수 있습니다.

#### 라이브러리 참조

라이브러리를 참조하는 것은 다른 프로젝트를 참조하는 방식과 동일합니다.

```groovy
dependencies {
  compile project(':libraries:lib1')
  compile project(':libraries:lib2')
}
```

> **참고:** 하나 이상의 라이브러리를 가질 경우 순서가 중요합니다. 이 것은 이전 빌드 시스템에서 project.properties 파일의 의존성 순서가 중요한 것과 비슷합니다.

#### 라이브러리 발행(Publication)

기본적으로 라이브러리는 ''release''버전으로만 발행됩니다. 따라서, 라이브러리를 참조하는 프로젝트에서 버전을 지정하더라도 무시됩니다. 이것은 Gradle의 제약때문에 임시로 생긴 제약이며 없애기 위해 노력중입니다. 개발자는 라이브러리를 발행할 때 버전을 제어할 수 있습니다.

```groovy
android {
  defaultPublishConfig "debug"
}
```

발행 설정 이름은 풀버전명을 사용합니다. ''Release''와 ''debug''는 flavor가 없는 경우에만 적용할 수 있습니다.  flavor를 사용할 때 기본 발행 버전을 변경하려면 다음과 같이 씁니다.

```groovy
android {
  defaultPublishConfig "flavor1Debug"
}
```

이런식으로 라이브러리의 모든 variant를 발행할 수 있습니다. 라이브러리도 일반적인 프로젝트간의 의존성을 사용할 수 있도록 할 계획입니다. 하지만 지금은 Gradle의 제약때문에 불가능하며 고치기 위해 노력중입니다. 
모든 variant의 게시는 기본적으로는 비활성화 되어 있습니다만 아래와 같이 이 기능을 활성화 할 수 있습니다.

```groovy
android {
  publishNonDefault true
}
```

여러 variant를 발행한다는 것은 하나의 aar 파일이 여러개의 variant를 가지는 것이 아니라 여러개의 aar 파일이 생성되는 것입니다. 각각의 aar 파일은 하나의 variant를 가집니다.

Gradle 기본 결과물의 의존성을 가질 때는 다음과 같이 씁니다.

```groovy
dependencies {
  compile project(':libraries:lib2')
}
```

발행된 라이브러리별 의존성을 가질때는 다음과 같이 지정할 수 있습니다.
```groovy
dependencies {
  flavor1Compile project(path: ':lib1', configuration: 'flavor1Release')
  flavor2Compile project(path: ':lib1', configuration: 'flavor2Release')
}
```

> **중요:** 발행된 설정은 빌드 타입을 포함한 full variant 이며 참조할 때도 마찬가지 입니다.

> **중요:** 기본값이 아닌 발행을 활성화할 때 Maven 플러그인은 이것들을 외부패키지처럼 발행할 것입니다. 따라서 maven 저장소와 진짜로 호환되지는 않습니다. 따라서 당신은 하나의 variant로 저장소에 게시하거나 프로젝트 내부의 의존성에 따라 모든 설정으로 발행하던가 해야 합니다.

## 테스트  

테스트 어플리케이션을 빌드하는 것은 어플리케이션 프로젝트에 통합되었으므로 이제 더이상 테스트 프로젝트를 따로 만들 필요가 없습니다.

### 유닛 테스트

1.1 버전부터 유닛 테스트를 지원합니다. [이 문서](http://tools.android.com/tech-docs/unit-testing-support)를 참고하세요. 이 섹션의 나머지 부분은 "instrumentation test"에 대해 설명할 것이며,  Instrumentation test는 실제 장치(혹은 에뮬레이터)에서 실행할 수 있으며 테스트를 위한 APK를 빌드할 필요가 있습니다.

### 기본 사항 및 구성

앞서 이야기했듯이, **androidTest** ''sourceSet''은 `src/androidTest/`에 있습니다. 이 ''sourceSet''은 테스트 APK를 빌드하기위해 사용됩니다. 데스트 APK는 안드로이드 테스트 프레임워크를 사용하며 장치에 설치할 수 있습니다. 이것은 안드로이드 유닛 테스트, instrumentation 테스트 및 UI automator 테스트를 포함합니다. 테스트 앱의 매니페스트의 `<instrumentation>` 노드는 자동으로 생성되지만 테스트 매니페스트에 다른 컴포넌트를 추가하기위해 `src/androidTest/AndroidManifest.xml` 파일을 생성할 수도 있습니다.

`<instrumentation>`노드를 구성하기위해 테스트앱에서 사용할 수 있는 몇 가지 값들이 있습니다. (자세한 것은 [ DSL 레퍼런스](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.ProductFlavor.html)를 참고하세요)

* **testApplicationId**
* **testInstrumentationRunner**
* **testHandleProfiling**
* **testFunctionTest**

이전에 본것과 같이, 이러한 값들은 **defaultConfig** 객체에서 사용됩니다.

```groovy
android {
  defaultConfig {
    testApplicationId "com.test.foo"
    testInstrumentationRunner "android.test.InstrumentationTestRunner"
    testHandleProfiling true 
    testFunctionalTest true
  }
}
```

테스트 어플리케이션 매니페스트 내의 instrumentation 노드의 targetPackage 속성은 테스트 앱의 패키지 이름으로 자동으로 채워집니다. 

또한, ''androidTest'' 소스 셋은 자신만의 의존성을 구성할 수 있습니다. 기본적으로 어플리케이션의 의존성이 테스트앱의 classpath에 추가 되지만 추가로 테스트앱에 필요한 의존성은 다음과 같이 확장가능합니다.

```groovy
dependencies {
  androidTestCompile 'com.google.guava:guava:11.0.2'
}
```

테스트앱은 **assembleAndroidTest** task에의해 빌드됩니다. 이 task는 **assemble** task의 의존성을 가지지 않고 테스트 실행시 자동으로 호출됩니다.

현재 하나의 ''빌드타입''만 테스트 가능합니다. 기본값은 **debug** 빌드 타입이며 다음과 같이 재구성할 수 있습니다.

```groovy
android {
  ...
  testBuildType "staging"
}
```

### 메인 APK와 테스트 APK간의 충돌 해결

Instrumentation 테스트를 실행할 때 메인 APK와 테스트 APK는 같은 classpath를 공유합니다. 만약 메인 APK와 테스트 APK에서 같은 라이브러리(예를들어 Guava)를 사용하지만 버전이 다르다면 Gradle 빌드는 실패할 것입니다. 만약 gradle이 인지하지 못했다고 하더라도 당신의 앱은 테스트시와 일반적인 실행시 다르게(혹은 크래시가 나던지) 동작할 수 있습니다.

빌드를 성공하려면 양쪽 APK에서 동일한 버전의 라이브러리를 사용하세요. 만약 간접적인 의존성(build.gradle에 명시되지 않은 라이브러리) 때문에 에러가 발생한다면, 그 라이브러리를 필요로하는 구성("compile" 혹은 "androidTestCompile")에 최신버전의 의존성을 추가해주세요. 아니면 [Gradle의 resolution strategy 메커니즘](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.ResolutionStrategy.html)을 사용할 수도 있습니다.  의존성 트리는 `./gradlew :app:dependencides`와 `./gradlew :app:androidDependencies`의 명령으로 검사할 수 있습니다.
