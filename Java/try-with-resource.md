# **📌** try-with-resource

일반적으로 자원(resource)를 사용하고 낸 후에는 자원을 해제해야한다.  여기서 말하는 자원은 외부의 데이터(DB, File, Network)를 말한다. 만약 자원을 다 쓰고 난 후에 해당 자원을 해제하지 않으면 자원 누수가 발생해 메모리 누수(memory leak)과 같은 현상이 발생할 수 있다. 따라서 **자원을 다 쓰고 난 후에는 close를 통해 해재해줘야한다.**

`JDK 1.7` 이전에는 사용하고 난 자원을 직접 반납하기 위해 `try- catch -finally` 를 사용했다.
하지만 이 구문은 자원을 직접 반납해야했기때문에 불편했고 `JDK1.7` 부터 자동으로 자원을 해제해주는
`try- with -resource` 가 등장했다.

# 1. try-catch-finally

**finally는 예외가 발생하더라도 무조건 실행되는 블록이다.** 따라서 finally 블록에서 자원을 항상 안전하게 해제할 수있다.

```java
public void tryFinally() {
        FileInputStream fileInputStream = null;

        try {
            fileInputStream = new FileInputStream("hello.txt");
				// 파일에서 데이터를 읽어오는 코드

        } catch (FileNotFoundException e) {
            throw new RuntimeException(e); //예외 처리
        } finally { //항상 실행 
            if(fileInputStream != null){
                try {
                    fileInputStream.close(); //fileInputStream을 닫는 코드
                }catch (IOException e){ //close 문도 예외를 발생시킬 수 있어 예외처리
                    e.printStackTrace();
                }
            }

        }
    }
```

위의 코드는 `FileInputStream` 클래스 객체를 생성하고 사용한 뒤 자원을 해제하는 코드이다.

코드를 보면 `finally` 안에 다시 `try-catch`가 들어가 코드가 다시 복잡하게 된다.  따라서 만약에 입출력 처리가 더 많은 프로그램을 작성하면 코드가 더더욱 복잡하게 될것이다. 뿐만 아니라 작업이 번거롭고 , 실수로 자원을 반납하지 못하는 경우가 발생할 수있다.
이런 단점을 해결하려고 나온 `try-with-resource`가 등장했다.

### **catch문에 return 이 존재한다면?**

try-catch-finally 를 실행할때 만약에 catch문에 return 이 존재하면 finally를 실행할까 말까?

실행한다.

> The `finally` block *always* executes when the `try` block exits. This ensures that the `finally` block is executed even if an unexpected exception occurs. But `finally` is useful for more than just exception handling — it allows the programmer to avoid having cleanup code accidentally bypassed by a `return`, `continue`, or `break`.
>

[The Java™ Tutorials :The finally Block ](https://docs.oracle.com/javase/tutorial/essential/exceptions/finally.html)

해당 공식문서에서 찾을 수 있듯이 `return` , `continue`, `break`문이 있어도 무조건 `finally` 문이 실행된다. `System.exit()` 단, 이 문을 실행하면 finally문이 실행되지 않는다.

# 2. try-with-resource

```java
public void tryResource(){
        try (FileInputStream input = new FileInputStream("file.txt")) {
            // 파일에서 데이터를 읽어오는 코드
        } catch (IOException e) {
            // 예외 처리 코드
        }
  }
```

`try- with -resource`  구문은 자원을 사용한 후 자동으로 닫아주는 기능을 제공한다. try() 블럭을 벗어나는 순간 자동적으로 close()가 호출된다. 그리고 후에 catch 문이 수행된다. 따라서 코드를 보면 자원을 닫는 코드가 존재하지 않는다.

단, **이런 기능을 구현하기 위해서는 `AutoCloseable`를 구현하고 있어야한다.**

```java
public interface AutoCloseable {

    void close() throws Exception;
}
```

<img width="508" alt="스크린샷 2024-01-16 오후 1 28 15" src="https://github.com/princenim/TIL/assets/59499600/ffccd2a4-3854-4d0c-85fd-4e9141528f38">


따라서 확인을 해보면 `FileInputStream`은 `AutoCloseable`  를 구현(implements)를 하고 있으며 따라서 자동으로 자원을 반납하는 기능을 가지고 있는 것을 알 수 있다.