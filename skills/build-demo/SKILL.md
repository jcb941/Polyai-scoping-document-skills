---
name: build-demo
description: "Build a complete PolyAI prospect demo end-to-end: landing page, mock API, voice agent (via /prospect-demo), PLUS a dark-themed demo narrative page. Use when someone says 'build a demo for [company]', 'spin up a demo site', or 'create a prospect demo'. This is the full pipeline."
---

# Build Demo — Full Pipeline

This skill orchestrates the complete PolyAI prospect demo build. It chains two skills together:

1. **`/prospect-demo`** (from `PolyAI-LDN/oisin-nikola-store`) — builds the landing page, mock API, voice agent, and deploys to Railway
2. **`/demo-narrative`** (from this repo) — adds a dark-themed narrative page with customer profile, journey timeline, scenario cards with transcripts, and key moment callouts

## Before You Start — Check Dependencies

The `/prospect-demo` skill lives in a separate repo. Check if it's available. If NOT available, tell the user:

> The `/prospect-demo` skill is required but not installed. Run this to add it:
>
> ```bash
> claude skills add PolyAI-LDN/oisin-nikola-store
> ```
>
> Then re-run `/build-demo`.

Do NOT proceed without `/prospect-demo`. The landing page must be built first.

## Pipeline

### Step 1: Build the Landing Page + Mock API + Voice Agent

Invoke `/prospect-demo` with the company name and all context the user provided. This skill handles:
- Company research
- 4 call types identification
- Mock API with customer record
- PolyAI-branded landing page (BUILD-KIT spec — 9 sections or 360 multi-tab site)
- Studio voice agent prompts
- Railway deployment

Wait for `/prospect-demo` to complete fully — the landing page must be deployed and accessible before Step 2.

### Step 2: Build the Demo Narrative Page

Once the landing page is live, invoke `/demo-narrative` to generate `narrative.html`. This skill:
- Reads the deployed landing page to extract prospect name, use cases, brand colors
- Reads the mock API (`/api/customer/:phone`) to get the customer persona data
- Generates a dark-themed narrative page following the visual spec (see `/demo-narrative` SKILL.md)
- Adds a "Demo Narrative" nav link to the landing page
- Saves `narrative.html` to the project's `public/` directory

### Step 3: Deploy the Update

After the narrative page is added:
- Push the updated project to Railway (`railway up`)
- Verify both pages are accessible:
  - Landing page: `https://<project>.up.railway.app/`
  - Narrative: `https://<project>.up.railway.app/narrative.html`

## What the User Needs to Provide

- **Company name** (required)
- Any known context: use cases, tech stack, meeting date, incumbent, deal stage
- Everything else is researched and generated automatically

## Output

A fully deployed Railway app with:
- `/` — PolyAI-branded landing page (prospect's brand + PolyAI accent)
- `/narrative.html` — dark-themed demo narrative (customer profile, scenarios, transcripts)
- `/api/customer/:phone` — mock API for the voice agent
- `/api/service-request` — mock action endpoint
- `/healthz` — health check
- `docs/` — system prompt, builder prompt, demo script, mock API payload
