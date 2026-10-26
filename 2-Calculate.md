# _Calculate

Calculations with logic: gaps, baskets, segments, rankings and the Strategy numbers. (84 measures)

## % of Total Revenue

```dax
% of Total Revenue =
DIVIDE ( [Total Revenue], [Grand Total Revenue] )
```

## Accessories Revenue

```dax
Accessories Revenue =
CALCULATE ( [Total Revenue], Dim_Product_Categories[CategoryName] = "Accessories"
 )
```

## Adjusted Revenue

```dax
Adjusted Revenue =
[Total Revenue] * ( 1 + [Price Adjustment % Value]
)
```

## Basket Flag

```dax
Basket Flag =
IF ( [Average Order Value (AOV)] > [Company AOV], 1, 0 )
```

## Basket Uplift $

```dax
Basket Uplift $ =
[Bike Plus Accessory AOV] - [Bike Only AOV]
```

## Biggest Basket AOV

```dax
Biggest Basket AOV =
MAXX (
    FILTER ( ALLSELECTED ( Dim_Territories[Country] ), [Total Orders] > 0 ),
    [Average Order Value (AOV)]
)
```

## Bike Attach Rate %

```dax
Bike Attach Rate % =
VAR Orders =
    ADDCOLUMNS (
        SUMMARIZE ( Fact_Sales, Fact_Sales[OrderNumber] ),
        "@HasBike", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Bikes" ) > 0,
        "@HasAccessory", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Accessories" ) > 0
    )
VAR BikeOrders = COUNTROWS ( FILTER ( Orders, [@HasBike] ) )
VAR AttachedOrders = COUNTROWS ( FILTER ( Orders, [@HasBike] && [@HasAccessory] ) )
RETURN
    DIVIDE ( AttachedOrders, BikeOrders )
```

## Bike Only AOV

```dax
Bike Only AOV =
VAR Orders =
    ADDCOLUMNS (
        SUMMARIZE ( Fact_Sales, Fact_Sales[OrderNumber] ),
        "@Revenue", CALCULATE ( [Total Revenue] ),
        "@HasBike", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Bikes" ) > 0,
        "@HasAccessory", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Accessories" ) > 0
    )
RETURN
    AVERAGEX ( FILTER ( Orders, [@HasBike] && NOT [@HasAccessory] ), [@Revenue] )
```

## Bike Only Margin %

```dax
Bike Only Margin % =
VAR Orders =
    ADDCOLUMNS (
        SUMMARIZE ( Fact_Sales, Fact_Sales[OrderNumber] ),
        "@Revenue", CALCULATE ( [Total Revenue] ),
        "@Cost", CALCULATE ( [Total Cost] ),
        "@HasBike", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Bikes" ) > 0,
        "@HasAccessory", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Accessories" ) > 0
    )
VAR Segment = FILTER ( Orders, [@HasBike] && NOT [@HasAccessory] )
VAR Rev = SUMX ( Segment, [@Revenue] )
VAR Cost = SUMX ( Segment, [@Cost] )
RETURN
    DIVIDE ( Rev - Cost, Rev )
```

## Bike Plus Accessory AOV

```dax
Bike Plus Accessory AOV =
VAR Orders =
    ADDCOLUMNS (
        SUMMARIZE ( Fact_Sales, Fact_Sales[OrderNumber] ),
        "@Revenue", CALCULATE ( [Total Revenue] ),
        "@HasBike", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Bikes" ) > 0,
        "@HasAccessory", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Accessories" ) > 0
    )
RETURN
    AVERAGEX ( FILTER ( Orders, [@HasBike] && [@HasAccessory] ), [@Revenue] )
```

## Bike Plus Accessory Margin %

```dax
Bike Plus Accessory Margin % =
VAR Orders =
    ADDCOLUMNS (
        SUMMARIZE ( Fact_Sales, Fact_Sales[OrderNumber] ),
        "@Revenue", CALCULATE ( [Total Revenue] ),
        "@Cost", CALCULATE ( [Total Cost] ),
        "@HasBike", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Bikes" ) > 0,
        "@HasAccessory", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Accessories" ) > 0
    )
VAR Segment = FILTER ( Orders, [@HasBike] && [@HasAccessory] )
VAR Rev = SUMX ( Segment, [@Revenue] )
VAR Cost = SUMX ( Segment, [@Cost] )
RETURN
    DIVIDE ( Rev - Cost, Rev )
```

## Bike Revenue

```dax
Bike Revenue =
CALCULATE ( [Total Revenue], Dim_Product_Categories[CategoryName] = "Bikes"
)
```

## Bridge Value

```dax
Bridge Value =
VAR s = SELECTEDVALUE ( 'Bridge Step'[Step] )
RETURN
    SWITCH (
        s,
        "H1 2022", [H1 Revenue Prior],
        "Bikes", CALCULATE ( [H1 Revenue Change], Dim_Product_Categories[CategoryName] = "Bikes" ),
        "Accessories", CALCULATE ( [H1 Revenue Change], Dim_Product_Categories[CategoryName] = "Accessories" ),
        "Clothing", CALCULATE ( [H1 Revenue Change], Dim_Product_Categories[CategoryName] = "Clothing" )
    )
```

## Company AOV

```dax
Company AOV =
CALCULATE ( [Average Order Value (AOV)], ALLSELECTED ( Dim_Territories ) )
```

## Decile Other Share

```dax
Decile Other Share =
IF ( [Decile Track Gap] <> 0, [Decile Revenue Share %] )
```

## Decile Revenue Share %

```dax
Decile Revenue Share % =
DIVIDE ( SUM ( 'Customer RFM'[Monetary] ), CALCULATE ( SUM ( 'Customer RFM'[Monetary] ), ALL ( 'Customer RFM' ) )
)
```

## Decile Top Share

```dax
Decile Top Share =
IF ( [Decile Track Gap] = 0, [Decile Revenue Share %] )
```

## Decile Track Gap

```dax
Decile Track Gap =
MAXX ( ALLSELECTED ( 'Customer RFM'[Decile Label] ), [Decile Revenue Share %] )
    - [Decile Revenue Share %]
```

## Grand Total Revenue

```dax
Grand Total Revenue =
CALCULATE(
    [Total Revenue],
    ALL (Fact_Sales)
)
```

## Income Band Avg Spend

```dax
Income Band Avg Spend =
DIVIDE ( [Income Band Revenue], [Income Band Customers]
)
```

## Income Band Customers

```dax
Income Band Customers =
COUNTROWS ( 'Customer RFM' )
```

## Income Band Other Spend

```dax
Income Band Other Spend =
IF ( [Income Band Track Gap] <> 0, [Income Band Avg Spend] )
```

## Income Band Revenue

```dax
Income Band Revenue =
SUM ( 'Customer RFM'[Monetary]
)
```

## Income Band Top Spend

```dax
Income Band Top Spend =
IF ( [Income Band Track Gap] = 0, [Income Band Avg Spend] )
```

## Income Band Track Gap

```dax
Income Band Track Gap =
MAXX ( ALLSELECTED ( 'Customer RFM'[Income Band] ), [Income Band Avg Spend] )
    - [Income Band Avg Spend]
```

## Is Top Country

```dax
Is Top Country =
IF ( SELECTEDVALUE ( Dim_Territories[Country] ) = [Top Country Name], 1, 0 )
```

## Is Top Product

```dax
Is Top Product =
IF (
    SELECTEDVALUE ( Dim_Products[ProductName] )
        = CALCULATE ( [Top Product by Revenue], ALL ( Dim_Products ) ),
    1,
    0
)
```

## Is Top RPC Market

```dax
Is Top RPC Market =
VAR Best =
    CALCULATE (
        MAXX ( VALUES ( Dim_Territories[Country] ), [Revenue per Customer] ),
        ALLSELECTED ( Dim_Territories )
    )
RETURN IF ( [Revenue per Customer] = Best, 1, 0 )
```

## Lowest Margin Product

```dax
Lowest Margin Product =
VAR RankTable = FILTER ( ADDCOLUMNS ( VALUES ( Dim_Products[ProductName] ), "Marg",
DIVIDE ( [Total Profit], [Total Revenue] ) ), NOT ISBLANK ( [Marg] ) ) VAR BottomProduct = TOPN ( 1, RankTable, [Marg], ASC ) RETURN MAXX ( BottomProduct, Dim_Products[ProductName]
)
```

## Lowest Margin Product Margin %

```dax
Lowest Margin Product Margin % =
VAR BottomName =
    [Lowest Margin Product] RETURN CALCULATE ( [Profit Margin %], Dim_Products[ProductName] = BottomName
)
```

## Lowest Margin Product Units

```dax
Lowest Margin Product Units =
VAR BottomName =
[Lowest Margin Product] RETURN CALCULATE ( [Total Unit Sold], Dim_Products[ProductName] = BottomName
)
```

## Margin % (Accessories)

```dax
Margin % (Accessories) =
DIVIDE (
    CALCULATE ( [Total Profit], Dim_Product_Categories[CategoryName] = "Accessories" ),
    [Total Revenue (Accessories)]
)
```

## Margin % (Apparel)

```dax
Margin % (Apparel) =
DIVIDE (
    CALCULATE ( [Total Profit], Dim_Product_Categories[CategoryName] = "Clothing" ),
    [Total Revenue (Apparel)]
)
```

## Margin % (Bikes)

```dax
Margin % (Bikes) =
DIVIDE (
    CALCULATE ( [Total Profit], Dim_Product_Categories[CategoryName] = "Bikes" ),
    [Total Revenue (Bikes)]
)
```

## Markets Below Avg Basket

```dax
Markets Below Avg Basket =
VAR Co = [Company AOV]
RETURN
    COUNTROWS (
        FILTER (
            ALLSELECTED ( Dim_Territories[Country] ),
            [Total Orders] > 0 && [Average Order Value (AOV)] < Co
        )
    )
```

## New Customers

```dax
New Customers =
VAR PeriodStart = MIN ( Dim_Calendar[Date] )
VAR PeriodEnd = MAX ( Dim_Calendar[Date] )
RETURN
    CALCULATE (
        DISTINCTCOUNT ( Fact_Sales[CustomerKey] ),
        FILTER (
            VALUES ( Fact_Sales[CustomerKey] ),
            VAR FirstOrderDate =
                CALCULATE ( MIN ( Fact_Sales[OrderDate] ), ALL ( Dim_Calendar ) )
            RETURN
                FirstOrderDate >= PeriodStart && FirstOrderDate <= PeriodEnd
        )
    )
```

## Occupation Other Spend

```dax
Occupation Other Spend =
IF ( [Occupation Track Gap] <> 0, [Revenue per Customer] )
```

## Occupation Top Spend

```dax
Occupation Top Spend =
IF ( [Occupation Track Gap] = 0, [Revenue per Customer] )
```

## Occupation Track Gap

```dax
Occupation Track Gap =
MAXX ( ALLSELECTED ( Dim_Customers[Occupation] ), [Revenue per Customer] )
    - [Revenue per Customer]
```

## Other Continent Revenue

```dax
Other Continent Revenue =
VAR Best = MAXX ( ALLSELECTED ( Dim_Territories[Continent] ), [Total Revenue] )
RETURN
    IF ( [Total Revenue] < Best, [Total Revenue] )
```

## Pct Revenue Cont (ALLSELECTED Pro)

```dax
Pct Revenue Cont (ALLSELECTED Pro) =
VAR _RowTotalRevenue = [Total Revenue]

VAR _GrandTotalRevenue =
CALCULATE(
    [Total Revenue],
    ALLSELECTED(Fact_Sales)
)

RETURN
    DIVIDE(
        _RowTotalRevenue,_GrandTotalRevenue
    )+0
```

## Projected Attach Revenue

```dax
Projected Attach Revenue =
VAR BikeOrders = CALCULATE ( DISTINCTCOUNT ( Fact_Sales[OrderNumber] ), Dim_Product_Categories[CategoryName] = "Bikes" )
VAR AvgAccessoryBasket =
    DIVIDE (
        CALCULATE ( [Total Revenue], Dim_Product_Categories[CategoryName] = "Accessories" ),
        CALCULATE ( DISTINCTCOUNT ( Fact_Sales[OrderNumber] ), Dim_Product_Categories[CategoryName] = "Accessories" )
    )
RETURN BikeOrders * [Bike Attach Rate Uplift % Value] * AvgAccessoryBasket
```

## Returning Customer Orders

```dax
Returning Customer Orders =
[Total Orders] - [New Customers]
```

## Revenue Impact of Price Change

```dax
Revenue Impact of Price Change =
[Adjusted Revenue] - [Total Revenue]
```

## Revenue Share % (Accessories)

```dax
Revenue Share % (Accessories) =
DIVIDE ( [Total Revenue (Accessories)], [Grand Total Revenue] )
```

## Revenue Share % (Apparel)

```dax
Revenue Share % (Apparel) =
DIVIDE ( [Total Revenue (Apparel)], [Grand Total Revenue] )
```

## Revenue Share % (Bikes)

```dax
Revenue Share % (Bikes) =
DIVIDE ( [Total Revenue (Bikes)], [Grand Total Revenue] )
```

## Revenue Share % (Components)

```dax
Revenue Share % (Components) =
DIVIDE ( [Total Revenue (Components)], [Grand Total Revenue] )
```

## Revenue Split (Top Quartile)

```dax
Revenue Split (Top Quartile) =
PERCENTILEX.INC(
    FILTER(Dim_Products, [Total Revenue] > 0),
    [Total Revenue],
    0.75
)
```

## Scatter Y Split

```dax
Scatter Y Split =
SWITCH(
    [Selected Scatter Mode],
    "Margin", MEDIANX(FILTER(Dim_Products, [Total Revenue] > 0), [Profit Margin %]),
    "Volume", MEDIANX(FILTER(Dim_Products, [Total Revenue] > 0), [Total Unit Sold])
)
```

## Scatter Y Value

```dax
Scatter Y Value =
SWITCH (
    [Selected Scatter Mode],
    "Margin", [Profit Margin %],
    "Volume", [Total Unit Sold]
)
```

## Segment Median Recency

```dax
Segment Median Recency =
MEDIANX ( 'Customer RFM', 'Customer RFM'[Recency] )
```

## Segment Rider Share %

```dax
Segment Rider Share % =
DIVIDE ( [Income Band Customers], CALCULATE ( [Income Band Customers], ALL ( 'Customer RFM' ) ) )
```

## Smallest Basket AOV

```dax
Smallest Basket AOV =
MINX (
    FILTER ( ALLSELECTED ( Dim_Territories[Country] ), [Total Orders] > 0 ),
    [Average Order Value (AOV)]
)
```

## Strat At Risk $

```dax
Strat At Risk $ =
VAR Rate = 0.05
RETURN
    CALCULATE ( SUM ( 'Customer RFM'[Monetary] ), 'Customer RFM'[Decile] = 1 ) * Rate
```

## Strat Attach Gap $

```dax
Strat Attach Gap $ =
VAR Orders =
    ADDCOLUMNS (
        SUMMARIZE ( Fact_Sales, Fact_Sales[OrderNumber] ),
        "@HasBike", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Bikes" ) > 0,
        "@HasAccessory", CALCULATE ( COUNTROWS ( Fact_Sales ), Dim_Product_Categories[CategoryName] = "Accessories" ) > 0
    )
VAR BikeOnly = COUNTROWS ( FILTER ( Orders, [@HasBike] && NOT [@HasAccessory] ) )
VAR AccBasket =
    DIVIDE (
        CALCULATE ( [Total Revenue], Dim_Product_Categories[CategoryName] = "Accessories" ),
        CALCULATE ( DISTINCTCOUNT ( Fact_Sales[OrderNumber] ), Dim_Product_Categories[CategoryName] = "Accessories" )
    )
RETURN
    BikeOnly * AccBasket
```

## Strat Basket Gap $

```dax
Strat Basket Gap $ =
VAR Weak = [Strat Weak Market]
VAR WeakOrders = CALCULATE ( [Total Orders], Dim_Territories[Country] = Weak )
VAR WeakAov = CALCULATE ( [Average Order Value (AOV)], Dim_Territories[Country] = Weak )
RETURN
    WeakOrders * ( [Company AOV] - WeakAov )
```

## Strat Mix Shift Profit $

```dax
Strat Mix Shift Profit $ =
VAR ShiftShare = 0.25
VAR RoadRev = CALCULATE ( [Total Revenue], Dim_Product_Subcategories[SubcategoryName] = "Road Bikes" )
VAR RoadMargin = CALCULATE ( [Profit Margin %], Dim_Product_Subcategories[SubcategoryName] = "Road Bikes" )
VAR MtnMargin = CALCULATE ( [Profit Margin %], Dim_Product_Subcategories[SubcategoryName] = "Mountain Bikes" )
RETURN
    RoadRev * ShiftShare * ( MtnMargin - RoadMargin )
```

## Strat Profit Upside

```dax
Strat Profit Upside =
[Strat Mix Shift Profit $] * [Mix Shift Slider Value] / 100
```

## Strat Reactivation $

```dax
Strat Reactivation $ =
VAR Rate = 0.05
RETURN
    CALCULATE ( SUM ( 'Customer RFM'[Monetary] ), 'Customer RFM'[Segment] = "Lapsing" ) * Rate
```

## Strat Revenue Upside

```dax
Strat Revenue Upside =
[Strat Basket Gap $] * [Canada Basket Slider Value] / 100
    + [Strat Attach Gap $] * [Attach Gap Slider Value] / 100
    + [Strat Reactivation $] * [Reactivation Slider Value] / 100
```

## Strat Upside Value

```dax
Strat Upside Value =
SWITCH (
    SELECTEDVALUE ( 'Strat Upside Step'[Order] ),
    1, [Total Revenue],
    2, [Strat Basket Gap $] * [Canada Basket Slider Value] / 100,
    3, [Strat Attach Gap $] * [Attach Gap Slider Value] / 100,
    4, [Strat Reactivation $] * [Reactivation Slider Value] / 100
)
```

## Subcategories Selling

```dax
Subcategories Selling =
CALCULATE (
    DISTINCTCOUNT ( Dim_Product_Subcategories[ProductSubcategoryKey] ),
    FILTER (
        VALUES ( Dim_Product_Subcategories[ProductSubcategoryKey] ),
        [Total Revenue] > 0
    )
)
```

## Top 10 Model Count

```dax
Top 10 Model Count =
VAR BaseTable =
    SUMMARIZE ( Dim_Products, Dim_Products[ProductName], Dim_Products[ModelName] )
VAR Top10 =
    TOPN ( 10, BaseTable, CALCULATE ( [Total Revenue] ), DESC )
VAR Result =
    COUNTROWS ( DISTINCT ( SELECTCOLUMNS ( Top10, "Model", [ModelName] ) ) )
RETURN Result
```

## Top 10 Revenue Share %

```dax
Top 10 Revenue Share % =
VAR Top10 =
    SUMMARIZE ( Dim_Products, Dim_Products[ProductName] )
VAR Top10Rev =
    SUMX ( TOPN ( 10, Top10, CALCULATE ( [Total Revenue] ), DESC ), CALCULATE ( [Total Revenue] ) )
VAR TotalRev = CALCULATE ( [Total Revenue], ALL ( Dim_Products ) )
VAR Result = DIVIDE ( Top10Rev, TotalRev )
RETURN Result
```

## Top Continent Revenue

```dax
Top Continent Revenue =
VAR Best = MAXX ( ALLSELECTED ( Dim_Territories[Continent] ), [Total Revenue] )
RETURN
    IF ( [Total Revenue] = Best, [Total Revenue] )
```

## Top Country AOV

```dax
Top Country AOV =
VAR TopName = [Top Country Name]
RETURN CALCULATE ( [Average Order Value (AOV)], Dim_Territories[Country] = TopName )
```

## Top Country Name

```dax
Top Country Name =
CALCULATE (
    CALCULATE (
        SELECTEDVALUE ( Dim_Territories[Country] ),
        TOPN ( 1, VALUES ( Dim_Territories[Country] ), CALCULATE ( [Total Revenue] ), DESC )
    ),
    ALL ( Dim_Territories )
)
```

## Top Country Revenue

```dax
Top Country Revenue =
VAR TopName = [Top Country Name]
RETURN CALCULATE ( [Total Revenue], Dim_Territories[Country] = TopName )
```

## Top Country Share %

```dax
Top Country Share % =
CALCULATE (
    CALCULATE (
        DIVIDE ( [Total Revenue], CALCULATE ( [Total Revenue], ALL ( Dim_Territories ) ) ),
        TOPN ( 1, VALUES ( Dim_Territories[Country] ), CALCULATE ( [Total Revenue] ), DESC )
    ),
    ALL ( Dim_Territories )
)
```

## Top Decile Share %

```dax
Top Decile Share % =
DIVIDE ( CALCULATE ( SUM ( 'Customer RFM'[Monetary] ), 'Customer RFM'[Decile] = 1 ), SUM ( 'Customer RFM'[Monetary] ) )
```

## Top Product by Revenue

```dax
Top Product by Revenue =
VAR RankTable =
    ADDCOLUMNS (
        VALUES ( Dim_Products[ProductName] ),
        "Rev", [Total Revenue]
    )
VAR TopProduct =
    TOPN ( 1, RankTable, [Rev], DESC )
RETURN
    MAXX ( TopProduct, Dim_Products[ProductName] )
```

## Top Product Margin %

```dax
Top Product Margin % =
VAR TopName = [Top Product by Revenue]
RETURN CALCULATE ( [Profit Margin %], Dim_Products[ProductName] = TopName
)
```

## Top Product Revenue

```dax
Top Product Revenue =
VAR TopName = [Top Product by Revenue]
RETURN CALCULATE ( [Total Revenue], Dim_Products[ProductName] = TopName )
```

## Top Product Share %

```dax
Top Product Share % =
VAR TopName = [Top Product by Revenue]
RETURN CALCULATE ( [% of Total Revenue], Dim_Products[ProductName] = TopName
)
```

## Top Product Units

```dax
Top Product Units =
VAR TopName = [Top Product by Revenue]
RETURN CALCULATE ( [Total Unit Sold], Dim_Products[ProductName] = TopName
)
```

## Top Two Deciles Share %

```dax
Top Two Deciles Share % =
CALCULATE ( [Decile Revenue Share %], 'Customer RFM'[Decile] <= 2 )
```

## Total Products (Bikes)

```dax
Total Products (Bikes) =
CALCULATE (
    [Total Products],
    Dim_Product_Categories[CategoryName] = "Bikes"
)
```

## Total Revenue

```dax
Total Revenue =
SUMX(
    Fact_Sales,
    Fact_Sales[OrderQuantity] * RELATED(Dim_Products[ProductPrice])
)
```

## Total Revenue (Accessories)

```dax
Total Revenue (Accessories) =
CALCULATE (
    [Total Revenue],
    Dim_Product_Categories[CategoryName] = "Accessories"
)
```

## Total Revenue (Apparel)

```dax
Total Revenue (Apparel) =
CALCULATE (
    [Total Revenue],
    Dim_Product_Categories[CategoryName] = "Clothing"
)
```

## Total Revenue (Bikes)

```dax
Total Revenue (Bikes) =
CALCULATE(
    [Total Revenue],
    Dim_Product_Categories[CategoryName] = "Bikes"
)
```

## Total Units Sold (Accessories)

```dax
Total Units Sold (Accessories) =
CALCULATE (
    [Total Unit Sold],
    Dim_Product_Categories[CategoryName] = "Accessories"
)
```

## Track Gap

```dax
Track Gap =
MAXX ( ALLSELECTED ( Dim_Territories[Continent] ), [Total Revenue] ) - [Total Revenue]
```
