# Analyzing Data Job Market

This project started as part of Luke Barousse's *Excel for Data Analytics* course, but I extended it with a few additional analyses that I found more interesting than the base curriculum. The core dataset comes from [datanerd.tech](https://datanerd.tech/about) and contains real job posting data --> titles, salaries, locations, required skills, and job types.

---

## Core Analysis (Course Curriculum)

The base analysis covers four questions:

- **Skill-Pay Correlation** --> Does asking for more skills in a posting correlate with higher pay?
- **Regional Salary Variations** --> How do salaries differ across countries?
- **Top Skill Demand** --> Which skills appear most often in data job postings?
- **Financial Value of Skills** --> What's the median salary associated with the top 10 skills?

## My Extensions

- **Skill Efficiency Metric** --> A custom DAX measure (`Median Salary / Skills per Job`) that shows financial return per skill required.
- **US Salary Premium** --> A DAX formula calculating the exact percentage gap between US and non-US salaries per role.
- **Skill Demand Distribution** --> A combo chart testing how much of total market demand is covered by the top 30% of skills.

---

## Excel Skills Used

- Pivot Tables & Pivot Charts
- DAX (Data Analysis Expressions)
- Power Query
- Power Pivot

---

# Core Analysis

## 1. Skill-Pay Correlation

**Tools used:** Power Query, Power Pivot, Pivot Chart

### Data Preparation

I loaded `data_salary_all.xlsx` into Power Query and split it into two tables: one for job information and one for skills. Basic cleaning steps --> removing unused columns, fixing data types, trimming whitespace --> then loaded both into the Excel Data Model.

![1.png](Images/1.png)

![2.png](Images/2.png)

### Relationship Model

![Relationship.png](Images/Relationship.png)

---

# Key Findings

- **SQL and Python appear in ~50% of postings** → strongest market coverage.
- **More skills ≠ more pay** → specialization often beats breadth.
- **Python outperforms SQL in salary** despite similar demand.
- **US premium varies heavily by role** — not every role benefits equally.
- **Skill Value Index favors high-demand skills over niche peaks.**

---

# Analysis Overview

---

## 1. Skill–Pay Correlation

Question:

> Does requiring more skills correlate with higher salary?

Scatter plot comparing:
- Median Salary
- Average Skills per Job

### Visualization

![Analysis1.png](Images/Analysis1.png)

- More required skills generally means higher pay --> but it's not linear. **Senior Data Scientists** sit at the top (~$157K median) while requiring fewer skills per posting (~4.8) than most other senior roles, suggesting that depth of specialization matters more than breadth.
- **Senior Data Engineers** are an outlier in the other direction: the highest average skill count (~8) in the dataset, yet a median salary (~$147K) lower than Senior Data Scientists. The technical breadth comes with a pay ceiling.
- The **Data Analyst → Senior Data Analyst** path shows a clean $94K → $112K jump with a modest skill increase --> one of the more predictable progressions in the data.

---

## 2. Regional Salary Analysis

Question:

> How much does geography impact compensation?

DAX measure:

```dax
Median Salary :=
MEDIAN(data_jobs_all[salary_year_avg])

US Median Salary :=
CALCULATE(
    MEDIAN(data_jobs_all[salary_year_avg]),
    data_jobs_all[job_country] = "United States"
)
```

### Visualization

![MedianCountry.png](Images/MedianCountry.png)

- The US premium is consistent across all 10 roles, but not uniform. **ML Engineers and Cloud Engineers** see the sharpest drop outside the US --> both go from ~$145–158K (US) to $110K internationally.
- **Senior Data Engineers** are the exception: $150K in the US vs. $147.5K internationally. The gap is almost negligible, which points to genuinely global demand for that specialization.
- The relative salary bump from junior to senior analyst is actually larger outside the US (27.3% vs. 21.1%), which is an interesting counterpoint to the absolute dollar advantage of working in the US.

---

## 3. Top Skills Analysis

**Tools used:** Power Pivot, Relational Data Modeling

I connected the jobs table to the skills table via a 1:N relationship on `job_id` --> this avoids duplicating job data when analyzing skills and allows correct cross-filtering in pivot tables.

![Relationship.png](Images/Relationship.png)

### Findings

![TopSkills.png](Images/TopSkills.png)

- SQL (~51%) and Python (~50%) appear in roughly half of all postings --> everything else drops off sharply. The next skill, AWS, sits at ~19%.
- Among BI tools, Tableau (~19%) leads Power BI (~13%) in this dataset. The gap isn't huge, but it's consistent.

---

## 4. Financial Valuation of Skills

Question:

> Which skills combine demand and compensation?

### Visualization

![ComboChart.png](Images/ComboChart.png)

### Findings

- **Spark ($145K) and AWS ($140K)** lead on salary but have low market likelihood (~14% and ~19%). High pay, narrow demand --> classic niche positioning.
- Python pays more than SQL (~$129K vs. ~$121.5K) despite nearly identical market presence. SQL gets you in the door; Python tends to push you higher.
- Excel has the lowest median salary (~$94K) but strong presence (~19%) --> it's a baseline tool, not a salary driver.

---

# Additional Analysis

## 5. Skill Efficiency Index

**Tools used:** DAX, `IFERROR`, `DIVIDE`

### Metric

Standard pay correlation doesn't capture effort-to-reward. This measure calculates salary per required skill to show which roles give the most financial return per unit of learning:

```dax
Salary Efficiency :=
IFERROR(
DIVIDE([Median Salary], [Skills Per Job]),
BLANK()
)
```

### Visualization

![Efficiency.png](Images/Efficiency%20.png)

- **Senior Data Scientists, Cloud Engineers, and Business Analysts** lead at $31K–$33K per skill. They require relatively few skills and pay well --> best efficiency in the dataset.
- General analytics roles (Data Analyst, Senior Data Analyst, Data Scientist, ML Engineer) cluster tightly around $28.5K–$29.2K/skill --> salary scales almost proportionally with skill requirements across this group.
- **Senior Data Engineers** have the lowest efficiency (~$17.5K/skill). High absolute salary, but the long list of required tools spreads that salary thin.

---

## 6. US Salary Premium *(Custom Metric)*

Measures exact US advantage.

```dax
US vs Non-US Premium :=
IFERROR(
DIVIDE(
[Median Salary US]-[Median Salary Non-US],
[Median Salary Non-US]
),
BLANK()
)
```

### Visualization

![USPremium.png](Images/USPremium.png)

- **ML Engineers (43.6%)** and **Cloud Engineers (31.8%)** have by far the highest US premium --> both fields are heavily concentrated in US-based companies, which drives up local rates.
- **Senior Data Engineers** again stand out with only a 1.69% premium, reinforcing that demand for senior data infrastructure talent is genuinely global.
- There's a seniority convergence pattern: for Data Analysts, the US premium drops from 9.95% at mid-level to 4.55% at senior level. As people specialize, international employers appear willing to pay closer to US rates.

---

## 6. Skill Value Index

**Tools used:** DAX, Expected Value Modeling

Median salary alone can mislead --> a skill might pay well but appear in only 1% of postings. This metric weights salary by how often a skill actually shows up:

$$\text{Skill Value Index} = \text{Median Salary} \times \text{Skill Likelihood}$$

```dax
Skill Value Index :=
IFERROR(
[Median Salary] * [Skill Likelihood],
BLANK()
)
```

### Visualization

![Skill Value Index.png](Images/Skill%20Value%20Index.png)

- **Python ($64K) and SQL ($61K)** dominate --> neither pays the absolute most, but both appear in ~50% of postings. That combination puts them in a tier of their own.
- **AWS ($27K)** ranks third. The salary is strong but its 19% likelihood keeps its expected yield well below the top two.
- **R ($22K) vs. SAS ($12K)**: R nearly doubles SAS's index score, reflecting the broader shift away from proprietary statistical software.

---

## 7. Pareto Analysis: Skill Prioritization

**Tools used:** Statistical Distribution Modeling, Cumulative Running Totals

The goal here was to find out whether a small number of skills account for most of the market demand --> essentially testing an 80/20 rule on the skill data.

**Methodology:**
1. Normalized each skill's likelihood against the total for the top 10 skills
2. Sorted descending and built a cumulative running total
3. Flagged anything below 60% cumulative coverage as "core" (I tuned the threshold to 60% rather than 80% --> at 80% most of the top 10 qualifies, which defeats the purpose of the analysis)

```excel
=IF(Cumulative_Coverage <= 0.60; "► Top 80%"; "Tail 20%")
```

### Findings

![Pareto.png](/data-job-market-analysis/Images/Pareto.png)

- **SQL and Python together account for ~44% of total top-skill coverage.** Just two tools make up nearly half the weight.
- Adding **Excel** pushes that to ~52% --> three tools cover the majority of top-tier demand.
- Beyond those three, the curve flattens into a long tail of cloud, engineering, and BI tools (AWS, Tableau, Azure, Spark, Power BI, R, SAS) each adding 5–8% individually. These are worth learning, but only after the core three are solid.
