# Logback 설정 파일

Logback은 Java 애플리케이션을 위한 강력하고 유연한 로깅 프레임워크입니다. Logback 설정 파일은 일반적으로 XML 형식으로 작성되며, 로깅의 동작을 정의합니다.

---
# 로그 단계별로 파일 저장 설정

Logback 설정 파일에서 로그를 단계별로 다른 파일에 저장하도록 설정했습니다. 각 로그 단계(INFO, WARN, ERROR)는 별도의 RollingFileAppender를 사용하여 개별 파일에 기록됩니다.

```angular2html
<appender name="ALL_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"></appender>
<appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"></appender>
<appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"></appender>
<appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"></appender>
```

---
# 로그 출력 레벨 설정

Root Logger는 모든 로거의 상위 로거입니다. 일반적으로 애플리케이션의 기본 로깅 레벨과 Appender를 정의합니다.

```angular2html
    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ALL_FILE"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="WARN_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>
```

---
# 로그 저장 파일 저장 및 관리 설정

```angular2html
    <property name="MAX_HISTORY" value="10"/>
    <property name="MAX_FILE_SIZE" value="10MB"/>
    <property name="TOTAL_SIZE" value="100MB"/>
```

## 1. MAX_HISTORY
- **설명**: 이 속성은 유지할 로그 파일의 최대 개수를 정의합니다.
- **사용 예**: 만약 `MAX_HISTORY`가 10으로 설정되어 있다면, 로그 백업 파일은 최대 10개까지만 유지됩니다. 새로운 로그 파일이 생성되면 가장 오래된 파일이 삭제됩니다.
- **용도**: 주로 RollingFileAppender에서 사용됩니다. 롤링 정책(rolling policy)을 설정할 때 유용합니다.

## 2. MAX_FILE_SIZE
- **설명**: 이 속성은 하나의 로그 파일이 가질 수 있는 최대 크기를 정의합니다.
- **사용 예**: `MAX_FILE_SIZE`가 10MB로 설정된 경우, 로그 파일이 10MB에 도달하면 새로운 로그 파일이 생성됩니다.
- **용도**: 파일 크기 기반의 롤링을 설정할 때 사용됩니다. 예를 들어, 로그 파일이 일정 크기를 초과하면 새로운 파일로 로그를 기록합니다.

## 3. TOTAL_SIZE
- **설명**: 이 속성은 모든 로그 파일이 차지할 수 있는 총 디스크 용량을 정의합니다.
- **사용 예**: `TOTAL_SIZE`가 100MB로 설정된 경우, 모든 로그 파일의 크기가 합쳐서 100MB를 넘지 않도록 관리됩니다. 만약 총 크기가 100MB를 초과하면 가장 오래된 파일부터 삭제됩니다.
- **용도**: 디스크 공간을 효율적으로 관리하기 위해 사용됩니다. 전체 로그 파일의 크기를 제한함으로써 디스크 공간 부족 문제를 방지할 수 있습니다.

---
# pattern 설정

```angular2html
<encoder>
    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
</encoder>
```

이 설정은 로그 메세지의 포맷을 정의 합니다.

- **`%d{yyyy-MM-dd HH:mm:ss.SSS}`**: 로그 메시지가 기록된 날짜와 시간을 `년-월-일 시:분:초.밀리초` 형식으로 표시합니다.
- **`[%thread]`**: 로그가 발생한 스레드 이름을 표시합니다.
- **`%-5level`**: 로그 레벨 (예: INFO, WARN, ERROR)을 5자리로 왼쪽 정렬하여 표시합니다.
- **`%logger{36}`**: 로거의 이름을 최대 36자리까지 표시합니다.
- **`- %msg`**: 로그 메시지 본문을 출력합니다.
- **`%n`**: 줄 바꿈 문자를 추가합니다.

# fileNamePattern 설정

```angular2html
<fileNamePattern>${LOG_PATH}/archived/${APP_NAME}-all.%d{yyyy-MM-dd}.log</fileNamePattern>
```

이 설정은 Logback에서 롤링 로그 파일의 이름 패턴을 정의합니다. 다음은 각 요소에 대한 설명입니다:

- **`${LOG_PATH}`**: 로그 파일이 저장될 기본 경로를 나타내는 변수입니다. 설정 파일 내에서 정의된 `LOG_PATH` 속성에 의해 설정됩니다.
- **`archived`**: 로그 파일이 저장될 서브디렉토리 이름입니다. 이 예에서는 로그 파일이 `archived`라는 하위 디렉토리에 저장됩니다.
- **`${APP_NAME}`**: 애플리케이션의 이름을 나타내는 변수입니다. 설정 파일 내에서 정의된 `APP_NAME` 속성에 의해 설정됩니다. 이 값은 로그 파일 이름에 포함되어 애플리케이션의 이름을 식별할 수 있게 합니다.
- **`-all.`**: 로그 파일의 기본 이름에 붙는 고정 접미사입니다. 이 예에서는 모든 로그가 저장될 파일이 이 이름 패턴을 따릅니다.
- **`%d{yyyy-MM-dd}`**: 날짜 형식을 나타내는 포맷입니다. 로그 파일 이름에 날짜를 포함시키는 데 사용됩니다. 여기서 `yyyy-MM-dd`는 연도-월-일 형식으로 날짜를 표시합니다. 이 부분은 로그 파일이 생성된 날짜를 파일 이름에 포함시키기 위해 사용됩니다.
- **`.log`**: 로그 파일의 확장자입니다. 이 예에서는 `.log`로 파일이 저장됩니다.

---
# 예제

## logback-spring.xml 파일
```angular2html
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <script/>
    <!--  Spring 애플리케이션 이름과 로그 경로를 가져오는 속성 정의  -->
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>
    <springProperty scope="context" name="logPath" source="logging.file.path"/>
    <!--  기본 로그 경로와 애플리케이션 이름 속성 설정  -->
    <property name="LOG_PATH" value="./logs"/>
    <property name="APP_NAME" value="5ritang-store-front"/>
    <!--  로그 파일 크기 설정  -->
    <property name="MAX_HISTORY" value="10"/>
    <property name="MAX_FILE_SIZE" value="10MB"/>
    <property name="TOTAL_SIZE" value="100MB"/>
    <!--  Spring Boot의 기본 로그백 설정 파일 포함  -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>


    <!--  기존의 파일 appender 설정 유지  -->
    <appender name="ALL_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APP_NAME}-all.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/archived/${APP_NAME}-all.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>${MAX_HISTORY}</maxHistory>
            <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>
            <totalSizeCap>${TOTAL_SIZE}</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!--  INFO 레벨 파일 appender 설정  -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APP_NAME}-info.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/archived/${APP_NAME}-info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>${MAX_HISTORY}</maxHistory>
            <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>
            <totalSizeCap>${TOTAL_SIZE}</totalSizeCap>
        </rollingPolicy>
        <!--  INFO 레벨 필터 추가  -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--  WARN 레벨 파일 appender 설정  -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APP_NAME}-warn.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/archived/${APP_NAME}-warn.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>${MAX_HISTORY}</maxHistory>
            <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>
            <totalSizeCap>${TOTAL_SIZE}</totalSizeCap>
        </rollingPolicy>
        <!--  WARN 레벨 필터 추가  -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>WARN</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--  ERROR 레벨 파일 appender 설정  -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APP_NAME}-error.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/archived/${APP_NAME}-error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>${MAX_HISTORY}</maxHistory>
            <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>
            <totalSizeCap>${TOTAL_SIZE}</totalSizeCap>
        </rollingPolicy>
        <!--  ERROR 레벨 필터 추가  -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- LogNCrashAppender 추가 -->
    <springProfile name="prod">
        <appender name="LOG_N_CRASH" class="com.nhnacademy.bookstorefront.global.util.LogNCrashAppender"/>

        <root level="info">
            <appender-ref ref="LOG_N_CRASH"/>
        </root>
    </springProfile>

    <!--  root 로거 설정  -->
    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ALL_FILE"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="WARN_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>

</configuration>
```