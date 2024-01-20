# 2.4 DockerFile

## 2.4.1 도커 이미지를 생성하는 방법

개발한 애플리케이션을 컨테이너화 할때 가장 먼저 생각나는 방법은 아래와 같다.

1. 아무것도 존재하지 안은 이미지로 컨테이너를 생성
2. 애플리케이션을 위한 환경을 서치하고 소스코드 등을 복사해서 동작하는 것을 확인
3. 컨테이너를 이미지로 커밋

이 방법을 사용하면 애플리케이션이 동작하는 환경을 구성하기 위해 일일이 수작업으로 패키지를 설치하고
소스코드를 깃으로 복제하거나 호스트에서 복사해야 한다. 

물론 직접 컨테이너에서 애플리케이션을 구동해보고 이미지로 커밋하기 떄문에 이미지의 동작을 보장할 수 있다는 장점이 있다.

```
FROM ununtu:14.04
RUN apt-get update
RUN apt-get install
....
```

도커는 위와 같은 일련의 과정을 손쉽게 기록하고 수행할 수 있는 빌드 명령어를 제공한다.

완성된 이미지를 생성하기 위해 컨테이너에 설치해야 하는 패키지, 추가해야 하는 소스코드, 실행해야 하는 명령어와 셀 스크립트 등을 하나의 파일에
기록해 두면 도커는 이 파일을 읽어 컨테이너 작업을 수행한 뒤 이미지로 만들어 낸다.

이러한 작업을 기록한 파일을 DockerFile 이라 하며 빌드 명령어는 DockerFIle 을 읽어 이미지를 생성한다.

이미지로 커밋해야 하는 번거로움을 덜 수 있을뿐더러 깃과 같은 개발 도구를 통해 애플리케이션의 빌드 및 배포를 자동화 할 수 있다.

Dockerfile 은 애플리케이션을 개발하는 용도 외에도 여러 목적으로 사용될 수 있다. 생성한 이미지를 도커 허브 등을 통해 배포할 때
이미지 자체를 배포하는 대신 이미지를 생성하는 방법을 기록해 둔 Dockerfile 을 배포할 수 있다.

### 2.4.2 Dockerfile 작성

앞에서 설명한 것처럼 Dockerfile 에는 컨테이너에서 수행해야 할 작업을 명시한다. 
이 작업을 Dockerfile 에 정의하기 위해서는 Dockerfile 에서 쓰이는 명령어를 알아둘 필요가 있다.

기존의 스크립트 언어와 바교했을 때 완전히 새로운 방식으로 쓰이지만 컨테이너에서 사용되는 기초적인 명령어를 알기 쉽게 변환한 것이라
어렵지 않게 익힐 수 있다.

```
mkdir dockerfile && cd dockerfile
echo test >> test.html

vi Dockerfile

FROM ubuntu:20.04
MAINTAINER alicek106
LABEL: "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["bin/bash", "c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachetl -DFOREGROUND
```

Docker 에서 시용되는 명령어에는 여러 가지가 있지만 앞에서는 FROM, RUN, ADD 등의 기초적인 명령어를 우선적으로 다룬다.

Docker 은 한 줄이 하나의 명령어가 되고 명령어를 명시한 뒤에 옵션을 추가하는 방식이다.

- FROM: 생성할 이미지의 베이스가 될 이미지를 뜻한다. FROM 명령어는 Dockerfile 을 작성할 떄 반드 한 번 이상 입력해야 하며
이미지 이름의 포맷인 dockerrun 명령어에서 이미지 이름을 사용했을 떄와 같다. 사용하려는 이미지가 도커에 없다면 자동으로 pull 한다.
- MAINTAINER: 이미지를 생성한 개발자의 정보를 나타낸다. 일반적으로 Dokcer 을 작성한 사람과 연락할 수 있는 이메일을 등을 나타낸다.
- LABEL: 이미지에 메타데이터를 추가한다. 키:벨류 값으로 저장되며 여러 개의 메타데이터가 저장될 수 있다. 추가된 메타데이터는 docker inspect
명령어로 이미지의 정보를 구해서 확인할 수 있다.
- RUN: 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행한다. 예제에서는 apt-get update 와 apt-get install apache2 명령어를 실행하기 떄문에
아파치 웹 서버가 설치된 이미지가 생성된다. 
- ADD: 파일을 이미지에 추가한다. 추가하는 파일은 Dockerfile이 위치한 디렉터리인 컨텍스트에서 가져온다.
- WORKDIR: 명령어를 실행한 디렉터리를 나타낸다. 배시 셀에서 cd 명령어를 입력하는 것과 같은 기능을 한다.
- EXPOSE: Dockerfile의 빌드로 생성된 이미지에서 노출할 포트를 설정한다. 그러나 EXPOSE를 설정한 이미지로 컨테이너를 생성했닥 해서 반드시 이 포트와
바인드되는 것은 아니며 단지 컨테이너의 80번 포트를 사용할 것임을 나타내는 것뿐이다. EXPOSE 는 컨테이너를 생성하는 run 명령어에서 모든 노툴된 컨테이너의 포트를
호스트에 퍼블리시 하는 -P 플래그와 함께 사용된다.
- CMD: CMD는 컨테이너가 시작될 때마다 실행할 명령어를 설정하며 Dockerfile에서 한 번만 사용할 수 있다.
DOCKERfile CMD 를 명시함으로서 이미지에 apachetil -DFOREGRROUND 라는 커멘드를 내장하면 컨테이너를 생성할 때 별도의 커멘드를 입력하지 않아도
적용되어 컨테이너가 시작될 떄 자동으로 아파치 서버가 시작될 것이다.

### 2.4.3 Dockerfile 빌드

```
docker build -t mybuild:0.0 ./
```

빌드 중 apache 설치를 할 떄 
```
 => => # enting
 => => # the time zones in which they are located.
 => => #   1. Africa      4. Australia  7. Atlantic  10. Pacific  13. Etc
 => => #   2. America     5. Arctic     8. Europe    11. SystemV
 => => #   3. Antarctica  6. Asia       9. Indian    12. US
 => => # Geographic area:
```

이 상태에서 입력을 할 수가없어서 docker BUILD 파일에 스크립트를 추가하였다.

```kubernetes helm
RUN DEBIAN_FRONTEND=noninteractive
```

### 다음 명령어를 입력해 생성된 이미지로 컨테이너를 실행하기.

```
docker run -d -P --name myserver mybuild:0.0
```

-P 옵션은 이미지에 설정된 EXPOSE 의 모든 포트를 호스트에 연결하도록 한다.

port 명령어를 입력하면 연결되었는걸 알 수 있다,
```
docker port myserver
```

docker 이미지의 라벨을 purpose=practice 로 설정했으므로 docker images 명령어의 필터에 이 라벨을 적용할 수 있다.

```
docker images --filter "label=purpose=practice"
```

## 2.4.3.2 빌드 과정 살펴보기

이미지 빌드를 시작하면 도커는 가장 먼저 빌드 컨텍스트를 읽어 들인다. 빌드 컨텍스트는 이미지를 생성하는 데 필요한 각종 파일
소스코드 메타데이터 등을 담고 있는 디렉터리를 의미하며 Dockerfile 이 위치한 디렉터리가 빌드 컨텍스트가 된다.

빌드 컨텍스트는 Dockerfile 에서 빌드될 이미지에 파일을 추가할 때 사용된다. Dockerfile 에서 이미지에 파일을 추가하는 방법은
앞에서 설명한 ADD 외에도 COPY 가 있는데 이 명령어들은 빌드 컨텍스트의 파일을 이미지에 추가한다. 위 예제에서도 빌드 경로를 ./ 로 지정함으로써
test.html 파일을 빌드 컨텍스트에 추가했고 ADD 명령어를 통해 빌드 컨텍스트에서 test.html 파일을 이미지에 추가하였다.

컨텍스트는 build 명령어의 맨 마지막에 지정된 위치에 있는 파일을 전부 포함한다.
깃과 같은 외부 URL 에서 Dockerfile 을 읽어 들인다면 해당 저장소에 있는 파일과 서브 모듈을 포함한다.

따라서 Dockerfile 이 위치한 곳에는 이미지 빌드에 필요한 파일이 있는 것이 바람직하며 루트 디렉토리 와 같은 곳에서 이미지를 빌드하지 않도록 주의해야 한다.

이를 위해 .gitignore 와 유사한 .dockerignore 파일을 사용할 수 있다.

### Dockerfile 을 이용한 컨테이너 생성과 커밋

build 명령어는 Dockerfile 에 기록된 대로 컨테이너를 실행한 뒤 완성된 이미지를 만들어 낸다.
각 Step은 Dockerfile에 기록된 명령어에 해당한다. ADD, RUN 등의 명령어가 실행될 때마다 새로운 컨테이너가 하나씩 생성되며

이를 이미지로 커밋한다. Dockerfile 에서 명령어 한 줄이 실행될 때마다 새로운 컨테이너가 생성되며 Dockerfile 명령어를 수행하고
다시 새로운 이미지 레이어로 저장한다.

### 캐시를 이용한 이미지 빌드

한 번 이미지 빌드를 마치고 난 뒤 다시 같은 빌드를 진행하면 이전의 이미지 빌드애서 사용헀던 캐시를 사용한다.

그러나 때로는 캐시 기능이 너무 친절한 나머지 오히려 캐시 기능이 필요하지 않을 경우가 있다.

이 경우 캐시를 사용하지 않으려면 build 명령어에 --no-cache 옵션을 추가해야 한다.
```
docker build --no-cache -t mybuild:0.0
```

또한 캐시로 사용할 이미지를 직접 지정할 수도 있다. 특정 Dockerfile 을 확장해서 사용한다면 기존의 Dockerfile 로 빌드한 이미지를 빌드 캐시로 사용할 수 있다.

```
docker build --cache-from nginx -t my_extend_nginx:0.0
```

### 멀티 스테이지를 이용한 Dockerfile 빌드하기

일반적으로 애플리케이션을 빌드할 때는 많은 의존성 패키지와 라이브러리를 필요로 한다.

GO 를 예시로 print("hello world") 를 출력하는 main.go 파일을 만든다고 했을때

이미지는 약 800MB 를 차지한다.

이럴 떄 멀티 스테이지 빌드 방법을 사용한다면 생성될 이미지의 크기를 줄일 수 있다.

EXAMPLE)

```
FROM golang
ADD main.go /root
WORKDIR /root
RUN go build -o /root/mainApp /root/main.go

FROM alpine:latest
WORKDIR /root
COPY --from=0 /root/mainApp
CMD ["./mainApp"]
```

일반적인 Dockerfile 과는 다르게 FROM 을 통해 2개의 이미지가 명시되었다.
첫 번째 FROM에 명시된 golang 이미지는 이전과 동일하게 main.go 파일을 /root/mainApp 으로 빌드된다.

그러나 두 번쨰 이미지는 alpine:latest 를 복사한다.
이 떄 --from=0 은 첫 번째 FROM 에서 빌드된 이미지의 최종 상태를 의미한다.

### 2.4.4.1 ENV, VOLUME, ARG, USER

ENV: Dockerfile 에서 사용될 환경변수를 지정한다. 설정한 환경변수는 ${ENV.NAME} 또는 #ENV_NAME 의 형태로 사용할 수 있다.

Docker 에서 환경변수의 값을 사용할 때 배시 셀에서 사용하는 것처럼 값이 설정되지 않은 겨웅와 설정된 경우를 구분하여 사용 가능하다.

VOLUME: 빌드된 이미지로 컨테이너를 생성했을 때 호스트와 공유할 컨테이너 내부의 디렉터리를 설정한다.

ARG: build 명령어를 실행할 때 추가로 입력을 받아 Dockerfile 내에서 사용될 변수의 값을 설정한다.

USER: USER로 컨테이너 내에서 사용될 사용자 계정의 이름이나 UID를 설정하면 그 아래의 명령어는 해당 사용자 권한으로 실행된다.
일반적으로 RUN으로 사용자의 그룹과 계정을 생성한 뒤 사용한다. 루트 권한이 필요하지 않다면 USER 를 사용하는 것을 권장한다.

### 2.4.4.2 Onbuild, Stopsignal, Healthcheck, Shell

ONBUILD: 빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile 로 생성될 떄 실행할 명령어를 추가한다.

```
vi Dockerfile

FROM ubuntu:14.04
RUN echo "this is onbuild test"
ONBULD RUN echo "onbuild!" >> /onbuild_file
```

STOPSIGNAL: 컨테이너가 정지될 때 사용될 시스템 콜의 종류를 지정한다. 아무것도 설정하지 않으면 기본적으로 SIGTERM 로 설정되지만
Dockerfile 에 STOPSIGNAL 을 정의해 컨테이너가 종료되는 데 사용될 신호를 선택할 수 있다.

HEALTHCHECK: 이미지로부터 생성된 컨테이너에서 동작하는 애플리케이션의 상태를 체크하도록 설정한다. 컨테이너 내부에서 동작 중인 애플리케이션의
프로세스가 종료되지는 않지만 애플리케이션이 동작하지 않은 상태를 방지하기 위해 사용될 수 있다.

```
vi Dockerfile
FROM nginx
RUN apt-get update -y && apt-get install curl -y
HEALTHCHECK --interval=1m --timeout=3s --retires=3 CMD curl -f http://localhost || exit 1
```

SHELL: Dodckerfile 에서 기본적으로 사용하는 셀은 리눅스에서 "/bin/sh -c" 윈도우에서 cmd /S /C 이다.

### 2.4.4.3 ADD, COPY

COPY는 로컬 디렉터리에서 읽어 들인 컨텍스트로부터 이미지에 파일을 복사하는 역할을 한다. COPY를 사용하는 형식은 ADD 와 같다.

```
COPY test.html /home/
COPY ["test.html", "/home/"]
```

ADD 와 COPY의 차이점은 없는 것처럼 보인다. 사실 ADD와 COPY의 기능은 그 자체의 기능만 봤을 떄는 같다.

그러나 COPY는 로컬의 파일만 이미지에 추가할 수 있지만 ADD는 외부 URL 및 tar 파일에서도 파일을 추가할 수 있다는 점에서 다르다.

즉 COPY가 ADD의 기능에 포함되는 샘이다.

### ENTRYPOINT, CMD

CMD는 커멘드에서 실행할 명령어를 설정 가능하다. 그러나
ENTRYPOINT와 CMD는 역할 자체는 비슷하짐나 서로 다른 역할을 담당하는 명령어이다. 

ENTRYPOINT는 커멘드와 동일하게 컨테이너가 시작될 때 수행할 명령을 지정한다는 점에서 같다.

그러나 entrypoint는 커멘드를 인자로 받아 사용할 수 있는 스크립트의 역할을 할 수 있다는 점에서 다르다.

