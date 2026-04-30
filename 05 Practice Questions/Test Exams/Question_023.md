# Question 023

![Question 023](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28194%29.png)

**The selected answer is WRONG. ✅ Correct answer: SQL, Python, and Markdown**

**Why:** Snowflake Notebooks natively supports exactly three cell types:

- **SQL** — run Snowflake queries directly
- **Python** — execute Python code (with access to Snowpark, pandas, etc.)
- **Markdown** — write formatted documentation, headers, descriptions inline

This is the same paradigm as Jupyter Notebooks — code + documentation in one interface.

**Why the selected answer (SQL, Python, and Java) is wrong:**

Java is **not** a supported cell type in Snowflake Notebooks. While Snowflake supports Java for Stored Procedures and UDFs written externally, it is not an interactive notebook cell type. Java is not an interpreted/REPL-friendly language suited for notebook execution.

**Why the other two are wrong:**

- **SQL, JavaScript, and Markdown** — JavaScript is not a notebook cell type. It's supported in Snowflake for UDFs and Stored Procedures, but not as a notebook cell.
- **SQL, Python, R, and Markdown** — R is not natively supported in Snowflake Notebooks. R has no native Snowflake runtime support at all.

**Memory anchor:**

> Snowflake Notebooks = **SQL + Python + Markdown**. Think of it as a Jupyter Notebook that lives inside Snowflake. Jupyter supports Python + Markdown — Snowflake just adds SQL on top.
