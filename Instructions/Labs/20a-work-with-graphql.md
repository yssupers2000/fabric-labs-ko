---
lab:
    title: 'Work with API for GraphQL in Microsoft Fabric'
    module: 'Get started with GraphQL in Microsoft Fabric'
---

# Microsoft Fabric에서 API for GraphQL 사용

Microsoft Fabric의 API for GraphQL은 널리 채택되고 친숙한 API 기술을 사용하여 여러 데이터 원본에 대해 빠르고 효율적인 쿼리를 가능하게 하는 데이터 액세스 계층입니다. 이 API를 사용하면 백엔드 데이터 원본의 세부 정보를 추상화하여 애플리케이션 논리에 집중하고, 클라이언트가 필요로 하는 모든 데이터를 단일 호출로 제공할 수 있습니다. GraphQL은 간단한 쿼리 언어와 쉽게 조작할 수 있는 결과 집합을 사용하여 Fabric에서 애플리케이션이 데이터에 액세스하는 데 걸리는 시간을 최소화합니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

Fabric에서 데이터를 사용하기 전에 Fabric 평가판이 활성화된 작업 영역을 만드세요.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2. 왼쪽 메뉴 모음에서 **새 작업 영역(New workspace)**을 선택합니다.
3. 원하는 이름으로 새 작업 영역을 만들고, Fabric Capacity(*Trial*, *Premium* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
4. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## 샘플 데이터가 있는 SQL 데이터베이스 만들기

작업 영역을 만들었으므로 이제 SQL 데이터베이스를 만들 차례입니다.

1. 왼쪽 메뉴 모음에서 **만들기(Create)**를 선택합니다. *새로 만들기(New)* 페이지의 *데이터베이스(Databases)* 섹션에서 **SQL 데이터베이스(SQL database)**를 선택합니다.

    >**참고**: **만들기(Create)** 옵션이 사이드바에 고정되어 있지 않다면, 먼저 말줄임표(**...**) 옵션을 선택해야 합니다.

2. 데이터베이스 이름으로 **AdventureWorksLT**를 입력하고 **만들기(Create)**를 선택합니다.
3. 데이터베이스를 만든 후 **샘플 데이터(Sample data)** 카드에서 데이터베이스에 샘플 데이터를 로드할 수 있습니다.

    1분 정도 후에 데이터베이스는 시나리오에 맞는 샘플 데이터로 채워질 것입니다.

    ![Screenshot of a new database loaded with sample data.](./Images/sql-database-sample.png)

## SQL 데이터베이스 쿼리

SQL 쿼리 편집기는 IntelliSense, 코드 완성, 구문 강조 표시, 클라이언트 측 구문 분석 및 유효성 검사 기능을 지원합니다. DDL(데이터 정의 언어), DML(데이터 조작 언어) 및 DCL(데이터 제어 언어) 문을 실행할 수 있습니다.

1. **AdventureWorksLT** 데이터베이스 페이지에서 **홈(Home)**으로 이동한 다음, **새 쿼리(New query)**를 선택합니다.
2. 새 빈 쿼리 창에 다음 T-SQL 코드를 입력하고 실행합니다.

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
    
    이 쿼리는 `Product` 및 `ProductCategory` 테이블을 조인하여 제품 이름, 해당 카테고리 및 정가(ListPrice)를 표시하고, 가격을 내림차순으로 정렬합니다.

3. 모든 쿼리 탭을 닫습니다.

## API for GraphQL 만들기

먼저, 판매 주문 데이터를 노출하기 위한 GraphQL 엔드포인트를 설정합니다. 이 엔드포인트를 사용하면 날짜, 고객, 제품과 같은 다양한 매개변수를 기반으로 판매 주문을 쿼리할 수 있습니다.

1. Fabric 포털에서 작업 영역으로 이동하여 **+ 새 항목(New item)**을 선택합니다.
2. **데이터 개발(Develop data)** 섹션으로 이동하여 **API for GraphQL**을 선택합니다.
3. 이름을 제공하고 **만들기(Create)**를 선택합니다.
4. API for GraphQL의 기본 페이지에서 **데이터 원본 선택(Select data source)**을 선택합니다.
5. 연결 옵션을 선택하라는 메시지가 표시되면 **SSO(Single Sign-On) 인증으로 Fabric 데이터 원본에 연결(Connect to Fabric data sources with single sign-on(SSO) authentication)**을 선택합니다.
6. **연결할 데이터 선택(Choose the data you want to connect)** 페이지에서 이전에 생성한 `AdventureWorksLT` 데이터베이스를 선택합니다.
7. **연결(Connect)**을 선택합니다.
8. **데이터 선택(Choose data)** 페이지에서 `SalesLT.Product` 테이블을 선택합니다. 
9. 데이터를 미리 보고 **로드(Load)**를 선택합니다.
10. **엔드포인트 복사(Copy endpoint)**를 선택하고 공개 URL 링크를 기록해 둡니다. 지금은 필요 없지만, 이곳에서 API 주소를 복사할 수 있습니다.

## Mutations 비활성화

API가 생성되었으므로, 이 시나리오에서는 읽기 작업만 가능하도록 판매 데이터를 노출하려고 합니다.

1. API for GraphQL의 **스키마 탐색기(Schema explorer)**에서 **Mutations**를 확장합니다.
2. 각 mutation 옆에 있는 말줄임표(**...**)를 선택하고 **비활성화(Disable)**를 선택합니다.

이렇게 하면 API를 통한 데이터 수정 또는 업데이트가 방지됩니다. 즉, 데이터는 읽기 전용이 되며, 사용자는 데이터를 보거나 쿼리할 수만 있고 변경할 수는 없습니다.

## GraphQL을 사용하여 데이터 쿼리

이제 GraphQL을 사용하여 이름이 "HL Road Frame."으로 시작하는 모든 제품을 찾아 데이터를 쿼리해 보겠습니다.

1. GraphQL 쿼리 편집기에 다음 쿼리를 입력하고 실행합니다.

```json
query {
  products(filter: { Name: { startsWith: "HL Road Frame" } }) {
    items {
      ProductModelID
      Name
      ListPrice
      Color
      Size
      ModifiedDate
    }
  }
}
```

이 쿼리에서 `products`는 기본 유형이며, `ProductModelID`, `Name`, `ListPrice`, `Color`, `Size`, `ModifiedDate` 필드를 포함합니다. 이 쿼리는 이름이 "HL Road Frame."으로 시작하는 제품 목록을 반환할 것입니다.

> **추가 정보**: 플랫폼에서 사용 가능한 다른 구성 요소에 대해 자세히 알아보려면 Microsoft Fabric 설명서의 [Microsoft Fabric API for GraphQL이란 무엇인가요?](https://learn.microsoft.com/fabric/data-engineering/api-graphql-overview)를 참조하세요.

이 실습에서는 Microsoft Fabric에서 GraphQL을 사용하여 SQL 데이터베이스의 데이터를 생성, 쿼리 및 노출했습니다.

## 리소스 정리

데이터베이스 탐색을 마쳤다면 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2. 도구 모음의 말줄임표(**...**) 메뉴에서 **작업 영역 설정(Workspace settings)**을 선택합니다.
3. **일반(General)** 섹션에서 **이 작업 영역 제거(Remove this workspace)**를 선택합니다.
