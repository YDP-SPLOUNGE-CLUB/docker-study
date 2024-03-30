## 4.3.2 도커 컴포즈 활용

도커 컴포즈를 실제 개발 환경에서 사용하려면 이제까지보다 더 많은 옵션과 명령어가 필요하다

YAML 파일을 작성하는 데 능숙해져야 기존 컨테이너의 설정을 손쉽게 도커 컴포즈 환경으로 옮기고 수정할 수 있다.

도커 컴포즈를 다루는 자세한 방법과 YAML 파일에서 자주 쓰이는 항목들을 알아보자

### 4.3.2.1 YAML 파일 작성

도커 컴포즈를 사용하려면 컨테아너 설정을 저장한 YAML 파일이 필요하다.

그러므로 기존에 사용하던 run 명령어를 YAML 파일로 변환하는 것이 도커 컴포즈 사용법의 대부분이다.

YAML 파일을 크게 버전 정의 서비스 정의 볼륨, 네트워크 정의 4가지 항목으로 구성된다.

가장 많이 사용되는 것은 서비스 정의이며 볼륨 정의와 네트워크 정의는 서비스로 생성된 컨테이너에 선택적으로 사용된다.


#### 버전 정의

YAML 파일 포맷에는 버전 1.2 2.1 3 이 있지만 이번 장에서는 도커 컴포즈 버전 1.10 에서 사용할 수 있는 버전 3을 기준으로 한다.

#### 서비스 정의

서비스는 도커 컴포즈로 생성할 컨테이너 옵션을 정의한다. 이 항목에 쓰인 각 서비스는 컨테이너로 구현되며

하나의 프로젝트로서 도커 컴포즈에 의해 관리된다. 서비스의 이름은 services 의 하위 항목으로 정의하고 컨테이너의 옵션은 서비스 이름의 하위 항목에 정의한다.

```
services:
    my_container 1: 
        image: ~~~~
    my_container 2: 
        image: ~~~~
```

#### image
서비스의 컨테이너를 생성할 때 쓰일 이미지의 이름을 설정한다. 이미지 이름 포맷은 docker run과 같다.
만일 이미지가 도커에 존재하지 않으면 저장소에서 내려받는다.

```
services:
    my_container 1: 
        image: alicek106/composetest:mysql
```

#### links

docker run 명령어의 --link 와 같으며 다른 서비스에 서비스명만으로 접근할 수 있도록 설정한다.

```
services:
    my_container 1: 
        links:
            - db
            - db:databse
            - radis
```

#### environment

docker run 명령어의 --env, -e 옵션과 동일하다. 서비스의 컨테이너 내부에서 사용할 환경변수를 지정하며
딕셔너리나 배열 형태로 사용할 수 있다.

```
services:
    my_container 1: 
        environment:
            - MYSQL_ROOT_PASSWORD=mypassword
            - MTSQL_DATABASE_NAME=mydb
        // 또는
            environment:
                MYSQL_ROOT_PASSWORD=mypassword
                MTSQL_DATABASE_NAME=mydb
```

#### command

컨테이너가 실행될 때 수행할 명령어를 설정 docker run 명령어의 마지막에 붙는 커맨드와 같다.

#### depends_on

컨테이너에 대한 의존 관계를 나타내며 이 항목에 명시된 컨테이너가 먼저 생성되고 실행된다.

다음 예제에서는 web 컨테이너보다 mysql 컨테이너가 먼저 생성된다.

links depends_on 과 같이 컨테이너의 생성 순서와 실행 순서를 정의하지만 depends_on 은 서비스 이름으로만 접근할 수 있다는 점이 다르다.

특정 서비스의 컨테이너만 생성하되 의존성이 없이 컨테이너를 생성하려면 --no-deps 옵션을 사용한다.

#### ports

docker run 명령어의 -p 옵션과 같다. 컨테이너를 개방할 포트를 설정할수 있다.

그러나 단일 호스트 환경에서 80:80 같이 호스트의 특정 포트를 서비스의 컨테이너에 연결하면 docker-compose scale 명령어로 서비스의 컨테이너의 수를 늘릴 수 없다.

#### build

build 항목에 정의된 도커파일에서 이미지를 빌드해 서비스의 컨테이너를 생성하도록 설정한다.

#### extends

다른 YAML 파일이나 현재 YAML 파일이나 현재 YAML 파일에서 서비스 속성을 상속받게 설정한다.

```
// docker-compose.yml
version: '3.0'
    service:
        web:
            extends:
                file: extend_compose.yml
                service: extend_web
                
// extend_compose.yml
version: '3.0'
    services:
        extends_web:
            image: ubuntu:14.04
            ports:
                - "80:80"
```

네트워크 정의 도커 컴포즈는 생성된 컨테이너를 위해 기본적으로 브리지 타입의 네트워크를 생성한다. 그러나 YAML 파일에서 driver 항목을 정의해
서비스의 컨테이너가 브리지 네트워크가 아닌 다른 네트워크를 사용하도록 설정할 수 있다.

특정 드라이버에 필요한 옵션은 하위 항목인 driver_ops 로 전달할 수 있다.

```
version: '3.0'
    services:
        myservice:
            network:
                - mynetwork    
                
// or

version: '3.0'
    services:
        myservice:
            network:
                - overlay
```

### 볼륨정의

#### driver

볼륨을 생성할 때 사용될 드라이버를 설정한다. 어떠한 설정도 하지 않으면 local 로 설정되며 사용하는 드라이버에 따라 변경해야 한다.
드라이버를 사용하기 위한 추가 옵션은 하위 항목인 driver_opts 를 통해 인자로 설정할 수 있다.

#### external

도커 컴포즈는 YAML 파일에서 volume, volume-form 옵션 등을 사용하면 프로젝트마다 볼륨을 생성한다.

이 때 extends 옵션을 설정하면 볼륨을 프로젝트를 생성할 때마다 매번 설정하지 않고 기존 볼륨을 사용하도록 설정 가능하다.

### YAML 파일 검증

YAML 파일을 작성할 때 오타 검사나 파일 포맷이 적절한지 등을 검사하려면 docker-compose config 명령어를 사용해서 검사 할 수 있다.

### 4.3.2.2 도커 컴포즈 네트워크

YAML 파일에 네트워크 항목을 정의하지 않으면 정의하지 않으면 도커 컴포즈는 프로젝트별로 브리지 타입의 네트워크를 생성한다.

생성된 네트워크 이름은 [프로젝트 이름]_default 로 설정되며 docker-compose up 명령어로 생성되고 docker-compose down 명령어로 삭제된다.

docker-compose up 명령어 뿐만 아니라 docker-compose scale 명령어로 생성되는 컨테이너 전부가 이 브리지 타입의 네트워크를 사용한다.
서비스 내의 컨테이너는 --net-alias 가 서비스의 이름을 갖도록 자동으로 설정되므로 이 네트워크에 속한 서비스의 이름으로 서비스 내의 컨테이너에
접근할 수 있다.

### 4.3.2.3 도커 스웜 모드와 함께 사용하기

도커 컴포즈 1.10 버전에서 스웜 모드와 함께 사용할 수 있는 YAML 버전 3 이 배포됨과 동시에

스웜 모드와 함께 사용되는 개념인 스택이 도커 엔진 1.13 버전에 추가되었다. 스택은 YAML 파일에서 생성된 컨테이너 묶음으로서 YAML 파일로 스택을 생성하면

YAML 파일에 정의된 서비스가 스웜 모드의 클러스터에서 일괄적으로 생성된다.

스택은 도커 컴포즈 명령어인 docker-compose 가 아닌 docker stack 으로 제어해야 한다.

스택은 도커 컴포즈에 의해서 생성된 것이 아닌 스웜 모드 클러스터의 매니저에 의해 생성된 것이기 떄문이다.

```

network: {}
services: 
    mysql:
        command: mysqlId
        image: alicek106/composetest:mysql
version: 3.0
volumes: {}
```

docker stack deploy 명령어에 --config-file 또는 -c 옵션으로 YAML 파일을 지정한 뒤 마지막에 스택의 이름을 입력

```
docker stack deploy -c docker-compose.yml mystack
```

만약 link, depends_on 과 같은 컨테아너 간의 의존성을 정의하는 항목은 사용 불가능하다.

생성된 스택은 docker stack ls 명령어를 통해 확인 가능하다.

만약 스택의 scale 사이즈를 조절하려면 docker-compose 가 아닌 docker service scale 을 통해서 컨테이너의 수를 조절해야 한다.