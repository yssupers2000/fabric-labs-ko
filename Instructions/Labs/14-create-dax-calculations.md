---
lab:
    title: 'Create DAX calculations in Power BI Desktop'
    module: 'Create DAX calculations in Power BI Desktop'
---

# Create DAX calculations in Power BI Desktop

## **랩 스토리**

이 랩에서는 DAX(Data Analysis Expressions)를 사용하여 계산 테이블(calculated tables), 계산 열(calculated columns) 및 간단한 측정값(measures)을 만듭니다.

이 랩에서는 다음 방법을 학습합니다:

- 계산 테이블 만들기
- 계산 열 만들기
- 측정값 만들기

**이 랩은 약 45분 정도 소요됩니다.**

## 시작하기

이 연습을 완료하려면 먼저 웹 브라우저를 열고 다음 URL을 입력하여 zip 폴더를 다운로드하세요:

`https://github.com/MicrosoftLearning/mslearn-fabric/raw/main/Allfiles/Labs/14/14-create-dax.zip`

폴더를 **C:\Users\Student\Downloads\14-create-dax** 폴더에 추출합니다.

**14-Starter-Sales Analysis.pbix** 파일을 엽니다.

> ***참고**: **취소(Cancel)**를 선택하여 로그인 프롬프트를 무시할 수 있습니다. 다른 정보 창은 모두 닫으세요. 변경 사항 적용을 요청하는 메시지가 나타나면 **나중에 적용(Apply Later)**을 선택하세요.*

## Salesperson 계산 테이블 만들기

이 작업에서는 **Sales**와 직접적인 관계를 갖는 **Salesperson** 계산 테이블을 만듭니다.

계산 테이블은 먼저 테이블 이름을 입력한 다음 등호(=)를 입력하고, 테이블을 반환하는 DAX 수식을 입력하여 만듭니다. 테이블 이름은 데이터 모델에 이미 존재할 수 없습니다.

수식 입력줄(formula bar)은 유효한 DAX 수식 입력을 지원합니다. 자동 완성(auto-complete), Intellisense 및 색상 코딩(color-coding)과 같은 기능을 포함하여 수식을 빠르고 정확하게 입력할 수 있도록 돕습니다.

1.  Power BI Desktop의 보고서 보기(Report view)에서 **모델링(Modeling)** 리본 메뉴의 **계산(Calculations)** 그룹 내에서 **새 테이블(New Table)**을 선택합니다.

     ![Picture 1](Images/create-dax-calculations-in-power-bi-desktop_image9.png)

2.  (계산을 만들거나 편집할 때 리본 메뉴 바로 아래에 열리는) 수식 입력줄에 **Salesperson =**을 입력하고 **Shift+Enter**를 누른 다음 **'Salesperson (Performance)'**를 입력하고 **Enter**를 누릅니다.

	> **참고**: *편의를 위해 이 랩의 모든 DAX 정의는 **14-create-dax\Snippets.txt** 파일에 있는 스니펫 파일에서 복사할 수 있습니다.*

	 ![Picture 4](Images/create-dax-calculations-in-power-bi-desktop_image10.png)

	> *이 테이블 정의는 **Salesperson (Performance)** 테이블의 복사본을 만듭니다. 데이터만 복사하지만, 가시성(visibility), 서식(formatting) 등과 같은 모델 속성(model properties)은 복사되지 않습니다.*

3.  **데이터(Data)** 창(pane)에서 테이블 아이콘 앞에 추가 계산기 아이콘이 있는 것을 확인하세요(계산 테이블을 나타냄).

	![Picture 10](Images/create-dax-calculations-in-power-bi-desktop_image11.png)

	> ***참고**: 계산 테이블은 테이블을 반환하는 DAX 수식을 사용하여 정의됩니다. 계산 테이블은 값을 구체화하고 저장하기 때문에 데이터 모델의 크기를 증가시킨다는 것을 이해하는 것이 중요합니다. 새 (미래) 날짜 값이 테이블에 로드될 때 이 데이터 모델의 경우처럼 수식 종속성(formula dependencies)이 새로 고쳐질 때마다 다시 계산됩니다.*
    >
	> *Power Query 원본 테이블과 달리 계산 테이블은 외부 데이터 원본에서 데이터를 로드하는 데 사용할 수 없습니다. 이미 데이터 모델에 로드된 데이터를 기반으로만 데이터를 변환할 수 있습니다.*

4.  모델 보기(Model view)로 전환하고 **Salesperson** 테이블을 사용할 수 있는지 확인하세요(테이블을 찾으려면 보기를 재설정해야 할 수도 있습니다).

5.  **Salesperson | EmployeeKey** 열에서 **Sales | EmployeeKey** 열로 관계(relationship)를 만듭니다.

6.  **Salesperson (Performance)** 및 **Sales** 테이블 간의 비활성 관계를 마우스 오른쪽 버튼으로 클릭한 다음 **삭제(Delete)**를 선택합니다. 삭제 확인 메시지가 나타나면 **예(Yes)**를 선택합니다.

7.  **Salesperson** 테이블에서 다음 열을 다중 선택한 다음 숨깁니다(**Is Hidden** 속성을 **예(Yes)**로 설정).

	- EmployeeID
	- EmployeeKey
	- UPN

8.  모델 다이어그램에서 **Salesperson** 테이블을 선택합니다.

9.  **속성(Properties)** 창에서 **설명(Description)** 상자에 **Salesperson related to Sales**를 입력합니다.
    
	> *사용자가 테이블 또는 필드 위에 커서를 올리면 **데이터(Data)** 창에 설명이 도구 설명(tooltips)으로 나타나는 것을 기억하실 것입니다.*

10. **Salesperson (Performance)** 테이블의 설명을 **Salesperson related to region(s)**으로 설정합니다.

*이제 데이터 모델은 영업 사원을 분석할 때 두 가지 대안을 제공합니다. **Salesperson** 테이블은 영업 사원이 판매한 판매를 분석할 수 있도록 하며, **Salesperson (Performance)** 테이블은 영업 사원에게 할당된 영업 지역에서 발생한 판매를 분석할 수 있도록 합니다.*

## Date 테이블 만들기

이 작업에서는 **Date** 테이블을 만듭니다.

1.  테이블 보기(Table view)로 전환합니다. **홈(Home)** 리본 탭의 **계산(Calculations)** 그룹 내에서 **새 테이블(New Table)**을 선택합니다.

	![Picture 5](Images/create-dax-calculations-in-power-bi-desktop_image15.png)

2.  수식 입력줄에 다음 DAX를 입력합니다:

	```DAX
	Date =  
	CALENDARAUTO(6)
	```

	![Picture 6](Images/create-dax-calculations-in-power-bi-desktop_image16.png)

	> *CALENDARAUTO() 함수는 날짜 값으로 구성된 단일 열 테이블을 반환합니다. "자동(auto)" 동작은 모든 데이터 모델 날짜 열을 스캔하여 데이터 모델에 저장된 가장 이른 날짜 값과 가장 늦은 날짜 값을 결정합니다. 그런 다음 이 범위 내의 각 날짜에 대해 한 행을 만들고, 전체 연도의 데이터가 저장되도록 양방향으로 범위를 확장합니다.*
    >
	> *이 함수는 연도의 마지막 월 숫자인 선택적 인수를 하나 사용할 수 있습니다. 생략하면 값은 12가 되며, 이는 12월이 연도의 마지막 달임을 의미합니다. 이 경우 6이 입력되어 6월이 연도의 마지막 달임을 의미합니다.*

3.  US 지역 설정(즉, mm/dd/yyyy)을 사용하여 서식이 지정된 날짜 값 열을 확인하세요.

	![Picture 7](Images/create-dax-calculations-in-power-bi-desktop_image17.png)

4.  왼쪽 하단의 상태 표시줄(status bar)에서 테이블 통계(table statistics)를 확인하세요. 1826개의 데이터 행이 생성되었으며, 이는 5년간의 전체 데이터를 나타냅니다.

	![Picture 9](Images/create-dax-calculations-in-power-bi-desktop_image18.png)

## 계산 열 만들기

이 작업에서는 다양한 기간별 필터링 및 그룹화를 가능하게 하는 열을 더 추가합니다. 또한 다른 열의 정렬 순서를 제어하기 위한 계산 열도 만듭니다.

> **참고**: *편의를 위해 이 랩의 모든 DAX 정의는 **Snippets.txt** 파일에서 복사할 수 있습니다.*

1.  **테이블 도구(Table Tools)** 상황별 리본 메뉴의 **계산(Calculations)** 그룹 내에서 **새 열(New Column)**을 선택합니다.

	> *계산 열은 먼저 열 이름을 입력한 다음 등호(=)를 입력하고, 단일 값 결과를 반환하는 DAX 수식을 입력하여 만듭니다. 열 이름은 테이블에 이미 존재할 수 없습니다.*

	![Picture 11](Images/create-dax-calculations-in-power-bi-desktop_image19.png)

2.  수식 입력줄에 다음을 입력(또는 스니펫 파일에서 복사)한 다음 **Enter**를 누릅니다:
	> *이 수식은 날짜의 연도 값을 사용하지만, 월이 6월 이후일 때는 연도 값에 1을 더합니다. 이는 Adventure Works에서 회계 연도(fiscal years)가 계산되는 방식입니다.*

   ```DAX
   Year =
   "FY" & YEAR('Date'[Date]) + IF(MONTH('Date'[Date]) > 6, 1)
   ```

3.  스니펫 파일 정의를 사용하여 **Date** 테이블에 다음 두 개의 계산 열을 만듭니다:

	- Quarter
	- Month

4.  새 열이 추가되었는지 확인합니다.

	![Picture 14](Images/create-dax-calculations-in-power-bi-desktop_image21.png)

5.  계산을 검증하려면 보고서 보기로 전환합니다.

6.  새 보고서 페이지(report page)를 만들려면 페이지 1 옆의 더하기 아이콘을 선택합니다.

	![Picture 15](Images/create-dax-calculations-in-power-bi-desktop_image22.png)

7.  새 보고서 페이지에 행렬 시각적 개체(matrix visual)를 추가하려면 **시각화(Visualizations)** 창에서 행렬 시각적 유형을 선택합니다.

	> *팁: 각 아이콘 위에 커서를 올리면 시각적 유형을 설명하는 도구 설명이 나타납니다.*

	![Picture 51](Images/create-dax-calculations-in-power-bi-desktop_image23.png)

8.  **데이터(Data)** 창의 **Date** 테이블에서 **Year** 필드를 **행(Rows)** 값/영역으로 끌어다 놓습니다.

	![Picture 17](Images/create-dax-calculations-in-power-bi-desktop_image24.png)

9.  **Month** 필드를 **Year** 필드 바로 아래의 **행(Rows)** 값/영역으로 끌어다 놓습니다.

10. 행렬 시각적 개체(matrix visual)의 오른쪽 상단(또는 시각적 개체의 위치에 따라 하단)에서 포크형 이중 화살표 아이콘을 선택합니다(모든 연도를 한 수준 아래로 확장합니다).

	![Picture 19](Images/create-dax-calculations-in-power-bi-desktop_image26.png)

11. 연도가 월로 확장되고, 월이 시간순이 아닌 알파벳순으로 정렬되는 것을 확인하세요.

	![Picture 20](Images/create-dax-calculations-in-power-bi-desktop_image27.png)

	> *기본적으로 텍스트 값은 알파벳순으로 정렬되고, 숫자는 가장 작은 값부터 가장 큰 값으로 정렬되며, 날짜는 가장 이른 날짜부터 가장 늦은 날짜로 정렬됩니다.*

12. **Month** 필드의 정렬 순서를 사용자 지정하려면 테이블 보기(Table view)로 전환합니다.

13. **Date** 테이블에 **MonthKey** 열을 추가합니다.

	```DAX
	MonthKey =
	(YEAR('Date'[Date]) * 100) + MONTH('Date'[Date])
	```

	> *이 수식은 각 연도/월 조합에 대한 숫자 값을 계산합니다.*

14. 테이블 보기에서 새 열에 숫자 값(예: 2017년 7월의 경우 201707 등)이 포함되어 있는지 확인합니다.

	![Picture 21](Images/create-dax-calculations-in-power-bi-desktop_image28.png)

15. 보고서 보기로 다시 전환합니다. **데이터(Data)** 창에서 **Month**를 선택합니다.

16. **열 도구(Column Tools)** 상황별 리본 메뉴의 **정렬(Sort)** 그룹 내에서 **열 기준으로 정렬(Sort by Column)**을 선택한 다음 **MonthKey**를 선택합니다.

	![Picture 22](Images/create-dax-calculations-in-power-bi-desktop_image29.png)

17. 행렬 시각적 개체에서 월이 이제 시간순으로 정렬되는 것을 확인하세요.

	![Picture 23](Images/create-dax-calculations-in-power-bi-desktop_image30.png)

## Date 테이블 완성하기

이 작업에서는 열을 숨기고 계층 구조(hierarchy)를 만들어 **Date** 테이블의 디자인을 완성합니다. 그런 다음 **Sales** 및 **Targets** 테이블과의 관계를 만듭니다.

1.  모델 보기로 전환합니다. **Date** 테이블에서 **MonthKey** 열을 숨깁니다(**Is Hidden**을 **예(Yes)**로 설정).

2.  **데이터(Data)** 오른쪽 창에서 **Date** 테이블을 선택하고 **Year** 열을 마우스 오른쪽 버튼으로 클릭한 다음 **계층 구조 만들기(create hierarchy)**를 선택합니다.

3.  새로 생성된 계층 구조를 마우스 오른쪽 버튼으로 클릭하고 **이름 바꾸기(Rename)**를 선택하여 **Fiscal**로 이름을 바꿉니다.

4.  **데이터(Data)** 창에서 다음 두 개의 나머지 필드를 선택하고 마우스 오른쪽 버튼으로 클릭한 다음 **계층 구조에 추가(Add to hierarchy)** -> **Fiscal**을 선택하여 Fiscal 계층 구조에 추가합니다.

	- Quarter
	- Month

	![Picture 24](Images/create-dax-calculations-in-power-bi-desktop_image31.png)

5.  다음 두 가지 모델 관계를 만듭니다:

	- **Date | Date**를 **Sales | OrderDate**로
	- **Date | Date**를 **Targets | TargetMonth**로

	> *랩에서는 필드를 참조하기 위해 약식 표기법을 사용합니다. 예를 들어 **Sales | Unit Price**와 같습니다. 이 예에서 **Sales**는 테이블 이름이고 **Unit Price**는 필드 이름입니다.*

6.  다음 두 열을 숨깁니다:

	- Sales | OrderDate
	- Targets | TargetMonth

## Date 테이블을 날짜 테이블로 표시

이 작업에서는 **Date** 테이블을 날짜 테이블로 표시합니다.

1.  보고서 보기로 전환합니다. **데이터(Data)** 창에서 **Date** 테이블(Date 필드가 아님)을 선택합니다.

2.  **테이블 도구(Table Tools)** 상황별 리본 메뉴의 **달력(Calendars)** 그룹 내에서 **날짜 테이블로 표시(Mark as Date Table)**를 선택합니다.

3.  **날짜 테이블로 표시(Mark as a Date Table)** 창에서 **날짜 테이블로 표시(Mark as a Date Table)** 속성을 **켜짐(On)**으로 전환하고 **날짜 열 선택(Choose a date column)** 드롭다운 목록에서 **Date**를 선택합니다. **저장(Save)**을 선택합니다.

	![Mark as date table](Images/create-dax-calculations-in-power-bi-desktop_date-table.png)

4.  Power BI Desktop 파일을 저장합니다.

> *이제 Power BI Desktop은 이 테이블이 날짜(시간)를 정의한다는 것을 이해합니다. 데이터 원본에 날짜 테이블이 없을 때 날짜 테이블에 대한 이 디자인 접근 방식이 적합합니다. 데이터 웨어하우스가 있는 경우 데이터 모델에서 날짜 논리를 "재정의"하는 대신 해당 날짜 차원 테이블에서 날짜 데이터를 로드하는 것이 적절할 것입니다.*

## 간단한 측정값 만들기

이 작업에서는 간단한 측정값을 만듭니다. 간단한 측정값은 단일 열의 값을 집계하거나 테이블의 행을 계산합니다.

1.  보고서 보기의 **페이지 2**에서 **데이터(Data)** 창의 **Sales | Unit Price** 필드를 행렬 시각적 개체로 끌어다 놓습니다.

	![Picture 27](Images/create-dax-calculations-in-power-bi-desktop_image35.png)

2.  (시각화(Visualizations) 창 아래에 위치한) 시각적 필드 창(visual fields pane)의 **값(Values)** 필드 값/영역에서 **Unit Price**가 **Unit Price의 평균(Average of Unit Price)**으로 표시되는 것을 확인하세요. **Unit Price**의 아래쪽 화살표를 선택한 다음 사용 가능한 메뉴 옵션을 확인하세요.

	![Picture 30](Images/create-dax-calculations-in-power-bi-desktop_image37.png)

	> *표시되는 숫자 열은 보고서 디자이너가 보고서 디자인 시 열 값을 어떻게 요약할지(또는 요약하지 않을지) 결정할 수 있도록 합니다. 이는 부적절한 보고로 이어질 수 있습니다. 하지만 일부 데이터 모델러는 이러한 상황을 운에 맡기지 않고, 이러한 열을 숨기고 대신 측정값에 정의된 집계 논리(aggregation logic)를 노출하는 것을 선택합니다. 이것이 이 랩에서 취할 접근 방식입니다.*

3.  측정값을 만들려면 **데이터(Data)** 창에서 **Sales** 테이블을 마우스 오른쪽 버튼으로 클릭한 다음 **새 측정값(New Measure)**을 선택합니다.

4.  수식 입력줄에 다음 측정값 정의를 추가합니다:

	```DAX
	Avg Price =  
	AVERAGE(Sales[Unit Price])
	```

5.  **Avg Price** 측정값을 행렬 시각적 개체에 추가하고, **Unit Price** 열과 동일한 결과(하지만 다른 서식)를 생성하는 것을 확인하세요.

6.  **값(Values)** 영역에서 **Avg Price** 필드의 상황별 메뉴(context menu)를 열고, 집계 방식(aggregation technique)을 변경할 수 없음을 확인하세요.

	![Picture 32](Images/create-dax-calculations-in-power-bi-desktop_image39.png)

	> *측정값의 집계 동작(aggregation behavior)을 수정할 수 없습니다.*

7.  스니펫 파일 정의를 사용하여 **Sales** 테이블에 다음 다섯 가지 측정값을 만듭니다:

	- Median Price
	- Min Price
	- Max Price
	- Orders
	- Order Lines

	> * **Orders** 측정값에 사용된 DISTINCTCOUNT() 함수는 주문을 한 번만 계산합니다(중복 무시). **Order Lines** 측정값에 사용된 COUNTROWS() 함수는 테이블에서 작동합니다.*
    >
	> *이 경우 주문 수는 고유한 **SalesOrderNumber** 열 값을 계산하여 산출되며, 주문 라인 수는 단순히 테이블 행의 수입니다(각 행은 주문의 한 라인임).*

8.  모델 보기로 전환한 다음 네 가지 가격 측정값인 **Avg Price**, **Max Price**, **Median Price**, **Min Price**를 다중 선택합니다.

9.  측정값의 다중 선택에 대해 다음 요구 사항을 구성합니다:

	- 서식을 소수점 이하 두 자리로 설정
	- **Pricing**이라는 표시 폴더(display folder)에 할당

	![Picture 33](Images/create-dax-calculations-in-power-bi-desktop_image40.png)

10. **Unit Price** 열을 숨깁니다.

	> *이제 **Unit Price** 열은 보고서 작성자에게 사용할 수 없습니다. 작성자는 모델에 추가한 가격 측정값을 사용해야 합니다. 이 디자인 접근 방식은 보고서 작성자가 가격을 부적절하게 집계(예: 합계)하는 것을 방지합니다.*

11. **Order Lines** 및 **Orders** 측정값을 다중 선택한 다음 다음 요구 사항을 구성합니다:

	- 서식에 천 단위 구분 기호 사용
	- **Counts**라는 표시 폴더에 할당

	![Picture 36](Images/create-dax-calculations-in-power-bi-desktop_image41.png)

12. 보고서 보기의 행렬 시각적 개체 **값(Values)** 값/영역에서 **Unit Price** 필드의 **X**를 선택하여 제거합니다.

13. 행렬 시각적 개체의 크기를 늘려 페이지 너비와 높이를 채웁니다.

14. 다음 다섯 가지 측정값을 행렬 시각적 개체에 추가합니다:

	- Median Price
	- Min Price
	- Max Price
	- Orders
	- Order Lines

15. 결과가 합리적이고 올바르게 서식이 지정되었는지 확인합니다.

	![Picture 39](Images/create-dax-calculations-in-power-bi-desktop_image43.png)

## 추가 측정값 만들기

이 작업에서는 더 복잡한 수식을 사용하는 추가 측정값을 만듭니다.

1.  보고서 보기에서 **페이지 1**을 선택하고 테이블 시각적 개체를 검토하여 **Target** 열의 총계를 확인합니다.

	![Picture 41](Images/create-dax-calculations-in-power-bi-desktop_image45.png)

2.  테이블 시각적 개체를 선택한 다음 **시각화(Visualizations)** 창에서 **Target** 필드를 제거합니다.

3.  **Targets | Target** 열의 이름을 **Targets | TargetAmount**로 바꿉니다.

	> *팁: 보고서 보기에서 열 이름을 바꾸는 방법은 여러 가지가 있습니다. **데이터(Data)** 창에서 열을 마우스 오른쪽 버튼으로 클릭한 다음 **이름 바꾸기(Rename)**를 선택하거나, 열을 두 번 클릭하거나, **F2**를 누를 수 있습니다.*

4.  **Targets** 테이블에 다음 측정값을 만듭니다:

	```DAX
	Target =
	IF(
	HASONEVALUE('Salesperson (Performance)'[Salesperson]),
	SUM(Targets[TargetAmount])
	)
	```

	> *HASONEVALUE() 함수는 **Salesperson** 열에 단일 값이 필터링되었는지 여부를 테스트합니다. 참인 경우 식은 대상 금액의 합계(해당 영업 사원에 대해서만)를 반환합니다. 거짓인 경우 BLANK가 반환됩니다.*

5.  **Target** 측정값의 서식을 소수점 이하 0자리로 지정합니다.

	> *팁: **측정값 도구(Measure Tools)** 상황별 리본 메뉴를 사용할 수 있습니다.*

6.  **TargetAmount** 열을 숨깁니다.

	> *팁: **데이터(Data)** 창에서 열을 마우스 오른쪽 버튼으로 클릭한 다음 **숨기기(Hide)**를 선택할 수 있습니다.*

7.  **Target** 측정값을 테이블 시각적 개체에 추가합니다.

8.  **Target** 열의 총계가 이제 BLANK인 것을 확인하세요.

	![Picture 43](Images/create-dax-calculations-in-power-bi-desktop_image47.png)

9.  스니펫 파일 정의를 사용하여 **Targets** 테이블에 다음 두 가지 측정값을 만듭니다:

	- Variance
	- Variance Margin

10. **Variance** 측정값의 서식을 소수점 이하 0자리로 지정합니다.

11. **Variance Margin** 측정값의 서식을 소수점 이하 두 자리의 백분율로 지정합니다.

12. **Variance** 및 **Variance Margin** 측정값을 테이블 시각적 개체에 추가합니다.

13. 모든 열과 행을 볼 수 있도록 테이블 시각적 개체의 크기를 조정합니다.

	![Picture 44](Images/create-dax-calculations-in-power-bi-desktop_image48.png)

	> *모든 영업 사원이 목표를 달성하지 못하는 것처럼 보이지만, 테이블 시각적 개체가 아직 특정 기간으로 필터링되지 않았음을 기억하세요.*

14. **데이터(Data)** 창의 오른쪽 상단에서 창을 축소했다가 다시 확장합니다.

	> *창을 축소하고 다시 열면 내용이 재설정됩니다.*

15. **Targets** 테이블이 이제 목록의 맨 위에 나타나는 것을 확인하세요.

	![Picture 46](Images/create-dax-calculations-in-power-bi-desktop_image50.png)

	*표시되는 측정값만으로 구성된 테이블은 목록의 맨 위에 자동으로 나열됩니다.*

## 랩 완료
