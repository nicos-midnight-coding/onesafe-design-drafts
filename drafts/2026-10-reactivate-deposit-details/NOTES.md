# Reactivate deposit details — notes

Ticket: DSN-62 · Source: Chris in Slack ("a way to see that an external account is deactivated and have them reactivate it")

## 1. Problem
When a recipient's bank account gets deactivated after returned payments, the client can't see why or fix it, so it lands on ops.

## 2. Prior art
- **Reuse (OneSafe):** our existing "this payment has failed to this person before" warning. Same slot, same pink warning box, now with an action.
- **Steal (Mercury):** inline warning on the recipient step with "verify their bank details" + Dismiss. We add the missing piece: a one-click reactivate.
- **New:** only the review modal (return history + confirm checkbox + Reactivate / Update details).

## 3. Deliverable size
LoFi-plus. Small surface change (banner + one modal + a pill), but it touches money movement and a state transition, so the states and copy are polished; layout is not.

## 4. Recommendation
**Soft warning, dismissable, with "View details" on the bank account as the way in.** Payer is never blocked. From the details modal they either confirm and reactivate, or update the details (which reactivates on save). If they ignore it and continue, sending the payment reactivates the account.

MVP
- Bank account row stays neutral (gray border, no status label); "View details" opens the modal
- Warning banner with Dismiss only, on send flow and recipient page (dismissed separately)
- Details modal: last returns (code, reason, amount, date), current details, confirm checkbox, Reactivate / Update details
- Update details: routing (9 digits) + full account number + type, bank chip from routing, blocks saving identical details
- Activity log entries: paused, returned, reactivated / updated
- Info box on Amount step if still paused: "We'll reactivate it when you send"
- Payroll flow (same pattern, same modal):
  - Payroll details (setup): no warning
  - Team members: warning banner above the list only while the paused member is selected; that row's status column shows a warning icon + "View details" instead of the "Active" pill
  - Pay amounts: warning inline under the paused member's row
  - Review: warning above the member list, adds "If you schedule anyway, we'll reactivate it when payroll runs"; same status cell as Team members
  - Dismiss is per screen, so the warning shows again on Review before money moves

V2
- Ask the recipient to confirm their own details (email link)
- Auto-suggest Update vs Reactivate from the return code (R03/R04 → nudge to Update; R01/R09 → Reactivate is fine)
- Admin/approver rules: who can reactivate

## 5. Dev rules
| Event | Result |
|---|---|
| Provider deactivates external account (returns ≥ threshold) | Status `paused`; warning banner on send flow and recipient page; activity "Account paused" |
| Payer clicks Dismiss | Banner hidden on that surface for this session only; status unchanged |
| Payer checks confirm + Reactivate | Status `active`; success banner; toast; activity "Reactivated by <user>" |
| Payer saves new details | New details become the active account; old one kept as inactive record; activity "Bank details updated" |
| Payer saves details identical to paused ones | Save disabled; inline hint to use Reactivate |
| Payer continues while paused | Allowed. Amount step shows info box; on Send, reactivate then submit |
| Paused member selected in payroll | Banner on Team members, inline on Pay amounts, banner on Review; untick member → banner disappears |
| Payroll scheduled with a paused member | Allowed. Reactivate that account when payroll runs, then pay |
| Payment returns again after reactivation | Back to `paused` (same rules) |
| Reactivation API fails | Keep `paused`, show error in modal, payment not submitted |

Sample data only: routing 110000000 is a test number.

## 6. Open items
- Payroll: should dismissing on Team members carry through to Review, or stay per screen (current)?
- Payroll: for a recurring payroll, does one reactivation cover all future runs?
- With no Paused label and no Review link, "View details" is the only path to reactivate. Check it's discoverable enough.
- **Bridge/Infinite:** can we reactivate an external account via API, or does "reactivate" mean creating a new external account with the same details? This decides the whole backend. (Erik / Jorge)
- Return threshold that triggers a pause: 1 or 2? Provider-driven or ours?
- Auto-reactivate on send: Chris to confirm he's OK with that vs requiring an explicit click.
- Do R02 (account closed) returns allow Reactivate at all, or only Update?
- Which roles can reactivate (all payers vs admins)?

## 7. Product angle
Turns a recurring ops ticket into a 10-second self-serve fix, and the return history builds trust ("we told you why"). The confirm checkbox gives us an audit trail if the next payment bounces too.
