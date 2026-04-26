<div align="center">

<img src="https://api.iconify.design/ph/snowflake-light.svg?color=%234A90D9&width=64&height=64" width="64" />

# SnowPro Advanced: Data Analyst
### 60-Day Study Repository

<img src="https://api.iconify.design/ph/calendar-blank-light.svg?color=%236B7280&width=16&height=16" width="16" /> **Exam Date:** June 24, 2026 &nbsp;&nbsp; <img src="https://api.iconify.design/ph/flag-light.svg?color=%236B7280&width=16&height=16" width="16" /> **Start Date:** April 25, 2026

</div>

<br>

<table width="100%">
  <tr>
    <td width="50%" style="padding:4px;">
      <img src="https://images.unsplash.com/photo-1608306448197-e83633f1261c?q=80&w=687&auto=format&fit=crop" width="100%" />
    </td>
    <td width="50%" style="padding:4px;">
      <img src="https://images.unsplash.com/photo-1511376777868-611b54f68947?q=80&w=1170&auto=format&fit=crop" width="100%" />
    </td>
  </tr>
  <tr>
    <td width="50%" style="padding:4px;">
      <img src="https://images.unsplash.com/photo-1618389041494-8fab89c3f22b?q=80&w=687&auto=format&fit=crop" width="100%" />
    </td>
    <td width="50%" style="padding:4px;">
      <img src="https://images.unsplash.com/photo-1698919585695-546e4a31fc8f?q=80&w=1331&auto=format&fit=crop" width="100%" />
    </td>
  </tr>
</table>

<br>

---

## <img src="https://api.iconify.design/ph/book-open-light.svg?color=%231F2937&width=22&height=22" width="22" /> Before You Start

This repo is structured so you never have to ask what to study next. Everything is sequenced, weighted by exam priority, and ready to work through one file at a time. Your only job is to open the next file and understand what you read.

A few principles worth anchoring to before you begin:

- **Don't try to memorize everything.** Snowflake rewards intuition over recall. Understand the *why* behind each concept and the syntax follows naturally.
- **One topic per day is enough.** Sixty days of consistent effort will outperform two weeks of panicked cramming every time.
- **SQL is your primary weapon.** Practice writing queries from scratch, not just reading them. The exam gives you scenarios, not fill-in-the-blanks.
- **Think in business problems.** Every question describes a situation a real data analyst faces. Ask yourself what outcome the business needs, then work backwards to the Snowflake feature.
- **Domains 3 and 4 are where the exam is won or lost.** Together they account for 60% of your score. Do not let the earlier domains consume disproportionate time just because they feel more familiar.

---

## <img src="https://api.iconify.design/ph/folder-simple-light.svg?color=%231F2937&width=22&height=22" width="22" /> Repository Structure

The repo is divided into six top-level directories. Work through them in order. The `00-exam-overview` folder contains the day-by-day schedule and a full breakdown of what each domain tests. Start there before touching anything else.

```
snowpro-da-prep/
│
├── 00-exam-overview/
│   ├── exam-breakdown.md          Domain weights, format, question types
│   └── 60-day-study-plan.md       Your day-by-day schedule
│
├── 01-data-ingestion-preparation/        17% of exam
│   ├── 1.1-retrieving-data.md
│   ├── 1.2-data-discovery.md
│   ├── 1.3-snowflake-marketplace.md
│   ├── 1.4-data-integrity.md
│   ├── 1.5-data-processing-pipelines.md
│   ├── 1.6-loading-data.md
│   └── 1.7-snowflake-functions.md
│
├── 02-data-transformation-modeling/      23% of exam
│   ├── 2.1-data-types-formats.md
│   ├── 2.2-data-cleaning.md
│   ├── 2.3-querying-and-working-data.md
│   ├── 2.4-data-modeling.md
│   └── 2.5-query-performance.md
│
├── 03-data-analysis/                     32% of exam
│   ├── 3.1-sql-extensibility.md
│   ├── 3.2-descriptive-analysis.md
│   ├── 3.3-diagnostic-analysis.md
│   └── 3.4-forecasting-predictive.md
│
├── 04-data-presentation-visualization/   28% of exam
│   ├── 4.1-dashboards-and-reports.md
│   ├── 4.2-maintaining-dashboards.md
│   └── 4.3-visualizations.md
│
├── 05-practice-questions/
│   ├── domain-1-questions.md
│   ├── domain-2-questions.md
│   ├── domain-3-questions.md
│   ├── domain-4-questions.md
│   └── full-mock-exam.md
│
└── 06-quick-reference/
    ├── functions-cheatsheet.md
    ├── sql-patterns-cheatsheet.md
    └── exam-day-checklist.md
```

---

## <img src="https://api.iconify.design/ph/chart-bar-light.svg?color=%231F2937&width=22&height=22" width="22" /> Domain Weights

Understanding how the exam is weighted tells you where to invest your time. Domain 3 (Data Analysis) is the single largest section at 32%. Domain 4 (Data Presentation and Visualization) follows at 28%. Prioritize accordingly.

| Domain | Topic | Weight | Priority |
|--------|-------|:------:|----------|
| 3.0 | Data Analysis | **32%** | <img src="https://api.iconify.design/ph/circle-fill.svg?color=%23EF4444&width=12" width="12" /> Highest |
| 4.0 | Data Presentation and Visualization | **28%** | <img src="https://api.iconify.design/ph/circle-fill.svg?color=%23EF4444&width=12" width="12" /> High |
| 2.0 | Data Transformation and Modeling | **23%** | <img src="https://api.iconify.design/ph/circle-fill.svg?color=%23F59E0B&width=12" width="12" /> Medium |
| 1.0 | Data Ingestion and Preparation | **17%** | <img src="https://api.iconify.design/ph/circle-fill.svg?color=%2310B981&width=12" width="12" /> Foundation |

Domain 1 is listed last here intentionally. It is foundational knowledge you likely already have some exposure to. Treat it as a warmup, not a destination. Get through it efficiently and move into the heavier domains where your score is actually determined.

---

## <img src="https://api.iconify.design/ph/check-square-light.svg?color=%231F2937&width=22&height=22" width="22" /> Progress Tracker

Mark topics as you complete them. Do not mark something done until you can explain the concept out loud without looking at your notes. That is the real test of readiness.

### Domain 1: Data Ingestion and Preparation
- [ ] 1.1 Retrieving Data
- [ ] 1.2 Data Discovery
- [ ] 1.3 Snowflake Marketplace
- [ ] 1.4 Data Integrity
- [ ] 1.5 Data Processing Pipelines
- [ ] 1.6 Loading Data
- [ ] 1.7 Snowflake Functions

### Domain 2: Data Transformation and Modeling
- [ ] 2.1 Data Types and Formats
- [ ] 2.2 Data Cleaning
- [ ] 2.3 Querying and Working with Data
- [ ] 2.4 Data Modeling
- [ ] 2.5 Query Performance

### Domain 3: Data Analysis
- [ ] 3.1 SQL Extensibility (UDFs, Stored Procs, Views)
- [ ] 3.2 Descriptive Analysis
- [ ] 3.3 Diagnostic Analysis
- [ ] 3.4 Forecasting and Predictive Analysis

### Domain 4: Data Presentation and Visualization
- [ ] 4.1 Dashboards and Reports
- [ ] 4.2 Maintaining Dashboards
- [ ] 4.3 Visualizations

### Practice
- [ ] Domain 1 Practice Questions
- [ ] Domain 2 Practice Questions
- [ ] Domain 3 Practice Questions
- [ ] Domain 4 Practice Questions
- [ ] Full Mock Exam

---

## <img src="https://api.iconify.design/ph/brain-light.svg?color=%231F2937&width=22&height=22" width="22" /> The Mindset

> *"The exam rewards people who understand how Snowflake thinks, not people who memorized syntax."*

Every time you learn something new in this repo, run it through three questions:

1. **What problem does this solve?** If you cannot answer this, you do not understand the feature yet.
2. **What would break if I did this wrong?** Understanding failure modes is what separates analysts from technicians.
3. **What scenario on the exam might test this?** Train yourself to think in business situations, not feature lists.

Sixty days is a generous runway. The people who fail this exam do so because they studied the wrong things for too long, not because they lacked the time. Use the domain weights. Trust the schedule. Stay consistent.
