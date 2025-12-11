---
lab:
    title: 'Secure data access in Microsoft Fabric'
    module: 'Secure data access in Microsoft Fabric'
---

# Secure data access in Microsoft Fabric
```

# Microsoft Fabric에서 데이터 액세스 보안 설정

Microsoft Fabric은 데이터 액세스를 관리하기 위한 다계층 보안 모델을 제공합니다. 보안은 전체 Workspace, 개별 Item 또는 각 Fabric 엔진 내의 세분화된 권한을 통해 설정할 수 있습니다. 이 실습에서는 Workspace 및 Item 액세스 제어와 OneLake 데이터 액세스 역할을 사용하여 데이터를 보호합니다.

> **참고**: 이 랩의 실습을 완료하려면 두 명의 사용자가 필요합니다. 한 사용자에게는 Workspace Admin 역할이 할당되어야 하고, 다른 사용자에게는 Workspace Viewer 역할이 있어야 합니다. Workspace에 역할을 할당하는 방법은 [Workspace에 액세스 권한 부여](https://learn.microsoft.com/fabric/get-started/give-access-workspaces)를 참조하세요. 동일한 조직 내에 두 번째 계정이 없는 경우에도 Workspace Admin으로 실습을 수행하고, Workspace Viewer 계정으로 수행하는 단계는 건너뛸 수 있으며, 실습 스크린샷을 참조하여 Workspace Viewer 계정이 액세스할 수 있는 내용을 확인할 수 있습니다.

이 랩은 완료하는 데 약 **45**분이 소요됩니다.

## Workspace 생성

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace를 생성하세요.

1.  브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`의 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 유사합니다.)를 선택합니다.
3.  원하는 이름으로 새 Workspace를 생성하고, Fabric Capacity를 포함하는 라이선싱 모드 (*Trial*, *Premium*, 또는 *Fabric*)를 선택합니다.
4.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Screenshot of an empty workspace in Fabric.](./Images/new-empty-workspace.png)

> **참고**: Workspace를 생성하면 자동으로 Workspace Admin 역할의 구성원이 됩니다.

## Data Warehouse 생성

다음으로, 생성한 Workspace에 Data Warehouse를 생성합니다:

1.  **+ New Item**을 클릭하세요. *새 Item* 페이지의 *데이터 저장* 섹션 아래에서 **Sample warehouse**를 선택하고 원하는 이름으로 새 Data Warehouse를 생성합니다.

    1분 정도 지나면 새 Warehouse가 생성됩니다:

    ![Screenshot of a new warehouse.](./Images/new-sample-warehouse.png)

## Lakehouse 생성

다음으로, 생성한 Workspace에 Lakehouse를 생성합니다.

1.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 🗇와 유사합니다.)를 선택합니다.
2.  생성한 Workspace를 선택합니다.
3.  Workspace에서 **+ New Item** 버튼을 선택한 다음 **Lakehouse**를 선택합니다. 원하는 이름으로 새 Lakehouse를 생성합니다. "Lakehouse schemas (Public Preview)" 옵션이 비활성화되어 있는지 확인하세요.

    1분 정도 지나면 새 Lakehouse가 생성됩니다:

    ![Screenshot of a new lakehouse in Fabric.](./Images/new-sample-lakehouse.png)

4.  **Start with sample data** 타일을 선택한 다음 **Public holidays** 샘플을 선택합니다. 1분 정도 지나면 Lakehouse에 데이터가 채워집니다.

## Workspace 액세스 제어 적용

Workspace 역할은 Workspace 및 그 안에 있는 콘텐츠에 대한 액세스를 제어하는 데 사용됩니다. Workspace 역할은 사용자가 Workspace의 모든 Item을 보거나, Workspace 액세스를 관리하거나, 새 Fabric Item을 생성하거나, Workspace에서 콘텐츠를 보거나, 수정하거나, 공유할 수 있는 특정 권한이 필요할 때 할당될 수 있습니다.

이 실습에서는 Workspace 역할에 사용자를 추가하고, 권한을 적용하며, 각 권한 세트가 적용될 때 무엇을 볼 수 있는지 확인합니다. 두 개의 브라우저를 열고 다른 사용자로 로그인합니다. 한 브라우저에서는 **Workspace Admin**으로, 다른 브라우저에서는 두 번째로 권한이 적은 사용자로 로그인합니다. 한 브라우저에서는 Workspace Admin이 두 번째 사용자의 권한을 변경하고, 다른 브라우저에서는 권한 변경의 영향을 확인할 수 있습니다.

1.  왼쪽 메뉴 바에서 **Workspaces** (아이콘은 &#128455;와 유사합니다.)를 선택합니다.
2.  다음으로, 생성한 Workspace를 선택합니다.
3.  화면 상단의 **Manage access**를 선택합니다.

> **참고**: Workspace를 생성했기 때문에 **Workspace Admin** 역할의 구성원인 로그인한 사용자를 볼 수 있습니다. 아직 다른 사용자에게 Workspace에 대한 액세스 권한이 할당되지 않았습니다.

4.  다음으로, Workspace에 대한 권한이 없는 사용자가 무엇을 볼 수 있는지 확인합니다. 브라우저에서 InPrivate 창을 엽니다. Microsoft Edge 브라우저에서는 오른쪽 상단 모서리에 있는 줄임표(…)를 선택하고 **New InPrivate Window**를 선택합니다.
5.  https://app.fabric.microsoft.com을 입력하고 테스트를 위해 사용하는 두 번째 사용자로 로그인합니다.
6.  화면 왼쪽 하단에서 **Microsoft Fabric**을 선택한 다음 **Data Warehouse**를 선택합니다. 다음으로 **Workspaces** (아이콘은 &#128455;와 유사합니다.)를 선택합니다.

> **참고:** 두 번째 사용자는 Workspace에 대한 액세스 권한이 없으므로 볼 수 없습니다.

7.  다음으로, 두 번째 사용자에게 **Workspace Viewer** 역할을 할당하고 해당 역할이 Workspace의 Warehouse에 읽기 액세스 권한을 부여하는지 확인합니다.
8.  Workspace Admin으로 로그인한 브라우저 창으로 돌아갑니다. 생성한 Workspace를 보여주는 페이지에 계속 있는지 확인하세요. 페이지 하단에는 새 Workspace Item과 샘플 Warehouse 및 Lakehouse가 나열되어 있어야 합니다.
9.  화면 오른쪽 상단의 **Manage access**를 선택합니다.
10. **Add people or groups**를 선택합니다. 테스트 중인 두 번째 사용자의 이메일을 입력합니다. **Add**를 선택하여 사용자에게 Workspace **Viewer** 역할을 할당합니다.
11. 두 번째 사용자로 로그인한 InPrivate 브라우저 창으로 돌아가서 브라우저의 새로고침 버튼을 선택하여 두 번째 사용자에게 할당된 세션 권한을 새로고침합니다.
12. 왼쪽 메뉴 바의 **Workspaces** 아이콘 (아이콘은 &#128455;와 유사합니다.)을 선택하고 Workspace Admin 사용자로 생성한 Workspace 이름을 선택합니다. 두 번째 사용자는 **Workspace Viewer** 역할이 할당되었기 때문에 이제 Workspace의 모든 Item을 볼 수 있습니다.

    ![Screenshot of workspace items in Fabric.](./Images/workspace-viewer-view.png)

13. Warehouse를 선택하고 엽니다.
14. **Date** 테이블을 선택하고 행이 로드될 때까지 기다립니다. Workspace Viewer 역할의 구성원으로서 Warehouse의 테이블에 대한 CONNECT 및 ReadData 권한을 가지고 있으므로 행을 볼 수 있습니다. Workspace Viewer 역할에 부여된 권한에 대한 자세한 정보는 [Workspace 역할](https://learn.microsoft.com/en-us/fabric/data-warehouse/workspace-roles)을 참조하세요.
15. 다음으로, 왼쪽 메뉴 바에서 **Workspaces** 아이콘을 선택한 다음 Lakehouse를 선택합니다.
16. Lakehouse가 열리면 화면 오른쪽 상단 모서리에 **Lakehouse**라고 표시된 드롭다운 상자를 클릭하고 **SQL analytics endpoint**를 선택합니다.
17. **publicholidays** 테이블을 선택하고 데이터가 표시될 때까지 기다립니다. 사용자가 SQL analytics endpoint에 대한 읽기 권한을 부여하는 Workspace Viewer 역할의 구성원이므로 Lakehouse 테이블의 데이터는 SQL analytics endpoint에서 읽을 수 있습니다.

## Item 액세스 제어 적용

Item 권한은 Workspace 내의 Warehouse, Lakehouse 및 Semantic models와 같은 개별 Fabric Item에 대한 액세스를 제어합니다. 이 실습에서는 이전 실습에서 적용된 **Workspace Viewer** 권한을 제거한 다음, Warehouse에 Item 수준 권한을 적용하여 권한이 적은 사용자가 Warehouse 데이터만 볼 수 있고 Lakehouse 데이터는 볼 수 없도록 합니다.

1.  Workspace Admin으로 로그인한 브라우저 창으로 돌아갑니다. 왼쪽 탐색 창에서 **Workspaces**를 선택합니다.
2.  생성한 Workspace를 선택하여 엽니다.
3.  화면 상단의 **Manage access**를 선택합니다.
4.  두 번째 사용자 이름 아래의 **Viewer**라는 단어를 선택합니다. 나타나는 메뉴에서 **Remove**를 선택합니다.

    ![Screenshot of workspace access dropdown in Fabric.](./Images/workspace-access.png)

5.  **Manage access** 섹션을 닫습니다.
6.  Workspace에서 Warehouse 이름 위로 마우스를 가져가면 줄임표(**...**)가 나타납니다. 줄임표를 선택하고 **Manage permissions**를 선택합니다.
7.  **Add user**를 선택하고 두 번째 사용자의 이름을 입력합니다.
8.  나타나는 상자에서 **Additional permissions** 아래의 **Read all data using SQL (ReadData)**를 선택하고 다른 모든 상자는 선택을 해제합니다.

    ![Screenshot of warehouse permissions being granted in Fabric.](./Images/grant-warehouse-access.png)

9.  **Grant**를 선택합니다.
10. 두 번째 사용자로 로그인한 브라우저 창으로 돌아갑니다. 브라우저 보기를 새로고침합니다.
11. 두 번째 사용자는 더 이상 Workspace에 대한 액세스 권한이 없으며 대신 Warehouse에만 액세스할 수 있습니다. 왼쪽 탐색 창에서 Workspace를 탐색하여 Warehouse를 찾을 수 없습니다. 왼쪽 탐색 메뉴에서 **OneLake catalog**를 선택하여 Warehouse를 찾으세요:

    ![Screenshot of OneLake catalog.](./Images/onelake-catalog.png)

12. Warehouse를 선택합니다. 나타나는 화면에서 상단 메뉴 바에서 **Open**을 선택합니다.
13. Warehouse 보기가 나타나면 **Date** 테이블을 선택하여 테이블 데이터를 봅니다. Warehouse에 Item 권한을 사용하여 ReadData 권한이 적용되었기 때문에 사용자가 여전히 Warehouse에 대한 읽기 액세스 권한을 가지고 있으므로 행을 볼 수 있습니다.

## Lakehouse에서 OneLake 데이터 액세스 역할 적용

OneLake 데이터 액세스 역할은 Lakehouse 내에서 사용자 지정 역할을 생성하고, 지정한 폴더에 읽기 권한을 부여할 수 있도록 합니다. OneLake 데이터 액세스 역할은 현재 미리 보기(Preview) 기능입니다.

이 실습에서는 Item 권한을 할당하고 OneLake 데이터 액세스 역할을 생성하며, 이들이 Lakehouse의 데이터에 대한 액세스를 제한하기 위해 어떻게 함께 작동하는지 실험합니다.

1.  두 번째 사용자로 로그인한 브라우저에 머무르세요.
2.  왼쪽 탐색 바에서 **OneLake catalog**를 선택합니다. 두 번째 사용자는 Lakehouse를 볼 수 없습니다.
3.  Workspace Admin으로 로그인한 브라우저로 돌아갑니다.
4.  왼쪽 메뉴에서 **Workspaces**를 선택하고 Workspace를 선택합니다. Lakehouse 이름 위로 마우스를 가져갑니다.
5.  줄임표(**...**) 오른쪽에 있는 줄임표를 선택하고 **Manage permissions**를 선택합니다.

    ![Screenshot of setting permissions on a lakehouse in Fabric.](./Images/lakehouse-manage-permissions.png)

6.  나타나는 화면에서 **Add user**를 선택합니다.
7.  두 번째 사용자에게 Lakehouse를 할당하고 **Grant People Access** 창의 체크박스 중 어떤 것도 선택되어 있지 않은지 확인합니다.

    ![Screenshot of the grant access lakehouse window in Fabric.](./Images/grant-people-access-window.png)

8.  **Grant**를 선택합니다. 두 번째 사용자는 이제 Lakehouse에 대한 읽기 권한을 가집니다. 읽기 권한은 사용자에게 Lakehouse의 메타데이터만 볼 수 있도록 허용하며, 기본 데이터는 볼 수 없습니다. 다음으로 이를 검증합니다.
9.  두 번째 사용자로 로그인한 브라우저로 돌아갑니다. 브라우저를 새로고침합니다.
10. 왼쪽 탐색 창에서 **OneLake**를 선택합니다.
11. Lakehouse를 선택하고 엽니다.
12. 상단 메뉴 바에서 **Open**을 선택합니다. 읽기 권한이 부여되었음에도 불구하고 테이블이나 파일을 확장할 수 없습니다.

    ![Screenshot of lakehouse unable to load data.](./Images/lakehouse-metadata-only-access.png)

13. 다음으로, OneLake 데이터 액세스 권한을 사용하여 두 번째 사용자에게 특정 폴더에 대한 액세스 권한을 부여합니다.
14. Workspace 관리자로 로그인한 브라우저로 돌아갑니다.
15. 왼쪽 탐색 바에서 **Workspaces**를 선택합니다.
16. Workspace 이름을 선택합니다.
17. Lakehouse를 선택합니다.
18. Lakehouse가 열리면 상단 메뉴 바에서 **Manage OneLake data access**를 선택하고 **Continue** 버튼을 선택하여 기능을 활성화합니다.

    ![Screenshot of the Manage OneLake data access (preview) feature on the menu bar in Fabric.](./Images/manage-onelake-roles.png)

19. 나타나는 **OneLake security** 화면에서 **+ New**를 선택합니다.

    ![Screenshot of the new role functionality in the manage OneLake data access feature.](./Images/create-onelake-role.png)

20. **publicholidays**라는 새 역할을 생성한 다음, **Selected data**와 **Browse Lakehouse**를 선택합니다. 새 창에서 publicholidays 테이블을 선택합니다.
21. **Add members to your role** 필드에 두 번째 사용자를 추가합니다.
22. **Preview role** 섹션에서 **publicholidays** 테이블이 읽기 권한과 함께 **Data preview** 탭에 추가되었는지, 그리고 두 번째 사용자가 **Members preview** 탭에 추가되었는지 확인합니다. **Create role**을 선택합니다.
23. 두 번째 사용자로 로그인한 브라우저로 돌아갑니다. Lakehouse가 열려 있는 페이지에 계속 있는지 확인하세요. 브라우저를 새로고침합니다.
24. **publicholidays** 테이블을 선택하고 데이터가 로드될 때까지 기다립니다. 사용자가 사용자 지정 OneLake 데이터 액세스 역할에 할당되었기 때문에 publicholidays 테이블의 데이터만 액세스할 수 있습니다. 이 역할은 publicholidays 테이블의 데이터만 볼 수 있도록 허용하며, 다른 테이블, 파일 또는 폴더의 데이터는 볼 수 없습니다.

    ![Screenshot of the lakehouse with table access.](./Images/lakehouse-table-access.png)

## 리소스 정리

이 실습에서는 Workspace 액세스 제어, Item 액세스 제어 및 OneLake 데이터 액세스 역할을 사용하여 데이터를 보호했습니다.

1.  왼쪽 탐색 바에서 Workspace 아이콘을 선택하여 포함된 모든 Item을 봅니다.
2.  상단 도구 모음의 메뉴에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.
