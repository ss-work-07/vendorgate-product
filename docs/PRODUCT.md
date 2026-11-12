# Vendorgate: product document

**Claude reads the vendor's documents. Vendorgate gets the vendor payment-ready and keeps you on the right side of Section 43B(h).**

Live app: https://vendorgate-production-3ec4dac7.up.railway.app

## 1. The problem, and how it was found

I started from the people reviewing this assignment. Volopay sells spend management: cards, bill pay, reimbursements and accounts payable. I looked for a job that sits next to that product, where Volopay's own documentation shows a gap, and that fits "Claude thinks, the app stays live."

**What I checked, and what I dropped**
- **Receipts and month‑end close.** I dropped this. Volopay already has receipt OCR ("Magic Scan"), automatic receipt matching and accounting mapping rules. Building it would have meant rebuilding their product.
- **Vendor onboarding.** I kept this. Volopay's help centre describes vendors being added by an admin who types the bank details in by hand. I found no vendor self‑onboarding, no document collection for PAN, GST or Udyam, and no 43B(h) tracking. Competitors such as Zoho BillPay, Peakflo and RazorpayX ship vendor self‑onboarding portals, so the demand is proven.

**Why this job hurts in India**
- Before the first payment, finance has to collect a PAN, a GST certificate, a cancelled cheque and, for MSMEs, a Udyam certificate. This usually happens over email and WhatsApp.
- One wrong character in a GSTIN breaks reconciliation. One wrong account number sends money to the wrong place. Paying an account held in a different name than the vendor is a classic fraud pattern.
- **Section 43B(h)** of the Income‑tax Act sets a deadline for paying micro and small enterprises: within 15 days of acceptance, or within the agreed term capped at 45 days if there is a written agreement. If you miss it, the expense can't be deducted until the year you actually pay. You can only track this if you recorded the vendor's Udyam type and activity at onboarding. Tally lists "not verifying Udyam status" as a common mistake.

**Positioning:** *Volopay pays your vendors. Vendorgate makes them payable.* It sits upstream of a payables tool and exports a clean vendor master.

## 2. Why the customer needs this app, and not just Claude

| Claude alone | With Vendorgate |
|---|---|
| The vendor isn't in your chat | The vendor gets a mobile link to fill in, upload photos to, and come back to when asked for a fix |
| Checks are judgement calls | GSTIN check digit, PAN inside the GSTIN, IFSC → bank, account holder vs legal name, duplicates. These are fixed rules that run the same way every time (`src/rules.js`) |
| Nothing is approved | A person approves, rejects or asks for changes. Claude can only recommend |
| Deadlines are forgotten | Every invoice gets a 43B(h) due date, and Claude can ask "what's due this week?" at any time |
| No record | Activity log, a vendor master CSV, and an approved vendor goes back to review if its bank details change |

## 3. How it works

``
You, in Claude ──"Onboard Sharma Packaging"──▶ invite_vendor ──▶ vendor record + invite message with link
Vendor, on phone ──fills link, uploads 4 photos──▶ instant format checks (same rules, in the browser)
Claude ──"Review vendors waiting"──▶ read_vendor_documents (images) ──▶ record_review (findings, recommendation)
Finance team ──review screen──▶ Approve / Ask for changes / Reject ──▶ vendor master CSV
Vendor, on the same link ──sends an invoice (PDF or photo)──▶ waiting for acceptance
Claude ──"Check the invoices vendors sent"──▶ read_invoice (file + vendor master) ──▶ accept_invoice / dispute_invoice
Vendorgate ──acceptance starts the clock──▶ 43B(h) pay-by date, MSMED Act interest if late ──▶ vendor sees the status
Finance / auditor ──Reports──▶ 43B(h) register CSV (Form 3CD clause 26), MSME-1 half-yearly CSV
``

**MCP tools** (`src/mcp.js`):

| Tool | What it does |
|---|---|
| `get_company_profile` | Reads the buying company's details |
| `invite_vendor` | Creates the vendor and saves the invite message Claude wrote, with the link filled in |
| `list_vendors` | Lists vendors with their pipeline status, failed checks and documents |
| `get_vendor` | Returns everything about one vendor |
| `read_vendor_documents` | Returns the uploaded documents as images, alongside what the vendor typed, so Claude can compare them |
| `record_review` | Saves Claude's findings and recommendation. It can fix provable typos but never bank details or names |
| `request_vendor_changes` | Sends the vendor a message and reopens their link |
| `update_vendor` | Changes details. A sensitive change on an approved vendor sends it back to review |
| `list_invoices`, `read_invoice` | Lists vendor invoices by status; gives Claude the invoice file with the vendor's GSTIN, PAN and past invoices to check it against |
| `accept_invoice`, `dispute_invoice` | Records acceptance (starts the 43B(h) clock) or raises a query the vendor sees on their link |
| `add_invoice`, `payments_due`, `mark_invoice_paid` | Records an invoice shared in chat; lists what is due, with MSMED Act interest already owed |
| `compliance_report` | 43B(h) position for the year (deductible, disallowed, at risk) and the MSME-1 half-year |

**Rules engines** (fixed rules, no AI): `src/rules.js` for vendor checks, the 43B(h) deadline and MSMED Act interest (compound, monthly rests, 3× the RBI bank rate set in Settings), and `src/compliance.js` for the 43B(h) register and MSME-1.

**Guardrails**
- Only a person can approve a vendor.
- Claude can't change bank details through a review.
- Changing the PAN, GSTIN or bank details of an approved vendor sends it back to review.
- The vendor's link locks once they submit.
- Only approved vendors can send invoices, and the same invoice number can't be sent twice.
- A pending invoice is timed from its invoice date (deemed acceptance), the safe assumption, until someone accepts it.
- CSV export neutralises spreadsheet formulas.

## 4. Accounts and security

| | |
|---|---|
| **Sign-in** | Email and password (scrypt, constant-time compare, same timing for unknown emails). Passwords need 8+ characters and can't be common ones. |
| **Sessions** | Random 256-bit token in an `HttpOnly`, `SameSite=Lax`, `Secure` (over HTTPS) cookie; expire after 30 days; expired ones are purged. |
| **Abuse limits** | Login: 8 failures per account per 15 min, 40 per network. Sign-up, password reset and token links are rate-limited too. |
| **Cross-site requests** | Cookies are `SameSite=Lax`, and any state-changing request from another site's Origin is refused. Login `?next=` only accepts same-site paths. |
| **Password reset** | Single-use link, valid 30 minutes, stored hashed. Same response whether or not the email exists. Resetting signs out every device and connected Claude. |
| **Email verification** | Sent at sign-up; a banner reminds until confirmed. |
| **Teammates** | Admins invite by email (single-use link, 7 days). Roles: **Admin** (everything), **Approver** (review, approve, accept and pay), **Viewer** (read-only, in the app and in Claude). Every action is logged under the person's name. Removing someone ends their sessions and Claude access at once. |
| **Claude connector** | OAuth 2.1 with dynamic client registration and PKCE (via the MCP SDK's auth router). A consent screen names the app, person and role. Access tokens last an hour; refresh tokens rotate; Disconnect revokes immediately. A Viewer's Claude gets read-only tools. The private-URL connector remains for clients without OAuth. |
| **Shared demo** | The demo password is private (the server keeps only its hash) and nobody needs it: the demo button signs you in. Password, email and team changes are off; a **Reset demo data** button and a nightly reset (about 3am IST) put it back to the starting point, keeping the connector URL. |

## 5. Business model

- **Cost to serve:** about ₹40–80 a month per customer. That covers hosting plus document storage (around 1 MB per vendor). AI costs ₹0 because the customer's own Claude does the reading.
- **Price:**
  - Starter is free, for 5 vendors a month.
  - **Team is ₹2,499 a month**, with unlimited vendors and MSME tracking.
  - Business is ₹7,999 a month, with multiple entities and an API.
- **The first 100 customers:**
  - Chartered accountancy firms, each of which handles vendor onboarding and 43B(h) for dozens of SME clients.
  - Finance‑ops communities.
  - Partnerships with payables and spend tools as their missing onboarding step.

## 6. How it's built

- Node 24, Express and SQLite (`node:sqlite`), with server‑rendered HTML. Three runtime dependencies: the MCP SDK, Express and Zod.
- `src/rules.js` runs both on the server and in the vendor's browser, so both sides apply the same rules.
- Photos are resized in the browser to 1600px JPEG before upload. That keeps uploads fast and gives Claude a sensible image.

- The demo data is fictional. Sample documents are marked SAMPLE.

## Sources
- Volopay Help: [Adding a new vendor](https://help.volopay.co/en/articles/4515156-adding-a-new-vendor), [Vendors in Bill Pay](https://help.volopay.co/en/articles/9223823-overview-of-vendors-in-bill-pay), [Receipts via OCR](https://help.volopay.co/en/articles/9223226-attach-receipts-via-ocr)
- Competitors: [Zoho BillPay vendor onboarding](https://www.zoho.com/billpay/help/vendors/vendor-onboarding/), [Peakflo](https://help.peakflo.co/get-started-peakflo-vendor-onboarding-management), [RazorpayX vendor portal](https://razorpay.com/docs/x/vendor-payments/portal/business.md)
- 43B(h): [Busy](https://busy.in/tds/section-43bh-msme-payment-rule-and-45-day-limit-explained/), [Tally](https://tallysolutions.com/gst/tds-compliance-changes)
- Vendor onboarding checklist (India): [Kleeto](https://www.kleeto.in/blogs/vendor-onboarding-checklist-india-finance-teams.php)
