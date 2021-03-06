# Fees (Disambiguation) #

The Ripple Consensus Ledger is a decentralized ledger, secured by cryptography, and operated by a distributed peer-to-peer network of servers. This means that no one party, not even the Ripple, Inc., can charge a fee for access to the network.

However, the rules of the Ripple Consensus Ledger include several types of fees, including the [_transaction cost_](tx-cost.html) and [_reserve requirement_](reserves.html), which protect the ledger against abuse. (These neutral fees are not paid to anyone.) There are also several optional ways that users of the Consensus Ledger can collect fees from each other, both inside and outside the Ripple Consensus Ledger.

## In the Ledger ##

The _transaction cost_ (sometimes called the transaction fee) is a miniscule amount of XRP destroyed in order to send a transaction. This cost scales with the load of the network, which protects it from being overwhelmed with spam. See [Transaction Cost](tx-cost.html) for more information.

The _account reserve_ is a minimum amount of XRP that an account must possess. It scales with the number of objects the account owns in the ledger. This serves to disincentivize spam that would increase the size of the ledger. See [Reserves](reserves.html) for more information.

_Transfer fees_ are optional percentage fees that gateways can charge to transfer the currencies they issue to other accounts within the Ripple Consensus Ledger. See [Transfer Fees](https://ripple.com/knowledge_center/transfer-fees/) for more information.

_Trust line quality_ is a setting that allows an account to value balances on a trust line at higher or lower than face value. This can lead to situations that are similar to charging a fee. Trust line quality does not apply to XRP, which is not tied to a trust line.

## Outside the Ledger ##

While the above values are the only fees built into the ledger, people can still invent ways to charge fees outside the Ripple ledger. A common example is withdrawal or deposit fees charged by gateways, which are assessed when you use the gateway to get money into or out of the Ripple Consensus Ledger. This may cover the costs of transferring balances on traditional payment rails, or it may be in addition to those costs.

Many other fees are also possible. Businesses might charge for access, account maintenance, exchange services (especially when buying XRP on a private market instead of directly within the Ripple Consensus Ledger) and any number of other services. Always be aware of the fee schedule before doing business with any gateway or other financial institution.
