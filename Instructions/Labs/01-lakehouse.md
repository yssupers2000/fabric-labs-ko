---
lab:
    title: 'Create a Microsoft Fabric Lakehouse'
    module: 'Get started with lakehouses in Microsoft Fabric'
---

# Microsoft Fabric Lakehouse 만들기

대규모 데이터 분석 솔루션은 전통적으로 데이터가 관계형 테이블에 저장되고 SQL을 사용하여 쿼리되는 *Data Warehouse*를 기반으로 구축되었습니다. "빅 데이터"(새로운 데이터 자산의 높은 *볼륨*, *다양성*, *속도*를 특징으로 함)의 성장과 저비용 스토리지 및 클라우드 규모 분산 컴퓨팅 기술의 가용성은 분석 데이터 저장에 대한 대안적인 접근 방식인 *Data Lake*로 이어졌습니다. Data Lake에서는 저장 시 고정된 스키마를 부과하지 않고 데이터를 파일로 저장합니다. 데이터 엔지니어와 분석가들은 점점 더 두 가지 접근 방식의 최상의 기능을 결합하여 *Lakehouse*에서 이점을 얻으려고 합니다. Lakehouse에서는 데이터가 Data Lake의 파일에 저장되고 관계형 스키마가 메타데이터 계층으로 적용되어 기존 SQL 시맨틱스(의미론)를 사용하여 쿼리할 수 있습니다.

Microsoft Fabric에서 Lakehouse는 *OneLake* 스토어(Azure Data Lake Store Gen2 기반)에서 고도로 확장 가능한 파일 스토리지를 제공하며, 오픈 소스 *Delta Lake* 테이블 형식 기반의 테이블 및 뷰와 같은 관계형 개체를 위한 메타스토어를 함께 제공합니다. Delta Lake를 사용하면 Lakehouse에서 테이블의 스키마를 정의하고 SQL을 사용하여 쿼리할 수 있습니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

Fabric에서 데이터를 사용하기 전에 Fabric 평가판이 활성화된 작업 영역을 만드세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces**(아이콘은 &#128455;와 유사하게 생겼습니다)를 선택합니다.
3.  선택한 이름으로 새 작업 영역을 만들고, **Advanced** 섹션에서 Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium* 또는 *Fabric*)를 선택합니다.
4.  새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## Lakehouse 만들기

이제 작업 영역이 있으니, 데이터 파일을 위한 Lakehouse를 만들 차례입니다.

1.  왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션 아래에서 **Lakehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다. "Lakehouse schemas (Public Preview)" 옵션이 비활성화되어 있는지 확인합니다.

    >**참고**: **Create** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    잠시 후, 새 Lakehouse가 생성됩니다:

    ![새 Lakehouse 스크린샷.](./Images/new-lakehouse.png)

2.  새 Lakehouse를 확인하고, 왼쪽의 **Lakehouse explorer** 창을 통해 Lakehouse의 테이블과 파일을 탐색할 수 있음을 확인합니다:
    -   **Tables** 폴더에는 SQL 시맨틱스(의미론)를 사용하여 쿼리할 수 있는 테이블이 포함되어 있습니다. Microsoft Fabric Lakehouse의 테이블은 Apache Spark에서 일반적으로 사용되는 오픈 소스 *Delta Lake* 파일 형식을 기반으로 합니다.
    -   **Files** 폴더에는 관리형 Delta 테이블과 연결되지 않은 Lakehouse용 OneLake 스토리지의 데이터 파일이 포함되어 있습니다. 이 폴더에 *shortcuts*(바로 가기)을 만들어 외부에 저장된 데이터를 참조할 수도 있습니다.

    현재 Lakehouse에는 테이블이나 파일이 없습니다.

## 파일 업로드

Fabric은 외부 소스에서 데이터를 복사하는 Pipeline에 대한 기본 제공 지원 및 Power Query 기반의 시각적 도구를 사용하여 정의할 수 있는 Data Flow (Gen 2)를 포함하여 Lakehouse에 데이터를 로드하는 여러 가지 방법을 제공합니다. 그러나 소량의 데이터를 수집하는 가장 간단한 방법 중 하나는 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에서 파일 또는 폴더를 업로드하는 것입니다.

1.  `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv`에서 [sales.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv) 파일을 다운로드하여 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에 **sales.csv**로 저장합니다.

    > **참고**: 파일을 다운로드하려면 브라우저에서 새 탭을 열고 URL을 붙여넣으세요. 데이터가 포함된 페이지의 아무 곳이나 마우스 오른쪽 버튼으로 클릭하고 **다른 이름으로 저장**을 선택하여 페이지를 CSV 파일로 저장합니다.

2.  Lakehouse가 포함된 웹 브라우저 탭으로 돌아가서 **Explorer** 창의 **Files** 폴더에 있는 **...** 메뉴에서 **New subfolder**를 선택하고 **data**라는 하위 폴더를 만듭니다.
3.  새 **data** 폴더의 **...** 메뉴에서 **Upload** 및 **Upload files**를 선택한 다음 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에서 **sales.csv** 파일을 업로드합니다.
4.  파일이 업로드된 후 **Files/data** 폴더를 선택하고 다음 그림과 같이 **sales.csv** 파일이 업로드되었는지 확인합니다:

    ![Lakehouse에 업로드된 sales.csv 파일 스크린샷.](./Images/uploaded-sales-file.png)

5.  **sales.csv** 파일을 선택하여 내용 미리보기를 확인합니다.

## Shortcuts 탐색

많은 시나리오에서 Lakehouse에서 작업해야 하는 데이터는 다른 위치에 저장될 수 있습니다. Lakehouse용 OneLake 스토리지에 데이터를 수집하는 여러 가지 방법이 있지만, 또 다른 옵션은 대신 *shortcut*(바로 가기)을 만드는 것입니다. Shortcuts을 사용하면 데이터를 복사하는 것과 관련된 오버헤드(부담) 및 데이터 불일치 위험 없이 외부 소스 데이터를 분석 솔루션에 포함할 수 있습니다.

1.  **Files** 폴더의 **...** 메뉴에서 **New shortcut**을 선택합니다.
2.  shortcuts(바로 가기)에 사용할 수 있는 데이터 소스 유형을 확인합니다. 그런 다음 shortcut(바로 가기)을 만들지 않고 **New shortcut** 대화 상자를 닫습니다.

## 파일 데이터를 테이블로 로드

업로드한 sales 데이터는 파일에 있으며, 데이터 분석가와 엔지니어는 Apache Spark 코드를 사용하여 직접 작업할 수 있습니다. 그러나 많은 시나리오에서 SQL을 사용하여 쿼리할 수 있도록 파일의 데이터를 테이블로 로드해야 할 수도 있습니다.

1.  **Explorer** 창에서 **Files/data** 폴더를 선택하여 포함된 **sales.csv** 파일을 볼 수 있도록 합니다.
2.  **sales.csv** 파일의 **...** 메뉴에서 **Load to Tables** > **New table**을 선택합니다.
3.  **Load to table** 대화 상자에서 테이블 이름을 **sales**로 설정하고 로드 작업을 확인합니다. 그런 다음 테이블이 생성되고 로드될 때까지 기다립니다.

    > **팁**: **sales** 테이블이 자동으로 나타나지 않으면 **Tables** 폴더의 **...** 메뉴에서 **Refresh**를 선택합니다.

4.  **Explorer** 창에서 생성된 **sales** 테이블을 선택하여 데이터를 확인합니다.

    ![테이블 미리보기 스크린샷.](./Images/table-preview.png)

5.  **sales** 테이블의 **...** 메뉴에서 **View files**를 선택하여 이 테이블의 기본 파일을 확인합니다.

    ![Delta 테이블 파일 스크린샷.](./Images/delta-table-files.png)

    Delta 테이블의 파일은 *Parquet* 형식으로 저장되며, 테이블에 적용된 트랜잭션(거래) 세부 정보가 기록되는 **_delta_log**라는 하위 폴더를 포함합니다.

## SQL을 사용하여 테이블 쿼리

Lakehouse를 만들고 그 안에 테이블을 정의하면, SQL `SELECT` 문을 사용하여 테이블을 쿼리할 수 있는 SQL endpoint가 자동으로 생성됩니다.

1.  Lakehouse 페이지의 오른쪽 상단에서 **Lakehouse**에서 **SQL analytics endpoint**로 전환합니다. 그런 다음 Lakehouse의 SQL analytics endpoint가 테이블을 쿼리할 수 있는 시각적 인터페이스로 열릴 때까지 잠시 기다립니다.
2.  **New SQL query** 버튼을 사용하여 새 쿼리 편집기를 열고 다음 SQL query를 입력합니다.

    ```sql
   SELECT Item, SUM(Quantity * UnitPrice) AS Revenue
   FROM sales
   GROUP BY Item
   ORDER BY Revenue DESC;
    ```
> **참고**: 랩 VM에 있고 SQL query를 입력하는 데 문제가 있는 경우, `https://github.com/MicrosoftLearning/mslearn-fabric/raw/main/Allfiles/Labs/01/Assets/01-Snippets.txt`에서 [01-Snippets.txt](https://github.com/MicrosoftLearning/mslearn-fabric/raw/main/Allfiles/Labs/01/Assets/01-Snippets.txt) 파일을 다운로드하여 VM에 저장할 수 있습니다. 그런 다음 텍스트 파일에서 query를 복사할 수 있습니다.

3.  **&#9655; Run** 버튼을 사용하여 query를 실행하고 결과를 확인합니다. 결과는 각 제품의 총 Revenue를 보여주어야 합니다.

    ![SQL query와 결과 스크린샷.](./Images/sql-query.png)

## 시각적 쿼리 만들기

많은 데이터 전문가들이 SQL에 익숙하지만, Power BI 경험이 있는 데이터 분석가들은 Power Query 기술을 적용하여 시각적 쿼리를 만들 수 있습니다.

1.  도구 모음에서 **New SQL query** 옵션을 확장하고 **New visual query**를 선택합니다.
2.  **sales** 테이블을 새로 열리는 시각적 쿼리 편집기 창으로 끌어다 놓아 다음 그림과 같이 Power Query를 만듭니다:

    ![시각적 쿼리 스크린샷.](./Images/visual-query.png)

3.  **Manage columns** 메뉴에서 **Choose columns**를 선택합니다. 그런 다음 **SalesOrderNumber** 및 **SalesOrderLineNumber** 열만 선택합니다.

    ![열 선택 대화 상자 스크린샷.](./Images/choose-columns.png)

4.  **Transform** 메뉴에서 **Group by**를 선택합니다. 그런 다음 다음 **Basic** 설정을 사용하여 데이터를 그룹화합니다:

    -   **Group by**: SalesOrderNumber
    -   **New column name**: LineItems
    -   **Operation**: Count distinct values
    -   **Column**: SalesOrderLineNumber

    완료되면 시각적 쿼리 아래의 결과 창에 각 판매 주문에 대한 LineItems(항목 수)가 표시됩니다.

    ![결과가 포함된 시각적 쿼리 스크린샷.](./Images/visual-query-results.png)

## 리소스 정리

이 실습에서는 Lakehouse를 만들고 데이터를 가져왔습니다. Lakehouse가 OneLake 데이터 스토어에 저장된 파일과 테이블로 구성되는 방식을 확인했습니다. 관리형 테이블은 SQL을 사용하여 쿼리할 수 있으며, 데이터 시각화를 지원하기 위한 기본 Semantic Model(의미 모델)에 포함됩니다.

Lakehouse 탐색을 마쳤다면, 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1.  왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2.  도구 모음에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.