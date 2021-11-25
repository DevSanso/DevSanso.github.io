---
title: "리시버 있는 함수 리터럴 유용한 사용"
date: 2021-11-25T22:53:23+09:00
draft: true
---



```{.kt}
open class Person {
    internal var name : String = ""
    internal var age : Int = 0
}
```


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
fun main() {
    var builder = PersonBuilder(Person())
    builder
        .name("ahn")
        .age(25)
        .build()
}
```


```{.kt}
package com.example.kotlinTest.hello

object PersonMethod {
    infix fun Person.age(other : Int) {
        this.age = other
    }
    infix fun Person.name(other : String) {
        this.name = other
    }

}

fun build(person: Person,block : PersonMethod.(Person) -> Unit) {
    PersonMethod.block(person)
    print("hi my name is : ${person.name}\n")
    println("my age is : ${person.age}\n")
}
```

```{.kt}
fun main() {
    build(Person()) {
        it name "ahn"
        it age  25
    }
}
```
