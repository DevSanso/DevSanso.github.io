---
title: "await와 then의 작동 차이"
date: 2021-09-10T00:41:00+09:00
draft: false
tags : [javascript,실험]
---

자바스크립트를 공부하다보면 비동기 함수 사용이라고 **setTimeout** , **Promise** 그리고 **async과 await**를 배우게 된다. 허나 이러한 사용법이 있다고 하고, 어떤 상황에 어떻게 써야하는지는 자세히 모르는경우 가 많았고, 그리고 이번에 이 궁금점을 풀기위해 코드를 작성하다가 알게된 내용을 적어볼려고한다.  
보통 자바스크립트는 싱글 스레드이기 때문에 비동기api를 이용하여 멀티스레딩을 한다고 이야기를 한다. 결국 비동기로 병렬처리를 하는것처럼 보여도 결국 싱글스레드에서 모든일을 한다는것이다, 그럼 만약 여러 비동기 함수안에 처리시간이 많은 루프가 있으면 어떻게 될까?  
해당 궁금점을 알기위해 코드를 작성하기로 마음먹고, 다음과 같은 규칙과 목표를 세웠다.

### 프로그램 규칙
1. 웹페이지를 전송주고, 특정 데이터를 보내주는 서버를 만든다, 그리고 데이터는 0~1000사이의 정수이며, 데이터의 값은 랜덤이다.
2. 웹사이트는 처음 로딩시 4개의 객체가 있고,각 객체만 무작위로 하나의 정수를 할당받는다.
3. 각 하나의 객체는, 각자의 정수값이 서버로부터 받는 정수데이터값하고 동일할때까지 계속 서버에 데이터값을 요청한다.
4. 만약 데이터가 동일하지 않으면 화면에 실패 횟수를 보여준다
5. 하나의 객체가, 서버로부터 온 데이터랑 동일하면 활동을 멈춘다

![대략적인 구조](/posts/javascript/difference_between_await_and_then/웹_서버.png)
설명이 부족해 보일수도 있기에, 일단 코드예시도 올리기로 결정했다.
``` go
//server source code
package main

import (
	"net/http"
	"log"
	"os"
	"strconv"
	"math/rand"
)

func ramdonIntHandler(w http.ResponseWriter,r *http.Request) {
	var s = rand.Int() %1001
	var conv = strconv.Itoa(s)
	w.WriteHeader(200)
	w.Write([]byte(conv))
}


func main() {
    //www : 웹페이지가 저장되있는 폴더
	fs := http.FileServer(http.FS(os.DirFS(".\\www")))
	http.Handle("/", fs)
	http.HandleFunc("/random",ramdonIntHandler)
	log.Println("Listening on :3000...")

	err := http.ListenAndServe(":3000",nil)
	if err != nil {log.Fatal(err)}
}
```

``` html
<!-- 웹페이지 html -->
<!DOCTYPE html>
<html>
    <head>
        <title>match number</title>
        <meta charset="utf-8">
        
    </head>
    <body>
        <section id="app">
            <div class="switchBox box1">
                <span>0</span>
            </div>
            <div class="switchBox box2">
                <span>0</span>
            </div>
            <div class="switchBox box3">
                <span>0</span>
            </div>
            <div class="switchBox box4">
                <span>0</span>
            </div>
        </section>
        <!-- 비동기로 서버로 요청하는 라이브러리-->
        <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
        <script src="./assets/js/main.js"></script>
        <button onclick="main();">Start</button>
    </body>
</html>
```

``` js

const Box = function(className) {
    this.switch = false;
    this.num = parseInt((Math.random() * 1000));
    this.className = className

    this.update = (data) => {
        this.switch = this.num == data ? true : false;
    }

    let doc =document.querySelector("."+className + " > span");

    return {
        //Promise.then을 이용한  함수
        loop :async () => {
            while(!this.switch) {
               axios({
                    method: 'get',
                    url: "/random"
                }).then((value)=> {
                    let d = parseInt(value.data);
                    this.update(d);
                    doc.textContent = parseInt(doc.textContent) + 1
                });
            }
            console.log("done " + this.className);
        },
        //setTimeout를 이용한 루프
        loopSetTimeout : () => {
            let body = () => {
                axios({
                    method: 'get',
                    url: "/random"
                }).then((value)=> {
                    let d = parseInt(value.data);
                    this.update(d);
                    doc.textContent = parseInt(doc.textContent) + 1
                    if(!this.switch)setTimeout(body,10);
                });
            }

            setTimeout(body,10);
        },
        //await를 이용한 함수
        loopAwait :async () => {
            while(!this.switch) {
                let res = await axios({
                    method: 'get',
                    url: "/random"
                });
                
                let d = parseInt(res.data);
                this.update(d);
                doc.textContent = parseInt(doc.textContent) + 1;
                
            }
            
            console.log("done " + this.className);
        },
        getNum : () => this.num
    }
}


function AllocBox(ele) {
    let element = new Box(ele);
    //element.loop();
    //element.loopSetTimeout();
    //element.loopAwait();
    console.log(ele + " Box Number = " +  element.getNum());
    
}

const main = () => {
    AllocBox("box1");
    AllocBox("box2");
    AllocBox("box3");
    AllocBox("box4");
}

```
자바스크립트에는 각자 어떻게 작동되는지 알기위해 각 3개의 함수를 작성하고 테스트를 해본 결과 뜻밖에 결과가 나왔다.  
결론부터 말씀드리면, setTimeout과 await는 정상적으로 작동되는걸 확인되었다.
![result](/posts/javascript/difference_between_await_and_then/await-사용.gif)
###### 해당 결과는 await의 결과지만, setTimeout도 비슷하게 작동하였다.
그 대신 promise.then쪽은 예상과 다르게 작동했는데, 결과값이 다른것도 아닌 아예 웹페이지 자체가 작동을 멈추게 되었다.
![result](/posts/javascript/difference_between_await_and_then/await-안쓸시.gif)
###### 새로고침, 페이지 종료창을 마우스로 클리해도 응답 자체가 되지 않았다.

우선 await부터 살펴보자, 사실 본래 await는 저렇게 객체들이 동시에 서버에 요청하기에 한꺼번에 숫자 횟수가 바끼는걸 볼수 있다. 사실 이건 본인이 예상했던 결과가 아니다.  
그 이유를 나열하자면
* 비동기로 처리해도 결국 하나의 스레드는 하나의 함수만 처리할수 있다.
* 그리고 실행중인 함수가 끝나야 이벤트큐에 들어있는 다음 비동기 함수를 실행할수 있다.
* 만약 처리하는 함수가 오래 걸리는 루프가 있을시, 해당 루프를 탈출하기 전까지 함수는 종료되지 않는다.
* 그렇기에 결국 한객체만 처리할수밖에 없는 상황이 된다. 


```
위 조건에 맞는것이 바로 promise.then를 사용한 방법이다. 
실제 함수 중간에 console.log 함수를 작성하여 테스트한 결과, 한 객체의 promise만이 수없이 작동되게 된다. 그렇게 이벤트 큐에 끊임없이 쌓이게되니 브라우저가 결국 먹통되었다고 추측하고 있다.
```

허나 그렇게 되지 않았고, 그 이유를 찾아본 결과, async함수에서 await블록으로 진입시, 해당 함수는 **await블록에서 일시 정지되고**, 다음 async함수가 실행되게 된다, 그후 일시정지된 함수가 다시 돌아오면 await가 처리 완료되면 async함수를 마저 처리하고, 만약 아직 완료가 안되면 또 일시 정지후 다음 함수가 실행된다는거다.

##### 정확한 내용을 알고싶으면 해당 사이트를 참고하면 좋을것이다.
* [mdn web docs](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/await)
* [stackoverflow question](https://stackoverflow.com/questions/49419393/event-loop-flow-with-async-await)