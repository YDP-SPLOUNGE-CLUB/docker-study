# 2.3 도커 이미지

모든 컨테이너는 이미지를 기반으로 생성되므로 이미지를 다루는 방법은 도커 관리에서 뺴놓을 수 없는 부분이다. 이미지의 일므을 구성하는 저장소
이미지 이름, 태그를 잘 관리하는 것뿐만이 아니라 이미지가 어떻게 생성되고 삭제되는지 이미지의 구조는 어떻게 돼 있는지 등을 아는것이 중요하다.

도커는 기본적으로 도커 허브라는 중앙 이미지 저장소에서 이미지를 내려받는다.
도커 허브는 도커가 공식적으로 제공하고 있는 이미지 저장소로서 도커 계정을 가지고 있다면 누구든지 이미지를 내려받을 수 있다.

dcoker create, run pull 명령어로 이미지를 내려받을 때 도커는 도커 허브에서 해당 이미지를 검색한 뒤 내려받는다.

필요한 대부분의 이미지는 도커 허브에서 공식적으로 제공하거나 다른 사람들이 도커 허브에 올려둔 경우가 대부분이라 애플리케이션 이미지를 직접 만들지 않아도
손쉽게 사용할 수 있다.

단 도커 허브는 누구나 올릴 수 있기 떄문에 공식 라벨이 없는 이미지는 사용법을 찾을 수 없거나
제대로 동작하지 않을 수 있다.

docker search 명령어를 사용해 직접 찾아볼 수 있다.

### 2.3.1 도커 이미지 생성

docker search 를 통해 검색한 이미지를 pull 명령어로 내려받아 사용할 수 있지만
도커로 개발하는 많은 경우에는 컨테이너에 애플리케이션을 위한 특정 개발 환경을 직접 구축한 뒤 사용자만의 이미지를 직접 생성해야 할 것이다.

이를 위해 컨테이너 안에서 작업한 내용을 이미지로 만드는 방법이 있다.

```
dokcer run -i -t --name commit_test ubuntu:14.04
```

docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]] 형식으로 이미지 저장이 가능하다.

Example) commit_test:first 라는 이미지를 생성하기

```
docker commit \
-a "author1" -m "my first commit" \
commit_test \
commit_test:first
```

-a 옵션은 작성자를 뜻하며 author1 로 설정되었다.

-m 옵션은 커밋메시지를 뜻한다.

docker images 명령어를 통해 이미지를 확인할 수 있다.

이번에는 commit_test:first 이미지로 새로운 이미지를 생성해 보자.

```
dokcer run -it --name commit_test2 commit_test:first

docker commit \
-a "author1" -m "my second commit" \
commit_test2 \
commit_test:second
```

### 2.3.2 이미지 구조 이해

컨테이너로 이미지를 만들 때 commit 명령어로 쉽게 수행 가능하다. 그러나 이미지를 좀 더 효율적으로 다루기 위해서는
컨테이너가 어떻게 이미지로 만들어지며 이미지의 구조는 어떻게 되어있는지 알 필요가 있다.

```
docker inspect ubuntu:14.04
docker inspect commit_test:first
docker inspect commit_test:second
```

docker inspect 명령어는 많은 정보를 출력하지만 주의 깊게 살펴본 항목은 가장 아랫부분에 있는 Layers 항목이다.

docker images 에서 위 3개의 이미지 크기가 188MB 라고 출력이 되어도 188 크기가 이미지가 3개 존재하는 것은 아니다.
이미지를 커밋할 때 컨테이너에서 변경된 사항만 새로운 레이어로 저장하고 그 레이어를 포함해 새로운 이미지를 생성하기 떄문에 전체 이미지의 실제 크기는
188MB + first 파일의 크기 + second 파일의 크기가 된다.


### docker rmi

도커 이미지를 삭제할 때는 rmi 명령어를 사용하여 삭제 가능하다.

하지만 사용중인 컨테이너가 존재한다면 에러가 발생한다. 
docker rm -f [컨테이너 이름] 처럼 -f 옵션을 추가해 이미지를 강제로 삭제할 수도 있지만 이는 이미지 레이어 파일을 실제로 삭제하는 것이 아닌

이미지 이름만 삭제하기 떄문에 의미가 없다.

```
docker stop commit_test2 && docker rm commit-test2

docker rmi commit_test:first
Untagged: commit_test:first
```

commit_test:first  를 삭제했다고 해서 실제로 해당 이미지의 레이어 파일이 삭제되지는 않는다.
commit_test:first 이미지를 기반으로 하는 하위 이미지인 commit_test:second 가 존재하기 때문이다.

따라서 실제 이미지 파일을 삭제하지 않고 레이어에 부여된 이름만 삭제한다. rm 명령어의 출력 결과인 Untagged ... 는 이미지에 부여된 이름만 삭제한다는 것을 뜻한다.

만약 하위 이미지나 사용중인 컨테이너가 없을 경우 Deleted ... 출력 메세지가 나올 것이다.

### 2.3.3 이미지 추출

도커 이미지를 별도로 저장하거나 옮기는 등 필요에 따라 이미지를 단일 바이너리 파일로 저장해야 할 때가 있다.

docker save 명령어를 사용하면 컨테이너의 커멘드 이미지 이름과 태그 등 이미지의 모든 메타데이터를 포함해 하나의 파일로 추출할 수 있다.
-o 옵션에는 추출될 파일명을 입력할 수 있다.

```
docker save -o ubuntu_14_04.tar ubuntu:14.04
```

추출된 이미지는 load 명령어로 도커에 다시 로드할 수 있다. save 명령어로 추출된 이미지는 이미지의 모든 메타데이터를 포함하기 때문에
load 명령어로 이미지를 로드하면 이전의 이미지와 완전히 동일한 이미지가 도커 엔진에 생성된다.


```
docker load -i ubuntu_14_04.tar
```

save 와 load 명령어와 유사하게 사용 가능한 export import 명령어가 있다.
docker commit 명령어로 컨테이너를 이미지로 만들면 컨테이너에서 변경된 사항뿐만 아니라
컨네팅너가 생성될 때 설정한 detached 모드, 커맨드 와 같은 컨테이너 설정 등도 이미지에 담긴다.

export 명령어는 컨테이너의 파일 시스템을 tar 파일로 추출하며 컨테이너 및 이미지에 대한 설정 정보를 저장하지 않는다.



### 2.3.4 이미지 배포

이미지를 생성했다면 이를 다른 도커 엔진에 배포할 방법이 필요하다.

첫 번째 방법은 도커에서 공식적으로 제공하는 도커 허브 이미지 저장소를 이용하는 방법이다. 도커 허브는 더커 이미지를 저장하기 위한 클라우드 서비스라고 생각하면
이해하기 쉽다.


### 2.3.4.1 도커 허브 저장소

https://hub.docker.com/search?q=

위 사이트에서 이미지 저장소를 클릭하면 해당 이미지에 대한 자세한 설명을 조회 가능


### 저장소에 이미지 올리기

```

docker run -it --name commit_test ubuntu:14.04

➜  ~ docker commit commit_test my-image-name:0.0
sha256:63ffdca23fb838ec20b621b6454569c4ebf038ad23de68b9a710946a62749b49
```

그러나 이 이름으로는 저장소에 올릴 수 없다. 도커 이미지에서 설명했듯이 이미지 이름의 접두어는 이미지가 저장되는 저장소 이름으로 설정한다.

특정 이름의 저장소에 이미지를 올릴려면 저장소 이름을 이미지 앞에 접두어로 추가해야 한다.

docker tag 명령어를 사용하면 이미지의 이름을 추가할 수 있다.
tag 명령어의 형식은 docker tag [기존의 이미지 이름]:[새롭게 생성될 이미지 이름] 이다.

```
docker tag my-image-name:0.0 slwhswk/my-image-name:0.0
```

```
docker login

// 유저네임 비밀번호 입력후 로그인
```

#### 도커 허브에 대한 오가니제이션 까지의 내용은 스킵을 해도 무방할 것이라 판단됩니다.

----

### 저장소 웹훅 추가

저장소에 이미지가 push 됐을 때 특정 URL로 http 요청을 전송하도록 설정할 수 있다.

이 기능을 웹훅이라 한다. 도커 허브의 웹훅 기능은 저장소에 새로운 이미지가 생성됐을 때
지정된 URL로 해당 이미지에 정보와 함께 http 요청을 전송한다.


### 도커 사설 레지스트리

도커 사설 레지스트리를 사용하면 개인 서버에 이미지를 저장할 수 있는 저장소를 만들 수 있다. 이 레지스트리는 컨테이너로서 구현되므로

이에 해당하는 도커 이미지가 존재한다. 도커에서 공식적으로 제공하고 있기 때문에 아래의 run 명령어로 사용할 수 있다.

```
docker run -d --name myregistry \
-p 5000:5000 \
--restart=always \
registry:2.6
```

레지스트리 컨테이너는 기본적으로 5000번 포트를 사용하므로 -p 옵션으로 컨테이너 5000번 포트를 호스트의 5000번 포트와 연결했으며 이 포트로 레지스트리 컨테이너
RESTful API 를 사용할 수 있다.

```
curl localhost:5000/v2/
```

레지스트리 컨테이너에 이미지를 올리려면 이미지의 접두어를 레지스트리 컨테이너가 존재하는 호스트의 IP와 레지스트리 컨테이너 5000번 포트와 연결된 호스트의 포트로
설정해야 한다. 

도커 데몬은 HTTPS 를 사용하지 않은 레지스트리 컨테이너에 접근하지 못하도록 설정하기 때문에
HTTPS 인증서를 별도로 설정해주어야 한다.

DOCKER_OPTS="--insecure-registry=[사용자 HOST IP]:[PORT]"

### Nginx 서버로 접근 권한 생성

별도의 인증 절차 없이 접근은 하게 할 수 있지만 도커 허브에서 저장소를 사용하기 위해
docker login 명령어를 사용한 것처럼 레지스트리 컨테이너 또한 미리 정의된 계정으로 로그인하도록 설정하여 접근을 제한할 수 있다.

로그인 인증 기능은 보안을 적용하지 않은 레지스트리 컨테이너에서는 사용할 수 없으므로 인증서를 발급하여 적용해야 한다.

( 이 기능은 솔직히 따로 해봐야 할 것 같은 내용인 것 같습니다. 도커와 관계 없는 내용인 듯.. )

```
mkdir certs

openssl genrsa -out ./certs/ca.key 2048

➜  ~ openssl genrsa -out ./certs/ca.key 2048
➜  ~ openssl req -x509 -new -key ./certs/ca.key -days 10000 -out ./cers/ca.crt
```

```
openssl req -new -key ./certs/domain.key -subj /CN=${DOCKER_HOST_IP} -out ./certs/domain.csr

# htpasswd -c htpaswd slwhswk

비밀번호 입력 

```

certs 디렉터리의 nginx.conf 파일로 저장하고 Nginx 서버에서 SSL 인증에 필요한 각종 파일의 위치와 레지스트리 컨테이너의 프록시를 설정한다.

### 해당 내용은 nginx.conf 자료를 찾자.

Nginx 서버 컨테이너를 생성을 하고 

nginx.conf domain.crt, domain.key 파일이 존재하는 디렉터링을 -v 옵션으로 컨테이너에 공유한다.

컨테이너의 포트 번호 Example) 443 포트는 Docker 호스트의 443번 포트와 바인딩 된다.

```
docker run -d --name nginx_front \
-p 443:443 \
--link myregistry:registry \
-v ${pwd}/certs/:/etc/nginx/conf.d \
nginx:1.9
```

도커 허브와 동일하게 docker login 명령어로 컨테이너로 로그인한다.

login 뒤에 https://${DOCKER_HOST_IP} 를 입력하면 된다.
https 로 사용했으므로 자동으로 호스트 IP 443 포트로 연결하며 Nginx 서버 컨테이너로 포워드 된다.

그러나 신뢰할 수 없는 인증서인 셀프 인증서를 만들었으므로 목록에 추가해줘야 한다.

```
cp certs/ca.crt /usr/local/share/ca-certificates/
update-ca-certificates
```

다시 한번 로그인을 요청하면 성공했다는 출력 결과를 확인 할 수 있다.

----

### 사설 레지스트리 옵션 설정

레지스트리 컨테이너는 컨테이너 내부의 환경변수를 써서 레지스트리 서비스를 설정한다. 그중 하나는 

REGISTRY_STOGAGE_DELETE_ENABLED 와 같은 환경변수이다. 이러한 환경변수로는 이미지 삭제 뿐만 아니라 스토리지 백엔드, 이미지 레이어 파일이
저장된 디렉터리, 웹 훅 설정 등이 있다.

docker run 명령어의 -e 옵션을 추가해 설정할 수 있다.

```
docker run -d -p 5001:5000 --name registry_delte_enabled --restart=always \
-e REGISTRY_STOGAGE_DELETE_ENABLED \
..... 
registry:2.6
```

그러나 -e 옵션으로 입력할 필요 없이 yml 파일을 정의해서 레지스트리 컨테이너 환경변수를 설정 가능하다.

직접 입력한 yml 파일을 적용한 레지스트리 컨테이너를 생성해보자.

```
docker run -d --name yml_registry \
-p 5002:5000 \
--restart=always \
-v $(pwd)/config.yml:/etc/docker/registry/config.yml
```

