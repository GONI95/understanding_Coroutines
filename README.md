# 👋 understanding_Coroutines <br> 📌실습을 통한 코루틴 이해

## 목차

### [1. runBlocking (CoroutineBuilder)](#1-runBlocking)
### [2. launch (CoroutineBuilder)](#2-launch)
### [3. suspend](#3-suspend)
### [4. join](#4-join)

### 1. runBlocking
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




### 2. launch
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
이를 `Thread.sleep`로 변경한 경우엔 코루틴이 아닌 스레드가 대기 상태가 되어 ***예제 1***과 같은 결과가 출력됨

<br>

* 예제 3
``` kotlin
import kotlinx.coroutines.*

fun main() {
    runBlocking {
        launch {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        }
        launch {
            println("launch2: ${Thread.currentThread().name}")
            println("1!")
        }
        println("runBlocking: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")
    }
    print("4!")
}
```

```
runBlocking: main
launch1: main
launch2: main
1!
2!
3!
4!
```
상위 코루틴은 하위 코루틴을 끝까지 책임지기 때문에 `runBlocking` 내부의 두 `launch`가 끝나기 전까지 종료되지 않고 상위 코루틴이 취소되면 하위 코루틴도 취소된다

---
<br>






### 3. suspend
중단 가능한 함수를 의미하며, 함수에서 `delay`, `launch`를 포함한 코드를 작성하고 싶은 경우 `suspend` 키워드 사용
중요한 점은 CoroutineBuilder는 일반적으로 코루틴 내부에서 호출되어야 하기 때문에 suspend 키워드가 있는 함수라고 해서 호출할 수 있는 것은 아님.

<br>

* 예제 1
``` kotlin
import kotlinx.coroutines.*

suspend fun doThree() {
    println("launch1: ${Thread.currentThread().name}")
    delay(1000L)
    println("3!")
}

fun doOne() {
    println("launch1: ${Thread.currentThread().name}")
    println("1!")
}

suspend fun doTwo() {
    println("runBlocking: ${Thread.currentThread().name}")
    delay(500L)
    println("2!")
}

fun main() = runBlocking {
    launch {
        doThree()
    }
    launch {
        doOne()
    }
    doTwo()
}
```

```
runBlocking: main
launch1: main
launch1: main
1!
2!
3!
```
--- 
<br>





### 4. join
코루틴 빌더인 launch는 Job 객체를 반환하며 이 Job 객체를 통해 이 Job을 대기, 취소 등의 핸들링이 가능
중요한 점은 join()이 호출된 시점부터 해당 Job 객체에 대한 처리를 기다린 후 아래 코드를 실행한다는 것
즉, join()이 호출되는 위치에 따라 결과가 달라질 수 있음.

<br>

* 예제 1 :  join()이 없는 경우 4 -> 1 -> 2 -> 3 -> 5 순으로 출력됨
``` kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    val job = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }
    
    job.join()

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}
```

```
launch1: main
3!
4!
launch2: main
1!
launch3: main
2!
runBlocking: main
5!
```
<br>

* 예제 2 :  join()이 없는 경우 4 -> 1 -> 2 -> 3 -> 5 순으로 출력됨
``` kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    val job = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    job.join()

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}
```

```
launch1: main
launch2: main
1!
3!
4!
launch3: main
2!
runBlocking: main
5!
```
<br>

* 예제 3 : join()이 없는 경우 4 -> 1 -> 2 -> 3 -> 5 순으로 출력됨
``` kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    val job = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")
    }

    job.join()
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}
```

```
launch1: main
launch2: main
1!
launch3: main
2!
3!
4!
runBlocking: main
5!
```
<br>


취소와 타임아웃


``` kotlin

```
```

```
