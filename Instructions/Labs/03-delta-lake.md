---
lab:
    title: 'Use delta tables in Apache Spark'
    module: 'Work with Delta Lake tables in Microsoft Fabric'
---

# Apache Spark에서 Delta Table 사용

Microsoft Fabric Lakehouse의 Table은 오픈 소스 Delta Lake 형식 기반입니다. Delta Lake는 배치 및 스트리밍 데이터 모두에 대한 관계형 시맨틱 지원을 추가합니다. 이 실습에서는 Delta Table을 생성하고 SQL 쿼리를 사용하여 데이터를 탐색합니다.

이 실습은 완료하는 데 약 **45**분 정도 소요됩니다.

> [!NOTE]
> 이 실습을 완료하려면 [Microsoft Fabric tenant](https://learn.microsoft.com/fabric/get-started/fabric-trial)에 대한 액세스 권한이 필요합니다.

## Workspace 생성

Fabric에서 데이터를 작업하기 전에, Fabric Capacity가 활성화된 tenant에 Workspace를 생성하세요.

1. 브라우저에서 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric-developer) (`https://app.fabric.microsoft.com/home?experience=fabric-developer`)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2. 왼쪽의 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 유사합니다)를 선택합니다.
3. 원하는 이름으로 새 Workspace를 생성하고, **Advanced** 섹션에서 Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택합니다.
4. 새 Workspace가 열리면 비어 있어야 합니다.

    ![Fabric의 빈 Workspace 스크린샷.](./Images/new-workspace.png)

## Lakehouse 생성 및 파일 업로드

이제 Workspace를 만들었으니, 데이터용 Data Lakehouse를 생성할 차례입니다.

1. 왼쪽의 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Lakehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다. "Lakehouse schemas (Public Preview)" 옵션이 비활성화되어 있는지 확인합니다.

    >**참고**: **Create** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    1분 정도 후에 새 Lakehouse가 생성됩니다:

    ![새 Lakehouse 스크린샷.](./Images/new-lakehouse.png)

2. 새 Lakehouse를 확인하고, 왼쪽에 있는 **Explorer** 창을 통해 Lakehouse의 Table과 파일을 탐색할 수 있음에 유의하세요:

이제 Lakehouse로 데이터를 ingest(수집)할 수 있습니다. 이를 위한 여러 가지 방법이 있지만, 지금은 텍스트 파일을 로컬 컴퓨터(해당하는 경우 랩 VM)로 다운로드한 다음 Lakehouse에 업로드합니다.

1. `https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv`에서 [데이터 파일](https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv)을 다운로드하여 *products.csv*로 저장합니다.
2. Lakehouse가 열려 있는 웹 브라우저 탭으로 돌아가서 Explorer 창의 **Files** 폴더 옆에 있는 … 메뉴를 선택합니다. *products*라는 **New subfolder**를 생성합니다.
3. products 폴더의 … 메뉴에서 로컬 컴퓨터(해당하는 경우 랩 VM)에 있는 *products.csv* 파일을 **업로드**합니다.
4. 파일이 업로드되면 **products** 폴더를 선택하여 파일이 업로드되었는지 확인합니다. 다음 그림과 같습니다:

    ![Lakehouse에 업로드된 products.csv 스크린샷.](Images/upload-products.png)

## DataFrame에서 데이터 탐색

이제 데이터 작업을 위해 Fabric Notebook을 생성할 수 있습니다. Notebook은 코드를 작성하고 실행할 수 있는 대화형 환경을 제공합니다.

1. 왼쪽의 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Notebook**을 선택합니다.

    **Notebook 1**이라는 새 Notebook이 생성되고 열립니다.

    ![새 Notebook 스크린샷.](./Images/new-notebook.png)

2. Fabric은 생성하는 각 Notebook에 Notebook 1, Notebook 2 등과 같은 이름을 할당합니다. 메뉴의 **Home** 탭 위에 있는 이름 패널을 클릭하여 더 설명적인 이름으로 변경합니다.
3. 첫 번째 셀(현재 코드 셀)을 선택한 다음, 오른쪽 상단 도구 모음에서 **M↓** 버튼을 사용하여 Markdown 셀로 변환합니다. 그러면 셀에 포함된 텍스트가 서식 있는 텍스트로 표시됩니다.
4. 🖉 (편집) 버튼을 사용하여 셀을 편집 모드로 전환한 다음, 아래와 같이 Markdown을 수정합니다.

    ```markdown
    # Delta Lake tables
    Use this notebook to explore Delta Lake functionality
    ```

5. 셀 외부의 Notebook 아무 곳이나 클릭하여 편집을 중지합니다.
6. **Explorer** 창에서 **Add data items**를 선택한 다음, **Existing data sources**를 선택합니다. 이전에 생성한 Lakehouse에 연결합니다.
7. 새 코드 셀을 추가하고 다음 코드를 추가하여 정의된 Schema를 사용하여 products 데이터를 DataFrame으로 읽습니다:

    ```python
   from pyspark.sql.types import StructType, IntegerType, StringType, DoubleType

   # define the schema
   schema = StructType() \
   .add("ProductID", IntegerType(), True) \
   .add("ProductName", StringType(), True) \
   .add("Category", StringType(), True) \
   .add("ListPrice", DoubleType(), True)

   df = spark.read.format("csv").option("header","true").schema(schema).load("Files/products/products.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/products/products.csv".
   display(df)
    ```

> [!TIP]
> 셰브론 « 아이콘을 사용하여 Explorer 창을 숨기거나 표시할 수 있습니다. 이를 통해 Notebook 또는 파일에 집중할 수 있습니다.

8. 셀 왼쪽에 있는 **Run cell** (▷) 버튼을 사용하여 셀을 실행합니다.

> [!NOTE]
> 이 Notebook에서 코드를 처음 실행하는 것이므로 Spark session을 시작해야 합니다. 이는 첫 실행에 1분 정도 소요될 수 있음을 의미합니다. 이후 실행은 더 빨라집니다.

9. 셀 코드가 완료되면, 다음 그림과 유사하게 표시되어야 하는 셀 아래의 출력을 검토합니다:

    ![products.csv 데이터 스크린샷.](Images/products-schema.png)

## Delta Table 생성

*saveAsTable* 메서드를 사용하여 DataFrame을 Delta Table로 저장할 수 있습니다. Delta Lake는 Managed Table과 External Table 생성을 모두 지원합니다:

*   **Managed** Delta Table은 Fabric이 Schema Metadata와 데이터 파일 모두를 관리하므로 더 높은 성능을 제공합니다.
*   **External** Table은 Metadata가 Fabric에 의해 관리되면서 데이터를 외부에 저장할 수 있도록 합니다.

### Managed Table 생성

데이터 파일은 **Tables** 폴더에 생성됩니다.

1. 첫 번째 코드 셀에서 반환된 결과 아래에서 + Code 아이콘을 사용하여 새 코드 셀을 추가합니다.

> [!TIP]
> + Code 아이콘을 보려면 마우스를 현재 셀의 출력 바로 아래쪽과 왼쪽으로 이동합니다. 또는 메뉴 바의 Edit 탭에서 **+ Add code cell**을 선택합니다.

2. Managed Delta Table을 생성하려면 새 셀을 추가하고 다음 코드를 입력한 다음 셀을 실행합니다:

    ```python
   df.write.format("delta").saveAsTable("managed_products")
    ```

3. Explorer 창에서 Tables 폴더를 **새로 고침**하고 Tables 노드를 확장하여 **managed_products** Table이 생성되었는지 확인합니다.

> [!NOTE]
> 파일 이름 옆의 삼각형 아이콘은 Delta Table을 나타냅니다.

Managed Table의 파일은 Lakehouse의 **Tables** 폴더에 저장됩니다. *managed_products*라는 폴더가 생성되어 Table의 Parquet 파일과 delta_log 폴더를 저장합니다.

### External Table 생성

External Table을 생성할 수도 있으며, 이 테이블은 Lakehouse가 아닌 다른 곳에 저장될 수 있으며 Schema Metadata는 Lakehouse에 저장됩니다.

1. Explorer 창의 **Files** 폴더에 대한 … 메뉴에서 **Copy ABFS path**를 선택합니다. ABFS path는 Lakehouse Files 폴더에 대한 전체 경로입니다.

2. 새 코드 셀에 ABFS path를 붙여넣습니다. 다음 코드를 추가하고, 잘라내기 및 붙여넣기를 사용하여 abfs_path를 코드의 올바른 위치에 삽입합니다:

    ```python
   df.write.format("delta").saveAsTable("external_products", path="abfs_path/external_products")
    ```

3. 전체 경로는 다음과 유사하게 표시되어야 합니다:

    ```python
   abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files/external_products
    ```

4. 셀을 **실행**하여 DataFrame을 Files/external_products 폴더에 External Table로 저장합니다.

5. Explorer 창에서 Tables 폴더를 **새로 고침**하고 Tables 노드를 확장하여 external_products Table이 Schema Metadata를 포함하여 생성되었는지 확인합니다.

6. Explorer 창의 Files 폴더에 대한 … 메뉴에서 **새로 고침**을 선택합니다. 그런 다음 Files 노드를 확장하여 Table의 데이터 파일을 위한 external_products 폴더가 생성되었는지 확인합니다.

### Managed Table과 External Table 비교

%%sql magic command를 사용하여 Managed Table과 External Table 간의 차이점을 탐색해 보겠습니다.

1. 새 코드 셀에서 다음 코드를 실행합니다:

    ```python
   %%sql
   DESCRIBE FORMATTED managed_products;
    ```

2. 결과에서 Table의 Location 속성을 확인합니다. Data type 열의 Location 값을 클릭하여 전체 경로를 확인합니다. OneLake 저장 위치가 /Tables/managed_products로 끝나는 것을 확인하세요.

3. DESCRIBE 명령을 수정하여 다음 그림과 같이 external_products Table의 세부 정보를 표시합니다:

    ```python
   %%sql
   DESCRIBE FORMATTED external_products;
    ```

4. 셀을 실행하고 결과에서 Table의 Location 속성을 확인합니다. Data type 열을 넓혀 전체 경로를 확인하고 OneLake 저장 위치가 /Files/external_products로 끝나는 것을 확인하세요.

5. 새 코드 셀에서 다음 코드를 실행합니다:

    ```python
   %%sql
   DROP TABLE managed_products;
   DROP TABLE external_products;
    ```

6. Explorer 창에서 Tables 폴더를 **새로 고침**하여 Tables 노드에 Table이 나열되지 않는지 확인합니다.
7. Explorer 창에서 Files 폴더를 **새로 고침**하고 external_products 파일이 삭제되지 *않았는지* 확인합니다. 이 폴더를 선택하여 Parquet 데이터 파일과 _delta_log 폴더를 확인합니다.

External Table의 Metadata는 삭제되었지만, 데이터 파일은 삭제되지 않았습니다.

## SQL을 사용하여 Delta Table 생성

이제 %%sql magic command를 사용하여 Delta Table을 생성합니다.

1. 다른 코드 셀을 추가하고 다음 코드를 실행합니다:

    ```python
   %%sql
   CREATE TABLE products
   USING DELTA
   LOCATION 'Files/external_products';
    ```

2. Explorer 창의 **Tables** 폴더에 대한 … 메뉴에서 **새로 고침**을 선택합니다. 그런 다음 Tables 노드를 확장하여 *products*라는 새 Table이 나열되는지 확인합니다. 이어서 Table을 확장하여 Schema를 확인합니다.
3. 다른 코드 셀을 추가하고 다음 코드를 실행합니다:

    ```python
   %%sql
   SELECT * FROM products;
    ```

## Table 버전 관리 탐색

Delta Table의 트랜잭션 기록은 delta_log 폴더의 JSON 파일에 저장됩니다. 이 트랜잭션 로그를 사용하여 데이터 버전 관리를 관리할 수 있습니다.

1. Notebook에 새 코드 셀을 추가하고 산악 자전거 가격을 10% 인하하는 다음 코드를 실행합니다:

    ```python
   %%sql
   UPDATE products
   SET ListPrice = ListPrice * 0.9
   WHERE Category = 'Mountain Bikes';
    ```

2. 다른 코드 셀을 추가하고 다음 코드를 실행합니다:

    ```python
   %%sql
   DESCRIBE HISTORY products;
    ```

결과는 Table에 기록된 트랜잭션 기록을 보여줍니다.

3. 다른 코드 셀을 추가하고 다음 코드를 실행합니다:

    ```python
   delta_table_path = 'Files/external_products'
   # Get the current data
   current_data = spark.read.format("delta").load(delta_table_path)
   display(current_data)

   # Get the version 0 data
   original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
   display(original_data)
    ```

두 개의 결과 세트가 반환됩니다. 하나는 가격 인하 후의 데이터를 포함하고, 다른 하나는 원본 버전의 데이터를 보여줍니다.

## SQL 쿼리로 Delta Table 데이터 분석

SQL magic command를 사용하면 PySpark 대신 SQL 구문을 사용할 수 있습니다. 여기서는 `SELECT` 문을 사용하여 products Table에서 임시 View를 생성합니다.

1. 새 코드 셀을 추가하고 다음 코드를 실행하여 임시 View를 생성하고 표시합니다:

    ```python
   %%sql
   -- Create a temporary view
   CREATE OR REPLACE TEMPORARY VIEW products_view
   AS
       SELECT Category, COUNT(*) AS NumProducts, MIN(ListPrice) AS MinPrice, MAX(ListPrice) AS MaxPrice, AVG(ListPrice) AS AvgPrice
       FROM products
       GROUP BY Category;

   SELECT *
   FROM products_view
   ORDER BY Category;
    ```

2. 새 코드 셀을 추가하고 다음 코드를 실행하여 제품 수 기준으로 상위 10개 Category를 반환합니다:

    ```python
   %%sql
   SELECT Category, NumProducts
   FROM products_view
   ORDER BY NumProducts DESC
   LIMIT 10;
    ```

3. 데이터가 반환되면 **+ New chart**를 선택하여 제안된 차트 중 하나를 표시합니다.

    ![SQL SELECT 문 및 결과 스크린샷.](Images/sql-select.png)

또는 PySpark를 사용하여 SQL 쿼리를 실행할 수 있습니다.

1. 새 코드 셀을 추가하고 다음 코드를 실행합니다:

    ```python
   from pyspark.sql.functions import col, desc

   df_products = spark.sql("SELECT Category, MinPrice, MaxPrice, AvgPrice FROM products_view").orderBy(col("AvgPrice").desc())
   display(df_products.limit(6))
    ```

## 스트리밍 데이터에 Delta Table 사용

Delta Lake는 스트리밍 데이터를 지원합니다. Delta Table은 Spark Structured Streaming API를 사용하여 생성된 데이터 스트림의 Sink 또는 Source가 될 수 있습니다. 이 예시에서는 시뮬레이션된 IoT(Internet of Things) 시나리오에서 일부 스트리밍 데이터의 Sink로 Delta Table을 사용합니다.

1. 새 코드 셀을 추가하고 다음 코드를 추가하여 실행합니다:

    ```python
    from notebookutils import mssparkutils
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    # Create a folder
    inputPath = 'Files/data/'
    mssparkutils.fs.mkdirs(inputPath)

    # Create a stream that reads data from the folder, using a JSON schema
    jsonSchema = StructType([
    StructField("device", StringType(), False),
    StructField("status", StringType(), False)
    ])
    iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

    # Write some event data to the folder
    device_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''

    mssparkutils.fs.put(inputPath + "data.txt", device_data, True)

    print("Source stream created...")
    ```

*Source stream created…* 메시지가 표시되는지 확인합니다. 방금 실행한 코드는 가상의 IoT 디바이스에서 읽은 데이터를 나타내는, 일부 데이터가 저장된 폴더를 기반으로 스트리밍 데이터 Source를 생성했습니다.

2. 새 코드 셀에 다음 코드를 추가하고 실행합니다:

    ```python
   # Write the stream to a delta table
   delta_stream_table_path = 'Tables/iotdevicedata'
   checkpointpath = 'Files/delta/checkpoint'
   deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
   print("Streaming to delta sink...")
    ```

이 코드는 스트리밍 디바이스 데이터를 Delta 형식으로 iotdevicedata라는 폴더에 씁니다. Tables 폴더에 있는 폴더 위치의 경로 때문에 해당 폴더에 Table이 자동으로 생성됩니다.

3. 새 코드 셀에 다음 코드를 추가하고 실행합니다:

    ```python
   %%sql
   SELECT * FROM IotDeviceData;
    ```

이 코드는 스트리밍 Source에서 가져온 디바이스 데이터를 포함하는 IotDeviceData Table을 쿼리합니다.

4. 새 코드 셀에 다음 코드를 추가하고 실행합니다:

    ```python
   # Add more data to the source stream
   more_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''

   mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

이 코드는 더 많은 가상의 디바이스 데이터를 스트리밍 Source에 씁니다.

5. 다음 코드가 포함된 셀을 다시 실행합니다:

    ```python
   %%sql
   SELECT * FROM IotDeviceData;
    ```

이 코드는 IotDeviceData Table을 다시 쿼리합니다. 이 테이블에는 이제 스트리밍 Source에 추가된 추가 데이터가 포함되어야 합니다.

6. 새 코드 셀에 스트림을 중지하는 코드를 추가하고 셀을 실행합니다:

    ```python
   deltastream.stop()
    ```

## 리소스 정리

이 실습에서는 Microsoft Fabric에서 Delta Table을 사용하는 방법을 배웠습니다.

Lakehouse 탐색을 마쳤다면, 이 실습을 위해 생성한 Workspace를 삭제할 수 있습니다.

1. 왼쪽 바에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2. 도구 모음의 … 메뉴에서 **Workspace settings**를 선택합니다.
3. General 섹션에서 **Remove this workspace**를 선택합니다.
