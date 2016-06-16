# Pywikibot 설치

**[Pywikibot](https://www.mediawiki.org/wiki/Manual:Pywikibot)**은 MediaWiki 기반의 사이트에서 동작하는 자동화 도구의 모음입니다. Pywikibot을 이용하여 wiki에 자동으로 글을 올리는 등의 작업을 자동으로 수행할 수 있습니다. 할 수 있는 작업들은 [Script 매뉴얼](https://www.mediawiki.org/wiki/Manual:Pywikibot/Scripts)을 참고하세요. 여기서는 Pywikibot을 설치하는 방법에 대해 설명합니다.

## 선행 조건

Pywikibot은 파이썬 스크립트로 작성되어 있습니다. 따라서 파이썬이 설치되어 있어야 합니다. 파이썬의 버전은 2.7.2이나 3.3 이상이면 가능합니다. 파이썬 버전은 `python --version`로 확인할 수 있습니다.

## Pywikibot 다운로드

http://tools.wmflabs.org/pywikibot/core_stable.tar.gz 파일을 다운받아 적절한 경로에 압축을 풉니다.

```bash
wget http://tools.wmflabs.org/pywikibot/core_stable.tar.gz
tar -xzvf core_stable.tar.gz
```

## Pywikibot 설정

### Family 파일 생성

먼저 미디어위키를 사용하고 있는 서버설정파일(Family 파일)을 생성하기 위해 `generate_family_file.py`을 실행합니다.

```bash
$ ./generate_family_file.py 
Usage: ./generate_family_file.py <url> <short name>
Example: ./generate_family_file.py https://www.mywiki.bogus/wiki/Main_Page mywiki
This will create the file families/mywiki_family.py
Please insert URL to wiki: https://www.l2r2.net/w
Please insert a short name (eg: freeciv): l2r2
Generating family file from https://www.l2r2.net/w

==================================
api url: http://l2r2.net/w/api.php
MediaWiki version: 1.26.2
==================================

Determining other languages...
Loading wikis... 
  * ko... in cache
Writing pywikibot/families/l2r2_family.py... 
```

위와 같이 위키의 **URL**과 서버를 나타내는 **short name**을 정해 입력하면 `families/<short name>_family.py` 파일이 생성됩니다.

### User 파일 생성

다음으로 bot으로 사용할 유저 정보 파일을 생성하기 위해 `generate_user_files`를 실행합니다.

```bash
$ ./generate_user_files.py 
Error: Python module httplib2 >= 0.6.0 is required.
```

저의 경우 위과 같은 오류가 발생하였는데, **httplib2**라는 파이썬 모듈이 설치되어 있지 않아 생기는 오류이므로 httplib2 모듈을 설치합니다. 설치방법은 `apt-get`를 이용하는 방법과 `pip`를 이용하는 방법이 있습니다.

* 방법 1
 ```bash
 $ sudo apt-get install python-httplib2
 ```

* 방법 2 
 ```bash
 $ sudo apt-get install python-pip
 $ pip install httplib2

 Collecting httplib2
  Downloading httplib2-0.9.2.zip (210kB)
    100% |████████████████████████████████| 215kB 1.3MB/s 
Building wheels for collected packages: httplib2
  Running setup.py bdist_wheel for httplib2 ... done
  Stored in directory:  /home/sogeuni/.cache/pip/wheels/c7/67/60/e0be8ccfc1e08f8ff1f50d99ea5378e204580ea77b0169fb55
 Successfully built httplib2
 Installing collected packages: httplib2
 Successfully installed httplib2
 ```

httplib2를 설치한 후 다시 `generate_user_files`을 실행하면 다음과 같이 정상적으로 사용자를 등록할 수 있습니다.

```bash
$ ./generate_user_files.py 

Your default user directory is "/home/sogeuni/www/pywikibot/core_stable"
Do you want to use that directory? ([Y]es, [n]o): y
Do you want to copy user files from an existing Pywikibot installation? ([y]es, [N]o): n
Create user-config.py file? Required for running bots. ([Y]es, [n]o): y
 1: anarchopedia
 2: battlestarwiki
 3: commons
 4: i18n
 5: incubator
 6: l2r2
 7: lyricwiki
 8: mediawiki
 9: meta
10: omegawiki
11: osm
12: outreach
13: species
14: strategy
15: test
16: vikidia
17: wikia
18: wikiapiary
19: wikibooks
20: wikidata
21: wikimedia
22: wikinews
23: wikipedia
24: wikiquote
25: wikisource
26: wikitech
27: wikiversity
28: wikivoyage
29: wiktionary
30: wowwiki
Select family of sites we are working on, just enter the number or name (default: wikipedia): 6
The only known language: ko
The language code of the site we're working on (default: ko): ko
Username on ko:l2r2: SogeuniBot
Do you want to add any other projects? ([y]es, [N]o): n
Would you like the extended version of user-config.py, with explanations included? ([Y]es, [n]o): n
'/home/sogeuni/www/pywikibot/core_stable/user-config.py' written.
Create user-fixes.py file? Optional and for advanced users. ([y]es, [N]o): n
```

사이트와 봇으로 사용할 사용자의 이름과 언어를 지정해주면 `user-config.py`이 자동으로 생성됩니다.

사이트의 경우 좀전에 생성한 패밀리설정의 **short name**을 적어주거나 목록에서 찾아 번호를 입력하면 됩니다. 사용자의 경우 미디어위키 사이트에 이미 등록된 유저의 아이디를 적습니다.

## 동작 테스트

이제 설치가 완료되었습니다. 스크립트를 실행하기 위해서는 다음과 같이 사용합니다.

```bash
python pwb.py [name of the script]
```

실행가능한 스크립트들은 `scripts` 디렉토리에 있으며 다음과 같이 로그인/로그아웃이 가능합니다.

```bash
// 로그인
$ ./pwb.py login
Password for user SogeuniBot on l2r2:ko (no characters will be shown): 
Logging in to l2r2:ko as SogeuniBot
Logged in on l2r2:ko as SogeuniBot.

// 로그아웃
$ ./pwb.py login -logout
Logged out of l2r2:ko.
```

### 트러블슈팅

#### HTTP 301 Error

이 사이트의 경우 http를 자동으로 https로 리다이렉션하도록 되어 있습니다. 이럴경우 로그인시 다음과 같은 오류가 발생하였습니다.

```bash
$ ./pwb.py login
WARNING: Http response status 301
WARNING: Non-JSON response received from server l2r2:ko; the server may be down.
WARNING: Waiting 5 seconds before retrying.
```

이 경우 좀전에 생성한 family 파일을 열어 Family 클래스에 다음 구문을 추가해 줍니다.

```python
def protocol(self, code):
    """Return the protocol for this family."""
    return 'https'
```

#### CERTIFICATE_VERIFY_FAILED

역시나 https 사이트에 로그인 시 다음과 같이 SSL 에러가 나는 경우가 있습니다.

```bash
./pwb.py login
ERROR: Traceback (most recent call last):
  File "/home/sogeuni/www/pywikibot/core_stable/pywikibot/data/api.py", line 1556, in submit
    body=body, headers=headers)
  File "/home/sogeuni/www/pywikibot/core_stable/pywikibot/tools/__init__.py", line 1105, in wrapper
    return obj(*__args, **__kw)
  File "/home/sogeuni/www/pywikibot/core_stable/pywikibot/comms/http.py", line 279, in request
    r = fetch(baseuri, method, body, headers, **kwargs)
  File "/home/sogeuni/www/pywikibot/core_stable/pywikibot/comms/http.py", line 381, in fetch
    error_handling_callback(request)
  File "/home/sogeuni/www/pywikibot/core_stable/pywikibot/comms/http.py", line 293, in error_handling_callback
    raise FatalServerError(str(request.data))
FatalServerError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:590)
```

인증서가 정상적인데도 불구하고 발생하는 에러라 정확한 해결방법은 찾지 못했으나, 인증서 verify 하는 부분을 건너뛰도록 수정하면 오류는 발생하지 않습니다. `pywikibot/comms/http.py` 파일을 열어 다음과 같이 수정합니다.

```python
"""
kwargs.setdefault("disable_ssl_certificate_validation",
                  site.ignore_certificate_error())
"""
kwargs.setdefault("disable_ssl_certificate_validation",
                  True)
```

## 참고

* https://www.mediawiki.org/wiki/Manual:Pywikibot/Installation : 설치방법 문서
* https://www.mediawiki.org/wiki/Manual:Pywikibot/Scripts : 스크립트 설명 문서
