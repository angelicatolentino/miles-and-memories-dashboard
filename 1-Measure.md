# _Measure

Base KPIs, colour rules, text and captions, and the readers for buttons. (124 measures)

## Action Tag

```dax
Action Tag =
IF (
    ISINSCOPE ( Dim_Product_Subcategories[SubcategoryName] ),
    [Subcategory Action],
    [Category Action]
)
```

## Active Page

```dax
Active Page =
SELECTEDVALUE ( 'Nav Page'[Order], 0 )
```

## Active Segment View

```dax
Active Segment View =
SELECTEDVALUE ( 'Segment View'[Order], 1 )
```

## Active View

```dax
Active View =
SELECTEDVALUE ( 'View Toggle'[Order], 1 )
```

## AOV Caption

```dax
AOV Caption =
"orders, AOV $" & FORMAT([Average Order Value (AOV)], "#,0")
```

## AOV Display

```dax
AOV Display =
VAR v = [Average Order Value (AOV)]
RETURN
    IF ( v >= 1000, FORMAT ( v / 1000, "$0.00" ) & "K", FORMAT ( v, "$0.00" ) )
```

## Average Order Value (AOV)

```dax
Average Order Value (AOV) =
VAR _TotalRevenue = [Total Revenue]

VAR _TotalOrders = DISTINCTCOUNT(Fact_Sales[OrderNumber])

RETURN
    DIVIDE(_TotalRevenue,_TotalOrders,0) +0

    --IFERROR(_TotalRevenue / _TotalOrders,0)
```

## Average Unit Revenue

```dax
Average Unit Revenue =
DIVIDE ( [Total Revenue], [Total Unit Sold] )
```

## Basket Effect Callout

```dax
Basket Effect Callout =
VAR HasData = NOT ISBLANK ( [Bike Plus Accessory Margin %] ) && NOT ISBLANK ( [Bike Only Margin %] )
VAR MarginDir = IF ( [Bike Plus Accessory Margin %] >= [Bike Only Margin %], "rises", "falls" )
RETURN
    IF (
        HasData,
        "Margin also " & MarginDir & ", " & FORMAT ( [Bike Only Margin %], "0.0%" ) & " to " & FORMAT ( [Bike Plus Accessory Margin %], "0.0%" ) & ".",
        "No bike orders with an accessory in these dates. Accessories launched in July 2022."
    )
```

## Basket Gap Callout

```dax
Basket Gap Callout =
VAR Co = [Company AOV]
VAR Below =
    FILTER (
        ALLSELECTED ( Dim_Territories[Country] ),
        [Total Orders] > 0 && [Average Order Value (AOV)] < Co
    )
VAR N = COUNTROWS ( Below )
VAR NameList =
    CONCATENATEX ( Below, Dim_Territories[Country], IF ( N = 2, " and ", ", " ), Dim_Territories[Country], ASC )
VAR CoTxt = "$" & FORMAT ( Co, "#,0.00" )
RETURN
    IF (
        N = 0,
        "Every market sits at or above the company average basket of " & CoTxt & ".",
        NameList & IF ( N = 1, " sits", " sit" ) & " below the company average basket of " & CoTxt & "."
    )
```

## Basket Gap Display

```dax
Basket Gap Display =
FORMAT ( DIVIDE ( [Biggest Basket AOV], [Smallest Basket AOV] ), "0.00" ) & "x"
```

## Basket Uplift Color

```dax
Basket Uplift Color =
IF ( [Basket Uplift $] >= 0, "#1E9E6B", "#D94F5C" )
```

## Basket Uplift Label

```dax
Basket Uplift Label =
IF ( [Basket Uplift $] >= 0, "higher basket when they do", "lower basket when they do" )
```

## Biggest Basket Caption

```dax
Biggest Basket Caption =
VAR T =
    ADDCOLUMNS (
        FILTER ( ALLSELECTED ( Dim_Territories[Country] ), [Total Orders] > 0 ),
        "@AOV", [Average Order Value (AOV)],
        "@Orders", [Total Orders]
    )
VAR BestRow = TOPN ( 1, T, [@AOV], DESC )
VAR Ord = MAXX ( BestRow, [@Orders] )
RETURN
    MAXX ( BestRow, Dim_Territories[Country] ) & ", on "
        & IF ( Ord >= 1000, FORMAT ( Ord / 1000, "0.00" ) & "K", FORMAT ( Ord, "0" ) ) & " orders"
```

## Bike Only Margin Caption

```dax
Bike Only Margin Caption =
FORMAT ( [Bike Only Margin %], "0.00%" ) & " margin"
```

## Bike Plus Accessory Margin Caption

```dax
Bike Plus Accessory Margin Caption =
FORMAT ( [Bike Plus Accessory Margin %], "0.00%" ) & " margin"
```

## Bridge Label Color

```dax
Bridge Label Color =
VAR s = SELECTEDVALUE ( 'Bridge Step'[Step] )
RETURN
    IF (
        ISBLANK ( s ) || s = "H1 2022",
        "#0B2D4D",
        IF ( [Bridge Value] < 0, "#D94F5C", "#1E9E6B" )
    )
```

## Category Action

```dax
Category Action =
SWITCH (
        SELECTEDVALUE ( Dim_Product_Categories[CategoryName] ), "Bikes", "Reprice", "Accessories", "Promote", "Clothing", "Expand", "Components", "Dormant", "Maintain" )
```

## Category Driver

```dax
Category Driver =
IF (
        ISINSCOPE ( Dim_Product_Categories[CategoryName] ),
        VAR Cats =
            ADDCOLUMNS (
                FILTER ( ALLSELECTED ( Dim_Product_Categories[CategoryName] ), [Total Revenue] > 0 ),
                "@Rev", [Total Revenue],
                "@Margin", [Profit Margin %]
            )
        VAR TopRev = MAXX ( TOPN ( 1, Cats, [@Rev], DESC ), Dim_Product_Categories[CategoryName] )
        VAR TopMargin = MAXX ( TOPN ( 1, Cats, [@Margin], DESC ), Dim_Product_Categories[CategoryName] )
        VAR ThisCat = SELECTEDVALUE ( Dim_Product_Categories[CategoryName] )
        RETURN
            IF ( ThisCat = TopRev, "Revenue driver", IF ( ThisCat = TopMargin, "Margin driver" ) )
    )
```

## Continent Bar Color

```dax
Continent Bar Color =
VAR Cur = [Total Revenue]
VAR Best = MAXX ( ALLSELECTED ( Dim_Territories[Continent] ), [Total Revenue] )
RETURN
    IF ( Cur = Best, "#1769C2", "#6DB7E8" )
```

## Date Range Label

```dax
Date Range Label =
VAR StartDate = MIN ( Dim_Calendar[Date] )
VAR EndDate = MAX ( Dim_Calendar[Date] )
VAR StartText = FORMAT ( StartDate, "mmm yyyy" )
VAR EndText = FORMAT ( EndDate, "mmm yyyy" )
RETURN
    IF ( StartText = EndText, StartText, StartText & " – " & EndText )
```

## Fastest Growth Caption

```dax
Fastest Growth Caption =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
VAR Valid = FILTER ( T, NOT ISBLANK ( [@G] ) )
RETURN MAXX ( TOPN ( 1, Valid, [@G], DESC ), Dim_Territories[Country] ) & ", trailing 3mo vs prior 3mo"
```

## Fastest Growth Color

```dax
Fastest Growth Color =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
VAR V = MAXX ( FILTER ( T, NOT ISBLANK ( [@G] ) ), [@G] )
RETURN
    IF ( ISBLANK ( V ), "#6B7F91", IF ( V >= 0, "#1E9E6B", "#D94F5C" ) )
```

## Fastest Growth Display

```dax
Fastest Growth Display =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
VAR Best = MAXX ( FILTER ( T, NOT ISBLANK ( [@G] ) ), [@G] )
RETURN IF ( ISBLANK ( Best ), "n/a", IF ( Best >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( Best ), "0.00%" ) )
```

## Greeting Date

```dax
Greeting Date =
"Today is " & FORMAT(TODAY(), "mmmm d, yyyy") & "   |   " & FORMAT(NOW(), "hh:mm AM/PM"
)
```

## Greeting Text

```dax
Greeting Text =
VAR _Hour = HOUR(NOW())
RETURN
SWITCH(
    TRUE(),
    _Hour < 12, "Good morning, Adventurer!",
    _Hour < 18, "Good afternoon, Adventurer!",
    "Good evening, Adventurer!"
)
```

## Gross Profit Caption

```dax
Gross Profit Caption =
"gross profit at " & FORMAT([Profit Margin %], "0%")
```

## Group Avg Spend

```dax
Group Avg Spend =
SWITCH (
    SELECTEDVALUE ( 'Customer Group'[Group] ),
    "Home owners", CALCULATE ( [Revenue per Customer], Dim_Customers[HomeOwner] = "Y" ),
    "Renters", CALCULATE ( [Revenue per Customer], Dim_Customers[HomeOwner] = "N" ),
    "Bachelors degree", CALCULATE ( [Revenue per Customer], Dim_Customers[EducationLevel] = "Bachelors" )
)
```

## Group Customers

```dax
Group Customers =
SWITCH (
    SELECTEDVALUE ( 'Customer Group'[Group] ),
    "Home owners", CALCULATE ( [Total Customers], Dim_Customers[HomeOwner] = "Y" ),
    "Renters", CALCULATE ( [Total Customers], Dim_Customers[HomeOwner] = "N" ),
    "Bachelors degree", CALCULATE ( [Total Customers], Dim_Customers[EducationLevel] = "Bachelors" )
)
```

## Growth Second Color

```dax
Growth Second Color =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
VAR Valid = FILTER ( T, NOT ISBLANK ( [@G] ) )
VAR V = MINX ( TOPN ( 2, Valid, [@G], DESC ), [@G] )
RETURN
    IF ( ISBLANK ( V ), "#6B7F91", IF ( V >= 0, "#1E9E6B", "#D94F5C" ) )
```

## Growth Second Display

```dax
Growth Second Display =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
VAR Valid = FILTER ( T, NOT ISBLANK ( [@G] ) )
RETURN FORMAT ( MINX ( TOPN ( 2, Valid, [@G], DESC ), [@G] ), "+0.00%;-0.00%" )
```

## Growth Second Name

```dax
Growth Second Name =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
VAR Valid = FILTER ( T, NOT ISBLANK ( [@G] ) )
RETURN MAXX ( TOPN ( 1, TOPN ( 2, Valid, [@G], DESC ), [@G], ASC ), Dim_Territories[Country] )
```

## Growth Slowest Color

```dax
Growth Slowest Color =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
VAR V = MINX ( FILTER ( T, NOT ISBLANK ( [@G] ) ), [@G] )
RETURN
    IF ( ISBLANK ( V ), "#6B7F91", IF ( V >= 0, "#1E9E6B", "#D94F5C" ) )
```

## Growth Slowest Display

```dax
Growth Slowest Display =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
RETURN FORMAT ( MINX ( FILTER ( T, NOT ISBLANK ( [@G] ) ), [@G] ), "+0.00%;-0.00%" )
```

## Growth Slowest Name

```dax
Growth Slowest Name =
VAR T = ADDCOLUMNS ( ALLSELECTED ( Dim_Territories[Country] ), "@G", [Trailing 3-Month Growth %] )
VAR Valid = FILTER ( T, NOT ISBLANK ( [@G] ) )
RETURN MAXX ( TOPN ( 1, Valid, [@G], ASC ), Dim_Territories[Country] ) & ", the slowest"
```

## Growth Source Callout

```dax
Growth Source Callout =
VAR T =
    ADDCOLUMNS (
        ALL ( Dim_Product_Categories[CategoryName] ),
        "@Gr", [H1 Revenue Latest] - [H1 Revenue Prior]
    )
VAR GrowthTotal = SUMX ( T, [@Gr] )
VAR TopGr = MAXX ( T, [@Gr] )
VAR TopName = MAXX ( FILTER ( T, [@Gr] = TopGr ), Dim_Product_Categories[CategoryName] )
VAR TopShare = DIVIDE ( TopGr, GrowthTotal )
VAR Others = FILTER ( T, Dim_Product_Categories[CategoryName] <> TopName )
VAR OtherNames =
    CONCATENATEX ( Others, Dim_Product_Categories[CategoryName], " and ", Dim_Product_Categories[CategoryName], ASC )
VAR FirstOrderDate =
    CALCULATE (
        MIN ( Fact_Sales[OrderDate] ),
        ALL ( Dim_Calendar ),
        FILTER ( ALL ( Dim_Product_Categories ), Dim_Product_Categories[CategoryName] <> TopName )
    )
RETURN
    TopName & " drive " & FORMAT ( TopShare, "0.00%" ) & " of the growth. " & OtherNames & " add "
        & FORMAT ( 1 - TopShare, "0.00%" ) & ", and none of it sold before " & FORMAT ( FirstOrderDate, "mmmm yyyy" ) & "."
```

## H1 Growth Color

```dax
H1 Growth Color =
VAR g = [H1 Growth %]
RETURN
    IF (
        ISBLANK ( g ), "#9DB0C0",
        IF ( g < 0, "#D94F5C", "#1E9E6B" )
    )
```

## H1 Growth Display

```dax
H1 Growth Display =
VAR g = [H1 Growth %]
RETURN
    IF (
        ISBLANK ( g ),
        "new line",
        IF ( g < 0, "▼ ", "▲ " ) & FORMAT ( ABS ( g ), "0.00%" )
    )
```

## Income Band Callout

```dax
Income Band Callout =
VAR T =
    ADDCOLUMNS (
        VALUES ( 'Customer RFM'[Income Band] ),
        "@Avg", [Income Band Avg Spend]
    )
VAR Hi = TOPN ( 1, T, [@Avg], DESC )
VAR Lo = TOPN ( 1, T, [@Avg], ASC )
VAR HiName = MAXX ( Hi, 'Customer RFM'[Income Band] )
VAR LoName = MAXX ( Lo, 'Customer RFM'[Income Band] )
VAR Ratio = DIVIDE ( MAXX ( Hi, [@Avg] ), MAXX ( Lo, [@Avg] ) )
RETURN
    HiName & " outspends " & LoName & " " & FORMAT ( Ratio, "0.00" ) & "× per customer"
```

## Latest Month Growth Display

```dax
Latest Month Growth Display =
VAR g = [Latest Month Growth %]
RETURN
    IF ( ISBLANK ( g ), "n/a", IF ( g < 0, "▼ ", "▲ " ) & FORMAT ( ABS ( g ), "0.00%" ) )
```

## Latest Month Note

```dax
Latest Month Note =
VAR LatestDate = CALCULATE ( MAX ( Dim_Calendar[Date] ), ALL ( Dim_Calendar ) )
VAR MonthStart = DATE ( YEAR ( LatestDate ), MONTH ( LatestDate ), 1 )
VAR BestEver = MAXX ( ALL ( Dim_Calendar[Start of Month] ), [Total Revenue] )
RETURN
    "on " & FORMAT ( EDATE ( MonthStart, -1 ), "mmmm" )
        & IF ( [Latest Month Revenue] >= BestEver, ", the best month yet", "" )
```

## Margin Callout

```dax
Margin Callout =
VAR AccKey =
    LOOKUPVALUE ( Dim_Product_Categories[ProductCategoryKey], Dim_Product_Categories[CategoryName], "Accessories" )
VAR Subs =
    ADDCOLUMNS (
        FILTER ( ALLSELECTED ( Dim_Product_Subcategories[SubcategoryName] ), [Total Revenue] > 0 ),
        "@Rev", [Total Revenue],
        "@Margin", [Profit Margin %],
        "@CatKey", CALCULATE ( MAX ( Dim_Product_Subcategories[ProductCategoryKey] ) )
    )
VAR Acc = FILTER ( Subs, [@CatKey] = AccKey )
VAR AccCount = COUNTROWS ( Acc )
VAR AccMin = MINX ( Acc, [@Margin] )
VAR Weakest = TOPN ( 1, Subs, [@Margin], ASC )
VAR WeakName = MAXX ( Weakest, Dim_Product_Subcategories[SubcategoryName] )
VAR WeakMargin = MAXX ( Weakest, [@Margin] )
RETURN
    "All " & AccCount & " accessory lines earn at least " & FORMAT ( AccMin, "0.00%" )
        & ". The weakest line in the catalogue is " & WeakName & ", at " & FORMAT ( WeakMargin, "0.00%" ) & "."
```

## Margin Spotlight

```dax
Margin Spotlight =
VAR AccKey =
    LOOKUPVALUE ( Dim_Product_Categories[ProductCategoryKey], Dim_Product_Categories[CategoryName], "Accessories" )
VAR Subs =
    ADDCOLUMNS (
        FILTER ( ALLSELECTED ( Dim_Product_Subcategories[SubcategoryName] ), [Total Revenue] > 0 ),
        "@Rev", [Total Revenue],
        "@Margin", [Profit Margin %],
        "@CatKey", CALCULATE ( MAX ( Dim_Product_Subcategories[ProductCategoryKey] ) )
    )
VAR TopAcc = TOPN ( 4, FILTER ( Subs, [@CatKey] = AccKey ), [@Rev], DESC )
VAR Weakest = TOPN ( 1, Subs, [@Margin], ASC )
VAR ThisSub = SELECTEDVALUE ( Dim_Product_Subcategories[SubcategoryName] )
RETURN
    IF (
        ThisSub IN SELECTCOLUMNS ( TopAcc, "@n", Dim_Product_Subcategories[SubcategoryName] )
            || ThisSub IN SELECTCOLUMNS ( Weakest, "@n", Dim_Product_Subcategories[SubcategoryName] ),
        1,
        0
    )
```

## MoM Fall Callout

```dax
MoM Fall Callout =
VAR Hist =
    ADDCOLUMNS (
        FILTER ( ALL ( 'Sales Trend & Forecast' ), 'Sales Trend & Forecast'[Is Forecast] = FALSE () ),
        "@Prev",
            LOOKUPVALUE (
                'Sales Trend & Forecast'[Period Sales],
                'Sales Trend & Forecast'[MonthIndex], 'Sales Trend & Forecast'[MonthIndex] - 1,
                'Sales Trend & Forecast'[Is Forecast], FALSE ()
            )
    )
VAR LastIdx = MAXX ( Hist, 'Sales Trend & Forecast'[MonthIndex] )
VAR Win =
    ADDCOLUMNS (
        FILTER ( Hist, 'Sales Trend & Forecast'[MonthIndex] > LastIdx - 12 && NOT ISBLANK ( [@Prev] ) ),
        "@G", DIVIDE ( 'Sales Trend & Forecast'[Period Sales] - [@Prev], [@Prev] )
    )
VAR Falls = FILTER ( Win, [@G] < -0.05 )
VAR N = COUNTROWS ( Falls )
VAR Worst = MINX ( Falls, [@G] )
VAR WorstIdx = MAXX ( FILTER ( Falls, [@G] = Worst ), 'Sales Trend & Forecast'[MonthIndex] )
VAR WorstDate = MAXX ( FILTER ( Win, 'Sales Trend & Forecast'[MonthIndex] = WorstIdx ), 'Sales Trend & Forecast'[Month Date] )
VAR PrevG = MAXX ( FILTER ( Win, 'Sales Trend & Forecast'[MonthIndex] = WorstIdx - 1 ), [@G] )
RETURN
    IF (
        N = 0,
        "Revenue held up in every one of the last 12 months, with no fall of more than 5.00%.",
        IF (
            N = 1,
            FORMAT ( WorstDate, "mmmm" ) & " is the only real fall, down " & FORMAT ( ABS ( Worst ), "0.00%" )
                & IF ( PrevG > 0, ", and it follows the " & FORMAT ( PrevG, "0.00%" ) & " " & FORMAT ( EDATE ( WorstDate, -1 ), "mmmm" ) & " jump", "" ) & ".",
            "There were " & N & " falls of more than 5.00% in the last 12 months. The largest was in " & FORMAT ( WorstDate, "mmmm" ) & ", down " & FORMAT ( ABS ( Worst ), "0.00%" ) & "."
        )
    )
```

## MoM Fill Color

```dax
MoM Fill Color =
IF ( [MoM Chart Growth %] < 0, "#CFE3F5", "#1769C2" )
```

## MoM Growth % Display

```dax
MoM Growth % Display =
VAR v = [MoM Growth %]
RETURN
SWITCH(
    TRUE(),
    v > 0, "▲ " & FORMAT(v, "0.00%"),
    v < 0, "▼ " & FORMAT(ABS(v), "0.00%"),
    "▬ " & FORMAT(v, "0.00%")
)
```

## MoM Growth Caption

```dax
MoM Growth Caption =
VAR CurrentRev = [Total Revenue]
VAR PriorRev = CALCULATE([Total Revenue], DATEADD('Dim_Calendar'[Date], -1, MONTH))
VAR Delta = CurrentRev - PriorRev
VAR PriorMonthDate = CALCULATE(MAX('Dim_Calendar'[Date]), DATEADD('Dim_Calendar'[Date], -1, MONTH))
VAR PriorLabel = FORMAT(PriorMonthDate, "MMM YYYY")
VAR DirectionWord = IF(Delta >= 0, "Growing", "Declining")
VAR ChangeSign = IF(Delta >= 0, "+", "-")
VAR Result = ChangeSign & "$" & FORMAT(ABS(Delta) / 1000, "#,0") & "K vs " & PriorLabel & " - " & DirectionWord
RETURN
Result
```

## MoM Label Color

```dax
MoM Label Color =
IF ( [MoM Chart Growth %] < 0, "#D94F5C", "#1E9E6B" )
```

## Order Growth Display

```dax
Order Growth Display =
VAR FirstOrderDate = CALCULATE ( MIN ( Fact_Sales[OrderDate] ), ALL ( Fact_Sales ) )
VAR FirstOrders =
    CALCULATE (
        [Total Orders],
        FILTER (
            ALL ( Dim_Calendar ),
            YEAR ( Dim_Calendar[Date] ) = YEAR ( FirstOrderDate )
                && MONTH ( Dim_Calendar[Date] ) = MONTH ( FirstOrderDate )
        )
    )
VAR g = DIVIDE ( [Latest Month Orders] - FirstOrders, FirstOrders )
RETURN
    IF ( ISBLANK ( g ), "n/a", IF ( g < 0, "▼ ", "▲ " ) & FORMAT ( ABS ( g ), "#,0.00%" ) )
```

## Order Growth Note

```dax
Order Growth Note =
VAR FirstOrderDate = CALCULATE ( MIN ( Fact_Sales[OrderDate] ), ALL ( Fact_Sales ) )
VAR FirstOrders =
    CALCULATE (
        [Total Orders],
        FILTER (
            ALL ( Dim_Calendar ),
            YEAR ( Dim_Calendar[Date] ) = YEAR ( FirstOrderDate )
                && MONTH ( Dim_Calendar[Date] ) = MONTH ( FirstOrderDate )
        )
    )
RETURN
    "vs " & FORMAT ( FirstOrders, "#,0" ) & " orders in " & FORMAT ( FirstOrderDate, "mmm yyyy" )
```

## Orders per Customer

```dax
Orders per Customer =
DIVIDE ( [Total Orders], [Total Customers] )
```

## Orders per Customer Caption

```dax
Orders per Customer Caption =
FORMAT ( DIVIDE ( [Total Orders], [Total Customers] ), "0.00" ) & " orders per customer"
```

## Profit Margin %

```dax
Profit Margin % =
DIVIDE(
    [Total Profit],[Total Revenue]
)+0
```

## Repeat Customer Rate %

```dax
Repeat Customer Rate % =
VAR RepeatCustomers =
    CALCULATE (
        DISTINCTCOUNT ( Fact_Sales[CustomerKey] ),
        FILTER (
            VALUES ( Fact_Sales[CustomerKey] ),
            CALCULATE ( DISTINCTCOUNT ( Fact_Sales[OrderNumber] ) ) > 1
        )
    )
RETURN
    DIVIDE ( RepeatCustomers, [Total Customers] )
```

## Repeat Rate Caption

```dax
Repeat Rate Caption =
FORMAT ( [Repeat Customer Rate %], "0.00%" ) & " order more than once"
```

## Returning Orders Caption

```dax
Returning Orders Caption =
VAR FirstReturn =
    MINX (
        FILTER ( ALL ( Dim_Calendar[Start of Month] ), [Returning Customer Orders] > 0 ),
        Dim_Calendar[Start of Month]
    )
VAR LatestDate = CALCULATE ( MAX ( Dim_Calendar[Date] ), ALL ( Dim_Calendar ) )
VAR LatestMonth =
    FILTER (
        ALL ( Dim_Calendar ),
        YEAR ( Dim_Calendar[Date] ) = YEAR ( LatestDate )
            && MONTH ( Dim_Calendar[Date] ) = MONTH ( LatestDate )
    )
VAR ReturnShare =
    DIVIDE (
        CALCULATE ( [Returning Customer Orders], LatestMonth ),
        CALCULATE ( [Total Orders], LatestMonth )
    )
RETURN
    "Returning orders were zero before " & FORMAT ( FirstReturn, "mmmm yyyy" )
        & ", since no one had a chance to return yet. By " & FORMAT ( LatestDate, "mmmm yyyy" )
        & " they are " & FORMAT ( ReturnShare, "0.00%" ) & " of monthly orders."
```

## Revenue by Category Callout

```dax
Revenue by Category Callout =
"Bikes remain the core of the business at " & FORMAT ( [Revenue Share % (Bikes)], "0.00%" ) & " of revenue, but accessories earn " & FORMAT ( [Margin % (Accessories)], "0.00%" ) & " margin against " & FORMAT ( [Margin % (Bikes)], "0.00%" ) & "."
```

## Revenue per Customer

```dax
Revenue per Customer =
DIVIDE ( [Total Revenue], [Total Customers] )
```

## Revenue Share Caption (Accessories)

```dax
Revenue Share Caption (Accessories) =
FORMAT([Revenue Share % (Accessories)], "0.00%") & " of revenue"
```

## RPC Callout

```dax
RPC Callout =
VAR T =
    ADDCOLUMNS (
        FILTER ( ALLSELECTED ( Dim_Territories[Country] ), [Total Customers] > 0 ),
        "@R", [Revenue per Customer]
    )
VAR Hi = TOPN ( 1, T, [@R], DESC )
VAR Lo = TOPN ( 1, T, [@R], ASC )
VAR HiV = MAXX ( Hi, [@R] )
VAR LoV = MAXX ( Lo, [@R] )
RETURN
    MAXX ( Hi, Dim_Territories[Country] ) & " leads at "
        & IF ( HiV >= 1000, FORMAT ( HiV / 1000, "$0.00" ) & "K", FORMAT ( HiV, "$0.00" ) )
        & " per customer, " & MAXX ( Lo, Dim_Territories[Country] ) & " trails at "
        & IF ( LoV >= 1000, FORMAT ( LoV / 1000, "$0.00" ) & "K", FORMAT ( LoV, "$0.00" ) ) & "."
```

## Scatter Dot Color

```dax
Scatter Dot Color =
VAR cat =
    MAXX(
        Dim_Products,
        RELATED(Dim_Product_Categories[CategoryName])
    )
RETURN
    SWITCH(
        cat,
        "Bikes", "#1769C2",
        "Accessories", "#1E9E6B",
        "Clothing", "#8E7BD8",
        "#9DB0C0"
    )
```

## Scatter Footnote

```dax
Scatter Footnote =
"Top-quartile revenue ($"
    & FORMAT([Revenue Split (Top Quartile)] / 1000, "#,0")
    & "K+) × "
    & SWITCH(
        [Selected Scatter Mode],
        "Margin", "median margin (" & FORMAT([Scatter Y Split], "0%") & ")",
        "Volume", "median volume (" & FORMAT([Scatter Y Split], "#,0") & " units)"
    )
```

## Seasonality Bar Color

```dax
Seasonality Bar Color =
VAR Cur = [Total Revenue]
VAR Best = MAXX ( ALLSELECTED ( Dim_Calendar[MonthName_Short] ), [Total Revenue] )
RETURN
    IF ( Cur = Best, "#1769C2", "#6DB7E8" )
```

## Seasonality Label Color

```dax
Seasonality Label Color =
VAR Cur = [Total Revenue]
VAR Best = MAXX ( ALLSELECTED ( Dim_Calendar[MonthName_Short] ), [Total Revenue] )
RETURN
    IF ( Cur = Best, "#1769C2", "#0B2D4D" )
```

## Seasonality Peak Display

```dax
Seasonality Peak Display =
VAR Act = FILTER ( ALL ( 'Sales Trend & Forecast' ), 'Sales Trend & Forecast'[Is Forecast] = FALSE () )
VAR LastIdx = MAXX ( Act, 'Sales Trend & Forecast'[MonthIndex] )
VAR Win = FILTER ( Act, 'Sales Trend & Forecast'[MonthIndex] > LastIdx - 12 )
VAR AvgSales = AVERAGEX ( Win, 'Sales Trend & Forecast'[Period Sales] )
VAR BestVal = MAXX ( Win, 'Sales Trend & Forecast'[Period Sales] )
RETURN
    "▲ " & FORMAT ( DIVIDE ( BestVal - AvgSales, AvgSales ), "0.00%" )
```

## Seasonality Peak Text

```dax
Seasonality Peak Text =
VAR Act = FILTER ( ALL ( 'Sales Trend & Forecast' ), 'Sales Trend & Forecast'[Is Forecast] = FALSE () )
VAR LastIdx = MAXX ( Act, 'Sales Trend & Forecast'[MonthIndex] )
VAR Win = FILTER ( Act, 'Sales Trend & Forecast'[MonthIndex] > LastIdx - 12 )
VAR AvgSales = AVERAGEX ( Win, 'Sales Trend & Forecast'[Period Sales] )
VAR BestVal = MAXX ( Win, 'Sales Trend & Forecast'[Period Sales] )
VAR BestDate = MAXX ( FILTER ( Win, 'Sales Trend & Forecast'[Period Sales] = BestVal ), 'Sales Trend & Forecast'[Month Date] )
RETURN
    FORMAT ( BestDate, "mmmm" ) & " is the best month, above the $" & FORMAT ( AvgSales / 1000000, "0.00" ) & "M average."
```

## Seasonality Trough Display

```dax
Seasonality Trough Display =
VAR Act = FILTER ( ALL ( 'Sales Trend & Forecast' ), 'Sales Trend & Forecast'[Is Forecast] = FALSE () )
VAR LastIdx = MAXX ( Act, 'Sales Trend & Forecast'[MonthIndex] )
VAR Win = FILTER ( Act, 'Sales Trend & Forecast'[MonthIndex] > LastIdx - 12 )
VAR AvgSales = AVERAGEX ( Win, 'Sales Trend & Forecast'[Period Sales] )
VAR WorstVal = MINX ( Win, 'Sales Trend & Forecast'[Period Sales] )
RETURN
    "▼ " & FORMAT ( DIVIDE ( AvgSales - WorstVal, AvgSales ), "0.00%" )
```

## Seasonality Trough Text

```dax
Seasonality Trough Text =
VAR Act = FILTER ( ALL ( 'Sales Trend & Forecast' ), 'Sales Trend & Forecast'[Is Forecast] = FALSE () )
VAR LastIdx = MAXX ( Act, 'Sales Trend & Forecast'[MonthIndex] )
VAR Win = FILTER ( Act, 'Sales Trend & Forecast'[MonthIndex] > LastIdx - 12 )
VAR WorstVal = MINX ( Win, 'Sales Trend & Forecast'[Period Sales] )
VAR WorstDate = MAXX ( FILTER ( Win, 'Sales Trend & Forecast'[Period Sales] = WorstVal ), 'Sales Trend & Forecast'[Month Date] )
RETURN
    FORMAT ( WorstDate, "mmmm" ) & " is the weakest. " & COUNTROWS ( Win ) & " months only, a first read."
```

## Segment Play

```dax
Segment Play =
SWITCH (
    SELECTEDVALUE ( 'Customer RFM'[Segment] ),
    "High value", "Retain",
    "Lapsing", "Win back",
    "Growing", "Nurture",
    "Loyal", "Reward",
    "At risk", "Reactivate"
)
```

## Segment Table Note

```dax
Segment Table Note =
VAR IsCount = SELECTEDVALUE ( 'Segment View'[Order], 1 ) = 2
VAR LapRiders = CALCULATE ( [Income Band Customers], 'Customer RFM'[Segment] = "Lapsing" )
VAR LapRev = CALCULATE ( [Income Band Revenue], 'Customer RFM'[Segment] = "Lapsing" )
VAR LapRec = CALCULATE ( [Segment Median Recency], 'Customer RFM'[Segment] = "Lapsing" )
VAR LapShare = DIVIDE ( LapRiders, CALCULATE ( [Income Band Customers], ALL ( 'Customer RFM' ) ) )
VAR RiderTxt = FORMAT ( LapRiders / 1000, "0.00" ) & "K"
VAR RevNote =
    "Lapsing is the biggest opportunity: " & RiderTxt & " riders, $"
        & FORMAT ( LapRev / 1000000, "0.00" ) & "M of past spend, median "
        & FORMAT ( LapRec, "0.00" ) & " days since the last order."
VAR CountNote =
    "Lapsing is the largest single segment by headcount: " & RiderTxt & " riders, "
        & FORMAT ( LapShare, "0.00%" ) & " of everyone in the file."
RETURN
    IF ( IsCount, CountNote, RevNote )
```

## Selected Metric

```dax
Selected Metric =
SELECTEDVALUE ( 'Metric Selector'[Metric], "Revenue" )
```

## Smallest Basket Caption

```dax
Smallest Basket Caption =
VAR T =
    ADDCOLUMNS (
        FILTER ( ALLSELECTED ( Dim_Territories[Country] ), [Total Orders] > 0 ),
        "@AOV", [Average Order Value (AOV)]
    )
RETURN MAXX ( TOPN ( 1, T, [@AOV], ASC ), Dim_Territories[Country] ) & ", the smallest"
```

## Strat Action 01

```dax
Strat Action 01 =
VAR Best = [Strat Best Market]
VAR BestAov = CALCULATE ( [Average Order Value (AOV)], Dim_Territories[Country] = Best )
RETURN
    "Test the accessory bundle and free shipping threshold from " & Best & ", where the basket is $"
        & FORMAT ( BestAov, "#,0.00" ) & ", in " & [Strat Weak Market] & "."
```

## Strat Action 02

```dax
Strat Action 02 =
VAR AccBasket =
    DIVIDE (
        CALCULATE ( [Total Revenue], Dim_Product_Categories[CategoryName] = "Accessories" ),
        CALCULATE ( DISTINCTCOUNT ( Fact_Sales[OrderNumber] ), Dim_Product_Categories[CategoryName] = "Accessories" )
    )
RETURN
    "Add the helmet, bottle and tyre bundle to checkout for every bike. An accessory order averages $"
        & FORMAT ( AccBasket, "#,0.00" ) & ", so closing the whole gap adds about $"
        & FORMAT ( [Strat Attach Gap $] / 1000, "#,0.00" ) & "K."
```

## Strat Action 04

```dax
Strat Action 04 =
VAR ShiftShare = 0.25
VAR RoadRev = CALCULATE ( [Total Revenue], Dim_Product_Subcategories[SubcategoryName] = "Road Bikes" )
RETURN
    "Assumes " & FORMAT ( ShiftShare, "0%" ) & " of road revenue (about $"
        & FORMAT ( RoadRev * ShiftShare / 1000000, "0.00" )
        & "M) shifts to mountain, holding total volume constant. Move promotion weight toward the mountain range and review road bike pricing."
```

## Strat Best Market

```dax
Strat Best Market =
VAR T =
    ADDCOLUMNS (
        FILTER ( ALLSELECTED ( Dim_Territories[Country] ), [Total Orders] > 0 ),
        "@Aov", [Average Order Value (AOV)]
    )
RETURN
    MAXX ( TOPN ( 1, T, [@Aov], DESC ), Dim_Territories[Country] )
```

## Strat Evidence 01

```dax
Strat Evidence 01 =
VAR Weak = [Strat Weak Market]
VAR WeakOrders = CALCULATE ( [Total Orders], Dim_Territories[Country] = Weak )
VAR WeakAov = CALCULATE ( [Average Order Value (AOV)], Dim_Territories[Country] = Weak )
VAR WeakMargin = CALCULATE ( [Profit Margin %], Dim_Territories[Country] = Weak )
RETURN
    Weak & " places " & FORMAT ( WeakOrders, "#,0" ) & " orders at $" & FORMAT ( WeakAov, "#,0.00" )
        & " each, against a $" & FORMAT ( [Company AOV], "#,0.00" ) & " company average. Its margin is "
        & FORMAT ( WeakMargin, "0.00%" ) & ", against " & FORMAT ( [Profit Margin %], "0.00%" ) & " for the company."
```

## Strat Evidence 02

```dax
Strat Evidence 02 =
VAR Orders =
    ADDCOLUMNS (
        SUMMARIZE ( Fact_Sales, Fact_Sales[OrderNumber] ),
        "@HasBike", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Bikes" ) > 0,
        "@HasAccessory", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Accessories" ) > 0
    )
VAR BikeOnly = COUNTROWS ( FILTER ( Orders, [@HasBike] && NOT [@HasAccessory] ) )
RETURN
    FORMAT ( [Bike Attach Rate %], "0.00%" ) & " of bike orders already include an accessory. The other "
        & FORMAT ( BikeOnly, "#,0" ) & " do not. Orders with an accessory earn a "
        & FORMAT ( [Bike Plus Accessory Margin %], "0.00%" ) & " margin, against "
        & FORMAT ( [Bike Only Margin %], "0.00%" ) & " without."
```

## Strat Evidence 03

```dax
Strat Evidence 03 =
VAR Lapsing = FILTER ( 'Customer RFM', 'Customer RFM'[Segment] = "Lapsing" )
RETURN
    FORMAT ( COUNTROWS ( Lapsing ), "#,0" ) & " riders hold $"
        & FORMAT ( SUMX ( Lapsing, 'Customer RFM'[Monetary] ) / 1000000, "0.00" )
        & "M of past spend at a median " & FORMAT ( MEDIANX ( Lapsing, 'Customer RFM'[Recency] ), "#,0" )
        & " days since their last order."
```

## Strat Evidence 04

```dax
Strat Evidence 04 =
VAR RoadRev = CALCULATE ( [Total Revenue], Dim_Product_Subcategories[SubcategoryName] = "Road Bikes" )
VAR MtnRev = CALCULATE ( [Total Revenue], Dim_Product_Subcategories[SubcategoryName] = "Mountain Bikes" )
VAR RoadMargin = CALCULATE ( [Profit Margin %], Dim_Product_Subcategories[SubcategoryName] = "Road Bikes" )
VAR MtnMargin = CALCULATE ( [Profit Margin %], Dim_Product_Subcategories[SubcategoryName] = "Mountain Bikes" )
RETURN
    "Road bikes earn " & FORMAT ( RoadMargin, "0.00%" ) & " on $" & FORMAT ( RoadRev / 1000000, "0.00" )
        & "M while mountain bikes earn " & FORMAT ( MtnMargin, "0.00%" ) & " on $"
        & FORMAT ( MtnRev / 1000000, "0.00" ) & "M."
```

## Strat Evidence 05

```dax
Strat Evidence 05 =
VAR TopRiders = FILTER ( 'Customer RFM', 'Customer RFM'[Decile] = 1 )
VAR TopSpend = SUMX ( TopRiders, 'Customer RFM'[Monetary] )
RETURN
    FORMAT ( COUNTROWS ( TopRiders ), "#,0" ) & " riders generate " & FORMAT ( [Top Decile Share %], "0.00%" )
        & " of revenue at $" & FORMAT ( DIVIDE ( TopSpend, COUNTROWS ( TopRiders ) ), "#,0.00" ) & " each."
```

## Strat Evidence 06

```dax
Strat Evidence 06 =
VAR Days =
    ADDCOLUMNS (
        GENERATESERIES ( 1, 7, 1 ),
        "@Rev",
            VAR d = [Value]
            RETURN
                CALCULATE (
                    [Total Revenue],
                    FILTER ( VALUES ( Dim_Calendar[Date] ), WEEKDAY ( Dim_Calendar[Date], 2 ) = d )
                )
    )
VAR Hi = MAXX ( Days, [@Rev] )
VAR Lo = MINX ( Days, [@Rev] )
VAR Spread = DIVIDE ( Hi - Lo, Lo )
RETURN
    IF (
        Spread < 0.1,
        "Revenue varies only " & FORMAT ( Spread, "0.00%" ) & " across the seven days, so day-of-week promotions have no signal to exploit.",
        "Revenue varies " & FORMAT ( Spread, "0.00%" ) & " across the seven days, so day-of-week promotions may be worth testing."
    )
```

## Strat Hero Note

```dax
Strat Hero Note =
"Six actions, three of them high priority. The three revenue actions add up to "
    & FORMAT ( [Strat Revenue Upside] / 1000000, "$0.00" ) & "M of upside at your current slider settings, with "
    & FORMAT ( [Strat At Risk $] / 1000, "$0.00" ) & "K of revenue at risk. Each card opens the page where the number was derived."
```

## Strat Pill 01

```dax
Strat Pill 01 =
VAR v = [Strat Basket Gap $] * [Canada Basket Slider Value] / 100
RETURN
    "Est. +$"
        & IF (
            v >= 1000000, FORMAT ( v / 1000000, "0.00" ) & "M",
            IF ( v >= 1000, FORMAT ( v / 1000, "0.00" ) & "K", FORMAT ( v, "0.00" ) )
        )
        & " revenue"
```

## Strat Pill 02

```dax
Strat Pill 02 =
VAR v = [Strat Attach Gap $] * [Attach Gap Slider Value] / 100
RETURN
    "Est. +$"
        & IF (
            v >= 1000000, FORMAT ( v / 1000000, "0.00" ) & "M",
            IF ( v >= 1000, FORMAT ( v / 1000, "0.00" ) & "K", FORMAT ( v, "0.00" ) )
        )
        & " revenue"
```

## Strat Pill 03

```dax
Strat Pill 03 =
VAR v = [Strat Reactivation $] * [Reactivation Slider Value] / 100
RETURN
    "Est. +$"
        & IF (
            v >= 1000000, FORMAT ( v / 1000000, "0.00" ) & "M",
            IF ( v >= 1000, FORMAT ( v / 1000, "0.00" ) & "K", FORMAT ( v, "0.00" ) )
        )
        & " revenue"
```

## Strat Pill 04

```dax
Strat Pill 04 =
VAR v = [Strat Mix Shift Profit $] * [Mix Shift Slider Value] / 100
RETURN
    "Est. +$"
        & IF (
            v >= 1000000, FORMAT ( v / 1000000, "0.00" ) & "M",
            IF ( v >= 1000, FORMAT ( v / 1000, "0.00" ) & "K", FORMAT ( v, "0.00" ) )
        )
        & " profit"
```

## Strat Pill 05

```dax
Strat Pill 05 =
VAR v = [Strat At Risk $]
RETURN
    "Est. $"
        & IF (
            v >= 1000000, FORMAT ( v / 1000000, "0.00" ) & "M",
            IF ( v >= 1000, FORMAT ( v / 1000, "0.00" ) & "K", FORMAT ( v, "0.00" ) )
        )
        & " at risk"
```

## Strat Slider Text 01

```dax
Strat Slider Text 01 =
[Canada Basket Slider Value] & "% of the " & [Strat Weak Market] & " basket gap applied"
```

## Strat Slider Text 02

```dax
Strat Slider Text 02 =
[Attach Gap Slider Value] & "% of the bike attach gap closed"
```

## Strat Slider Text 03

```dax
Strat Slider Text 03 =
[Reactivation Slider Value] & "% of the assumed 5% reactivation rate applied"
```

## Strat Slider Text 04

```dax
Strat Slider Text 04 =
[Mix Shift Slider Value] & "% of the assumed road-to-mountain shift applied"
```

## Strat Upside Label Color

```dax
Strat Upside Label Color =
VAR s = SELECTEDVALUE ( 'Strat Upside Step'[Step] )
RETURN
    IF (
        ISBLANK ( s ) || s = "Today",
        "#0B2D4D",
        IF ( [Strat Upside Value] < 0, "#D94F5C", "#1E9E6B" )
    )
```

## Strat Weak Market

```dax
Strat Weak Market =
VAR T =
    ADDCOLUMNS (
        FILTER ( ALLSELECTED ( Dim_Territories[Country] ), [Total Orders] > 0 ),
        "@Aov", [Average Order Value (AOV)]
    )
RETURN
    MAXX ( TOPN ( 1, T, [@Aov], ASC ), Dim_Territories[Country] )
```

## Subcategory Action

```dax
Subcategory Action =
VAR CurrentRevenue = [Total Revenue]
VAR CurrentMargin = DIVIDE ( [Total Profit], [Total Revenue] )
VAR MaxRevenueInGroup = MAXX ( ALLSELECTED ( Dim_Product_Subcategories[SubcategoryName] ), [Total Revenue] )
RETURN SWITCH ( TRUE (), CurrentRevenue = MaxRevenueInGroup, "Promote", NOT ISBLANK ( CurrentMargin ) && CurrentMargin < 0.4, "Reprice", "Maintain"
)
```

## Table Note

```dax
Table Note =
VAR Selected = SELECTEDVALUE ( 'View Toggle'[View], "All" )

// ALL view
VAR FirstNonBike =
    CALCULATE ( MIN ( Fact_Sales[OrderDate] ), Dim_Product_Categories[CategoryName] <> "Bikes", ALL ( Dim_Calendar ) )
VAR BikeGrowth =
    CALCULATE ( [H1 Growth %], Dim_Product_Categories[CategoryName] = "Bikes" )
VAR AllNote =
    "Accessories and clothing only appear from " & FORMAT ( FirstNonBike, "mmmm yyyy" )
        & ", so their first half growth has no prior year to compare against. Bikes "
        & IF ( BikeGrowth >= 0, "grew ", "fell " ) & FORMAT ( ABS ( BikeGrowth ), "0.00%" ) & " like for like."

// BIKES view
VAR BikeSubs =
    CALCULATETABLE (
        ADDCOLUMNS ( VALUES ( Dim_Product_Subcategories[SubcategoryName] ), "@Rev", [Total Revenue], "@Growth", [H1 Growth %] ),
        Dim_Product_Categories[CategoryName] = "Bikes"
    )
VAR BikeSold = FILTER ( BikeSubs, [@Rev] > 0 )
VAR BikeTotal = SUMX ( BikeSold, [@Rev] )
VAR BikeTop = TOPN ( 1, BikeSold, [@Rev], DESC )
VAR BikeTopName = MAXX ( BikeTop, Dim_Product_Subcategories[SubcategoryName] )
VAR BikeTopShare = DIVIDE ( MAXX ( BikeTop, [@Rev] ), BikeTotal )
VAR BikeNew = FILTER ( BikeSold, ISBLANK ( [@Growth] ) )
VAR BikeNewCount = COUNTROWS ( BikeNew )
VAR BikeNewNames = CONCATENATEX ( BikeNew, Dim_Product_Subcategories[SubcategoryName], ", ", [@Rev], DESC )
VAR BikeNote =
    BikeTopName & " carries the category at " & FORMAT ( BikeTopShare, "0.00%" ) & " of Bikes revenue. "
        & IF (
            BikeNewCount = 0,
            "Every bike subcategory has a prior-year first half to compare against.",
            BikeNewNames
                & IF (
                    BikeNewCount = 1,
                    " has no prior-year first half to compare against, so its growth reads as a new line.",
                    " have no prior-year first half to compare against, so their growth reads as new lines."
                )
        )

// ACCESSORIES view
VAR AccSubs =
    CALCULATETABLE (
        ADDCOLUMNS ( VALUES ( Dim_Product_Subcategories[SubcategoryName] ), "@Rev", [Total Revenue], "@Growth", [H1 Growth %] ),
        Dim_Product_Categories[CategoryName] = "Accessories"
    )
VAR AccSold = FILTER ( AccSubs, [@Rev] > 0 )
VAR AccTotal = SUMX ( AccSold, [@Rev] )
VAR AccTop = TOPN ( 1, AccSold, [@Rev], DESC )
VAR AccTopName = MAXX ( AccTop, Dim_Product_Subcategories[SubcategoryName] )
VAR AccTopShare = DIVIDE ( MAXX ( AccTop, [@Rev] ), AccTotal )
VAR AccSoldCount = COUNTROWS ( AccSold )
VAR AccWithGrowth = COUNTROWS ( FILTER ( AccSold, NOT ISBLANK ( [@Growth] ) ) )
VAR FirstAcc =
    CALCULATE ( MIN ( Fact_Sales[OrderDate] ), Dim_Product_Categories[CategoryName] = "Accessories", ALL ( Dim_Calendar ) )
VAR AccNote =
    AccTopName & " carries the category at " & FORMAT ( AccTopShare, "0.00%" ) & " of Accessories revenue. "
        & IF (
            AccWithGrowth = 0,
            "Accessories only began selling in " & FORMAT ( FirstAcc, "mmmm yyyy" )
                & ", so none of its subcategories have a prior-year first half to compare against yet.",
            AccWithGrowth & " of " & AccSoldCount & " accessory subcategories have a prior-year first half to compare against."
        )

RETURN
    SWITCH ( Selected, "Bikes", BikeNote, "Accessories", AccNote, AllNote )
```

## Territory Call

```dax
Territory Call =
VAR Aov = [Average Order Value (AOV)]
VAR MeanAov =
    AVERAGEX (
        FILTER ( ALLSELECTED ( Dim_Territories[Country] ), [Total Orders] > 0 ),
        [Average Order Value (AOV)]
    )
VAR G = [Trailing 3-Month Growth %]
RETURN
    IF (
        Aov < MeanAov * 0.85, "Build basket",
        IF ( NOT ISBLANK ( G ) && G > 0.25, "Invest", "Maintain" )
    )
```

## Territory Growth Display

```dax
Territory Growth Display =
VAR G = [Trailing 3-Month Growth %]
RETURN IF ( ISBLANK ( G ), "n/a", IF ( G >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( G ), "0.00%" ) )
```

## Territory Note

```dax
Territory Note =
VAR Sel = SELECTEDVALUE ( 'View Toggle'[View], "All" )
VAR Label = SWITCH ( Sel, "Bikes", "bikes only", "Accessories", "accessories only", "all categories" )
VAR Base =
    CALCULATETABLE (
        ADDCOLUMNS (
            ALLSELECTED ( Dim_Territories[Country] ),
            "@M", [Profit Margin %],
            "@O", [Total Orders]
        ),
        FILTER (
            ALL ( Dim_Product_Categories[CategoryName] ),
            Sel = "All" || Dim_Product_Categories[CategoryName] = Sel
        )
    )
VAR Sold = FILTER ( Base, [@O] > 0 )
VAR Lo = MINX ( Sold, [@M] )
VAR Hi = MAXX ( Sold, [@M] )
RETURN
    "Margin ranges from " & FORMAT ( Lo, "0.00%" ) & " to " & FORMAT ( Hi, "0.00%" )
        & " across markets, " & Label & ", a gap of " & FORMAT ( ( Hi - Lo ) * 100, "0.00" ) & " points."
```

## Top Country Pill Right

```dax
Top Country Pill Right =
FORMAT ( [Top Country Share %], "0.00%" ) & " of revenue"
```

## Top Products Callout Text

```dax
Top Products Callout Text =
VAR ShareTxt = FORMAT ( [Top 10 Revenue Share %], "0.0%" )
VAR ModelCount = [Top 10 Model Count]
VAR ModelTxt = IF ( ModelCount = 1, "the same model", "the same " & ModelCount & " models" )
VAR Result =
    "The top ten products are " & ShareTxt & "% of revenue, and all ten are " & ModelTxt & " in different sizes."
RETURN Result
```

## Top Products Subtitle

```dax
Top Products Subtitle =
VAR Top5 =
    TOPN(
        5,
        ADDCOLUMNS(
            SUMMARIZE('Dim_Products', 'Dim_Products'[ProductName], 'Dim_Products'[ModelName]),
            "@Rev", [Total Revenue]
        ),
        [@Rev], DESC
    )
VAR ModelCount = COUNTROWS(DISTINCT(SELECTCOLUMNS(Top5, "Model", [ModelName])))
VAR SingleModel = IF(ModelCount = 1, MINX(Top5, [ModelName]))
VAR Result =
    IF(
        ModelCount = 1,
        "Every one of them a " & SingleModel & " variant",
        "Spans " & ModelCount & " different models"
    )
RETURN
    Result
```

## Top Rider Caption

```dax
Top Rider Caption =
VAR Orders = [Total Orders]
VAR Market = SELECTEDVALUE ( Dim_Territories[Country], "multiple markets" )
RETURN
    Orders & IF ( Orders = 1, " order, ", " orders, " ) & Market
```

## Top Riders Callout

```dax
Top Riders Callout =
VAR Top4 = TOPN ( 4, ALL ( Dim_Customers[FullName] ), [Total Revenue], DESC )
VAR WithCountry =
    ADDCOLUMNS (
        Top4,
        "@Orders", [Total Orders],
        "@Country", CALCULATE ( SELECTEDVALUE ( Dim_Territories[Country] ) )
    )
VAR Lo = MINX ( WithCountry, [@Orders] )
VAR Hi = MAXX ( WithCountry, [@Orders] )
VAR CountryCount = COUNTROWS ( DISTINCT ( SELECTCOLUMNS ( WithCountry, "c", [@Country] ) ) )
VAR OneCountry = MAXX ( WithCountry, [@Country] )
VAR AvgOrders = DIVIDE ( [Total Orders], [Total Customers] )
VAR OrderRange = Lo & IF ( Lo = Hi, "", " to " & Hi )
RETURN
    IF (
        CountryCount = 1,
        "The four top riders are all in " & OneCountry & ", ordering " & OrderRange & " times against a " & FORMAT ( AvgOrders, "0.00" ) & " average.",
        "The four top riders ordered " & OrderRange & " times against a " & FORMAT ( AvgOrders, "0.00" ) & " average."
    )
```

## Total Cost

```dax
Total Cost =
SUMX(
    Fact_Sales,
    Fact_Sales[OrderQuantity] *
    RELATED(
        Dim_Products[ProductCost]
        )
)
```

## Total Customers

```dax
Total Customers =
DISTINCTCOUNT ( Fact_Sales[CustomerKey] )
```

## Total Orders

```dax
Total Orders =
DISTINCTCOUNT ( Fact_Sales[OrderNumber] )
```

## Total Products

```dax
Total Products =
DISTINCTCOUNT ( Fact_Sales[ProductKey] )
```

## Total Profit

```dax
Total Profit =
[Total Revenue] - [Total Cost]
```

## Total Revenue (Components)

```dax
Total Revenue (Components) =
CALCULATE ( [Total Revenue], Dim_Product_Categories[CategoryName] = "Components" ) + 0
```

## Total Unit Sold

```dax
Total Unit Sold =
SUM(Fact_Sales[OrderQuantity])
```

## Trend AOV Delta

```dax
Trend AOV Delta =
VAR Trail = DIVIDE ( [Trailing 3M Revenue], [Trailing 3M Orders] )
VAR Prior = DIVIDE ( [Prior 3M Revenue], [Prior 3M Orders] )
RETURN
Trail - Prior
```

## Trend Best Month Caption

```dax
Trend Best Month Caption =
"best month, " & FORMAT ( [Trend Best Month Date], "MMM yyyy" )
```

## Trend Best Month Display

```dax
Trend Best Month Display =
IF (
    [Selected Metric] = "Orders",
    FORMAT ( [Trend Best Month Value], "#,##0" ),
    "$" & FORMAT ( [Trend Best Month Value] / 1000000, "0.00" ) & "M"
)
```

## Trend Chart Subtitle

```dax
Trend Chart Subtitle =
VAR MonthCount = DISTINCTCOUNT ( Dim_Calendar[Start of Month] )
RETURN
MonthCount & " months of actuals. The step change is the accessories and apparel launch."
```

## Trend Margin Delta Pts

```dax
Trend Margin Delta Pts =
VAR Trail = DIVIDE ( [Trailing 3M Profit], [Trailing 3M Revenue] )
VAR Prior = DIVIDE ( [Prior 3M Profit], [Prior 3M Revenue] )
RETURN
( Trail - Prior ) * 100
```

## Trend Metric Callout

```dax
Trend Metric Callout =
VAR M = [Selected Metric]
VAR IsOrders = M = "Orders"
VAR Hist =
    ADDCOLUMNS (
        FILTER (
            ALL ( 'Sales Trend & Forecast'[MonthIndex], 'Sales Trend & Forecast'[Month Date] ),
            'Sales Trend & Forecast'[MonthIndex] <= 12
        ),
        "@Val",
            VAR m = 'Sales Trend & Forecast'[Month Date]
            RETURN
                CALCULATE (
                    [Trend Bar Value],
                    ALL ( Dim_Calendar ),
                    Dim_Calendar[Start of Month] = m
                ),
        "@Prev",
            VAR p = EDATE ( 'Sales Trend & Forecast'[Month Date], -1 )
            RETURN
                CALCULATE (
                    [Trend Bar Value],
                    ALL ( Dim_Calendar ),
                    Dim_Calendar[Start of Month] = p
                )
    )
VAR LastIdx = MAXX ( Hist, 'Sales Trend & Forecast'[MonthIndex] )
VAR LastDown =
    MAXX (
        FILTER ( Hist, NOT ( [@Prev] > 0 ) || [@Val] <= [@Prev] ),
        'Sales Trend & Forecast'[MonthIndex]
    )
VAR Streak = LastIdx - LastDown
VAR StartRow = FILTER ( Hist, 'Sales Trend & Forecast'[MonthIndex] = LastDown )
VAR EndRow = FILTER ( Hist, 'Sales Trend & Forecast'[MonthIndex] = LastIdx )
VAR StartVal = MAXX ( StartRow, [@Val] )
VAR EndVal = MAXX ( EndRow, [@Val] )
VAR StartDate = MAXX ( StartRow, 'Sales Trend & Forecast'[Month Date] )
VAR EndDate = MAXX ( EndRow, 'Sales Trend & Forecast'[Month Date] )
VAR F1Val = CALCULATE ( [Trend Metric Projection], 'Sales Trend & Forecast'[MonthIndex] = 13 )
VAR FLVal = CALCULATE ( [Trend Metric Projection], 'Sales Trend & Forecast'[MonthIndex] = 15 )
VAR F1Date = CALCULATE ( MAX ( 'Sales Trend & Forecast'[Month Date] ), 'Sales Trend & Forecast'[MonthIndex] = 13 )
VAR FLDate = CALCULATE ( MAX ( 'Sales Trend & Forecast'[Month Date] ), 'Sales Trend & Forecast'[MonthIndex] = 15 )
VAR StartTxt = IF ( IsOrders, FORMAT ( StartVal, "#,##0" ) & " orders", "$" & FORMAT ( StartVal / 1000000, "0.00" ) & "M" )
VAR EndTxt = IF ( IsOrders, FORMAT ( EndVal, "#,##0" ) & " orders", "$" & FORMAT ( EndVal / 1000000, "0.00" ) & "M" )
VAR F1Txt = IF ( IsOrders, FORMAT ( F1Val, "#,##0" ) & " orders", "$" & FORMAT ( F1Val / 1000000, "0.00" ) & "M" )
VAR FLTxt = IF ( IsOrders, FORMAT ( FLVal, "#,##0" ) & " orders", "$" & FORMAT ( FLVal / 1000000, "0.00" ) & "M" )
VAR Verb = IF ( IsOrders, "have", "has" )
VAR StreakWord =
    SWITCH ( Streak, 1, "one", 2, "two", 3, "three", 4, "four", 5, "five", 6, "six", 7, "seven", 8, "eight", 9, "nine", 10, "ten", 11, "eleven", 12, "twelve", FORMAT ( Streak, "0" ) )
VAR Proj =
    "The trend projects about " & F1Txt & " in " & FORMAT ( F1Date, "mmmm" )
        & " and " & FLTxt & " by " & FORMAT ( FLDate, "mmmm" )
        & ". * The H1 comparison spans the July 2022 catalogue expansion, so it is not like for like."
RETURN
    IF (
        M = "Revenue",
        [Trend Streak Callout],
        IF (
            Streak <= 0,
            M & " fell in " & FORMAT ( EndDate, "mmmm yyyy" ) & ", so the run of monthly gains has ended. " & Proj,
            M & " " & Verb & " risen " & StreakWord & IF ( Streak = 1, " month", " months" ) & " in a row, from " & StartTxt & " in "
                & FORMAT ( StartDate, "mmmm" ) & " to " & EndTxt & " in " & FORMAT ( EndDate, "mmmm yyyy" ) & ". " & Proj
        )
    )
```

## Trend MoM Growth Color

```dax
Trend MoM Growth Color =
IF ( [Trend MoM Growth 3M Avg] < 0, "#D94F5C", "#1E9E6B" )
```

## Trend MoM Growth Display

```dax
Trend MoM Growth Display =
VAR v = [Trend MoM Growth 3M Avg]
RETURN
IF ( v < 0, UNICHAR ( 9660 ), UNICHAR ( 9650 ) ) & " " & FORMAT ( ABS ( v ), "0.0%" )
```

## Trend Sales Subtitle

```dax
Trend Sales Subtitle =
VAR T = ALL ( 'Sales Trend & Forecast' )
VAR FitN =
    COUNTROWS (
        FILTER (
            T,
            'Sales Trend & Forecast'[Is Forecast] = FALSE ()
                && 'Sales Trend & Forecast'[MonthIndex] >= 1
        )
    )
VAR ForecastN =
    COUNTROWS ( FILTER ( T, 'Sales Trend & Forecast'[Is Forecast] = TRUE () ) )
VAR LastActual =
    FORMAT ( CALCULATE ( MAX ( Dim_Calendar[Date] ), ALL ( Dim_Calendar ) ), "mmmm yyyy" )
VAR FWord =
    SWITCH ( ForecastN, 1, "one", 2, "two", 3, "three", 4, "four", 5, "five", 6, "six", FORMAT ( ForecastN, "0" ) )
VAR AWord =
    SWITCH ( FitN, 6, "six", 7, "seven", 8, "eight", 9, "nine", 10, "ten", 11, "eleven", 12, "twelve", FORMAT ( FitN, "0" ) )
RETURN
    "Actuals to " & LastActual & ", then a " & FWord & " month linear trend projection fit on the " & AWord & " months since the catalogue expanded"
```

## Trend Streak Callout

```dax
Trend Streak Callout =
VAR Hist =
    ADDCOLUMNS (
        FILTER ( ALL ( 'Sales Trend & Forecast' ), 'Sales Trend & Forecast'[Is Forecast] = FALSE () ),
        "@Prev",
            LOOKUPVALUE (
                'Sales Trend & Forecast'[Period Sales],
                'Sales Trend & Forecast'[MonthIndex], 'Sales Trend & Forecast'[MonthIndex] - 1,
                'Sales Trend & Forecast'[Is Forecast], FALSE ()
            )
    )
VAR Fc = FILTER ( ALL ( 'Sales Trend & Forecast' ), 'Sales Trend & Forecast'[Is Forecast] = TRUE () )
VAR LastIdx = MAXX ( Hist, 'Sales Trend & Forecast'[MonthIndex] )
VAR LastDown =
    MAXX (
        FILTER ( Hist, ISBLANK ( [@Prev] ) || 'Sales Trend & Forecast'[Period Sales] <= [@Prev] ),
        'Sales Trend & Forecast'[MonthIndex]
    )
VAR Streak = LastIdx - LastDown
VAR StartRow = FILTER ( Hist, 'Sales Trend & Forecast'[MonthIndex] = LastDown )
VAR EndRow = FILTER ( Hist, 'Sales Trend & Forecast'[MonthIndex] = LastIdx )
VAR F1Row = FILTER ( Fc, 'Sales Trend & Forecast'[MonthIndex] = MINX ( Fc, 'Sales Trend & Forecast'[MonthIndex] ) )
VAR FLRow = FILTER ( Fc, 'Sales Trend & Forecast'[MonthIndex] = MAXX ( Fc, 'Sales Trend & Forecast'[MonthIndex] ) )
VAR StreakWord =
    SWITCH ( Streak, 1, "one", 2, "two", 3, "three", 4, "four", 5, "five", 6, "six", 7, "seven", 8, "eight", 9, "nine", 10, "ten", 11, "eleven", 12, "twelve", FORMAT ( Streak, "0" ) )
VAR StartTxt = "$" & FORMAT ( MAXX ( StartRow, 'Sales Trend & Forecast'[Period Sales] ) / 1000000, "0.00" ) & "M"
VAR EndTxt = "$" & FORMAT ( MAXX ( EndRow, 'Sales Trend & Forecast'[Period Sales] ) / 1000000, "0.00" ) & "M"
VAR F1Txt = "$" & FORMAT ( MAXX ( F1Row, 'Sales Trend & Forecast'[Period Sales] ) / 1000000, "0.00" ) & "M"
VAR FLTxt = "$" & FORMAT ( MAXX ( FLRow, 'Sales Trend & Forecast'[Period Sales] ) / 1000000, "0.00" ) & "M"
VAR BandTxt = "$" & FORMAT ( MAXX ( FLRow, 'Sales Trend & Forecast'[CI Upper] - 'Sales Trend & Forecast'[Period Sales] ) / 1000000, "0.00" ) & "M"
VAR Proj =
    "The trend projects about " & F1Txt & " in " & FORMAT ( MAXX ( F1Row, 'Sales Trend & Forecast'[Month Date] ), "mmmm" )
        & " and " & FLTxt & " by " & FORMAT ( MAXX ( FLRow, 'Sales Trend & Forecast'[Month Date] ), "mmmm" )
        & ", within about " & BandTxt & " either side. * The H1 comparison spans the July 2022 catalogue expansion, so it is not like for like."
RETURN
    IF (
        Streak <= 0,
        "Revenue fell in " & FORMAT ( MAXX ( EndRow, 'Sales Trend & Forecast'[Month Date] ), "mmmm yyyy" ) & ", so the run of monthly gains has ended. " & Proj,
        "Revenue has risen " & StreakWord & IF ( Streak = 1, " month", " months" ) & " in a row, from " & StartTxt & " in "
            & FORMAT ( MAXX ( StartRow, 'Sales Trend & Forecast'[Month Date] ), "mmmm" ) & " to " & EndTxt & " in "
            & FORMAT ( MAXX ( EndRow, 'Sales Trend & Forecast'[Month Date] ), "mmmm yyyy" ) & ". " & Proj
    )
```

## Trend Third Stat Color

```dax
Trend Third Stat Color =
IF (
    [Selected Metric] = "Orders",
    IF ( [Trend AOV Delta] >= 0, "#1E9E6B", "#D94F5C" ),
    IF ( [Trend Margin Delta Pts] >= 0, "#1E9E6B", "#D94F5C" )
)
```

## Trend Third Stat Display

```dax
Trend Third Stat Display =
IF (
    [Selected Metric] = "Orders",
    IF ( [Trend AOV Delta] >= 0, "+$", "-$" ) & FORMAT ( ABS ( [Trend AOV Delta] ), "#,##0" ),
    IF ( [Trend Margin Delta Pts] >= 0, "+", "-" ) & FORMAT ( ABS ( [Trend Margin Delta Pts] ), "0.00" ) & " pts"
)
```

## Trend Third Stat Label

```dax
Trend Third Stat Label =
IF ( [Selected Metric] = "Orders", "AOV, trailing 3mo vs prior 3mo", "margin, trailing 3mo vs prior 3mo" )
```
