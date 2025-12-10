`---
lab:
    title: 'Load data into a warehouse using T-SQL'
    module: 'Load data into a warehouse in Microsoft Fabric'
---

`# T-SQL을 사용하여 Data Warehouse에 데이터 로드

Microsoft Fabric에서 Data Warehouse는 대규모 분석을 위한 관계형 데이터베이스를 제공합니다. Lakehouse에 정의된 테이블의 기본 읽기 전용 SQL endpoint와 달리, Data Warehouse는 테이블에 데이터를 삽입, 업데이트, 삭제하는 기능을 포함하여 전체 SQL semantic(SQL 의미론)을 제공합니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## Workspace 생성

Fabric에서 데이터를 사용하기 전에, Fabric 평가판이 활성화된 Workspace를 생성하세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명(credentials)으로 로그인합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 유사합니다)를 선택합니다.
3.  원하는 이름으로 새 Workspace를 생성하고, Fabric Capacity(*Trial*, *Premium*, 또는 *Fabric*)를 포함하는 라이선싱 모드(licensing mode)를 선택하세요.
4.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Lakehouse 생성 및 파일 업로드

이 시나리오에서는 사용 가능한 데이터가 없으므로, Data Warehouse 로딩에 사용할 데이터를 ingest(수집)해야 합니다. Data Warehouse를 로드하는 데 사용할 데이터 파일용 Lakehouse를 생성합니다.

1.  **+ New item**을 선택하고 원하는 이름으로 새 **Lakehouse**를 생성하세요.

    1분 정도 후에 새 빈 Lakehouse가 생성됩니다. 분석을 위해 Data Lakehouse로 일부 데이터를 ingest(수집)해야 합니다. 이를 수행하는 여러 가지 방법이 있지만, 이 실습에서는 CSV 파일을 로컬 컴퓨터(또는 해당되는 경우 랩 VM)로 다운로드한 다음 Lakehouse에 업로드합니다.

2.  `https://github.com/MicrosoftLearning/dp-data/raw/main/sales.csv`에서 이 실습용 파일을 다운로드합니다.

3.  Lakehouse가 포함된 웹 브라우저 탭으로 돌아가서, **Explorer** 창의 **Files** 폴더에 있는 **...** 메뉴에서 **Upload**와 **Upload files**를 선택한 다음, 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에서 **sales.csv** 파일을 Lakehouse에 업로드합니다.

4.  파일이 업로드된 후 **Files**를 선택하세요. 여기에 표시된 대로 CSV 파일이 업로드되었는지 확인합니다.

    ![Screenshot of uploaded file in a lakehouse.](./Images/sales-file-upload.png)

## Lakehouse에 테이블 생성

1.  **Explorer** 창의 **sales.csv** 파일에 있는 **...** 메뉴에서 **Load to tables**를 선택한 다음 **New table**을 선택합니다.

2.  **Load file to new table** 대화 상자에 다음 정보를 제공합니다.
    -   **New table name:** staging_sales
    -   **Use header for columns names:** Selected
    -   **Separator:** ,

3.  **Load**를 선택합니다.

## Warehouse 생성

이제 Workspace, Lakehouse, 그리고 필요한 데이터가 있는 sales 테이블이 있으니, Data Warehouse를 생성할 차례입니다.

1.  왼쪽 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Warehouse* 섹션에서 **Warehouse**를 선택합니다. 원하는 고유한 이름을 지정하세요.

    >**참고**: **Create** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표 (**...**) 옵션을 선택해야 합니다.

    1분 정도 후에 새 Warehouse가 생성됩니다.

    ![Screenshot of a new warehouse.](./Images/new-empty-data-warehouse.png)

## Fact table, dimension 및 View 생성

Sales 데이터에 대한 Fact table(팩트 테이블)과 dimension(차원)을 생성해 봅시다. Lakehouse를 가리키는 view(뷰)도 생성할 것인데, 이는 로드에 사용할 stored procedure(저장 프로시저)의 코드를 단순화합니다.

1.  Workspace에서 생성한 Warehouse를 선택합니다.

2.  Warehouse 도구 모음에서 **New SQL query**를 선택한 다음, 다음 쿼리를 복사하여 실행합니다.

    ```sql
    CREATE SCHEMA [Sales]
    GO
        
    IF OBJECT_ID('Sales.Fact_Sales', 'U') IS NULL
    	CREATE TABLE Sales.Fact_Sales (
    		CustomerID VARCHAR(255) NOT NULL,
    		ItemID VARCHAR(255) NOT NULL,
    		SalesOrderNumber VARCHAR(30),
    		SalesOrderLineNumber INT,
    		OrderDate DATE,
    		Quantity INT,
    		TaxAmount FLOAT,
    		UnitPrice FLOAT
    	);
    
    IF OBJECT_ID('Sales.Dim_Customer', 'U') IS NULL
        CREATE TABLE Sales.Dim_Customer (
            CustomerID VARCHAR(255) NOT NULL,
            CustomerName VARCHAR(255) NOT NULL,
            EmailAddress VARCHAR(255) NOT NULL
        );
        
    ALTER TABLE Sales.Dim_Customer add CONSTRAINT PK_Dim_Customer PRIMARY KEY NONCLUSTERED (CustomerID) NOT ENFORCED
    GO
    
    IF OBJECT_ID('Sales.Dim_Item', 'U') IS NULL
        CREATE TABLE Sales.Dim_Item (
            ItemID VARCHAR(255) NOT NULL,
            ItemName VARCHAR(255) NOT NULL
        );
        
    ALTER TABLE Sales.Dim_Item add CONSTRAINT PK_Dim_Item PRIMARY KEY NONCLUSTERED (ItemID) NOT ENFORCED
    GO
    ```

    > **중요**: Data Warehouse에서 foreign key constraint(외래 키 제약 조건)는 테이블 수준에서 항상 필요한 것은 아닙니다. Foreign key constraint(외래 키 제약 조건)가 데이터 무결성(data integrity)을 보장하는 데 도움이 될 수 있지만, ETL (Extract, Transform, Load) 프로세스에 오버헤드(overhead)를 추가하고 데이터 로딩 속도를 늦출 수도 있습니다. Data Warehouse에서 foreign key constraint(외래 키 제약 조건) 사용 결정은 데이터 무결성(data integrity)과 성능(performance) 간의 trade-off(장단점)를 신중하게 고려하여 이루어져야 합니다.

3.  **Explorer**에서 **Schemas >> Sales >> Tables**로 이동합니다. 방금 생성한 *Fact_Sales*, *Dim_Customer*, *Dim_Item* 테이블을 확인합니다.

    > **참고**: 새 schema(스키마)가 보이지 않으면, **Explorer** 창의 **Tables**에 있는 **...** 메뉴를 열고 **Refresh**를 선택하세요.

4.  새 **New SQL query** 편집기를 열고, 다음 쿼리를 복사하여 실행합니다. *<your lakehouse name>*을 생성한 Lakehouse 이름으로 업데이트하세요.

    ```sql
    CREATE VIEW Sales.Staging_Sales
    AS
	SELECT * FROM [<your lakehouse name>].[dbo].[staging_sales];
    ```

5.  **Explorer**에서 **Schemas >> Sales >> Views**로 이동합니다. 생성한 *Staging_Sales* view(뷰)를 확인합니다.

## Warehouse로 데이터 로드

이제 fact 테이블과 dimension 테이블이 생성되었으므로, Lakehouse에서 Data Warehouse로 데이터를 로드하는 stored procedure(저장 프로시저)를 생성해 보겠습니다. Lakehouse 생성 시 자동으로 생성되는 SQL endpoint 덕분에 T-SQL과 cross-database query(교차 데이터베이스 쿼리)를 사용하여 Warehouse에서 Lakehouse의 데이터에 직접 액세스할 수 있습니다.

이 사례 연구의 단순화를 위해 고객 이름과 항목 이름을 primary key(기본 키)로 사용합니다.

1.  새 **New SQL query** 편집기를 생성하고, 다음 쿼리를 복사하여 실행합니다.

    ```sql
    CREATE OR ALTER PROCEDURE Sales.LoadDataFromStaging (@OrderYear INT)
    AS
    BEGIN
    	-- Load data into the Customer dimension table
        INSERT INTO Sales.Dim_Customer (CustomerID, CustomerName, EmailAddress)
        SELECT DISTINCT CustomerName, CustomerName, EmailAddress
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1
            FROM Sales.Dim_Customer
            WHERE Sales.Dim_Customer.CustomerName = Sales.Staging_Sales.CustomerName
            AND Sales.Dim_Customer.EmailAddress = Sales.Staging_Sales.EmailAddress
        );
        
        -- Load data into the Item dimension table
        INSERT INTO Sales.Dim_Item (ItemID, ItemName)
        SELECT DISTINCT Item, Item
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1
            FROM Sales.Dim_Item
            WHERE Sales.Dim_Item.ItemName = Sales.Staging_Sales.Item
        );
        
        -- Load data into the Sales fact table
        INSERT INTO Sales.Fact_Sales (CustomerID, ItemID, SalesOrderNumber, SalesOrderLineNumber, OrderDate, Quantity, TaxAmount, UnitPrice)
        SELECT CustomerName, Item, SalesOrderNumber, CAST(SalesOrderLineNumber AS INT), CAST(OrderDate AS DATE), CAST(Quantity AS INT), CAST(TaxAmount AS FLOAT), CAST(UnitPrice AS FLOAT)
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear;
    END
    ```
2.  새 **New SQL query** 편집기를 생성하고, 다음 쿼리를 복사하여 실행합니다.

    ```sql
    EXEC Sales.LoadDataFromStaging 2021
    ```

    > **참고**: 이 경우 2021년 데이터만 로드하고 있습니다. 하지만 이전 연도의 데이터를 로드하도록 수정할 수 있습니다.

## 분석 쿼리 실행

Data Warehouse의 데이터를 검증하기 위해 분석 쿼리를 실행해 봅시다.

1.  상단 메뉴에서 **New SQL query**를 선택한 다음, 다음 쿼리를 복사하여 실행합니다.

    ```sql
    SELECT c.CustomerName, SUM(s.UnitPrice * s.Quantity) AS TotalSales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Customer c
    ON s.CustomerID = c.CustomerID
    WHERE YEAR(s.OrderDate) = 2021
    GROUP BY c.CustomerName
    ORDER BY TotalSales DESC;
    ```

    > **참고**: 이 쿼리는 2021년 총 판매액(total sales) 기준 고객을 보여줍니다. 지정된 연도에 가장 높은 총 판매액을 기록한 고객은 **Jordan Turner**이며, 총 판매액은 **14686.69**입니다.

2.  상단 메뉴에서 **New SQL query**를 선택하거나 동일한 편집기를 재사용하고, 다음 쿼리를 복사하여 실행합니다.

    ```sql
    SELECT i.ItemName, SUM(s.UnitPrice * s.Quantity) AS TotalSales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Item i
    ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
    GROUP BY i.ItemName
    ORDER BY TotalSales DESC;

    ```

    > **참고**: 이 쿼리는 2021년 총 판매액(total sales) 기준 베스트셀러 항목을 보여줍니다. 이 결과는 *Mountain-200 bike* 모델(블랙 및 실버 색상 모두)이 2021년 고객들 사이에서 가장 인기 있는 항목이었음을 시사합니다.

3.  상단 메뉴에서 **New SQL query**를 선택하거나 동일한 편집기를 재사용하고, 다음 쿼리를 복사하여 실행합니다.

    ```sql
    WITH CategorizedSales AS (
    SELECT
        CASE
            WHEN i.ItemName LIKE '%Helmet%' THEN 'Helmet'
            WHEN i.ItemName LIKE '%Bike%' THEN 'Bike'
            WHEN i.ItemName LIKE '%Gloves%' THEN 'Gloves'
            ELSE 'Other'
        END AS Category,
        c.CustomerName,
        s.UnitPrice * s.Quantity AS Sales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Customer c
    ON s.CustomerID = c.CustomerID
    JOIN Sales.Dim_Item i
    ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
    ),
    RankedSales AS (
        SELECT
            Category,
            CustomerName,
            SUM(Sales) AS TotalSales,
            ROW_NUMBER() OVER (PARTITION BY Category ORDER BY SUM(Sales) DESC) AS SalesRank
        FROM CategorizedSales
        WHERE Category IN ('Helmet', 'Bike', 'Gloves')
        GROUP BY Category, CustomerName
    )
    SELECT Category, CustomerName, TotalSales
    FROM RankedSales
    WHERE SalesRank = 1
    ORDER BY TotalSales DESC;
    ```

    > **참고**: 이 쿼리의 결과는 총 판매액(total sales)을 기준으로 각 Category(카테고리)인 Bike, Helmet, Gloves의 상위 고객을 보여줍니다. 예를 들어, **Joan Coleman**은 **Gloves** Category(카테고리)의 상위 고객입니다.
    >
    > dimension 테이블에 별도의 category column(카테고리 열)이 없으므로, category 정보는 string manipulation(문자열 조작)을 사용하여 `ItemName` 열에서 추출되었습니다. 이 접근 방식은 항목 이름이 일관된 명명 규칙(naming convention)을 따른다고 가정합니다. 항목 이름이 일관된 명명 규칙을 따르지 않는 경우, 결과가 각 항목의 실제 category(카테고리)를 정확하게 반영하지 못할 수 있습니다.

이 실습에서는 여러 테이블이 있는 Lakehouse와 Data Warehouse를 생성했습니다. 데이터를 ingest(수집)하고 cross-database query(교차 데이터베이스 쿼리)를 사용하여 Lakehouse에서 Warehouse로 데이터를 로드했습니다. 또한 쿼리 도구를 사용하여 분석 쿼리를 수행했습니다.

## 리소스 정리

Data Warehouse 탐색을 마쳤으면, 이 실습을 위해 생성한 Workspace를 삭제할 수 있습니다.

1.  왼쪽 바에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2.  **Workspace settings**를 선택하고, **General** 섹션에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.
3.  **Delete**를 선택하여 Workspace를 삭제합니다.