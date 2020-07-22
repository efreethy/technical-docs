# **Payment subscriptions**

Status: Draft
Last edited: Oct 10, 2018

## **Background**

Live events can be as expensive as they are exciting. 

Some orders, such as those containing membership plans, can cost thousands or even tens of thousands of dollars.

 In service of both the customer and our org partners, [company] will lessen the financial burden on the purchaser by supporting flexible** installment plans** on orders.


## **Important detail about naming**

Internally we have opted for the name payment subscription because it is consistent with its neighbors in the payments architecture. We also get more flexibility to support other forms of subscriptiond payments, like subscription plans.

It should be noted that these behaviors will be advertised to the Org and the Fan as **installment plans.**


## **What are the use cases?**

Some anticipated use cases for payment subscriptions include:

1. Orders containing plans (Nuggets full-season/half-season plan, flex plans, concert series, etc.) 
2. Membership renewals



## **What is a payment subscription?**

A **payment subscription** defines a timeline to pay off a **balance**. This timeline is used as a reference to generate invoices and subsequently make payments.

Payment subscriptions are tied to **orders **in our system. 

Each subscription:

* Describes a finite set of **billing periods, **which establish the cadence for invoicing and automatic payments.
* May also specify a down payment in the form of a **deposit.**
    * Deposits are invoiced immediately as due. Automatic payments are made immediately as a result.

## **Lifecycle**

Payment subscriptions have a pretty slim state machine.

* A subscription begins its life as **ACTIVE**
* If the order balance reaches zero by any means - the payment subscription is considered **COMPLETE**
* If a user chooses to cancel their payment subscription for whatever reason, it will be marked as **CANCELLED**
    * Other parts of the system which care about this change will listen over change capture.



## **Payment subscription properties**

* **order_id - **The order this payment subscription corresponds to. This order is used to look up the remaining balance.
* **start_date -** The day the payment subscription is instantiated.
* **total_periods -** Maximum number of periods over which the balance is paid
* **interval_unit - **The type of billing interval, eg: “month”, or “week”
* **interval_count - **Number of intervals that define the billing period, eg: 1 or 6
* **deposit - **Value to be paid up front on the total amount
* **status - **Current status of the subscription.
* **payment_method_id - **Payment method used to execute automatic payments



## **Bare bones model**

![image info](./images/subscriptions.png)



## **External interactions: async monitoring/change** data capture****

It can be helpful to think of payment subscriptions in terms of their role in a larger ecosystem of commerce behaviors.

Other parts of our system will be interested in the creation of new payment subscriptions.

More specifically, new payment subscriptions will trigger the creation of an initial invoice for the initial billing period.

* This invoice is given a due date that equal the end of the first billing period.
* This invoice is initially marked as DUE
* It has the potential to be marked PAST_DUE if the invoice is not paid before the end of the due date.


Additionally, an async cron-like job will inspect invoices that are due for payment, and execute an automatic payment on any invoice that is applicable.

After a payment is executed, the next invoice is generated using the payment subscription.

* Invoice amounts will be calculated using information from both the order and the payment subscription.
* The invoice amount will always equal
    * (order total - unmatched order payments) / remaining periods.
    * If a remaining balance is already zero, the invoice amount is zero, and the payment subscription is marked as COMPLETE.




# CDC Payment Flow Conceptual Design

![image info](./images/subscriptions2.png)

### Explanation

When a client wants to create a payment, they're actually creating a `Paymentsubscription` (`subscriptionPayment`?) entity, via an endpoint something like /subscription_payment.  Single payments will still be subscriptiond via this endpoint and will be a subscription of 1 installment, much like bank transfers or credit card payments behave.

The creation of this entity triggers the creation of the invoice for the deposit.  The creation of the invoice triggers the creation of the payment (for the deposit, it would be marked due immediately).  The payment creation stream consumer will only create payments for stream events where the invoice is past due.

The invoice creation stream consumer will also listen for payment creation events.  It will evaluate the payment subscription the payment belongs to, and will create a new invoice if applicable (i.e. the order still has a balance).

The last piece of the puzzle is a subscriptiond process that runs on a subscriptiond interval, and will update the state of invoices who's due dates are < the current date to past due.  This will be detected by the payment creation stream consumer, which will then create a payment.
