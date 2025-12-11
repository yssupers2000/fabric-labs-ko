---
lab:
    title: 'Generate batch predictions using a deployed model in Microsoft Fabric'
    module: 'Generate batch predictions using a deployed model in Microsoft Fabric'
---

# Generate batch predictions using a deployed model in Microsoft Fabric

이 랩에서는 머신러닝 모델을 사용하여 당뇨병의 정량적 측정값을 예측합니다.

이 랩을 완료하면 예측을 생성하고 결과를 시각화하는 실습 경험을 얻게 됩니다.

이 랩을 완료하는 데 약 **20**분이 소요됩니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 작업 영역을 생성합니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
1. 왼쪽 메뉴 바에서 **작업 영역**(아이콘은 &#128455;와 유사하게 생겼습니다.)을 선택합니다.
1. 원하는 이름으로 새 작업 영역을 생성하고, Fabric Capacity(*평가판*, *Premium* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Notebook 만들기

이 연습에서는 *Notebook*을 사용하여 모델을 학습하고 사용합니다.

1. 왼쪽 메뉴 바에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *데이터 과학* 섹션에서 **Notebook**을 선택합니다. 원하는 고유한 이름을 지정합니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않은 경우 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    몇 초 후에 단일 *셀*이 포함된 새 Notebook이 열립니다. Notebook은 *코드* 또는 *Markdown*(서식 있는 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

1. 첫 번째 셀(현재 *코드* 셀)을 선택한 다음, 셀의 오른쪽 상단에 있는 동적 도구 모음에서 **M&#8595;** 버튼을 사용하여 셀을 *Markdown* 셀로 변환합니다.

    셀이 Markdown 셀로 변경되면 포함된 텍스트가 렌더링됩니다.

1. 필요한 경우 **&#128393;** (편집) 버튼을 사용하여 셀을 편집 모드로 전환한 다음, 내용을 삭제하고 다음 텍스트를 입력합니다.

    ```text
   # Train and use a machine learning model
    ```

## 머신러닝 모델 학습

먼저, *회귀* 알고리즘을 사용하여 당뇨병 환자의 관심 반응(baseline 1년 후 질병 진행의 정량적 측정값)을 예측하는 머신러닝 모델을 학습시켜 보겠습니다.

1. Notebook에서 최신 셀 아래에 있는 **+ 코드** 아이콘을 사용하여 새 코드 셀을 Notebook에 추가합니다.

    > **팁**: **+ 코드** 아이콘을 보려면 마우스를 현재 셀 출력 바로 아래 왼쪽으로 이동합니다. 또는 메뉴 바의 **편집** 탭에서 **+ 코드 셀 추가**를 선택합니다.

1. 다음 코드를 입력하여 데이터를 로드하고 준비한 다음 모델을 학습하는 데 사용합니다.

    ```python
   import pandas as pd
   import mlflow
   from sklearn.model_selection import train_test_split
   from sklearn.tree import DecisionTreeRegressor
   from mlflow.models.signature import ModelSignature
   from mlflow.types.schema import Schema, ColSpec

   # Get the data
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r""
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   df = spark.read.parquet(wasbs_path).toPandas()

   # Split the features and label for training
   X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)

   # Train the model in an MLflow experiment
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
   with mlflow.start_run():
       mlflow.autolog(log_models=False)
       model = DecisionTreeRegressor(max_depth=5)
       model.fit(X_train, y_train)
       
       # Define the model signature
       input_schema = Schema([
           ColSpec("integer", "AGE"),
           ColSpec("integer", "SEX"),\
           ColSpec("double", "BMI"),
           ColSpec("double", "BP"),
           ColSpec("integer", "S1"),
           ColSpec("double", "S2"),
           ColSpec("double", "S3"),
           ColSpec("double", "S4"),
           ColSpec("double", "S5"),
           ColSpec("integer", "S6"),
        ])
       output_schema = Schema([ColSpec("integer")])
       signature = ModelSignature(inputs=input_schema, outputs=output_schema)
   
       # Log the model
       mlflow.sklearn.log_model(model, "model", signature=signature)
    ```

1. 셀의 왼쪽에 있는 **&#9655; 셀 실행** 버튼을 사용하여 셀을 실행합니다. 또는 키보드에서 **SHIFT** + **ENTER**를 눌러 셀을 실행할 수 있습니다.

    > **참고**: 이 세션에서 Spark 코드를 처음 실행하는 것이므로 Spark Pool이 시작되어야 합니다. 즉, 세션의 첫 번째 실행은 완료하는 데 약 1분 정도 걸릴 수 있습니다. 이후 실행은 더 빨라집니다.

1. 셀 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 새 코드 셀을 Notebook에 추가하고, 이전 셀에서 실험을 통해 학습된 모델을 등록하는 다음 코드를 입력합니다.

    ```python
   # Get the most recent experiement run
   exp = mlflow.get_experiment_by_name(experiment_name)
   last_run = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=1)
   last_run_id = last_run.iloc[0]["run_id"]

   # Register the model that was trained in that run
   print("Registering the model from run :", last_run_id)
   model_uri = "runs:/{}/model".format(last_run_id)
   mv = mlflow.register_model(model_uri, "diabetes-model")
   print("Name: {}".format(mv.name))
   print("Version: {}".format(mv.version))
    ```

    이제 모델이 **diabetes-model**로 작업 영역에 저장됩니다. 선택적으로 작업 영역의 찾아보기 기능을 사용하여 작업 영역에서 모델을 찾고 UI를 사용하여 탐색할 수 있습니다.

## Lakehouse에서 테스트 데이터 세트 만들기

모델을 사용하려면 당뇨병 진단을 예측해야 하는 환자 세부 정보 데이터 세트가 필요합니다. 이 데이터 세트를 Microsoft Fabric Lakehouse의 테이블로 생성합니다.

1. Notebook 편집기에서 왼쪽의 **탐색기** 창에 있는 **+ 데이터 원본**을 선택하여 Lakehouse를 추가합니다.
1. **새 Lakehouse**를 선택하고 **추가**를 선택한 다음, 원하는 유효한 이름으로 새 **Lakehouse**를 생성합니다.
1. 현재 세션을 중지하라는 메시지가 나타나면 **지금 중지**를 선택하여 Notebook을 다시 시작합니다.
1. Lakehouse가 생성되어 Notebook에 연결되면 새 코드 셀을 추가하고 다음 코드를 실행하여 데이터 세트를 생성하고 Lakehouse의 테이블에 저장합니다.

    ```python
   from pyspark.sql.types import IntegerType, DoubleType

   # Create a new dataframe with patient data
   data = [
       (62, 2, 33.7, 101.0, 157, 93.2, 38.0, 4.0, 4.8598, 87),
       (50, 1, 22.7, 87.0, 183, 103.2, 70.0, 3.0, 3.8918, 69),
       (76, 2, 32.0, 93.0, 156, 93.6, 41.0, 4.0, 4.6728, 85),
       (25, 1, 26.6, 84.0, 198, 131.4, 40.0, 5.0, 4.8903, 89),
       (53, 1, 23.0, 101.0, 192, 125.4, 52.0, 4.0, 4.2905, 80),
       (24, 1, 23.7, 89.0, 139, 64.8, 61.0, 2.0, 4.1897, 68),
       (38, 2, 22.0, 90.0, 160, 99.6, 50.0, 3.0, 3.9512, 82),
       (69, 2, 27.5, 114.0, 255, 185.0, 56.0, 5.0, 4.2485, 92),
       (63, 2, 33.7, 83.0, 179, 119.4, 42.0, 4.0, 4.4773, 94),
       (30, 1, 30.0, 85.0, 180, 93.4, 43.0, 4.0, 5.3845, 88)
   ]
   columns = ['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']
   df = spark.createDataFrame(data, schema=columns)

   # Convert data types to match the model input schema
   df = df.withColumn("AGE", df["AGE"].cast(IntegerType()))
   df = df.withColumn("SEX", df["SEX"].cast(IntegerType()))
   df = df.withColumn("BMI", df["BMI"].cast(DoubleType()))
   df = df.withColumn("BP", df["BP"].cast(DoubleType()))
   df = df.withColumn("S1", df["S1"].cast(IntegerType()))
   df = df.withColumn("S2", df["S2"].cast(DoubleType()))
   df = df.withColumn("S3", df["S3"].cast(DoubleType()))
   df = df.withColumn("S4", df["S4"].cast(DoubleType()))
   df = df.withColumn("S5", df["S5"].cast(DoubleType()))
   df = df.withColumn("S6", df["S6"].cast(IntegerType()))

   # Save the data in a delta table
   table_name = "diabetes_test"
   df.write.format("delta").mode("overwrite").saveAsTable(table_name)
   print(f"Spark dataframe saved to delta table: {table_name}")
    ```

1. 코드가 완료되면 **Lakehouse 탐색기** 창에서 **테이블** 옆의 **...**을 선택하고 **새로 고침**을 선택합니다. **diabetes_test** 테이블이 나타나야 합니다.
1. 왼쪽 창에서 **diabetes_test** 테이블을 확장하여 포함된 모든 필드를 확인합니다.

## 예측을 생성하기 위해 모델 적용

이제 이전에 학습한 모델을 사용하여 테이블에 있는 환자 데이터 행에 대한 당뇨병 진행 예측을 생성할 수 있습니다.

1. 새 코드 셀을 추가하고 다음 코드를 실행합니다.

    ```python
   import mlflow
   from synapse.ml.predict import MLFlowTransformer

   ## Read the patient features data 
   df_test = spark.read.format("delta").load(f"Tables/{table_name}")

   # Use the model to generate diabetes predictions for each row
   model = MLFlowTransformer(
       inputCols=["AGE","SEX","BMI","BP","S1","S2","S3","S4","S5","S6"],
       outputCol="predictions",
       modelName="diabetes-model",
       modelVersion=1)
   df_test = model.transform(df)

   # Save the results (the original features PLUS the prediction)
   df_test.write.format('delta').mode("overwrite").option("mergeSchema", "true").saveAsTable(table_name)
    ```

1. 코드가 완료되면 **Lakehouse 탐색기** 창에서 **diabetes_test** 테이블 옆의 **...**을 선택하고 **새로 고침**을 선택합니다. 새 필드 **predictions**가 추가됩니다.
1. Notebook에 새 코드 셀을 추가하고 **diabetes_test** 테이블을 해당 셀로 끌어옵니다. 테이블 내용을 볼 수 있는 필요한 코드가 나타납니다. 셀을 실행하여 데이터를 표시합니다.

## 리소스 정리

이 연습에서는 모델을 사용하여 배치 예측(batch predictions)을 생성했습니다.

Notebook 탐색을 완료했다면 이 연습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.