# Question 003

![Question 003](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28174%29.png)

**Answer: Intermediate results exceeded both available memory and local disk capacity, causing data to be written to remote cloud storage during execution**

**Explanation:**

"Bytes spilled to remote storage" indicates a serious resource constraint during query execution. Here's the hierarchy of how queries process data:

1. **Memory (RAM)** - Fastest, first choice
2. **Local disk** - Slower, used when memory is exhausted
3. **Remote storage** - Slowest, last resort

When intermediate results (like JOIN operations, aggregations, sorts, etc.) are too large to fit in memory OR local disk, the warehouse must write temporary data to remote cloud storage. This is extremely slow because:
- Network I/O is much slower than memory or local disk
- Remote storage has higher latency
- Data must be serialized/deserialized across the network

**This is a major performance bottleneck** and explains why the query is running slowly.

**Why the other options are wrong:**
- **Option 1** - Display size limits affect result presentation, not query execution speed
- **Option 2** - Result cache is about storing completed results for reuse, not execution performance
- **Option 3** - Warehouse suspension would pause/fail the query, not cause slow execution

**Solution:** Increase warehouse size (more memory/local disk) or optimize the query to process less data.
