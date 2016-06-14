# Android Auto 앱 테스트하기

안드로이드 오토용 앱을 테스트하기 위해 구글은 Desktop Head Unit (DHU)라는 헤드유닛 에뮬레이터를 제공합니다. DHU는 PC를 자동차의 헤드유닛처럼 에뮬레이션하여 USB로 휴대폰을 연결하여 안드로이드 오토 앱을 실행할 수 있게 합니다.

## DHU 설치하기

DHU는 윈도우, 맥, 리눅스 OS에 설치가능하며 설치 절차는 다음과 같습니다.

* 먼저 모바일 장치의 **개발자 옵션**을 활성화 합니다. 개발자 옵션을 활성화 하기 위해서는 **설정 > 휴대전화 정보 > 빌드 번호** 를 일곱번 탭하면 됩니다.<ref>단말에 따라 위치는 조금씩 다를 수 있습니다. 삼성폰의 경우 **설정 > 디바이스 정보 > 소프트웨어 정보 > 빌드번호** 입니다.</ref>
* 개발환경 및 설치될 모바일 장치의 안드로이드 버전은 5.0(API level 21) 이상이어야 합니다. 
* 모바일 장치에 [https://play.google.com/store/apps/details?id=com.google.android.projection.gearhead&hl=en Android Auto app]을 설치합니다. 하지만 현재 국내에서는 마켓을 통해 Android Auto 앱을 설치할 수가 없습니다. google에 "Android Auto apk" 등으로 검색해서 apk를 설치할 수 있습니다.
* **SDK Manager**를 열고 ```SDK Tools``` 탭을 선택한 다음 **Android Auto Desktop Head Unit emulator**를 설치합니다. DHU는 ```<sdk>/extras/google/auto/``` 디렉토리에 설치됩니다. sdk의 위치는 SDK Manager 상단에 **Android SDK Location**을 확인하시면 됩니다.
* 리눅스에서 DHU를 실행하려면 ```portaudio```, ```sdl2```, ```sdl2_ttf``` 라이브러리가 필요합니다.
```bash
$ sudo apt-get install libsdl2-2.0-0 libsdl2-ttf-2.0-0 libportaudio2 libpng12-0
```

## DHU 실행하기

개발 PC와 모바일 장치를 USB로 연결하고, ADB를 통해 헤드유닛 서버를 연결하기 위한 설정을 하여 DHU를 실행합니다. 실행방법은 다음과 같습니다.

1. 모바일 장치에서 좀전에 설치한 안드로이드 오토앱을 실행한 다음 **개발자 모드**를 활성화 합니다. 활성화 화는 방법은 ''Android Auto''라고 적혀 있는 툴바를 열번 탭하면 됩니다. 개발자 모드가 활성화 되었다는 toast가 나오면 메뉴에 이전에 보이지 않던 Developer settings 등의 메뉴가 보이게 됩니다. 이 작업은 최초 한번만 수행하면 됩니다.
2. 앱 메뉴의 **Start head unit server**를 선택하여 서버를 실행합니다. 실행되면 notification 영역에 서버가 실행되고 있다고 표시됩니다.
3. 오토 앱의 기어 모양의 아이콘을 클릭하여 설정으로 들어간 다음 **Add new cars to Android Auto** 옵션이 활성화 되어 있는지 확인합니다. 만약 비활성화 되어 있는경우 에뮬레이터연결시에도 Auto 모드로 동작하지 않으므로 옵션을 활성화 시킵니다.
4. 모바일 장치를 PC에 USB로 연결합니다.
5. 모바일 장치의 화면 잠금이 해제되었는지 확인합니다. 그렇지 않으면 DHU가 실행되지 않습니다.
6. PC와 모바일 장치를 동일한 5277 포트로 연결하기 위해 다음의 ```adb``` 명령을 실행합니다. 이 설정은 DHU가 모바일에 실행중인 헤드유닛 서버에 TCP 소켓으로 연결되도록 합니다.
  
  ```
  $ adb forward tcp:5277 tcp:5277
  ```
7. ```<sdk>/extras/google/auto/``` 디렉토리의 ```desktop-head-unit.exe``` (윈도우)나 ```./desktop-head-unit``` (맥 또는 리눅스) 명령어를 실행하여 DHU를 시작합니다.
  
  ```shell
  $ cd <sdk>/extras/google/auto&#10;
  $ ./desktop-head-unit
  ```
  기본적으로 헤드유닛 서버는 5277번 포트로 연결됩니다. SSH를 사용하는 등, host나 port를 변경해야 한다면 다음의 예제처럼 ```desktop-head-unit --adb <[localhost:]port>``` flag를 사용하여 변경이 가능합니다.
  ```shell
  $ ./desktop-head-unit --adb 5999
  ```
  기본으로 실행되는 DHU는 가장 일반적인 형태의 터치스크린 입력을 가지는 헤드유닛을 에뮬레이팅하며, 터치스크린 입력은 마우스 클릭으로 시뮬레이트합니다. 만약 컨트롤러를 가지는 DHU를 실행시키려면 ```-i controller``` flag를 사용합니다.
  ```shell
  $ ./desktop-head-unit -i controller
  ```
  단, 이경우 터치스크린 입력은 되지 않으며 컨트롤러는 키보드 단축키로 맵핑됩니다. 컨트롤러키와 그에 할당된 키보드 단축키는 [여기](https://developer.android.com/training/auto/testing/index.html#cmd-bindings)를 참고하세요.

DHU가 실행되면 커맨드라인이나 키보드 단축키를 이용하여 앱을 테스트할 수 있습니다.

## DHU 명령어 입력하기

안드로이드 오토의 기능을 이용하여 앱을 테스트 하기위애 DHU는 몇가지 명령을 가집니다. 자세한 명령옵션은 [여기](https://developer.android.com/training/auto/testing/index.html#cmd-bindings)를 참고하시기 바랍니다.

### Day mode와 night 모드 전환

일반적으로 차량의 헤드유닛은 밤에는 운전에 방해가 되지 않도록 어두운 색상으로 변경이 됩니다. 안드로이드 오토 역시 이러한 기능을 지원하여 낮과 밤에 따라 다른 색상구성을 가집니다. 따라서 작성한 어플리케이션도 day와 night 모드에서 테스트를 하여야 합니다. 헤드유닛의 모드를 변경하기 위해서는 DHU를 실행안 터미널에서 ```daynight``` 입력하던가 키보드의 ```N```키를 누르면 됩니다.

### 마이크 테스트

DHU는 마이크를 이용한 음성입력을 지원합니다. 또한, 다음의 명령어를 사용하여 미리 녹음된 음성 wav 파일을 음성입력으로 사용할 수도 있습니다.

```shell
$ mic play <sound_file_path>/<sound_file>.wav
```

```<sdk>/extras/google/auto/voice/``` 디렉토리에 일반적인 명령어 음성파일이 몇개 있습니다.

Here's an inline link to [Google](http://www.google.com/).
Here's a reference-style link to [Google][1].
Here's a very readable link to [Yahoo!][yahoo].

  [1]: http://www.google.com/
  [yahoo]: http://www.yahoo.com/
