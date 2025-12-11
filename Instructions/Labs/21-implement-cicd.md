---
lab:
    title: 'Implement deployment pipelines in Microsoft Fabric'
    module: 'Implement CI/CD in Microsoft Fabric'
---

# Microsoft Fabric에서 배포 Pipeline 구현

Microsoft Fabric의 배포 Pipeline은 개발, 테스트, 프로덕션과 같은 환경 간에 Fabric item의 콘텐츠에 대한 변경 사항을 복사하는 프로세스를 자동화할 수 있게 해줍니다. 최종 사용자에게 도달하기 전에 콘텐츠를 개발하고 테스트하는 데 배포 Pipeline을 사용할 수 있습니다. 이 실습에서는 배포 Pipeline을 생성하고, Pipeline에 stage를 할당합니다. 그런 다음 개발 workspace에서 일부 콘텐츠를 생성하고 배포 Pipeline을 사용하여 Development, Test 및 Production Pipeline stage 간에 배포합니다.

> **참고**: 이 실습을 완료하려면 Fabric workspace의 admin role 멤버여야 합니다. role을 할당하려면 [Microsoft Fabric의 workspace role](https://learn.microsoft.com/en-us/fabric/get-started/roles-workspaces)을 참조하세요.

이 실습은 완료하는 데 약 **20**분이 소요됩니다.

## workspace 생성

Fabric trial이 활성화된 세 개의 workspace를 생성합니다.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명(credentials)으로 로그인합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 유사합니다)를 선택합니다.
3.  Fabric Capacity(*Trial*, *Premium* 또는 *Fabric*)를 포함하는 라이선스 모드를 선택하여 Development라는 새 workspace를 생성합니다.
4.  단계 1과 2를 반복하여 Test 및 Production이라는 두 개의 workspace를 추가로 생성합니다. 귀하의 workspace는 Development, Test, Production입니다.
5.  왼쪽 메뉴 바에서 **Workspaces** 아이콘을 선택하고 Development, Test, Production이라는 세 개의 workspace가 있는지 확인합니다.

> **참고**: workspace에 고유한 이름을 입력하라는 메시지가 표시되면 Development, Test 또는 Production 단어 뒤에 하나 이상의 임의 숫자를 추가하세요.

## 배포 Pipeline 생성

다음으로, 배포 Pipeline을 생성합니다.

1.  왼쪽 메뉴 바에서 **Workspaces**를 선택합니다.
2.  **Deployment Pipelines**를 선택한 다음 **New pipeline**을 선택합니다.
3.  **새 배포 Pipeline 추가** 창에서 Pipeline에 고유한 이름을 지정하고 **다음**을 선택합니다.
4.  새 Pipeline 창에서 **만들기 및 계속**을 선택합니다.

## 배포 Pipeline의 stage에 workspace 할당

배포 Pipeline의 stage에 workspace를 할당합니다.

1.  왼쪽 메뉴 바에서 생성한 Pipeline을 선택합니다.
2.  나타나는 창에서 각 배포 stage의 **workspace 할당** 아래 옵션을 확장하고 해당 stage 이름과 일치하는 workspace 이름을 선택합니다.
3.  각 배포 stage에 대해 확인 표시 **할당**을 선택합니다.

  ![Screenshot of deployment pipeline.](./Images/deployment-pipeline.png)

## 콘텐츠 생성

아직 Fabric item이 workspace에 생성되지 않았습니다. 다음으로, 개발 workspace에서 Lakehouse를 생성합니다.

1.  왼쪽 메뉴 바에서 **Workspaces**를 선택합니다.
2.  **Development** workspace를 선택합니다.
3.  **새 Item**을 선택합니다.
4.  나타나는 창에서 **Lakehouse**를 선택하고 **새 Lakehouse 창**에서 Lakehouse 이름을 **LabLakehouse**로 지정합니다. "Lakehouse schemas (Public Preview)" 옵션이 비활성화되어 있는지 확인합니다.
5.  **생성**을 선택합니다.
6.  Lakehouse Explorer 창에서 **샘플 데이터로 시작**을 선택하여 새 Lakehouse에 데이터를 채웁니다.

  ![Screenshot of Lakehouse Explorer.](./Images/lakehouse-explorer.png)

7.  샘플 **NYCTaxi**를 선택합니다.
8.  왼쪽 메뉴 바에서 생성한 Pipeline을 선택합니다.
9.  **Development** stage를 선택하면 배포 Pipeline Canvas 아래에서 생성한 Lakehouse를 stage item으로 볼 수 있습니다. **Test** stage의 왼쪽 가장자리에는 원 안에 **X**가 있습니다. 이 **X**는 Development 및 Test stage가 동기화되지 않았음을 나타냅니다.
10. **Test** stage를 선택하면 배포 Pipeline Canvas 아래에서 생성한 Lakehouse가 원본(이 경우 **Development** stage를 의미)에만 stage item으로 있음을 볼 수 있습니다.

  ![Screenshot the deployment pipeline showing content mismatches between stages.](./Images/lab-pipeline-compare.png)

## stage 간에 콘텐츠 배포

**Development** stage에서 **Test** 및 **Production** stage로 Lakehouse를 배포합니다.
1.  배포 Pipeline Canvas에서 **Test** stage를 선택합니다.
2.  배포 Pipeline Canvas 아래에서 Lakehouse item 옆의 확인란(checkbox)을 선택합니다. 그런 다음 **배포** 버튼을 선택하여 Lakehouse를 현재 상태로 **Test** stage에 복사합니다.
3.  나타나는 **다음 stage로 배포** 창에서 **배포**를 선택합니다.
 이제 배포 Pipeline Canvas의 Production stage에 원 안에 X가 있습니다. Lakehouse는 Development 및 Test stage에는 존재하지만 Production stage에는 아직 존재하지 않습니다.
4.  배포 Canvas에서 **Production** stage를 선택합니다.
5.  배포 Pipeline Canvas 아래에서 Lakehouse item 옆의 확인란(checkbox)을 선택합니다. 그런 다음 **배포** 버튼을 선택하여 Lakehouse를 현재 상태로 **Production** stage에 복사합니다.
6.  나타나는 **다음 stage로 배포** 창에서 **배포**를 선택합니다. stage 사이의 녹색 확인 표시는 모든 stage가 동기화(in sync)되어 동일한 콘텐츠를 포함하고 있음을 나타냅니다.
7.  배포 Pipeline을 사용하여 stage 간에 배포하면 배포 stage에 해당하는 workspace의 콘텐츠도 업데이트됩니다. 이를 확인해 봅시다.
8.  왼쪽 메뉴 바에서 **Workspaces**를 선택합니다.
9.  **Test** workspace를 선택합니다. Lakehouse가 그곳으로 복사되었습니다.
10. 왼쪽 메뉴의 **Workspaces** 아이콘에서 **Production** workspace를 엽니다. Lakehouse가 Production workspace에도 복사되었습니다.

## 정리

이 실습에서는 배포 Pipeline을 생성하고 Pipeline에 stage를 할당했습니다. 그런 다음 개발 workspace에서 콘텐츠를 생성하고 배포 Pipeline을 사용하여 Pipeline stage 간에 배포했습니다.

*   왼쪽 탐색 바에서 **Deployment Pipelines**를 선택하고, Pipeline을 선택한 다음, 설정 메뉴에서 **이 Pipeline 삭제**를 선택하여 배포 Pipeline을 제거합니다.

![Screenshot of deployment pipeline, highlighting the Delete pipeline action.](./Images/delete-pipeline.png)

*   Pipeline을 삭제한 후, 각 workspace의 아이콘을 선택하여 포함된 모든 item을 확인합니다.
*   상단 도구 모음의 메뉴에서 **Workspace 설정**을 선택합니다.
*   **일반** 섹션에서 **이 workspace 제거**를 선택합니다.
