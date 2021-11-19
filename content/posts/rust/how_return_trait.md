---
title: "trait를 리턴하는 방법"
date: 2021-10-30T13:09:42+09:00
draft: false
tags : [tip,rust,정리노트]
---

러스트로 코딩하던 도중에 트레이드를 리턴해야하는 함수를 만들어야 하는 상황이 있었다.
그래서 go언어를 코딩하는것과 마찬가지로 리턴타입을 트레이드(go언어에선 인터페이스)로 하여 리턴 하는 함수를 작성후
컴파일 해본적이 있었는데 그때의 일을 적어볼려고 한다.

``` go
//go 코드 예시
package main

import "fmt"

type Hi interface {
    hi()
}

type String struct {
    s string
}

func new_hi(s string) Hi {
    return &String{s}
}
func (s *String)hi() {
    fmt.Println("hi : "s.s)
}

func main() {
    var str = "sunken ahn"
    new_hi(str).hi()
}
```

이런한 기능을 하는 프로그램을 작성한다고 할때, 이것을 러스트로 재작성할려고 한다.  
그래서 내 생각대로 러스트 코드를 작성하게 되었다.
``` rust
//rust 코드 예시
trait Hi {
    fn hi(&self);
}

impl Hi for String {
    fn hi(&self) {
        println!("hi : {}",self);
    }
}

fn new_hi(val : String) -> Hi {
    return val;
}

fn main() {
    let name = String::from("sunken ahn");
    new_hi(name).hi();
}
```
이렇게 작성하고 컴파일 하니 에러메세지가 나타났다.
![result](/posts/rust/how_return_trait/cant_compile.PNG)
그래서 처음에는 __이게 왜?__ 였다. 해당 에러가 리턴 타입 지정에서 에러가 나왓는데
다른언어들은 원하는 타입을 리턴하고 싶으면 함수에 리턴 타입을 지정하는데 말이다.


그리고 찾아보니, rust의 컴파일 방식 때문인걸 알게 되었다.
러스트는 메모리 안전성을 위해 다른언어들과 다르게 컴파일 단계에서 메모리 관리하고 안전성을 추척한다.
그렇기에 메모리 관리를 위해 변수의 메모리 크기등을 컴파일러가 알아야 하는데, 트레이드는 이른바 일종의 여러 메소드의 묶음박스일뿐,
그 자체가 타입이 아니다. 그렇기에 변수의 메모리 크기의 정보가 없기에, 컴파일러는 해당 함수의 리턴값이 어느정도의 크기인지 알수없기에
컴파일 에러를 보내는것이다.  

그렇기에 다른방식으로 값을 리턴시키면 된다.
첫번째로 제너릭으로 함수 인수값(Hi를 구현한 String)을 받은후, 해당 인수타입으로 내보내는 방법.
![first](/posts/rust/how_return_trait/use_Generic.PNG)
두번째로 박스라는 러스트의 힙할당 구조체를 사용하여 리턴하는 방법.
![second](/posts/rust/how_return_trait/use_box.PNG)
그리고 내가 알고 있는 마지막 방법으로 impl trait를 리턴 타입으로 적어서 쓰는 방법이다.
![three](/posts/rust/how_return_trait/use_impl.PNG)

##### 참고
[Traits: Defining Shared Behavior](https://doc.rust-lang.org/book/ch10-02-traits.html)