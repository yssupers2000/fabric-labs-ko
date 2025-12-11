---
lab:
    title: 'Design scalable semantic models'
    module: 'Design scalable semantic models'
---

# 확장 가능한 시맨틱 모델 디자인

이 연습에서는 DAX 함수를 사용하여 데이터 모델의 유연성과 효율성을 향상시키고, 특히 Calculation groups와 Field parameters와 같은 기능을 활용합니다. 이러한 기능을 함께 사용하면 여러 시각적 개체나 복잡한 DAX 식 없이도 대화형 보고서를 만들 수 있어, 매우 유연하고 확장 가능한 시맨틱 모델을 생성할 수 있습니다.

이 연습에서는 다음 방법을 배웁니다.

- DAX 함수를 사용하여 관계(relationship) 동작을 수정하는 방법.
- Calculation groups를 생성하고 동적 시간 인텔리전스 계산에 적용하는 방법.
- Field parameters를 생성하여 다양한 필드와 측정값을 동적으로 선택하고 표시하는 방법.

이 랩은 완료하는 데 약 **30**분 정도 소요됩니다.

## 시작하기 전에

1. `https://github.com/MicrosoftLearning/mslearn-fabric/raw/main/Allfiles/Labs/15/15-scalable-semantic-models.zip`에서 [Sales Analysis 시작 파일](https://github.com/MicrosoftLearning/mslearn-fabric/raw/main/Allfiles/Labs/15/15-scalable-semantic-models.zip)을 다운로드하여 로컬에 저장합니다.

2. 폴더를 **C:\Users\Student\Downloads\15-scalable-semantic-models** 폴더에 압축 해제합니다.

3. **15-Starter-Sales Analysis.pbix** 파일을 엽니다.

    > 변경 사항을 적용하라는 경고가 나타나면 무시하고 닫으세요. *변경 내용 무시(Discard changes)*를 선택하지 마세요.

## 관계(Relationship) 작업

이 작업에서는 미리 개발된 Power BI Desktop 솔루션을 열어 데이터 모델에 대해 알아봅니다. 그런 다음 활성 모델 관계(relationship)의 동작을 탐색합니다.

1. Power BI Desktop에서 왼쪽의 **모델(Model)** 보기로 전환합니다.

    ![](Images/design-scalable-semantic-models-image1.png)

2. 모델 다이어그램을 사용하여 모델 디자인을 검토합니다.

    ![](Images/design-scalable-semantic-models-image2.png)

3. **Date** 테이블과 **Sales** 테이블 사이에 세 가지 관계(relationship)가 있음을 확인하세요.

    ![](Images/design-scalable-semantic-models-image3.png)

    > **Date** 테이블의 **Date** 열은 관계(relationship)의 "하나(one)" 측면을 나타내는 고유한 열입니다. **Date** 테이블의 어떤 열에 적용된 필터도 관계 중 하나를 사용하여 **Sales** 테이블로 전파됩니다.

4. 세 가지 관계(relationship) 각각 위에 커서를 올려놓아 **Sales** 테이블의 "다수(many)" 측면 열을 강조 표시하세요.

5. **Date**와 **OrderDate** 사이의 관계(relationship)가 활성 상태임을 확인하세요. 현재 모델 디자인은 **Date** 테이블이 역할 분담 차원(role-playing dimension)임을 나타냅니다. 이 차원은 주문 날짜, 기한 날짜 또는 배송 날짜의 역할을 할 수 있습니다. 어떤 역할인지는 보고서의 분석 요구 사항에 따라 달라집니다.

> 나중에 DAX를 사용하여 다른 날짜 열에 대한 두 개의 활성 관계(relationship)를 얻기 위해 또 다른 테이블을 만들 필요 없이 이러한 비활성 관계(relationship)를 사용합니다.

### 날짜별 매출 데이터 시각화

이 작업에서는 연도별 총 매출을 시각화하고 비활성 관계(relationship)를 사용합니다.

1. **보고서(Report)** 보기로 전환합니다.

    ![Screenshot of the Report view.](Images/design-scalable-semantic-models-image4.png)

2. 테이블 시각적 개체를 추가하려면 **시각화(Visualizations)** 창에서 **테이블(Table)** 시각적 개체 아이콘을 선택합니다.

    ![Screenshot with table visual selected.](Images/design-scalable-semantic-models-image5.png)

3. 테이블 시각적 개체에 열을 추가하려면 (오른쪽에 있는) **데이터(Data)** 창에서 먼저 **Date** 테이블을 확장합니다.

4. **Year** 열을 끌어서 테이블 시각적 개체로 놓습니다.

5. **Sales** 테이블을 확장한 다음 **Total Sales** 열을 끌어서 테이블 시각적 개체로 놓습니다.

6. 테이블 시각적 개체를 검토합니다.

![Table visual with Year and Total Sales.](Images/table-year-total-sales.png)

> 테이블 시각적 개체는 연도별로 그룹화된 **Total Sales** 열의 합계를 보여줍니다. 그러나 **Year**는 무엇을 의미할까요? **Date** 테이블과 **Sales** 테이블 사이에 **OrderDate** 열에 대한 활성 관계(relationship)가 있기 때문에 **Year**는 주문이 이루어진 회계 연도를 의미합니다.

### 비활성 관계(Relationship) 사용

이 작업에서는 `USERELATIONSHIP` 함수를 사용하여 비활성 관계(relationship)를 활성화합니다.

1. **데이터(Data)** 창에서 **Sales** 테이블을 마우스 오른쪽 버튼으로 클릭한 다음 **새 측정값(New measure)**을 선택합니다.

    ![](Images/design-scalable-semantic-models-image10.png)

2. (리본 아래에 있는) 수식 입력줄에서 텍스트를 다음 측정값 정의로 바꾼 다음 **Enter** 키를 누릅니다.

    ```DAX
    Sales Shipped =
    CALCULATE (
    SUM ('Sales'[Sales]),
    USERELATIONSHIP('Date'[Date], 'Sales'[ShipDate])
    )
    ```

    > 이 수식은 CALCULATE 함수를 사용하여 필터 컨텍스트를 수정합니다. 이 측정값에 대해서만 **ShipDate** 관계(relationship)를 활성화하는 것은 USERELATIONSHIP 함수입니다.

3. **Sales Shipped** 측정값을 테이블 시각적 개체에 추가합니다.

4. 모든 열이 완전히 보이도록 테이블 시각적 개체를 넓힙니다. **Total** 행은 같지만 **Total Sales** 및 **Sales Shipped**의 각 연도별 매출 금액이 다름을 확인하세요. 이 차이는 특정 연도에 주문이 접수되었지만 다음 연도에만 배송되거나 아직 배송되지 않았기 때문입니다.

![Table with Year, Total Sales, and Sales Shipped.](Images/relationship-table-final.png)

> 관계(relationship)를 일시적으로 활성 상태로 설정하는 측정값을 생성하는 것은 역할 분담 차원(role-playing dimension)으로 작업하는 한 가지 방법입니다. 그러나 많은 측정값에 대해 역할 분담 버전(role-playing version)을 생성해야 할 경우 번거로울 수 있습니다. 예를 들어, 10개의 매출 관련 측정값과 3개의 역할 분담 날짜가 있다면 30개의 측정값을 생성해야 할 수도 있습니다. Calculation groups로 생성하면 이 프로세스가 더 쉬워집니다.

## Calculation groups 만들기

이 작업에서는 시간 인텔리전스(Time Intelligence) 분석을 위한 Calculation group을 생성합니다.

1. **모델(Model)** 보기로 전환합니다.

2. 모델 보기에서 **계산 그룹(Calculation Group)**을 선택하여 새 계산 그룹 테이블, 그룹 열 및 항목을 만듭니다. 경고 창이 나타나면 **예(Yes)**를 선택하여 Calculation group 생성을 확인합니다.

   ![](Images/design-scalable-semantic-models-image13.png)

    > 참고: 암시적 측정값(implicit measure)은 보고서 보기에서 데이터 창의 데이터 열을 시각적 개체에서 직접 사용할 때 발생합니다. 시각적 개체는 이를 SUM, AVERAGE, MIN, MAX 또는 기타 기본 집계로 집계할 수 있도록 하며, 이는 암시적 측정값이 됩니다. Calculation group을 생성하면 Power BI Desktop은 더 이상 암시적 측정값을 생성하지 않으므로, 데이터 열을 집계하려면 측정값을 명시적으로 생성해야 합니다.

3. Calculation group의 이름을 *Time Calculations*로, 계산 열의 이름을 *Yearly Calculations*로 변경합니다.

4. **데이터(Data)** 창의 **모델(Model)** 탭에서 Calculation group으로 자동 생성된 계산 항목을 선택합니다.

5. 항목의 수식을 다음으로 바꾸고 커밋합니다.

    ```DAX
   Year-to-Date (YTD) = CALCULATE(SELECTEDMEASURE(), DATESYTD('Date'[Date]))
    ```

6. **계산 항목(Calculation items)** 필드를 마우스 오른쪽 버튼으로 클릭하고 **새 계산 항목(New calculation item)**을 선택합니다.

7. 새 항목에 다음 DAX 수식을 사용합니다.

    ```DAX
   Previous Year (PY) = CALCULATE(SELECTEDMEASURE(), PREVIOUSYEAR('Date'[Date]))
    ```

8. 다음 DAX 수식으로 세 번째 항목을 만듭니다.

    ```DAX
   Year-over-Year (YoY) Growth = 
   VAR MeasurePriorYear =
   CALCULATE(
       SELECTEDMEASURE(),
       SAMEPERIODLASTYEAR('Date'[Date])
   )
   RETURN
   DIVIDE(
       (SELECTEDMEASURE() - MeasurePriorYear),
       MeasurePriorYear
   )
    ```

마지막 계산 항목은 백분율 값만 반환해야 하므로, 영향을 미치는 측정값의 형식을 변경하기 위한 동적 형식 문자열(dynamic format string)이 필요합니다.

9. YoY 항목의 **속성(Properties)** 창에서 **동적 형식 문자열(Dynamic format string)** 기능을 활성화합니다.

10. DAX 수식 입력줄에서 왼쪽 필드가 **형식(Format)**으로 설정되어 있는지 확인하고 다음 형식 문자열을 작성합니다: `"0.##%"`

11. Calculation group이 다음과 같이 보이는지 확인합니다.

    ![](Images/design-scalable-semantic-models-image14.png)

### 측정값에 Calculation group 적용

이 작업에서는 계산 항목이 시각적 개체의 측정값에 어떻게 영향을 미치는지 시각화합니다.

1. **보고서(Report)** 보기로 전환합니다.

2. 캔버스 하단에서 **개요(Overview)** 탭을 선택합니다.

3. 캔버스에 이미 생성된 매트릭스 시각적 개체를 선택하고 **데이터(Data)** 창에서 **Yearly Calculations** 계산 열을 **시각화(Visualizations)** 창의 **열(Columns)** 필드로 끌어 놓습니다.

    ![](Images/design-scalable-semantic-models-image15.png)

4. 이제 매트릭스에 각 계산 항목에 대한 매출 수치(sales figures) 세트가 있음을 확인합니다.

   ![Screenshot of the matrix with the Yearly Calculations added.](Images/calculation-group-matrix.png)

> 이 모든 정보를 하나의 시각적 개체에서 한 번에 보는 것은 읽기 어려울 수 있으므로, 시각적 개체를 한 번에 하나의 매출 수치로 제한하는 것이 편리합니다. 이를 위해 Field parameter를 사용할 수 있습니다.

## Field parameters 만들기

이 작업에서는 시각적 개체를 변경하기 위한 Field parameters를 생성합니다.

1. 상단 리본에서 **모델링(Modeling)** 탭을 선택한 다음 **새 매개 변수(New parameter)** 버튼을 확장하고 **필드(Fields)**를 선택합니다.

    ![](Images/design-scalable-semantic-models-image16.png)

2. 매개 변수 창에서 매개 변수 이름을 **Sales Figures**로 변경하고, **이 페이지에 슬라이서 추가(Add slicer to this page)** 옵션이 선택되어 있는지 확인한 다음 **Sales** 테이블에서 다음 필드를 추가합니다.

   - Total Sales
   - Profit
   - Profit Margin
   - Orders

    ![](Images/design-scalable-semantic-models-image17.png)

3. **만들기(Create)**를 선택합니다.

4. 슬라이서가 생성되면 매트릭스를 선택하고 시각화 창의 **값(Values)**에서 모든 필드를 제거한 다음 대신 Sales Figures Field parameter를 추가할 수 있습니다.

    ![](Images/design-scalable-semantic-models-image18.png)

5. 슬라이서에서 다른 매출 수치(sales figures)를 확인하고, 각각을 선택할 때 매트릭스가 어떻게 변경되는지 확인합니다.

6. Sales Figures Field parameter에 대한 슬라이서를 사용하여 Profit 필드가 어떻게 선택되는지 확인하세요. 이것은 위에 있는 것과 동일한 매트릭스이며, 슬라이서 때문에 세 가지 계산 항목(PY, YoY, YTD)이 Profit에만 적용되는 것을 볼 수 있습니다.

    ![Screenshot of matrix with calculation group and field parameters and one field selected.](Images/matrix-parameters-selected.png)

### Field parameters 편집

이 작업에서는 DAX 식을 직접 수정하여 **Sales Figures** Field parameter를 편집합니다.

1. 캔버스 하단의 **Salesperson Performance** 탭을 선택합니다. 월별 매출(Sales by Month)과 월별 목표(Target by Month) 사이에서 차트를 전환하는 묶음 막대형 차트(clustered bar chart)를 확인하세요.

    > 책갈피 버튼(bookmark button)을 생성하여 각 옵션으로 시각적 개체 유형을 변경할 수 있지만, 여러 측정값 사이를 전환해야 하는 경우 각 측정값에 대해 책갈피 버튼을 생성해야 하므로 매우 시간이 많이 소요될 수 있습니다. 대신, 분석하려는 모든 측정값을 포함하는 Field parameter를 사용하여 측정값 사이를 빠르게 전환할 수 있습니다.

    ![Salesperson performance page before changes.](Images/field-parameters-bar-chart-start.png)

2. 막대형 차트 시각적 개체를 선택하고 **X축(X-axis)**의 **Total Sales** 필드를 **Sales Figures** Field parameter로 바꿉니다.

3. **슬라이서(Slicer)** 시각적 개체를 생성하고 **Sales Figures** 매개 변수를 **필드(Field)** 영역으로 끌어 놓습니다.

이 시각적 개체의 경우, 아직 Field parameter에 없는 월별 목표(Target by Month)를 평가해야 합니다.

4. 데이터 창에서 **Sales Figures** 매개 변수를 선택하고 매개 변수의 DAX 식에 Target 필드를 다음과 같이 추가합니다.

    ```DAX
   Sales Figures = {
    ("Total Sales", NAMEOF('Sales'[Total Sales]), 0),
    ("Profit", NAMEOF('Sales'[Profit]), 1),
    ("Profit Margin", NAMEOF('Sales'[Profit Margin]), 2),
    ("Orders", NAMEOF('Sales'[Orders]), 3),
    ("Target", NAMEOF('Targets'[Target]), 4)
   }
    ```

5. 변경 사항을 커밋하고 다른 매출 수치(Sales figures)를 선택할 때 시각적 개체가 변경되는지 확인합니다.

6. 책갈피 버튼(bookmark button)을 삭제하고 보고서 페이지의 최종 상태를 확인합니다.

    ![Final state of Salesperson Performance with field parameters.](Images/field-parameters-bar-chart-final.png)

## 랩 완료

연습을 마치려면 Power BI Desktop을 닫으세요. 파일을 저장할 필요는 없습니다.