# @PathVariable / DTO

다음 addUserRole()을 호출하는 메서드(컨트롤러)는 Path Variable를 인자로 받아야할까, DTO를 인자로 받아야할까?

userId, roleId는 각각 Users, Roles 테이블의 기본키로 URI에 노출이 되어서는 안 된다고 생각했다.

```java
package com.nhnacademy.bookstoreback.userrole.domain.service;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.nhnacademy.bookstoreback.role.domain.entity.Role;
import com.nhnacademy.bookstoreback.role.exception.RoleNotFoundException;
import com.nhnacademy.bookstoreback.role.repository.RoleRepository;
import com.nhnacademy.bookstoreback.user.domain.entity.User;
import com.nhnacademy.bookstoreback.user.exception.UserNotFoundException;
import com.nhnacademy.bookstoreback.user.repository.UserRepository;
import com.nhnacademy.bookstoreback.userrole.domain.dto.response.AddUserRoleResponse;
import com.nhnacademy.bookstoreback.userrole.domain.dto.response.GetUserRoleResponse;
import com.nhnacademy.bookstoreback.userrole.domain.entity.UserRole;
import com.nhnacademy.bookstoreback.userrole.domain.repository.UserRoleRepository;

import lombok.RequiredArgsConstructor;

@Service
@RequiredArgsConstructor
@Transactional
public class UserRoleService {
	private final UserRoleRepository userRoleRepository;
	private final UserRepository userRepository;
	private final RoleRepository roleRepository;

	public AddUserRoleResponse addUserRole(Long userId, Long roleId) {
		User user = userRepository.findById(userId)
			.orElseThrow(() -> new UserNotFoundException(userId));

		Role role = roleRepository.findById(roleId)
			.orElseThrow(() -> new RoleNotFoundException(roleId));

		UserRole userRole = UserRole.builder()
			.user(user)
			.role(role)
			.build();
		UserRole savedUserRole = userRoleRepository.save(userRole);

		return AddUserRoleResponse.fromEntity(savedUserRole);
	}

	public GetUserRoleResponse getUserRoles(Long)
}

```

`@PathVariable`과 DTO(Data Transfer Object)는 서로 다른 용도로 사용됩니다. 이 둘의 사용 시점과 목적을 이해하는 것이 중요합니다. 아래에서 각각의 용도와 언제 사용하는 것이 좋은지 설명하겠습니다.

### @PathVariable

`@PathVariable`은 URL 경로에 포함된 값을 메서드 인수로 전달할 때 사용됩니다. 이는 주로 특정 리소스에 접근할 때 유용합니다. 예를 들어, 리소스의 ID를 URL 경로에 포함시켜 리소스를 식별하고 접근하는 경우입니다.

### 사용 시점

- **특정 리소스를 식별**: 리소스의 고유 식별자가 URL에 포함되어야 할 때.
- **간단한 데이터**: 경로 변수는 일반적으로 간단한 데이터를 전달하는 데 사용됩니다 (예: ID, 이름).

### 예시

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{userId}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long userId) {
        // userId를 사용하여 사용자 정보를 조회합니다.
        UserResponse userResponse = userService.getUserById(userId);
        return ResponseEntity.ok(userResponse);
    }
}

```

### DTO (Data Transfer Object)

DTO는 계층 간 데이터 전송을 위한 객체입니다. 주로 클라이언트에서 서버로, 서버에서 클라이언트로 데이터를 전송할 때 사용됩니다. DTO는 입력과 출력 데이터를 캡슐화하고, 유효성 검사를 수행하며, API 계약을 명확히 하는 데 도움을 줍니다.

### 사용 시점

- **복잡한 데이터 구조**: 요청 본문이나 응답 본문에 복잡한 데이터를 전송해야 할 때.
- **입력 유효성 검사**: 요청 데이터의 유효성을 검사해야 할 때.
- **API 계약 명확화**: API의 입력과 출력을 명확히 정의하고 문서화해야 할 때.

### 예시

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @PostMapping
    public ResponseEntity<UserResponse> createUser(@RequestBody @Valid CreateUserRequest createUserRequest) {
        // createUserRequest를 사용하여 새로운 사용자를 생성합니다.
        UserResponse userResponse = userService.createUser(createUserRequest);
        return ResponseEntity.status(HttpStatus.CREATED).body(userResponse);
    }
}

```

### 요약

- **@PathVariable**: 간단하고 고유한 데이터를 URL 경로에 포함시켜 리소스를 식별할 때 사용합니다. 예를 들어, 리소스의 ID를 URL에 포함시켜 특정 리소스에 접근하는 경우입니다.
- **DTO**: 복잡한 데이터를 전송하거나, 입력 유효성 검사를 수행하거나, API 계약을 명확히 할 때 사용합니다. 주로 요청 본문이나 응답 본문에 사용됩니다.

### 사용 사례 정리

1. **특정 리소스 조회** (간단한 경로 변수 사용):

    ```java
    @GetMapping("/{userId}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long userId) {
        // ...
    }
    
    ```

2. **데이터 생성/수정** (복잡한 데이터 구조와 유효성 검사 필요):

    ```java
    @PostMapping
    public ResponseEntity<UserResponse> createUser(@RequestBody @Valid CreateUserRequest createUserRequest) {
        // ...
    }
    
    ```


이렇게 하면 `@PathVariable`과 DTO를 각각의 용도에 맞게 사용할 수 있습니다.