# video-game-market-analytics-sql
# 🎮 Video Game Market Analytics & Player Engagement SQL Project

An end-to-end data analysis project investigating global video game sales performance, publisher market share, and consumer review dynamics using **MySQL**.

---

## 📊 Project Overview
The objective of this analysis is to identify which game titles and publishers generate the highest customer engagement and financial revenue. By examining sales data alongside review metrics, this project provides data-driven insights into top-performing genres and strategic resource allocation.

* **Total Global Revenue Analyzed:** \$1.61 Billion+ across all released titles.
* **Core Technical Stack:** MySQL (Multi-table Joins, Aggregations, Subqueries, Window Functions).

---

## 🗄️ Database Schema & Joining Strategy
The database consists of three core relational tables linked via `game_id`[cite: 1, 2]:
* **`games_description`**: Master catalog containing game names, genres, publishers, and release years[cite: 1, 2].
* **`games_revenue`**: Financial metrics including purchase counts and unit pricing[cite: 1, 2].
* **`games_reviews`**: Customer sentiment tracking review counts, English review breakdowns, hours played, and community tags (`Helpful`, `Funny`)[cite: 2].

> **SQL Architecture Note:** Utilized `LEFT JOIN` and `INNER JOIN` operations to integrate master catalog records while preventing data loss for titles with missing review or revenue entries[cite: 1, 2].

---

## 🔍 Key Insights & Findings

### 1. Revenue & Genre Dominance
* **Top Revenue Genre:** **Action games** dominate commercial performance, generating over **\$1.25 billion**, far outpacing Simulation (\$184M) and Role-Playing (\$73M).
* **Top Individual Titles:** 
  * *Counter* (Action / Valve) leads overall revenue at **\$407.36M** (25.22% of total revenue)[cite: 1].
  * *PUBG: B* (\$105.5M) and *Dota 2* (\$91.0M) round out the top tier[cite: 1].
* **Peak Revenue Year:** **2012** generated the highest annual revenue (\$427.99M) in the dataset[cite: 1].

### 2. Publisher Landscape
* **Valve** is the leading publisher by total revenue at **\$604.9M**, anchored by high-volume action titles[cite: 1].
* Other top-performing publishers include **KRAFTON**, **Rockstar Games**, **Electronic Arts**, and **CD PROJEKT**[cite: 1].

### 3. Player Engagement & Localization
* **Review Participation:** Action games exhibit the highest review conversion rates (66%–71% of purchasers leaving reviews), indicating strong community feedback loops[cite: 1].
* **The "Engagement vs. Revenue" Paradox:** High community engagement does not always correlate with high financial return (e.g., sports/racing titles like *The Ram* achieved massive social traction and 1000% English review concentration but only \$3,300 in total revenue)[cite: 1].
* **Localization Opportunities:** Titles with low English review shares (e.g., *Shawarma*, *Naraka*) indicate strong regional or international player bases that would benefit from multilingual localization strategies[cite: 1].

---

## ⚙️ Advanced SQL: Window Functions
To evaluate performance fairly within specific market segments, MySQL **Window Functions** (`RANK() OVER (PARTITION BY genre)`) were implemented[cite: 1]:
* **Revenue Ranking:** Segmented top-earning titles within their respective genres (e.g., identifying *Stardew Valley* as the RPG revenue leader and *Garry's Mod* for Simulation)[cite: 1].
* **Review Ranking:** Evaluated community activity levels per genre to isolate titles driving long-term player interest[cite: 1].

```sql
-- Sample SQL Query: Ranking games by revenue within their genre
SELECT 
    gd.genre, 
    gd.game_name, 
    FORMAT(SUM(gr.number_of_purchases * gr.unit_price), 0) AS total_revenue, 
    RANK() OVER (PARTITION BY gd.genre ORDER BY SUM(gr.number_of_purchases * gr.unit_price) DESC) AS revenue_rank 
FROM games_revenue gr 
LEFT JOIN games_description gd ON gr.game_id = gd.game_id 
GROUP BY gd.genre, gd.game_name 
ORDER BY gd.genre, revenue_rank;
