---
title: "비교연산자가 포함된 if문을 비트연산으로 변화"
date: 2021-07-21T21:04:42+09:00
draft: false
tags : [tip,정리노트]
---

여가시간에 프로그래밍 정보를 얻어보고자 [stackoverflow](https://stackoverflow.com/)에서 상당히 좋은글을 보게 되었다.

보통 코딩을 할때 **"n개의 정수배열에서 y 보다 작거나 같은 Xn을 서로 합하여라"** 라는 문제가 있을경우 보통 이렇게 코드를 작성하게 된다.
```{.cpp} 
//cpp 예시코드
for(int i;i<=array_size;i++)
{
    int x = array[i];
    if(x <= y)
    {
        sum += x;
    }
}      
```
이렇게 해당 소스코드를 작성할것이다.

해당 소스코드는 1개의 if,한개의 비교연산자를 이용하여 **x가 y보다 작거나 같으면, sum 변수에 더한다** 라는 간단한 기능이 구현되어있다,그럼 여기서 어떻게 해야 성능 최적화가 되는건가?그것에 대한 답은 바로 비트연산에 있다.

```
※비트연산이란?

비트 연산(bitwise operation)은 한 개 혹은 두 개의 이진수에 대해 비트 단위로 적용되는 연산이다.

```





#### 일단 해답 코드를 이해하기 위해 사전정보로써 다음과 같은정보를 알고 있어야한다.

* (x < y)에서 x-y를 하면 값은 음수다.
* 32비트 정수에서 음수값을 right shift로 31번 쉬프트하면 32비트들이 다 1로 바낀다.
* 32비트 정수에서 양수값을 right shift로 31번 쉬프트하면 32비트들이 다 0으로 바낀다.
* not 비트연산자를 사용하면 비트값이 반전된다
* and 비트연산자는 두개의 비트값이 1일때, 결과값이 1이다.

다 해당 정보들을 외워놨으면 다음 코드가 이해할수 있을거다.

```{.cpp} 
//cpp 예시코드
{
    int tmp = (y-x) >> 31;
    /*
    만약 tmp의 비트값이 전부 1이면 not연산자로 인해 0로 전부 바끼며,
    0으로 and연산 하면 결과값이 0임으로 결국 sum에 0을 더한다.
    즉 아무것도 더하기 않은것과 동일한 결과가 나온다. 
    */ 
    sum += ~tmp & x;
}      
```
해당 코드는 if문도 없고,비교연산자가 없지 작동이 가능하지만, 그것만으로 성능이 더 좋아질수 있을까라고 생각할것이다. 그래서 다음같은 조건으로 테스트를 해보았다.


##### 테스트 기기
```
기기이름 : 라즈베리파이4 모델 B
CPU : 1.5GHz ARM Cortex-A72 MP4
RAM : 8GB LPDDR4

```

##### 프로그램 조건
* 10000000개의 정수배열
* 배열의 각 요소값은 랜덤
* 1562값보다 작을경우 더하여, 출력


#### if문을 사용한 코드
```{.c}
#include<stdio.h>
#include<time.h>
#include<stdlib.h>


const int  count = 10000000;
const int cut = 1562;

int main()
{
    srand(time(NULL));
    clock_t s,l;
    int i = 0;
    int sum = 0;
    int *datas = (int*)malloc(sizeof(int)*count);
    for(;i<count;i++)
    {
        datas[i] = rand();
    }
    i=0;

    s = clock();
    
    //-----------테스트  코드 --------------
    
    do {
        i++;
        if(datas[i] <= cut)
            sum += datas[i];
    }while(i<count);
    //-------------------------------------
    l = clock();

    printf("time : %0.3lf(sec) ",
                    (double)(l-s)/CLOCKS_PER_SEC);
    printf("sum : %d\n",sum);

    return 0;
}
```
##### 결과
![result](/posts/optimizing_the_if_less_operator/use_if.png)

#### 비트연산을 사용한 코드
```
#include<stdio.h>
#include<time.h>
#include<stdlib.h>


const int count = 10000000;
const int cut = 1562;

int main()
{
    srand(time(NULL));
    clock_t s,l;
    int i = 0;
    int sum = 0;
    int *datas = (int*)malloc(sizeof(int)*count);
    int temp=0;

    for(;i<count;i++)
    {
        datas[i] = rand();
    }
    i=0;

    s = clock();
    //-------------- 테스트 코드 --------------------
    do {
        i++;
        /*
        / same
        / if(datas[i] < cut)
        /    sum += datas[i];
        */
        temp = (cut - datas[i]) >> 31;
        sum += ~temp & datas[i];
    }while(i<count);
    //-----------------------------------------------
    l = clock();
    printf("time : %0.3lf(sec) ",
                        (double)(l-s)/CLOCKS_PER_SEC);
    printf("sum : %d\n",sum);

    return 0;
}
```
##### 결과
![result](/posts/optimizing_the_if_less_operator/use_bit.png)

##### 결론
테스트 결과 대략 5배정도 차이나는 놀라운 성능 최적화를 보여주였다.
이렇게 비트 연산을 사용하여 최적화한 글을 적어보면서, 프로그램 최적화는 끝은 없다고 생각하게 되었다. 이렇게 코드를 최적화하는 방법을 찾아보며, 포스트글을 적으면서 공부하는것도 나쁘지 않다고 생각하며 해당글은 여기서 마치겠습니다.


### 참고
비트연산이란? : [위키백과](https://ko.wikipedia.org/wiki/%EB%B9%84%ED%8A%B8_%EC%97%B0%EC%82%B0)  
라즈베리파이4 모델 b 사양 : [공식사이트](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/specifications/)  
해당 게시글(정확한 주제는 분기예측이다) : [stack  overflow](https://stackoverflow.com/questions/11227809/why-is-processing-a-sorted-array-faster-than-processing-an-unsorted-array)
