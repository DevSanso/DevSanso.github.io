---
title: "c#-'mlnet 코드 예제로 공부1'"
date: 2021-08-05T23:03:51+09:00
draft: false
tags : [csharp,프로그래밍 언어,mlnet,기계 학습]
---

mlnet를 공부하기위해 사이트를 들어갔더니, 본인이 공부할수 있는 양을 넘어셔서 코드줄을 하나씩 분석하면서 공부하기로 마음먹게 되었다.  
  

##### 해당 글 사이트 주소
[자습서: ML.NET에서 이진 분류를 사용하여 웹 사이트 주석의 감정 분석](https://docs.microsoft.com/ko-kr/dotnet/machine-learning/tutorials/sentiment-analysis)

우선 해당 사이트에선 첫번째  소스코드에선 라이브러리를 임포트하라고 하는데, 하나씩 살펴볼려고 한다.(System 네임스페이스 모듈들은 기본 지원 라이브러리니 따보 살펴보진 않을 예정이다.)
```C#
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Microsoft.ML;
using Microsoft.ML.Data;
using static Microsoft.ML.DataOperationsCatalog;
// 모든 코드를 작성해본 결과 해당 2개의 모듈들은 사용되지 않은 모양이라 주석 처리 하였습니다.
//using Microsoft.ML.Trainers;
//using Microsoft.ML.Transforms.Text;
```
|네임스페이스|설명|
|------|:---|
|[Microsoft.ML](https://docs.microsoft.com/ko-kr/dotnet/api/microsoft.ml?view=ml-dotnet)|프로그램이 데이터 처리 및 작업 처리에 관련된 기본적인 모듈로 보인다.|
|[Microsoft.ML.Data](https://docs.microsoft.com/ko-kr/dotnet/api/microsoft.ml.data?view=ml-dotnet)|디스크에 저장된 데이터 불려오기 또는 데이터를 디스크에 저장, 데이터 구조를 정의하는데 관련된 모듈로 보인다|
|[Microsoft.ML.DataOperationsCatalog](https://docs.microsoft.com/ko-kr/dotnet/api/microsoft.ml.dataoperationscatalog?view=ml-dotnet)|데이터뷰 객체를 좀더 쉽게 다루기 위해 있는 클래스인것같다,TrainTestData 구조체를 사용할려고 임포트 한듯하다.|

###### 참고
[using 정적 지시문](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/using-static)

그리고 해당 예제소스를 따라가다보면 정적 변수랑, 메인문,클래스가 이렇게 작성되있을거다.
```C#

    class SentimentData
    {
        [LoadColumn(0)]
        public string SentimentText;
        [LoadColumn(1),ColumnName("Label")]
        public bool Sentiment;
    }

    class SentimentPrediction : SentimentData
    {
        [ColumnName("PredictedLabel")]
        public bool Prediction{get;set;}
        public float Probability {get;set;}
        public float Score {get;set;}
    }


        static readonly string _dataPath = Path.Combine(Environment.CurrentDirectory, 
        "Data", "yelp_labelled.txt");


        public static void Main(String[] args) 
        {
            MLContext mlContext = new MLContext();
            TrainTestData splitDataView = LoadData(mlContext);
            ITransformer model = BuildAndTrainModel(mlContext, splitDataView.TrainSet);
            Evaluate(mlContext, model, splitDataView.TestSet);
            UseModelWithBatchItems(mlContext, model);

        }
```
메인함수에서 MLContext라는 클래스가 있는데, 모든 MlNet 의 처리과정은 해당 컨텍스트를 통해 처리된다는걸 알면 된다.

~~~
@추가 설명
앞으로 사용할 MLContext에 있는 Data변수,Transforms변수들의 원형 클래스들은 생성자가 internal로 접근제한이 되있기 때문에,   
사용자가 의도적으로 별도로 객체를 생성이 불가능하다.  
그렇기에 결국 MLContext로만 해당 객체들의 접근이 가능하다.
~~~

그리고 코드내의  LoadData, BuildAndTrainModel, Evalutae, UseModeWithBatchItems함수들은 다음에 하나씩 살펴볼 예정이다.