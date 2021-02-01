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

