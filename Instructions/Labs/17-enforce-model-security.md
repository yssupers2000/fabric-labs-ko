---
lab:
    title: 'Enforce semantic model security'
    module: 'Enforce semantic model security'
---

# 시맨틱 모델 보안 적용

이 실습에서는 보안을 적용하기 위해 미리 개발된 데이터 모델을 업데이트합니다. 특히 Adventure Works 회사 영업사원은 할당된 영업 지역과 관련된 판매 데이터만 볼 수 있어야 합니다.

이 실습에서 다음 방법을 배웁니다.

- 정적 역할(static roles)을 생성합니다.
- 동적 역할(dynamic roles)을 생성합니다.
- 역할을 유효성 검사합니다.
- 보안 주체(security principals)를 시맨틱 모델 역할(semantic model roles)에 매핑합니다.

이 랩(lab)은 완료하는 데 약 **45**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 Power BI 환경에 액세스할 수 있어야 합니다.

## 시작하기 전에

환경에 다음이 설치되어 있는지 확인하십시오.

* Power BI Desktop
* SQL Server
* SQL Server Management Studio (SSMS)
* SSMS를 통해 복원된 [AdventureWorksDW2022](https://github.com/MicrosoftLearning/mslearn-fabric/raw/refs/heads/main/Allfiles/Labs/00-Setup/DatabaseBackup/AdventureWorksDW2022.bak) 데이터베이스 백업

>**참고**: SSMS를 사용하여 데이터베이스 백업을 복원하는 방법에 대한 자세한 정보는 [여기](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/restore-a-database-backup-using-ssms?view=sql-server-ver16)에서 확인할 수 있습니다.

## 시작하기

이 실습에서는 환경을 준비합니다.

### Power BI 시작 파일 다운로드

1. `https://aka.ms/fabric-security-starter`에서 [Sales Analysis 시작 파일](https://aka.ms/fabric-security-starter)을 다운로드하여 로컬 컴퓨터 (아무 폴더)에 저장합니다.

1. 다운로드한 파일로 이동하여 Power BI Desktop에서 엽니다.

1. 메시지가 표시되면 회사 또는 학교 계정으로 로그인합니다.

### Power BI 서비스에 로그인

이 작업에서는 Power BI 서비스에 로그인하고, 평가판 라이선스를 시작하고, "내 작업 영역(My workspace)"에 액세스합니다.

1. 웹 브라우저에서 `https://app.powerbi.com/`으로 이동합니다.

2. Power BI Desktop에서 사용한 것과 동일한 계정으로 로그인 프로세스를 완료합니다.

    *중요: Power BI Desktop에서 로그인하는 데 사용한 것과 동일한 자격 증명(credentials)을 사용해야 합니다.*

    *팁: Power BI 웹 브라우저 경험은 **Power BI 서비스**라고 알려져 있습니다.*

## 정적 역할 생성

이 실습에서는 정적 역할을 생성하고 유효성을 검사한 다음, 보안 주체(security principals)를 시맨틱 모델 역할(semantic model roles)에 매핑하는 방법을 알아봅니다.

### 데이터 모델 검토

이 작업에서는 데이터 모델을 검토합니다.

1. Power BI Desktop의 왼쪽에서 **모델(Model)** 보기로 전환합니다.

    ![](Images/enforce-model-security-image8.png)

2. 모델 다이어그램을 사용하여 모델 디자인을 검토합니다.

    ![](Images/enforce-model-security-image9.png)

    *이 모델은 6개의 차원 테이블(dimension tables)과 1개의 팩트 테이블(fact table)로 구성됩니다. **Sales** 팩트 테이블은 판매 주문 세부 정보를 저장합니다. 이는 고전적인 스타 스키마(star schema) 디자인입니다.*

3. **Sales Territory** 테이블을 확장하여 엽니다.

    ![](Images/enforce-model-security-image10.png)

4. 테이블에 **Region** 열이 포함되어 있음을 확인합니다.

    * **Region** 열은 Adventure Works 영업 지역을 저장합니다. 이 조직의 영업사원은 할당된 영업 지역과 관련된 데이터만 볼 수 있도록 허용됩니다. 이 랩(lab)에서는 데이터 권한(data permissions)을 적용하기 위해 두 가지 다른 행 수준 보안(row-level security) 기술을 구현합니다.*

### 정적 역할 생성

이 작업에서는 두 개의 정적 역할(static roles)을 생성합니다.

1. **보고서(Report)** 보기로 전환합니다.

    ![](Images/enforce-model-security-image11.png)

2. 누적 세로 막대형 차트(stacked column chart) 시각적 개체(visual)의 범례에서 (현재) 많은 지역을 볼 수 있음을 확인합니다.

    ![](Images/enforce-model-security-image12.png)

    *현재 차트가 너무 복잡해 보입니다. 모든 지역이 표시되기 때문입니다. 솔루션이 행 수준 보안(row-level security)을 적용하면 보고서 소비자(report consumer)는 하나의 지역만 보게 됩니다.*

3. 보안 역할(security role)을 추가하려면 **모델링(Modeling)** 리본 탭(ribbon tab)의 **보안(Security)** 그룹 내에서 **역할 관리(Manage roles)** 를 선택합니다.

    ![](Images/enforce-model-security-image13.png)

4. **역할 관리(Manage roles)** 창에서 **+ 새로 만들기(New)** 를 선택합니다.

5. 역할 이름을 지정하려면 선택된 *제목 없음(Untitled)* 텍스트를 **Australia**로 바꾸고 **Enter**를 누릅니다.

6. **테이블 선택(Select tables)** 목록에서 **Sales Territory**를 선택한 다음 **규칙(Rules)** 섹션에서 **+ 새로 만들기(New)** 를 선택합니다.

7. 새 규칙 행에서 다음 설정을 지정합니다.
	* **열(Column)**: Region
 	* **조건(Condition)**: 다음(Equals)과 같음
  	* **값(Value)**: Australia

    ![](Images/enforce-model-security-image16.png)

    *이 규칙은 **Region** 열을 **Australia** 값으로 필터링합니다.*

10. 다른 역할을 생성하려면 **역할(Roles)** 섹션에서 **+ 새로 만들기(New)** 를 누릅니다.

11. 이 작업의 단계를 반복하여 **Canada** 값을 기준으로 **Region** 열을 필터링하는 **Canada**라는 역할을 생성합니다.

    ![](Images/enforce-model-security-image19.png)

    *이 랩(lab)에서는 두 개의 역할만 생성합니다. 그러나 실제 솔루션에서는 11개의 Adventure Works 지역 각각에 대해 역할이 생성되어야 합니다.*

12. **저장(Save)** 을 선택합니다.

### 정적 역할 유효성 검사

이 작업에서는 정적 역할(static roles) 중 하나의 유효성을 검사합니다.

1. **모델링(Modeling)** 리본 탭(ribbon tab)의 **보안(Security)** 그룹 내에서 **역할로 보기(View as)** 를 선택합니다.

    ![](Images/enforce-model-security-image21.png)

2. **역할로 보기(View as roles)** 창에서 **Australia** 역할을 선택합니다.

    ![](Images/enforce-model-security-image22.png)

3. **확인(OK)** 을 선택합니다.

    ![](Images/enforce-model-security-image23.png)

4. 보고서 페이지에서 누적 세로 막대형 차트(stacked column chart) 시각적 개체(visual)가 Australia 데이터만 표시함을 확인합니다.

    ![](Images/enforce-model-security-image24.png)

5. 보고서 상단에서 적용된 역할을 확인하는 빨간색 배너를 확인합니다.

    ![](Images/enforce-model-security-image25.png)

6. 역할 사용을 중지하려면 빨간색 배너의 오른쪽에 있는 **보기 중지(Stop viewing)** 를 선택합니다.

	![](Images/enforce-model-security-image26.png)

### 보고서 게시

이 작업에서는 보고서를 게시합니다.

1. Power BI Desktop 파일을 저장합니다. 보류 중인 변경 사항을 적용하라는 메시지가 나타나면 **나중에 적용(Apply later)** 을 선택합니다.

	![](Images/enforce-model-security-image27.png)

2. 보고서를 게시하려면 **홈(Home)** 리본 탭(ribbon tab)에서 **게시(Publish)** 를 선택합니다.

    ![](Images/enforce-model-security-image28.png)

3. **Power BI에 게시(Publish to Power BI)** 창에서 **"내 작업 영역(My workspace)"** 을 선택한 다음 **선택(Select)** 을 선택합니다.

    ![](Images/enforce-model-security-image29.png)

4. 게시가 성공하면 **확인(Got it)** 을 선택합니다.

    ![](Images/enforce-model-security-image30.png)

#### 행 수준 보안 구성 (선택 사항)

이 작업에서는 Power BI 서비스에서 행 수준 보안(row-level security)을 구성하는 방법을 알아봅니다.

> 참고: 이 작업은 작업 중인 테넌트(tenant)에 **Salespeople_Australia** 보안 그룹(security group)이 존재해야 합니다. 이 보안 그룹은 테넌트에 자동으로 존재하지 않습니다. 테넌트에 대한 권한이 있는 경우 아래 단계를 따를 수 있습니다. 교육에서 제공된 테넌트를 사용하는 경우 보안 그룹을 생성할 적절한 권한이 없습니다. 작업을 읽어보되, 보안 그룹이 없는 경우에는 이 작업을 완료할 수 없다는 점을 유의하십시오. **읽은 후에는 정리(Clean Up) 작업으로 진행하십시오.**

1. Power BI 서비스(웹 브라우저)로 전환합니다.

2. 작업 영역 랜딩 페이지(workspace landing page)에서 **Sales Analysis - Enforce model security** 시맨틱 모델(semantic model)을 확인합니다.

    ![](Images/enforce-model-security-image31.png)

3. 시맨틱 모델 위로 커서를 가져가서 줄임표(ellipsis)가 나타나면 줄임표를 선택한 다음 **보안(Security)** 을 선택합니다.

    ![](Images/enforce-model-security-image32.png)

    * **보안(Security)** 옵션은 보안 그룹(security groups) 및 사용자(users)를 포함하는 Microsoft Entra 보안 주체(security principals) 매핑을 지원합니다.*

4. 왼쪽에서 역할(roles) 목록과 **Australia**가 선택되어 있음을 확인합니다.

    ![](Images/enforce-model-security-image33.png)

5. **멤버(Members)** 상자에서 **Salespeople_Australia** 입력을 시작합니다.

    *단계 5부터 8까지는 Salespeople_Australia 보안 그룹의 생성 또는 존재 여부에 따라 달라지므로 데모 목적으로만 사용됩니다. 보안 그룹을 생성할 권한과 지식이 있는 경우 자유롭게 진행하십시오. 그렇지 않은 경우 정리(Clean Up) 작업으로 계속 진행하십시오.*

    ![](Images/enforce-model-security-image34.png)

6. **추가(Add)** 를 선택합니다.

    ![](Images/enforce-model-security-image35.png)

7. 역할 매핑(role mapping)을 완료하려면 **저장(Save)** 을 선택합니다.

    ![](Images/enforce-model-security-image36.png)

    *이제 **Salespeople_Australia** 보안 그룹의 모든 멤버는 **Australia** 역할에 매핑되어 데이터 액세스(data access)를 호주 판매 데이터만 볼 수 있도록 제한합니다.*

    *실제 솔루션에서는 각 역할이 보안 그룹에 매핑되어야 합니다.*

    *이러한 디자인 접근 방식은 각 지역에 보안 그룹이 존재할 때 간단하고 효과적입니다. 그러나 단점도 있습니다. 생성 및 설정에 더 많은 노력이 필요합니다. 또한 새로운 지역이 온보딩될 때 시맨틱 모델(semantic model)을 업데이트하고 다시 게시해야 합니다.*

    *다음 실습에서는 데이터 기반의 동적 역할을 생성합니다. 이 디자인 접근 방식은 이러한 단점을 해결하는 데 도움이 될 수 있습니다.*

8. 작업 영역 랜딩 페이지(workspace landing page)로 돌아가려면 **탐색(Navigation)** 창에서 작업 영역을 선택합니다.

### 솔루션 정리

이 작업에서는 시맨틱 모델(semantic model)과 모델 역할(model roles)을 제거하여 솔루션을 정리합니다.

1. 시맨틱 모델(semantic model)을 제거하려면 시맨틱 모델 위로 커서를 가져가서 줄임표(ellipsis)가 나타나면 줄임표를 선택한 다음 **삭제(Delete)** 를 선택합니다.

    ![](Images/enforce-model-security-image37.png)

    *다음 실습에서 수정된 시맨틱 모델(semantic model)을 다시 게시합니다.*

2. 삭제를 확인하라는 메시지가 나타나면 **삭제(Delete)** 를 선택합니다.

    ![](Images/enforce-model-security-image38.png)

3. Power BI Desktop으로 전환합니다.

4. 보안 역할(security roles)을 제거하려면 **모델링(Modeling)** 리본 탭(ribbon tab)의 **보안(Security)** 그룹 내에서 **역할 관리(Manage roles)** 를 선택합니다.

    ![](Images/enforce-model-security-image39.png)

5. **역할 관리(Manage roles)** 창에서 첫 번째 역할을 제거하려면 역할 옆의 줄임표(ellipsis)를 선택한 다음 **삭제(Delete)** 를 선택합니다.

    ![](Images/enforce-model-security-image40.png)

6. 두 번째 역할도 제거합니다.

7. **저장(Save)** 을 선택합니다.

## 동적 역할 생성

이 실습에서는 모델에 테이블을 추가하고, 동적 역할(dynamic role)을 생성하고 유효성을 검사한 다음, 보안 주체(security principal)를 시맨틱 모델 역할(semantic model role)에 매핑합니다.

### Salesperson 테이블 추가

이 작업에서는 **Salesperson** 테이블을 모델에 추가합니다.

1. **모델(Model)** 보기로 전환합니다.

    ![](Images/enforce-model-security-image43.png)

2. **홈(Home)** 리본 탭(ribbon tab)의 **쿼리(Queries)** 그룹 내에서 **데이터 변환(Transform data)** 아이콘을 선택합니다.

    ![](Images/enforce-model-security-image44.png)

    *연결 방법을 지정하라는 메시지가 표시되면 **자격 증명 편집(Edit Credentials)** 을 선택하고 로그인 방법을 지정하십시오.*

    ![](Images/work-with-model-relationships-image52.png)

    * **연결(Connect)** 을 선택합니다.*

     ![](Images/work-with-model-relationships-image53.png)

    * **암호화 지원(Encryption Support)** 페이지에서 **확인(OK)** 을 선택합니다.*

3. **Power Query 편집기(Power Query Editor)** 창의 **쿼리(Queries)** 창 (왼쪽에 위치)에서 **Customer** 쿼리(query)를 마우스 오른쪽 버튼으로 클릭한 다음 **복제(Duplicate)** 를 선택합니다.

    ![](Images/enforce-model-security-image45.png)

    * **Customer** 쿼리에는 이미 데이터 웨어하우스(data warehouse)에 연결하는 단계가 포함되어 있으므로 이를 복제하는 것은 새로운 쿼리 개발을 시작하는 효율적인 방법입니다.*

4. **쿼리 설정(Query Settings)** 창 (오른쪽에 위치)의 **이름(Name)** 상자에서 텍스트를 **Salesperson**으로 바꿉니다.

    ![](Images/enforce-model-security-image46.png)

5. **적용된 단계(Applied Steps)** 목록에서 **다른 열 제거(Removed Other Columns)** 단계 (세 번째 단계)를 마우스 오른쪽 버튼으로 클릭한 다음 **끝까지 삭제(Delete Until End)** 를 선택합니다.

    ![](Images/enforce-model-security-image47.png)

6. 단계 삭제를 확인하라는 메시지가 나타나면 **삭제(Delete)** 를 선택합니다.

    ![](Images/enforce-model-security-image48.png)

7. 다른 데이터 웨어하우스(data warehouse) 테이블에서 데이터를 가져오려면 **적용된 단계(Applied Steps)** 목록의 **탐색(Navigation)** 단계 (두 번째 단계)에서 톱니바퀴 아이콘 (오른쪽에 위치)을 선택합니다.

    ![](Images/enforce-model-security-image49.png)

8. **탐색(Navigation)** 창에서 **DimEmployee** 테이블을 선택합니다.

    ![](Images/enforce-model-security-image50.png)

9. **확인(OK)** 을 선택합니다.

    ![](Images/enforce-model-security-image51.png)

10. 불필요한 열을 제거하려면 **홈(Home)** 리본 탭(ribbon tab)의 **열 관리(Manage Columns)** 그룹 내에서 **열 선택(Choose Columns)** 아이콘을 선택합니다.

    ![](Images/enforce-model-security-image52.png)

11. **열 선택(Choose Columns)** 창에서 **(모든 열 선택(Select All Columns))** 항목을 선택 해제합니다.

    ![](Images/enforce-model-security-image53.png)

12. 다음 세 열을 선택합니다.

    - EmployeeKey

    - SalesTerritoryKey

    - EmailAddress

13. **확인(OK)** 을 선택합니다.

    ![](Images/enforce-model-security-image54.png)

14. **EmailAddress** 열의 이름을 바꾸려면 **EmailAddress** 열 머리글을 두 번 클릭합니다.

15. 텍스트를 **UPN**으로 바꾸고 **Enter**를 누릅니다.

    *UPN은 사용자 계정 이름(User Principal Name)의 약어입니다. 이 열의 값은 Azure AD 계정 이름과 일치합니다.*

    ![](Images/enforce-model-security-image55.png)

16. 테이블을 모델로 로드하려면 **홈(Home)** 리본 탭(ribbon tab)에서 **닫기 및 적용(Close & Apply)** 아이콘을 선택합니다.

    ![](Images/enforce-model-security-image56.png)

17. 테이블이 모델에 추가되면 **Sales Territory** 테이블과의 관계(relationship)가 자동으로 생성되었음을 확인합니다.

### 관계 구성

이 작업에서는 새 관계의 속성을 구성합니다.

1. **Salesperson**과 **Sales Territory** 테이블 간의 관계를 마우스 오른쪽 버튼으로 클릭한 다음 **속성(Properties)** 을 선택합니다.

    ![](Images/enforce-model-security-image57.png)

2. **관계 편집(Edit relationship)** 창의 **교차 필터 방향(Cross filter direction)** 드롭다운 목록에서 **양방향(Both)** 을 선택합니다.

3. **양방향으로 보안 필터 적용(Apply security filter in both directions)** 확인란을 선택합니다.

    ![](Images/enforce-model-security-image58.png)

    * **Sales Territory** 테이블에서 **Salesperson** 테이블로의 일대다 관계(one-to-many relationship)가 있으므로 필터는 **Sales Territory** 테이블에서 **Salesperson** 테이블로만 전파됩니다. 다른 방향으로 전파를 강제하려면 교차 필터 방향을 양방향으로 설정해야 합니다.*

4. **확인(OK)** 을 선택합니다.

    ![](Images/enforce-model-security-image59.png)

5. 테이블을 숨기려면 **Salesperson** 테이블의 오른쪽 상단에서 눈 아이콘을 선택합니다.

    ![](Images/enforce-model-security-image60.png)

    * **Salesperson** 테이블의 목적은 데이터 권한(data permissions)을 적용하는 것입니다. 숨겨지면 보고서 작성자(report authors)와 Q&A 경험(Q&A experience)은 테이블이나 해당 필드를 볼 수 없습니다.*

### 동적 역할 생성

이 작업에서는 모델의 데이터에 기반하여 권한을 적용하는 동적 역할(dynamic role)을 생성합니다.

1. **보고서(Report)** 보기로 전환합니다.

    ![](Images/enforce-model-security-image61.png)

2. 보안 역할(security role)을 추가하려면 **모델링(Modeling)** 리본 탭(ribbon tab)의 **보안(Security)** 그룹 내에서 **역할 관리(Manage roles)** 를 선택합니다.

    ![](Images/enforce-model-security-image62.png)

3. **역할 관리(Manage roles)** 창에서 **+ 새로 만들기(New)** 를 선택합니다.

    ![](Images/enforce-model-security-image63.png)

4. 역할 이름을 지정하려면 선택된 텍스트를 **Salespeople**로 바꿉니다.

    ![](Images/enforce-model-security-image64.png)

    *이번에는 하나의 역할만 생성하면 됩니다.*

5. **Salesperson** 테이블을 선택하고 **규칙(Rules)** 탭에서 **DAX 편집기(DAX editor)로 전환**을 선택합니다.

    ![](Images/enforce-model-security-image65.png)

6. **DAX 편집기(DAX editor)** 상자에 다음 식(expression)을 입력합니다.

    ```DAX
   [UPN] = USERPRINCIPALNAME()
    ```

    ![](Images/enforce-model-security-image66.png)

    *이 식은 인증된 사용자(authenticated user)의 UPN(User Principal Name)을 반환하는 USERPRINCIPALNAME 함수를 사용하여 **UPN** 열을 필터링합니다.*

    *UPN이 **Salesperson** 테이블을 필터링하면 **Sales Territory** 테이블이 필터링되고, 이는 다시 **Sales** 테이블을 필터링합니다. 이러한 방식으로 인증된 사용자는 할당된 지역의 판매 데이터만 볼 수 있습니다.*

7. **저장(Save)** 을 선택합니다.

    ![](Images/enforce-model-security-image67.png)

### 동적 역할 유효성 검사

이 작업에서는 동적 역할(dynamic role)을 유효성 검사합니다.

1. **모델링(Modeling)** 리본 탭(ribbon tab)의 **보안(Security)** 그룹 내에서 **역할로 보기(View as)** 를 선택합니다.

    ![](Images/enforce-model-security-image68.png)

2. **역할로 보기(View as roles)** 창에서 **다른 사용자(Other user)** 를 선택한 다음 해당 상자에 `michael9@adventure-works.com`을 입력합니다.

    ![](Images/enforce-model-security-image69.png)

    *테스트 목적으로, **다른 사용자(Other user)** 는 USERPRINCIPALNAME 함수에 의해 반환될 값입니다. 이 영업사원은 **Northeast** 지역에 할당됩니다.*

3. **Salespeople** 역할을 선택합니다.

    ![](Images/enforce-model-security-image70.png)

4. **확인(OK)** 을 선택합니다.

    ![](Images/enforce-model-security-image71.png)

5. 보고서 페이지에서 누적 세로 막대형 차트(stacked column chart) 시각적 개체(visual)가 Northeast 데이터만 표시함을 확인합니다.

    ![](Images/enforce-model-security-image72.png)

6. 보고서 상단에서 적용된 역할을 확인하는 빨간색 배너를 확인합니다.

    ![](Images/enforce-model-security-image73.png)

7. 역할 사용을 중지하려면 빨간색 배너의 오른쪽에 있는 **보기 중지(Stop viewing)** 를 선택합니다.

    ![](Images/enforce-model-security-image74.png)

### 디자인 마무리

이 작업에서는 보고서를 게시하고 보안 그룹(security group)을 역할에 매핑하여 디자인을 마무리합니다.

*이 작업의 단계는 의도적으로 간략하게 설명되었습니다. 전체 단계 세부 정보는 이전 실습의 작업 단계를 참조하십시오.*

1. Power BI Desktop 파일을 저장합니다.

2. 보고서를 **"내 작업 영역(My workspace)."** 에 게시합니다.

3. Power BI Desktop을 닫습니다.

4. Power BI 서비스(웹 브라우저)로 전환합니다.

5. **Sales Analysis - Enforce model security** 시맨틱 모델(semantic model)의 보안 설정으로 이동합니다.

6. **Salespeople** 보안 그룹(security group)을 **Salespeople** 역할에 매핑합니다.

    ![](Images/enforce-model-security-image76.png)

    *이제 **Salespeople** 보안 그룹의 모든 멤버는 **Salespeople** 역할에 매핑됩니다. 인증된 사용자(authenticated user)가 **Salesperson** 테이블의 행으로 표시되면 할당된 판매 지역이 판매 테이블을 필터링하는 데 사용됩니다.*

    *이러한 디자인 접근 방식은 데이터 모델(data model)이 사용자 계정 이름(user principal name) 값을 저장할 때 간단하고 효과적입니다. 영업사원이 추가되거나 제거되거나 다른 판매 지역에 할당될 때 이 디자인 접근 방식은 단순히 작동할 것입니다.*