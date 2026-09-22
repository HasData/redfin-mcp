# Redfin MCP Server

<!-- mcp-name: com.hasdata/redfin -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client two read-only Redfin tools. Search for-sale, for-rent and sold listings with the filter set Redfin shows a visitor, and read one property page in full, both as structured JSON, with no MLS licence and nothing to host.

It reads public Redfin pages that a signed-out visitor can see.

**1,000 free credits every month, no card required**, which is 200 Redfin calls at the 5-credit rate.

```
https://mcp.hasdata.com/api/mcp?apis=redfin
```

[![Glama score](https://glama.ai/mcp/servers/HasData/redfin-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/redfin-mcp)
[![tool contract](https://github.com/HasData/redfin-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/redfin-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://mcp.hasdata.com/api/mcp?apis=redfin)
[![Tools](https://img.shields.io/badge/tools-2-10b981?style=flat-square)](#tools)
[![npm](https://img.shields.io/npm/v/@hasdata/redfin-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/redfin-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-redfin-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-redfin-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [Tool selection](#tool-selection)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp), free to create with no card, and the free tier covers about 200 calls a month at the 5-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/redfin-mcp` on npm and `hasdata-redfin-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/api/mcp?apis=redfin` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http redfin "https://mcp.hasdata.com/api/mcp?apis=redfin" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Settings, then Connectors, then Add custom connector, then paste `https://mcp.hasdata.com/api/mcp?apis=redfin` and sign in.

For the config-file route, Claude Desktop loads only local (stdio) servers, so it reaches a remote server through a stdio launcher. The `@hasdata/redfin-mcp` package is that launcher, and it reads the key from the environment. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "redfin": {
      "command": "npx",
      "args": ["-y", "@hasdata/redfin-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

For Python instead of Node, swap the launcher for the PyPI package, which `uvx` runs without a manual install:

```json
{
  "mcpServers": {
    "redfin": {
      "command": "uvx",
      "args": ["hasdata-redfin-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

`~/.cursor/mcp.json` for every project, or `.cursor/mcp.json` for one:

```json
{
  "mcpServers": {
    "redfin": {
      "url": "https://mcp.hasdata.com/api/mcp?apis=redfin",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Windsurf</b></summary>

`~/.codeium/windsurf/mcp_config.json`. Windsurf calls the field `serverUrl`, not `url`:

```json
{
  "mcpServers": {
    "redfin": {
      "serverUrl": "https://mcp.hasdata.com/api/mcp?apis=redfin",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

`.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "redfin": {
      "type": "http",
      "url": "https://mcp.hasdata.com/api/mcp?apis=redfin",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Each of these lands on one tool, or on two in sequence when the second needs the URL the first returns.

- Find three-bedroom houses for sale in 78741 under $500,000 and sort them by price per square foot.
- What sold in Austin in the last three months, and how does that compare with what is listed now?
- Pull the full property page for this Redfin URL and summarise the condition from the description.
- Which rentals in Austin allow dogs and include in-unit laundry?
- Show me for-sale homes in the Austin Independent School District where the assigned elementary school rates 8 or better.
- Find fixer-uppers built before 1970 in this zipcode that have been on Redfin more than 30 days.

A prompt that names a market goes to the search tool. A prompt that hands you a Redfin URL goes straight to the property tool. Searching for a full street address is a third case, covered below, because it answers with one property rather than a list.

## Tools

| Tool | What it returns |
| --- | --- |
| `hasdata_redfin_listing_getRealEstateListings` | Each listing with address, Redfin URL, list price, beds/baths, square footage, lot size, year built, days on market, status, coordinates, photos, MLS number, and HOA; an…. 5 credits a call |
| `hasdata_redfin_property_getPropertyDetails` | Address, list/sold price, price history, Redfin Estimate, beds/baths, square footage, lot size, year built, property type, HOA, days on market, school ratings, tax…. 5 credits a call |

Two tools, 5 credits per successful call.

### Get Redfin real estate listings

[`hasdata_redfin_listing_getRealEstateListings`](https://docs.hasdata.com/apis/redfin/listing?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp)

A page of listings for a location, or one property when the location is a single address.

Two parameters are required, and the rest of the schema mirrors Redfin's own filter panel.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `keyword` | string | yes | A zipcode, city, neighborhood, school, school district, apartment building name, or a full street address |
| `type` | string | yes | `forSale`, `forRent` or `sold` |
| `sort` | string | | `recommended`, `newest`, `oldest`, `priceLowToHigh`, `priceHighToLow`, `bedrooms`, `bathrooms`, `lotSize`, `squareFeetPrice` and more |
| `page` | number | | Result page, starting at 1 |

The filters are named after the nested structure Redfin uses internally, flattened with underscores, so they have to be passed exactly as the schema spells them. `price_min_` and `price_max_` are the price band, `beds_min_` and `beds_max_` the bedroom count, `monthlyPayment_interestRate_` a mortgage assumption. A doubled underscore marks an array.

The ones worth knowing:

| Parameter | Type | Notes |
| :--- | :--- | :--- |
| `price_min_` / `price_max_` | number | Price band |
| `beds_min_` / `beds_max_` | number | Bedroom count |
| `baths` | string | Minimum baths, `one` through `four`, plus `oneAndHalf` and `twoAndHalf` |
| `homeTypes__` | array | `house`, `townhouse`, `townhome`, `condo`, `land`, `multiFamily`, `mobile`, `coOp`, `apartment`, `other`. Which values apply depends on `type` |
| `statusOptions__` | array | `active`, `comingSoon`, `contingentPending` |
| `listingType_category___` | array | `byAgent`, `byOwnerFsbo`, `newConstruction`, `foreclosures` |
| `timeOnRedfin` | string | `newListing` through `moreThan45Days` |
| `soldWithinOption` | string | Sold window, from `lastOneWeek` to `lastFiveYear`. See the warning below |
| `yearBuilt_min_` / `yearBuilt_max_` | string | A year from a fixed ladder, `1940` through `2026` |
| `forSaleSquareFeet_min_` / `_max_` | string | Floor area from a fixed ladder, `750` through `10000` |
| `lotSize_min_` / `lotSize_max_` | string | `2000 sqft` through `100 acres`, as written |
| `cost_hoa_` | number | Maximum monthly HOA fee |
| `cost_priceReduced_` | string | `inTheLastDay` through `moreThan120Days` |
| `homeFeatures_options___` | array | `waterfront`, `hasAView`, `fireplace`, `fixerUpper`, `guestHouse`, `elevator`, `greenHome`, `accessibleHome` and more |
| `homeFeatures_poolType_` | string | `privatePool`, `communityPool`, `privateOrCommunityPool`, `noPrivatePool` |
| `homeFeatures_keywordSearch_` | string | Free text against the listing description |
| `schools_greatSchoolRating_` | number | Minimum GreatSchools rating, 1 to 10 |
| `transportScores_walkScore_` | number | Minimum walk score, 1 to 100 |
| `rentalAmenities__` | array | `inUnitWasherDryer`, `parkingAllowed`, `utilitiesIncluded`, `furnished`, `pool` and more |
| `pets__` | array | `dogsAllowed`, `catsAllowed` |
| `moveInDate` | string | `MM/DD/YYYY` |

A market search returns `searchInformation` with `totalResults`, a `properties` array of 40, and `pagination` with `currentPage`, `nextPage` and an `otherPages` map. A for-sale or sold property carries `id`, `mlsId`, `url`, `homeType`, `status`, `price`, `beds`, `baths`, `area`, `yearBuilt`, `daysOnSite`, `addressRaw`, a parsed `address`, `latitude`, `longitude`, `description`, `atAGlanceFacts` and `photos`.

```json
{
  "id": 31625298,
  "mlsId": "2190201772333567097",
  "url": "https://www.redfin.com/TX/Austin/1721-Deerfield-Dr-78741/home/31625298",
  "homeType": "House",
  "status": "FOR_SALE",
  "price": 675000,
  "beds": 3,
  "baths": 2,
  "area": 1667,
  "yearBuilt": 1963,
  "daysOnSite": 0,
  "addressRaw": "1721 Deerfield Dr, Austin, TX 78741",
  "address": { "street": "1721 Deerfield Dr", "city": "Austin", "state": "TX", "zipcode": "78741" },
  "latitude": 30.231372,
  "longitude": -97.734893,
  "atAGlanceFacts": [
    { "factLabel": "Property Type", "factValue": "Single-family" },
    { "factLabel": "Year Built", "factValue": "1963" },
    { "factLabel": "Price/Sq.Ft.", "factValue": "$405" }
  ]
}
```

A rental is a building rather than a home, so `type: forRent` returns a different shape. `price`, `beds`, `baths` and `area` each become a `{ min, max }` object across the available units, and the entry adds `propertyName`, `availableUnits`, `agentEmail` and `agentPhone` while dropping `mlsId`, `homeType`, `yearBuilt` and `daysOnSite`.

```json
{
  "id": "31510362",
  "propertyName": "The Sonata",
  "status": "FOR_RENT",
  "availableUnits": 12,
  "price": { "min": 745, "max": 1300 },
  "beds": { "min": 1, "max": 2 },
  "baths": { "min": 1, "max": 2 },
  "area": { "min": 474, "max": 976 },
  "addressRaw": "1070 Mearns Meadow Blvd, Austin, TX 78758"
}
```

### Get Redfin property details

[`hasdata_redfin_property_getPropertyDetails`](https://docs.hasdata.com/apis/redfin/property?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp)

One property page in full, by its Redfin URL.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `url` | string | yes | The Redfin property URL, as the search tool returns it |

Returns a `property` object. On top of everything the search result carries, it adds `propertyDetails`, `schools`, `nearby`, `agentInfo`, `viewsActivity`, `openHouseSchedule`, `updatedAt`, a `geo` object and the full `photos` collection, which ran to 61 images on the property below.

`propertyDetails` is the specification block, grouped into `parking`, `interior`, `exterior`, `utilities` and `publicFacts`. Each group is an array of labelled sections, and each section is an array of `label` and `value` pairs, so it reads as the page presents it rather than as a typed object. Look up a fact by its label instead of by position.

`schools.assignedSchools` carries the catchment schools with `greatSchoolsRating`, `parentRating`, `distanceInMiles` and a `servesHome` flag, which is the field that says whether the school actually serves this address.

```json
{
  "id": 31625298,
  "homeType": "Single Family Residential",
  "status": "COMING SOON",
  "price": 675000,
  "beds": 3,
  "baths": 2,
  "area": 1667,
  "yearBuilt": 1963,
  "geo": { "latitude": 30.231372, "longitude": -97.734893 },
  "updatedAt": "Sep 9, 2026 4:04 AM",
  "viewsActivity": { "views": 98, "favorites": 5 },
  "agentInfo": {
    "agentName": "Lilly Rockwell",
    "agentPhoneNumber": "512-413-1975",
    "brokerName": "Compass",
    "brokerPhoneNumber": ""
  },
  "propertyDetails": {
    "parking": [{ "parkingInformation": [{ "label": "Has Garage", "value": "yes" }] }],
    "utilities": [{ "utilitiesInformation": [{ "label": "Has Air Conditioning", "value": "yes" }] }]
  },
  "schools": { "assignedSchools": [{ "greatSchoolsRating": 6, "parentRating": 5, "servesHome": true }] }
}
```

## Errors and failure paths

Plan for these rather than assuming a happy path.

**`soldWithinOption` is currently broken and silently returns for-sale homes labelled `SOLD`.** Passing any of its values puts the value straight into the Redfin filter, Redfin does not recognise it, and the response is the active for-sale list with `status` stamped as `SOLD`. Every listing came back identical to the plain for-sale search in our checks. Leave the parameter off. `type: sold` on its own works correctly and covers the last three months, which is Redfin's own default window.

**The search tool returns three different shapes, and which one you get depends on the keyword.** A market keyword answers with `searchInformation`, `properties` and `pagination`. A full street address or a named building answers with a single `property` object and no `properties` array, no `searchInformation` and no `pagination`. A rental search answers with the range-shaped entries shown above. Branch on the presence of `properties` before you iterate it.

**`totalResults` tops out at 350, and that is a ceiling rather than a count.** Austin and New York both report 350 while a single zipcode reports 168 and a small town 57. Pagination stops at nine pages of 40. To enumerate a large market, slice it by zipcode, price band or home type instead of paging, because there is no page ten.

**Sold search does not give you sold prices as a separate field.** `price` holds whatever the page shows for that status, so a for-sale price and a sold price arrive in the same field. Read `status` alongside it every time.

**There is no price history, tax history or Redfin Estimate in the property response.** Those sit on the page but are not in what the tool returns today. What you get instead is `propertyDetails.publicFacts`, which carries the assessor-style facts as label and value pairs.

**`nearby.pointsOfInterest` is nearby places, not comparable sales.** Its `categories` come from a third-party places dataset and are frequently wrong, so a title loan office can arrive tagged as a bar. Use the names and coordinates, and do not trust the category.

**`openHouseSchedule` can be an array holding an empty object** when the page has the section but no dates in it. Test the contents, not the length.

**`brokerPhoneNumber` and other agent fields come back as empty strings rather than null.** Treat empty string as absent.

Results that carry data also carry a `requestMetadata.id` worth quoting in support.

## Pricing, free tier and limits

Each Redfin tool costs **5 credits per successful call**. Response size does not change the price, so a 40-listing page and a single property cost the same.

The free tier is **1,000 credits every month with no card**, which is 200 Redfin calls at the base rate. It renews with the billing cycle, so a low-volume agent runs on the free tier indefinitely.

Paid plans start at **$59 a month** for 200,000 credits, which is 40,000 calls. The unit price falls with volume, from **$1.48 per 1,000 calls** on the entry plan to **$0.60** on Basic and **$0.41** across the Growth tiers. Current figures live on the [pricing page](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp).

Your plan also sets concurrency. The free tier allows 1 request at a time, Startup 5, Basic 15, and the Growth tiers run from 50 to 500. Retry on the 429 with a backoff in anything unattended, because an agent that walks a list of properties will reach the ceiling before you do.

A request that comes back non-200 is not billed. A successful call that finds nothing is still a call.

## Tool selection

Start from what the prompt gives you. A market, a zipcode or a school district goes to the search tool. A Redfin URL goes straight to the property tool. Spending a search call to reach a URL you already have is the most common waste.

Then pick by depth. The search result is enough for ranking, price sweeps and market summaries, and it already carries price, beds, baths, area, year built and days on site. The property tool is the only one that returns the specification block, the assigned schools and the listing agent, and it is worth one call per property you care about rather than one per row.

Filter server-side. The schema mirrors Redfin's filter panel precisely, so a query like "three beds, under $500k, built before 1970, in this zipcode" is one call with four parameters, not a page sweep followed by local filtering.

## How it compares

There is no public Redfin API, so the real alternative is an MLS or IDX feed.

| | MLS or IDX feed | This server |
| :--- | :--- | :--- |
| Eligibility | A licensed brokerage or an agent relationship | An API key |
| Setup | Per-MLS application, contract and compliance review | One header |
| Coverage | One MLS per feed, hundreds nationally | Whatever Redfin publishes, in one place |
| Sold data | Full history where the MLS permits it | The recent window Redfin shows |
| Rentals | Often a separate feed or absent | The same tool, with a range shape |
| Redistribution | Contractually restricted | Your responsibility to check |
| Cost | Setup fees plus monthly, per MLS | Paid past the free tier, 5 credits a call |

The row that decides it is eligibility. An MLS feed is the authoritative source and it needs a licence you cannot buy as a developer, which rules it out for research, prototypes and anything an agent does on your behalf. When you are a brokerage with a feed already, the feed is more complete and more current, and you should use it.

## FAQ

### Is there an official Redfin MCP server?

Redfin does not publish one, and it does not publish a public API either. This one is maintained by HasData and reads public Redfin pages.

### What is a Redfin MCP server?

An MCP server exposes tools an AI client can call. This one turns Redfin search results and property pages into JSON an agent can reason over, without a browser or a scraping library in your stack.

### Do I need an MLS licence or a Redfin account?

No. The only credential is your HasData key.

### Why does searching an address return one property instead of a list?

Because Redfin resolves a full street address to that property's page rather than to a result set. The tool passes that through, so the response holds a single `property` object in the same shape the property tool returns. It is a useful shortcut when you have an address but not a URL.

### How do I pull every listing in a city?

You cannot, in one sweep. Redfin caps a result set at 350 across nine pages, so a large market has to be cut into smaller queries by zipcode, price band or home type, and the slices stitched together.

### Can I filter sold listings by date?

Not reliably right now. `type: sold` works and returns Redfin's default three-month window, but `soldWithinOption` is not being translated into a filter Redfin accepts, and passing it returns active listings labelled `SOLD`. Leave it off until that is fixed.

### Does it cover rentals?

Yes, through `type: forRent`. Expect the range shape rather than the single-home shape, because a rental result is a building with several available units.

### Can I use this together with other HasData APIs?

Yes. One key covers everything, and one endpoint serves them all through the `apis` parameter. Point a client at `?apis=redfin,zillow` to get both tool sets in one connection, or at [`mcp.hasdata.com/api/mcp`](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp) for the full catalogue.

### Is HasData affiliated with Redfin?

No. HasData is an independent service and is not affiliated with, endorsed by, or sponsored by Redfin. Redfin is a trademark of its respective owner. The tools work with publicly available data only, and you are responsible for using the results in line with Redfin's terms and the law that applies to you.

### Compliance and personal data

Listings carry agent contact details. A for-sale property returns `agentInfo` with a name and a direct phone number, and a rental returns `agentEmail` and `agentPhone`. Those belong to identifiable people, published in a professional capacity, which does not take them out of scope of the GDPR or the CCPA. Marketing to them is regulated separately again, and real estate agents are a common target of exactly that, so check your obligations before you build a contact list. Market analysis does not need those fields at all.

## HasData links

- [Redfin Scraper API](https://hasdata.com/apis/redfin-api?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp), the REST endpoints behind these tools
- [API documentation](https://docs.hasdata.com/apis/redfin/listing?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp)
- [MCP server documentation](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp)
- [Pricing](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp)
- [Dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=redfin-mcp)

Other HasData MCP servers: [Google Search](https://github.com/HasData/google-search-mcp), [Google Maps](https://github.com/HasData/google-maps-mcp), [Google Trends](https://github.com/HasData/google-trends-mcp), [Google Flights](https://github.com/HasData/google-flights-mcp), [DuckDuckGo](https://github.com/HasData/duckduckgo-mcp), [YouTube](https://github.com/HasData/youtube-mcp), [TikTok](https://github.com/HasData/tiktok-mcp), [Instagram](https://github.com/HasData/instagram-mcp), [Amazon](https://github.com/HasData/amazon-mcp), [Shopify](https://github.com/HasData/shopify-mcp), [Yelp](https://github.com/HasData/yelp-mcp), [Zillow](https://github.com/HasData/zillow-mcp), [Airbnb](https://github.com/HasData/airbnb-mcp), [Booking.com](https://github.com/HasData/booking-mcp), [Indeed](https://github.com/HasData/indeed-mcp).

## Development

The launcher is a thin stdio bridge to the remote server, so there is nothing to build.

```bash
npm install
HASDATA_API_KEY=your_key_here npm test
```

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=redfin` returns the expected tool count, that no name changed, that every tool still declares its required parameters and carries a description, that the filter enums this README documents are still the ones the schema offers, and that the key in use is actually accepted. That last check calls a tool for real and costs 5 credits, which is the price of a canary that can fail for the right reason.

The contract suite also runs weekly on a schedule, because the upstream tool list can change without anyone touching this repository.

## Contributing

A tool table, a response sample or a documented behaviour that does not match reality is worth an issue. There is a template for exactly that. Pull requests are welcome for the same, and for anything in the launcher.

## License

MIT, see [LICENSE](LICENSE).
