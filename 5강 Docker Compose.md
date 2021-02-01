### Docker Compose란 무엇인가?

-   docker compose는 다중 컨테이너 도커 어플리케이션을 정의하고 실행하기 위한 도구이다.

-   이제부터 compose를 이용해 새로운 어플리케이션을 만들면서 docker compose에 대해 학습해보자

### 어플리케이션 소스 작성하기

🙋‍♂️ 레디스란?

-   Redis (REmote Dictionary Server)는 메모리 기반의 키-값 구조 데이터 관리 시스템이며, 모든 데이터를 메모리에 저장하고 빠르게 조회할 수 있는 비관계형 데이터 베이스이다(NoSql).

🙋‍♀️ 레디스를 쓰는 이유?

-   메모리에 저장을 하기 때문에 MySql같은 데이터베이스에 데이터를 저장하는 것과 데이터를 불러 올 때 훨씬 빠르게 처리할 수 있다.

-   비록 메모리에 저장하지만 영속적으로도 보관이 가능하다.

-   그래서 서버를 재부팅해도 데이터를 유지할 수 있는 장점이 있다.

  📢 Node.js 환경에서 Redis 사용 방법

  - 먼저 redis-server를 작동시킨다.

-   그리고 redis 모듈을 다운 받는다.

-   레디스 모듈을 받은 후 레디스 클라이언트를 생성하기 위해서 레디스에서 제공하는 createClient() 함수를 이용해서 redis.createClient로 레디스 클라이언트를 생성해준다

-   하지만 여기서 redis server가 작동하는 곳과 Node.js 앱이 작동하는 곳이 다른 곳이라면 host 인자와 port 인자를 명시해주어야 한다.

ex)
```js
    const client = redis.createClient({
        host : "https://redis-server.com",
        port : 6379
    })
```

👀 도커 환경에서 레디스 클라이언트 생성시 주의사항

-   보통 도커를 사용하지 않는 환경에서는 Redis 서버가 작동하고 있는 곳의 host 옵션을 URL로 위의 예시처럼 주면 되지만, 도커 Compose를 사용할때는 host 옵션을 docker-compose.yml 파일에 명시한 컨테이너 이름으로 주면 된다

ex)
```js
    const client = redis.createClient({
        host : "redis-server",
        port : 6379
    })
```

소스 코드

``` js

const express = require("express");
const redis = require("redis");

const client = redis.createClient({
  host: "redis-server",
  port: 6379,
});

const app = express();

// 숫자는 0부터 시작합니다.
client.set("number", 0);

app.get("/", (req, res) => {
  client.get("number", (err, number) => {
    // 현재 숫자를 가져온 후 1 증가
    // numer가 string으로 오기에 parseInt로 변환
    client.set("number", parseInt(number, 10) + 1);
    res.send(`숫자가 1씩 올라갑니다. 숫자 : ${number}`);
  });
});

app.listen(8000, () => console.log("Server is Running"));

```

### Dockerfile 작성하기

-   Nodejs를 위한 이미지를 만들기 위해서 Dockerfile을 작성하는 것이기에 저번에 nodejs 앱을 위한 Dockerfile과 똑같이 만들어 주면 된다.


### Docker containers간 통신 할 때 나타나는 에러

- 이제 어플리케이션 소스와 도커 파일까지 작성했으니 실제로 어플을 실행해보자.

- 우선 어플이 어떤식으로 실행이 되는지 살펴보자.


✔ 컨테이너(노드 JS앱 + redis 클라이언트) + 컨테이너(레디스 서버)

🙌 실행 흐름

- 레디스 클라이언트가 작동하려면 레디스 서버가 켜져있어야 하기 때문에 먼저 레디스 서버를 위한 컨테이너를 실행하고 노드 js를 위한 컨테이너를 실행해보자

`docker build 도커아이디/앱이름 ./`

`docker run  -d -p 5000:8000 이미지 이름` 

🤦‍♂️ 그런데 !! 레디스 서버를 위한 컨테이너를 실행하고 노드 js를 위한 컨테이너를 실행하면 에러 발생

``` 
Error: Redis connection to redis-server:6379 failed - getaddrinfo ENOTFOUND redis-server redis-server:6379
    at GetAddrInfoReqWrap.onlookup [as oncomplete] (dns.js:56:26)
```

📌 에러가 발생하는 이유

- 서로 다른 컨테이너에 있는데 컨테이너 사이에는 아무런 설정 없이는 접근을 할 수 없기에 노드 JS앱에서 레디스 서버에 접근 할 수 없어서 에러 발생.

😎 그러면 어떻게 컨테이너 사이에 통신을 할 수 있을까?

- 멀티 컨테이너 상황에서 쉽게 네트워크를 연결 시켜주기 위해서 Docker Compose를 이용하면 된다.

