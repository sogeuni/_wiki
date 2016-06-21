# PacktPub FREE eBook 자동 다운로드하기

[Packt Publishing](https://www.packtpub.com/packt/offers/free-learning)에서는 자사의 서적중 매일 한권씩 free eBook으로 제공하고 있습니다. 하지만, 24시간마다 책이 바뀌기 때문에 매일 꼬박꼬박 받지 않으면 지나간 책은 더이상 받지 못하게 됩니다.

매일 클릭하는 귀찮음과 까먹는 걸 해결하기 위해 스크립트를 만들어볼까 하다 혹시 몰라서 [github를 검색](https://github.com/search?o=desc&q=packt+free&ref=searchresults&s=stars&type=Repositories&utf8=%E2%9C%93)해 보니 아니나 다를까 고맙게도(?) 비슷한 고민을 하는 사람이 많았던거 같습니다. 

## packtpub-crawler

여러 스크립트가 많습니다만 그중에서 [packtpub-crawler](https://github.com/niqdev/packtpub-crawler)가 맘에 드네요.

일단 자동 다운로드 기능 말고도, email notify라던가 구글 드라이브 업로드 같은 추가 기능을 제공합니다. 파이썬이 설치된 환경에서 동작하며 자세한 설정방법은 github에 잘 설명되어 있습니다.

다운로드 테스트를 마친후 `crontab`에 다음과 같이 등록하여 매일 자동으로 다운로드 하도록 하였습니다.

```
1 8 * * * cd /path/to/packtpub-crawler && /usr/bin/python2.7 script/spider.py --config config/prod.cfg --all --extras --upload drive 
```

국내 시간으로 아침 8시에 새로운 책으로 갱신되므로 매일 아침 8시 1분에 수행되도록 하였습니다.
