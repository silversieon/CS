# 📌보안

### 1. 물리적 환경에 대한 보안

자연 재해처럼 DB에 물리적 손실을 발생시키는 위험으로부터 보호

### 2. 권한 관리를 통한 보안

접근이 허락된 사용자만 부여된 권한 내에서 DB를 사용할 수 있도록 함

- 계정이 발급된 사용자만 DB에 접근할 수 있도록 허용
- 사용자별로 DB의 사용 범위와 수행 가능한 작업 내용 제한

### 3. 운영 관리를 통한 보안

데이터 무결성 유지를 위한 올바른 제약조건 정의 및 사용자들이 정의된 제약조건을 위반하지 않도록 통제해야 한다

# 📌권한 관리

## ✅ 권한 관리란

접근 제어 : 계정이 발급된 사용자가 로그인에 성공했을 경우에만 접근이 가능하도록 함

1. 데이터베이스 관리자는 DB 전체의 보안 관리를 위해 모든 권한을 가지고 있다
2. 사용자별로 데이터베이스의 사용 범위와 수행 가능한 작업을 제한할 수 있음
3. 테이블, 뷰와 같은 모든 객체는 일반적으로 해당 객체를 생성한 사용자만 권한을 가지게 됨
4. 객체의 소유자 및 관리자는 소유한 객체에 대한 사용 권한을 부여할 수 있다

## ✅ 권한 부여 (GRANT)

> 객체의 소유자가 타 사용자에게 객체에 대한 사용 권한을 부여하기 위해 사용
>

```sql
grant select on `user` to Hong;//Hong에게 select 허용

grant insert, delete on `user` to public; //모든 사용자에게 허용

grant references on `user` to Kim;
//references 권한을 부여 받은 사용자는 해당 테이블의 기본키를
//외래키로 참조하는 자신의 테이블을 생성할 수 있다.

grant update(name, age) on `user` to Park;//두 속성 권한 부여(insert도 가능)

grant select on `user` to Lee WITH GRANT OPTION;
//부여받은 검색 권한을 다른 grant문을 통해 타 사용자에게 부여 가능
//Lee도 WITH GRANT OPTION 사용 가능
```

> 시스템 권한(데이터 정의어 관련) 권한 부여
>

```sql
grant create table to Song;

grant create view to Shin;
```

## ✅ 권한 취소 (REVOKE)

- Cascade : 연쇄적 권한 삭제(With Grant Option을 썼을 때)
- Restrict : 독립적 권한 삭제(아래의 경우 삭제 불가능 됨)

> 객체에 관한 권한 부여 취소
>

```sql
//WITH GRANT OPTION을 사용하여 Kim -> Park -> Lee 순으로 권한을 줬을 때

//아래 쿼리문은 Kim이 사용한다고 가정
revoke select on product from park cascade; //Lee에게 부여된 권한까지 연쇄 삭제

revoke select on product from park restrict; 
//Park이 Lee에게 부여한 권한으로 인해서 해당 쿼리문은 사용 안 됨
```

> 시스템 권한 부여 취소
>

```sql
revoke create table from hong;
```

## ✅역할 부여와 취소

여러 사용자에게 동일한 권한들을 부여하고 취소하는 번거로운 작업을 역할로 편리하게 함

```sql
//1. 새로운 역할 생성 (데이터베이스 관리자)
create role role_1;

//2. role_1 역할에 특정 테이블에 대한 권한을 부여 (해당 테이블 소유자)
grant select, insert, delete on product to role_1;

//3. 권한이 부여된 역할을 사용자에게 부여(데이터베이스 관리자)
grant role_1 to Hong, Kim;

//4. 사용자에게 부여한 역할 취소 (데이터베이스 관리자)
revoke role_1 from Hong;

//5. 역할 제거 (데이터베이스 관리자)
drop role role_1;
```