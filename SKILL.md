---
name: meta-ads-launch-skill
description: Launch angle-based Meta (Facebook/Instagram) CBO ad campaigns via the Graph API. Use when an agent needs to autonomously create and activate a product campaign with 3 ad sets × 3 creatives, each targeting a distinct angle.
tags: [meta-ads, facebook, instagram, cbo, advertising, automation, e-commerce, graph-api]
---

# Meta Ads Launch Skill

## Purpose

Give agents the ability to create, configure, and activate Meta ad campaigns using the Graph API — structured around distinct product angles for systematic message testing.

---

## Campaign Structure

```
1 Campaign (CBO, daily budget)
├── Ad Set 1: Angle A (Pain Point or Dream Outcome)
│   ├── Creative 1A (Image 1 + Angle A copy)
│   ├── Creative 2A (Image 2 + Angle A copy)
│   └── Creative 3A (Image 3 + Angle A copy)
├── Ad Set 2: Angle B
│   └── Creative 1B, 2B, 3B
└── Ad Set 3: Angle C
    └── Creative 1C, 2C, 3C
```

**Rules:**
- Within an ad set: identical copy across all 3 creatives (only images differ)
- Between ad sets: copy targets a different angle
- CBO handles budget distribution automatically

---

## Phase 1: Angle Research

Before creating anything, identify 3 distinct angles for the product.

### What is an angle?

An angle = a specific **pain point** or **dream outcome** the product addresses.

**Examples:**

| Product | Angle 1 (Pain) | Angle 2 (Pain) | Angle 3 (Dream) |
|---------|---------------|---------------|-----------------|
| Puzzle Feeder | Fast eating / digestion | Boredom / destruction | Mental stimulation |
| Snuffle Mat | Anxiety / stress | Rainy day energy | Nose work enrichment |

**Angle prompt for the agent:**

```
Analyze this [product] and identify 3 distinct angles (pain points or dream outcomes):

Product data: [description, reviews, images]

Output:
Angle 1: [Name] — Type: [Pain/Dream] — Hook: [one sentence]
Angle 2: [Name] — Type: [Pain/Dream] — Hook: [one sentence]
Angle 3: [Name] — Type: [Pain/Dream] — Hook: [one sentence]
```

**Name angles descriptively:**
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

### Step 2 — Create 3 Ad Sets (one per angle)

**Naming:** `[Product] | [Angle Name]`

**Scheduling:** Always set `start_time` to tomorrow at 08:00:00 UTC. No end time.

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

### Step 3 — Create Ad Creatives (9 total)

3 per angle. Same copy within ad set, different images.

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

### Step 4 — Create Ads (9 total)

Attach creatives to their respective ad sets.

```bash
curl -s -X POST "https://graph.facebook.com/v22.0/act_YOUR_AD_ACCOUNT_ID/ads" \
  -d "name=Product | Angle A | Ad 1" \
  -d "adset_id=ADSET_ID" \
  -d "creative={\"creative_id\":\"CREATIVE_ID\"}" \
  -d "status=PAUSED" \
  -d "access_token=$META_ACCESS_TOKEN"
```

### Step 5 — Activate

Activate in this order: Campaign → Ad Sets → Ads

---

## Ad Copy Framework (PAS)

**Headline:** Angle-specific hook. Never mention the product name directly.

**Primary text:** Problem → Agitation → Solution, targeted to the angle.

**Description:** `⭐ X Stars · Free Shipping` (pull from reviews)

**Pain angle example:**
- Headline: "Does your dog destroy furniture when you're away?"
- Text: Problem (separation anxiety) → Agitation (cost, stress) → Solution (product as fix)

**Dream angle example:**
- Headline: "Give your high-energy dog the workout they need"
- Text: Desired outcome → Current frustration → Solution

---

## Stop-Loss Protocol (Creative-Level Only)

Evaluate at the **individual creative** level — never at campaign or ad set level.

| Condition | Action |
|-----------|--------|
| Spend ≥ 1.5× product COGS, 0 purchases | Toggle OFF this creative |
| CTR > 3%, 0 purchases | Toggle OFF (traffic not converting) |
| All 3 creatives in one ad set fail | Consider pausing that ad set |
| All 9 creatives fail | Pause campaign |

**Fetch creative-level insights:**

```
GET /{campaign-id}/insights?level=ad&fields=ad_id,ad_name,spend,ctr,actions
```

---

## Settings Reference

| Setting | Value |
|---------|-------|
| Bid strategy | `LOWEST_COST_WITHOUT_CAP` |
| Targeting | US, Advantage+ enabled |
| Start time | Tomorrow at 08:00:00 UTC |
| Initial status | `PAUSED` |
| Activation order | Campaign → Ad Sets → Ads |

---

## Common Mistakes

1. Varying copy within an ad set — keep it identical per angle
2. Generic angle names ("Angle 1") instead of descriptive ones
3. Skipping angle research — generic angles = poor results
4. Overlapping angles — each must target a truly distinct hook
5. Stop-loss at campaign level instead of creative level

---

## Security

- Store `META_ACCESS_TOKEN` in environment variables
- Never hardcode ad account IDs, page IDs, or pixel IDs in shared files
- Reference credentials via env vars: `$META_ACCESS_TOKEN`
