---
title: "모듈 임포트"
date: 2021-11-04T22:14:44+09:00
tag : [tip,rust]
draft: true
---


러스트로 코딩하는도중, 소스코드를 분리할 필요성을 느껴 별도의 모듈로 만들고나서 임포트 할려고 할때, 해당 프로젝트안의
로컬 모듈들을 어떻게 임포트 하는지, 기억하기 힘들어 따로 이렇게 글을 작성할 필요성을 느꼇다.  

러스트에서는 mod라는 키워드가 있는데, 해당 키워드는 두가지의 기능을 수행한다.
1. c++.c#처럼 네임스페이스 비슷한 기능을 한다.
2. 다른 모듈들을 인클루드 한다.
예시로 이런 코드가 있다고 치자
``` rust
// path : src/main.rs

/* similar c++
namespace module {
    void hello() {
        std::cout << "hello world" << std::endl;
   }
};  

int main() {
    module::hello();
    return 0;
}
*/
mod module {
    pub fn hello() {
        println!("hello world");
    }
}


fn main() {
    module::hello();
}
```
해당 소스코드는 module이라는 모듈안의 hello 함수를 호출하는 소스코드 이다.  
이렇게 보면 네임스페이스랑 비슷하게 보이기도 한다. 허나 중요한건 이것이 아닌, 만약
module라는 모듈이 빈내용이면 어떻게 될까?
``` rust
mod module;

fn main() {}
```

```
//결과창
error[E0583]: file not found for module `module`
 --> src/main.rs:1:1
  |
1 | mod module;
  | ^^^^^^^^^^^
  |
  = help: to create the module `module`, create file "src/module.rs" or "src/module/mod.rs"

For more information about this error, try `rustc --explain E0583`.
error: could not compile `playground` due to previous error
```
경고창의 일부 내용에서 create file "src/module.rs" or "src/module/mod.rs" 나오는걸 볼때, 빈 mod이면
다른 파일을 임포트하는것을 볼수있다.  

즉 만약 소스코드를 모듈단위로 분리하고 싶다면 mod를 이용하는것이 좋다.
#### 참고
1. [File hierarchy](https://doc.rust-lang.org/rust-by-example/mod/split.html)
2. [keyword](https://doc.rust-lang.org/std/keyword.mod.html)