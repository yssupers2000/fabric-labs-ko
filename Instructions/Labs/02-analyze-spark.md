---
lab:
    title: 'Analyze data with Apache Spark'
    module: 'Use Apache Spark to work with files in a lakehouse'
---

# Fabric에서 Apache Spark로 데이터 분석

이 랩에서는 Fabric Lakehouse로 데이터를 수집하고 PySpark를 사용하여 데이터를 읽고 분석합니다.

이 랩을 완료하는 데 약 45분이 소요됩니다.

> [!Note]
> 이 실습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)에 대한 액세스 권한이 필요합니다.

## 작업 영역 만들기

Fabric에서 데이터를 사용하기 전에 Fabric Capacity가 활성화된 테넌트에 작업 영역을 만드세요.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric-developer`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric-developer)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2. 왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 비슷합니다)를 선택합니다.
3. 원하는 이름으로 새 작업 영역을 만들고, **Advanced** 섹션에서 Fabric Capacity(*Trial*, *Premium* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
4. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## Lakehouse를 만들고 파일 업로드

이제 작업 영역을 만들었으니, 데이터용 Data Lakehouse를 만들 차례입니다.

1. 왼쪽 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Lakehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다. "Lakehouse schemas (Public Preview)" 옵션이 비활성화되어 있는지 확인하세요.

    >**Note**: **Create** 옵션이 사이드바에 고정되어 있지 않으면 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    1분 정도 지나면 새 Lakehouse가 생성됩니다.

    ![새 Lakehouse 스크린샷.](./Images/new-lakehouse.png)

2. 새 Lakehouse를 보고, 왼쪽에 있는 **Lakehouse explorer** 창을 통해 Lakehouse의 테이블과 파일을 탐색할 수 있다는 점을 확인합니다.

이제 데이터를 Lakehouse로 수집할 수 있습니다. 이를 수행하는 방법은 여러 가지가 있지만, 지금은 텍스트 파일 폴더를 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에 다운로드한 다음 Lakehouse로 업로드합니다.

3. `https://github.com/MicrosoftLearning/dp-data/raw/main/orders.zip`에서 데이터 파일을 다운로드합니다.
4. 압축된 아카이브의 압축을 풀고 2019.csv, 2020.csv, 2021.csv 세 개의 CSV 파일이 포함된 *orders*라는 폴더가 있는지 확인합니다.
5. 새 Lakehouse로 돌아갑니다. **Explorer** 창에서 **Files** 폴더 옆의 **…** 메뉴를 선택하고 **Upload** 및 **Upload folder**를 선택합니다. 로컬 컴퓨터(또는 해당되는 경우 랩 VM)에서 orders 폴더로 이동하여 **Upload**를 선택합니다.
6. 파일 업로드가 완료되면 **Files**를 확장하고 **orders** 폴더를 선택합니다. 다음 그림과 같이 CSV 파일이 업로드되었는지 확인합니다.

    ![새 Fabric 작업 영역에 업로드된 CSV 파일 스크린샷.](Images/uploaded-files.png)

## Notebook 만들기

이제 데이터를 사용하여 Fabric Notebook을 만들 수 있습니다. Notebook은 코드를 작성하고 실행할 수 있는 대화형 환경을 제공합니다.

1. 왼쪽 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Notebook**을 선택합니다.

    **Notebook 1**이라는 새 Notebook이 생성되어 열립니다.

    ![새 Notebook 스크린샷.](./Images/new-notebook.png)

2. Fabric은 Notebook 1, Notebook 2 등과 같이 생성하는 각 Notebook에 이름을 할당합니다. 메뉴의 **Home** 탭 위에 있는 이름 패널을 클릭하여 이름을 더 설명적인 이름으로 변경하세요.
3. 첫 번째 셀(현재 코드 셀)을 선택한 다음, 오른쪽 상단 도구 모음에서 **M↓** 버튼을 사용하여 Markdown 셀로 변환합니다. 그러면 셀에 포함된 텍스트가 서식 있는 텍스트로 표시됩니다.
4. 🖉 (Edit) 버튼을 사용하여 셀을 편집 모드로 전환한 다음, 아래 그림과 같이 Markdown을 수정합니다.

    ```markdown
   # Sales order data exploration
   Use this notebook to explore sales order data
    ```

    ![Markdown 셀이 있는 Fabric Notebook 스크린샷.](Images/name-notebook-markdown.png)

작업을 마쳤으면 셀 외부의 Notebook 아무 곳이나 클릭하여 편집을 중지합니다.

## DataFrame 만들기

이제 작업 영역, Lakehouse 및 Notebook을 만들었으므로 데이터를 사용할 준비가 되었습니다. Fabric Notebook의 기본 언어이자 Spark에 최적화된 Python 버전인 PySpark를 사용합니다.

>[!NOTE]
> Fabric Notebook은 Scala, R 및 Spark SQL을 포함한 여러 프로그래밍 언어를 지원합니다.

1. 왼쪽 바에서 새 작업 영역을 선택합니다. Lakehouse 및 Notebook을 포함하여 작업 영역에 포함된 항목 목록이 표시됩니다.
2. Lakehouse를 선택하여 **orders** 폴더를 포함한 Explorer 창을 표시합니다.
3. 상단 메뉴에서 **Open notebook**, **Existing notebook**을 선택한 다음 이전에 만든 Notebook을 엽니다. 이제 Notebook이 Explorer 창 옆에 열려 있어야 합니다. Lakehouses를 확장하고 Files 목록을 확장한 다음 orders 폴더를 선택합니다. 업로드한 CSV 파일이 다음과 같이 Notebook 편집기 옆에 나열됩니다.

    ![Explorer 보기의 csv 파일 스크린샷.](Images/explorer-notebook-view.png)

4. 2019.csv에 대한 … 메뉴에서 **Load data** > **Spark**를 선택합니다. 다음 코드가 새 코드 셀에 자동으로 생성됩니다.

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/orders/2019.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/orders/2019.csv".
   display(df)
    ```

>[!TIP]
> « 아이콘을 사용하여 왼쪽의 Explorer 창을 숨기거나 표시할 수 있습니다. 이렇게 하면 Notebook에 더 많은 공간이 생깁니다.

5. 셀 왼쪽에 있는 ▷ **Run cell**을 선택하여 코드를 실행합니다.

>[!NOTE]
> Spark 코드를 처음 실행하면 Spark Session이 시작됩니다. 몇 초 이상 걸릴 수 있습니다. 동일한 Session 내에서 후속 실행은 더 빠릅니다.

6. 셀 코드가 완료되면 셀 아래의 출력을 검토합니다. 다음 그림과 같아야 합니다.

    ![자동 생성된 코드 및 데이터를 보여주는 스크린샷.](Images/auto-generated-load.png)

7. 출력은 2019.csv 파일의 데이터가 열과 행으로 표시되는 것을 보여줍니다. 열 머리글에 데이터의 첫 번째 줄이 포함되어 있음을 확인합니다. 이를 수정하려면 코드의 첫 번째 줄을 다음과 같이 수정해야 합니다.

    ```python
   df = spark.read.format("csv").option("header","false").load("Files/orders/2019.csv")
    ```

8. DataFrame이 첫 번째 행을 데이터로 올바르게 식별하도록 코드를 다시 실행합니다. 열 이름이 이제 _c0, _c1 등으로 변경되었음을 확인합니다.

9. 설명적인 열 이름은 데이터를 이해하는 데 도움이 됩니다. 의미 있는 열 이름을 만들려면 스키마와 데이터 형식을 정의해야 합니다. 또한 데이터 형식을 정의하기 위해 표준 Spark SQL 유형 집합을 가져와야 합니다. 기존 코드를 다음으로 바꿉니다.

    ```python
   from pyspark.sql.types import *

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

   df = spark.read.format("csv").schema(orderSchema).load("Files/orders/2019.csv")

   display(df)
    ```

10. 셀을 실행하고 출력을 검토합니다.

    ![스키마가 정의된 코드와 데이터를 보여주는 스크린샷.](Images/define-schema.png)

11. 이 DataFrame에는 2019.csv 파일의 데이터만 포함되어 있습니다. orders 폴더의 모든 파일을 읽을 수 있도록 파일 경로에 * 와일드카드를 사용하도록 코드를 수정합니다.

    ```python
    from pyspark.sql.types import *

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

    df = spark.read.format("csv").schema(orderSchema).load("Files/orders/*.csv")

    display(df)
    ```

12. 수정된 코드를 실행하면 2019년, 2020년 및 2021년 판매를 볼 수 있습니다. 행의 일부만 표시되므로 모든 연도의 행이 표시되지 않을 수 있습니다.

>[!NOTE]
> 결과 옆의 **...**를 선택하여 셀의 출력을 숨기거나 표시할 수 있습니다. 이렇게 하면 Notebook에서 작업하기가 더 쉽습니다.

## DataFrame에서 데이터 탐색

DataFrame 객체는 데이터를 필터링, 그룹화 및 조작하는 기능과 같은 추가 기능을 제공합니다.

### DataFrame 필터링

1. 현재 셀 또는 해당 출력 위나 아래에 마우스를 가져가면 나타나는 **+ Code**를 선택하여 코드 셀을 추가합니다. 또는 리본 메뉴에서 **Edit** 및 **+ Add code cell below**를 선택합니다.

2. 다음 코드는 두 개의 열만 반환되도록 데이터를 필터링합니다. 또한 *count* 및 *distinct*를 사용하여 레코드 수를 요약합니다.

    ```python
    customers = df['CustomerName', 'Email']

    print(customers.count())
    print(customers.distinct().count())

    display(customers.distinct())
    ```

3. 코드를 실행하고 출력을 검토합니다.

    *   코드는 원본 **df** DataFrame에서 열의 하위 집합을 포함하는 **customers**라는 새 DataFrame을 만듭니다. DataFrame 변환을 수행할 때 원본 DataFrame을 수정하지 않고 새 DataFrame을 반환합니다.
    *   동일한 결과를 얻는 다른 방법은 select 메서드를 사용하는 것입니다.

    ```
   customers = df.select("CustomerName", "Email")
    ```

    *   DataFrame 함수 *count* 및 *distinct*는 고객 수 및 고유 고객 수에 대한 합계를 제공하는 데 사용됩니다.

4. *select*와 *where* 함수를 사용하여 코드의 첫 번째 줄을 다음과 같이 수정합니다.

    ```python
   customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
   print(customers.count())
   print(customers.distinct().count())

   display(customers.distinct())
    ```

5. 수정된 코드를 실행하여 Road-250 Red, 52 제품을 구매한 고객만 선택합니다. 여러 함수를 함께 "연결"하여 한 함수의 출력이 다음 함수의 입력이 되도록 할 수 있습니다. 이 경우 *select* 메서드에 의해 생성된 DataFrame은 필터링 기준을 적용하는 데 사용되는 **where** 메서드의 원본 DataFrame입니다.

### DataFrame에서 데이터 집계 및 그룹화

1. 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   productSales = df.select("Item", "Quantity").groupBy("Item").sum()

   display(productSales)
    ```

2. 코드를 실행합니다. 결과는 제품별로 그룹화된 주문 수량의 합계를 보여줍니다. *groupBy* 메서드는 Item별로 행을 그룹화하고, 후속 *sum* 집계 함수는 나머지 숫자 열(이 경우 *Quantity*)에 적용됩니다.

3. Notebook에 다른 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   from pyspark.sql.functions import *

   yearlySales = df.select(year(col("OrderDate")).alias("Year")).groupBy("Year").count().orderBy("Year")

   display(yearlySales)
    ```

4. 셀을 실행합니다. 출력을 검토합니다. 이제 결과는 연도별 판매 주문 수를 보여줍니다.

    *   *import* 문을 사용하면 Spark SQL 라이브러리를 사용할 수 있습니다.
    *   *select* 메서드는 SQL year 함수와 함께 *OrderDate* 필드의 연도 구성 요소를 추출하는 데 사용됩니다.
    *   *alias* 메서드는 추출된 연도 값에 열 이름을 할당하는 데 사용됩니다.
    *   *groupBy* 메서드는 파생된 Year 열별로 데이터를 그룹화합니다.
    *   각 그룹의 행 수는 *orderBy* 메서드가 결과 DataFrame을 정렬하는 데 사용되기 전에 계산됩니다.

    ![DataFrame에서 데이터 집계 및 그룹화 결과를 보여주는 스크린샷.](Images/spark-sql-dataframe.png)

## Spark를 사용하여 데이터 파일 변환

데이터 엔지니어와 데이터 과학자에게 일반적인 작업은 추가 다운스트림 처리 또는 분석을 위해 데이터를 변환하는 것입니다.

### DataFrame 메서드 및 함수를 사용하여 데이터 변환

1. Notebook에 코드 셀을 추가하고 다음을 입력합니다.

    ```python
   from pyspark.sql.functions import *

   # Create Year and Month columns
   transformed_df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

   # Create the new FirstName and LastName fields
   transformed_df = transformed_df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)).withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

   # Filter and reorder columns
   transformed_df = transformed_df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month", "FirstName", "LastName", "Email", "Item", "Quantity", "UnitPrice", "Tax"]

   # Display the first five orders
   display(transformed_df.limit(5))
    ```

2. 셀을 실행합니다. 원본 주문 데이터에서 다음 변환을 통해 새 DataFrame이 생성됩니다.

    - OrderDate 열을 기반으로 Year 및 Month 열이 추가되었습니다.
    - CustomerName 열을 기반으로 FirstName 및 LastName 열이 추가되었습니다.
    - 열이 필터링되고 재정렬되었으며, CustomerName 열이 제거되었습니다.

3. 출력을 검토하고 데이터에 변환이 이루어졌는지 확인합니다.

Spark SQL 라이브러리를 사용하여 행을 필터링하고, 열을 파생, 제거, 이름 변경하고, 다른 데이터 수정 사항을 적용하여 데이터를 변환할 수 있습니다.

>[!TIP]
> DataFrame 개체에 대한 자세한 내용은 [Apache Spark dataframe](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html) 설명서를 참조하세요.

### 변환된 데이터 저장

이 시점에서 변환된 데이터를 저장하여 추가 분석에 사용할 수 있도록 할 수 있습니다.

*Parquet*은 데이터를 효율적으로 저장하고 대부분의 대규모 데이터 분석 시스템에서 지원되므로 인기 있는 데이터 저장 형식입니다. 실제로 데이터 변환 요구 사항은 CSV와 같은 한 형식의 데이터를 Parquet으로 변환하는 것입니다.

1. 변환된 DataFrame을 Parquet 형식으로 저장하려면 코드 셀을 추가하고 다음 코드를 추가합니다.

    ```python
   transformed_df.write.mode("overwrite").parquet('Files/transformed_data/orders')

   print ("Transformed data saved!")
    ```

2. 셀을 실행하고 데이터가 저장되었다는 메시지가 나타날 때까지 기다립니다. 그런 다음 왼쪽의 Explorer 창에서 Files 노드의 … 메뉴에서 **Refresh**를 선택합니다. transformed_data 폴더를 선택하여 orders라는 새 폴더가 포함되어 있는지 확인합니다. 이 폴더에는 하나 이상의 Parquet 파일이 포함되어 있습니다.

3. 다음 코드를 포함하는 셀을 추가합니다.

    ```python
   orders_df = spark.read.format("parquet").load("Files/transformed_data/orders")
   display(orders_df)
    ```

4. 셀을 실행합니다. *transformed_data/orders* 폴더의 Parquet 파일에서 새 DataFrame이 생성됩니다. 결과에 Parquet 파일에서 로드된 주문 데이터가 표시되는지 확인합니다.

    ![Parquet 파일을 보여주는 스크린샷.](Images/parquet-files.png)

### 분할된 파일에 데이터 저장

대량의 데이터를 처리할 때 분할(partitioning)은 성능을 크게 향상시키고 데이터를 더 쉽게 필터링할 수 있도록 합니다.

1. 데이터프레임을 저장하고 데이터를 Year 및 Month별로 분할하는 코드를 포함하는 셀을 추가합니다.

    ```python
   orders_df.write.partitionBy("Year","Month").mode("overwrite").parquet("Files/partitioned_data")

   print ("Transformed data saved!")
    ```

2. 셀을 실행하고 데이터가 저장되었다는 메시지가 나타날 때까지 기다립니다. 그런 다음 왼쪽의 Lakehouses 창에서 Files 노드의 … 메뉴에서 **Refresh**를 선택하고 partitioned_data 폴더를 확장하여 *Year=xxxx* 형식의 폴더 계층 구조가 포함되어 있는지 확인합니다. 각 폴더에는 *Month=xxxx* 형식의 폴더가 포함되어 있습니다. 각 Month 폴더에는 해당 월의 주문이 포함된 Parquet 파일이 있습니다.

    ![Year 및 Month별로 분할된 데이터를 보여주는 스크린샷.](Images/partitioned-data.png)

3. orders.parquet 파일에서 새 DataFrame을 로드하는 다음 코드를 포함하는 새 셀을 추가합니다.

    ```python
   orders_2021_df = spark.read.format("parquet").load("Files/partitioned_data/Year=2021/Month=*")

   display(orders_2021_df)
    ```

4. 셀을 실행하고 결과에 2021년 판매 주문 데이터가 표시되는지 확인합니다. 경로에 지정된 분할 열(Year 및 Month)이 DataFrame에 포함되지 않습니다.

## 테이블 및 SQL 작업

이제 DataFrame 객체의 기본 메서드를 사용하여 파일에서 데이터를 쿼리하고 분석하는 방법을 살펴보았습니다. 그러나 SQL 구문을 사용하여 테이블로 작업하는 것이 더 익숙할 수 있습니다. Spark는 관계형 테이블을 정의할 수 있는 Metastore를 제공합니다.

Spark SQL 라이브러리는 Metastore의 테이블을 쿼리하기 위한 SQL 문 사용을 지원합니다. 이를 통해 데이터 레이크의 유연성과 관계형 데이터 웨어하우스의 구조화된 데이터 스키마 및 SQL 기반 쿼리가 제공됩니다. 이것이 "Data Lakehouse"라는 용어가 생겨난 이유입니다.

### 테이블 만들기

Spark Metastore의 테이블은 데이터 레이크에 있는 파일에 대한 관계형 추상화입니다. 테이블은 Metastore에 의해 *관리*되거나, Metastore와 독립적으로 관리되는 *외부* 테이블일 수 있습니다.

1. Notebook에 코드 셀을 추가하고 sales order 데이터의 DataFrame을 *salesorders*라는 테이블로 저장하는 다음 코드를 입력합니다.

    ```python
    # Create a new table
    df.write.format("delta").saveAsTable("salesorders")

    # Get the table description
    spark.sql("DESCRIBE EXTENDED salesorders").show(truncate=False)
    ```

>[!NOTE]
> 이 예에서는 명시적인 경로가 제공되지 않으므로 테이블에 대한 파일은 Metastore에 의해 관리됩니다. 또한 테이블은 관계형 데이터베이스 기능을 테이블에 추가하는 Delta 형식으로 저장됩니다. 여기에는 트랜잭션, 행 버전 관리 및 기타 유용한 기능 지원이 포함됩니다. Fabric의 Data Lakehouse에는 Delta 형식으로 테이블을 만드는 것이 선호됩니다.

2. 코드 셀을 실행하고 새 테이블의 정의를 설명하는 출력을 검토합니다.

3. **Explorer** 창에서 Tables 폴더의 … 메뉴에서 **Refresh**를 선택합니다. 그런 다음 **Tables** 노드를 확장하고 **salesorders** 테이블이 생성되었는지 확인합니다.

    ![salesorders 테이블이 생성되었음을 보여주는 스크린샷.](Images/salesorder-table.png)

4. salesorders 테이블의 … 메뉴에서 **Load data** > **Spark**를 선택합니다. 다음 코드와 유사한 코드를 포함하는 새 코드 셀이 추가됩니다.

    ```pyspark
   df = spark.sql("SELECT * FROM [your_lakehouse].salesorders LIMIT 1000")

   display(df)
    ```

5. 새 코드를 실행합니다. 이 코드는 Spark SQL 라이브러리를 사용하여 PySpark 코드에 *salesorder* 테이블에 대한 SQL 쿼리를 포함하고 쿼리 결과를 DataFrame으로 로드합니다.

### 셀에서 SQL 코드 실행

PySpark 코드가 포함된 셀에 SQL 문을 포함할 수 있다는 것은 유용하지만, 데이터 분석가는 종종 SQL에서 직접 작업하기를 원합니다.

1. Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```SparkSQL
   %%sql
   SELECT YEAR(OrderDate) AS OrderYear,
          SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
   FROM salesorders
   GROUP BY YEAR(OrderDate)
   ORDER BY OrderYear;
    ```

2. 셀을 실행하고 결과를 검토합니다. 다음 사항을 관찰합니다.

    *   셀 시작 부분의 **%%sql** 명령(Magic이라고 함)은 언어를 PySpark 대신 Spark SQL로 변경합니다.
    *   SQL 코드는 이전에 생성한 *salesorders* 테이블을 참조합니다.
    *   SQL 쿼리의 출력은 셀 아래에 결과로 자동으로 표시됩니다.

>[!NOTE]
> Spark SQL 및 DataFrame에 대한 자세한 내용은 [Apache Spark SQL](https://spark.apache.org/sql/) 설명서를 참조하세요.

## Spark로 데이터 시각화

차트는 수천 개의 데이터 행을 스캔하는 것보다 패턴과 추세를 더 빨리 파악하는 데 도움이 됩니다. Fabric Notebook에는 기본 제공 차트 보기가 포함되어 있지만 복잡한 차트용으로 설계되지는 않았습니다. DataFrame의 데이터에서 차트를 만드는 방법을 더 자세히 제어하려면 *matplotlib* 또는 *seaborn*과 같은 Python 그래픽 라이브러리를 사용하세요.

### 결과를 차트로 보기

1. 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   %%sql
   SELECT * FROM salesorders
    ```

2. 이전에 만든 salesorders 뷰에서 데이터를 표시하기 위해 코드를 실행합니다. 셀 아래 결과 섹션에서 **+ New chart**를 선택합니다.

3. 결과 섹션의 오른쪽 하단에 있는 **Build my own** 버튼을 사용하여 차트 설정을 지정합니다.

    *   차트 유형 (Chart type): 막대형 차트 (Bar chart)
    *   X축 (X-axis): Item
    *   Y축 (Y-axis): Quantity
    *   계열 그룹 (Series Group): 비워 두기 (leave blank)
    *   집계 (Aggregation): 합계 (Sum)
    *   누락 및 NULL 값 (Missing and NULL values): 0으로 표시 (Display as 0)
    *   누적 (Stacked): 선택 해제 (Unselected)

4. 차트는 다음 그림과 비슷하게 보여야 합니다.

    ![Fabric Notebook 차트 보기 스크린샷.](Images/built-in-chart.png)

### matplotlib 시작하기

1. 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                   SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue, \
                   COUNT(DISTINCT SalesOrderNumber) AS YearlyCounts \
               FROM salesorders \
               GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
               ORDER BY OrderYear"
   df_spark = spark.sql(sqlQuery)
   df_spark.show()
    ```

2. 코드를 실행합니다. 연간 수익과 주문 수를 포함하는 Spark DataFrame이 반환됩니다. 데이터를 차트로 시각화하기 위해 먼저 matplotlib Python 라이브러리를 사용합니다. 이 라이브러리는 많은 다른 라이브러리의 핵심 플로팅 라이브러리이며 차트를 만드는 데 뛰어난 유연성을 제공합니다.

3. 새 코드 셀을 추가하고 다음 코드를 추가합니다.

    ```python
   from matplotlib import pyplot as plt

   # matplotlib requires a Pandas dataframe, not a Spark one
   df_sales = df_spark.toPandas()

   # Create a bar plot of revenue by year
   plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'])

   # Display the plot
   plt.show()
    ```

4. 셀을 실행하고 결과를 검토합니다. 각 연도의 총 매출 총액을 나타내는 세로 막대형 차트(column chart)로 구성됩니다. 코드를 검토하고 다음 사항에 유의합니다.

    *   matplotlib 라이브러리는 Pandas DataFrame을 필요로 하므로 Spark SQL 쿼리에서 반환된 Spark DataFrame을 변환해야 합니다.
    *   matplotlib 라이브러리의 핵심은 *pyplot* 객체입니다. 이것은 대부분의 플로팅 기능의 기반입니다.
    *   기본 설정으로 사용할 수 있는 차트가 생성되지만, 사용자 지정할 수 있는 범위가 상당합니다.

5. 차트를 다음과 같이 그리기 위해 코드를 수정합니다.

    ```python
    from matplotlib import pyplot as plt

    # Clear the plot area
    plt.clf()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

6. 코드 셀을 다시 실행하고 결과를 확인합니다. 이제 차트를 더 쉽게 이해할 수 있습니다.
7. Plot은 Figure 내에 포함됩니다. 이전 예제에서는 Figure가 암시적으로 생성되었지만 명시적으로 생성할 수도 있습니다. 차트를 다음과 같이 그리기 위해 코드를 수정합니다.

    ```python
   from matplotlib import pyplot as plt

   # Clear the plot area
   plt.clf()

   # Create a Figure
   fig = plt.figure(figsize=(8,3))

   # Create a bar plot of revenue by year
   plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

   # Customize the chart
   plt.title('Revenue by Year')
   plt.xlabel('Year')
   plt.ylabel('Revenue')
   plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
   plt.xticks(rotation=45)

   # Show the figure
   plt.show()
    ```

8. 코드 셀을 다시 실행하고 결과를 확인합니다. Figure는 Plot의 모양과 크기를 결정합니다.
9. Figure는 자체 축에 여러 Subplot을 포함할 수 있습니다. 차트를 다음과 같이 그리기 위해 코드를 수정합니다.

    ```python
   from matplotlib import pyplot as plt

   # Clear the plot area
   plt.clf()

   # Create a figure for 2 subplots (1 row, 2 columns)
   fig, ax = plt.subplots(1, 2, figsize = (10,4))

   # Create a bar plot of revenue by year on the first axis
   ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
   ax[0].set_title('Revenue by Year')

   # Create a pie chart of yearly order counts on the second axis
   ax[1].pie(df_sales['YearlyCounts'])
   ax[1].set_title('Orders per Year')
   ax[1].legend(df_sales['OrderYear'])

   # Add a title to the Figure
   fig.suptitle('Sales Data')

   # Show the figure
   plt.show()
    ```

10. 코드 셀을 다시 실행하고 결과를 확인합니다.

>[!NOTE]
> matplotlib을 사용한 Plotting에 대한 자세한 내용은 [matplotlib](https://matplotlib.org/) 설명서를 참조하세요.

### seaborn 라이브러리 사용

*matplotlib*을 사용하면 다양한 차트 유형을 만들 수 있지만, 최상의 결과를 얻기 위해서는 다소 복잡한 코드가 필요할 수 있습니다. 이러한 이유로 matplotlib의 복잡성을 추상화하고 기능을 향상시키기 위해 새로운 라이브러리가 구축되었습니다. 그중 하나가 seaborn 라이브러리입니다.

1. Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   import seaborn as sns

   # Clear the plot area
   plt.clf()

   # Create a bar chart
   ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)

   plt.show()
    ```

2. seaborn 라이브러리를 사용하여 생성된 막대형 차트를 표시하기 위해 코드를 실행합니다.
3. 코드를 다음과 같이 수정합니다.

    ```python
   import seaborn as sns

   # Clear the plot area
   plt.clf()

   # Set the visual theme for seaborn
   sns.set_theme(style="whitegrid")

   # Create a bar chart
   ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)

   plt.show()
    ```

4. 수정된 코드를 실행하고 seaborn을 통해 Plot에 대한 색상 테마를 설정할 수 있음을 확인합니다.
5. 코드를 다시 다음과 같이 수정합니다.

    ```python
    import seaborn as sns

    # Clear the plot area
    plt.clf()

    # Create a line chart
    ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)

    plt.show()
    ```

6. 수정된 코드를 실행하여 연간 수익을 선형 차트 (line chart)로 확인합니다.

>[!NOTE]
> seaborn을 사용한 Plotting에 대한 자세한 내용은 [seaborn](https://seaborn.pydata.org/index.html) 설명서를 참조하세요.

## 리소스 정리

이 실습에서는 Spark를 사용하여 Microsoft Fabric에서 데이터와 작업하는 방법을 배웠습니다.

데이터 탐색을 마쳤으면 Spark Session을 종료하고 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. Notebook 메뉴에서 **Stop session**을 선택하여 Spark Session을 종료합니다.
2. 왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
3. **Workspace settings**를 선택하고 **General** 섹션에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.
4. **Delete**를 선택하여 작업 영역을 삭제합니다.
