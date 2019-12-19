---
title: "Tax Considerations"
date: 2019-12-18T10:41:53-08:00
author: <a href="https://twitter.com/JeremyRubin">Jeremy Rubin</a>
---

In this article, we'll discuss basic tax law around payments, establish an analogy between
OP_CHECKTEMPLATEVERIFY and traditional certified cheques, and then discuss the relevant tax
consequences.


In the "legacy" financial world, a cheque[^3]  is commonly used to move funds from
financial institutions in lieu of inconvenient physical cash transfers. To date, Bitcoin has not
needed the concept of a cheque because Bitcoin transactions are direct transfers with no
intermediary, i.e., as cash is to a cheque.


The introduction of OP_CHECKTEMPLATEVERIFY enables a new kind of payment on Bitcoin that more
closely resembles a cheque than a cash transfer. Any new form of payment may have differences in tax
treatment.


To start, what is a cheque, exactly? A cheque is an endorsed instrument instructing a bank to transfer funds
from one account to a recipient. A cheque has a few critical pieces of information:

- An account number
- An issuing bank or & drawee (by Bank ID)
- A signature from the account holder
- An amount of money
- A recipient or payee (by name)
- A date on which the cheque is made (and thereafter redeemable, if post-dated)
 
The cheque, once written, is then presented to the drawee bank by the payee (or by the payee's bank)
at some arbitrary later time where it is redeemed for the amount of money specified[^4].


Generally, the form of payment - money, cheque, or property - is irrelevant for cash basis
taxpayers[^1]. [Notice 2014-21](https://www.irs.gov/pub/irs-drop/n-14-21.pdf) provides that bitcoin is
property[^5], and it also states, regarding employment taxes, that,
*"[g]enerally, the medium in which remuneration for services is paid is immaterial to the
determination of whether the remuneration constitutes wages for employment tax purposes."*
While the medium of payment is generally immaterial, a deduction is only allowed for the
taxable year in which the amount is actually paid. A taxpayer on the cash method of accounting
cannot deduct an amount by issuing a promissory note[^6]. A promissory note is simply a promise to pay
cash rather than a payment of cash[^7]. Furthermore, while cash basis taxpayers are subject to the
doctrine of constructive receipt of income, the corollary doctrine for deductions - i.e.,
constructive payment -  is not recognized for federal income tax purposes.


While the above makes clear that it doesn't matter if we're using a cheque or Bitcoin for a payment,
it's worthwhile to compare the form of the payment contract in Bitcoin to that of a cheque.

In a traditional Bitcoin transaction, the constituent parts are the same as a cheque. What differs
is that there is no asynchronous "at some arbitrary later time" redemption step required by the
bitcoin recipient to receive the funds - assuming no time lock, bitcoin is treated as sent and
received at the same time for tax purposes.

You could emulate this asynchronous step in Bitcoin by sending to a user a "pre-signed" transaction
which has not yet been broadcast to the Bitcoin network. Then, when the user wants the funds, they
could broadcast the transaction to the network. Functionally, this would be almost identical to a
cheque. The issue with this technique is that it's possible for the funds sent in such a cheque to
be "double-spent", that is, for the accounts to be insolvent against the obligation and the cheque
bounces.

As noted above, bank accounts may have insufficient funds in an account to redeem a cheque. To
address the issue of insufficient funds for traditional cheques, banks introduced a notion of a
"certified" cheque. In a certified cheque, the bank attests to the solvency of the account and
guarantees sufficient funds when redeemed when the cheque is written. As un-certified cheques are
already treated as actual-cash, the legal distinction between a certified and uncertified cheque is
irrelevant for this document. Given the difficulties in catching and enforcing the law against
writers of fraudulent cheques, it is important to make clear that we are analogizing to the less
fraught certified cheque.

A transaction making use of OP_CHECKTEMPLATEVERIFY is a direct analog for a certified cheque. When
using OP_CHECKTEMPLATEVERIFY, first a sender creates (but does not sign & broadcast) a *collective root*
transaction paying multiple recipients, and *branches* to *individualized leaf* outputs. Next, the
sender must communicate all parts of the *tree* of transaction data needed by the recipient to claim
their bitcoin, or else they may not have sufficient information to ensure the payment. The sender
could collect digital signatures from the recipient for receipt of the transaction tree data before
making the payment, or use standard email notification agreements. Exchange of such signatures can
happen automatically in application code without user interaction. This notice requirement is not
unique to payments made using OP_CHECKTEMPLATEVERIFY, any Bitcoin transaction may require such
notice. This is similar to a certified cheque, which much be transferred physically to the recipient
(and unlike a wire, which is received automatically). Then, the sender signs and broadcasts the
transaction.

Similar to regular Bitcoin transactions, receipt is guaranteed after a reasonable number of
confirmations on the root transaction; however, recipients may individually defer creating the
outputs until fees are lower. At this point, recipients are in constructive receipt of income at
this point (because actual receipt is at their option), while sender will have a current deduction.
This is equivalent to issuing a certified cheque and redeeming it later. Typically, a Bitcoin
transaction making use of  OP_CHECKTEMPLATEVERIFY would be immediately spendable by the recipient if
the recipient were willing to pay the current market price for transaction fees. As such, it would
be considered paid by the sender, and actually (or constructively) received by its recipients after
a reasonable number of confirmations.

Thus, we've established a strong analogy matching cheques and Bitcoin payments made using
OP_CHECKTEMPLATEVERIFY. This establishes an intuitive and common sense treatment of
OP_CHECKTEMPLATEVERIFY payments which is in accordance with long standing precedent for payments by
cheque such as *Estate of Modie J.  Spiegel v. Commisisoners*[^8]. Simply put,
OP_CHECKTEMPLATEVERIFY works just like you would expect it to.



*Pleae Note: This material is presented and prepared solely for informational purposes, and as such, should not be relied on for legal, tax, accounting, or other purposes. Consult your own advisors before making any transactions using this information.*

[^1]: If payment is made using property such as bitcoin, the payor may realize a gain or loss if the obligation is not equal to the payor’s adjusted basis in the property. See Notice 2014-21, Q&A-6.
[^3]: Also spelled *check*. British spelling used to disambiguate from check as it is used in
  Bitcoin verification.
[^4]: Assuming, of course, that the account has sufficient funds.
[^5]: Notice 2014-21, Q&A-1 (“For federal tax purposes, virtual currency is treated as property. General tax principles applicable to property transactions apply to transactions using virtual currency.”).
[^6]: Helvering v. Price, 309 US 409 (1940) (taxpayer's own note not equivalent of cash). Note, however, that an expense paid with funds borrowed from a third party can give rise to a current deduction.
[^7]:  A promissory note is differentiated from a cheque as a cheque is
considered actual cash, i.e., a bounced cheque can be considered criminal fraud similar to using
counterfeit bills, whereas failure to pay a promissory note could be considered only a breach of an
agreement (contractually binding or not). 
[^8]: It is well established that a check constitutes payment by a cash basis taxpayer. See
    Spiegel's Estate v. CIR, 12 TC 524 (1949) (acq.). Rev. Rul. 54-465, 1954-2 CB 93 (holding "[a]
    charitable contribution in the form of a check is deductible in the taxable year in which the
    check is delivered provided the check is honored and paid and there are no restrictions as to time
    and manner of payment thereof.").

    Because a certified cheque is guaranteed to be honored and paid, we may also ignore this distinction
    when considering a OP_CHECKTEMPLATEVERIFY certified cheque.

    The only consideration to make is if there are additional restrictions as to the time and manner of
    payment.  It is possible to post-date a cheque using OP_CHECKTEMPLATEVERIFY, which would delay
    payment as well as the time when a recipient could access their funds[^12].

    For now, let's assume that no additional encumbrance is placed on a OP_CHECKTEMPLATEVERIFY
    transaction[^10]. In such a case, the sender can consider the funds as paid as soon as the transaction is
    confirmed. The sender may also use this time for computing their gains and losses from the
    transaction.

    Moreover, the day following the day that a root transaction is confirmed is the date on which the
    recipient’s holding period begins[^9], even if the individual UTXO is not redeemed until later.





[^9]:  [IRS FAQs, Q&A-5](https://www.irs.gov/individuals/international-taxpayers/frequently-asked-questions-on-virtual-currency-transactions)

[^10]: When time locks get involved, the tax issues become less certain. While lock times are
  not a unique feature of OP_CHECKTEMPLATEVERIFY transactions, the congestion control motivation
  makes it likely that the features will be used together in common practice. In principle, the
  correct application of timelocks and contractual agreements could allow a sender and recipient to
  optimize for purposes of taxable income and loss realization. 

    A Bitcoin transaction which is subject to a time lock specified by the sender may be deductible
    at the time it is confirmed, or it may constitute a prepayment. Generally, the determination is
    made by applying the three-pronged test for deductibility provided by Rev. Rul. 79-229[^11].
    The outcome in applying the three-pronged test to a time locked payment is highly fact
    dependent; generally, the ruling provides that expenditures for prepaid amounts are allowable
    if:
    1. the expenditure represented a real "payment" and not a deposit;
    2. the prepayment was
    made for a business purpose "and not merely for tax avoidance;" and
    3. there was no "material
    distortion of income."

    The IRS identified as factors "indicative of a deposit" a contract calling for an indefinite
    quantity, a right to refund, or a right to substitute other products.

    Thus, a bitcoin transaction subject to a time lock may be deductible if it is not a deposit, it
    is made with a business purpose, and it does not materially distort the income of the sender;
    however, a significant transaction that is treated by its sender as paid in a taxable year
    preceding the taxable year(s) in which the time lock expires may be challenged by the IRS.

[^11]: Grynberg v. Commissioner, 83 T.C. 255, 265 (1984) (the three-pronged test is generally applicable to deductions claimed by cash-basis taxpayers).
[^12]: See Spiegel's Estate v. CIR, 12 TC 524 (1949) (acq.). Rev. Rul. 54-465, 1954-2 CB 93.


*We gratefully acknowledge the comments provided by [Jim Calvin](https://twitter.com/Jim_Calvin).*
