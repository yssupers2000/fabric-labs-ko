---
lab:
    title: 'Preprocess data with Data Wrangler in Microsoft Fabric'
    module: 'Preprocess data with Data Wrangler in Microsoft Fabric'
---

# Microsoft Fabric에서 Data Wrangler를 사용하여 데이터 전처리

이 랩에서는 Microsoft Fabric의 Data Wrangler를 사용하여 데이터를 전처리하고, 일반적인 데이터 과학 작업 라이브러리를 사용하여 코드를 생성하는 방법을 배웁니다.

이 랩은 완료하는 데 약 **30**분 정도 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## Workspace 생성

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace를 생성하세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
1.  왼쪽 메뉴 모음에서 **Workspaces** (아이콘은 &#128455;와 비슷하게 생겼습니다.)를 선택합니다.
1.  원하는 이름으로 새 Workspace를 생성하고, Fabric Capacity(*Trial*, *Premium* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
1.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Fabric의 비어 있는 Workspace 스크린샷.](./Images/new-workspace.png)

## Notebook 생성

모델을 학습시키려면 *Notebook*을 생성할 수 있습니다. Notebook은 대화형 환경을 제공하며, 이 환경에서 *실험*(experiments)으로 코드(여러 언어)를 작성하고 실행할 수 있습니다.

1.  왼쪽 메뉴 모음에서 **생성**을 선택합니다. *새로 만들기* 페이지의 *데이터 과학* 섹션에서 **Notebook**을 선택합니다. 원하는 고유한 이름을 지정합니다.

    > **참고**: **생성** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    몇 초 후, 단일 *셀*(cell)이 포함된 새 Notebook이 열립니다. Notebook은 *코드* 또는 *Markdown*(서식 있는 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

1.  첫 번째 셀(현재는 *코드* 셀입니다)을 선택하고, 셀의 오른쪽 상단에 있는 동적 도구 모음에서 **M&#8595;** 버튼을 사용하여 셀을 *Markdown* 셀로 변환합니다.

    셀이 Markdown 셀로 변경되면 셀에 포함된 텍스트가 렌더링됩니다.

1.  필요한 경우, **&#128393;** (편집) 버튼을 사용하여 셀을 편집 모드로 전환한 다음, 내용을 삭제하고 다음 텍스트를 입력합니다.

    ```text
    # Perform data exploration for data science

    Use the code in this notebook to perform data exploration for data science.
    ```

## Dataframe에 데이터 로드

이제 데이터를 가져오는 코드를 실행할 준비가 되었습니다. Azure Open Datasets의 [**OJ Sales 데이터 세트**](https://learn.microsoft.com/en-us/azure/open-datasets/dataset-oj-sales-simulated?tabs=azureml-opendatasets?azure-portal=true)를 사용합니다. 데이터를 로드한 후 Data Wrangler에서 지원하는 구조인 Pandas dataframe으로 데이터를 변환합니다.

1.  Notebook에서 최신 셀 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가합니다.

    > **팁**: **+ 코드** 아이콘을 보려면 현재 셀의 출력 바로 아래와 왼쪽에 마우스를 가져다 놓습니다. 또는 메뉴 모음의 **편집** 탭에서 **+ 코드 셀 추가**를 선택합니다.

1.  다음 코드를 입력하여 데이터 세트를 dataframe에 로드합니다.

    ```python
    # Azure storage access info for open dataset diabetes
    blob_account_name = "azureopendatastorage"
    blob_container_name = "ojsales-simulatedcontainer"
    blob_relative_path = "oj_sales_data"
    blob_sas_token = r"" # Blank since container is Anonymous access
    
    # Set Spark config to access  blob storage
    wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
    spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
    print("Remote blob path: " + wasbs_path)
    
    # Spark reads csv
    df = spark.read.csv(wasbs_path, header=True)
    ```

1.  셀 왼쪽의 **&#9655; 셀 실행** 버튼을 사용하여 셀을 실행합니다. 또는 키보드에서 `SHIFT` + `ENTER`를 눌러 셀을 실행할 수 있습니다.

    > **참고**: 이번 세션에서 Spark 코드를 처음 실행하므로 Spark Pool이 시작되어야 합니다. 이는 세션에서 첫 번째 실행이 완료되는 데 1분 정도 걸릴 수 있음을 의미합니다. 후속 실행은 더 빠를 것입니다.

1.  셀 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고, 다음 코드를 입력합니다.

    ```python
    import pandas as pd

    df = df.toPandas()
    df = df.sample(n=500, random_state=1)
    
    df['WeekStarting'] = pd.to_datetime(df['WeekStarting'])
    df['Quantity'] = df['Quantity'].astype('int')
    df['Advert'] = df['Advert'].astype('int')
    df['Price'] = df['Price'].astype('float')
    df['Revenue'] = df['Revenue'].astype('float')
    
    df = df.reset_index(drop=True)
    df.head(4)
    ```

1.  셀 명령이 완료되면 셀 아래의 출력을 검토합니다. 다음과 유사하게 보일 것입니다.

    |   |WeekStarting|Store|Brand|Quantity|Advert|Price|Revenue|
    |---|---|---|---|---|---|---|---|
    |0|1991-10-17|947|minute.maid|13306|1|2.42|32200.52|
    |1|1992-03-26|1293|dominicks|18596|1|1.94|36076.24|
    |2|1991-08-15|2278|dominicks|17457|1|2.14|37357.98|
    |3|1992-09-03|2175|tropicana|9652|1|2.07|19979.64|
    |...|...|...|...|...|...|...|...|

    출력은 OJ Sales 데이터 세트의 처음 네 행을 보여줍니다.

## 요약 통계 보기

데이터를 로드했으므로 다음 단계는 Data Wrangler를 사용하여 데이터를 전처리하는 것입니다. 전처리는 모든 기계 학습 워크플로우에서 중요한 단계입니다. 데이터 정제 및 기계 학습 모델에 공급될 수 있는 형식으로 변환하는 과정을 포함합니다.

1.  Notebook 리본에서 **데이터**를 선택한 다음, **Data Wrangler 시작** 드롭다운을 선택합니다.
1.  `df` 데이터 세트를 선택합니다. Data Wrangler가 시작되면 **요약** 패널에 dataframe에 대한 설명적 개요를 생성합니다.
1.  **Revenue** Feature를 선택하고 이 Feature의 데이터 분포를 관찰합니다.
1.  **요약** 사이드 패널의 세부 정보를 검토하고 통계 값을 관찰합니다.

    ![요약 패널 세부 정보를 보여주는 Data Wrangler 페이지 스크린샷.](./Images/data-wrangler-summary.png)

    여기에서 어떤 통찰력을 얻을 수 있나요? 평균 Revenue는 약 **$33,459.54**이며, 표준 편차는 **$8,032.23**입니다. 이는 Revenue 값이 평균을 중심으로 약 **$8,032.23** 범위에 퍼져 있음을 시사합니다.

## 텍스트 데이터 형식 지정

이제 **Brand** Feature에 몇 가지 변환(transformations)을 적용해 보겠습니다.

1.  **Data Wrangler** 대시보드에서 그리드의 `Brand` Feature를 선택합니다.
1.  **Operations** 패널로 이동하여 **찾기 및 바꾸기**를 확장한 다음, **찾기 및 바꾸기**를 선택합니다.
1.  **찾기 및 바꾸기** 패널에서 다음 속성을 변경합니다.

    *   **이전 값:** "`.`"
    *   **새 값:** "` `" (공백 문자)

    작업 결과가 디스플레이 그리드에 자동으로 미리 보기됩니다.

1.  **적용**을 선택합니다.
1.  **Operations** 패널로 돌아가서 **형식**을 확장합니다.
1.  **첫 글자 대문자로 변환**을 선택합니다. **모든 단어 대문자로 변환** 토글을 켜고, **적용**을 선택합니다.
1.  **Notebook에 코드 추가**를 선택합니다. 또한, 코드를 복사하여 변환된 데이터 세트를 CSV 파일로 저장할 수도 있습니다.

    > **참고**: 코드가 Notebook 셀에 자동으로 복사되며, 사용할 준비가 되었습니다.

1.  Data Wrangler에서 생성된 코드는 원래 dataframe을 덮어쓰지 않으므로, 10행과 11행을 `df = clean_data(df)` 코드로 바꿉니다. 최종 코드 블록은 다음과 같아야 합니다.

    ```python
    def clean_data(df):
        # Replace all instances of "." with " " in column: 'Brand'
        df['Brand'] = df['Brand'].str.replace(".", " ", case=False, regex=False)
        # Capitalize the first character in column: 'Brand'
        df['Brand'] = df['Brand'].str.title()
        return df
    
    df = clean_data(df)
    ```

1.  코드 셀을 실행하고 `Brand` 변수를 확인합니다.

    ```python
    df['Brand'].unique()
    ```

    결과는 *Minute Maid*, *Dominicks*, *Tropicana* 값을 보여주어야 합니다.

텍스트 데이터를 시각적으로 조작하는 방법과 Data Wrangler를 사용하여 코드를 쉽게 생성하는 방법을 배웠습니다.

## 원-핫 인코딩 변환 적용

이제 전처리 단계의 일부로 데이터에 원-핫 인코딩(one-hot encoding) 변환을 적용하는 코드를 생성해 보겠습니다. 시나리오를 더욱 실용적으로 만들기 위해 일부 샘플 데이터를 생성하는 것으로 시작합니다. 이를 통해 실제 상황을 시뮬레이션하고 작동 가능한 Feature를 제공할 수 있습니다.

1.  상단 메뉴에서 `df` dataframe에 대해 Data Wrangler를 시작합니다.
1.  그리드의 `Brand` Feature를 선택합니다.
1.  **Operations** 패널에서 **수식**을 확장한 다음, **원-핫 인코딩**을 선택합니다.
1.  **원-핫 인코딩** 패널에서 **적용**을 선택합니다.

    Data Wrangler 디스플레이 그리드의 끝으로 이동합니다. 세 개의 새 Feature(`Brand_Dominicks`, `Brand_Minute Maid`, `Brand_Tropicana`)를 추가하고 `Brand` Feature를 제거했음을 알 수 있습니다.

1.  코드를 생성하지 않고 Data Wrangler를 종료합니다.

## 정렬 및 필터링 작업

특정 상점의 Revenue 데이터를 검토하고 제품 가격을 정렬해야 한다고 가정해 봅시다. 다음 단계에서 Data Wrangler를 사용하여 `df` dataframe을 필터링하고 분석합니다.

1.  `df` dataframe에 대해 Data Wrangler를 시작합니다.
1.  **Operations** 패널로 돌아가서 **정렬 및 필터링**을 확장합니다.
1.  **필터**를 선택합니다.
1.  **필터** 패널에서 다음 조건을 추가합니다.

    *   **대상 열**: `Store`
    *   **작업**: `같음`
    *   **값**: `1227`
    *   **동작**: `일치하는 행 유지`

1.  **적용**을 선택하고 Data Wrangler 디스플레이 그리드의 변경 사항을 확인합니다.
1.  **Revenue** Feature를 선택한 다음, **요약** 사이드 패널의 세부 정보를 검토합니다.

    여기에서 어떤 통찰력을 얻을 수 있나요? 왜도(skewness)는 **-0.751**이며, 약간의 왼쪽 꼬리 분포(음의 왜도)를 나타냅니다. 이는 분포의 왼쪽 꼬리가 오른쪽 꼬리보다 약간 더 길다는 것을 의미합니다. 즉, 평균보다 상당히 낮은 Revenue를 기록한 기간이 많다는 뜻입니다.

1.  **Operations** 패널로 돌아가서 **정렬 및 필터링**을 확장합니다.
1.  **값 정렬**을 선택합니다.
1.  **값 정렬** 패널에서 다음 속성을 선택합니다.

    *   **열 이름**: `Price`
    *   **정렬 순서**: `내림차순`

1. **적용**을 선택합니다.

    상점 **1227**의 최고 제품 가격은 **$2.68**입니다. 몇 개의 레코드만으로는 최고 제품 가격을 식별하기 쉽지만, 수천 개의 결과를 다룰 때는 복잡성을 고려해야 합니다.

## 단계 찾아보기 및 제거

이전 단계에서 생성한 정렬을 제거해야 하는 실수를 저질렀다고 가정해 봅시다. 다음 단계를 따라 제거합니다.

1.  **클리닝 단계** 패널로 이동합니다.
1.  **값 정렬** 단계를 선택합니다.
1.  삭제 아이콘을 선택하여 제거합니다.

    ![찾기 및 바꾸기 패널을 보여주는 Data Wrangler 페이지 스크린샷.](./Images/data-wrangler-delete.png)

    > **중요**: 그리드 보기와 요약은 현재 단계로 제한됩니다.

    변경 사항이 이전 단계인 **필터** 단계로 되돌려졌음을 확인합니다.

1.  코드를 생성하지 않고 Data Wrangler를 종료합니다.

## 데이터 집계

각 Brand별로 생성된 평균 Revenue를 파악해야 한다고 가정해 봅시다. 다음 단계에서 Data Wrangler를 사용하여 `df` dataframe에 대해 Group by 작업을 수행합니다.

1.  `df` dataframe에 대해 Data Wrangler를 시작합니다.
1.  **Operations** 패널로 돌아가서 **그룹화 및 집계**를 선택합니다.
1.  **그룹화할 열** 패널에서 `Brand` Feature를 선택합니다.
1.  **집계 추가**를 선택합니다.
1.  **집계할 열** 속성에서 `Revenue` Feature를 선택합니다.
1.  **집계 유형** 속성으로 `평균`을 선택합니다.
1.  **적용**을 선택합니다.
1.  **코드를 클립보드에 복사**를 선택합니다.
1.  코드를 생성하지 않고 Data Wrangler를 종료합니다.
1. `clean_data(df)` 함수 내의 `Brand` 변수 변환 코드와 집계 단계에서 생성된 코드를 결합합니다. 최종 코드 블록은 다음과 같아야 합니다.

    ```python
   def clean_data(df):    
       # Replace all instances of "." with " " in column: 'Brand'    
       df['Brand'] = df['Brand'].str.replace(".", " ", case=False, regex=False)    
       # Capitalize the first character in column: 'Brand'    
       df['Brand'] = df['Brand'].str.title()
        
       # Performed 1 aggregation grouped on column: 'Brand'    
       df = df.groupby(['Brand']).agg(Revenue_mean=('Revenue', 'mean')).reset_index()    
        
       return df    
        
   df = clean_data(df)
    ```

1. 셀 코드를 실행합니다.
1. dataframe의 데이터를 확인합니다.

    ```python
   print(df)
    ```

    결과:

    |   |Brand|Revenue_mean|
    |---|---|---|
    |0|Dominicks|33206.330958|
    |1|Minute Maid|33532.999632|
    |2|Tropicana|33637.863412|

일부 전처리 작업에 대한 코드를 생성하고, 코드를 Notebook으로 다시 함수로 복사했습니다. 이 함수는 필요에 따라 실행, 재사용 또는 수정할 수 있습니다.

## Notebook 저장 및 Spark Session 종료

모델링을 위한 데이터 전처리를 마쳤으므로, Notebook을 의미 있는 이름으로 저장하고 Spark Session을 종료할 수 있습니다.

1.  Notebook 메뉴 모음에서 ⚙️ **설정** 아이콘을 사용하여 Notebook 설정을 봅니다.
1.  Notebook의 **이름**을 **Data Wrangler를 사용하여 데이터 전처리**로 설정한 다음, 설정 창을 닫습니다.
1.  Notebook 메뉴에서 **세션 중지**를 선택하여 Spark Session을 종료합니다.

## 리소스 정리

이 실습에서 Notebook을 생성하고 Data Wrangler를 사용하여 기계 학습 모델을 위한 데이터를 탐색하고 전처리했습니다.

전처리 단계를 모두 살펴보셨다면, 이 실습을 위해 생성한 Workspace를 삭제할 수 있습니다.

1.  왼쪽 막대에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
1.  도구 모음의 **...** 메뉴에서 **Workspace 설정**을 선택합니다.
1.  **일반** 섹션에서 **이 Workspace 제거**를 선택합니다.