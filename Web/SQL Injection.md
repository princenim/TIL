# 📌 SQL Injection
# 1. SQL Injection

`Sql injection`이란 악의적인 사용자가 보안상의 취약점을 이용해 임의의 `sql` 문을 주입하고 실행되게 하여 데이터베이스가 비정상적인 동작을 하도록 조작하는 행위를 말한다. 비교적 쉬운 공격으로 공격에 성공할 경우 큰 피해를 입힐 수 있는 공격이다.

## 1.1 공격 방법

### Error based SQL Injection

sql  공격 기법은 여러가지가 있는 논리적 에러를 이용한 sql inject이 가장 대중적인 공격방법이다.

```sql
select * from user where id = 'input1' and password = 'input2'
```

위의 쿼리문제서 `input1`에 `‘or 1 = 1 —`라는 sql 구문을 주입하는 것이다.

```sql
select * from user where id = ''or 1 = 1 —' and password = 'input2'
```

이렇게 바꾸면 user 테이블에 있는 모든 정보를 조회하게 됨으로 가장 먼저 만들어진 계정으로 로그인에 성공하게 된다. 보통 관리자 계정을 맨 처음 만들기 때문에 관리자 계정에 로그인할 수 있다.

### **Union based SQL Injection**

`union` 명령어를 이용한 `sql injection`이다. `union` 키워드는 두개의 쿼리문에 대한 결과를 통합해 하나의 테이블로 보여주게 하는 키워드이다. 만약 정상적인 쿼리문에 `union` 키워드를 사용하려 주입에 성공하면 원라는 쿼리문을 실행 할 수 있다.

```sql
//board라는 게시판에서 게시글을 검색하는 쿼리문
select * from board where title like '%input%' or contents '%input%'
```

이 input에 만약에 ‘union select null,id, password, from users—’ 를 넣는것이다.

```sql
select * from board where title 
like '%union select null,id, password, from users—%' 
and contents '%union select null,id, password, from users—%'
```

이렇게 하면 두 쿼리문이 합쳐져서 하나의 테이블로 보여지게 된다. 따라서 injection 이 성공하게 되면 사용자의 개인정보가 게시글과 함께 화면에 보여지게된다.

## 1.2 대응 방안

1. **입력 값에 대한 검증** : sql injection에 사용되는 기법과 키워드는 매우 많다. 따라서 이를 블랙리스트 기반으로 검증하게 되면 수많은 차단리스트를 등록해야하고 하나라도 빠지면 주입이 성공한다. 따라서 화이트리스트기반으로 검증을 해야한다.
2. **Prepared Statement 구문사용 :** Prepared Statement 구문을 사용하게 되면 사용자의 입력값이 db에 들어가기전에 dbms가 미리 컴파일하여 실행하지 않고 대기한다. 그 후 사용자의 입력값을 문자열로 인식하게 하여 공격 쿼리가 들어간다하더라도 입력은 단순 문자열이기때문에 공격자의 의도대로 쿼리가 작동하지 않는다.
3. **Error Message 노출 금지 :** 데이터 베이스 에러를 따로 처리해줘야한다. 만약에 에러를 그대로 반환하면 쿼리문이 노출될수 있개때문에 사용자에게 보여질 에러를 따로 만들어야한다.
4. **웹 방화벽** : 웹 방화벽을 사용하는것도 좋은 방법이다. 웹 방화벽은 소프트웨어 형, 하드웨어 형, 프록시 형 이렇게 3가지 종류로 나눌 수 있다. 소프트웨어는 서버 내에 직접 설치하는 것이고 , 하드웨어는 네트워크 상에서 서버 앞 단에 직접 하드웨어 장비로 구성하는 것.그리고 프록시 형은 DNS 서버 주소를 웹 방화벽으로 바꾸고 서버로 가는 트래픽이 웹 방화벽을 먼저 거치도록 하는 방법이다.