# 📌 Mybatis 개념과 동작원리

먼저 `mybatis`를 알기전에 `JDBC`를 알아보자
# 1. JDBC
`JDBC(Java Database Connectivity)`는 자바에서 데이터베이스에 접속하고 데이터를 저장하거나 업데이트하는 등의 작업을 수행하기 위한 `API`이다. 이 `API`는 데이터베이스와의 통신을 `추상화`하여 자바 어플리케이션에서 데이터베이스에 쉽게 접근할 수 있도록 한다. .

![jdbc1](https://github.com/princenim/TIL/assets/59499600/4681d94c-afef-45dd-a173-c67a53d232bc)

더 쉽게 말하면 세상에는 수많은 DB가 있고 모든 DB에 맞춰 코드를 구현하게 되면 DB에 맞춰서 코드를 수정해야한다. 하지만 `JDBC`는 모든 지원하는 데이터 베이스에 대해 일관된 인터페이스를 제공한다. 따라서 개발자는 데이터베이스를 변경하더라고 코드를 변경하지 않고 동일한 `JDBC 메소드`를 사용하여 작업을 수행할 수 있다. 따라서 데이터베이스 회사는 데이터베이스에대한 `JDBC 드라이버`를 제공하고, 개발자는 `JDBC 드라이버` 통해 데이터베이스와의 연결 및 쿼리실행과 같은 작업만 신경쓰면된다.

## 1.1 JDBC의 흐름

`JDBC`의 흐름은 다음과 같다.

![jdbc2](https://github.com/princenim/TIL/assets/59499600/c3aacf61-57d7-4d30-9386-29e859618193)

1.  `DriverManager`가 `JDBCDriver`를 로딩한다. 이 `Driver`는 `JDBC`가 제공하는 `인터페이스`로 각 DB마다 `인터페이스를` 구현하는 클래스를 제공한다. 예를들어 `Mysql`을 사용하는 경우 `Mysql`이 제공하는 `JDBC 드라이버`, `Oracle`이라면 `Oracle`에서 제공하는 드라이버를 사용해야한다.
2. `DriverManager`객체를 사용해 `Connection` 인스턴스를 얻는다.
3. `Connection` 인스턴스를 통해 `Statement`을 얻는다.
4. `Statement`를 통해 `ResultSet`을 얻는다.

하지만 이 `JDBC`는 1개의 클래스에 sql, db 연결, 자바언어가 모두 존재하기 때문에 재사용성이 좋지않다. 이런 단점을 해결하기 위해 `Mybatis`와 같은 프레임워크를 사용할 수 있다.

# 1. Mybatis

[mybatis – 마이바티스 3 | 시작하기](https://mybatis.org/mybatis-3/ko/getting-started.html)

`mybatis`는 `RDB`를 더욱 편하게 이용할 수있도록 개발된 `SQL Mapper`이다. 자바 객체와 sql 쿼리를 매핑하여 데이터베이스와의 상호작용을 편리하게 도와준다. `myabtis`를 사용하면 별도의 `xml 파일`에 쿼리를 사용하며, 작성된 `sql 쿼리`는 `mybatis`에서 제공하는 `Mapper 인터페이스`를 통해 자바 코드에서 호출 가능하다.

### ORM과 SQL Mapper의 차이점

`ORM`은 객체와 데이터베이스 간의 매핑을 담당한다. `ORM`은 데이터베이스의 테이블을 객체로 매핑하고, sql 쿼리 대신 객체 지향적인 방식으로 데이터를 조작한다. 주로 JPA가 대표적인 ORM 프레임워크이다.

`SQL Mapper`는 `SQL쿼리`와 데이터 베이스 간의 매핑을 담당한`다. SQL Mapper`는 개발자가 직접 쿼리를 작성하고, 이를 데이터 베이스와 연결한 결과를 매핑한다. 대표적으로 `mybatis`가 있다.

### Mybatis 특징

- sql 문과 자바 코드가 분리되어 있어 쉽게 유지보수가 가능하다.
- JDBC의 모든 기능을 Mybatis가 대부분 제공한다.

## 2.1 Mybatis 동작원리

![mybatis3](https://github.com/princenim/TIL/assets/59499600/c3490bcc-75e0-45c4-a9ab-46ed17ba3cd3)

1. 어플리케이션 시작시 `SqlSessionFactoryBuilder`가 설정파일을 참고해 `SqlSessionFactory`를 생성한다. 설정파일에는 `mybatis`의 구성정보를 담고있다.
2. `SqlSessionFactory`는 매 요청마다 `SqlSessio` 객체를 생성한다.
3. `SqlSession`을 사용해 매퍼 파일에 정의된 `sql 쿼리`를 호출하여 실행한다.
4. 쿼리의 실행결과는 자바 객체로 매핑되어 `SqlSession`를 통해 반환된다.
5. 데이터베이스와의 상호작용이 완료되면 `SqlSession` 을 닫아야한다. 그리고 커밋되지 않은 트랜잭션을 사용할 경우 명시적으로 롤백 이나 커밋을 수행해야한다.

그림에서`(1)~(3)`까지는 어플리케이션 시작시 한번 수행되며, `(4)~(10)`까지는 매 요청마다 수행된다.

## 2.2 Mybatis 설정

1. `gradle`로 의존성 설정

```groovy
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.3'
```

2. `mybatis` 설정하기 : `myabtis` 설정하는 방법은 두 가지이다.
   1. 자바 코드로 작성 : `mybatis`설정을 xml이 아닌 설정을 제공하는 Configuration 클래스를 사용할 수 있다.
   2. `mybatis-confing.xml` 작성:  `Mybatis-confing.xml` 는 `mybatis` 실행 시 필요한 설정 정보를 갖고 있는 파일이며 프로젝트의 리소스 폴더에 위치한다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--mybatis 설정파일 -->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <mappers>
        <!--MyBatis 매퍼 파일을 이 설정 파일에 등록하는 것.해당 매퍼파일에는 쿼리를 정의한다. -->
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
```


3. 설정 파일 위치 지정 : `mybatis-confing.xml` 파일을 사용하려면 해당 파일의 위치를 지정해줘야한다. `application.properties`에 위치를 지정한다.

```xml
# Mapper.xml 파일 위치 지정 
mybatis.mapper-locations=classpath:mapper/**/*.xml
```

4. `sql mapper` 작성하기 :  이 방법에는 2가지 방법이 있다.
   1.  인터페이스 기반 매퍼 작성 : sql 쿼리와 결과 매핑 설정을 자바 인터페이스에 작성한다. `mybatis` 어노테이션을 사용해 sql 쿼리를 정의한다. 
   2.  `XML` 기반 매퍼 작성 : `XML` 기반 매퍼는 sql 쿼리와 매핑 설정을 xml 파일에 작성한다. `매퍼명Mapper.xml` 과 같은 형식으로 작성한다. 그리고 보통 `src/main/resources/mapper` 디렉토리에 위치시킨다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.kboticketing.kboticketing.dao.UserMapper">
    <insert id="insertUser" parameterType="com.kboticketing.kboticketing.domain.User">
        INSERT INTO users (name, `email`, password, role, created_at)
        VALUES (#{name}, #{email}, #{password}, #{role}, #{createdAt})
    </insert>
</mapper>

```
 