# **📌** ArrayList의 특징과  메소드의 시간복잡도

# 1. ArrayList

![collection](https://github.com/princenim/TIL/assets/59499600/07cbe881-653c-45e5-9f9a-0b101a4ac06a)

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{}

```

- `ArrayList`는 자바의 컬렉션 프레임워크의 일부로 `List` 인터페이스를 구현받은 클래스이다. 따라서  메모리 **데이터의 저장순서가 유지되고, 중복을 허용**한다는 특징을 갖는다.
- 연속적인 데이터를 갖는 리스트로 중간에 빈공간이 있으면 안 된다.
- 내부적으로 `Object[]` 배열을 이용해서 요소를 저장한다. 따라서 인덱스를 이용해 빠르게 요소에 접근할 수 있다.
- 데이터를 중간에 삽입/삭제할 경우 중간에 빈 공간이 생기지 않도록 요소돌의 위치를 앞뒤로 자동으로 이동시키기 때문에 삽입/삭제 동작은 느리다.
- 동적으로 길이가 변한다.

### 더블링(Doubling)
ArrayList의 가장 큰 특징은 가변길이로 내부가 배열로 구현되어있는데 이 배열의 크기가 꽉차면 새로운 배열을 만들어 기존 요소값들을 복사한다.
자세히 설명하면  `ArrayList`는 생성하면 내부에서 데이터를 저장하기 위한 `default 10`인 `capacity` 크기의 저장공간의 배열을 만든다.

```java
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;
```

그리고 데이터가 추가되어 데이터의 크기가 `capacity`(= 용량)를 넘어가면 기존 배열의 `1.5배` 크기의 더 큰 새로운 배열을 생성해 새로운 배열로 하나씩 복사한다. 이러한 원리로 동적으로 `ArrayList`의 크기가 동적으로 늘어난다. 참고로 `size`와 `capacity`는 다른다. `size`는 요소의 개수를 말하고, `capacity`는 용량을 의미한다.

`ArrayList`를 사용할때 알아야할점은 기존 배열의 `디폴트 값인 10`을 넘으면 내부적으로 새로운 배열이 생기고 그 배열에 복사하는 작업이 발생한다. 즉 **복사라는 작업이 공짜가 아니기때문에** 동적으로 크기를 조절한다는 장점만 보고 `ArrayList`를 무작정 사용할 것이 아니라 내부적으로 작업이 일어난다는 것을 이해하고, 배열의 크기가 정해져있다면 배열을 사용하고 그 외의 동적으로 배열의 크기를 조절해야할때는 `ArrayList`를 사용하는 것이 좋다.

### 로드팩터 (load factor)

로드팩터란 용량 대비 데이터가 어느정도 찼을 때 내부적으로 사이즈 확장을 하는 자료구조에서 사용되는 개념으로 **언제 사이즈를 늘려야할까?** 의 기준점이라고 생각하면 된다. `ArrayList`는 배열이 꽉 찼을 때 새로운 배열을 생성하므로 1이다.  참고로 HashMap의 로드팩터는 0.75이다.

### 지연로딩(lazy loading)

`ArrayList`는 `new` 키워드를 통해 객체를 생선하는 시점에 메모리에 객체가 생성되지 않고 `add()`메소드가 호출되어 값이 추가되는 시점에 메모리에 생성된다. 이렇게 선언과 사용을 하는 시점이 다른 것을 지연로딩이라고 하며 이 지연로딩을 사용함으로써 속도를 높이고 메모리의 사용량을 줄일 수 있다.

즉, `ArrayList`가 지연로딩인이유도 `ArrayList`가 선언되더라도 언제 사용할지 몰라 사용되는 시점에 로딩하여 성능을 올리기 위함이다.



# 2.배열(Array)과의 차이점

`Arraylist` 와 `배열`는 모두 데이터를 저장하는 자료구조이다. 하지만 중요한 차이점이 있다.

1. **크기의 가변성(resizable)** : `배열`은 고정길이를 갖는다. 따라서 생성할때 정한 길이의 크기에 `배열`이 생성되고 더 이상 **길이를 늘릴거나 값을 삭제할 수 없다.** 하지만 `ArrayList`는 가변길이이다.  생성한 후에도 동적으로 사이즈가 늘어난다. 따라서 생성한 후에도 요소를 추가하거나 삭제함으로써 `ArrayList`의 길이가 변할 수 있다.
2. **속도와 성능** : 속도와 성능은 수행되는 작업에 따라달라진다. 요소를 인덱스로 가져오는 작업에서는 비슷한 성능을 보인다. 하지만 `ArrayList`는 크기를 동적으로 조절하기 때문에 삽입하거나 삭제시 요소를 이동할때  약간의 오버헤드가 발생할 수 있다. 하지만 일반적인 상황에서는 성능 차이가 크게 나지 않는다.
3. **제네릭** : 배열은 제네릭을 사용할 수 없지만, ArrayList는 타입의 안정성을 보장해주는 제네릭을 사용할 수 있다.

# 3. 주요 메소드의 시간복잡도

1. `add(E element)` :  ArrayList  마지막에 객체를 추가한다.
    1.  시간복잡도 : `O(1)`
2. `add(int index, E element)`: 지정된 index에 객체를 저장한다.
    1. 시간복잡도 : `최악의 경우 O(N)`으로 특정 인덱스에 요소를 추가하는 경우 해당 인덱스 이후의 모든 요소를 뒤로 이동시켜야 한다.
3. `remove(Object o)`: 지정한 객체를 제거한다.
    1. 시간복잡도 : `최악의 경우 O(N)`으로 해당 요소를 찾고나서 삭제하면 해당 인덱스 이후의 모든 요소를 한칸 앞으로 이동시켜야한다.
4. `remove(int index):` index에 위치한 객체를 제거한다.
    1. 시간복잡도 : `최악의 경우 O(N)`으로 특정 인덱스의 요소를 삭제하면 해당 인덱스 이후의 모든 요소를 이동시켜야 한다.
5. `get(int index):` index에 저장된 객체를 반환한다.
    1. 시간복잡도 : `O(1)`로 배열기반으로 인덱스를 이용해 직접 접근해 객체를 찾을수 있다.
6. `size():` `ArrayList`에 저장된 객체의 개수를 반환한다.
    1. 시간복잡도 : `O(1)` 로 `ArrayList`는 내부적으로 저장된 현재요소의 개수를 갖고 있다. 따라서 추가적인 계산이나 순회가 필요하지 않다.
7. `contains(Object o):` : 지정된 객체(o)가 `ArrayList`에 포함되어있는지 확인
    1. 시간복잡도 : `최악의 경우 O(N)`으로 특정 요소가 `ArrayList`에 존재하는지 확인하려면 모든 요소를 순회해야하기 때문이다.