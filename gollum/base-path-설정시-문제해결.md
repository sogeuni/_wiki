# `base_path` 설정 시 문제해결

## GlobalTOC

GlobalTOC 매크로의 링크에 base_path가 적용되지 않아 링크가 제대로 열리지 않음

**해결**: `lib/gollum-lib/macro/global_toc.rb` 파일의 6라인을 다음과 같이 수정

```
#result = '<ul>' + @wiki.pages.map { |p| "<li><a href=\"/#{p.url_path}\">#{p.url_path_display}</a></li>" }.join + '</ul>'
result = '<ul>' + @wiki.pages.map { |p| "<li><a href=\"#{@wiki.base_path}/#{p.url_path}\">#{p.url_path_display}</a></li>" }.join + '</ul>'
```

## omnigollum

omnigollum 의 callback url에 base_path가 적용되지 않아 제대로 열리지 않음

**해결**: `lib/omnigollum.rb` 파일의 91라인을 다음과 같이 수정 (https://github.com/arr2036/omnigollum/pull/30/commits/6274a0904aa86e666be0ae19c3b0e9538af8aff6)

```ruby
#redirect options[:route_prefix] + '/auth/' + options[:provider_names].first.to_s + "?origin=" +
redirect (request.script_name || '') + options[:route_prefix] + '/auth/' + options[:provider_names].first.to_s + "?origin=" +
```
