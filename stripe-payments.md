# Supporting payments with Stripe



## Overview

* Clarify the key concepts of a payment.
* Breakout some common payment scenarios on the [company] platform.
* Introduce Stripe, Stripe Accounts, and discuss the pros/cons of different account types when applied to [company]'s anticipated payment scenarios.

## So what is a payment anyway?

A payment, in the most general sense, describes a transfer of funds from a ***payer* **to a ***payee***.

**Key Concept: **We can think of both the payer and payee as **payment participants** on a payments platform. Participants can exchange funds because they have both registered with the platform.


## What do payments mean to [company]?

* The [company] platform will support a variety of *payment scenarios*.
* For each scenario, [company] must identify the relevant **payment participants** and facilitate a transfer of funds between them.
* [company] must satisfy additional business rules that surround payments, such as scoping inventory to owners, gating access on event day, meeting compliance requirements, etc.

## So what are these payment scenarios - really?

In most scenarios, the **payment participant **is one of:

*  ***[company] user***
*  ***[company] org **(an org on the [company] platform, like [customer])***
*  ***The [company] platform***

And in future scenarios, the **payment participant **could include:

* ***Third party reseller***
* ***Tertiary org***
* TBD

### Examples

**Vanilla payment scenario - an order purchase :**

* The payer is a [company] user, and the payee is a [company] org hosting an event.
* Funds flow from the [company] User to the [company] Org.
* In exchange for the funds, the [company] User is granted event access on event day.
* The [company] platform will ensure the payment is processed:
    * In the happy path, [company] will scope the user to their selected inventory, to be used later on event day for granting access.
    * In some failure cases, [company] must ensure funds are reversed, in addition to relinquishing any assigned inventory.

**Other relevant payment scenarios:**

* Refunds: The payer and payee may be switched. Funds will return to the [company] User, taken from the [company] org.
* Platform fee: The [company] platform might be an additional payee.
* Peer to Peer resale: The payer and payee could both be [company] Users
* Group Sales, B2B payouts: The payer and payee could both be [company] orgs


**PAYMENT SCENARIO BREAKOUT**

|Urgency	|Scenario	|Payer → Payee	|
|---	|---	|---	|
|Live launch	|Traditional payments applied to orders	|[company] User → [company] Org	|
|Live launch	|Order Refunds, Disputed Charges	|[company] Org → [company] User	|
|Live launch	|Optional [company] platform fee	|[company] User → [company] Platform	|
|Live launch	|Canceled Events	|[company] Org → [company] User                              
and/or ([company] Platform → [company] Org)
	|
|	|	|	|
|Post-launch	|Peer to peer ticket resales	|[company] User → [company] User	|
|Post-launch	|Tools for B2B payout 
(eg: LA [team] send the 49ers a proportion of total ticket sales)	|[company] Org → [company] Org    
         
and/or ([company] Org → Tertiary Org)
	|
|Post-launch	|Funds corresponding to a purchase made on a third party site, like StubHub	|Third party reseller → [company] Platform	|


A couple things to note about the above:

*  We see that [company] Users, [company] Orgs, and the [company] Platform all have the potential to be either the payer or the payee.
* Funds can flow in arbitrary directions.



## Enter Stripe

As a payments platform, [Stripe](https://stripe.com/) enables the transfer of real money between pre-configured account holders. These accounts can be tied to **payment participants** in our system.

* Note: The payer does not need to be connected to a stripe account** **in all cases, and can be the source of one-off charges. 

The most fundamental capability of Stripe is the **Stripe Account**:

* Stripe Accounts can be connected to [real sources of money,](https://stripe.com/payments/payment-methods-guide#availability) known as payment methods. These include:
    * Debit / Credit cards
    * Direct deposits
    * Apple Pay
    * Android Pay
    * Foreign payments
* Payments can be created which move funds across Stripe accounts
* Stripe accounts are stateful and hold a balance.
* Stripe Account balances can be paid out to the owner via their preferred payment method. Payout scheduling is configurable through the API. Instant payouts are possible.
* For refunds, the Stripe API can reverse the transfer of funds for a particular payment.
* For disputes, Stripe will automatically handle the resolution of funds between participants
* Stripe has a webhook event system which notifies API customers who might need to complete additional tasks based on updated charges.

At [company], any entity capable of being a **payment participant** can be granted a Stripe Account by our System. In the future, if [company] chooses to integrate with other payment platforms, these entities can be the **payment participants **of other payment platforms.

## Stripe Account Tiers

Stripe exposes Stripe Accounts through a product called [Stripe Connect](https://stripe.com/docs/connect). Pricing can be found [here](https://stripe.com/us/connect/[entity]).

Stripe Connect offers three different tiers of Stripe Accounts. Each category varies widely in the user registration experience.
[Image: Image.jpg]
### 1. Standard Account

Account holders configure this account independently with Stripe. Later on, in the [company] platform, account holders authorize [company] platform to apply payments to/from this account.
Pros

* Standard accounts are the easiest to integrate
* Stripe account management happens on Stripe, which provides a [powerful management dashboard.](https://stripe.com/docs/dashboard)

Cons

* The platform cannot schedule payouts
* No instant payouts
* No direct account debits
* With Standard accounts, the platform assumes fraud and dispute liability when using destination charges
* Requires an additional out of band step with Stripe. This could be fine for organizations, but definitely not a burden we would want for our general consumers.

### 2. Express Account

Account holders begin their journey in the [company] platform and are redirected to a branded flow on Stripe. On Stripe they will provision their Stripe account details and are later redirected back to the [company] platform on completion.

* Pros:
    * Express accounts are flexible enough to support most payment scenarios
    * Works ideally for payments made between our platform and [company] organizations,  or even third-party/tertiary organizations, as this is something that could be a one-time onboarding step.
    * Stripe account management happens on Stripe, which provides a [powerful management dashboard.](https://stripe.com/docs/dashboard)
* Cons:
    * Express account holders must be based in the US and must have an SMS enabled phone number for registration.
    * Stripe is not a hidden concept in this case - users are sent through an explicit account setup flow with Stripe. We get a lot of control over the branding of this process, however, it is made clear to the user that they are activating an additional Stripe account. 
    * With Standard accounts, the platform assumes fraud and dispute liability when using destination charges, but the user (merchant) is responsible when using direct charges.

The big drawback to this approach is in the general consumer case. Any consumers who wish to pay / re-sell tickets would need to go through this additional flow to set up a Stripe account. Payments could only be processed once this account is established, otherwise, they would need to make anonymous purchases.


### 3. Custom Account

With a Custom Account, the Stripe account setup process is completely governed by [company].

Pros

* Custom accounts are really the Swiss army knife of account types, as they can support any payment scenario. 

* Stripe is virtually invisible to the account owner.
* The real long-term value for this account type lies in its ability to support the general consumer case since it permits a single registration flow for users. There is no additional flow to activate a Stripe account.

Cons

* The [company] platform is responsible for building all front-end behaviors and collecting all required account information.
* [company] is ultimately responsible for any loss due to fraud.
* Stripe account management would happen on the [company] platform through the Stripe API, so we would need to build out additional tooling.

