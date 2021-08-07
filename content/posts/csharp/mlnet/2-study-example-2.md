---
title: "c#-'mlnet 코드 예제로 공부2"
date: 2021-08-07T15:36:05+09:00
draft: true
tags : [csharp,프로그래밍 언어,mlnet,기계 학습]
---
###### 참고
전내용 : [mlnet 코드 예제로 공부1](1-study-code-example.md)

메인문에 있던 LoadData는 TrainTestData 구조체를 리턴하고, MLContext 인자로 받는 함수이다.
```C#
public static TrainTestData LoadData(MLContext mlContext)
```
그리고 공식 자습서에서 나오는 해당 함수의 작업은
* 데이터를 로드합니다
* 로드된 데이터 세트를 학습 및 테스트 세트로 분할합니다.
* 분할된 학습 및 테스트 데이터 세트를 반환합니다.

그럼 코드를 살펴보면서 어디서 해당 작업이 이루어 지는지 확인을 해보겠다.
우선 첫줄이 이렇게 생겼는데.
```C#
//IDataView : ML 데이터를 표현하기 위한 인터페이스
IDataView dataView = mlContext.Data.LoadFromTextFile<SentimentData>(_dataPath, hasHeader: false);
```
우선 mlcontext.Data의 원형 클래스는 [DataOperationsCatalog](https://docs.microsoft.com/ko-kr/dotnet/api/microsoft.ml.dataoperationscatalog?view=ml-dotnet)이다.  
 그럼 Data에서 LoadFromTextFile 함수를 호출하니 해당 객체에 함수가 정의 되어있다고 생각하지만,사실 문서를 보면 해당 함수에 대한 내용이 없다,사실 LoadFromTextFile는 확장메서드로써 Microsoft.ML 네임스페이스에 있는 [TextLoaderSaverCatalog](https://docs.microsoft.com/ko-kr/dotnet/api/microsoft.ml.textloadersavercatalog?view=ml-dotnet)에 정의 되어 있다. 그다음

 ```C#
 //TrainTestData : 학습 및 테스트 집합에 대 한 데이터 집합 쌍입니다.
 //앞으로 학습 및 평가할때 사용할 구조체입니다.
 TrainTestData splitDataView = mlContext.Data.TrainTestSplit(dataView, testFraction: 0.2);
 ```
TrainTestSplit함수를 통해 TrainTestData를 생성후 해당 값을 리턴함으로써 LoadData 함수가 종료됩니다.
```C#
        //완성본
        public static TrainTestData LoadData(MLContext mlContext)
        {
            IDataView dataView = mlContext.Data.LoadFromTextFile<SentimentData>(_dataPath, hasHeader: false);
            TrainTestData splitDataView = mlContext.Data.TrainTestSplit(dataView, testFraction: 0.2);
            return splitDataView;
        }
```
이렇게 LoadData함수를 살펴보았다. 그다음은 모델 빌드 및 학습하는 BuildAndTrainModel 함수를 알아볼 생각이다.