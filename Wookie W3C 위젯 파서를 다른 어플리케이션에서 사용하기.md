# Wookie W3C 위젯 파서를 다른 어플리케이션에서 사용하기

원문: http://wookie.apache.org/docs/developer/parser.html

[Wookie](http://wookie.apache.org/index.html)에서 W3C 위젯의 패키지를 풀고 파싱하기 위해 사용하는 코드는 단독 라이브러리로 다른 프로젝트나 어플리케이션에서 사용할 수 있습니다. 파서는 메인 Wookie 소스트리의 `parser/java/` 내의 서브 프로젝트로 찾을 수 있습니다.

## Maven project에서 파서 사용하기

Maven 프로젝트에서 파서를 사용하려면 프로젝트에 다음과 같이 dependency 설정을 추가합니다.

```maven
<dependency>
  <groupId>org.apache.wookie</groupId>
  <artifactId>wookie-parser</artifactId>
  <type>jar</type>
  <scope>compile</scope>
  <version>1.0.0</version>
</dependency>
```

안드로이드 스튜디오를 사용한다면 `build.gradle`의 `dependencies` 섹션에 다음과 같이 추가해 주면 됩니다.

```groovy
dependencies {
  compile 'org.apache.wookie:wookie-parser:1.0.0'
}
```

## 파서 jar를 다운로드 하여 사용하기

Maven을 사용하기를 원하지 않는다면 [https://repository.apache.org/content/groups/snapshots/org/apache/wookie/wookie-parser/ Apache repository]에서 다운로드<ref>현재 라이브러리가 존재하지 않음</ref>하던지 [http://wookie.apache.org/docs/download.html 최신의 Wookie 릴리즈]에서 배포되는 사본을 이용할 수 있습니다. 이 경우 Wookie가 의존성을 가지는 라이브러리 역시 포함하여야 합니다. (**Jdom, icu4j, slf4j, commons-httpclient, commons-logging, commons-compress, commons-io, commons-lang**)

## 파서 빌드하기

파서를 .jar로 빌드하기 위해서는 파서 폴더에서 `ant publish-local`을 실행하며, 빌드된 jar파일은 `parser/dist`에 저장될 것입니다. 의존성 또한 `parser/dist` 폴더의 **ivy.xml**파일에 서술되어 있습니다.

## 파서 사용하기

파서는 `W3CWidgetFactory` 오브젝트를 사용하여 호출됩니다. 이 것은 설정이 가능한 속성들을 가지며, 아래에서 다시 설명할 것입니다. .wgt 파일을 파싱하기 위해 factory를 사용할 때, 대부분의 속성은 선택사항이지만 위젯의 압축을 풀기위한 유효한 디렉토리는 제공하여야 합니다.

예를들어, .wgt 파일을 풀고 콘솔에 위젯의 이름을 출력하는 간단한 예제는 다음과 같습니다.

```java
public static void main(String[] args) {
    File out = new File("out");
    File zipFile = new File("in/butterfly.wgt");

    W3CWidgetFactory fac = new W3CWidgetFactory();
    fac.setOutputDirectory(out.getAbsolutePath());
    fac.setLocalPath("/out")
    try {
        W3CWidget widget = fac.parse(zipFile);
        System.out.println(widget.getLocalName("en"));
    } catch (BadWidgetZipFileException e) {
        e.printStackTrace();
    } catch (BadManifestException e) {
        e.printStackTrace();
    }
}
```

W3CWidgetFactory는 로컬 파일시스템에서 뿐만 아니라 URL로 부터 직접 위젯을 다운로드하고 설치할 수도 있습니다.

```java
W3CWidget widget = fac.parse(new URL("http://foo.com/bar.wgt"));
```

만약 위젯이 `application/widget` content-type을 사용하지 않는다면 다음과 같이 **conent-type을 무시**하도록 설정하여야 하는것을 잊지마세요.

```java
W3CWidget widget = fac.parse(new URL("http://foo.com/bar.wgt"), true);
```

## 현지화된 엘리먼트(Localized elements)

W3CWidget 오브젝트의 대부분의 엘리먼트는 현지화되어 있으며, 이러한 엘리먼트들은 모두 **ILocalizedElement** 인터페이스를 구현합니다. 현지화된 엘리먼트는 LocalizationUtils 클래스를 이용하여 선호하는 로케일 목록을 기반으로 처리할 수 있습니다. 예를 들어, 위젯의 이름을 가장 선호하는 “fr” 로케일로 추출하기 위해서는 다음과 같이 사용할 수 있습니다.

```java
NameEntity[] names = widget.getNames().toArray(new INameEntity[widget.getNames().size()]);
String[] locales = new String[]{"fr"};
INameEntity name = (INameEntity)LocalizationUtils.getLocalizedElement(names, locales,widget.getDefaultLocale());
```

LocalizationUtils은 언어의 다양성과 확장성에 기초하여 적절한 언어를 선택하여 엘리먼트를 처리하기 위해 **icu4j** 라이브러리를 사용합니다.

## Outputter

파서 라이브러리는 [http://www.w3.org/TR/widgets/ W3C Widgets: Packaging and Configuration] 스펙에 따라 config.xml파일을 생성하는데 사용할 수 있는 **WidgetOutputter** 클래스를 가지고 있습니다.

W3CWidget 오브젝트의 config.xml을 String으로 내보내기 위해 다음과 같이 사용합니다.

```java
W3CWidget widget = myWidget;
WidgetOutputter outputter = new WidgetOutputter();
outputter.setWidgetFolder("/widgets");
String configXml = outputter.outputXMLString(widget);
```

OutputStream으로 내보내려면 다음과 같이 사용합니다.

```java
W3CWidget widget = myWidget;
WidgetOutputter outputter = new WidgetOutputter();
outputter.setWidgetFolder("/widgets");   
FileOutputStream out = new FileOutputStream(new File("config.xml"));
outputter.outputXML(widget, out);
```

## Factory 속성 레퍼런스

### outputDirectory
위젯이 저장될 디렉토리입니다. factory는 이 디렉토리내에 위젯의 식별자를 이름으로 하는 디렉토리를 만들어 위젯의 압축을 풀 것입니다.

### startPageProcessor
**IStartPageProcessor** 인터페이스의 구현입니다. 이 속성을 설정하는 것은 위젯의 시작페이저 전에 처리되는 클래스(예를들어, 자바스크립트, HTML, 템플릿 코드 등)를 삽입하는 것을 허용합니다. 만약 이것이 설정되어 있지 않다면 factory에서는 어떠한 전처리도 하지 않습니다.

### locales
위젯에서 처리될 지원하는 로케일(BCP47 language-tags[^BCP47]) 입니다. 이것은 위젯의 시작파일, 아이콘 및 다른 지역화된 엘리먼트 중 어떤 것이 사용될지를 결정하며 기본값은 “en”입니다.

### encodings
위젯에서 처리될 지원하는 인코딩입니다. 이것은 시작파일에서 허용되는 사용자 정의 인코딩을 결정하며, UTF-8로 기본 설정되어 있습니다.

### localPath
위젯의 압축이 풀리는 기본 URL(“/myapp/widget” 등과 같은) 입니다. 시작파일의 URL은 기본 URL에 생성된 위젯 URL이 추가됩니다. 이 속성의 기본값은 “/widgets” 입니다.

### features
추가구현을 위한 feature이며 “http://wave.google.com”과 같이 IRIs[^IRIs]의 형태로 제공되어야 합니다. feature는 위젯에서 요청한 feature와 일치하여야 하며, 만약 위젯이 요청한 feature를 지원하지 않는다면 위젯 패키지 파싱시에 Exception을 발생시킵니다. 이 속성의 기본값은 빈 String array 입니다.

[^BCP47]: Tags for Identifying Languages
[^IRIs]: **I**nternationalized **R**esource **I**dentifiers