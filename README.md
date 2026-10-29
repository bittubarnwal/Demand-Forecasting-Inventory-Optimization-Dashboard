# Demand forecasting, supply planning and inventory optimization

Portfolio case study for a GET – Logistics & Warehousing application. This is independent analysis of public retail data, not an ITC project or an analysis of ITC operations.

## 1. Project overview

The workbook connects real historical sales to a transparent planning model for 20 products. Excel Power Query performs the cleaning and weekly aggregation; Excel formulas calculate forecasts, validation errors, hypothetical receipts, inventory and replenishment. Power BI imports the saved Excel tables into a star schema and presents three interactive pages.

The planning horizon is **5 December 2011–20 February 2012**, the 12 weeks following the last complete source week. These are historical case-study forecasts, not forecasts for today's business.

## 2. Business problem and objectives

Estimate future fulfilled-sales demand, compare it with illustrative supply, identify safety-stock breaches and unmet demand, and propose replenishment using lead time and an explicit target-stock policy. Compare a baseline with a 20% demand increase and a two-week supply delay. Do not claim realized savings or operational service improvements from this demonstration.

## 3. Dataset and official download

- [UCI Online Retail dataset](https://archive.ics.uci.edu/dataset/352/online%2Bretail)
- [Official ZIP download](https://archive.ics.uci.edu/static/public/352/online+retail.zip)
- Source workbook: `Online Retail.xlsx`, worksheet `Online Retail`.
- Eight source columns: InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID and Country.
- UCI attribution: Daqing Chen, Online Retail, UCI Machine Learning Repository, DOI [10.24432/C5BW33](https://doi.org/10.24432/C5BW33), CC BY 4.0.

The original file is stored separately at `C:\Users\BITTU KUMAR\AppData\Local\DemandSupplyProject\Online Retail.xlsx`. The three project deliverables do not include another copy of the raw data. The source path is editable in `Read_Me!B5`.

## 4. Tools used

Microsoft Excel 16.0, Excel Power Query M, Excel formulas including FORECAST.ETS, a PivotTable and charts; Microsoft Power BI Desktop, Power Query M and DAX. Windows automation was used to author and inspect the native files. Business-data transformations and calculations run in Excel/Power Query/Power BI, with no Python, SQL, R, external forecasting service or machine-learning platform.

## 5. End-to-end workflow

Historical sales → schema enforcement → duplicate audit → mutually exclusive cleaning exclusions → complete Monday weeks → top-product cohort → dense product/week grid → chronological validation → selected 12-week forecast → illustrative receipts → lost-sales inventory balances → advisory replenishment → scenarios → Power BI.

The workbook contains exactly eight worksheets. Imported tables and formula tables have filters, frozen headers and consistent formats. Blue cells identify editable planning assumptions. Conditional formatting highlights unmet demand and stock below safety stock.

## 6. Data cleaning and quality rules

`Data_Quality` contains observed raw issue counts, sequential exclusion counts and reconciliation checks. Raw flags overlap and must not be added together.

Apply the following priority after removing exact duplicates across all eight original columns:

1. Exclude missing InvoiceNo, StockCode, InvoiceDate, Quantity or UnitPrice.
2. Exclude InvoiceNo beginning with C, case-insensitive.
3. Exclude remaining Quantity ≤ 0.
4. Exclude remaining UnitPrice ≤ 0.
5. Exclude nonmerchandise codes whose first character is not a digit.
6. Retain all other rows.

Missing customer IDs and descriptions are audited but do not invalidate product-unit sales. Description is a deterministic source label (first alphabetically sorted non-null description for a product in the ranking period), not a newly invented product identity. Product categories are explicitly illustrative mappings. Country is present in the source but pooled across countries for planning; a country slicer is deliberately omitted because neither the forecast nor illustrative warehouse inventory is country-specific.

Power Query explicitly types identifiers as text, Quantity as integer, InvoiceDate as datetime and UnitPrice as number. Type conversion failures must stop a refresh for investigation; do not silently replace errors. A duplicate means an identical eight-field row; the source has no unique line ID, so legitimate repeated identical purchases might be removed by this conservative rule. Cancellation/return transactions are not netted into the gross fulfilled-sales series and do not become inventory receipts.

Reconciliation: original records = duplicate excess + all mutually exclusive exclusions + retained records. Retained units = selected-product complete-week units + other-product complete-week units + partial-week units. Weekly selected-product units must equal the selected-product source aggregate.

## 7. Weekly aggregation and cohort selection

Weeks start Monday. Exclude the incomplete source boundary weeks starting 29 November 2010 and 5 December 2011. The observation window is the 52 complete weeks from 6 December 2010 through 28 November 2011.

Select the 20 products with the largest total retained fulfilled units in the **44-week training period**, ending 3 October 2011. Break ties by StockCode ascending. This documented ranking window ensures the later validation actuals do not influence cohort selection. The chart ranks historical units within this selected cohort; it is not a claim that these remain the full-period top 20.

`tblWeekly` contains 20 × 52 = 1,040 rows, sorted by StockCode then WeekStart. UnitsSold sums positive retained quantities; SalesValue sums Quantity × UnitPrice in GBP. A full product/week grid distinguishes a missing aggregate from a recorded transaction. Grid gaps become zero *recorded sales* and carry an availability-unknown label. This does not prove zero customer demand: stockouts, late listings and incomplete extraction cannot be observed. No future source dates are invented as actual sales.

## 8. Forecast methods and chronological validation

Training: 44 weeks, 6 December 2010–3 October 2011. Validation: eight weeks, 10 October–28 November 2011. The eight forecasts share one origin immediately after the training window; none uses any validation actual in its inputs.

- **MA4:** mean of the four final training weeks, held constant across the eight-week holdout. For the final 12-week forecast, recalculate the mean using the last four complete historical weeks and hold that level constant. This is a fixed-origin level baseline, not a moving average recalculated with future actuals.
- **ETS:** `MAX(0,FORECAST.ETS(target,values,timeline,0,1,1))`. Seasonality is explicitly 0; one annual cycle is insufficient evidence for annual seasonal estimation. Inputs use only the training window for validation and all 52 historical weeks for the future forecast. Errors/unsupported ETS return blank, never a fabricated score.
- Select the lower product-level holdout MAE; exact ties and incomplete ETS coverage select MA4. ETS requires all eight numeric validation forecasts.
- Final forecasts are nonnegative, potentially fractional expected units. Advisory orders and assumed safety/opening stock round up to whole units.

The same holdout selects the method and measures its performance. Selected-method metrics are therefore **model-selection validation results**, not an untouched test-set accuracy estimate. Sparse/extreme retail orders can dominate error. Selection should be retested on additional data before operational use.

`tblForecast` has 400 rows: 160 validation and 240 future rows. ActualUnits is blank for future rows. Error = selected forecast − actual; positive signed error means overforecasting. Candidate errors remain visible alongside the selected result.

## 9. MAE and WAPE

MAE = SUM(absolute error) / number of validated product-weeks. It expresses average error in units. WAPE = SUM(absolute error) / SUM(actual units), with a blank result if the denominator is zero. Overall WAPE is a ratio of totals, never an average of product percentages. MAE/WAPE compare validation rows only; future actuals are unknown.

Measured results and the observed source audit appear in the validation section at the end of this file.

## 10. Supply planning

Each product starts with illustrative stock equal to two average forecast weeks. Three illustrative supplier labels are assigned cyclically; these are not actual suppliers. Baseline receipts occur in planning weeks 3, 7 and 11, each at three average forecast weeks rounded up. They intentionally create a constrained supply case for analysis, not an optimized purchasing plan.

`tblSupply` has 60 receipt rows per scenario, 180 in total. Each scenario is an alternative world; never add all scenarios together. BaseReceiptDate preserves the original illustrative schedule. ExpectedReceiptDate adds the scenario delay. Receipt quantities are the same in all three scenarios, including demand uplift. A delay can move the final receipt beyond the 12-week horizon; that receipt remains in the schedule but is not counted as in-horizon supply.

Scenario multipliers and delays are editable in `Scenario_Analysis`. Product lead time, safety weeks, opening-stock weeks and review weeks are editable in `Product_Master`. To configure individual supply commitments, replace the appropriate illustrative receipt formula with an assumption and record the change; retain the scenario key and use Monday receipt dates. The date granularity is weekly: receipts are available at the start of the week.

## 11. Inventory projection

`tblInventory` contains 20 × 12 × 3 = 720 rows. Each scenario/product sequence is independent.

```text
AvailableStock       = OpeningStock + PlannedReceipts
PhysicalClosingStock = MAX(0, AvailableStock - ForecastDemand)
UnmetDemand          = MAX(0, ForecastDemand - AvailableStock)
Next OpeningStock    = Previous PhysicalClosingStock
BalanceCheck         = AvailableStock - ForecastDemand + UnmetDemand - PhysicalClosingStock
```

Unmet demand is treated as **lost sales**, not a backorder. No negative physical inventory is permitted. These are projected outcomes, not recorded warehouse balances. Safety stock remains fixed across scenarios to expose the pressure created by higher demand. Stockout status takes priority over below-safety status; exactly meeting safety stock is Healthy.

## 12. Reorder point, safety stock and advisory orders

SafetyStock = ROUNDUP(SafetyWeeks × average 12-week baseline forecast, 0). It is a configurable coverage assumption, not a statistically calibrated service-level safety stock. Default lead time is two weeks; default review period is one week.

```text
ReorderPoint = lead weeks × scenario average weekly demand + safety stock
TargetStock  = (lead weeks + review weeks) × scenario average demand + safety stock
InventoryPosition = policy closing stock + outstanding suggested orders
                    + committed receipts due within lead time
SuggestedReplenishment = IF(position < reorder point,
                            ROUNDUP(MAX(0,target-position),0), 0)
```

The policy uses a **separate simulated ledger**. Previous suggestions enter PolicyClosingStock only when their SuggestedArrivalWeek is reached; they never enter the baseline PhysicalClosingStock. OutstandingSuggestions includes only previous recommendations that have not arrived. Committed receipts are included once in each independent ledger. This prevents repeated recommendations for the same exposure and avoids falsely treating advisory orders as committed receipts.

Suggestions are evaluated at week end, with arrival labelled current WeekStart + seven times LeadTimeWeeks. That is a weekly bucket convention, not an exact daily lead-time promise. A two-week lead time means receipt at the start of the week two buckets later. Positive integer lead times are required. The target uses average forecast demand, including for coverage beyond the visible horizon; it is a transparent approximation rather than a time-varying stochastic optimizer. Some suggestions arrive after the display horizon. Suggestions cannot retrospectively prevent earlier lost sales.

## 13. Hypothetical scenarios

| Scenario | Demand multiplier | Receipt delay |
|---|---:|---:|
| Baseline | 1.00 | 0 weeks |
| Demand +20% | 1.20 | 0 weeks |
| Supply delay 2w | 1.00 | 2 weeks |

Scenario names describe defaults; rename labels consistently in all tables if changing their meaning. Summary closing stock is the sum across products at the **last** planning week, not a sum of weekly stock snapshots. Breach and stockout counts on the Excel summary are product-weeks. Power BI risk cards count distinct products with at least one breach in the selected weeks; these are intentionally different units.

## 14. Power BI dashboard pages

Exactly three pages use model fields and DAX measures:

1. **Demand Forecasting:** historical units, future forecast, holdout MAE/WAPE; actual/forecast timeline; product trends; MA4-versus-ETS future forecast comparison; selected-cohort historical ranking; accuracy table.
2. **Inventory Planning:** final selected-week stock, distinct products below safety and with unmet demand, unmet units; stock/safety timeline; product risk; weekly status matrix; advisory orders; weekly projection table.
3. **Supply Planning:** scenario demand, in-horizon receipts, supply minus demand, advisory replenishment; weekly supply/demand; gap; recommended orders; scenario comparison; illustrative supplier receipt analysis.

Light backgrounds, white visual panels and teal charts distinguish the report. Product and week slicers filter through dimensions. Scenario is a single-choice slicer on planning pages. When no scenario is selected, aggregate planning measures deliberately use Baseline; a scenario chart evaluates each scenario independently in its axis context. Clear the scenario slicer to compare all scenarios. Date selection can blank accuracy measures when no validation weeks remain, and can blank future demand when no future weeks remain. Clear the week slicer for the full case overview.

