에러 - 필터에서 발생하는 문자열 filter-hanlder-after-error 를 캐치

```ruby
fields @logStream, @message
| parse @message /[\w\.]+[\s|\t]+:[\s|\t]+(?<err>filter-handler-after-error):[\s](?<reqId1>[\w\d\-]+)[\s](?<exception>[\w\s]+)/
| filter @message like /(?i)filter-handler-after-error:/
| filter @logStream like /builder-server/
```

1l: 보여질 필드 logStream(링크), message(메시지내용)
2l: message 파싱 > err필드, reqId1필드, exception필드 > 도합 보여질 필드 5개,
3l: 메시지 필터 like "filter-handler-after-error:" 
4l: 로그스트림 필터 like "builder-server"

테스트 스트링

```ruby
2021-07-10 02:41:23.246 ERROR 1 --- [or-http-epoll-1] c.o.c.b.config.filter.BaseRouteFilter    : filter-handler-after-error: e752be3d-11964 No such User
2021-07-08 04:43:40.705 ERROR 1 --- [ndedElastic-440] c.o.c.b.config.filter.BaseRouteFilter    : filter-handler-after-error: 4bb73dfc-22618 Image fetch failure
```

grep결과

![error_grep](https://raw.githubusercontent.com/mycode01/linkimages/master/aws_dashboard/ss_0712.png)



슬로 - 로그에 느렸던 api에 태그를 붙여주셔야함 

```ruby
fields @logStream, @message
| parse @message /[\w\.]+[\s|\t]+:[\s|\t]+(?<slow>SLOW_API):[\s]+end[\w\d,:\s\(\)-]+path[\s\:]+(?<path>[\/\w\d\_\-]+)/
| filter @message like /(?i)SLOW_API/
| filter @logStream like /builder-server/
```

1l: 보여질 필드 logStream, message
2l: message 파싱 > slow, path > 도합 보여질 필드 4개,
3l: 메시지 필터 like "SLOW_API"
4l: 로그스트림 필터 like "builder-server"

테스트 스트링

```ruby
2021-07-12 07:20:48.862  INFO 1 --- [ndedElastic-772] c.o.c.b.config.filter.BaseRouteFilter    : SLOW_API: end(after): c8d990dc-37841 200 OK, elapsed : 1947ms, path : /builder/internal/uploadImage
2021-07-12 01:06:40.097  INFO 1 --- [ndedElastic-671] c.o.c.b.config.filter.BaseRouteFilter    : SLOW_API: end(after): 573e878b-34750 200 OK, elapsed : 8643ms, path : /builder/order/makeBlueprint
```

grep결과

![grep_slowapi](https://raw.githubusercontent.com/mycode01/linkimages/master/aws_dashboard/ss_0712_2.png)

꺾은선 그래프 - 위 2개 쿼리 내용과 비슷함

```ruby
parse @message /[\w\.]+[\s|\t]+:[\s|\t]+(?<err>filter-handler-after-error):[\s](?<reqId1>[\w\d\-]+)[\s](?<exception>[\w\s]+)/
| parse @message /[\w\.]+[\s|\t]+:[\s|\t]+(?<slow>SLOW_API):[\s]+end[\w\d,:\s\(\)-]+path[\s\:]+(?<path>[\/\w\d\_\-]+)/
| filter @message like /(?i)SLOW_API|filter-handler-after-error:/
| filter @logStream like /builder-server/
| stats count(err), count(slow) by bin(1h)
```

1l: message 파싱 > err, reqId1, exception
2l: message 파싱 > slow
3l: 메시지 필터 like "SLOW_API" or "filter-handler-after-error:"
4l: 로그스트림 필터 like "builder-server"
5l: 통계명령 count(필드err), count(필드slow) 기준 1시간

참고문서 
https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html