视频审核平台

##### 获取指定内容 /api/contents/check

```apl
curl 'http://test-vms.yingshidq.com.cn/api/contents/check?pushTime=2021-07-01,2021-08-04&source=&category=' \
  -H 'Connection: keep-alive' \
  -H 'Accept: application/json, text/plain, */*' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36' \
  -H 'Project-Id: 1' \
  -H 'Referer: http://test-vms.yingshidq.com.cn/h5/dq_insideWeb/dq_examine-vms_fe/' \
  -H 'Accept-Language: zh-CN,zh;q=0.9' \
  -H 'Cookie: Auth-Token=eyJhbGciOiJIUzUxMiJ9.eyJleHAiOjE2MjgwNzk0NTQsInVzZXJJZCI6MjJ9.qGZu0TZfbDYrR8fp6zfNX2uM1hx08Z-fpJZypmuiO3hqOplmRav4Q7noAQN-lVwCMN7DKkBZ7Mq1P-bXHGHNGA' \
  --compressed \
  --insecure
```

##### 获取一级分类 /vms/common/v1/categories

```apl
curl 'http://test-vms.yingshidq.com.cn/vms/common/v1/categories?name=' \
  -H 'Connection: keep-alive' \
  -H 'Accept: application/json, text/plain, */*' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36' \
  -H 'Project-Id: 1' \
  -H 'Referer: http://test-vms.yingshidq.com.cn/h5/dq_insideWeb/dq_examine-vms_fe/' \
  -H 'Accept-Language: zh-CN,zh;q=0.9' \
  -H 'Cookie: Auth-Token=eyJhbGciOiJIUzUxMiJ9.eyJleHAiOjE2MjgwNzk1NjQsInVzZXJJZCI6MjJ9.SHQOSfIacZ3oL3cYuipnEWMyzouVXMP35piCthDhRNWVDkkHEtB0D1Yu-e8XQy3aIYT5dssY-lYIWsFLrZA44A' \
  --compressed
```

##### 获取来源 (渠道分类) /vms/common/v1/sources

```apl
curl 'http://test-vms.yingshidq.com.cn/vms/common/v1/sources?name=' \
  -H 'Connection: keep-alive' \
  -H 'Accept: application/json, text/plain, */*' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36' \
  -H 'Project-Id: 1' \
  -H 'Referer: http://test-vms.yingshidq.com.cn/h5/dq_insideWeb/dq_examine-vms_fe/' \
  -H 'Accept-Language: zh-CN,zh;q=0.9' \
  -H 'Cookie: Auth-Token=eyJhbGciOiJIUzUxMiJ9.eyJleHAiOjE2MjgwNzk1NjQsInVzZXJJZCI6MjJ9.SHQOSfIacZ3oL3cYuipnEWMyzouVXMP35piCthDhRNWVDkkHEtB0D1Yu-e8XQy3aIYT5dssY-lYIWsFLrZA44A' \
  --compressed \
  --insecure
```



