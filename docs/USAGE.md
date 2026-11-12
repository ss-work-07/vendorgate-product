# Using Vendorgate, step by step

Live app: https://vendorgate-production-3ec4dac7.up.railway.app

## A. Get started

**Step 1: Open the app**
- Go to the live app. To try it, click **Try the live demo**: you're signed in to a sample company with 5 vendors.
- For your own company, click **Sign in**, then **Create a workspace**. Enter your name, company, work email and a password of 8+ characters.

**Step 2: Connect Claude (one time)**
1. In Vendorgate, open **Connect Claude** in the top menu and copy the connector URL: `https://vendorgate-production-3ec4dac7.up.railway.app/mcp`
2. In claude.ai, open **Settings → Connectors → Add custom connector**.
3. Name it `Vendorgate`, paste the URL, then click **Add** and **Connect**.
4. On the Vendorgate sign-in page, log in, or click **open the live demo**.
5. Click **Allow access**.
6. In a new Claude chat, open the tools menu and switch **Vendorgate** on.

Screenshots of each step: [CONNECT.md](CONNECT.md).

## B. Onboard a vendor

**Step 3: Ask Claude to onboard the vendor**
> "We're starting to buy corrugated boxes from Sharma Packaging. Contact is Rakesh Sharma, rakesh@sharmapack.in, +91 98201 44556. 30-day terms, signed agreement. Onboard them in Vendorgate."

Claude creates the vendor and writes an invite message containing the vendor's personal link.

*Without Claude:* on **Vendors**, click **Add manually**, enter the vendor's legal name, and click **Create**.

**Step 4: Send the link to the vendor**
- Open **Vendors**, then the vendor, and use **Send on WhatsApp** or **Copy vendor link**.
- The vendor shows as **Invited** until they start.

**Step 5: The vendor fills it in on their phone**
- They enter their PAN, GSTIN, bank account, IFSC and, if they're an MSME, their Udyam number.
- They upload photos of their PAN card, GST certificate, cancelled cheque and Udyam certificate.
- Typing mistakes are flagged straight away. When they submit, the link locks and the vendor moves to **Ready for review**.

## C. Review and approve

**Step 6: Have Claude review it**
> "Review vendors waiting."

Claude reads the uploaded documents, compares them with what the vendor typed (for example, whether the name on the cheque matches the company), and saves its findings.

**Step 7: Decide in the app**
1. Open **Vendors** and click the **Ready for your review** tile.
2. Open the vendor. You'll see the check results (passed, warnings, failed), Claude's findings and the documents.
3. Choose one:
   - **Approve**: the vendor is ready to pay.
   - **Ask for changes**: write what's wrong. The vendor's link reopens with your message, and the vendor shows as **Waiting on vendor**.
   - **Reject**.

Only a person can approve; Claude can only recommend. If an approved vendor's bank details, PAN or GSTIN change later, the vendor goes back to review automatically.

## D. Invoices and MSME payments

**Step 8: Get an invoice.** Approved vendors use **Send an invoice** on their same link. You can also add one yourself: open the vendor, then **Add invoice manually**.

**Step 9: Accept or dispute it**
> "Check the invoices vendors sent."

Or, in the app, open the vendor's invoices and click **Accept**, or **Dispute** with a reason the vendor will see. Accepting an invoice starts the Section 43B(h) payment clock.

**Step 10: Pay on time**
- Open **MSME payments** in the top menu. It lists unpaid invoices by deadline, with any MSMED Act interest already owed.
- Or ask Claude: *"What's due this week?"*
- After paying, click **Mark paid**.

## E. Reports and export

**Step 11: Year end**
- **Reports**: choose the year and click **Download CSV** for the 43B(h) register (for your auditor) and the MSME-1 half-yearly return.
- **Vendors**, then **Export approved (CSV)**: the clean vendor master to load into your payables tool, such as Volopay.

## F. Team and settings

**Step 12: Add teammates.** In **Settings**, use **Invite by email** and pick a role:
- **Admin**: everything.
- **Approver**: review, approve, accept and pay.
- **Viewer**: read-only, in the app and in Claude.

**Step 13: Settings**
- Update your company details and the RBI bank rate, which is used for interest on late MSME payments.
- In the demo, **Reset demo data** in the banner puts everything back to the start.

## Handy Claude prompts
- "Which vendors are stuck, and why?"
- "Ask Coastal Freight to re-upload a clearer cancelled cheque."
- "Any MSME payments overdue? How much interest do we owe?"
- "How are we doing on 43B(h) this year?"
