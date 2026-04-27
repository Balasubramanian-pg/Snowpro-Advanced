# 60-Day Study Plan: SnowPro Core Data Analyst Certification

## Before You Start: Reality Check

This plan assumes you can commit 1.5–2 hours on weekdays and 3–4 hours on weekends. If you can't protect that time consistently, postpone the exam. Cramming doesn't work for applied exams like this one—you either understand the patterns or you don't.

Also: if you're not executing queries in a Snowflake account while studying, you're wasting time. Theory without practice is decoration. Get a trial account or use your work environment. Break things. Fix them. That's how you learn.

---

## Exam Structure Reference (Know This Cold)

| Domain | Weight | What It Actually Tests |
|--------|--------|----------------------|
| Data Ingestion & Preparation | ~20% | Can you stage, validate, and load data without breaking governance or performance? |
| Data Transformation & Modeling | ~30% | Can you write correct, efficient SQL that produces the right grain and handles edge cases? |
| Data Analysis & Visualization | ~25% | Can you derive insights, use window functions correctly, and build dashboards that don't lie? |
| Security, Governance & Performance | ~15% | Do you understand RBAC, policies, pruning, and caching well enough to avoid costly mistakes? |
| Snowflake Architecture & Services | ~10% | Do you know how the pieces fit together well enough to troubleshoot when things go wrong? |

The exam is scenario-based. You won't be asked "what does COPY INTO do?" You'll be given a broken pipeline and asked which change fixes it. Study accordingly.

---

## Phase 1: Foundation (Days 1–15)
*Goal: Build mental models, not memorize syntax*

### Week 1 (Days 1–7): Snowflake Architecture & SQL Refresher
**Daily commitment**: 1.5 hours weekdays, 3 hours weekend

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 1 | Account structure, virtual warehouses, storage/compute decoupling | Diagram the architecture from memory; explain to someone (or rubber duck) why this matters for cost and performance | Can you explain why resizing a warehouse doesn't move data? |
| 2 | RBAC fundamentals: roles, privileges, hierarchy | Create 3 roles in a dev account; grant/revoke privileges; test access as different roles | Can you predict what a user can see after a specific grant chain? |
| 3 | Basic SELECT, filtering, NULL handling | Write 10 queries that intentionally misuse `= NULL`, `AND`/`OR` precedence, case sensitivity; observe results | Can you rewrite a broken filter without running it first? |
| 4 | JOINs: INNER, LEFT, RIGHT, FULL | Build a small dataset with intentional mismatches; test each JOIN type; document when rows disappear | Can you explain why a LEFT JOIN + WHERE on right table becomes INNER? |
| 5 | Aggregation: GROUP BY, COUNT variants, HAVING | Write queries that mix COUNT(*), COUNT(col), COUNT(DISTINCT); break them with NULLs; fix them | Can you predict aggregation results before executing? |
| 6 | Date/time functions, string manipulation | Convert timezones, truncate dates, parse strings; test edge cases (leap years, DST) | Can you write a query that groups by week starting Monday, not Sunday? |
| 7 | Review + mini-assessment | Re-run all Week 1 queries from memory; explain each choice aloud | If you hesitate on >20%, repeat the weak areas before moving on |

### Week 2 (Days 8–15): Data Loading & Semi-Structured Data
**Daily commitment**: 1.5 hours weekdays, 4 hours weekend

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 8 | Internal vs external stages, PUT/GET, COPY INTO <table> | Load a CSV into a stage; copy to table with different ON_ERROR settings; observe load history | Can you recover from a failed load without reloading everything? |
| 9 | File formats: CSV, JSON, Parquet; NULL_IF, SKIP_HEADER, etc. | Load the same data in 3 formats; compare parsing behavior, performance, storage | Can you choose the right format for a given downstream use case? |
| 10 | VARIANT basics: path extraction, casting, FLATTEN | Parse nested JSON; extract arrays; handle missing keys safely with TRY_CAST | Can you write a query that won't fail if a JSON key is missing? |
| 11 | INFER_SCHEMA vs explicit schema; validation patterns | Use INFER_SCHEMA on a messy file; compare to hand-defined schema; document trade-offs | Can you explain when to use each approach in production? |
| 12 | COPY INTO <location> for export; compression, partitioning | Export a table to external stage in Parquet + SNAPPY; verify file structure downstream | Can you design an export that a Spark job can consume without transformation? |
| 13 | Error handling: load history, validation queries, quarantine patterns | Intentionally break a load; trace errors through SYSTEM$LOAD_HISTORY; build a simple quarantine table | Can you isolate bad rows without stopping the entire pipeline? |
| 14–15 | Phase 1 review + practice questions | Rebuild Week 1–2 queries from scratch; answer 20 scenario-based questions (create your own if needed) | Can you explain *why* each answer is correct, not just which one? |

**Phase 1 checkpoint**: If you can't load a CSV, parse JSON, and explain RBAC without referencing docs, don't proceed. Fix gaps now.

---

## Phase 2: Core Competencies (Days 16–35)
*Goal: Master the patterns that appear in 80% of exam questions*

### Week 3 (Days 16–22): Transformation Fundamentals
**Daily commitment**: 2 hours weekdays, 4 hours weekend

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 16 | CTEs vs subqueries: readability, reuse, performance | Rewrite the same logic 3 ways; compare explain plans; document when each is appropriate | Can you justify your structural choice for a given scenario? |
| 17 | CASE, COALESCE, NULLIF, IFF: conditional logic patterns | Build a query that handles 5 different NULL/edge cases; test with intentionally messy data | Can you predict output for a row with all NULL inputs? |
| 18 | Type conversion: CAST vs TRY_CAST, implicit vs explicit | Break queries with type mismatches; fix with explicit casting; document when implicit works | Can you spot a silent coercion bug before it causes wrong results? |
| 19 | Deduplication patterns: ROW_NUMBER + QUALIFY, DISTINCT, GROUP BY | Create duplicate data; apply 3 dedup strategies; compare results, performance, idempotency | Can you choose the right pattern for "keep latest" vs "keep any"? |
| 20 | Handling NULLs in aggregations, joins, business logic | Write queries that produce different results based on NULL handling; document the business impact | Can you explain to a stakeholder why their metric changed after a NULL fix? |
| 21 | Window functions intro: PARTITION BY, ORDER BY, default frame | Compute running totals, rank within groups, lag/lead; break them with missing ORDER BY | Can you explain why a window function without ORDER BY is often useless? |
| 22 | Review + scenario practice | Solve 5 transformation scenarios end-to-end; time yourself; compare to reference solutions | Are you optimizing for correctness first, then performance? |

### Week 4 (Days 23–29): Advanced SQL & Data Modeling
**Daily commitment**: 2 hours weekdays, 4 hours weekend

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 23 | Window frames: ROWS vs RANGE vs GROUPS; explicit frame clauses | Compute moving averages with different frames; observe how ties affect RANGE | Can you choose the right frame for a business question? |
| 24 | QUALIFY deep dive: filtering after window calc, common mistakes | Rewrite subquery-based filters using QUALIFY; break them with alias references | Can you explain why QUALIFY can't reference SELECT aliases in all contexts? |
| 25 | Star schema basics: fact/dim design, grain definition, surrogate keys | Model a simple sales domain; define grain; write queries that respect the grain | Can you explain why querying at the wrong grain produces wrong totals? |
| 26 | One Big Table vs normalized: trade-offs, when to use each | Build the same report two ways; compare query complexity, performance, maintenance | Can you justify your modeling choice for a given use case? |
| 27 | Clustering keys: when to use, how to evaluate, pruning mechanics | Cluster a large table; test pruning with SYSTEM$CLUSTERING_INFORMATION; measure impact | Can you predict whether a filter will prune before running the query? |
| 28 | Materialization: views, materialized views, dynamic tables, transient tables | Create each type; test refresh behavior, cost, query compatibility | Can you choose the right materialization for a dashboard vs batch pipeline? |
| 29 | Phase 2 review + integration exercise | Build a mini-pipeline: load → transform → model → query; document decisions at each step | Does your pipeline handle errors, NULLs, and performance without hand-holding? |

### Week 5 (Days 30–35): Analysis Patterns & Dashboard Prep
**Daily commitment**: 2 hours weekdays, 4 hours weekend

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 30 | Descriptive stats: AVG, STDDEV, PERCENTILE, CORR; NULL handling | Compute metrics on messy data; compare results with/without NULL filtering | Can you explain why your average changed after handling NULLs? |
| 31 | Time-series analysis: DATE_TRUNC, LAG/LEAD, running calculations | Build a cohort analysis; compute period-over-period growth; handle missing periods | Can you make a trend chart that doesn't lie about missing data? |
| 32 | Dashboard query patterns: parameterization, $FILTER substitution, determinism | Write a query that works with dashboard filters; test with different selections | Can you explain why non-deterministic queries break scheduled refresh? |
| 33 | Result caching: when it works, how to validate, when to bypass | Run identical queries; check cache hits with SYSTEM$RESULT_CACHE_INFO; force re-execution | Can you predict whether a query will hit cache before running it? |
| 34 | Row Access Policies & Dynamic Data Masking: evaluation order, testing | Create policies; test as different roles; verify dashboard behavior matches expectations | Can you explain why a dashboard shows different data to different users? |
| 35 | Phase 2 checkpoint assessment | Take a 50-question practice exam (create from docs or use official prep); score honestly | If <70%, spend Days 36–37 reviewing weak domains before proceeding |

**Phase 2 checkpoint**: You should now be able to look at a business question and sketch the SQL pattern before opening Snowsight. If you're still translating word-for-word, slow down and practice pattern recognition.

---

## Phase 3: Advanced Patterns & Integration (Days 36–50)
*Goal: Connect the dots; handle edge cases; optimize for real-world constraints*

### Week 6 (Days 36–42): Performance, Governance, Troubleshooting
**Daily commitment**: 2 hours weekdays, 4 hours weekend

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 36 | Query profiling: EXPLAIN, Query Profile, ACCOUNT_USAGE | Run a slow query; identify bottleneck; apply one optimization; measure improvement | Can you spot a full scan vs pruned scan in a profile without running it? |
| 37 | Pruning deep dive: sargable predicates, clustering alignment, common anti-patterns | Rewrite 5 non-sargable filters to be pruning-eligible; validate with SYSTEM$CLUSTERING_INFORMATION | Can you convert a function-wrapped filter to a range predicate on the fly? |
| 38 | Warehouse sizing, multi-cluster, auto-suspend: cost vs performance trade-offs | Test the same query on different warehouse sizes; measure time vs credits; find the knee of the curve | Can you recommend a warehouse size for a given SLA and budget? |
| 39 | Security review: RBAC audit, policy testing, sharing boundaries | Audit a role's effective privileges; test a shared dashboard as recipient; document gaps | Can you predict what a user sees before granting access? |
| 40 | Data quality patterns: validation queries, quarantine, reconciliation | Build a validation framework for a table; test with injected bad data; verify quarantine works | Can you detect a schema drift before it breaks downstream? |
| 41 | Troubleshooting methodology: systematic diagnosis, not guesswork | Break a working pipeline in 3 ways; practice diagnosing from symptoms to root cause | Can you isolate whether a problem is data, query, or infrastructure without changing everything? |
| 42 | Week 6 review + scenario drills | Solve 3 complex scenarios end-to-end; time yourself; compare approach to reference | Are you asking the right diagnostic questions first, or jumping to fixes? |

### Week 7 (Days 43–49): Visualization, Notebooks, Real-World Workflows
**Daily commitment**: 2 hours weekdays, 4 hours weekend

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 43 | Snowsight dashboards: tile configuration, filter binding, refresh modes | Build a dashboard with 3 tiles, 2 filters, one scheduled refresh; test as different roles | Can you explain why a filter doesn't affect a tile without reconfiguring it? |
| 44 | Chart selection: bar vs scatter vs heat grid vs scorecard; when each misleads | Create the same data as 4 chart types; document which questions each answers well/poorly | Can you choose a chart that doesn't hide the insight? |
| 45 | Notebooks vs Worksheets: when to use each, result caching, promotion patterns | Prototype an analysis in both; promote a validated query to a dashboard tile; document the workflow | Can you explain why you wouldn't build a production dashboard directly in a Notebook? |
| 46 | Naming conventions, query tagging, documentation: governance in practice | Apply a naming standard to a messy query; add QUERY_TAG; validate with catalog tools | Can you make a query self-documenting without comments? |
| 47 | End-to-end workflow: from raw file to dashboard | Ingest a CSV → validate → transform → model → visualize → share; document each decision | Does your workflow handle errors, NULLs, performance, and governance without manual intervention? |
| 48 | Exam-style scenario practice: 10 complex questions | Answer questions that combine multiple domains; explain reasoning aloud; time yourself | Are you recognizing patterns, or re-deriving from first principles every time? |
| 49 | Phase 3 checkpoint: full practice exam | Take a timed, 100-question practice exam; score; analyze weak areas | If <80%, spend Days 50–52 targeting gaps before final review |

**Phase 3 checkpoint**: You should now be able to look at a broken dashboard and diagnose whether the issue is query logic, filter binding, policy evaluation, or caching—without guessing. If you're still troubleshooting by changing one thing at a time randomly, practice systematic diagnosis.

---

## Phase 4: Final Review & Exam Readiness (Days 50–60)
*Goal: Close gaps, build confidence, execute on exam day*

### Week 8 (Days 50–56): Targeted Review & Weakness Closure
**Daily commitment**: 2 hours weekdays, 4 hours weekend

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 50 | Analyze practice exam results; identify top 3 weak domains | List specific concepts you missed; rank by exam weight and your confidence | Do you know *why* you got each question wrong, not just which answer was right? |
| 51–53 | Deep dive on weak domains | Re-study, re-practice, re-explain the top 3 weak areas; create your own practice questions | Can you teach each concept to someone else without notes? |
| 54 | Memorize defaults & traps table (from cheatsheet) | Write the defaults table from memory; verify; repeat until perfect | Can you recall 90% of defaults without looking? |
| 55 | Speed drills: rewrite patterns under time pressure | Take 10 common patterns; rewrite each from memory in <2 minutes; check accuracy | Are you sacrificing correctness for speed? Slow down if so. |
| 56 | Full mock exam under timed conditions | Simulate exam environment: no docs, no breaks, strict timing; score honestly | Does your score reflect readiness, or did you get lucky? |

### Week 9 (Days 57–60): Final Prep & Execution
**Daily commitment**: 1 hour weekdays, exam day as scheduled

| Day | Focus | Action Items | Success Check |
|-----|-------|-------------|---------------|
| 57 | Light review: skim cheatsheet, re-run 5 key queries from memory | No new learning; reinforce what you know | Do you feel confident, or are you still discovering gaps? If gaps, postpone exam. |
| 58 | Logistics: confirm exam time, tech check, ID ready, environment quiet | Test webcam, internet, Snowflake trial access (if needed for reference during prep) | Is everything ready so exam day is about performance, not logistics? |
| 59 | Mental prep: sleep, hydrate, avoid cramming | No SQL after 6 PM; relax; trust your preparation | Are you rested, or trying to squeeze in "one more thing"? Stop. |
| 60 | Exam day: execute | Read questions carefully; flag uncertain ones; manage time; trust your patterns | Did you apply your methodology, or panic and guess? |

---

## What to Skip (Low-Yield Topics)

Don't waste time memorizing:
- Exact syntax of every COPY option (know the common ones; you can look up the rest)
- Every Snowflake function parameter (understand the pattern; docs are open-book in real work)
- Historical feature changes (exam tests current behavior)
- Edge-case error codes (know the categories; not the numeric values)

Do focus on:
- Patterns that appear repeatedly: pruning, caching, NULL handling, window functions
- Defaults that change behavior: ON_ERROR, RESULT_CACHE_ACTIVE, CURRENT_ROLE()
- Evaluation order: WHERE vs HAVING, QUALIFY timing, policy application
- Trade-offs: clustering cost vs benefit, materialization choices, warehouse sizing

---

## How to Practice Effectively (Not Just "Study")

1. **Write, don't just read**: For every concept, write a query that demonstrates it. Break it. Fix it.
2. **Explain aloud**: If you can't explain why a pattern works to a rubber duck, you don't understand it yet.
3. **Time yourself**: Exam is timed. Practice answering questions in <90 seconds each.
4. **Create your own scenarios**: Take a working query; break it in 3 ways; practice diagnosing.
5. **Review mistakes deeply**: Don't just note the right answer. Understand why your reasoning was wrong.

---

## Common Pitfalls & How to Avoid Them

| Pitfall | Why It Happens | How to Avoid |
|---------|---------------|--------------|
| Cramming the week before | Underestimating applied nature of exam | Start early; use this plan; trust the process |
| Passive reading without execution | Feels productive; isn't | Every study session includes writing/running SQL |
| Ignoring weak areas | Uncomfortable; easier to review strengths | Schedule targeted review; use practice exams to expose gaps |
| Memorizing syntax over patterns | Easier to recall; harder to apply | Focus on "when/why" not just "what"; practice scenario questions |
| Skipping governance/performance | Seems less "analytical"; lower weight | These are tie-breakers; know them well enough to avoid costly mistakes |
| Not simulating exam conditions | Real exam feels different; panic sets in | Take at least one full timed mock under exam-like constraints |

---

## Exam Day Strategy

1. **Read the question twice**: Identify what's actually being asked before looking at answers.
2. **Eliminate obviously wrong answers first**: Increases odds even if you're uncertain.
3. **Flag and move on**: Don't spend 5 minutes on one question; come back if time allows.
4. **Trust your patterns**: If you've practiced systematically, your first instinct is usually right.
5. **Manage time**: ~1 minute per question; leave 10 minutes at end to review flagged items.

---

## After the Exam

- If you pass: Celebrate, then immediately apply one thing you learned to a real project. Certification is a tool, not an endpoint.
- If you don't: Review your score report; identify domains; adjust this plan for a retake. Most people pass on the second attempt with targeted review.

---

## Final Thought

This plan is aggressive but realistic—if you execute. The SnowPro Core Data Analyst exam isn't about knowing everything; it's about applying fundamentals correctly under time pressure. If you can look at a scenario and recognize the pattern, you'll pass.

Stop planning. Start doing. Day 1 is today.
