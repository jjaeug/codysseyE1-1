# codysseyE1-1 — 개발 워크스테이션 구축

터미널, Docker, Git을 직접 세팅하고 **실행 결과(로그·접속·데이터 유지)로 동작을 검증**한 기록입니다.
이 문서만 읽어도 전체 수행 내용과 증거를 확인할 수 있도록 정리했습니다.

- 저장소: https://github.com/jjaeug/codysseyE1-1
- 스크린샷: [`p/`](p/) 폴더 (브라우저 접속, GitHub, VSCode 화면)
- 터미널 증거: 각 절의 코드블록 (명령어 + 출력)

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [실행 환경](#2-실행-환경)
3. [수행 체크리스트](#3-수행-체크리스트)
4. [검증 방법 요약](#4-검증-방법-요약)
5. [터미널 기본 조작](#5-터미널-기본-조작)
6. [파일 권한 실습](#6-파일-권한-실습)
7. [Docker 설치 및 점검](#7-docker-설치-및-점검)
8. [Docker 기본 운영 명령](#8-docker-기본-운영-명령)
9. [컨테이너 실행 실습](#9-컨테이너-실행-실습)
10. [Dockerfile 커스텀 이미지](#10-dockerfile-커스텀-이미지)
11. [포트 매핑](#11-포트-매핑)
12. [바인드 마운트](#12-바인드-마운트)
13. [Docker 볼륨 영속성](#13-docker-볼륨-영속성)
14. [Git 설정 및 GitHub 연동](#14-git-설정-및-github-연동)
15. [보안 및 개인정보 보호](#15-보안-및-개인정보-보호)
16. [트러블슈팅](#16-트러블슈팅)
17. [저장소 구성 및 재현 방법](#17-저장소-구성-및-재현-방법)

---

## 1. 프로젝트 개요

- **목표**: 코드가 "내 컴퓨터에서만" 돌아가는 문제를 줄이기 위해, 누구나 같은 방식으로 실행·배포·디버깅할 수 있는 개발 환경을 직접 구성한다.
- **사용 도구**: 리눅스 CLI(터미널), Docker(컨테이너), Git/GitHub(버전 관리·협업)
- **수행 범위**
  1. 터미널로 작업 디렉토리를 만들고 파일·권한을 다룬다.
  2. Docker를 설치·점검하고 컨테이너를 실행·관리한다(이미지, 로그, 리소스).
  3. Dockerfile로 커스텀 이미지를 만들고, 포트 매핑으로 접속을 확인한다.
  4. 바인드 마운트로 "변경 반영", 볼륨으로 "데이터 영속성"을 검증한다.
  5. Git 설정 후 GitHub 저장소와 연동한다.

## 2. 실행 환경

| 항목 | 값 |
|---|---|
| OS | Ubuntu 24.04.4 LTS (Windows WSL2, 커널 `6.6.87.2-microsoft-standard-WSL2`) |
| Shell | bash |
| Terminal | WSL Ubuntu 터미널 / VSCode 통합 터미널 (WSL 연결) |
| Docker | Docker Engine (`docker.io` 패키지, WSL 내부 설치) — 버전은 [7절](#7-docker-설치-및-점검) 출력 참고 |
| Git | 2.43.0 |
| 메모리 | PC 3.4GB / WSL 할당 약 1.5GB |
| 작업 디렉토리 | `~/dev-mission` (= `/home/jjw/dev-mission`) |

환경 확인 명령:

```bash
cat /etc/os-release | head -2
uname -r
echo $SHELL
docker --version
git --version
```

> ⚠️ **TODO**: 위 명령의 실제 출력을 아래 블록에 붙여넣기

```text
(실행 결과 붙여넣기)
```

> **개인 PC 종속 주의**
> - `~/dev-mission`은 제 WSL 홈 디렉토리 기준 경로입니다. 다른 환경에서는 이 저장소를 clone한 폴더에서 실행하고, 경로가 필요한 곳은 `$(pwd)`로 바꿔 쓰면 됩니다.
> - Windows에서는 `/mnt/c/...`(Windows 드라이브)가 아니라 **WSL 홈(`~`) 아래**에서 작업해야 합니다. 기본 설정에서는 Windows 드라이브의 파일에 `chmod`가 반영되지 않습니다.
> - 자세한 재현 절차는 [17절](#17-저장소-구성-및-재현-방법) 참고

## 3. 수행 체크리스트

- [ ] 터미널 기본 조작 및 폴더 구성
- [ ] 권한 변경 실습 (파일 1개, 디렉토리 1개)
- [ ] Docker 설치 및 점검
- [ ] Docker 기본 운영 명령 (`images` / `ps` / `logs` / `stats`)
- [ ] `hello-world` 실행
- [ ] `ubuntu` 컨테이너 진입 후 명령 실행
- [ ] 컨테이너 종료/유지 (attach / exec) 차이 관찰
- [ ] Dockerfile 커스텀 이미지 빌드 및 실행
- [ ] 포트 매핑 접속 (8080, 8090 — 2회)
- [ ] 바인드 마운트 변경 반영
- [ ] 볼륨 영속성 검증
- [ ] Git 사용자 정보·기본 브랜치 설정 + GitHub 저장소 연동
- [ ] VSCode GitHub 로그인 및 저장소 연동

## 4. 검증 방법 요약

| 항목 | 확인 명령 | 확인 기준 | 증거 |
|---|---|---|---|
| 터미널 조작 | `pwd`, `ls -la`, `mkdir`, `cp`, `mv`, `rm`, `cat`, `touch` | 생성·복사·이동·삭제가 `ls` 결과에 반영됨 | [5절](#5-터미널-기본-조작) |
| 권한 | `ls -l`, `chmod 755`, `chmod 700` | 변경 전/후 권한 문자열이 달라짐 | [6절](#6-파일-권한-실습) |
| Docker 점검 | `docker --version`, `docker info` | 버전 출력, `Server Version` 표시(데몬 동작) | [7절](#7-docker-설치-및-점검) |
| 운영 명령 | `docker images`, `docker ps -a`, `docker logs`, `docker stats` | 이미지·컨테이너 목록, 접속 로그, CPU/메모리 사용량 | [8절](#8-docker-기본-운영-명령) |
| hello-world | `docker run hello-world` | `Hello from Docker!` 출력 | [9절](#9-컨테이너-실행-실습) |
| 커스텀 이미지 | `docker build -t my-ubuntu .`, `docker run my-ubuntu` | 빌드 성공, CMD 메시지 출력 | [10절](#10-dockerfile-커스텀-이미지) |
| 포트 매핑 | `docker run -p 8080:80 ...`, 브라우저/`curl` 접속 | `localhost:8080`, `localhost:8090`에서 nginx 응답 | [11절](#11-포트-매핑), [p/browser-8080.png](p/browser-8080.png), [p/browser-8090.png](p/browser-8090.png) |
| 바인드 마운트 | `docker run -v $(pwd)/bind-test:...`, `curl localhost:8090` | 호스트 파일 수정이 재시작 없이 반영 | [12절](#12-바인드-마운트) |
| 볼륨 | `docker volume create`, 컨테이너 삭제 후 재연결 | 새 컨테이너에서 기존 데이터 확인 | [13절](#13-docker-볼륨-영속성) |
| Git | `git config --list`, `git remote -v` | `user.name`, `user.email`, `init.defaultBranch` 설정, 원격 저장소 연결 | [14절](#14-git-설정-및-github-연동), [p/2.png](p/2.png), [p/vscode-github.png](p/vscode-github.png) |

---

## 5. 터미널 기본 조작

### 개념: 절대 경로 vs 상대 경로

| 구분 | 기준 | 예시 |
|---|---|---|
| 절대 경로 | 루트 `/`부터 전체 위치를 적음. 현재 위치와 상관없이 항상 같은 곳을 가리킴 | `/home/jjw/dev-mission/hello.txt` |
| 상대 경로 | **현재 위치(`pwd`)** 기준. `.`은 현재 폴더, `..`은 상위 폴더 | `./hello.txt`, `../dev-mission/hello.txt` |

예: `~/dev-mission`에 있을 때 `cat hello.txt`와 `cat /home/jjw/dev-mission/hello.txt`는 같은 파일을 읽지만, `cd /tmp`로 이동하면 상대 경로 `hello.txt`는 찾을 수 없습니다.

### 실행 명령

```bash
pwd                                  # 현재 위치 확인
ls -la                               # 목록 확인 (숨김 파일 포함)
mkdir -p practice/sub                # 디렉토리 생성
cd practice                          # 이동
pwd
touch empty.txt                      # 빈 파일 생성
ls -l empty.txt                      # 크기 0 확인
echo "Hello, Terminal!" > hello.txt  # 내용 있는 파일 생성
cat hello.txt                        # 파일 내용 확인
cp hello.txt hello-copy.txt          # 복사
mv hello-copy.txt renamed.txt        # 이름 변경
mv renamed.txt sub/                  # 이동
ls -la sub/
rm sub/renamed.txt                   # 파일 삭제
rm -r sub                            # 디렉토리 삭제
ls -la
cd ..
```

> ⚠️ **TODO**: 위 명령을 실행한 터미널 출력(명령어 + 결과)을 그대로 붙여넣기

```text
(실행 결과 붙여넣기)
```

## 6. 파일 권한 실습

### 개념: r / w / x 와 숫자 표기

`ls -l` 결과의 `-rwxr-xr-x`는 맨 앞 1글자(파일 종류: `-` 파일, `d` 디렉토리) 뒤에 **소유자 / 그룹 / 기타 사용자** 순서로 3글자씩 권한을 나타냅니다.

| 권한 | 숫자 | 파일에서 | 디렉토리에서 |
|---|---|---|---|
| `r` | 4 | 내용 읽기 | 목록 보기 (`ls`) |
| `w` | 2 | 내용 수정 | 안에 파일 생성·삭제 |
| `x` | 1 | 실행 | 들어가기 (`cd`) |

각 자리 숫자는 4 + 2 + 1을 더한 값입니다.

| 숫자 | 문자 | 해석 |
|---|---|---|
| `755` | `rwxr-xr-x` | 소유자 7(rwx), 그룹 5(r-x), 기타 5(r-x) → 소유자만 수정, 모두 읽기·실행 가능 (스크립트, 공용 디렉토리) |
| `644` | `rw-r--r--` | 소유자 6(rw-), 그룹 4(r--), 기타 4(r--) → 소유자만 수정, 모두 읽기 가능 (일반 문서의 기본값) |
| `700` | `rwx------` | 소유자만 모든 권한, 나머지는 접근 불가 (개인 디렉토리) |

### 실행 명령

`permission-test.txt`는 이전 실습에서 이미 755로 바꿔 커밋된 상태라(아래 Git 기록 참고), 변경 전/후 비교는 새 파일 `perm-file.txt`와 새 디렉토리 `perm-dir`로 수행했습니다.

```bash
# 파일 1개: perm-file.txt → 755
touch perm-file.txt
ls -l perm-file.txt     # 변경 전
chmod 755 perm-file.txt
ls -l perm-file.txt     # 변경 후

# 디렉토리 1개: perm-dir → 700
mkdir perm-dir
ls -ld perm-dir         # 변경 전
chmod 700 perm-dir
ls -ld perm-dir         # 변경 후
```

> ⚠️ **TODO**: 변경 전/후 `ls -l` 출력을 붙여넣고 아래 표 채우기

```text
(실행 결과 붙여넣기)
```

| 대상 | 변경 전 | 변경 후 |
|---|---|---|
| `perm-file.txt` | (TODO) | (TODO) |
| `perm-dir/` | (TODO) | (TODO) |

### 추가 증거: Git에 기록된 실행 권한

Git은 파일의 실행 권한(x)을 기록합니다. 저장소에 커밋된 `permission-test.txt`가 `100755`(실행 가능)로 저장되어 있어 `chmod 755`가 적용된 상태로 커밋되었음을 확인할 수 있습니다.

```text
$ git ls-files -s permission-test.txt
100755 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	permission-test.txt
```

## 7. Docker 설치 및 점검

### 설치: WSL 안에 Docker Engine 설치

Docker Desktop은 PC 메모리 4GB 이상이 필요한데 이 노트북은 3.4GB라서, WSL Ubuntu 안에 Docker Engine을 직접 설치했습니다.
`/etc/wsl.conf`에 `systemd=true`가 설정되어 있어 WSL이 켜질 때 Docker 데몬이 자동으로 시작됩니다.

```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER   # sudo 없이 docker 명령 사용
```

그룹 변경을 적용하려면 Windows PowerShell에서 `wsl --shutdown`을 실행한 뒤 WSL 터미널을 다시 엽니다.

### 점검

```bash
docker --version   # 클라이언트 버전 확인
docker info        # 데몬 동작 여부 (Server Version, Containers, Images 등이 나오면 정상)
```

> ⚠️ **TODO**: 두 명령의 출력을 붙여넣기 (`docker info`는 `Server Version`, `Containers`, `Images`, `Storage Driver` 부분 발췌 가능)

```text
(실행 결과 붙여넣기)
```

## 8. Docker 기본 운영 명령

> 실행 순서: `test-nginx` 컨테이너는 [11절](#11-포트-매핑)에서 만들고, `curl`과 브라우저로 접속한 **뒤에** 이 절의 명령을 실행해야 로그와 리소스 사용량이 나옵니다.

### 이미지 목록 — `docker images`

```text
$ docker images
(TODO: 실행 결과 붙여넣기)
```

### 컨테이너 목록 — `docker ps` / `docker ps -a`

`docker ps`는 실행 중인 컨테이너만, `docker ps -a`는 종료된 컨테이너까지 모두 보여줍니다.

```text
$ docker ps -a
(TODO: 실행 결과 붙여넣기)
```

### 로그 확인 — `docker logs test-nginx`

`curl`과 브라우저로 `localhost:8080`에 접속한 기록이 남습니다. 브라우저로 접속하면 `favicon.ico` 404 기록이 함께 남을 수 있는데, 브라우저가 아이콘을 자동 요청했지만 기본 nginx 페이지에 아이콘 파일이 없어서 생기는 것으로 동작에는 문제가 없습니다.

```text
$ docker logs test-nginx
(TODO: 실행 결과 붙여넣기)
```

### 리소스 확인 — `docker stats --no-stream`

`MEM USAGE / LIMIT`의 LIMIT은 WSL에 할당된 메모리(약 1.5GiB)로 표시됩니다.

```text
$ docker stats --no-stream
(TODO: 실행 결과 붙여넣기)
```

### 컨테이너 중지/재시작

```bash
docker stop test-nginx
docker ps -a        # STATUS가 Exited로 바뀐 것 확인
docker start test-nginx
docker ps           # 다시 Up 상태 확인
```

> ⚠️ **TODO**: 실행 결과 붙여넣기

```text
(실행 결과 붙여넣기)
```

## 9. 컨테이너 실행 실습

### hello-world

로컬에 이미지가 없으면 Docker Hub에서 자동으로 받은 뒤(`Unable to find image` → `Pulling`) 컨테이너를 실행합니다.

```text
$ docker run hello-world
(TODO: 실행 결과 붙여넣기 — "Hello from Docker!"까지)
```

### ubuntu 컨테이너 진입

```bash
docker run -it --name ubuntu-test ubuntu:22.04 bash
# 컨테이너 내부
ls
echo "hello from container"
exit
```

> ⚠️ **TODO**: 실행 결과 붙여넣기

```text
(실행 결과 붙여넣기)
```

### 컨테이너 종료/유지 차이 (attach vs exec)

| 방식 | 동작 | `exit` 했을 때 |
|---|---|---|
| `docker run -it` / `docker attach` | 컨테이너의 **메인 프로세스(PID 1, 여기선 bash)** 에 직접 연결 | 메인 프로세스가 끝나므로 **컨테이너도 종료(Exited)** |
| `Ctrl+P` → `Ctrl+Q` | attach 상태에서 연결만 끊기(detach) | 컨테이너는 **계속 실행** |
| `docker exec -it <이름> bash` | 실행 중인 컨테이너 안에 **새 프로세스**를 하나 더 띄움 | 그 프로세스만 끝나고 **컨테이너는 계속 실행** |

관찰용 명령:

```bash
docker run -dit --name keep-test ubuntu:22.04 bash
docker exec -it keep-test bash   # 안에서 exit
docker ps                        # keep-test 여전히 Up
docker attach keep-test          # 안에서 exit
docker ps -a                     # keep-test Exited
```

> ⚠️ **TODO**: 위 관찰 결과 붙여넣기

```text
(실행 결과 붙여넣기)
```

## 10. Dockerfile 커스텀 이미지

### 선택한 방식과 베이스

- **방식 (B)**: Linux 베이스 이미지 + 기능(패키지) 추가
- **베이스 이미지**: `ubuntu:22.04` (Docker Hub 공식 이미지)

### 작성한 Dockerfile

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y curl

CMD ["echo", "Hello from my container!"]
```

### 커스텀 포인트와 목적

| 줄 | 목적 |
|---|---|
| `FROM ubuntu:22.04` | `latest` 대신 버전 태그를 고정해 언제 빌드해도 같은 OS 버전이 나오도록 함 (재현성). 호스트(WSL)가 24.04여도 컨테이너는 이미지에 적힌 22.04로 동작 |
| `RUN apt-get update && apt-get install -y curl` | 컨테이너 안에서 HTTP 요청을 확인할 수 있도록 `curl` 설치. `update`와 `install`을 한 줄로 묶어 오래된 패키지 목록 캐시가 재사용되는 문제를 막고, `-y`로 빌드 중 확인 질문을 자동 승인 |
| `CMD ["echo", ...]` | 컨테이너 실행 시 기본 동작을 지정해 이미지가 정상 실행되는지 바로 확인 |

**이미지 vs 컨테이너**: 이미지는 실행에 필요한 파일·설정을 묶은 읽기 전용 템플릿이고, 컨테이너는 그 이미지로 띄운 실행 인스턴스입니다. 같은 이미지로 컨테이너를 여러 개 만들 수 있습니다.

### 빌드

buildx가 설치되어 있지 않으면 `DEPRECATED: The legacy builder is deprecated` 경고가 나오지만 빌드에는 문제가 없습니다.
마지막 줄이 `Successfully tagged my-ubuntu:latest`(legacy builder) 또는 `naming to docker.io/library/my-ubuntu ... done`(BuildKit)이면 성공입니다.

```text
$ docker build -t my-ubuntu .
(TODO: 실행 결과 붙여넣기 — 성공 메시지까지)
```

### 실행

```bash
docker run --rm my-ubuntu                   # CMD 메시지 확인
docker run --rm my-ubuntu curl --version    # 추가한 curl 동작 확인
```

> ⚠️ **TODO**: 실행 결과 붙여넣기 (예상: `Hello from my container!`, `curl 7.x ...`)

```text
(실행 결과 붙여넣기)
```

## 11. 포트 매핑

### 개념: 왜 포트 매핑이 필요한가

컨테이너는 호스트와 분리된 자체 네트워크를 가집니다. nginx가 컨테이너 안에서 80번 포트로 대기해도, 호스트(브라우저)에서는 그 포트에 바로 접근할 수 없습니다.
`-p <호스트 포트>:<컨테이너 포트>`는 "호스트의 이 포트로 들어온 요청을 컨테이너의 이 포트로 전달"하는 규칙입니다.
컨테이너마다 네트워크가 분리되어 있으므로 **컨테이너 포트(80)는 겹쳐도 되지만, 호스트 포트는 겹치면 안 됩니다.** (→ [트러블슈팅 1](#16-트러블슈팅))

### 접속 1: 8080 → 80

```bash
docker run -d --name test-nginx -p 8080:80 nginx
```

```text
$ curl http://localhost:8080
(TODO: 실행 결과 붙여넣기 — "Welcome to nginx!" HTML)
```

브라우저 접속 화면 (주소창 `localhost:8080` 포함):

> ⚠️ **TODO**: 브라우저 `localhost:8080` 화면을 `p/browser-8080.png`로 저장한 뒤 아래 줄 앞의 `<!--`, `-->` 지우기

<!-- ![localhost:8080](./p/browser-8080.png) -->

### 접속 2: 8090 → 80

`bind-nginx` 컨테이너를 8090 포트로 실행했습니다. ([12절](#12-바인드-마운트))

> ⚠️ **TODO**: 브라우저 `localhost:8090` 화면을 `p/browser-8090.png`로 저장한 뒤 아래 줄 앞의 `<!--`, `-->` 지우기

<!-- ![localhost:8090](./p/browser-8090.png) -->

## 12. 바인드 마운트

호스트의 폴더를 컨테이너 안의 경로에 **직접 연결**합니다. 호스트에서 파일을 바꾸면 컨테이너 안에서도 즉시 바뀝니다.

### 실행 명령

```bash
# ~/dev-mission 에서 실행
docker run -d \
  --name bind-nginx \
  -p 8090:80 \
  -v "$(pwd)/bind-test":/usr/share/nginx/html \
  nginx
```

### 변경 전/후 비교

```bash
echo '<h1>Hello from Host!</h1>' > bind-test/index.html
curl http://localhost:8090      # 변경 전

echo '<h1>Hello, Changed!</h1>' > bind-test/index.html
curl http://localhost:8090      # 변경 후 (컨테이너 재시작 없음)
```

| 시점 | 호스트 `bind-test/index.html` | `localhost:8090` 응답 |
|---|---|---|
| 변경 전 | `<h1>Hello from Host!</h1>` | `Hello from Host!` |
| 변경 후 | `<h1>Hello, Changed!</h1>` | `Hello, Changed!` (재시작 없이 즉시 반영) |

현재 저장소의 [`bind-test/index.html`](bind-test/index.html)은 변경 후 상태(`Hello, Changed!`)입니다.

> ⚠️ **TODO**: 변경 전/후 `curl` 출력을 붙여넣기

```text
(실행 결과 붙여넣기)
```

## 13. Docker 볼륨 영속성

### 개념: 볼륨이 필요한 이유

컨테이너 안에서 만든 파일은 컨테이너의 쓰기 레이어에 저장되어, **컨테이너를 삭제하면 함께 사라집니다.**
볼륨은 Docker가 컨테이너 밖에서 따로 관리하는 저장 공간이라, 컨테이너를 지우고 새로 만들어도 같은 볼륨을 연결하면 데이터가 남아 있습니다.

| 구분 | 바인드 마운트 | 볼륨 |
|---|---|---|
| 저장 위치 | 내가 지정한 호스트 폴더 | Docker가 관리하는 영역 |
| 주 용도 | 개발 중 소스 수정 즉시 반영 | DB 데이터 등 영속 데이터 보관 |
| 예시 | `-v $(pwd)/bind-test:/usr/share/nginx/html` | `-v my-volume:/data` |

### 생성 → 연결 → 쓰기 → 삭제 → 재연결

```bash
docker volume create my-volume
docker volume ls

# 1) 첫 번째 컨테이너에서 데이터 쓰기
docker run -it --name volume-test -v my-volume:/data ubuntu:22.04 bash
echo "볼륨 테스트 성공!" > /data/test.txt
cat /data/test.txt
exit

# 2) 컨테이너 삭제 후 목록에서 사라진 것 확인
docker rm volume-test
docker ps -a

# 3) 새 컨테이너에 같은 볼륨 연결 후 확인
docker run -it --name volume-test2 -v my-volume:/data ubuntu:22.04 bash
cat /data/test.txt
exit
```

> ⚠️ **TODO**: 위 과정 전체 출력 붙여넣기 (삭제 전 쓰기 → `docker rm` → 새 컨테이너에서 같은 내용이 나오는 것까지)

```text
(실행 결과 붙여넣기)
```

## 14. Git 설정 및 GitHub 연동

### 개념: Git vs GitHub

| 구분 | Git | GitHub |
|---|---|---|
| 정체 | 내 PC에서 동작하는 **버전 관리 도구** | Git 저장소를 올려두는 **원격 호스팅·협업 서비스** |
| 하는 일 | 커밋으로 변경 이력 기록, 브랜치, 되돌리기 | 원격 백업, 공유, Pull Request, 이슈, 코드 리뷰 |
| 인터넷 | 없어도 동작 | 필요 |

둘은 `git push`(내 PC → GitHub), `git pull`(GitHub → 내 PC)로 연결됩니다.

### 사용자 정보·기본 브랜치 설정

```bash
git config --global user.name "jjaeug"
git config --global user.email "<이메일>"
git config --global init.defaultBranch main
```

확인 (이메일은 `sed`로 가려서 출력):

```text
$ git config --list | sed 's/\(user.email=\).*/\1****@gmail.com/'
(TODO: 실행 결과 붙여넣기 — user.name, user.email(가림), init.defaultBranch=main 포함)
```

### GitHub 저장소 연동

```bash
# 처음 저장소를 만들 때
git init
git remote add origin https://github.com/jjaeug/codysseyE1-1.git
git push -u origin main

# 노트북(WSL)에서 작업할 때
git clone https://github.com/jjaeug/codysseyE1-1.git ~/dev-mission
```

```text
$ git remote -v
(TODO: 실행 결과 붙여넣기)
```

GitHub 저장소 화면:

![GitHub 저장소](./p/2.png)

### VSCode 연동

WSL에서 `code .`로 VSCode를 열어 GitHub 계정으로 로그인하고, 소스 제어 패널에서 커밋·push했습니다.

> ⚠️ **TODO**: VSCode 소스 제어 패널 또는 왼쪽 아래 계정 메뉴(GitHub 로그인 상태) 화면을 `p/vscode-github.png`로 저장(이메일 가리기)한 뒤 아래 줄 앞의 `<!--`, `-->` 지우기

<!-- ![VSCode GitHub 연동](./p/vscode-github.png) -->

## 15. 보안 및 개인정보 보호

| 점검 항목 | 처리 |
|---|---|
| 이메일 주소 | `git config --list` 출력은 `sed`로 가려서 기록, VSCode 화면은 가림 처리 |
| 비밀번호 / 토큰 / 개인키 | 문서·스크린샷에 포함하지 않음 |
| GitHub 인증 정보 | VSCode의 GitHub 로그인 사용, 저장소 파일에 저장하지 않음 |

민감정보가 노출된 경우의 대응 절차:

1. 노출된 토큰은 GitHub `Settings → Developer settings → Personal access tokens`에서 **즉시 폐기(revoke)** 후 재발급
2. 문서·이미지에서 해당 내용을 지우고 커밋
3. 커밋 이력에 남은 경우 `git filter-repo` 등으로 히스토리에서 제거 후 강제 push

## 16. 트러블슈팅

### 케이스 1: 포트 충돌

- **문제**: `test-nginx`가 실행 중인 상태에서 두 번째 nginx 컨테이너를 `-p 8080:80`으로 실행하자 실패했다.
- **원인 가설**: 호스트의 8080 포트를 이미 다른 컨테이너가 사용하고 있다.
- **확인**: `docker ps`의 PORTS 열에서 `test-nginx`가 `0.0.0.0:8080->80/tcp`를 사용 중인 것을 확인 ([8절](#8-docker-기본-운영-명령)).
- **해결**: 두 번째 컨테이너(`bind-nginx`)의 호스트 포트를 8090으로 바꿔 실행 (`-p 8090:80`). 컨테이너 내부 포트 80은 그대로 둬도 된다.
- **대안**: 기존 컨테이너가 필요 없으면 `docker stop test-nginx` 후 8080을 재사용.

재현 로그 (`test-nginx` 실행 중일 때):

```text
$ docker run -d --name conflict-test -p 8080:80 nginx
(TODO: 실행 결과 붙여넣기 — "port is already allocated" 에러)
$ docker rm conflict-test
```

### 케이스 2: Dockerfile 빌드 중 apt-get 실패

- **문제**: `RUN apt-get install curl` 단계에서 빌드가 실패했다.
- **원인 가설**: `apt-get install`이 `Do you want to continue? [Y/n]` 확인을 요구하는데, `docker build`는 키보드 입력을 받을 수 없다.
- **확인**: 빌드 로그에 `Do you want to continue? [Y/n] Abort.`가 출력된 뒤 RUN 단계가 `returned a non-zero code`로 종료됨 → 입력 대기로 멈춘 것이 아니라 자동 취소(Abort)된 것.
- **해결**: `apt-get install -y curl`로 자동 승인 옵션을 추가해 빌드 성공.
- **참고**: `tzdata`처럼 설치 중 설정 질문을 하는 패키지는 `ENV DEBIAN_FRONTEND=noninteractive`도 함께 지정해야 한다.

## 17. 저장소 구성 및 재현 방법

```text
codysseyE1-1/
├── README.md              # 기술 문서 (이 파일)
├── Dockerfile             # ubuntu:22.04 + curl 커스텀 이미지
├── bind-test/
│   └── index.html         # 바인드 마운트로 nginx에 연결한 페이지
├── hello.txt              # 터미널 실습 파일
├── permission-test.txt    # 권한 실습 파일 (755)
└── p/                     # 스크린샷 (브라우저 접속, GitHub, VSCode)
```

### 다른 환경에서 재현하기

```bash
git clone https://github.com/jjaeug/codysseyE1-1.git ~/dev-mission
cd ~/dev-mission

# 커스텀 이미지
docker build -t my-ubuntu .
docker run --rm my-ubuntu

# 포트 매핑
docker run -d --name test-nginx -p 8080:80 nginx
curl http://localhost:8080

# 바인드 마운트
docker run -d --name bind-nginx -p 8090:80 -v "$(pwd)/bind-test":/usr/share/nginx/html nginx
curl http://localhost:8090

# 볼륨
docker volume create my-volume
docker run --rm -v my-volume:/data ubuntu:22.04 bash -c 'echo "볼륨 테스트 성공!" > /data/test.txt'
docker run --rm -v my-volume:/data ubuntu:22.04 cat /data/test.txt

# 정리
docker rm -f test-nginx bind-nginx
docker volume rm my-volume
```

주의사항:

- 8080/8090 포트를 다른 프로그램이 쓰고 있으면 `-p 8081:80`처럼 호스트 포트만 바꿔서 실행
- Windows에서는 WSL2 Ubuntu의 홈 디렉토리(`~`)에 clone해서 실행 (`/mnt/c` 아래에서는 `chmod`가 반영되지 않음)
- Docker 데몬이 실행 중이어야 함 (`docker info`로 확인)
- 메모리가 작은 환경에서는 다 쓴 컨테이너를 `docker rm -f`로 정리
