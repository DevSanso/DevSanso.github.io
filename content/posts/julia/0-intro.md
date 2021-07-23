---
title: "0-'julia 이란?'"
date: 2021-07-23T18:39:55+09:00
draft: false
tags : [julia,프로그래밍 언어,기초]
---

최근들어 알고리즘, 영어, 수치 해석, 정보 처리 기사 공부들이 하기 힘들어서 구글링을 하며 시간을 보내던중,
 **스택오버플로우 2020 survey**에서 개발하고 있고, 또는 개발하고 싶은 언어 상위권을 보던도중 julia라는 새로보는 언어가 있는걸 알게되고, 요즘 공부도 별로 안되니 취미로써(?) 해당 언어를 배우기로 결정하였다.
   
 ![stackoverflow rank](/posts/julia/0/stackoverflow_rank.PNG)
 
 
### julia 언어
해당 언어에 대해 알아본 결과 수치해석에 사용하는 프로그래밍 언어로 보인다. 또한 공식 사이트에서 해당 언어의 장점이 설명되있는데 부족한 영어실력으로 발변역을 해본결과..
![julia website](/posts/julia/0/julia_web_intro.PNG)
 * Fast : 높은 성능에 맞쳐 디자인 하였으며, LLVM를 통해 효과적으로 멀티 플랫품에서 네이티브 코드로 컴파일 하였습니다.
 * Dynamic : 스크립트 언어처럼 동적타입 언어이다.
 * General : 비동기 I/O,메타 프로그래밍, 디버깅등을 제공한다.
  
  등 다른 장점들을 구루 갖춘 언어인걸 알수있다.

### 실행 환경
julia를 자습하기위해 설치를 해야하겠지만, 아직 기초만 배울 생각이고 계속 사용할지는 의문이라, 사용후 제거하기 편하고, 또한 설치가 편한(?) host(윈도우)가 아닌 docker로 리눅스 이미지 환경을 구축후 해당 환경에 설치하여 사용하기로 마음 먹었다.
![julia website](/posts/julia/0/julia_in_docker.PNG)
```
실행환경 : docker
이미지 : 페도라 34
```

앞으로 해당 언어를 자습하다가, 뭔가 새롭게 느껴지는게 있으면 복습 및 저장용으로 블로그에 글을 올릴 생각이다.

###### 참고
1. [stackoverflow 2020 developer survery](https://insights.stackoverflow.com/survey/2020)
2. [julia offical website](https://julialang.org/)