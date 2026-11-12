---
name: outreach-copy
description: Writes short cold email copy from outreach-brief.md. Produces two first-email options (a one-line open question and a short value email with a soft call to action), a value or proof follow-up, a breakup follow-up, and LinkedIn messages if the brief uses LinkedIn. Each piece stays in its own word range (25 to 80 words). Use after outreach-interview and before outreach-sequence. Works without Salesforge.
---

# Outreach copy

Needs: `outreach-brief.md` (run `outreach-interview` if missing). Works without any Salesforge connection.

## Always
- Simple, human, not formal. Write like one person to one person.
- Never invent proof. Use only the proof in the brief. If the brief says "no proof yet", skip option B and write option A only.
- No links of any kind in cold emails, and no booking link. The booking link is only for replies to interested prospects.
- No unsubscribe link. An optional closing line is fine: "Not for you? Just say so." Ask the user whether to include it (see Questions). If included, put it on email 1 only.
- No spam words in cold emails: free, guarantee, act now, limited time, risk-free, urgent, discount. If the offer is a free trial or free plan, leave it out of cold emails and write "keep it for replies to interested prospects" in the notes. Never word it around the ban ("no-cost", "complimentary").
- Also avoid these words and phrases, because the sequence spam check flags them: get, lose, stop, here, leave, now, off, rate, marketing, problem, all, sales, today, new, for you. Write around them from the start, for example "software for B2B teams" instead of "marketing software", "38% of people replied" instead of "reply rate", "reach" instead of "get", "calls with prospects" instead of "sales calls", "these days" instead of "today". The spam check also flags other words now and then; the sequence skill rewords those.
- Never mention other companies' tools.
- Spintax is added later by the sequence skill. Write clean text.
- Variables: only `{{first_name}}` and `{{company}}` are safe, because every imported lead has them. `{{localized_greeting}}` is built in. Use job title, location or other fields only if `list_custom_variables` shows the variable and the leads actually have values.
- Personalization lines must stay true for every lead on the list. A lead list is never 100% on target, so write about the role or situation the whole list shares (founders of small teams, people who run outbound), not about what each company sells. "Noticed {{company}} sells software for B2B teams" is false for part of the list; "Running {{company}} with a small team usually means outreach fights for time" is true for all of it.
- Word counts include the greeting line and variables. Show the count under each version.

## Questions
Ask both in one message, with letter answers, so the user answers once ("A, B"):
1. Personalization mode:
   - **A. Localized greeting (default):** every email starts with `{{localized_greeting}} {{first_name}},`. `{{localized_greeting}}` is a built-in Salesforge variable that greets the prospect in their own language.
   - **B. Full AI email:** the user later switches on "Generate with AI" on the email in the Salesforge sequence builder, and Salesforge writes each email per contact, with this copy as the fallback. The tools cannot switch this on, so the sequence skill tells the user where to click.
2. Closing line on email 1: A add "Not for you? Just say so.", B no closing line.

## What to write
Write all of these for the brief's one avatar. Option A and option B are both written (if the brief has proof). The sequence skill asks the user which one becomes email 1.

**First email, option A: the open question.** 25 to 40 words. The sequence spam check rates anything under 25 words as "ok" at best, so never go below 25. A greeting, one short personalization line, and one open question that starts a conversation. No pitch, no link.
Shape: "{{localized_greeting}} {{first_name}}, [one line about their role or situation]. [one open question about the offer's topic]?"

**First email, option B: the short value email.** 50 to 80 words. Only if the brief has proof. Three parts:
1. A personalization line.
2. One value line: "We helped [company type] reach [specific result] by [what the product does]." Use a real result from the brief. If the user has no number, say what the result was without a number and do not invent one. If the proof company is not the same type as the avatar, do not write "a company like yours": say who it was ("an outbound agency") and what exactly happened ("one client campaign"), and copy the brief's limits. Never imply the avatar got the result.
3. A soft call to action: "Want to hear more?" or "Would this be interesting to you?"
Also give a fill-in version of option B, with only the bracketed parts to replace.

**Follow-up 1 (value or proof).** 40 to 70 words. Never "just following up". Either one useful, specific tip about the avatar's problem that the reader can use without buying anything, or a proof line that option B does not already use (option B may become email 1, so never repeat its result). With no proof, always the tip.

**Follow-up 2 (breakup).** 25 to 40 words. Either ask to be redirected ("If I reached the wrong person, who should I talk to?") or give one more piece of value and close the loop politely.

## Subjects
Short, plain, lowercase style, 2 to 5 words, no clickbait, no all caps, no fake "re:" or "fwd:". Give two per first-email option. Follow-ups have no subject: they stay in the same thread.

## LinkedIn copy
Only if the brief's channels include LinkedIn:
- A message for after the connection request is accepted: 20 to 40 words, no pitch, plain text.
- An InMail (subject plus 40 to 70 word body, same tone), only if the brief says the account has InMail credits. If it says no, skip it and say why in one line.
The connection request itself is sent blank, so it needs no copy.

## Self-check before saving
For every piece: word count in range, no banned word (check whole words), no link, only safe variables, personalization true for an off-target lead. Fix anything that fails before showing it.

## Output
Save `outreach-copy.md` with:
- Every version labelled, its word count, and a one-line note on when to use it.
- The two answers to the questions above.
- A short notes section for the sequence skill: follow-ups stay in the same thread with an empty subject, no fake "re:", the closing line goes on email 1 only (if chosen), and the free trial (if any) is kept for replies.

Tell the user the next skill is `outreach-sequence`. Then stop. Do not start it.
