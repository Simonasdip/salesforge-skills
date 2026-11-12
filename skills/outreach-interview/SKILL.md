---
name: outreach-interview
description: Interviews you step by step and writes outreach-brief.md (offer, one avatar, one problem, proof, channels, volume, domain, Salesforge workspace). Use this first, before finding leads, setting up mailboxes, writing copy or building a sequence. Use it when someone wants to start cold outreach, fix outreach that did not work, or scale a small outreach system.
---

# Outreach interview

Goal: write `outreach-brief.md` in the current folder. Every other skill in this pack reads that file, so the answers here decide the quality of everything after.

## Always
- Ask one step at a time. Put all questions of one step in one message, numbered, so the user can answer in one reply. Give one short example per question. Wait for the answer before the next step.
- If an answer is missing or unclear, ask only that question again before moving on.
- Keep the avatar narrow: one avatar, one problem. If an answer is broad ("small businesses", "marketers"), say it is too broad and ask for one slice, for example one role in one industry in one region.
- Never invent an answer. If the user is unsure, offer two or three options and let them pick.
- Never mention other companies' tools or products.
- Steps 1 to 5 and the brief work without any Salesforge connection.

## Step 1: starting point
Ask which of these fits:
- A. I have an offer but no outbound system yet.
- B. I have a small outbound system and want to scale it.
- C. I tried outbound and it did not work.

If B: in the same follow-up message, ask current volume per day, channels in use, reply rate if known, and what limits them.
If C: ask what they tried (volume, where the list came from, the copy, domains and mailboxes used), the result, and what they think went wrong. Record it under "What happened before". Look for the usual causes: audience too broad, weak offer, sending from the primary domain, no warm-up, bad data.

## Step 2: the offer
Ask:
1. What do you sell, in one sentence?
2. Who is it for?
3. Price: if there are several plans, give each plan's price, billing period (monthly or yearly) and its main limits. Never combine the cheapest price with a higher plan's features in one sentence.
4. The one result a customer gets.
5. The first step for a new customer (call, demo, trial). If it is a free trial or free plan, note it: it is kept for replies, not for cold emails.

## Step 3: the avatar (exactly one)
Ask:
1. Job title or role, and seniority.
2. Company type and industry.
3. Company size (employees).
4. Country or region.
5. The one problem this person has, and how it shows up day to day.
6. Buying intent: what would tell you this person needs a solution right now (hiring, new funding, a new location, a bad review, a new tool).
7. What they have tried to solve it: alternatives, workarounds, or doing nothing.
8. Why now.

Then check: "Could you describe this person to a stranger in one sentence?" If not, narrow it down before moving on.

## Step 4: proof
Ask:
1. Real proof only: results with numbers, case studies, customer names you may use. If there is none, write "no proof yet". The copy skill then uses the open-question style and claims nothing.
2. Links to the product docs, pricing page, FAQ and case studies. They go under "Product links". The replies skill reads them to answer questions.

## Step 5: how you reach them
Ask:
1. Channels: email only, or email plus LinkedIn. LinkedIn needs a LinkedIn account connected in Salesforge.
2. If LinkedIn: does that account have InMail credits (a paid LinkedIn plan)? If not, the sequence skips InMail.
3. Primary domain: the main website domain. It is never used for cold sending.
4. Daily volume goal, in cold emails per day. If unsure, suggest 30 emails per mailbox per day on 2 to 3 mailboxes.
5. Existing cold email mailboxes, if any (addresses).
6. Time zone of the prospects. Write it as an IANA name, for example `America/New_York` or `Europe/London`. If the user says "EST" or "London time", convert it and confirm.
7. Booking link, if you have one. It is used only when replying to interested prospects, never in a cold email. If there is none, the replies skill offers two time slots instead.
8. Never-contact list: competitors and existing customers, as domains.

## Step 6: Salesforge workspace
Call `get_me`.
- If it works, call `list_workspaces` (`limit: 100`, then `offset` for more). Show only the workspace names, numbered. Ask which one to use, and recommend a fresh, empty workspace for a first run, because the other skills create contacts, tags and a sequence there. If the user wants a new one, tell them to create it in Salesforge (or offer `create_workspace` with the name they choose, after a yes), then list again.
- Write the chosen workspace name and its ID (`wks_...`) into the brief. Every later skill needs the ID.
- If `get_me` fails, say that the Forge MCP server (`https://mcp.salesforge.ai/mcp`) is not connected, that this brief and the copy skill still work, and that the leads, infrastructure, sequence and replies skills need it. Write "not connected yet" in the workspace section.

## Step 7: write the brief
Save `outreach-brief.md` using this structure, in the user's own words:

```
# Outreach brief
## Situation
(A, B or C) + What happened before
## Offer
One sentence / who it is for / result / first step
## Pricing
Plan: price, billing period, main limits (one line per plan)
## Avatar
Role / seniority / company type + industry / size / region
## Problem
## Buying intent
## Alternatives they tried
## Proof
(or "no proof yet")
## Product links
## Channels
Email only | Email + LinkedIn (InMail credits: yes/no)
## Primary domain
(never used for cold sending)
## Daily volume goal
## Existing mailboxes
## Time zone (IANA)
## Booking link
(replies only; or "none")
## Never contact (domains)
## Salesforge workspace
Name (wks_...)
```

Read back a five-line summary and ask for corrections. Then tell the user the order of the next skills: `outreach-infrastructure` first if they need mailboxes (warm-up takes at least 14 days, so start it now), then `outreach-leads`, `outreach-copy`, `outreach-sequence`, and `outreach-replies` once replies arrive.

Then stop. Do not start the next skill. The user runs each skill separately, when they are ready.
