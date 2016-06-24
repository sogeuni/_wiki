# `base_path` 설정 시 문제해결

## GlobalTOC

`lib/gollum-lib/macro/global_toc.rb` 파일의 6라인을 다음과 같이 수정

```
#result = '<ul>' + @wiki.pages.map { |p| "<li><a href=\"/#{p.url_path}\">#{p.url_path_display}</a></li>" }.join + '</ul>'
result = '<ul>' + @wiki.pages.map { |p| "<li><a href=\"#{@wiki.base_path}/#{p.url_path}\">#{p.url_path_display}</a></li>" }.join + '</ul>'
```

## omnigollum

`lib/omnigollum.rb` 파일의 91라인을 다음과 같이 수정

```ruby
#redirect options[:route_prefix] + '/auth/' + options[:provider_names].first.to_s + "?origin=" +
redirect (request.script_name || '') + options[:route_prefix] + '/auth/' + options[:provider_names].first.to_s + "?origin=" +
```
