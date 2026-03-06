# 디자인 패턴 - 싱글턴 패턴

결국 static member 변수에 instance 를 미리 할당해두고, 생성자를 private 으로 막아서 외부에서 new 로 생성하지 못하게 막는 방식.
구현 방식은 크게 2가지가 있다.

## Class Loading 시점에 instance 할당
```kotlin
class SingletonEager private constructor() {
    companion object {
        private val instance: SingletonEager = SingletonEager()

        fun getInstance(): SingletonEager {
            return instance
        }
    }
}
```
- Class Loading 시점에 instance 가 할당된다.
- multi-threaded 환경에서 안전하다. 
- 사용될지 안될지 모르는 상황에서 미리 memory 를 점유하기 때문에 자원 낭비가 있을 수 있다.
  - 그러나 singleton pattern 을 고려하는 instance 라면 이미 사용처가 명확한 것.

## 최초 호출 시점에 instance 할당 
```kotlin
class SingletonLazy private constructor() {
    companion object {
        private var instance: SingletonLazy? = null

        @Synchronized
        fun getInstance(): SingletonLazy {
            if (instance == null) {
                instance = SingletonLazy()
            }
            return instance!!
        }
    }
}
```

- 최초 호출 시점에 instance 가 할당된다. Class Loading 시점에 비해 자원 낭비가 없다.
- multi-threaded 환경에서 안전하다.
- 다만 최초 호출 시점에 동기화(synchronized) 처리가 필요하다.
  - `@Synchronized` 키워드를 통해 method 전체를 동기화 처리한다.
  - 동기화 처리로 인해 성능 저하가 발생한다.

```kotlin
class SingletonLazy private constructor() {
    companion object {
        @Volatile
        private var instance: SingletonLazy? = null

        fun getInstance(): SingletonLazy {
            return instance ?: synchronized(this) {
                instance ?: SingletonLazy().also { instance = it }
            }
        }
    }
}
```
- 최초 호출 시점에 instance 가 할당된다. Class Loading 시점에 비해 자원 낭비가 없다.
- 역시나 multi-threaded 환경에서 안전하다.
- 다만 최초 호출 시점에 동기화(synchronized) 처리가 필요하다.
  - `@Volatile` 는 각 Thread 들에게 instance 변수에 대한 가시성을 보장(Main memory 직접 접근)한다.
  - 다만 Main memory 에 바로 접근하는 방식으로 인해 성능 저하가 발생한다.

## 사실은 enum 이...

위 모든 사항을 고려한 singleton pattern 구현 방식이 사실 enum 이다.

```kotlin
enum class SingletonEnum(
// ...
) {
    INSTANCE,
    ;
}
```

우리 enum 사랑해주실거죠?
