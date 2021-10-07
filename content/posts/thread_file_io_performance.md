---
title: "싱글,멀티 스레드 파일 입출력 성능 "
date: 2021-10-07T10:27:57+09:00
draft: false
tags: [tip,etc]
---

프로그래밍 하는 도중 파일을 입출력 코드를 작성하는 상황이 있었는데, 여기서 "파일 입출력을 싱글 스레드 그리고 멀티 스레드중 어느것이 나을까?"라고 궁금중이 생겨서, 따로 코드를 작성하고 테스트했던것을 적어볼려고 한다.  
우선 나는 어느것이 성능이 좋을지 생각하면서  어떤 프로그램 요구서를 생각하고, 그것에 따라서 작성해보았다.


### 프로그램 계획
##### 요구 : 랜덤 해쉬값을 생성후, 해당값을 파일이름 그리고 파일내용으로 작성하여 특정폴더에 저장한다.

1. 프로그램은 싱글,멀티스레드 선택값,폴더경로 그리고 파일 생성 갯수값을 인수로 입력받는다
2. 해쉬값을 랜덤으로 생성한다
3. 해당값으로 파일명으로 해서 생성후, 해당 파일에 해당값을 저장
4. 이 행동을 생성 갯수값만큼 반복한다
5. 반복이 완료후의 시간을 측정하고, 출력한다
6. 프로그램 종료

그리고 해당 요구서를 토대로 코틀린으로 작성해보았다.
```{.kt} 
//main.kt

//메인 함수
import java.lang.Exception
import kotlin.system.measureTimeMillis

fun main(args: Array<String>) {

    val c = args.first().toInt() // 1 : 멀티스레드,2: 싱글스레드
    val root = args[1] // 폴더 경로
    val count = args[2].toInt() // 파일 생성 갯수
    val fIO : IoTest = when(c) {
        1 -> AsyncFileIO(count,root,4)// 스레드는 4개 생성후 사용
        2 -> NonFileIO(count,root)
        else -> throw Exception("not 1 and 2")
    }

    val time = measureTimeMillis {
        fIO.run()
    }

    println("time : ${time}(ms)")
}
```

```{.kt} 
//IoTest.kt

//추상클래스
abstract class IoTest
constructor(protected val maxCount : Int,protected val root : String) {
    abstract fun run()
}
```


```{.kt} 
//NonFileIO.kt
//싱글 스레드 작동 객체

import kotlin.io.*
import java.io.File
import java.nio.charset.Charset
import java.util.UUID
import java.nio.file.Paths



class NonFileIO(maxCount: Int, root: String) : IoTest(maxCount, root) {
    override fun run() {
        var i = 0;
        var random_uuid = UUID.randomUUID().toString()

        while(i < maxCount) {
            var file = File(Paths.get(root,random_uuid).toString())
            file.createNewFile()
            file.writeText(random_uuid, Charset.defaultCharset())
            random_uuid = UUID.randomUUID().toString()
            i++
        }
    }

}
```

```{.kt}
//AsyncFileIO.kt
//멀티스레드 작성 객체
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel
import java.io.File
import java.nio.charset.Charset
import java.nio.file.Paths
import java.util.*

class AsyncFileIO(
        maxCount: Int, root: String,private val tCount : Int) : IoTest(maxCount, root) {


    private suspend fun createWorker (channel : Channel<String>) {
        for(i in 1..tCount) {
            //해당 코드를 사용하면 자바내부에 스레드가 생성된다.
            CoroutineScope(Dispatchers.IO).launch {
                while(!channel.isClosedForSend) {
                    if(channel.isEmpty)continue
                    val msg = channel.receive()
                    var file = File(Paths.get(root,msg).toString())
                    file.createNewFile()
                    file.writeText(msg, Charset.defaultCharset())
                }
            }
        }

    }

    override  fun run() : Unit = runBlocking {
        val channel = Channel<String>()


        val job = async {
            createWorker(channel)
        }
        for(i in 1..maxCount) {
            var random_uuid = UUID.randomUUID().toString()
            channel.send(random_uuid)
        }
        channel.close()
        job.await()
    }



}
```

### 간단 설명 이미지
![멀티 스레드 설명](/posts/thread_file_io_performance/설명.png)
![싱글 스레드 설명](/posts/thread_file_io_performance/설명2.png)

이것을 토대로 10000개의 파일을 생성하는 테스트를 해본결과...  
### 결과창 
![결과](/posts/thread_file_io_performance/결과창.PNG)
|방식|시간표|
|-----|--------|
|싱글스레드|68.953s|
|멀티스레드|46.261s|

이렇게 꽤 차이 나는 결과를 알게되었다.
