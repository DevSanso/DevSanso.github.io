---
title: "몇몇 타입이 구조체의 맴버변수를 리턴 못하는 이유"
date: 2021-12-05T14:24:22+09:00
tags : [rust,정리노트,기초]
draft: false
---

코딩하던 도중에, 구조체의 변수값을 리턴하는 함수를 만들던 도중에 있던일을 적을라고 한다.
일단 구조체에 있는 문자열 변수를 읽는 함수를 제작했다 하자.  
``` rust

struct A {
    num : u32,
    s : String
}
impl A {
    fn get_string(&self) -> String {
        self.s
    }
/***/
}

fn main() {
    let object = A{num : 0,s : String::from("")};
    let _copyS = object.get_string();
/***/
}

```
다른 언어에서 작성하던것처럼 문자열의 getter를 함수로 만들어 놓고 컴파일 해보니 이런 에러가 뜬다.
```
Compiling playground v0.0.1 (/playground)
error[E0507]: cannot move out of `self.s` which is behind a shared reference
 --> src/main.rs:7:9
  |
7 |         self.s
  |         ^^^^^^ move occurs because `self.s` has type `String`, which does not implement the `Copy` trait

For more information about this error, try `rustc --explain E0507`.
error: could not compile `playground` due to previous error
```
아 러스트는 대입이 아닌 이동이니 그런걸수 있겠다고 생각할수 있지만, 정수형으로 함수를 작성하면 정상적으로 작동하는걸 알수있다.
``` rust
struct A {
    num : u32,
    s : String
}
impl A {
    fn get_u32(&self) -> u32 {
        self.num
    }
/***/
}

fn main() {
    let object = A{num : 0,s : String::from("")};
    let _copyS = object.get_u32();
/***/
}
/*
경고창을 나오지만 컴파일 에러는 나오지 않는다.
*/
```
그래서 찾아보니 Copy라고 복사하고 관련된 트레이드 인데, 해당 트레이드를 구현하는 타입들은 
대입연산자(=)에서 묵시적으로 변수 이동이 아닌 변수 복사가 된다고 한다.  
즉 u32은 기본적으로 copy트레이드가 구현되 있지만, String 타입은 해당 트레이드 구현이 안되있다, 그래서 
getter 함수에서는 값 이동으로 작동하는데, self가 참조형이여서 이동이 안된다고 컴파일 에러가 나는것같다.
그럼 참조형을 안쓰면 되는거 하면?, getter함수 호출후에 구조체 사용이 불가능한다는 문제가 있다.
``` rust
struct A {
    num : u32,
    s : String
}
impl A {
    fn get_string(self) -> String {
        self.s
    }
/***/
}

fn main() {
 let object = A{num : 0,s : String::from("")};
    let _copyS = object.get_string();
    println!("{}",object.get_string())
/***/
}
/* output
Compiling playground v0.0.1 (/playground)
error[E0382]: use of moved value: `object`
  --> src/main.rs:15:19
   |
13 |  let object = A{num : 0,s : String::from("")};
   |      ------ move occurs because `object` has type `A`, which does not implement the `Copy` trait
14 |     let _copyS = object.get_string();
   |                         ------------ `object` moved due to this method call
15 |     println!("{}",object.get_string())
   |                   ^^^^^^ value used here after move
   |
note: this function takes ownership of the receiver `self`, which moves `object`
  --> src/main.rs:6:19
   |
6  |     fn get_string(self) -> String {
   |                   ^^^^

For more information about this error, try `rustc --explain E0382`.
error: could not compile `playground` due to previous error
*/
```
그래서 해결법을 몇시간들어 찾아봤는데, 문자열 타입이 Clone 트레이드(Copy트레이드랑 비슷한 기능) 구현을 지원한다고 한다.
즉 함수에서 clone()를 호출하면 해결된다.
``` rust
struct A {
    num : u32,
    s : String
}
impl A {
    fn get_string(&self) -> u32 {
        self.s.clone()
    }
/***/
}

```