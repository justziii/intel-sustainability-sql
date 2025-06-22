# Intel Sustainability SQL Analysis

This project analyzes device repurposing and environmental impact using SQL and Intel-provided data. By joining device and sustainability datasets, the analysis calculates energy savings, CO₂ emissions reductions, and device usage trends across time, region, and type.

**Skills Used:** SQL (joins, CTEs, CASE logic, aggregation), sustainability analytics, data storytelling  
**Tools:** SQL, GitHub, Markdown  
**Goal:** To uncover patterns in device reuse and quantify the environmental benefits of extending hardware lifecycles.




## 🔹 Query 1: Add `device_age` Column to Joined Dataset

**Prompt:**  
To your joined dataset, add a new column called `device_age` calculated by subtracting the `model_year` from 2024. For example, a 2019 device should be 5 years old.

```sql
WITH sustain_data AS (
  SELECT
    d.*,
    i.*
  FROM intel.device_data AS d
  LEFT JOIN intel.impact_data AS i 
    ON d.device_id = i.device_id
)

SELECT
  *,
  2024 - model_year AS device_age
FROM sustain_data;
>  **💡 Insight:** This query calculates the age of each device relative to 2024 by subtracting the `model_year`.
The output helps in evaluating the environmental impact of devices over their lifespan.


## Query 2: Identify Repurposing Patterns by Device Age

**Prompt:**  
Order your joined data by `model_year` (oldest to newest). Do you notice more older (5+ years) or newer (under 5 years) devices being repurposed? What might that indicate?

```sql
-- Sort the dataset by model year
SELECT *
FROM sustain_data
ORDER BY model_year ASC;

>  **💡 Insight:**
The older devices are being repurposed more frequently than the newer ones. This indicates that organizations are extending the lifecycle of technology by diverting aging devices from e-waste streams. Repurposing older hardware supports sustainability goals by reducing the environmental impact associated with manufacturing new devices and lowering overall electronic waste.




## Query 3: Create Device Age Buckets

**Prompt:**  
Bucket the `device_age` to analyze trends and CO₂ impact more effectively. Add a new column called `device_age_bucket` using a `CASE WHEN` clause based on calculated device age:

- If age is ≤ 3 → `"newer"`  
- If age is > 3 and ≤ 6 → `"mid-age"`  
- If age is > 6 → `"older"`

```sql
-- Joined table for device and impact data
WITH sustain_data AS (
  SELECT
    d.*,
    i.*
  FROM
    intel.device_data AS d
    LEFT JOIN intel.impact_data AS i ON d.device_id = i.device_id
),
-- Device age calculation, wrapped in another CTE to make it cleaner 
device_age_calc AS (
  SELECT
    *,
    recycling_rate,
    2024 - model_year AS device_age
  FROM
    sustain_data
)

-- Device bucket age calculated here
SELECT
  *,
  CASE
    WHEN device_age <= 3 THEN 'newer'
    WHEN device_age > 3 AND device_age <= 6 THEN 'mid-age'
    ELSE 'older'
  END AS device_age_bucket
FROM
  device_age_calc;
>  **💡 Insight:**
This query simplifies lifecycle analysis by grouping devices into age categories: "newer", "mid-age", and "older". These buckets make it easier to evaluate usage and recycling behavior patterns, and to prioritize sustainability efforts based on the typical performance or impact of each device age group.



## Query 4: Calculate Summary Metrics for Repurposed Devices in 2024

**Prompt:**  
Write a query that returns the following metrics for repurposed devices in 2024:
- Total number of devices repurposed  
- Average age of repurposed devices  
- Average estimated energy savings (kWh/year)  
- Total CO₂ emissions saved (in tons)

Note: Since `co2_saved_kg_yr` is measured in kilograms, divide the total by 1,000 to convert to tons.

```sql
-- Joined table for device and impact data
WITH sustain_data AS (
  SELECT
    d.*,
    i.*
  FROM
    intel.device_data AS d
    LEFT JOIN intel.impact_data AS i ON d.device_id = i.device_id
),
-- Device age calculation, wrapped in another CTE to make it cleaner
device_age_calc AS (
  SELECT
    *,
    recycling_rate,
    2024 - model_year AS device_age
  FROM
    sustain_data
) -- Don't need the bucket data for this calculation, got rid of it for cleaner code
SELECT
  COUNT(*) AS total_devices,
  ROUND(AVG(device_age), 2) AS avg_device_age,
  ROUND(AVG(energy_savings_yr), 2) AS avg_energy_savings_kwh,
  ROUND(SUM(co2_saved_kg_yr) / 1000, 2) AS total_co2_saved_tons
FROM
  device_age_calc;

>  **💡 Insight:**
This query summarizes key sustainability metrics for repurposed devices in 2024. By calculating the average age, annual energy savings, and total CO₂ reduction (in tons), the analysis quantifies both the environmental impact and efficiency gains of extending device lifespans through reuse.



## Query 5: Compare Sustainability Metrics by Device Type

**Prompt:**  
Write a query that returns the total number of devices, the average energy savings, and the average CO₂ emissions saved (in tons), grouped by `device_type`.

Note: Divide `AVG(co2_saved_kg_yr)` by 1,000 to convert kilograms to tons.

```sql
-- Joined table for device and impact data
WITH sustain_data AS (
  SELECT
    d.*,
    i.*
  FROM
    intel.device_data AS d
    LEFT JOIN intel.impact_data AS i ON d.device_id = i.device_id
),
-- Device age calculation, wrapped in another CTE to make it cleaner
device_age_calc AS (
  SELECT
    *,
    recycling_rate,
    2024 - model_year AS device_age
  FROM
    sustain_data
) -- Don't need the previous block of aggregation for this calculation, got rid of it for cleaner code
SELECT
  device_type,
  COUNT(*) AS total_devices,
  ROUND(AVG(energy_savings_yr), 2) AS avg_energy_savings_kwh,
  ROUND(AVG(co2_saved_kg_yr) / 1000, 2) AS avg_co2_saved_tons
FROM
  sustain_data
GROUP BY
  device_type;


>  **💡 Insight:**
This query helps identify which device types contribute the most to sustainability goals. For example, devices with higher average energy or CO₂ savings may be prioritized for repurposing initiatives, while less efficient types could be candidates for design improvement or early retirement.



## Query 6: Analyze Sustainability Impact by Device Age Bucket

**Prompt:**  
Write a query that returns the total number of devices, the average energy savings, and the average CO₂ emissions saved (in tons), grouped by `device_age_bucket`.

```sql
WITH sustain_data AS (
  SELECT
    d.*,
    i.*
  FROM
    intel.device_data AS d
    LEFT JOIN intel.impact_data AS i ON d.device_id = i.device_id
),

device_age_calc AS (
  SELECT
    *,
    2024 - model_year AS device_age
  FROM
    sustain_data
),

device_buckets AS (
  SELECT
    *,
    CASE
      WHEN device_age <= 3 THEN 'newer'
      WHEN device_age > 3 AND device_age <= 6 THEN 'mid-age'
      ELSE 'older'
    END AS device_age_bucket
  FROM
    device_age_calc
)

SELECT
  device_age_bucket,
  COUNT(*) AS total_devices,
  ROUND(AVG(energy_savings_yr), 2) AS avg_energy_savings_kwh,
  ROUND(AVG(co2_saved_kg_yr) / 1000, 2) AS avg_co2_saved_tons
FROM
  device_buckets
GROUP BY
  device_age_bucket;

>  **💡 Insight:**
It looks like newer devices are being repurposed the most, but they save the least amount of energy on average. On the other side of things, older devices save the most energy and CO₂ per device, but far fewer of them are being reused. This suggests that while repurposing newer devices happens at scale, there’s a lot of environmental value in targeting older devices too—they offer a bigger impact per unit.


## Query 7: Analyze Sustainability Metrics by Region

**Prompt:**  
Write a query that returns the total number of devices, the average energy savings, and the average CO₂ emissions saved (in tons), grouped by region.

```sql
WITH sustain_data AS (
  SELECT
    d.*,
    i.*
  FROM
    intel.device_data AS d
    LEFT JOIN intel.impact_data AS i ON d.device_id = i.device_id
),

device_age_calc AS (
  SELECT
    *,
    2024 - model_year AS device_age
  FROM
    sustain_data
)

SELECT
  region,
  COUNT(*) AS total_devices,
  ROUND(AVG(energy_savings_yr), 2) AS avg_energy_savings_kwh,
  ROUND(AVG(co2_saved_kg_yr) / 1000, 2) AS avg_co2_saved_tons
FROM
  device_age_calc
GROUP BY
  region;


>  **💡 Insight:**
This query breaks down sustainability metrics by region, offering insight into which areas are repurposing devices more effectively. It can help inform where to scale up programs or target outreach to regions with lower average energy savings or higher emissions potential.





## Final Insight: Environmental Impact at Scale

One of the most impactful comparisons was realizing that the energy saved from repurposing devices—nearly **4 gigawatt-hours**—could power around **360 homes** for an entire year.

Even more striking, the program prevented over **6,700 tons of CO₂ emissions**, which is equivalent to taking about **1,470 gasoline-powered cars off the road**.

These savings show just how significant device repurposing can be when scaled—helping reduce both electricity demand and environmental impact in a way that feels tangible and familiar.



## What I Learned

- How to use SQL to perform multi-level analysis with CTEs and conditional logic  
- How to apply data analytics to real-world sustainability challenges  
- The importance of framing insights in terms of business and environmental impact

## Next Steps

- Visualize these results using Tableau or Power BI  
- Run time-series analysis if more years of data become available  
- Explore predictive modeling for forecasting sustainability gains
