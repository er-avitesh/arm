# SolarEPC — Wishlist

Things noted but deliberately not built. Nothing here is a bug; each is either a design choice
you have shortlisted, or a decision that is yours to make rather than mine.

Last updated: 20 September 2026

---

## Design shortlist

*(nothing pending)*

### Decided

**Stat tile treatment — settled 20 Sept 2026, applied.** Shortlisted as a flat soft colour wash,
then developed into the version that shipped: a diagonal gradient of the tile's own colour fading
into white, a soft band of light sweeping across it, a faint coloured shadow, and the tile's icon
enlarged behind the figure as a greyscale watermark. Applied to every stat tile in the app.

---

## Decisions waiting on you

**The priced quotation builder is not reachable.** A full bill-of-materials quotation builder —
panels, inverter, cabling, structure, GST, subsidy — exists in the code but nothing renders it. The
Quotation tab shows only a reference number and date, so a quote's value only ever comes from the
figure typed into the negotiation screen. Wiring the builder in would make the quoted value flow
through to conversion and the project value automatically.

**Customers are keyed by name.** Two different customers with the same name merge into a single
customer card and pool each other's projects and payments. Invoices are matched by project *or* by
customer name, so a repeat customer's earlier invoices can be counted against a later project.
Fixing it properly means backfilling a project reference onto older invoices, which changes existing
records — your call whether that is worth doing now or left until the data grows.

**The milestone payment schedule is a fixed table.** Profitability shows a 30/40/20/10 payment split
against stage names that do not match the app's actual milestones. It should either follow the real
milestones or become a per-project payment schedule you set when the project is created.

**Leave entitlements are defined but never enforced.** The leave types carry annual entitlements in
the code, but no balance is ever calculated, shown, or checked, so staff can apply for unlimited
leave. Needs a decision on whether balances should be tracked per person per year.

**Dealer logins link by email only.** A dealer account finds its dealer record by matching the email
typed into Add Dealer against the email they signed up with. If those differ they see "Dealer Profile
Not Linked" permanently. A picker on the dealer form to choose the user account directly would remove
the guesswork.

**Dealers appear in staff lists.** Dealer accounts are included wherever approved users are listed,
so they show up in attendance and in the daily target assignment screen alongside your eight staff.

**Assigning today's targets replaces everyone's.** The target assignment form opens blank, so
assigning a second batch later in the day wipes the targets set that morning for everyone else.

**"Today" is fixed when the app loads.** If the app is left open overnight, the next day's attendance
and entries are still booked against the previous date until the page is refreshed.

**Two modules are not wired into navigation.** Master Data (suppliers/consumers) and the Subsidy &
Loan Tracker exist in the code but are unreachable, having been superseded. They can be deleted or
revived.
