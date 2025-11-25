# Docker Volume(도커 볼륨) 

## 🤷🏻‍♂️ 컨테이너가 가진 문제점이 무엇인가요?

Docker를 활용하면 특정 프로그램을 컨테이너 형태로 띄울 수 있다는 장점이 있습니다. 하지만 어떤 한 프로그램에서 새로운 기능이 추가가 되어 해당 기능을 사용하고 싶다면, 기존 컨테이너를 수정하는 것이 아니라 통째로 갈아끼우는 방식으로 교체해야 합니다. 

처음에는 위 방식이 효율적이라고 판단했지만, MySQL 컨테이너로 예를 들면 달라집니다. 만약 MySQL 컨테이너를 쭉 쓰다가 새로운 기능을 쓰고 싶어서 컨테이너를 갈아끼운다고 가정해 봅시다. 그렇게 된다면 그동안 쌓아뒀던 데이터를 모두 날려야 합니다. 

그렇기 때문에 컨테이너 내부에 저장된 데이터가 삭제되면 안 되는 경우에는 "볼륨"이라는 개념을 활용해야 합니다.

## 🤷🏻‍♂️ Docker Volume이 무엇인가요?

Docker Volume은 도커 컨테이너에서 데이터를 영속적으로 저장하기 위한 방법입니다. 볼륨은 컨테이너 자체의 저장 공간을 사용하지 않고, 호스트 자체의 저장 공간을 공유해서 사용하는 형태입니다. 

## 🤷🏻‍♂️ 볼륨을 사용하는 명령어엔 무엇이 있나요?

```bash
$ docker run -v [호스트의 디렉토리 절대경로]:[컨테이너의 디렉토리 절대경로] [이미지명]:[태그명]
```

- `[호스트의 디렉토리 절대 경로]`에 디렉토리가 이미 존재할 경우, 호스트의 디렉터리가 컨테이너의 디렉터리를 덮어씌웁니다.
- `[호스트의 디렉토리 절대 경로]`에 디렉토리가 존재하지 않을 경우, 호스트의 디렉터리 절대 경로에 디렉터리를 새로 만들고 컨테이너의 디렉터리에 있는 파일들을 호스트의 디렉터리로 복사해 옵니다.

그래서 아래 예로 MySQL을 동작시키는 방법입니다.

```bash
$ cd /Users/jaeseong/Documents/Develop
$ mkdir docker-mysql # MySQL 데이터를 저장하고 싶은 폴더 만들기

# docker run -e MYSQL_ROOT_PASSWORD=password123 -p 3306:3306 -v {호스트의 절대경로}/mysql_data:/var/lib/mysql -d mysql
$ docker run -e MYSQL_ROOT_PASSWORD=password123 -p 3306:3306 -v /Users/jaeseong/Documents/Develop/docker-mysql/mysql_data:/var/lib/mysql -d mysql
```

- `docker pull` 과정은 생략해도 상관없습니다. 왜냐하면 `docker run mysql`로 실행시켰을 때, 로컬에 이미지가 없으면 Dockerhub으로부터 MySQL 이미지를 알아서 다운받아서 실행시키기 때문입니다.
- `-e MYSQL_ROOT_PASSWORD=password123` : `-e` 옵션은 컨테이너의 환경 변수를 설정하는 옵션입니다.
- Dockerhub의 MySQL 공식 문서를 보면 환경 변수로 `MYSQL_ROOT_PASSWORD`를 정해주어야만 정상적으로 컨테이너가 실행된다고 나와있습니다.
- `mysql_data` 디렉토리를 미리 만들어 놓으면 안 됩니다. 그래야 처음 이미지를 실행시킬 때 mysql 내부에 있는 `/var/lib/mysql` 파일들을 호스트 컴퓨터로 공유받을 수 있습니다. `mysql_data` 디렉토리를 미리 만들어놓을 경우, 기존 컨테이너의 `/var/lib/mysql` 파일들을 전부 삭제한 뒤에 `mysql_data`로 덮어씌워 버립니다.
- DB에 관련된 데이터가 저장되는 곳이 `/var/lib/mysql`인지는 Dockerhub MySQL의 공식 문서에 나와있습니다.

여기서 조심해야 할 부분은 만약 처음 MYSQL_ROOT_PASSWORD를 123으로 하고, 그 후 컨테이너를 새로 띄울 때 456으로 한다면 비밀번호는 바뀌지 않습니다. 그 이유는 볼륨으로 설정해둔 폴더에 이미 비밀번호 정보가 저장되어 있기 때문입니다. 