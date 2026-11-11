# 개발 워크스테이션 구축

## 1. 프로젝트 개요
터미널, Docker, Git/GitHub를 활용해 개발 환경을 구성하고,
웹 서버 실행·포트 연결·파일 변경 반영·데이터 영속성을 검증한다.

## 2. 실행 환경
- OS: macOS 15.6.1 (Build 24G90)
- Shell: zsh (/bin/zsh)
- 터미널: macOS 기본 터미널 앱
- Docker 버전: 29.8.2 (OrbStack 사용)
- Git 버전: 2.50.1 (Apple Git-155)

## 3. 수행 체크리스트
- [x] 터미널 기본 조작
- [x] 파일·디렉토리 권한 변경
- [x] Docker 설치 및 점검
- [x] hello-world·Ubuntu 실행
- [x] attach/exec 및 컨테이너 종료·유지 관찰
- [x] 이미지·컨테이너 관리, 로그·리소스 확인
- [x] Dockerfile 작성 및 커스텀 이미지 빌드·실행
- [x] 포트 매핑 및 접속 확인
- [x] 바인드 마운트 변경 반영
- [x] Docker 볼륨 영속성 검증
- [x] Git 사용자 정보·기본 브랜치 설정
- [x] GitHub·VSCode 연동

## 4. 터미널 조작 및 권한 실습
### 수행 내용
- `pwd`: 현재 위치 확인
- `ls -la`: 숨김 파일을 포함한 목록 확인
- `cd`: 디렉토리 이동
- `touch`: 빈 파일 생성
- `echo`와 `cat`: 내용 저장 및 확인
- `cp`: 파일 복사
- `mv`: 이름 변경 및 파일 이동
- `mkdir`: 디렉토리 생성
- `rm`과 `rmdir`: 파일 및 빈 디렉토리 삭제
- `ls -l`, `ls -ld`, `chmod`: 파일·디렉토리 권한 확인 및 변경

### 주요 명령 및 실제 결과 발췌
아래 명령은 전체 로그 중 일부를 발췌한 것이다.

```text
$ pwd
/Users/jiyoung/dev-workstation

$ touch memo.txt
$ ls -l memo.txt
-rw-r--r--  1 jiyoung  staff  0 11  6 05:55 memo.txt

$ echo "My first terminal practice" > memo.txt
$ cat memo.txt
My first terminal practice

$ cp memo.txt memo-copy.txt
$ mv memo-copy.txt memo-renamed.txt
$ mkdir archive
$ mv memo-renamed.txt archive/
$ ls -l archive
total 8
-rw-r--r--  1 jiyoung  staff  27 11  6 05:55 memo-renamed.txt

$ rm archive/memo-renamed.txt
$ rmdir archive

$ mv memo.txt practice/
$ cd practice

$ ls -l memo.txt
-rw-r--r--  1 jiyoung  staff  27 11  6 05:55 memo.txt
$ chmod 600 memo.txt
$ ls -l memo.txt
-rw-------  1 jiyoung  staff  27 11  6 05:55 memo.txt

$ mkdir permission-test
$ ls -ld permission-test
drwxr-xr-x  2 jiyoung  staff  64 11  6 05:58 permission-test
$ chmod 700 permission-test
$ ls -ld permission-test
drwx------  2 jiyoung  staff  64 11  6 05:58 permission-test
```

### 권한 해석
권한 숫자는 소유자·그룹·기타 사용자 순서이며,
읽기(r)=4, 쓰기(w)=2, 실행(x)=1을 더해 표현한다.

- 파일: 644 → 600. 소유자만 읽고 쓸 수 있도록 변경했다.
- 디렉토리: 755 → 700. 소유자만 읽기·쓰기·접근할 수 있도록 변경했다.
- 디렉토리의 실행(x) 권한은 해당 디렉토리를 통과하거나 들어가는 데 필요하다.

### 경로
- 절대 경로: `/Users/jiyoung/dev-workstation/practice`
- 상대 경로: `dev-workstation`에서의 `practice`
- 다른 PC에서는 사용자 이름에 맞춰 `~/dev-workstation`을 사용한다.

### 전체 실행 증거
[터미널 전체 명령 및 출력 로그](01-terminal.txt)

원본 로그에는 터미널 화면 제어 문자가 포함되어 있어,
주요 결과는 위 코드블록에 별도로 정리했다.

## 5. Docker 설치 및 운영

### 설치 및 점검
macOS에 OrbStack을 설치·실행한 뒤 Docker CLI와 엔진의 동작을 확인했다.

```text
$ docker --version
Docker version 29.8.2, build 7fc2dff

$ docker info
Client:
 Version:    29.8.2
 Context:    orbstack

Server:
 Server Version: 29.8.2
 Operating System: OrbStack
 OSType: linux
 Architecture: aarch64
```

위 출력은 전체 결과 중 주요 항목을 발췌한 것이다.

### hello-world 실행
```text
$ docker run --name hello-test hello-world
Hello from Docker!
```

`docker ps -a`에서 hello-test의 `Exited (0)`을 확인했다.
메시지 출력 후 정상 종료된 상태다.

### Ubuntu 내부 명령 실행
```text
$ docker run -it --name ubuntu-practice ubuntu:24.04 bash

# ls
bin   dev  home  media  opt   root  sbin  sys  usr
boot  etc  lib   mnt    proc  run   srv   tmp  var

# echo "Inside Ubuntu container"
Inside Ubuntu container

# cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.5 LTS"
```

`#`는 컨테이너 내부, `$`는 맥 터미널의 명령을 구분한 표시다.
OS 정보는 첫 줄만 발췌했다.

### exec와 attach 비교
다음 순서로 명령을 실행하고 상태를 관찰했다.

```text
# exit
$ docker ps -a
$ docker start ubuntu-practice
$ docker ps
$ docker exec -it ubuntu-practice bash
# exit
$ docker ps
$ docker attach ubuntu-practice
# exit
$ docker ps -a
```

| 관찰 상황 | 결과 |
|---|---|
| 최초 실행한 bash에서 exit | Exited (0) |
| exec로 별도로 연 bash에서 exit | Up 상태 유지 |
| attach로 연결한 주 실행 bash에서 exit | Exited (0) |

컨테이너의 생존 여부는 주 프로세스의 실행 여부에 달려 있다.
exec는 새 프로세스를 실행하고, attach는 기존 주 프로세스의 입출력에 연결한다.

### 이미지 및 운영 상태 확인
```text
$ docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   c567b231f656       18.5kB         10.3kB    U
ubuntu:24.04         fe81560990fa        139MB         30.9MB    U

$ docker logs hello-test
Hello from Docker!

$ docker start ubuntu-practice
ubuntu-practice

$ docker stats --no-stream ubuntu-practice
CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O    PIDS
2283c6cb75f2   ubuntu-practice   0.00%     1.441MiB / 7.816GiB   0.02%     1.13kB / 126B   3.6MB / 0B   1

$ docker stop ubuntu-practice
ubuntu-practice
```

로그 출력은 첫 메시지만 발췌했다.

마지막 `docker ps -a`에서 ubuntu-practice는 `Exited (137)`로 표시됐다.
중지는 완료됐지만 종료 코드 0의 정상 종료와 구분해 기록한다.
이 로그만으로 강제 종료에 이른 상세 원인은 확정할 수 없다.

### 전체 실행 증거
[Docker 전체 명령 및 출력 로그](02-docker.txt)
## 6. Dockerfile 기반 웹 서버

### 소스코드
- [웹 페이지](app/index.html)
- [Dockerfile](Dockerfile)

### 베이스 이미지와 커스텀 목적
`nginx:stable-alpine`을 선택했다.
정적 HTML을 제공하는 NGINX와 경량 Alpine Linux를 기반으로 한다.

- LABEL: 이미지에 프로젝트 이름을 기록한다.
- COPY: 기본 페이지 대신 직접 작성한 HTML을 제공한다.
- EXPOSE: 컨테이너에서 사용하는 80번 포트를 명시한다.
  실제 호스트 포트 연결은 실행 시 `-p`로 설정한다.

### Dockerfile
```dockerfile
FROM nginx:stable-alpine
LABEL org.opencontainers.image.title="jiyoung-workstation-web"
COPY app/ /usr/share/nginx/html/
EXPOSE 80
```

### 이미지 빌드
저장소 최상위 폴더에서 실행한다.

```text
$ docker build -t workstation-web:1.0 .
[+] Building 7.3s (7/7) FINISHED
```

빌드 출력은 완료 부분을 발췌했다.
현재 폴더를 빌드 컨텍스트로 사용해 `workstation-web:1.0` 이미지를 만들었다.

## 7. 포트 매핑 및 접속

### 컨테이너 실행
```text
$ docker run -d --name workstation-web -p 127.0.0.1:8080:80 workstation-web:1.0
51293396e4a9073230a44410dd6607a758ac52758bf39a829febcbb5d5964c4b
```

호스트의 127.0.0.1:8080을 컨테이너의 80번 포트에 연결했다.
컨테이너의 웹 서버에 맥 브라우저에서 접근하기 위한 설정이다.

`docker ps` 결과에서 다음을 확인했다.
- 상태: Up 4 seconds
- 포트: 127.0.0.1:8080->80/tcp
- 컨테이너 이름: workstation-web

### curl 검증
```text
$ curl http://localhost:8080
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>나의 개발 워크스테이션</title>
</head>
<body>
  <h1>나의 첫 Docker 웹 서버</h1>
  <p>NGINX 커스텀 이미지로 실행한 페이지입니다.</p>
  <p>페이지 버전: 1</p>
</body>
</html>
```

### 브라우저 검증
`http://localhost:8080`에 접속해 직접 작성한 페이지가 표시되는 것을 확인했다.

![8080 포트 접속 성공](web-8080.png)

### 전체 실행 증거
[빌드·실행·접속 명령과 출력 로그](03-web.txt)

### 재실행 참고
같은 이름의 컨테이너가 이미 있다면 새로 생성하는 대신
`docker start workstation-web`으로 시작한다.
8080 포트는 다른 프로그램이 사용하지 않는 상태여야 한다.

## 8. 바인드 마운트

### 목적
호스트의 HTML 파일을 수정했을 때 이미지 재빌드나
컨테이너 재생성 없이 웹 서버 응답에 반영되는지 검증했다.

### 실행
프로젝트 최상위 폴더에서 실행했다.

```bash
cp -R app bind-site
docker run -d --name workstation-bind -p 127.0.0.1:8081:80 --mount "type=bind,source=$PWD/bind-site,target=/usr/share/nginx/html,readonly" workstation-web:1.0
```

`$PWD/bind-site`는 현재 프로젝트 안의 폴더이므로,
다른 PC에서도 프로젝트 최상위에서 실행하면 경로를 맞출 수 있다.
컨테이너에서는 읽기 전용으로 연결하고 호스트에서 파일을 수정했다.

### 변경 전후 검증
명령 출력은 버전이 표시된 부분만 발췌했다.

변경 전:
```text
$ curl http://localhost:8081
  <p>페이지 버전: 1</p>
```

호스트 파일 수정:
```bash
sed -i '' 's/페이지 버전: 1/페이지 버전: 2/' bind-site/index.html
```

위 sed 명령은 macOS 기준이다.

변경 후:
```text
$ cat bind-site/index.html
  <p>페이지 버전: 2</p>

$ curl http://localhost:8081
  <p>페이지 버전: 2</p>
```

같은 컨테이너에서 호스트 파일 변경이 응답에 반영됨을 확인했다.

### 브라우저 증거
변경 전:
![바인드 마운트 변경 전](bind-before.png)

변경 후:
![바인드 마운트 변경 후](bind-after.png)

## 9. 볼륨 영속성

### 목적
컨테이너를 삭제한 뒤 새 컨테이너에 같은 볼륨을 연결해
저장된 데이터가 유지되는지 검증했다.

### 볼륨 생성과 첫 컨테이너 실행
```text
$ docker volume create workstation-data
workstation-data
```

```bash
docker run -d --name volume-first --mount type=volume,source=workstation-data,target=/data ubuntu:24.04 sleep infinity
```

### 데이터 저장 및 삭제 전 확인
```text
$ docker exec volume-first sh -c 'echo "Volume persistence verified" > /data/proof.txt'

$ docker exec volume-first cat /data/proof.txt
Volume persistence verified
```

### 첫 컨테이너 삭제
```text
$ docker rm -f volume-first
volume-first
```

### 같은 볼륨을 연결한 새 컨테이너 실행
```bash
docker run -d --name volume-second --mount type=volume,source=workstation-data,target=/data ubuntu:24.04 sleep infinity
```

### 삭제 후 데이터 확인
```text
$ docker exec volume-second cat /data/proof.txt
Volume persistence verified
```

첫 컨테이너를 삭제한 뒤에도 동일한 내용이 출력됐다.
데이터가 컨테이너와 별도로 볼륨에 보관됨을 확인했다.

### 바인드 마운트와 볼륨의 차이
- 바인드 마운트: 호스트의 지정한 파일·폴더를 컨테이너에 연결한다.
- 볼륨: Docker가 관리하는 저장 공간을 컨테이너에 연결한다.

### 전체 실행 증거
[바인드 마운트·볼륨 실습 명령과 출력](04-storage.txt)
## 10. Git 및 GitHub·VSCode 연동

### Git 설정
```bash
git config --global user.name "jiyoung-ship-it"
git config --global user.email "268129282+jiyoung-ship-it@users.noreply.github.com"
git config --global init.defaultBranch main
```

GitHub의 noreply 이메일을 사용했다.

### 전체 설정 확인
아래는 실제 출력 중 주요 항목을 발췌한 것이다.
전체 출력은 Git 실습 로그에서 확인할 수 있다.

```text
$ git --no-pager config --list
credential.helper=osxkeychain
init.defaultbranch=main
user.name=jiyoung-ship-it
user.email=268129282+jiyoung-ship-it@users.noreply.github.com
remote.origin.url=https://github.com/jiyoung-ship-it/All.git
branch.main.remote=origin
branch.main.merge=refs/heads/main
```

### 저장소 복제 및 원격 연결
```bash
git clone https://github.com/jiyoung-ship-it/All.git submission
cd submission
```

```text
$ git remote -v
origin  https://github.com/jiyoung-ship-it/All.git (fetch)
origin  https://github.com/jiyoung-ship-it/All.git (push)

$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

위 상태는 복제 직후 확인한 결과다.

### VSCode 연동
복제한 저장소 폴더에서 다음 명령으로 VSCode를 열었다.

```bash
open -a "Visual Studio Code" .
```

VSCode에서 프로젝트를 열고 `jiyoung-ship-it` 계정으로
GitHub 로그인을 완료했다.

![VSCode GitHub 로그인 증거](vscode-github.png)

Git은 로컬 코드 변경 이력을 관리하는 도구이고,
GitHub는 Git 저장소를 온라인으로 공유하는 협업 서비스다.

[Git 설정·복제·원격 연결 전체 로그](05-git.txt)

## 11. 재현 방법 및 증거 링크

### 준비
- Git과 Docker 실행 환경이 필요하다.
- macOS 실습에서는 OrbStack을 실행했다.
- 이미지 다운로드를 위한 인터넷 연결이 필요하다.
- 호스트의 8080·8081 포트가 비어 있어야 한다.

### 웹 서버 실행
처음 복제하는 새 작업 폴더를 기준으로 한다.

```bash
git clone https://github.com/jiyoung-ship-it/All.git workstation-review
cd workstation-review
docker build -t workstation-web:1.0 .
docker run -d --name review-web -p 127.0.0.1:8080:80 workstation-web:1.0
curl http://localhost:8080
```

브라우저에서 `http://localhost:8080`에 접속한다.
기존 실습 컨테이너가 8080 포트를 사용 중이면 먼저 중지한다.

```bash
docker stop workstation-web
```

`review-web`이 이미 존재한다면 새로 생성하지 않고
`docker start review-web`을 사용한다.

### 바인드 마운트 재현
8절의 명령을 저장소 최상위 폴더에서 실행한다.
`bind-site` 폴더는 `cp -R app bind-site`로 생성한다.
수정 전 버전 1과 수정 후 버전 2의 응답을 비교한다.

### 볼륨 영속성 재현
9절의 순서대로 볼륨 생성 → 파일 저장 → 첫 컨테이너 삭제
→ 같은 볼륨을 연결한 새 컨테이너 생성 → 파일 확인을 수행한다.

같은 이름의 실습 컨테이너나 볼륨이 이미 있다면
새 이름을 정하고 관련 명령에 일관되게 적용한다.
컨테이너 삭제와 볼륨 삭제는 별도 작업이며,
영속성 검증 중에는 볼륨을 삭제하지 않는다.

### 증거 목록
| 항목 | 증거 |
|---|---|
| 터미널·권한 | [01-terminal.txt](01-terminal.txt) |
| Docker 점검·운영 | [02-docker.txt](02-docker.txt) |
| 이미지 빌드·웹 접속 | [03-web.txt](03-web.txt) |
| 브라우저 접속 | [web-8080.png](web-8080.png) |
| 바인드 마운트·볼륨 | [04-storage.txt](04-storage.txt) |
| 변경 전 | [bind-before.png](bind-before.png) |
| 변경 후 | [bind-after.png](bind-after.png) |
| Git 설정·원격 연결 | [05-git.txt](05-git.txt) |
| VSCode 로그인 | [vscode-github.png](vscode-github.png) |

원본 script 로그에는 터미널 화면 제어 문자가 포함되어 있다.
주요 명령과 결과는 README 코드블록에 별도로 정리했다.

## 12. 트러블슈팅

### 사례 1: Docker 명령을 찾을 수 없음
- 문제: `docker --version` 실행 시 다음 오류가 발생했다.

```text
zsh: command not found: docker
```

- 원인 가설: Docker CLI가 설치되지 않았거나 명령 검색 경로에 없을 수 있다.
- 확인: 초기 버전 확인은 실패했으며, OrbStack 설치·실행 후 같은 명령을 다시 실행했다.
- 해결: OrbStack 설치·초기 설정 후 Docker 버전과 엔진 정보를 확인했다.

```text
$ docker --version
Docker version 29.8.2, build 7fc2dff
```

- 결과: `docker info`에서 Server 정보를 확인하고 hello-world 실행에 성공했다.
- 증거: [설치 후 검증 로그](02-docker.txt)

### 사례 2: VSCode 앱을 찾을 수 없음
- 문제: 저장소 폴더에서 VSCode를 열 때 다음 오류가 발생했다.

```text
$ open -a "Visual Studio Code" .
Unable to find application named 'Visual Studio Code'
```

- 원인 가설: VSCode가 설치되지 않았거나 macOS에서 앱을 찾을 수 없는 위치에 있을 수 있다.
- 확인: 실행 명령에서 앱 검색 오류를 확인했다.
- 해결: VSCode를 설치·응용 프로그램 폴더에 배치한 뒤 같은 명령을 다시 실행했다.
- 결과: 저장소 폴더가 열리고 GitHub 로그인까지 완료했다.
- 증거: [실패 및 재실행 로그](05-git.txt), [로그인 캡처](vscode-github.png)


---

## 보너스 과제 — Docker Compose 및 GitHub SSH

### 1. Docker Compose 기초

기존에 제작한 NGINX 커스텀 이미지 `workstation-web:1.0`을 활용하여 단일 웹 서비스를 실행하였다.

`compose-single.yml`에 서비스 이름, 이미지, 포트 매핑을 정의하였다.

- 실행 명령: `docker compose up -d`
- 상태 확인: `docker compose ps`
- 접속 확인: `curl http://localhost:8082`
- 종료 명령: `docker compose down`

실행 결과 NGINX 웹 서버가 정상적으로 시작되었으며, HTML 응답을 확인하였다.

Docker Compose를 활용하면 긴 컨테이너 실행 명령을 설정 파일로 관리하고 동일한 환경을 재현할 수 있다.

증거: `bonus-evidence/compose-single.yml`, `bonus-evidence/compose-single-log.txt`

### 2. Docker Compose 멀티 컨테이너

기존 NGINX 웹 서버인 `web`과 Alpine Linux 기반 보조 서비스인 `helper`를 함께 실행하였다.

`helper` 컨테이너에서 `wget` 명령어로 `web` 서비스에 접속하여 HTML 응답을 확인하였다.

- 웹 서버: `workstation-web:1.0`
- 보조 서비스: `alpine:3.20`
- 통신 확인: `docker compose exec helper wget -qO- http://web`

두 컨테이너가 같은 Compose 네트워크에서 서비스 이름을 이용하여 통신할 수 있음을 확인하였다.

환경 변수 실습 이후에는 NGINX의 내부 포트가 8085로 변경되어 기존 80번 포트 접속이 실패하였다. 이후 `http://web:8085`로 접속하여 통신 성공을 재확인하였다.

증거: `bonus-evidence/docker-compose-before-env.yml`, `bonus-evidence/compose-network-retest.txt`

### 3. Docker Compose 운영 명령어

다음 명령어를 활용하여 컨테이너를 관리하였다.

- `docker compose up -d`: 서비스 실행
- `docker compose ps`: 실행 상태 확인
- `docker compose logs --tail=20 web`: 웹 서버 로그 확인
- `docker compose down`: 서비스 종료 및 제거

NGINX의 실행 기록과 HTTP 요청에 대한 200 응답을 확인하였다.

이를 통해 Compose를 활용한 실행, 상태 확인, 로그 확인, 종료 절차를 학습하였다.

증거: `bonus-evidence/compose-run-log.txt`

### 4. 환경 변수 활용

NGINX 서버의 포트를 환경 변수로 설정하는 실습을 수행하였다.

`default.conf.template` 파일에 `${NGINX_PORT}`를 사용하고, Compose 설정에서 `NGINX_PORT=8085`를 지정하였다.

- 컨테이너 내부 포트: 8085
- 호스트 접속 포트: 8086
- 접속 확인: `curl http://localhost:8086`
- 환경 변수 확인: `docker compose exec web printenv NGINX_PORT`
- NGINX 설정 확인: `docker compose exec web grep listen /etc/nginx/conf.d/default.conf`

실행 결과 환경 변수 값 `8085`와 NGINX 설정의 `listen 8085;`를 확인하였다. 웹페이지 접속도 정상적으로 성공하였다.

이를 통해 프로그램의 설정을 코드와 분리하여 관리하는 방법을 학습하였다.

증거: `bonus-evidence/docker-compose.yml`, `bonus-evidence/default.conf.template`, `bonus-evidence/compose-config.txt`, `bonus-evidence/compose-run-log.txt`

### 5. GitHub SSH 키 설정

ED25519 방식의 SSH 키를 생성하고 공개키를 GitHub에 등록하였다.

이후 `ssh -T git@github.com` 명령어를 실행하여 SSH 인증 성공 메시지를 확인하였다.

기존 HTTPS 저장소 주소를 다음 SSH 주소로 변경하였다.

`git@github.com:jiyoung-ship-it/All.git`

`git remote -v`로 연결 주소를 확인하고, `git push origin main`을 실행하였다.

최종적으로 `Everything up-to-date` 메시지가 출력되어 SSH 방식으로 GitHub에 연결하고 푸시 명령을 정상 처리할 수 있음을 확인하였다. 새로운 커밋이 없어 실제 업로드된 변경 사항은 없었다.

SSH 개인키는 GitHub에 업로드하거나 기술 문서에 포함하지 않았다.

증거: `bonus-evidence/ssh-github-log.txt`

### 보너스 과제 결과

Docker Compose를 사용하여 단일 서비스와 멀티 컨테이너를 실행하고, 서비스 간 통신과 운영 명령어를 확인하였다. 환경 변수를 이용한 서버 설정 변경을 실습하였으며, GitHub SSH 인증과 저장소 연결도 완료하였다.

관련 설정 파일과 실행 결과는 `bonus-evidence` 폴더에 보관하였다.
