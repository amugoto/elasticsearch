# ElasticSearch + Kibana + Cerebro 로컬 테스트용 환경 세팅

# env

```env
STACK_VERSION=
ELASTIC_PASSWORD=
```

# 로그인 계정

> **username**: elastic  
> **password**: {ELASTIC_PASSWORD}

# Kibana

> 로그인 하고나서 Completing setup 단계에서 로딩이 계속돌면 새로고침 한번해주면 됩니다.

## 토큰 생성

```bash
> docker exec -it es bin/elasticsearch-create-enrollment-token --scope kibana
```

## verification code 생성

```bash
> docker exec -it kibana ./bin/kibana-verification-code
```

# cerebro

```bash
> docker cp es:/usr/share/elasticsearch/config/certs/http_ca.crt .
> openssl x509 -in ./http_ca.crt -out elastic.pem -outform PEM
> docker cp ./elastic.pem cerebro:/opt/cerebro/
> docker cp cerebro:/opt/cerebro/conf/application.conf ./application.conf
> docker cp ./application.conf cerebro:/opt/cerebro/conf/application.conf
> docker restart cerebro
```

## application.conf

1. 처음에 `application.conf` 파일을 cerebro 컨테이너에서 복사해옵니다.
2. 아래 내용을 복사해온 파일에 추가해주고 다시 cerebro 컨테이너에 넣어줍니다.
   - `hosts` 부분은 선택사항 입니다. 입력하면 굳이 로그인정보 다 입력안하고 접근 가능합니다.
3. `cerebro` 컨테이너를 재시작해줍니다.

```conf
# 필수
play {
  server.http.port = ${?CEREBRO_PORT}
  ws.ssl {
    trustManager = {
      stores = [
        { type = "PEM", path = "/opt/cerebro/elastic.pem" }
      ]
    }
    loose.acceptAnyCertificate=true
  }
}

# 선택사항
hosts = [
  {
    host = "https://{elasticsearch host}:{port}"
    name = "docker-cluster"
    auth = {
      username = "elastic"
      password = "{ELASTIC_PASSWORD}"
    }
  }
]
```
