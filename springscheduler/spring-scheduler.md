


### 1. 프로젝트 요구사항중에 쿠폰을 만료시키는부분을 해결하기위해 스프링스케줄러를 사용하였다.
스프링스케쥴러는 @Scheduled 어노테이션만 사용하면 되기에 아주 간단해보여서 사용.


---
### 2. fixedRate, fixedDelay, cron을 이용하여 스프링스케줄러 주기설정
스프링스케줄러상 fixedRate, fixedDelay, cron와같은 설정을 할 수있다.
각 설정마다 차이가 있으니 사용에 유의해야한다. 어떤차이가 있는지 알아보자.




#### - fixedRate
fixedRate는 현재 Schedule 상에 걸린 작업의 완료 여부와 상관 없이 Scheduler가 시작한 시간으로부터 카운팅되는 형태이다.
테스트용 코드를 살펴보자.
```java

	@Scheduled(fixedRate = 1000)
	public void fixedRateJob() throws InterruptedException {
		log.info("{} fixedRatedJob", Thread.currentThread().getName());
		Thread.sleep(500);
	}




	@Scheduled( fixedRate = 1000)
	public void fixedRateJobSleep() throws InterruptedException {
		log.info("{} fixedRateJobSleep started.", Thread.currentThread().getName());
		Thread.sleep(2500);
		log.info("fixedRateJobSleep is finished.");
	}
```
fixedRateJob을 실행해보면 다음과같다.
![](https://velog.velcdn.com/images/lkh/post/ea97786c-50eb-4606-b953-aae0a6120385/image.png)
해당메소드가 실행되면 0.5초간 sleep하는데 이 sleep 하는 시간을 포함해서 1초간격으로 실행되는 것을 알 수 있다.

그렇다면 sleep하는시간을 주기시간보다 길게잡으면 어떻게될까?
fixedRateJobSleep을 실행해보자
![](https://velog.velcdn.com/images/lkh/post/572eb243-4da2-4cd3-b56d-772b605a69a3/image.png)
주기간격은 1초지만 sleep 시간이 2.5초라 sleep시간에 맞춰서 2.5초 간격으로 실행되는 것을 알 수있다.

따라서 fixedRate를 사용할때는 주기시간이 아니라 해당 메소드의 실행시간을 고려해서 사용하자

추가적으로 만약 fixedRateJob, fixedRateJobSleep 둘다 실행하면 어떻게 될까?
![](https://velog.velcdn.com/images/lkh/post/4fb40be7-8e6d-4fa4-8e4a-38c590a50017/image.png)
흥미로운 부분은 스프링스케줄러가 싱글스레드 기반이라 하나의 스레드(scheduling-1)로 동기화되어서 실행된다는 점이다.


#### - fixedDelay
fixedDelay는 현재 Schedule 상에 걸린 작업을 모두 끝난 이후에 설정된 시간이 카운팅되는 형태이다.
테스트용 코드를 살펴보자.


```java

	@Scheduled(fixedDelay = 1000)
	public void fixedDelayJob() throws InterruptedException {
		log.info("{} fixedDelayJob", Thread.currentThread().getName());
		Thread.sleep(500);
	}


	@Scheduled( fixedDelay = 1000)
	public void fixedDelayJobSleep() throws InterruptedException {
		log.info("{} fixedDelayJobSleep started.", Thread.currentThread().getName());
		Thread.sleep(2500);
		log.info("fixedDelayJobSleep is finished.");
	}
    
 ```
fixedDelayJob을 실행해보면 다음과 같다.
![](https://velog.velcdn.com/images/lkh/post/c1c5bcdb-a44e-4651-b55e-2e664bfec333/image.png)

fixedRateJob과 다르게 fixedDelayJob메소드가 실행후(sleep 0.5초)에 1초뒤에 실행되는것을 알 수있다.
따라서 fixedRateJob과은 총 주기가 1초지만 fixedDelayJob메소드는 총주기가 1.5초인 것이다.


그렇다면 sleep하는시간을 주기시간보다 길게잡으면 어떻게될까?
fixedRateJobSleep과는 다르게  메소드가 끝나고 1초후에 실행되는 것을 알 수있다.
즉 sleep 시간 2.5초 + 주기시간 1초 = 총 3.5초의 사이클인 것이다.

![](https://velog.velcdn.com/images/lkh/post/0339bf0d-1888-4565-888c-2892bae41c80/image.png)



#### - cron
크론(cron)은 유닉스 계열의 잡 스케줄러다. 크론 표현식을 구현해서 스케줄링의 디테일을 보장할 수 있다.
필드는 총 7개이며, 연도는 생략이 가능 따라서 주로 6개로 사용함.
![](https://velog.velcdn.com/images/lkh/post/856e6ea7-ca20-4f89-be2e-77a01e433704/image.png)


cron에서 사용하는 특수문자는 다음과 같다.
○  * : 모든 값을 뜻함.

○  ? : 특정한 값이 없음을 뜻.

○  - : 범위를 뜻함.

○  , : 특별한 값일 때만 동작

○  / : 시작시간 / 단위

○  L : 일에서 사용하면 마지막 일, 요일에서는 마지막 요일(토요일)

○  W : 가장 가까운 평일

○  # : 몇째 주의 무슨 요일을 표현






 ```java
 
	@Scheduled(cron = "*/1 * * * * *") // 매초 실행
	public void runEverySecond() throws InterruptedException {
		log.info("{} JobCron started.", Thread.currentThread().getName());
		Thread.sleep(100); // 실제 작업 내용
		log.info("{} JobCron finished.", Thread.currentThread().getName());
	} // 1초간격으로 실행 0.1초 sleep
  ```
첫번째 테스트를 보자 주기시간은 1초고 sleep은 0.1초를 주었다.
sleep 시간이 주기시간보다 작으므로 1초간격으로 잘 실행되는 것을 알 수 있다.

![](https://velog.velcdn.com/images/lkh/post/6f61237b-9438-499b-868a-8ef1d38b2edc/image.png)


두번째 테스트를 보자 주기시간은 1초고 sleep은 1.2초를 주었다.
sleep 시간이 주기시간보다 크므로 sleep이 끝나는동안은 다음주기를 실행시킬 수가 없다. 따라서 해당주기는 skip하고
1초간격이 아니라 2초간격으로 실행되는것을 알 수 있다.
이와같은 경우를 fixedRate에 대입할 경우 주기시간이 sleep보다 작기 때문에 sleep시간이 주기시간이 되어서 동기화처리되어 순서대로 실행된다.
cron은 해당주기시간이 sleep시간보다 작으면 주기간격이 sleep에 맞춰지는 것이 아니라 해당 메소드 실행을 스킵하고 다음 주기로 넘어간다.

```java

	
	@Scheduled(cron = "*/1 * * * * *") // 매초 실행
	public void runEverySecond() throws InterruptedException {
		log.info("{} JobCron started.", Thread.currentThread().getName());
		Thread.sleep(1200); // 실제 작업 내용
		log.info("{} JobCron finished.", Thread.currentThread().getName());
	}
```


![](https://velog.velcdn.com/images/lkh/post/2bbd1831-1012-4ce5-b5cc-1b5dc4e03008/image.png)




---
### 3. 구현
```java
	// 쿠폰 만료처리 스케줄링   매일 새벽 2시 시작 설정
	@Override
	@Scheduled(cron = "0 0 2 * * *")
	public void findExpiredCoupons() {
		log.warn("기한만료 쿠폰 체크로직 발동");
		LocalDateTime now = LocalDateTime.now();

		// 오늘 자정 이후에 만료된 쿠폰들을 조회하여 처리
		List<UserAndCoupon> expiredCoupons = userAndCouponRepository.findByExpiredDateBeforeAndIsUsedIsFalse(now);

		for (UserAndCoupon coupon : expiredCoupons) {
			coupon.update(coupon.getExpiredDate(), true);
		}

		log.warn("기한만료 쿠폰 체크로직 실행완료");
	}

```
정말 간단하게 어노테이션만 달아주면 구현가능하다.


---
### 4. 문제점
구현한 로직에서 만약 회원이 1000명이라고 가정하고 500번대에서 만료쿠폰처리를 하다가 문제가 발생하여 스케줄링이 멈춰버리면 어떻게 대응할까에 대한 고민을 하게되었다.

---
### 5. 해결방안

우리는 spring batch를 이용하여 batch서버를 만들어서 로깅, 스케쥴링과 같은 부분들은 따로 빼서 관리해보는 것을 고려해보고 있었다. 하지만 스케쥴링을 하는데 굳이 배치서버까지 또 만드는건 버거운 일이었고 기존 스케줄러 로직에서 예외처리를 하고 카운트를 줘서 원하는 횟수만큼 스케쥴링 로직을 실행할 수 있게끔 하였다.


```java

private static final int MAX_RETRY_ATTEMPTS = 3;
	private static final int RETRY_DELAY_MS = 5000; // 5초 대기

	// 쿠폰 만료처리 스케줄링   매일 새벽 2시 시작 설정
	@Scheduled(cron = "0 0 2 * * *")
	public void findExpiredCouponsScheduler() {
		log.warn("기한만료 쿠폰 스케쥴러 시작");

		try {
			// 실제 로직 수행
			findExpiredCoupons();
		} catch (Exception e) {
			log.error("기한만료 쿠폰 체크로직 실행 중 오류 발생", e);
			// 재시도 로직 호출
			retryLogic(1);
		}


	}

	@Override
	public void findExpiredCoupons() {

		log.warn("기한만료 쿠폰 체크로직 발동");
		LocalDateTime now = LocalDateTime.now();

		// 오늘 자정 이후에 만료된 쿠폰들을 조회하여 처리
		List<UserAndCoupon> expiredCoupons = userAndCouponRepository.findByExpiredDateBeforeAndIsUsedIsFalse(now);

		for (UserAndCoupon coupon : expiredCoupons) {
			coupon.update(coupon.getExpiredDate(), true);
		}
		log.warn("기한만료 쿠폰 체크로직 실행완료");
	}

	private void retryLogic(int attempt) {
		if (attempt > MAX_RETRY_ATTEMPTS) {
			log.error("최대 재시도 횟수 초과");
			// 재시도 실패 알림 전송 또는 다른 처리
			return;
		}

		try {
			Thread.sleep(RETRY_DELAY_MS); // 대기 시간
			log.warn("재시도 - 시도 " + attempt);

			new Thread(() -> {
				try {
					findExpiredCoupons();
				} catch (Exception e) {
					log.error("재시도 중 오류 발생", e);
					retryLogic(attempt + 1); // 다음 시도
				}
			}).start();
		} catch (InterruptedException e) {
			Thread.currentThread().interrupt();
			log.error("재시도 중 오류 발생", e);
		}
	}

```


### 6. 참조
https://seodeveloper.tistory.com/entry/Spring-Batch-%ED%99%98%EC%9C%A8-%EC%A0%95%EB%B3%B4-API-%EB%A5%BC-%EA%B0%84%EB%8B%A8%ED%95%9C-%EB%B0%B0%EC%B9%98-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%9F%AC-%EC%B6%94%EA%B0%80-%EC%98%88%EC%A0%9C%ED%8E%B8

https://spring.io/projects/spring-batch