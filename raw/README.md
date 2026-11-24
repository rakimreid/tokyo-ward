# Raw data: Inside Airbnb, Tokyo (snapshot 2025-09-29)

Source: Inside Airbnb, Tokyo, data compiled on 29 September 2025 (scraped 29–30 September 2025), https://insideairbnb.com/get-the-data/. Licensed under Creative Commons Attribution 4.0 International (CC BY 4.0), https://creativecommons.org/licenses/by/4.0/.

Changes made for this repository:
- `listings.csv.gz`: only the 12 columns the pipeline reads (`id, neighbourhood_cleansed, latitude, longitude, host_neighbourhood, neighborhood_overview, price, review_scores_location, review_scores_rating, number_of_reviews, description, last_scraped`). Host names and the other columns were removed.
- `reviews_part1–5.csv.xz`: only `id, listing_id, date, comments`, split into five consecutive row blocks. Reviewer names and IDs were removed.
- Values are otherwise unchanged (verified: reading these files gives exactly the same table as reading those columns from the original files).

The notebook also accepts the original full files: put `listings.csv(.gz)` and `reviews.csv(.gz)` in this folder instead.
