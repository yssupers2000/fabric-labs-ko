---
lab:
    title: 'Monitor a data warehouse in Microsoft Fabric'
    module: 'Monitor a data warehouse in Microsoft Fabric'
---

# Microsoft Fabric에서 Data Warehouse 모니터링하기


Microsoft Fabric에서 Data Warehouse는 대규모 분석을 위한 관계형 데이터베이스를 제공합니다. Microsoft Fabric의 Data Warehouse는 활동 및 쿼리를 모니터링하는 데 사용할 수 있는 동적 관리 뷰(DMV)를 포함합니다.

이 랩은 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 생성하기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 작업 영역을 생성하세요.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://learn.microsoft.com/fabric/get-started/fabric-trial)로 이동하여 Fabric 자격 증명으로 로그인하세요.
1. 왼쪽 메뉴 바에서 **Workspaces**를 선택합니다(아이콘은 &#128455;와 유사하게 생겼습니다).
1. 원하는 이름으로 새 작업 영역을 생성하고, Fabric Capacity를 포함하는 라이선싱 모드(*Trial*, *Premium* 또는 *Fabric*)를 선택합니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## 샘플 Data Warehouse 생성하기

이제 작업 영역이 있으므로 Data Warehouse를 생성할 차례입니다.

1. 왼쪽 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Warehouse* 섹션 아래에서 **Sample warehouse**를 선택하고 **sample-dw**라는 이름으로 새 Data Warehouse를 생성하세요.

    > **참고**: **Create** 옵션이 사이드바에 고정되어 있지 않다면 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    1분 정도 지나면 새로운 warehouse가 생성되고 택시 탑승 분석 시나리오를 위한 샘플 데이터로 채워집니다.

    ![Screenshot of a new warehouse.](./Images/sample-data-warehouse.png)

## 동적 관리 뷰(DMV) 탐색하기

Microsoft Fabric Data Warehouse는 Data Warehouse 인스턴스에서 현재 활동을 식별하는 데 사용할 수 있는 동적 관리 뷰(DMV)를 포함합니다.

1. **sample-dw** Data Warehouse 페이지의 **New SQL query** 드롭다운 목록에서 **New SQL query**를 선택합니다.
1. 새 빈 쿼리 창에 다음 Transact-SQL 코드를 입력하여 **sys.dm_exec_connections** DMV를 쿼리하세요.

    ```sql
   SELECT * FROM sys.dm_exec_connections;
    ```

1. **&#9655; Run** 버튼을 사용하여 SQL 스크립트를 실행하고 Data Warehouse에 대한 모든 연결의 세부 정보를 포함하는 결과를 확인하세요.
1. 다음과 같이 SQL 코드를 수정하여 **sys.dm_exec_sessions** DMV를 쿼리하세요.

    ```sql
   SELECT * FROM sys.dm_exec_sessions;
    ```

1. 수정된 쿼리를 실행하고 모든 인증된 세션의 세부 정보를 보여주는 결과를 확인하세요.
1. 다음과 같이 SQL 코드를 수정하여 **sys.dm_exec_requests** DMV를 쿼리하세요.

    ```sql
   SELECT * FROM sys.dm_exec_requests;
    ```

1. 수정된 쿼리를 실행하고 Data Warehouse에서 실행 중인 모든 요청의 세부 정보를 보여주는 결과를 확인하세요.
1. 다음과 같이 SQL 코드를 수정하여 DMV를 조인하고 동일한 데이터베이스에서 현재 실행 중인 요청에 대한 정보를 반환하세요.

    ```sql
   SELECT connections.connection_id,
    sessions.session_id, sessions.login_name, sessions.login_time,
    requests.command, requests.start_time, requests.total_elapsed_time
   FROM sys.dm_exec_connections AS connections
   INNER JOIN sys.dm_exec_sessions AS sessions
       ON connections.session_id=sessions.session_id
   INNER JOIN sys.dm_exec_requests AS requests
       ON requests.session_id = sessions.session_id
   WHERE requests.status = 'running'
       AND requests.database_id = DB_ID()
   ORDER BY requests.total_elapsed_time DESC;
    ```

1. 수정된 쿼리를 실행하고 데이터베이스에서 실행 중인 모든 쿼리의 세부 정보(현재 쿼리 포함)를 보여주는 결과를 확인하세요.
1. **New SQL query** 드롭다운 목록에서 **New SQL query**를 선택하여 두 번째 쿼리 탭을 추가하세요. 그런 다음 새 빈 쿼리 탭에서 다음 코드를 실행하세요.

    ```sql
   WHILE 1 = 1
       SELECT * FROM Trip;
    ```

1. 쿼리를 계속 실행 상태로 두고, DMV를 쿼리하는 코드가 있는 탭으로 돌아가서 다시 실행하세요. 이번에는 결과에 다른 탭에서 실행 중인 두 번째 쿼리가 포함되어야 합니다. 해당 쿼리의 경과 시간을 기록해두세요.
1. 몇 초 기다린 다음 DMV를 쿼리하는 코드를 다시 실행하세요. 다른 탭의 쿼리에 대한 경과 시간이 증가해야 합니다.
1. 쿼리가 여전히 실행 중인 두 번째 쿼리 탭으로 돌아가서 **Cancel**을 선택하여 쿼리를 취소하세요.
1. DMV를 쿼리하는 코드가 있는 탭으로 돌아가서 쿼리를 다시 실행하여 두 번째 쿼리가 더 이상 실행되지 않는지 확인하세요.
1. 모든 쿼리 탭을 닫으세요.

> **추가 정보**: DMV 사용에 대한 자세한 내용은 Microsoft Fabric 설명서의 [DMV를 사용하여 연결, 세션 및 요청 모니터링](https://learn.microsoft.com/fabric/data-warehouse/monitor-using-dmv)을 참조하세요.

## Query insights 탐색하기

Microsoft Fabric Data Warehouse는 Data Warehouse에서 실행 중인 쿼리에 대한 세부 정보를 제공하는 특별한 뷰(view) 집합인 *Query insights*를 제공합니다.

1. **sample-dw** Data Warehouse 페이지의 **New SQL query** 드롭다운 목록에서 **New SQL query**를 선택합니다.
1. 새 빈 쿼리 창에 다음 Transact-SQL 코드를 입력하여 **exec_requests_history** 뷰(view)를 쿼리하세요.

    ```sql
   SELECT * FROM queryinsights.exec_requests_history;
    ```

1. **&#9655; Run** 버튼을 사용하여 SQL 스크립트를 실행하고 이전에 실행된 쿼리의 세부 정보를 포함하는 결과를 확인하세요.
1. 다음과 같이 SQL 코드를 수정하여 **frequently_run_queries** 뷰(view)를 쿼리하세요.

    ```sql
   SELECT * FROM queryinsights.frequently_run_queries;
    ```

1. 수정된 쿼리를 실행하고 자주 실행되는 쿼리의 세부 정보를 보여주는 결과를 확인하세요.
1. 다음과 같이 SQL 코드를 수정하여 **long_running_queries** 뷰(view)를 쿼리하세요.

    ```sql
   SELECT * FROM queryinsights.long_running_queries;
    ```

1. 수정된 쿼리를 실행하고 모든 쿼리 및 해당 지속 시간의 세부 정보를 보여주는 결과를 확인하세요.

> **추가 정보**: Query insights 사용에 대한 자세한 내용은 Microsoft Fabric 설명서의 [Fabric Data Warehousing의 Query insights](https://learn.microsoft.com/fabric/data-warehouse/query-insights)를 참조하세요.

## 리소스 정리하기

이 실습에서는 Microsoft Fabric Data Warehouse의 활동을 모니터링하기 위해 동적 관리 뷰(DMV)와 Query insights를 사용했습니다.

Data Warehouse 탐색을 마쳤다면 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인하세요.
1. **Workspace settings**를 선택하고 **General** 섹션에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.
1. **Delete**를 선택하여 작업 영역을 삭제합니다.