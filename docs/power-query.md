# Power Query (M)

[← Back to README](../README.md)

## Pipeline

```mermaid
flowchart LR
    S["Source<br/>Excel.Workbook"] --> N["Navigate to sheet<br/>Master Sales Database"]
    N --> H["Promoted Headers"]
    H --> T["Changed Type<br/>7 explicit column types"]
    T --> R["Added Custom<br/>Revenue = Qty × Price"]
    R --> L[("Loaded to model")]
```

## M code — `Master Sales Database`

```powerquery
let
    Source = Excel.Workbook(
        File.Contents("<path>\Master_Sales_Database.xlsx"), null, true
    ),
    #"Master Sales Database_Sheet" =
        Source{[Item = "Master Sales Database", Kind = "Sheet"]}[Data],
    #"Promoted Headers" =
        Table.PromoteHeaders(#"Master Sales Database_Sheet", [PromoteAllScalars = true]),
    #"Changed Type" = Table.TransformColumnTypes(
        #"Promoted Headers",
        {
            {"Serial No",    Int64.Type},
            {"Item Name",    type text},
            {"Item Details", type text},
            {"Price",        Int64.Type},
            {"Quarter",      type text},
            {"Qty",          Int64.Type},
            {"Year",         Int64.Type}
        }
    ),
    #"Added Custom" =
        Table.AddColumn(#"Changed Type", "Revenue", each [Qty] * [Price])
in
    #"Added Custom"
```

> `<path>` replaces the original local folder. Update it via **Transform data → Data source settings → Change source**.

## Step notes

| Step | Purpose |
|---|---|
| Source | Reads the workbook; `true` enables delay-typed columns |
| Navigate | Selects the sheet by name and kind, so extra sheets in the workbook are ignored |
| Promoted Headers | First row becomes column names |
| Changed Type | Explicit types prevent text-number mismatches in measures |
| Added Custom | Row-level revenue, so every measure aggregates one consistent column |

## Planned hardening (v2)

```powerquery
// Normalise product names before load
#"Clean Names" = Table.TransformColumns(
    #"Changed Type",
    {{"Item Name", each Text.Upper(Text.Clean(Text.Trim(Text.Replace(_, "#(00A0)", " ")))), type text}}
),
// Type the calculated column
#"Added Revenue" = Table.AddColumn(#"Clean Names", "Revenue", each [Qty] * [Price], Int64.Type)
```

Rationale in [model-review.md](model-review.md#1-data-quality-findings).
