---
title: "mlnet"
date: 2021-08-02T23:55:26+09:00
draft: true
---

훗날 학교에서 c# 프로그래밍을 하는 과목이 있다고 들어, 복습 겸 공부해볼 라이브러리를 찾는도중, ML.NET이라는 net core에서 작동하는 기계학습 라이브러리가 있다고 해서 알아볼라고 한다.

그러기 위해서 우선 공부하기 위한 프로젝트 폴더를 생성해야하는데, 현재 사용하는 환경이 리눅스이다 보니, 리눅스 관점에서 서술할려고한다.

일단 윈도우 환경이 아니기 때문에 visual studio 사용이 안된다(...), 그렇기에 보통 cli 툴을 먼저 설치하기 위해 netcore 사이트가서 설치해야하지만, 다행스럽게도 내가 사용하는 아치 기반 리눅스의 저장소에서  netnore 툴이 있었기에 저장소에서 손쉽게 설치를 하였다

```
#아치 리눅스 사용자를 이 명령어로 설치하자
sudo pacman -Sy dotnet-sdk
```

그리고 c# 코드를 작성할때 쓸 비주얼 스튜디오 코드도 설치하자
```
#pacman에서 설치하면 일부 플러그인을 사용할수 없는걸 주의해야 한다.
yay -Sy visual-studio-code-bin
```

그리고 터미널을 키고 원하는 폴더에 프로젝트를 생성하고, 프로젝트에 ml.net 라이브러리를 설치한후 비주얼 스튜디오 코드로 들어가면 된다.
```
#프로젝트 생성
dotnet new console -o mlnet
cd mlnet
#이 명령어로 ml 패키지를 설치하자
dotnet add package Microsoft.ML


```

그럼 이제  예제 소스 코드를 작성하고, 작동이 가능한지 알기 위해 직접 실행해보았다.

```C#
 using System;
 using Microsoft.ML;
 using Microsoft.ML.Data;

   class Program
   {
       public class HouseData
       {
           public float Size { get; set; }
           public float Price { get; set; }
       }

       public class Prediction
       {
           [ColumnName("Score")]
           public float Price { get; set; }
       }

       static void Main(string[] args)
       {
           MLContext mlContext = new MLContext();

           // 1. Import or create training data
           HouseData[] houseData = {
               new HouseData() { Size = 1.1F, Price = 1.2F },
               new HouseData() { Size = 1.9F, Price = 2.3F },
               new HouseData() { Size = 2.8F, Price = 3.0F },
               new HouseData() { Size = 3.4F, Price = 3.7F } };
           IDataView trainingData = mlContext.Data.LoadFromEnumerable(houseData);

           // 2. Specify data preparation and model training pipeline
           var pipeline = mlContext.Transforms.Concatenate("Features", new[] { "Size" })
               .Append(mlContext.Regression.Trainers.Sdca(labelColumnName: "Price", maximumNumberOfIterations: 100));

           // 3. Train model
           var model = pipeline.Fit(trainingData);

           // 4. Make a prediction
           var size = new HouseData() { Size = 2.5F };
           var price = mlContext.Model.CreatePredictionEngine<HouseData, Prediction>(model).Predict(size);

           Console.WriteLine($"Predicted price for size: {size.Size*1000} sq ft= {price.Price*100:C}k");

           // Predicted price for size: 2500 sq ft= $261.98k
       }
   }
```

# 실행결과


![result](posts/csharp/mlnet/0/result.png)


