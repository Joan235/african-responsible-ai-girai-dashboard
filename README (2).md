# African Responsible AI Challenge: GIRAI Power BI Dashboard

An interactive Power BI analysis of responsible AI readiness across African countries, built with the Global Index on Responsible AI (GIRAI) 2026 dataset for the DSN 2026 AI Bootcamp in-house hackathon.

## Table of Contents
1. [Overview](#overview)
2. [Summary](#summary)
3. [Introduction](#introduction)
4. [Problem Statement](#problem-statement)
5. [Data Overview](#data-overview)
6. [Data Cleaning Process](#data-cleaning-process)
7. [Data Analysis](#data-analysis)
8. [Insights](#insights)
9. [Dashboard](#dashboard)
10. [Recommendations](#recommendations)
11. [Conclusion](#conclusion)

---

## Overview

| Item | Detail |
|---|---|
| Event | DSN 2026 AI Bootcamp: Data Analytics In-House Hackathon |
| Track | Data Analysis |
| Dataset | Global Index on Responsible AI (GIRAI) 2026 |
| Scope | 135 countries worldwide, 39 African countries analysed |
| Tool | Power BI (Power Query, data modelling, DAX) |
| Deliverables | Interactive dashboard (PDF export), summary document, presentation |

## Summary

- The African average GIRAI score is **21.8**, against **35.0** globally.
- Scores range from **4.0** (South Sudan) to **45.9** (Nigeria). Nigeria, Egypt and Kenya lead.
- The biggest gap is AI-specific governance: **AI Policy 13.7** and **Civil Society Engagement 9.3**, against **Enabling Conditions 42.5**.
- **Inclusion and Diversity** is the weakest of the five dimensions (16.7).
- **Central Africa** averages 9.9, far behind every other subregion.
- Recommendations focus on AI policy adoption, algorithm transparency, civil society participation, inclusion, and regional collaboration.

## Introduction

AI adoption is accelerating across Africa, but countries differ widely in readiness, governance capacity, inclusion, skills and safeguards.

The GIRAI is a human-rights-based global benchmark of responsible AI, run by the Global Center on AI Governance. Each country receives a score from 0 to 100, built from three pillars and five dimensions:

- **Five dimensions:** Inclusion and Diversity, Ethics and Sustainability, Labour and Skills, Trust and Safety, AI Use in Public Service.
- **Three pillars:** AI Policy (60% weight), Civil Society Engagement (10%), Enabling Conditions (30%).
- **Penalty:** governments with documented unacceptable-risk AI deployments have their final score reduced by a multiplier (`urai_penalty`).

This project uses that data to show where African countries stand and where intervention would help most.

## Problem Statement

The task was to act as a data analyst supporting organisations that promote responsible and inclusive AI development across Africa, and to analyse the GIRAI data to answer five questions:

1. Which African countries show the strongest and weakest responsible AI readiness?
2. Which dimensions present the biggest gaps across Africa?
3. What factors appear to influence responsible AI performance?
4. How do countries compare across inclusion, ethics, skills, trust and public-sector AI?
5. What practical interventions could improve responsible AI readiness?

The output had to be an interactive Power BI dashboard with KPIs, analytical visuals, a map, and filters, plus a short summary of insights and recommendations.

## Data Overview

The workbook `GIRAI_rankings_and_scores.xlsx` has four sheets, each with one row per country. A second file, `GIRAI_rankings_and_scores_dictionary.xlsx`, defines every column.

### `ranking_and_scores`
Overall results per country.

| Column | Meaning |
|---|---|
| `ranking` | Global rank by final score |
| `iso3` | Three-letter country code (the join key) |
| `country`, `region` | Country name and region (Africa, Europe, and so on) |
| `girai` | Final GIRAI score (0 to 100) after the penalty |
| `girai_raw` | Score before the penalty |
| `urai_penalty` | Multiplier for unacceptable-risk AI use: 1, 0.96, 0.93, 0.91 or 0.90 |
| Five dimension scores | Inclusion and Diversity, Ethics and Sustainability, Labour and Skills, Trust and Safety, AI Use in Public Service |
| Three pillar scores | AI Policy, Civil Society Engagement, Enabling Conditions |

### `pillar_scores_by_dimension`
Fifteen columns: each of the three pillars scored inside each of the five dimensions.

### `all_indicators`
The 38 individual indicators (0 to 100) that roll up into the pillars. Examples:
- **AI Policy:** Gender Equality, Children's Rights, AI Literacy, Labour Protections, Public Procurement, Environmental Impact, Public Disclosure of Government Algorithmic Systems.
- **Enabling Conditions:** Rule of Law, Cybersecurity, Skills and Literacy, Egalitarian Democracy, Data Protection and Privacy, Device affordability.
- **Civil Society Engagement:** civil society engagement indicators, for example in Trust and Safety.

### `country_classifications`

| Column | Meaning |
|---|---|
| `country`, `ISO3` | Country name and code |
| `region`, `subregion` | Geographic grouping (Africa has five subregions) |
| `developing_status` | Developing or developed |
| `WB_income_group` | World Bank income group |
| `GDP_per_capita_PPP` | GDP per capita at purchasing power parity |

## Data Cleaning Process

1. **Fixed the two-row header** in `ranking_and_scores`. Row 1 held group labels ("Dimension scores", "Pillar scores"). I promoted the correct row to headers and renamed the dimension and pillar columns.
2. **Removed blank rows.** `all_indicators` loaded as 999 rows because of empty rows. I used Remove Blank Rows and filtered null `iso3`, bringing each table to 135 rows.
3. **Checked for nulls.** The only nulls were 20 `subregion` values for non-African countries (Middle East, Caribbean, Northern America), so they were left alone.
4. **Treated zeros as real scores.** About a quarter of indicator values are 0. These are genuine scores, not missing data.
5. **Validated keys.** There were no duplicate countries, and every `iso3` matched across sheets.
6. **Set data types.** Scores as decimal numbers, ranks as whole numbers, names and codes as text.
7. **Unpivoted the dimension and pillar columns** into two long tables (`Dimension` and `Score`, `Pillar` and `Score`) so a single chart can compare all dimensions or pillars.
8. **Kept all 135 countries** in the model and filtered to Africa in the report, so Africa can be compared with the global average.

### Data model
Automatic relationship detection was switched off because the unpivoted tables repeat each `iso3`, which caused load errors. Relationships were built manually on the country code, with `country_classifications` as the central table:

```
ranking_and_scores        (1:1)  ──┐
pillar_scores_by_dimension (1:1) ──┼── country_classifications
all_indicators            (1:1)  ──┤
dimension / pillar tables (1:*)  ──┘
```

A small disconnected `Benchmark` table (Africa, Global) supports the Africa against global chart.

## Data Analysis

### Part 1: KPIs and DAX measures

| KPI | What it shows | Value |
|---|---|---|
| Africa Avg GIRAI | Fixed average of all 39 African countries | 21.79 |
| Selected Avg GIRAI | Average for whatever is selected (a country, subregion or income group) | Changes with slicers |
| Top Country | Highest-scoring country in the current selection | Nigeria |
| Lowest Country | Lowest-scoring country in the current selection | South Sudan |
| Gap vs Africa Avg | Selected score minus the African average (positive means above average) | 0.00 with no selection |
| Dimension scores | Five averages, one per dimension | 16.7, 23.4, 23.3, 26.4, 19.7 |
| Global Avg GIRAI | Benchmark across all 135 countries | 35.02 |

**Key DAX measures**

```DAX
Selected Avg GIRAI = AVERAGE(ranking_and_scores[girai])

Africa Avg GIRAI =
CALCULATE(
    AVERAGE(ranking_and_scores[girai]),
    ALL(country_classifications),
    country_classifications[region] = "Africa"
)

Global Avg GIRAI =
CALCULATE(AVERAGE(ranking_and_scores[girai]), ALL(country_classifications))

Gap vs Africa Avg = [Selected Avg GIRAI] - [Africa Avg GIRAI]

Top Country =
MAXX(
    TOPN(1, VALUES(country_classifications[country]), [Selected Avg GIRAI], DESC),
    country_classifications[country]
)

Benchmark Score =
SWITCH(
    SELECTEDVALUE(Benchmark[Benchmark]),
    "Africa", [Africa Avg GIRAI],
    "Global", [Global Avg GIRAI]
)
```

**Why these measures work this way**
- `CALCULATE` changes the filters before calculating, which lets a measure ignore the user's slicers.
- `ALL(country_classifications)` clears every slicer filter first. Without it, picking "West Africa" would quietly turn the "Africa average" into the West Africa average.
- `Selected Avg GIRAI` responds to slicers, while `Africa Avg` and `Global Avg` stay fixed. Together they power the gap measure.
- `Lowest Country` uses the same pattern as `Top Country` with `ASC` instead of `DESC`.
- Each dimension score (`Inclusion Score`, `Ethics Score`, and so on) is an `AVERAGE` of its column, and `Strongest Dimension` and `Weakest Dimension` apply the `TOPN` and `MAXX` pattern across the five dimensions for each country.

### Part 2: Analytical views
- **Country ranking and map:** all 39 countries ranked by final score, coloured above or below the African average.
- **Regional comparison:** average score by subregion, and Africa against the global average.
- **Dimension and pillar comparison:** which of the five dimensions and three pillars are weakest, and how subregions differ.
- **Strengths and gaps matrix:** every country scored on each of the five dimensions.
- **Governance and enabling conditions:** AI Policy against Enabling Conditions per country, key enabling indicators, the weakest policy indicators, and score by income group.

## Insights

### General insights
1. **Africa trails the world.** 21.8 against 35.0 globally. Only 10 of 39 countries score 30 or more, 5 score below 10, and 18 are above the African average.
2. **Leaders and laggards.** Nigeria (45.9), Egypt (41.3) and Kenya (39.5) lead. South Sudan (4.0), Chad (6.5) and Burundi (8.7) trail.
3. **Every dimension is below its global average.** Inclusion and Diversity is weakest (16.7 against 34.4 globally), then AI Use in Public Service (19.7 against 31.4). Trust and Safety is strongest (26.4).
4. **Governance, not foundations, is the gap.** AI Policy averages 13.7 and Civil Society Engagement 9.3, against 42.5 for Enabling Conditions.

### Deeper insights
1. **Policy drives rankings, partly by design.** AI Policy correlates about 0.95 with the final score, but it carries 60% of the weighting, so this confirms where the score comes from.
2. **Foundations help but do not guarantee policy.** Enabling Conditions correlate about 0.61 with the final score. The five top-scoring countries average 48.2 on Enabling Conditions, against 22.9 for the bottom five. However, the link between Enabling Conditions and AI Policy is only about 0.34, so countries such as South Africa and Mauritius have strong foundations but low policy scores.
3. **Large regional gaps.** North Africa 27.1, West Africa 24.8, Southern Africa 23.7, East Africa 21.0, and Central Africa 9.9. Central Africa scores lowest on every dimension.
4. **Income only partly explains performance.** Low-income countries average 15.5 and lower-middle-income countries 27.4. GDP per capita correlates only about 0.35 with the score. The upper-middle-income group (24.2) has only five countries.
5. **Specific indicator gaps.** The weakest indicators across Africa are government algorithm disclosure (0.0), Labour Protections (2.8), civil society in Trust and Safety (5.2), Public Procurement (6.5), AI-facilitated misinformation (6.6) and Environmental Impact (7.1). Cybersecurity (64.8) is a relative strength.
6. **Penalty has a small effect in Africa.** Only Kenya, Ghana and Uganda are penalised, each by 4% (multiplier 0.96).

## Dashboard

**Overview:** KPIs, map, country ranking, regional comparison, Africa against global.

![Overview page](<img width="1022" height="596" alt="Screenshot_20261117_235128" src="https://github.com/user-attachments/assets/b18142df-aeb0-470a-ae76-97e1b34a64b3" />)

**Dimension:** dimension scores, dimension and pillar comparison, dimensions by subregion, strengths and gaps matrix.

![Dimension page](<img width="1022" height="586" alt="Screenshot_20261117_235319" src="https://github.com/user-attachments/assets/4a8d07f9-3c08-41da-90ac-6780c3f4a33b" />
)

**Governance:** policy against enabling conditions, enabling indicators, weakest policy indicators, score by income group.

![Governance page](<img width="1079" height="617" alt="Screenshot_20261117_235450" src="https://github.com/user-attachments/assets/b18a2e37-8b00-425f-af98-12fb72e95687" />)

Filters: subregion, World Bank income group and country, with cross-filtering between visuals.

## Recommendations

1. **Prioritise AI policy where it is weakest.** Target the 17 countries scoring below 5 on AI Policy, starting with national frameworks and then enforcement.
2. **Introduce government algorithm transparency and responsible procurement rules.** Disclosure scores 0.0 and procurement 6.5, so these are clear and low-cost starting points.
3. **Institutionalise civil society participation.** Create formal roles for civil society and researchers in AI policy, especially on trust and safety.
4. **Invest in inclusion and workforce protection.** Pair AI policy with gender, child-rights and worker safeguards, and build public-sector AI skills.
5. **Use regional collaboration to lift lagging subregions.** Central Africa needs targeted support, and leaders such as Nigeria, Egypt and Kenya can share frameworks through regional bodies.

## Conclusion

Africa's responsible AI readiness (21.8) sits well below the global average (35.0), with wide gaps between countries and subregions. The weakness is concentrated in AI-specific governance and civil society engagement, not in underlying digital and institutional foundations, and Inclusion and Diversity is the weakest dimension. Wealth alone does not explain performance, which means targeted policy, transparency and inclusion measures can lift scores regardless of income level.

**Limitations:** GIRAI scores reflect documented evidence, so a low score can reflect weak policy or limited documented evidence. The strong policy-to-score relationship is partly built into the index weighting, some groups (such as upper-middle-income) are small, and the analysis shows association, not cause.


## Tools and Files
- Power BI Desktop (Power Query, data model, DAX)
- Data: GIRAI 2026 rankings and scores workbook and data dictionary
- Source: [Global Index on Responsible AI](https://www.global-index.ai)

## Author
Joan Fafure, DSN 2026 AI Bootcamp
