---
lab:
    title: 'Monitor Fabric activity in the monitoring hub'
    module: 'Monitoring Fabric'
---

# Monitor Fabric activity in the monitoring hub

Microsoft Fabric의 *Monitoring Hub*는 활동을 모니터링할 수 있는 중앙 집중식 공간을 제공합니다. Monitoring Hub를 사용하여 볼 권한이 있는 항목과 관련된 이벤트를 검토할 수 있습니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)에 액세스해야 합니다.

## 작업 영역 만들기

Fabric에서 데이터를 작업하기 전에 Fabric Capacity가 활성화된 테넌트에서 작업 영역을 만듭니다.

1.  브라우저에서 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric-developer)(`https://app.fabric.microsoft.com/home?experience=fabric-developer`)로 이동한 후 Fabric 자격 증명으로 로그인하세요.
2.  왼쪽의 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 &#128455;와 유사합니다).
3.  선택한 이름으로 새 작업 영역을 만들고, **고급** 섹션에서 Fabric Capacity(*평가판*, *프리미엄* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택합니다.
4.  새 작업 영역이 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-workspace.png)

## Lakehouse 만들기

작업 영역이 준비되었으므로 이제 데이터용 Lakehouse를 만들 차례입니다.

1.  왼쪽의 메뉴 모음에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *데이터 엔지니어링* 섹션에서 **Lakehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다. "Lakehouse 스키마(공개 미리 보기)" 옵션이 비활성화되어 있는지 확인합니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되어 있지 않으면 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    약 1분 후에 새 Lakehouse가 생성됩니다.

    ![Screenshot of a new lakehouse.](./Images/new-lakehouse.png)

2.  새 Lakehouse를 확인하고, 왼쪽의 **Lakehouse 탐색기** 창을 통해 Lakehouse의 테이블과 파일을 탐색할 수 있다는 점을 확인합니다.

    현재 Lakehouse에는 테이블이나 파일이 없습니다.

## Dataflow 만들고 모니터링

Microsoft Fabric에서 Dataflow(Gen2)를 사용하여 다양한 원본에서 데이터를 수집할 수 있습니다. 이 실습에서는 Dataflow를 사용하여 CSV 파일에서 데이터를 가져오고 Lakehouse의 테이블로 로드합니다.

1.  Lakehouse의 **홈** 페이지에서 **데이터 가져오기** 메뉴에서 **새 Dataflow Gen2**를 선택합니다.
2.  새 Dataflow 이름을 `Get Product Data`로 지정하고 **만들기**를 선택합니다.

    ![Screenshot of a new dataflow.](./Images/new-data-flow.png)

3.  Dataflow 디자이너에서 **텍스트/CSV 파일에서 가져오기**를 선택합니다. 그런 다음 데이터 가져오기 마법사를 완료하여 `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/products.csv`에 연결하고 익명 인증을 사용하여 데이터 연결을 만듭니다. 마법사를 완료하면 다음과 같이 Dataflow 디자이너에 데이터 미리 보기가 표시됩니다.

    ![Screenshot of a dataflow query.](./Images/data-flow-query.png)

4.  Dataflow를 게시합니다.
5.  왼쪽의 탐색 모음에서 **모니터**를 선택하여 Monitoring Hub를 확인하고 Dataflow가 진행 중인지 확인합니다(표시되지 않으면 표시될 때까지 보기를 새로 고치세요).

    ![Screenshot of the monitoring hub with a dataflow in-progress.](./Images/monitor-dataflow.png)

6.  몇 초 기다렸다가 페이지를 새로 고쳐 Dataflow의 상태가 **성공**으로 바뀔 때까지 기다립니다.
7.  탐색 창에서 Lakehouse를 선택합니다. 그런 다음 **테이블** 폴더를 확장하여 **products**라는 테이블이 Dataflow에 의해 생성되고 로드되었는지 확인합니다(**테이블** 폴더를 새로 고쳐야 할 수도 있습니다).

    ![Screenshot of the products table in the lakehouse page.](./Images/products-table.png)

## Spark Notebook 만들고 모니터링

Microsoft Fabric에서 Notebook을 사용하여 Spark 코드를 실행할 수 있습니다.

1.  왼쪽의 메뉴 모음에서 **만들기**를 선택합니다. *새로 만들기* 페이지의 *데이터 엔지니어링* 섹션에서 **Notebook**을 선택합니다.

    **Notebook 1**이라는 새 Notebook이 생성되고 열립니다.

    ![Screenshot of a new notebook.](./Images/new-notebook.png)

2.  Notebook의 왼쪽 상단에서 **Notebook 1**을 선택하여 세부 정보를 확인하고 이름을 `Query Products`로 변경합니다.
3.  Notebook 편집기에서 **탐색기** 창의 **데이터 항목 추가**를 선택한 다음 **기존 데이터 원본**을 선택합니다.
4.  이전에 만든 Lakehouse를 추가합니다.
5.  Lakehouse 항목을 확장하여 **products** 테이블에 도달합니다.
6.  **products** 테이블의 **...** 메뉴에서 **데이터 로드** > **Spark**를 선택합니다. 그러면 여기에 표시된 것처럼 새 코드 셀이 Notebook에 추가됩니다.

    ![Screenshot of a notebook with code to query a table.](./Images/load-spark.png)

7.  **&#9655; 모두 실행** 단추를 사용하여 Notebook의 모든 셀을 실행합니다. Spark 세션을 시작하는 데 잠시 시간이 걸리며, 그런 다음 코드 셀 아래에 쿼리 결과가 표시됩니다.

    ![Screenshot of a notebook with query results.](./Images/notebook-output.png)

8.  도구 모음에서 **&#9723;**(*세션 중지*) 단추를 사용하여 Spark 세션을 중지합니다.
9.  탐색 모음에서 **모니터**를 선택하여 Monitoring Hub를 확인하고 Notebook 활동이 나열되는지 확인합니다.

    ![Screenshot of the monitoring hub with a notebook activity.](./Images/monitor-notebook.png)

## 항목 기록 모니터링

작업 영역의 일부 항목은 여러 번 실행될 수 있습니다. Monitoring Hub를 사용하여 실행 기록을 확인할 수 있습니다.

1.  탐색 모음에서 작업 영역 페이지로 돌아갑니다. 그런 다음 **&#8635;**(*지금 새로 고침*) 단추를 사용하여 **Get Product Data** Dataflow를 다시 실행합니다.
2.  탐색 창에서 **모니터** 페이지를 선택하여 Monitoring Hub를 확인하고 Dataflow가 진행 중인지 확인합니다.
3.  **Get Product Data** Dataflow의 **...** 메뉴에서 **실행 기록**을 선택하여 Dataflow의 실행 기록을 확인합니다.

    ![Screenshot of the monitoring hub historical runs view.](./Images/historical-runs.png)

4.  실행 기록 중 하나의 **...** 메뉴에서 **세부 정보 보기**를 선택하여 실행 세부 정보를 확인합니다.
5.  **세부 정보** 창을 닫고 **기본 보기로 돌아가기** 단추를 사용하여 Monitoring Hub 기본 페이지로 돌아갑니다.

## Monitoring Hub 보기 사용자 지정

이 실습에서는 몇 가지 활동만 실행했으므로 Monitoring Hub에서 이벤트를 찾기가 매우 쉬울 것입니다. 그러나 실제 환경에서는 많은 이벤트를 검색해야 할 수 있습니다. 필터 및 기타 보기 사용자 지정을 사용하면 더 쉽게 찾을 수 있습니다.

1.  Monitoring Hub에서 **필터** 단추를 사용하여 다음 필터를 적용합니다.
    -   **상태**: 성공
    -   **항목 유형**: Dataflow Gen2

    필터를 적용하면 성공적인 Dataflow 실행만 나열됩니다.

    ![Screenshot of the monitoring hub with a filter applied.](./Images/monitor-filter.png)

2.  **열 옵션** 단추를 사용하여 보기에 다음 열을 포함합니다(변경 내용을 적용하려면 **적용** 단추를 사용하세요).
    -   활동 이름
    -   상태
    -   항목 유형
    -   시작 시간
    -   제출자
    -   위치
    -   종료 시간
    -   기간
    -   새로 고침 유형

    모든 열을 보려면 가로로 스크롤해야 할 수도 있습니다.

    ![Screenshot of the monitoring hub with custom columns.](./Images/monitor-columns.png)

## 리소스 정리

이 실습에서는 Lakehouse, Dataflow 및 Spark Notebook을 만들었으며 Monitoring Hub를 사용하여 항목 활동을 확인했습니다.

Lakehouse 탐색을 완료했다면 이 실습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1.  왼쪽 모음에서 작업 영역 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
2.  도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3.  **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
