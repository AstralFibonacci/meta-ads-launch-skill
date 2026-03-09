# Meta Ads — Claude Agent Skill

Launch angle-based Meta (Facebook/Instagram) ad campaigns programmatically using the Graph API. Implements a 3 Ad Sets × 3 Creatives CBO structure, each targeting a distinct pain point or dream outcome.

---

## What This Skill Does

- Research and define 3 distinct angles per product
- Create CBO campaigns with angle-based ad sets
- Upload images and generate ad creatives per angle
- Activate campaigns and apply creative-level stop-loss rules

---

## Requirements

- Meta access token (`META_ACCESS_TOKEN`)
- A Meta Business ad account
- Facebook Page and Instagram account connected to your ad account

---

## Campaign Structure

```
1 Campaign (CBO, your daily budget)
├── Ad Set 1: [Angle A — Pain Point or Dream Outcome]
│   ├── Creative 1A (Image 1 + Angle A Copy)
│   ├── Creative 2A (Image 2 + Angle A Copy)
│   └── Creative 3A (Image 3 + Angle A Copy)
├── Ad Set 2: [Angle B]
│   ├── Creative 1B
│   ├── Creative 2B
│   └── Creative 3B
└── Ad Set 3: [Angle C]
    ├── Creative 1C
    ├── Creative 2C
    └── Creative 3C
```

**Key rules:**
- Within an ad set: identical copy across all 3 creatives (only images differ)
- Between ad sets: copy targets a different angle
- CBO handles budget distribution across angles automatically

---

## Phase 1: Angle Research

Before creating any campaign, identify 3 distinct angles.

### What is an Angle?

An angle = a specific **pain point** or **dream outcome** your product addresses.

**Examples:**

| Product | Angle 1 (Pain) | Angle 2 (Pain) | Angle 3 (Dream) |
|---------|---------------|---------------|-----------------|
| Puzzle Feeder | Fast eating / digestion | Boredom / destruction | Mental stimulation |
| Snuffle Mat | Anxiety / stress | Rainy day energy | Nose work enrichment |

### Angle Prompt for Agent

```
Analyze this [product] and identify 3 distinct angles (pain points or dream outcomes):

Product data: [description, reviews]
Product images: [what scenarios do they show?]

Output:
Angle 1: [Name] — Type: [Pain/Dream] — Hook: [one sentence]
Angle 2: [Name] — Type: [Pain/Dream] — Hook: [one sentence]
Angle 3: [Name] — Type: [Pain/Dream] — Hook: [one sentence]
```

**Name angles clearly:**
- ❌ "Angle 1"
- ✅ "Snuffle Mat | Anxiety Relief"

---

## Phase 2: Campaign Creation

### Step 1 — Create Campaign

```bash
curl -s -X POST "https://graph.facebook.com/v22.0/act_YOUR_AD_ACCOUNT_ID/campaigns" \
  -d "name=Product | $25 CBO | YYYY-MM-DD" \
  -d "objective=OUTCOME_SALES" \
  -d "status=PAUSED" \
  -d "daily_budget=2500" \
  -d "bid_strategy=LOWEST_COST_WITHOUT_CAP" \
  -d "special_ad_categories=[]" \
  -d "access_token=$META_ACCESS_TOKEN"
```

### Step 2 — Create 3 Ad Sets (One Per Angle)

**Naming:** `[Product] | [Angle Name]`

**Scheduling rule:** Always set `start_time` to tomorrow at 08:00:00 UTC. No end time.

```bash
curl -s -X POST "https://graph.facebook.com/v22.0/act_YOUR_AD_ACCOUNT_ID/adsets" \
  -d "name=Product | Angle Name" \
  -d "campaign_id=CAMPAIGN_ID" \
  -d "status=PAUSED" \
  -d "optimization_goal=OFFSITE_CONVERSIONS" \
  -d "billing_event=IMPRESSIONS" \
  -d "bid_strategy=LOWEST_COST_WITHOUT_CAP" \
  -d "start_time=TOMORROW_08:00:00Z" \
  -d "promoted_object={\"pixel_id\":\"YOUR_PIXEL_ID\",\"custom_event_type\":\"PURCHASE\"}" \
  -d "targeting={\"geo_locations\":{\"countries\":[\"US\"]}}" \
  -d "access_token=$META_ACCESS_TOKEN"
```

### Step 3 — Create Ad Creatives (9 Total)

3 creatives per angle (same copy, different image within ad set).

```bash
curl -s -X POST "https://graph.facebook.com/v22.0/act_YOUR_AD_ACCOUNT_ID/adcreatives" \
  -d "name=Product | Angle A | Creative 1" \
  -d "object_story_spec={
    \"page_id\": \"YOUR_PAGE_ID\",
    \"link_data\": {
      \"image_hash\": \"IMAGE_HASH\",
      \"link\": \"https://yourstore.com/products/product-slug\",
      \"message\": \"Primary text for Angle A\",
      \"name\": \"Headline for Angle A\",
      \"call_to_action\": {\"type\": \"SHOP_NOW\"}
    }
  }" \
  -d "access_token=$META_ACCESS_TOKEN"
```

### Step 4 — Create Ads and Activate

```bash
# Create ads linking creatives to their ad sets
curl -s -X POST "https://graph.facebook.com/v22.0/act_YOUR_AD_ACCOUNT_ID/ads" \
  -d "name=Product | Angle A | Ad 1" \
  -d "adset_id=ADSET_ID" \
  -d "creative={\"creative_id\":\"CREATIVE_ID\"}" \
  -d "status=PAUSED" \
  -d "access_token=$META_ACCESS_TOKEN"

# Then activate in order: Campaign → Ad Sets → Ads
```

---

## Ad Copy Framework (PAS)

**Headline:** Angle-specific hook. Never mention the product name.

**Primary Text:** Problem → Agitation → Solution, tailored to the angle's pain point or dream.

**Description:** `⭐ [Rating] Stars · Free Shipping`

**Example — Pain Angle:**
- Headline: "Does your dog destroy furniture when alone?"
- Text: Problem (separation anxiety) → Agitation (cost of damage) → Solution (product keeps them engaged)

**Example — Dream Angle:**
- Headline: "Give your high-energy dog the workout they need"
- Text: Desired outcome (tired, happy dog) → Agitation (current guilt) → Solution (indoor exercise)

---

## Stop-Loss Protocol (Creative-Level Only)

Evaluate performance at the **individual creative level**, never at campaign or ad set level.

| Condition | Action |
|-----------|--------|
| Spend ≥ 1.5× COGS, 0 purchases | Toggle OFF this specific creative |
| CTR > 3%, 0 purchases | Toggle OFF (traffic not converting) |
| All 3 creatives in one ad set fail | Consider pausing that angle's ad set |
| All 9 creatives fail | Pause entire campaign |

**Fetch creative-level insights:**

```
GET /{campaign-id}/insights?level=ad&fields=ad_id,ad_name,spend,ctr,actions
```

---

## Settings Reference

| Setting | Value |
|---------|-------|
| Bid strategy | `LOWEST_COST_WITHOUT_CAP` |
| Targeting | US only, Advantage+ enabled |
| Start time | Tomorrow at 08:00:00 UTC |
| Campaign status on create | `PAUSED` |
| Activation order | Campaign → Ad Sets → Ads |

---

## Common Mistakes to Avoid

1. Varying copy within an ad set — copy must be identical per angle
2. Naming angles generically ("Angle 1") instead of descriptively
3. Skipping angle research — generic angles produce generic results
4. Making angles that are too similar — each must target a truly distinct hook
5. Evaluating stop-loss at campaign level instead of creative level

---

## Security

- Store `META_ACCESS_TOKEN` in environment variables
- Never hardcode ad account IDs, page IDs, or pixel IDs in shared skill files
- Use `$META_ACCESS_TOKEN` references in scripts, sourced from your config
