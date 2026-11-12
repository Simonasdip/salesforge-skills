---
name: outreach-sequence
description: Builds an email-first Salesforge sequence from outreach-brief.md and outreach-copy.md, with an optional LinkedIn fallback, branching conditions, spam checks, sender profiles and a schedule. Builds it as a draft, then asks before enrolling contacts and asks again before launching. Use after outreach-copy, outreach-leads and warmed mailboxes.
---

# Outreach sequence

Needs: `outreach-brief.md` (workspace ID, IANA time zone, channels), `outreach-copy.md`, `leads-summary.md` (tag ID) for enrollment, and the Forge MCP connected.

## Always
- Build the sequence as a draft. Ask before enrolling contacts. Ask again before launching: a launch sends real email and LinkedIn actions.
- Work only in the workspace named in the brief. Never edit or delete existing sequences. Never change an existing sender profile's mailboxes or name; the only allowed edit is attaching a LinkedIn account to a profile that has none, after the user says yes.
- Never launch on an unchecked or unfinished warm-up (see Warm-up check).
- Never print lists from other workspaces (Warmforge returns workspaces from other accounts).
- Never mention other companies' tools.
- If a step cannot be done with the tools, give the user the exact in-app steps, wait for "done", check with a read-only call, and continue.

## Step 0: what already exists
`list_sequences` for the workspace. Give the new sequence a name no existing one uses. Read `list_sender_profiles` once: it shows the mailboxes and the `linkedinAccount` on each profile (needed for question 1).

## Questions (ask all four in one message, letter answers, e.g. "A, A, B, A")
1. LinkedIn fallback, if the brief's channels include LinkedIn: A yes, B email only. If no sender profile has a `linkedinAccount`, say so and offer: A build the LinkedIn path now (nodes can be built before an account is connected; sending needs it), B email only.
2. Senders: A all mailboxes in the workspace's sender profiles (default; warm-up is checked before launch), B pick mailboxes. A draft sends nothing, so mailboxes that are still warming are fine for building it.
3. InMail: take it from the brief ("InMail credits"). Ask only if the brief does not say. Without credits, the LinkedIn path ends after the withdrawal.
4. First email: A the open question (option A from the copy), B the value email (option B). If the copy file has no option B, skip this question and use A.

Everything else comes from the brief and the copy file.

## Structure
Email first. LinkedIn only when email gets no reply or there is no email. This protects the limited LinkedIn quota.

```
root
 condition 3 has_email_address
  Yes -> email 1 -> (wait 3d) email 2 -> (wait 3d) email 3
         -> condition 8 email replied within 3 days
              Yes -> end
              No  -> [LinkedIn path copy 1]   (if LinkedIn)
  No  -> [LinkedIn path copy 2]               (if LinkedIn; otherwise end)

LinkedIn path (each copy):
 condition 1 has_linkedin_url
  No  -> end
  Yes -> view profile -> (1d) follow profile -> (1d) connection request, blank
         -> condition 2 request accepted within 14 days
              Yes -> LinkedIn message (from the copy file)
              No  -> withdraw request -> InMail (only with InMail credits)
```

- Branches never merge, so the LinkedIn path is built twice.
- The `has_email_address` No branch only receives contacts without an email (`linkedin_only`). It is used when such contacts are enrolled (see Enrollment).
- Waits between emails: 2 to 4 days. LinkedIn steps may use 1 day. `has_email_address` and `has_linkedin_url` use `minutesToWait: 0`. Email 1, view profile, the LinkedIn message and the withdrawal use `waitDays: 0`.
- A blank connection request gets the best acceptance, so do not add a note.
- Other condition ids, only if the user asks: 5 `li_is_already_connected` (skip the request for existing connections), 4 `li_is_open_linkedin_profile`, 6 `if_email_bounced`, 7 `check_email_validation_status`, 9 `li_contact_replied_within_days` (put after the LinkedIn message to stop on a reply).

## Building it with the tools
`create_sequence_with_steps` cannot set waits per step, so build with the node tools:
1. `create_sequence` with `workspaceId`, the new `name`, and `timezone` from the brief (IANA, for example `America/New_York`).
2. `list_sequence_branches` (workspace, sequence). It returns each branch's `id`, `fromNodeId`, `toNodeId` and `name`. A branch with `"toNodeId": null` is an open slot to build on. The root has one branch; a condition's two branches are named `Yes` and `No`. (Use this, not `list_sequence_nodes`, which prints every email body.)
3. Build one level at a time. After each level, call `list_sequence_branches` again and take the open branches. Keep a small table as you go:

   | slot | branch id | next node |
   |---|---|---|
   | root | 1001 | condition 3 |
   | has_email Yes | 1002 | email 1 |
   | has_email No | 1003 | condition 1 (copy 2) |

   Nodes created in the same message get branch IDs in the order of the calls, but always confirm with `list_sequence_branches` before the next level.
4. `create_action_node` with `workspaceId`, `sequenceId`, `branchId`, `actionId`, `waitDays`, and `variants`. Action ids: 3 send_email, 1 connection request, 2 LinkedIn message, 4 view profile, 5 withdraw request, 6 like latest post, 7 follow profile, 8 InMail.
   - Email and LinkedIn message: `variants: [{"exposureInPercentage": 100, "isEnabled": true, "metadata": {"name": "A", "subject": "...", "message": "..."}}]`.
   - Blank connection request: `variants: [{"exposureInPercentage": 100, "isEnabled": true, "metadata": {"name": "A", "message": ""}}]`.
   - View profile, follow, like and withdraw take no `variants`.
   - `waitDays` is in **days**: pass `3`, not `4320`. Responses show it back as `waitInMinutes` (3 days = 4320).
5. `create_condition_node` with `workspaceId`, `sequenceId`, `branchId`, `conditionId` (3 has_email_address, 1 has_linkedin_url, 8 email replied within days, 2 request accepted within days) and `minutesToWait` (3 days = 4320, 14 days = 20160).
6. To change a saved node: `update_action_node` with the node ID, `wait_in_minutes` for the wait, and `variants` that include the existing variant `id`.

## Copy touches
- **Emails are HTML.** Write each body as `<p>` paragraphs with `<p><br></p>` between them for a blank line. Plain newlines (`\n`) show as one block of text in the Salesforge editor. LinkedIn messages are plain text: no tags.
- Spintax uses double braces with pipes: `{{Hi|Hello|Hey}}`. Variables also use double braces: `{{first_name}}`, `{{company}}`. Single braces are not recognized.
- Add spintax to almost every phrase, not just the greeting. Wrap whole phrases, for example `{{Running outreach|Handling outreach}} {{with a small team|on a small team}}`. The spam check rates the share of spintax and variables: aim for 80% or more ("good").
- Never nest spintax or put a variable inside spintax (`{{a|b {{company}}}}`). Keep `{{company}}` outside the braces.
- The subject of email 1 may use spintax too, with variables outside the braces, for example `{{outbound|outreach}} at {{company}}`.
- Keep each email between 25 and 100 words. Under 25 is rated "ok" at best.
- Avoid words the spam check flags: get, lose, stop, here, leave, now, off, rate, marketing, problem, all, sales, today, new, for you. Reword them. The check reads every spintax alternative, so keep flagged words out of every side of `{{a|b}}`. If the check flags a word not on this list, reword it and tell the user once which word it was.
- Never fake a reply in the subject ("re:", "fwd:"). Emails 2 and 3 use an empty subject so they stay in the same thread; the spam check skips an empty subject.
- The closing line ("Not for you? Just say so."), if the copy file chose it, goes on email 1 only.
- Personalization: the first line is `{{localized_greeting}} {{first_name}},`. If the copy file chose full AI emails, tell the user where to switch it on: in the Salesforge sequence builder, open each email step and turn on **Generate with AI**; the copy stays as the fallback. The tools cannot switch it on.
- Run `spam_check_sequence_node` on every **email** node (LinkedIn nodes have no spam check). Aim for "good" on every metric. If one is "ok" or "bad", fix the text, save with `update_action_node`, and check again. Usually one to three rounds. Ignore the recommendation "We recommend using plain text": the `<p>` HTML is needed for the editor, and plain-text sending is set in Settings.

## Settings
- `update_sequence_settings` with `settings: {"espMatchingEnabled": true, "plainTextEmailsEnabled": true, "openTrackingEnabled": false, "clickTrackingEnabled": false, "bounceShieldEnabled": true, "sequentialCompanySendingEnabled": true}`.
- `update_sequence_schedule` with `timezone` (IANA) and a `schedule` object for all seven days. Every day needs `enabled`, `from` and `to`, even disabled days: Monday to Friday `{"enabled": true, "from": 8, "to": 17}`, Saturday and Sunday `{"enabled": false, "from": 8, "to": 17}`.

## Senders
1. Use a sender profile that holds the chosen mailboxes. If none exists, create one with `create_sender_profile` (`name`, `mailboxIds`). Several profiles: `bulk_create_sender_profiles`.
2. If the sequence includes LinkedIn, the LinkedIn account must be on the same sender profile, or LinkedIn steps cannot send:
   - Account already connected but not on a profile: `update_sender_profile` with `updates: {"linkedinAccountId": <id>}`, or pass `linkedinAccountId` when creating the profile.
   - No account connected: never ask for the user's LinkedIn password. If the user chose to build the LinkedIn path now, finish the draft and list this as a to-do before launch. Before launch, tell them: in Salesforge open **Sender profiles**, open this profile, and connect LinkedIn there. Wait for "done", then check `list_sender_profiles` shows a `linkedinAccount` on the profile. Do not launch a sequence with LinkedIn steps until it does.
3. `assign_sender_profiles_to_sequence` with the profile IDs. It can return zero counts even when it worked, so check with `list_sequence_sender_profiles`.
4. Tell the user the daily email capacity: warm mailboxes × 30.

## Final check
`get_sequence` once: it returns the whole tree, settings and sender profiles. Branches are in its top-level `branches` array, not on each node. Disabled weekend days may show a different `to` hour; that is fine. Check only these points: every email has a body, every condition has two branches that end in a node or a deliberate end, waits are as planned, the sender profile is attached. Then show the tree in the Structure format with the real waits.
Ask the user to open the sequence in Salesforge once and look at email 1 in the editor, to see the paragraphs as the prospect will.

## Warm-up check (before launch)
1. Find the Warmforge workspace: `warmforge_list_workspaces` with `page: 1, page_size: 100`, and read further pages until `currentPage` equals `totalPages`. Match the `name` exactly to the Salesforge workspace name. **Never print the list.** Say only "matched" or "no match".
2. `warmforge_list_mailboxes` with that `workspaceId`, `page: 1`, `page_size: 100`, `status: "all"`. For the sequence's mailboxes, `warm` must be true and `warmupDaysLeft` 0.
3. No match, or a mailbox is missing or not warm: do not launch. Say which mailboxes are not ready and the earliest date, and tell the user to check warm-up in Salesforge under **Mailboxes**.

## Enrollment and launch
1. Tag ID: from `leads-summary.md`, or `list_tags` with `search: "<tag name>"`.
2. `preflight_enrollments` with:
   ```
   {"workspaceId": "...", "sequenceId": "...", "selectionScope": "not_in_sequence",
    "filters": {"tagIds": ["<tag ID>"], "validationStatuses": ["safe", "catch_all"]}}
   ```
   If the sequence has the LinkedIn path and the leads include LinkedIn-only contacts, run a second preflight with `"validationStatuses": ["linkedin_only"]`.
   Show the counts. Do not add counts across `moveGroups`. Say that counts can include do-not-contact or unsubscribed contacts, which are enrolled but get no outreach.
3. Ask: "Enroll these N contacts?" Only after yes, call `confirm_enrollment_preflight` with `workspaceId`, `sequenceId`, the returned `preflightId`, and `action: "skip"` (enrolls only contacts that need no decision; never moves contacts out of other sequences). A preflight expires after 15 minutes; if it has, run step 2 again.
4. Run the warm-up check, and check the LinkedIn account is on the sender profile if the sequence has LinkedIn steps. Then ask: "Launch the sequence now? This starts sending." Only after yes, call `launch_sequence`.

Save `sequence-summary.md`: sequence name and ID, structure with waits, settings, schedule, senders, spam check results, enrollment counts and status. Then stop. Do not start another skill.

The booking link is never put in a cold email or a LinkedIn message. It belongs only in replies.
