---
lab:
    title: 'Get started with Real-time Dashboards in Microsoft Fabric'
    module: 'Get started with Real-Time Dashboards in Microsoft Fabric'
---

# Microsoft Fabric에서 실시간 대시보드 시작하기

Microsoft Fabric의 실시간 대시보드는 Kusto Query Language(KQL)를 사용하여 스트리밍 데이터를 시각화하고 탐색할 수 있도록 합니다. 이 실습에서는 실시간 데이터 소스를 기반으로 실시간 대시보드를 생성하고 사용하는 방법을 알아봅니다.

이 랩을 완료하는 데 약 **25**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)가 필요합니다.

## 작업 영역 만들기

Fabric에서 데이터를 작업하기 전에 Fabric Capacity가 활성화된 작업 영역을 생성해야 합니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2. 왼쪽 메뉴 바에서 **작업 영역(Workspaces)** (아이콘은 &#128455;와 비슷합니다)을 선택합니다.
3. 원하는 이름으로 새 작업 영역을 생성하고, Fabric Capacity를 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택합니다.
4. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## Eventhouse 생성

이제 작업 영역이 있으므로 실시간 인텔리전스 솔루션에 필요한 Fabric 항목을 생성할 수 있습니다. Eventhouse를 생성하는 것부터 시작하겠습니다.

1. 왼쪽 메뉴 바에서 **만들기(Create)** 를 선택합니다. *새로 만들기(New)* 페이지의 *실시간 인텔리전스(Real-Time Inteligence)* 섹션에서 **Eventhouse**를 선택합니다. 원하는 고유한 이름을 지정합니다.

    >**참고**: **만들기(Create)** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

2. 새롭고 비어 있는 Eventhouse가 표시될 때까지 표시되는 팁이나 프롬프트를 닫습니다.

    ![새 Eventhouse 스크린샷](./Images/create-eventhouse.png)

3. 왼쪽 창에서 Eventhouse가 Eventhouse와 동일한 이름을 가진 KQL 데이터베이스를 포함하고 있음을 확인합니다.
4. KQL 데이터베이스를 선택하여 확인합니다.

## Eventstream 생성

현재 데이터베이스에는 테이블이 없습니다. Eventstream을 사용하여 실시간 소스에서 테이블로 데이터를 로드할 것입니다.

1. KQL 데이터베이스의 메인 페이지에서 **데이터 가져오기(Get data)** 를 선택합니다.
2. 데이터 소스(data source)로 **Eventstream** > **새 Eventstream(New eventstream)** 을 선택합니다. Eventstream 이름을 `Bicycle-data`로 지정합니다.

    ![새 Eventstream 스크린샷.](./Images/empty-eventstream.png)

    작업 영역에서 새 Eventstream 생성이 잠시 후 완료됩니다. 생성되면 Eventstream의 데이터 소스(data source)를 선택하도록 자동으로 리디렉션됩니다.

3. **샘플 데이터 사용(Use sample data)** 을 선택합니다.
4. 소스 이름(source name)을 `Bicycles`로 지정하고, **Bicycles** 샘플 데이터를 선택합니다.

    스트림이 매핑되고 **eventstream 캔버스**에 자동으로 표시됩니다.

   ![Eventstream 캔버스 검토](./Images/real-time-intelligence-eventstream-sourced.png)

5. **대상 추가(Add destination)** 드롭다운 목록에서 **Eventhouse**를 선택합니다.
6. **Eventhouse** 창에서 다음 설정 옵션을 구성합니다.
   - **데이터 수집 모드(Data ingestion mode)**: 수집 전 이벤트 처리
   - **대상 이름(Destination name)**: `bikes-table`
   - **작업 영역(Workspace)**: *이 실습 시작 시 생성한 작업 영역을 선택합니다.*
   - **Eventhouse**: *Eventhouse를 선택합니다.*
   - **KQL 데이터베이스(KQL database)**: *KQL 데이터베이스를 선택합니다.*
   - **대상 테이블(Destination table)**: `bikes`라는 새 테이블을 생성합니다.
   - **입력 데이터 형식(Input data format)**: JSON

   ![Eventstream 대상 설정.](./Images/kql-database-event-processing-before-ingestion.png)

7. **Eventhouse** 창에서 **저장(Save)** 을 선택합니다.
8. **Bicycles-data** 노드의 출력을 **bikes-table** 노드에 연결한 다음 **게시(Publish)** 를 선택합니다.
9. 데이터 대상이 활성화될 때까지 1분 정도 기다립니다. 그런 다음 디자인 캔버스에서 **bikes-table** 노드를 선택하고 아래의 **데이터 미리 보기(Data preview)** 창에서 수집된 최신 데이터를 확인합니다.

   ![Eventstream의 대상 테이블 스크린샷.](./Images/stream-data-preview.png)

10. 몇 분 정도 기다린 다음 **새로 고침(Refresh)** 버튼을 사용하여 **데이터 미리 보기(Data preview)** 창을 새로 고칩니다. 스트림은 계속 실행되므로 새 데이터가 테이블에 추가되었을 수 있습니다.

## 실시간 대시보드 만들기

이제 Eventhouse의 테이블로 실시간 데이터 스트림이 로드되므로, 실시간 대시보드를 사용하여 이를 시각화할 수 있습니다.

1. 왼쪽 메뉴 바에서 **만들기(Create)** 를 선택합니다. *새로 만들기(New)* 페이지의 *실시간 인텔리전스(Real-Time Inteligence)* 섹션에서 **실시간 대시보드(Real-Time Dashboard)** 를 선택하고 `bikes-dashboard`라고 이름을 지정합니다.

    >**참고**: **만들기(Create)** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    새 비어 있는 대시보드가 생성됩니다.

    ![새 대시보드 스크린샷.](./Images/new-dashboard.png)

2. 도구 모음에서 **새 데이터 소스(New data source)** 를 선택한 다음 **Eventhouse / KQL Database**를 선택합니다. 그런 다음 Eventhouse를 선택하고 다음 설정으로 새 데이터 소스(data source)를 생성합니다.
    - **표시 이름(Display name)**: `Bike Rental Data`
    - **데이터베이스(Database)**: *Eventhouse의 기본 데이터베이스*.
    - **패스스루 ID(Passthrough identity)**: *선택됨*

3. **추가(Add)** 를 선택합니다.
4. 대시보드 디자인 캔버스에서 **타일 추가(Add tile)** 를 선택합니다.
5. 쿼리 편집기에서 **Bike Rental Data** 소스(source)가 선택되어 있는지 확인하고 다음 KQL 코드를 입력합니다.

    ```kql
    bikes
        | where ingestion_time() between (ago(30min) .. now())
        | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
        | project Neighbourhood, latest_observation, No_Bikes, No_Empty_Docks
        | order by Neighbourhood asc
    ```

6. 쿼리를 실행하면 지난 30분 동안 각 Neighbourhood(지역)에서 관찰된 자전거 수와 빈 도크 수가 표시됩니다.
7. 변경 사항을 적용하여 대시보드의 타일에서 테이블로 표시되는 데이터를 확인합니다.

   ![테이블을 포함하는 타일이 있는 대시보드 스크린샷.](./Images/tile-table.png)

8. 타일에서 **편집(Edit)** 아이콘(연필 모양)을 선택합니다. 그런 다음 **시각적 서식(Visual Formatting)** 창에서 다음 속성을 설정합니다.
    - **타일 이름(Tile name)**: Bikes and Docks
    - **시각화 유형(Visual type)**: Bar chart
    - **시각화 형식(Visual format)**: Stacked bar chart
    - **Y축 열(Y columns)**: No_Bikes, No-Empty_Docks
    - **X축 열(X column)**: Neighbourhood
    - **계열 열(Series columns)**: infer
    - **범례 위치(Legend location)**: 하단

    편집된 타일은 다음과 같아야 합니다.

   ![막대 차트를 포함하도록 편집 중인 타일 스크린샷.](./Images/tile-bar-chart.png)

9. 변경 사항을 적용한 다음 타일 크기를 조정하여 대시보드 왼쪽의 전체 높이를 차지하도록 합니다.

10. 도구 모음에서 **새 타일(New tile)** 을 선택합니다.
11. 쿼리 편집기에서 **Bike Rental Data** 소스(source)가 선택되어 있는지 확인하고 다음 KQL 코드를 입력합니다.

    ```kql
    bikes
        | where ingestion_time() between (ago(30min) .. now())
        | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
        | project Neighbourhood, latest_observation, Latitude, Longitude, No_Bikes
        | order by Neighbourhood asc
    ```

12. 쿼리를 실행하면 지난 30분 동안 각 Neighbourhood(지역)에서 관찰된 자전거의 위치와 수가 표시됩니다.
13. 변경 사항을 적용하여 대시보드의 타일에서 테이블로 표시되는 데이터를 확인합니다.
14. 타일에서 **편집(Edit)** 아이콘(연필 모양)을 선택합니다. 그런 다음 **시각적 서식(Visual Formatting)** 창에서 다음 속성을 설정합니다.
    - **타일 이름(Tile name)**: Bike Locations
    - **시각화 유형(Visual type)**: Map
    - **위치 정의 기준(Define location by)**: Latitude and longitude
    - **위도 열(Latitude column)**: Latitude
    - **경도 열(Longitude column)**: Longitude
    - **레이블 열(Label column)**: Neighbourhood
    - **크기(Size)**: 표시
    - **크기 열(Size column)**: No_Bikes

15. 변경 사항을 적용한 다음 지도 타일의 크기를 조정하여 대시보드에서 사용 가능한 공간의 오른쪽을 채우도록 합니다.

   ![차트와 지도가 있는 대시보드 스크린샷.](./Images/dashboard-chart-map.png)

## 기본 쿼리 생성

대시보드에는 유사한 쿼리를 기반으로 하는 두 개의 시각화(visual)가 포함되어 있습니다. 중복을 피하고 대시보드의 유지 관리성을 높이려면 공통 데이터를 단일 *기본 쿼리(base query)* 로 통합할 수 있습니다.

1. 대시보드 도구 모음에서 **기본 쿼리(Base queries)** 를 선택합니다. 그런 다음 **+ 추가(+Add)** 를 선택합니다.
2. 기본 쿼리 편집기에서 **변수 이름(Variable name)** 을 `base_bike_data`로 설정하고 **Bike Rental Data** 소스(source)가 선택되어 있는지 확인합니다. 그런 다음 다음 쿼리를 입력합니다.

    ```kql
    bikes
        | where ingestion_time() between (ago(30min) .. now())
        | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
    ```
3. 쿼리를 실행하고 대시보드의 두 시각화(visual) 모두에 필요한 모든 열(및 기타 일부 열)을 반환하는지 확인합니다.

   ![기본 쿼리 스크린샷.](./Images/dashboard-base-query.png)

4. **완료(Done)** 를 선택한 다음 **기본 쿼리(Base queries)** 창을 닫습니다.
5. **Bikes and Docks** 막대 차트 시각화(visual)를 편집하고 쿼리를 다음 코드로 변경합니다.

    ```kql
    base_bike_data
    | project Neighbourhood, latest_observation, No_Bikes, No_Empty_Docks
    | order by Neighbourhood asc
    ```

6. 변경 사항을 적용하고 막대 차트가 여전히 모든 Neighbourhood(지역)에 대한 데이터를 표시하는지 확인합니다.

7. **Bike Locations** 지도 시각화(visual)를 편집하고 쿼리를 다음 코드로 변경합니다.

    ```kql
    base_bike_data
    | project Neighbourhood, latest_observation, No_Bikes, Latitude, Longitude
    | order by Neighbourhood asc
    ```

8. 변경 사항을 적용하고 지도가 여전히 모든 Neighbourhood(지역)에 대한 데이터를 표시하는지 확인합니다.

## 매개 변수 추가

대시보드는 현재 모든 Neighbourhood(지역)에 대한 최신 자전거, 도크 및 위치 데이터를 표시합니다. 이제 특정 Neighbourhood(지역)를 선택할 수 있도록 매개 변수(parameter)를 추가해 보겠습니다.

1. 대시보드 도구 모음의 **관리(Manage)** 탭에서 **매개 변수(Parameters)** 를 선택합니다.
2. 자동으로 생성된 기존 매개 변수(예: *시간 범위(Time range)* 매개 변수)를 확인합니다. 그런 다음 **삭제(Delete)** 합니다.
3. **+ 추가(+ Add)** 를 선택합니다.
4. 다음 설정으로 매개 변수(parameter)를 추가합니다.
    - **레이블(Label)**: `Neighbourhood`
    - **매개 변수 유형(Parameter type)**: 다중 선택
    - **설명(Description)**: `Choose neighbourhoods`
    - **변수 이름(Variable name)**: `selected_neighbourhoods`
    - **데이터 유형(Data type)**: string
    - **페이지에 표시(Show on pages)**: 모두 선택
    - **소스(Source)**: 쿼리
    - **데이터 소스(Data source)**: Bike Rental Data
    - **쿼리 편집(Edit query)**:

        ```kql
        bikes
        | distinct Neighbourhood
        | order by Neighbourhood asc
        ```

    - **값 열(Value column)**: Neighbourhood
    - **레이블 열(Label column)**: 값 선택과 일치
    - **"모두 선택(Select all)" 값 추가**: *선택됨*
    - **"모두 선택(Select all)" 시 빈 문자열 전송**: *선택됨*
    - **기본값으로 자동 재설정**: 선택됨
    - **기본값(Default value)**: 모두 선택

5. **완료(Done)** 를 선택하여 매개 변수(parameter)를 생성합니다.

이제 매개 변수(parameter)를 추가했으므로, 선택한 Neighbourhood(지역)를 기반으로 데이터를 필터링하도록 기본 쿼리(base query)를 수정해야 합니다.

6. 도구 모음에서 **기본 쿼리(Base queries)** 를 선택합니다. 그런 다음 **base_bike_data** 쿼리를 선택하고 다음 코드에 표시된 대로 선택된 매개 변수(parameter) 값을 기반으로 필터링하도록 **where** 절에 **and** 조건을 추가하여 편집합니다.

    ```kql
    bikes
        | where ingestion_time() between (ago(30min) .. now())
          and (isempty(['selected_neighbourhoods']) or Neighbourhood  in (['selected_neighbourhoods']))
        | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
    ```

7. **완료(Done)** 를 선택하여 기본 쿼리(base query)를 저장합니다.

8. 대시보드에서 **Neighbourhood** 매개 변수(parameter)를 사용하여 선택한 Neighbourhood(지역)를 기반으로 데이터를 필터링합니다.

   ![매개 변수가 선택된 대시보드 스크린샷.](./Images/dashboard-parameters.png)

9. **재설정(Reset)** 을 선택하여 선택된 매개 변수(parameter) 필터를 제거합니다.

## 페이지 추가

대시보드는 현재 단일 페이지로 구성되어 있습니다. 더 많은 데이터를 제공하기 위해 페이지를 추가할 수 있습니다.

1. 대시보드 왼쪽에서 **페이지(Pages)** 창을 확장하고 **+ 페이지 추가(+ Add page)** 를 선택합니다.
2. 새 페이지 이름을 **Page 2**로 지정합니다. 그런 다음 해당 페이지를 선택합니다.
3. 새 페이지에서 **+ 타일 추가(+ Add tile)** 를 선택합니다.
4. 새 타일의 쿼리 편집기에 다음 쿼리를 입력합니다.

    ```kql
    base_bike_data
    | project Neighbourhood, latest_observation
    | order by latest_observation desc
    ```

5. 변경 사항을 적용합니다. 그런 다음 타일의 크기를 조정하여 대시보드 높이를 채우도록 합니다.

   ![두 페이지가 있는 대시보드 스크린샷](./Images/dashboard-page-2.png)

## 자동 새로 고침 구성

사용자는 대시보드를 수동으로 새로 고칠 수 있지만, 설정된 간격으로 데이터를 자동으로 새로 고치도록 하는 것이 유용할 수 있습니다.

1. 대시보드 도구 모음의 **관리(Manage)** 탭에서 **자동 새로 고침(Auto refresh)** 을 선택합니다.
2. **자동 새로 고침(Auto refresh)** 창에서 다음 설정을 구성합니다.
    - **사용(Enabled)**: *선택됨*
    - **최소 시간 간격(Minimum time interval)**: 모든 새로 고침 간격 허용
    - **기본 새로 고침 빈도(Default refresh rate)**: 30분
3. 자동 새로 고침 설정을 적용합니다.

## 대시보드 저장 및 공유

이제 유용한 대시보드가 있으므로 이를 저장하고 다른 사용자와 공유할 수 있습니다.

1. 대시보드 도구 모음에서 **저장(Save)** 을 선택합니다.
2. 대시보드가 저장되면 **공유(Share)** 를 선택합니다.
3. **공유(Share)** 대화 상자에서 **링크 복사(Copy link)** 를 선택하고 대시보드 링크를 클립보드에 복사합니다.
4. 새 브라우저 탭을 열고 복사한 링크를 붙여넣어 공유 대시보드로 이동합니다. 메시지가 표시되면 Fabric 자격 증명으로 다시 로그인합니다.
5. 대시보드를 탐색하고, 이를 사용하여 도시 전역의 자전거 및 빈 자전거 도크에 대한 최신 정보를 확인합니다.

## 리소스 정리

대시보드 탐색을 마쳤으면 이 실습을 위해 생성한 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 바에서 작업 영역의 **아이콘**을 선택합니다.
2. 도구 모음에서 **작업 영역 설정(Workspace settings)** 을 선택합니다.
3. **일반(General)** 섹션에서 **이 작업 영역 제거(Remove this workspace)** 를 선택합니다.