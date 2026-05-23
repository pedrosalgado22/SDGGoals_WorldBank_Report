# Global Socio-Economic and Gender Equity Outlook

Power BI Report available [here](https://app.powerbi.com/view?r=eyJrIjoiM2Q1YmI4NmItZmMyZi00YTE1LWIxMDktYWI1ODQ5YWNjZDA2IiwidCI6ImU0YmQ2OWZmLWU2ZjctNGMyZS1iMjQ3LTQxYjU0YmEyNDkwZSIsImMiOjh9).

An interactive Business Intelligence report built in Power BI for the World Bank, examining gender equality through the lens of SDG 5 and its intersections with SDG 3 (Good Health and Well-being), SDG 4 (Quality Education), and SDG 8 (Decent Work and Economic Growth). Twelve analysis areas are covered across 14 datasets from the World Bank, IMF, Eurostat, and ITU, spanning two decades of global data and supporting full drill-down, roll-up, and cross-filtering interactivity.

**Course:** Databases II / Business Intelligence 2025/2026 — Nova IMS, Universidade Nova de Lisboa  
**Consulting Identity:** Cardial Enterprises

---

## Research Question

*How do gender disparities manifest across health, education, labour, legal rights, and economic participation globally, how have they evolved over the past two decades, and what structural factors — legal, economic, or cultural — appear most associated with better or worse outcomes for women?*

---

## Repository Contents

| File | Description |
|---|---|
| `DBIIProject.pbix` | Power BI Desktop file — full semantic model, all DAX measures, and all 12 report pages |
| `Data.zip` | All 14 raw and transformed source datasets used in the model |
| `DAXMEASURES.xlsx` | Reference table documenting all DAX measures developed for the project |
| `SERIESNAME.xlsx` | Reference table documenting all series names selected from each dataset |
| `WB_Presentation.pdf` | Slide deck — cover, table of contents, 12 analysis slides, SDG alignment slide, and conclusions |
| `WB_Report.pdf` | Written report — data model documentation, M query and DAX measure explanations, technical decisions, and APA 7 references |
| `LICENSE` | MIT License |

---

## Analysis Areas

| Page | Title | Primary Dataset | SDG Connection |
|---|---|---|---|
| 1 | Childbirth and Survival | Health, Nutrition and Population Statistics (WB) | SDG 3 |
| 2 | Representation in Power | World Development Indicators (WB) | SDG 5 |
| 3 | Women at Work | Jobs Statistics (WB) | SDG 8 |
| 4 | The Pay Gap in Europe | Gender Pay Gap — Eurostat | SDG 8 |
| 5 | Women's Health Around the World | Health Equity and Financial Protection — HEFPI (WB) | SDG 3 |
| 6 | Girls in School | Education Statistics (WB) | SDG 4 |
| 7 | The Literacy Divide | Gender Statistics (WB) | SDG 4 |
| 8 | Access to Finance | Global Findex Database (WB) | SDG 8 |
| 9 | Fertility and Demographic Change | Population Estimates and Projections (WB) | SDG 3 / SDG 5 |
| 10 | The Digital Divide | ICT Data Portal — ITU | SDG 5 |
| 11 | A Composite View of Inequality | Gender Inequality Index — IMF | SDG 5 |
| 12 | Laws, Rights, and Constitutions | SDG 5 Statistics (WB) | SDG 5 |

---

## Data

Fourteen datasets were loaded, cleaned, and transformed into a shared long-format schema with one observation per country per indicator per year. All datasets were sourced from their most recent publicly available versions.

| Dataset | Source | Primary Page | Series Selected |
|---|---|---|---|
| Health, Nutrition and Population Statistics | World Bank | 1 | Maternal mortality ratio, infant mortality, births attended by skilled staff, adolescent fertility rate |
| World Development Indicators | World Bank | 2 | Proportion of seats held by women in national parliaments (%) |
| Jobs Statistics | World Bank | 3 | Female and male labour force participation rate (ILO modelled), youth literacy, lifetime risk of maternal death |
| Gender Pay Gap (SDG_05_20) | Eurostat | 4 | Gender pay gap in unadjusted form |
| Health Equity and Financial Protection (HEFPI) | World Bank | 5 | Survival to age 65, antiretroviral therapy coverage, female/male |
| Education Statistics | World Bank | 6 | Gross enrolment ratios by level and sex |
| Gender Statistics | World Bank | 7 | Adult and youth literacy rates by sex |
| Global Findex Database | World Bank | 8 | Account ownership by sex, financial institution access |
| Population Estimates and Projections | World Bank | 9 | Fertility rate, infant mortality by sex, sex ratio at birth |
| ICT Data Portal | ITU | 10 | Individuals using the Internet |
| Composite Gender Indices (GII) | IMF | 11 | Gender Inequality Index (1990–2014) |
| SDG 5 Statistics | World Bank | 12 | Women Business and Law Index, female senior management share, constitutional gender clause |
| World Economic Outlook (April 2025) | IMF | Cross-indicator | GDP per capita PPP, real GDP growth |
| Human Capital Index | World Bank | Cross-indicator | Human Capital Index (HCI) score |

The model satisfies the mandatory source requirement: ten World Bank datasets, one IMF primary source, one Eurostat source, one ITU source, and one additional IMF source used as a cross-indicator.

---

## Semantic Model

The semantic model follows a star schema with two shared dimension tables connecting all fourteen fact tables.

**Calendar** — created with `CALENDARAUTO()` and subsequently replaced with a DAX `ADDCOLUMNS` expression whose date range is bounded dynamically using a `MIN` over `MAX` expressions across representative fact tables. This prevents ghost years from appearing in time intelligence visuals when the year slicer is set to Select All. The table is marked as a date table using `Calendar[Date]`. Attributes include Year, Month, Quarter, and MonthSort (YYYY-MM for correct chronological sorting).

**Geography** — built in Power Query via a web connection to the United Nations geoscheme on Wikipedia, harmonised to the World Bank country classification standard through a two-stage concordance merge. Contains Country Code, Country Name, Subregion, and Continent. An inner join against the World Bank Country Mapping Table automatically excludes overseas territories and non-sovereign entities.

All fourteen fact tables share a consistent output schema: Country Name, Country Code, Series Name, Year, Value, and Date. Each is connected to Calendar via `Calendar[Date]` and to Geography via `Geography_table[Country Code]` using one-to-many relationships with single-direction filtering from dimensions to facts.

A `RowsNotEmpty` calculated column was added to the Calendar table for each of the fourteen fact tables, using the pattern `CALCULATE ( NOT ISEMPTY ( 'FactTable' ) )`. This enables the `LASTDATE` and `FIRSTDATE` boundary-value measure pattern across all percentage and ratio indicators in the model.

---

## DAX Measures

All measures are stored in a dedicated DAX Measures table created via Enter Data and are explicitly defined — no Quick Measures were used. The full set is documented in `DAXMEASURES.xlsx`.

Key patterns applied across the model:

**Boundary-value pattern for non-additive indicators** — all percentage and ratio series use `CALCULATE` with `LASTDATE` or `FIRSTDATE` applied to a `CALCULATETABLE` expression that filters `Calendar[Date]` to rows where the relevant `RowsNotEmpty` column is `TRUE`. This prevents meaningless aggregation of point-in-time snapshots across years.

```dax
Maternal Mortality Rate =
CALCULATE (
    AVERAGE ( 'Health_Nutritition_Statistics_WB'[Value] ),
    LASTDATE (
        CALCULATETABLE (
            VALUES ( 'Calendar'[Date] ),
            'Calendar'[RowsNotEmpty_HealthNutrition] = TRUE
        )
    )
)
```

**Filter context manipulation** — `ALL`, `REMOVEFILTERS`, `ALLEXCEPT`, and `KEEPFILTERS` are used to construct denominators for share and benchmark measures. `AVERAGEX` over `VALUES` or `ALL` on Geography produces global and regional averages that remain stable under country-level slicers.

**Threshold and ranking measures** — `COUNTROWS` with `FILTER` over `VALUES` on Country Code counts countries meeting defined analytical thresholds. `RANKX` over `ALL` on Country Code ranks countries within the current year context.

**Cross-table measures** — several pages draw measures from more than one fact table simultaneously, such as the Lifetime Maternal Death Risk measure on page 1 sourcing from `Jobs_Statistics_WB` while the page's primary fact table is `Health_Nutritition_Statistics_WB`.

Selected measure examples by analysis area:

| Page | Measure | Pattern |
|---|---|---|
| 1 | Maternal Mortality Rate | LASTDATE + CALCULATETABLE + RowsNotEmpty |
| 1 | Countries Exceeding 100 | COUNTROWS + FILTER + VALUES |
| 1 | Regional Average Maternal Mortality | CALCULATE + ALL (Geography) |
| 2 | Political to Corporate Leadership Gap | Derived subtraction of two LASTDATE measures |
| 2 | Global Average Women in Parliament | AVERAGEX + ALL (Geography) |
| 3 | Gender Participation Gap | Derived subtraction of LASTDATE + AVERAGE measures |
| 3 | GDP per Capita PPP | LASTDATE cross-table from IMF_WEO_Statistics_ |
| 4 | European Average Pay Gap | AVERAGEX + VALUES (Country Code) + ALL (Geography) |
| 6 | Gender Parity Index Secondary | DIVIDE of two LASTDATE measures |
| 8 | Countries with Financial Inclusion Gap Above 10 | COUNTROWS + FILTER + CALCULATE |
| 11 | GII Rank | RANKX + ALL (Country Code) + DESC + DENSE |
| 12 | Gender Constitutional Clause | MAX + REMOVEFILTERS (Calendar) |
| 12 | Average WBL with Constitutional Clause | AVERAGEX + FILTER |

---

## Power Query (M)

All queries are organised into Staging and Model groups. A `DataFolderPath` parameter created via Manage Parameters stores the local data folder path and is referenced in the Source step of every query, ensuring the solution remains portable when the storage location changes.

All ten World Bank datasets arrived in wide format and followed a consistent transformation sequence: remove Series Code, select identifier columns, unpivot year columns using Unpivot Other Columns, extract the four-character year from the Attribute column, rename to Year and set to Whole Number, set Value to Decimal Number, replace conversion errors with null, and add a Date column constructed as the first of January of each year. All applied steps were renamed with descriptive labels documenting the purpose of each transformation.

The four external datasets required adapted approaches. The Eurostat dataset was loaded via the Excel connector and required removal of metadata header rows, footer rows, and Eurostat flag columns before unpivoting. The IMF Gender Inequality Index required sheet selection, removal of the Estimates Start After metadata column, and a two-stage concordance merge resolving over thirty country name mismatches against the World Bank standard. The ITU and HCI datasets arrived in long format and required only column selection, renaming, and Date column addition.

Country identifier harmonisation was implemented through two concordance tables for the IMF GII dataset (one for codes, one for names) and a country mapping table built from the Education Statistics query. A Wikipedia UN geoscheme web connection provided subregion and continental region attributes for Geography, merged via the same concordance pipeline.

---

## Report Structure and Design

Each of the twelve analysis pages follows a standardised structural template. The top strip contains the page number and topic label, three dropdown slicers for Year, Continent, and Country, and a navigation arrow. A full-width bold finding headline states the key analytical conclusion of the page, formatted with a thin coloured left-border accent rectangle. Two main analytical visual areas sit below the headline, each with its own sub-headline. The bottom section contains key findings as bullet points, source attribution, and the Cardial Enterprises logo. The canvas background uses a light lavender brand colour with individual chart areas rendered on white card backgrounds with shadow formatting.

The page canvas height is set to a custom value enabling vertical scrolling, replicating the scrollable narrative layout of World Bank interactive publications while remaining fully deliverable in PBIX format and readable on mobile via Power BI Service.

Custom Deneb visuals using Vega-Lite specifications are used on pages 1 and 4. The page 4 dot matrix represents the European average gender pay gap as a ten-by-ten proportional unit chart, with dots coloured to distinguish earned share from the gap.

---

## Key Findings

- Maternal mortality has been virtually eradicated in Western economies while Sub-Saharan Africa carries a disproportionate share of global deaths. African maternal deaths fell 43.4% from 30,586 in 2000 to 17,303 in 2020 — meaningful progress that still falls short of the SDG 3.1 target of below 70 per 100,000 live births.
- Women hold approximately one quarter of parliamentary seats and slightly more than one third of management roles globally. The female share of parliamentary seats more than doubled from 12% in 2000 to 26% in 2025; management share increased by eight percentage points over the same period.
- Female labour force participation remains well below male participation globally. The relationship between GDP per capita and female participation is not linear — high rates appear in both very poor and very rich regions — suggesting cultural and social structures are stronger determinants than income level alone.
- The gender pay gap within Europe varies considerably across member states. Countries widely considered more progressive in social policy do not uniformly outperform the European average on this indicator.
- Legal protections for women encoded in constitutions do not reliably translate into better outcomes on representation, health, or economic indicators, revealing a persistent gap between formal rights and lived equality.
- The digital gender divide remains significant in low-income regions, with women disproportionately excluded from internet access in Sub-Saharan Africa and South Asia, compounding exclusion from economic and educational opportunity.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Power BI Desktop | Semantic model, DAX development, report building, and visualisation |
| Power Query (M) | Data extraction, transformation, harmonisation, and loading for all 14 datasets |
| DAX | All analytical measures, calculated columns, and the Calendar table |
| Deneb + Vega-Lite | Custom dot matrix visuals on pages 1 and 4 |
| Microsoft Excel | Preliminary data inspection and concordance table authoring |
| Azure Maps | Filled map visuals for geographic distribution analysis |
| Gemini (Google) | Assistance with Vega-Lite specification for the dot matrix visual on page 4 (documented in written report) |

---

## Data Sources

World Bank Open Data. *Health Nutrition and Population Statistics, World Development Indicators, Jobs Statistics, Education Statistics, Gender Statistics, Global Findex Database, Population Estimates and Projections, SDG 5 Statistics, Health Equity and Financial Protection (HEFPI), Human Capital Index*. [https://data.worldbank.org](https://data.worldbank.org)

International Monetary Fund. *World Economic Outlook Database (April 2025)*. [https://www.imf.org/en/Publications/WEO](https://www.imf.org/en/Publications/WEO)

International Monetary Fund. *Composite Gender Indices (Gender Inequality Index)*. [https://www.imf.org](https://www.imf.org)

Eurostat. *Gender Pay Gap in Unadjusted Form (SDG_05_20)*. [https://ec.europa.eu/eurostat](https://ec.europa.eu/eurostat)

International Telecommunication Union. *ICT Data Portal — Individuals Using the Internet*. [https://datahub.itu.int](https://datahub.itu.int)

United Nations Statistics Division. *UN Geoscheme — Geographic Regions*. [https://unstats.un.org/unsd/methodology/m49/overview/](https://unstats.un.org/unsd/methodology/m49/overview/)

---

## Report and Presentation

Full technical documentation, M query explanations, DAX measure documentation, data model diagrams, and analytical conclusions are in [`WB_Report.pdf`](./WB_Report.pdf).

The slide deck with one slide per analysis area, key findings, illustrative graphs, and source attribution is in [`WB_Presentation.pdf`](./WB_Presentation.pdf).

---

*Nova IMS — Universidade Nova de Lisboa | Databases II / Business Intelligence 2025/2026*
