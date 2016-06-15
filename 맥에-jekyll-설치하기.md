# 맥에 jekyll 설치하기

[jekyll](http://jekyllrb-ko.github.io/)(지킬과 하이드의 지킬 맞습니다)은 정적 사이트 생성기입니다. 또한 **[GitHub Pages](https://pages.github.com/)**의 내부 엔진이기도 해서 github에 웹페이지를 호스팅하기 위해 많이 쓰지요.

게다가, jekyll은 블로그 친화적이라 이를 잘(?) 이용하면 공짜로 블로그를 호스팅 할 수 있습니다. github에 웹페이지를 호스팅하기 위해서는 먼저 jekyll을 깔아야 하는데 여기서는 mac 에서 jekyll 설치하면서 생긴 오류들에 대해 포스팅하고자 합니다.

맥에서 jekyll을 까는건 아래의 한 줄로 충분합니다. (보통은 루비가 깔려있으므로.. 단, Xcode Command-line tools가 설치되어 있지 않다면 먼저 설치해 주어야 합니다.)

```bash
gem install jekyll
```

하지만 경험상 세상일이 한방에 잘 되는 경우는 잘 없었던거 같네요. 제 경우엔 다음과 같은 문제가 발생하였습니다.

```bash
darrens-MacBook-Pro:_posts darrenpark$ gem install jekyll
Fetching: liquid-2.6.3.gem (100%)
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory.
```

말 그대로 해당 폴더에 접근권한이 없다는 에러인데, 이거 해결하려고 `sudo`를 쓴다던가 폴더의 권한을 바꾼다던가 하는건 권장하지 않는다고 하니 다른 방법을 찾아보기로 했습니다.

먼저 RVM 설치합니다. 참고로 Ruby Version Manager의 약자로 여러 버전의 루비를 쉽게 관리할 수 있는 스크립트라네요. 아래 **\\**는 오타가 아니에요.

```bash
\curl -L https://get.rvm.io | bash -s stable
```

설치후에는 다음과 같이 rvm 스크립트를 현재 쉘에 로드해줍니다.

```bash
source ~/.rvm/scripts/rvm
```

그런다음에 rvm을 이용하여 루비를 설치합니다.

```bash
rvm install ruby
```

뭔가 잘 진행되는듯 하다가 갑자기 또 에러를 내뱉습니다.

```bash
...
++ printf %b 'Failed to update Homebrew, follow instructions here:
    https://github.com/Homebrew/homebrew/wiki/Common-Issues
and make sure `brew update` works before continuing.\n'
Failed to update Homebrew, follow instructions here:
    https://github.com/Homebrew/homebrew/wiki/Common-Issues
and make sure `brew update` works before continuing.
++ return 1
Requirements installation failed with status: 1.
```

뭔진 잘 모르겠지만 `brew update`를 해보라기해 해봤습니다만 결과는 똑같네요.

```bash
darrens-MacBook-Pro:~ darrenpark$ brew update
error: The following untracked working tree files would be overwritten by merge:
	Library/Formula/gupnp-tools.rb
	Library/Formula/pdf2svg.rb
	Library/Formula/py3cairo.rb
Please move or remove them before you can merge.
Aborting
Error: Failure while executing: git pull -q origin refs/heads/master:refs/remotes/origin/master
```

또다시 폭풍 검색후에 다음과 같은 방법으로 해결하였습니다.

```bash
darrens-MacBook-Pro:git darrenpark$ cd /usr/local/git/
darrens-MacBook-Pro:git darrenpark$ git reset --hard FETCH_HEAD
HEAD is now at 8253821 pandoc-citeproc: update 0.7.3 bottle.
```

이후에 다시 `rvm install ruby`를 실행하면 정상적으로 설치가 진행됩니다.

설치된 ruby의 버전을 체크해 보니 처음과는 버전이 달라졌습니다.

```bash
darrens-MacBook-Pro:git darrenpark$ ruby -v
ruby 2.2.1p85 (2015-02-26 revision 49769) [x86_64-darwin14]
```

마지막으로 다시 처음으로 돌아가서 `gem install jekyll` 하면 에러없이 설치가 됩니다. 설치를 확인하기 위해서 `jekyll -v`로 버전확인을 하니 **jekyll 2.5.3**이라고 잘 나옵니다.

이렇게 우여곡절끝에 jekyll을 깔고 블로그를 셋팅할 준비가 되었습니다.