---
title: "c#10에서 추가된거"
date: 2021-11-19T23:27:30+09:00
draft: false
tags : [뉴스,c#]
---
이번 11월에 netcore 6이 정식버젼이 배포되면서 c#이 10으로 업그레이드 됬다고 하여,
무엇이 바끼었나 보려 가보았다.

## global using 추가
``` cs
//example
global using System;
```
해당 지시문이 추가되면, 프로젝트 안에서 어느 파일이든 바로 using이 적용된다. 
또한 static, 별칭 지시문도 똑같이 적용이 가능한다고 하낟.
```
global using static System.Console;
global using Env = System.Environment;
```
또한 해당 지시문이 어떤 파일에 있든 상관없이 똑같이 적용되기에, 해당 게시글에선
Program.cs 혹은 globalusings.cs을 생성하여 그곳에 추가하려 권장하고 있다.


## record structs

record라고 c#9에서 추가된 기능에서 좀더 다른 기능이 추가된걸로 보인다.  
살펴보니 코틀린에 있는 data class랑 비슷한 역할을 하는걸로 보인다.
``` cs
//c#10 이전
public record struct Car
{
    public string Company { get; init; }
    public string Name { get; init; }
}
//c#10 이후
public record struct Car(string Company,string Name);
``` 
자세한건 [record](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) 문서에서
확인할수 있다.  
  
지금 c#보단 다른걸 하다보니, 지금으로썬 이해가 쉽게되고, 흥미로운 기능들은 저거 2개이다.  
물론 여러 기능들도 생겼다
1. #### Extended property patterns (c#8에 추가된 속성 패턴기능 확장)
2. #### Parameterless struct constructors and field initializers (init 접근자 초기화를 생성자에서 초기화하는게 아닌,필드 초기화시점에서 할수 있게 한거)
3. #### File-scoped namespaces (자세히는 모르겠지만, 네임스페이스를 괄호없이 사용하면서, 클래스 정의도 할수 있게 하는것같다.)
4. #### Natural types for lambdas (잘 몰랐던 사실인데, 원래 람다를 변수에 대입할때 델리게이트 혹은 Func등을 사용해야하는걸 이제 var 키워드도 가능하게 한거같다)
허나 내가 c#을 처음 봤을때가 6버젼일때고, 그 6버젼조차 다 모르는 마당에 이렇게 여러 기능들이 추가되니, 지금이라도 복습하지 않으면 
앞으로 c#을 쳐다보기도 매우 힘든 언어가 될것같다. 말 그대로(..)



더 많은 내용은 [net core6의 새로운점](https://devblogs.microsoft.com/dotnet/announcing-net-6/)에 다한 문서를 보면 될것이다.