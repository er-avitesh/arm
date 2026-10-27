# SolarEPC v23 — QA Findings Report (updated)

**Tested:** full Project lifecycle, live in a browser (React app + localStorage), logged in as `MohitKesharwani` (Admin).

**What I did:** Created a fresh demo project ("Rajesh Verma (Demo)", 5 kW, Katni MP) from a lead conversion, then worked through it like a real user would — completed milestone tasks across all 12 stages, applied the 5 kW budget template and edited line items, logged expenses, checked Finance/Bank cash position, filled in Compliance items, opened Docs Vault and Generation tracking, edited Warranty records, added a subcontractor and recorded payments against them, attached and viewed files in the Document Vault, and reviewed the Audit log after every action. Reloaded the page repeatedly to confirm localStorage persistence held up throughout.

This is the second pass: after the first round of testing, you asked me to go ahead and fix everything I'd flagged rather than just leave it as discussion points. Below is what actually changed, confirmed by re-testing each one live in the browser afterward — not just read in the code.

All fixes are already saved directly into your `SolarEPC_v23.html` in the ARM_App folder. Nothing further to download or replace.

---

## Fixed

**1. Warranty tab — Edit button was completely dead.** Clicking ✏️ Edit on any warranty record never opened the edit form, because warranty records were regenerated with a new random ID on every render, so the ID being compared never matched — and even if it had, the save handler closed the form after the very first keystroke. Both are fixed: warranty IDs are now stable (derived from the component name), and the form stays open until you click Done.

**2. Subcontractor payments now flow into Finance, Budget, and Expenses.** Recording a payment against a subcontractor used to only update that subcontractor's own "paid so far" figure — it never touched the project's actual cash-out numbers. Recording a payment now also creates a matching expense entry (category "Subcontractor," which was already wired into the Execution budget group but never used), so Finance's Cash Paid Out, the Budget tab's Execution actuals, and the Expenses list all move together. Verified with a real ₹10,000 payment — Spent, Expenses count, and the new line item all updated in lockstep.

**3. Audit log now captures money and warranty changes.** Previously the audit trail only logged milestones, compliance, and budget-template application — expenses, subcontractor payments, and warranty edits vanished without a trace. Now logging an expense, editing or deleting one, recording a subcontractor payment, and editing a warranty record (once, on Done — not spammed per keystroke) all produce a clear audit entry with who and when.

**4. Milestone audit entries now name the specific task.** Instead of a generic "Milestone 'Project Documentation' updated" repeated for every checkbox, entries now read like "Portal Submission: 'PM Surya Ghar Application' marked done."

**5. Expenses can now be dated and backdated.** The date field existed in the data model but had no input anywhere. Both the "Log Expense" and inline edit forms now have a Date field, defaulting to today but fully editable — useful for entering a receipt from a few days ago.

**6. Budget variance no longer shows an alarming "-100%" on untouched lines.** A budget group with nothing spent yet now reads "not started" instead of a red-looking -100% variance, which was mathematically correct but looked like a problem where there wasn't one.

**7. "Margin To-Date" no longer shows a misleading 100% before any costs exist.** On a brand-new project with zero spend, the stat now shows "—" instead of 100%, until there's real cost to compare against revenue.

**8. Tab bar now shows a scroll hint.** The project detail panel's tab row (Milestones through Audit) overflows on most screens, hiding Labour, Expenses, and Audit with no visual cue. A subtle fade-and-arrow now appears at the edge whenever there's more to scroll to, and disappears once you've scrolled all the way.

**9. Document Vault now supports actual file attachments.** Previously it was status-only — no way to attach the real document. You can now attach a scan or photo per document (JPG/PNG/PDF, capped at 700KB per file with a clear message if something's too big — since everything lives in the browser's local storage, keeping individual files small protects the rest of your data from ever hitting a storage limit), view it in an in-app preview, and remove it. Note on the file-size cap: this app has no server, so every attachment is stored inside the browser itself — I kept the cap conservative on purpose. If you're regularly attaching multi-page PDFs or full-resolution photos, it's worth a conversation about whether file storage should move outside the browser entirely (e.g. Google Drive links pasted into the notes field) rather than raising the cap, since browser storage can silently corrupt or hit its limit across *all* your projects at once, not just one.

---

## Left open — genuinely worth a conversation, not fixed

**Role-based permissions untested.** The demo environment only has your Admin account, so I couldn't verify how milestone edit rights, budget visibility, etc. behave for Sales/Technician/Finance/Dealer roles. Worth a dedicated pass once you have test users for the other roles — I'd rather not guess at what those restrictions should be.

**Document Vault file storage is still browser-only.** As noted above, I built attachments to work within the existing local-only architecture rather than changing that architecture — a bigger call that affects the whole app, not just this tab, so I left it as a decision for you rather than making it unilaterally.

---

## Summary

Everything flagged in the first pass has been fixed and re-verified live except the two items above, which are genuine judgment calls about scope (roles need real test users; file storage architecture affects the whole app) rather than something I could respond to with a straightforward code fix. The Warranty bug and the subcontractor-payment/audit gaps were the two most consequential — one was a fully dead button, the other meant real money movements weren't showing up in your numbers at all.
