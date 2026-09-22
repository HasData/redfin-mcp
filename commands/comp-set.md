---
description: A comparable set for a home, built from recent Redfin sales
---

Build a comp set.

Ask me for the area and the profile of the home if I have not given them, meaning the bedrooms, the bathrooms and the rough size in square feet.

Then:

1. Call `hasdata_redfin_listing_getRealEstateListings` with the area as `keyword` and `type: sold`, sorted by `mostRecentlySold`, and push the profile into the query with `beds_min_`, `beds_max_`, `baths` and the `forSaleSquareFeet_min_` and `forSaleSquareFeet_max_` band rather than filtering afterwards. Add `soldWithinOption` when I gave a window.
2. Report what `searchInformation.totalResults` says and how many you are working from. A call returns about forty listings, so an unfiltered market of thousands cannot be summarised from one page.
3. Drop listings with no `price` and say how many you dropped. A missing price is withheld, not zero.
4. If the kept set is thin, ask for more with `page` rather than loosening the profile, and say how many pages you pulled.
5. List the comps with address, price, `area` in square feet, `beds`, `baths`, `yearBuilt` and `daysOnSite`.
6. Give the median price and the median price per square foot across the kept set, and name the two homes that pull the range at each end.
7. Run the same search with `type: forSale` when I ask what it would list for today, and keep the two sets apart in the answer.

Say these are Redfin's figures and give the date you pulled them. A comp set without a date is a guess about a market that moves weekly.
