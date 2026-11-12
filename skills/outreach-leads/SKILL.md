---
name: outreach-leads
description: Builds a lead list from outreach-brief.md using Leadsforge through the Salesforge MCP. Picks the best search method for the avatar (B2B people search, local businesses, lookalike companies, company followers), counts and previews for free, shows the credit cost, then finds emails (and LinkedIn profiles if the brief uses LinkedIn), imports the contacts into the Salesforge workspace with a tag, and validates the emails. Use after outreach-interview.
---

# Outreach leads

Needs: `outreach-brief.md` (with the Salesforge workspace ID; run `outreach-interview` if missing) and the Forge MCP connected (`get_me` works).
If no `leadsforge_*` tools are in the tool list, the MCP connection has no Leadsforge key. Stop and tell the user: create a key in Leadsforge (Settings → API), add it to the Forge MCP connection as the `X-Leadsforge-Key` header, and restart Claude Code. If they have two Salesforge connections, they should keep only the one with all their keys.

## Always
- Work only in the workspace named in the brief. Never touch another workspace.
- Free: `leadsforge_count_search_results`, `leadsforge_search` previews, `leadsforge_preview_lookalikes`, and the `leadsforge_get_*_filters` tools. Everything else costs credits: `leadsforge_search_lookalikes` (per company), `leadsforge_search_company_followers` (per follower), `leadsforge_search_local_businesses` (per business), `leadsforge_enrich_business_owners` (per owner found), finding emails, finding LinkedIn profiles and validating. Show the cost and wait for a clear yes before any of them.
- Tell the user once that each tool call may show a permission prompt, and that counting and previewing people are free.
- Aim for at least 1,000 leads. If the avatar cannot reach 1,000, say so and offer to widen one filter at a time (region first, then company size). Never widen the role or the problem, because that makes the list generic.
- Keep only emails that pass validation. Never guess an email address.
- Never mention other companies' tools.

## Step 1: pick the method from the brief
- People at companies (roles, industries, sizes): `leadsforge_search`.
- Local or street-level businesses (dentists, salons, restaurants): `leadsforge_search_local_businesses`, then `leadsforge_enrich_business_owners`.
- The user named companies they want more of: `leadsforge_preview_lookalikes` (free), then `leadsforge_search_lookalikes`, then people at those companies with `leadsforge_search` and `companyDomains`.
- The audience follows a certain company page: `leadsforge_search_company_followers`.
Say which method you chose and why, in one sentence.

## Step 2: filters
Build filters from the brief. If the brief is silent on something, use these defaults and say so:
- Company size up to 50 employees.
- Countries: United States, United Kingdom, Canada, Australia, Ireland, New Zealand.
- Seniority: owner, founder, c_suite, director, vp, head, manager. Leave out interns and junior staff, who cannot decide.
- Always add `leadJobTitles` from the brief's role (for example owner, founder, CEO, head of sales) on top of seniority, because seniority alone lets in roles that cannot buy (for example content managers). Add title excludes for roles that look similar but do not buy: CTO, CPO, fractional, consultant, coach.
- Use `companyKeywords` for free-form descriptors and `companyIndustries` only for exact industry names (lowercase, standard industry list). Use several specific keywords, not one broad phrase: a broad phrase lets in unrelated industries. Add `companyBusinessModels` (for example `b2b`) when the brief names one.
- If the brief's buying intent is fresh funding, offer `companyFundingRounds` as an optional extra filter.
- Set `maxContactsPerCompany: 1`, so one company is never contacted twice.
- Exclude the never-contact domains and the primary domain with `excludeDomains`.

Exact filter shape (the tool rejects plain lists and size buckets like "11-50"):
```
leadJobTitles: {include: ["founder","CEO"], exclude: ["CTO","consultant"]}
leadSeniorities: {include: ["owner","founder","c_suite"]}
leadLocations: {include: ["United States"]}
companyEmployeeNumberRange: {min: 2, max: 50}
companyIndustries: {include: ["software development"]}
companyBusinessModels: {include: ["b2b"]}
companyKeywords: {include: ["marketing automation","email marketing"]}
maxContactsPerCompany: 1
excludeDomains: ["competitor.com"]
```

## Step 3: count and preview
1. `leadsforge_count_search_results` with the filters. Show the count.
2. `leadsforge_search` with the same filters and `limit: 10`. Read each lead's job title and company keywords, and say "about N of 10 fit the avatar", naming the misses in a few words (for example "a dev agency, a recruiting firm").
3. If fewer than 6 of 10 fit, tighten (more title excludes, more specific keywords, an industry or business model, keyword excludes), count and preview again. At most two tightening rounds. Never let tightening drop the count under 1,000.
4. After that, give the user the choice in one message: A use the best version so far (say its count and fit, for example "1,200 leads, about 5 of 10 fit"), or B describe the companies differently and try once more. Keyword data is sparse, so 4 to 6 of 10 is a normal fit; validation and a personalization line that stays true for everyone handle the rest.
Local businesses, lookalikes and company followers have no free count. Plan the search (a centre point and radius per city plus categories from `leadsforge_get_maps_category_filters`; or the lookalike companies from `leadsforge_preview_lookalikes`; or the company page and follower filters), then go to step 4 and get a yes before running it. Local search: if one search returns fewer businesses than its limit, split into smaller circles and de-duplicate by business ID.

## Step 4: show the cost, ask
Call `leadsforge_get_balance`. It returns `availableCredits`, `emailEnrichmentPrice` and `linkedinEnrichmentPrice` (credits per person).
- Email finding: leads × `emailEnrichmentPrice`.
- LinkedIn profile finding, only if the brief's channels include LinkedIn: leads × `linkedinEnrichmentPrice`.
- Lookalike companies: companies × `companyLookalikePrice`. Company followers: followers × `companyFollowersPrice`. Local businesses: up to the search `limit` per search, plus owners found.
- Validation (step 6) uses a separate validation credit pool, about 1 per email.
- Local businesses are charged per business found and per owner found; owner jobs reserve credits up front and charge nothing when no owner is found. Small businesses often have no owner email, so expect fewer contacts than businesses.
Credits are reserved up front and only the found results are charged, so the real cost is usually a bit lower.
Say: "This will use up to N Leadsforge credits (emails X, LinkedIn Y) of your M, plus about Z validation credits. Go ahead?" Wait for yes.

## Step 5: get the leads and enrich them
1. `leadsforge_search` with the approved filters and `limit` up to 2,000 per call; pass the returned `cursor` for the next page. Keep each lead's `id`.
2. `leadsforge_enrich_emails` with `personIDs` (up to 500 per call) and an `idempotencyKey` per batch, so a retry never charges twice.
3. If LinkedIn is in the brief: `leadsforge_enrich_linkedin` with the same `personIDs` batches and its own `idempotencyKey`.
4. Both are asynchronous. Poll `leadsforge_get_enrichment_job` until `status` is `completed`, then read `leadsforge_get_enrichment_results` with `jobID`, `limit: 100` and `offset` for the next page.
5. Each result row has `personID`, `status`, `payload` (`email` or `linkedinURL`) and `person` (`firstName`, `lastName`, `jobTitle`, `company` name, `location`). Join the email and LinkedIn results by `personID` into one row per person.
6. Local business flow is asynchronous too: poll the maps search job, read businesses, then run the owner job (one at a time).
Enrichment does not verify emails. Do not call them verified before step 6.

## Step 6: import and validate
1. Tag name: the avatar in a few words plus today's date, for example `us-b2b-saas-founders-2026-11-11`.
2. `bulk_create_contacts` with the workspace ID, 100 contacts per call. Each contact: `firstName` (required; skip a row without one), `lastName`, `email`, `company`, `position` (job title), `linkedinUrl`, `tags: ["<tag name>"]`. Import people who have an email, and also people with only a LinkedIn URL if the brief uses LinkedIn.
3. `list_tags` with `search: "<tag name>"` to read the tag ID (`ltg_...`).
4. `start_email_validation` with `filters: {"tagIds": ["<tag ID>"], "hasEmail": true}`. Keep the returned `validationJobID`. Poll `get_validation_results` with `runId` set to that ID until it is `completed` or `partially_completed`. Never resubmit a failed run automatically; tell the user.
5. Report the split from `totals.byStatus` (the `summary` part breaks it down by email provider; you do not need it). Keep `safe` and `catch_all`. Say that `catch_all` means the company domain accepts all mail, so it carries more bounce risk; small company domains are often `catch_all`. The other statuses (`invalid`, `disabled`, `disposable`, `spamtrap`, `role_account`, `inbox_full`, `unknown`) are not enrolled. The results spell the spam trap status `spam_trap`, but filters (in the sequence skill) spell it `spamtrap`. Contacts with only a LinkedIn URL show as `linkedin_only`.
6. Add the never-contact domains to the workspace with `add_dnc_entries`, so they are never contacted even if they are imported later.

## Personalization data
No per-lead writing is needed. The sequence uses the built-in `{{localized_greeting}}` variable, or Salesforge "Generate with AI" if the user turns it on. Job title and company are stored on each contact so AI personalization has data to work with.

## Step 7: report
Save `leads-summary.md`: method, final filters, count found, enriched (emails found, LinkedIn found), imported, the validation split, credits used (Leadsforge and validation), the tag name **and tag ID**, the validation run ID, and which statuses to enroll (`safe`, `catch_all`, and `linkedin_only` if LinkedIn is used). Tell the user the next skills: `outreach-copy`, then `outreach-sequence`. Then stop. Do not start them.
