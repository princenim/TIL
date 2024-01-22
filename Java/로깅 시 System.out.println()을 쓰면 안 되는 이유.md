
# 📌 로깅 시 System.out.println()을 쓰면 안 되는 이유와 로깅 프레임워크

로깅할때 `system.out.println()` 를 사용하지말라고 한다. 그 이유가 무엇일까?
# 1. System.out.println()

```java
System.out.println("hello world");
```

먼저 `System.out.println()`은 자바에서 화면을 출력하는데 사용되는 구문이다. 단순 출력으로 편리하지만지만 확실한 단점이 존재한다.

1. **가독성이 떨어진다.** : `System.out.println()` 을 사용하면 코드에서 직접적으로 출력을 담당해 코드가 길어지면 가독성이 떨어질 수 있다. 따라서 여러곳에 출력하는 부분이 존재할때 유지보수가 어려워질 가능성이 존재한다.

2. **휘발된다.** : `System.out.println()` 으로 출력된 내용은   파일로 저장되지 않고 바로 사라지기 때문에 대신에 후에 테스트하거나 분석할 수 없다. 따라서 에러가 발생했을 때 문제를 진단하고 , 재현하고, 고치는 것이 불가능하다.

3. **로그 출력 레벨을 사용할 수 없다.** : 로컬에서 개발할때는 디버깅을 위한 상세한 정보가 출력되어 확인할 수 있어야한다. 하지만 상용 레벨에서는 에러/장애가 발생할때 문제를 진단할 수 있는 정보만을 남겨야한다. 하지만  `System.out.println()`은 이를 환경에 맞게 로그를 출력할 수 있는 기능을 제공하지않는다. 따라서 로깅 프레임워크를 사용해 환경에 맞는 로그를 출력해야한다.

4. **성능저하의 원인이 될수 있다.** : 
`System.out.println()` 은 단순한 출력이지만 대규모 애플리케이션에서 사용할 경우 성능에 영향을 미칠 수 있다.

```java
public void println(String x) {
        if (getClass() == PrintStream.class) {
            writeln(String.valueOf(x));
        } else {
            synchronized (this) {
                print(x);
                newLine();
            }
        }
    }
```

`print()` 메서드를 보면  `syncronized` 로 동기화되어 있는 것을 확인할 수 있다. 따라서 기록을 남길때마다 쓰레드lock이 걸리기 때문에 엄청난 성능 저하를 일으킨다.

그렇다면 자바에서 어떻게 로그를 남겨야할까?

# 2. 로깅(logging)

로깅이란란 정보를 제공하는 일련의 기록인 **로그(log)**를 기록하는 하는 것을 말한다. 로그를 남기면 개발을 할때 운영 중 발생하는 문제점을 모니터링하거나 추적하는데 용이하다. 또한 로그 데이터를 분석해 통계를 낼 수도 있다. 하지만 불필요한 로그를 사용히 무의민한 로그가 쌓이기때문에 주의해야한다.

# 3. 자바의 Logging 프레임워크

자바에는 다양한 로깅 프레임워크가 존재한다.

## 3.1 SLF4J(Simple Logging Facade for Java)

![slf](https://github.com/princenim/TIL/assets/59499600/29c5011b-f46d-41de-a46c-ec0871d106c8)


- 로깅에 대한 추상 레이어를 제공하는 인터페이스의 모음이다. 즉 SLF4J 자체가 프레임워크가 아니라 다양한 로깅 프레임워크을 같은 API를 사용해서 접근할 수 있도록 해주는 추상화 계층이다. 그래서 다른 로그 프레임워크와 같이 사용해야하는데 보통 `Logback`, `Log4j`을 사용한다.  다시말해 **어플리케이션은 `SLF4J`를 인터페이스를 통해 호출하지만 실제로 호출되는 로깅 프레임워크는 다른 프레임워크라는 이야기이다.**
- 유연하고 간결한 API 를 제공하면 쉽게 다른 로깅 시스템으로 전환할 수 있는 장점이 있다.
- `SLF4J`는 바인딩을 사용하는 클래스패스에서 찾은 바인딩을 사용해 실제로 로깅을 수행하는 구현체를 선택한다. 예를들어 `Logback` 을 사용하려면 `Logback`의 `SLF4J` 바인딩인 `logback-classic` 라이브러리를 클래스패스에 추가해야 한다.

## 3.2 Logback

- Logback은 SLF4J의 구현체 중 하나로, SLF4J와의 연동이 용이하다.
- Log4J의 후속  버전으로 속도와 효율성에서 뛰어난 성능을 보인다.

## 3.3 Log4j

- 오래된 로깅 프레임워크로써 2015년 개발중단이 되었다.
- xml 또는 속성파일을 사용해 로깅구성을 정의할 수 있어서 유연한 사용이 가능하다.

## 3.4 java.util.logging(JUL)

- JDK에서 기본적으로 포함된 자바 표준 로깅 API이다.
- 간단하고 가벼우며 추가적인 라이브러리없이 사용 가능하나 다른 로깅 프레임워크에 비해 유연성과 기능이 제한적이다.

## 3.5 Log4j2

- Log4j의 최신 버전이다.
- XML, YAML, JSON과 같은 다양한 형식의 설정 파일을 지원한다.

## 3.6 로그 사용법
### 로그 레벨

로그의 레벨은 5개로 설정에 따라 **설정 레벨과 그보다 레벨이 높은 로그를 출력한다**

| 레벨         | 설명 |
|------------| --- |
| Fetal      | 아주 심각한 에러가 발생한 상태. 시스템적으로 문제가 발생해 어플리케이션 작동이 불가능할 경우가 해당하는데 일반적으로 사용할 일이 없다.  |
| ERROR      | 요청을 처리하는 중 문제가 발생한 상태를 나타낸다.  |
| WARN       | 처리 가능한 문제이지만 향후 시스템 에러의 원인이 될 수 있는 경고성 메세지를 나타낸다.  |
| INFO       | 로그인, 상태변경과 같은 정보성 메세지를 나타낸다.  |
| DEBUG      | 개발시 디버그 용도로 사용한 메세지를 나타낸다.  |
| TRACE      | DEBUG 레벨이 너무 광범위한 것을 해결하기 위해서 좀 더 상세한 상태를 나타낸다.  |

### 로그 설정

사용하기위해서는 의존성 추가 및 레벨을 설정해야한다.

```groovy
dependencies {
    implementation 'ch.qos.logback:logback-classic:1.4.1'//logback
    implementation 'org.slf4j:slf4j-api:2.0.3' //slf4j
}
```

```yaml
#application.yml
logging:
  level:
    root: debug #debug로 설정시 debug부터 debug보다 레벨이 높은 로그를 남긴다. 
```

만약 기본 설정 외의 더 자세하고 설정할때는 `logback.xml` 파일을 `resource` 디렉토리안에 만들어서 설정하면 된다. 로그의 출력 형식이을 지정할 수 있으며 파일에 로그를 기록할 수 있다.

### 로그 사용

1. **Logger 객체를 선언**

```groovy

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyClass {
    public static void main(String[] args) {

				//org.slf4j 패키지의 LoggerFactory를 이용하여 Logger 객체 생성
        Logger logger = LoggerFactory.getLogger(MyClass.class); 

        logger.debug("Debug message");
        logger.info("Information message");
        logger.warn("Warning message");
        logger.error("Error message");
    }
}
```

1. `lombok`의 `@Slf4j` 어노테이션을 사용해 자동으로 설정된 log를 사용