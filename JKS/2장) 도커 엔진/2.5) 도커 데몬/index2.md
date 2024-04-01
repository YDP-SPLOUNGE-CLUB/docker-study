## Btrfs 드라이버 

Btrfs 는 리눅스 파일시스템 중 하나로 SSD 최적화 데이터 압축 등 다양한 기능을 제공한다.
Btrfs 드라이버는 Devicemapper 나 AUFS, OverlayFS 와는 다르게 파일시스템을 별도로 구성하지 않으면 도커에서 사용이 불가능하다.
**/var/lib/docker 디렉터리가 btrfs 파일 시스템을 사용하는 공간에 마운트돼 있어야만 도커는 Btrfs 스토리지 드라이버로 인식한다.**

대부분의 리눅스 배포판에서 사용할 수 있다.

(ubuntu)
1. apt-get install btrfs-tools

(Centos)
1. yum install btrfs-progs


2. mkfs.btrfs -f /dev/{name}
스토리지 풀을 생성 /dev/{name} 를 입력해 해당 디바이스를 Btrfs 로 만들고 사용하는 디바이스에 맞는 이름을 입력

3. vi /etc/fstab
/dev/{name} /var/lib/docker btrfs defaults 0 0

4. mounb - a 명령어를 입력해 fstab 파일에 명시된 파일시스템을 마운트한다.


## Btrfs 드라이버는

Btrfs 드라이버는 이미지와 컨테이너를 서브 볼륨과 스냅숏 단위로 관리한다. 구조 자체는 AUFS, devicemapper 등 다른 스토리지 드라이버와 유사하다.
이미지에서 가장 아래에 있는 베이스 레이어가 서브볼륨이 되고 그 위에 쌓이는 자식 레이어가 베이스 레이어에 대한 스냅숏으로 생성된다.

새로운 컨테이너가 생성되면 컨테이너 레이어가 이미지의 맨 위에 있는 레이어의 스냅숏으로 생성된다.

Btrfs 는 자체적으로 SSD 에 최적화돼 있으며 대체적으로 우수한 성능을 보여준다. 리눅스의 파일시스템이 제공하지 않는 여러 기능을 제공한다는 장점이 있다.

## ZFS 드라이버 사용하기

ZFS 는 썬 마이크로시스템즈에서 개발했으며 Btrfs 처럼 압축 레플리카 데이터 중복 제거 등 다양한 기능을 제공한다. ZFS 는 라이선스 문제로 리눅스커널에 기본적으로 탑재돼 있지 않기 떄문에
별도의 설치 과정이 필요하다.

```
service docker stop

// ubuntu 16 / 18 에서는 관련 유틸리티 설치 가능
apt install zfsutils-linux

modprobe zfs

// 새로운 zpool 생성
zpool create -f zpool-codker /dev/${name}

// ZFS 파일시스템 생성 및 마운트
zfs create -o mountpoint=/var/lib/docker zpool-docker/docker
```

## ZFS 란

ZFS 도 용어의 차이만 있을 뿐 다른 스토리지 드라이버와 유사한 구조를 띤다. 이미지와 컨테이너 레이어는 ZFS 파일시스템 클론 스냅숏으로 관리된다.
이미지의 베이스 레이어가 ZFS 파일시스템이 되고 스냅숏을 생성해 하나의 공간으로 관리한다.

Btrfs 와 유사하게 RoW 를 사용한다.

ZFS는 성능 뿐만 아니라 안정점에 초점을 두고있으며 데이터 중복 제거, 압축 등 여러 기능을 제공한다.

## 컨테이너 저장 공간 설저

컨테이너 내부에서 사용되는 파일시스템의 크기는 도커가 사용하고 있는 스토리지에 따라 다르다.

### devicemapper 에서의 컨테이너 저장 공간 설정

위 예시에서 호스트의 저장 공간은약 30G 이며 컨테이너 또한 해당 저장공간을 그대로 사용할 것이다.

할당하는 방식이 다르다.

자체적으로 제공하고 있지 않다.

```
DOKCER_OPTS="... --storage-driver=devicemapper --storage dm.baseize=205...;
```

```
service docker stop
rm -rf /var/lib/docker
service docker start
```

위 옵션이 정상적으로 적용됐다면 컨테이너 내부의 파일시스템 크기가 20G 로 늘어난 것을 확인 할 수 있다.
```
docker run -i -t --name centos2 centos:7

df -h
```

또는 컨테이너 생성할 떄 --storage-opt 옵션을 통해 저장 공간의 크기를 제한할 수 있다.

docker info | grep Base 를 사용해서 dm.basesize 의 값을 확인하여 그 값보다 높은 값이면 설정 가능하다.

### overlay2 에서의 컨테이너 저장 공간 설정

스토리지 드라이버로 overlay2 를 사용하고 있으며 도커 데이터가 저장되어 있는 디스크가 xfs파일 시스템일 경우
project quota 라는 기능을 이용해 컨테이너의 저장 공간을 제한할 수 있다.

1. 호스트 서버에 새로운 디스크를 추가한다.
2. 해당 디스크를 xfs 파일 시스템으로 포맷하고 호스트 서버에 마운트한다.
3. 도커 엔진이 데이터를 저장하는 경로를 xfs 파일 시스템의 디렉터리로 변경한다.

새로운 디스크를 호스트에 추가하는 방법은 작업 환경에 적합한 방법을 선택해야한다.

AWS 의 EC2 인스턴스에서 도커를 사용하고 있다면 새로운 EBS 볼륨을 생성해 인스턴스에 연결할 수 있다.

버추얼 박스에서는 가상 디스크를 생성하고 가상 머신에 연결해도 된다. 또는 라즈베리파이나 물리 서버 등에서 도커를 사용하고 있다면 단순히 사용하지 않는 usb 를
서버에 꽃아서 새로운 디스크로 사용 가능하다.

xfs 파일 시스템으로 포맷한다.
```
mkfs.xfs /dev/xvdf
```

해당 디스크를 마운트 하기 위한 디렉터리 생성뒤 디스크를 마운트한다.
```
mkdir /mnt/xfs
mount /dev/xvdf /mnt/xfs -o rw,pquota

DOCKER_OPTS="--storage-driver=overlay2 --data-root=/mbnt/xfs" 
// 위 커멘드를 입력해 도커 데몬에 추가하고 도커를 재시작
```

## 2.5.4 도커 데몬 모니터링

도커 데몬을 모니터링 하는 데는 여러 이우가 있을 수 있다. 많은 수의 도커 서버를 효율적으로 관리하거나
도커로 컨테이너 애플리케이션을 개발하다가 문제가 생겼을 때 그 원인을 찾기 위해서일 수도 있다.
Paas로써? 제공하기 위해 실시간으로 도커 데몬의 상태를 체크해야 할 수도 있다.

하나의 컨테이너 뿐만이 아닌 데몬 자체를 모니터링 하는 방법을 알아볼 수 있다.

### 2.5.4.1 도커 데몬 디버그 모드

도커에서 어떤 일이 일어나고  있는지 가장 확실하고 정확하게 그리고 자세히 알아내는 방법은 디버그 옵션으로 실행하는 것이다.

Remote API 입출력만이 아니고 오가는 모든 로그를 출력한다.

```
dockerd -D 
```

### 2.5.4.2 events, stats, system df 명령어

### events

도커를 사용하는 가장 쉬운 방법은 도커 자체가 제공하는 기능을 사용하는 것이며
events 명령어도 도커가 기본으로 제공하는 명령어이다. 도커 데몬에 어떤 일이 일어나고 있는지를 실시간 스트림 로그로 보여준다.

사용법도 간단하다.

```
docker events
docker system events
```

도커 데몬에서 실행되는 명령어의 결과를 로그로 출력한다. 그러나 도커 클라이언트에서 입력하는 모든 명령어가 출력되는 것이 아니다.
attach, commit, copy, create, 등의 컨테이너 관련 명령어 delete, import, load, pull, push 등의 이미지 관련 명령어 볼륨 네트워크 플러그인 등에 관한 명령어의 수행 결과가 출력된다.

events 명령어는 filter 를 걸어서 내가 보고싶은 로그만 보도록 설정할 수 있다.

출력은 container, image, volume, network, plugin, daemon 등으로 필터를 걸 수 있다

```
docker events --filter 'type=image'
```

### stats

docker stats 명령어는 실행 중인 모든 컨테이너의 자원 사용량을 스트림으로 출력한다.

```
docker stats
```

### system df

system df 명령어는 도커에서 사용하고 있는 이미지 컨테이너 로컬 볼륨의 총 개수 및 사용 중인 개수 크기, 를 삭제함으로써 확보 가능한 공간을 출력한다.

### 2.5.4.3 CAdvisor

구글이 만든 컨테이너 모니터링 도구, 컨테이ㅏ너로서 간단히 설치 가능하며 컨테이너별 실시간 자원 사용량 및 도커 모니터링 정보를 시각화하여 보여준다.

오픈소스로서 깃허브에서 소스코드로 사용 가능 도커 허브에서 도커 이미지로도 배포되고 있다.

```
docker run \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:ro \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/dev/disk/:/dev/disk:ro \
--publish=8080:8080 \
--detach=true \
--name=cdvisor \
google/cadvisor:latest
```

CAdisor 에서는 생성된 모든 컨테이너의 자원 사용량을 확인 가능하며 도커 데몬의 정보 상태 호스트의 자원 사용량까지 한번에 확인 가능하다.

도커 데몬의 정보를 가져올 수 있는 호스트의 모든 디렉터리를 CAdvisor 컨테이너의 볼륨으로서 마운트 했기 때문에 해당 자료를 전부 볼 수 있었다.

다만 CAdvisor 는 단일 도커 호스트만을 모니터링 할 수 있다는 한계가 있다.

여러 호스트로 도커를 사용한다면 알맞지 않다.

## 2.5.5 Remote API 라이브러리를 이용한 도커 사용

-H 옵션을 원격의 도커 데몬을 제어하기 위해사용하는 것도 좋은 방법이 될 수 있지만

컨테이너 애플리케이션이 수행해야 할 작업이 많거나 애플리케이션 초기화 등에 복잡한 과정이 포함돼 있다면 도커를 제어하는 라이브러리를 사용할 수도 있다.
