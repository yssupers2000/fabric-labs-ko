---
lab:
    title: 'Chat with your data using Microsoft Fabric data agents'
    module: 'Implement Fabric Data Agents'
---

# Microsoft Fabric data agent를 사용하여 데이터와 대화하기

Microsoft Fabric data agent는 일반 영어로 질문하고 구조화되고 사람이 읽을 수 있는 응답을 받을 수 있도록 하여 데이터와 자연스러운 상호 작용을 가능하게 합니다. SQL (Structured Query Language), DAX (Data Analysis Expressions), KQL (Kusto Query Language)과 같은 쿼리 언어를 이해할 필요성을 없앰으로써, data agent는 기술 수준과 관계없이 조직 전체에서 데이터 인사이트(data insights)에 접근할 수 있게 합니다.

이 실습은 완료하는 데 약 **30**분이 소요됩니다.

## 학습 내용

이 실습을 완료하면 다음을 수행하게 됩니다.

- 자연어 데이터 분석을 위한 Microsoft Fabric data agent의 목적과 이점을 이해합니다.
- Fabric workspace 및 Data Warehouse를 생성하고 구성하는 방법을 학습합니다.
- 스타 스키마(star schema) 판매 데이터셋(dataset)을 로드하고 탐색하는 실습 경험을 얻습니다.
- data agent가 일반 영어 질문을 SQL 쿼리로 변환하는 방법을 확인합니다.
- 효과적인 분석 질문을 하고 AI 생성 결과를 해석하는 기술을 개발합니다.
- AI 도구를 활용하여 데이터 접근 및 인사이트(insights)를 민주화하는 데 자신감을 얻습니다.

## 시작하기 전에

이 실습을 완료하려면 Copilot이 활성화된 [Microsoft Fabric Capacity (F2 이상)](https://learn.microsoft.com/fabric/fundamentals/copilot-enable-fabric)이 필요합니다.

## 실습 시나리오

이 실습에서는 판매 Data Warehouse를 생성하고 데이터를 로드한 다음 Fabric data agent를 생성합니다. 그런 다음 다양한 질문을 하고 data agent가 자연어를 SQL 쿼리로 변환하여 인사이트(insights)를 제공하는 방법을 탐색합니다. 이 실습 접근 방식은 깊이 있는 SQL 지식 없이도 AI 지원 데이터 분석의 강력함을 보여줄 것입니다. 시작해 봅시다!

## Workspace 생성

Fabric에서 데이터를 사용하기 전에 Fabric이 활성화된 workspace를 생성하세요. Microsoft Fabric의 workspace는 Lakehouse, Notebook, Dataset(dataset)을 포함한 모든 데이터 엔지니어링 아티팩트(artifacts)를 구성하고 관리할 수 있는 협업 환경 역할을 합니다. 데이터 분석에 필요한 모든 리소스(resources)를 포함하는 프로젝트 폴더로 생각하세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명(credentials)으로 로그인하세요.

1.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 비슷합니다.)를 선택하세요.

1.  원하는 이름으로 새 workspace를 생성하고, Fabric Capacity (*Premium* 또는 *Fabric*)를 포함하는 라이선스 모드(licensing mode)를 선택하세요. *Trial*은 지원되지 않습니다.

    > **중요한 이유**: Copilot이 작동하려면 유료 Fabric Capacity가 필요합니다. 이는 이 실습 전반에 걸쳐 코드를 생성하는 데 도움이 되는 AI 기반 기능에 접근할 수 있도록 보장합니다.

1.  새 workspace가 열리면 비어 있어야 합니다.

![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Data Warehouse 생성

이제 workspace가 생겼으니 Data Warehouse를 생성할 차례입니다. Data Warehouse는 다양한 소스에서 구조화된 데이터를 저장하며, 분석 쿼리 및 보고에 최적화된 중앙 집중식 저장소입니다. 이 실습에서는 data agent 상호 작용의 기반 역할을 할 간단한 판매 Data Warehouse를 생성할 것입니다. 새 Warehouse를 생성하는 바로 가기를 찾으세요.

1.  왼쪽 메뉴 바에서 **생성**을 선택하세요. *새로 만들기* 페이지의 *Data Warehouse* 섹션에서 **Warehouse**를 선택하세요. 원하는 고유한 이름을 지정하세요.

    >**참고**: **생성** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    1분 정도 후에 새 Warehouse가 생성됩니다.

    ![Screenshot of a new warehouse.](./Images/new-data-warehouse2.png)

## 테이블 생성 및 데이터 삽입

Warehouse는 테이블 및 기타 개체(objects)를 정의할 수 있는 관계형 데이터베이스(relational database)입니다. data agent를 유용하게 사용하려면 샘플 판매 데이터로 채워야 합니다. 실행할 스크립트(script)는 차원 테이블(dimension tables)(설명 속성(attributes) 포함)과 팩트 테이블(fact table)(측정 가능한 비즈니스 이벤트 포함)을 사용하여 일반적인 Data Warehouse 스키마(schema)를 생성합니다. 이 스타 스키마(star schema) 디자인은 data agent가 생성할 분석 쿼리에 최적화되어 있습니다.

1.  **홈** 메뉴 탭에서 **새 SQL 쿼리** 버튼을 사용하여 새 쿼리를 생성하세요. 그런 다음 `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-fabric/refs/heads/main/Allfiles/Labs/22d/create-dw.txt`에서 Transact-SQL 코드를 복사하여 새 쿼리 창에 붙여넣으세요.

    > **이 스크립트(script)의 기능**: 이 스크립트(script)는 고객 정보, 제품 세부 정보, 날짜 차원(date dimensions) 및 판매 트랜잭션(transactions)을 포함하는 완전한 판매 Data Warehouse를 생성합니다. 이 현실적인 Dataset(dataset)을 통해 data agent에 의미 있는 비즈니스 질문을 할 수 있습니다.

1.  쿼리를 실행하세요. 이는 간단한 Data Warehouse 스키마(schema)를 생성하고 일부 데이터를 로드합니다. 스크립트(script) 실행에는 약 30초가 소요됩니다.

1.  도구 모음에서 **새로 고침** 버튼을 사용하여 보기를 새로 고치세요. 그런 다음 **탐색기** 창에서 Data Warehouse의 **dbo** 스키마(schema)에 다음 네 개의 테이블이 포함되어 있는지 확인하세요.

    -   **DimCustomer** - 이름, 위치 및 연락처 세부 정보를 포함한 고객 정보를 포함합니다.
    -   **DimDate** - 시간 기반 분석을 위한 회계 연도, 분기, 월과 같은 날짜 관련 속성(attributes)을 포함합니다.
    -   **DimProduct** - 이름, 범주(categories) 및 가격 책정을 포함한 제품 정보를 포함합니다.
    -   **FactSalesOrder** - 고객, 제품 및 날짜를 연결하는 실제 판매 트랜잭션(transactions)을 포함합니다.

    > **팁**: 스키마(schema) 로드에 시간이 오래 걸리는 경우, 브라우저 페이지를 새로 고치세요.

## Fabric data agent 생성

Fabric data agent는 데이터에 대한 자연어 질문을 이해하고 질문에 답하기 위해 적절한 쿼리를 자동으로 생성할 수 있는 AI 기반 어시스턴트(assistant)입니다. 이는 사용자가 SQL, KQL 또는 DAX 구문(syntax)을 알 필요성을 없애면서도 정확하고 데이터 기반의 인사이트(insights)를 제공합니다. data agent를 생성하고 구성해 봅시다.

1.  새 data agent를 생성하세요.

    ![Screenshot of creating a new data agent](./Images/copilot-fabric-data-agent-new.png)

1.  **`sales-data-agent`**와 같은 이름을 지정하세요.

    > **이름 지정이 중요한 이유**: 설명적인 이름은 귀하와 귀하의 팀이 이 data agent의 목적과 범위를 이해하는 데 도움이 됩니다. 특히 다양한 데이터 도메인(domains)에 대한 여러 agent를 관리할 때 더욱 그렇습니다.

    ![Screenshot of creating a new data agent and assigning it a name.](./Images/copilot-fabric-data-agent-create.png)

1.  **데이터 소스 추가**를 선택하세요.

    ![Screenshot of data agent created.](./Images/copilot-fabric-data-agent-created.png)

1.  이전에 생성한 Data Warehouse를 선택하세요.

    > **데이터 연결**: data agent는 스키마(schema)와 관계(relationships)를 이해하기 위해 테이블에 접근해야 합니다. 이를 통해 질문에 기반한 정확한 SQL 쿼리를 생성할 수 있습니다.

1.  Data Warehouse를 확장하고 **DimCustomer**, **DimDate**, **DimProduct**, **FactSalesOrder**를 확인하세요.

    > **테이블 선택 전략**: 네 개의 테이블을 모두 선택함으로써 data agent에 전체 데이터 모델(data model)에 대한 접근 권한을 부여하는 것입니다. 이를 통해 고객 위치별 판매 추세 또는 시간 경과에 따른 제품 성능과 같이 여러 테이블에 걸쳐 있는 복잡한 질문에 답할 수 있습니다.

    ![Screenshot of data agent warehouse tables selected.](./Images/copilot-fabric-data-agent-select-tables.png)

## 질문하기

이제 data agent에 질문을 하고 실험을 시작할 시간입니다. 이 섹션은 자연어가 SQL 쿼리로 어떻게 변환될 수 있는지 보여주며, 기술적인 SQL 지식 없이도 사용자들이 데이터 분석에 접근할 수 있도록 합니다. 각 질문은 답변과 생성된 기본 쿼리를 모두 보여줄 것입니다.

1.  다음 프롬프트(prompt)를 입력하여 질문하세요.

    ```copilot-prompt
    How many products did we sell by fiscal year?
    ```

    결과 답변을 확인하세요: 회계 연도 2021년에 총 12,630개의 제품을 판매했고, 2022년 회계 연도에는 13,336개의 제품을 판매했습니다.

1.  완료된 단계와 하위 단계를 확장하세요. 이는 질문에 답하기 위해 data agent가 생성한 SQL 쿼리를 보여줍니다.

    > **학습 기회**: 생성된 SQL을 검토함으로써 data agent가 질문을 어떻게 해석했는지 이해하고 기본 데이터 관계(data relationships)에 대해 학습할 수 있습니다. 이러한 투명성은 AI 생성 결과에 대한 신뢰를 구축합니다.

    ![Screenshot of data agent query steps explained](./Images/copilot-fabric-data-agent-query-1-explanation.png)

    Copilot은 다음 SQL 코드를 생성했습니다. 이는 환경 및 Copilot의 최신 업데이트에 따라 약간 다를 수 있습니다.

    ```sql
    SELECT d.Year, SUM(f.Quantity) AS TotalProductsSold
    FROM dbo.FactSalesOrder f
    JOIN dbo.DimDate d ON f.SalesOrderDateKey = d.DateKey
    GROUP BY d.Year
    ORDER BY d.Year;
    ```

    > **SQL 설명**: 이 쿼리는 팩트 테이블(FactSalesOrder)을 날짜 차원(DimDate)과 조인(join)하여 연도별 판매를 그룹화하고 수량을 합산합니다. data agent가 "products sold"가 Quantity 필드를 나타내고 "fiscal year"가 날짜 차원(date dimension)의 Year 필드에 매핑(map)된다는 것을 자동으로 이해한 방식에 주목하세요.

1.  다음 질문을 계속하세요.

    ```copilot-prompt
    What are the top 10 most popular products all time?
    ```

    > **예상 결과**: 이 질문은 data agent가 순위 작업(ranking operations)을 수행하고 제품 정보와 판매 데이터를 조인(join)하여 베스트셀러를 식별하는 방법을 보여줄 것입니다.

1.  이 질문으로 이어가세요.

    ```copilot-prompt
    What are the historical trends across all my data?
    ```

    > **고급 분석**: 이 광범위한 질문은 data agent가 여러 차원(dimensions)에 걸쳐 추세 분석(trend analysis)을 제공하는 방법을 보여줄 것입니다. 잠재적으로 판매의 시간 기반 패턴, 고객 행동 및 제품 성능을 포함하여 말입니다.

1.  데이터의 다양한 측면을 탐색하기 위해 추가 질문을 시도해 보세요.

    ```copilot-prompt
    In which countries are our customers located?
    ```

    ```copilot-prompt
    How many products did we sell in the United States?
    ```

    ```copilot-prompt
    How much revenue did we make in FY 2022?
    ```

    ```copilot-prompt
    How much was our total sales revenue, by fiscal year, fiscal quarter and month name?
    ```

    > **전문가 팁**: 이 질문들은 각각 지리적 분석, 필터링된 집계(aggregations), 수익 계산, 계층적 시간 분석과 같은 다른 분석 시나리오(scenarios)를 목표로 합니다. 다양한 질문 스타일(styles)에 data agent가 어떻게 적응하는지 확인하기 위해 여러 변형을 실험해 보세요.

## 데이터 구조 이해하기

질문을 실험할 때, 보다 목표에 맞는 질문을 하기 위해 다음 데이터 특성(characteristics)을 염두에 두세요.

-   **회계 연도 시기**: 회계 연도는 7월(7번째 달)에 시작합니다. 따라서 1분기(Q1)는 7월-9월, 2분기(Q2)는 10월-12월, 3분기(Q3)는 1월-3월, 4분기(Q4)는 4월-6월입니다.

-   **고객 식별**: CustomerAltKey 필드에는 고객 이메일 주소가 포함되어 있으며, 이는 고객별 쿼리에 유용할 수 있습니다.

-   **통화**: 모든 정가(list prices) 및 총 판매액은 GBP (영국 파운드)로 표시됩니다.

-   **데이터 관계**: FactSalesOrder 테이블은 외래 키(foreign keys)를 통해 고객, 제품 및 날짜를 연결하여 복잡한 다차원 분석(multi-dimensional analysis)을 가능하게 합니다.

> **추가 실험**: "2022 회계 연도 1분기(Q1)의 수익은 얼마였습니까?" 또는 "영국의 어떤 고객이 가장 비싼 제품을 구매했습니까?" 와 같이 이러한 요소들을 결합한 질문을 시도해 보세요. data agent는 이러한 질문에 답하는 데 필요한 복잡한 조인(joins) 및 계산을 자동으로 처리할 것입니다.

## 요약

축하합니다! 다음을 성공적으로 완료했습니다.

-   **Fabric workspace를 생성했습니다.** 그리고 현실적인 판매 Dataset(dataset)으로 Data Warehouse를 생성했습니다.
-   **data agent를 구축하고 구성했습니다.** 이는 데이터에 대한 자연어 질문을 이해할 수 있습니다.
-   **AI 기반 데이터 분석을 경험했습니다.** 일반 영어로 질문하고 질문이 SQL 쿼리로 변환되는 방식을 확인하여 말입니다.
-   **다양한 유형의 분석 질문을 탐색했습니다.** 간단한 집계(aggregations)부터 복잡한 추세 분석(trend analysis)까지 말입니다.

### 주요 내용

-   **데이터 접근 민주화**: data agent는 SQL 지식과 관계없이 사용자들이 분석에 접근할 수 있도록 합니다.
-   **투명성과 신뢰**: 생성된 SQL을 항상 검사하여 질문이 어떻게 답변되는지 이해할 수 있습니다.
-   **자연어 유연성**: AI는 질문의 구문(phrasing) 변형과 사소한 오타까지 처리할 수 있습니다.
-   **복잡한 쿼리 생성**: agent는 자연어 입력에 기반하여 조인(joins), 집계(aggregations) 및 필터(filters)를 자동으로 처리합니다.

### 다음 단계

다음을 탐색해 보세요.

-   **사용자 지정 지침**: data agent의 응답을 개선하기 위해 비즈니스별 컨텍스트(context)를 추가하세요.
-   **추가 데이터 소스**: agent의 지식을 확장하기 위해 더 많은 테이블 또는 Dataset(dataset)을 연결하세요.
-   **고급 질문**: 여러 기간, 고객 세그먼트(customer segments) 또는 제품 범주(categories)와 관련된 더 복잡한 분석 시나리오(scenarios)를 시도해 보세요.
-   **통합**: data agent 인사이트(insights)를 보고서, 대시보드(dashboards) 또는 비즈니스 애플리케이션(applications)에 포함하세요.

Fabric data agent는 조직 전체에서 데이터 인사이트(data insights)에 진정으로 접근할 수 있도록 하는 중요한 단계를 나타내며, 데이터와 의사 결정(decision-making) 사이의 격차를 해소합니다.