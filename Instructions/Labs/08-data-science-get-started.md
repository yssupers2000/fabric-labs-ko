---
lab:
    title: 'Get started with data science in Microsoft Fabric'
    module: 'Get started with data science in Microsoft Fabric'
---

# Get started with data science in Microsoft Fabric

이 랩에서는 데이터를 수집하고(ingest data), Notebook에서 데이터를 탐색하고, Data Wrangler로 데이터를 처리하고, 두 가지 유형의 모델을 학습합니다. 이 모든 단계를 수행하여 Microsoft Fabric의 데이터 과학 기능을 탐색할 수 있습니다.

이 랩을 완료하면 머신러닝(machine learning) 및 모델 추적(model tracking)에 대한 실습 경험을 얻고 Microsoft Fabric에서 *Notebooks*, *Data Wrangler*, *experiments* 및 *models*를 사용하는 방법을 배웁니다.

이 랩을 완료하는 데 약 **20**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 워크스페이스 만들기

Fabric에서 데이터를 작업하기 전에 Fabric 평가판이 활성화된 워크스페이스를 만드세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces** (아이콘은 &#128455;와 유사하게 생겼습니다.)를 선택합니다.
3.  원하는 이름으로 새 워크스페이스를 만들고 Fabric Capacity(*Trial*, *Premium*, 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
4.  새 워크스페이스가 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Notebook 만들기

코드를 실행하려면 *Notebook*을 만들 수 있습니다. Notebook은 코드를 작성하고 실행할 수 있는 (여러 언어로) 대화형 환경을 제공합니다.

1.  왼쪽 메뉴 모음에서 **만들기**를 선택합니다. *새 항목* 페이지의 *데이터 과학* 섹션 아래에서 **Notebook**을 선택합니다. 원하는 고유한 이름을 지정하세요.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않은 경우 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    몇 초 후 단일 *cell*을 포함하는 새 Notebook이 열립니다. Notebook은 *코드* 또는 *markdown* (서식 있는 텍스트)을 포함할 수 있는 하나 이상의 cell로 구성됩니다.

2.  첫 번째 cell (현재 *코드* cell입니다)을 선택한 다음, 오른쪽 상단의 동적 도구 모음에서 **M&#8595;** 버튼을 사용하여 cell을 *markdown* cell로 변환합니다.

    cell이 markdown cell로 변경되면 해당 텍스트가 렌더링됩니다.

3.  **&#128393;** (편집) 버튼을 사용하여 cell을 편집 모드로 전환한 다음, 내용을 삭제하고 다음 텍스트를 입력합니다.

    ```text
   # Data science in Microsoft Fabric
    ```

## 데이터 가져오기

이제 코드를 실행하여 데이터를 가져오고 모델을 학습할 준비가 되었습니다. Azure Open Datasets의 [당뇨병 데이터 세트](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)로 작업할 것입니다. 데이터를 로드한 후 데이터를 Pandas dataframe으로 변환합니다. 이는 행과 열의 데이터로 작업하기 위한 일반적인 구조입니다.

1.  Notebook에서 최신 cell 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 cell을 추가합니다.

    > **팁**: **+ 코드** 아이콘을 보려면 마우스를 현재 cell 출력 바로 아래 왼쪽으로 이동하세요. 또는 메뉴 모음의 **편집** 탭에서 **+ 코드 cell 추가**를 선택합니다.

2.  새 코드 cell에 다음 코드를 입력합니다.

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

3.  cell 왼쪽에 있는 **&#9655; Cell 실행** 버튼을 사용하여 실행합니다. 또는 키보드에서 `SHIFT` + `ENTER`를 눌러 cell을 실행할 수 있습니다.

    > **참고**: 이 세션에서 Spark 코드를 처음 실행하는 것이므로 Spark pool이 시작되어야 합니다. 이는 세션의 첫 번째 실행에 완료하는 데 1분 정도 걸릴 수 있음을 의미합니다. 이후 실행은 더 빨라집니다.

4.  cell 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 cell을 추가하고 다음 코드를 입력합니다.

    ```python
   display(df)
    ```

5.  cell 명령이 완료되면 cell 아래의 출력을 검토하세요. 다음 그림과 유사하게 표시되어야 합니다.

    |AGE|SEX|BMI|BP|S1|S2|S3|S4|S5|S6|Y|
    |---|---|---|--|--|--|--|--|--|--|--|
    |59|2|32.1|101.0|157|93.2|38.0|4.0|4.8598|87|151|
    |48|1|21.6|87.0|183|103.2|70.0|3.0|3.8918|69|75|
    |72|2|30.5|93.0|156|93.6|41.0|4.0|4.6728|85|141|
    |24|1|25.3|84.0|198|131.4|40.0|5.0|4.8903|89|206|
    |50|1|23.0|101.0|192|125.4|52.0|4.0|4.2905|80|135|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    출력은 당뇨병 데이터 세트의 행과 열을 보여줍니다.

6.  렌더링된 테이블 상단에는 **테이블**과 **+ 새 차트**의 두 개의 탭이 있습니다. **+ 새 차트**를 선택합니다.
7.  새 시각화를 만들기 위해 차트 오른쪽에 있는 **직접 만들기** 옵션을 선택합니다.
8.  다음 차트 설정을 선택합니다.
    *   **Chart Type**: `Box plot`
    *   **Y-axis**: `Y`
9.  레이블 열 `Y`의 분포를 보여주는 출력을 검토하세요.

## 데이터 준비

이제 데이터를 수집하고 탐색했으므로 데이터를 변환할 수 있습니다. Notebook에서 코드를 실행하거나 Data Wrangler를 사용하여 코드를 자동으로 생성할 수 있습니다.

1.  데이터는 Spark dataframe으로 로드됩니다. Data Wrangler는 Spark 또는 Pandas dataframe을 모두 허용하지만, 현재 Pandas와 함께 작동하도록 최적화되어 있습니다. 따라서 데이터를 Pandas dataframe으로 변환할 것입니다. Notebook에서 다음 코드를 실행하세요.

    ```python
   df = df.toPandas()
   df.head()
    ```

2.  Notebook 리본에서 **Data Wrangler**를 선택한 다음, `df` 데이터 세트를 선택합니다. Data Wrangler가 실행되면 **요약** 패널에 dataframe의 설명적인 개요를 생성합니다.

    현재 레이블 열은 `Y`이며, 이는 연속 변수입니다. Y를 예측하는 머신러닝 모델을 학습하려면 회귀 모델을 학습해야 합니다. Y의 (예측된) 값은 해석하기 어려울 수 있습니다. 대신, 당뇨병 발병에 대한 저위험 또는 고위험 여부를 예측하는 분류 모델 학습을 탐색할 수 있습니다. 분류 모델을 학습하려면 `Y`의 값을 기반으로 이진 레이블 열을 만들어야 합니다.

3.  Data Wrangler에서 `Y` 열을 선택합니다. `220-240` bin에서 빈도가 감소하는 것에 주목하세요. 75번째 백분위수 `211.5`는 히스토그램에서 두 영역의 전환과 대략적으로 일치합니다. 이 값을 저위험 및 고위험의 임계값으로 사용해 봅시다.
4.  **작업** 패널로 이동하여 **수식**을 확장한 다음, **수식에서 열 만들기**를 선택합니다.
5.  다음 설정으로 새 열을 만듭니다.
    *   **Column name**: `Risk`
    *   **Column formula**: `(df['Y'] > 211.5).astype(int)`
6.  **적용**을 선택합니다.
7.  미리 보기에 추가된 새 열 `Risk`를 검토하세요. 값 `1`을 가진 행의 수가 전체 행의 약 25%여야 함을 확인합니다 (`Y`의 75번째 백분위수이므로).
8.  **notebook에 코드 추가**를 선택합니다.
9.  Data Wrangler에 의해 생성된 코드를 포함하는 cell을 실행하세요.
10. `Risk` 열이 예상대로 구성되었는지 확인하기 위해 새 cell에서 다음 코드를 실행하세요.

    ```python
   df_clean.describe()
    ```

## 머신러닝 모델 학습

데이터를 준비했으므로 이제 이를 사용하여 당뇨병을 예측하는 머신러닝 모델을 학습시킬 수 있습니다. 데이터 세트로 두 가지 다른 유형의 모델을 학습할 수 있습니다. 회귀 모델(`Y` 예측) 또는 분류 모델(`Risk` 예측)입니다. scikit-learn 라이브러리를 사용하여 모델을 학습하고 MLflow로 모델을 추적합니다.

### 회귀 모델 학습

1.  데이터를 학습 및 테스트 데이터 세트로 분할하고, 예측하려는 레이블 `Y`에서 특성(feature)을 분리하기 위해 다음 코드를 실행하세요.

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

2.  Notebook에 새 코드 cell을 추가하고, 다음 코드를 입력한 다음 실행합니다.

    ```python
   import mlflow
   experiment_name = "diabetes-regression"
   mlflow.set_experiment(experiment_name)
    ```

    이 코드는 `diabetes-regression`이라는 이름의 MLflow experiment를 생성합니다. 모델은 이 experiment에서 추적됩니다.

3.  Notebook에 새 코드 cell을 추가하고, 다음 코드를 입력한 다음 실행합니다.

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    ```

    이 코드는 Linear Regression을 사용하여 회귀 모델을 학습합니다. 매개 변수(Parameters), 메트릭(metrics), 아티팩트(artifacts)는 MLflow로 자동으로 로깅됩니다.

### 분류 모델 학습

1.  데이터를 학습 및 테스트 데이터 세트로 분할하고, 예측하려는 레이블 `Risk`에서 특성(feature)을 분리하기 위해 다음 코드를 실행하세요.

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Risk'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

2.  Notebook에 새 코드 cell을 추가하고, 다음 코드를 입력한 다음 실행합니다.

    ```python
   import mlflow
   experiment_name = "diabetes-classification"
   mlflow.set_experiment(experiment_name)
    ```

    이 코드는 `diabetes-classification`이라는 이름의 MLflow experiment를 생성합니다. 모델은 이 experiment에서 추적됩니다.

3.  Notebook에 새 코드 cell을 추가하고, 다음 코드를 입력한 다음 실행합니다.

    ```python
   from sklearn.linear_model import LogisticRegression
    
   with mlflow.start_run():
       mlflow.sklearn.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)
    ```

    이 코드는 Logistic Regression을 사용하여 분류 모델을 학습합니다. 매개 변수(Parameters), 메트릭(metrics), 아티팩트(artifacts)는 MLflow로 자동으로 로깅됩니다.

## Experiment 탐색

Microsoft Fabric은 모든 experiment를 추적하고 시각적으로 탐색할 수 있도록 합니다.

1.  왼쪽의 허브 메뉴 모음에서 워크스페이스로 이동합니다.
2.  `diabetes-regression` experiment를 선택하여 엽니다.

    > **팁:**
    > 기록된 experiment 실행이 보이지 않으면 페이지를 새로 고치세요.

3.  **실행 메트릭**을 검토하여 회귀 모델의 정확도를 탐색하세요.
4.  홈 페이지로 돌아가서 `diabetes-classification` experiment를 선택하여 엽니다.
5.  **실행 메트릭**을 검토하여 분류 모델의 정확도를 탐색하세요. 다른 유형의 모델을 학습했으므로 메트릭 유형이 다름에 유의하세요.

## 모델 저장

experiment에서 학습한 머신러닝 모델을 비교한 후 가장 성능이 좋은 모델을 선택할 수 있습니다. 가장 성능이 좋은 모델을 사용하려면 모델을 저장하고 이를 사용하여 예측을 생성합니다.

1.  experiment 리본에서 **ML 모델로 저장**을 선택합니다.
2.  새로 열린 팝업 창에서 **새 ML 모델 만들기**를 선택합니다.
3.  `model` 폴더를 선택합니다.
4.  모델 이름을 `model-diabetes`로 지정하고 **저장**을 선택합니다.
5.  모델이 생성될 때 화면 오른쪽 상단에 표시되는 알림에서 **ML 모델 보기**를 선택합니다. 창을 새로 고칠 수도 있습니다. 저장된 모델은 **ML 모델 버전** 아래에 연결됩니다.

모델, experiment 및 experiment 실행이 연결되어 있어 모델이 어떻게 학습되었는지 검토할 수 있습니다.

## Notebook 저장 및 Spark 세션 종료

모델 학습 및 평가를 마쳤으므로 이제 의미 있는 이름으로 Notebook을 저장하고 Spark 세션을 종료할 수 있습니다.

1.  Notebook 메뉴 모음에서 ⚙️ **설정** 아이콘을 사용하여 Notebook 설정을 확인합니다.
2.  Notebook의 **이름**을 **모델 학습 및 비교**로 설정한 다음, 설정 창을 닫습니다.
3.  Notebook 메뉴에서 **세션 중지**를 선택하여 Spark 세션을 종료합니다.

## 리소스 정리

이 실습에서는 Notebook을 만들고 머신러닝 모델을 학습했습니다. scikit-learn을 사용하여 모델을 학습했고 MLflow를 사용하여 성능을 추적했습니다.

모델 및 experiment 탐색을 마쳤으면 이 실습을 위해 만든 워크스페이스를 삭제할 수 있습니다.

1.  왼쪽 표시줄에서 워크스페이스 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2.  도구 모음의 **...** 메뉴에서 **워크스페이스 설정**을 선택합니다.
3.  **일반** 섹션에서 **이 워크스페이스 제거**를 선택합니다.