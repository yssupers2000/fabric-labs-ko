---
lab:
    title: 'Work with SQL Database in Microsoft Fabric'
    module: 'Get started with SQL Database in Microsoft Fabric'
---

# Work with SQL Database in Microsoft Fabric

Microsoft Fabric의 SQL Database는 Azure SQL Database를 기반으로 하는 개발자 친화적인 트랜잭션 데이터베이스로, Fabric에서 운영 데이터베이스를 쉽게 생성할 수 있도록 해줍니다. Fabric의 SQL Database는 Azure SQL Database와 동일한 SQL Database 엔진을 사용합니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 생성

Fabric에서 데이터를 사용하기 전에 Fabric 평가판이 활성화된 작업 영역을 생성합니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
1. 왼쪽의 메뉴 모음에서 **새 작업 영역**을 선택합니다.
1. 원하는 이름으로 새 작업 영역을 생성하고, Fabric Capacity(*평가판*, *Premium* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## 샘플 데이터가 포함된 SQL database 생성

작업 영역을 생성했으므로 이제 SQL database를 생성할 차례입니다.

1. 왼쪽의 메뉴 모음에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *데이터베이스* 섹션 아래에서 **SQL database**를 선택합니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않은 경우 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

1. 데이터베이스 이름으로 **AdventureWorksLT**를 입력하고 **만들기**를 선택합니다.
1. 데이터베이스를 생성한 후 **샘플 데이터** 카드에서 데이터베이스에 샘플 데이터를 로드할 수 있습니다.

    약 1분 정도 지나면 데이터베이스가 시나리오에 맞는 샘플 데이터로 채워집니다.

    ![Screenshot of a new database loaded with sample data.](./Images/sql-database-sample.png)

## SQL database 쿼리

SQL 쿼리 편집기는 IntelliSense, 코드 완성(code completion), 구문 강조 표시(syntax highlighting), 클라이언트 측 구문 분석(client-side parsing) 및 유효성 검사(validation)를 지원합니다. Data Definition Language (DDL), Data Manipulation Language (DML) 및 Data Control Language (DCL) 문을 실행할 수 있습니다.

1. **AdventureWorksLT** 데이터베이스 페이지에서 **홈**으로 이동하여 **새 쿼리**를 선택합니다.

1. 새로운 빈 쿼리 창에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    SELECT 
        p.Name AS ProductName,
        pc.Name AS CategoryName,
        p.ListPrice
    FROM 
        SalesLT.Product p
    INNER JOIN 
        SalesLT.ProductCategory pc ON p.ProductCategoryID = pc.ProductCategoryID
    ORDER BY 
    p.ListPrice DESC;
    ```
    
    이 쿼리는 `Product` 및 `ProductCategory` 테이블을 조인하여 제품 이름, 카테고리 및 정가를 가격 내림차순으로 정렬하여 표시합니다.

1. 새 쿼리 편집기에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
   SELECT 
        c.FirstName,
        c.LastName,
        soh.OrderDate,
        soh.SubTotal
    FROM 
        SalesLT.Customer c
    INNER JOIN 
        SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
    ORDER BY 
        soh.OrderDate DESC;
    ```

    이 쿼리는 고객 목록과 해당 주문 날짜 및 소계(subtotal)를 주문 날짜 내림차순으로 정렬하여 검색합니다. 

1. 모든 쿼리 탭을 닫습니다.

## 외부 데이터 원본과 데이터 통합

공휴일에 대한 외부 데이터를 판매 주문과 통합할 것입니다. 그런 다음, 공휴일과 겹치는 판매 주문을 식별하여 공휴일이 판매 활동에 어떤 영향을 미치는지에 대한 통찰력을 제공합니다.

1. **홈**으로 이동하여 **새 쿼리**를 선택합니다.

1. 새로운 빈 쿼리 창에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    CREATE TABLE SalesLT.PublicHolidays (
        CountryOrRegion NVARCHAR(50),
        HolidayName NVARCHAR(100),
        Date DATE,
        IsPaidTimeOff BIT
    );
    ```

    이 쿼리는 다음 단계를 위해 `SalesLT.PublicHolidays` 테이블을 생성합니다.

1. 새 쿼리 편집기에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    INSERT INTO SalesLT.PublicHolidays (CountryOrRegion, HolidayName, Date, IsPaidTimeOff)
    VALUES
        ('Canada', 'Victoria Day', '2024-02-19', 1),
        ('United Kingdom', 'Christmas Day', '2024-12-25', 1),
        ('United Kingdom', 'Spring Bank Holiday', '2024-05-27', 1),
        ('United States', 'Thanksgiving Day', '2024-11-28', 1);
    ```
    
    이 예시에서 이 쿼리는 2024년 캐나다, 영국, 미국 공휴일을 `SalesLT.PublicHolidays` 테이블에 삽입합니다.    

1. 새로운 또는 기존 쿼리 편집기에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    -- Insert new addresses into SalesLT.Address
    INSERT INTO SalesLT.Address (AddressLine1, City, StateProvince, CountryRegion, PostalCode, rowguid, ModifiedDate)
    VALUES
        ('123 Main St', 'Seattle', 'WA', 'United States', '98101', NEWID(), GETDATE()),
        ('456 Maple Ave', 'Toronto', 'ON', 'Canada', 'M5H 2N2', NEWID(), GETDATE()),
        ('789 Oak St', 'London', 'England', 'United Kingdom', 'EC1A 1BB', NEWID(), GETDATE());
    
    -- Insert new orders into SalesOrderHeader
    INSERT INTO SalesLT.SalesOrderHeader (
        SalesOrderID, RevisionNumber, OrderDate, DueDate, ShipDate, Status, OnlineOrderFlag, 
        PurchaseOrderNumber, AccountNumber, CustomerID, ShipToAddressID, BillToAddressID, 
        ShipMethod, CreditCardApprovalCode, SubTotal, TaxAmt, Freight, Comment, rowguid, ModifiedDate
    )
    VALUES
        (1001, 1, '2024-12-25', '2024-12-30', '2024-12-26', 1, 1, 'PO12345', 'AN123', 1, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), 'Ground', '12345', 100.00, 10.00, 5.00, 'New Order 1', NEWID(), GETDATE()),
        (1002, 1, '2024-11-28', '2024-12-03', '2024-11-29', 1, 1, 'PO67890', 'AN456', 2, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), 'Air', '67890', 200.00, 20.00, 10.00, 'New Order 2', NEWID(), GETDATE()),
        (1003, 1, '2024-02-19', '2024-02-24', '2024-02-20', 1, 1, 'PO54321', 'AN789', 3, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Sea', '54321', 300.00, 30.00, 15.00, 'New Order 3', NEWID(), GETDATE()),
        (1004, 1, '2024-05-27', '2024-06-01', '2024-05-28', 1, 1, 'PO98765', 'AN321', 4, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Ground', '98765', 400.00, 40.00, 20.00, 'New Order 4', NEWID(), GETDATE());
    ```

    이 코드는 데이터베이스에 새로운 주소와 주문을 추가하여 여러 국가의 가상 주문을 시뮬레이션합니다.

1. 새로운 또는 기존 쿼리 편집기에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    ```

    잠시 시간을 내어 결과를 확인하고, 쿼리가 각 국가의 공휴일과 겹치는 판매 주문을 어떻게 식별하는지 주목합니다. 이는 주문 패턴과 공휴일이 판매 활동에 미칠 수 있는 잠재적 영향에 대한 귀중한 통찰력을 제공할 수 있습니다.

1. 모든 쿼리 탭을 닫습니다.

## 데이터 보안

특정 사용자 그룹이 보고서 생성을 위해 미국 데이터에만 액세스해야 한다고 가정해 봅시다.

이전에 사용했던 쿼리를 기반으로 View를 생성하고 필터를 추가해 보겠습니다.

1. 새로운 빈 쿼리 창에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    CREATE VIEW SalesLT.vw_SalesOrderHoliday AS
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    WHERE a.CountryRegion = 'United Kingdom';
    ```

1. 새로운 또는 기존 쿼리 편집기에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    -- Create the role
    CREATE ROLE SalesOrderRole;
    
    -- Grant select permission on the view to the role
    GRANT SELECT ON SalesLT.vw_SalesOrderHoliday TO SalesOrderRole;
    ```

    `SalesOrderRole` Role에 멤버로 추가된 모든 사용자는 필터링된 View에만 액세스할 수 있습니다. 이 Role의 사용자가 다른 사용자 개체에 액세스하려고 하면 다음과 유사한 오류 메시지를 받게 됩니다.

    ```
    Msg 229, Level 14, State 5, Line 1
    The SELECT permission was denied on the object 'ObjectName', database 'DatabaseName', schema 'SchemaName'.
    ```

> **추가 정보**: 플랫폼에서 사용할 수 있는 다른 구성 요소에 대해 자세히 알아보려면 Microsoft Fabric 설명서의 [Microsoft Fabric이란 무엇입니까?](https://learn.microsoft.com/fabric/get-started/microsoft-fabric-overview)를 참조하세요.

이 실습에서는 Microsoft Fabric의 SQL database에서 데이터를 생성하고, 쿼리하고, 보안을 설정했습니다.

## 리소스 정리

데이터베이스 탐색을 마쳤다면 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
