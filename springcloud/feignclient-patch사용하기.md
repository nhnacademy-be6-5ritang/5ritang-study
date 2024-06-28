

```java
 @PatchMapping("/auth/users/{userId}")
    public String updateUserAndCouponExpired(@PathVariable("userId") String userId, @ModelAttribute UserAndCouponRequestCreateDTO requestDTO) {
        UserAndCouponResponseDTO responseDTO = userAndCouponService.updateUserAndCoupon(userId, requestDTO);
        return "my-coupons";
    }
```
해당 메소드를 프론드단에서 적용하여 coupon api로 보내주는걸 목표로 했는데
작동이 되지않는다.

원인을 보아하니 feign은 patch를 지원하지 않는다고 함.
따라서 다음과같은 설정을 해주어야함

```java
@Configuration
class FeignOkHttpConfiguration {
    @Bean
    fun client(): OkHttpClient {
        return OkHttpClient()
    }
}
config 파일하나 만들어서 bean 등록해주고 

@FeignClient(name = "feign", configuration = [FeignOkHttpConfiguration::class])
적용할 feignclient interface에 위의 어노테이션을 달아주자. 어노테이션을 달떄 configuration 클래스 이름을 넣어주자
```


> ⚙️ **출처**: https://ystc1247.tistory.com/entry/FeignClient-%EC%82%AC%EC%9A%A9%EC%8B%9C-PATCH-method-%EA%B0%80-%EC%95%88%EB%90%98%EB%8A%94-%EC%98%A4%EB%A5%98