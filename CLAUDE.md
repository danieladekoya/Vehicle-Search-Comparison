# Auto Comparism – Project Context

## Project Overview
An automated vehicle search and comparison system for Ontario, Canada. When a customer searches for a vehicle make and model, the system confirms their location, searches major Ontario car dealership platforms nearby and automobile marketplaces, collects and normalizes vehicle listings, compares them on key buyer decision factors, and delivers an AI-powered best deal recommendation with a clear, plain-English buyer report.

The solution is built as an **n8n workflow** integrated with **Claude AI** (claude-sonnet-4-6), with **Firecrawl** as the scraping engine.

---

## Architecture

### Process Flow (mapped on Miro board)
1. Customer opens search page → Browser auto-requests geolocation (lat/lng)
2. Geolocation Permission Check — if denied, fallback postal code / city input shown
3. Customer selects radius (25 / 50 / 100 / 200 km) and enters vehicle make, model, and year → submits
4. n8n Webhook receives the payload
5. IF Node validates: make + model + year present AND (lat/lng OR fallback city) present
6. Geocoding Node converts lat/lng → Ontario postal code + city name
7. Two parallel Firecrawl scraping branches fire simultaneously:
   - **Branch A (Location-Scoped):** Ontario dealer sites nearby based on user's location — queried with postal code + radius parameters
   - **Branch B (All-Ontario):** AutoTrader.ca + CarGurus Canada + Cars.ca + Clutch.ca — all Ontario scope
8. Merge Node fans-in all results from both branches
9. Code Node normalizes and deduplicates listings by VIN; tags each listing with `source_tier` and `seller_type`
10. AI Agent Node (OpenRouter) produces buyer-friendly comparison report + recommendation → Respond to Webhook returns two-tier results to the website

### Frontend ↔ n8n Integration
- Website sends a `POST` request to the n8n Webhook URL with the search payload (make, model, filters, location, radius)
- Website immediately shows a **loading screen** while n8n processes ("Searching Ontario for your Toyota Camry...")
- Frontend HTTP timeout must be set to **3 minutes** to allow full processing time (30–90 seconds typical)
- n8n's "Respond to Webhook" node is placed at the **end** of the workflow so it holds the connection until all scraping and AI analysis is complete
- When complete, website receives the full JSON response and dynamically renders the two-tier results page — no page reload needed

### Results Page Layout
```
[Search header: "Toyota Camry — Within 50 km of Mississauga, ON — 8 nearby · 16 across Ontario"]

⭐ NEARBY DEALERS (within X km)
  [Vehicle cards — ranked, best deal highlighted at top]

🌍 ALL ONTARIO LISTINGS
  [Vehicle cards — ranked]

💡 RECOMMENDATION
  [2–3 sentence plain-English best deal summary]

⚠️ WATCH OUT
  [Any flags about accident history, expiring warranty, high mileage]
```

### Error Handling
- n8n error or timeout → "Something went wrong. Please try again."
- No listings found → "No listings found for [make model] in your area. Try increasing your radius."
- Geolocation denied + no city entered → inline form validation error before submission

### Miro Board
- URL: https://miro.com/app/board/uXjVGuCunBs=/
- Contains: process flowchart, vehicle comparison matrix table, architecture notes doc

---

## Ontario Data Sources (6 platforms)
| Platform | Type | Notes |
|---|---|---|
| AutoTrader.ca | Marketplace | Dealer listings only (`?seller=dealer`); supports location filter |
| Kijiji Autos | Marketplace | All-Ontario search only |
| CarGurus Canada | Marketplace | Supports location filter |
| Cars.ca | Marketplace | Supports location filter |
| Clutch.ca | Online dealer | Inherently dealer-only; Ontario-wide delivery with location filter |
| Ontario dealer websites | Dealer sites | Toyota, Honda, Ford, GM, Hyundai, Kia, Nissan, Mazda, Subaru, VW, BMW, Mercedes, Audi, Lexus, etc. |

**Removed:** Facebook Marketplace (requires login + JavaScript, not scrapable), eBay Motors Canada (not commonly used by Ontario buyers)

---

## Vehicle Comparison Fields
| Field | Notes |
|---|---|
| VIN | Unique identifier; used for deduplication and history lookup |
| Make | e.g. Toyota, Honda, Ford |
| Model | e.g. Camry, Civic, F-150 |
| Year | Model year |
| Mileage (km) | Odometer reading in kilometres |
| Transmission | Automatic / Manual / CVT / Semi-Auto |
| Seats | Seating capacity |
| Price (CAD $) | Listed price |
| Condition | New / Used / Certified Pre-Owned (CPO) |
| Fuel Type | Gasoline / Hybrid / Electric / Diesel |
| Accident History | Clean / Minor / Major |
| Warranty | Active / Expired / Extended Available |
| Body Type | Sedan / SUV / Truck / Van / Coupe / etc. |
| Colour | Exterior colour |
| Engine | Engine size and type |
| Location (ON) | City/region in Ontario |
| Source Platform | Which platform the listing came from |
| Days on Market | How long the listing has been live |
| Deal Score | AI-assigned: Excellent / Good / Fair / Poor |
| Source Tier | `nearby_dealer` (within radius) or `all_ontario` |
| Seller Type | `dealer` or `private` |

---

## n8n Workflow Notes

### n8n Instance
- URL: https://n8n.binavotechnologies.cloud/
- Workflow name: Auto Comparism — Ontario Vehicle Search
- Workflow ID: 1TQgqeOTfd369oRj
- Workflow editor: https://n8n.binavotechnologies.cloud/workflow/1TQgqeOTfd369oRj
- Webhook endpoint: https://n8n.binavotechnologies.cloud/webhook/auto-comparism
- Test endpoint: https://n8n.binavotechnologies.cloud/webhook-test/auto-comparism

### n8n Node Mapping
| Step | Node Type | Purpose |
|---|---|---|
| Webhook | Webhook Node | Receives search payload from website |
| Validation | IF Node | Validates make + model + location present |
| Geocoding | HTTP Request Node | Converts lat/lng → postal code + city (Google Maps or Nominatim API) |
| Branch A Scraping | Firecrawl Nodes (×4) | Location-scoped scrape: AutoTrader, CarGurus, Cars.ca, Clutch.ca |
| Branch B Scraping | Firecrawl Nodes (×6) | All-Ontario scrape: AutoTrader, Kijiji, CarGurus, Cars.ca, Clutch.ca, dealer sites |
| Fan-in | Merge Node | Combines results from both branches |
| Normalization | Code Node | Deduplicates by VIN; tags source_tier + seller_type |
| AI Analysis | AI Agent Node | Scores, ranks, and generates buyer report |
| Response | Respond to Webhook Node | Returns full two-tier JSON result to website |

### Credentials Required
- Claude API key (Anthropic) — for AI Agent Node
- Firecrawl API key — for all scraping nodes
- Google Maps Geocoding API key (or use OpenStreetMap Nominatim — free, no key required)

---

## AI Recommendation Logic
The Claude AI Agent node receives the full normalized listing dataset and:
1. Filters listings outside budget and location preferences
2. Scores each vehicle 0–100 across all buyer factors with weighted criteria
3. Ranks vehicles from best to worst overall value — separated into two tiers (nearby dealers / all Ontario)
4. Identifies the #1 Best Deal in each tier with a plain-English justification
5. Flags any vehicles with accident history, high mileage, or expiring warranty — in plain language
6. Returns a structured JSON response with:
   - **Summary header** — total listings found, nearby vs. all-Ontario counts
   - **Nearby dealer listings** — ranked cards with distance, price, key specs, deal score, flags
   - **All Ontario listings** — ranked cards with platform, city, key specs, deal score, flags
   - **Recommendation** — 2–3 sentences max, plain English, no jargon
   - **Watch-out flags** — summarized at the bottom (e.g., "3 listings have accident history")

### Buyer Report Card Format (per listing)
```
[BEST DEAL badge if applicable]
[Year] [Make] [Model] [Trim] — $[Price]
📍 [Dealer name] ([distance] km away)  OR  📍 [City, ON] via [Platform]
🔢 [Mileage] km | [Transmission] | [Condition]
⛽ [Fuel Type] | [Seats] seats | [Colour]
✅/⚠️ [Accident history] | [Warranty status] | [Days on market] days on market
Deal Score: [Rating] ([Score]/100)
🔗 [View Listing]
```

### Plain-English Flags
- ⚠️ "This vehicle has a minor accident on record"
- ⚠️ "Warranty expires within 3 months"
- ⚠️ "High mileage for its year"
- ✅ "Clean history" / "Warranty active" / "Certified Pre-Owned"

---

## Skills

| Skill | Command | Description |
|---|---|---|
| Miro Plan | `/miro-plan [board-id-or-url]` | Reads a Miro board, extracts the process map, saves docs to `docs/`, and produces a phased implementation plan. Pauses for confirmation before coding. |

**Miro Board:** `https://miro.com/app/board/uXjVGuCunBs=/`
Quick access: `/miro-plan uXjVGuCunBs=`


| Frontend Design | `/frontend-design [description]` | Creates distinctive, production-grade frontend interfaces with high design quality. Use when building search pages, results pages, vehicle cards, loading screens, or any UI component for this project. |


---

## Project Notes

<!-- Use this section to add important context, decisions, constraints, or anything else Claude should know about this project -->
