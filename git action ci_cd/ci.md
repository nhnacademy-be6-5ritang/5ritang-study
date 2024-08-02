# Continuous Integration

이 문서는 GitHub Actions를 사용하여 Continuous Integration(CI : 지속적 통합) 파이프라인을 설정하는 방법을 설명합니다. 이 파이프라인은 `pull_request` 이벤트가 발생할 때 `develop` 브랜치에 대해 실행됩니다.

---
## Trigger

```yaml
on:
  pull_request:
    branches: [ "develop" ]
```
- on: 워크플로우가 언제 트리거되는지 정의합니다.
- pull_request: 풀 리퀘스트가 생성되거나 업데이트될 때 트리거됩니다.
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

```yaml
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
```
- uses: 외부 액션을 사용할 때 사용합니다. actions/checkout@v3는 리포지토리의 코드를 체크아웃합니다.
- name: 이 단계의 이름을 정의합니다.
- uses: actions/setup-java@v3 액션을 사용하여 JDK를 설정합니다.
- with: 이 섹션은 추가적인 설정을 포함합니다.
- java-version: JDK 버전을 정의합니다. 여기서는 21 버전을 사용합니다.
- distribution: JDK 배포판을 정의합니다. 여기서는 temurin이 사용됩니다.
- cache: Maven 종속성 캐시를 설정합니다.
- name: 단계의 이름을 정의합니다.
- run: 쉘 명령어를 실행합니다. 여기서는 Maven을 사용하여 빌드합니다.

---
# SonarQube를 사용한 테스트 커버리지 측정

```yaml
- name: Build and analyze with SonarQube
  env:
    SONAR_PROJECT_KEY: "5ritang-front"
    SONAR_PROJECT_NAME: "5ritang-front"
  run: mvn sonar:sonar -Dsonar.projectName=${{ env.SONAR_PROJECT_NAME }} -Dsonar.projectKey=${{ env.SONAR_PROJECT_KEY }} -Dsonar.host.url=${{secrets.SONAR_HOST}} -Dsonar.login=${{secrets.SONAR_TOKEN}}
```
- env: 환경 변수를 설정합니다.
  - SONAR_PROJECT_KEY: SonarQube 프로젝트 키를 정의합니다.
  - SONAR_PROJECT_NAME: SonarQube 프로젝트 이름을 정의합니다.
- run: SonarQube 분석을 수행하는 Maven 명령어를 실행합니다. 
  - 여기서 보안을 위해 SONAR_HOST와 SONAR_TOKEN는 GitHub Secrets에 저장하여 사용합니다.

---
# 전체 코드

```yaml
name: Continuous Integration

on:
  pull_request:
    branches: [ "develop" ]

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

      # 소나큐브 테스트 커버리지 측정
      - name: Build and analyze with SonarQube
        env:
          SONAR_PROJECT_KEY: "5ritang-front"
          SONAR_PROJECT_NAME: "5ritang-front"
        run: mvn sonar:sonar -Dsonar.projectName=${{ env.SONAR_PROJECT_NAME }} -Dsonar.projectKey=${{ env.SONAR_PROJECT_KEY }} -Dsonar.host.url=${{secrets.SONAR_HOST}} -Dsonar.login=${{secrets.SONAR_TOKEN}}
```
