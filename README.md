# Vendorgate

**Vendor onboarding for Indian finance teams that runs on your own Claude.**

Ask Claude to onboard a vendor. Vendorgate sends the vendor a link to fill in on their phone, checks every PAN, GSTIN and IFSC against fixed rules, and has a person approve. It then tracks the Section 43B(h) payment deadline on every invoice that vendor sends.

*Volopay pays your vendors. Vendorgate makes them payable.*

| | |
|---|---|
| **Live app** | https://vendorgate-production-3ec4dac7.up.railway.app |
| **Try it** | Click **Try the live demo** on the home page. You're signed straight in, no password needed. |
| **Claude connector URL** | `https://vendorgate-production-3ec4dac7.up.railway.app/mcp` |
| **Product document** | [docs/PRODUCT.md](docs/PRODUCT.md): the problem, why Claude alone isn't enough, how it works, security, business model |
| **Step-by-step guide** | [docs/USAGE.md](docs/USAGE.md) |

![Vendorgate home page](docs/screenshots/00-home.png)

## Connect Vendorgate to Claude

**1. Copy the connector URL.** In Vendorgate, open **Connect Claude** in the top menu. The URL is `https://vendorgate-production-3ec4dac7.up.railway.app/mcp`.

![Connect Claude page with the connector URL](docs/screenshots/01-connector-url.png)

**2. Add it in claude.ai.** Open **Settings → Connectors → Add custom connector**. Name it **Vendorgate**, paste the URL, then click **Add** and **Connect**.

**3. Sign in.** Claude opens the Vendorgate sign-in page. Log in to your workspace, or click **open the live demo** to use the sample company.

![Vendorgate sign-in page opened by Claude](docs/screenshots/02-sign-in.png)

**4. Allow access.** Check the workspace and role, then click **Allow access**. Claude can read and update vendors and invoices, but it can never approve a vendor. Every action is logged under your name.

![Consent screen: Connect Claude to Vendorgate?](docs/screenshots/03-allow-access.png)

**5. Try it.** In a new Claude chat, switch Vendorgate on from the tools menu and ask:

> We're starting to buy corrugated boxes from Sharma Packaging. Contact is Rakesh Sharma, rakesh@sharmapack.in, +91 98201 44556. 30-day terms, signed agreement. Onboard them in Vendorgate.

Everything in the demo is fictional: the company, the vendors, the people and the documents. The demo workspace is shared and resets every night.
