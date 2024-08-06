# Log & Crash Search

Log & Crash Search 는 클라이언트와 서버의 로그를 수집하여 원하는 로그를 검색하고 조회하는 시스템입니다. 모바일 앱에서 발생하는 크래시를 분석하고 통계 작업을 수행하여 크래시 발생 원인에 대한 다양한 정보도 제공합니다.

---
## Log & Crash Search 특·장점

- **`게임 서버와 로그 서버 분리`**: 대량의 로그로 인한 문제점을 제거할 수 있습니다.
- **`통합 오류 관리`**: 배포된 모든 클라이언트에서 발생하는 오류를 한 곳에 모아서 조회하고 분석이 가능합니다.
- **`빠른 로그 검색`**: 대량의 로그에서 원하는 로그를 빠르게 검색할 수 있으며, 로그 패턴별 조회가 가능합니다.
- **`실시간 모니터링`**: 로그와 크래시를 5분 단위로 실시간 모니터링할 수 있습니다.
- **`서비스 연속성`**: 사용량 증가에 따른 로그량이 증가하더라도 서비스 정지 없이 이용 가능합니다.
- **`다양한 지원 환경`**: 오류, 크래시 덤프, 웹 어플리케이션 로그, 커스텀 메시지 형식 등을 지원하며, Windows, Linux와 Java 환경에서 사용 가능합니다.

출처: https://docs.nhncloud.com/ko/Data%20&%20Analytics/Log%20&%20Crash%20Search/ko/Overview/

---
# 기본 파라미터

- **`projectName`**: `string`, 필수  
  [in] 앱키.

- **`projectVersion`**: `string`, 필수  
  [in] 버전. 사용자 지정 가능. "A~Z, a~z, 0~9,-._"만 포함.

- **`logVersion`**: `string`, 필수  
  [in] 로그 포맷 버전. "v2".

- **`body`**: `string`, 옵션  
  [in] 로그 메시지.

- **`logSource`**: `string`, 옵션  
  [in] 로그 소스. Log Search에서 필터링을 위해 사용. 정의되지 않으면 "http".

- **`logType`**: `string`, 옵션  
  [in] 로그 타입. Log Search에서 필터링을 위해 사용. 정의되지 않으면 "log".

- **`platform`**: `string`, 옵션  
  [in] 로그를 보내는 단말 이름

- **`host`**: `string`, 옵션  
  [in] 로그를 보내는 단말의 주소. 정의되지 않으면 수집 서버에서 peer-address를 사용해 자동으로 채움.

- **`logLevel`**: `string`, 옵션  
  [in] Syslog 이벤트용.

출처: https://docs.nhncloud.com/ko/Data%20&%20Analytics/Log%20&%20Crash%20Search/ko/api-guide/

---
# 예제

## logback-spring.xml 파일
- 운영 환경에서만 Info 레벨 수준으로 작동하도록 설정
```angular2html
    <!-- LogNCrashAppender 추가 -->
    <springProfile name="prod">
        <appender name="LOG_N_CRASH" class="com.nhnacademy.bookstorefront.global.util.LogNCrashAppender"/>

        <root level="info">
            <appender-ref ref="LOG_N_CRASH"/>
        </root>
    </springProfile>
```

## Java 코드
```java
public class LogNCrashAppender extends AppenderBase<ILoggingEvent> {
	private final RestTemplate restTemplate = new RestTemplate();
	private static final Logger logger = LoggerFactory.getLogger(LogNCrashAppender.class);

	@Override
	protected void append(ILoggingEvent loggingEvent) {
		Map<String, Object> logData = new HashMap<>();
		logData.put("projectName", "Xyx7DoyszcG66ULx");
		logData.put("projectVersion", "1.0.0");
		logData.put("logVersion", "v2");
		logData.put("body", loggingEvent.getFormattedMessage());
		logData.put("logSource", "http");
		logData.put("logType", "log");
		logData.put("platform", "5ritang-Front");
		logData.put("host", "192.168.0.75");
		logData.put("logLevel", loggingEvent.getLevel().toString());

		String url = "https://api-logncrash.nhncloudservice.com/v2/log";

		try {
			restTemplate.postForEntity(url, logData, String.class);
		} catch (Exception e) {
			logger.error("외부 서비스로 로그를 보내는 동안 에러가 발생했습니다.", e);
		}

	}
}
```