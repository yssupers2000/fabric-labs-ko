---
lab:
    title: 'Work with data in a Microsoft Fabric eventhouse'
    module: 'Work with data in a Microsoft Fabric eventhouse'
---

# Microsoft Fabric Eventhouse에서 데이터 작업

Microsoft Fabric에서 *Eventhouse*는 이벤트와 관련된 실시간 데이터를 저장하는 데 사용되며, 종종 스트리밍 데이터 소스에서 *Eventstream*을 통해 캡처됩니다.

Eventhouse 내에서 데이터는 하나 이상의 KQL database에 저장되며, 각 KQL database는 Kusto Query Language (KQL) 또는 Structured Query Language (SQL)의 하위 집합을 사용하여 쿼리할 수 있는 테이블 및 기타 개체를 포함합니다.

이 실습에서는 택시 탑승과 관련된 일부 샘플 데이터로 Eventhouse를 생성하고 채운 다음, KQL 및 SQL을 사용하여 데이터를 쿼리합니다.

이 실습을 완료하는 데 약 **25**분이 소요됩니다.

## Workspace 생성

Fabric에서 데이터 작업을 시작하기 전에, Fabric Capacity가 활성화된 Workspace를 생성하세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽의 메뉴 바에서 **Workspaces**를 선택하세요 (아이콘은 &#128455;와 비슷하게 생겼습니다).
3.  원하는 이름으로 새 Workspace를 생성하고, Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택하세요.
4.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Eventhouse 생성

이제 Fabric Capacity를 지원하는 Workspace가 있으므로, 그 안에 Eventhouse를 생성할 수 있습니다.

1.  왼쪽의 메뉴 바에서 **Workloads**를 선택하세요. 그런 다음 **Real-Time Intelligence** 타일(tile)을 선택하세요.
2.  **Real-Time Intelligence** 홈 페이지에서 **Explore Real-Time Intelligence Sample** 타일(tile)을 선택하세요. 그러면 자동으로 **RTISample**이라는 Eventhouse가 생성됩니다:

   ![Screenshot of a new eventhouse with sample data.](./Images/create-eventhouse-sample.png)

3.  왼쪽 창에서 Eventhouse가 Eventhouse와 동일한 이름의 KQL database를 포함하고 있음을 확인하세요.
4.  **Bikestream** 테이블도 생성되었는지 확인하세요.

## KQL을 사용하여 데이터 쿼리

Kusto Query Language (KQL)는 KQL database를 쿼리하는 데 사용할 수 있는 직관적이고 포괄적인 언어입니다.

### KQL을 사용하여 테이블에서 데이터 검색

1.  Eventhouse 창의 왼쪽 창에서 KQL database 아래에 있는 기본 **queryset** 파일을 선택하세요. 이 파일에는 시작하는 데 도움이 되는 몇 가지 샘플 KQL 쿼리가 포함되어 있습니다.
2.  첫 번째 예제 쿼리를 다음과 같이 수정하세요.

    ```kql
    Bikestream
    | take 100
    ```

    > **참고:**
    > KQL에서 Pipe ( | ) 문자는 테이블 형식 표현식 문장에서 쿼리 Operator를 구분하는 것을 포함하여 두 가지 용도로 사용됩니다. 또한 대괄호 또는 둥근 괄호 내에서 논리적 OR Operator로 사용하여 파이프 문자로 구분된 항목 중 하나를 지정할 수 있음을 나타냅니다.

3.  쿼리 코드를 선택하고 실행하여 테이블에서 100개 행을 반환하세요.

   ![Screenshot of the KQL query editor.](./Images/kql-take-100-query.png)

    `project` Keyword를 사용하여 쿼리하려는 특정 Attribute를 추가하고, `take` Keyword를 사용하여 엔진에 반환할 레코드 수를 알려줌으로써 더 정확하게 지정할 수 있습니다.

4.  다음 쿼리를 입력, 선택 및 실행하세요:

    ```kql
    // Use 'project' and 'take' to view a sample number of records in the table and check the data.
    Bikestream
    | project Street, No_Bikes
    | take 10
    ```

    > **참고:** //의 사용은 Comment를 나타냅니다.

    분석에서 또 다른 일반적인 관행은 Queryset의 Column 이름을 변경하여 더 사용자 친화적으로 만드는 것입니다.

5.  다음 쿼리를 시도해 보세요:

    ```kql
    Bikestream
    | project Street, ["Number of Empty Docks"] = No_Empty_Docks
    | take 10
    ```

### KQL을 사용하여 데이터 요약

*summarize* Keyword를 Function과 함께 사용하여 데이터를 집계하고 조작할 수 있습니다.

1.  다음 쿼리를 시도해 보세요. 이 쿼리는 **sum** Function을 사용하여 대여 데이터를 요약하여 총 자전거 수를 확인합니다:

    ```kql

    Bikestream
    | summarize ["Total Number of Bikes"] = sum(No_Bikes)
    ```

    지정된 Column 또는 Expression별로 요약된 데이터를 그룹화할 수 있습니다.

2.  다음 쿼리를 실행하여 각 지역에서 사용 가능한 자전거 수를 확인하기 위해 지역별로 자전거 수를 그룹화하세요:

    ```kql
    Bikestream
    | summarize ["Total Number of Bikes"] = sum(No_Bikes) by Neighbourhood
    | project Neighbourhood, ["Total Number of Bikes"]
    ```

    어떤 자전거 지점에 Neighbourhood(지역)에 대한 null 또는 비어 있는 항목이 있는 경우, 요약 결과에는 비어 있는 값이 포함되며, 이는 분석에 결코 좋지 않습니다.

3.  여기에 표시된 대로 쿼리를 수정하여 *case* Function과 *isempty*, *isnull* Function을 함께 사용하여 Neighbourhood(지역)가 알려지지 않은 모든 Trip을 후속 조치를 위해 ***Unidentified*** Category로 그룹화하세요.

    ```kql
    Bikestream
    | summarize ["Total Number of Bikes"] = sum(No_Bikes) by Neighbourhood
    | project Neighbourhood = case(isempty(Neighbourhood) or isnull(Neighbourhood), "Unidentified", Neighbourhood), ["Total Number of Bikes"]
    ```

    >**참고**: 이 샘플 Dataset은 잘 관리되어 있으므로, 쿼리 결과에 Unidentified 필드가 없을 수도 있습니다.

### KQL을 사용하여 데이터 정렬

데이터를 더 잘 이해하기 위해 일반적으로 Column별로 데이터를 정렬하며, 이 과정은 KQL에서 *sort by* 또는 *order by* Operator를 사용하여 수행됩니다 (두 Operator는 동일하게 작동합니다).

1.  다음 쿼리를 시도해 보세요:

    ```kql
    Bikestream
    | summarize ["Total Number of Bikes"] = sum(No_Bikes) by Neighbourhood
    | project Neighbourhood = case(isempty(Neighbourhood) or isnull(Neighbourhood), "Unidentified", Neighbourhood), ["Total Number of Bikes"]
    | sort by Neighbourhood asc
    ```

2.  쿼리를 다음과 같이 수정하고 다시 실행하여, *order by* Operator가 *sort by*와 동일하게 작동함을 확인하세요:

    ```kql
    Bikestream
    | summarize ["Total Number of Bikes"] = sum(No_Bikes) by Neighbourhood
    | project Neighbourhood = case(isempty(Neighbourhood) or isnull(Neighbourhood), "Unidentified", Neighbourhood), ["Total Number of Bikes"]
    | order by Neighbourhood asc
    ```

### KQL을 사용하여 데이터 필터링

KQL에서 *where* Clause는 데이터를 필터링하는 데 사용됩니다. *and* 및 *or* 논리 Operator를 사용하여 *where* Clause의 조건을 결합할 수 있습니다.

1.  다음 쿼리를 실행하여 Chelsea 지역의 자전거 지점만 포함하도록 자전거 데이터를 필터링하세요:

    ```kql
    Bikestream
    | where Neighbourhood == "Chelsea"
    | summarize ["Total Number of Bikes"] = sum(No_Bikes) by Neighbourhood
    | project Neighbourhood = case(isempty(Neighbourhood) or isnull(Neighbourhood), "Unidentified", Neighbourhood), ["Total Number of Bikes"]
    | sort by Neighbourhood asc
    ```

## Transact-SQL을 사용하여 데이터 쿼리

KQL Database는 Transact-SQL을 기본적으로 지원하지 않지만, Microsoft SQL Server를 에뮬레이트하고 데이터를 T-SQL 쿼리할 수 있도록 하는 T-SQL Endpoint를 제공합니다. T-SQL Endpoint는 네이티브 SQL Server와 몇 가지 제한 및 차이점이 있습니다. 예를 들어, 테이블 생성, 변경 또는 삭제, 데이터 삽입, 업데이트 또는 삭제를 지원하지 않습니다. 또한 KQL과 호환되지 않는 일부 T-SQL Function 및 Syntax도 지원하지 않습니다. KQL을 지원하지 않는 시스템이 KQL Database 내의 데이터를 T-SQL로 쿼리할 수 있도록 하기 위해 생성되었습니다. 따라서 KQL Database의 기본 쿼리 언어로는 T-SQL보다 더 많은 기능과 성능을 제공하는 KQL을 사용하는 것이 좋습니다. KQL이 지원하는 count, sum, avg, min, max 등과 같은 일부 SQL Function도 사용할 수 있습니다.

### Transact-SQL을 사용하여 테이블에서 데이터 검색

1.  Queryset에서 다음 Transact-SQL 쿼리를 추가하고 실행하세요:

    ```sql
    SELECT TOP 100 * from Bikestream
    ```

2.  특정 Column을 검색하기 위해 쿼리를 다음과 같이 수정하세요:

    ```sql
    SELECT TOP 10 Street, No_Bikes
    FROM Bikestream
    ```

3.  **No_Empty_Docks**를 더 사용자 친화적인 이름으로 변경하는 Alias를 할당하도록 쿼리를 수정하세요.

    ```sql
    SELECT TOP 10 Street, No_Empty_Docks as [Number of Empty Docks]
    from Bikestream
    ```

### Transact-SQL을 사용하여 데이터 요약

1.  다음 쿼리를 실행하여 사용 가능한 총 자전거 수를 찾으세요:

    ```sql
    SELECT sum(No_Bikes) AS [Total Number of Bikes]
    FROM Bikestream
    ```

2.  쿼리를 수정하여 지역별로 총 자전거 수를 그룹화하세요:

    ```sql
    SELECT Neighbourhood, Sum(No_Bikes) AS [Total Number of Bikes]
    FROM Bikestream
    GROUP BY Neighbourhood
    ```

3.  쿼리를 추가로 수정하여 *CASE* Statement를 사용하여 출처를 알 수 없는 자전거 지점을 후속 조치를 위해 ***Unidentified*** Category로 그룹화하세요.

    ```sql
    SELECT CASE
             WHEN Neighbourhood IS NULL OR Neighbourhood = '' THEN 'Unidentified'
             ELSE Neighbourhood
           END AS Neighbourhood,
           SUM(No_Bikes) AS [Total Number of Bikes]
    FROM Bikestream
    GROUP BY CASE
               WHEN Neighbourhood IS NULL OR Neighbourhood = '' THEN 'Unidentified'
               ELSE Neighbourhood
             END;
    ```

### Transact-SQL을 사용하여 데이터 정렬

1.  다음 쿼리를 실행하여 그룹화된 결과를 지역별로 정렬하세요:

    ```sql
    SELECT CASE
             WHEN Neighbourhood IS NULL OR Neighbourhood = '' THEN 'Unidentified'
             ELSE Neighbourhood
           END AS Neighbourhood,
           SUM(No_Bikes) AS [Total Number of Bikes]
    FROM Bikestream
    GROUP BY CASE
               WHEN Neighbourhood IS NULL OR Neighbourhood = '' THEN 'Unidentified'
               ELSE Neighbourhood
             END
    ORDER BY Neighbourhood ASC;
    ```

### Transact-SQL을 사용하여 데이터 필터링

1.  다음 쿼리를 실행하여 그룹화된 데이터를 필터링하여 "Chelsea" 지역을 가진 행만 결과에 포함되도록 하세요:

    ```sql
    SELECT CASE
             WHEN Neighbourhood IS NULL OR Neighbourhood = '' THEN 'Unidentified'
             ELSE Neighbourhood
           END AS Neighbourhood,
           SUM(No_Bikes) AS [Total Number of Bikes]
    FROM Bikestream
    GROUP BY CASE
               WHEN Neighbourhood IS NULL OR Neighbourhood = '' THEN 'Unidentified'
               ELSE Neighbourhood
             END
    HAVING Neighbourhood = 'Chelsea'
    ORDER BY Neighbourhood ASC;
    ```

## 리소스 정리

이 실습에서는 Eventhouse를 생성하고 KQL 및 SQL을 사용하여 데이터를 쿼리했습니다.

KQL database 탐색을 마쳤으면, 이 실습을 위해 생성한 Workspace를 삭제할 수 있습니다.

1.  왼쪽 바에서 Workspace 아이콘을 선택하세요.
2.  툴바에서 **Workspace settings**를 선택하세요.
3.  **General** 섹션에서 **Remove this workspace**를 선택하세요.