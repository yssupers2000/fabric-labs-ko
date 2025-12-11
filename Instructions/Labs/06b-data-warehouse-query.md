---
lab:
    title: 'Query a data warehouse in Microsoft Fabric'
    module: 'Query a data warehouse in Microsoft Fabric'
---


```
# Microsoft Fabric에서 Data Warehouse 쿼리

Microsoft Fabric에서 Data Warehouse는 대규모 분석을 위한 관계형 데이터베이스를 제공합니다. Microsoft Fabric workspace에 내장된 풍부한 기능들은 고객이 Power BI와 DirectLake 모드로 통합된, 쉽게 소비 가능하며 항상 연결된 Semantic Model을 통해 인사이트를 얻는 시간을 단축할 수 있도록 합니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 체험](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## Workspace 만들기

Fabric에서 데이터 작업을 시작하기 전에, Fabric 체험이 활성화된 Workspace를 생성하세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
1.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455; 와 유사하게 생겼습니다)를 선택합니다.
1.  원하는 이름으로 새 Workspace를 생성하고, Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택합니다.
1.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## 샘플 Data Warehouse 만들기

이제 Workspace를 만들었으니, Data Warehouse를 생성할 차례입니다.

1.  왼쪽 메뉴 바에서 **만들기(Create)**를 선택합니다. *새로 만들기(New)* 페이지의 *Data Warehouse* 섹션에서 **샘플 Data Warehouse(Sample warehouse)**를 선택하고 **sample-dw**라는 새 Data Warehouse를 생성합니다.

    >**참고**: **만들기(Create)** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    1분 정도 후에 새 Warehouse가 생성되고 택시 탑승 분석 시나리오를 위한 샘플 데이터로 채워집니다.

    ![Screenshot of a new warehouse.](./Images/sample-data-warehouse.png)

## Data Warehouse 쿼리

SQL 쿼리 편집기는 IntelliSense, 코드 완성, 구문 강조 표시, 클라이언트 측 구문 분석(parsing) 및 유효성 검사를 지원합니다. Data Definition Language (DDL), Data Manipulation Language (DML) 및 Data Control Language (DCL) 문을 실행할 수 있습니다.

1.  **sample-dw** Data Warehouse 페이지에서 **새 SQL 쿼리(New SQL query)** 드롭다운 목록에서 **새 SQL 쿼리(New SQL query)**를 선택합니다.

1.  새 빈 쿼리 창에 다음 Transact-SQL 코드를 입력합니다.

    ```sql
    SELECT 
    D.MonthName, 
    COUNT(*) AS TotalTrips, 
    SUM(T.TotalAmount) AS TotalRevenue 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.MonthName;
    ```

1.  **&#9655; 실행(Run)** 버튼을 사용하여 SQL 스크립트를 실행하고 결과를 확인합니다. 결과는 월별 총 이동 횟수(trips)와 총 수익(revenue)을 보여줍니다.

1.  다음 Transact-SQL 코드를 입력합니다.

    ```sql
   SELECT 
    D.DayName, 
    AVG(T.TripDurationSeconds) AS AvgDuration, 
    AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1.  수정된 쿼리를 실행하고 결과를 확인합니다. 결과는 요일별 평균 이동 시간(duration)과 거리(distance)를 보여줍니다.

1.  다음 Transact-SQL 코드를 입력합니다.

    ```sql
    SELECT TOP 10 
        G.City, 
        COUNT(*) AS TotalTrips 
    FROM dbo.Trip AS T
    JOIN dbo.Geography AS G
        ON T.DropoffGeographyID=G.GeographyID
    GROUP BY G.City
    ORDER BY TotalTrips DESC;
    ```

1.  수정된 쿼리를 실행하고 결과를 확인합니다. 결과는 가장 인기 있는 상위 10개 탑승(pickup) 및 하차(dropoff) 위치를 보여줍니다.

1.  모든 쿼리 탭을 닫습니다.

## 데이터 일관성 확인

데이터 일관성을 확인하는 것은 분석 및 의사 결정에 있어 데이터가 정확하고 신뢰할 수 있도록 보장하는 데 중요합니다. 일관되지 않은 데이터는 잘못된 분석과 오해의 소지가 있는 결과를 초래할 수 있습니다.

데이터 일관성을 확인하기 위해 Data Warehouse를 쿼리해봅시다.

1.  **새 SQL 쿼리(New SQL query)** 드롭다운 목록에서 **새 SQL 쿼리(New SQL query)**를 선택합니다.

1.  새 빈 쿼리 창에 다음 Transact-SQL 코드를 입력합니다.

    ```sql
    -- Check for trips with unusually long duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds > 86400; -- 24 hours
    ```

1.  수정된 쿼리를 실행하고 결과를 확인합니다. 결과는 비정상적으로 긴 이동 시간(duration)을 가진 모든 이동(trips)의 세부 정보를 보여줍니다.

1.  **새 SQL 쿼리(New SQL query)** 드롭다운 목록에서 **새 SQL 쿼리(New SQL query)**를 선택하여 두 번째 쿼리 탭을 추가합니다. 그런 다음 새 빈 쿼리 탭에서 다음 코드를 실행합니다.

    ```sql
    -- Check for trips with negative trip duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

1.  새 빈 쿼리 창에 다음 Transact-SQL 코드를 입력하고 실행합니다.

    ```sql
    -- Remove trips with negative trip duration
    DELETE FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

    > **참고**: 일관성 없는 데이터를 처리하는 방법에는 여러 가지가 있습니다. 데이터를 제거하는 대신 평균(mean) 또는 중앙값(median)과 같은 다른 값으로 대체할 수도 있습니다.

1.  모든 쿼리 탭을 닫습니다.

## 뷰로 저장

보고서를 생성하기 위해 데이터를 사용할 사용자 그룹을 위해 특정 이동(trips)을 필터링해야 한다고 가정해 봅시다.

이전에 사용했던 쿼리를 기반으로 뷰를 생성하고, 필터를 추가해봅시다.

1.  **새 SQL 쿼리(New SQL query)** 드롭다운 목록에서 **새 SQL 쿼리(New SQL query)**를 선택합니다.

1.  새 빈 쿼리 창에 다음 Transact-SQL 코드를 다시 입력하고 실행합니다.

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1.  쿼리를 수정하여 `WHERE D.Month = 1`을 추가합니다. 이렇게 하면 1월의 레코드만 포함하도록 데이터가 필터링됩니다. 최종 쿼리는 다음과 같아야 합니다.

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    WHERE D.Month = 1
    GROUP BY D.DayName
    ```

1.  쿼리에서 SELECT 문의 텍스트를 선택합니다. 그런 다음 **&#9655; 실행(Run)** 버튼 옆에 있는 **뷰로 저장(Save as view)**을 선택합니다.

1.  **vw_JanTrip**이라는 새 뷰를 생성합니다.

1.  **탐색기(Explorer)**에서 **스키마(Schemas) >> dbo >> 뷰(Views)**로 이동합니다. 방금 생성한 *vw_JanTrip* 뷰를 확인합니다.

1.  모든 쿼리 탭을 닫습니다.

> **추가 정보**: Data Warehouse 쿼리에 대한 자세한 내용은 Microsoft Fabric 설명서의 [SQL 쿼리 편집기를 사용하여 쿼리(Query using the SQL query editor)](https://learn.microsoft.com/fabric/data-warehouse/sql-query-editor)를 참조하세요.

## 리소스 정리

이 연습에서는 Microsoft Fabric Data Warehouse의 데이터에 대한 인사이트를 얻기 위해 쿼리를 사용했습니다.

Data Warehouse 탐색을 마쳤다면, 이 연습을 위해 생성한 Workspace를 삭제할 수 있습니다.

1.  왼쪽 바에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
1.  **Workspace 설정(Workspace settings)**을 선택하고 **일반(General)** 섹션에서 아래로 스크롤하여 **이 Workspace 제거(Remove this workspace)**를 선택합니다.
1.  Workspace를 삭제하려면 **삭제(Delete)**를 선택합니다.