---
title: "코틀린 하면서 느낀 infix 유용함"
date: 2021-11-23T23:58:33+09:00
draft: false
tags : [kotlin,정리노트,기초]
---

코틀린으로 spring 프로젝트를 코딩하고 있는데, 데이터베이스를 사용해야해서 JetBrains(코틀린 만든회사)에서
만든 exposed 라는 sql프레임워크를 사용하게 되었다.  
그렇게 해당 라이브러리로 테이블 객체를 만들고, select를 수행하는 함수에서 where 문을 추가해야 하는 상황에서
infix 함수를 사용하게 되었다.  
일단 예시로 평소에 나는 이렇게 코딩하고 있었다

``` {.kt}
/*
...
...
...
*/
data class User : Table() (val name : String,val age : Int,/*..*/)
data class NameAndAge(/*..*/)
fun selectUser() : List<NameAndAge> {
    //select 안에 where 관련 기능이 추가된다.
    return User.select({
        User.name.eq("name").and(User.age.eq(12))
    }).map({
        NameAndAge(/*..*/)
    })
}
```
이렇게 selectUser함수를 작성하는도중 다른 기능이 필요하여 exposed 저장소에서 예제를 보는도중, 내가 작성한거랑
전혀 다른 예제가 있었다. 그전에 일단 infix함수가 무엇인가에 알아보자
### infix function
``` {.kt}
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2

// is the same as
1.shl(2)

```
해당 예시는 공식문서에서 갖고온거다. 공식 문서에 설명에서는 함수에 infix 예약어를 표시함으로써 infix 표기법으로 사용할수가 있다고 한다.  
개인적인 감상으로썬 보통 내가 사용하는 함수표기법이 컴퓨터에 맞쳐졌다면, 이걸 사람이 더 알아보게 쉽게 코드를 작성할수 있게 한거같다고 느꼈다. 
그리고 이러한 방식은 아까 사용했다는 sql프레임 워크에서도 유용하게 사용 됬는데, 위 코드를 infix 함수로 바꾸면 이렇게 변한다.
``` {.kt}
/*
...
...
...
*/
data class User : Table() (val name : String,val age : Int,/*..*/)
data class NameAndAge(/*..*/)
fun selectUser() : List<NameAndAge> {
    return User.select {
        (User.name eq "name") and (User.age eq 12)
    }.map {
        NameAndAge(/*..*/)
    }
}
```
아까전 코드보다 더 좋게 변하였다.  
이렇게 하다보니 느낌점이 있다. 여러 프로그래밍 언어들 하다보면 각 언어의 장점들이 있는데, 여러개 쓰다보면 해당 장점들을 사용할 생각이 안들고 
그저 다른언어에서 사용하던 평소느낌의 코딩 방식을 고수한다.  
그것이 좋을지도 모르지만, 각 언어의 장점들을 이용해 코드를 좀더 보기 좋게, 코드길이를 더 적게 할수도 있는 선택지를 포기하는 단점이 있다는걸 알았다. 
그래서 앞으로 코딩할땐 언어의 특징들을 자세히 찾아보게 하는게 나을것같다.


