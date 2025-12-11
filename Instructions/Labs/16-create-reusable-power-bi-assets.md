---
lab:
    title: 'Create reusable Power BI assets'
    module: 'Create reusable Power BI assets'
---

# 재사용 가능한 Power BI 자산 만들기

이 연습에서는 Semantic model 및 report 개발을 지원하는 재사용 가능한 자산(assets)을 만듭니다. 이러한 자산에는 Power BI Project 및 Template 파일이 포함됩니다.

   > 참고: 이 연습은 Fabric 라이선스를 요구하지 않으며, Power BI 라이선스가 있는 Power BI 또는 Microsoft Fabric 환경에서 완료할 수 있습니다.

이 연습을 완료하는 데 약 **30**분이 소요됩니다.

## 시작하기 전에

이 연습을 시작하기 전에 웹 브라우저를 열고 다음 URL을 입력하여 zip 폴더를 다운로드해야 합니다:

`https://github.com/MicrosoftLearning/mslearn-fabric/raw/refs/heads/main/Allfiles/Labs/16b/16-reusable-assets.zip`

zip 파일을 **C:\Users\Student\Downloads\16-reusable-assets** 폴더에 압축 해제합니다.

## 새 Power BI Project 만들기

이 작업에서는 report를 Power BI Project 파일(*.pbip*)로 저장합니다. Power BI Project 파일은 report 및 semantic model 세부 정보를 소스 제어(source control)와 함께 작동하는 플랫 파일(flat files)에 저장합니다. Visual Studio Code를 사용하여 이 파일들을 수정하거나 Git을 사용하여 변경 사항을 추적할 수 있습니다.

1. **16-reusable-assets** 폴더 내의 **16-Starter-Sales Analysis.pbix** 파일을 엽니다.

1. **파일(File)** > **옵션 및 설정(Options and settings)** > **옵션(Options)** > **미리 보기 기능(Preview features)**을 선택하고 **TMDL 형식으로 semantic model 저장(Store semantic model using TMDL format)** 옵션을 선택한 다음 **확인(OK)**을 클릭합니다.

    > 이는 현재 미리 보기 기능인 Tabular Model Definition Language (TMDL)를 사용하여 semantic model을 저장하는 옵션을 활성화합니다. Power BI Desktop을 다시 시작하라는 메시지가 표시되면 연습을 계속하기 전에 다시 시작하십시오.

    ![미리 보기 기능 범주에서 사용 가능한 옵션 스크린샷](./Images/power-bi-enable-tmdl.png)

1. **다른 이름으로 저장(Save as)**을 선택하고, 파일을 저장할 때 드롭다운 메뉴의 화살표를 클릭하여 파일 형식을 선택합니다.

1. **.pbip** 파일 확장자를 선택한 다음, report의 이름을 지정하고 기억하기 쉬운 폴더에 저장합니다.

    ![드롭다운 메뉴가 확장된 상태의 다른 이름으로 저장 선택 스크린샷](./Images/power-bi-save-file-types.png)

1. Power BI Desktop 창 상단에 report 이름 옆에 **(Power BI Project)**가 있는 것을 확인합니다.

1. Power BI Desktop을 **닫고(Close)**, 메시지가 표시되면 변경 사항을 저장합니다.

1. Power BI Desktop을 **다시 열고(Reopen)**, 방금 저장한 **.pbip** 파일을 엽니다.

1. 프로젝트를 TMDL 형식으로 **업그레이드(Upgrade)**하라는 메시지가 표시되면, **업그레이드(Upgrade)**를 선택하여 semantic model에 대한 Tabular Model Definition Language 형식을 활성화합니다.

    > 이 업그레이드는 TMDL 형식을 사용하여 semantic model을 저장하는 데 필요하며, 다음 단계에서 .tmdl 파일을 볼 수 있게 합니다.

1. 파일을 **저장합니다(Save)**.

### Power BI Project 파일 세부 정보 검토

Power BI Desktop의 변경 사항이 .tmdl 파일에 어떻게 반영되는지 살펴보겠습니다.

1. 바탕 화면에서 파일 탐색기(File explorer)를 사용하여 **.pbip** 파일을 저장한 폴더로 이동합니다.
1. 다음 항목을 볼 수 있습니다:

    - YourReport.pbip 파일
    - YourReport.Report 폴더
    - YourReport.SemanticModel 폴더
    - .gitignore Git Ignore Source 파일

## report에 새 테이블 추가

이 작업에서는 semantic model에 필요한 모든 데이터가 없기 때문에 새 테이블을 추가합니다.

1. Power BI Desktop에서 새 데이터를 추가하려면 **데이터 가져오기(Get data) > 웹(Web)**으로 이동합니다.

1. 연결되면 웹에서(From Web) 대화 상자가 나타납니다. **기본(Basic)** 라디오 버튼을 선택된 상태로 유지합니다. URL 경로로 다음 파일 경로를 입력합니다.

    `C:\Users\Student\Downloads\16-reusable-assets\us-resident-population-estimates-2020.html`

1. **HTML 테이블(HTML Tables) > 테이블 2(Table 2)** 상자를 선택한 다음, **데이터 변환(Transform Data)**을 선택하여 진행합니다.

    ![로드하거나 변환할 콘텐츠를 선택하는 탐색기 대화 상자 스크린샷](./Images/power-bi-navigator-html.png)

1. 테이블 2(Table 2) 데이터 미리 보기와 함께 새 Power Query 편집기(Editor) 창이 열립니다.
1. **테이블 2(Table 2)**의 이름을 *US Population*으로 변경합니다.
1. STATE를 **State**로, NUMBER를 **Population**으로 이름을 변경합니다.
1. RANK 열을 제거합니다.
1. **닫기 및 적용(Close & Apply)**을 선택하여 변환된 데이터를 semantic model에 로드합니다.
1. *잠재적 보안 위험(Potential security risk)* 대화 상자가 나타나면 **확인(OK)**을 선택합니다.
1. 파일을 **저장합니다(Save)**.
1. 메시지가 표시되면 Power BI Report 향상된 형식으로 **업그레이드하지 마십시오(Don't upgrade)**.

### 관계 만들기

이 작업에서는 Power BI Desktop에서 report에 변경 사항을 적용하고, 플랫 .tmdl 파일에서 변경 사항을 확인합니다.

1. 파일 탐색기(File explorer)에서 ***YourReport*.SemanticModel** 파일 폴더를 찾습니다.
1. definition 폴더를 열고 다른 파일들을 확인합니다.
1. Notepad에서 **relationships.tmdl** 파일을 열고 9개의 관계(relationships)가 나열되어 있음을 확인합니다. 파일을 닫습니다.
1. Power BI Desktop으로 돌아가 리본 메뉴에서 **모델링(Modeling)** 탭으로 이동합니다.
1. **관계 관리(Manage relationships)**를 선택하고 9개의 관계(relationships)가 있음을 확인합니다.
1. 다음과 같이 새 관계를 만듭니다:
    - **시작(From)**: Key 열로 State-Province를 가진 Reseller
    - **대상(To)**: Key 열로 State를 가진 US Population
    - **카디널리티(Cardinality)**: 다대일(Many-to-one) (*:1)
    - **교차 필터 방향(Cross-filter direction)**: 양쪽(Both)

    ![이전에 설명된 대로 구성된 새 관계 대화 상자 스크린샷](./Images/power-bi-new-relationship-us-population.png)

1. 파일을 **저장합니다(Save)**.
1. **relationships.tmdl** 파일에서 다시 확인하면 새 관계(relationship)가 추가된 것을 알 수 있습니다.

> 이러한 플랫 파일(flat files)의 변경 사항은 이진 파일인 *.pbix* 파일과 달리 소스 제어(source control) 시스템에서 추적 가능합니다.

## report에 measure 및 visual 추가

이 작업에서는 semantic model을 확장하고 visual에서 measure를 사용하기 위해 measure 및 visual을 추가합니다.

1. Power BI Desktop에서 데이터(Data) 창으로 이동하여 Sales 테이블을 선택합니다.
1. 상황별 테이블 도구(Table tools) 리본 메뉴에서 **새 measure(New measure)**를 선택합니다.
1. 수식 입력줄(formula bar)에 다음 코드를 입력하고 적용합니다:

    ```DAX
    Sales per Capita =
    DIVIDE(
        SUM(Sales[Sales]),
        SUM('US Population'[Population])
    )
    ```

1. 새로 생성된 **Sales per Capita** measure를 찾아 캔버스(canvas)로 끌어다 놓습니다.
1. **Sales \| Sales**, **US Population \| State**, **US Population \| Population** 필드를 동일한 visual로 끌어다 놓습니다.

   > *이 랩에서는 필드를 참조하기 위한 약식 표기법을 사용합니다. 예를 들어 **Sales \| Unit Price**와 같습니다. 이 예에서 **Sales**는 테이블 이름이고 **Unit Price**는 필드 이름입니다.*

1. visual을 선택하고 **테이블(Table)**로 변경합니다.
1. Sales per Capita 및 Population 데이터의 일관성 없는 서식을 확인합니다.
1. 데이터(Data) 창에서 각 필드를 선택하고 서식(format)과 소수점 자릿수(decimal places)를 변경합니다.
    - Sales per Capita: 통화(Currency) \| 소수점 이하 4자리(4 decimal places)
    - Population: 정수(Whole number) \| 쉼표로 구분(Comma separated) \| 소수점 이하 0자리(0 decimal places)

    ![서식이 구성된 Sales per Capita measure 스크린샷](./Images/power-bi-measure-details.png)

    > 팁: 실수로 잘못된 테이블에 measure를 만들었다면, 이전 이미지에 표시된 대로 홈 테이블(Home table)을 쉽게 변경할 수 있습니다.

1. 파일을 저장합니다.

    > 참고: 테이블은 4개의 열과 올바르게 서식이 지정된 숫자로 다음 이미지와 같이 보여야 합니다.

    ![State, Population, Sales per Capita, Sum of Sales를 보여주는 몇 줄의 테이블 visual 스크린샷](./Images/power-bi-sales-per-capita-table.png)


1. Notepad를 엽니다.
1. 다음 파일을 엽니다: `C:\Users\Student\Downloads\16-Starter-Sales Analysis.SemanticModel\definition\tables\Sales.tmdl`
1. 163번 줄 근처의 코드를 찾아 measure가 다음과 같이 추가된 것을 확인합니다:
   
    ```tmdl
    
    	measure 'Sales per Capita' =
    			
    			DIVIDE(
    			    SUM(Sales[Sales]),
    			    SUM('US Population'[Population])
    			)
    		formatString: \$#,0.0000;(\$#,0.0000);\$#,0.0000
    		lineageTag: c95a1828-af50-484b-8310-64614fe2504b
    
    		annotation PBI_FormatHint = {"currencyCulture":"en-US"}
    ```

1. Notepad를 닫습니다.
2. Power BI Desktop을 닫습니다. 변경 사항을 저장할 필요는 없습니다.

## 재사용 가능한 Power BI Template (PBIT) 만들기

이 연습의 다음 부분 목표는 **Power BI Template (PBIT 파일)**이 Power Query parameter—이 경우에는 **Region** parameter—를 사용하여 report 로직을 데이터에서 분리하는 방법을 시연하는 것입니다. 작은 report를 만들고, 어떤 Region의 데이터가 로드될지를 제어하는 parameter를 정의한 다음, report를 재사용 가능한 template으로 내보냅니다. template을 다시 열 때, 사용자는 단순히 region 값을 선택할 수 있으며, report는 해당 dataset을 자동으로 로드하여 template이 동일한 데이터 모델의 여러 변형에 걸쳐 확장 가능하고 표준화된 report를 어떻게 가능하게 하는지 보여줍니다.

### Region 판매 데이터 탐색

1. Windows 탐색기(Windows Explorer)를 열고 Region 데이터가 저장된 폴더로 이동합니다. 경로는 `C:\Users\student\Downloads\16-reusable-assets\data`와 같을 것입니다. 이 폴더 안에 두 개의 CSV 파일을 찾을 수 있습니다:
   
   - region-north.csv
   - region-south.csv

1. 각 파일(Notepad, Excel 또는 다른 텍스트 편집기 사용 가능)을 잠시 열어 내용을 탐색합니다. 데이터의 구조, 사용 가능한 열(columns), 그리고 두 regional dataset이 어떻게 유사한지 관찰하십시오. Region north와 south를 주목하십시오. 이러한 익숙함은 Power BI가 나중에 무엇을 로드할지, 그리고 Region parameter가 어떻게 이들 사이를 전환하는 데 사용될지 이해하는 데 도움이 될 것입니다.

### Power BI report 생성 및 데이터 로드

1. Power BI Desktop을 열고, 빈 report를 만듭니다.
1. **홈(Home)** 리본 메뉴에서 **데이터 가져오기(Get Data)**를 선택하고 **텍스트/CSV(Text/CSV)**를 선택합니다.
1. `C:\Users\Student\Downloads\16-reusable-assets\data\region-north.csv` 파일을 로드합니다.
1. **데이터 변환(Transform Data)**을 선택합니다.
1. 쿼리(query)의 이름을 `sales`로 변경합니다.

![쿼리 설정 스크린샷](./Images/sales-query.png)

### Power Query Parameter 만들기

1. Power Query에서 **홈(Home)** 리본 메뉴에서 **매개 변수 관리(Manage Parameters)**를 선택하고 **새 매개 변수(New Parameter)**를 선택합니다.
1. 다음 값으로 parameter를 생성합니다:
   
    | 속성             | 값                 |
    | ---------------- | --------------------- |
    | 이름             | `Region`.             |
    | 유형             | Text                  |
    | 제안된 값         | List of values        |
    | 테이블            | north, south          |
    | 기본값           | north                 |
    | 현재 값           | north                 |

    ![Power BI Desktop 매개 변수 관리 대화 상자 스크린샷](./Images/manage-parameter.png)

### Parameter를 사용하도록 쿼리 수정

사용자가 template을 열 때 Power BI에서 Region을 선택할 수 있도록 하려고 합니다.

1. **sales** 쿼리(query)를 다시 선택합니다.
1. **보기(View)** 리본 메뉴에서 **고급 편집기(Advanced Editor)**를 선택합니다.
1. 쿼리(query)가 region-north.csv로 하드코딩(hard-coded)되어 있음을 확인합니다.

    ```m
    let
        Source = Csv.Document(File.Contents("C:\Users\Student\Downloads\16-reusable-assets\data\region-north.csv"),[Delimiter=",", Columns=5, Encoding=1252, QuoteStyle=QuoteStyle.None]),
        #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
        #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Date", type date}, {"Region", type text}, {"Product", type text}, {"Units", Int64.Type}, {"Revenue", Int64.Type}})
    in
        #"Changed Type"
    ```
    
1. 하드코딩된 Region (north)을 parameter **Region**으로 대체합니다. 
    
    ```m
    let
        Source = Csv.Document(File.Contents("C:\Users\Student\Downloads\16-reusable-assets\data\region-" & Region & ".csv"),[Delimiter=",", Columns=5, Encoding=1252, QuoteStyle=QuoteStyle.None]),
        #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
        #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Date", type date}, {"Region", type text}, {"Product", type text}, {"Units", Int64.Type}, {"Revenue", Int64.Type}})
    in
        #"Changed Type"
    ```
    
    > **참고**: Power Query (M language)에서 텍스트 값은 따옴표 " " 안에 있어야 하며, 앰퍼샌드(&) 연산자를 사용하여 텍스트 조각을 결합(concatenate)합니다.

1. **홈(Home)** 리본 메뉴에서 **닫기 및 적용(Close & Apply)**을 선택합니다.

### 간단한 report 만들기

1. report 보기(view)로 돌아갑니다.
1. 다음을 만듭니다:
   - 총 수익(Revenue)을 보여주는 카드(Card)
   - 제품별(by Product) 수익(Revenue)을 보여주는 세로 막대형 차트(Column Chart)
   - 모든 필드(fields)를 보여주는 테이블(Table)

1. 제목을 추가합니다: `Regional Sales Report Template`

report는 다음과 같을 수 있습니다. 레이아웃(layout)에 대해서는 걱정하지 마십시오.

![alt text](./Images/report-template-example.png)

### Template (PBIT)으로 저장

1. **홈(Home)** 리본 메뉴에서 **파일(File)** > **다른 이름으로 저장(Save as)**을 선택합니다.
1. 폴더 위치(예: **Downloads**)를 선택하고 파일 이름을 입력합니다. 예를 들어 `regional-sales`와 같습니다.
1. 파일 확장자로 **PBIT**를 선택합니다.

    ![PBIT 파일로 저장하는 방법을 보여주는 스크린샷](./Images/save-as-pbit.png)

1. template 설명(description)을 요청받으면 다음 텍스트를 입력합니다:

    ```txt
    Select your region.
    ```
    
    ![Power BI Desktop 내보내기 template 대화 상자 스크린샷](./Images/export-template.png)

### template 테스트

1. Power BI Desktop을 닫습니다. 변경 사항을 저장할지 묻는 메시지가 표시되면 **저장 안 함(Don't save)**을 선택할 수 있습니다.
1. `regional-sales.pbit` 파일을 엽니다.
1. Region을 선택하도록 요청하는 parameter 프롬프트가 나타나는 것을 확인합니다. 

    ![Region parameter를 보여주는 대화 상자](./Images/select-region-sales-parameter.png)

1. 드롭다운 목록에서 **south**를 선택합니다.
1. 데이터를 로드하고 report를 엽니다.

report가 south-Region 값을 올바르게 여는 방법을 확인합니다.

![열린 report 예시 스크린샷](./Images/report-template-opened.png)

이제 완전히 재사용 가능하고 parameter화된 report framework를 갖게 되었습니다. 

1. report를 닫습니다. 저장할 필요는 없습니다.

## 정리

이 연습을 성공적으로 완료했습니다. Power BI Project 및 Template 파일을 만들었습니다. workspace와 모든 로컬 자산(assets)을 안전하게 삭제할 수 있습니다.