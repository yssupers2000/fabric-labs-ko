---
lab:
    title: 'Analyze data with Apache Spark and Copilot in Microsoft Fabric notebooks'
    module: 'Get started with Copilot in Fabric for data engineering'
---

# Apache Spark 및 Copilot을 사용하여 Microsoft Fabric Notebook에서 데이터 분석

이 랩에서는 Fabric Data Engineering용 Copilot을 사용하여 Notebook에서 Lakehouse에 데이터를 로드, 변환 및 저장합니다. Notebook은 코드, 시각화 및 설명 텍스트를 단일 문서에 결합하는 대화형 환경을 제공합니다. 이 형식은 워크플로를 문서화하고, 추론을 설명하고, 다른 사람들과 결과를 공유하는 것을 쉽게 만듭니다. Notebook을 사용하면 코드를 반복적으로 개발하고 테스트하며, 각 단계에서 데이터를 시각화하고, 분석 프로세스에 대한 명확한 기록을 유지할 수 있습니다. 이 접근 방식은 협업, 재현성 및 이해도를 향상시켜 Notebook을 데이터 엔지니어링 및 분석 작업에 이상적인 도구로 만듭니다.

전통적으로 데이터 엔지니어링을 위해 Notebook을 사용하는 것은 Python 또는 Scala와 같은 언어로 코드를 작성하고 Apache Spark 및 pandas와 같은 프레임워크와 라이브러리에 대한 확실한 이해를 필요로 합니다. 이는 프로그래밍에 익숙하지 않거나 이러한 도구에 익숙하지 않은 사람들에게는 어려울 수 있습니다. Fabric Notebook의 Copilot을 사용하면 자연어(natural language)로 데이터 작업을 설명할 수 있으며, Copilot이 필요한 코드를 생성하여 기술적 복잡성 대부분을 처리하고 분석에 집중할 수 있도록 돕습니다.

이 실습을 완료하는 데 약 **30**분 정도 소요됩니다.

## 학습 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- 데이터 엔지니어링 작업을 위한 Microsoft Fabric 작업 영역(workspace)과 Lakehouse를 생성하고 구성합니다.
- Fabric Notebook에서 Copilot을 사용하여 자연어 프롬프트(natural language prompts)로부터 코드를 생성합니다.
- Apache Spark 및 Copilot 지원 워크플로를 사용하여 데이터를 수집(ingest), 정리(clean) 및 변환합니다.
- 분할(splitting), 필터링(filtering) 및 데이터 형식(data types) 변환을 통해 분석을 위한 통계 데이터셋(statistical datasets)을 정규화(normalize)하고 준비합니다.
- 변환된 데이터를 Lakehouse에 테이블(table)로 저장하여 다운스트림 분석(downstream analytics)에 활용합니다.
- Copilot을 사용하여 데이터 탐색(data exploration) 및 유효성 검사(validation)를 위한 쿼리(queries) 및 시각화(visualizations)를 생성합니다.
- Microsoft Fabric에서 데이터 정리, 변환 및 협업 분석(collaborative analytics)을 위한 모범 사례(best practices)를 이해합니다.

## 시작하기 전에

이 실습을 완료하려면 Copilot이 활성화된 [Microsoft Fabric Capacity (F2 이상)](https://learn.microsoft.com/fabric/fundamentals/copilot-enable-fabric)가 필요합니다.

> **참고**: 편의를 위해 이 실습의 모든 프롬프트가 포함된 Notebook을 다음에서 다운로드할 수 있습니다.

`https://github.com/MicrosoftLearning/mslearn-fabric/raw/refs/heads/main/Allfiles/Labs/22b/Starter/eurostat-notebook.ipynb`

## 실습 시나리오

다중 전문 병원 네트워크인 Contoso Health가 EU 내에서 서비스를 확장하고자 하며 예측 인구 데이터를 분석하려고 한다고 가정해 보겠습니다. 이 예시에서는 [Eurostat](https://ec.europa.eu/eurostat/web/main/home) (유럽 연합 통계청)의 인구 예측 데이터셋(dataset)을 사용합니다.

출처: EUROPOP2023 Population on January 1 by age, sex, and type of projection [[proj_23np](https://ec.europa.eu/eurostat/databrowser/product/view/proj_23np?category=proj.proj_23n)], 최종 업데이트: 2023년 6월 28일.

## 작업 영역 생성

Fabric에서 데이터를 작업하기 전에 Fabric이 활성화된 작업 영역(workspace)을 생성합니다. Microsoft Fabric의 작업 영역은 Lakehouse, Notebook 및 데이터셋(datasets)을 포함한 모든 데이터 엔지니어링 아티팩트(artifacts)를 구성하고 관리할 수 있는 협업 환경으로 사용됩니다. 이는 데이터 분석에 필요한 모든 리소스가 포함된 프로젝트 폴더(project folder)라고 생각하면 됩니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.

1. 왼쪽 메뉴 모음에서 **Workspaces**를 선택합니다 (아이콘은 &#128455;와 유사합니다).

1. Fabric Capacity(*Premium* 또는 *Fabric*)가 포함된 라이선싱 모드(licensing mode)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다. *Trial*은 지원되지 않습니다.
   
    > **이것이 중요한 이유**: Copilot은 유료 Fabric Capacity에서 작동해야 합니다. 이는 이 랩 전체에서 코드를 생성하는 데 도움이 될 AI 기반 기능에 액세스할 수 있도록 보장합니다.

1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## Lakehouse 생성

이제 작업 영역이 생겼으니 데이터를 수집(ingest)할 Lakehouse를 만들 차례입니다. Lakehouse는 데이터 레이크(data lake)(다양한 형식으로 원시 데이터 저장)의 이점과 데이터 웨어하우스(data warehouse)(분석에 최적화된 구조화된 데이터)의 이점을 결합합니다. 이는 원시 인구 데이터를 위한 저장 위치이자 정리되고 변환된 데이터셋(dataset)의 대상 역할을 할 것입니다.

1. 왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Lakehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다. "Lakehouse schemas (Public Preview)" 옵션이 비활성화되어 있는지 확인합니다.

    >**참고**: **Create** 옵션이 사이드바에 고정되어 있지 않으면 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

![Fabric의 만들기 버튼 스크린샷.](./Images/copilot-fabric-notebook-create.png)

약 1분 정도 후에 새 빈 Lakehouse가 생성됩니다.

![새 Lakehouse 스크린샷.](./Images/new-lakehouse.png)

## Notebook 생성

이제 데이터를 작업하기 위해 Fabric Notebook을 생성할 수 있습니다. Notebook은 코드를 작성하고 실행하며, 결과를 시각화하고, 데이터 분석 프로세스(data analysis process)를 문서화할 수 있는 대화형 환경을 제공합니다. 이는 탐색적 데이터 분석(exploratory data analysis) 및 반복적 개발(iterative development)에 이상적이며, 각 단계의 결과를 즉시 확인할 수 있도록 합니다.

1. 왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Notebook**을 선택합니다.

    **Notebook 1**이라는 새 Notebook이 생성되고 열립니다.

    ![새 Notebook 스크린샷.](./Images/new-notebook.png)

1. Fabric은 Notebook 1, Notebook 2 등 생성하는 각 Notebook에 이름을 할당합니다. 메뉴의 **Home** 탭 위에 있는 이름 패널을 클릭하여 더 설명적인 이름으로 변경합니다.

    ![이름을 바꿀 수 있는 새 Notebook 스크린샷.](./Images/copilot-fabric-notebook-rename.png)

1. 첫 번째 셀(현재 코드 셀)을 선택한 다음, 오른쪽 상단 도구 모음에서 **M↓** 버튼을 사용하여 Markdown 셀로 변환합니다. 그러면 셀에 포함된 텍스트가 서식 있는 텍스트로 표시됩니다.

    > **Markdown 셀을 사용하는 이유**: Markdown 셀을 사용하면 서식 있는 텍스트로 분석 내용을 문서화하여 Notebook을 더 읽기 쉽고 다른 사람들이 (또는 나중에 다시 볼 때 자신도) 이해하기 쉽게 만들 수 있습니다.

    ![첫 번째 셀이 Markdown으로 변경되는 Notebook 스크린샷.](./Images/copilot-fabric-notebook-markdown.png)

1. 🖉 (편집) 버튼을 사용하여 셀을 편집 모드로 전환한 다음, 아래와 같이 Markdown을 수정합니다.

    ```md
    # Explore Eurostat population data.
    Use this notebook to explore population data from Eurostat
    ```
    
    ![Markdown 셀이 있는 Fabric Notebook 화면.](Images/copilot-fabric-notebook-step-1-created.png)
    
    완료되면 셀 바깥의 Notebook 아무 곳이나 클릭하여 편집을 중지합니다.

## Lakehouse를 Notebook에 연결

Notebook에서 Lakehouse의 데이터를 사용하려면 Lakehouse를 Notebook에 연결해야 합니다. 이 연결을 통해 Notebook이 Lakehouse 저장소에서 읽고 쓸 수 있어 분석 환경과 데이터 저장소 간의 원활한 통합이 이루어집니다.

1. 왼쪽 바에서 새 작업 영역을 선택합니다. 작업 영역에 포함된 항목 목록(Lakehouse 및 Notebook 포함)이 표시됩니다.

1. Lakehouse를 선택하여 탐색기 창(Explorer pane)을 표시합니다.

1. 상단 메뉴에서 **Open notebook**, **Existing notebook**을 선택한 다음, 이전에 생성한 Notebook을 엽니다. 이제 Notebook이 탐색기 창 옆에 열려야 합니다. Lakehouses를 확장하고 Files 목록을 확장합니다. 현재 Notebook 편집기 옆에는 다음과 같이 테이블이나 파일이 나열되어 있지 않습니다.

    ![탐색기 뷰의 CSV 파일 스크린샷.](Images/copilot-fabric-notebook-step-2-lakehouse-attached.png)

    > **표시되는 내용**: 왼쪽의 탐색기 창은 Lakehouse 구조를 보여줍니다. 현재는 비어 있지만, 데이터를 로드하고 처리하면 **Files** 섹션에 파일이 나타나고 **Tables** 섹션에 테이블이 나타날 것입니다.

## 데이터 로드

이제 Copilot을 사용하여 Eurostat API에서 데이터를 다운로드합니다. 처음부터 Python 코드를 작성하는 대신, 자연어(natural language)로 수행하려는 작업을 설명하면 Copilot이 적절한 코드를 생성합니다. 이는 AI 지원 코딩의 주요 이점 중 하나를 보여줍니다. 즉, 기술적 구현 세부 사항보다는 비즈니스 로직(business logic)에 집중할 수 있습니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다. Copilot이 코드를 생성하도록 지시하려면 셀의 첫 번째 지침으로 `%%code`를 사용합니다.

    > **`%%code` 매직 명령어(magic command) 정보**: 이 특별한 지침은 Copilot에게 자연어 설명에 따라 Python 코드를 생성하도록 지시합니다. 이는 Copilot과 더 효과적으로 상호 작용하는 데 도움이 되는 여러 "매직 명령어" 중 하나입니다.

    ```copilot-prompt
    %%code
    
    Download the following file from this URL:
    
    https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/proj_23np$defaultview/?format=TSV
     
    Then write the file to the default lakehouse into a folder named temp. Create the folder if it doesn't exist yet.
    ```
    
1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

    Copilot은 환경 및 Copilot의 최신 업데이트에 따라 약간 다를 수 있는 다음 코드를 생성합니다.
    
    ![Copilot이 생성한 코드 스크린샷.](Images/copilot-fabric-notebook-step-3-code-magic.png)
    
    > **Copilot 작동 방식**: Copilot이 자연어 요청을 작동하는 Python 코드로 어떻게 번역하는지 주목하십시오. Copilot은 HTTP 요청(HTTP request)을 만들고, 파일 시스템(file system)을 처리하며, Lakehouse의 특정 위치에 데이터를 저장해야 함을 이해합니다.
    
    실행 중 예외(exceptions)가 발생할 경우를 대비하여 다음은 전체 코드입니다.
    
    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    import requests
    import os
    
    # Define the URL and the local path
    url = "https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/proj_23np$defaultview/?format=TSV"
    local_path = "/lakehouse/default/Files/temp/"
    file_name = "proj_23np.tsv"
    file_path = os.path.join(local_path, file_name)
    
    # Create the temporary directory if it doesn't exist
    if not os.path.exists(local_path):
        os.makedirs(local_path)
    
    # Download the file
    response = requests.get(url)
    response.raise_for_status()  # Check that the request was successful
    
    # Write the content to the file
    with open(file_path, "wb") as file:
        file.write(response.content)
    
    print(f"File downloaded and saved to {file_path}")
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 파일은 Lakehouse의 임시 폴더에 다운로드되어 저장되어야 합니다.

    > **참고**: Lakehouse Files를 새로 고치려면 줄임표(...)를 선택해야 할 수 있습니다.
    
    ![Lakehouse에 생성된 임시 파일 스크린샷.](Images/copilot-fabric-notebook-step-4-lakehouse-refreshed.png)

1. 이제 Lakehouse에 원시 데이터 파일이 있으므로, 분석하고 변환할 수 있도록 Spark DataFrame으로 로드해야 합니다. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    > **정보**: DataFrame은 데이터베이스의 테이블이나 스프레드시트와 유사하게 명명된 열(named columns)로 구성된 분산 데이터 컬렉션(distributed collection of data)입니다.

    ```copilot-prompt
    %%code
    
    Load the file 'Files/temp/proj_23np.tsv' into a spark dataframe.
    
    The fields are separated with a tab.
    
    Show the contents of the DataFrame using display method.
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. DataFrame은 TSV 파일의 데이터를 포함해야 합니다. 다음은 생성된 코드의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    # Load the file 'Files/temp/proj_23np.tsv' into a spark dataframe.
    # The fields have been separated with a tab.
    file_path = "Files/temp/proj_23np.tsv"
    
    spark_df = spark.read.format("csv").option("delimiter", "\t").option("header", "true").load(file_path)
    
    # Show the contents of the DataFrame using display method
    display(spark_df)
    ```

다음은 출력의 예시입니다.

| freq,projection,sex,age,unit,geo\TIME_PERIOD |      2022  |      2023  |   ...  |      2100  |
| -------------------------------------------- | ---------- | ---------- | ------ | ---------- |
|                         A,BSL,F,TOTAL,PER,AT |   4553444  |   4619179  |   ...  |   4807661  |
|                         A,BSL,F,TOTAL,PER,BE |   5883978  |   5947528  |   ...  |   6331785  |
|                         A,BSL,F,TOTAL,PER,BG |   3527626  |   3605059  |   ...  |   2543673  |
|                                          ... |       ...  |       ...  |   ...  |   5081250  |
|                         A,BSL,F,TOTAL,PER,CY |    463622  |    476907  |   ...  |    504781  |

> **데이터 구조 이해**: 첫 번째 열에는 쉼표로 구분된 여러 값(frequency, projection type, sex, age, unit, geographic location)이 포함되어 있으며, 나머지 열은 인구 값을 가진 연도를 나타냅니다. 이 구조는 통계 데이터셋(statistical datasets)에서 흔하지만 효과적인 분석을 위해 정리해야 합니다.

## 데이터 변환: 필드 분할

이제 데이터를 변환해 보겠습니다. 첫 번째 필드가 별도의 열로 분할되었는지 확인해야 합니다. 또한 올바른 데이터 형식(data types)으로 작업하고 필터링(filtering)을 적용해야 합니다.

> **필드를 분할해야 하는 이유**: 첫 번째 열에는 여러 정보(frequency, projection type, sex, age group, unit, geographic code)가 함께 연결(concatenated)되어 있습니다. 적절한 분석을 위해서는 각 정보가 자체 열에 있어야 합니다. 이 프로세스를 데이터 구조 "정규화(normalizing)"라고 합니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    From the currently loaded DataFrame, split the first field 'freq,projection,sex,age,unit,geo\TIME_PERIOD' using a comma into 6 separate fields. Then, remove the original column.
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 다음은 출력의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    from pyspark.sql.functions import split, col
    
    # Split the first field 'freq,projection,sex,age,unit,geo\TIME_PERIOD' into 6 separate fields
    spark_df = spark_df.withColumn("freq", split(col("freq,projection,sex,age,unit,geo\\TIME_PERIOD"), ",").getItem(0)) \
                       .withColumn("projection", split(col("freq,projection,sex,age,unit,geo\\TIME_PERIOD"), ",").getItem(1)) \
                       .withColumn("sex", split(col("freq,projection,sex,age,unit,geo\\TIME_PERIOD"), ",").getItem(2)) \
                       .withColumn("age", split(col("freq,projection,sex,age,unit,geo\\TIME_PERIOD"), ",").getItem(3)) \
                       .withColumn("unit", split(col("freq,projection,sex,age,unit,geo\\TIME_PERIOD"), ",").getItem(4)) \
                       .withColumn("geo", split(col("freq,projection,sex,age,unit,geo\\TIME_PERIOD"), ",").getItem(5))
    
    # Show the updated DataFrame
    display(spark_df)
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다. 테이블을 오른쪽으로 스크롤해야 테이블에 추가된 새 필드를 볼 수 있습니다.

    ![추가 필드가 포함된 결과 테이블 스크린샷.](Images/copilot-fabric-notebook-split-fields.png)

## 데이터 변환: 필드 제거

테이블의 일부 필드는 단일 고유 항목(single distinct entry)만 포함하므로 의미 있는 값을 제공하지 않습니다. 모범 사례(best practice)로서 이러한 필드를 데이터셋(dataset)에서 제거해야 합니다.

> **데이터 정리 원칙**: 고유한 값이 하나뿐인 열은 분석적 가치(analytical value)를 제공하지 않으며 데이터셋을 불필요하게 복잡하게 만들 수 있습니다. 이를 제거하면 데이터 구조(data structure)를 간소화(simplify)하고 성능(performance)을 향상시킬 수 있습니다. 이 경우 'freq'(frequency), 'age'(모든 레코드가 TOTAL 표시), 'unit'(모든 레코드가 PER for persons 표시)은 모든 행에서 일정합니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    From the currently loaded DataFrame, remove the fields 'freq', 'age', 'unit'.
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 다음은 출력의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    # Remove the fields 'freq', 'age', 'unit'
    spark_df = spark_df.drop("freq", "age", "unit")
    
    # Show the updated DataFrame
    display(spark_df)
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

## 데이터 변환: 필드 재배치

가장 중요한 식별 열(identifying columns)을 먼저 배치하여 데이터를 구성하면 읽고 이해하기가 더 쉬워집니다. 데이터 분석(data analysis)에서는 범주/차원 열(categorical/dimensional columns)(예: 예측 유형, 성별, 지리적 위치)을 숫자/측정 열(numerical/measure columns)(연도별 인구 값) 앞에 배치하는 것이 일반적인 관행입니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    From the currently loaded DataFrame, the fields 'projection', 'sex', 'geo' should be positioned first.
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 다음은 출력의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    # Reorder the DataFrame with 'projection', 'sex', 'geo' fields first
    new_column_order = ['projection', 'sex', 'geo'] + [col for col in spark_df.columns if col not in {'projection', 'sex', 'geo'}]
    spark_df = spark_df.select(new_column_order)
    
    # Show the reordered DataFrame
    display(spark_df)
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

## 데이터 변환: 값 대체

`projection` 필드에는 현재 사용자 친화적이지 않은 암호화된 코드(cryptic codes)가 포함되어 있습니다. 더 나은 가독성 및 분석(readability and analysis)을 위해 이러한 코드를 각 예측 시나리오(projection scenario)가 무엇을 나타내는지 명확하게 설명하는 설명적인 이름(descriptive names)으로 대체할 것입니다.

> **예측 시나리오 이해**: 통계 기관(statistical organizations)은 종종 미래 인구 변화를 모델링(model future population changes)하기 위해 다양한 시나리오를 사용합니다. 기준선(baseline)은 가장 가능성 있는 시나리오를 나타내며, 민감도 테스트(sensitivity tests)는 출산율(fertility rates), 사망률(mortality rates) 및 이주 패턴(migration patterns)에 대한 다른 가정 하에서 인구가 어떻게 변할 수 있는지를 보여줍니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    The 'projection' field contains codes that should be replaced with the following values:
        _'BSL' -> 'Baseline projections'.
        _'LFRT' -> 'Sensitivity test: lower fertility'.
        _'LMRT' -> 'Sensitivity test: lower mortality'.
        _'HMIGR' -> 'Sensitivity test: higher migration'.
        _'LMIGR' -> 'Sensitivity test: lower migration'.
        _'NMIGR' -> 'Sensitivity test: no migration'.
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 다음은 출력의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    from pyspark.sql.functions import when
    
    # Replace projection codes
    spark_df = spark_df.withColumn("projection", 
                                   when(spark_df["projection"] == "BSL", "Baseline projections")
                                   .when(spark_df["projection"] == "LFRT", "Sensitivity test: lower fertility")
                                   .when(spark_df["projection"] == "LMRT", "Sensitivity test: lower mortality")
                                   .when(spark_df["projection"] == "HMIGR", "Sensitivity test: higher migration")
                                   .when(spark_df["projection"] == "LMIGR", "Sensitivity test: lower migration")
                                   .when(spark_df["projection"] == "NMIGR", "Sensitivity test: no migration")
                                   .otherwise(spark_df["projection"]))
    
    # Display the updated DataFrame
    display(spark_df)
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

    ![프로젝트 필드 값이 대체된 결과 테이블 스크린샷.](Images/copilot-fabric-notebook-replace-values.png)
    
## 데이터 변환: 데이터 필터링

인구 예측 테이블에는 존재하지 않는 두 개의 국가에 대한 행이 포함되어 있습니다: EU27_2020 (*유럽 연합 27개국의 총계*) 및 EA20 (*유로존 20개국*). 데이터가 최소 세분화 수준(lowest grain)으로만 유지되도록 이 두 행을 제거해야 합니다.

> **데이터 세분화 원칙(Data granularity principle)**: 상세한 분석을 위해서는 가능한 한 가장 세분화된 수준에서 데이터로 작업하는 것이 중요합니다. 집계 값(aggregated values)(예: EU 총계)은 필요할 때 항상 계산할 수 있지만, 이를 기본 데이터셋에 포함하면 중복 계산(double-counting)이나 분석 혼란(confusion in analysis)을 초래할 수 있습니다.

![EA20 및 EU2_2020 지역이 강조 표시된 테이블 스크린샷.](Images/copilot-fabric-notebook-europe.png)

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    Filter the 'geo' field and remove values 'EA20' and 'EU27_2020' (these are not countries).
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 다음은 출력의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    # Filter out 'geo' values 'EA20' and 'EU27_2020'
    spark_df = spark_df.filter((spark_df['geo'] != 'EA20') & (spark_df['geo'] != 'EU27_2020'))
    
    # Display the filtered DataFrame
    display(spark_df)
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

    인구 프로젝트 테이블에는 다음과 같은 고유 값(distinct values)을 포함하는 'sex' 필드도 포함되어 있습니다.
    
    - M: Male (남성)
    - F: Female (여성)
    - T: Total (male + female) (총계: 남성 + 여성)

    다시 말하지만, 우리는 총계를 제거하여 데이터를 가장 낮은 수준의 세부 정보로 유지해야 합니다.

    > **총계를 제거하는 이유**: 지리적 집계(geographic aggregations)와 마찬가지로, 개별 성별 카테고리(sex categories)(남성 및 여성)만 유지하고 총계 값은 제외하고자 합니다. 이렇게 하면 더 유연한 분석이 가능합니다. 남성과 여성 값을 합산하여 총계를 얻을 수 있지만, 총계를 구성 요소(components)로 다시 분할(split totals back into components)할 수는 없습니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    Filter the 'sex' field and remove 'T' (these are totals).
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 다음은 출력의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    # Filter out 'sex' values 'T'
    spark_df = spark_df.filter(spark_df['sex'] != 'T')
    
    # Display the filtered DataFrame
    display(spark_df)
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

## 데이터 변환: 공백 제거

인구 예측 테이블의 일부 필드 이름에는 끝에 공백이 있습니다. 이러한 필드 이름에 트리밍 작업(trim operation)을 적용해야 합니다.

> **데이터 품질 문제(Data quality concern)**: 열 이름의 추가 공백은 데이터를 쿼리(querying data)하거나 시각화(creating visualizations)할 때 문제를 일으킬 수 있습니다. 특히 데이터가 외부 소스(external sources)에서 오거나 다른 시스템에서 내보내진(exported from other systems) 경우 흔한 데이터 품질 문제입니다. 공백을 제거하면 일관성(consistency)을 보장하고 나중에 디버깅하기 어려운 문제(hard-to-debug issues)를 방지할 수 있습니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    Strip spaces from all field names in the dataframe.
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 다음은 출력의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    from pyspark.sql.functions import col
    
    # Strip spaces from all field names
    spark_df = spark_df.select([col(column).alias(column.strip()) for column in spark_df.columns])
    
    # Display the updated DataFrame
    display(spark_df)
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

## 데이터 변환: 데이터 형식 변환

나중에 Power BI 또는 SQL을 사용하여 데이터를 적절하게 분석하려면 숫자 및 날짜/시간과 같은 데이터 형식(data types)이 올바르게 설정되었는지 확인해야 합니다.

> **올바른 데이터 형식의 중요성**: 텍스트 파일에서 데이터를 로드할 때 모든 열은 초기에는 문자열로 처리됩니다. 연도 열을 정수로 변환하면 수학적 연산(mathematical operations)(예: 계산 및 집계)이 가능하며 적절한 정렬(proper sorting)을 보장합니다. 이 단계는 다운스트림 분석(downstream analytics) 및 시각화 도구(visualization tools)에 매우 중요합니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    Convert the data type of all the year fields to integer.
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행하고 출력을 확인합니다. 다음은 출력의 예시입니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    from pyspark.sql.functions import col
    
    # Convert the data type of all the year fields to integer
    year_columns = [col(column).cast("int") for column in spark_df.columns if column.strip().isdigit()]
    spark_df = spark_df.select(*spark_df.columns[:3], *year_columns)
    
    # Display the updated DataFrame
    display(spark_df)
    ```
    
1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다. 다음은 출력의 예시입니다(간결성을 위해 열과 행을 제거했습니다).

|          projection|sex|geo|    2022|    2023|     ...|    2100|
|--------------------|---|---|--------|--------|--------|--------| 
|Baseline projections|  F| AT| 4553444| 4619179|     ...| 4807661|
|Baseline projections|  F| BE| 5883978| 5947528|     ...| 6331785|
|Baseline projections|  F| BG| 3527626| 3605059|     ...| 2543673|
|...                 |...|...|     ...|     ...|     ...|     ...|
|Baseline projections|  F| LU|  320333|  329401|     ...|  498954|

>[!TIP]
> 모든 열을 확인하려면 테이블을 오른쪽으로 스크롤해야 할 수 있습니다.

## 데이터 저장

다음으로, 변환된 데이터를 Lakehouse에 저장하고자 합니다.

> **변환된 데이터를 저장하는 이유**: 이 모든 데이터 정리 및 변환 작업을 마친 후에는 결과를 유지(persist the results)하고자 합니다. 데이터를 Lakehouse에 테이블로 저장하면 우리와 다른 사람들이 변환 프로세스를 반복할 필요 없이 이 정리된 데이터셋을 다양한 분석 시나리오(analytics scenarios)에 사용할 수 있습니다. 또한 Microsoft Fabric 에코시스템(Microsoft Fabric ecosystem)의 다른 도구(예: Power BI, SQL Analytics Endpoint 및 Data Factory)가 이 데이터를 사용할 수 있도록 합니다.

1. Notebook에 새 셀을 만들고 다음 지침을 복사합니다.

    ```copilot-prompt
    %%code
    
    Save the dataframe as a new table named 'Population' in the default lakehouse.
    ```
    
1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다. Copilot은 환경 및 Copilot의 최신 업데이트에 따라 약간 다를 수 있는 코드를 생성합니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    spark_df.write.format("delta").saveAsTable("Population")
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

## 유효성 검사: 질문하기

이제 데이터 분석(data analysis)을 위한 Copilot의 강력한 기능을 살펴보겠습니다. 복잡한 SQL 쿼리(SQL queries) 또는 시각화 코드를 처음부터 작성하는 대신, 데이터에 대한 자연어(natural language) 질문을 Copilot에 하면 Copilot이 이에 답하는 적절한 코드를 생성합니다.

1. 데이터가 올바르게 저장되었는지 확인하려면 Lakehouse의 테이블을 확장하고 내용을 확인합니다 (줄임표(...)를 선택하여 Tables 폴더를 새로 고쳐야 할 수 있습니다).

    ![이제 'Population'이라는 새 테이블을 포함하는 Lakehouse 스크린샷.](Images/copilot-fabric-notebook-step-5-lakehouse-refreshed.png)

1. 홈 리본에서 Copilot 옵션을 선택합니다.

    > **Copilot 채팅 인터페이스(Copilot chat interface)**: Copilot 패널은 자연어로 데이터에 대한 질문을 할 수 있는 대화형 인터페이스를 제공합니다. 분석을 위한 코드를 생성하고, 시각화를 만들고, 데이터셋의 패턴(patterns in your dataset)을 탐색하는 데 도움을 줄 수 있습니다.

    ![Copilot 패널이 열려 있는 Notebook 스크린샷.](Images/copilot-fabric-notebook-step-6-copilot-pane.png)

1. 다음 프롬프트를 입력합니다.

    ```copilot-prompt
    What are the projected population trends for geo BE  from 2020 to 2050 as a line chart visualization. Make sure to sum up male and female numbers. Use only existing columns from the population table. Perform the query using SQL.
    ```

    > **이것이 보여주는 것**: 이 프롬프트는 Copilot이 컨텍스트(context)(우리의 Population 테이블)를 이해하고, SQL 쿼리(SQL queries)를 생성하며, 시각화(visualizations)를 만들 수 있는 능력을 보여줍니다. 단일 요청에서 데이터 쿼리와 시각화를 결합하기 때문에 특히 강력합니다.

1. 생성된 출력을 확인합니다. 이는 환경 및 Copilot의 최신 업데이트에 따라 약간 다를 수 있습니다. 코드 조각을 새 셀에 복사합니다.

    ```python
    #### ATTENTION: AI-generated code can include errors or operations you didn't intend. Review the code in this cell carefully before running it.
    
    import plotly.graph_objs as go
    
    # Perform the SQL query to get projected population trends for geo BE, summing up male and female numbers
    result = spark.sql(
        """
        SELECT projection, sex, geo, SUM(`2022`) as `2022`, SUM(`2023`) as `2023`, SUM(`2025`) as `2025`,
               SUM(`2030`) as `2030`, SUM(`2035`) as `2035`, SUM(`2040`) as `2040`,
               SUM(`2045`) as `2045`, SUM(`2050`) as `2050`
        FROM Population
        WHERE geo = 'BE' AND projection = 'Baseline projections'
        GROUP BY projection, sex, geo
        """
    )
    df = result.groupBy("projection").sum()
    df = df.orderBy("projection").toPandas()
    
    # Extract data for the line chart
    years = df.columns[1:].tolist()
    values = df.iloc[0, 1:].tolist()
    
    # Create the plot
    fig = go.Figure()
    fig.add_trace(go.Scatter(x=years, y=values, mode='lines+markers', name='Projected Population'))
    
    # Update the layout
    fig.update_layout(
        title='Projected Population Trends for Geo BE (Belgium) from 2022 to 2050',
        xaxis_title='Year',
        yaxis_title='Population',
        template='plotly_dark'
    )
    
    # Display the plot
    fig.show()
    ```

1. 셀 왼쪽의 ▷ **Run cell**을 선택하여 코드를 실행합니다.

    생성된 차트를 확인합니다.
    
    ![생성된 꺾은선형 차트가 있는 Notebook 스크린샷.](Images/copilot-fabric-notebook-step-7-line-chart.png)
    
    > **달성한 성과**: Copilot을 성공적으로 사용하여 벨기에의 시간 경과에 따른 인구 추세를 보여주는 시각화를 생성했습니다. 이는 AI 지원(AI assistance)을 통해 데이터 수집(data ingestion), 변환(transformation), 저장(storage) 및 분석(analysis)에 이르는 엔드 투 엔드 데이터 엔지니어링 워크플로(end-to-end data engineering workflow)를 보여줍니다.

## 리소스 정리

이 실습에서는 Copilot과 Spark를 사용하여 Microsoft Fabric에서 데이터로 작업하는 방법을 배웠습니다.

데이터 탐색을 마쳤다면 Spark 세션(Spark session)을 종료하고 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1.	Notebook 메뉴에서 **Stop session**을 선택하여 Spark 세션을 종료합니다.
1.	왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
1.	**Workspace settings**를 선택하고 **General** 섹션에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.
1.	**Delete**를 선택하여 작업 영역을 삭제합니다.