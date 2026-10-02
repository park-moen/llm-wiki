# Stripe API: The Balance Transaction object

> Source: https://docs.stripe.com/api/balance_transactions/object
> Collected: 2026-10-02
> Published: Unknown

## Attributes

* `amount` (integer): Gross amount of this transaction (in the smallest currency unit). A positive value represents funds charged to another party, and a negative value represents funds sent to another party.
* `net` (integer): Net impact to a Stripe balance (in the smallest currency unit). A positive value represents incrementing a Stripe balance, and a negative value decrementing a Stripe balance. You can calculate the net impact of a transaction on a balance by `amount` - `fee`.
* `type` (enum): Transaction type includes `payment`, `payment_refund`, `refund`, `payment_reversal`, and `payout_cancel`.
