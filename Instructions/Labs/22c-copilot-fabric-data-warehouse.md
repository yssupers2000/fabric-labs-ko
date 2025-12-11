---
lab:
    title: 'Use Copilot in Microsoft Fabric data warehouse'
    module: 'Get started with Copilot in Fabric for Data Warehouse'
---

# Use Copilot in Microsoft Fabric Data Warehouse

In Microsoft Fabric, a Data Warehouse provides a relational database for large-scale analytics. Unlike the default read-only SQL Endpoint for tables defined in a Lakehouse, a Data Warehouse provides full SQL semantics; including the ability to insert, update, and delete data in the tables. In this lab, 우리는 SQL 쿼리 생성을 위해 Copilot을 활용하는 방법을 알아봅니다.

이 실습은 완료하는 데 약 **30**분이 소요됩니다.

## 학습 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Microsoft Fabric에서 Data Warehouse의 역할을 이해합니다.
- Fabric에서 작업 영역과 Data Warehouse를 생성하고 구성합니다.
- SQL을 사용하여 샘플 데이터를 로드하고 탐색합니다.
- Copilot을 사용하여 자연어 프롬프트에서 SQL 쿼리를 생성, 개선 및 문제 해결합니다.
- AI 지원 SQL 생성을 통해 뷰를 생성하고 고급 데이터 분석을 수행합니다.
- 데이터 탐색 및 분석 작업을 가속화하기 위해 Copilot의 기능을 적용합니다.

## 시작하기 전에

이 실습을 완료하려면 Copilot이 활성화된 [Microsoft Fabric Capacity (F2 이상)](https://learn.microsoft.com/fabric/fundamentals/copilot-enable-fabric)가 필요합니다.

## 실습 시나리오

이 실습에서 여러분은 Microsoft Fabric을 사용하여 판매 실적을 더 잘 이해하고자 하는 소매 회사의 데이터 분석가입니다. 여러분의 팀은 최근 Fabric의 Data Warehouse 기능을 도입했으며, 데이터 탐색 및 보고를 가속화하기 위해 Copilot을 활용하는 데 관심을 가지고 있습니다. 새로운 Data Warehouse를 생성하고, 샘플 소매 판매 데이터를 로드하고, Copilot을 사용하여 SQL 쿼리를 생성 및 개선할 것입니다. 랩을 마칠 때쯤에는 Fabric 환경 내에서 AI를 사용하여 판매 동향을 분석하고, 재사용 가능한 뷰를 생성하고, 고급 데이터 분석을 수행하는 실습 경험을 갖게 될 것입니다.

## 작업 영역 생성

Fabric에서 데이터를 작업하기 전에 Fabric이 활성화된 작업 영역을 생성하세요. Microsoft Fabric의 작업 영역은 Lakehouse, Notebook 및 Dataset을 포함한 모든 데이터 엔지니어링 아티팩트를 구성하고 관리할 수 있는 협업 환경 역할을 합니다. 데이터 분석에 필요한 모든 리소스가 포함된 프로젝트 폴더라고 생각하시면 됩니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.

1. 왼쪽 메뉴 바에서 **작업 영역** (아이콘은 &#128455;와 유사합니다)을 선택하세요.

1. 원하는 이름으로 새 작업 영역을 생성하고, Fabric Capacity (*Premium* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택하세요. *Trial*은 지원되지 않습니다.
   
    > **중요한 이유**: Copilot이 작동하려면 유료 Fabric Capacity가 필요합니다. 이는 이 랩 전체에서 코드 생성을 돕는 AI 기반 기능에 액세스할 수 있도록 보장합니다.

1. 새 작업 영역이 열리면 비어 있어야 합니다.

![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Data Warehouse 생성

작업 영역이 생겼으니 이제 Data Warehouse를 생성할 차례입니다. Microsoft Fabric의 Data Warehouse는 분석 워크로드에 최적화된 관계형 데이터베이스입니다. 트랜잭션 작업용으로 설계된 기존 데이터베이스와 달리, Data Warehouse는 대량의 데이터와 복잡한 쿼리를 효율적으로 처리하도록 구조화되어 있습니다. 새 Warehouse를 생성하는 바로 가기를 찾으세요.

1. 왼쪽 메뉴 바에서 **생성**을 선택하세요. *새 항목* 페이지의 *Data Warehouse* 섹션에서 **Warehouse**를 선택합니다. 원하는 고유한 이름을 지정하세요. 이 이름은 작업 영역 내에서 Data Warehouse를 식별하므로, 목적을 반영하는 설명적인 이름을 선택하세요.

    >**참고**: **생성** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    1분 정도 후, 새 Warehouse가 생성됩니다. 프로비저닝 프로세스는 기본 인프라를 설정하고 분석 데이터베이스에 필요한 구성 요소를 생성합니다.

    ![Screenshot of a new warehouse.](./Images/new-data-warehouse2.png)

## 테이블 생성 및 데이터 삽입

Warehouse는 테이블 및 기타 개체를 정의할 수 있는 관계형 데이터베이스입니다. Copilot의 기능을 시연하려면 작업할 샘플 데이터가 필요합니다. 차원 테이블(고객, 날짜, 제품)과 팩트 테이블(판매 주문)을 포함하는 일반적인 소매 판매 스키마를 생성할 것입니다. 이는 Data Warehousing에서 스타 스키마라고 불리는 일반적인 패턴입니다.

1. **홈** 메뉴 탭에서 **새 SQL 쿼리** 버튼을 사용하여 새 쿼리를 생성하세요. 그러면 Transact-SQL 명령을 작성하고 실행할 수 있는 SQL 편집기가 열립니다. 그런 다음 `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-fabric/refs/heads/main/Allfiles/Labs/22c/create-dw.txt`의 Transact-SQL 코드를 새 쿼리 창에 복사하여 붙여넣습니다. 이 스크립트에는 샘플 Dataset을 구축하는 데 필요한 모든 CREATE TABLE 및 INSERT 문이 포함되어 있습니다.

1. 쿼리를 실행하면 간단한 Data Warehouse 스키마가 생성되고 일부 데이터가 로드됩니다. 스크립트는 실행하는 데 약 30초 정도 소요됩니다. 이 시간 동안 데이터베이스 엔진은 테이블 구조를 생성하고 샘플 소매 판매 데이터로 채웁니다.

1. 도구 모음의 **새로 고침** 버튼을 사용하여 뷰를 새로 고치세요. 그런 다음 **탐색기** 창에서 Data Warehouse의 **dbo** 스키마에 다음 네 개의 테이블이 포함되어 있는지 확인합니다.
   
    - **DimCustomer** - 이름 및 주소를 포함한 고객 정보
    - **DimDate** - 달력 정보(연, 월, 요일 이름 등)를 포함하는 날짜 차원 테이블
    - **DimProduct** - 범주, 이름 및 가격 정보를 포함하는 제품 카탈로그
    - **FactSalesOrder** - 차원 테이블에 대한 외래 키가 있는 판매 트랜잭션을 포함하는 중앙 팩트 테이블

    > **팁**: 스키마 로드에 시간이 걸리면 브라우저 페이지를 새로 고치세요. 탐색기 창은 데이터베이스 구조를 보여주며 테이블 및 기타 데이터베이스 개체를 쉽게 찾아볼 수 있도록 합니다.

## Data Warehouse 테이블 쿼리

Data Warehouse는 관계형 데이터베이스이므로 SQL을 사용하여 테이블을 쿼리할 수 있습니다. 그러나 처음부터 복잡한 SQL 쿼리를 작성하는 것은 시간이 많이 걸리고 오류가 발생하기 쉽습니다. Copilot과 함께 작업하면 SQL 쿼리 생성이 훨씬 빨라집니다! Copilot은 인공 지능을 사용하여 자연어 요청을 이해하고 적절한 SQL 구문으로 번역하여 데이터 분석을 더 쉽게 접근할 수 있도록 합니다.

1. 현재 **SQL 쿼리 1**을 닫으세요. 이렇게 하면 작업 영역이 지워지므로 쿼리 생성을 위해 Copilot 사용에 집중할 수 있습니다.

1. 홈 리본에서 Copilot 옵션을 선택하세요. 그러면 AI와 상호 작용하여 쿼리를 생성할 수 있는 Copilot 지원 창이 열립니다.

    ![Screenshot of Copilot pane opened in the warehouse.](./Images/copilot-fabric-data-warehouse-start.png)

1. Copilot이 무엇을 할 수 있는지 탐색하는 것으로 시작해 봅시다. `Copilot이 무엇을 할 수 있나요?`라는 레이블이 지정된 제안을 클릭하고 프롬프트로 전송하세요.

    출력을 읽고 Copilot이 현재 미리 보기 상태이며 브레인스토밍, SQL 쿼리 생성, 쿼리 설명 및 수정 등에 도움이 될 수 있음을 관찰하세요.
    
    ![Screenshot of Copilot pane with help in the warehouse.](./Images/copilot-fabric-data-warehouse-pane.png)
    
1. 월별 판매 수익을 분석하려고 합니다. 이는 일반적인 비즈니스 요구 사항으로, 시간 경과에 따른 수익 동향을 이해하는 것은 계절적 패턴, 성장 동향 및 성과 지표를 식별하는 데 도움이 됩니다. 다음 프롬프트를 입력하고 전송하세요.

    ```copilot-prompt
    /generate-sql Calculate monthly sales revenue
    ```

1. 생성된 출력을 검토하세요. 이는 환경 및 Copilot의 최신 업데이트에 따라 약간 다를 수 있습니다. Copilot이 요청을 해석하고 팩트 및 차원 테이블 간에 적절한 JOIN 문을 생성하여 월별 판매 데이터를 집계하는 방식에 주목하세요.

1. 쿼리 오른쪽 상단 모서리에 있는 **코드 삽입** 아이콘을 선택하세요. 그러면 생성된 SQL이 Copilot 창에서 SQL 편집기로 전송되어 실행할 수 있습니다.

    ![Screenshot of Copilot pane with first sql query.](./Images/copilot-fabric-data-warehouse-sql-1.png)

1. 쿼리 위에 있는 ▷ **실행** 옵션을 선택하여 쿼리를 실행하고 출력을 관찰하세요. 시간대별로 판매 데이터가 어떻게 집계되는지를 보여주는 월별 수익 합계를 볼 수 있을 것입니다.

    ![Screenshot of sql query results.](./Images/copilot-fabric-data-warehouse-sql-1-results.png)

1. **새 SQL 쿼리**를 생성하고, 결과에 월 이름과 판매 지역도 포함하도록 후속 질문을 하세요. 이는 Copilot을 사용하여 쿼리를 반복적으로 개선하는 방법을 보여줍니다. 즉, 이전 요청을 기반으로 더 상세한 분석을 생성하는 것입니다.

    ```copilot-prompt
    /generate-sql Retrieves sales revenue data grouped by year, month, month name and sales region
    ```

1. **코드 삽입** 아이콘을 선택하고 ▷ 쿼리를 **실행**하세요. 반환되는 출력을 관찰하세요. Copilot이 핵심 수익 계산 논리를 유지하면서 추가 차원을 포함하도록 쿼리를 조정하는 방식에 주목하세요.

1. 이 쿼리에서 뷰를 생성하기 위해 Copilot에 다음 질문을 해봅시다. 뷰는 쿼리 논리를 저장하는 가상 테이블로, 복잡한 쿼리를 더 쉽게 재사용하고 보고 및 분석을 위한 일관된 데이터 액세스 패턴을 제공합니다.

    ```copilot-prompt
    /generate-sql Create a view in the dbo schema that shows sales revenue data grouped by year, month, month name and sales region
    ```

1. **코드 삽입** 아이콘을 선택하고 ▷ 쿼리를 **실행**하세요. 생성되는 출력을 검토하세요. 

    쿼리가 성공적으로 실행되지 않습니다. SQL 문에 데이터베이스 이름이 접두사로 포함되어 있는데, 이는 뷰를 정의할 때 Data Warehouse에서 허용되지 않습니다. 이는 다른 데이터베이스 플랫폼에서 작업할 때 흔히 발생하는 구문 문제입니다. 한 환경에서 작동하는 것이 다른 환경에서는 조정이 필요할 수 있습니다.

1. **쿼리 오류 수정** 옵션을 선택하세요. Copilot이 쿼리를 어떻게 수정하는지 관찰하세요. 이는 Copilot의 강력한 기능 중 하나를 보여줍니다. 쿼리를 생성할 수 있을 뿐만 아니라 구문 오류를 자동으로 문제 해결하고 수정할 수도 있습니다.

    ![Screenshot of sql query with error.](./Images/copilot-fabric-data-warehouse-view-error.png)
    
    다음은 수정된 쿼리의 예시입니다. 어떤 변경이 이루어졌는지 설명하는 `Auto-Fix` 주석에 주목하세요.
    
    ```sql
    -- Auto-Fix: Removed the database name prefix from the CREATE VIEW statement
    CREATE VIEW [dbo].[SalesRevenueView] AS
    SELECT 
        [DD].[Year],
        [DD].[Month],
        [DD].[MonthName],
        -- NOTE: I couldn't find SalesRegion information in your warehouse schema
        SUM([FS1].[SalesTotal]) AS [TotalRevenue]
    FROM 
        [dbo].[FactSalesOrder] AS [FS1] -- Auto-Fix: Removed the database name prefix
    JOIN 
        [dbo].[DimDate] AS [DD] ON [FS1].[SalesOrderDateKey] = [DD].[DateKey] -- Auto-Fix: Removed the database name prefix
    -- NOTE: I couldn't find SalesRegion information in your warehouse schema
    GROUP BY 
        [DD].[Year],
        [DD].[Month],
        [DD].[MonthName]; 
    ```
    
    Copilot이 구문 오류를 수정했을 뿐만 아니라 변경 사항을 설명하고 현재 스키마에 판매 지역 정보가 없음을 지적하는 유용한 주석도 제공했음을 주목하세요.

1. 범주별로 정리된 상세 제품 목록을 검색하는 다른 프롬프트를 입력하세요. 이 쿼리는 그룹 내에서 데이터 순위를 지정하는 윈도우 함수와 같은 더 고급 SQL 기능을 시연할 것입니다. 각 제품 범주에 대해 사용 가능한 제품과 정가를 표시하고 가격을 기준으로 해당 범주 내에서 순위를 지정해야 합니다.

    ```copilot-prompt
    /generate-sql Retrieve a detailed product listing, organized by category. For each product category, it should display the available products along with their list prices and rank them within their respective categories based on price. 
    ```

1. **코드 삽입** 아이콘을 선택하고 ▷ 쿼리를 **실행**하세요. 반환되는 출력을 관찰하세요. 

    이를 통해 동일한 범주 내에서 제품을 쉽게 비교하여 가장 비싸거나 저렴한 항목을 식별하는 데 도움이 됩니다. 이 순위 기능은 제품 관리, 가격 분석 및 재고 결정에 특히 유용합니다.

## 요약

이 실습에서는 여러 테이블을 포함하는 Data Warehouse를 생성했습니다. Copilot을 사용하여 Data Warehouse의 데이터를 분석하는 SQL 쿼리를 생성했습니다. AI가 복잡한 SQL 쿼리 작성 프로세스를 가속화하고, 오류를 자동으로 수정하며, 데이터를 더 효율적으로 탐색하는 데 어떻게 도움이 되는지 경험했습니다.

이 랩 전체에서 다음을 학습했습니다.
- 자연어 프롬프트를 활용하여 SQL 쿼리 생성
- Copilot의 오류 수정 기능을 사용하여 구문 문제 해결
- AI 지원을 통해 뷰 및 복잡한 분석 쿼리 생성
- 데이터 분석을 위해 순위 함수 및 그룹화 적용

## 리소스 정리

Microsoft Fabric Data Warehouse에서 Copilot 탐색을 마쳤다면, 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 브라우저에서 Microsoft Fabric으로 이동합니다.
1. 왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인하세요.
1. **작업 영역 설정**을 선택하고 **일반** 섹션에서 아래로 스크롤하여 **이 작업 영역 제거**를 선택하세요.
1. **삭제**를 선택하여 작업 영역을 삭제합니다.