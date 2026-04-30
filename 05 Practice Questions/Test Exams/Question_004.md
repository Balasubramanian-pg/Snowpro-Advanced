# Question 004

![Question 004](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28175%29.png)

**The correct answer is the LAST option:**

**"Intermediate results exceeded both available memory and local disk capacity, causing data to be written to remote cloud storage during execution"**

The currently selected answer (first option about display size limit) is **INCORRECT**.

**Why:**
- "Bytes spilled to remote storage" is a **query execution performance metric** that shows up in Query Profile
- It means during query processing, temporary/intermediate data (from JOINs, aggregations, sorts, etc.) was so large it couldn't fit in:
  1. Memory (RAM) → first level
  2. Local disk → second level  
  3. So it had to spill to **remote cloud storage** → slowest option

This causes **severe performance degradation** because writing/reading to remote storage over the network is much slower than using memory or local disk.

**The first option is wrong because:**
- Display size limits affect how results are shown to users in the UI
- This has nothing to do with query execution performance or "bytes spilled"
- It's about presentation, not processing

**You need to select the 4th (last) option, not the 1st.**
