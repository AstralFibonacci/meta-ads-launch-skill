# meta-ads-launch-skill

An agent skill for launching angle-based Meta (Facebook & Instagram) ad campaigns programmatically via the Graph API. Implements a CBO structure with 3 Ad Sets × 3 Creatives, each targeting a distinct pain point or dream outcome. Compatible with Claude Code, OpenClaw, and any agent framework that supports skill files.

## What it does

- Research and define 3 distinct angles per product
- Create CBO campaigns with angle-based ad set structure
- Upload images and generate ad creatives per angle
- Activate campaigns and apply creative-level stop-loss logic

## When to use this skill

- An agent needs to autonomously launch a Meta ad campaign for a product
- You want to systematically test multiple messaging angles in one campaign
- You're building an e-commerce automation pipeline: product → angles → ads → activation

## Requirements

| Requirement | Details |
|-------------|---------|
| Meta Business account | [business.facebook.com](https://business.facebook.com) |
| Access token | `META_ACCESS_TOKEN` env var |
| Ad account | Your ad account ID |

## Structure

```
meta-ads-launch-skill/
├── README.md     ← you are here (overview)
└── SKILL.md      ← full implementation guide for agents
```

## Campaign structure

```
1 Campaign (CBO)
├── Ad Set 1: Angle A (Pain Point / Dream Outcome)
│   ├── Creative 1A, 2A, 3A
├── Ad Set 2: Angle B
│   ├── Creative 1B, 2B, 3B
└── Ad Set 3: Angle C
    └── Creative 1C, 2C, 3C
```

See [`SKILL.md`](./SKILL.md) for the full step-by-step implementation.

## Tags

`ai-agent` `meta-ads` `facebook-ads` `instagram-ads` `advertising` `cbo` `automation` `e-commerce` `claude` `openclaw` `agent-skill` `marketing-automation` `graph-api`
