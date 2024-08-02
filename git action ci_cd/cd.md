# Continuous Deployment to Ubuntu Server

이 문서는 GitHub Actions를 사용하여 무중단 Continuous Deployment(CD : 지속적 배포) 파이프라인을 설정하는 방법을 설명합니다. 이 파이프라인은 `push` 이벤트가 발생할 때 `develop` 브랜치에 대해 실행됩니다.

---

## Trigger

```yaml
on:
  push:
    branches:
      - develop
```
- on: 워크플로우가 언제 트리거되는지 정의합니다.
- push: 코드가 리포지토리에 푸시될 때 트리거됩니다.
- branches: 특정 브랜치에 대한 조건을 정의합니다. 여기서는 develop 브랜치에 대해 트리거됩니다.

---
# Jobs

각 워크플로우 파일은 여러 작업(jobs)으로 구성될 수 있습니다. 이 예제에서는 build라는 작업이 정의되어 있습니다.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

- jobs: 여러 작업을 정의할 수 있는 섹션입니다.
- build: 작업의 이름입니다.
- runs-on: 작업을 실행할 환경을 정의합니다. 여기서는 ubuntu-latest가 사용됩니다.

---
# Steps

build 작업은 여러 단계로 구성되어 있습니다.

# 1.소스 코드 체크아웃
```yaml
steps:
  - uses: actions/checkout@v3
```
- uses: 외부 액션을 사용할 때 사용합니다. actions/checkout@v3는 리포지토리의 코드를 체크아웃합니다.

# 2.JDK 21 설정
```yaml
steps:
  -- name: Set up JDK 21
  uses: actions/setup-java@v3
  with:
    java-version: '21'
    distribution: 'temurin'
    cache: maven
```

- name: 이 단계의 이름을 정의합니다.
- uses: actions/setup-java@v3 액션을 사용하여 JDK를 설정합니다.
- with: 추가적인 설정을 포함합니다.
  - java-version: JDK 버전을 정의합니다. 여기서는 21 버전을 사용합니다.
  - distribution: JDK 배포판을 정의합니다. 여기서는 temurin이 사용됩니다.
  - cache: Maven 종속성 캐시를 설정합니다.

# 3.Maven을 사용한 빌드

```yaml
steps:
  - name: Build with Maven
    run: mvn -B package --file pom.xml
    env:
      REDIS_PASSWORD: ${{ secrets.REDIS_PASSWORD }}
```
- name: 단계의 이름을 정의합니다.
- run: 쉘 명령어를 실행합니다. 여기서는 Maven을 사용하여 빌드합니다.
- env: 환경 변수를 설정합니다. 보안을 위해 Redis 비밀번호는 GitHub Secrets에 저장된 REDIS_PASSWORD를 사용합니다.

# 4.파일 전송 (SCP를 사용)

```yaml
- name: Copy files via SCP
  uses: appleboy/scp-action@v0.1.7
  with:
    host: ${{ secrets.SSH_IP }}
    username: ${{ secrets.SSH_ID }}
    key: ${{ secrets.SSH_KEY }}
    port: ${{ secrets.SSH_PORT }}
    source: "target/*.jar"
    target: "~/"
    rm: false
    timeout: '30s'
    command_timeout: '10m'
    use_insecure_cipher: false
    debug: true

```
- name: 단계의 이름을 정의합니다.
- uses: SCP를 사용하여 파일을 전송하는 액션입니다.
- with: SCP 설정을 정의합니다.
  - host: 대상 서버의 IP 주소입니다.
  - username: SSH 접속을 위한 사용자 이름입니다.
  - key: SSH 키입니다.
  - port: SSH 접속 포트입니다.
  - source: 전송할 파일의 경로입니다. 여기서는 target/*.jar를 전송합니다.
  - target: 파일이 복사될 대상 경로입니다.
  - rm: 복사 후 원본 파일을 삭제할지 여부를 정의합니다. 여기서는 false로 설정되어 있습니다.
  - timeout: SCP 연결의 타임아웃 시간입니다.
  - command_timeout: 명령 실행의 타임아웃 시간입니다.
  - use_insecure_cipher: 비보안 암호화 사용 여부를 정의합니다.
  - debug: 디버깅 모드를 활성화합니다.

# 5. 기존 애플리케이션 종료 (포트 A)

```yaml
- name: Stop existing Spring Boot application A
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.SSH_IP }}
    username: ${{ secrets.SSH_ID }}
    key: ${{ secrets.SSH_KEY }}
    port: ${{ secrets.SSH_PORT }}
    script: |
      echo "Stopping existing application..."
      pid=$(lsof -t -i:${{ secrets.FRONT_PORT_A }}) && kill -9 $pid || echo "No application running on port ${{ secrets.FRONT_PORT_A }}"

```
- uses: SSH를 사용하여 원격 서버에서 명령을 실행하는 액션입니다.
- with: SSH 설정과 실행할 스크립트를 정의합니다.
  - script: 기존 애플리케이션을 종료하는 명령입니다. 포트 A에서 실행 중인 애플리케이션을 종료합니다.

# 6. 새로운 애플리케이션 시작 (포트 A)

```yaml
- name: Execute shell script A
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.SSH_IP }}
    username: ${{ secrets.SSH_ID }}
    key: ${{ secrets.SSH_KEY }}
    port: ${{ secrets.SSH_PORT }}
    script_stop: true
    script: |
      echo "Starting new application..."
      nohup java -Xmx256m -jar ~/target/BOOK-STORE-FRONT.jar --spring.profiles.active=${{secrets.SPRING_PROFILE}} --server.port=${{ secrets.FRONT_PORT_A }} --spring.data.redis.password=${{ secrets.REDIS_PASSWORD }} > FRONT-A.log 2>&1 &
      echo "New application started. Check app.log for details."
```
- uses: SSH를 사용하여 원격 서버에서 명령을 실행하는 액션입니다.
- with: SSH 설정과 실행할 스크립트를 정의합니다.
  - script: 새로운 애플리케이션을 포트 A에서 시작하는 명령입니다. 로그는 FRONT-A.log에 기록됩니다.

# 7. JAR 파일 실행 확인 (포트 A)

```yaml
- name: Check if JAR is running by checking log file
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.SSH_IP }}
    username: ${{ secrets.SSH_ID }}
    key: ${{ secrets.SSH_KEY }}
    port: ${{ secrets.SSH_PORT }}
    script: |
      echo "Checking if application is running by checking log file..."
      for i in {1..30}; do
        if grep -q "Started BookStoreFrontApplication" FRONT-A.log; then
          echo "Application is running"
          break
        else
          echo "Waiting for application to start..."
          sleep 5
        fi
      done
      if ! grep -q "Started BookStoreFrontApplication" FRONT-A.log; then
        echo "Application did not start in expected time" >&2
        exit 1
      fi
```
- uses: SSH를 사용하여 원격 서버에서 명령을 실행하는 액션입니다.
- with: SSH 설정과 실행할 스크립트를 정의합니다.
  - script: 무중단 배포를 위해 애플리케이션 로그를 확인하여 애플리케이션이 실행 중인지 확인하는 명령입니다.

# 8. 기존 애플리케이션 종료 (포트 B)

```yaml
- name: Stop existing Spring Boot application B
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.SSH_IP }}
    username: ${{ secrets.SSH_ID }}
    key: ${{ secrets.SSH_KEY }}
    port: ${{ secrets.SSH_PORT }}
    script: |
      echo "Stopping existing application..."
      pid=$(lsof -t -i:${{ secrets.FRONT_PORT_B }}) && kill -9 $pid || echo "No application running on port ${{ secrets.FRONT_PORT_B }}"
```
- uses: SSH를 사용하여 원격 서버에서 명령을 실행하는 액션입니다.
- with: SSH 설정과 실행할 스크립트를 정의합니다.
  - script: 기존 애플리케이션을 종료하는 명령입니다. 포트 B에서 실행 중인 애플리케이션을 종료합니다.

# 9. 새로운 애플리케이션 시작 (포트 B)

```yaml
- name: Execute shell script B
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.SSH_IP }}
    username: ${{ secrets.SSH_ID }}
    key: ${{ secrets.SSH_KEY }}
    port: ${{ secrets.SSH_PORT }}
    script_stop: true
    script: |
      echo "Starting new application..."
      nohup java -Xmx256m -jar ~/target/BOOK-STORE-FRONT.jar --spring.profiles.active=prod --server.port=${{ secrets.FRONT_PORT_B }} --spring.data.redis.password=${{ secrets.REDIS_PASSWORD }} > FRONT-B.log 2>&1 &
      echo "New application started. Check app.log for details."
```
- uses: SSH를 사용하여 원격 서버에서 명령을 실행하는 액션입니다.
- with: SSH 설정과 실행할 스크립트를 정의합니다.
   - script: 새로운 애플리케이션을 포트 B에서 시작하는 명령입니다. 로그는 FRONT-B.log에 기록됩니다.

---
# 전체 코드

```yaml
name: CD to Ubuntu Server #

on:
  push:
    branches:
      - develop

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 21
        uses: actions/setup-java@v3
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: maven
      - name: Build with Maven
        run: mvn -B package --file pom.xml
        env:
          REDIS_PASSWORD: ${{ secrets.REDIS_PASSWORD }}

      - name: Copy files via SCP
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.SSH_IP }}
          username: ${{ secrets.SSH_ID }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{secrets.SSH_PORT}}
          source: "target/*.jar"
          target: "~/"
          rm: false
          timeout: '30s'
          command_timeout: '10m'
          use_insecure_cipher: false
          debug: true


      # 앱 포트로 실행된 jar 파일 종료
      - name: Stop existing Spring Boot application A
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_IP }}
          username: ${{ secrets.SSH_ID }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            echo "Stopping existing application..."
            pid=$(lsof -t -i:${{ secrets.FRONT_PORT_A }}) && kill -9 $pid || echo "No application running on port ${{ secrets.FRONT_PORT_A }}"

      #앱 포트로 배포 한 jar 파일 실행
      - name: execute shell script A
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_IP }}
          username: ${{ secrets.SSH_ID }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script_stop: true
          script: |
            echo "Starting new application..."
            nohup java -Xmx256m -jar ~/target/BOOK-STORE-FRONT.jar --spring.profiles.active=${{secrets.SPRING_PROFILE}} --server.port=${{ secrets.FRONT_PORT_A }} --spring.data.redis.password=${{ secrets.REDIS_PASSWORD }} > FRONT-A.log 2>&1 &
            echo "New application started. Check app.log for details."

      # JAR 파일 실행 확인
      - name: Check if JAR is running by checking log file
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_IP }}
          username: ${{ secrets.SSH_ID }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            echo "Checking if application is running by checking log file..."
            for i in {1..30}; do
              if grep -q "Started BookStoreFrontApplication" FRONT-A.log; then
                echo "Application is running"
                break
              else
                echo "Waiting for application to start..."
                sleep 5
              fi
            done
            if ! grep -q "Started BookStoreFrontApplication" FRONT-A.log; then
              echo "Application did not start in expected time" >&2
              exit 1
            fi

      # 앱 포트로 실행된 jar 파일 종료
      - name: Stop existing Spring Boot application B
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_IP }}
          username: ${{ secrets.SSH_ID }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            echo "Stopping existing application..."
            pid=$(lsof -t -i:${{ secrets.FRONT_PORT_B }}) && kill -9 $pid || echo "No application running on port ${{ secrets.FRONT_PORT_B }}"

      #앱 포트로 배포 한 jar 파일 실행
      - name: execute shell script B
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_IP }}
          username: ${{ secrets.SSH_ID }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script_stop: true
          script: |
            echo "Starting new application..."
            nohup java -Xmx256m -jar ~/target/BOOK-STORE-FRONT.jar --spring.profiles.active=prod --server.port=${{ secrets.FRONT_PORT_B }} --spring.data.redis.password=${{ secrets.REDIS_PASSWORD }} > FRONT-B.log 2>&1 &
            echo "New application started. Check app.log for details."
```