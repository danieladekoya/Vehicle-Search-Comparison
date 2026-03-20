# Auto Comparism – Project Context

## Project Overview
An automated vehicle search and comparison system for Ontario, Canada. When a customer searches for a vehicle make and model, the system searches all major Ontario car dealership websites and automobile marketing platforms, collects and normalizes vehicle listings, compares them on key buyer decision factors, and delivers an AI-powered best deal recommendation.

The solution is built as an **n8n workflow** integrated with **Claude AI** (claude-sonnet-4-6).

---

## Architecture

### Process Flow (mapped on Miro board)
1. Customer submits vehicle search (make, model, optional filters) via website
2. n8n Webhook receives the request
3. IF Node validates required fields
4. Parallel HTTP Request nodes scrape all Ontario platforms simultaneously
5. Merge Node fans-in all results
6. Code Node normalizes and deduplicates listings by VIN
7. AI Agent Node (Claude) scores, ranks, and recommends the best deal
8. Respond to Webhook returns the comparison table + recommendation to the website

### Miro Board
- URL: https://miro.com/app/board/uXjVGuCunBs=/
- Contains: process flowchart, vehicle comparison matrix table, architecture notes doc

---

## Ontario Data Sources
- AutoTrader.ca
- Kijiji Autos
- CarGurus Canada
- Cars.ca
- Facebook Marketplace (Ontario)
- eBay Motors Canada
- Ontario dealer websites (Toyota, Honda, Ford, GM, Hyundai, Kia, Nissan, Mazda, Subaru, VW, BMW, Mercedes, Audi, Lexus, etc.)

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

---

## n8n Workflow Notes

<!-- Add your n8n instance URL, credentials setup notes, and any workflow IDs here -->

### n8n Instance
- URL:
- Workflow name:
- Webhook endpoint:

### Credentials Required
- Claude API key (Anthropic)
- Any platform API keys (if applicable)

---

## AI Recommendation Logic
The Claude AI Agent node receives the full normalized listing dataset and:
1. Filters listings outside budget and location preferences
2. Scores each vehicle 0–100 across all buyer factors with weighted criteria
3. Ranks vehicles from best to worst overall value
4. Identifies the #1 Best Deal with a written justification
5. Flags any vehicles with accident history, high mileage, or expiring warranty
6. Returns a structured JSON response with ranked list + recommendation summary

---

## Project Notes

<!-- Use this section to add important context, decisions, constraints, or anything else Claude should know about this project -->

