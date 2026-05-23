# Measuring Equality: A Portrait of Women's Progress


## A Cross-Regional Analysis of Gender Equity Progress Under SDGs 3, 4, 5, and 8 across the 2000s




---


Power BI Report available [here](https://app.powerbi.com/view?r=eyJrIjoiM2Q1YmI4NmItZmMyZi00YTE1LWIxMDktYWI1ODQ5YWNjZDA2IiwidCI6ImU0YmQ2OWZmLWU2ZjctNGMyZS1iMjQ3LTQxYjU0YmEyNDkwZSIsImMiOjh9).

An interactive Business Intelligence report built in Power BI for the World Bank, examining gender equality through the lens of SDG 5 and its intersections with SDG 3 (Good Health and Well-being), SDG 4 (Quality Education), and SDG 8 (Decent Work and Economic Growth). Twelve analysis areas are covered across 15 datasets from the World Bank, IMF, Eurostat, and ITU, spanning two decades of global data and supporting full drill-down, roll-up, and cross-filtering interactivity.

**Course:** Business Intelligence 2025/2026 — Nova IMS, Universidade Nova de Lisboa  
**Consulting Identity:** Cardial Enterprises

---

<img width="1297" height="719" alt="Screenshot_20260523_103954" src="https://github.com/user-attachments/assets/c1273ea5-85c0-443e-bd09-f6a112eae791" />


---

## Research Question

*How do gender disparities manifest across health, education, labour, legal rights, and economic participation globally, how have they evolved over the past two decades, and what structural factors (legal, economic, or cultural) appear most associated with better or worse outcomes for women?*

---

## Repository Contents

| File | Description |
|---|---|
| `DBIIProject.pbix` | Power BI Desktop file with full semantic model, all DAX measures, and all 12 report pages |
| `Data.zip` | All 14 raw and transformed source datasets used in the model |
| `DAXMEASURES.xlsx` | Reference table documenting all DAX measures developed for the project |
| `SERIESNAME.xlsx` | Reference table documenting all series names selected from each dataset |
| `WB_Presentation.pdf` | Slide deck, including cover, table of contents, 12 analysis slides, SDG alignment slide, and conclusions |
| `WB_Report.pdf` | Written report with data model documentation, M query and DAX measure explanations, technical decisions, and APA 7 references |
| `LICENSE` | MIT License |

---

## Analysis Areas

| Page | Title |
|---|---|
| 1 | Maternity Mortality |
| 2 | Women in Political and Corporate Leadership |
| 3 | Woman in the workforce |
| 4 | Wage Gap in Europe |
| 5 | Women's Health |
| 6 | Gender Gap in Education Access |
| 7 | Gender Literacy Divide |
| 8 | Financial Inclusion |
| 9 | Fertility Rates and Life Expectancy |
| 10 | The Digital Gender Divide and Internet Access |
| 11 | IMF Gender Inequality Index |
| 12 | Women's Rights and SDG5 Progress |

---

<img width="1452" height="838" alt="Screenshot_20260523_105629" src="https://github.com/user-attachments/assets/429f578c-2dc7-4ba8-a41b-d5e3d9b42fe2" />

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

<img width="1458" height="836" alt="Screenshot_20260523_105651" src="https://github.com/user-attachments/assets/734f34e9-8f8f-4da1-a0e9-53873d7b3c45" />


---

## Semantic Model

The model follows a star schema. Two shared dimension tables, Calendar and Geography_table, connect to all fourteen fact tables through one-to-many relationships with single-direction filtering from dimensions to facts. The join keys are `Calendar[Date]` and `Geography_table[Country Code]`. Country Code was preferred over Country Name as the geography key due to its greater stability and the significant name treatment required throughout the project.

The Calendar dimension was created using `ADDCOLUMNS` over a `CALENDAR` expression enriched with Year, Month, Quarter, and MonthSort (YYYY-MM for correct chronological sort order), and marked as a date table. `CALENDARAUTO` was initially used but generated ghost years extending back to 1970, causing rendering and filtering issues. The table was standardised to a fixed start of January 1, 2000. The end date is computed dynamically using a `MAXX` expression across all fourteen fact tables, ensuring the Calendar extends automatically to the latest year present across the model when data is refreshed.

One `RowsNotEmpty` calculated column was added to the Calendar table per fact table, returning TRUE only for dates where the corresponding fact table contains at least one observation. All `LASTDATE` and `FIRSTDATE` measures wrap the date argument in `CALCULATETABLE` filtered to `RowsNotEmpty = TRUE`, ensuring boundary dates are restricted to valid observation dates. This is the core architectural response to the non-additive nature of the data.

All DAX measures are held in a dedicated empty table created via Enter Data. The complete update procedure is: replace the source files in the Data folder with updated versions using the same filenames, then select Refresh All. The M pipelines process the updated files via the `DataFolderPath` parameter, the Calendar extends automatically via the `MAXX` expression, and all `RowsNotEmpty` columns recalculate.


---



## DAX Measures

All measures are stored in a dedicated DAX Measures table created via Enter Data and are explicitly defined, with no Quick Measures were used. The full set is documented in `DAXMEASURES.xlsx`.

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


---

<img width="1459" height="830" alt="Screenshot_20260523_105715" src="https://github.com/user-attachments/assets/295d0fb6-7893-4fc1-b904-0b0d97d06597" />

---

## Power Query (M)

A query parameter named `DataFolderPath` was created before any data was loaded. It is referenced in the Source step of every query so that a change in storage location requires only a single parameter update rather than modifying every individual query. All applied steps are renamed with descriptive labels to support replicability and future updates.

All ten World Bank datasets arrived in wide format with year values spread across columns. A consistent transformation sequence was applied across all of them: the Series Code column was removed as it carried no analytical value; Country Name, Country Code, and Series Name were retained; remaining year columns were unpivoted using Unpivot Other Columns into unified Attribute and Value columns; the Attribute column was cleaned to isolate the four-character year label, renamed to Year, and set to Whole Number; the Value column was converted to Decimal Number with missing value markers replaced by null; and a Date column was constructed using `DATE(Year, 1, 1)` with a null fallback.

The four external datasets required adapted approaches. The ITU datasets were delivered natively in long format and required only column selection, renaming, and Date column addition. The Eurostat dataset was loaded via the Excel connector and required removal of metadata header and footer rows and Eurostat flag columns before unpivoting, after which a dummy Series Name column and a null Country Code column were appended to match the shared schema. The IMF GII dataset required sheet selection, removal of the Estimates Start After metadata column, and unpivoting of year columns, with a static Series Name and null Country Code added for the same reason.

The main harmonisation challenge was country identifier inconsistencies across sources. A Country Mapping Table was built from the World Bank Education Statistics query to serve as the reference standard. The IMF tables required two concordance tables: one for code-level inconsistencies covering Palestine and Kosovo, and one for name mismatches covering over thirty cases, including French Republic to France, Czech Republic to Czechia, Turkey to Türkiye, and the State of Palestine to West Bank and Gaza. The Geography dimension was built from a live Wikipedia UN Geoscheme web connection and merged through the same concordance pipeline. An inner join against the Country Mapping Table restricted the output to sovereign countries aligned with the World Bank classification, automatically excluding overseas territories and non-sovereign entities. Three helper tables (the two concordance tables and the Country Mapping Table) were disabled from loading to keep the analytical layer clean.



---

## Report Structure and Design

All twelve pages share a standardised structural template. The top strip contains the page number and topic label, three dropdown slicers for Year, Continent, and Country, and a navigation arrow. A full-width bold finding headline states the key analytical conclusion of the page, formatted with a thin coloured left-border accent rectangle shape. Two main analytical visual areas sit below the headline, each with its own sub-headline. The bottom zone includes key findings as bullet points, source attribution, and the Cardial Enterprises logo. The canvas background uses a light lavender brand colour with individual chart areas rendered on white card backgrounds with shadow formatting, creating a floating card effect.

A mobile layout was also built for every page, featuring a white background, topic label and page number at the top, the three dropdown slicers immediately below, fully scrollable interactive content, and source attribution and the company logo at the bottom.


---

## Key Findings

- Maternal mortality has been virtually eradicated in Western economies while Sub-Saharan Africa carries a disproportionate share of global deaths. African maternal deaths fell 43.4% from 30,586 in 2000 to 17,303 in 2020 — meaningful progress that still falls short of the SDG 3.1 target of below 70 per 100,000 live births.
- Women hold approximately one quarter of parliamentary seats and slightly more than one third of management roles globally. The female share of parliamentary seats more than doubled from 12% in 2000 to 26% in 2025; management share rose by eight percentage points over the same period. Developed regions tend to show more balanced representation in both spheres, while many fast-growing developing economies offer women greater opportunity in business than in politics.
- Female labour force participation remains well below male participation globally. The relationship between GDP per capita and female participation is not linear — high rates appear in both very poor and very rich regions for entirely different structural reasons — suggesting cultural and social factors are stronger determinants than income level alone. The lowest rates are concentrated in Western Asia and Northern Africa, though religion alone is not a reliable predictor given Eastern Africa's high participation rates.
- The gender pay gap within Europe varies considerably across member states. Countries widely considered progressive in social policy do not uniformly outperform the European average on this indicator.
- Girls are now more likely to complete school than at any point in history, and the Women Business and Law Index rose fifteen points since 2000. Yet legal protections encoded in constitutions do not reliably translate into better outcomes on representation, health, or economic indicators, revealing a persistent gap between formal rights and lived equality.
- The financial inclusion gap has barely moved in fifteen years and the digital gender divide has remained roughly constant even as internet penetration grew dramatically globally, compounding exclusion from economic and educational opportunity in low-income regions.
- The Gender Snapshot 2025 confirms the world is on a trajectory to miss every SDG 5 indicator by 2030. The pace of progress, while directionally correct, is not sufficient.

---



<img width="1453" height="831" alt="Screenshot_20260523_105741" src="https://github.com/user-attachments/assets/b320476f-0c16-4e1d-bab5-41d50e967256" />

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Power BI Desktop (2024) | Core BI platform — semantic model, DAX development, report building, and visualisation |
| Power Query (M) | ETL — column unpivoting, cleaning, country harmonisation, and file path parameterisation via Advanced Editor |
| DAX | All analytical measures, calculated columns, Calendar table, and `RowsNotEmpty` columns |
| Deneb + Vega-Lite | Custom dot matrix visuals on pages 1 and 4 |
| Azure Maps | Filled map visuals for geographic distribution across multiple pages |
| Microsoft Excel (2023) | Preliminary data exploration, concordance table authoring, and DAX measure reference tables |
| Microsoft Word | Written report compiled to match course formatting constraints (Calibri, 1.5 spacing, 15-page ceiling) |
| Microsoft PowerPoint | Presentation and defence deck exported to PDF |
| Wikipedia UN Geoscheme (live web connection) | Continental and subregional classifications for the Geography dimension |
| Claude Sonnet 4.6 (Anthropic) | AI coding assistance for complex DAX syntax and Vega-Lite specification development |
| Gemini (Google) | AI table writing assistance |

---

## Data Sources

Eurostat. (2025). *Gender pay gap in unadjusted form* [Dataset SDG_05_20]. Eurostat Data Browser. https://ec.europa.eu/eurostat

International Monetary Fund. (2016). *IMF gender composite indices — Gender Inequality Index* [Dataset]. https://www.imf.org

International Monetary Fund. (2025). *World Economic Outlook Database, April 2025* [Dataset]. https://www.imf.org/en/Publications/WEO

International Telecommunication Union. (2025). *ICT Data Portal* [Dataset]. https://datahub.itu.int

United Nations Statistics Division. (2025). *UN M.49 geoscheme*. https://unstats.un.org/unsd/methodology/m49/overview/

UN Women & UN DESA. (2025). *Progress on the Sustainable Development Goals: The Gender Snapshot 2025*. https://www.unwomen.org

World Bank. (2025). *Education Statistics, Gender Statistics, Global Findex Database, Health Nutrition and Population Statistics, Human Capital Index, Jobs Statistics, Population Estimates and Projections, SDG 5 Statistics, World Development Indicators* [Datasets]. World Bank DataBank. https://data.worldbank.org

---



## Report and Presentation

Full technical documentation, M query explanations, DAX measure documentation, series name reference, data model diagrams, limitations, and conclusions are in [`WB_Report.pdf`](./WB_Report.pdf).

The slide deck with one slide per analysis area, key findings, illustrative graphs, and source attribution is in [`WB_Presentation.pdf`](./WB_Presentation.pdf).

---

*Nova IMS — Universidade Nova de Lisboa | Databases II / Business Intelligence 2025/2026*
