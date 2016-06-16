# GitHub Pages를 Maven 저장소로 사용하기

자신이 만든 라이브러리를 maven으로 배포하기 위해서는 [Nexus](http://www.sonatype.com/nexus-repository-sonatype)등과 같은 무료 maven repository를 이용하거나, 본인의 서버에 maven repository를 구축하여 배포할 수 있는데, 왠만큼 큰 규모의 프로젝트가 아닌이상 이런 방법으로 배포하는 것은 배보다 배꼽이 더 클 수가 있습니다.

따라서, 이 글에서는 GitHub Pages를 이용하여 간단하게 maven 저장소를 만드는 방법에 대해 설명합니다.

## GitHub Pages 만들기

자신이 배포하는 라이브러리가 하나 이상일 경우가 있겠죠. 하지만 배포하는 maven 주소를 하나로 유지하기 위해서 여기서는 자신의 GitHub에 **repo**라는 이름의 하나의 repository를 만들도록 하겠습니다. 이름이 맘에 들지 않는다면 바꾸셔도 상관없습니다. 이제 여기서 지지고 볶고 할거에요. 

먼저 [[GitHub Pages를 이용한 웹호스팅]]문서를 참고하여 **repo** 프로젝트를 위한 GitHub 페이지를 생성합니다. **gh-pages** 브랜치에 작업하는거 잊지 마시구요. http://sogeuni.github.io/repo/ 로 들어가 보시면 페이지가 만들어 진 것을 확인할 수 있습니다. 이제 이곳에 maven artifact 들을 올리면 됩니다. 

https://github.com/sogeuni/repo 를 샘플로 만들어 두었으니 참고하시기 바랍니다.

## Maven artifacts 만들기

앞에서 maven 저장소로 사용할 GitHub 저장소를 만들었으니 두번째로 할 일은 실제로 올리기 위한 Maven artifact를 만드는 것입니다. artifact를 딱히 번역할 말이 없는데 쉽게 말해 library와 그것을 설명하는 pom.xml (Project Object Model) 파일등의 모음이라고 할 수 있겠습니다.

다행이 gradle에는 [maven 플러그인](https://docs.gradle.org/current/userguide/maven_plugin.html)이 있습니다. 따라서, 안드로이드 프로젝트의 메인모듈의 build.gradle 파일에 다음과 같이 task를 추가해 줍니다.

```gradle
uploadArchives {
    repositories.mavenDeployer {
        pom.groupId = "net.l2r2"
        pom.artifactId = "testlib"
        pom.version = android.defaultConfig.versionName
        repository(url: uri("repo"))
    }
}
```

이때 maven 플러그인을 사용하겠다는 구문을 빠뜨리면 sync시 에러가 발생할 것입니다.

```gradle
apply plugin: 'maven'
```

groupId와 artifactId는 라이브러리의 package 명을 참조하여 만들었고, version은 라이브러리의 안드로이드 **versionName**을 가져오도록 했습니다. 그리고 repository는 아까 만든 저장소의 이름인 **repo**를 사용합니다. https://github.com/sogeuni/test-library 에 샘플 라이브러리 프로젝트를 올려놓았으니 참고하세요.

task를 만들었으면 라이브러리 프로젝트의 루트에서 `./gradlew uploadArchives`를 실행하면 프로젝트의 app 폴더 아래에 **repo**라는 폴더가 생성이 됩니다. 열어보시면 라이브러리와 pom 파일등 meven artifacts 가 생성된 것을 확인할 수 있습니다.

이렇게 생성된 artifacts를 앞서 만든 **repo** git 저장소에 올리면 됩니다. 역시나 외부에서 접근가능하도록 **gh-pages** 브랜치에 올리는 것을 잊지마시기 바랍니다. 라이브러리가 업로드된 모습은 https://github.com/sogeuni/repo/tree/gh-pages 를 참고하세요.

## 라이브러리 사용하기

이제 다른 프로젝트에서 github maven 저장소에 올린 라이브러리를 import해서 사용할 수 있게 되었습니다. 라이브러리를 사용하기 위해서는 두가지를 추가해주어야 합니다.

먼저, 프로젝트 루트의 **build.gradle** 파일에 **repository**를 추가해 줍니다.

```gradle
repositories {
   maven {
       url "http://sogeuni.github.io/repo/"
   }
}
```

url은 처음에 만든 GitHub Page 의 주소를 넣으시면 됩니다.

다음으로, 라이브러리를 사용할 모듈의 **build.gradle** 파일에 다음과 같이 **dependency**를 추가합니다.

```gradle
dependencies {
    ...
    compile 'net.l2r2:testlib:1.1@aar'
}
```

이제 어플리케이션에서 라이브러리를 사용할 수 있게 되었습니다.
