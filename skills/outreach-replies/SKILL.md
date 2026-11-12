---
name: outreach-replies
description: Reads Salesforge Primebox replies (email and LinkedIn), drafts answers for you to approve, learns your approved style into reply-playbook.md, keeps a follow-ups list for "not now" replies, and can send approved reply types automatically after you say so. Never replies to stop requests. Use after a sequence is live and replies are coming in.
---

# Outreach replies

Needs: the Forge MCP connected, `outreach-brief.md` (workspace ID, offer, proof, booking link if any, time zone, product links), and `product-knowledge.md` (step 0 creates it).

## Always
- First run is training. Draft every reply, show it, and send only what the user approves.
- Auto-send only reply types the user has switched to `auto` in `reply-playbook.md`. Never auto-send to Negative, Wrong contact, anything angry, or anything legal.
- Never reply to someone who asks to stop or to be removed. Label the thread Negative, add their email to the do-not-contact list with `add_dnc_entries` (`workspaceId`, `dncs: ["<email>"]`), and tell the user.
- Answer only from `product-knowledge.md`, the brief and the playbook. Never invent proof, numbers, prices, features or links. If something is not covered, put `[need answer: ...]` in the draft, tell the user what is missing, and after they answer, add it to `product-knowledge.md`.
- When quoting a price, say which plan and which billing period it is for.
- Never mention other companies' tools.
- Work only in the workspace named in the brief.

## Step 0: product knowledge (first run, or when something is missing)
1. Read `product-knowledge.md` if it exists, and the brief's "Pricing", "Proof" and "Product links".
2. Read each product link (pricing, docs, FAQ, case studies) and write the facts into `product-knowledge.md`: plans and prices per billing period, limits, trial, FAQ answers.
3. Ask the user only for what is still missing, one topic per message:
   1. The 5 to 10 questions prospects ask most, with the answer they give to each.
   2. The common objections (price, timing, "we already have something", "send me info") and how they answer each.
   3. Things the skill must never promise or say.
4. Save the answers under "User's answers" in `product-knowledge.md`, in the user's words.

## Step 1: read
1. Read `follow-ups.md` if it exists. List any follow-up whose date is today or earlier, and draft those first (as type "Follow-up due").
2. `list_primebox_labels` for the label IDs. Primebox has these labels: Positive, Negative, Out of office, Meeting booked, Meeting completed, Closed, Wrong contact.
3. `list_primebox_threads` with the `workspaceId`, `filter: "unread"`, and `limit: 50` (use `offset` for more). Salesforge labels replies by sentiment already.
4. `get_thread` for each thread before drafting. It returns the email and LinkedIn messages, the contact, the sequence, the mailbox, and for LinkedIn the sending account ID.

## Step 2: reply type and label
The reply type is your reading of the message. The Primebox label stays one of the seven real labels. Change a label with `update_thread_label` only when Salesforge's label is clearly wrong.

| Reply type (your reading) | Primebox label | What to do |
|---|---|---|
| Interested (wants to hear more or talk) | Positive | Reply fast. Move to a call: if the brief has a booking link, give it. If not, offer two specific time slots on different days in the brief's time zone, or ask for their preferred time. Never invent a link. |
| Question | Positive (or leave as labelled) | Answer plainly in two to four sentences from `product-knowledge.md`. Add the matching link from the brief's product links if there is one. End with one easy next step. |
| Objection (price, timing, already have something) | leave as labelled | Acknowledge it, answer with the user's own objection answer from `product-knowledge.md`, add one line of real proof only if the brief has proof, then one easy question. |
| Not now / later | leave as labelled | Thank them and ask when to check back (offer two options, for example "early January or later in Q1"). Write the contact, email, thread ID and the date to `follow-ups.md` (no reminder tool exists). |
| Wrong contact | Wrong contact | Thank them and ask who the right person is. |
| Out of office | Out of office | Do not reply. If the message gives a return date, add it to `follow-ups.md`. |
| Stop, unsubscribe, angry | Negative | Do not reply (see Always). |
| Meeting time agreed | Meeting booked | Confirm the time in one line. |

## Step 3: training run
For each thread, show: contact name and company, the message (short quote), the reply type, the label, and your draft. Ask: send, edit, or skip. Apply edits and show the final version before sending.

**Email threads:** send with `reply_to_email` (`workspaceId`, `mailboxId` from the thread, `emailId` of the message you reply to from `get_thread`, and `body`).
- The body is HTML: `<p>` paragraphs, with `<p><br></p>` for a blank line. Plain newlines show as one block of text.
- Signature: call `get_mailbox` once per mailbox and use its `resolvedSignature`. Put it after the body, separated by `<div>&nbsp;</div><div>--</div>`. If the signature is empty, sign with the sender's first name.
- Reply in the same thread; do not change the subject.

**LinkedIn threads:** send with `reply_to_linkedin_thread` (`workspaceId`, `threadId`, `accountId` from the LinkedIn messages in `get_thread`, `message`). Plain text only, no HTML, no signature. Keep it under about 60 words.

After sending, set the label with `update_thread_label` if it changed (for example Meeting booked once a time is set).

## Step 4: learn
After each approved reply, add it to `reply-playbook.md` under its reply type: the situation in one line, the reply the user approved, and any rule they gave ("always offer two time slots"). Each type has a mode, `ask` by default. Offer to switch a type to `auto` only after the user has approved five replies of that type without edits, and change it only when the user says yes.

## Step 5: later runs
Start with due follow-ups from `follow-ups.md`. Then: `auto` types are drafted from the playbook and sent, `ask` types are queued and shown for approval. Remove a follow-up from `follow-ups.md` once it has been sent or the contact replied.
End every run with three short lists: sent, waiting for approval, skipped (with the reason).

To check replies on a schedule, the user can run this skill on a repeat with Claude Code's `/loop` command. It only runs while Claude Code is open.
