# Helper tables

Helper measures that sit in the parameter or selector table they belong to (sliders, metric selector, scatter mode). (10 measures)

## Attach Gap Slider Value (Attach Gap Slider)

```dax
Attach Gap Slider Value =
MAX ( 'Attach Gap Slider'[Attach Gap Slider] )
```

## Bike Attach Rate Uplift % Value (Bike Attach Rate Uplift %)

```dax
Bike Attach Rate Uplift % Value =
SELECTEDVALUE('Bike Attach Rate Uplift %'[Bike Attach Rate Uplift %], 0)
```

## Canada Basket Slider Value (Canada Basket Slider)

```dax
Canada Basket Slider Value =
MAX ( 'Canada Basket Slider'[Canada Basket Slider] )
```

## Mix Shift Slider Value (Mix Shift Slider)

```dax
Mix Shift Slider Value =
MAX ( 'Mix Shift Slider'[Mix Shift Slider] )
```

## Price Adjustment % Value (Price Adjustment %)

```dax
Price Adjustment % Value =
SELECTEDVALUE('Price Adjustment %'[Price Adjustment %], 0)
```

## Reactivation Slider Value (Reactivation Slider)

```dax
Reactivation Slider Value =
MAX ( 'Reactivation Slider'[Reactivation Slider] )
```

## Selected Scatter Mode (Metric Selector)

```dax
Selected Scatter Mode =
SELECTEDVALUE('Scatter Mode'[Mode], "Margin")
```

## Trend Bar Value (Metric Selector)

```dax
Trend Bar Value =
SWITCH (
    TRUE (),
    [Selected Metric] = "Revenue", [Total Revenue],
    [Selected Metric] = "Profit", [Total Profit],
    [Selected Metric] = "Orders", [Total Orders]
)
```

## Trend Line Value (Metric Selector)

```dax
Trend Line Value =
SWITCH (
    TRUE (),
    [Selected Metric] = "Revenue", [Total Profit],
    [Selected Metric] = "Profit", [Total Revenue],
    [Selected Metric] = "Orders", [Trend Orders 3M Avg]
)
```

## Trend Orders 3M Avg (Metric Selector)

```dax
Trend Orders 3M Avg =
VAR CurrentMonth = MAX ( Dim_Calendar[Start of Month] )
VAR Last3Months =
    FILTER (
        ALL ( Dim_Calendar[Start of Month] ),
        Dim_Calendar[Start of Month] <= CurrentMonth
            && Dim_Calendar[Start of Month] > EDATE ( CurrentMonth, -3 )
    )
RETURN
AVERAGEX ( Last3Months, CALCULATE ( [Total Orders] ) )
```
