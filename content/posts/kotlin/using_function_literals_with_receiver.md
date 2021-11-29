---
title: "리시버 있는 함수 리터럴 유용한 사용"
date: 2021-11-25T22:53:23+09:00
draft: false
tags : [kotlin,정리노트,기초]
---

### 계기 
코틀린으로 코딩하는도중 예전에 쓰던 exposed 프레임워크의 코드줄에서 궁금증이 생겼는데.

```{.kt}
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.transaction

object Users : Table() {
    ...
    val name = varchar("name", length = 50)
    ...
}
transaction {
    ....
    ....
    Users.update({ Users.name eq "city"}) {
        it[name] = "bob"
    }
}
```
이라는 코드줄이 있다고 할때, User 싱글톤 객체에 name에 infix 함수인 eq가 있다. 그럼 이걸 update함수 밖에서 
써도 될까에서 시작해서 그렇게 작성해서 컴파일 해봤는데.

```{.kt}
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.transaction

object Users : Table() {
    ...
    val name = varchar("name", length = 50)
    ...
}
transaction {
    ....
    ....
    val exist = Users.name eq "city"
    Users.update({ exist }) {
        it[name] = "bob"
    }
}


/*
e: main.kt: (13, 32): Unresolved reference: eq
*/
```
이러한 에러가 나왔다. 분명 update안에는 컴파일이 됬는데 밖에서 사용하면 해당 함수는 없다고 에러가 나온다.
그래서 찾아본 결과 코틀린의 확장함수랑 관련된걸 알게 됬다.

```
저 에러를 해결하는 법은 import org.jetbrains.exposed.sql.SqlExpressionBuilder.eq 추가하면 된다.
```

### 본문 및 설명

우선 사람의 이름과 나이를 출력하는 프로그램을 프로그래밍 한다고 치자.

```{.kt}
open class Person {
    internal var name : String = ""
    internal var age : Int = 0
}
```
단순하게 사람을 추상화한 클래스이다. 이것을 빌더패턴을 사용하는 방식으로 작성한다고 치자.
그래서 평소에 다른언어에서 하던 방식으로 작성하였다.

```{.kt}
package com.example.kotlinTest.hello

class PersonBuilder(val person: Person) {

    fun name(n : String) :PersonBuilder {
        this.person.name = n
        return this
    }
    fun age(a : Int) : PersonBuilder {
        this.person.age = a
        return this
    }
    fun build() {
        print("hi my name is : ${person.name}\n")
        println("my age is : ${person.age}\n")
    }
}
```

```{.kt}
import com.example.kotlinTest.hello.*

fun main() {
    var builder = PersonBuilder(Person())
    builder
        .name("ahn")
        .age(25)
        .build()
}
```
이렇게 작성하고보니 비록 본인의 코딩실력이 좋지않아도 이정도면 괜찮지 않을까 싶었지만, 
사실 코틀린은 다른언어보다 지원하는 문법들이 많았고 이것을 활용하면 더 보기 좋은 코드를 작성할수 있다고 한다. 
그래서 찾아보니 infix 함수랑 리시버 함수가 있다고 한다.

해당 함수들의 설명은 공식 문서에 가면 있으니 먼저 보고 오는게 좋다.  
*infix 함수 : <https://kotlinlang.org/docs/functions.html#infix-notation>*  
*리시버 함수 : <https://kotlinlang.org/docs/lambdas.html#function-literals-with-receiver>*  
  
이것들을 활용하면 위의 update함수처럼 별도의 import가 없으면 안에서는 사용할수 있지만, 밖에서는 사용못하게 할수도 있다.
이제 infix,리시버들을 활용한 소스코드이다.
```{.kt}
package com.example.kotlinTest.hello
/*
본래 해당 객체안의 확장 함수들은, PersonMethod를 임포트 안하면 사용이 안된다.
*/
object PersonMethod {
    infix fun Person.age(other : Int) {
        this.age = other
    }
    infix fun Person.name(other : String) {
        this.name = other
    }

}
/*
block 변수로 리시터 함수 리터럴을 받게 됨으로써, PersonMethod안의 함수들이 사용가능해 진다.
*/
fun build(person: Person,block : PersonMethod.(Person) -> Unit) {
    PersonMethod.block(person)
    print("hi my name is : ${person.name}\n")
    println("my age is : ${person.age}\n")
}
```

```{.kt}
import com.example.kotlinTest.hello.build
import com.example.kotlinTest.hello.Person

fun main() {
    build(Person()) {
        it name "ahn"
        it age  25
    }
}
```
예시가 좋지 못하지만, 이렇게 소스코드를 작성할수가 있다. 즉 PersonMethod 객체를 임포트 안해도 Person 확장 함수를 사용할수 있다.

```
위 계기의 소스코드 해결법처럼 확장함수가 정의된 import com.example.kotlinTest.hello.PersonMethod를  추가하면 밖에서도 사용이 가능하다.
```
