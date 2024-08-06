# 토스 페이먼츠 컨트롤러

이 문서에서는 토스 페이먼츠 ![img.png](img.png)결제 성공 처리, 결제 조회, 결제 취소와 관련된 컨트롤러 메서드를 설명합니다.

---

## 1. 결제 성공 처리 (`/success`)

```java
@GetMapping("/success")
public ModelAndView paymentSuccess(@RequestParam String orderId, @RequestParam String paymentKey, @RequestParam String amount) {
    String apiUrl = "https://api.tosspayments.com/v1/payments/confirm";
    String authToken = keyManagerService.getSecret("176afe7c8c07476cb319873dbb99af00");

    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    headers.set("Authorization", authToken);

    HttpEntity<PaymentConfirmationRequest> entity = new HttpEntity<>(PaymentConfirmationRequest.form(paymentKey, Integer.parseInt(amount), orderId), headers);
    String response = paymentRestTemplate.postForObject(apiUrl, entity, String.class);

    paymentServiceImpl.savePaymentResponse(response);
    ModelAndView view = new ModelAndView();
    view.setViewName("redirect:/api/orders/complete/" + orderId);
    return view;
}
```
- @GetMapping("/success"): 결제 성공 후 호출되는 엔드포인트입니다.
- @RequestParam: 클라이언트로부터 전달된 orderId, paymentKey, amount 파라미터를 받아옵니다.
- apiUrl: 결제 확인을 위한 API 엔드포인트입니다.
- authToken: 인증 토큰을 가져옵니다.
- HttpHeaders: 요청 헤더를 설정합니다.
- HttpEntity: HTTP 요청 본문을 설정합니다.
- paymentRestTemplate.postForObject: 결제 확인 요청을 보냅니다.
- paymentServiceImpl.savePaymentResponse: 결제 응답을 저장합니다.
- ModelAndView: 성공 시 주문 완료 페이지로 리다이렉트합니다.

---

## 2. 결제 조회 (/transactions/{order_info_id})

```java
@GetMapping("/transactions/{order_info_id}")
public ModelAndView paymentTransactions(@PathVariable("order_info_id") String orderInfoId) {
	GetOrderByInfoResponse order = orderServiceImpl.findByOrderInfoId(orderInfoId);
	if (order.price().equals(new BigDecimal("0.00"))) {
		ModelAndView modelAndView = new ModelAndView();
		modelAndView.addObject("paymentInfo", paymentServiceImpl.getPayment(orderInfoId));
		modelAndView.addObject("orderStatus", paymentServiceImpl.findByOrder(orderInfoId).status());
		modelAndView.setViewName("toss/transactions-public");
		return modelAndView;
	}
	String url = "https://api.tosspayments.com/v1/payments/orders/" + orderInfoId;

	HttpHeaders headers = new HttpHeaders();
	headers.set("Authorization", keyManagerService.getSecret("176afe7c8c07476cb319873dbb99af00"));

	HttpEntity<String> entity = new HttpEntity<>(headers);
	ResponseEntity<String> response = paymentRestTemplate.exchange(url, HttpMethod.GET, entity, String.class);
	String responseBody = response.getBody();

	ModelAndView modelAndView = new ModelAndView();
	modelAndView.addObject("paymentInfo", paymentServiceImpl.transactions(responseBody));
	modelAndView.addObject("orderStatus", paymentServiceImpl.findByOrder(orderInfoId).status());
	modelAndView.setViewName("toss/transactions");
	return modelAndView;
}
```
- @GetMapping("/transactions/{order_info_id}"): 특정 주문의 결제 조회를 위한 엔드포인트입니다.
- @PathVariable("order_info_id"): 경로 변수로부터 orderInfoId를 가져옵니다.
- orderServiceImpl.findByOrderInfoId: 주문 정보를 조회합니다.
- 결제 금액이 0일 경우: toss/transactions-public 뷰로 이동하여 결제 정보를 표시합니다.
- 결제 금액이 0이 아닐 경우: 결제 API에서 결제 정보를 조회하고 toss/transactions 뷰로 이동합니다.


---

## 3. 결제 취소 (/cancel/{order_info_id})

```java
@PostMapping("/cancel/{order_info_id}")
public ModelAndView paymentCancel(@PathVariable("order_info_id") String orderInfoId, @Valid @ModelAttribute
CancelTextRequest cancelTextRequest) {
	GetOrderByInfoResponse order = orderServiceImpl.findByOrderInfoId(orderInfoId);
	if (order.price().equals(new BigDecimal("0.00"))) {
		CancelResponse cancelResponse = paymentServiceImpl.paymentFindByOrderInfoId(orderInfoId);
		paymentServiceImpl.cancelPointSalePayment(cancelResponse.paymentId());
		ModelAndView modelAndView = new ModelAndView();
		modelAndView.setViewName("redirect:/api/payments/transactions/" + orderInfoId);
		return modelAndView;
	}
	CancelResponse cancelResponse = paymentServiceImpl.paymentFindByOrderInfoId(orderInfoId);

	String url = "https://api.tosspayments.com/v1/payments/" + cancelResponse.paymentKey() + "/cancel";

	HttpHeaders headers = new HttpHeaders();
	headers.set("Authorization", keyManagerService.getSecret("176afe7c8c07476cb319873dbb99af00"));
	headers.setContentType(MediaType.APPLICATION_JSON);

	// 요청 본문 데이터 생성
	String requestBody = String.format("{\"cancelReason\":\"%s\"}", cancelTextRequest.reason());

	HttpEntity<String> entity = new HttpEntity<>(requestBody, headers);
	ResponseEntity<String> response = paymentRestTemplate.exchange(url, HttpMethod.POST, entity, String.class);
	String responseBody = response.getBody();

	paymentServiceImpl.updatePayment(responseBody, cancelResponse.paymentId());
	ModelAndView modelAndView = new ModelAndView();
	modelAndView.addObject("paymentInfo", paymentServiceImpl.transactions(responseBody));
	modelAndView.setViewName("toss/transactions");
	return modelAndView;
}
```
- @PostMapping("/cancel/{order_info_id}"): 결제 취소를 위한 엔드포인트입니다.
- @PathVariable("order_info_id"): 경로 변수로부터 orderInfoId를 가져옵니다.
- @ModelAttribute CancelTextRequest cancelTextRequest: 취소 요청에 대한 데이터를 가져옵니다.
- 결제 금액이 0일 경우: 포인트 결제를 취소하고 결제 조회 페이지로 리다이렉트합니다.
- 결제 금액이 0이 아닐 경우
  - CancelResponse cancelResponse = paymentServiceImpl.paymentFindByOrderInfoId(orderInfoId);: 주문 정보를 조회합니다.
  - 결제 취소 요청: 결제 취소 API에 요청을 보내고 응답을 처리합니다.
  - 결제 정보 업데이트: 취소 응답을 저장하고 업데이트합니다.
  - 결제 정보 페이지: 결제 정보를 화면에 출력합니다.