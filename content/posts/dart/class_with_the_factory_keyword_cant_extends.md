---
title: "factory 키워드를 사용하여 하나의 생성자만 있는 클래스는 확장이 안된다."
date: 2021-12-19T16:47:14+09:00
draft: false
tags : [dart,기초,정리노트]
---

최근 플러터를 통하여 앱 프로젝트를 만들고 코딩하고있을때 있던 일이였다.  
어플에 http통신을 구현할려고 하는데, dart언어에 기본적으로 httpclient 객체가 지원하고 있었다. 
그래서 해당 객체의 자식클래스를 만들어서 코딩할려고 할려고 하니 갑자기 에러창이 뜨게 되었다.
``` .dart

import "dart:io";


class Custom extends HttpClient {/*****/}

//error message
/*The class 'Custom' cannot extend 'HttpClient' because 'HttpClient' only has factory constructors (no generative constructors), and 'Custom' has at least one generative constructor.
Try implementing the class instead, adding a generative (not factory) constructor to the superclass Custom, or a factory constructor to the subclass.*/
```
HttpClient클래스는 추상클래스이고, 추상 메소드들도 있기에, 상속하고나서 추상메소드들도 구현했는데도 저런 에러메세지가 나왔다. 
그래서 메세지를 토대로 찾아보아 factory에 알아봤다.  
일단 factory 키워드가 붙으면 해당문은 생성자로 취급되는데,이때 해당 클래스가 factory생성자 단 한개라면 상속이 되지 않는다.  
즉 HttpClient클래스는 factory 생성자가 한개뿐이라 상속이 안되었던것, 그렇기에 만약 해당 클래스랑 어떠한 관계를 만들고 싶다면 
with 혹은 implements를 사용하는 방법, 흑은 해당 클래스가 다른 생성자를 만드는방법밖에 없는거 같다.  
허나 해당 클래스가 내가 만든게 아닌 기본적으로 지원하는 클래스이기에 with 그리고 implements를 사용하든지, 아니면 그냥 상속관계를 포기하고 
맴버변수로 사용하는방법으로 가는걸로 선택해야할것같다.

참고 : [dart 공식문서 ](https://dart.dev/guides/language/language-tour#factory-constructors)