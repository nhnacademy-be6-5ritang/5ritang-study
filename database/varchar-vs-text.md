# 🌟 varchar vs text



---

## 📋 **varchar**

- 테이블 생성시 한행당 65535byte까지만 생성가능함. 따라서 테이블생성시 varchar를 여러개 쓸때 최대바이트수를 계산하고 적용해야함.
- MySQL 서버는 레코드의 컬럼 데이터는 B-Tree (Clustering Index)에 저장(이를 Inline 저장이라고 함).
- 하지만 MySQL 서버는 varchar 타입의 컬럼을 항상 B-Tree로 저장하지는 않고, 길이가 길어서 저장 공간이 많이 필요한 경우에는 Off-Page로 저장
- varchar의 경우 메모리공간(records[2])이 TABLE 구조체의해 정의되고 TABLE 구조체는 Mysql 서버내에 캐싱되어서 여러커넥션에서 공유해서 사용됨.
- 주의해야할것이 varchar에 저장된값이 길어 off-page로 저장될 경우 새롭게 메모리를 할당함. 따라서 varchar에 매우큰 값이 저장되는경우 주의할 것.
- VARCHAR는 전체 문자열에 대한 인덱스를 생성할 수 있음. 더 정확히는 특정 길이 내에서는 전체 문자열에 대해 생성할 수 있음.
```mysql
CREATE TABLE tb_varchar (
id INT PRIMARY KEY,
fd VARCHAR(100)
);

alter table tb_varchar add index ix_fd(fd); 생성 가능! 
```

---

## 📋 **text**

- clob타입 (TEXT(또는 CLOB)나 BLOB와 같은 대용량 데이터를 저장하는 컬럼 타입을 LOB(Large Object) 타입.)
- RDBMS 서는 LOB 데이터를 Off-Page 라고 하는 외부 공간에 저장.
- 용량이 큰 LOB 데이터는 B-Tree 외부의 Off-Page 페이지(MySQL 서버 메뉴얼에서는 External off-page storage)로 저장
- 하지만 MySQL 서버는 LOB 타입의 컬럼을 항상 Off-Page로 저장하지는 않고, 길이가 길어서 저장 공간이 많이 필요한 경우에만 Off-Page로 저장
- VARCHAR 대신 TEXT 타입을 사용하면 varchar에서 문제가 되던 길이 제한이 싹 사라짐.
- varchar와 달리 text 와 같은 lob타입은 메모리공간(records[2])이 따로 할당되지 않는다. 따라서 매번 레코드를 읽고 쓸때마다 필요한만큼 메모리 할당이 필요함.
- TEXT 데이터 타입은 기본적으로 전체 문자열을 기준으로 인덱스를 생성할 수 없고, 반드시 일정 길이를 명시적으로 지정하여야만 인덱스를 생성할 수 있음.
``` mysql
create table tb_text (
id int primary key,
tx text
);

>alter table tb_text add index  ix_tx(tx(50)); 생성가능 
``` 




---

## 📋 **둘 중 뭘 쓸 까 ?**



###  컬럼 타입 선정 규칙
>VARCHAR
> - 최대 길이가 (상대적으로) 크지 않은 경우
> - 테이블 데이터를 읽을 때 항상 해당 컬럼이 필요한 경우
> - DBMS 서버의 메모리가 (상대적으로) 충분한 경우

>TEXT
> - 최대 길이가 (상대적으로) 큰 경우
> - 테이블에 길이가 긴 문자열 타입 컬럼이 많이 필요한 경우
> - 테이블 데이터를 읽을 때 해당 컬럼이 자주 필요치 않은 경우

---

> ⚙️ **출처**: https://medium.com/daangn/varchar-vs-text-230a718a22a1
>
> ⚙️ **출처**: https://dkswnkk.tistory.com/714



