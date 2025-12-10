---
lab:
    title: 'Secure data in a data warehouse'
    module: 'Secure data in a data warehouse'
--- 

# Data Warehouse에서 데이터 보안

Microsoft Fabric 권한과 세분화된 SQL 권한은 함께 작동하여 Warehouse 접근 및 사용자 권한을 관리합니다. 이 실습에서는 세분화된 권한, 열 수준 보안(Column-level security), 행 수준 보안(Row-level security), 동적 데이터 마스킹(Dynamic data masking)을 사용하여 데이터를 보호합니다.

> **참고**: 이 실습에는 결과 유효성 검사를 위해 두 번째 사용자 계정이 필요한 선택적 단계가 있습니다. 한 사용자에게는 Workspace Admin 역할을 할당하고, 다른 사용자에게는 Workspace Viewer 역할을 할당해야 합니다. 작업 영역에 역할을 할당하려면 [작업 영역에 액세스 권한 부여](https://learn.microsoft.com/fabric/get-started/give-access-workspaces)를 참조하세요. 동일한 조직 내에 두 번째 계정에 접근할 수 없는 경우, Workspace Admin으로 실습을 진행할 수 있으며, 실습의 **스크린샷**을 참고하여 Workspace Viewer 계정이 어떤 항목에 접근할 수 있는지 확인할 수 있습니다.

이 랩을 완료하는 데 약 **45**분이 소요됩니다.

## Workspace 생성

Fabric에서 데이터를 사용하기 전에 Fabric 평가판이 활성화된 Workspace를 생성하세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 유사합니다)를 선택합니다.
3.  원하는 이름으로 새 Workspace를 생성하고, Fabric Capacity(*Trial*, *Premium* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
4.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Fabric의 빈 Workspace 스크린샷.](./Images/new-empty-workspace.png)

> **참고**: Workspace를 생성하면 자동으로 Workspace Admin 역할의 멤버가 됩니다. 이 실습에서 구성된 기능을 테스트하기 위해 환경의 두 번째 사용자를 Workspace Viewer 역할에 추가할 수 있습니다. 이는 Workspace 내에서 **접근 관리**를 선택한 다음 **사용자 또는 그룹 추가**를 통해 수행할 수 있습니다. 이렇게 하면 두 번째 사용자가 Workspace 콘텐츠를 볼 수 있습니다.

## Data Warehouse 생성

다음으로, 생성한 Workspace에 Data Warehouse를 생성합니다.

1.  왼쪽 메뉴 바에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *Data Warehouse* 섹션에서 **Warehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    잠시 후 새 Warehouse가 생성됩니다.

    ![새 Warehouse 스크린샷.](./Images/new-empty-data-warehouse.png)

## 테이블의 Column에 동적 데이터 마스킹 규칙 적용

동적 데이터 마스킹 규칙은 테이블 수준의 개별 Column에 적용되므로 모든 Query가 마스킹의 영향을 받습니다. 기밀 데이터를 볼 명시적인 권한이 없는 사용자는 Query 결과에서 마스킹된 값을 보게 되며, 데이터를 볼 명시적인 권한이 있는 사용자는 마스킹되지 않은 데이터를 봅니다. 마스크에는 기본(default), 이메일(email), 무작위(random), 사용자 지정 문자열(custom string)의 네 가지 유형이 있습니다. 이 실습에서는 기본 마스크, 이메일 마스크, 사용자 지정 문자열 마스크를 적용합니다.

1.  Warehouse에서 **T-SQL** 타일을 선택하고, 다음 T-SQL 문을 사용하여 테이블을 생성하고 데이터를 삽입하고 조회합니다.

    ```T-SQL
   CREATE TABLE dbo.Customers
   (   
       CustomerID INT NOT NULL,   
       FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
       LastName varchar(50) NOT NULL,     
       Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
       Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
   );
   
   INSERT dbo.Customers (CustomerID, FirstName, LastName, Phone, Email) VALUES
   (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
   (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
   (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
   
   SELECT * FROM dbo.Customers;
    ```

    마스크 해제된 데이터를 볼 수 없는 사용자가 테이블을 Query할 때, **FirstName** Column은 문자열의 첫 글자와 XXXXXXX를 표시하고 마지막 문자는 표시하지 않습니다. **Phone** Column은 xxxx를 표시합니다. **Email** Column은 이메일 주소의 첫 글자 다음에 `XXX@XXX.com`을 표시합니다. 이 접근 방식은 민감한 데이터가 기밀을 유지하도록 보장하면서도 제한된 사용자가 테이블을 Query할 수 있도록 합니다.

2.  **&#9655; 실행** 버튼을 사용하여 SQL 스크립트를 실행합니다. 이 스크립트는 Data Warehouse의 **dbo** Schema에 **Customers**라는 새 테이블을 생성합니다.

3.  그런 다음, **탐색기** 창에서 **Schemas** > **dbo** > **Tables**를 확장하고 **Customers** 테이블이 생성되었는지 확인합니다. Workspace 생성자로서 마스크 해제된 데이터를 볼 수 있는 Workspace Admin 역할의 멤버이므로, `SELECT` 문은 마스크 해제된 데이터를 반환합니다.

    >**참고**: **Viewer** Workspace 역할의 멤버인 테스트 사용자로 연결하여 **Customers** 테이블에서 `SELECT` 문을 실행하면 마스킹된 데이터에 대해 다음 결과를 볼 수 있습니다.

    ![마스킹된 데이터가 있는 Customers 테이블 스크린샷.](./Images/masked-table.png)

    테스트 사용자에게 UNMASK 권한이 부여되지 않았습니다. 따라서 `CREATE TABLE` 문에서 해당 Column들이 마스크와 함께 정의되었기 때문에 FirstName, Phone, Email Column에 대해 반환된 데이터는 마스킹됩니다.

## 행 수준 보안(Row-level security) 적용

행 수준 보안(RLS)은 Query를 실행하는 사용자의 ID 또는 역할에 따라 행에 대한 접근을 제한하는 데 사용될 수 있습니다. 이 실습에서는 보안 정책과 인라인 테이블 값 함수(inline table-valued function)로 정의된 보안 Predicate를 생성하여 행 접근을 제한합니다.

1.  이전 실습에서 생성한 Warehouse에서 **새 SQL Query** 드롭다운을 선택하고 **새 SQL Query**를 선택합니다.

2.  테이블을 생성하고 데이터를 삽입합니다. 이후 단계에서 행 수준 보안을 구현하기 위해 `<username1>@<your_domain>.com`을 가상의 사용자 이름 또는 환경의 실제 사용자 이름(**Viewer** 역할)으로 대체하고, `<username2>@<your_domain>.com`을 본인의 사용자 이름(**Admin** 역할)으로 대체합니다.

    ```T-SQL
   CREATE TABLE dbo.Sales  
   (  
       OrderID INT,  
       SalesRep VARCHAR(60),  
       Product VARCHAR(10),  
       Quantity INT  
   );
    
   --Populate the table with 6 rows of data, showing 3 orders for each test user. 
   INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
   (1, '<username1>@<your_domain>.com', 'Valve', 5),   
   (2, '<username1>@<your_domain>.com', 'Wheel', 2),   
   (3, '<username1>@<your_domain>.com', 'Valve', 4),  
   (4, '<username2>@<your_domain>.com', 'Bracket', 2),   
   (5, '<username2>@<your_domain>.com', 'Wheel', 5),   
   (6, '<username2>@<your_domain>.com', 'Seat', 5);  
    
   SELECT * FROM dbo.Sales;  
    ```

3.  **&#9655; 실행** 버튼을 사용하여 SQL 스크립트를 실행합니다. 이 스크립트는 Data Warehouse의 **dbo** Schema에 **Sales**라는 새 테이블을 생성합니다.

4.  그런 다음, **탐색기** 창에서 **Schemas** > **dbo** > **Tables**를 확장하고 **Sales** 테이블이 생성되었는지 확인합니다.
5.  새 Schema, Function으로 정의된 보안 Predicate, 그리고 보안 정책을 생성합니다.

    ```T-SQL
   --Create a separate schema to hold the row-level security objects (the predicate function and the security policy)
   CREATE SCHEMA rls;
   GO
   
   /*Create the security predicate defined as an inline table-valued function.
   A predicate evaluates to true (1) or false (0). This security predicate returns 1,
   meaning a row is accessible, when a row in the SalesRep column is the same as the user
   executing the query.*/   
   --Create a function to evaluate who is querying the table
   CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
       RETURNS TABLE  
   WITH SCHEMABINDING  
   AS  
       RETURN SELECT 1 AS fn_securitypredicate_result   
   WHERE @SalesRep = USER_NAME();
   GO   
   /*Create a security policy to invoke and enforce the function each time a query is run on the Sales table.
   The security policy has a filter predicate that silently filters the rows available to 
   read operations (SELECT, UPDATE, and DELETE). */
   CREATE SECURITY POLICY SalesFilter  
   ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
   ON dbo.Sales  
   WITH (STATE = ON);
   GO
    ```

6.  **&#9655; 실행** 버튼을 사용하여 SQL 스크립트를 실행합니다.
7.  그런 다음, **탐색기** 창에서 **Schemas** > **rls** > **Functions** > **Table-valued Functions**를 확장하고 Function이 생성되었는지 확인합니다.

    > **참고**: `<username1>@<your_domain>.com`을 대체한 사용자로 연결하여 **Sales** 테이블에서 `SELECT` 문을 실행하면 행 수준 보안에 대해 다음 결과를 볼 수 있습니다.

    ![RLS가 적용된 Sales 테이블 스크린샷.](./Images/rls-table.png)

## 열 수준 보안(Column-level security) 구현

열 수준 보안은 어떤 사용자가 테이블의 특정 Column에 접근할 수 있는지 지정할 수 있도록 합니다. 이는 Column 목록과 Column을 읽을 수 있거나 없는 사용자 또는 역할을 지정하여 테이블에 `GRANT` 또는 `DENY` 문을 발행함으로써 구현됩니다. 접근 관리를 간소화하기 위해 개별 사용자 대신 역할에 권한을 할당합니다. 이 실습에서는 테이블을 생성하고, 테이블의 Column 하위 집합에 접근 권한을 부여하며, 제한된 Column이 본인 외의 사용자에게는 보이지 않는지 테스트합니다.

1.  이전 실습에서 생성한 Warehouse에서 **새 SQL Query** 드롭다운을 선택한 다음 **새 SQL Query**를 선택합니다.

2.  테이블을 생성하고 데이터를 삽입합니다.

    ```T-SQL
   CREATE TABLE dbo.Orders
   (   
       OrderID INT,   
       CustomerID INT,  
       CreditCard VARCHAR(20)      
   );   
   INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
   (1234, 5678, '111111111111111'),
   (2341, 6785, '222222222222222'),
   (3412, 7856, '333333333333333');   
   SELECT * FROM dbo.Orders;
    ```

3.  테이블의 Column을 볼 수 있는 권한을 거부합니다. T-SQL 문은 `<username1>@<your_domain>.com`이 Orders 테이블의 CreditCard Column을 보지 못하도록 합니다. `DENY` 문에서 `<username1>@<your_domain>.com`을 Workspace에 Viewer 권한을 가진 사용자의 사용자 이름으로 대체합니다.

    ```T-SQL
   DENY SELECT ON dbo.Orders (CreditCard) TO [<username1>@<your_domain>.com];
    ```

    > **참고**: `<username1>@<your_domain>.com`을 대체한 사용자로 연결하여 **Orders** 테이블에서 `SELECT` 문을 실행하면 열 수준 보안에 대해 다음 결과를 볼 수 있습니다.

    ![오류가 있는 Orders 테이블 Query 스크린샷.](./Images/cls-table.png)

    스크린샷에 표시된 오류는 `CreditCard` Column에 대한 접근이 제한되었기 때문에 발생합니다. `OrderID` 및 `CustomerID` Column만 선택하면 Query가 성공적으로 실행됩니다.

## T-SQL을 사용하여 SQL 세분화된 권한 구성

Fabric은 Workspace 수준 및 항목 수준에서 데이터 접근을 제어할 수 있는 권한 모델을 가지고 있습니다. Fabric Warehouse의 보안 개체(securables)를 사용자들이 무엇을 할 수 있는지 더 세밀하게 제어해야 할 때, 표준 SQL 데이터 제어 언어(DCL) 명령어인 `GRANT`, `DENY`, `REVOKE`를 사용할 수 있습니다. 이 실습에서는 객체(objects)를 생성하고, `GRANT` 및 `DENY`를 사용하여 객체를 보호한 다음, Query를 실행하여 세분화된 권한 적용의 효과를 확인합니다.

1.  이전 실습에서 생성한 Warehouse에서 **새 SQL Query** 드롭다운을 선택합니다. **새 SQL Query**를 선택합니다.

2.  저장 프로시저(Stored Procedure)와 테이블을 생성합니다. 그런 다음 프로시저를 실행하고 테이블을 Query합니다.

    ```T-SQL
   CREATE PROCEDURE dbo.sp_PrintMessage
   AS
   PRINT 'Hello World.';
   GO   
   CREATE TABLE dbo.Parts
   (
       PartID INT,
       PartName VARCHAR(25)
   );
   
   INSERT dbo.Parts (PartID, PartName) VALUES
   (1234, 'Wheel'),
   (5678, 'Seat');
    GO
   
   /*Execute the stored procedure and select from the table and note the results you get
   as a member of the Workspace Admin role. Look for output from the stored procedure on 
   the 'Messages' tab.*/
   EXEC dbo.sp_PrintMessage;
   GO   
   SELECT * FROM dbo.Parts
    ```

3.  다음으로, **Workspace Viewer** 역할의 멤버인 사용자에게 테이블에 대한 `DENY SELECT` 권한을 부여하고, 동일한 사용자에게 프로시저에 대한 `GRANT EXECUTE` 권한을 부여합니다. `<username1>@<your_domain>.com`을 Workspace에 **Viewer** 권한을 가진 사용자의 사용자 이름으로 대체합니다.

    ```T-SQL
   DENY SELECT on dbo.Parts to [<username1>@<your_domain>.com];

   GRANT EXECUTE on dbo.sp_PrintMessage to [<username1>@<your_domain>.com];
    ```

    > **참고**: `<username1>@<your_domain>.com`을 대체한 사용자로 연결하여 저장 프로시저를 실행하고 **Parts** 테이블에서 `SELECT` 문을 실행하면 세분화된 권한에 대해 다음 결과를 볼 수 있습니다.

    ![오류가 있는 Parts 테이블 Query 스크린샷.](./Images/grant-deny-table.png)

## 리소스 정리

이 실습에서는 테이블의 Column에 동적 데이터 마스킹 규칙을 적용하고, 행 수준 보안을 적용했으며, 열 수준 보안을 구현하고 T-SQL을 사용하여 SQL 세분화된 권한을 구성했습니다.

1.  왼쪽 탐색 바에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2.  상단 도구 모음 메뉴에서 **Workspace 설정**을 선택합니다.
3.  **일반** 섹션에서 **이 Workspace 제거**를 선택합니다.