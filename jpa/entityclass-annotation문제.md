### 프로젝트 진행시 "entity class annotation을 어떻게 설정할까?"에 대한 고민

```java

@Getter
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "coupons_policies")
public class CouponPolicy {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "coupon_policy_id")
	private Long id;

	@NotNull
	@Column(name = "coupon_policy_min_order_price")
	private BigDecimal minOrderPrice;

	@Column(name = "coupon_policy_sale_price")
	private BigDecimal salePrice;

	@Column(name = "coupon_policy_sale_rate")
	private BigDecimal saleRate;

	@Column(name = "coupon_policy_max_sale_price")
	private BigDecimal maxSalePrice;

	@NotNull
	@Column(name = "coupon_policy_type", length = 10)
	private String type;

	@Builder
	public CouponPolicy(BigDecimal minOrderPrice, BigDecimal salePrice, BigDecimal saleRate, BigDecimal maxSalePrice,
		String type) {
		this.minOrderPrice = minOrderPrice;
		this.salePrice = salePrice;
		this.saleRate = saleRate;
		this.maxSalePrice = maxSalePrice;
		this.type = type;
	}

	public void update(BigDecimal minOrderPrice, BigDecimal salePrice, BigDecimal saleRate, BigDecimal maxSalePrice)
		 {
		this.minOrderPrice = minOrderPrice;
		this.salePrice = salePrice;
		this.saleRate = saleRate;
		this.maxSalePrice = maxSalePrice;
	}

}
```
보이는 것처럼 각 Entity class마다 적용하기로 함. 물론 어쩔 수 없이 변형해야한다면 관용을 주어서 허용함.

그렇다면 왜 해당 어노테이션을 기준으로 entity class를 적용할까에 대한 이유를 적어봄.

>### 1. @Getter
값을 가져올려면 무조건 필요함.
@Getter말고 @Setter를 넣지않은 이유는 객체의 값을 변형 할 우려가 있음.  또한 생성자를 만들어 @Builder를 줬기 때문에 생성하는데 문제가없음.
>### 2. @Entity
jpa에서 엔티티 매핑할려면 당연히 해야함.
>### 3. @NoArgsConstructor(access = AccessLevel.PROTECTED)
![jpa문서에 정의된 것](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkXMQS%2FbtrRY1FuCk1%2FGmLD6QDXbELhKUUE3HEhd0%2Fimg.png)
jpa 공식문서를 보면 다음과 같이 public or protected인 기본생성자가 있어야된다고함.
왜냐하면 jpa는 entity객체를 인스턴스화 하고 필드에 값을 채워넣을 때 reflection을 사용함.
따라서 런타임 시점에 동적으로 기본생성자를 통해 클래스를 인스턴스화 하여 값을 매핑하기 때문에 기본생성자가 필요함.
그렇다면 왜  AccessLevel.PROTECTED or AccessLevel.PUBLIC일까?
jpa가 entity와 연관관계를 갖는 다른 entity를 조회하는 방식이 두가지인데 즉시로딩, 지연로딩이 있음.
즉시로딩은 바로 조회하지만 지연로딩은 proxy객체로 조회함.
따라서 실제 proxy로 감아져있는 entity에 직접 접근할때 쿼리가 수행되어 실제값을 가져옴.
이런 원리 때문에 proxy로 감아져있는 entity를 만들때 해당 proxy 객체는 entity class를 상속 받아만들어진 객체이므로 entity를 생성하면 생성자의 AccessLevel이 Private일 수는 없고 AccessLevel.PROTECTED or AccessLevel.PUBLIC으로 설정하면됨.
>### 5. "왜 AccessLevel.PROTECTED일까?"
본인은 @Builder를 주고 객체생성을 처리하기 때문에 굳이 해당 기본생성자를 사용할일이 없음. 따라서 객체일관성 유지 차원에서 protected로 하는 것이 좋아보임.
>### 4. @Table(name = "coupons_policies")
테이블이름 매핑할려면 당연히 해야함.
>### 5. @Builder
생성자에 사용한 이유는 모든매개변수를 처리하지 않을 경우를 대비해서 적용함.
@Builder자체를 @NoArgsConstructor와 함께 클래스에 달았을경우 실행시 에러가발생함 왜냐하면 @Builder는 기본생성자가 있을 경우 전체생성자를 만들어 주지 않음. 따라서 @NoArgsConstructor제거하면 전체생성자를 만들어줘서 문제없이 @Builder를 사용가능함.
여기서 문제가 기존설정이 @NoArgsConstructor를 사용하는 룰인데 이걸 어길수는 없기에 객체의 생성자를 만들어 해당생성자에 @Builder를 달아주는 것으로 결정함.
```java
	@Builder
	public CouponPolicy(BigDecimal minOrderPrice, BigDecimal salePrice, BigDecimal saleRate, BigDecimal maxSalePrice,
		String type) {
		this.minOrderPrice = minOrderPrice;
		this.salePrice = salePrice;
		this.saleRate = saleRate;
		this.maxSalePrice = maxSalePrice;
		this.type = type;
	}
    private Long id의 경우 mysql에 의해 자동으로 증가되는 필드이기 때문에 안넣어도되고 나머지 필수 필드만 넣어줄 수 있음.
```




> ⚙️ **출처**: https://ystc1247.tistory.com/entry/FeignClient-%EC%82%AC%EC%9A%A9%EC%8B%9C-PATCH-method-%EA%B0%80-%EC%95%88%EB%90%98%EB%8A%94-%EC%98%A4%EB%A5%98

 


