---
lab:
    title: 'Ingest data with a pipeline in Microsoft Fabric'
    module: 'Use Data Factory pipelines in Microsoft Fabric'
---

# Microsoft Fabric에서 Pipeline을 사용하여 데이터 수집

Lakehouse는 클라우드 규모 분석 솔루션을 위한 일반적인 분석 데이터 저장소입니다. 데이터 엔지니어의 핵심 작업 중 하나는 여러 운영 데이터 원본에서 Lakehouse로 데이터를 수집하는 것을 구현하고 관리하는 것입니다. Microsoft Fabric에서 *추출, 변환, 로드* (ETL) 또는 *추출, 로드, 변환* (ELT)과 같은 데이터 수집 솔루션을 *Pipeline* 생성을 통해 구현할 수 있습니다.

Fabric은 Apache Spark도 지원하여 코드를 작성하고 실행하여 대규모로 데이터를 처리할 수 있도록 합니다. Fabric의 Pipeline 및 Spark 기능을 결합하여 외부 원본에서 Lakehouse가 기반으로 하는 OneLake 저장소로 데이터를 복사한 다음 Spark 코드를 사용하여 사용자 지정 데이터 변환을 수행한 후 분석을 위해 테이블에 로드하는 복잡한 데이터 수집 논리를 구현할 수 있습니다.

이 랩을 완료하는 데 약 **45분**이 소요됩니다.

> [!참고]
> 이 연습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)에 액세스해야 합니다.

## 작업 영역 생성

Fabric에서 데이터를 작업하기 전에 Fabric 평가판이 활성화된 작업 영역을 생성합니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric-developer`의 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric-developer)로 이동하여 Fabric 자격 증명으로 로그인합니다.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다 (아이콘은 &#128455;와 비슷하게 생겼습니다).
1. 원하는 이름으로 새 작업 영역을 생성하고, **고급** 섹션에서 Fabric Capacity (Rule 2)가 포함된 라이선스 모드(*평가판*, *Premium* 또는 *Fabric*)를 선택합니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Lakehouse 생성

이제 작업 영역이 생겼으므로 데이터를 수집할 데이터 Lakehouse를 생성할 시간입니다.

1. 왼쪽 메뉴 모음에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *데이터 엔지니어링* 섹션에서 **Lakehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다. "Lakehouse schemas (Public Preview)" 옵션이 비활성화되어 있는지 확인합니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않은 경우 먼저 줄임표 (**...**) 옵션을 선택해야 합니다.

    1분 정도 지나면 **테이블**이나 **파일**이 없는 새 Lakehouse가 생성됩니다.

1. 왼쪽의 **탐색기** 창에서 **파일** 노드의 **...** 메뉴에서 **새 하위 폴더**를 선택하고 **new_data**라는 이름의 하위 폴더를 생성합니다.

## Pipeline 생성

데이터를 수집하는 간단한 방법은 Pipeline에서 **데이터 복사** 활동을 사용하여 원본에서 데이터를 추출하고 Lakehouse의 파일로 복사하는 것입니다.

1. Lakehouse의 **홈** 페이지에서 **데이터 가져오기**를 선택한 다음 **새 데이터 Pipeline**을 선택하고 `Ingest Sales Data`라는 이름의 새 데이터 Pipeline을 생성합니다.
1. **데이터 복사** 마법사가 자동으로 열리지 않으면 Pipeline 편집기 페이지에서 **데이터 복사 > 복사 도우미 사용**을 선택합니다.
1. **데이터 복사** 마법사의 **데이터 원본 선택** 페이지에서 검색창에 HTTP를 입력한 다음 **새 원본** 섹션에서 **HTTP**를 선택합니다.

    ![Screenshot of the Choose data source page.](./Images/choose-data-source.png)

1. **데이터 원본에 연결** 창에서 데이터 원본 연결을 위해 다음 설정을 입력합니다.
    - **URL**: `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv`
    - **연결**: 새 연결 만들기
    - **연결 이름**: *고유한 이름을 지정하세요*
    - **데이터 게이트웨이**: (없음)
    - **인증 종류**: 익명
1. **다음**을 선택합니다. 그런 다음 다음 설정이 선택되어 있는지 확인합니다.
    - **상대 URL**: *비워둡니다*
    - **요청 메서드**: GET
    - **추가 헤더**: *비워둡니다*
    - **이진 복사**: <u>선택 안 됨</u>
    - **요청 시간 제한**: *비워둡니다*
    - **최대 동시 연결**: *비워둡니다*
1. **다음**을 선택하고, 데이터 샘플링을 기다린 다음 다음 설정이 선택되어 있는지 확인합니다.
    - **파일 형식**: DelimitedText
    - **열 구분 기호**: 쉼표 (,)
    - **행 구분 기호**: 줄 바꿈 (\n)
    - **첫 행을 헤더로**: 선택됨
    - **압축 유형**: 없음
1. **데이터 미리 보기**를 선택하여 수집될 데이터의 샘플을 확인합니다. 그런 다음 데이터 미리 보기를 닫고 **다음**을 선택합니다.
1. **데이터 대상에 연결** 페이지에서 다음 데이터 대상 옵션을 설정한 다음 **다음**을 선택합니다.
    - **루트 폴더**: Files
    - **폴더 경로 이름**: new_data
    - **파일 이름**: sales.csv
    - **복사 동작**: 없음
1. 다음 파일 형식 옵션을 설정한 다음 **다음**을 선택합니다.
    - **파일 형식**: DelimitedText
    - **열 구분 기호**: 쉼표 (,)
    - **행 구분 기호**: 줄 바꿈 (\n)
    - **파일에 헤더 추가**: 선택됨
    - **압축 유형**: 없음
1. **복사 요약** 페이지에서 복사 작업의 세부 정보를 검토한 다음 **저장 + 실행**을 선택합니다.

    다음 그림과 같이 **데이터 복사** 활동을 포함하는 새 Pipeline이 생성됩니다.

    ![Screenshot of a pipeline with a Copy Data activity.](./Images/copy-data-pipeline.png)

1. Pipeline이 실행되기 시작하면 Pipeline 디자이너 아래의 **출력** 창에서 상태를 모니터링할 수 있습니다. **&#8635;** (*새로 고침*) 아이콘을 사용하여 상태를 새로 고치고 성공할 때까지 기다립니다.
1. 왼쪽 메뉴 모음에서 Lakehouse를 선택합니다.
1. **홈** 페이지의 **탐색기** 창에서 **파일**을 확장하고 **new_data** 폴더를 선택하여 **sales.csv** 파일이 복사되었는지 확인합니다.

## Notebook 생성

1. Lakehouse의 **홈** 페이지에 있는 **Notebook 열기** 메뉴에서 **새 Notebook**을 선택합니다.

    몇 초 후에 단일 *셀*을 포함하는 새 Notebook이 열립니다. Notebook은 *코드* 또는 *Markdown* (서식 있는 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

2. Notebook에 있는 기존 셀(간단한 코드를 포함함)을 선택한 다음, 기본 코드를 다음 변수 선언으로 바꿉니다.

    ```python
   table_name = "sales"
    ```

3. 셀의 **...** 메뉴(오른쪽 상단)에서 **매개 변수 셀 전환**을 선택합니다. 이 설정은 Notebook을 Pipeline에서 실행할 때 셀에 선언된 변수를 매개 변수로 처리하도록 구성합니다.

4. 매개 변수 셀 아래에서 **+ 코드** 버튼을 사용하여 새 코드 셀을 추가합니다. 그런 다음 다음 코드를 추가합니다.

    ```python
   from pyspark.sql.functions import *

   # Read the new sales data
   df = spark.read.format("csv").option("header","true").load("Files/new_data/*.csv")

   ## Add month and year columns
   df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

   # Derive FirstName and LastName columns
   df = df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)).withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

   # Filter and reorder columns
   df = df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month", "FirstName", "LastName", "EmailAddress", "Item", "Quantity", "UnitPrice", "TaxAmount"]

   # Load the data into a table
   df.write.format("delta").mode("append").saveAsTable(table_name)
    ```

    이 코드는 **데이터 복사** 활동에 의해 수집된 sales.csv 파일에서 데이터를 로드하고, 일부 변환 논리를 적용하며, 변환된 데이터를 테이블로 저장합니다. 테이블이 이미 존재하는 경우 데이터를 추가합니다.

5. Notebook이 이와 유사하게 보이는지 확인한 다음, 도구 모음의 **&#9655; 모두 실행** 버튼을 사용하여 포함된 모든 셀을 실행합니다.

    ![Screenshot of a notebook with a parameters cell and code to transform data.](./Images/notebook.png)

    > **참고**: 이 세션에서 Spark 코드를 처음 실행하는 것이므로 Spark Pool (Rule 2)을 시작해야 합니다. 따라서 첫 번째 셀이 완료되는 데 1분 정도 걸릴 수 있습니다.

6. Notebook 실행이 완료되면 왼쪽의 **탐색기** 창에 있는 **테이블**의 **...** 메뉴에서 **새로 고침**을 선택하고 **sales** 테이블이 생성되었는지 확인합니다.
7. Notebook 메뉴 모음에서 ⚙️ **설정** 아이콘을 사용하여 Notebook 설정을 확인합니다. 그런 다음 Notebook의 **이름**을 `Load Sales`로 설정하고 설정 창을 닫습니다.
8. 왼쪽 허브 메뉴 모음에서 Lakehouse를 선택합니다.
9. **탐색기** 창에서 보기를 새로 고칩니다. 그런 다음 **테이블**을 확장하고 **sales** 테이블을 선택하여 포함된 데이터의 미리 보기를 확인합니다.

## Pipeline 수정

이제 데이터를 변환하고 테이블에 로드하는 Notebook을 구현했으므로, Notebook을 Pipeline에 통합하여 재사용 가능한 ETL 프로세스를 생성할 수 있습니다.

1. 왼쪽 허브 메뉴 모음에서 이전에 생성한 **Ingest Sales Data** Pipeline을 선택합니다.
2. **활동** 탭의 **모든 활동** 목록에서 **데이터 삭제**를 선택합니다. 그런 다음 새 **데이터 삭제** 활동을 **데이터 복사** 활동의 왼쪽에 배치하고, 다음 그림과 같이 해당 **완료 시** 출력을 **데이터 복사** 활동에 연결합니다.

    ![Screenshot of a pipeline with Delete data and Copy data activities.](./Images/delete-data-activity.png)

3. **데이터 삭제** 활동을 선택하고 디자인 캔버스 아래 창에서 다음 속성을 설정합니다.
    - **일반**:
        - **이름**: `Delete old files`
    - **원본**
        - **연결**: *사용자의 Lakehouse*
        - **파일 경로 유형**: 와일드카드 파일 경로
        - **폴더 경로**: Files / **new_data**
        - **와일드카드 파일 이름**: `*.csv`        
        - **재귀적으로**: *선택됨*
    - **로그 설정**:
        - **로깅 사용**: *<u>선택 안 됨</u>*

    이 설정은 **sales.csv** 파일을 복사하기 전에 기존의 모든 .csv 파일이 삭제되도록 합니다.

4. Pipeline 디자이너의 **활동** 탭에서 **Notebook**을 선택하여 Pipeline에 **Notebook** 활동을 추가합니다.
5. **데이터 복사** 활동을 선택한 다음, 다음 그림과 같이 해당 **완료 시** 출력을 **Notebook** 활동에 연결합니다.

    ![Screenshot of a pipeline with Copy Data and Notebook activities.](./Images/pipeline.png)

6. **Notebook** 활동을 선택한 다음, 디자인 캔버스 아래 창에서 다음 속성을 설정합니다.
    - **일반**:
        - **이름**: `Load Sales notebook`
    - **설정**:
        - **Notebook**: Load Sales
        - **기본 매개 변수**: *다음 속성을 사용하여 새 매개 변수를 추가합니다:*
            
            | Name | Type | Value |
            | -- | -- | -- |
            | table_name | String | new_sales |

    **table_name** 매개 변수는 Notebook으로 전달되어 매개 변수 셀의 **table_name** 변수에 할당된 기본값을 재정의합니다.

7. **홈** 탭에서 **&#128427;** (*저장*) 아이콘을 사용하여 Pipeline을 저장합니다. 그런 다음 **&#9655; 실행** 버튼을 사용하여 Pipeline을 실행하고 모든 활동이 완료될 때까지 기다립니다.

    ![Screenshot of a pipeline with a Dataflow activity.](./Images/pipeline-run.png)

> 참고: *Spark SQL 쿼리는 Lakehouse 컨텍스트에서만 가능합니다. 계속하려면 Lakehouse를 연결하세요.*라는 오류 메시지가 나타나는 경우: Notebook을 열고, 왼쪽 창에서 생성한 Lakehouse를 선택한 다음 **모든 Lakehouse 제거**를 선택하고 다시 추가합니다. Pipeline 디자이너로 돌아가서 **&#9655; 실행**을 선택합니다.

8. 포털의 왼쪽 가장자리에 있는 허브 메뉴 모음에서 Lakehouse를 선택합니다.
9. **탐색기** 창에서 **테이블**을 확장하고 **new_sales** 테이블을 선택하여 포함된 데이터의 미리 보기를 확인합니다. 이 테이블은 Notebook이 Pipeline에 의해 실행될 때 생성되었습니다.

이 연습에서는 Pipeline을 사용하여 외부 원본에서 Lakehouse로 데이터를 복사한 다음 Spark Notebook을 사용하여 데이터를 변환하고 테이블에 로드하는 데이터 수집 솔루션을 구현했습니다.

## 리소스 정리

이 연습에서는 Microsoft Fabric에서 Pipeline을 구현하는 방법을 배웠습니다.

Lakehouse 탐색을 마쳤다면, 이 연습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
1. **작업 영역 설정**을 선택하고 **일반** 섹션에서 아래로 스크롤하여 **이 작업 영역 제거**를 선택합니다.
1. **삭제**를 선택하여 작업 영역을 삭제합니다.