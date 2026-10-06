# DAX Measures

[← Back to README](../README.md)

All measures live in the **Master Sales Database** table. 30 measures in 5 groups.

## Contents

- [Core KPIs](#core-kpis)
- [Year-over-Year engine](#year-over-year-engine)
- [Product concentration & Pareto](#product-concentration--pareto)
- [Pricing](#pricing)
- [Lifecycle & risk](#lifecycle--risk)

---

## Core KPIs

Base aggregations every other measure builds on.

| Measure | Purpose |
|---|---|
| `Total Revenue` | Sum of row-level revenue. |
| `Total Units` | Sum of units sold. |
| `Avg SP` | Average selling price = revenue ÷ units (weighted, not a simple average of prices). |
| `Revenue Per Unit` | Same logic as Avg SP; used on the Pricing page. |
| `Active Products` | Distinct products in the current filter context. |

### Total Revenue

```DAX
Total Revenue =
SUM('Master Sales Database'[Revenue])
```

### Total Units

```DAX
Total Units =
SUM('Master Sales Database'[Qty])
```

### Avg SP

```DAX
Avg SP =
DIVIDE([Total Revenue],[Total Units])
```

### Revenue Per Unit

```DAX
Revenue Per Unit =
DIVIDE([Total Revenue],[Total Units])
```

### Active Products

```DAX
Active Products =
DISTINCTCOUNT('Master Sales Database'[Item Name])
```

---

## Year-over-Year engine

Each KPI follows the same pattern: *Previous Year X* → *YoY X %* → *Latest X YoY %* (feeds the KPI card variance).

| Measure | Purpose |
|---|---|
| `Previous Year Revenue` | Revenue for MAX(Year) − 1. |
| `YoY Revenue Growth %` | (Current − Previous) ÷ Previous. |
| `Latest YoY Growth %` | YoY revenue growth evaluated for the latest year in context. |
| `Previous Year Units` | — |
| `YoY Units Growth %` | — |
| `Latest Units YoY %` | — |
| `Previous Year Avg SP` | — |
| `YoY Avg SP %` | — |
| `Previous Year Active Products` | — |
| `YoY Active Products %` | — |
| `Latest Active Products YoY %` | — |
| `Previous Year Premium Products` | — |
| `YoY Premium Products %` | — |
| `Latest Premium Products YoY %` | — |

### Previous Year Revenue

```DAX
Previous Year Revenue =
CALCULATE(
    [Total Revenue],
    FILTER(
        ALL('Master Sales Database'[Year]),
        'Master Sales Database'[Year]
        = MAX('Master Sales Database'[Year]) - 1
    )
)
```

### YoY Revenue Growth %

```DAX
YoY Revenue Growth % =
DIVIDE(
    [Total Revenue] - [Previous Year Revenue],
    [Previous Year Revenue]
)
```

### Latest YoY Growth %

```DAX
Latest YoY Growth % =
VAR LatestYear =
    MAX('Master Sales Database'[Year])

RETURN
CALCULATE(
    [YoY Revenue Growth %],
    'Master Sales Database'[Year] = LatestYear
)
```

### Previous Year Units

```DAX
Previous Year Units =
CALCULATE(
    [Total Units],
    FILTER(
        ALL('Master Sales Database'[Year]),
        'Master Sales Database'[Year]
        = MAX('Master Sales Database'[Year]) - 1
    )
)
```

### YoY Units Growth %

```DAX
YoY Units Growth % =
DIVIDE(
    [Total Units] - [Previous Year Units],
    [Previous Year Units]
)
```

### Latest Units YoY %

```DAX
Latest Units YoY % =
VAR LatestYear =
    MAX('Master Sales Database'[Year])

RETURN
CALCULATE(
    [YoY Units Growth %],
    'Master Sales Database'[Year] = LatestYear
)
```

### Previous Year Avg SP

```DAX
Previous Year Avg SP =
CALCULATE(
    [Avg SP],
    FILTER(
        ALL('Master Sales Database'[Year]),
        'Master Sales Database'[Year]
        = MAX('Master Sales Database'[Year]) - 1
    )
)
```

### YoY Avg SP %

```DAX
YoY Avg SP % =
DIVIDE(
    [Avg SP] - [Previous Year Avg SP],
    [Previous Year Avg SP]
)
```

### Previous Year Active Products

```DAX
Previous Year Active Products =
CALCULATE(
    [Active Products],
    FILTER(
        ALL('Master Sales Database'[Year]),
        'Master Sales Database'[Year]
        = MAX('Master Sales Database'[Year]) - 1
    )
)
```

### YoY Active Products %

```DAX
YoY Active Products % =
DIVIDE(
    [Active Products] - [Previous Year Active Products],
    [Previous Year Active Products]
)
```

### Latest Active Products YoY %

```DAX
Latest Active Products YoY % =
VAR LatestYear =
    MAX('Master Sales Database'[Year])

RETURN
CALCULATE(
    [YoY Active Products %],
    'Master Sales Database'[Year] = LatestYear
)
```

### Previous Year Premium Products

```DAX
Previous Year Premium Products =
CALCULATE(
    [Premium Products],
    FILTER(
        ALL('Master Sales Database'[Year]),
        'Master Sales Database'[Year]
        = MAX('Master Sales Database'[Year]) - 1
    )
)
```

### YoY Premium Products %

```DAX
YoY Premium Products % =
DIVIDE(
    [Premium Products] - [Previous Year Premium Products],
    [Previous Year Premium Products]
)
```

### Latest Premium Products YoY %

```DAX
Latest Premium Products YoY % =
VAR LatestYear =
    MAX('Master Sales Database'[Year])

RETURN
CALCULATE(
    [YoY Premium Products %],
    'Master Sales Database'[Year] = LatestYear
)
```

---

## Product concentration & Pareto

Ranking, share and the cumulative curve behind the Pareto threshold chart.

| Measure | Purpose |
|---|---|
| `Revenue Share %` | Product revenue ÷ total revenue (ignores all filters on the table). |
| `Revenue Rank` | Rank of each product by revenue, descending. |
| `Cumulative Revenue %` | Running share of revenue for all products ranked at or above the current one — the Pareto line. |
| `Top Product Contribution` | Largest single-product revenue share in context. |

### Revenue Share %

```DAX
Revenue Share % =
DIVIDE([Total Revenue],
CALCULATE([Total Revenue],ALL('Master Sales Database')))
```

### Revenue Rank

```DAX
Revenue Rank =
RANKX(
    ALL('Master Sales Database'[Item Name]),
    'Master Sales Database'[Total Revenue],
    ,
    Desc
)
```

### Cumulative Revenue %

```DAX
Cumulative Revenue % =
VAR CurrentRevenue =
    [Total Revenue]

VAR RunningTotal =
    CALCULATE(
        [Total Revenue],
        FILTER(
            ALLSELECTED('Master Sales Database'[Item Name]),
            [Total Revenue] >= CurrentRevenue
        )
    )

VAR TotalRevenue =
    CALCULATE(
        [Total Revenue],
        ALLSELECTED('Master Sales Database'[Item Name])
    )

RETURN
DIVIDE(RunningTotal, TotalRevenue)
```

### Top Product Contribution

```DAX
Top Product Contribution =
MAXX(
    VALUES('Master Sales Database'[Item Name]),
    [Revenue Share %]
)
```

---

## Pricing

| Measure | Purpose |
|---|---|
| `Highest Asp` | Highest selling price evaluated row by row. |
| `Premium Products` | Count of products whose Avg SP exceeds ₹5,000. |

### Highest Asp

```DAX
Highest Asp =
MAXX('Master Sales Database',[Avg SP])
```

### Premium Products

```DAX
Premium Products =
CALCULATE(
    DISTINCTCOUNT('Master Sales Database'[Item Name]),
    FILTER(
        ALL('Master Sales Database'),
        [Avg SP] > 5000
    )
)
```

---

## Lifecycle & risk

| Measure | Purpose |
|---|---|
| `Revenue Growth %` | Revenue vs previous year (same logic as YoY Revenue Growth %), evaluated per product. |
| `Growing Products` | Products with Revenue Growth % > 0. |
| `Declining Products` | Products with Revenue Growth % < 0. |
| `Dead Products` | Products with total revenue under ₹10,000 (shown as *Inactive Products*). |
| `Revenue Volatility` | Population standard deviation of yearly revenue — higher = less predictable demand. |

### Revenue Growth %

```DAX
Revenue Growth % =
DIVIDE(
    [Total Revenue] - [Previous Year Revenue],
    [Previous Year Revenue]
)
```

### Growing Products

```DAX
Growing Products =
CALCULATE(
    DISTINCTCOUNT('Master Sales Database'[Item Name]),
    FILTER(
        VALUES('Master Sales Database'[Item Name]),
        [Revenue Growth %] > 0
    )
)
```

### Declining Products

```DAX
Declining Products =
CALCULATE(
    DISTINCTCOUNT('Master Sales Database'[Item Name]),
    FILTER(
        VALUES('Master Sales Database'[Item Name]),
        [Revenue Growth %] < 0
    )
)
```

### Dead Products

```DAX
Dead Products =
CALCULATE(
    DISTINCTCOUNT('Master Sales Database'[Item Name]),
    FILTER(
        VALUES('Master Sales Database'[Item Name]),
        [Total Revenue] < 10000
    )
)
```

### Revenue Volatility

```DAX
Revenue Volatility =
STDEVX.P(
    VALUES('Master Sales Database'[Year]),
    [Total Revenue]
)
```

---

Calculated columns (`Product Segment`, `Lifecycle Status`) are documented in [data-model.md](data-model.md).
