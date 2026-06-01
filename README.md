# NYC Ride-Hailing Operations and Market Share Analysis

## Background and Overview
As our team plans the upcoming operational and marketing strategies for the New York City market, we need a clear understanding of current passenger demand and competitor positioning. The objective of this analysis is to evaluate 21 million recent trip records across the NYC High Volume For-Hire Vehicle sector. By analyzing over $534 million in gross fare revenue, this report identifies how volume translates to actual revenue, where operational friction exists in the dispatch process, and what temporal trends dictate passenger behavior.
*(For a detailed breakdown of the data cleaning processes and the DuckDB aggregation pipeline, please refer to the [Technical Documentation Link](Technical_Architecture_and_Pipeline.ipynb)).*

## Data Structure Overview
The foundation of this analysis relies on a dimensional model built from raw TLC trip records. The central fact table tracks individual trips and is filtered to remove physical anomalies like impossible city speeds or negative fares. This fact table is connected to lookup dimensions including NYC Taxi Zones, time intervals, and company identifiers to allow for accurate cross-filtering.

## Executive Summary
Uber currently maintains a dominant market position with 15.1 million trips compared to Lyft's 5.9 million trips. However, spatial analysis reveals that outer-borough revenue is driven more by trip type than sheer volume, for example, despite processing 2 million fewer trips than Brooklyn, Queens generated $1.7M more in gross fare revenue, driven entirely by airport route concentration at JFK and LaGuardia. Furthermore, dispatch data indicates that higher neighborhood trip volume correlates with lower passenger wait times, pointing to coverage rather than traffic as the primary friction point. Finally, temporal trends confirm that weekday commuting is the primary driver of platform usage, representing an opportunity to test scheduled retention products.

![KPI Banner](assets/KPI%20Banner.jpg)

## Insights Deep Dive

### Volume vs. Ticket Size in the Outer Boroughs
The data shows a distinct difference in how boroughs generate revenue. Brooklyn processed approximately 6 million trips, while Queens processed roughly 4 million. Despite the lower volume, Queens generated more total gross fare revenue ($91.1M in Queens versus $89.4M in Brooklyn).

**The "So What":** Queens contains both JFK and LaGuardia airports. This borough relies on low-frequency, high-ticket airport routes. Brooklyn relies on high-frequency, low-ticket neighborhood hops. Manhattan remains the highest grossing zone overall due to a combination of high volume and premium pricing constraints.

![Geographic Demand Distribution](assets/Geographic%20Demand%20Distribution.jpg)

### Wait Times and Driver Density
When mapping average total wait time against trip volume for every zone in the dataset, a clear inverse relationship emerges. The scatter plot shows the upper-left quadrant (low volume, high wait time) is the most densely populated region of the chart. As trip volume increases, wait times consistently fall, with virtually all high-volume zones clustering below the platform average of 5.49 minutes.

**The "So What":** This is a density problem, not a traffic problem. The dispatch algorithm performs efficiently wherever driver supply is concentrated. The friction exists precisely where supply is thin, and the plot shows this is not an isolated geographic issue but a systemic pattern affecting a significant portion of the zone network. Addressing individual zones treats symptoms. Addressing the structural under-supply in low-volume zones treats the cause.

![Wait Time vs. Neighborhood Demand](assets/Wait%20Time%20vs.%20Neighborhood%20Demand.jpg)

### The Commuter Backbone
Analyzing trip volume by hour and day type reveals a behavioral pattern with direct fleet management implications. Total platform demand drops by over 50% on weekends, confirming that these platforms function primarily as commuter infrastructure, not leisure transportation. On weekdays, demand follows a precise two-peak structure: a sharp morning spike approaching 1 million total trips at 8:00 AM, followed by a sustained plateau between 4:00 PM and 7:00 PM.

**The "So What":** These two peaks represent two entirely separate fleet positioning problems. The 8:00 AM spike originates at residential zones, meaning drivers need to be pre-positioned in outer boroughs before demand materializes, not reacting to it after. The evening plateau originates at commercial and office-dense zones, a different geographic concentration entirely. A fleet strategy that treats these as one "rush hour" problem will underperform both. The operational implication is that driver incentives, positioning bonuses, and supply targets should be calibrated to two distinct zone profiles at two distinct times, not applied uniformly across the day.

![Intra-Day Commuter Behavior](assets/Intra-Day%20Commuter%20Behavior.jpg)

### Driver Compensation and AOV
Uber drivers currently yield a higher active hourly return at $66.04 compared to Lyft drivers at $62.95. On the passenger side, Uber's Average Order Value is $25.77 while Lyft's is $24.64, a $1.13 difference per trip.

**The "So What":** Uber charges passengers slightly more per trip and pays drivers slightly more per hour. Whether this reflects a deliberate premium positioning strategy, a different trip-type mix, or Uber's larger driver supply absorbing more surge-priced trips requires deeper analysis beyond this dataset. What the data does confirm is that Lyft's lower cost to passengers does not translate into a meaningful savings for drivers. The $3.09 hourly gap suggests Lyft's efficiency advantage in wait times does not convert into equivalent driver earnings.

![Driver Hourly Earnings by Company](assets/Driver%20Hourly%20Earnings%20by%20Company.jpg)

## Recommendations
Based on the insights above, we recommend the following actions for the cross-functional teams:

1. **Segmented Marketing Campaigns:** The marketing team should allocate promotions based on spatial behavior. Campaigns in Queens should heavily target airport travelers to capture high-margin long hauls, while Brooklyn campaigns should focus on high-frequency user retention.
2. **Explore Commuter Retention Products:** Since the data proves weekdays are anchored by strict commuting hours, the product team should explore a pilot program for a scheduled, subscription-based daily pickup. Securing these 8:00 AM riders through recurring billing could lock in revenue that is currently left to ad-hoc daily booking.
3. **Boarding Time UI Tests:** The data indicates Uber passengers take an average of 0.9 minutes to board a vehicle after it arrives, compared to 0.6 minutes for Lyft. The product team should A/B test the UI of our arrival notifications. Getting passengers to the curb just 15 seconds faster would save tens of thousands of idle labor hours across millions of trips.
4. **Systemic Driver Supply Rebalancing:** The scatter plot reveals a platform-wide pattern: low-volume zones disproportionately suffer from above-average wait times while high-volume zones self-correct through driver density. This is a systemic supply distribution issue, not a localized geographic one. Operations should implement a volume-tiered incentive structure that automatically raises driver earnings floors in any zone falling below a defined trip threshold during peak commuter windows, making the program self-updating as demand patterns shift rather than requiring manual zone identification. Success metric: reduce the proportion of zones in the high-wait, low-volume quadrant by 20% within 90 days, measured by re-running this scatter analysis on the following month's data.

## Caveats & Assumptions
To ensure the broader team understands the parameters of this analysis, please note the following data limitations:
* **Gross vs. Net Revenue:** All revenue metrics are calculated strictly using `base_passenger_fare`. This excludes tolls, taxes, tips, and platform fees. Therefore, this metric represents the Gross Economic Value of the trip, not the platform's final net take-home margin.
* **Pre-Aggregation:** To analyze 21 million rows efficiently, data was pre-aggregated by hour and zone. Metrics like Wait Time are represented as averages rather than medians. While extreme outliers were trimmed prior to aggregation, right-skewed data inherently pulls averages slightly higher.
* **Scope Definition:** This analysis is strictly limited to App-Based For-Hire Vehicles. Traditional Yellow and Green NYC Taxi cabs were excluded to keep the focus entirely on algorithmic dispatch platforms.
* **Efficiency Metrics vs. Causality:** This analysis utilizes "Revenue per Mile" as a benchmark to compare borough performance. While this is a perfectly valid and effective efficiency KPI, it should not be interpreted as a predictive formula for pricing. Because fares are dynamically adjusted by "hidden ingredients" (surge multipliers, time-based traffic delays, and tolls), distance acts as a baseline for efficiency, not a direct driver of fare cost.

## Data Source
This analysis is based on the **High Volume For-Hire Vehicle (HVFHV) Trip Records** provided by the New York City Taxi & Limousine Commission (TLC). 
* **Specific Dataset:** The analysis utilizes records from [March 2026](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).
* **Scope:** The dataset is limited to app-based dispatch platforms (Uber and Lyft). Traditional yellow and green medallion taxis are excluded from this specific dataset.
* **Access:** The complete historical data catalog is available for public download via the official [NYC TLC Trip Record Data portal](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).