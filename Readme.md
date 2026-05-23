# APC Customer Renewal Dashboard

A portfolio project demonstrating customer renewal risk analytics for the flexible packaging industry. Built on synthetic data but informed by my work as a pricing analyst at American Packaging Corporation.

At APC I work in Excel every day — quote sheets, margin reviews, quarterly pricing updates. This dashboard is what I'd build if I were redesigning that workflow in Power BI from scratch: a single view that tells account managers who's at risk, how much revenue is on the line, and where to focus. The model, the visuals, the DAX, and the business questions are all built around how flexible packaging actually works as an industry.

> The dataset is synthetic — modeled after real packaging industry structure (customer segments, contract terms, IMP-level pricing) but containing no real APC data. The `.pbix` is in this repo if you want to inspect the full work.

## Questions this dashboard answers

- Which customers are most at risk of not renewing?
- How much revenue is on the line in the next 90 days?
- Which account managers have the heaviest renewal pipeline?
- Where is revenue concentrated by region and segment?
- Is the customer base trending up or down month over month?

## What's in it

Built in Power BI Desktop. Data lives in Excel and feeds a star schema — 3 dimension tables, 4 fact tables, 8 relationships. The dim_Date table is marked for time intelligence so YoY and MoM measures actually work. All relationships flow dimension → fact, single direction.

![Data Model](data-model.png)

## Three pages

### Page 1 — Customer Overview
Three KPI cards at the top (Total Revenue, Total Orders, Total Customers), then revenue by customer, monthly revenue trend, and a region slicer.

![Customer Overview](page1-customer-overview.png)

### Page 2 — Revenue Trends & Breakdown
Where the revenue's coming from. Multi-line chart of revenue over time broken by segment, donut for regional share, and a bar chart by account manager.

![Revenue Trends](page2-revenue-trends.png)

### Page 3 — Renewal Risk
The page that earns its keep. Contracts get a Red/Amber/Green/Expired status based on days until expiry. The color-coded table in the center lets a viewer scan it in five seconds and know who to call. The bar chart on the right ranks account managers by Revenue at Risk so it's clear who has the most cleanup to do.

![Renewal Risk](page3-renewal-risk.png)

## DAX measures and calculated columns

I created a dedicated `_Measures` table so all DAX is in one place rather than scattered across the model.

```dax
Total Revenue = SUM(fact_Orders[LineAmount_USD])

Total Orders = DISTINCTCOUNT(fact_Orders[OrderID])

Total Customers = DISTINCTCOUNT(fact_Orders[CustomerID])

Average Order Value = DIVIDE([Total Revenue], [Total Orders])

Contracts Expiring 90 Days =
CALCULATE(
    DISTINCTCOUNT(Contracts[CustomerID]),
    Contracts[RAG Status] = "Red"
)

Revenue at Risk =
CALCULATE(
    SUM(Contracts[ContractValueUSD]),
    Contracts[RAG Status] IN {"Red", "Expired"}
)

Days Until Renewal = DATEDIFF(TODAY(), Contracts[ContractEnd], DAY)

RAG Status =
SWITCH(
    TRUE(),
    Contracts[Days Until Renewal] < 0, "Expired",
    Contracts[Days Until Renewal] < 90, "Red",
    Contracts[Days Until Renewal] < 180, "Amber",
    "Green"
)
```

## What I'd add next

- A business unit dimension to slice by APC's plant operations (Roto, Flexo, ELNC) — leadership would see renewal risk by line, not just by region
- Quote pipeline layered into the risk score — if a customer's contract is expiring but they're sitting on three active quotes, they're probably not actually at risk
- Parameterized RAG thresholds so the 90/180 day cutoffs aren't hardcoded — they vary by business unit in reality

## Author

Meenakshi Sundar
[GitHub](https://github.com/meenakshisundar06-tech)
