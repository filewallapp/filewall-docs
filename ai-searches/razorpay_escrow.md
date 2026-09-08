Yes. I checked the **current 2026 documentation** for Razorpay, Stripe and PayPal/Braintree specifically against Mitfloww's model. There is an important conclusion:

> **Razorpay Escrow is the most directly relevant option I found for Mitfloww's Indian marketplace/advance-payment model. Stripe Connect is a marketplace product, but its current India restrictions make your exact "hold → later transfer" flow problematic. PayPal has delayed disbursement for approved marketplace partners, but you cannot simply sign up and use it as generic escrow.**

Let's break this down in simple terms.

---

# 1. First, what Mitfloww actually needs

Your business is essentially:

```text
Client
   │
   │ Advance
   ▼
┌─────────────────┐
│ Money is held   │
│ safely          │
└────────┬────────┘
         │
         │ Freelancer works
         │ Revisions
         │
         ▼
Client approves
         │
         │ Final payment
         ▼
Money is released
         │
         ├── Mitfloww platform fee
         ├── Extra revision fee
         └── Freelancer
```

There are **two separate money movements**:

### A. Advance

Client pays ₹X before the freelancer gets the money.

You want:

```text
Client
  ↓
₹5,000
  ↓
Escrow
  ↓
WAIT
```

Then later:

```text
Project completed
       ↓
Client approves
       ↓
₹5,000 becomes part of freelancer's payout
```

### B. Final payment

Suppose:

```text
Project = ₹20,000
Advance = ₹5,000
Remaining = ₹15,000
```

Client eventually pays ₹15,000.

Then Mitfloww calculates:

```text
Advance                 ₹5,000
Final payment           ₹15,000
                        ────────
Total project money     ₹20,000

Platform fee             -₹2,000
Extra revision fee         -₹500
                        ────────
Freelancer gets          ₹17,500
```

**That calculation should be done by Mitfloww.**

The payment provider's job is primarily to **hold/move the actual money according to the approved fund-flow structure**.

---

# 2. Razorpay Escrow — this is the interesting one

Razorpay currently has a product called **RazorpayX-powered Escrow Account**.

And this isn't just a normal Razorpay payment account.

Razorpay explicitly says its escrow product is suitable for things including **online marketplaces**, and describes the account as a mechanism where money is released when agreed obligations are met. ([Razorpay][1])

Even more importantly, Razorpay describes the actual onboarding process:

```text
1. Validate use case
        ↓
2. Sign escrow agreement
        ↓
3. Open escrow account
   with partner bank
        ↓
4. Go live
```

They say the first step involves an **escrow checklist** that is reviewed with their banking partners. ([Razorpay][1])

They also describe the escrow arrangement as involving the depositor, trustee if applicable, and the bank, with RazorpayX acting as the technology partner. ([Razorpay][1])

### This is much closer to what Mitfloww needs.

And importantly, their current page explicitly lists **"Marketplace Escrow"** as a supported use case. ([Razorpay][1])

---

# 3. Does the ₹40 lakh problem apply to this?

This is where we need to be precise.

The **₹40 lakh requirement you heard about is associated with Razorpay's third-party settlement / Route terms**, not simply "Razorpay cannot process your payments below ₹40 lakh."

Razorpay's current terms say that if you instruct Razorpay to settle transaction amounts to a third party, you represent that your annual turnover exceeds **₹40 lakh**, or annual export turnover exceeds **₹5 lakh**, as applicable. ([Razorpay][2])

Razorpay's current Route documentation also has financial-turnover eligibility requirements under the September 2025 RBI Payment Aggregator guidelines. ([Razorpay][3])

So:

### Razorpay Route

```text
Client
   ↓
Razorpay
   ↓
Split between Mitfloww + freelancer
```

**Potential ₹40L eligibility issue.**

### Razorpay Escrow

```text
Client
   ↓
Escrow account
   ↓
Conditional release
   ↓
Beneficiary
```

**Different product and different onboarding process.**

I therefore **would not assume the ₹40 lakh Route restriction automatically means Razorpay Escrow is unavailable to Mitfloww.**

But Razorpay has to approve **your exact use case** and the banking partner has to approve the escrow arrangement. Their documentation explicitly says the use case is validated with banking partners. ([Razorpay][1])

So this is the question I would send to Razorpay:

> "Mitfloww is an online freelance marketplace. A client pays an advance for a freelancer's project. The funds must remain in escrow until project completion/client approval. The client then pays the remaining balance. Mitfloww deducts its platform fee and any additional revision fees and releases the remaining amount to the freelancer. Can RazorpayX Escrow support this exact marketplace use case for a startup below ₹40 lakh annual turnover?"

That question is much better than asking:

> "Can Razorpay do escrow?"

---

# 4. What would Mitfloww actually have to provide?

Razorpay's published material says the process starts with **use-case validation**, followed by an escrow agreement and partner-bank account opening. ([Razorpay][1])

Their KYC documentation also specifically lists additional documents for escrow accounts, including:

* Escrow Board Resolution
* Finalized/signed Escrow Agreement
* Trustee Board Resolution

along with the normal business-entity KYC. ([Razorpay][4])

So expect something along the lines of:

```text
Mitfloww company information
        +
Business KYC
        +
Directors/beneficial-owner KYC
        +
Business model explanation
        +
Client/freelancer fund-flow explanation
        +
Refund/dispute rules
        +
Escrow agreement
        +
Board/trustee resolutions where applicable
```

The **exact requirements will come from Razorpay/bank during onboarding**, so I would not hard-code assumptions about which documents you'll need.

---

# 5. Stripe — does it support this?

### Globally: YES, Stripe Connect is designed for marketplaces.

Stripe explicitly positions Connect for marketplaces that onboard freelancers/sellers, collect payments, manage fees and pay recipients. ([Stripe][5])

So conceptually:

```text
Client
  ↓
Stripe
  ↓
Mitfloww marketplace
  ↓
Freelancer
```

is exactly the kind of business model Stripe Connect is designed for.

Stripe also supports platform fees and payout timing as part of Connect. ([Stripe][5])

---

# 6. But Stripe India has a major problem for your exact architecture

This is extremely important.

Stripe's current India marketplace documentation says:

* Stripe India is **invite-only**
* Some Connect integrations require contacting Sales
* **Separate charges and transfers are currently not supported**
* Standalone transfers to connected accounts are currently not supported
* Manual payouts are supported only for limited use cases ([Stripe Support][6])

So if your planned architecture is:

```text
Client pays
     ↓
Mitfloww controls the balance
     ↓
Wait
     ↓
Mitfloww calculates final amount
     ↓
Transfer to freelancer
```

I would **not design your Indian payment architecture around Stripe Connect until Stripe explicitly approves your exact use case.**

Stripe itself says Indian Connect has restrictions around these fund-flow models. ([Stripe Support][6])

---

# 7. Stripe also requires KYC for your freelancers

If Stripe does approve your Connect setup, freelancers/connected accounts aren't just arbitrary bank accounts.

Stripe requires onboarding and verification.

For Indian connected accounts, requirements depend on:

* individual/company status
* capabilities
* business structure
* representative
* bank/external account
* PAN
* identity information
* business information

etc. ([Stripe Support][7])

For example, Stripe's India requirements include verification information such as PAN and company/LLP registration details depending on the connected-account structure. ([Stripe Support][7])

That's actually **good for Mitfloww** because you don't want to blindly send money to an arbitrary bank account.

---

# 8. PayPal — surprisingly, yes, there is something relevant

PayPal currently has a feature called **Delayed Disbursement**.

The documentation says:

> hold funds from a buyer before disbursing them to the seller.

That's almost exactly the conceptual requirement you have. ([PayPal Developer][8])

The flow can be:

```text
Client
   ↓
PayPal
   ↓
Funds held
   ↓
Mitfloww decides when appropriate
   ↓
Seller receives funds
```

However, there is a **very important restriction**:

> You must be an **approved partner** and onboard sellers with the `DELAY_FUNDS_DISBURSEMENT` feature.

So this isn't:

> "Create PayPal account → enable escrow → done."

It's a marketplace-partner arrangement. ([PayPal Developer][8])

And PayPal's documentation currently says delayed disbursement automatically disburses funds after **28 days**, so this isn't necessarily an unlimited escrow mechanism where you can hold the money indefinitely. ([PayPal Developer][8])

---

# 9. Don't confuse PayPal's normal payment holds with your escrow

This is another important distinction.

PayPal can hold payments because of:

* new seller
* risk
* disputes
* unusual activity
* verification
* etc.

Those are **PayPal's risk holds**.

They aren't your Mitfloww escrow system.

PayPal says these risk-based holds can generally last up to 21 days, with exceptions for disputes. ([PayPal][9])

That's completely different from:

```text
Mitfloww:
"Hold this project's ₹5,000 until the client approves."
```

Don't build around PayPal's ordinary risk holds.

---

# 10. Braintree/PayPal has another interesting marketplace capability

Braintree's marketplace documentation actually describes **escrow**:

```text
Transaction
    ↓
Escrow
    ↓
Release
    ↓
Sub-merchant
```

Their documentation says escrow allows you to withhold sub-merchant disbursements and later release them. ([PayPal Developer][10])

But there is a significant constraint:

> When using escrow, Braintree requires you to hold/release the **entire transaction**, including service fees.

If you need partial disbursements, they say you need separate transactions. ([PayPal Developer][10])

That's something we'd need to carefully evaluate against your:

```text
platform fee
+
revision fee
+
freelancer amount
```

model.

And Braintree says new merchants looking for a marketplace solution should contact Sales. ([PayPal Developer][11])

So again: **enterprise/marketplace onboarding, not ordinary PayPal checkout.**

---

# 11. So what should Mitfloww choose?

Based on what I found **today**, I'd rank the options like this for your specific requirements:

| Provider                     | Marketplace | Conditional holding       | India                                   | Mitfloww fit            |
| ---------------------------- | ----------- | ------------------------- | --------------------------------------- | ----------------------- |
| **Razorpay Escrow**          | Yes         | **Yes**                   | **Yes**                                 | **Best candidate**      |
| Razorpay Route               | Yes         | Not the same escrow model | Yes                                     | ⚠️ ₹40L eligibility     |
| Stripe Connect               | **Yes**     | Platform payout controls  | India, invite-only/restricted           | ⚠️ Need Stripe approval |
| PayPal delayed disbursement  | Yes         | **Yes**                   | Depends on product/marketplace approval | ⚠️ Partner approval     |
| Braintree Marketplace Escrow | **Yes**     | **Yes**                   | Need availability confirmation          | ⚠️ Sales/approval       |

The important thing is that **Razorpay Escrow currently explicitly advertises marketplace escrow in India**, which makes it the first provider I'd approach for your Indian flow. ([Razorpay][1])

---

# 12. Now let's make Mitfloww's actual payment flow very simple

Suppose:

### Project

```text
Project price = ₹20,000
Advance = ₹5,000
```

Client chooses:

> "Pay ₹5,000 advance"

### Step 1 — Client pays

```text
CLIENT
  │
  │ ₹5,000
  ▼
RAZORPAY
  │
  ▼
ESCROW
```

Mitfloww receives a webhook:

```text
advance_payment = PAID
```

Now Mitfloww allows the freelancer into the revision workflow.

---

# 13. Freelancer works

Nothing gets paid to freelancer yet.

```text
Escrow
  │
  │ ₹5,000
  │
  ├── HOLD
  │
  ▼
Project work
  ↓
Revisions
  ↓
Final file
```

Mitfloww's database knows:

```text
Project:
    price = ₹20,000

Advance:
    ₹5,000
    status = HELD
```

---

# 14. Client approves

Client presses:

> **Approve final file**

Now Mitfloww knows:

```text
Advance = ₹5,000
Already paid
Remaining = ₹15,000
```

So Mitfloww asks the client for:

```text
Pay ₹15,000
```

---

# 15. Client pays the final ₹15,000

```text
CLIENT
  │
  │ ₹15,000
  ▼
PAYMENT PROVIDER
  │
  ▼
ESCROW / APPROVED FUND FLOW
```

Webhook:

```text
final_payment = PAID
```

Now Mitfloww's internal calculation happens.

---

# 16. Mitfloww calculates the freelancer's final amount

Example:

```text
Advance                     ₹5,000
Final payment              ₹15,000
                            ───────
Total project money        ₹20,000

Platform fee                ₹2,000
Extra revision fee            ₹500
                            ───────
Freelancer receives        ₹17,500
```

Very important:

### The ₹500 revision fee isn't necessarily "another payment."

It can simply be an **amount allocated from the project's money** if your commercial model says the freelancer owes/receives that amount differently.

You need to decide exactly what "extra revision fee" means economically.

For example, if the client pays an extra ₹500 specifically for a revision, you might have:

```text
Client pays revision fee ₹500
        ↓
Project ledger +₹500
        ↓
Platform takes its configured share
        ↓
Freelancer gets remainder
```

Don't just subtract arbitrary fees from the freelancer's money. Your terms should define **who pays each fee**.

---

# 17. Then Mitfloww tells the payment system to release

Conceptually:

```text
Escrow
 │
 │ ₹20,000
 │
 ├── Platform fee → Mitfloww
 │
 └── Freelancer → ₹17,500
```

The exact API mechanics depend on the provider and the approved escrow agreement.

**Your backend should not simply move money based on a frontend button.**

It should be:

```text
Client approves
       ↓
Server validates project state
       ↓
Server confirms payment status
       ↓
Server calculates ledger
       ↓
Server creates release instruction
       ↓
Provider/escrow executes
       ↓
Webhook confirms release
       ↓
Mitfloww marks payout COMPLETED
```

That's the architecture I strongly recommend.

---

# 18. What your database should look like

Don't store this as just:

```text
project.paid = true
```

You need an actual financial ledger.

Something conceptually like:

```text
Project
 ├── agreedAmount
 ├── advanceAmount
 └── status

Payment
 ├── type = ADVANCE
 ├── provider = RAZORPAY
 ├── providerPaymentId
 ├── amount
 └── status

Payment
 ├── type = FINAL
 ├── provider = RAZORPAY
 ├── providerPaymentId
 ├── amount
 └── status

Fee
 ├── type = PLATFORM
 └── amount

Fee
 ├── type = REVISION
 └── amount

Payout
 ├── freelancerId
 ├── amount
 ├── provider
 ├── providerTransferId
 └── status
```

And ideally a ledger:

```text
Ledger

+ ₹5,000   Client advance
+ ₹15,000  Client final payment
- ₹2,000   Platform fee
- ₹500     Revision fee
-------------------------
= ₹17,500  Freelancer payout
```

This gives you a proper audit trail.

---

# 19. The REALLY important architecture decision

I would make Mitfloww's payment system **provider-independent**.

Don't write your project logic like:

```ts
if (razorpay) {
   ...
}
```

Instead:

```text
Mitfloww Payment Domain
          │
          ├── Payment Intent
          ├── Escrow
          ├── Ledger
          ├── Fees
          ├── Payout
          └── Refund
                    │
              Provider Adapter
                    │
        ┌───────────┼────────────┐
        │           │            │
    Razorpay      Stripe       PayPal
     Escrow       Connect      Marketplace
```

Then your project logic doesn't care whether the underlying provider is Razorpay or Stripe.

---

# 20. What I'd do for Phase 1

For **Indian users**, I'd approach Razorpay with this exact model:

```text
                 MITFLOWW

                    │
                    ▼

        Razorpay Payment Gateway
                    │
                    │ UPI / Cards
                    ▼
             Razorpay Escrow
                    │
          ┌─────────┴─────────┐
          │                   │
      Advance              Final
      payment              payment
          │                   │
          └─────────┬─────────┘
                    │
                    ▼
              Mitfloww Ledger
                    │
           ┌────────┴────────┐
           │                 │
       Platform fee      Freelancer
       Revision fee       payout
```

And **before implementing it**, get Razorpay to confirm:

1. Mitfloww qualifies for their Escrow product.
2. A freelance marketplace is an approved use case for your exact model.
3. Whether the ₹40 lakh Route requirement has **any bearing** on the proposed escrow arrangement.
4. How UPI collections enter the escrow account.
5. Whether the advance can remain there until project completion.
6. How long funds can remain in escrow.
7. Whether partial releases are supported.
8. How refunds work before/after final approval.
9. How disputes/chargebacks are handled.
10. How Mitfloww's platform fee is deducted.
11. Whether revision fees can be deducted from the escrowed/project balance.
12. How freelancer KYC and bank-account verification works.
13. Whether the freelancer needs to be onboarded as a beneficiary/linked account.
14. Exact fees for the escrow + collection + payout components.

**Do not start coding the money movement until those answers are confirmed.**

---

# 21. One subtle problem we haven't discussed yet

There's a potentially important issue with your **advance + final payment** model.

Suppose:

```text
Project = ₹20,000
Advance = ₹5,000
```

Client pays ₹5,000.

Then freelancer does 10 revisions.

Then client says:

> "I don't want the project anymore. Refund my ₹5,000."

What happens?

You need rules for:

```text
Client cancellation
Freelancer cancellation
Client disputes
Freelancer disputes
Partial refund
Full refund
Client disappears
Freelancer disappears
Revision limit exceeded
Project deadline expires
Client approves accidentally
Chargeback
Payment failure
Escrow expiry
```

The payment provider can give you the **financial mechanism**, but **Mitfloww must define the business rules**.

That is why I would build the payment domain before integrating the provider.

---

## Bottom line

**Yes, there is a realistic path for Mitfloww.**

For India, **RazorpayX Escrow is currently the most promising thing I found for your exact use case** because Razorpay explicitly advertises marketplace escrow and describes a bank-backed escrow onboarding process. ([Razorpay][1])

**Razorpay Route is a different thing**, and its current terms/documentation do contain the turnover requirements you heard about. ([Razorpay][3])

**Stripe Connect is excellent conceptually for marketplaces, but Stripe's current India restrictions make your desired delayed-transfer architecture something you must get approved rather than assume will work.** ([Stripe Support][6])

**PayPal has genuine delayed-disbursement functionality, but it requires approved-partner status and has a 28-day automatic-disbursement rule for that feature.** ([PayPal Developer][8])

So I would **not redesign Mitfloww's product yet**. I'd first pursue **Razorpay Escrow for India**, while architecting the backend so Stripe/PayPal can be added later.

And one final caution: this is a **payment-regulatory/business-structure issue**, not merely an API integration. Razorpay's own documentation says the escrow use case is validated with banking partners, so getting their written approval for **Mitfloww's exact flow** is essential before treating this architecture as production-ready. ([Razorpay][1])

[1]: https://razorpay.com/x/escrow-accounts/?utm_source=chatgpt.com "Escrow Account in India | Escrow Services | Razorpay Escrow"
[2]: https://razorpay.com/terms/?utm_source=chatgpt.com "Razorpay Terms & Conditions"
[3]: https://razorpay.com/docs/payments/route/?preferred-country=IN&utm_source=chatgpt.com "Route | Razorpay Docs"
[4]: https://razorpay.com/docs/x/idfc-first/kyc/?utm_source=chatgpt.com "KYC for Current Operators | Razorpay Docs"
[5]: https://stripe.com/in/connect/marketplaces?utm_source=chatgpt.com "Stripe Connect | Marketplace Payment Processing"
[6]: https://support.stripe.com/questions/stripe-india-support-for-marketplaces?locale=en-GB&utm_source=chatgpt.com "Stripe India support for marketplaces : Stripe: Help & Support"
[7]: https://support.stripe.com/questions/onboarding-requirements-for-stripe-connect-in-india?locale=en-GB&utm_source=chatgpt.com "Onboarding requirements for Stripe Connect in India : Stripe: Help & Support"
[8]: https://developer.paypal.com/platforms/checkout/delayed-disbursement/?utm_source=chatgpt.com "Delay disbursement | PayPal Developer"
[9]: https://www.paypal.com/in/legalhub/paypal/useragreement-full?utm_source=chatgpt.com "PAYPAL USER AGREEMENT"
[10]: https://developer.paypal.com/braintree/articles/guides/braintree-marketplace/processing?utm_source=chatgpt.com "Braintree SDK Docs"
[11]: https://developer.paypal.com/braintree/articles/guides/braintree-marketplace/funding?utm_source=chatgpt.com "Braintree SDK Docs"



------------

Yes. I checked the current pricing/documentation because this is exactly where the architecture can become unexpectedly expensive.

The short answer is:

> **Razorpay Escrow does not publish a simple "Escrow = X%" price on its public page.** It appears to be a separately negotiated/business product, so you should not assume it costs only the normal Razorpay 2–3%/4% processing fee.
>
> **Stripe Connect has clearly published additional marketplace/payout fees in India.**
>
> **PayPal charges its normal transaction fees, and its delayed-disbursement feature is not documented as a separate percentage fee—but you need marketplace-partner approval and PayPal deducts its fees from seller funds.**

## 1. Razorpay Escrow

This is the one where I would **not budget based on the normal Razorpay payment rate yet**.

Razorpay's public Escrow page does **not publish a percentage price**. Instead, it asks businesses to contact its Escrow experts and says the process involves:

1. Use-case validation
2. Escrow agreement
3. Partner-bank escrow account
4. Go live through RazorpayX

It explicitly lists online marketplaces as an escrow use case. ([Razorpay][1])

So there can potentially be **multiple cost components**:

```text
Client payment
      │
      ├── Payment processing fee
      │
      ├── Escrow/service fee
      │
      └── Applicable taxes
      │
      ▼
Escrow
      │
      └── Payout/transfer-related charges
```

I **cannot honestly give you "Razorpay Escrow = 2% + X"**, because Razorpay doesn't publish that number publicly in the documentation I found.

And that's important for Mitfloww's economics.

### Ask Razorpay this exact question

> "What is the complete fee structure for RazorpayX Escrow for an online freelance marketplace? Please provide the payment collection fee, escrow account fee, escrow transaction fee, payout/disbursement fee, refund fee, and any minimum monthly/annual commitment."

Also ask whether **UPI collection into the escrow** has a different rate.

---

# 2. Stripe Connect is much more transparent

Stripe's current India pricing gives us actual numbers.

If **Stripe handles pricing for connected accounts**, Stripe says there is:

* **No additional Connect platform fee**
* No additional account fee
* No additional payout-volume/per-payout fee for the platform under that model

But normal payment processing fees still apply. ([Stripe][2])

If **Mitfloww handles pricing**, Stripe currently lists:

* **₹150 per monthly active connected account**
* **0.25% + ₹20 per payout**
* **0.25% of payout volume** for funds routing/platform management

in addition to the payment-processing economics. ([Stripe][2])

For Indian cards, Stripe's current standard pricing is **2% for cards issued in India** and **3% for cards issued outside India**. International payments are listed at **4.3%**, with an additional **2% currency-conversion fee** where applicable. ([Stripe][3])

So imagine a ₹20,000 transaction.

Very simplified:

```text
Client pays                  ₹20,000
        │
        ├── Stripe processing fee
        │
        ├── Connect/platform-related fee
        │
        └── payout fee
        │
        ▼
Freelancer
```

The exact total depends heavily on **which Connect pricing model Stripe approves for Mitfloww**.

### But remember our previous issue

Stripe Connect being technically capable of marketplace payments **doesn't mean Mitfloww can automatically use every Connect fund-flow model in India**.

Stripe Connect India is currently **invite-only**, so you'd need Stripe to approve Mitfloww's marketplace setup. ([Stripe][4])

---

# 3. PayPal

PayPal is considerably more expensive for an Indian business receiving international payments.

PayPal's current India merchant-fee page lists:

**4.40% + fixed fee** for receiving international commercial transactions. For INR, the fixed fee shown is **₹3**. ([PayPal][5])

So:

### $1,000 international payment

Conceptually:

```text
$1,000
 - 4.40%
 - fixed fee
 - potentially currency conversion cost
```

PayPal also states that currency conversion can add a markup; its India fee documentation currently lists **3% above the base exchange rate** for converting balances/payments received into another currency, and **4%** for other conversions. ([PayPal][5])

That's why I'd consider PayPal a **convenience/additional payment option**, rather than automatically making it Mitfloww's primary international payment rail.

---

# 4. But what about PayPal's escrow-like delayed disbursement?

This part is interesting.

PayPal's current documentation says its **Delayed Disbursement** lets you hold buyer funds before paying the seller. ([PayPal Developer][6])

But:

> You must be an **approved PayPal partner** and onboard sellers with the `DELAY_FUNDS_DISBURSEMENT` feature.

And there is a very important limitation:

> **Funds are automatically disbursed to the seller after 28 days.**

PayPal also says it deducts its fees from the seller's funds. ([PayPal Developer][6])

So it isn't a perfect equivalent of:

```text
Client pays
     ↓
Mitfloww holds for 2 months
     ↓
Client approves
     ↓
Release
```

You have a **28-day automatic-disbursement constraint**.

That's potentially a serious problem for Mitfloww projects that take longer than 28 days.

---

# 5. Here's the important comparison

|                                 | Razorpay Escrow                   | Stripe Connect                   | PayPal Delayed Disbursement           |
| ------------------------------- | --------------------------------- | -------------------------------- | ------------------------------------- |
| Marketplace                     | **Yes**                           | **Yes**                          | **Yes, with approval**                |
| Hold funds                      | **Yes**                           | Depends on approved Connect flow | **Yes**                               |
| Conditional release             | **Yes, escrow model**             | Payout timing/control            | Limited                               |
| Public escrow price             | **No**                            | Connect pricing published        | Normal PayPal fees published          |
| Extra platform/payout fees      | Need quote                        | **Yes, depending on model**      | Partner arrangement                   |
| India                           | **Yes**                           | Invite-only                      | India supports international payments |
| Special approval                | **Yes**                           | **Yes**                          | **Yes**                               |
| Automatic 28-day release        | No such limitation shown publicly | Depends on setup                 | **Yes**                               |
| Best fit for your advance model | **Potentially best**              | Potentially                      | Less ideal                            |

---

# 6. One thing I want to correct from our previous discussion

I previously described Razorpay Escrow as if it were necessarily a straightforward replacement for Route.

That's **too simplistic**.

Razorpay Escrow is a **separate bank-backed escrow arrangement**, not simply:

> "Turn on Escrow API and everything works."

Razorpay's own documentation says the use case goes through approval with its banking partners and requires an escrow agreement. ([Razorpay][7])

So the **price and exact operational model need to be negotiated/confirmed**.

---

# 7. What matters most for Mitfloww

Don't compare providers simply like:

```text
Razorpay = 2%
Stripe = 2%
PayPal = 4.4%
```

That's misleading for your business.

You need to compare the **total cost of moving ₹100 from client → escrow/marketplace → freelancer**.

For example:

```text
                ₹100 client payment
                       │
              Payment processing
                       │
                 Escrow/Connect
                       │
                  Platform fee
                       │
                    Payout
                       │
              Freelancer receives
```

Your real cost could be:

```text
Payment processing
+
Escrow/marketplace fee
+
Payout fee
+
Currency conversion
+
GST/tax on provider fees
+
Refund/dispute costs
```

---

# 8. This affects your Mitfloww platform fee

This is actually a **business-model question you should solve now**.

Suppose Mitfloww charges freelancers **10% platform fee**.

Project:

```text
₹20,000
```

Mitfloww revenue:

```text
₹2,000
```

But suppose payment infrastructure costs:

```text
₹500
```

Then your real gross revenue is:

```text
₹2,000
-  ₹500
───────
₹1,500
```

And that's before:

* GST/taxes
* refunds
* chargebacks
* fraud
* support
* payout costs
* infrastructure

So the payment-provider pricing isn't a minor implementation detail. **It directly affects Mitfloww's unit economics.**

---

## My recommendation right now

Before choosing the provider, I'd get **three actual commercial quotes**:

### Razorpay

Ask for:

> **RazorpayX Escrow + UPI + marketplace + conditional release + freelancer payout**

and request the **complete fee schedule**.

### Stripe

Ask for:

> **Stripe Connect India marketplace + platform-controlled pricing + delayed/manual payouts + platform fee collection**

and specifically ask whether Mitfloww's **freelance marketplace and advance-hold model** is supported for an Indian platform.

### PayPal

Ask for:

> **PayPal Marketplace + Delayed Disbursement + India-based platform + international buyers + freelancer payouts**

and ask whether the **28-day automatic disbursement** can accommodate Mitfloww's project lifecycle.

[Razorpay Escrow](https://razorpay.com/x/escrow-accounts/?utm_source=chatgpt.com) · [Stripe Connect India pricing](https://stripe.com/in/connect/pricing?utm_source=chatgpt.com) · [PayPal India merchant fees](https://www.paypal.com/in/business/paypal-business-fees?utm_source=chatgpt.com)

**The key point:** don't assume Razorpay Escrow is "normal Razorpay 2–3% + nothing." Its public documentation doesn't give us enough information to make that claim. Get the commercial quote first.

If you give me a **typical Mitfloww project amount** (e.g. ₹5,000 / ₹20,000 / ₹1 lakh) and your planned **platform fee %**, I can calculate a realistic fee waterfall for Razorpay vs Stripe vs PayPal so you can see exactly how much Mitfloww and the freelancer would receive.

[1]: https://razorpay.com/x/escrow-accounts/?r=rera_compliance_blog&utm_source=chatgpt.com "Escrow Account in India | Escrow Services | Razorpay Escrow"
[2]: https://stripe.com/in/connect/pricing?utm_source=chatgpt.com "Pricing information | Stripe Connect"
[3]: https://stripe.com/in/pricing?utm_source=chatgpt.com "Pricing & Fees"
[4]: https://stripe.com/in/connect?utm_source=chatgpt.com "Stripe Connect | Platform and Marketplace Payment Solutions"
[5]: https://www.paypal.com/in/business/paypal-business-fees?utm_source=chatgpt.com "PayPal Fees for Sellers | PayPal IN"
[6]: https://developer.paypal.com/platforms/checkout/delayed-disbursement/?utm_source=chatgpt.com "Delay disbursement | PayPal Developer"
[7]: https://razorpay.com/x/escrow-accounts/?source=know_more&utm_source=chatgpt.com "Escrow Account in India | Escrow Services | Razorpay Escrow"
