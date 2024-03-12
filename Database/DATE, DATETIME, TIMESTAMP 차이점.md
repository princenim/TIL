# 📌 DATE , DATETIME, TIMESTAMP 차이점

`mysql`에서 날짜를 표기할때 `DATE`, `DATETIME`, `TIMESTAMP`를 사용할 수있다. 그렇다면 어떤차이가 있을까?


|  | DATE                    | DATETIME                                  | TIMESTAMP           |
| --- |-------------------------|-------------------------------------------|---------------------|
| FORMAT | YYYY-MM-DD              | YYYY-MM-DD hh:mm:ss                       | YYYY-MM-DD hh:mm:ss |
| Supported | 1000-01-01 ~ 9999-12-31 | 1000-01-01 00:00:00~ 9999-12-31 23:59:59  | 1970~01-01 ~ 2038-01-19 (UTC)|
| 용량 | 3 bytes                 | 8 bytes                                   | 4 bytes             |


`DATE`는 년,월,일까지 표현가능하다. 하지만 `DATETIME`과 `TIMESTAMEP`은 년,월,일 시,분,초까지 표현할 수있다. 그렇다면 무슨 차이가 있을까?

결론부터 말하면 **TIMEZONE(시간대)의 적용 여부이다.**  `DATETIME`은 `timezone`을 저장하지 않고, `TIMESTAMP`는 `timezone`을 저장한다.

### TIMEZONE

타임존은 지구상의 특정 지역에 해당하는 표준 시간을 나타낸다.
- `UTC(협정 세계시)` : 세계 표준 시간이다. 다른 표준 시간대는 UTC로부터 얼마나 떨어져있는지를 나타낸다.
- `Asia/Seoul` : 한국 표준시이다. UTC+9 시간대에 해당한다.
- [TIMEZONE 목록](https://docs.oracle.com/middleware/12212/wcs/tag-ref/MISC/TimeZones.html)

## 1.1  DATETIME

`DATETIME`은 `timezone`(시간대)를 저장하지 않는다. 즉, 저장된 날짜와 시간은 서버의 시간대에 기반해 저장된다. 항상 `8byte`를 차지하며 `TIMESTAMP`보다 더 넓은 범위의 시간대를 저장할 수있다.

```sql
CREATE TABLE example_table (
    id INT PRIMARY KEY,
    event_time DATETIME 
);
```

## 1.2 TIMESTAMEP

`TIMESTAMP`는 `timezone`(시간대)를 갖는다. 더 정확히 말하면 `TIMESTMAP`는 데이터를 저장할때 내부적으로 `UTC(협정 세계시)`로 변환해 저장한다. 그리고  데이터를 조회할때 `time_zone`이라는 시스템 변수로 저장된 값을 기반으로해 해당 시간대로 변환해 출력한다. `4byte`의 크기를 차지한다.

```sql
CREATE TABLE example_table (
    id INT PRIMARY KEY,
    event_time TIMESTAMP 
);
```

위의 예시는 MYSQL 서버의 기본 시간대인 전역 설정인   `@@global.time_zone`에 따라 결정된다.

`timezone` 은 다음과 같이 확인가능하다.

```sql
# version 8.3.0
select @@global.time_zone;
```

## 1.3 무엇을 사용할까?

정리하면 `TIMESTAMP`가 더 작은 공간을 차지하고, 시간대 변환을 자동으로 처리하기때문에 현재 시간을 저장하는 용도라면 `TIMESTAMP`가 유리하고, 날짜 범위가 중요하다면 더 넓은 범위를 가진 `DATETIME`이 더 낫다.