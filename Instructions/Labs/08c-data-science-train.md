---
lab:
    title: 'Train and track machine learning models with MLflow in Microsoft Fabric'
    module: 'Train and track machine learning models with MLflow in Microsoft Fabric'
---

# Microsoft Fabric에서 MLflow를 사용하여 머신러닝 모델 훈련 및 추적

이 실습에서는 당뇨병의 정량적 측정값을 예측하기 위해 머신러닝 모델을 훈련합니다. scikit-learn으로 회귀 모델을 훈련하고, MLflow를 사용하여 모델을 추적하고 비교합니다.

이 실습을 완료함으로써 머신러닝 및 모델 추적에 대한 실습 경험을 얻고, Microsoft Fabric에서 *노트북*, *실험 (Experiment)* 및 *모델* 작업 방법을 학습합니다.

이 실습은 완료하는 데 약 **25**분 정도 소요됩니다.

> **Note**: 이 연습을 완료하려면 [Microsoft Fabric 평가판 (trial)](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 워크스페이스 (workspace) 만들기

Fabric에서 데이터를 사용하기 전에 Fabric 평가판 (trial)이 활성화된 워크스페이스를 만드세요.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
1. 왼쪽 메뉴 모음에서 **Workspaces** (아이콘은 &#128455;와 유사하게 생겼습니다)를 선택합니다.
1. 원하는 이름으로 새 워크스페이스를 만들고, Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium* 또는 *Fabric*)를 선택합니다.
1. 새 워크스페이스가 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## 노트북 만들기

모델을 훈련하려면 *노트북*을 만들 수 있습니다. 노트북은 여러 언어로 코드를 작성하고 실행할 수 있는 대화형 환경을 제공합니다.

1. 왼쪽 메뉴 모음에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *데이터 과학* 섹션 아래에서 **Notebook**을 선택합니다. 원하는 고유한 이름을 지정하세요.

    >**Note**: **만들기** 옵션이 사이드바에 고정되어 있지 않으면 먼저 줄임표 (**...**) 옵션을 선택해야 합니다.

    몇 초 후에 단일 *셀*을 포함하는 새 노트북이 열립니다. 노트북은 *코드* 또는 *마크다운* (서식 있는 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

1. 첫 번째 셀 (현재 *코드* 셀임)을 선택한 다음, 오른쪽 상단의 동적 도구 모음에서 **M&#8595;** 버튼을 사용하여 셀을 *마크다운* 셀로 변환합니다.

    셀이 마크다운 셀로 변경되면 포함된 텍스트가 렌더링됩니다.

1. 필요한 경우 **&#128393;** (편집) 버튼을 사용하여 셀을 편집 모드로 전환한 다음, 내용을 삭제하고 다음 텍스트를 입력합니다.

    ```text
   # Train a machine learning model and track with MLflow
    ```

## 데이터를 데이터프레임 (dataframe)으로 로드

이제 데이터를 가져오고 모델을 훈련하기 위한 코드를 실행할 준비가 되었습니다. Azure Open Datasets의 [당뇨병 데이터셋](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)으로 작업합니다. 데이터를 로드한 후, 데이터를 Pandas 데이터프레임으로 변환합니다. Pandas 데이터프레임은 행과 열의 데이터를 다루는 일반적인 구조입니다.

1. 노트북에서 가장 최근 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 노트북에 새 코드 셀을 추가합니다.

    > **Tip**: **+ 코드** 아이콘을 보려면 마우스를 현재 셀의 출력 바로 아래 왼쪽으로 이동하세요. 또는 메뉴 모음의 **편집** 탭에서 **+ 코드 셀 추가**를 선택하세요.

1. 여기에 다음 코드를 입력합니다.

    ```python
   # Azure storage access info for open dataset diabetes
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r"" # Blank since container is Anonymous access
    
   # Set Spark config to access  blob storage
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   print("Remote blob path: " + wasbs_path)
    
   # Spark read parquet, note that it won't load any data yet by now
   df = spark.read.parquet(wasbs_path)
    ```

1. 셀 왼쪽의 **&#9655; 셀 실행** 버튼을 사용하여 실행합니다. 또는 키보드에서 **SHIFT** + **ENTER**를 눌러 셀을 실행할 수 있습니다.

    > **Note**: 이번 세션에서 Spark 코드를 처음 실행하는 경우 Spark 풀을 시작해야 합니다. 즉, 세션에서 첫 번째 실행은 완료하는 데 1분 정도 소요될 수 있습니다. 이후 실행은 더 빨라집니다.

1. 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 노트북에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   display(df)
    ```

1. 셀 명령이 완료되면 셀 아래의 출력을 검토합니다. 출력은 다음과 유사해야 합니다.

    |AGE|SEX|BMI|BP|S1|S2|S3|S4|S5|S6|Y|
    |---|---|---|--|--|--|--|--|--|--|--|
    |59|2|32.1|101.0|157|93.2|38.0|4.0|4.8598|87|151|
    |48|1|21.6|87.0|183|103.2|70.0|3.0|3.8918|69|75|
    |72|2|30.5|93.0|156|93.6|41.0|4.0|4.6728|85|141|
    |24|1|25.3|84.0|198|131.4|40.0|5.0|4.8903|89|206|
    |50|1|23.0|101.0|192|125.4|52.0|4.0|4.2905|80|135|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    출력은 당뇨병 데이터셋의 행과 열을 보여줍니다.

1. 데이터는 Spark 데이터프레임으로 로드됩니다. scikit-learn은 입력 데이터셋이 Pandas 데이터프레임일 것으로 예상합니다. 데이터셋을 Pandas 데이터프레임으로 변환하려면 아래 코드를 실행합니다.

    ```python
   import pandas as pd
   df = df.toPandas()
   df.head()
    ```

## 머신러닝 모델 훈련

이제 데이터를 로드했으므로 이 데이터를 사용하여 머신러닝 모델을 훈련하고 당뇨병의 정량적 측정값을 예측할 수 있습니다. scikit-learn 라이브러리를 사용하여 회귀 모델을 훈련하고 MLflow로 모델을 추적합니다.

1. 데이터를 훈련 및 테스트 데이터셋으로 분할하고, 예측하려는 레이블에서 특성 (features)을 분리하려면 다음 코드를 실행합니다.

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. 노트북에 새 코드 셀을 하나 더 추가하고 다음 코드를 입력한 후 실행합니다.

    ```python
   import mlflow
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
    ```

    이 코드는 **experiment-diabetes**라는 이름의 MLflow 실험 (Experiment)을 생성합니다. 모델은 이 실험 (Experiment)에서 추적됩니다.

1. 노트북에 새 코드 셀을 하나 더 추가하고 다음 코드를 입력한 후 실행합니다.

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    
      mlflow.log_param("estimator", "LinearRegression")
    ```

    이 코드는 Linear Regression을 사용하여 회귀 모델을 훈련합니다. 매개변수 (parameter), 메트릭 및 아티팩트가 MLflow로 자동으로 로깅됩니다. 또한 **estimator**라는 매개변수 (parameter)를 *LinearRegression* 값으로 로깅하고 있습니다.

1. 노트북에 새 코드 셀을 하나 더 추가하고 다음 코드를 입력한 후 실행합니다.

    ```python
   from sklearn.tree import DecisionTreeRegressor
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = DecisionTreeRegressor(max_depth=5) 
      model.fit(X_train, y_train)
    
      mlflow.log_param("estimator", "DecisionTreeRegressor")
    ```

    이 코드는 Decision Tree Regressor를 사용하여 회귀 모델을 훈련합니다. 매개변수 (parameter), 메트릭 및 아티팩트가 MLflow로 자동으로 로깅됩니다. 또한 **estimator**라는 매개변수 (parameter)를 *DecisionTreeRegressor* 값으로 로깅하고 있습니다.

## MLflow를 사용하여 실험 (Experiment) 검색 및 보기

MLflow로 모델을 훈련하고 추적한 후 MLflow 라이브러리를 사용하여 실험 (Experiment)과 해당 세부 정보를 검색할 수 있습니다.

1. 모든 실험 (Experiment)을 나열하려면 다음 코드를 사용합니다.

    ```python
   import mlflow
   experiments = mlflow.search_experiments()
   for exp in experiments:
       print(exp.name)
    ```

1. 특정 실험 (Experiment)을 검색하려면 이름으로 가져올 수 있습니다.

    ```python
   experiment_name = "experiment-diabetes"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. 실험 (Experiment) 이름을 사용하여 해당 실험 (Experiment)의 모든 작업을 검색할 수 있습니다.

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. 작업 실행 (run) 및 출력을 더 쉽게 비교하려면 검색을 구성하여 결과를 정렬할 수 있습니다. 예를 들어, 다음 셀은 결과를 *start_time*으로 정렬하고 최대 2개의 결과만 표시합니다.

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. 마지막으로, 여러 모델의 평가 메트릭을 나란히 플로팅하여 모델을 쉽게 비교할 수 있습니다.

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_r2_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_r2_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("R2 score")
   ax.set_title("R2 score by Estimator")
   for i, v in enumerate(df_results["metrics.training_r2_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    출력은 다음 이미지와 유사해야 합니다.

    ![Screenshot of the plotted evaluation metrics.](./Images/data-science-metrics.png)

## 실험 (Experiment) 탐색

Microsoft Fabric은 모든 실험 (Experiment)을 추적하고 시각적으로 탐색할 수 있도록 합니다.

1. 왼쪽 메뉴 모음에서 워크스페이스로 이동합니다.
1. **experiment-diabetes** 실험 (Experiment)을 선택하여 엽니다.

    > **Tip:**
    > 로깅된 실험 실행 (run)이 보이지 않으면 페이지를 새로 고침하세요.

1. **보기** 탭을 선택합니다.
1. **실행 목록 (Run list)**을 선택합니다.
1. 각 상자를 선택하여 최근 두 실행 (run)을 선택합니다.

    그 결과, 최근 두 실행 (run)이 **메트릭 비교** 창에서 서로 비교됩니다. 기본적으로 메트릭은 실행 (run) 이름별로 플로팅됩니다.

1. 각 실행 (run)에 대한 평균 절대 오차를 시각화하는 그래프의 **&#128393;** (편집) 버튼을 선택합니다.
1. **시각화 유형**을 **막대**로 변경합니다.
1. **X축**을 **estimator**로 변경합니다.
1. **바꾸기**를 선택하고 새 그래프를 탐색합니다.
1. 선택적으로, **메트릭 비교** 창의 다른 그래프에 대해서도 이 단계를 반복할 수 있습니다.

로깅된 추정기 (estimator)별로 성능 메트릭을 플로팅하여 어떤 알고리즘이 더 나은 모델을 만들었는지 검토할 수 있습니다.

## 모델 저장

실험 실행 (run)을 통해 훈련한 머신러닝 모델을 비교한 후, 가장 성능이 좋은 모델을 선택할 수 있습니다. 가장 성능이 좋은 모델을 사용하려면 모델을 저장하고 예측을 생성하는 데 사용합니다.

1. 실험 (Experiment) 개요에서 **보기** 탭이 선택되어 있는지 확인합니다.
1. **실행 세부 정보**를 선택합니다.
1. 가장 높은 훈련 R2 score를 가진 실행 (run)을 선택합니다.
1. **실행 (run)을 모델로 저장** 상자에서 **저장**을 선택합니다 (이것이 보이지 않으면 오른쪽으로 스크롤해야 할 수 있습니다).
1. 새로 열린 팝업 창에서 **새 모델 만들기**를 선택합니다.
1. **모델** 폴더를 선택합니다.
1. 모델 이름을 `model-diabetes`로 지정하고 **저장**을 선택합니다.
1. 모델이 생성될 때 화면 오른쪽 상단에 나타나는 알림에서 **ML 모델 보기**를 선택합니다. 창을 새로 고침할 수도 있습니다. 저장된 모델은 **모델 버전** 아래에 연결됩니다.

모델, 실험 (Experiment) 및 실험 실행 (run)이 연결되어 있어 모델이 어떻게 훈련되었는지 검토할 수 있습니다.

## 노트북 저장 및 Spark 세션 종료

이제 모델 훈련 및 평가를 마쳤으므로 의미 있는 이름으로 노트북을 저장하고 Spark 세션을 종료할 수 있습니다.

1. 노트북으로 돌아가서 노트북 메뉴 모음에서 ⚙️ **설정** 아이콘을 사용하여 노트북 설정을 봅니다.
2. 노트북의 **이름**을 **모델 훈련 및 비교**로 설정한 다음, 설정 창을 닫습니다.
3. 노트북 메뉴에서 **세션 중지**를 선택하여 Spark 세션을 종료합니다.

## 리소스 정리

이 연습에서는 노트북을 만들고 머신러닝 모델을 훈련했습니다. scikit-learn을 사용하여 모델을 훈련하고 MLflow를 사용하여 성능을 추적했습니다.

모델 및 실험 (Experiment) 탐색을 마쳤다면 이 연습을 위해 생성한 워크스페이스를 삭제할 수 있습니다.

1. 왼쪽 바에서 워크스페이스 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **워크스페이스 설정**을 선택합니다.
3. **일반** 섹션에서 **이 워크스페이스 제거**를 선택합니다.