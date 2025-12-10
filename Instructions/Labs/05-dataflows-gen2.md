`---
lab:
    title: 'Create and use Dataflows (Gen2) in Microsoft Fabric'
    module: 'Ingest Data with Dataflows Gen2 in Microsoft Fabric'
---

# Create and use Dataflows (Gen2) in Microsoft Fabric`

Microsoft Fabric에서 Dataflows (Gen2)는 다양한 데이터 원본에 연결하고 Power Query Online에서 변환을 수행합니다. 그런 다음 Data Pipelines에서 Lakehouse 또는 다른 분석 저장소로 데이터를 수집하거나 Power BI 보고서용 Dataset을 정의하는 데 사용될 수 있습니다.

이 실습은 Dataflows (Gen2)의 다양한 요소를 소개하고, 기업 환경에 존재할 수 있는 복잡한 솔루션을 생성하지 않도록 설계되었습니다. 이 실습은 완료하는 데 **약 30분**이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 생성

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 작업 영역을 생성합니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
1. 왼쪽 메뉴 바에서 **작업 영역** (아이콘은 &#128455;와 유사합니다.)을 선택합니다.
1. 원하는 이름으로 새 작업 영역을 생성하고, Fabric Capacity (*Trial*, *Premium*, 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## Lakehouse 생성

이제 작업 영역이 있으므로 데이터를 수집할 Data Lakehouse를 생성할 차례입니다.

1. 왼쪽 메뉴 바에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *데이터 엔지니어링* 섹션에서 **Lakehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다. "Lakehouse 스키마 (공개 미리 보기)" 옵션이 비활성화되어 있는지 확인합니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않은 경우 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    약 1분 후에 새 빈 Lakehouse가 생성됩니다.

 ![새 Lakehouse.](./Images/new-lakehouse.png)

## 데이터를 수집하는 Dataflow (Gen2) 생성

이제 Lakehouse가 있으므로 일부 데이터를 Lakehouse로 수집해야 합니다. 이를 수행하는 한 가지 방법은 *추출, 변환 및 로드* (ETL) 프로세스를 캡슐화하는 Dataflow를 정의하는 것입니다.

1. Lakehouse 홈 페이지에서 **데이터 가져오기** > **새 Dataflow Gen2**를 선택합니다. 몇 초 후에 여기에 표시된 대로 새 Dataflow용 Power Query 편집기가 열립니다.

 ![새 Dataflow.](./Images/new-dataflow.png)

2. **텍스트/CSV 파일에서 가져오기**를 선택하고 다음 설정으로 새 데이터 원본을 생성합니다.
 - **파일 연결**: *선택됨*
 - **파일 경로 또는 URL**: `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/orders.csv`
 - **연결**: 새 연결 생성
 - **연결 이름**: *고유한 이름을 지정합니다.*
 - **데이터 게이트웨이**: (없음)
 - **인증 종류**: 익명

3. **다음**을 선택하여 파일 데이터를 미리 보고, **만들기**를 선택하여 데이터 원본을 생성합니다. Power Query 편집기는 여기에 표시된 대로 데이터 원본과 데이터를 형식화하기 위한 초기 쿼리 단계 집합을 보여줍니다.

 ![Power Query 편집기의 쿼리.](./Images/power-query.png)

4. 도구 모음 리본에서 **열 추가** 탭을 선택합니다. 그런 다음 **사용자 지정 열**을 선택하고 새 열을 생성합니다.

5. *새 열 이름*을 `MonthNo`로 설정하고, *데이터 형식*을 **정수**로 설정한 다음 다음 수식을 추가합니다. `Date.Month([OrderDate])` - 여기에 표시된 대로:

 ![Power Query 편집기의 사용자 지정 열.](./Images/new-column.png)

6. **확인**을 선택하여 열을 생성하고, 사용자 지정 열을 추가하는 단계가 쿼리에 추가되는 방식을 확인합니다. 결과 열은 데이터 창에 표시됩니다.

 ![사용자 지정 열 단계가 있는 쿼리.](./Images/custom-column-added.png)

> **팁:** 오른쪽의 쿼리 설정 창에서 **적용된 단계**에 각 변환 단계가 포함되어 있음을 확인합니다. 하단에서 **다이어그램 흐름** 버튼을 전환하여 단계의 시각적 다이어그램을 켤 수도 있습니다.
>
> 단계는 위아래로 이동할 수 있으며, 톱니바퀴 아이콘을 선택하여 편집할 수 있습니다. 각 단계를 선택하여 미리 보기 창에서 변환이 적용되는 것을 볼 수 있습니다.

7. **OrderDate** 열의 데이터 형식이 **날짜**로 설정되어 있고, 새로 생성된 **MonthNo** 열의 데이터 형식이 **정수**로 설정되어 있는지 확인합니다.

## Dataflow에 데이터 대상 추가

1. 도구 모음 리본에서 **홈** 탭을 선택합니다. 그런 다음 **쿼리** 섹션에서 **데이터 대상 추가**를 선택합니다.

    ![리본의 스크린샷, 데이터 대상 추가 옵션이 강조 표시됨.](./Images/add-data-destination.png)

2. **Lakehouse**를 선택합니다.

3. **기본 데이터 대상에 연결** 대화 상자에서 연결을 편집하고 Power BI 조직 계정을 사용하여 로그인하여 Dataflow가 Lakehouse에 액세스하는 데 사용하는 ID를 설정합니다.

    ![데이터 대상 구성 페이지.](./Images/dataflow-connection.png)

4. **다음**을 선택하고 사용 가능한 작업 영역 목록에서 작업 영역을 찾아 이 실습 시작 시 생성한 Lakehouse를 선택합니다. 그런 다음 **orders**라는 새 테이블을 지정합니다.

   ![데이터 대상 구성 페이지.](./Images/data-destination-target.png)

5. **다음**을 선택하고 **대상 설정 선택** 페이지에서 **자동 설정 사용** 옵션을 비활성화하고, **추가(Append)**를 선택한 다음 **설정 저장**을 선택합니다.
    > **참고:** 데이터 형식 업데이트에는 *Power Query* 편집기를 사용하는 것을 권장하지만, 원한다면 이 페이지에서도 수행할 수 있습니다.

    ![데이터 대상 설정 페이지.](./Images/destination-settings.png)

6. 메뉴 바에서 **보기**를 열고 **다이어그램 보기**를 선택합니다. Power Query 편집기의 쿼리에서 **Lakehouse** 대상이 아이콘으로 표시되는 것을 확인합니다.

   ![Lakehouse 대상을 포함하는 쿼리.](./Images/lakehouse-destination.png)

7. 도구 모음 리본에서 **홈** 탭을 선택합니다. 그런 다음 **저장 및 실행**을 선택하고 **Dataflow 1** Dataflow가 작업 영역에 생성될 때까지 기다립니다.

## Pipeline에 Dataflow 추가

Pipeline에 Dataflow를 활동으로 포함할 수 있습니다. Pipeline은 데이터 수집 및 처리 활동을 오케스트레이션하는 데 사용되며, 단일 예약된 프로세스에서 Dataflow를 다른 종류의 작업과 결합할 수 있도록 합니다. Pipeline은 Data Factory 환경을 포함하여 몇 가지 다른 환경에서 생성될 수 있습니다.

1. Fabric이 활성화된 작업 영역에서 **+ 새 항목** > **데이터 Pipeline**을 선택한 다음, 프롬프트가 표시되면 **Load data**라는 새 Pipeline을 생성합니다.

   Pipeline 편집기가 열립니다.

   ![빈 데이터 Pipeline.](./Images/new-pipeline.png)

   > **팁**: 데이터 복사 마법사가 자동으로 열리면 닫으세요!

2. **Pipeline 활동**을 선택하고 Pipeline에 **Dataflow** 활동을 추가합니다.

3. 새 **Dataflow1** 활동이 선택된 상태에서 **설정** 탭의 **Dataflow** 드롭다운 목록에서 **Dataflow 1** (이전에 생성한 Dataflow)을 선택합니다.

   ![Dataflow 활동이 포함된 Pipeline.](./Images/dataflow-activity.png)

4. **홈** 탭에서 **&#128427;** (저장) 아이콘을 사용하여 Pipeline을 저장합니다.
5. **&#9655; 실행** 버튼을 사용하여 Pipeline을 실행하고 완료될 때까지 기다립니다. 몇 분 정도 걸릴 수 있습니다.

   ![Dataflow가 성공적으로 완료된 Pipeline.](./Images/dataflow-pipeline-succeeded.png)

6. 왼쪽 메뉴 바에서 Lakehouse를 선택합니다.
7. **테이블**의 **...** 메뉴에서 **새로 고침**을 선택합니다. 그런 다음 **테이블**을 확장하고 Dataflow에 의해 생성된 **orders** 테이블을 선택합니다.

   ![Dataflow에 의해 로드된 테이블.](./Images/loaded-table.png)

> **팁**: Power BI Desktop에서 *Power BI dataflows (Legacy)* 커넥터를 사용하여 Dataflow로 수행된 데이터 변환에 직접 연결할 수 있습니다.
>
> 또한 추가 변환을 수행하고, 새 Dataset으로 게시하며, 특수 Dataset을 위해 의도된 대상에게 배포할 수도 있습니다.
>
>![Power BI 데이터 원본 커넥터](Images/pbid-dataflow-connectors.png)

## 리소스 정리

Microsoft Fabric에서 Dataflow 탐색을 마쳤다면 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 브라우저에서 Microsoft Fabric으로 이동합니다.
1. 왼쪽 바에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 볼 수 있습니다.
1. **작업 영역 설정**을 선택하고 **일반** 섹션에서 아래로 스크롤하여 **이 작업 영역 제거**를 선택합니다.
1. **삭제**를 선택하여 작업 영역을 삭제합니다.