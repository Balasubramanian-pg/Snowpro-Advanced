# Question 001

![Question 001](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28172%29.png)

**Answer: Set auto-suspend to a low value such as 5-10 minutes to minimize idle credit consumption**

**Reasoning:**

For ad-hoc queries at unpredictable intervals, you need to balance two competing priorities:
- **Cost**: Don't pay for idle time
- **Availability**: Be ready when analysts need to run queries

Here's why the other options don't work as well:

1. **Disable auto-suspend** - This wastes money. You're paying 24/7 for a warehouse that only gets used intermittently during the day.

2. **Match longest query duration** - This doesn't make sense. Auto-suspend is about *idle time between queries*, not how long queries take to run.

3. **30 minutes** - Too long. You'd pay for 30 minutes of idle time after every query batch. If analysts run queries at 9 AM and don't come back until 2 PM, you just wasted 30 minutes of credits.

**Why 5-10 minutes is the sweet spot:**
- Modern data warehouses (Snowflake, BigQuery, etc.) resume in 3-5 seconds
- Analysts running ad-hoc queries typically work in bursts - they'll run several queries within a few minutes, then step away
- A 5-10 minute window captures natural work patterns while avoiding long idle periods
- The brief resume time is an acceptable trade-off for significant cost savings

The "unpredictable" timing actually supports a shorter suspend time - if you can't predict when queries come, you shouldn't assume they'll come within 30 minutes.
