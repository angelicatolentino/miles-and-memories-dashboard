# _TIME INT

Anything that compares periods: trailing, growth, half year, trend and forecast. (37 measures)

## Actual Sales

```dax
Actual Sales =
CALCULATE ( SUM ( 'Sales Trend & Forecast'[Period Sales] ),
'Sales Trend & Forecast'[Is Forecast] = FALSE ()
)
```

## First Month AOV

```dax
First Month AOV =
VAR FirstOrderDate = CALCULATE ( MIN ( Fact_Sales[OrderDate] ), ALL ( Fact_Sales ) )
VAR FirstMonth =
    FILTER (
        ALL ( Dim_Calendar ),
        YEAR ( Dim_Calendar[Date] ) = YEAR ( FirstOrderDate )
            && MONTH ( Dim_Calendar[Date] ) = MONTH ( FirstOrderDate )
    )
RETURN
    DIVIDE ( CALCULATE ( [Total Revenue], FirstMonth ), CALCULATE ( [Total Orders], FirstMonth ) )
```

## Forecast Sales

```dax
Forecast Sales =
IF (
    MAX ( 'Sales Trend & Forecast'[MonthIndex] ) = 12,
    CALCULATE ( SUM ( 'Sales Trend & Forecast'[Period Sales] ), 'Sales Trend & Forecast'[Is Forecast] = FALSE () ),
    CALCULATE ( SUM ( 'Sales Trend & Forecast'[Period Sales] ), 'Sales Trend & Forecast'[Is Forecast] = TRUE () )
)
```

## H1 Growth %

```dax
H1 Growth % =
VAR CurrentYear = MAX ( Dim_Calendar[Year] ) VAR CurrentH1Revenue = CALCULATE ( [Total Revenue],
Dim_Calendar[Year] = CurrentYear, Dim_Calendar[Month] <= 6 )
VAR PriorH1Revenue =
CALCULATE( [Total Revenue], Dim_Calendar[Year] = CurrentYear - 1, Dim_Calendar[Month] <= 6 ) RETURN DIVIDE ( CurrentH1Revenue - PriorH1Revenue, PriorH1Revenue )
```

## H1 Revenue Change

```dax
H1 Revenue Change =
[H1 Revenue Latest] - [H1 Revenue Prior]
```

## H1 Revenue Latest

```dax
H1 Revenue Latest =
VAR Y = MAX ( Dim_Calendar[Year] )
RETURN
    CALCULATE ( [Total Revenue], Dim_Calendar[Year] = Y, Dim_Calendar[Month] <= 6 )
```

## H1 Revenue Prior

```dax
H1 Revenue Prior =
VAR Y = MAX ( Dim_Calendar[Year] ) - 1
RETURN
    CALCULATE ( [Total Revenue], Dim_Calendar[Year] = Y, Dim_Calendar[Month] <= 6 )
```

## Latest Month AOV

```dax
Latest Month AOV =
DIVIDE ( [Latest Month Revenue], [Latest Month Orders] )
```

## Latest Month Growth %

```dax
Latest Month Growth % =
VAR LatestDate = CALCULATE ( MAX ( Dim_Calendar[Date] ), ALL ( Dim_Calendar ) )
VAR MonthStart = DATE ( YEAR ( LatestDate ), MONTH ( LatestDate ), 1 )
VAR PrevRev =
    CALCULATE (
        [Total Revenue],
        FILTER (
            ALL ( Dim_Calendar ),
            Dim_Calendar[Date] >= EDATE ( MonthStart, -1 ) && Dim_Calendar[Date] < MonthStart
        )
    )
RETURN
    DIVIDE ( [Latest Month Revenue] - PrevRev, PrevRev )
```

## Latest Month Orders

```dax
Latest Month Orders =
VAR LatestDate = CALCULATE ( MAX ( Dim_Calendar[Date] ), ALL ( Dim_Calendar ) )
RETURN
    CALCULATE (
        [Total Orders],
        FILTER (
            ALL ( Dim_Calendar ),
            YEAR ( Dim_Calendar[Date] ) = YEAR ( LatestDate ) &&
                MONTH ( Dim_Calendar[Date] ) = MONTH ( LatestDate )
        )
    )
```

## Latest Month Revenue

```dax
Latest Month Revenue =
VAR LatestDate = CALCULATE ( MAX ( Dim_Calendar[Date] ), ALL ( Dim_Calendar ) )
RETURN
    CALCULATE (
        [Total Revenue],
        FILTER (
            ALL ( Dim_Calendar ),
            YEAR ( Dim_Calendar[Date] ) = YEAR ( LatestDate ) &&
                MONTH ( Dim_Calendar[Date] ) = MONTH ( LatestDate )
        )
    )
```

## MoM Chart Growth %

```dax
MoM Chart Growth % =
VAR ThisMonth = MAX ( Dim_Calendar[Start of Month] )
VAR Cur = [Total Revenue]
VAR Prev =
    CALCULATE (
        [Total Revenue],
        REMOVEFILTERS ( Dim_Calendar ),
        Dim_Calendar[Start of Month] = EDATE ( ThisMonth, -1 )
    )
RETURN
    IF ( ISBLANK ( Prev ), BLANK (), DIVIDE ( Cur - Prev, Prev ) )
```

## MoM Growth %

```dax
MoM Growth % =
DIVIDE([Total Revenue] - [Prev. Month Revenue], [Prev. Month Revenue])
```

## MoM Worst Fall %

```dax
MoM Worst Fall % =
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
RETURN
    MINX ( Win, [@G] )
```

## Prev. Month Profit

```dax
Prev. Month Profit =
CALCULATE(
    [Total Profit],
    PREVIOUSMONTH(Dim_Calendar[Date])
)
```

## Prev. Month Revenue

```dax
Prev. Month Revenue =
CALCULATE(
    [Total Revenue],
    DATEADD(Dim_Calendar[Date],-1,MONTH)
)
```

## Prev. Month Units Sold

```dax
Prev. Month Units Sold =
CALCULATE(
    [Total Unit Sold],
    PREVIOUSMONTH(Dim_Calendar[Date])
)
```

## Prior 3M Orders

```dax
Prior 3M Orders =
CALCULATE (
    [Total Orders],
    DATESINPERIOD ( Dim_Calendar[Date], EOMONTH ( MAX ( Dim_Calendar[Date] ), -3 ), -3, MONTH )
)
```

## Prior 3M Profit

```dax
Prior 3M Profit =
CALCULATE (
    [Total Profit],
    DATESINPERIOD ( Dim_Calendar[Date], EOMONTH ( MAX ( Dim_Calendar[Date] ), -3 ), -3, MONTH )
)
```

## Prior 3M Revenue

```dax
Prior 3M Revenue =
CALCULATE (
    [Total Revenue],
    DATESINPERIOD ( Dim_Calendar[Date], EOMONTH ( MAX ( Dim_Calendar[Date] ), -3 ), -3, MONTH )
)
```

## Seasonality Avg Revenue

```dax
Seasonality Avg Revenue =
AVERAGEX ( ALLSELECTED ( Dim_Calendar[Month] ), [Total Revenue] )
```

## SPLY Revenue

```dax
SPLY Revenue =
CALCULATE(
    [Total Revenue],
    SAMEPERIODLASTYEAR(Dim_Calendar[Date])
)
```

## Trailing 3-Month Growth %

```dax
Trailing 3-Month Growth % =
DIVIDE (
    [Trailing 3M Revenue] - [Prior 3M Revenue],
    [Prior 3M Revenue]
)
```

## Trailing 3M Avg Revenue

```dax
Trailing 3M Avg Revenue =
DIVIDE ( [Trailing 3M Revenue], 3 )
```

## Trailing 3M Orders

```dax
Trailing 3M Orders =
CALCULATE (
    [Total Orders],
    DATESINPERIOD ( Dim_Calendar[Date], MAX ( Dim_Calendar[Date] ), -3, MONTH )
)
```

## Trailing 3M Profit

```dax
Trailing 3M Profit =
CALCULATE (
    [Total Profit],
    DATESINPERIOD ( Dim_Calendar[Date], MAX ( Dim_Calendar[Date] ), -3, MONTH )
)
```

## Trailing 3M Revenue

```dax
Trailing 3M Revenue =
CALCULATE (
    [Total Revenue],
    DATESINPERIOD ( Dim_Calendar[Date], MAX ( Dim_Calendar[Date] ), -3, MONTH )
)
```

## Trend 3M Avg Revenue

```dax
Trend 3M Avg Revenue =
VAR i = MAX ( 'Sales Trend & Forecast'[MonthIndex] )
RETURN
    IF (
        i < 3 || i > 12,
        BLANK (),
        AVERAGEX (
            FILTER (
                ALL ( 'Sales Trend & Forecast' ),
                'Sales Trend & Forecast'[MonthIndex] <= i
                    && 'Sales Trend & Forecast'[MonthIndex] > i - 3
                    && 'Sales Trend & Forecast'[Is Forecast] = FALSE ()
            ),
            'Sales Trend & Forecast'[Period Sales]
        )
    )
```

## Trend Best Month Date

```dax
Trend Best Month Date =
VAR MaxVal = [Trend Best Month Value]
RETURN
CALCULATE (
    MAX ( Dim_Calendar[Start of Month] ),
    FILTER ( ALL ( Dim_Calendar[Start of Month] ), [Trend Bar Value] = MaxVal )
)
```

## Trend Best Month Value

```dax
Trend Best Month Value =
CALCULATE (
    MAXX ( ALL ( Dim_Calendar[Start of Month] ), [Trend Bar Value] )
)
```

## Trend Metric 3M Avg

```dax
Trend Metric 3M Avg =
VAR d = MAX ( 'Sales Trend & Forecast'[Month Date] )
VAR i = MAX ( 'Sales Trend & Forecast'[MonthIndex] )
RETURN
    IF (
        [Selected Metric] = "Revenue",
        [Trend 3M Avg Revenue],
        IF (
            i < 3 || i > 12,
            BLANK (),
            AVERAGEX (
                FILTER (
                    ALL ( Dim_Calendar[Start of Month] ),
                    Dim_Calendar[Start of Month] <= d
                        && Dim_Calendar[Start of Month] > EDATE ( d, -3 )
                ),
                CALCULATE ( [Trend Bar Value] )
            )
        )
    )
```

## Trend Metric Actual

```dax
Trend Metric Actual =
VAR d = MAX ( 'Sales Trend & Forecast'[Month Date] )
VAR i = MAX ( 'Sales Trend & Forecast'[MonthIndex] )
RETURN
    IF (
        [Selected Metric] = "Revenue",
        [Actual Sales],
        IF (
            i <= 12,
            CALCULATE (
                [Trend Bar Value],
                ALL ( Dim_Calendar ),
                Dim_Calendar[Start of Month] = d
            )
        )
    )
```

## Trend Metric Projection

```dax
Trend Metric Projection =
VAR i = MAX ( 'Sales Trend & Forecast'[MonthIndex] )
VAR d = MAX ( 'Sales Trend & Forecast'[Month Date] )
VAR FitRows =
    ADDCOLUMNS (
        FILTER (
            ALL ( 'Sales Trend & Forecast'[MonthIndex], 'Sales Trend & Forecast'[Month Date] ),
            'Sales Trend & Forecast'[MonthIndex] >= 1
                && 'Sales Trend & Forecast'[MonthIndex] <= 12
        ),
        "@y",
            VAR m = 'Sales Trend & Forecast'[Month Date]
            RETURN
                CALCULATE (
                    [Trend Bar Value],
                    ALL ( Dim_Calendar ),
                    Dim_Calendar[Start of Month] = m
                )
    )
VAR n = COUNTROWS ( FitRows )
VAR SumOfX = SUMX ( FitRows, 'Sales Trend & Forecast'[MonthIndex] )
VAR SumOfY = SUMX ( FitRows, [@y] )
VAR SumOfXY = SUMX ( FitRows, 'Sales Trend & Forecast'[MonthIndex] * [@y] )
VAR SumOfX2 = SUMX ( FitRows, 'Sales Trend & Forecast'[MonthIndex] * 'Sales Trend & Forecast'[MonthIndex] )
VAR Slope = DIVIDE ( n * SumOfXY - SumOfX * SumOfY, n * SumOfX2 - SumOfX * SumOfX )
VAR Intercept = DIVIDE ( SumOfY - Slope * SumOfX, n )
RETURN
    IF (
        [Selected Metric] = "Revenue",
        [Forecast Sales],
        IF (
            i = 12,
            CALCULATE (
                [Trend Bar Value],
                ALL ( Dim_Calendar ),
                Dim_Calendar[Start of Month] = d
            ),
            IF ( i >= 13 && i <= 15, Slope * i + Intercept )
        )
    )
```

## Trend MoM Growth 3M Avg

```dax
Trend MoM Growth 3M Avg =
VAR LatestMonth = CALCULATE ( MAX ( Dim_Calendar[Start of Month] ), ALL ( Dim_Calendar ) )
VAR Last3Months =
    FILTER (
        ALL ( Dim_Calendar[Start of Month] ),
        Dim_Calendar[Start of Month] <= LatestMonth
            && Dim_Calendar[Start of Month] > EDATE ( LatestMonth, -3 )
    )
VAR MoMPercents =
    ADDCOLUMNS (
        Last3Months,
        "@MoM",
            VAR ThisVal = CALCULATE ( [Trend Bar Value] )
            VAR PrevVal = CALCULATE ( [Trend Bar Value], DATEADD ( Dim_Calendar[Date], -1, MONTH ) )
            RETURN DIVIDE ( ThisVal - PrevVal, PrevVal )
    )
RETURN
AVERAGEX ( MoMPercents, [@MoM] )
```

## vs Prev. Month Revenue ($)

```dax
vs Prev. Month Revenue ($) =
[Total Revenue] - [Prev. Month Revenue]
```

## vs Prev. Month Revenue (%)

```dax
vs Prev. Month Revenue (%) =
VAR _TotalRevenue =
    SUMX(
        VALUES(Dim_Calendar[Start of Month]),
        [Total Revenue]
    )

VAR _TotalPrevRevenue =
    SUMX(
        VALUES(Dim_Calendar[Start of Month]),
        [Prev. Month Revenue]
    )

RETURN
DIVIDE(_TotalRevenue - _TotalPrevRevenue, _TotalPrevRevenue)
```

## Year-to-Date

```dax
Year-to-Date =
TOTALYTD(
    [Total Revenue],
    Dim_Calendar[Date]
)
```
