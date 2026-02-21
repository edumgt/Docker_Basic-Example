# WSL(Ubuntu) + VSCode 환경에서 실행하기

이 문서는 **WSL 기반 Ubuntu + VSCode(원격 WSL)** 환경에서 이 프로젝트를 실행하기 위해 필요한 **라이브러리 설치**, **소스 설정 변경**, **Docker 설정 변경** 과정을 정리합니다.

## 1) 사전 준비

### 1-1. WSL/VSCode 기본 구성

- Windows에서 WSL2 및 Ubuntu 설치
- VSCode에 **Remote - WSL** 확장 설치
- VSCode로 Ubuntu WSL 환경 열기 (좌측 하단 `><` → WSL)

### 1-2. 필수 패키지 설치 (Ubuntu)

```bash
sudo apt update
sudo apt install -y git curl unzip openjdk-17-jdk
```

> 이 프로젝트는 `Java 17`을 사용합니다 (`build.gradle`의 `toolchain` 설정).

### 1-3. Docker 설치

WSL에서 Docker를 사용할 경우 보통 **Docker Desktop + WSL integration**을 사용합니다.

- Windows에 Docker Desktop 설치
- Docker Desktop → Settings → Resources → WSL Integration에서 Ubuntu 활성화
- WSL 터미널에서 Docker 동작 확인

```bash
docker --version
docker compose version
```

## 2) 데이터/로그 디렉터리 준비

프로젝트에서 파일 저장 및 로그 경로로 사용될 디렉터리를 WSL 환경에 맞게 준비합니다.

```bash
mkdir -p ~/spring-data/logs
```

## 3) 소스 설정 (WSL 자동 대응)

이제 기본값이 Linux/WSL 친화적으로 리팩토링되어, 별도 소스 수정 없이 실행할 수 있습니다.

- `app.cdn-directory`: `${APP_CDN_DIRECTORY:${user.home}/spring-data/}`
- `log4j2-dev.xml`의 `APP_LOG_ROOT`: `${sys:APP_LOG_ROOT:-${sys:user.home}/spring-data/logs}`

필요하면 환경 변수로 경로를 덮어쓸 수 있습니다.

```bash
export APP_CDN_DIRECTORY=/home/<your-user>/spring-data/
export APP_LOG_ROOT=/home/<your-user>/spring-data/logs
```

### 3-1. DB 접속 설정 확인

`application-dev.yml`의 `spring.datasource` 설정을 WSL 환경에 맞게 수정합니다.

예시 (로컬 DB 또는 Docker MySQL에 맞춤):

```yml
spring:
  datasource:
    url: jdbc:mysql://${MYSQL_HOST:localhost}:3308/spring_starter?allowPublicKeyRetrieval=true&useSSL=false
    username: root
    password: <your-password>
```

## 4) Docker 설정 (환경 변수 기반)

`docker-compose.yml`은 WSL에서 바로 동작하도록 기본 Linux 경로를 사용합니다.

```yml
volumes:
  - ${SPRING_DATA_DIR:-/home/ubuntu/spring-data}:/usr/spring-data
  - ${SPRING_DATA_LOG_DIR:-/home/ubuntu/spring-data/logs}:/usr/spring-data/logs
```

사용자 계정 경로를 쓰고 싶다면 실행 전에 지정하세요.

```bash
export SPRING_DATA_DIR=/home/<your-user>/spring-data
export SPRING_DATA_LOG_DIR=/home/<your-user>/spring-data/logs
```

## 5) 실행 방법

### 5-1. 로컬 실행 (WSL에서 직접 실행)

```bash
./gradlew bootRun
```

또는 개발 프로파일로 실행:

```bash
./gradlew runDev
```

### 5-2. Docker 실행

```bash
docker compose build
docker compose up -d
```

컨테이너 포트 매핑은 기본적으로 `9080 -> 8080`, `9081 -> 8080`으로 되어 있습니다.

## 6) 참고

- API 문서: `/api-docs`, `/swagger-ui/index.html`
- 기본 서버 포트: `8080`
- 개발 프로파일: `spring.profiles.active=dev`

---

필요 시 위 설정을 기반으로 팀 환경에 맞는 `.env` 파일 또는 별도 `application-wsl.yml`을 추가하여 설정을 분리하는 것을 권장합니다.
