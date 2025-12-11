---
lab:
    title: 'Work smarter with Copilot in Microsoft Fabric Dataflow Gen2'
    module: 'Get started with Copilot in Fabric for data engineering'
---

# Work smarter with Copilot in Microsoft Fabric Dataflow Gen2

Microsoft Fabric에서 Dataflows (Gen2)는 다양한 데이터 소스에 연결하여 Power Query Online에서 변환을 수행합니다. 그런 다음 Data Pipelines에서 lakehouse 또는 기타 분석 저장소로 데이터를 수집하거나 Power BI 보고서용 데이터셋을 정의하는 데 사용할 수 있습니다. 이 랩은 복잡한 엔터프라이즈 솔루션을 구축하는 데 초점을 맞추기보다 Dataflows (Gen2)의 Copilot에 대한 소개를 제공합니다.

이 연습은 완료하는 데 약 **30**분이 소요됩니다.

## 학습할 내용

이 랩을 완료하면 다음을 수행하게 됩니다:

- Microsoft Fabric Dataflow Gen2에서 Copilot을 사용하여 데이터 변환 작업을 가속화하는 방법을 이해합니다.
- Copilot의 도움을 받아 Power Query Online을 사용하여 데이터를 수집, 정리 및 변환하는 방법을 학습합니다.
- 열 이름 변경, 불필요한 문자 제거, 적절한 데이터 형식 설정 등 데이터 품질을 위한 모범 사례를 적용합니다.
- 데이터 흐름 내에서 XML 데이터를 구문 분석하고 확장하는 경험을 얻습니다.
- 연속 데이터를 분석을 위한 의미 있는 그룹으로 분류합니다.
- 변환된 데이터를 lakehouse에 게시하고 결과를 검증합니다.
- 생산성 및 데이터 품질 향상을 위한 AI 지원 데이터 엔지니어링의 가치를 인식합니다.

## 시작하기 전에

이 연습을 완료하려면 Copilot이 활성화된 [Microsoft Fabric Capacity (F2 이상)](https://learn.microsoft.com/fabric/fundamentals/copilot-enable-fabric)가 필요합니다.

## 연습 시나리오

글로벌 소매 회사인 Contoso는 Microsoft Fabric을 사용하여 데이터 인프라를 현대화하고 있습니다. 데이터 엔지니어로서 분석을 위해 매장 정보를 준비하는 작업을 맡았습니다. 원시 데이터는 CSV 파일에 저장되며 내장된 XML 필드, 일치하지 않는 열 이름 및 불필요한 문자를 포함하고 있습니다. 목표는 Dataflow Gen2에서 Copilot을 사용하여 이 데이터를 수집, 정리, 변환 및 보강하여 lakehouse에서 보고 및 분석을 위한 준비를 마치는 것입니다. 이 실습 연습은 각 단계를 안내하며, Copilot이 일반적인 데이터 엔지니어링 작업을 어떻게 가속화하고 단순화하는지 보여줄 것입니다.

## 작업 영역 만들기

Fabric에서 데이터를 작업하기 전에 Fabric이 활성화된 workspace를 생성하십시오. workspace는 모든 Fabric 항목의 컨테이너 역할을 하며 팀을 위한 공동 작업 기능을 제공합니다.

1. 브라우저에서 `https://app.fabric.microsoft.com/home?experience=fabric`에 있는 [Microsoft Fabric 홈 페이지](https://app.fabric.microsoft.com/home?experience=fabric)로 이동하여 Fabric 자격 증명으로 로그인합니다.

1. 왼쪽 메뉴 바에서 **Workspaces**를 선택합니다 (아이콘은 &#128455;와 유사합니다).

1. 선택한 이름으로 새 workspace를 생성하고, Fabric Capacity (*Premium* 또는 *Fabric*)를 포함하는 라이선싱 모드를 선택합니다. *Trial*은 지원되지 않습니다.

    > **중요**: Fabric의 Copilot 기능은 유료 Capacity (F2 이상)가 필요합니다. Trial workspace는 Copilot 기능을 지원하지 않습니다.

1. 새 workspace가 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## Lakehouse 만들기

이제 workspace가 있으므로 데이터를 수집할 lakehouse를 생성할 차례입니다.

1. 왼쪽 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Lakehouse**를 선택합니다. 선택한 고유한 이름을 지정하십시오. "Lakehouse schemas (Public Preview)" 옵션은 비활성화되어 있는지 확인하십시오.

    >**참고**: **Create** 옵션이 사이드바에 고정되어 있지 않으면 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    잠시 후 새 빈 lakehouse가 생성됩니다.

    ![새 lakehouse 스크린샷.](./Images/new-lakehouse.png)

## 데이터를 수집하기 위한 Dataflow (Gen2) 만들기

이제 lakehouse가 있으므로 데이터를 수집해야 합니다. 이를 수행하는 한 가지 방법은 *추출, 변환 및 로드(extract, transform, and load - ETL)* 프로세스를 캡슐화하는 dataflow를 정의하는 것입니다.

1. workspace의 홈 페이지에서 **Get data** > **New Dataflow Gen2**를 선택합니다. 몇 초 후 새 dataflow의 Power Query editor가 여기에 표시된 대로 열립니다.

    ![새 데이터 흐름 스크린샷.](./Images/copilot-fabric-dataflow-new.png)

1. **Import from a Text/CSV file**을 선택하고 다음 설정을 사용하여 새 데이터 소스를 생성합니다.

   - **Link to file**: *Selected*
   - **File path or URL**: `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-fabric/refs/heads/main/Allfiles/Labs/22a/Store.csv`
   - **Connection**: Create new connection
   - **Connection name**: *고유한 이름 지정*
   - **data gateway**: (none)
   - **Authentication kind**: Anonymous
   - **Privacy Level**: None
  
> **참고**: 연결이 이미 존재하는 경우 새 연결을 생성하는 대신 기존 연결을 선택할 수 있습니다.

1. **Next**를 선택하여 파일 데이터를 미리 보고 **Create**를 선택하여 데이터 소스를 생성합니다. Power Query editor는 데이터 소스와 데이터를 포맷하기 위한 초기 쿼리 단계 집합을 여기에 표시된 대로 보여줍니다.

    ![Power Query 편집기의 쿼리 스크린샷.](./Images/copilot-fabric-dataflow-power-query.png)

1. **Home** 리본 탭의 **Insights** 그룹 내에서 여기에 표시된 대로 **Copilot**을 선택합니다.
    
    ![데이터 흐름에서 Copilot 창이 열린 스크린샷.](./Images/copilot-fabric-dataflow-copilot-pane.png)

1. 현재 열 이름은 너무 일반적이고 명확한 의미가 없습니다(대부분 Column1, Column2 등으로 표시될 수 있음). 의미 있는 열 이름은 데이터 이해 및 다운스트림 처리에 매우 중요합니다. 다음 프롬프트를 사용하여 열 이름을 세분화하고 의도한 정보를 정확하게 전달하는지 확인하십시오.

    ```copilot-prompt
    Rename columns to BusinessEntityID, Name, SalesPersonID, Demographics, rowguid, ModifiedDate
    ```

    이제 열 이름이 정확하고 설명적입니다. 또한, Copilot이 내부적으로 Power Query M code를 자동으로 생성하는 방식을 보여주는 추가 단계가 Applied Steps 목록에 통합되었습니다.
    
    ![Copilot 프롬프트 결과로 열 이름이 변경된 스크린샷.](./Images/copilot-fabric-dataflow-step.png)

1. 특정 열에는 텍스트 값 끝에 '+' 문자가 포함되어 있습니다. 이는 데이터 분석 및 다운스트림 처리에 방해가 될 수 있는 일반적인 데이터 품질 문제입니다.

    !['+' 문자가 포함된 특정 열의 스크린샷.](./Images/copilot-fabric-dataflow-plus-character.png)
    
    다음 프롬프트를 사용하여 이러한 불필요한 문자를 제거해 보겠습니다.
    
    ```copilot-prompt
    Delete the last character from the columns Name, Demographics, rowguid
    ```
    
    **왜 이것이 중요한가**: 불필요한 문자를 제거하면 데이터 일관성이 보장되고 나중에 문자열 작업이나 데이터 조인(join)을 수행할 때 문제가 발생하는 것을 방지할 수 있습니다.

1. 테이블에는 데이터셋을 간소화하고 처리 효율성을 향상하기 위해 제거해야 하는 일부 중복 열이 포함되어 있습니다. 다음 프롬프트를 사용하여 데이터를 적절하게 세분화하십시오.

    ![제거해야 하는 특정 열의 스크린샷.](./Images/copilot-fabric-dataflow-remove-columns.png)
    
    ```copilot-prompt
    Remove the rowguid and Column7 columns
    ```
    
    **참고**: `rowguid` 열은 일반적으로 내부 데이터베이스 작업에 사용되며 분석에는 필요하지 않습니다. `Column7`은 데이터셋에 가치를 추가하지 않는 비어 있거나 관련 없는 열로 보입니다.
    
1. Demographics 열에는 XML 데이터 구문 분석을 방해하는 보이지 않는 유니코드 문자(Byte Order Mark - BOM) \ufeff가 포함되어 있습니다. 올바른 처리를 위해 이를 제거해야 합니다. Copilot 창에 다음 프롬프트를 입력하십시오.

    ```copilot-prompt
    Remove the Byte Order Mark (BOM) \ufeff from the Demographics column
    ```
    
    **BOM 이해**: Byte Order Mark는 텍스트 인코딩의 바이트 순서를 나타내기 위해 텍스트 파일 시작 부분에 나타날 수 있는 유니코드 문자입니다. 파일 인코딩 감지에는 유용하지만 XML과 같은 구조화된 데이터를 구문 분석할 때 문제를 일으킬 수 있습니다.
    
    문자를 제거하기 위해 생성된 수식(formula)을 확인하십시오.
    
    ![BOM 문자가 제거된 데이터 흐름 수식의 스크린샷.](./Images/copilot-fabric-dataflow-bom-character.png)
    
1. 이제 XML 데이터를 구문 분석하고 별도의 열로 확장할 준비가 되었습니다. Demographics 열에는 연간 매출, 면적 및 기타 비즈니스 지표와 같은 귀중한 매장 정보를 담고 있는 XML 형식의 데이터가 포함되어 있습니다.

    ![XML 필드에 초점을 맞춘 데이터 흐름 테이블의 스크린샷.](./Images/copilot-fabric-dataflow-xml.png)
    
    Copilot 창에 다음 프롬프트를 입력하십시오.
    
    ```copilot-prompt
    Parse this XML and expand it's columns
    ```
    
    **XML 구문 분석 이해**: XML (eXtensible Markup Language)은 계층적 정보를 저장하는 데 일반적으로 사용되는 구조화된 데이터 형식입니다. XML을 구문 분석하고 확장함으로써 중첩된 데이터를 분석하기 쉬운 평면적인 테이블 형식 구조로 변환합니다.
    
    새 열이 테이블에 추가되었습니다(오른쪽으로 스크롤해야 할 수도 있습니다).
    
    ![XML 필드가 확장된 데이터 흐름 테이블의 스크린샷.](./Images/copilot-fabric-dataflow-copilot-xml-fields-expanded.png)

1. 모든 귀중한 정보를 별도의 열로 추출했으므로 더 이상 필요하지 않은 Demographics 열을 제거하십시오. Copilot 창에 다음 프롬프트를 입력하십시오.

    ```copilot-prompt
    Remove the Demographics column.
    ```

    **이 열을 제거하는 이유**: XML을 구문 분석하고 각 정보 조각에 대한 개별 열을 생성했으므로, 원시 XML을 포함하는 원래 Demographics 열은 중복되며 데이터셋을 깨끗하게 유지하기 위해 안전하게 제거할 수 있습니다.

1. ModifiedDate 열은 값 끝에 앰퍼샌드(&)가 있습니다. 적절한 데이터 처리를 위해 구문 분석 전에 제거해야 합니다.

    ![앰퍼샌드가 있는 수정된 날짜를 포함하는 데이터 흐름 테이블의 스크린샷.](./Images/copilot-fabric-dataflow-modified-date.png)
    
    Copilot 창에 다음 프롬프트를 입력하십시오.
    
    ```copilot-prompt
    Remove the last character from the ModifiedDate
    ```

1. 이제 날짜/시간 작업 및 분석을 위해 데이터 형식을 DateTime으로 변환할 준비가 되었습니다. Copilot 창에 다음 프롬프트를 입력하십시오.

    ```copilot-prompt
    Set the data type to DateTime
    ```

    **데이터 형식의 중요성**: 올바른 데이터 형식으로 변환하는 것은 다운스트림 분석에서 적절한 정렬, 필터링 및 날짜 기반 계산을 가능하게 하는 데 중요합니다.

    ModifiedDate 데이터 형식이 DateTime으로 변경된 것을 확인하십시오.
    
    ![수정된 데이터 흐름의 날짜 형식이 올바른 스크린샷.](./Images/copilot-fabric-dataflow-modified-date-type-correct.png)
    
1. 수학 연산 및 적절한 집계를 가능하게 하기 위해 현재 지정된 형식이 없는 여러 열의 데이터 형식을 숫자 값으로 조정하십시오. Copilot 창에 다음 프롬프트를 입력하십시오.

    ```copilot-prompt
    Set the data type to whole number for the following columns: AnnualSales, AnnualRevenue, SquareFeet, NumberEmployee
    ```
    
    **숫자로 변환하는 이유**: 숫자 데이터 형식은 텍스트 기반 데이터로는 불가능한 적절한 수학적 계산, 집계(합계, 평균 등) 및 통계 분석을 가능하게 합니다.
    
1. SquareFeet 필드는 6,000에서 80,000까지의 숫자 값을 가집니다. 연속적인 숫자 데이터에서 범주형 그룹을 생성하는 것은 데이터를 해석하고 분석하기 쉽게 만드는 일반적인 분석 기술입니다.

    ![SquareFeet 열 프로필이 강조 표시된 데이터 흐름 테이블의 스크린샷.](./Images/copilot-fabric-dataflow-square-feet.png)
    
    매장 크기를 적절히 분류하기 위한 새 열을 생성해 보겠습니다. Copilot 창에 다음 프롬프트를 입력하십시오.
    
    ```copilot-prompt
    Add a column StoreSize, based on the SquareFeet:
        0 - 10000: Small
        10001 - 40000: Medium
        40001 - 80000: Large
    ```
    
    SquareFeet 열을 기반으로 하는 수식과 함께 StoreSize라는 새 열이 추가된 것을 확인하십시오. 또한 열 프로필에 Small, Medium, Large의 3가지 고유한 값이 있음을 확인하십시오.
    
    ![매장 크기 필드, 수식 및 열 프로필을 포함하는 데이터 흐름 테이블의 스크린샷.](./Images/copilot-fabric-dataflow-store-size.png)
    
1. 현재 지정된 형식이 없는 열의 데이터 형식을 수정하십시오. Copilot 창에 다음 프롬프트를 입력하십시오.

    ```copilot-prompt
    Set the datatype of the following columns to text: Name, BankName, BusinessType, YearOpened, Specialty, Brands, Internet, StoreSize
    ```
    
    **데이터 형식 일관성**: 데이터 형식을 명시적으로 설정하면 다운스트림 프로세스에서 예측 가능한 동작이 보장되고 예기치 않은 결과를 초래할 수 있는 자동 형식 유추(inference)가 방지됩니다.
    
## 코드 설명

1. 몇 가지 변환을 수행했습니다. Copilot에게 우리가 취한 단계를 요약해 달라고 요청해 봅시다.

    ```copilot-prompt
    Describe this query
    ```
    
    결과가 Copilot 창에 나타나는 것을 확인하십시오. 아래는 제공된 설명의 예시입니다. AI 생성 콘텐츠는 실수가 있을 수 있으므로 결과가 약간 다를 수 있습니다.
    
    ![설명된 Copilot 데이터 흐름의 스크린샷.](./Images/copilot-fabric-dataflow-result.png)
    
    *다음은 **Store**에 대한 설명입니다: CSV 파일을 로드하고 변환하며, XML 데이터를 구문 분석하고, 매장을 크기별로 분류합니다.*
    
    - _**Source**: 파이프 구분 기호와 특정 인코딩을 사용하여 URL에서 CSV 파일을 로드합니다._
    - _**Changed column type**: 열의 데이터 형식을 변경합니다._
    - _**Rename columns**: 열 이름을 의미 있는 이름으로 바꿉니다._
    - _**Custom**: "Name", "Demographics", "rowguid" 열에서 마지막 문자를 제거합니다._
    - _**Remove columns**: "rowguid" 및 "Column7" 열을 제거합니다._
    - _**Custom 1**: "Demographics" 열에서 선행 특수 문자를 제거합니다._
    - _**Custom 2**: "Demographics" 열을 XML로 구문 분석하고 여러 열로 확장합니다._
    - _**Remove columns 1**: 원본 "Demographics" 열을 제거합니다._
    - _**Transform columns**: "ModifiedDate" 열에서 마지막 문자를 제거합니다._
    - _**Transform columns 1**: "ModifiedDate" 열을 DateTime 형식으로 변환합니다._
    - _**Change type**: "AnnualSales", "AnnualRevenue", "SquareFeet", "NumberEmployees" 열의 데이터 형식을 정수로 변경합니다._
    - _**Conditional column**: "SquareFeet" 값에 따라 매장을 "Small", "Medium", "Large"로 분류하는 새 열 "StoreSize"를 추가합니다._
    - _**Change type 1**: 여러 열의 데이터 형식을 텍스트로 변경합니다._
    
## Dataflow에 데이터 대상 추가

1. 도구 모음 리본에서 **Home** 탭을 선택합니다. 그런 다음 **Default data destination** 드롭다운 메뉴에서 **Add**를 선택합니다.

    > **참고:** 이 옵션이 회색으로 표시되어 있다면 이미 데이터 대상이 설정되어 있을 수 있습니다. Power Query editor 오른쪽의 Query settings 창 하단에서 데이터 대상을 확인하십시오. 대상이 이미 설정되어 있다면 톱니바퀴(gear) 아이콘을 사용하여 변경할 수 있습니다.

1. **Lakehouse**를 선택합니다.

1. **Connect to default data destination** 대화 상자에서 연결을 편집하고 Power BI 조직 계정으로 로그인하여 dataflow가 lakehouse에 액세스하는 데 사용하는 ID를 설정합니다.

    ![데이터 대상 구성 페이지.](./Images/dataflow-connection.png)

1. **Next**를 선택하고 사용 가능한 workspace 목록에서 여러분이 이 연습 시작 시 생성한 workspace와 그 안에 있는 lakehouse를 찾아 선택합니다. 그런 다음 **Store**라는 새 테이블을 지정합니다.

    ![데이터 대상 구성 페이지.](./Images/copilot-fabric-dataflow-choose-destination.png)

1. **Next**를 선택하고 **Choose destination settings** 페이지에서 **Use automatic settings** 옵션을 비활성화한 다음 **Append**를 선택하고 **Save settings**를 선택합니다.

    > **참고:** 데이터 형식을 업데이트하는 데 *Power query editor*를 사용하는 것을 권장하지만, 원한다면 이 페이지에서도 수행할 수 있습니다.

    ![데이터 대상 설정 페이지.](./Images/copilot-fabric-dataflow-destination-column-mapping.png)

1. **Save & run**을 선택하여 dataflow를 게시합니다. 그런 다음 workspace에 **Dataflow 1** dataflow가 생성될 때까지 기다립니다.

## 작업 유효성 검사

이제 dataflow의 ETL process를 검증하고 모든 변환이 올바르게 적용되었는지 확인할 차례입니다.

1. workspace로 돌아가서 이전에 생성한 lakehouse를 엽니다.

1. lakehouse에서 **Store** 테이블을 찾아 엽니다. (dataflow가 데이터를 처리하는 동안 몇 분 정도 기다려야 할 수도 있습니다.)

1. 변환된 데이터의 다음 주요 측면을 관찰하십시오.

   - **열 이름**: 지정한 의미 있는 이름(BusinessEntityID, Name, SalesPersonID 등)과 일치하는지 확인하십시오.
   - **데이터 형식**: 숫자 열은 숫자로, DateTime 열은 날짜/시간으로, 텍스트 열은 텍스트로 표시되는지 확인하십시오.
   - **데이터 품질**: 불필요한 문자(+, &)가 제거되었는지 확인하십시오.
   - **XML 확장**: 원본 XML Demographics 데이터에서 추출된 개별 열을 확인하십시오.
   - **StoreSize 분류**: SquareFeet 값에 따라 Small/Medium/Large 범주가 올바르게 생성되었는지 확인하십시오.
   - **데이터 완전성**: 변환 과정에서 중요한 데이터가 손실되지 않았는지 확인하십시오.

   ![데이터 흐름으로 로드된 테이블.](./Images/copilot-fabric-dataflow-lakehouse-result.png)

    **왜 이것이 중요한가**: 최종 테이블은 의미 있는 열 이름, 적절한 데이터 형식, 그리고 새로운 StoreSize 범주형 열을 포함하는 깨끗하고 잘 구조화된 데이터를 포함해야 합니다. 이는 Copilot이 원시의 지저분한 데이터를 깨끗하고 분석 준비가 된 dataset으로 변환하는 데 어떻게 도움이 되는지 보여줍니다.

## 리소스 정리

Microsoft Fabric에서 dataflow 탐색을 마쳤다면 이 연습을 위해 생성한 workspace를 삭제할 수 있습니다.

1. 브라우저에서 Microsoft Fabric으로 이동합니다.
1. 왼쪽 바에서 workspace 아이콘을 선택하여 포함된 모든 항목을 확인합니다.
1. **Workspace settings**를 선택하고 **General** 섹션에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.
1. **Delete**를 선택하여 workspace를 삭제합니다.