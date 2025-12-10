---
lab:
    title: 'Analyze data in a data warehouse'
    module: 'Get started with data warehouses in Microsoft Fabric'
---

# 데이터 웨어하우스에서 데이터 분석

Microsoft Fabric에서 Data Warehouse는 대규모 분석을 위한 관계형 데이터베이스를 제공합니다. Lakehouse에 정의된 테이블의 기본 읽기 전용 SQL endpoint와 달리, Data Warehouse는 테이블에 데이터를 삽입, 업데이트 및 삭제하는 기능을 포함하여 완전한 SQL 의미 체계를 제공합니다.

이 실습은 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 체험판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## workspace 생성

Fabric에서 데이터 작업을 시작하기 전에 Fabric 체험판이 활성화된 workspace를 만드세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인하세요.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 유사하게 생겼습니다)를 선택하세요.
3.  원하는 이름으로 새 workspace를 만들고, Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택하세요.
4.  새 workspace가 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Data Warehouse 생성

이제 workspace가 있으므로 Data Warehouse를 만들 시간입니다.

1.  왼쪽 메뉴 바에서 **Create**를 선택하세요. *New* 페이지의 *Data Warehouse* 섹션에서 **Warehouse**를 선택하세요. 원하는 고유한 이름을 지정하세요.

    >**참고**: **Create** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    1분 정도 후에 새 warehouse가 생성됩니다.

    ![Screenshot of a new warehouse.](./Images/new-data-warehouse2.png)

## 테이블 생성 및 데이터 삽입

Warehouse는 테이블 및 기타 개체(objects)를 정의할 수 있는 관계형 데이터베이스입니다.

1.  새 warehouse에서 **T-SQL** 타일(tile)을 선택하고 다음 CREATE TABLE 문을 사용하세요.

    ```sql
   CREATE TABLE dbo.DimProduct
   (
       ProductKey INTEGER NOT NULL,
       ProductAltKey VARCHAR(25) NULL,
       ProductName VARCHAR(50) NOT NULL,
       Category VARCHAR(50) NULL,
       ListPrice DECIMAL(5,2) NULL
   );
   GO
    ```

2.  **&#9655; Run** 버튼을 사용하여 SQL 스크립트를 실행하면 Data Warehouse의 **dbo** schema에 **DimProduct**라는 새 테이블이 생성됩니다.
3.  도구 모음에서 **Refresh** 버튼을 사용하여 보기를 새로 고치세요. 그런 다음, **Explorer** pane에서 **Schemas** > **dbo** > **Tables**를 확장하고 **DimProduct** 테이블이 생성되었는지 확인하세요.
4.  **Home** 메뉴 탭에서 **New SQL Query** 버튼을 사용하여 새 쿼리를 생성하고 다음 INSERT 문을 입력하세요.

    ```sql
   INSERT INTO dbo.DimProduct
   VALUES
   (1, 'RING1', 'Bicycle bell', 'Accessories', 5.99),
   (2, 'BRITE1', 'Front light', 'Accessories', 15.49),
   (3, 'BRITE2', 'Rear light', 'Accessories', 15.49);
   GO
    ```

5.  새 쿼리를 실행하여 **DimProduct** 테이블에 세 개의 행을 삽입하세요.
6.  쿼리가 완료되면 **Explorer** pane에서 **DimProduct** 테이블을 선택하고 세 개의 행이 테이블에 추가되었는지 확인하세요.
7.  **Home** 메뉴 탭에서 **New SQL Query** 버튼을 사용하여 새 쿼리를 생성하세요. 그런 다음 `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/create-dw.txt`에서 Transact-SQL 코드를 복사하여 새 query pane에 붙여넣으세요.
8.  쿼리를 실행하면 간단한 Data Warehouse schema가 생성되고 일부 데이터가 로드됩니다. 스크립트 실행에는 약 30초가 소요됩니다.
9.  도구 모음에서 **Refresh** 버튼을 사용하여 보기를 새로 고치세요. 그런 다음 **Explorer** pane에서 Data Warehouse의 **dbo** schema에 이제 다음 네 개의 테이블이 포함되어 있는지 확인하세요.
    -   **DimCustomer**
    -   **DimDate**
    -   **DimProduct**
    -   **FactSalesOrder**

    > **팁**: schema 로드에 시간이 오래 걸리면 브라우저 페이지를 새로 고치세요.

## Data Warehouse 테이블 쿼리

Data Warehouse는 관계형 데이터베이스이므로 SQL을 사용하여 해당 테이블을 쿼리할 수 있습니다.

### Fact 및 Dimension 테이블 쿼리

관계형 Data Warehouse의 대부분의 쿼리는 관련 테이블(JOIN 절 사용)에서 데이터 집계 및 그룹화(집계 함수 및 GROUP BY 절 사용)를 포함합니다.

1.  warehouse가 있는 브라우저 탭으로 돌아가서 새 SQL Query를 생성하고 다음 코드를 실행하세요.

    ```sql
   SELECT  d.[Year] AS CalendarYear,
            d.[Month] AS MonthOfYear,
            d.MonthName AS MonthName,
           SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   GROUP BY d.[Year], d.[Month], d.MonthName
   ORDER BY CalendarYear, MonthOfYear;
    ```

    날짜 dimension의 특성을 통해 fact 테이블의 측정값(measures)을 여러 계층 수준(이 경우 연도 및 월)으로 집계할 수 있습니다. 이는 Data Warehouse의 일반적인 패턴입니다.

2.  집계에 두 번째 dimension을 추가하기 위해 쿼리를 다음과 같이 수정하세요.

    ```sql
   SELECT  d.[Year] AS CalendarYear,
           d.[Month] AS MonthOfYear,
           d.MonthName AS MonthName,
           c.CountryRegion AS SalesRegion,
          SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   JOIN DimCustomer AS c ON so.CustomerKey = c.CustomerKey
   GROUP BY d.[Year], d.[Month], d.MonthName, c.CountryRegion
   ORDER BY CalendarYear, MonthOfYear, SalesRegion;
    ```

3.  수정된 쿼리를 실행하고 결과를 검토하세요. 결과에는 이제 연도, 월 및 판매 지역별로 집계된 판매 수익이 포함됩니다.

## 뷰(view) 생성

Microsoft Fabric의 Data Warehouse는 관계형 데이터베이스에서 사용하던 것과 동일한 많은 기능을 가지고 있습니다. 예를 들어, SQL 로직을 캡슐화하기 위해 *뷰* 및 *저장 프로시저*와 같은 데이터베이스 개체를 만들 수 있습니다.

1.  이전에 생성한 쿼리를 다음과 같이 수정하여 뷰를 생성하세요 (뷰를 생성하려면 ORDER BY 절을 제거해야 합니다).

    ```sql
   CREATE VIEW vSalesByRegion
   AS
   SELECT  d.[Year] AS CalendarYear,
           d.[Month] AS MonthOfYear,
           d.MonthName AS MonthName,
           c.CountryRegion AS SalesRegion,
          SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   JOIN DimCustomer AS c ON so.CustomerKey = c.CustomerKey
   GROUP BY d.[Year], d.[Month], d.MonthName, c.CountryRegion;
    ```

2.  쿼리를 실행하여 뷰를 생성하세요. 그런 다음 Data Warehouse schema를 새로 고치고 새 뷰가 **Explorer** pane에 나열되어 있는지 확인하세요.
3.  새 SQL 쿼리를 생성하고 다음 SELECT 문을 실행하세요.

    ```SQL
   SELECT CalendarYear, MonthName, SalesRegion, SalesRevenue
   FROM vSalesByRegion
   ORDER BY CalendarYear, MonthOfYear, SalesRegion;
    ```

## Visual Query 생성

SQL 코드를 작성하는 대신 그래픽 쿼리 디자이너를 사용하여 Data Warehouse의 테이블을 쿼리할 수 있습니다. 이 경험은 코드 없이 데이터 변환 단계(data transformation steps)를 만들 수 있는 Power Query online과 유사합니다. 더 복잡한 작업의 경우 Power Query의 M(Mashup) 언어를 사용할 수 있습니다.

1.  **Home** 메뉴에서 **New SQL query** 아래의 옵션을 확장하고 **New visual query**를 선택하세요.

2.  **FactSalesOrder**를 **canvas**로 드래그하세요. 아래의 **Preview** pane에 테이블 미리보기가 표시됩니다.

3.  **DimProduct**를 **canvas**로 드래그하세요. 이제 쿼리에 두 개의 테이블이 있습니다.

4.  canvas의 **FactSalesOrder** 테이블에서 **(+)** 버튼을 사용하여 **Merge queries**하세요.
    ![Screenshot of the canvas with the FactSalesOrder table selected.](./Images/visual-query-merge.png)

5.  **Merge queries** 창에서 병합(merge)할 오른쪽 테이블로 **DimProduct**를 선택하세요. 두 쿼리 모두에서 **ProductKey**를 선택하고, 기본 **Left outer** join type을 그대로 둔 다음 **OK**를 클릭하세요.

6.  **Preview**에서 새 **DimProduct** 열이 FactSalesOrder 테이블에 추가되었음을 확인하세요. 열 이름 오른쪽에 있는 화살표를 클릭하여 열을 확장하세요. **ProductName**을 선택하고 **OK**를 클릭하세요.

    ![Screenshot of the preview pane with the DimProduct column expanded, with ProductName selected.](./Images/visual-query-preview.png)

7.  관리자 요청에 따라 단일 제품에 대한 데이터를 보려는 경우, 이제 **ProductName** 열을 사용하여 쿼리의 데이터를 필터링할 수 있습니다. **ProductName** 열을 필터링하여 **Cable Lock** 데이터만 보세요.

8.  여기에서 **Visualize results** 또는 **Download Excel file**을 선택하여 이 단일 쿼리의 결과를 분석할 수 있습니다. 이제 관리자가 요청했던 내용을 정확히 확인할 수 있으므로 더 이상 결과를 분석할 필요가 없습니다.

## (선택 사항) 데이터 모델 정의

**참고**: 이 작업은 전적으로 선택 사항이지만, semantic model을 생성하고 편집하려면 Power BI 라이선스 또는 Fabric F64 SKU가 필요합니다.

관계형 Data Warehouse는 일반적으로 *fact* 테이블 및 *dimension* 테이블로 구성됩니다. fact 테이블에는 비즈니스 성과를 분석하기 위해 집계할 수 있는 숫자 측정값(예: 판매 수익)이 포함되어 있으며, dimension 테이블에는 데이터를 집계할 수 있는 엔터티의 특성(예: 제품, 고객 또는 시간)이 포함되어 있습니다. Microsoft Fabric Data Warehouse에서는 이러한 키를 사용하여 테이블 간의 관계를 캡슐화하는 데이터 모델을 정의할 수 있습니다.

1.  도구 모음에서 **New semantic model**을 선택하세요.
2.  **New semantic model** 창에서 semantic model의 이름을 지정하고 네 개의 테이블을 모두 선택하세요. **Confirm**을 선택하세요.
3.  새 semantic model과 함께 새 브라우저 탭이 자동으로 열립니다. 모델 pane에서 Data Warehouse의 테이블을 재배열하여 **FactSalesOrder** 테이블이 다음과 같이 가운데에 오도록 하세요.

    ![Screenshot of the data warehouse model page.](./Images/model-dw.png)

4.  **FactSalesOrder** 테이블에서 **ProductKey** 필드를 드래그하여 **DimProduct** 테이블의 **ProductKey** 필드에 놓으세요. 그런 다음 다음 관계 세부 정보를 확인하세요.
    -   **From table**: FactSalesOrder
    -   **Column**: ProductKey
    -   **To table**: DimProduct
    -   **Column**: ProductKey
    -   **Cardinality**: Many to one (*:1)
    -   **Cross filter direction**: Single
    -   **Make this relationship active**: Selected
    -   **Assume referential integrity**: Unselected

5.  이 과정을 반복하여 다음 테이블 사이에 다대일(many-to-one) 관계를 생성하세요.
    -   **FactSalesOrder.CustomerKey** &#8594; **DimCustomer.CustomerKey**
    -   **FactSalesOrder.SalesOrderDateKey** &#8594; **DimDate.DateKey**

    모든 관계가 정의되면 모델은 다음과 같이 보여야 합니다.

    ![Screenshot of the model with relationships.](./Images/dw-relationships.png)

## 리소스 정리

이 실습에서 여러 테이블을 포함하는 Data Warehouse를 만들었습니다. SQL을 사용하여 테이블에 데이터를 삽입하고 T-SQL 및 visual query 도구를 사용하여 테이블을 쿼리했습니다. 마지막으로, 다운스트림 분석 및 보고용으로 Data Warehouse의 기본 dataset에 대한 데이터 모델을 개선했습니다.

Data Warehouse 탐색을 마쳤다면 이 실습을 위해 생성한 workspace를 삭제할 수 있습니다.

1.  왼쪽 바에서 workspace 아이콘을 선택하여 포함된 모든 항목을 볼 수 있습니다.
2.  도구 모음의 **...** 메뉴에서 **Workspace settings**를 선택하세요.
3.  **General** 섹션에서 **Remove this workspace**를 선택하세요.