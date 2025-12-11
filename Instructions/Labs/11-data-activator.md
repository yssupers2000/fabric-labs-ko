---
lab:
    title: 'Use Activator in Microsoft Fabric'
    module: 'Use Activator in Microsoft Fabric'
---
# Use Activator in Fabric

Microsoft Fabric의 Activator는 데이터에서 발생하는 상황에 따라 작업을 수행합니다. Activator를 통해 데이터를 모니터링하고 데이터 변경 사항에 반응하는 Trigger를 생성할 수 있습니다.

이 Lab을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 시나리오

이 시나리오에서 여러분은 다양한 제품을 판매하고 배송하는 회사의 데이터 분석가입니다. Redmond 시로의 모든 배송 및 판매 데이터는 여러분이 담당하고 있습니다. 배송 중인 Package를 모니터링하는 Alert Rule을 생성하려고 합니다. 여러분이 배송하는 제품 카테고리 중 하나는 운송 중 특정 온도로 냉장 보관해야 하는 의료 처방약입니다. 처방약이 포함된 Package의 온도가 특정 임계값보다 높거나 낮으면 배송 부서에 이메일을 보내는 Alert를 생성하려고 합니다. 이상적인 온도는 33도에서 41도 사이여야 합니다. Activator Event에 이미 유사한 Trigger가 포함되어 있으므로, Redmond 시로 배송되는 Package에 대한 Trigger를 특별히 생성합니다. 시작해 보겠습니다!

## Workspace 생성

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace를 생성하세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric-developer`의 [Microsoft Fabric 홈 페이지](https://learn.microsoft.com/fabric/get-started/fabric-trial)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 유사하게 생겼습니다)를 선택합니다.
3.  원하는 이름으로 새 Workspace를 생성하고, Fabric Capacity를 포함하는 라이선싱 모드(*Trial*, *Premium* 또는 *Fabric*)를 선택합니다.
4.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Fabric의 빈 Workspace 스크린샷.](./Images/new-empty-workspace.png)

이 Lab에서는 Fabric의 Activator를 사용하여 데이터 변경 사항을 기반으로 Trigger를 생성합니다. Fabric의 Activator는 Activator의 기능을 탐색하는 데 사용할 수 있는 Sample Dataset을 편리하게 제공합니다. 이 Sample Data를 사용하여 실시간 데이터 Streaming을 분석하고 Condition이 충족될 때 이메일을 보내는 Trigger를 생성합니다.

> **참고**: Activator Sample Process는 백그라운드에서 일부 무작위 데이터를 생성합니다. Condition과 Filter가 복잡할수록 Trigger하는 데 더 많은 시간이 소요됩니다. Graph에 데이터가 표시되지 않으면 몇 분 정도 기다렸다가 페이지를 새로 고치세요. 하지만, Lab을 계속 진행하기 위해 Graph에 데이터가 표시될 때까지 기다릴 필요는 없습니다.

## Activator 생성

다음으로, 생성한 Workspace에서 Activator를 생성합니다.

1.  왼쪽 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Real-Time Intelligence* 섹션에서 **Activator**를 선택합니다.

    >**참고**: **Create** 옵션이 사이드바에 고정되어 있지 않은 경우, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    약 1분 후에 새 Activator가 생성됩니다.

    ![Activator 홈 화면 스크린샷.](./Images/activator-home-screen.png)

    실제 Production 환경에서는 고유한 데이터를 사용합니다. 하지만 이 Lab에서는 Activator에서 제공하는 Sample Data를 사용합니다.

2.  **Try sample** Tile을 선택하여 Activator에 Sample Data를 채웁니다.

    기본적으로 Activator는 *Activator YYYY-MM-DD hh:mm:ss*라는 이름으로 생성됩니다. Workspace에 여러 Activator가 있을 수 있으므로 기본 이름을 더 설명적인 이름으로 변경해야 합니다.

3.  왼쪽 상단 모서리에 있는 현재 Activator 이름 옆의 드롭다운을 선택하고, 예시를 위해 이름을 ***Contoso Shipping Activator***로 변경합니다.

    ![Activator 홈 화면 스크린샷.](./Images/activator-reflex-home-screen.png)

이제 Activator Project가 생성되었으며, 해당 Object, Property, Rule을 탐색할 수 있습니다.

## Activator 홈 화면 익히기

이 Sample이 기반으로 하는 Eventstream Data를 살펴보겠습니다.

1.  **Explorer** Pane에서 아래로 스크롤하여 **Package delivery events** Stream을 선택합니다.

    이 Event들은 배송 중인 Package의 실시간 상태를 보여줍니다.

    ![Event 상세 라이브 Table 스크린샷.](./Images/activator-event-details.png)

2.  **Event details** 라이브 Table에서 데이터를 검토합니다. 각 Data Point에는 수신 Event에 대한 정보가 포함되어 있습니다. 모든 내용을 보려면 스크롤해야 할 수 있습니다.

    **Explorer** Pane은 Eventstream의 데이터를 사용하는 Object를 표시합니다. 이러한 Object에는 Rule을 생성할 수 있는 Property가 있습니다. 이 예시에서는 **Package delivery events** Eventstream에서 생성된 Object는 **Package**입니다.

3.  **Explorer** Pane의 **Temperature** Property 아래에서 **Too hot for medicine** Rule을 선택합니다.
4.  **Definition** Pane에서 Rule이 작동하는 방식을 검토합니다. **Monitor** 섹션에서는 **Temperature** Property가 모니터링되는 Attribute로 선택됩니다. 온도 값은 Eventstream에서 이전에 보았던 **Event details** Table의 Temperature Column에서 가져옵니다.

    ![온도 Rule 스크린샷.](./Images/activator-temperature-rule.png)

5.  **Condition** 섹션에서 섭씨 20도보다 높은 온도를 모니터링하는 Rule Condition을 볼 수 있습니다.
6.  **Property filter** 섹션에서 의료 처방약이 포함된 Package에만 Rule이 적용되도록 하는 Customized Filter를 볼 수 있습니다. Eventstream Table에서 Rule은 **SpecialCare**라는 Column을 확인하며, 여기서는 *Special care contents* Property로 표시됩니다. *Special care contents* Property에서 일부 Package는 Medicine 값을 가집니다.
7.  마지막으로 **Action** 섹션이 있습니다. Rule은 Condition이 충족되면 Teams 메시지를 보내도록 설정되어 있습니다. 대신 이메일을 보내도록 설정할 수도 있습니다.
8.  선호하는 Action Type을 선택하고, 본인이 수신자인지 확인한 다음, **Send me a test action**을 선택합니다. **Message** 필드에 설정된 메시지와 Activation Time 및 Package ID와 같은 Trigger에 대한 세부 정보가 포함된 메시지를 수신하게 됩니다.

## Object 생성

실제 시나리오에서는 Activator Sample에 이미 *Package*라는 Object가 포함되어 있으므로 이 Eventstream에 대한 새 Object를 생성할 필요가 없을 수도 있습니다. 하지만 이 Lab에서는 Object 생성 방법을 시연하기 위해 새 Object를 생성합니다. *Redmond Packages*라는 새 Object를 생성해 보겠습니다.

1.  **Package delivery events** Eventstream을 선택한 다음, Ribbon에서 **New object**를 선택합니다.

2.  오른쪽에 있는 **Build object** Pane에서 다음 값을 입력합니다.
    -   **Object name**: `Redmond Packages`
    -   **Unique Identifier**: **PackageId**
    -   **Properties**: **City**, **ColdChainType**, **SpecialCare**, **Temperature**

3.  **Create**를 선택합니다.

    ![Activator Build object Pane 스크린샷.](./Images/activator-build-object.png)

**Explorer** Pane에 **Redmond Packages**라는 새 Object가 추가되었습니다. 이제 Rule을 생성할 차례입니다.

## Rule 생성

Rule이 수행할 작업을 검토해 보겠습니다. *처방약이 포함된 Package의 온도가 특정 임계값보다 높거나 낮으면 배송 부서에 이메일을 보내는 Alert Rule을 생성하려고 합니다. 이상적인 온도는 20도 미만이어야 합니다. Package Object에 이미 유사한 Rule이 포함되어 있으므로 Redmond 시로 배송되는 Package에 대한 Rule을 특별히 생성합니다.*

1.  *Redmond Packages* Object 내의 **Temperature** Property를 선택하고, 아직 선택되지 않았다면 Ribbon에서 **New Rule** 버튼을 선택합니다.
2.  **Create rule** Pane에서 다음 값을 입력합니다.
    -   **Condition**: Increases above
    -   **Value**: `20`
    -   **Occurrence**: Every time the condition is met
    -   **Action**: Send me an email

3.  **Create**를 선택합니다.
4.  기본 이름 *Temperature alert*로 새 Rule이 생성됩니다. 중앙 Pane에서 Rule 이름 옆의 연필 아이콘을 선택하여 이름을 ***Medicine temp out of range***로 변경합니다.

    ![Activator 새 Rule 스크린샷.](./Images/activator-new-rule.png)

    지금까지 Rule이 실행될 Property와 Condition을 정의했지만, 필요한 모든 Parameter가 아직 포함되지 않았습니다. Trigger가 **Redmond** *City*와 **Medicine** *Special care* 유형에 대해서만 실행되도록 해야 합니다. 해당 Condition에 대한 Filter를 몇 개 추가해 보겠습니다.

5.  **Definition** Pane에서 **Property filter** 섹션을 확장합니다.
6.  **Filter 1** 상자에서 Attribute를 **City**로, Operation을 **Is equal to**로 설정하고, 값을 **Redmond**로 선택합니다.
7.  **Add filter**를 선택한 다음, **SpecialCare** Attribute로 새 Filter를 추가하고, **Is equal to**로 설정한 후 값을 **Medicine**으로 입력합니다.
8.  처방약이 냉장 보관되는지 확인하기 위해 Filter를 하나 더 추가해 보겠습니다. **Add filter** 버튼을 선택하고, ***ColdChainType*** Attribute를 **Is equal to**로 설정한 후 값을 **Refrigerated**로 입력합니다.

    ![Filter가 설정된 Activator Rule 스크린샷.](./Images/activator-rule-filters.png)

    거의 다 되었습니다! Trigger가 실행될 때 수행할 Action을 정의하기만 하면 됩니다. 이 경우 배송 부서에 이메일을 보내려고 합니다.

9.  **Action** 섹션에서 다음 값을 입력합니다.
    -   **Type**: Email
    -   **To**: 현재 사용자 계정이 기본적으로 선택되어야 하며, 이 Lab에는 적합합니다.
    -   **Subject**: *Redmond Medical Package outside acceptable temperature range*
    -   **Headline**: *Temperature too high*
    -   **Context**: Checkbox 목록에서 *Temperature* Property를 선택합니다.

    ![Activator Action 정의 스크린샷.](./Images/activator-define-action.png)

10. **Save and start**를 선택합니다.

    이제 Activator에서 Rule을 생성하고 시작했습니다. Rule은 매시간 여러 번 Trigger되어야 합니다.

11. Rule이 작동하는 것을 확인하면 Ribbon의 **Stop** 버튼을 사용하여 끌 수 있습니다.

## 리소스 정리

이 실습에서는 Alert Rule이 있는 Activator를 생성했습니다. 이제 Activator 인터페이스와 Object, Property, Rule을 생성하는 방법에 익숙해졌을 것입니다.

Activator 탐색을 마쳤으면 이 실습을 위해 생성한 Workspace를 삭제할 수 있습니다.

1.  왼쪽 Navigation Bar에서 Workspace 아이콘을 선택하여 포함된 모든 Item을 확인합니다.
2.  상단 Toolbar의 메뉴에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.