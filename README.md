# 👋 understanding_Coroutines <br> 📌실습을 통한 코루틴 이해

### runBlocking (Coroutine Builder)
코드 블록이 수행이 끝날 때까지 thread를 blocking하고 작업을 실행하기 때문에 Test 용도에 사용하는 것을 권장

<br>

* 예제 1
``` kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    println(this)
    println(Thread.currentThread().name)
    println("Hello")
}
```
```
BlockingCoroutine{Active}@5b37e0d2
main
Hello
```

--- 
<br>

### launch (Coroutine Builder)
thread를 blocking 하지 않는 코루틴 작업을 실행하며, 결과를 반환할 필요가 없는 작업에 사용하며 Job 객체를 반환

<br>

* 예제 1
``` kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch: ${Thread.currentThread().name}")
        println("World!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    println("Hello")
}
```
```
runBlocking: main
Hello
launch: main
World!
```
두 코루틴 빌더 모두 메인 스레드를 사용하기 때문에 `runBlocking` 의 코드들이 메인 스레드를 다 사용할 때 까지 `launch`의 코드 블록이 기다림

<br>

* 예제 2
``` kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch: ${Thread.currentThread().name}")
        println("World!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    println("Hello")
}
```
```
runBlocking: main
Hello
launch: main
World!
```
`delay` 함수를 사용해 `runBlocking`의 코루틴이 대기 상태가 되고 `launch`의 코드 블록이 먼저 수행됨
이를 `Thread.sleep`로 변경한 경우엔 코루틴이 아닌 스레드가 대기 상태가 되어 

<span style="color:red">붉은 색</span>

과 같은 결과가 출력됨

<br>










``` kotlin

```
```

```
