---
title: "julia-'이름 규칙'"
date: 2021-07-24T12:36:38+09:00
draft: false
tags : [julia,프로그래밍 언어,기초]
---

julia 공식 매뉴얼에서 변수 관련 문서를 보고 코드를 작성하면서 몇가지 특징을 알게 되었다.

첫째로 요즘 언어들도 갖고있는 특성이지만 유니코드를 지원하여, 한글로 변수명이 가능하다는것과 심지어 특수기호로도 변수명이 되는걸 알수있다.
![exmaple](/posts/julia/1/unicode_name.PNG)

그렇다고 해서 한글 또는 특수기호로 변수명을 작성해서 사용하는건 비추천한다, 왜냐하면 julia는 이름 규칙이 따로 존재하기 때문이다.

### julia 명명 규칙
###### (발번역 주의)
1. 변수명은 소문자로 한다
   ```
   #example
   name = "Sunken Ahn"
   ```
2. 단구 구분은 언더바(_)로 구분, 단 이름이 읽기 어려운게 아니면 사용은 자세
   ```
   #example
   my_name = "Sunken Ahn"
   ```
3. 모듈,타입은 단어 시작을 대문자로, 단어구분도 단어 첫글자를 대문자로 하여 구분
   ```
   #example
   struct Foo
       bar
       baz
   end
   foo = Foo(1,2)
   ```
4. 함수, 매코르는 언더바(_)없이 소문자
   ```
   #example
   function f(x,y)
       x + y
   end
   ```
5. 인수로 쓰는 함수("mutating" or "in-place" functions)는 이름이 !로 끝나야 한다
   ```
   #example
   function first_element_change_one!(v::Vector{Int64})
       v[0] = 1
   end
   ```

#### 참고
매뉴얼 : [ variables](https://docs.julialang.org/en/v1/manual/variables/)