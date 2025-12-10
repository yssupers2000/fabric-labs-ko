---
lab:
    title: 'Explore data for data science with notebooks in Microsoft Fabric'
    module: 'Explore data for data science with notebooks in Microsoft Fabric'
---

# Microsoft Fabric에서 Notebook을 사용하여 데이터 과학을 위한 데이터 탐색

이 랩에서는 데이터 탐색을 위해 Notebook을 사용합니다. Notebook은 데이터를 대화식으로 탐색하고 분석하기 위한 강력한 도구입니다. 이 실습 동안 데이터셋을 탐색하고, 요약 통계(summary statistics)를 생성하며, 데이터를 더 잘 이해하기 위한 시각화를 생성하기 위해 Notebook을 생성하고 사용하는 방법을 배웁니다. 이 랩을 마치면 데이터 탐색 및 분석을 위해 Notebook을 사용하는 방법에 대한 확실한 이해를 갖게 될 것입니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 워크스페이스 생성

Fabric에서 데이터를 사용하기 전에 Fabric 평가판이 활성화된 워크스페이스를 생성합니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
1. 왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 비슷합니다)를 선택합니다.
1. 원하는 이름으로 새 워크스페이스를 생성하고, Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium* 또는 *Fabric*)를 선택합니다.
1. 새 워크스페이스가 열리면 비어 있어야 합니다.

    ![Fabric의 비어 있는 워크스페이스 스크린샷.](./Images/new-workspace.png)

## Notebook 생성

모델을 학습시키려면 *Notebook*을 생성할 수 있습니다. Notebook은 *실험(experiments)*으로 코드를 작성하고 실행(여러 언어 지원)할 수 있는 대화형 환경을 제공합니다.

1. 왼쪽 메뉴 바에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *데이터 과학* 섹션에서 **Notebook**을 선택합니다. 원하는 고유한 이름을 지정합니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않으면 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    몇 초 후에 단일 *셀*을 포함하는 새 Notebook이 열립니다. Notebook은 *코드* 또는 *Markdown* (형식화된 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

1. 첫 번째 셀(현재 *코드* 셀)을 선택한 다음, 오른쪽 상단에 있는 동적 도구 모음에서 **M&#8595;** 버튼을 사용하여 셀을 *Markdown* 셀로 변환합니다.

    셀이 Markdown 셀로 변경되면 포함된 텍스트가 렌더링(rendered)됩니다.

1. 필요한 경우 **&#128393;** (편집) 버튼을 사용하여 셀을 편집 모드로 전환한 다음, 내용을 삭제하고 다음 텍스트를 입력합니다.

    ```text
   # Perform data exploration for data science

   Use the code in this notebook to perform data exploration for data science.
    ```

## 데이터프레임으로 데이터 로드

이제 데이터를 가져오는 코드를 실행할 준비가 되었습니다. Azure Open Datasets에서 [**당뇨병 데이터셋**](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)을 사용하여 작업합니다. 데이터를 로드한 후에는 데이터를 Pandas 데이터프레임으로 변환합니다. 이는 행과 열의 데이터로 작업하기 위한 일반적인 구조입니다.

1. Notebook에서 최신 셀 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가합니다.

    > **팁**: **+ 코드** 아이콘을 보려면 마우스를 현재 셀 출력의 바로 아래쪽과 왼쪽으로 이동합니다. 또는 메뉴 바의 **편집** 탭에서 **+ 코드 셀 추가**를 선택합니다.

1. 데이터셋을 데이터프레임으로 로드하기 위해 다음 코드를 입력합니다.

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

1. 셀의 왼쪽에 있는 **&#9655; 셀 실행** 버튼을 사용하여 실행합니다. 또는 키보드에서 **SHIFT** + **ENTER**를 눌러 셀을 실행할 수 있습니다.

    > **참고**: 이 세션에서 Spark 코드를 처음 실행하는 것이므로 Spark 풀을 시작해야 합니다. 이는 세션에서 첫 번째 실행이 완료하는 데 약 1분 정도 걸릴 수 있음을 의미합니다. 이후 실행은 더 빠릅니다.

1. 셀 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   display(df)
    ```

1. 셀 명령이 완료되면 셀 아래의 출력을 검토합니다. 출력은 다음과 같아야 합니다.

    |AGE|SEX|BMI|BP|S1|S2|S3|S4|S5|S6|Y|
    |---|---|---|--|--|--|--|--|--|--|--|
    |59|2|32.1|101.0|157|93.2|38.0|4.0|4.8598|87|151|
    |48|1|21.6|87.0|183|103.2|70.0|3.0|3.8918|69|75|
    |72|2|30.5|93.0|156|93.6|41.0|4.0|4.6728|85|141|
    |24|1|25.3|84.0|198|131.4|40.0|5.0|4.8903|89|206|
    |50|1|23.0|101.0|192|125.4|52.0|4.0|4.2905|80|135|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    출력은 당뇨병 데이터셋의 행과 열을 보여줍니다. 데이터는 10개의 기준 변수(나이, 성별, 체질량 지수, 평균 혈압 및 당뇨병 환자의 6가지 혈청 측정값)와 관심 반응(기준 시점 1년 후 질병 진행의 정량적 측정)으로 구성되며, 이는 **Y**로 레이블(labelled)되어 있습니다.

1. 데이터는 Spark 데이터프레임으로 로드됩니다. Scikit-learn은 입력 데이터셋이 Pandas 데이터프레임일 것으로 예상합니다. 데이터셋을 Pandas 데이터프레임으로 변환하려면 아래 코드를 실행합니다.

    ```python
   df = df.toPandas()
   df.head()
    ```

## 데이터 형태 확인

이제 데이터를 로드했으므로 행과 열의 수, 데이터 유형, 누락된 값(missing values)과 같은 데이터셋의 구조를 확인할 수 있습니다.

1. 셀 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   # Display the number of rows and columns in the dataset
   print("Number of rows:", df.shape[0])
   print("Number of columns:", df.shape[1])

   # Display the data types of each column
   print("\nData types of columns:")
   print(df.dtypes)
    ```

    데이터셋은 **442개의 행**과 **11개의 열**을 포함합니다. 이는 데이터셋에 442개의 샘플과 11개의 특징(features) 또는 변수가 있음을 의미합니다. **SEX** 변수는 범주형(categorical) 또는 문자열(string) 데이터를 포함할 가능성이 높습니다.

## 누락된 데이터 확인

1. 셀 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   missing_values = df.isnull().sum()
   print("\nMissing values per column:")
   print(missing_values)
    ```

    이 코드는 누락된 값(missing values)을 확인합니다. 데이터셋에 누락된 데이터가 없음을 확인하십시오.

## 숫자 변수에 대한 기술 통계 생성

이제 숫자 변수의 분포를 이해하기 위해 기술 통계(descriptive statistics)를 생성해 보겠습니다.

1. 셀 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   df.describe()
    ```

    평균 연령은 약 48.5세이며, 표준 편차는 13.1세입니다. 가장 어린 개인은 19세이고 가장 나이 많은 개인은 79세입니다. 평균 BMI는 약 26.4이며, 이는 [WHO 표준](https://www.who.int/health-topics/obesity#tab=tab_1)에 따라 **과체중** 범주에 속합니다. 최소 BMI는 18이고 최대 BMI는 42.2입니다.

## 데이터 분포 플롯

BMI 특징(feature)을 확인하고, 그 특성을 더 잘 이해하기 위해 분포를 플롯(plot)해 보겠습니다.

1. Notebook에 다른 코드 셀을 추가합니다. 그런 다음, 이 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
   import numpy as np
    
   # Calculate the mean, median of the BMI variable
   mean = df['BMI'].mean()
   median = df['BMI'].median()
   
   # Histogram of the BMI variable
   plt.figure(figsize=(8, 6))
   plt.hist(df['BMI'], bins=20, color='skyblue', edgecolor='black')
   plt.title('BMI Distribution')
   plt.xlabel('BMI')
   plt.ylabel('Frequency')
    
   # Add lines for the mean and median
   plt.axvline(mean, color='red', linestyle='dashed', linewidth=2, label='Mean')
   plt.axvline(median, color='green', linestyle='dashed', linewidth=2, label='Median')
    
   # Add a legend
   plt.legend()
   plt.show()
    ```

    이 그래프에서 데이터셋의 BMI 범위와 분포를 관찰할 수 있습니다. 예를 들어, 대부분의 BMI는 23.2에서 29.2 사이에 있으며, 데이터는 오른쪽으로 치우쳐(right skewed) 있습니다.

## 다변량 분석 수행

데이터 내의 패턴과 관계를 밝히기 위해 산점도(scatter plots) 및 상자 그림(box plots)과 같은 시각화를 생성해 보겠습니다.

1. 셀 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns

   # Scatter plot of BMI vs. Target variable
   plt.figure(figsize=(8, 6))
   sns.scatterplot(x='BMI', y='Y', data=df)
   plt.title('BMI vs. Target variable')
   plt.xlabel('BMI')
   plt.ylabel('Target')
   plt.show()
    ```

    BMI가 증가함에 따라 대상 변수(target variable)도 증가하는 것을 볼 수 있으며, 이는 이 두 변수 사이에 양의 선형 관계가 있음을 나타냅니다.

1. Notebook에 다른 코드 셀을 추가합니다. 그런 다음, 이 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   import seaborn as sns
   import matplotlib.pyplot as plt
    
   fig, ax = plt.subplots(figsize=(7, 5))
    
   # Replace numeric values with labels
   df['SEX'] = df['SEX'].replace({1: 'Male', 2: 'Female'})
    
   sns.boxplot(x='SEX', y='BP', data=df, ax=ax)
   ax.set_title('Blood pressure across Gender')
   plt.tight_layout()
   plt.show()
    ```

    이러한 관찰 결과는 남성 및 여성 환자의 혈압 프로필에 차이가 있음을 시사합니다. 평균적으로 여성 환자는 남성 환자보다 혈압이 더 높습니다.

1. 데이터를 집계(aggregating)하면 시각화 및 분석을 위해 더 관리하기 쉽게 만들 수 있습니다. Notebook에 다른 코드 셀을 추가합니다. 그런 다음, 이 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   # Calculate average BP and BMI by SEX
   avg_values = df.groupby('SEX')[['BP', 'BMI']].mean()
    
   # Bar chart of the average BP and BMI by SEX
   ax = avg_values.plot(kind='bar', figsize=(15, 6), edgecolor='black')
    
   # Add title and labels
   plt.title('Avg. Blood Pressure and BMI by Gender')
   plt.xlabel('Gender')
   plt.ylabel('Average')
    
   # Display actual numbers on the bar chart
   for p in ax.patches:
       ax.annotate(format(p.get_height(), '.2f'), 
                   (p.get_x() + p.get_width() / 2., p.get_height()), 
                   ha = 'center', va = 'center', 
                   xytext = (0, 10), 
                   textcoords = 'offset points')
    
   plt.show()
    ```

    이 그래프는 남성 환자에 비해 여성 환자의 평균 혈압이 더 높음을 보여줍니다. 또한 평균 체질량 지수(BMI)가 남성보다 여성에게서 약간 더 높음을 보여줍니다.

1. Notebook에 다른 코드 셀을 추가합니다. 그런 다음, 이 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   plt.figure(figsize=(10, 6))
   sns.lineplot(x='AGE', y='BMI', data=df, errorbar=None)
   plt.title('BMI over Age')
   plt.xlabel('Age')
   plt.ylabel('BMI')
   plt.show()
    ```

    19세에서 30세 사이의 연령 그룹은 평균 BMI 값이 가장 낮고, 65세에서 79세 사이의 연령 그룹에서 평균 BMI가 가장 높게 나타납니다. 또한 대부분의 연령 그룹에서 평균 BMI가 과체중 범위에 속함을 관찰할 수 있습니다.

## 상관 분석

특징(features) 간의 관계와 의존성을 이해하기 위해 서로 다른 특징 간의 상관 관계를 계산해 보겠습니다.

1. 셀 출력 아래에 있는 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   df.corr(numeric_only=True)
    ```

1. 히트맵(heatmap)은 변수 쌍 간의 관계 강도와 방향을 빠르게 시각화하는 데 유용한 도구입니다. 이는 강한 양의 상관 관계 또는 음의 상관 관계를 강조하고, 상관 관계가 없는 쌍을 식별할 수 있습니다. 히트맵을 생성하려면 Notebook에 다른 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   plt.figure(figsize=(15, 7))
   sns.heatmap(df.corr(numeric_only=True), annot=True, vmin=-1, vmax=1, cmap="Blues")
    ```

    S1 및 S2 변수는 **0.89**의 높은 양의 상관 관계를 가지며, 이는 같은 방향으로 움직임을 나타냅니다. S1이 증가하면 S2도 증가하는 경향이 있고 그 반대도 마찬가지입니다. 또한 S3 및 S4는 **-0.73**의 강한 음의 상관 관계를 가집니다. 이는 S3이 증가함에 따라 S4가 감소하는 경향이 있음을 의미합니다.

## Notebook 저장 및 Spark 세션 종료

이제 데이터 탐색을 마쳤으므로 의미 있는 이름으로 Notebook을 저장하고 Spark 세션을 종료할 수 있습니다.

1. Notebook 메뉴 바에서 ⚙️ **설정** 아이콘을 사용하여 Notebook 설정을 봅니다.
2. Notebook의 **이름**을 **Explore data for data science**로 설정한 다음, 설정 창을 닫습니다.
3. Notebook 메뉴에서 **세션 중지**를 선택하여 Spark 세션을 종료합니다.

## 리소스 정리

이 실습에서는 데이터 탐색을 위해 Notebook을 생성하고 사용했습니다. 또한 요약 통계(summary statistics)를 계산하고, 데이터의 패턴과 관계를 더 잘 이해하기 위한 시각화를 생성하는 코드를 실행했습니다.

모델 및 실험(experiments) 탐색을 마쳤으면 이 실습을 위해 생성한 워크스페이스를 삭제할 수 있습니다.

1. 왼쪽 바에서 워크스페이스 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **워크스페이스 설정**을 선택합니다.
3. **일반** 섹션에서 **이 워크스페이스 제거**를 선택합니다.