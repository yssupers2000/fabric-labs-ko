---
lab:
    title: 'Get started with Real-Time Intelligence in Microsoft Fabric'
    module: 'Get started with Real-Time Intelligence in Microsoft Fabric'
---

# Microsoft Fabric에서 Real-Time Intelligence 시작하기

Microsoft Fabric은 Real-Time Intelligence를 제공하여 실시간 데이터 스트림을 위한 분석 솔루션을 생성할 수 있도록 합니다. 이 실습에서는 Microsoft Fabric의 Real-Time Intelligence 기능을 사용하여 실시간 주식 시장 데이터 스트림을 수집(ingest), 분석 및 시각화합니다.

이 랩은 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)가 필요합니다.

## 작업 영역 만들기

Fabric에서 데이터를 사용하기 전에 Fabric Capacity가 활성화된 작업 영역을 생성해야 합니다.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 &#128455;와 유사하게 생겼습니다).
3.  원하는 이름으로 새 작업 영역을 생성하고, Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium* 또는 *Fabric*)를 선택합니다.
4.  새 작업 영역이 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Eventstream 만들기

이제 스트리밍 소스에서 실시간 데이터를 찾아 수집(ingest)할 준비가 되었습니다. 이를 위해 Fabric Real-Time Hub에서 시작합니다.

> **팁**: Real-Time Hub를 처음 사용할 때 *시작하기* 팁이 표시될 수 있습니다. 이 팁을 닫을 수 있습니다.

1.  왼쪽 메뉴 모음에서 **Real-Time** 허브를 선택합니다.

    Real-Time 허브는 스트리밍 데이터 소스를 쉽게 찾고 관리할 수 있는 방법을 제공합니다.

    ![Screenshot of the real-time hub in Fabric.](./Images/real-time-hub.png)

2.  Real-Time 허브의 **연결 대상** 섹션에서 **데이터 소스**를 선택합니다.
3.  **Stock market** 샘플 데이터 소스를 찾고 **연결**을 선택합니다. 그런 다음 **연결** 마법사에서 소스 이름을 `stock`으로 지정하고 기본 Eventstream 이름을 `stock-data`로 변경합니다. 이 데이터와 연결된 기본 스트림은 자동으로 *stock-data-stream*으로 명명됩니다.

    ![Screenshot of a new eventstream.](./Images/name-eventstream.png)

4.  **다음**을 선택한 다음 **연결**을 선택하고 소스와 Eventstream이 생성될 때까지 기다립니다. 그런 다음 **Eventstream 열기**를 선택합니다. Eventstream은 디자인 캔버스에 **stock** 소스와 **stock-data-stream**을 보여줄 것입니다.

   ![Screenshot of the eventstream canvas.](./Images/new-stock-stream.png)

## Eventhouse 만들기

Eventstream은 실시간 주식 데이터를 수집(ingest)하지만 현재 이 데이터를 가지고 아무것도 하지 않습니다. 캡처된 데이터를 테이블에 저장할 Eventhouse를 만들어 보겠습니다.

1.  왼쪽 메뉴 모음에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *Real-Time Intelligence* 섹션에서 **Eventhouse**를 선택합니다. 원하는 고유한 이름을 지정합니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않은 경우 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    새 비어 있는 Eventhouse가 표시될 때까지 표시되는 모든 팁이나 프롬프트는 닫습니다.

    ![Screenshot of a new eventhouse](./Images/create-eventhouse.png)

2.  왼쪽 창에서 Eventhouse와 동일한 이름을 가진 KQL database가 Eventhouse에 포함되어 있음을 확인하세요. 이 database에 실시간 데이터용 테이블을 생성하거나 필요에 따라 추가 database를 생성할 수 있습니다.
3.  database를 선택하고 연결된 *Queryset*이 있음을 확인하세요. 이 파일에는 database의 테이블을 쿼리하는 데 사용할 수 있는 몇 가지 샘플 KQL 쿼리가 포함되어 있습니다.

    하지만 현재 쿼리할 테이블이 없습니다. Eventstream에서 새 테이블로 데이터를 가져와서 이 문제를 해결해 보겠습니다.

4.  KQL database의 기본 페이지에서 **데이터 가져오기**를 선택합니다.
5.  데이터 소스의 경우 **Eventstream** > **기존 Eventstream**을 선택합니다.
6.  **대상 테이블 선택 또는 생성** 창에서 `stock`이라는 새 테이블을 생성합니다. 그런 다음 **데이터 소스 구성** 창에서 작업 영역과 **stock-data** Eventstream을 선택하고 연결 이름을 `stock-table`로 지정합니다.

   ![Screenshot of configuration for loading a table from an eventstream.](./Images/configure-destination.png)

7.  **다음** 버튼을 사용하여 데이터를 검사하는 단계를 완료한 다음 구성을 마칩니다. 그런 다음 구성 창을 닫아 stock 테이블이 있는 Eventhouse를 확인합니다.

   ![Screenshot of and eventhouse with a table.](./Images/eventhouse-with-table.png)

    스트림과 테이블 간의 연결이 생성되었습니다. Eventstream에서 이를 확인해 보겠습니다.

8.  왼쪽 메뉴 모음에서 **Real-Time** 허브를 선택합니다. **stock-data-stream** 스트림의 **...** 메뉴에서 **Eventstream 열기**를 선택합니다.

    이제 Eventstream은 스트림의 대상을 보여줍니다.

   ![Screenshot an eventstream with a destination.](./Images/eventstream-destination.png)

    > **팁**: 디자인 캔버스에서 대상을 선택하고, 그 아래에 데이터 미리 보기가 표시되지 않으면 **새로 고침**을 선택합니다.

    이 실습에서는 실시간 데이터를 캡처하여 테이블에 로드하는 매우 간단한 Eventstream을 생성했습니다. 실제 솔루션에서는 일반적으로 시간 창(temporal windows)을 통해 데이터를 집계하는 변환(예: 5분 간격으로 각 주식의 평균 가격을 캡처)을 추가할 것입니다.

    이제 캡처된 데이터를 쿼리하고 분석하는 방법을 살펴보겠습니다.

## 캡처된 데이터 쿼리

Eventstream은 실시간 주식 시장 데이터를 캡처하여 KQL database의 테이블에 로드합니다. 이 테이블을 쿼리하여 캡처된 데이터를 확인할 수 있습니다.

1.  왼쪽 메뉴 모음에서 Eventhouse database를 선택합니다.
2.  database의 *Queryset*을 선택합니다.
3.  쿼리 창에서 여기에 표시된 대로 첫 번째 예제 쿼리를 수정합니다.

    ```kql
    stock
    | take 100
    ```

4.  쿼리 코드를 선택하고 실행하여 테이블에서 100개의 데이터 행을 확인합니다.

    ![Screenshot of a KQL query.](./Images/kql-stock-query.png)

5.  결과를 검토한 다음 마지막 5분 동안 각 주식 심볼의 평균 가격을 검색하도록 쿼리를 수정합니다.

    ```kql
    stock
    | where ["time"] > ago(5m)
    | summarize avgPrice = avg(todecimal(bidPrice)) by symbol
    | project symbol, avgPrice
    ```

6.  수정된 쿼리를 강조 표시하고 실행하여 결과를 확인합니다.
7.  몇 초 기다렸다가 다시 실행하여 실시간 스트림에서 새 데이터가 테이블에 추가됨에 따라 평균 가격이 변경되는 것을 확인합니다.

## 실시간 대시보드 만들기

이제 데이터 스트림으로 채워지는 테이블이 있으므로 실시간 대시보드를 사용하여 데이터를 시각화할 수 있습니다.

1.  쿼리 편집기에서 지난 5분 동안의 평균 주식 가격을 검색하는 데 사용한 KQL 쿼리를 선택합니다.
2.  도구 모음에서 **대시보드에 저장**을 선택합니다. 그런 다음 다음 설정으로 쿼리를 **새 대시보드에** 고정합니다.
    - **대시보드 이름**: `Stock Dashboard`
    - **타일 이름**: `Average Prices`
3.  대시보드를 생성하고 엽니다. 다음과 같이 표시됩니다.

    ![Screenshot of a new dashboard.](./Images/stock-dashboard-table.png)

4.  대시보드 오른쪽 상단에서 **보기** 모드에서 **편집** 모드로 전환합니다.
5.  **Average Prices** 타일에 대해 **편집**(*연필*) 아이콘을 선택합니다.
6.  **시각적 서식** 창에서 **시각화**를 *테이블*에서 *세로 막대형 차트*로 변경합니다.

    ![Screenshot of a dashboard tile being edited.](./Images/edit-dashboard-tile.png)

7.  대시보드 상단에서 **변경 내용 적용**을 선택하고 수정된 대시보드를 확인합니다.

    ![Screenshot of a dashboard with a chart tile.](./Images/stock-dashboard-chart.png)

    이제 실시간 주식 데이터의 라이브 시각화를 얻었습니다.

## 경고 만들기

Microsoft Fabric의 Real-Time Intelligence에는 *Activator*라는 기술이 포함되어 있으며, 이는 실시간 이벤트를 기반으로 작업을 트리거(trigger)할 수 있습니다. 평균 주식 가격이 특정 금액만큼 증가할 때 경고를 받도록 이를 사용해 보겠습니다.

1.  주식 가격 시각화가 포함된 대시보드 창의 도구 모음에서 **경고 설정**을 선택합니다.
2.  **경고 설정** 창에서 다음 설정으로 경고를 생성합니다.
    - **쿼리 실행 주기**: 5분
    - **확인**: 각 이벤트를 다음으로 그룹화
    - **그룹화 필드**: symbol
    - **조건**: avgPrice
    - **상태**: ~만큼 증가
    - **값**: 100
    - **작업**: 이메일 보내기
    - **저장 위치**:
        - **작업 영역**: *사용자의 작업 영역*
        - **항목**: 새 항목 생성
        - **새 항목 이름**: *사용자가 선택한 고유 이름*

    ![Screenshot of alert settings.](./Images/configure-activator.png)

3.  경고를 생성하고 저장될 때까지 기다립니다. 그런 다음 생성되었음을 확인하는 창을 닫습니다.
4.  왼쪽 메뉴 모음에서 작업 영역 페이지를 선택합니다(프롬프트가 표시되면 대시보드의 저장되지 않은 변경 사항을 저장합니다).
5.  작업 영역 페이지에서 경고를 위한 Activator를 포함하여 이 실습에서 생성한 항목을 확인합니다.
6.  Activator를 열고, 해당 페이지의 **avgPrice** 노드 아래에서 경고의 고유 식별자를 선택합니다. 그런 다음 **기록** 탭을 확인합니다.

    경고가 트리거(trigger)되지 않았을 수 있으며, 이 경우 기록에는 데이터가 포함되지 않습니다. 평균 주식 가격이 100 이상 변경되면 Activator가 이메일을 보내고 경고가 기록에 기록됩니다.

## 리소스 정리

이 실습에서는 Eventhouse를 생성하고, Eventstream을 사용하여 실시간 데이터를 수집(ingest)했으며, KQL database 테이블에서 수집된 데이터를 쿼리했고, 실시간 데이터를 시각화하기 위한 실시간 대시보드를 생성했으며, Activator를 사용하여 경고를 구성했습니다.

Fabric에서 Real-Time Intelligence 탐색을 마쳤다면 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1.  왼쪽 바에서 작업 영역 아이콘을 선택합니다.
2.  도구 모음에서 **작업 영역 설정**을 선택합니다.
3.  **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.