### 섹션 설명

📌 Node Js 공식 홈페이지에서 도커를 이용해 Node.js를 이용하는 예시 부분을 사용하여 도커를 실전에 도입하는 연습을 해보자

✔ 이번강에서 가장 중점적으로 봐야 할 것

-   Dockerfile을 어떤식으로 작성하는지

✔ 순서

1.  NodeJs App 만들기

2.  도커 이미지 생성 후 컨테이너에서 실행

### 도커 파일 작성하기

-   NodeJs 앱을 도커 환경에서 실행하려면 먼저 이미지를 생성하고 그 이미지를 이용해서 컨테이너를 실행한 후 그 컨테이너 안에서 NodeJs 앱을 실행해야 한다.

-   그래서 그 이미지를 먼저 생성하려면 Dockerfile을 작성해야 한다.

```js
# 베이스 이미지를 명시해준다.
FROM node:10

# 추가적으로 필요한 파일들을 다운로드 받는다.
RUN npm install

# 컨테이너 시작 시 실행될 명령어를 명시해준다.
CMD [ "node", "index.js" ]
```

🤷‍♀️ node 이미지를 사용한 이유는?

-   npm이 들어있는 베이스 이미지를 찾아야 하는데 그것들중 하나가 node 이미지이다.

🙋‍♀️ npm install은 무엇인가?

-   npm은 nodeJs로 만들어진 모듈을 웹에서 받아서 설치하고 관리해주는 프로그램이다.

-   npm install은 package.json에 적혀있는 종속성들을 웹에서 자동으로 다운 받아서 설치해주는 명령어이다.

-   결론적으로 현재 NodeJs 앱을 만들 때 필요한 모듈들을 다운 받아 설치하는 역할을 한다.

🙋‍♂️ "node", "index"는 무엇인가?

-   노드 웹서버를 작동시키려면 node+엔트리 파일 이름을 입력해야 한다.

이렇게 해서 dockerfile을 다 작성한거 같은데 `docker build ./`를 해서 dockerfile에 작성한 내용을 도커 서버에 보내줘서 이미지를 생성해보겠다.

🤦‍♂️ 에러 발생!!

-   package.json이 없다라는 에러가 발생

-   왜 이러한 에러가 발생하는지 살펴보자

### package.json 파일이 없다고 나오는 이유

- 도커 파일을 build 할 때 Node 베이스 이미지로 임시 컨테이너를 생성한다.

- 그리고 그 임시컨테이너로 이미지를 만든다.

-   하지만 그 임시컨테이너에는 package.json이 Node 이미지 파일 스냅샷에 포함되어 있지 않다

-   그래서 packge.json은 컨테이너 밖에 있는 상황이 된다.


📝 정리

-   npm install =>  어플리케이션 종속성을 다운 => 이렇게 다운 받을때 package.json을 보고 그곳에 명시된 종속성들을 다운 받아 설치 => 하지만 package.json 파일은 컨테이너 안에 없기에 에러 발생

📌 이러한 이유 때문에 `COPY`를 이용해서 package.json을 컨테이너 안으로 넣어줘야한다!!

`COPY package.json ./`

package.json => 복사할 파일 경로

./ => 컨테이너 내에서 파일이 복사될 경로

```js
# 베이스 이미지를 명시해준다.
FROM node:10

COPY package.json ./

# 추가적으로 필요한 파일들을 다운로드 받는다.
RUN npm install

# 컨테이너 시작 시 실행될 명령어를 명시해준다.
CMD [ "node", "index.js" ]
```

=> 또 다시 에러 발생, package.json 파일과 마찬가지로 index.js를 찾지 못하기 때문

```js
# 베이스 이미지를 명시해준다.
FROM node:10

// 전체를 copy
COPY ./ ./

# 추가적으로 필요한 파일들을 다운로드 받는다.
RUN npm install

# 컨테이너 시작 시 실행될 명령어를 명시해준다.
CMD [ "node", "index.js" ]
```

### 생성한 이미지로 어플리케이션 실행 시 접근이 안 되는 이유

📌 앞으로 컨테이너를 실행하기 위해 사용 할 명령어

`docker run -p 49160:8000 이미지 이름` (49160 포트번호는 개발자가 임의로 설정가능, 브라우저에서 접근하는 포트번호이다)

🙋‍♀️ 새롭게 추가된 부분은 무엇인가?

-   우리가 이미지를 만들때 로컬에 있던(package.json)등을 컨테이너에 복사해줘야 했다.

-   그것과 비슷하게 네트워크도 로컬 네트워크에 있던 것을 컨테이너 내부에 있는 네트워크에 연결을 시켜줘야한다.

### Working Directory 명시해주기

```js
WORKDIR /usr/src/app
```

-   이미지안에서 어플리케이션 소스코드를 갖고 있을 디렉토리를 생성하는 것이다.

-   그리고 이 디렉토리가 어플리케이션에 working 디렉토리가 된다.

🤷‍♂️ 그런데 왜 따로 working 디렉토리가 있어야 하는가?

`docker run -it magicnc7/nodejs ls`를 찍어보면

```js
Dockerfile  dev   index.js  media         opt                proc  sbin  tmp
bin         etc   lib       mnt           package-lock.json  root  srv   usr
boot        home  lib64     node_modules  package.json       run   sys   var
```

🤦‍♀️ 이렇게 workdir를 지정하지 않고 그냥 COPY할때 생기는 문제점

1. 혹시 이 중에서 원래 이미지에 있던 파일과 이름이 같다면?

ex) 베이스 이미지에 이미 HOME이라는 폴더가 있고 COPY를 함으로써 새로 추가되는 폴더 중에 HOME이라는 폴더가 있다면 중복이 되므로 원래 있던 폴더가 덮어씌어져 버린다.

2. 모든 파일이 한 디렉토리에 들어가버려 너무 정리 정돈이 안되있다.

=> 그래서 모든 어플리케이션을 위한 소스들은 WORK 디렉토리를 따로 만들어서 보관한다.

👊 만드는 방법

```js
# 베이스 이미지를 명시해준다.
FROM node:10

WORKDIR /usr/src/app

COPY ./ ./ 

# 추가적으로 필요한 파일들을 다운로드 받는다.
RUN npm install

# 컨테이너 시작 시 실행될 명령어를 명시해준다.
CMD [ "node", "index.js" ]
```

📌 WORKDIR 설정 후 터미널에 들어오면 기본적으로 work 디렉토리에서 시작하게 됨 

`docker run -it magicnc7/nodejs ls`

```js
Dockerfile  index.js  node_modules  package-lock.json  package.json
```

Root 디렉토리 확인하려면 
명령어 cd/

### 어플리케이션 소스 변경으로 다시 빌드하는 것에 대한 문제점

-   어플리케이션을 만들다 보면 소스 코드를 계속 변경시켜줘야 하며 그에 따라서 변경된 부분을 확인하면서 개발을 해나가야한다.

-   그래서 도커를 이용해서 어떻게 실시간으로 소스가 반영되게 하는지 알아보자.

🤦‍♂️ 현재 방식의 문제점

-   COPY ./ ./ 이 부분으로 인해서 소스를 변화시킨 부분은 server.js 뿐인데 모든 node_module에 있는 종속성까지 다시 다운 받아주어야한다.

-   그리고 소스 하나 변경 시켰을 뿐인데 이미지를 다시 생성하고 다시 컨테이너를 실행시켜주어야 한다.

😎 이러한 비효율성을 해결할 방법은?

### 어플리케이션 소스 변경으로 재빌드 시 효율적으로 하는 법

``` JS
# 베이스 이미지를 명시해준다.
FROM node:10

WORKDIR /usr/src/app

COPY package.json ./ 

# 추가적으로 필요한 파일들을 다운로드 받는다.
RUN npm install

COPY ./ ./

# 컨테이너 시작 시 실행될 명령어를 명시해준다.
CMD [ "node", "index.js" ]
```

위에 코드를 보면 `COPY package.json ./` 이 추가 되고 원래의 `COPY ./ ./`를 `RUN npm install` 아래로 내렸다.

그 이유는 무엇일까?

-   npm install 할 때 불필요한 다운로드를 피하기 위해서

-   원래 모듈을 다시 받는 것은 모듈에 변화가 생겨야만 다시 받아야 하는데 소스 코드에 조금의 변화만 생겨도 모듈 전체를 다시 받는 문제가 발생한다.

📝 정리

-   이전 코드에서는 `RUN npm install` 전 단계에서 COPY할때 조금이라고 바뀐 부분이 있으면 npm install이 다시 실행된다.

-   그러기에 `RUN npm install` 전 단계에서 COPY 할 때는 오직 module에 관한 것만 해준다. 그리고 `RUN npm install` 이후에 다시 모든 파일들을 COPY 해준다.

-   이렇게 해줘서 모듈은 모듈이 변화가 생길때만 다시 다운 받아주며, 소스 코드에 변화가 생길 때 모듈을 다시 받는 문제를 없애준다.    

### Docker Volumes에 대하여

-   이제는 npm install 전에 package.json만 따로 변경을 해줘서 쓸데없이 모듈을 다시 받지 않게 되었다.

-   하지만 아직도 소스를 변경할 때 마다 변경된 소스 부분은 COPY 한 후 이미지를 다시 빌드해주고 컨테이너를 다시 실행해야 변경된 소스가 화면에 반영된다.

-   이러한 작업은 너무 시간 소요가 크며 이미지도 너무 많이 빌드하게 된다

-   이러한 문제점을 해결하기 위해 `Volume`을 사용해보자

✨ Volume 사용해서 어플리케이션 실행하는 법

`docker run -d -p 5000:8000 -v /usr/src/app/node_modules -v $(pwd):/usr/src/app <이미지 아이디>`

 1. -v /usr/src/app/node_modules : 호스트 디렉토리에 node_module은 없기에 컨테이너에 맵핑 하지 말라고 하는 것

 2. -v $(pwd):/usr/src/app : pwd 경로에 있는 디렉토리 혹은 파일을 /usr/src/app에서 참조

-   PWD (print working directory)

    -   현재 작업 중인 디렉터리의 이름을 출력하는 데 쓰인다.


😎 `volume`을 이용해서 run을 하면 소스를 바꾸더라도 바꾼 코드가 `적용이 된다` (바꾼 코드를 적용하려면 컨테이너를 종료시키고 다시 실행해야함)