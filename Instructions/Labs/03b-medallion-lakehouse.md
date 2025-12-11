---
lab:
    title: 'Create a medallion architecture in a Microsoft Fabric lakehouse'
    module: 'Organize a Fabric lakehouse using medallion architecture design'
---

# Microsoft Fabric 레이크하우스에 메달리온 아키텍처 구축하기

이 실습에서는 Notebook을 사용하여 Fabric Lakehouse에 메달리온 아키텍처를 구축합니다. 작업 영역을 생성하고, Lakehouse를 생성하며, 브론즈 계층에 데이터를 업로드하고, 데이터를 변환하여 실버 Delta 테이블에 로드하고, 데이터를 추가로 변환하여 골드 Delta 테이블에 로드한 다음, 의미론적 모델을 탐색하고 관계를 생성합니다.

이 실습을 완료하는 데 약 **45**분이 소요됩니다.

> [!Note]
> 이 실습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)에 액세스해야 합니다.

## 작업 영역 생성

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 작업 영역을 생성해야 합니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric-developer` 주소의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric-developer)로 이동하여 Fabric 자격 증명으로 로그인하세요.
2. 왼쪽 메뉴 모음에서 **작업 영역 (Workspaces)**을 선택합니다 (아이콘은 &#128455;와 비슷합니다).
3. 원하는 이름으로 새 작업 영역을 생성하고, **고급 (Advanced)** 섹션에서 Fabric Capacity(*Trial*, *Premium*, *Fabric*)를 포함하는 라이선싱 모드를 선택합니다.
4. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Lakehouse 생성 및 브론즈 계층에 데이터 업로드

이제 작업 영역이 생겼으므로, 분석할 데이터를 위한 데이터 Lakehouse를 생성할 차례입니다.

1. 방금 생성한 작업 영역에서 **+ 새로 만들기 (+ New item)** 버튼을 선택하여 **Sales**라는 새 **Lakehouse**를 생성합니다.

    잠시 후 새 빈 Lakehouse가 생성됩니다. 다음으로, 분석을 위해 일부 데이터를 데이터 Lakehouse로 수집합니다. 여러 가지 방법이 있지만, 이 실습에서는 간단히 텍스트 파일을 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에 다운로드한 다음 Lakehouse에 업로드합니다.

2. `https://github.com/MicrosoftLearning/dp-data/blob/main/orders.zip`에서 이 실습용 데이터 파일을 다운로드하세요. 파일을 추출하고 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에 원래 이름으로 저장합니다. 3년간의 판매 데이터가 포함된 3개의 파일(2019.csv, 2020.csv, 2021.csv)이 있어야 합니다.

3. Lakehouse가 있는 웹 브라우저 탭으로 돌아가서, **탐색기 (Explorer)** 창의 **Files** 폴더에 있는 **...** 메뉴에서 **새 하위 폴더 (New subfolder)**를 선택하고 **bronze**라는 폴더를 생성합니다.

4. **bronze** 폴더의 **...** 메뉴에서 **업로드 (Upload)** 및 **파일 업로드 (Upload files)**를 선택한 다음, 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에서 3개의 파일(2019.csv, 2020.csv, 2021.csv)을 Lakehouse로 업로드합니다. Shift 키를 사용하여 세 개의 파일을 한 번에 업로드하세요.

5. 파일이 업로드된 후 **bronze** 폴더를 선택하고, 여기에 표시된 대로 파일이 업로드되었는지 확인합니다.

    ![Lakehouse에 업로드된 products.csv 파일 스크린샷](./Images/bronze-files.png)

## 데이터 변환 및 실버 Delta 테이블에 로드

이제 Lakehouse의 브론즈 계층에 데이터가 있으므로, Notebook을 사용하여 데이터를 변환하고 실버 계층의 Delta 테이블에 로드할 수 있습니다.

1. 데이터 레이크의 **bronze** 폴더 내용을 보는 동안 **홈 (Home)** 페이지에서 **Notebook 열기 (Open notebook)** 메뉴에서 **새 Notebook (New notebook)**을 선택합니다.

    몇 초 후에 단일 *셀*을 포함하는 새 Notebook이 열립니다. Notebook은 *코드* 또는 *Markdown* (형식화된 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

2. Notebook이 열리면 Notebook 왼쪽 상단에 있는 **Notebook xxxx** 텍스트를 선택하고 새 이름을 입력하여 `Transform data for Silver`로 이름을 변경합니다.

    ![Transform data for silver로 이름이 지정된 새 Notebook 스크린샷](./Images/sales-notebook-rename.png)

3. Notebook에서 간단한 주석 처리된 코드가 포함된 기존 셀을 선택합니다. 이 두 줄을 강조 표시하고 삭제하세요. 이 코드는 필요하지 않습니다.

   > **참고**: Notebook은 Python, Scala, SQL을 포함한 다양한 언어로 코드를 실행할 수 있도록 합니다. 이 실습에서는 PySpark와 SQL을 사용합니다. 코드 문서화를 위해 형식화된 텍스트와 이미지를 제공하는 Markdown 셀을 추가할 수도 있습니다.

4. 다음 코드를 셀에 **붙여넣기**하세요:

    ```python
   from pyspark.sql.types import *
    
   # Create the schema for the table
   orderSchema = StructType([
       StructField("SalesOrderNumber", StringType()),
       StructField("SalesOrderLineNumber", IntegerType()),
       StructField("OrderDate", DateType()),
       StructField("CustomerName", StringType()),
       StructField("Email", StringType()),
       StructField("Item", StringType()),
       StructField("Quantity", IntegerType()),
       StructField("UnitPrice", FloatType()),
       StructField("Tax", FloatType())
       ])
    
   # Import all files from bronze folder of lakehouse
   df = spark.read.format("csv").option("header", "false").schema(orderSchema).load("Files/bronze/*.csv")
    
   # Display the first 10 rows of the dataframe to preview your data
   display(df.head(10))
    ```

5. 셀 왼쪽에 있는 ****&#9655;** (셀 실행)** 버튼을 사용하여 코드를 실행합니다.

    > **참고**: 이 Notebook에서 Spark 코드를 처음 실행하는 것이므로 Spark 세션이 시작되어야 합니다. 이는 첫 실행에 약 1분 정도 걸릴 수 있음을 의미합니다. 이후 실행은 더 빨라집니다.

6. 셀 명령이 완료되면 셀 아래의 **출력을 검토**하세요. 출력은 다음과 유사해야 합니다:

    | Index | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | Email | Item | Quantity | UnitPrice | Tax |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO49172 | 1 | 2021-01-01 | Brian Howard | brian23@adventure-works.com | Road-250 Red, 52 | 1 | 2443.35 | 195.468 |
    | 2 |  SO49173 | 1 | 2021-01-01 | Linda Alvarez | linda19@adventure-works.com | Mountain-200 Silver, 38 | 1 | 2071.4197 | 165.7136 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    실행한 코드는 **bronze** 폴더의 CSV 파일에서 데이터를 Spark dataframe으로 로드한 다음, dataframe의 첫 몇 행을 표시했습니다.

    > **참고**: 출력 창의 왼쪽 상단에 있는 **...** 메뉴를 선택하여 셀 출력 내용을 지우거나 숨기거나 자동으로 크기를 조정할 수 있습니다.

7. 이제 PySpark dataframe을 사용하여 열을 추가하고 기존 열의 일부 값을 업데이트하여 **데이터 유효성 검사 및 정리용 열을 추가**합니다. **+ 코드 (Code)** 버튼을 사용하여 **새 코드 블록을 추가**하고 다음 코드를 셀에 추가하세요:

    ```python
   from pyspark.sql.functions import when, lit, col, current_timestamp, input_file_name
    
   # Add columns IsFlagged, CreatedTS and ModifiedTS
   df = df.withColumn("FileName", input_file_name()) \
       .withColumn("IsFlagged", when(col("OrderDate") < '2019-08-01',True).otherwise(False)) \
       .withColumn("CreatedTS", current_timestamp()).withColumn("ModifiedTS", current_timestamp())
    
   # Update CustomerName to "Unknown" if CustomerName null or empty
   df = df.withColumn("CustomerName", when((col("CustomerName").isNull() | (col("CustomerName")=="")),lit("Unknown")).otherwise(col("CustomerName")))
    ```

    코드의 첫 줄은 PySpark에서 필요한 함수를 가져옵니다. 그런 다음 원본 파일 이름, 관심 회계 연도 이전에 주문이 플래그 지정되었는지 여부, 행이 생성 및 수정된 시기를 추적할 수 있도록 새 열을 dataframe에 추가합니다.

    마지막으로, CustomerName 열이 null이거나 비어 있으면 "Unknown"으로 업데이트합니다.

8. ****&#9655;** (셀 실행)** 버튼을 사용하여 셀을 실행하여 코드를 실행합니다.

9. 다음으로, Delta Lake 형식을 사용하여 sales 데이터베이스의 **sales_silver** 테이블에 대한 스키마를 정의합니다. 새 코드 블록을 생성하고 다음 코드를 셀에 추가하세요:

    ```python
   # Define the schema for the sales_silver table
    
   from pyspark.sql.types import *
   from delta.tables import *
    
   DeltaTable.createIfNotExists(spark) \
       .tableName("sales.sales_silver") \
       .addColumn("SalesOrderNumber", StringType()) \
       .addColumn("SalesOrderLineNumber", IntegerType()) \
       .addColumn("OrderDate", DateType()) \
       .addColumn("CustomerName", StringType()) \
       .addColumn("Email", StringType()) \
       .addColumn("Item", StringType()) \
       .addColumn("Quantity", IntegerType()) \
       .addColumn("UnitPrice", FloatType()) \
       .addColumn("Tax", FloatType()) \
       .addColumn("FileName", StringType()) \
       .addColumn("IsFlagged", BooleanType()) \
       .addColumn("CreatedTS", DateType()) \
       .addColumn("ModifiedTS", DateType()) \
       .execute()
    ```

10. ****&#9655;** (셀 실행)** 버튼을 사용하여 셀을 실행하여 코드를 실행합니다.

11. 탐색기 (Explorer) 창의 테이블 섹션에서 **...**를 선택하고 **새로 고침 (Refresh)**을 선택합니다. 이제 새 **sales_silver** 테이블이 나열된 것을 볼 수 있습니다. **&#9650;** (삼각형 아이콘)은 Delta 테이블임을 나타냅니다.

    > **참고**: 새 테이블이 보이지 않으면 몇 초 기다린 다음 다시 **새로 고침 (Refresh)**을 선택하거나 전체 브라우저 탭을 새로 고침하세요.

12. 이제 Delta 테이블에서 **upsert 작업 (upsert operation)**을 수행하여 특정 조건에 따라 기존 레코드를 업데이트하고 일치하는 항목이 없는 경우 새 레코드를 삽입합니다. 새 코드 블록을 추가하고 다음 코드를 붙여넣으세요:

    ```python
   # Update existing records and insert new ones based on a condition defined by the columns SalesOrderNumber, OrderDate, CustomerName, and Item.

   from delta.tables import *
    
   deltaTable = DeltaTable.forPath(spark, 'Tables/sales_silver')
    
   dfUpdates = df
    
   deltaTable.alias('silver') \
     .merge(
       dfUpdates.alias('updates'),
       'silver.SalesOrderNumber = updates.SalesOrderNumber and silver.OrderDate = updates.OrderDate and silver.CustomerName = updates.CustomerName and silver.Item = updates.Item'
     ) \
      .whenMatchedUpdate(set =
       {
          
       }
     ) \
    .whenNotMatchedInsert(values =
       {
         "SalesOrderNumber": "updates.SalesOrderNumber",
         "SalesOrderLineNumber": "updates.SalesOrderLineNumber",
         "OrderDate": "updates.OrderDate",
         "CustomerName": "updates.CustomerName",
         "Email": "updates.Email",
         "Item": "updates.Item",
         "Quantity": "updates.Quantity",
         "UnitPrice": "updates.UnitPrice",
         "Tax": "updates.Tax",
         "FileName": "updates.FileName",
         "IsFlagged": "updates.IsFlagged",
         "CreatedTS": "updates.CreatedTS",
         "ModifiedTS": "updates.ModifiedTS"
       }
     ) \
     .execute()
    ```

13. ****&#9655;** (셀 실행)** 버튼을 사용하여 셀을 실행하여 코드를 실행합니다.

    이 작업은 특정 열의 값을 기반으로 테이블의 기존 레코드를 업데이트하고 일치하는 항목이 없는 경우 새 레코드를 삽입할 수 있도록 하므로 중요합니다. 이는 기존 및 새 레코드에 대한 업데이트를 포함할 수 있는 원본 시스템에서 데이터를 로드할 때 흔히 요구되는 사항입니다.

    이제 추가 변환 및 모델링을 위해 실버 Delta 테이블에 데이터가 준비되었습니다.

14. 마지막 셀을 실행한 후 리본 위에 있는 **실행 (Run)** 탭을 선택한 다음 **세션 중지 (Stop session)**를 선택하여 Notebook에서 사용 중인 컴퓨팅 리소스를 중지합니다.

## SQL endpoint를 사용하여 실버 계층의 데이터 탐색

이제 실버 계층에 데이터가 있으므로, SQL 분석 엔드포인트 (SQL analytics endpoint)를 사용하여 데이터를 탐색하고 몇 가지 기본 분석을 수행할 수 있습니다. 이는 SQL에 익숙하고 데이터에 대한 기본적인 탐색을 수행하려는 경우 유용합니다. 이 실습에서는 Fabric의 SQL endpoint 보기를 사용하지만, SQL Server Management Studio (SSMS) 및 Azure Data Explorer와 같은 다른 도구를 사용할 수도 있습니다.

1. 작업 영역으로 돌아가면 이제 여러 항목이 나열되어 있음을 알 수 있습니다. **Sales SQL 분석 엔드포인트 (Sales SQL analytics endpoint)**를 선택하여 Lakehouse를 SQL 분석 엔드포인트 보기에서 엽니다.

    ![Lakehouse의 SQL endpoint 스크린샷](./Images/sql-endpoint-item.png)

2. 리본에서 **새 SQL 쿼리 (New SQL query)**를 선택하면 SQL 쿼리 편집기가 열립니다. 탐색기 (Explorer) 창의 기존 쿼리 이름 옆에 있는 **...** 메뉴 항목을 사용하여 쿼리 이름을 변경할 수 있습니다.

   다음으로, 데이터를 탐색하기 위해 두 개의 SQL 쿼리를 실행합니다.

3. 다음 쿼리를 쿼리 편집기에 붙여넣고 **실행 (Run)**을 선택하세요:

    ```sql
   SELECT YEAR(OrderDate) AS Year
       , CAST (SUM(Quantity * (UnitPrice + Tax)) AS DECIMAL(12, 2)) AS TotalSales
   FROM sales_silver
   GROUP BY YEAR(OrderDate) 
   ORDER BY YEAR(OrderDate)
    ```

    이 쿼리는 sales_silver 테이블에서 각 연도의 총 매출을 계산합니다. 결과는 다음과 같아야 합니다:

    ![Lakehouse에서 SQL 쿼리 결과 스크린샷](./Images/total-sales-sql.png)

4. 다음으로, 어떤 고객이 가장 많이 구매하는지(수량 측면에서) 검토합니다. 다음 쿼리를 쿼리 편집기에 붙여넣고 **실행 (Run)**을 선택하세요:

    ```sql
   SELECT TOP 10 CustomerName, SUM(Quantity) AS TotalQuantity
   FROM sales_silver
   GROUP BY CustomerName
   ORDER BY TotalQuantity DESC
    ```

    이 쿼리는 sales_silver 테이블에서 각 고객이 구매한 항목의 총 수량을 계산한 다음, 수량 측면에서 상위 10명의 고객을 반환합니다.

    실버 계층에서의 데이터 탐색은 기본적인 분석에 유용하지만, 더 고급 분석 및 보고를 가능하게 하려면 데이터를 추가로 변환하고 스타 스키마로 모델링해야 합니다. 다음 섹션에서 이를 수행합니다.

## 골드 계층을 위한 데이터 변환

브론즈 계층에서 데이터를 가져와 변환하고 실버 Delta 테이블에 로드하는 데 성공했습니다. 이제 새 Notebook을 사용하여 데이터를 추가로 변환하고 스타 스키마로 모델링한 다음, 골드 Delta 테이블에 로드합니다.

이 모든 작업을 단일 Notebook에서 수행할 수도 있었지만, 이 실습에서는 데이터를 브론즈에서 실버로, 그리고 실버에서 골드로 변환하는 프로세스를 보여주기 위해 별도의 Notebook을 사용합니다. 이는 디버깅, 문제 해결 및 재사용에 도움이 될 수 있습니다.

1. 작업 영역 홈 페이지로 돌아가서 `Transform data for Gold`라는 새 Notebook을 생성합니다.

2. 탐색기 (Explorer) 창에서 **데이터 항목 추가 (Add data items)**를 선택한 다음 이전에 생성한 **Sales** Lakehouse를 선택하여 **Sales** Lakehouse를 추가합니다. 탐색기 (Explorer) 창의 **테이블 (Tables)** 섹션에 **sales_silver** 테이블이 나열된 것을 볼 수 있습니다.

3. 기존 코드 블록에서 주석 처리된 텍스트를 제거하고, dataframe에 데이터를 로드하고 스타 스키마 구축을 시작하기 위해 **다음 코드를 추가**한 다음 실행합니다:

    ```python
   # Load data to the dataframe as a starting point to create the gold layer
   df = spark.read.table("Sales.sales_silver")
    ```

    > **참고**: 첫 번째 셀을 실행할 때 `[TooManyRequestsForCapacity]` 오류가 발생하면, 첫 번째 Notebook에서 이전에 실행 중이던 세션을 중지했는지 확인하십시오.
 
4. **새 코드 블록을 추가**하고 다음 코드를 붙여넣어 날짜 차원 테이블을 생성하고 실행합니다:

    ```python
   from pyspark.sql.types import *
   from delta.tables import*
    
   # Define the schema for the dimdate_gold table
   DeltaTable.createIfNotExists(spark) \
       .tableName("sales.dimdate_gold") \
       .addColumn("OrderDate", DateType()) \
       .addColumn("Day", IntegerType()) \
       .addColumn("Month", IntegerType()) \
       .addColumn("Year", IntegerType()) \
       .addColumn("mmmyyyy", StringType()) \
       .addColumn("yyyymm", StringType()) \
       .execute()
    ```

    > **참고**: 언제든지 `display(df)` 명령을 실행하여 작업 진행 상황을 확인할 수 있습니다. 이 경우 'display(dfdimDate_gold)'를 실행하여 dimDate_gold dataframe의 내용을 볼 수 있습니다.

5. 새 코드 블록에서 날짜 차원인 **dimdate_gold**에 대한 dataframe을 생성하기 위해 **다음 코드를 추가하고 실행**합니다:

    ```python
   from pyspark.sql.functions import col, dayofmonth, month, year, date_format
    
   # Create dataframe for dimDate_gold
    
   dfdimDate_gold = df.dropDuplicates(["OrderDate"]).select(col("OrderDate"), \
           dayofmonth("OrderDate").alias("Day"), \
           month("OrderDate").alias("Month"), \
           year("OrderDate").alias("Year"), \
           date_format(col("OrderDate"), "MMM-yyyy").alias("mmmyyyy"), \
           date_format(col("OrderDate"), "yyyyMM").alias("yyyymm"), \
       ).orderBy("OrderDate")

   # Display the first 10 rows of the dataframe to preview your data

   display(dfdimDate_gold.head(10))
    ```

6. 데이터를 변환할 때 Notebook에서 무슨 일이 일어나는지 이해하고 확인할 수 있도록 코드를 새 코드 블록으로 분리하고 있습니다. 다른 새 코드 블록에서 새 데이터가 들어올 때 날짜 차원을 업데이트하기 위해 **다음 코드를 추가하고 실행**합니다:

    ```python
   from delta.tables import *
    
   deltaTable = DeltaTable.forPath(spark, 'Tables/dimdate_gold')
    
   dfUpdates = dfdimDate_gold
    
   deltaTable.alias('gold') \
     .merge(
       dfUpdates.alias('updates'),
       'gold.OrderDate = updates.OrderDate'
     ) \
      .whenMatchedUpdate(set =
       {
          
       }
     ) \
    .whenNotMatchedInsert(values =
       {
         "OrderDate": "updates.OrderDate",
         "Day": "updates.Day",
         "Month": "updates.Month",
         "Year": "updates.Year",
         "mmmyyyy": "updates.mmmyyyy",
         "yyyymm": "updates.yyyymm"
       }
     ) \
     .execute()
    ```

    이제 날짜 차원이 설정되었습니다. 이제 고객 차원을 생성합니다.
7. 고객 차원 테이블을 구축하기 위해 **새 코드 블록을 추가**하고 다음 코드를 붙여넣고 실행합니다:

    ```python
   from pyspark.sql.types import *
   from delta.tables import *
    
   # Create customer_gold dimension delta table
   DeltaTable.createIfNotExists(spark) \
       .tableName("sales.dimcustomer_gold") \
       .addColumn("CustomerName", StringType()) \
       .addColumn("Email",  StringType()) \
       .addColumn("First", StringType()) \
       .addColumn("Last", StringType()) \
       .addColumn("CustomerID", LongType()) \
       .execute()
    ```

8. 새 코드 블록에서 중복 고객을 제거하고, 특정 열을 선택하고, "CustomerName" 열을 분할하여 "First" 및 "Last" 이름 열을 생성하기 위해 **다음 코드를 추가하고 실행**합니다:

    ```python
   from pyspark.sql.functions import col, split
    
   # Create customer_silver dataframe
    
   dfdimCustomer_silver = df.dropDuplicates(["CustomerName","Email"]).select(col("CustomerName"),col("Email")) \
       .withColumn("First",split(col("CustomerName"), " ").getItem(0)) \
       .withColumn("Last",split(col("CustomerName"), " ").getItem(1)) 
    
   # Display the first 10 rows of the dataframe to preview your data

   display(dfdimCustomer_silver.head(10))
    ```

    여기서는 중복 제거, 특정 열 선택, "CustomerName" 열을 분할하여 "First" 및 "Last" 이름 열을 생성하는 등 다양한 변환을 수행하여 새 DataFrame dfdimCustomer_silver를 생성했습니다. 결과는 "CustomerName" 열에서 추출된 별도의 "First" 및 "Last" 이름 열을 포함하여 정리되고 구조화된 고객 데이터가 있는 DataFrame입니다.

9. 다음으로 **고객을 위한 ID 열을 생성**합니다. 새 코드 블록에 다음을 붙여넣고 실행합니다:

    ```python
   from pyspark.sql.functions import monotonically_increasing_id, col, when, coalesce, max, lit
    
   dfdimCustomer_temp = spark.read.table("Sales.dimCustomer_gold")
    
   MAXCustomerID = dfdimCustomer_temp.select(coalesce(max(col("CustomerID")),lit(0)).alias("MAXCustomerID")).first()[0]
    
   dfdimCustomer_gold = dfdimCustomer_silver.join(dfdimCustomer_temp,(dfdimCustomer_silver.CustomerName == dfdimCustomer_temp.CustomerName) & (dfdimCustomer_silver.Email == dfdimCustomer_temp.Email), "left_anti")
    
   dfdimCustomer_gold = dfdimCustomer_gold.withColumn("CustomerID",monotonically_increasing_id() + MAXCustomerID + 1)

   # Display the first 10 rows of the dataframe to preview your data

   display(dfdimCustomer_gold.head(10))
    ```

    여기서는 dimCustomer_gold 테이블에 이미 존재하는 중복을 제외하기 위해 left anti join을 수행하고, monotonically_increasing_id() 함수를 사용하여 고유한 CustomerID 값을 생성함으로써 고객 데이터(dfdimCustomer_silver)를 정리하고 변환합니다.

10. 이제 새 데이터가 들어올 때 고객 테이블이 최신 상태를 유지하도록 합니다. **새 코드 블록에** 다음을 붙여넣고 실행합니다:

    ```python
   from delta.tables import *

   deltaTable = DeltaTable.forPath(spark, 'Tables/dimcustomer_gold')
    
   dfUpdates = dfdimCustomer_gold
    
   deltaTable.alias('gold') \
     .merge(
       dfUpdates.alias('updates'),
       'gold.CustomerName = updates.CustomerName AND gold.Email = updates.Email'
     ) \
      .whenMatchedUpdate(set =
       {
          
       }
     ) \
    .whenNotMatchedInsert(values =
       {
         "CustomerName": "updates.CustomerName",
         "Email": "updates.Email",
         "First": "updates.First",
         "Last": "updates.Last",
         "CustomerID": "updates.CustomerID"
       }
     ) \
     .execute()
    ```

11. 이제 **이러한 단계를 반복하여 제품 차원을 생성**합니다. 새 코드 블록에 다음을 붙여넣고 실행합니다:

    ```python
   from pyspark.sql.types import *
   from delta.tables import *
    
   DeltaTable.createIfNotExists(spark) \
       .tableName("sales.dimproduct_gold") \
       .addColumn("ItemName", StringType()) \
       .addColumn("ItemID", LongType()) \
       .addColumn("ItemInfo", StringType()) \
       .execute()
    ```

12. **새 코드 블록을 추가**하여 **product_silver** dataframe을 생성합니다.
  
    ```python
   from pyspark.sql.functions import col, split, lit, when
    
   # Create product_silver dataframe
    
   dfdimProduct_silver = df.dropDuplicates(["Item"]).select(col("Item")) \
       .withColumn("ItemName",split(col("Item"), ", ").getItem(0)) \
       .withColumn("ItemInfo",when((split(col("Item"), ", ").getItem(1).isNull() | (split(col("Item"), ", ").getItem(1)=="")),lit("")).otherwise(split(col("Item"), ", ").getItem(1))) 
    
   # Display the first 10 rows of the dataframe to preview your data

   display(dfdimProduct_silver.head(10))
    ```

13. 이제 **dimProduct_gold 테이블**을 위한 ID를 생성합니다. 다음 구문을 새 코드 블록에 추가하고 실행합니다:

    ```python
   from pyspark.sql.functions import monotonically_increasing_id, col, lit, max, coalesce
    
   #dfdimProduct_temp = dfdimProduct_silver
   dfdimProduct_temp = spark.read.table("Sales.dimProduct_gold")
    
   MAXProductID = dfdimProduct_temp.select(coalesce(max(col("ItemID")),lit(0)).alias("MAXItemID")).first()[0]
    
   dfdimProduct_gold = dfdimProduct_silver.join(dfdimProduct_temp,(dfdimProduct_silver.ItemName == dfdimProduct_temp.ItemName) & (dfdimProduct_silver.ItemInfo == dfdimProduct_temp.ItemInfo), "left_anti")
    
   dfdimProduct_gold = dfdimProduct_gold.withColumn("ItemID",monotonically_increasing_id() + MAXProductID + 1)
    
   # Display the first 10 rows of the dataframe to preview your data

   display(dfdimProduct_gold.head(10))
    ```

    이는 테이블의 현재 데이터를 기반으로 다음 사용 가능한 제품 ID를 계산하고, 이 새 ID를 제품에 할당한 다음, 업데이트된 제품 정보를 표시합니다.

14. 다른 차원과 마찬가지로, 새 데이터가 들어올 때 제품 테이블이 최신 상태를 유지하도록 해야 합니다. **새 코드 블록에** 다음을 붙여넣고 실행합니다:

    ```python
   from delta.tables import *
    
   deltaTable = DeltaTable.forPath(spark, 'Tables/dimproduct_gold')
            
   dfUpdates = dfdimProduct_gold
            
   deltaTable.alias('gold') \
     .merge(
           dfUpdates.alias('updates'),
           'gold.ItemName = updates.ItemName AND gold.ItemInfo = updates.ItemInfo'
           ) \
           .whenMatchedUpdate(set =
           {
               
           }
           ) \
           .whenNotMatchedInsert(values =
            {
             "ItemName": "updates.ItemName",
             "ItemInfo": "updates.ItemInfo",
             "ItemID": "updates.ItemID"
             }
             ) \
             .execute()
    ```

    이제 차원이 구축되었으므로, 마지막 단계는 팩트 테이블을 생성하는 것입니다.

15. **새 코드 블록에** 다음 코드를 붙여넣고 실행하여 **팩트 테이블**을 생성합니다:

    ```python
   from pyspark.sql.types import *
   from delta.tables import *
    
   DeltaTable.createIfNotExists(spark) \
       .tableName("sales.factsales_gold") \
       .addColumn("CustomerID", LongType()) \
       .addColumn("ItemID", LongType()) \
       .addColumn("OrderDate", DateType()) \
       .addColumn("Quantity", IntegerType()) \
       .addColumn("UnitPrice", FloatType()) \
       .addColumn("Tax", FloatType()) \
       .execute()
    ```

16. **새 코드 블록에** 다음 코드를 붙여넣고 실행하여 고객 ID, 항목 ID, 주문 날짜, 수량, 단가 및 세금을 포함한 고객 및 제품 정보와 판매 데이터를 결합하는 **새 dataframe**을 생성합니다:

    ```python
   from pyspark.sql.functions import col
    
   dfdimCustomer_temp = spark.read.table("Sales.dimCustomer_gold")
   dfdimProduct_temp = spark.read.table("Sales.dimProduct_gold")
    
   df = df.withColumn("ItemName",split(col("Item"), ", ").getItem(0)) \
       .withColumn("ItemInfo",when((split(col("Item"), ", ").getItem(1).isNull() | (split(col("Item"), ", ").getItem(1)=="")),lit("")).otherwise(split(col("Item"), ", ").getItem(1))) \
    
    
   # Create Sales_gold dataframe
    
   dffactSales_gold = df.alias("df1").join(dfdimCustomer_temp.alias("df2"),(df.CustomerName == dfdimCustomer_temp.CustomerName) & (df.Email == dfdimCustomer_temp.Email), "left") \
           .join(dfdimProduct_temp.alias("df3"),(df.ItemName == dfdimProduct_temp.ItemName) & (df.ItemInfo == dfdimProduct_temp.ItemInfo), "left") \
       .select(col("df2.CustomerID") \
           , col("df3.ItemID") \
           , col("df1.OrderDate") \
           , col("df1.Quantity") \
           , col("df1.UnitPrice") \
           , col("df1.Tax") \
       ).orderBy(col("df1.OrderDate"), col("df2.CustomerID"), col("df3.ItemID"))
    
   # Display the first 10 rows of the dataframe to preview your data
    
   display(dffactSales_gold.head(10))
    ```

17. 이제 **새 코드 블록에** 다음 코드를 실행하여 판매 데이터가 최신 상태를 유지하도록 합니다:

    ```python
   from delta.tables import *
    
   deltaTable = DeltaTable.forPath(spark, 'Tables/factsales_gold')
    
   dfUpdates = dffactSales_gold
    
   deltaTable.alias('gold') \
     .merge(
       dfUpdates.alias('updates'),
       'gold.OrderDate = updates.OrderDate AND gold.CustomerID = updates.CustomerID AND gold.ItemID = updates.ItemID'
     ) \
      .whenMatchedUpdate(set =
       {
          
       }
     ) \
    .whenNotMatchedInsert(values =
       {
         "CustomerID": "updates.CustomerID",
         "ItemID": "updates.ItemID",
         "OrderDate": "updates.OrderDate",
         "Quantity": "updates.Quantity",
         "UnitPrice": "updates.UnitPrice",
         "Tax": "updates.Tax"
       }
     ) \
     .execute()
    ```

    여기서는 Delta Lake의 병합 (merge) 작업을 사용하여 factsales_gold 테이블을 새로운 판매 데이터(dffactSales_gold)와 동기화하고 업데이트합니다. 이 작업은 기존 데이터(실버 테이블)와 새 데이터(업데이트 DataFrame) 간의 주문 날짜, 고객 ID, 항목 ID를 비교하여 일치하는 레코드를 업데이트하고 필요에 따라 새 레코드를 삽입합니다.

이제 보고 및 분석에 사용할 수 있는 큐레이션되고 모델링된 **골드** 계층이 생겼습니다.

## (선택 사항) 의미론적 모델 생성

**참고**: 이 작업은 완전히 선택 사항이지만, 의미론적 모델을 생성하고 편집하려면 Power BI 라이선스 또는 Fabric F64 SKU가 필요합니다.

작업 영역에서 이제 골드 계층을 사용하여 보고서를 생성하고 데이터를 분석할 수 있습니다. 작업 영역에서 의미론적 모델 (semantic model)에 직접 액세스하여 보고를 위한 관계 및 측정값을 생성할 수 있습니다.

Lakehouse를 생성할 때 자동으로 생성되는 **기본 의미론적 모델 (default semantic model)**은 사용할 수 없습니다. 탐색기 (Explorer)에서 이 실습에서 생성한 골드 테이블을 포함하는 새 의미론적 모델을 생성해야 합니다.

1. 작업 영역에서 **Sales** Lakehouse로 이동합니다.
2. 탐색기 보기의 리본에서 **새 의미론적 모델 (New semantic model)**을 선택합니다.
3. 새 의미론적 모델에 **Sales_Gold** 이름을 할당합니다.
4. 변환된 골드 테이블을 선택하여 의미론적 모델에 포함하고 **확인 (Confirm)**을 선택합니다.
   - dimdate_gold
   - dimcustomer_gold
   - dimproduct_gold
   - factsales_gold

    이렇게 하면 Fabric에서 의미론적 모델이 열리며, 여기에 표시된 대로 관계 및 측정값을 생성할 수 있습니다:

    ![Fabric의 의미론적 모델 스크린샷](./Images/dataset-relationships.png)

여기에서 사용자 또는 데이터 팀의 다른 구성원은 Lakehouse의 데이터를 기반으로 보고서 및 대시보드를 생성할 수 있습니다. 이 보고서들은 Lakehouse의 골드 계층에 직접 연결되므로 항상 최신 데이터를 반영합니다.

## 리소스 정리

이 실습에서는 Microsoft Fabric Lakehouse에 메달리온 아키텍처를 생성하는 방법을 배웠습니다.

Lakehouse 탐색을 마쳤으면 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정 (Workspace settings)**을 선택합니다.
3. **일반 (General)** 섹션에서 **이 작업 영역 제거 (Remove this workspace)**를 선택합니다.
