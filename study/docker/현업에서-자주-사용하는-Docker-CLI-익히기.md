# 현업에서 자주 사용하는 Docker CLI 익히기

## 🤷🏻‍♂️ Docker 이미지 다운받는 법

```bash
# docker pull 이미지명
$ docker pull nginx # docker pull nginx:latest와 동일하게 작동
```

이미지를 다운로드 할 때 Dockerhub에서 이미지를 다운받습니다.

Dockerhub은 Github처럼 이미지를 저장 및 다운받을 수 있는 저장소 역할을 하고 있습니다.


만약 특정 버전에 대해서 다운을 받고 싶다면 `docker pull 이미지명:태그명` 이렇게 작성하면 됩니다.

## 🤷🏻‍♂️ 다운받은 모든 이미지 조회하는 법

```bash
$ docker image ls
```

<img src="../../assets/docker-image-ls.png">

- `ls` : list의 약자
- `REPOSITORY` : 이미지 이름(이미지명)
- `TAG` : 이미지 태그명
- `IMAGE ID` : 이미지 ID
- `CREATED` : 이미지가 생성된 날짜 (다운받은 날짜 X)
- `SIZE` : 이미지 크기

## 🤷🏻‍♂️ 이미지 삭제하는 법

### 특정 이미지 삭제

```bash
$ docker image rm [이미지 ID 또는 이미지명]
```

컨테이너에서 사용하고 있지 않은 이미지만 삭제가 가능합니다.

### 중지된 컨테이너에서 사용하고 있는 이미지 강제 삭제하기

```bash
$ docker image rm -f [이미지 ID 또는 이미지명]
```

실행 중인 컨테이너에서 사용하고 있는 이미지는 강제로 삭제할 수 없습니다.

### 전체 이미지 삭제

```bash
# 컨테이너에서 사용하고 있지 않은 이미지만 전체 삭제
$ docker image rm $(docker images -q)

# 컨테이너에서 사용하고 있는 이미지를 포함해서 전체 이미지 삭제
$ docker image rm -f $(docker images -q)
```

`docker images -q`는 시스템에 있는 모든 이미지의 ID를 반환합니다. 여기서 -q 옵션은 quite를 의미하며, 상세 정보 대신에 각 이미지의 고유한 ID만 표시하도록 지시합니다.


## 🤷🏻‍♂️ 컨테이너 생성 및 실행 하는 법

### 컨테이너 생성

이미지를 바탕으로 컨테이너를 생성합니다. 하지만 컨테이너를 실행시키지는 않습니다.

```bash
# docker create 이미지명[:태그명]
$ docker create nginx

$ docker ps -a # 모든 컨테이너 조회
```

만약 로컬 환경에 다운받은 이미지가 없다면 Dockerhub으로부터 이미지를 다운(docker pull)받아서 컨테이너를 생성합니다.

### 컨테이너 실행

정지되어 있는 컨테이너를 실행시킵니다.

```bash
# docker start 컨테이너명[또는 컨테이너 ID]
$ docker start 컨테이너명[또는 컨테이너 ID]

$ docker ps # 실행중인 컨테이너 조회

# Nginx 컨테이너 중단 후 삭제하기
$ docker ps # 실행 중인 컨테이너 조회
$ docker stop {nginx를 실행시킨 Contnainer ID} # 컨테이너 중단
$ docker rm {nginx를 실행시킨 Contnainer ID} # 컨테이너 삭제
$ docker image rm nginx # Nginx 이미지 삭제
```

### 컨테이너 생성 + 실행

이미지를 바탕으로 컨테이너를 생성한 뒤, 컨테이너를 실행까지 시킵니다.

```bash
# docker run 이미지명[:태그명]
$ docker run nginx # 포그라운드에서 실행 (추가적인 명령어 조작을 할 수가 없음)

# Ctrl + C로 종료할 수 있음
```

만약 백그라운드로 실행을 시키고 싶다면, 아래와 같이 해주면 됩니다.

```bash
# docker run -d 이미지명[:태그명]
$ docker run -d nginx

# Nginx 컨테이너 중단 후 삭제하기
$ docker ps # 실행 중인 컨테이너 조회
$ docker stop {nginx를 실행시킨 Contnainer ID} # 컨테이너 중단
$ docker rm {nginx를 실행시킨 Contnainer ID} # 컨테이너 삭제
$ docker image rm nginx # Nginx 이미지 삭제
````

### 컨테이너에 이름을 붙여서 생성 및 실행

```bash
# docker run -d --name [컨테이너 이름] 이미지명[:태그명]
$ docker run -d --name my-web-server nginx

# Nginx 컨테이너 중단 후 삭제하기
$ docker ps # 실행 중인 컨테이너 조회
$ docker stop {nginx를 실행시킨 Contnainer ID} # 컨테이너 중단
$ docker rm {nginx를 실행시킨 Contnainer ID} # 컨테이너 삭제
$ docker image rm nginx # Nginx 이미지 삭제
```

### 호스트의 포트와 컨테이너의 포트를 연결

```bash
# docker run -d -p [호스트 포트]:[컨테이너 포트] 이미지명[:태그명]
$ docker run -d -p 4000:80 nginx
```

위와 같이 입력하게 된다면, 도커를 실행하는 호스트의 4000번 포트를 컨테이너의 80번 포트로 연결하도록 설정합니다.

## 🤷🏻‍♂️ 컨테이너 조회 / 중지 / 삭제하는 법

### 컨테이너 조회

```bash
$ docker ps
```

하지만 위 명령어는 실행 중인 컨테이너만 보여주기 때문에, 정지된 컨테이너도 같이 포함해서 보고 싶다면 `docker ps -a` 이라고 작성하면 됩니다.

### 컨테이너 중지

```bash
$ docker stop 컨테이너명[또는 컨테이너 ID]
$ docker kill 컨테이너명[또는 컨테이너 ID]
```

위 명령어의 차이를 집에 있는 컴퓨터로 비유하자면 stop은 시스템 종료 버튼을 통해 정상적으로 컴퓨터를 종료하는 걸 의미하고, kill은 본체 버튼을 눌러 무식하게 종료하는 걸 의미합니다.

### 컨테이너 삭제

```bash
$ docker rm 컨테이너명[또는 컨테이너 ID]
```

하지만 위 명령어는 실행 중인 컨테이너는 중지한 후에만 삭제가 가능합니다. 그렇기 때문에 rm 전에 stop을 해주어야 하며, 만약 이 과정이 귀찮다면 `docker rm -f 컨테이너명[또는 컨테이너 ID]` 이렇게 작성하면 됩니다.

중지되어 있는 모든 컨테이너를 삭제하고 싶다면 `docker rm $(docker ps -qa)`, 실행되고 있는 모든 컨테이너 삭제하고 싶다면 `docker rm -f $(docker ps -qa)`입니다. 

## 🤷🏻‍♂️ 컨테이너 로그 조회하는 법

### 특정 컨테이너의 모든 로그 조회

```bash
# docker logs [컨테이너 ID 또는 컨테이너명]

$ docker run -d nginx
$ docker logs [nginx가 실행되고 있는 컨테이너 ID]
```

### 최근 로그 10줄만 조회

```bash
# dokcer logs --tail [로그 끝부터 표시할 줄 수] [컨테이너 ID 또는 컨테이너명]
$ dokcer logs --tail 10 [컨테이너 ID 또는 컨테이너명]
```

### 기존 로그 조회 + 생성되는 로그를 실시간으로 보고 싶은 경우

```bash
# docker logs -f [컨테이너 ID 또는 컨테이너명]

# Nginx의 컨테이너에 실시간으로 쌓이는 로그 확인하기
$ docker run -d -p 80:80 nginx
$ docker logs -f
```

-f는 follow를 뜻합니다.

### 기존 로그는 조회하지 않기 + 생성되는 로그를 실시간으로 보고 싶은 경우

```bash
$ docker logs --tail 0 -f [컨테이너 ID 또는 컨테이너명]
```

## 실행중인 컨테이너 내부에 접속하는 방법

컨테이너는 미니 컴퓨터입니다. 즉, 호스트 컴퓨터 안에 다른 새로운 컴퓨터가 여러개 있는 것과 같습니다. 따라서 각각의 컨테이너는 자기만의 컴퓨터 공간(OS, 저장 공간, 프로그램 등)을 가지고 있습니다.

### 실행 중인 컨테이너 내부에 접속하기

```bash
# docker exec -it 컨테이너명[또는 컨테이너 ID] bash

$ docker run -d nginx
$ docker exec -it [Nginx가 실행되고 있는 컨테이너 ID] bash
$ ls # 컨테이너 내부 파일 조회
$ cd /etc/nginx 
$ cat nginx.conf
```

- 컨테이너 내부에서 나오려면 `Ctrl + D` 또는 `exit`을 입력하면 됩니다.
- `bash` : 쉘(Shell)의 일종입니다.
- `-it` : `-it`옵션을 사용해야 명령어를 입력하고 결과를 확인할 수 있습니다. `-it`옵션을 적지 않으면 명령어를 1번만 실행시키고 종료되어 버립니다. 즉, `-it` 옵션을 적어야 계속해서 명령어를 입력할 수 있습니다.