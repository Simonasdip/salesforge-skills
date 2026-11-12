# Claude skills for outreach

Six Claude Code skills that take you from "I have an offer" to a running cold outreach campaign in Salesforge: a written brief, a lead list, sending domains and mailboxes, short cold emails, an email-first sequence with a LinkedIn fallback, and replies drafted for your approval.

## The skills, in order

| # | Skill | What it does | Needs Salesforge? |
|---|---|---|---|
| 1 | `outreach-interview` | Asks you a few questions per step and writes `outreach-brief.md`: offer, pricing, one avatar, one problem, proof, channels, volume, domain, Salesforge workspace | Only for the last step (picking the workspace) |
| 2 | `outreach-infrastructure` | Sizes your domains and mailboxes, finds lookalike domains, buys them only after you approve the exact list and total, gets them into your workspace, and checks warm-up | Yes |
| 3 | `outreach-leads` | Picks the best Leadsforge search for your avatar, counts and previews for free, shows the credit cost, finds emails (and LinkedIn profiles), imports and validates them | Yes |
| 4 | `outreach-copy` | Writes short first emails, a value follow-up, a breakup email and LinkedIn messages, already clear of common spam words | No |
| 5 | `outreach-sequence` | Builds an email-first sequence with a LinkedIn fallback as a draft, runs the spam check on every email, then asks before enrolling and again before launching | Yes |
| 6 | `outreach-replies` | Reads replies (email and LinkedIn), drafts answers for your approval, learns your style, keeps a list of "check back later" contacts | Yes |

Order in real life: run 1, then start 2 right away (new mailboxes need at least 14 days of warm-up). Do 3, 4 and 5 while they warm up. Run 6 when replies arrive. Each skill stops when it is done; you start the next one yourself.

## Set up

1. Install Claude Code.
2. Install the skills. In Claude Code, run these two commands:
   ```
   /plugin marketplace add Simonasdip/salesforge-skills
   /plugin install salesforge-outreach@salesforge-skills
   ```
   Then restart Claude Code. The skills show up as `/salesforge-outreach:outreach-interview` and so on, and you can also just ask for them in plain words.

   Without the plugin: click **Code → Download ZIP** on this page, unzip it, and copy the six folders inside `skills/` into your Claude skills folder (`~/.claude/skills/` on Mac and Linux, `%USERPROFILE%\.claude\skills\` on Windows). Restart Claude Code.
3. Connect the Forge MCP server (needed for skills 2, 3, 5 and 6):
   - Salesforge MCP and API access are part of the **Growth** plan (check the pricing page for your plan).
   - Create an API key in each product you use: open the product, then **Settings → API**, and generate a key.
   - Add the server, with one header per product you use. Leave out products you do not use; their tools will not appear.
     ```
     claude mcp add --transport http salesforge https://mcp.salesforge.ai/mcp --scope user \
       --header "X-Salesforge-Key: YOUR_SALESFORGE_KEY" \
       --header "X-Leadsforge-Key: YOUR_LEADSFORGE_KEY" \
       --header "X-Mailforge-Key: YOUR_MAILFORGE_KEY" \
       --header "X-Warmforge-Key: YOUR_WARMFORGE_KEY" \
       --header "X-Infraforge-Key: YOUR_INFRAFORGE_KEY" \
       --header "X-Primeforge-Key: YOUR_PRIMEFORGE_KEY"
     ```
     On Windows (PowerShell or cmd), put the whole command on one line and remove the `\` characters.
   - Skill 3 needs the Leadsforge key, skill 2 the Mailforge (or Infraforge, Primeforge) and Warmforge keys.
   - Keep one Salesforge connection only. If you have two (for example one added in Claude Code and one added as a connector in the Claude app), Claude may pick the one without your Leadsforge or Mailforge tools.
4. Start with: "Run the outreach interview."

## What it costs
Nothing is bought or spent without your explicit yes to the exact amount. Before each step the skill shows the cost:
- **Leadsforge credits:** 1 credit per email found and 1 per LinkedIn profile found (prices are read live from your balance). Counting and previewing people is free. Lookalike companies (1 per company), company followers (1 per follower) and local business searches (per business and per owner found) also cost credits, and the skill asks before each.
- **Email validation credits:** about 1 per email, from a separate validation credit pool.
- **Domains:** the price per domain is shown before buying (Mailforge `.com` domains were $14 each when this was written).
- **Mailboxes:** billed by the mailbox provider. The tools do not return a mailbox price, so the skill asks you to check it on your billing page before you approve.

## What you do by hand (the skill tells you when, step by step, and waits)
- **Mailforge mailboxes into Salesforge:** no tool can move them, so you click **Export → Export to Salesforge** in Mailforge once (5 clicks). The skill gives the exact steps, waits for you, and checks that every mailbox arrived. Infraforge mailboxes are moved by the tools, and Primeforge mailboxes too when your Primeforge key is added.
- **Connecting LinkedIn:** connect your LinkedIn account on the sender profile in Salesforge. The skill never asks for your LinkedIn password.
- **AI-written emails (optional):** turning on "Generate with AI" on an email step is a switch in the sequence builder.
- **Your product answers:** the replies skill asks you once for your most common prospect questions, objections and anything it must never say.

## Safety rules built in
- Nothing is bought, enriched, validated, enrolled, launched or sent without your explicit yes.
- Cold email is never sent from your primary domain.
- The skills work only in the workspace you pick; use a fresh workspace for your first run.
- Lists that can show other workspaces or other mailboxes on your account are filtered, never printed.
- No launch until the mailboxes are warm (at least 14 days).
- Anyone who asks to stop is never emailed again and goes on your do-not-contact list.
- Replies are drafted for your approval; automatic sending is only switched on per reply type, by you.

## Troubleshooting
- **"The Forge MCP is not connected" / `get_me` fails:** check the server with `claude mcp list`, and that your Salesforge key is valid and your plan includes MCP access.
- **Leads, Mailforge or Warmforge tools are missing:** the MCP connection has no key header for that product. Add the header (see Set up) and restart Claude Code.
- **"Warm-up workspace not matched":** the warm-up tool could not find a workspace with exactly the same name as your Salesforge workspace. Check warm-up in Salesforge under **Mailboxes**.
- **Emails look like one block of text:** email bodies must be HTML paragraphs; the sequence skill writes them that way. Open the email step in the sequence builder to check.
