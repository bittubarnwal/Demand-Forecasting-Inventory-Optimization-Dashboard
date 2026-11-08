# Demand Forecasting & Supply Planning

An Excel and Power BI case study showing how demand forecasts support supply and inventory decisions. Built around 20 products from public UCI Online Retail sales data.

## What the project does

Forecasts 12 weeks of product demand following the historical observation period, compares it with assumed supplier deliveries, identifies potential shortages, and suggests replenishment. One dashboard brings the results together.

This independent portfolio project uses public retail data. It demonstrates skills relevant to supply chain and manufacturing planning. Production scheduling, machine capacity and material requirements are outside its scope.

## Project workflow

```mermaid
flowchart LR
    A[Retail sales] --> B[Clean and aggregate]
    B --> C[Compare forecast methods]
    C --> D[12-week demand forecast]
    D --> E[Supply and shortage planning]
    E --> F[Power BI dashboard]
```

1. **Clean:** Excel Power Query removes exact duplicates, cancellations and invalid sales records.
2. **Aggregate:** Select 20 high-volume products using the training period; build 52 complete weeks of sales.
3. **Compare:** Use 44 weeks for training and the next eight for validation. Compare a four-week average (MA4) with nonseasonal Excel ETS.
4. **Forecast:** Choose the lower-MAE method for each product and forecast 12 weeks, from 5 December 2011 to 20 February 2012.
5. **Plan:** Compare demand with illustrative opening stock and supplier deliveries. Calculate nonnegative closing stock and unmet demand, treated as lost sales.
6. **Recommend:** Use lead time, safety stock and outstanding orders to suggest replenishment in a separate policy simulation. Suggestions are not confirmed receipts.
7. **Review:** Explore products and planning scenarios in Power BI.

## Dashboard

- **Two filters:** product and optional planning scenario (Baseline initially).
- **Four cards:** forecast demand, validation WAPE, planned supply and projected unmet demand.
- **Main chart:** historical sales and future baseline forecast. Future forecasts begin in December 2011; validation forecasts are omitted from this chart to make the boundary clearer.
- **Supply chart:** weekly scenario demand versus planned receipts.
- **Product table:** product, selected forecast method, forecast demand, validation WAPE, planned supply, unmet demand and suggested orders.

The report uses one page for the forecast overview and product-level planning. Detailed calculations are available in the Excel workbook.

Scenario choices are Baseline, Demand +20%, and Supply delay 2w. Scenario changes affect planning demand, supply, shortages and recommendations. Historical sales, baseline forecasts, selected methods and validation errors do not change with a hypothetical scenario.

## Results and limitations

| Baseline measure | Result |
|---|---:|
| 12-week forecast demand | 153,938 units |
| Planned receipts within the horizon | 115,482 units |
| Projected unmet demand | 24,981 units |
| Suggested order quantities | 59,063 units |
| Selected-method validation WAPE | 54.35% |

WAPE is total absolute forecast error divided by total actual sales in the validation period. Lower is better; it is **not an accuracy percentage**. Product-level WAPE uses the same definition within each product. The overall value is a ratio of totals, not an average of product percentages.

The same validation window was used to select the methods, so these are model-selection results, not independent test results. Improving performance requires additional experiments and an untouched evaluation period.

Sales are a proxy for demand. Opening stock, supply, safety coverage and lead times are illustrative. Suggested orders are simulated separately and do not eliminate shortages in the original supply plan. These results do not demonstrate realized savings.

## Files and tools

| File/tool | Purpose |
|---|---|
| [Demand_Supply_Inventory.xlsx](Demand_Supply_Inventory.xlsx) | Power Query cleaning, Excel forecasting, assumptions and planning calculations |
| [Demand_Supply_Dashboard.pbix](Demand_Supply_Dashboard.pbix) | One interactive Power BI report, with DAX summaries |

## Excel workbook guide

Open **Start_Here**, the first worksheet, for the workflow, live forecast/error summary, links to all seven supporting sheets, and update instructions.

- **Blue input cells:** `Product_Master!E2:H21` for lead time, safety, opening stock and review coverage; `Scenario_Analysis!B2:C4` for demand multipliers and supply delays. Header notes explain each input.
- **Teal tabs:** demand history and forecasts.
- **Blue tabs:** editable planning assumptions.
- **Grey tabs:** supporting calculations and data quality.
- **Source file setting:** edit `Start_Here!C28` only if the original UCI workbook moves. Power Query reads it through the preserved `SourcePath` named cell.

The workbook has eight sheets. The former `Read_Me` guide is merged into `Start_Here`; the duplicate weekly PivotTable and chart have been removed. Existing table names, column names, formulas, query definitions and planning results are preserved for Power BI compatibility.

## Open and update

**Requirements:** Microsoft Power BI Desktop on Windows and Microsoft Excel Desktop with Power Query and `FORECAST.ETS` support. Download the files locally; GitHub does not render an interactive PBIX report. The PBIX includes saved results, so the raw dataset is needed only to refresh the source data.

1. Open the PBIX in Power BI Desktop to explore the saved results.
2. To change assumptions, start with the guide on `Start_Here`, then edit the blue input cells in Excel. Recalculate and save Excel, then refresh Power BI.
3. For a source refresh, ensure the original `Online Retail.xlsx` exists at the path in `Start_Here!C28`; run Excel **Data > Refresh All**, recalculate, and save.
4. If the project folder moves, update Power BI's `WorkbookPath` parameter through **Transform data > Manage parameters**.

Dashboard filters do not modify Excel assumptions. The workbook is designed for the fixed 20-product, 52-week case study; changing its product population or historical period requires rebuilding the formula blocks.

## Project summary

> I used Excel and Power BI to connect demand forecasting with supply planning. I cleaned retail transactions, built weekly sales histories for 20 products, and compared a four-week average with exponential smoothing. I selected a method per product using historical validation, forecast the next 12 weeks, and compared demand against assumed supply. My dashboard highlights forecast reliability, shortages and suggested replenishment. In manufacturing, these demand forecasts could feed production planning, with additional capacity and material data.

## Data source

UCI Online Retail, Daqing Chen, CC BY 4.0. [Dataset and attribution](https://archive.ics.uci.edu/dataset/352/online%2Bretail). The raw workbook is stored separately from the deliverables.

## Dashboard verification

The report was opened and saved in Power BI Desktop. Visual inspection confirmed one page, populated charts and the product table, and Baseline cards of 153.9K forecast demand, 54.3% validation WAPE, 115.5K planned supply and 25.0K unmet demand. Report package integrity was also checked. Displayed card values are rounded; the results table above uses greater precision.

The Excel navigation update was saved in Excel Desktop. All seven existing named tables were checked for unchanged ranges, formulas and values, along with all seven Power Query definitions and the source-path input. The new summary formulas reproduce 153,938.245 forecast units and 54.349138% WAPE.

After sheet cleanup, the seven core named tables, formulas, values and Power Query definitions were compared with the backup and remained unchanged. `SourcePath` now refers to `Start_Here!C28`.
