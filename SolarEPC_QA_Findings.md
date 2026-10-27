# SolarEPC v23 — QA Findings Report (full app pass, including "fix all issues" follow-up)

**Tested:** every module in the app, live in a browser (React app + localStorage), logged in as `MohitKesharwani` (Admin) — Dashboard, Leads, Projects (full lifecycle), Inventory, Finance, Bank, Attendance, After-Sales, Customers, Calendar, Tasks, Compliance, Procurement, Dealers, Reports, and Users, plus a dedicated pass logged in as a freshly-created Sales-role account to verify role permissions. This report folds together four rounds of testing: an initial single-project deep pass, a wide pass across the rest of the app, a round where every item previously left open for discussion was either fixed and verified or confirmed not a real bug, and a final "can everything be edited, and does editing actually change what it should" pass across every data type and configuration screen in the app.

All fixes below are already saved directly into your `SolarEPC_v23.html` in the ARM_App folder — nothing further to download or replace. Every fix was verified live in the browser afterward, not just read in the code.

---

## Latest round — 18-item fix batch (Leads, Projects, Roles, Milestones, Budget, Docs) + full project-journey test

This round worked through an 18-item list covering IVRS removal, lead-visibility and assignment bugs, customer deletion, a mandatory-survey loophole, a full role-based data-visibility pass on Projects, Project Coordinator/Team Leader roles replacing free-for-all "assign" blocks, milestone/task prerequisite logic, a circular weighted progress indicator, Finance tile-overflow, an editable/auto-suggested budget, removal of three unused Projects sub-tabs, a document checklist replacing the old document vault, a milestone date bug, and a "Custom Project" flow to replace the old freeform "Add Project" button. Every item was fixed and then live-verified in the browser with real interaction — creating test users, test leads, and a test project and driving it through its stages — not just read in the code. Screenshots were taken at each step to visually confirm results, not just to check the underlying data.

1. **IVRS auto-fetch, consumption report, and the "Download Report" button removed from Leads.** All three were the leftover fetch-by-IVRS feature from an earlier round; removed the fields from the Add Lead form, the Overview tab, and the two backing functions entirely. Verified live: the Add Lead form and an existing lead's Overview tab no longer show any IVRS/consumption-report UI.

2. **Sales staff no longer see leads that aren't assigned to them.** Previously the Leads tab showed every lead to every signed-in user regardless of role; now a non-Admin user sees only leads where they are the Assigned SPOC. Verified live: created one lead assigned to a test Sales account and one left unassigned, then logged in as that Sales account and confirmed only the assigned lead appeared ("1 total" instead of the full list).

3. **Customers can now be deleted.** A 🗑 button on each customer card removes them from the Customers list (with a confirmation that clarifies their projects and payment history are kept, not deleted — only the list entry is hidden). Verified live: deleted a test customer and confirmed it disappeared from the list while its underlying project data remained intact.

4. **Site Survey "Assigned Person" now offers the whole staff list, not just the Admin.** This was a copy-paste bug that fed the form only the current logged-in user as if they were the entire staff list. Verified live: opened the Schedule Site Visit form and confirmed the Assigned Person dropdown listed all three staff accounts (Admin, Sales, Technician).

5. **Role-based data visibility audited and tightened on Projects.** This was scoped to the Projects module specifically (an exhaustive line-by-line audit of every screen in the app was not attempted given the size of the app) since that's where the user's own example (item 6) lived, and it's where money-related data is most concentrated. See item 6 below for what was found and fixed.

6. **Sales can see a project's existence and progress, but no longer its money.** This was the deepest issue found this round, and required going beyond the original ask to actually close: the Finance/Budget/Expenses/Audit tabs were already correctly hidden from non-Admin/non-Finance roles, but live-testing under a Sales-role login turned up two more leaks — the project detail panel's header still showed a "💰 Value" and "💸 Spent" tile to every role, and the project list card still showed a "₹… spent" figure underneath the progress ring. Both are now hidden from any role except Admin and Finance. Verified live: logged in as the Sales test account and confirmed the project panel header shows only Start/Target dates (no Value/Spent tiles) and the project list card shows no spent figure, while an Admin login still sees both.

7. **A lead can no longer be converted into a project without a completed site survey, from either conversion path.** The "Won" path already had this gate; the gap was the second path — clicking "Log Update," marking the update "Converted," and confirming — which skipped straight to project creation even with no survey on file. Both paths now go through the same survey gate. Verified live end-to-end: drove a test lead to Negotiation with its survey never filled in, used Log Update → Converted → Confirm, and confirmed the app stopped and asked for the survey instead of creating a project (checked directly against the saved data, not just the screen, that no project had been created); filled in the survey, and confirmed it then proceeded straight to the project-creation form as expected.

8. **The free-for-all "assign" controls on individual milestones are gone; every project now gets a Project Coordinator and a Team Leader instead.** Both are admin-assignable staff pickers shown at the top of the project panel (read-only for everyone else), so every project has exactly two named people who are accountable for it end-to-end, rather than a different assignee per milestone. The Coordinator or Team Leader (in addition to Admin) can now edit the project's milestones. Verified live: assigned a Sales-role staff member as Coordinator and a Technician-role staff member as Team Leader on a test project and confirmed both names and roles displayed correctly.

9. **Milestone/task prerequisite warnings are now in place**, scoped to the clearest, highest-value real-world dependencies rather than every one of the ~80 individual tasks across all 12 milestones (a full dependency graph of that size wasn't attempted given the size of the ask). The user's own example — trying to mark "Inverter Mounting" done under Site Development before "Inverter Ready" is marked done under Procurement — now triggers a warning that explains exactly what's missing and lets you override if you're confident it's correct, rather than blocking outright (since real projects sometimes do have a legitimate reason to work out of the textbook order). Verified live with the exact scenario: clicked "Inverter Mounting" with "Inverter Ready" still unchecked and got the warning verbatim ("⚠️ This is usually blocked: 'Inverter Ready' isn't marked done yet..."); cancelling left the task unchecked, and confirming (override) marked it done as expected.

10. **Project progress is now a circular ring with the percentage shown in the middle**, replacing the old flat progress bar, on both the project list cards and the project detail panel. Each of the 12 milestones is weighted by my own judgment of its real-world effort and criticality to the project timeline (weights sum to 100%, split evenly across that milestone's own tasks), so finishing a few tasks in a small milestone moves the needle less than finishing the same number of tasks in a big one like Site Development. Verified live: watched the ring correctly advance from 0% to 1% after completing a single task in the 22-task Site Development milestone.

11. **Finance tab tiles no longer overflow their box when the number is large — and, per your follow-up feedback, numbers now stay on one line instead of wrapping.** My first pass let long ₹ figures wrap onto a second line to avoid spilling past the tile edge, but you flagged that this broke numbers apart mid-digit-group (e.g. "₹3,40,0" / "00"), which looked worse than the original overflow. Redesigned it properly: the stat tiles are now wider (so more space is available per number), and each number's font size automatically shrinks just enough to fit its own length — a short number like "₹75,000" stays full-size, while a long one like "₹3,40,000" shrinks slightly — but every number now always renders intact on a single line, never split across two rows. Verified live and visually across three separate locations (a project's Finance tab, the Bank Overview tiles, and a spot-check on the Dashboard for regressions): every ₹ figure, including 7-digit values, now displays cleanly on one line.

12. **Budget line items can now be added or removed per project, and a full standard pricing matrix now drives an "✨ Suggest Budget" auto-fill.** Previously only 9 of 28 standard budget line items had a per-kW rate configured (the rest were ₹0); all 28 are now filled in with a standard EPC pricing matrix, so clicking "✨ Suggest Budget" on a project auto-populates a complete, sensible starting budget from its system size, which can then be revisited/adjusted. Admins can also add a custom line item to any budget group, or remove/hide a standard one from a specific project, without affecting other projects' budgets. Verified live: clicked Suggest Budget on a 5kW test project and got a correctly-calculated ₹2,58,000 budget across all three groups (Materials/Execution/Other Direct Cost), then confirmed the Materials group correctly flagged as over budget once real material costs were logged against it (see the full project-journey test below).

13. **Compliance tab removed from under Projects** (the global Compliance module, outside Projects, is untouched). Verified live: a project panel's tab list no longer includes Compliance.

14. **The Docs tab under Projects is now a document checklist, not a document vault.** Since the app doesn't actually store uploaded files, the old "upload a document" UI has been replaced with a per-document-type status marker, and different document types get different status options depending on what actually happens to them: KYC/site/legal documents (e.g. Aadhaar Card) get Pending/Collected; DISCOM and subsidy documents get Pending/Submitted/Approved; and documents your team creates and hands to the customer (e.g. Warranty Cards, Handover Acknowledgement) get Pending/Created/Shared — matching your own example exactly. Verified live: opened a project's Docs tab and confirmed all 18 document types are grouped correctly by category with the right status options for each.

15. **Fixed: a milestone's "started on" date now clears correctly when you uncheck its last completed task.** Previously the date stuck around from before even after every task in that milestone was unchecked, so re-checking a task later would show the wrong original start date instead of a fresh one. Verified live: checked a task (start date recorded), unchecked it (confirmed the milestone's start date directly in the saved data — cleared to none), then re-checked it (confirmed a fresh start date was recorded).

16. **Generation tab removed from Projects.** Verified live: gone from every project's tab list.

17. **Warranty tab removed from Projects.** Verified live: gone from every project's tab list.

18. **The "Add Project" button is gone; projects can now only come from a converted lead, or from a new "+ Custom Project" button for manual/one-off projects.** A Custom Project starts with zero milestones instead of the standard 12, and Admins get a "+ Add Milestone" button (name + a comma-separated task list) to build it out by hand, exactly as needed for that project. Lead-converted projects still get the full standard 12-milestone template automatically, unchanged. Verified live: confirmed the button now reads "+ Custom Project," and that a lead conversion still produces the full standard milestone set.

### Two more role-visibility leaks found only by testing under a Sales login (not on the original list, fixed anyway)

Closing item 6 properly meant actually logging in as a non-Admin user and looking, not just checking that the Finance/Budget tabs were hidden — that's how the header-tile and project-card leaks described in item 6 above were caught. Both are now fixed and verified the same way.

### A real cross-tab data bug found only by running the full project-journey test you asked for

Running the exact "create demo inventory, payment in, payment out, material in, material out" journey you asked for turned up one more bug, upstream of anything on the original list: **Finance → Profitability was silently double-counting material costs** for any project with inventory issued to it. Issuing material to a project already correctly logs it once as a project expense (visible in the project's own Expenses tab and Budget tab); the Profitability tab's cost calculation was *also* separately re-adding the same material transactions from the inventory system's own records, so every project with materials issued to it showed roughly double its real cost and a badly overstated loss. Found and fixed by tracing exactly where the Profitability tab's numbers came from and confirming, with real data, that the two sources were describing the same transaction rather than two different ones.

**Full project-journey test performed to catch this and confirm everything else works together:**
- Created two inventory items (a 540W solar panel and a 5kW inverter) with realistic pricing.
- **Material In:** received 100 panels and 10 inverters into stock; confirmed stock quantities and total inventory value (₹16,50,000) updated correctly.
- **Material Out:** issued 20 panels and 1 inverter to a test project; confirmed stock dropped correctly (100→80, 10→9), inventory value dropped correctly (₹16,50,000→₹13,65,000), and — critically — that this automatically created two matching expense entries against the project (₹2,40,000 + ₹45,000), which is what then let the double-counting bug above be caught.
- **Payment In:** recorded a ₹2,00,000 partial payment against the project's invoice; confirmed it correctly showed as "partial, 59% collected, ₹1,40,000 due" both on the invoice and in the Finance module's totals, and appeared correctly as a deposit in the Bank module.
- **Payment Out:** logged a ₹40,000 labour expense against the same project; confirmed it appeared in the project's Expenses list and correctly raised its Total Spent.
- **Over-budget scenario:** ran "✨ Suggest Budget" on the project (correctly calculated ₹2,58,000 across all three budget groups from its 5kW system size), then confirmed that once real material costs were logged, the Materials group correctly flagged itself as +47% over its ₹1,93,500 budget with a clear warning message and a ₹91,500 forecast overrun.
- **In-budget scenario:** confirmed, on the same project, that the Execution and Other Direct Cost budget groups — which had approved budgets but no costs logged against them yet — correctly showed as "not started" rather than incorrectly flagged as over or under anything.
- **Cross-tab consistency:** after all of the above, independently checked that the same numbers agreed everywhere they should — the project header, the project list card, the Expenses tab, the Budget tab, the global Finance Invoices tab, the Bank module, and (after the fix above) the Finance Profitability tab all showed consistent figures for the same underlying transactions.

---

## Newest round — Lost leads can now be revived anytime

**Request:** a lead marked Lost should be recoverable, not a dead end.

**Fixed.** A "🔄 Revive Lead" button now appears on any lead's detail panel whenever its stage is Lost (it replaces the "❌ Mark Lost" button, which correctly reappears once the lead is active again). Reviving a lead puts it back exactly where it was before it was lost — for example, a lead that was Lost while at "Site Survey" goes back to "Site Survey," not to square one — and resets its stage-entry date so stage-aging timers ("days in stage") start counting fresh from the revive, rather than showing a stale, misleadingly large number. For a small number of leads that may have been marked Lost before this feature existed (and so have no record of which stage they were lost from), reviving falls back to "Contacted" as a sensible default rather than failing or guessing wrong.

Lost leads are reached the same way they always were: on the Leads tab's Pipeline view, click the "❌ Lost (N)" filter button to see them as cards, then open any one to revive it.

Verified live end-to-end: marked a test lead Lost from its initial stage, confirmed it disappeared from the active pipeline and the "❌ Lost" filter count went up, opened it from the Lost filter and confirmed the Revive button appeared (and Mark Lost did not), clicked Revive, and confirmed the lead returned to its original stage with the pipeline counts updating correctly. Separately verified the fallback path by placing a simulated "old-style" Lost lead (no recorded prior stage) directly into the data and confirming it revived correctly to Contacted rather than erroring.

---

## Prior round — DISCOM zones, Lost option, Site Survey rework, Quotation gate, IVRS auto-fetch, Gift Slab cross-tab fix

This round worked through eight specific requests: standardizing DISCOM naming, restoring a missing "Lost" option, letting site surveys be edited and reworking when they're required, linking a won project back to its originating survey, gating the move into Negotiation on a quotation number, adding IVRS-based auto-fetch of consumer data with a downloadable consumption report, letting a lead's assigned person be changed anytime, and fixing the Gift Slab update bug. All eight are implemented and live-verified in the browser with real interaction (not just read in the code), with one caveat noted below for the IVRS live-fetch step.

**1. DISCOM names standardized to zone codes (MPEZ / MPWZ / MPCZ).** Previously the app used "MPMKVVCL" and "MPPKVVCL" as DISCOM options, but both of those are ambiguous shorthand that different people use for different DISCOMs in practice. Replaced everywhere in the app — new lead defaults, the DISCOM dropdown options, the Compliance regulatory-info cards, and the Site Survey form's DISCOM field — with the three actual zone codes: MPEZ (East, now the default), MPWZ (West), MPCZ (Central), plus "Other". Verified live: confirmed the new dropdown options, the default on a fresh lead, and the three correctly-labeled Compliance cards.

**2. "Lost" option restored.** A lead can now be explicitly marked Lost from the lead detail panel via a "❌ Mark Lost" button, available at any stage before the lead is already Won or Lost. Verified live: marked a test lead Lost and confirmed it left the active pipeline and showed correctly as Lost.

**3. Gift Slab amount updates now show up everywhere, including the Dealer's own view.** This was the reported bug: editing a slab amount in Dealers → Gift Slab saved correctly in that one screen's own state, but didn't propagate to other already-open views of the same data (including a Dealer's own view), because the app only re-read the saved data once on page load — it never picked up a change made elsewhere without a manual refresh. Fixed by adding a live cross-tab/cross-view sync listener for both dealers and gift-slab data, so a change made in one place now appears immediately everywhere else that's showing it, without needing a reload. Verified live with two browser tabs open at once: edited a slab amount in one tab and confirmed it updated automatically in the other, unrefreshed tab.

**4. Site Survey can now be edited anytime, and the "required before Won" rule now works as intended.** Previously the survey form always opened blank even when a survey already existed for that lead (silently discarding the old answers if you saved again — a real data-loss risk), there was no way to revisit a completed survey, and skipping the survey left the lead stuck rather than progressing. Fixed all three: (a) the survey form now correctly pre-fills every field from the lead's existing saved survey when you reopen it, so editing an already-filled survey no longer risks losing data; (b) an "✏️ Edit Survey" button is available on the lead's Site Survey tab anytime a survey exists; (c) skipping the survey at the site-visit stage now automatically advances the lead to Proposal Sent (Overview still correctly shows the survey status as pending, since it was skipped, not completed); (d) when moving a lead to Won, if no survey was ever filled in, the app now stops and asks you to fill it in first, then continues to Won once it's done — so a project can never be created without a survey on file. Verified live end-to-end: filled and re-edited a survey confirming values persisted correctly; skipped a survey and confirmed the lead moved to Proposal Sent while Overview still showed the pending indicator; attempted to move a no-survey lead to Won and confirmed it was blocked with a prompt to fill the survey, then completed normally afterward.

**5. Won projects now link back to their originating Site Survey.** A project created from a converted lead now has a "📋 Site Survey" button in its own panel that opens a read-only view of that lead's original survey answers, so the survey no longer has to be found by going back to the (now-inactive) lead. Verified live: opened a project created from a survey-completed lead and confirmed the survey details displayed correctly and matched what was originally filled in.

**6. Moving a lead into Negotiation now requires a quotation number and date first.** Previously a lead could skip straight from Proposal Sent to Negotiation with no quotation on record. Fixed so that attempting to advance to Negotiation now prompts for the Quotation Reference Number and Date if one hasn't been entered yet, and only proceeds once that's filled in. Verified live: attempted the stage move on a lead with no quotation, confirmed the gate appeared, filled it in, and confirmed the lead then advanced normally.

**7. Lead assignment can now be changed anytime.** The Edit Lead form now includes an "Assign to" field (the same searchable staff picker used elsewhere in the app) so the person responsible for a lead can be reassigned at any point, not just at creation. Verified live: reassigned an in-progress lead to a different staff member and confirmed the new name showed correctly on the lead's Overview afterward.

**8. Residential-only subsidy toggle, IVRS auto-fetch, and a downloadable consumption report.** Three related pieces on the New Lead form:
- The "PM Surya Ghar Subsidy" toggle (Taking Subsidy / Giving Up Subsidy / Need to Check) now only appears when Property Type is Residential, since the subsidy scheme doesn't apply to other property types — it's hidden automatically for Commercial/Industrial leads. Verified live by switching Property Type back and forth and confirming the field appears/disappears correctly.
- A new IVRS field with a "🔄 Fetch" button pulls consumer name, sanctioned load, recommended solar capacity, and yearly usage in units directly from your existing QuoteBuilder app's local capacity-calculation engine (the same recommended-capacity formula it already uses), over the connection it already exposes on your machine at `127.0.0.1:8733` — no changes were needed to that app. The same fetch/refresh action is also available from an already-created lead's Overview tab, so consumption data can be pulled or refreshed after the lead exists, not just at creation.
- Once consumption data has been fetched, a "📄 Download Consumption Report" button appears on the Overview tab, generating a printable report (consumer details, DISCOM, yearly and monthly usage, recommended capacity) that opens in a new tab with a "Print / Save as PDF" button, so it downloads as a PDF through the browser's own Print dialog.
  **One important caveat:** the actual live network fetch from IVRS could only be verified up to the point of reaching your local QuoteBuilder server — this sandbox has no network path to `127.0.0.1:8733` on your machine, so I could not personally trigger a real, successful fetch end-to-end. What I did verify live: the graceful "couldn't reach the server" message appears correctly when the local server isn't reachable (confirming the failure path is handled cleanly rather than breaking the form), and — by placing sample consumption data directly into the lead's record — that the Overview display, the recommended-capacity calculation, and the downloadable report all render correctly once real data is present. Please do one real test with your QuoteBuilder app (`arm_server.py`) actually running on your machine and an IVRS you know is valid, to confirm the live fetch itself works end-to-end on your side — if anything looks off there, let me know and I'll adjust it.

---

## Latest round — "can every data record and setting be edited, and does the edited value actually take effect"

This pass had two parts: (1) an audit of every module to find any data type that had no way to be edited once created, and (2) live-testing every existing Edit form by actually changing a value and confirming the change shows up everywhere it should (summary stats, related tabs, other modules) — not just sitting unused in the form.

**Found and fixed — two real gaps where editing was missing entirely:**

**1. Inventory items had no Edit function at all.** You could Add and Delete an item, but once created, a typo in the name, spec, price, or minimum stock level could never be corrected — the only way around it was deleting and re-adding, which loses transaction history. Fixed by adding a real `updateItem` function, an ✏️ Edit button on every item card (Admin-only), and a full Edit Item modal covering every field (name, make, spec, description, category, unit, min stock, price, supplier). Verified live: edited a test panel's price from ₹0 to ₹18,500 and stock minimum from 0 to 10 — the item card updated immediately, and the module's header "Value" stat correctly recalculated (70 units × ₹18,500 = ₹12,95,000), proving the price is actually read from the item record everywhere, not cached or hardcoded.

**2. Staff/User details couldn't be edited after approval.** The User detail modal had an `onUpdate` function passed in but nothing in the UI ever called it — designation, department, phone, email, blood group, address, and Aadhar were permanently frozen at whatever was entered during signup, with no way for Admin to correct a typo or update contact info later. Fixed by adding an "✏️ Edit Details" button (Admin-only) that switches the read-only info grid into an editable form, with Save/Cancel. Verified live end-to-end: updated a staff member's Department to "Sales & Marketing" and Blood Group to "O+", saved, and confirmed the values appeared correctly in the read-only view, in the background user card on the Users list, and — critically — still showed correctly after a full logout and re-login (proving it's written to persistent storage, not just React state).

**Verified working correctly — existing Edit forms, tested live with real value changes to confirm propagation:**

- **Lead Edit** — changed a lead's Capacity from 5 kW to 8 kW; the pipeline card immediately reflected "8 kW".
- **Project Edit** — changed Contract Value from ₹5,00,000 to ₹5,50,000; the project's "VALUE" stat updated immediately, and the Finance tab's "Contract Value" figure picked up the same new number.
- **Budget line items** — changed the "AC Cable" budget line from ₹5,000 to ₹7,000 (inline, no separate save step needed); the "Materials" category subtotal and the project's overall "Approved Budget" total both recalculated correctly (+₹2,000 in each).
- **Global Expenses (Finance → Expenses)** — logged a test expense, edited its amount via the ✏️ icon from ₹1,500 to ₹2,500; the Expenses total and Net Profit figures both recalculated correctly. (Cleaned up the test entry afterward.)
- **Warranty records (per-project)** — edited the Inverter warranty from "5 Years" to "10 Years"; the summary card updated immediately.
- **Subcontractors (per-project Labour tab)** — added a test subcontractor, changed its status from "Pending" to "Completed" via the dropdown; the card's status color and the underlying record both updated correctly. (Cleaned up the test entry afterward.) Note: the "Record Payment" action uses a native browser prompt() popup rather than an in-app form — functionally fine for a real user in a real browser, but worth knowing if you ever want a more polished in-app payment-entry dialog instead of a system popup.
- **Company Settings** (sidebar logo/name) — changed the company name; it updated immediately in the sidebar branding. (Reverted after confirming.)
- **My Profile** (own account) — updated phone number and email; confirmed both values persisted correctly on reopening the modal.

No hardcoded or stale values were found anywhere in this pass — every edit made a real, traceable change to the underlying data, and every dependent total, stat, or display recalculated from the new value rather than showing a cached number.

---

## Fixed this round (in response to "fix all issues mentioned above")

**1. Finance → Commission tab now actually gets data.** Previously the Commission tab had a full UI but no code anywhere ever created a commission record, so it was permanently empty. Fixed by adding a "Commission Amount (₹)" field to the "Mark as Won" modal when you attribute a win to an ARM Employee — entering an amount there now creates a real commission record (salesperson, project, value, amount, pending status) the moment the lead converts, the same way dealer attribution already fed the Dealers leaderboard. Verified live end-to-end: converted a lead with ₹15,000 commission entered, confirmed the record appeared correctly on the Commission tab, and confirmed "✓ Approve Full Payout" correctly marked it paid.

**2. Procurement GRNs now update Inventory stock.** Previously receiving goods (via either the detailed "+ Record GRN" flow or the one-click "✅ Mark Received" button) never touched Inventory — stock levels and Procurement were two completely disconnected ledgers. Fixed so that when a GRN is recorded for a line item that matches a real Inventory item, it now automatically creates the matching "Material In" stock transaction and increases that item's stock level. (Manually-typed PO line items that were never linked to an actual Inventory item are left alone, since there's nothing to safely match them to — the fix specifically targets the common, correct-usage path of Purchase Request → PO → GRN where items come from your actual Inventory list.) Verified live: created a PR for "Test Panel 540W" (qty 20), approved it, created a PO from it, clicked "Mark Received" — Inventory stock correctly jumped from 50 to 70, and a new "+20 In" transaction appeared in the Inventory Transactions log tagged as sourced from the GRN.

**3. Invoice payments now automatically create a matching Bank transaction.** Previously recording a payment against an invoice in Finance and recording a Bank transaction were two completely separate, manual actions, so Bank and Invoices could silently disagree about what had actually been collected. Fixed so that recording an invoice payment now also creates a matching "Customer Receipt" credit transaction in Bank automatically. Verified live: recorded a ₹2,00,000 payment against an invoice, confirmed Bank's "Total Deposits" correctly showed ₹2,00,000 and the transaction appeared in Bank's "All Transactions" list with a clear remark linking it back to the invoice and customer.

**4. Dealer "Edit" can now update every field, not just the name.** Previously clicking ✏️ Edit on a dealer only opened a bare browser prompt box for the name — company, phone, email, service area, and notes couldn't be changed once a dealer was created. Fixed by building a proper Edit Dealer modal matching the full Add Dealer form. Verified live: edited an existing dealer's service area and phone number, confirmed both changes saved and displayed correctly.

**5. Negotiated lead price now updates the lead's headline "Est. Value."** (Carried over — code fix was already written last round, confirmed live this round.) Entering a negotiated price during the Negotiation stage previously updated the quotation internally but left the lead's main "Est. Value" figure frozen at the original amount. Fixed so negotiating a price now updates that figure too. Verified live: negotiated a lead from ₹4,00,000 down to ₹3,50,000, confirmed "Est. Value" on both the Overview tab and the pipeline card updated to ₹3,50,000 immediately.

**6. Finance → Profitability tab's Material/Labour split is now accurate.** Previously the "Material" column only ever showed Inventory stock consumed against a project, while the "Labour" column lumped in *every* logged expense regardless of category — so a ₹50,000 expense you deliberately categorized as "Material" when logging it would show up entirely under "Labour" instead, misrepresenting the project's real cost breakdown. Fixed so material-categorized expenses are now correctly counted toward the Material figure alongside Inventory consumption, and Labour reflects everything else. Verified live: logged a ₹50,000 expense categorized as "Material" against a test project, confirmed the Profitability tab correctly showed Material: ₹50,000 / Labour: ₹0 (previously it showed Material: ₹0 / Labour: ₹50,000).

## Investigated and confirmed NOT bugs (no fix applied, no fix needed)

These three items were flagged as "left open" in earlier passes. This round I traced every code path that writes and reads the relevant fields and attempted to reproduce each one live — I could not find or produce the described issue with the current code, so rather than guess at a speculative fix for something that isn't actually broken, I'm reporting this honestly:

- **"Days in stage" showing NaN for some lead** — every place a lead's stage-entry date is written (`add`, `moveStage`) sets a valid date, and every place it's displayed uses a safe fallback for missing dates. Could not reproduce with any lead state.
- **"Assigned To" showing a raw internal ID instead of a name** (Leads) — every display of assigned staff correctly resolves the ID to a name with a fallback. Could not reproduce.
- **`entryBy` showing a raw internal ID in 3 spots** (Inventory) — traced the full Material In flow; the staff name is computed and stored correctly at submit time. Live-tested a full Material In entry and confirmed "Entry By" displayed the actual staff name, not an ID.

If either of these ever resurfaces with a specific lead or item, the exact record it happens on would help pin down what's different about it, since the code as written handles every path correctly.

## Role-based permissions — spot-checked this round

Created a fresh test account, approved it as a Sales-role user, and logged in as that user. The sidebar correctly showed only the modules a Sales rep should see (Dashboard, Leads, Projects, Attendance, Customers, Calendar, Tasks, Compliance) and correctly hid admin/finance-only modules (Inventory, Finance, Bank, After-Sales, Procurement, Dealers, Reports, Users) — matching the app's role configuration exactly. This confirms the underlying permission mechanism works correctly; a full pass logging in as one test user per remaining role (Technician, Finance, Dealer) to check every edit-permission boundary individually would still be worth doing before wider rollout, but the core mechanism is sound.

---

## Fixed in earlier passes

**AMC "Next Service Date" never moved.** Fixed so recording a visit now recalculates Next Service proportionally, landing on the end date once all visits are used.

**Referral "Reward Given" became permanently unreachable after "Mark Converted."** Fixed so conversion status and reward status are independent, as they should be.

**Task delete confirmation — canceling still closed the task panel.** Fixed; Cancel now truly cancels.

**Compliance "Mark Done" was setting the wrong final status for Net Metering, CEI Inspection, and SLD Approval** — these were being silently recorded as rejected/failed instead of done. This was the most consequential fix across all passes. Fixed and verified across item types; demo records that had been mis-set during earlier testing were manually corrected.

**Procurement "Mark Received" skipped the GRN record and then locked the PO out of ever getting one.** Fixed so it now creates a proper GRN record (and — as of this round — also syncs into Inventory, see above).

**Warranty tab Edit button was completely dead.** Fixed — stable IDs, form stays open until Done.

**Subcontractor payments now flow into Finance, Budget, and Expenses.**

**Audit log now captures money and warranty changes**, and **milestone audit entries now name the specific task**.

**Expenses can now be dated and backdated.**

**Budget variance no longer shows an alarming "-100%" on untouched lines**, and **"Margin To-Date" no longer shows a misleading 100% before any costs exist.**

**Tab bar now shows a scroll hint** where it overflows.

**Document Vault now supports actual file attachments** (capped at 700KB each, browser-storage-based).

---

## Still worth a conversation (product-scope, not bugs)

**Referral "Mark Converted" doesn't create a Lead.** If referrals are meant to feed your sales pipeline, that link doesn't exist — it only flips a status flag.

**Raised Task Requests with no assignee can't be acted on by anyone** except deletion — there's no "claim this" escape hatch.

**Document Vault file storage is still browser-only** — a bigger architectural call affecting the whole app, not just this tab.

**Commission rate/amount is manually entered per conversion**, not calculated from a company-wide commission policy or rate table. This round wired up the mechanism (see fix #1 above); if you want automatic percentage-based commission calculation instead of manual entry each time, that's a follow-up worth scoping separately.

---

## What I checked and found working correctly (no bugs)

Dashboard (clear "being redesigned" placeholder, not broken), Inventory stock in/out recording, global Finance invoices/expenses/P&L totals reconciling against Reports, Attendance check-in/out and leave, After-Sales tickets and warranty claims, Customers list and detail views, Calendar and Tasks, global Compliance tracking, the full Dealer attribution flow (deal count, capacity, margin, Leaderboard position, and gift-slab tier all update correctly on conversion), Dealer Delete (gated behind confirmation), and the entire Users module — self-service Sign Up, the multi-step registration form, Admin approval with role assignment, and Revoke Access, including the confirm-to-cancel path. Role-based sidebar filtering verified working correctly for the Sales role (see above).

---

## Summary

The newest round fixed all 18 requested items on Leads, Projects, roles, milestones, budget, and documents, then ran the full project-journey test you asked for (demo inventory, material in/out, payment in/out, over-budget and in-budget scenarios) — which caught and fixed one real cross-tab bug (double-counted material cost in Finance → Profitability) plus two more role-visibility leaks in the Projects panel that testing under a live Sales-role login turned up beyond the original ask. Every one of the 18 items, plus all three of these extra findings, was verified by real interaction in the browser (creating test users, leads, and a project and driving it through its stages), not just by reading the code.

The prior round closed two real "can't edit this at all" gaps — Inventory items and Staff/User details — and then live-tested every other Edit form in the app (Leads, Projects, Budget, global Expenses, Warranty, Subcontractors, Company Settings, and My Profile) by actually changing values and confirming the change shows up everywhere it should. Nothing was found to be hardcoded or stale.

The prior round fixed five real, verified issues on top of the nine fixed in earlier passes: Commission tab wiring, Procurement→Inventory stock sync, Bank↔Invoice auto-sync, a full Dealer Edit modal, and the Finance Profitability Material/Labour split — plus confirmed the negotiated-price fix from that round is working live. Three previously-flagged display bugs (NaN days-in-stage, ID leaks in Leads and Inventory) were investigated thoroughly and found to already work correctly in the current code — no fix was needed or applied for those. Role-based permissions were spot-checked with a live Sales-role test account and confirmed working as configured. Every fix in this document has been verified by actually clicking through the flow in a browser with real test data, not just by reading the code.

## Round 6 (16 Sept 2026) — GST tracking + manual invoice creation

You asked for GST tracking on both purchases and sales, with the net difference reflected in profit/loss, plus manual control over invoice numbers/dates and payment tracking. Built and verified live:

1. **GST rate is now capped at the real slabs: 0%, 5%, 12%, 18%.** The quotation screen previously also offered 28% — removed, since you confirmed only three real slabs apply plus 0% for non-GST items/services. The same four options are used consistently everywhere GST is entered (expenses, invoices, quotations).

2. **Input GST (on purchases) is now captured on every project expense.** The "Log Expense" form (Material, Labour, Transport, Equipment, Subcontractor, etc.) has a new GST Rate field alongside Amount. The amount you enter stays exactly what you actually paid (GST-inclusive) — the GST portion is automatically extracted from it for tracking, so nothing about your existing cost totals changes. Leaving it at 0% means "no GST on this purchase," matching non-GST purchases.

3. **Invoice creation is now a manual, explicit action — no more silent auto-invoice.** Previously, the app quietly created one invoice per project the moment a lead was marked Won, with an internal ID and today's date standing in for the real thing. That's gone. Finance → Invoices now has a "+ New Invoice" button where you pick the project and enter the **Invoice Number and Invoice Date exactly as raised in your ERP**, plus the Taxable Value and GST Rate — the GST amount and total are computed automatically and shown on the invoice card ("Taxable: ₹X + GST @ Y%: ₹Z"). You can raise as many invoices per project as you actually issue (advance, milestone, final) rather than being stuck with one lump figure.

4. **Payments now record Date and Cash/Account mode.** "+ Record Payment" on any invoice now asks for the payment date (editable, defaults to today) and whether it came in as Cash or via Account (bank/online), shown next to each payment entry with a 💵/🏦 marker — so the record has a real audit trail instead of always stamping "today" with no payment-method trace.

5. **Net GST is calculated and deducted from profit, per project and company-wide.** Finance → Profitability now shows Output GST (from your invoices), Input GST/ITC (from your logged purchases), and Net GST Payable as their own figures — both as a summary tile at the top and as a new "Net GST" column in the per-project table — and the Profit figure for every project (and the company-wide Gross Profit tile) is now net of this GST liability, exactly as you asked.

Verified live end-to-end: logged a ₹1,18,000 purchase at 18% GST (₹18,000 input GST extracted), raised a new manual invoice for ₹3,00,000 taxable + 18% GST (₹54,000 output GST, ₹3,54,000 total, invoice number and date exactly as entered), recorded a payment with a specific date and Cash mode, and confirmed the Profitability tab showed Net GST Payable = ₹36,000 (₹54,000 output − ₹18,000 input) with the project's profit correctly reduced by that amount. Ran the full navigation smoke test afterward with zero console errors.

**One design call made without a direct answer from you:** invoice creation moved to a fully manual, multi-invoice-per-project model (rather than keeping one auto-created invoice and just adding number/date fields to it), since you described invoicing as something that happens "when we create the invoice on our ERP" — implying a distinct action, possibly more than once per project. If you actually wanted the simpler one-invoice-per-project version, let me know and I'll adjust it.

## Round 7 (16 Sept 2026) — Cash flow: where your investment is moving, and where it's stuck

You asked to see, plainly, how cash is moving on a project and where money is currently tied up or at risk — not a profit number, but the actual movement of cash in and out.

1. **Every project's Finance tab now has a "Cash Flow" section** — a running-balance chart (green above the line, red below) showing your net cash position over the project's life, followed by a plain chronological ledger: every payment received and every expense paid, in order, each with the running balance right after it. A status banner up top says outright, in plain words, whether the project currently has your money tied up ("₹X of your own money is currently tied up in this project") or is cash-positive. This was built on cash-tracking logic that already existed in the app's code but was never actually shown anywhere — the tiles above it (Cash Received, Cash Paid Out, Working Capital Invested, Max Working Capital Exposure) were already there; what was missing was the ability to see the movement, not just the current totals.

2. **A new company-wide "🌊 Cash Flow" tab in the global Finance module** shows the same running-balance chart, but combining every active project's cash events into one timeline — your total capital currently deployed across all live projects, and how that total has moved. Below it, a table lists every project by current cash position, worst-exposure first, with a clear status: 🟢 Positive, 🟡 Invested, or ⚠️ High Exposure (flagged automatically when a project's tied-up capital exceeds 30% of its contract value) — so you can see at a glance exactly which projects are currently eating your working capital.

3. **Found and fixed a real bug while building this:** every recorded invoice payment was being counted twice in the cash-position numbers — once as the payment itself, once through a bank-ledger entry the app silently creates alongside every payment for bank reconciliation. This was invisible as long as only a single "Cash Received" total was shown, but it meant that total (and everything derived from it — Working Capital Invested, Max Exposure, the company-wide "capital tied up" figure) was **understating your real exposure**, sometimes by a lot. On the test project, this bug was hiding ₹2,00,000 of real exposure — it showed ₹43,000 tied up when the true figure was ₹2,43,000. Fixed at the source so every cash figure in the app (not just the new chart) is now accurate.

Verified live: confirmed the chart and ledger render correctly with the right colors and running balances, confirmed the fix by checking the ledger no longer double-lists a payment, confirmed the company-wide table correctly flagged the test project as "⚠️ High Exposure" once the real numbers were in. Ran the full navigation and Finance-sub-tab smoke test afterward — zero console errors.

## Round 8 (16 Sept 2026) — App theme changed to Ocean Teal

You asked to see theme options before building, picked "Ocean Teal" from six mockups, then confirmed "go ahead and apply everywhere."

1. **The app's brand color has been swept from indigo to teal everywhere it appears** — the sidebar (now a near-black teal instead of near-black indigo), every primary button, active nav highlights, progress rings and bars, tile borders and value colors, links, badges, box-shadow glows, and the login page's background gradient. This was a full, systematic pass: every indigo hex value used for branding (10 distinct shades, plus two `rgba()` glow variants) was mapped to an equivalent teal/cyan shade and replaced across the entire file — 292 individual replacements in total, confirmed with a search afterward for any leftover indigo hex codes (none found).
2. **Status colors were deliberately left untouched.** Green for positive/paid/success, red for negative/overdue/danger, amber for warnings/pending, and the blue/violet colors used to tag specific data categories (e.g. certain chart series) are unrelated to the brand color and still mean exactly what they meant before — including the cash-flow chart's green-above/red-below fill, which still reads clearly against the new teal palette.

Verified live across the login page, sidebar/navigation, Dashboard, the Projects list, a project's Finance tab (including the cash-flow chart and stat tiles), the global Finance tabs (Invoices and the company-wide Cash Flow view), and every one of the 16 sidebar modules — all render correctly with the new teal theme and no leftover indigo. Ran the full navigation smoke test afterward across all 16 modules with zero console errors, and confirmed the app's script is still syntactically valid.

A follow-up re-scan of every color in the file (not just the ones already on the replacement list) turned up two more spots the first sweep missed — a dealer-portal gradient card and a milestone progress-bar gradient — both still showing old indigo. Both fixed and re-verified against a full color re-scan; nothing indigo remains anywhere in the file.

## Round 9 (17 Sept 2026) — Milestones/tasks (multi-task, delete, reorder), a real 20-item document checklist, and a subcontractor payment gap closed

You raised four things: milestones could only get one task at creation with no way to add more; nothing manually added could be deleted; milestones and tasks couldn't be reordered to match real site sequence; the Docs tab needed a proper Pending/Collected/Not Required checklist with your exact 20 document types; and a subcontractor's initial "paid so far" amount wasn't showing up in Finance.

1. **Every milestone — the 13 built-in ones and any you add — now has a "+ Add Task" control**, so you're no longer limited to whatever tasks you typed in when the milestone was created. New tasks are added right inside the expanded milestone panel.
2. **Anything you add manually can now be deleted.** A 🗑 next to a custom milestone (one created via "+ Add Milestone") removes it and all its tasks, with a confirmation first since this can't be undone. A 🗑 next to any manually-added task removes just that task, and the milestone's status (Pending/Active/Done) recalculates correctly afterward. The 13 built-in milestones and their original template tasks stay fixed and can't be deleted, only completed — that's intentional, since those represent your standard project template.
3. **Milestones and the tasks within a milestone can now be dragged into a new order** — a ⠿ handle on each row. Drag one milestone onto another to swap its position in the sequence; do the same with tasks inside an expanded milestone. Verified live: dragged "Portal Submission" above "Project Documentation" and confirmed the new order held.
4. **The project completion % and the circular progress indicator already correctly account for added milestones and tasks** — this turned out to be built on a weighted-percentage formula that already handles any number of milestones and any task count per milestone, so no fix was needed there; adding the "+ Add Task" and delete controls above just needed to keep feeding it the same way. Verified live: on a fresh project, adding one custom milestone with one task and marking it done moved completion from 0% to 7% (1 of 14 total milestones now weighted in), and the ring updated to match.
5. **The Docs tab checklist has been replaced with your exact 20 items** — Aadhar Card, Pan Card, Bank Passbook/Cancelled Cheque, Electricity Bill, Land Acquisition Paper, Passport Photo, Site Survey Form, Signed Work Order, Signed Discom Documents, Site Photographs, Roof/Terrace Photo, Existing Meter Photo, Existing LT Panel Photo, Earthing Space Photo, Wiring Route Video, Signed Net Metering Agreement, Approved SLD Drawing, Panel Warranty Card, Inverter Warranty Card, and Handover Acknowledgement — grouped into KYC/Site/Legal/DISCOM/Warranty/Handover sections. This fully replaces the old 18-item list.
6. **Each document now has exactly three statuses: Pending, Collected, or Not Required.** This turned out to already exist in the app's underlying document-tracking logic (it just wasn't labeled clearly) — marking a document "Not Required" now correctly counts it toward the completion bar and %, so a project that genuinely doesn't need, say, a Land Acquisition Paper isn't stuck below 100%. Verified live: marking one document Not Required moved the tab from 0/20 (0%) to 1/20 complete (5%).
7. **Found and fixed a real gap while working on the subcontractor request:** the "Record Payment" button on an existing subcontractor already correctly created a matching Expense entry — but the *initial* "Paid So Far" amount entered when you first add a subcontractor did not, so that money silently never showed up in Expenses, Finance, or P&L. Fixed so adding a subcontractor with money already paid now creates a matching expense (category: Subcontractor) at the same time. Verified live end-to-end: added a subcontractor with a ₹5,000 agreed amount and ₹2,000 already paid, and confirmed a "Payment to [name] (Civil & Structure)" expense for ₹2,000 appeared immediately in the Expenses tab and the project's "Spent" tile increased by exactly ₹2,000.

Verified live across all of the above, plus a full 16-module navigation smoke test afterward — zero console errors, and the app's script is syntactically valid.

## Round 10 (17 Sept 2026) — Making the Round 9 fixes apply to data you already had, not just new entries

You pointed out, correctly, that a fix to the app's code doesn't automatically fix data you already entered before that fix existed — you gave the exact example of a subcontractor you'd added and paid *before* the Round 9 finance-sync fix, whose payment still wasn't showing up in Expenses even after the fix was installed. That's expected of a code fix on its own (it only changes what happens on the *next* action), so this round adds an automatic, one-time repair that runs the moment you open the updated app, and fixes your existing projects in place — no manual re-entry needed.

1. **Subcontractor payments made before the fix are now backfilled into Expenses automatically.** On load, the app checks every subcontractor on every project: if the amount they've been paid (in total) is more than what's actually been logged as an expense for them, it creates one backfill expense for the missing difference, clearly labeled "— backfilled from before the Finance sync fix" so it's obviously distinguishable from a normal manual entry. This is safe to run every time the app opens — once the gap is closed, it won't create a duplicate on the next load.
2. **The Docs checklist migration had a real bug of its own, caught during this round's testing.** The first attempt at moving old document records onto the new 20-item checklist only carried over items that had a direct old-to-new match, but didn't fill in the *rest* of the new checklist — so a project that had, say, 3 old document records ended up with a checklist that only knew about 6 total documents instead of the full 20, showing a wrong percentage like "2/6 complete" instead of "2/20". Fixed so every project's document list is now always reconciled against the complete current 20-item checklist. Old multi-stage statuses like "Submitted" or "Approved" (from document types that no longer exist in the simplified 3-state model) are now correctly folded into "Collected" rather than silently reverting to "Pending".
3. **This migration runs automatically and silently every time the app loads**, for every project — you don't need to do anything, and it's safe to run repeatedly (it only ever fills in genuine gaps, never duplicates or overwrites something already correct).

Verified live: manually recreated your exact scenario — injected a subcontractor with a paid amount but no matching expense, and a project with old-format document statuses (including one using the old "Submitted" status that no longer exists in the new checklist) — then reloaded the app the same way you would after downloading the update. Confirmed: the missing subcontractor expense appeared automatically (Expenses count went from 4 to 5, "Spent" tile increased by the exact paid amount), the document checklist correctly showed all 20 items with the old statuses correctly carried over ("Submitted" became "Collected"), and the completion percentage was correct (3/20 = 15%, not the broken 2/6 = 33% the first migration attempt produced). Reloaded a second time to confirm no duplicate expenses or data corruption on repeat runs. Ran the full 16-module smoke test afterward — zero console errors, and the app's script is syntactically valid.

**Worth knowing:** a few things from earlier rounds don't need this kind of repair, and I checked each one specifically so nothing was quietly skipped. GST rate on old expenses defaults to 0% if it was never entered (there's no way to know a purchase's real historical GST rate, so this is the honest default, not a bug). Old invoice payments made before the "date + Cash/Account mode" tracking was added simply show a blank date/mode rather than a made-up one — same reasoning. Milestones and tasks needed no migration at all, since the completion-percentage formula was already built to handle any project's milestone/task set dynamically.

## Round 11 (18 Sept 2026) — Deleting a project now asks what else to delete with it

You asked that deleting a project prompt you about which other tab entries tied to it should also go, rather than silently leaving them behind (or the old behavior, which just silently left everything else behind with a bare "are you sure?").

1. **Deleting a project (from either the Projects grid's 🗑 button or the project detail panel) now opens a checklist modal instead of a plain confirm.** It scans every other module for records linked to that specific project and shows each category that actually has something linked, with a count and — where relevant — the ₹ amount involved: Invoices, Bank Transactions, Salesperson Commission Entries, Inventory Stock Transactions, After-Sales Tickets, Tasks/Requests, and Purchase Requests. A category is only shown if it has at least one linked record — an empty project's delete modal just says so and deletes cleanly.
2. **Every checkbox is checked by default, per your answer** — the assumption is you want a clean sweep unless you say otherwise. Uncheck anything you want kept (it'll be left in place, referencing a project that no longer exists, which every one of these modules already handles safely — they store the customer/project name directly on the record rather than looking it up live, so nothing breaks or shows blank).
3. **Purchase Orders and GRNs tied to this project's Purchase Requests are traced and offered too, per your answer.** If a Purchase Request belonging to this project has a Purchase Order raised against it, and that PO has a GRN (goods-received note) recorded against it, both are shown as their own checklist rows and included in the same "checked by default" sweep — so a full procurement chain that only ever existed for this project doesn't get orphaned as unreferenced paperwork.
4. **Dealer stats are corrected automatically, with no checkbox** — if the project was linked to a dealer (won through them), deleting it now also reduces that dealer's deal count by 1 and their margin total by exactly what this project contributed, so the Dealers tab doesn't keep counting a deal that no longer exists. This isn't a checkbox because it isn't really "an entry to delete" — it's a running counter on the dealer that needs correcting to stay accurate, and the modal tells you plainly this is happening.
5. **Inventory stock levels are deliberately left untouched even if you delete the linked Stock Transaction records.** Deleting the paperwork trail of a material movement doesn't undo the physical movement — the material was still physically taken from or returned to the store. Reversing stock counts on a project-delete would fabricate stock back into existence (or double-remove it) rather than reflect reality, so this was kept as a firm default rather than another checkbox to avoid a genuinely risky, easy-to-misclick option.
6. **Found and fixed a real bug while building this:** Salesperson Commission entries (Finance → the commission created when an employee wins a lead) were never actually being saved to your browser's storage at all — they lived only in memory and vanished on every single page reload, regardless of this project-delete feature. This was almost certainly invisible day-to-day since the Commissions view still looked populated within a session, but the moment the app was closed and reopened, every commission record silently disappeared. Fixed so commissions now persist and reload correctly like every other module — this was necessary groundwork for this round's feature anyway, since a "Salesperson Commission" delete checkbox would have been pointless against data that vanishes on its own.

Verified live: created a fully-linked test project — one invoice, one bank transaction, one salesperson commission, one stock transaction, one ticket, one task, one purchase request with a purchase order and GRN raised against it, and a dealer link with a margin figure — then opened the delete modal and confirmed all 9 rows appeared with correct counts and ₹ amounts, and all checkboxes were checked by default. Unchecked "Invoices" to test toggling, then confirmed the delete: the project and all 8 selected categories were removed, the unchecked invoice was correctly left in place, the linked dealer's deal count and margin dropped by exactly the right amounts, and there were zero console errors. Ran the full 16-module smoke test afterward — zero console errors.

## Round 12 (18 Sept 2026) — GST slab added to Inventory items and bulk upload

You asked for a GST slab on Inventory items, including on bulk upload, so GST can be factored into stock value.

1. **Every inventory item now has a GST Slab (%) field** — a dropdown (0/5/12/18%, matching the same GST slabs already used elsewhere in the app for expenses and invoices) on both "+ Add Item" and "✏️ Edit Item", sitting next to Default Supplier. It defaults to 0% for existing items that predate this change (no way to know a real historical GST slab, same honest-default reasoning as the Round 10 migration notes).
2. **Bulk Import now asks for GST Slab too.** The downloadable Excel template has a new "GST Slab (%)" column (sample rows show 12% and 18% as examples), the upload parser reads it per row, and the row-by-row preview before import shows each item's parsed GST% so you can catch a typo before importing. Leaving the column blank for a row defaults that item to 0% rather than rejecting the row.
3. **Stock cards and the inventory summary now show the "GST included" figure, not just the base price.** Each item card shows its GST% and the GST-inclusive unit price underneath the base price (e.g. "+18% GST · ₹59,000 incl."), and the top-of-page "Value: ₹X" summary now also shows the GST-inclusive total stock value in brackets, computed by applying each item's own GST slab to its price × current stock — so you can see both what your stock is worth on paper and what it actually represents once GST is factored in.

Verified live: added an item manually at 18% GST and confirmed it saved and displayed correctly (price ₹50,000 → shown as "+18% GST · ₹59,000 incl."), then built and uploaded a bulk-import Excel file with two items at 18% and 12% GST and confirmed both imported with the correct GST slab attached and showed correctly in the preview step beforehand. Ran the full 16-module smoke test afterward — zero console errors, and the app's script is syntactically valid.

## Round 13 (18 Sept 2026) — Inventory card icons were overlapping each other

You flagged, with a screenshot, that on Inventory item cards the category badge, edit pencil, and delete bin were crowding into each other and becoming unreadable — a direct side effect of the GST line added in Round 12 making cards more cramped, though the root cause was already there before that: the category badge sat in normal page flow while the edit/delete buttons floated on top of it with no reserved space between them, so on a card with a short name (or after the low-stock warning made cards render more tightly) the badge and buttons landed on the same spot.

1. **Fixed by reserving proper space for the edit/delete buttons up front**, instead of letting the category badge flow into wherever the buttons happened to float. The item name/spec column and the category badge now share a row that stops short of the button zone (Admin view only, where the buttons exist at all), so the badge always renders fully to the left of the edit and delete icons with a clean gap — never underneath or through them.

Verified live: reproduced your exact scenario (narrow cards, low-stock items) and confirmed the category badge, ✏️ edit, and 🗑 delete now render as three clearly separated elements with no overlap, across items with both short and long names. Ran the full 16-module smoke test afterward — zero console errors, and the app's script is syntactically valid.

## Round 14 (18 Sept 2026) — Invoicing solar modules at their real blended GST rate (8.9%)

You pointed out that solar module invoices are actually a mix of 5% and 18% GST items, and in practice you raise the invoice at 8.9% of the overall basic amount. This lines up with the standard GST treatment for solar EPC works contracts in India: the contract value is split 70% Goods (taxed at 5%) and 30% Services (taxed at 18%) — 0.7×5% + 0.3×18% = 8.9% — rather than a single flat rate. The existing invoice form only had a flat GST Rate dropdown (0/5/12/18%), with no way to represent this blend, so you were presumably computing 8.9% by hand each time.

1. **The "+ New Invoice" form now has a GST Type toggle: "Flat Rate" or "Solar EPC (Blended)".** Flat Rate behaves exactly as before. Solar EPC (Blended) asks for the Goods Share (%) and the two underlying rates — defaulting to 70% Goods @ 5% and 30% Services @ 18%, which computes to 8.9% — and shows the Services share and the resulting blended effective rate live as you adjust it, so nothing is hardcoded: if the goods/services split or either rate ever changes (as GST rates on solar equipment periodically do), you just update the inputs and it recalculates.
2. **The live preview and the saved invoice both show the full breakup, not just the blended number** — e.g. "Goods GST @5% on 70% (₹17,500) + Services GST @18% on 30% (₹27,000) = 8.9% blended, ₹44,500" — so the invoice itself carries the audit trail of how the rate was derived, in case a client or a GST auditor asks. A flat-rate invoice still just shows "Taxable: ₹X + GST @ Y%: ₹Z" as before — the breakup line only appears for invoices actually raised in blended mode.

Verified live: raised a ₹5,00,000 taxable invoice in Solar EPC (Blended) mode at the default 70%/5% + 30%/18% split and confirmed the live preview, the saved invoice record, and the invoice card all showed the identical figures — Goods GST ₹17,500 + Services GST ₹27,000 = ₹44,500 total GST at an 8.9% effective rate, ₹5,44,500 total invoice value. Ran the full 16-module smoke test afterward — zero console errors, and the app's script is syntactically valid.

## Round 15 (19 Sept 2026) — The real Dashboard, built role by role with live charts

You asked for an interactive, chart-driven dashboard, decided by role and confidentiality — I first showed you a standalone mockup with demo data so you could see the look and the data split before anything touched the real app. You approved it with one correction: Net Profit should be Admin-only, not visible to Finance. This round replaces the "Dashboard is being redesigned" placeholder with the real thing, wired to your actual projects, leads, invoices, tasks, tickets, commissions, inventory, and bank data — no more demo numbers.

1. **Admin** sees the full picture: Total Revenue Collected, Net Profit, Cash Position, and Active Projects as headline figures; a Revenue vs Expenses trend, the Project Portfolio by status (Active/Completed/On Hold/Cancelled), the company-wide Lead Pipeline Funnel, a Needs Attention panel (invoices overdue 30+ days, projects with open compliance alerts, inventory below minimum stock — all computed live, not hardcoded), a Cash Flow trend from your Bank data, and a Top Dealers leaderboard by margin earned.
2. **Finance** sees Collected, Outstanding, Net GST Payable (Output GST minus Input GST, the same formula already used in Finance → Profitability), and an Overdue Invoices count — deliberately **no Net Profit tile**, matching your instruction and the same admin-only boundary the Finance module's own Profitability/P&L/Commission tabs already use elsewhere in the app. Also gets Revenue vs Expenses, Expenses by Category (Material/Labour/Transport/Equipment/Subcontractor/Overhead), the Cash Flow trend, and an Outstanding Invoice Aging list.
3. **Sales** sees only their own book: My Active Leads, My Conversions, Win Rate, and My Commission (with pending payout called out) — plus their own Pipeline Funnel, Follow-ups Due, Wins by Month, and Commission Status (paid vs pending). No company-wide figures, no other rep's numbers.
4. **Technician** sees only their own assignments: Open Tasks, Tickets Assigned, Completed This Month, and Overdue — plus My Assigned Projects (milestone completion per project they're actually on), My Task Status, weekly Tickets Resolved, and Today's Tasks by priority. No pricing, invoicing, or company financials anywhere.
5. **Dealer** — since a Dealer's landing page was already its own dedicated portal rather than the shared Dashboard tab, I added the same chart treatment directly into that portal instead of duplicating it: a Margin Earned by Month chart and a My Projects by Status donut, sitting alongside the existing deal count, capacity, total margin, and reward-slab progress. Still strictly their own deals only.

All charts (line, bar, donut, funnel) are genuinely interactive — hover for an exact-value tooltip with a crosshair on trend lines, per-bar and per-segment hover on bar and donut charts — and every chart has a graceful "no data yet" state instead of breaking or showing blank space on a freshly-started account.

**Found and fixed a second real bug while building this**, same shape as the Round 11 commissions bug: General Expenses (Finance → Expenses, the overhead entries like Salaries/Rent/Travel/Utilities/Marketing) were never actually being saved to your browser's storage either — they lived only in memory and vanished on every page reload. Since the new dashboard's Net Profit and Expenses figures depend directly on this data, I fixed the persistence as part of this round rather than build a dashboard on top of numbers that would silently reset to zero.

Verified live: seeded a full realistic dataset — leads across every pipeline stage, seven converted deals spanning six months with a mix of active/completed/on-hold/cancelled projects, invoices with partial and full payments, salesperson commissions paid and pending, technician tasks and tickets in various states, dealer-linked projects, low-stock inventory items, and six months of bank transactions — then logged in as an Admin, a Finance user, a Sales rep, a Technician, and a Dealer in turn and confirmed each dashboard showed only what that role should see, with Net Profit appearing solely on the Admin view, and zero console errors on any of the five. Ran the full 16-module smoke test afterward — zero console errors, and the app's script is syntactically valid on both the CDN and local-test builds.

## Round 16 (19 Sept 2026) — Full-app audit: every entry checked against where it should appear

You asked me to go through the whole app, make test entries, confirm each one is actually linked to the records it belongs to and visible as entered, and find anything hardcoded that should instead follow the data you type. I audited all nineteen modules line by line and then drove real entries through the UI. Twenty-seven separate problems came out of it, all now fixed. They fall into four groups.

**Things you entered that were silently thrown away on reload.** Attendance records, leave applications, and calendar/daily site assignments were never being written to storage at all — the same class of bug as the commission records in Round 11 and the general expenses in Round 15. Every check-in, every leave request, and every day's site assignment disappeared the moment the app was closed. All three now persist and reload like everything else.

**Hardcoded values that overrode what you actually typed.** The biggest one: on a new lead, the "First Follow-Up Action" and "Action Due By" fields you fill in were discarded on save and replaced with the literal text "Make first contact call" due tomorrow — so a salesperson typing "Visit site with survey kit, 30 Sept" got something else entirely on the card and in the Follow-ups list. The calendar had three invented staff names baked into it (Arun Kumar, Deepak Singh, Rahul Sharma) left over from demo data, which would show against older assignments instead of the real person. Marking someone present from the admin attendance screen stamped a fake 09:00 check-in and wiped their genuine check-in/check-out times. And a general expense was always dated today with no way to back-date it, so June's rent entered in September landed in September on every chart — there is now a date field on that form.

**Figures that were simply wrong.** Net Profit and the whole P&L — in Finance, in Reports, and on the new Admin dashboard — counted only company overheads and excluded every rupee of project cost. A job with ₹8 lakh of panels and labour logged against it contributed nothing to expenses, so profit was overstated by the entire cost of doing the work. The P&L now shows Project Costs and Company Overheads as separate lines with a correct Net Profit beneath them. The Bank Balance card read ₹0 whenever an invoice receipt recorded inside the app was the most recent transaction, because those rows carry no bank-statement balance; the balance now carries forward correctly from the last statement figure. Project progress was being calculated two different ways — the Projects module used the weighted formula while Reports, Customers and the dealer portal used a flat milestone count, so the same job read 58% in one place and 31% in another; everything now uses the weighted figure. Projects created directly (not from a lead) were given no milestones at all, which made them show "NaN%" complete. The sidebar Tasks badge counted a task status that does not exist in the app, so it could never display a number no matter how much work was assigned. Compliance alerts could only ever fire on one status, meaning six of the seven checklist items — CEI inspection, agreements, net meter installation and the rest — could sit overdue indefinitely without ever appearing in the alert count. The AMC count included contracts that had already expired. And the "SLA at risk" warning used a flat four-hour window for every ticket, so urgent tickets were flagged as at-risk the instant they were raised while a low-priority one warned only in its last four hours; it is now a proportion of that ticket's own SLA.

**Entries that were never linked to where they belong.** Service tickets were being assigned by staff *name* while the technician's dashboard matched on staff *id* — so a technician's "Tickets Assigned" tile was permanently zero and field staff never saw their own service jobs. Tasks, tickets, purchase requests, purchase orders and goods receipts all carried a project link that no project screen ever displayed, so the link you took care to set bought you nothing; projects now have a **Site Activity** tab listing all of it. The customer's email address was captured and validated on the lead form and then shown nowhere and never carried to the project. The "Consumer Remarks / Special Requirements" captured at the moment of winning the deal were written to a field nothing read, and were being lost anyway because the conversion passed a stale copy of the lead. A dealer's phone number and notes were only ever visible inside the edit form — not on their card, portal or the leaderboard. Emergency contact phone and relation, and date of birth, were collected at signup and unreachable from anywhere in the app, which matters most in exactly the situation they exist for. A rejected sign-up request vanished from every tab with no way to restore it and no way for that person to re-register, so one mis-click was permanent; there is now a Rejected tab with an approve action. The dealer name shown on a project was a snapshot taken at conversion, so renaming a dealer left old projects showing the old name. Deleting a Material Out reversed the stock but left the material cost sitting on the project forever, and there was nothing stopping you issuing more of an item than you actually had. On the procurement side, raising a purchase order against an approved request never moved that request out of "approved", so it stayed in the dropdown and could be ordered twice, and the project selected on the request was never copied onto the purchase order or the goods receipt; a short delivery also marked the order fully received, so the balance quantity could never be booked in.

Verified live: entered a real lead through the UI with a custom follow-up action, a custom due date and an email address, and confirmed all three saved and displayed exactly as typed; walked it through the site survey gate and the Mark-as-Won conversion and confirmed the resulting project carried the email, the consumer remarks and the full twelve-milestone set. Logged a company expense back-dated to 15 June and confirmed it kept that date. Logged ₹1,80,000 of project material cost and confirmed the P&L now shows it as its own line rather than omitting it. Linked a task and a service ticket to the project and confirmed both appear under Site Activity. Marked attendance, applied leave and created a calendar assignment, then reloaded and confirmed all three survived and that no fake 09:00 check-in was written. Issued material to a project, deleted the transaction, and confirmed the stock went back to 100 and the project's material cost was removed with it. Signed in as a technician and confirmed the assigned ticket and task now show on their dashboard. Confirmed the Bank balance reads ₹6,20,000 rather than ₹0 with an app-recorded receipt as the latest row. Ran the full sixteen-module smoke test and all five role dashboards afterward — zero console errors throughout, and the app's script is syntactically valid on both the CDN and local-test builds.

A few things I found and deliberately left alone, because they are product decisions rather than defects, and I would rather you choose: the detailed BOM quotation builder exists in the code but is not reachable from the Quotation tab, so a quote's value only ever comes from the negotiation screen; customers are keyed by name, so two different customers with the same name merge into one card and a repeat customer's invoices can be counted against both projects; the milestone payment schedule shown in Profitability is a fixed 30/40/20/10 table whose stage names do not match the app's real milestones; leave entitlements are defined in the code but no balance is ever calculated or enforced; a dealer login still links to its dealer record only by matching email, with no way to pick the user account directly; dealer accounts appear in staff lists for attendance and daily targets; assigning today's targets replaces everyone's targets rather than merging; and the app's notion of "today" is fixed when the page loads, so an app left open overnight books to the previous day.

## Round 17 (20 Sept 2026) — New look for the stat tiles, applied across the app

You wanted the tiles restyled rather than rebuilt — same figures, same labels, different treatment —
and after working through three rounds of options you picked the gradient wash with a gloss sweep.

Each tile now carries a diagonal gradient of its own colour fading into white, a soft band of light
sweeping across it like a reflection on glass, and a faint drop shadow in the same colour so it lifts
off the page. The label moves above the figure in small capitals and the figure itself grows by
roughly a third, so the number is the first thing the eye lands on. The icon is no longer a small
badge beside the value — it is enlarged behind it as a greyscale watermark bleeding off the bottom
corner. Draining the colour out of that watermark is what keeps it readable; left in full colour at
low opacity the icons turn into visual noise.

This reaches every stat tile in the app rather than just the Dashboard. Fifty-six of them already
came from one shared component, so those changed together. The other nine were small stat grids
written inline in the Subcontractors, Tasks, After-Sales, Profitability and Dealer Portal views —
those have been routed through the same component with a compact setting, which trims the padding,
the figure and the watermark for the tighter grids they sit in. That means there is now exactly one
place in the code that decides how a stat tile looks.

Verified live: signed in and stepped through Dashboard, Finance, Bank, After-Sales, Tasks and Reports
confirming both the full-size and compact tiles render correctly at their real widths, then adjusted
the compact watermark down after finding it crowded the figure in the narrower five-across grids. Ran
the full sixteen-module smoke test and all five role dashboards — zero console errors, and the script
is syntactically valid on both the CDN and local-test builds.

## Round 18 (20 Sept 2026) — Bulk inventory import: dropdowns, stock-free item creation, duplicate handling

Three things you asked for after using the bulk import in anger.

**The template now has real Excel dropdowns.** Category, GST Slab and Unit are proper data-validation
dropdowns in the downloaded file, so those columns cannot be mistyped or filled with a value the app
does not recognise. Category and GST Slab are strict — Excel refuses anything off the list — while
Unit is a suggestion list you can still type past. A second sheet, "Valid Values", spells out the
accepted entries for anyone filling the file by hand. Worth noting how this was done: the spreadsheet
library the app already loads cannot write data validation in its free version, so the template file
is now assembled directly rather than through that library. It was checked with an independent
spreadsheet reader to confirm it is a genuinely valid workbook and that all three dropdowns are
present. If a browser ever fails to produce it, the import falls back to the old plain template rather
than leaving you with nothing.

**Items can now be created without any stock, which is the default.** The import has two modes. The
first, "Create item records only", ignores the Current Stock column entirely and creates every item at
zero — you then book the real quantity in through Material In, so each unit has a transaction behind
it. That matches how the manual Add Item form has always worked and closes a gap noted in the audit,
where imported opening stock appeared in the app with nothing in the ledger to explain it. The second
mode, "Include opening stock", still reads the column but now also writes a matching Material In entry
for every item, tagged to that import, so stock levels and the transaction ledger agree.

**Duplicates are caught before anything is written.** On upload, every row is checked against your
existing inventory and against the rest of the file, matching on item name, make and specification and
ignoring case and spacing. Anything that collides is flagged and, by default, skipped rather than
silently creating a second copy. Tapping a flagged row opens it for editing right there — change the
name, make or specification and the flag clears as soon as it is genuinely unique, with the counts at
the top updating as you go. For a row that matches something you already stock, there is also the
option to update the existing item instead, which refreshes its make, specification, category, unit,
minimum level, price, GST slab and supplier while deliberately leaving its stock alone, since stock
should only ever move through Material In and Material Out. A row that is missing a required field is
still blocked, and a category the app does not recognise is flagged on the row instead of quietly
becoming "Others".

Verified live: generated the template and confirmed with an independent spreadsheet reader that the
file is valid and carries all three dropdowns with the correct value lists. Built a test file
deliberately containing a duplicate of an existing item, a repeated row within the file, a row missing
its make, and an unrecognised category, then uploaded it and confirmed each was flagged correctly and
the clean rows counted. Chose "update the existing item" on the duplicate and confirmed it updated the
price and supplier in place without touching its stock and without creating a second copy. Edited the
repeated row's specification in the preview and watched the duplicate flag clear and the import count
rise. Ran the same file again in opening-stock mode and confirmed every item's stock matched the file
and a Material In entry was logged for each one at the matching quantity. Ran the full sixteen-module
smoke test and all five role dashboards — zero console errors, and the script is syntactically valid on
both builds.

## Round 19 (20 Sept 2026) — Undo/redo, inventory multi-select, and a batch of inventory fixes

**Undo and redo, three steps deep.** Every data change you make is now reversible from the ↶ and ↷
buttons at the top right, or with Ctrl+Z and Ctrl+Shift+Z. It covers entries across the whole app —
leads, projects, inventory, stock movements, invoices, expenses, tasks, tickets, dealers, procurement
and the rest — restoring all of them together so related records never fall out of step. Three steps
is the limit you asked for, and an undone change is saved, not just shown, so it survives a reload.
User accounts and company settings are deliberately outside it, since those are administration rather
than day-to-day entries. The history lives only in the current session and starts empty each time the
app is opened.

**Bulk import and the Transactions tab.** Transactions were in fact being written, but only in the
"Include opening stock" mode — the default mode creates item records at zero stock on purpose, and it
never said so clearly once the import finished. There is now a summary across the top of the Inventory
tab after every import, which either confirms the Material In entries were recorded or states plainly
that stock is zero and no transactions exist yet, and points you to Material In. Alongside that, a real
gap is closed: rows you resolved as "update the existing item" were having their Current Stock column
ignored even in opening-stock mode. Those quantities are now applied and get a matching Material In
entry like any other.

**Multi-select on inventory items.** Each item card carries a checkbox for Admins. Selecting anything
opens a bar above the grid showing the count, with select-all for the current category view, a bulk
category change, and a bulk delete that lists exactly what it is about to remove and how many stock
transactions reference those items. Anything done here is undoable like any other change.

**Deleting an item no longer blanks out its history.** Every stock transaction now stores the item's
name, make, specification and unit at the moment it is written, so the ledger stays readable when the
item is later removed — the row keeps the real name with a small "item deleted" marker instead of the
dash you were seeing, and the ledger's search box can find those rows again. Deleting an item now also
tells you up front how many transactions reference it, and back-fills the name onto them as it goes,
so history recorded before this change is preserved too. Records from before that have no stored name
degrade to "Unknown item" rather than an empty cell.

**Other problems found in the Inventory tab and fixed.** Selecting the same item on two rows of one
Material In or Material Out batch moved stock only once while recording both rows, so the ledger and
the project's material cost disagreed with the actual stock — rows for the same item are now merged
before anything is applied, and the shortage check sees the combined quantity. Issuing more than you
hold was correctly refused, but the form closed anyway and the whole batch was lost; it now stays open
so the quantity can be corrected. Deleting a receipt whose stock had already been issued out silently
clamped the stock at zero instead of failing — it is now refused with an explanation. Receipts recorded
against a purchase order were written in a different shape from every other transaction, so they showed
a dash for source and purpose in the ledger; they now match. A purchase order already partly received
could be marked fully received a second time, adding the stock twice — that button no longer appears
once a receipt exists. Items hidden from the low-stock warning stayed hidden forever unless stock was
added through one specific route; goods receipts and imports now bring them back too. And Add Item and
Bulk Import were open to Technicians while Edit and Delete were Admin-only, even though the import's
update path can rewrite an existing item's price and category — both are now Admin-only.

Verified live: deleted three items one at a time, undid three times and confirmed all three returned in
the right order, confirmed a fourth undo does nothing, redid twice, and confirmed the undone state
survives a page reload. Selected two items, changed their category in bulk, deleted them in bulk, and
undid it. Recorded a transaction against an item, deleted the item, and confirmed the ledger still shows
its name with a deleted marker. Unit-tested the row-merging helper on duplicate rows, string quantities
and mixed items, and the ledger's name resolution for live, deleted and pre-existing records. Ran the
import in both modes and confirmed the summary matches what was actually written. Ran the full
sixteen-module smoke test and all five role dashboards — zero console errors, syntax valid on both builds.
