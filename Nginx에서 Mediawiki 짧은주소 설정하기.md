# Nginx에서 Mediawiki 짧은주소 설정하기

Nginx에서 `l2r2.net/w/index.php?title=Blahblah` 를 `l2r2.net/w/Blahblah` 로 접속하기위한 **rewrite** 설정 

* `/etc/nginx/sites-enabled/default` 혹은 enable 된 다른 설정파일

```linux-config
location / {
  index index.php index.html;
  try_files $uri $uri/ @redirect;
}

location @redirect {
  rewrite ^/([^?]*)(?:\?(.*))? /w/index.php?title=$1&$2 last;
  rewrite ^/w([^?]*)(?:\?(.*))? /w/index.php?title=$1&$2 last;
}

location ^~ /maintenance/ {
  return 403;
}
```

* `LocalSettings.php`

```php
$wgScriptPath = "/w";
$wgArticlePath = "/w/$1";
$wgUsePathInfo = true;
```

수정후 `sudo service nginx reload`로 설정을 재적용한다.
