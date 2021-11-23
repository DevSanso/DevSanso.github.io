---
title: "julia-'dot 연산자'"
date: 2021-07-26T20:24:39+09:00
draft: false
tags : [julia,기초]
---

julia에 dot 연산자라고, 배열안의 요소별로 각각 연산해주는것이 있다.
예시로 백터를 생성후 요소값들을 각각 하나씩 더하고 싶을때 보통은 for문등 방법을 사용하는게 보통이다.

```
julia> x = [1,2,3,4,5]

#output
#5-element Vector{Int64}:
# 1
# 2
# 3
# 4
# 5

julia> i = 1

julia> while i <= 5
       x[i] = x[i]+2
       global i += 1
       end

julia> x
#output
#5-element Vector{Int64}:
# 3
# 4
# 5
# 6
# 7

```

허나 julia에서는 dot 연산자를 사용해 한줄코드로 간단하게 사용할수 있다.
```
julia> x .+ 2
#output
#5-element Vector{Int64}:
# 3
# 4
# 5
# 6
# 7
```

또한 julia에서는 "@."라는 매크로가 존재하는데, 계산식에 많은 dot(.)이 있을경우 표현식의 길이가 길어질수 있는데, 해당 매크로를 사용하면 dot를 각 연산자마다 사용할 필요가 없어져서, 표현식이 더 간결해질수 있다.

```
julia> x = [1,2,3]
#output
#3-element Vector{Int64}:
# 1
# 2
# 3

julia> 2 .* x.^2 .+ sin.(x)
#output
#3-element Vector{Float64}:
#  2.8414709848078967
#  8.909297426825681
# 18.14112000805987

julia> @. 2 * x^2 + sin(x)
#output
#3-element Vector{Float64}:
#  2.8414709848078967
#  8.909297426825681
# 18.14112000805987

```


##### 주의
```
#자세한 이유는 모르나 인덱스를 1부터 시작하는걸로 보인다
#인텍스를 0으로 해서 첫번째 요소를 읽을라고 하면 오류가 발생한다.

julia> x[0]
#output

#ERROR: BoundsError: attempt to access 5-element Vector{Int64} at index [0]
#Stacktrace:
# [1] getindex(A::Vector{Int64}, i1::Int64)
#   @ Base ./array.jl:801
# [2] top-level scope
#   @ REPL[38]:1
```

### 참고
[매뉴얼](https://docs.julialang.org/en/v1/manual/mathematical-operations/)