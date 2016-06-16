# Awaitility - 비동기 시스템의 쉬운 테스트를 위한 Jave DSL

비동기 시스템을 테스트 하는 일은 쉽지 않습니다. 쓰레드와 타임아웃, 거기다 동시성 이슈까지 핸들링해야 할 뿐 아니라 이런것을 처리하기 위해 들어간 코드들이 실제 테스트하고자 하는 코드를 잘 보이지 않게 합니다. Awaility는 읽기 쉽고 간결한 방식으로 비동기 시스템을 표현할 수 있는 DSL 입니다.

## 설치방법

Gradle을 사용할 경우 build.gradle에 다음을 추가합니다.

```bash
testCompile 'com.jayway.awaitility:awaitility-groovy:1.7.0'
```

다른 빌드툴에서의 설치방법은 https://github.com/jayway/awaitility/wiki/Getting_started 을 참고하시기 바랍니다.

Awaitility를 효과적으로 사용하기 위해서 다음의 메소드를 static import 하는 것을 추천합니다.

* `com.jayway.awaitility.Awaitility.*` It may also be useful to import these methods:
* `com.jayway.awaitility.Duration.*`
* `java.util.concurrent.TimeUnit.*`
* `org.hamcrest.Matchers.*`
* `org.junit.Assert.*`

## 간단한 예제

우리의 비동기 시스템에 다음과 같이 "add user" 메시지를 보낸다고 가정해 봅시다.

```java
publish(new AddUserMessage("John Doe"));
```

Test case에서 Awaitility는 데이터베이스가 업데이트 되었는지를 쉽게 확인할 수 있도록 도와 줍니다. 가장 간단한 형태로는 다음과 같이 표현할 수 있습니다.

```java
publish(new AddUserMessage("John Doe"));
```

Awaitility는 이제 데이터베이스에 사용자가 추가될 때 까지 테스트 실행을 block한 채 기다릴 것입니다. `newUserIsAdded`는 test case에서 직접 구현해야 하는 메소드 며, Awaitility가 대기를 멈추기 위해 필요한 조건을 지정해야 합니다.

```java
private Callable<boolean> newUserIsAdded() {
  return new Callable<boolean>() {
    public Boolean call() throws Exception {
      return userRepository.size() == 1; // The condition that must be fulfilled
    }
  };
}
```

기본적으로 Awaitility는 10초동안 user repository의 size가 1이 되지 않으면 `TimeoutException`를 발생하여 test를 실패로 간주합니다. 만약 timeout 시간을 조절하고 싶다면 다음과 같이 정의할 수 있습니다.

```java
await().atMost(5, SECONDS).until(newUserWasAdded());
```

## 코드 재사용을 위한 더 좋은 방법

Awaitility는 공통부분을 재사용할 수 있도록 조건을 분리할 수도 있습니다.  위의 예제는 다음과 같이 표현할 수 있습니다.

```java
await().until( userRepositorySize(), equalTo(1) );
```

`userRepositorySize` 메소드는 이제 Integer 형의 **Callable** 객체를 리턴합니다.

```java
private Callable<integer> userRepositorySize() {
  return new Callable<integer>() {
    public Integer call() throws Exception {
      return userRepository.size(); // The condition supplier part
    }
  };
}
```

조건을 지정하는 `equalTo`는 표준의 [Hamcrest](http://code.google.com/p/hamcrest/) matcher 입니다.

우리는 이제 `userRepositorySize`를 다른 테스트에서 재사용할 수 있습니다. 예를들어 동시에 3명의 사용자를 추가하는 테스트를 한다고 가정해 봅시다.

```java
publish(new AddUserMessage("User 1"), new AddUserMessage("User 2"), new AddUserMessage("User 3"));
```

이제 Hamcrest matcher를 업데이트 하는 것만으로 간단히 `userRepositorySize`를 재사용할 수 있습니다.

```java
await().until( userRepositorySize(), equalTo(3) );
```

## 장황함을 줄이기

Java는 매우 장황하기 때문에 Awitility는 Callable을 정의하지 않고도 같은 결과를 얻을 수 있는 방법 또한 제공합니다.

```java
await().untilCall( to(userRepository).size(), equalTo(3) );
```

`to`는 await 구문에 inline으로 정의할 수 있게 해주는 Awaitility 메소드 입니다. 어떤 옵션이 가장 좋을지는 use case와 가독성에 따라 달라집니다.

## 더 많은 기능들

Awaitility는 조건이 충족되었는지를 체크하기위해 폴링을 사용합니다. 폴링 interval과 delay(최초 폴링이 일어나기까지의 시간)은 쉽게 지정가능합니다.

```java
with().pollInterval(400, MILLISECONDS).and().pollDelay(2, SECONDS).await().forever().until(somethingHappens());
```

어떤 경우에는 하나의 테스트에 여러개의 await가 필요할 수 있는데 이럴때는 await의 이름을 만드는 것이 유용할 수 있습니다. 다음의 예제처럼요.

```java
publish(new AddUserMessage("User 1"));
await("user 1").until(userRepositorySize(), is(equalTo(1)));
publish(new AddUserMessage("User 2"));
await("user 2").until(userRepositorySize(), is(equalTo(2)));
```

만약 첫번째 await 구분에서 `TimeoutException` 이 발생하였다면 exception은 "user 1"을 표시할 것입니다.

## 결론

Awaitility는 Java에서 비동기 동작을 동기화할 필요가 있을 때 쉽게 사용할 수 있습니다. 하지만 폴링을 사용하기 때문에 정확한 성능측정이 필요할 때는 추천하지 않습니다. 이러한 경우에는 컴파일타임에 적용되는 AspectJ나 leverage와 같은 AOP 프레임워크를 사용하는 것이 좋습니다. 

## 참고
* http://www.jayway.com/2010/07/20/awaitility-java-dsl-for-easy-testing-of-asynchronous-systems/
* https://github.com/jayway/awaitility
