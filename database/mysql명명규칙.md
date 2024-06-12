# 🌟 MySQL 명명 규칙 (Naming Convention)

MySQL에서 일관되고 가독성 높은 명명 규칙을 적용하기 위해 다음과 같은 규칙을 따릅니다.

---

## 📋 **일반 규칙**

- **소문자**를 사용합니다.
- **축약어 사용을 최소화**합니다.
- 축약어를 사용하는 경우에도 **소문자**를 사용합니다.

---

## 📦 **테이블**

- **스네이크 케이스**(`snake_case`)를 사용합니다.
- **복수형 명사**를 사용합니다.

### 🔗 **교차 테이블 (Many-to-Many 관계)**

- 적절한 명사가 없다면 **두 테이블을 and로 연결**합니다.
- 예시: `movies` 테이블과 `genres` 테이블 사이의 교차 테이블 ⇒ `movies_and_genres`

---

## 🔧 **컬럼**

- **스네이크 케이스**(`snake_case`)를 사용합니다.
- 특별한 이유가 없다면 **Auto Increment 속성의 대체키 (Surrogate Key)를 Primary Key로 사용**하고, 컬럼의 이름은 "테이블 이름의 단수형 + `_id`" 형식으로 합니다.
    - 예시: `movies` 테이블의 PK 컬럼: `movie_id`
- **참조 관계에서 자식 테이블의 Foreign Key 컬럼**은 부모 테이블에서 사용한 원래의 컬럼 이름을 그대로 사용합니다.

### 🔄 **예외 사항**
- **재귀 참조**인 경우
- **동일한 부모 테이블을 자식 테이블에서 2회 이상 참조**하는 경우

### 🚥 **특수 컬럼**
- **BIT 유형의 컬럼**은 “`is`” 또는 “`has`” 와 같은 접두사를 붙입니다.
    - 완료 여부: `is_completed`
    - 닉네임이 있는지 여부: `has_nickname`
- **날짜 컬럼**에는 “`date`” 접미사를 붙입니다.
    - 생성 일자: `create_date`
- **코드 컬럼**의 데이터 유형은 `tinyint UNSIGNED`를 사용합니다.
    - 운영자 상태: `operator_status`
- **컬럼 확장 주석**을 중첩 작성할 수 있습니다.

---

## 🗂️ **인덱스**

- “접두사”, “테이블 이름”, “컬럼 이름”의 순서로 **케밥 케이스**(`kebab-case`)를 사용하여 연결합니다.

### 📑 **접두사**
- **Unique Index**: `ux`
- **Full Text Index**: `fx`
- **Spatial Index**: `sx`
- **기타 Index**: `ix`

### 📊 **예시**
- `movies_and_genres` 테이블의 `genre_id` 컬럼에 만든 인덱스 ⇒ `ix-movies_and_genres-genre_id`

---

## 🔗 **참조키 제약 조건**

- “`fk`”, “자식 테이블 이름”, “부모 테이블 이름”의 순서로 **케밥 케이스**(`kebab-case`)를 사용하여 연결합니다.
- 예시: `order_products` 테이블이 `orders` 테이블을 참조하는 제약 조건 ⇒ `fk-order_products-orders`

### 💡 **예외**
- 하나의 부모 테이블을 두 번 이상 참조하는 경우 접미사로 자식 테이블의 참조 컬럼 이름을 추가합니다.
    - 예시: `fk-departments-employees-manager_employee_id`

---

## 👁️ **뷰**

- **스네이크 케이스**(`snake_case`)를 사용합니다.
- 접두사 “`v`”를 붙입니다.
- 이하의 규칙은 **“테이블”**과 동일합니다.

---

## 💻 **스토어드 프로시저**

- “`sp`”, “도메인”, “리포지토리”, “메서드”, “버전”의 순서로 **케밥 케이스**(`kebab-case`)를 사용하여 연결합니다.

### 🔍 **구성 요소**
- **도메인**: SP가 속한 서비스 도메인
    - 어드민: `ad`
    - B2B 서비스: `bb`
    - B2C 서비스: `bc`
- **리포지토리 (Repository)**
    - 단일 테이블: `operators`
    - 여러 테이블로 구성된 리포지토리: `operator_menu_permissions`
- **메서드 (Method)**
    - 생성: `c`
    - 단일 조회: `ro`
    - 전체 조회: `ra`
    - 페이지네이션 포함 전체 조회: `rp`
    - 수정: `u`
    - 삭제: `d`
- **버전**: “`v`” 뒤에 버전 번호를 붙입니다.

### 📘 **예시**
- `sp-ad-operator_menu_permissions-u-v2`
- `sp-bc-profiles-ro-v3`

---

## 🧩 **함수**

- “`f`”, “리턴 값 정의”의 순서로 **케밥 케이스**(`kebab-case`)를 사용하여 연결합니다.

### 🔧 **리턴 값 정의**
- **스네이크 케이스**를 사용합니다.
- 함수가 리턴하는 값을 정의합니다.
    - 예시:
        - `f-random_number`
        - `f-has_permission`

---

## ⚡ **트리거**

- “`tr`”, “테이블 이름”, “트리거 타임”, “트리거 이벤트”의 순서로 **케밥 케이스**(`kebab-case`)를 사용하여 연결합니다.

### ⏰ **트리거 타임 (Trigger Time)**
- `BEFORE`: `b`
- `AFTER`: `a`

### 📆 **트리거 이벤트 (Trigger Event)**
- `INSERT`: `i`
- `UPDATE`: `u`
- `DELETE`: `d`

### 💬 **예시**
- `tr-users-a-i`
    - `CREATE TRIGGER tr-users-a-i AFTER INSERT ON users`
- `tr-orders-b-iud`
    - `CREATE TRIGGER tr-orders-b-iud BEFORE INSERT, UPDATE, DELETE ON orders`

---

## 🗓️ **이벤트**

- “`ev`”, “반복 주기”, “작업 이름”의 순서로 **케밥 케이스**(`kebab-case`)를 사용하여 연결합니다.

### 📅 **반복 주기 (Interval)**
- `YEAR`: `y`
- `QUARTER`: `q`
- `MONTH`: `mo`
- `DAY`: `d`
- `HOUR`: `h`
- `MINUTE`: `m`
- `WEEK`: `w`

### 📝 **예시**
- `ev-d-log_partition_slide`
- `ev-m-revenue_settlement`

---

> ⚙️ **참고**: 위 규칙은 데이터베이스 일관성과 가독성을 높이기 위한 가이드라인입니다. 프로젝트에 맞춰 적절히 수정할 수 있습니다.

---

이처럼 규칙을 잘 지켜 데이터베이스를 설계하면, 일관성 있고 관리하기 쉬운 시스템을 구축할 수 있습니다. 🎉
