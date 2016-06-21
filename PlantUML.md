# PlantUML

PlantUML은 text기반의 간단한 스크립트 언어로 UML 다이어그램을 만들 수 있는 오픈소스 도구입니다. text 기반의 DSL[^DSL]이므로 형상 관리하기가 용이하며 다양한 어플리케이션에서 쉽게 적용할 수 있습니다. 또한, 저처럼 그림 솜씨가 형편없어도 쉽게(?) 다이어그램을 만들 수 있겠네요.

## PlantUML 서버

[PlantUML 서버](https://github.com/plantuml/plantuml-server)는 즉석에서 UML 다이어그램을 생성하는 웹 어플리케이션 입니다.

### 로컬에 PlantUML 서버 설치하기

PlantUML 서버는 jdk 1.6 이상과 maven 3.0.2 이상을 필요로 합니다. 또한, UML을 그리기 위해 [graphviz](http://www.graphviz.org/)를 사용합니다.

다음은 우분투 환경에서 PlanUML을 설치하는 방법입니다.

```shell
sudo apt-get install maven graphviz openjdk-7-jdk git-core  
git clone https://github.com/plantuml/plantuml-server.git
cd plantuml-server
mvn package
```

패키지 빌드가 끝났다면 [jetty 서버](http://www.eclipse.org/jetty/)를 이용하여 로컬 서버를 시작할 수 있습니다.

```
mvn jetty:run
```

서버는 `http://localhost:8080/plantuml`로 접속 가능합니다.

### 톰캣을 이용하여 구동하기

먼저 톰캣을 설치합니다.

```
sudo apt-get install tomcat7
```

위에서 빌드가 성공했다면 `target`폴더에 `plantuml.war` 파일이 생성되어 있을 것입니다. 이 파일을 톰캣의 `webapps` 폴더로 복사한다음 tomcat 서버를 재구동 합니다.

```
sudo cp plantuml.war /var/lib/tomcat7/webapps/plantuml.war
sudo chown tomcat7:tomcat7 /usr/share/jetty/webapps/plantuml.war
sudo service restart tomcat7
```

톰캣은 기본적으로 `8080`포트로 동작합니다. 포트를 변경하려면 `/etc/tomcat7/server.xml`파일을 수정한 후 서버를 재구동 합니다.


[^DSL]: Domain Specific Language