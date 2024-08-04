### 프로젝트 막바지에 rest api를 어떻게 보여줄까 고민하다 swagger를 선택하게되었다.
사실 swagger 말고도 spring restdocs도 있다.
둘의 가장큰차이는 swagger는 의존성만주입하면 자동으로 api문서를 만들어주지만 spring restdocs은 테스트코드를 구현하고 그것을 통과한 controller만 문서로 만들어준다. 또한 ui같은 부분도 직접 만들어야하는 부분도 있어서 가장 간단하게 구현할려면 swagger를 쓰는 것이 좋다.
표로 둘의 장단점을 보여주자면 다음과 같다.




#### Swagger

| **장점**                                                                                             | **단점**                                                                                           |
|------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| API 문서가 자동으로 생성됩니다.                                                                       | 프로덕션 코드에 문서화 관련 코드가 포함됩니다.                                                      |
| 어노테이션(annotation)을 통해 문서가 생성되기 때문에 API 현행화가 쉽습니다.                              | 서버 실행 시 문서가 생성되므로 API 스펙만 별도로 관리하기 어렵습니다.                                |
| 화면에서 API를 직접 호출하여 테스트해볼 수 있습니다.                                                   | 검증되지 않은 API가 생성될 수 있습니다.                                                             |

#### Spring REST Docs

| **장점**                                                                                             | **단점**                                                                                           |
|------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| 테스트 코드 통과 후 API 문서가 생성되므로 신뢰할 수 있습니다.                                           | 문서가 추가될 때마다 AsciiDoc 문서를 일일이 편집해야 합니다.                                           |
|                                             | 테스트코드를 구현해야 만들 수 있습니다. |
| 커스터마이징이 자유롭습니다.                                                                         | 문서 커스터마이징의 자유도가 있지만, 설정과 관리가 복잡할 수 있습니다.            

---
### 구현 목표

1. front server에서 쿠폰 api 버튼을 누르면 front server에 있는 controller에서 feignclient를 호출함.
2. couponserver의 swagger api controller가 실행되어 실시간으로 쿠폰 api json파일을 전송해줌.
3. 그것을 frontserver coupon-api.html 에서 자동으로 렌더링함.


---

### 구현 과정

1. 쿠폰서버에 해당 의존성을 주입해줌.
   ![](https://velog.velcdn.com/images/lkh/post/840932e7-778c-413e-ab90-bbe33f69ee80/image.png)


2. 쿠폰서버 application.yaml에 다음과같이 경로를 설정해주자. 동적으로 구현될때 필요한것은 api-docs.path이기때문에 이것경로를 기억해주자.

```
springdoc:
  api-docs:
    path: /coupon-api   # json 형식으로 응답하는 경로
  default-consumes-media-type: application/json
  default-produces-media-type: application/json
  swagger-ui:
    operations-sorter: alpha
    tags-sorter: alpha
    path: /coupon-api.html # html형식으로 응답하는 경로
    disable-swagger-default-url: true
```

3. 쿠폰서버에 swagger config를 설정해줌.
```java
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info().title("Coupon API").version("1.0").description("쿠폰 API 명세서")); // api버전, 이름, 상세설명 설정
    }

    @Bean
    public GroupedOpenApi api() {
        String[] paths = {"/coupons/**"}; // 어디까지 경로로 둘지 설정
        String[] packagesToScan = {"com.nhnacademy.bookstorecoupon"}; // 어디부터 스캔할지 경로 설정
        return GroupedOpenApi.builder()
                .group("coupon-api")
                .pathsToMatch(paths)
                .packagesToScan(packagesToScan)
                .build();
    }
}



```


4. controller에 swagger annotation을 달아줌.
```java

/**
 * @author 이기훈
 * 쿠폰정책관련 기능을 수행하는 컨트롤러입니다.
 */
@Tag(name = "CouponPolicy", description = "쿠폰 정책관련 API") // 해당 컨트롤러에 대한 설명을 보여주는 기능
@RestController
@RequestMapping("/coupons/policies")
public class CouponPolicyController {
	private final CouponPolicyService couponPolicyService;

	public CouponPolicyController(CouponPolicyService couponPolicyService) {
		this.couponPolicyService = couponPolicyService;
	}



	/**
	 * 웰컴쿠폰 정책을 생성하는 컨트롤러
	 * @param couponPolicyRequestDTO 쿠폰정책 생성 dto
	 */
	@Operation( // 해당 컨트롤러 메소드에 대한 설명을 추가하는 기능
		summary = "웰컴쿠폰정책 생성",
		description = "웰컴쿠폰정책을 생성합니다"
	)
	@ApiResponses(value = { // 응답에 대한 설명을 추가하는 기능
		@ApiResponse(responseCode = "201", description = "웰컴쿠폰정책이 성공적으로 발행되었습니다."),
		@ApiResponse(responseCode = "400", description = "잘못된 요청입니다.")
	})
	@AuthorizeRole({"COUPON_ADMIN", "HEAD_ADMIN"})
	@PostMapping("/welcome")
	public ResponseEntity<Void> issueWelcomeCoupon(
		@Parameter(description = "웰컴쿠폰정책 발행 요청 데이터", required = true) @Valid @RequestBody CouponPolicyRequestDTO couponPolicyRequestDTO) {
		validateSaleFields(couponPolicyRequestDTO);
		couponPolicyService.issueWelcomeCoupon(couponPolicyRequestDTO);
		return ResponseEntity.status(HttpStatus.CREATED).build();
	}
```
여기까지는 아주 기본적인 것만 달아보았고
dto에 직접 예시값을 주어 swagger api로 보았을때 사용자의 이해도를 높히는 방식도 있다.
이는 추후에 팀원들과 상의하여 업데이트 해볼 계획이다.


여기까지만 하면 서버를 구동시키고 url에 yaml에 설정했던 경로로 호출하면 자동으로 swagger문서가 생성된다.
사실 이건 기본적인것이고
우리팀은 frontserver에서 feignclient로 호출해서 렌더링까지 시키는것이 목표이기때문에 추가적으로 더 적어보고자 한다.

5. frontserver에서 다음과 같이 설정을 해주자

```java

@FeignClient(name = "swaggerClient", url = "http://localhost:8090")
public interface SwaggerApiClient {
	@GetMapping("/coupons/api")// feignclient로 쿠폰서버에 있는 sawgger json을 불러오는 것을 구현해주어야한다. 
	ResponseEntity<String> getSwaggerJson();

	@GetMapping("/api")
	ResponseEntity<String> getBackSwaggerJson();
} 



@Controller
public class SwaggerController {

	private final SwaggerApiClient swaggerApiClient;

	public SwaggerController(SwaggerApiClient swaggerApiClient) {
		this.swaggerApiClient = swaggerApiClient;
	}

// front server에서 swaggercontroller를 호출하여 feignclient로 couponserver의 json을 받아온다. 
// 그리고 그것을 coupon-api.html로 보내주고 렌더링 시켜주면 된다.
	@GetMapping("/coupons/api")
	public String getCouponApi(Model model) {
		try {
			ResponseEntity<String> response = swaggerApiClient.getSwaggerJson();
			String swaggerJson = response.getBody() != null ? response.getBody() : "{}";
			model.addAttribute("swaggerJson", swaggerJson);
		} catch (Exception e) {
			model.addAttribute("swaggerJson", "{}");
		}
		return "api/coupon-api";
	}

	@GetMapping("/api")
	public String getBackApi(Model model) {
		try {
			ResponseEntity<String> response = swaggerApiClient.getBackSwaggerJson();
			String swaggerJson = response.getBody() != null ? response.getBody() : "{}";
			model.addAttribute("swaggerJson", swaggerJson);
		} catch (Exception e) {
			model.addAttribute("swaggerJson", "{}");
		}
		return "api/back-api";
	}
}
```

6. swagger json객체를 coupon-api.html로 가져와서 렌더링하기
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Swagger API Documentation</title>
    <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/4.12.0/swagger-ui.css">
    <style>
        html, body {
            margin: 0;
            padding: 0;
            height: 100%;
            overflow: hidden;
        }
        #swagger-ui {
            height: 100%;
            overflow-y: scroll;
        }
    </style> <!-- 해당 스타일은 커스텀이 가능하다, 본인은 swagger ui js로 스크롤바가 생기지 않아서 추가해줬음-->
</head>
<body>
<div id="swagger-ui"></div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/4.12.0/swagger-ui-bundle.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/4.12.0/swagger-ui-standalone-preset.js"></script>
<script th:inline="javascript">
    document.addEventListener('DOMContentLoaded', function () {
        // Thymeleaf 변수를 JavaScript 변수로 변환함. 그리고 swagger-ui js 파일들을 이용하여 ui를 렌더링해준다.
        const swaggerJson = /*[[${swaggerJson}]]*/ '{}';

        SwaggerUIBundle({
            spec: JSON.parse(swaggerJson),
            dom_id: '#swagger-ui',
            presets: [
                SwaggerUIBundle.presets.apis,
                SwaggerUIStandalonePreset
            ],
            layout: "StandaloneLayout"
        });
    });
</script>
</body>
</html>
```

7. 호출할 버튼 만들기

```html
      <tr>
                    <td>
                        <a th:href="@{/coupons/api}"><span>쿠폰 API</span></a>
                    </td>
                </tr> <!-- 다음과같이 호출버튼을 만들어주면 동적으로 coupon-api.html이 보여지게 된다-->
```

---
### 구현결과

![](https://velog.velcdn.com/images/lkh/post/89958c1b-e9a5-42f0-8d92-76cbc1fb0045/image.png)



---

### 느낀점
개인적으로 설정해줘야할게 너무많다는 생각이 든다. 단순히 tag, operation만하면 문제가되진 않는데 example까지 만들고 할려면 프로젝트 초창기때부터 controller단이 해결되면 swagger도 빨리 진행해줘야하지 않을까 싶다.



---
### 참조
https://sm-studymemo.tistory.com/139