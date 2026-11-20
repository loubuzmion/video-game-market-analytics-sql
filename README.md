# Video Game Sales & Reviews Analysis with SQL

A SQL analysis of a video game dataset to find which **game titles, genres and publishers** generate the most revenue and customer engagement. It uses joins, aggregation and window functions (`RANK() OVER (PARTITION BY ...)`).

## Team

This was a **group project of 3 people** (Reyna, Lou and Francis).

## Objective

Help a business decide where to focus its resources by answering:
1. Which games, genres and release years earn the most revenue?
2. What share of customers leave reviews, and how many of those reviews are in English?
3. Which publishers perform best?
4. How do games rank against others in the same genre, by revenue and by reviews?

## Dataset

Three related tables, joined on `game_id`:

| Table | Key columns |
|---|---|
| `games_description` | `game_id`, `game_name`, `genre`, `publisher`, `year_released` |
| `games_revenue` | `game_id`, `number_of_purchases`, `unit_price` |
| `games_reviews` | `game_id`, `game_name`, `number_of_reviews_from_purchased_people`, `number_of_english_reviews` (plus Helpful / Funny tag counts) |

**Revenue** is calculated as `number_of_purchases * unit_price`.

> Game and publisher names are truncated in the source data (e.g. "Counter", "PUBG: B", "Rocksta"), so they appear that way throughout this project.

## Tools

SQL (MySQL syntax: `FORMAT()`, `LIMIT`), Microsoft Word for the written report.

## Approach

| Task | What was done | SQL techniques |
|---|---|---|
| **Revenue** | Total revenue per game, top 5 games, top 3 genres, best year | `LEFT JOIN`, `SUM()`, `GROUP BY`, `ORDER BY`, `LIMIT` |
| **Review percentage** | % of purchasers who left a review | Three-table `LEFT JOIN`, `ROUND()` |
| **English reviews** | % of reviews written in English | Ratio calculation |
| **Publisher insights** | Top 5 publishers by revenue, top titles by purchases | `INNER JOIN`, `GROUP BY` publisher |
| **Engagement ratios** | Highest and lowest engagement percentages | Joins, `SUM()` ratios |
| **Window functions** | Rank games by revenue and by reviews within each genre | `RANK() OVER (PARTITION BY genre ORDER BY ...)` |

### Why a LEFT JOIN?
`games_description` is the master list of games, so a `LEFT JOIN` keeps every game, including those with no revenue record. Those rows show `NULL` instead of being dropped, which highlights data gaps. `COALESCE(..., 0)` can turn the NULLs into 0 for display. For the publisher ranking an `INNER JOIN` was used. The team confirmed a `LEFT JOIN` gives the same top 5, because games with no revenue add nothing to a total.

### Example query: rank games within each genre
```sql
SELECT gd.genre,
       gd.game_name,
       FORMAT(SUM(gr.number_of_purchases * gr.unit_price), 0) AS total_revenue,
       RANK() OVER (PARTITION BY gd.genre
                    ORDER BY SUM(gr.number_of_purchases * gr.unit_price) DESC) AS revenue_rank
FROM games_revenue gr
LEFT JOIN games_description gd ON gr.game_id = gd.game_id
GROUP BY gd.genre, gd.game_name
ORDER BY gd.genre, revenue_rank;
```

## Key Findings

### Revenue
- **Total revenue: $1,615,107,933** across all released games.
- **One game dominates:** Counter earned **$407,366,540 (25.22% of all revenue)**. Every other top-5 game is below 7%.

| Rank | Game | Revenue | Share of total |
|---|---|---|---|
| 1 | Counter | $407,366,540 | 25.22% |
| 2 | PUBG: B | $105,578,436 | 6.54% |
| 3 | Dota2 | $91,050,736 | 5.64% |
| 4 | Grand T | $48,013,200 | 2.97% |
| 5 | Terrari | $41,601,392 | 2.58% |

- **Action is by far the top genre:** $1,258,756,620, about **78% of all revenue**. Simulation is second at $184,076,708 and Role-Playing third at $73,065,268.
- **2012 was the highest-earning year**, at $427,990,780.

### Publishers
| Publisher | Total revenue |
|---|---|
| Valve | $604,943,693 |
| KRAFTON | $105,578,436 |
| Rocksta | $57,446,151 |
| Electro | $49,495,729 |
| CD PROJ | $47,518,459 |

- **Valve earns about 37% of all revenue.** Counter is 67% of Valve's total (11,639,044 units at $35.00), with the other $197,577,153 split across their 6 other titles.
- The top titles of the other publishers are all **Action games**: PUBG: B (KRAFTON), Grand T (Rocksta), Apex Le (Electro) and The Wit (CD PROJ). Valve has 5 titles in the top 20 most-purchased games.

### Genre rankings (window functions)
| Genre | Revenue leader | Review leader |
|---|---|---|
| Action | Counter ($407M), then PUBG and Dota 2 | Counter (8.3M+ reviews) |
| Role-Playing | Stardew Valley (more than double the runner-up) | Stardew Valley (644K) |
| Simulation | Garry's Mod ($33M), then War Thunder and Euro Truck Simulator | Garry's Mod (945K) |
| Strategy | Bloons (over $9M) | Bloons (289K) |
| Adventure | Baldur's, Lethal, Palworld (close race) | Same three titles |
| Sports & Racing | Fishing, Russian, Need for Speed (close) | Need for Speed, Russian, Fishing |

- In Action, Simulation and Role-Playing the **revenue leader is also the review leader**, which suggests revenue and community activity go together.
- Adventure and Sports & Racing are **more evenly matched**, with several titles close together.

### Reviews and engagement
- Review participation (reviews as a share of purchases) is highest in Action games (about 66–71%), moderate in Simulation, Role-Playing and Adventure (about 50–55%), and lowest in Strategy and Sports & Racing (about 33–50%).
- The English-review share is very high for most games. Games with the **lowest English share** (Shawarm 6.41%, Black M 7.93%, Naraka 8.08%, Russian 8.53%, All In 0%) point to strong non-English audiences.
- Engagement does not guarantee revenue: **The Ram** has a very high review ratio but only **$3,300** in revenue. This result needs validating (see Limitations).

## Recommendations
- Focus development and marketing on the high-performing genres (**Action, Simulation, Role-Playing**).
- For games with high engagement but low revenue, explore monetisation such as cosmetics, expansions or seasonal events.
- Add in-game review prompts or small incentives for titles with low review participation.
- Add **localisation** (e.g. Chinese, Russian, Spanish) for games with a low English share.
- In genres with balanced competition, differentiate new titles with unique features.

## Limitations
- **English review percentages above 100%** (e.g. The Ram at 1000%). The English count and the total review count probably cover different groups of reviewers, so the "highest English share" and "most engaged" results for these games are unreliable until validated.
- **The Engagement Ratios queries calculate the English-review share, not the Helpful/Funny ratio** that the task asks for (see Possible Improvements).
- The data has no overall total of reviews (only reviews from purchasers), so true engagement rates cannot be calculated.
- Names are truncated, and some analysis depends on a single game (Counter is a quarter of revenue), so company-wide conclusions are sensitive to it.
- Revenue is derived from purchases × unit price, so refunds, discounts and in-game purchases are not captured.

## Possible Improvements
- **Fix the Engagement Ratio queries** to calculate Helpful and Funny tags as a percentage of total reviews, e.g. `(SUM(helpful) + SUM(funny)) / SUM(total_reviews) * 100`, using `NULLIF(..., 0)` to avoid division by zero.
- **Filter or flag impossible values** (English % over 100) in the query, or investigate the source.
- **Show only the top N per genre** with a CTE, so the ranking output is easier to read:
  ```sql
  WITH ranked AS (
    SELECT gd.genre, gd.game_name,
           SUM(gr.number_of_purchases * gr.unit_price) AS revenue,
           RANK() OVER (PARTITION BY gd.genre
                        ORDER BY SUM(gr.number_of_purchases * gr.unit_price) DESC) AS revenue_rank
    FROM games_revenue gr
    JOIN games_description gd ON gr.game_id = gd.game_id
    GROUP BY gd.genre, gd.game_name
  )
  SELECT * FROM ranked WHERE revenue_rank <= 3 ORDER BY genre, revenue_rank;
  ```
- **Return numbers, not text.** `FORMAT()` produces text, so keep raw numbers and format only for display.
- **Calculate review participation by genre** (`AVG` grouped by genre) so the genre-level claims are backed by a query.
- Add **revenue share** (`SUM(...) OVER ()`) and **running totals by year** with more window functions, and compare a game's revenue with its genre average.
- Visualise the results (Power BI or Python) with bar charts of revenue by genre and publisher.

## Conclusion

Revenue is highly concentrated: Action games, and Valve's Counter in particular, drive most of the company's income. Revenue and review leaders match in the biggest genres, but high engagement does not always mean high revenue. The review and engagement findings should be treated cautiously until the over-100% English percentages are validated and the Helpful/Funny ratio is calculated properly.
