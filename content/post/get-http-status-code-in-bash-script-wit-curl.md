+++
date = "2016-07-29T15:44:25+08:00"
draft = true
title = "get http status code in bash script wit curl"

+++


A simple piece of bash to know if your service is up or not

```bash
response=$(
    curl YOUR_URL \
        --write-out %{http_code} \
        --silent \
        --output /dev/null \
)
test "$response" -ge 200 && test "$response" -le 299
```

You can replace the test by whatever suits you of course
