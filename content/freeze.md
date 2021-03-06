Freeze Features
===============

The Ripple Consensus Ledger gives accounts the ability to freeze non-XRP balances, which can be useful to comply with regulatory requirements, or while investigating suspicious activity. There are three settings related to freezes:

* [**Individual Freeze**](#individual-freeze) - Freeze one counterparty.
* [**Global Freeze**](#global-freeze) - Freeze all counterparties.
* [**No Freeze**](#no-freeze) - Permanently give up the ability to freeze individual counterparties. Also gives up the ability to end a global freeze.

Because no party has a privileged place in the Ripple Consensus Ledger, the freeze feature cannot prevent a counterparty from conducting transactions in XRP or funds issued by other counterparties. No one can freeze XRP.

All freeze settings are independent of whether the balance is positive or negative. Either the currency issuer or the currency holder can freeze a trust line. In both cases, the balance on that trust line can only change in transactions that go directly from one party of the trust line to the other.


Individual Freeze
-----------------

The **Individual Freeze** feature is a setting on a trust line. When an account enables the Individual Freeze setting, the counterparty of that trust line can no longer send or receive issuances on the frozen trust line, except in transactions that go directly to and from the account itself. 

A gateway can freeze the trust line linking it to a counterparty if that counterparty shows suspicious activity or violates the gateway's terms of use.

An individual can freeze the trust line to a gateway. This has no effect on transactions between the gateway and other users. It does, however, prevent other accounts, including [hot wallets](gateway_guide.html#hot-and-cold-wallets) from sending that gateway's issued currency to the individual.

The Individual Freeze applies to a single currency only. In order to freeze multiple currencies with a particular counterparty, the account must enable Individual Freeze on the trust lines for each currency individually.

An account cannot enable the Individual Freeze setting if it has previously enabled the [No Freeze](#no-freeze) setting.


Global Freeze
-------------

The **Global Freeze** feature is a setting on an account. When an issuing account enables the Global Freeze feature, all counterparties can only send and receive the issuing account's funds directly to and from the issuing account itself. (This includes any [hot wallet](gateway_guide.html#hot-and-cold-wallets) accounts.)

It can be useful to enable Global Freeze on a gateway's [cold wallet](gateway_guide.html#hot-and-cold-wallets) if a hot wallet is compromised, or immediately after regaining control of a compromised issuing account. This stops the flow of funds, preventing attackers from getting away with any more money or at least making it easier to track what happened.

It can also be useful to enable Global Freeze if a gateway intends to migrate its cold wallet to a new Ripple account, or if the gateway intends to cease doing business. This locks the funds at a specific point in time, so users cannot trade them away for other currencies.

Global Freeze applies to _all_ currencies issued and held by the account. You cannot enable Global Freeze for only one currency. If you want to have the ability to freeze some currencies and not others, you should use different accounts for each currency.

An account can always enable the Global Freeze setting. However, if the account has previously enabled the [No Freeze](#no-freeze) setting, it can never _disable_ Global Freeze.


No Freeze
---------

The **No Freeze** feature is a setting on an account that permanently gives up the ability to freeze counterparties. A business can use this feature to treat its issued funds as "more like physical money" in the sense that the business cannot interfere with customers trading it among themselves. The NoFreeze setting has two effects:

* The issuing account can no longer use enable Individual Freeze on any counterparty.
* The issuing account can still enable Global Freeze to enact a global freeze, but the account cannot _disable_ Global Freeze.

The Ripple Consensus Ledger cannot force a gateway to honor the obligations that its issued funds represent, so giving up the ability to enable a Global Freeze cannot protect customers. However, giving up the ability to _disable_ a Global Freeze ensures that the Global Freeze feature is not used unfairly against some customers.

The No Freeze setting applies to all currencies issued to and from an account. If you want to be able to freeze some currencies but not others, you should use different accounts for each currency.

You can only enable the No Freeze setting with a transaction signed by your account's master key. You cannot use a [Regular Key](transactions.html#setregularkey) or a [multi-signed transaction](https://wiki.ripple.com/Multisign) to enable No Freeze.


# Technical Details #

## Enabling or Disabling Individual Freeze ##

### Using `rippled` ###

To enable or disable Individual Freeze on a specific trust line, send a `TrustSet` transaction. Use the [`tfSetFreeze` flag](transactions.html#trustset-flags) to enable a freeze, and the `tfClearFreeze` flag to disable it. The fields of the transaction should be as follows:

| Field                | Value  | Description |
|----------------------|--------|-------------|
| Account              | String | The address of your Ripple account. |
| TransactionType      | String | `TrustSet` |
| LimitAmount          | Object | Object defining the trust line to freeze. |
| LimitAmount.currency | String | Currency of the trust line |
| LimitAmount.issuer   | String | The Ripple address of the counterparty to freeze |
| LimitAmount.value    | String | The amount of currency you trust this counterparty to issue to you, as a quoted number. From the perspective of a gateway, this is typically `"0"`. |
| Flags                | Number | To enable a freeze, use a value with the bit `0x00100000` (tfSetFreeze) enabled. To disable a freeze, use a value with the bit `0x00200000` (tfClearFreeze) enabled instead. |

Set the `Fee`, `Sequence`, and `LastLedgerSequence` parameters [in the typical way](transactions.html#signing-and-sending-transactions).

Example of submitting a TrustSet transaction to enable an individual freeze using the [WebSocket API](rippled-apis.html#websocket-api):

```
{
  "id": 12,
  "command": "submit",
  "tx_json": {
    "TransactionType": "TrustSet",
    "Account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
    "Fee": "12000",
    "Flags": 1048576,
    "LastLedgerSequence": 18103014,
    "LimitAmount": {
      "currency": "USD",
      "issuer": "rsA2LpzuawewSBQXkiju3YQTMzW13pAAdW",
      "value": "110"
    },
    "Sequence": 340
  },
  "secret": "s████████████████████████████",
  "offline": false,
  "fee_mult_max": 1000
}
```

(**Reminder**: Never transmit your account secret to an untrusted server or over an insecure channel.)


### Using RippleAPI ###

To enable or disable Individual Freeze on a specific trust line, prepare a *Trustline* transaction using the [prepareTrustline](rippleapi.html#preparetrustline) method. The fields of the `trustline` parameter should be set as follows:

| Field        | Value  | Description |
|--------------|--------|-------------|
| currency     | String | The [currency](rippleapi.html#currency) of the trust line to freeze |
| counterparty | String | The [Ripple address](rippleapi.html#ripple-address) of the counterparty |
| limit        | String | The amount of currency you trust this counterparty to issue to you, as a quoted number. From the perspective of a gateway, this is typically `"0"`. |
| frozen       | Boolean | `true` to enable Individual Freeze on this trust line. `false` to disable Individual Freeze. |

The rest of the [transaction flow](rippleapi.html#transaction-flow) is the same as any other transaction.

Example JavaScript (ECMAScript 6) code to enable Individual Freeze on a trust line:

```js
const {RippleAPI} = require('ripple-lib');
 
const api = new RippleAPI({
  server: 'wss://s1.ripple.com' // Public rippled server hosted by Ripple, Inc.
});

const issuing_address = "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn";
const issuing_secret = "s████████████████████████████";
    //Best practice: get your secret from an encrypted config file instead
const address_to_freeze = "rUpy3eEg8rqjqfUoLeBnZkscbKbFsKXC3v";
const currency_to_freeze = "USD";

api.connect().then(() => {
  
  // Look up current state of trust line
  var options = {counterparty: address_to_freeze, currency: currency_to_freeze};
  console.log("looking up", currency_to_freeze, "trust line from", 
              issuing_address, "to", address_to_freeze);
  return api.getTrustlines(issuing_address, options);
  
}).then(data => {

  //Prepare a trustline transaction to enable freeze
  if (data.length != 1) {
    console.log("trustline not found, making a default one");
    var trustline = {
      currency: currency_to_freeze,
      counterparty: address_to_freeze,
      limit: 0
    };
  } else {
    var trustline = data[0].specification;
    console.log("trustline found. previous state:", trustline);
  }
  
  trustline.frozen = true;
  
  console.log("preparing trustline transaction for line:",trustline);
  return api.prepareTrustline(issuing_address, trustline);
  
}).then(prepared_tx => {

  //Sign and submit the trustline transaction
  console.log("signing tx:",prepared_tx.txJSON);
  var signed1 = api.sign(prepared_tx.txJSON, issuing_secret);
  console.log("submitting tx:", signed1.id);
  
  return api.submit(signed1.signedTransaction)
}).then(() => {
  return api.disconnect();
}).catch(console.error);
```


## Enabling or Disabling Global Freeze ##

### Using `rippled` ###

To enable Global Freeze on an account, send an `AccountSet` transaction with the [asfGlobalFreeze flag value](transactions.html#accountset-flags) in the `SetFlag` field. To disable Global Freeze, put the asfGlobalFreeze flag value in the `ClearFlag` field instead.

Example of submitting an AccountSet transaction to enable Global Freeze using the [WebSocket API](rippled-apis.html#websocket-api):

```
{
  "id": 12,
  "command": "submit",
  "tx_json": {
    "TransactionType": "AccountSet",
    "Account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
    "Fee": "12000",
    "Flags": 0,
    "SetFlag": 7,
    "LastLedgerSequence": 18122753,
    "Sequence": 349
  },
  "secret": "s████████████████████████████",
  "offline": false,
  "fee_mult_max": 1000
}
```

(**Reminder**: Never transmit your account secret to an untrusted server or over an insecure channel.)


### Using RippleAPI ###

To enable or disable Global Freeze on an account, prepare a **Settings** transaction using the [prepareSettings](rippleapi.html#preparesettings) method. The `settings` parameter should be an object set as follows:

| Field        | Value  | Description |
|--------------|--------|-------------|
| globalFreeze | Boolean | `true` to enable a Global Freeze on this account. `false` to disable Global Freeze. |

The rest of the [transaction flow](rippleapi.html#transaction-flow) is the same as any other transaction.

Example code to enable Global Freeze on an account:

```
const {RippleAPI} = require('ripple-lib');
 
const api = new RippleAPI({
  server: 'wss://s1.ripple.com' // Public rippled server hosted by Ripple, Inc.
});
api.on('error', (errorCode, errorMessage) => {
  console.log(errorCode + ': ' + errorMessage);
});

const issuing_address = "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn";
const issuing_secret = "s████████████████████████████";
    //Best practice: get your secret from an encrypted config file instead

api.connect().then(() => {

  //Prepare a settings transaction to enable global freeze
  var settings = {
    "globalFreeze": true
  }
  
  console.log("preparing settings transaction for account:",issuing_address);
  return api.prepareSettings(issuing_address, settings);
  
}).then(prepared_tx => {

  //Sign and submit the trustline transaction
  console.log("signing tx:",prepared_tx.txJSON);
  var signed1 = api.sign(prepared_tx.txJSON, issuing_secret);
  console.log("submitting tx:", signed1.id);
  
  return api.submit(signed1.signedTransaction)
}).then(() => {
  return api.disconnect();
}).catch(console.error);
```



## Enabling No Freeze ##

### Using `rippled` ###

To enable No Freeze on an account, send an `AccountSet` transaction with the [asfNoFreeze flag value](transactions.html#accountset-flags) in the `SetFlag` field. You must sign this transaction using the master key. Once enabled, you cannot disable No Freeze.

Example of submitting an AccountSet transaction to enable No Freeze using the [WebSocket API](rippled-apis.html#websocket-api):

WebSocket request:

```
{
  "id": 12,
  "command": "submit",
  "tx_json": {
    "TransactionType": "AccountSet",
    "Account": "raKEEVSGnKSD9Zyvxu4z6Pqpm4ABH8FS6n",
    "Fee": "12000",
    "Flags": 0,
    "SetFlag": 6,
    "LastLedgerSequence": 18124917,
    "Sequence": 4
  },
  "secret": "s████████████████████████████",
  "offline": false,
  "fee_mult_max": 1000
}
```

(**Reminder**: Never transmit your account secret to an untrusted server or over an insecure channel.)

### Using RippleAPI ###

To enable No Freeze on an account, prepare a **Settings** transaction using the [prepareSettings](rippleapi.html#preparesettings) method. Once enabled, you cannot disable No Freeze. The `settings` parameter should be an object set as follows:

| Field    | Value   | Description |
|----------|---------|-------------|
| noFreeze | Boolean | `true`      |

You must [sign](rippleapi.html#sign) this transaction using the master key. The rest of the [transaction flow](rippleapi.html#transaction-flow) is the same as any other transaction.

Example code to enable No Freeze on an account:

```
const {RippleAPI} = require('ripple-lib');
 
const api = new RippleAPI({
  server: 'wss://s1.ripple.com' // Public rippled server hosted by Ripple, Inc.
});
api.on('error', (errorCode, errorMessage) => {
  console.log(errorCode + ': ' + errorMessage);
});

const issuing_address = "rUpy3eEg8rqjqfUoLeBnZkscbKbFsKXC3v";
const issuing_secret = "s████████████████████████████";
    //Best practice: get your secret from an encrypted config file instead

api.connect().then(() => {

  //Prepare a settings transaction to enable no freeze
  var settings = {
    "noFreeze": true
  }
  
  console.log("preparing settings transaction for account:",issuing_address);
  return api.prepareSettings(issuing_address, settings);
  
}).then(prepared_tx => {

  //Sign and submit the trustline transaction
  console.log("signing tx:",prepared_tx.txJSON);
  var signed1 = api.sign(prepared_tx.txJSON, issuing_secret);
  console.log("submitting tx:", signed1.id);
  
  return api.submit(signed1.signedTransaction)
}).then(() => {
  return api.disconnect();
}).catch(console.error);
```


## Checking for Individual Freeze ##

### Using `rippled` ###

To see if a trust line has an Individual Freeze enabled, use the [`account_lines` method](rippled-apis.html#account-lines) with the following parameters:

| Field    | Value   | Description |
|----------|---------|-------------|
| account  | String  | The Ripple address of the issuing account |
| peer     | String  | The Ripple address of the counterparty account |
| ledger\_index | String | Use `validated` to get the most recently validated information. |

The response contains an array of trust lines, for each currency in which the issuing account and the counterparty are linked. Look for the following fields in each trust line object:

| Field        | Value   | Description |
|--------------|---------|-------------|
| freeze       | Boolean | (May be omitted) `true` if the issuing account has [frozen](freeze.html) this trust line. If omitted, that is the same as `false`. |
| freeze\_peer | Boolean | (May be omitted) `true` if the counterparty has [frozen](freeze.html) this trust line. If omitted, that is the same as `false`. |

Example WebSocket request to check for individual freeze:

```
{
  "id": 15,
  "command": "account_lines",
  "account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
  "ledger": "validated",
  "peer": "rsA2LpzuawewSBQXkiju3YQTMzW13pAAdW"
}
```

Example WebSocket response:

```
{
  "id": 15,
  "status": "success",
  "type": "response",
  "result": {
    "account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
    "lines": [
      {
        "account": "rsA2LpzuawewSBQXkiju3YQTMzW13pAAdW",
        "balance": "10",
        "currency": "USD",
        "freeze": true,
        "limit": "110",
        "limit_peer": "0",
        "peer_authorized": true,
        "quality_in": 0,
        "quality_out": 0
      }
    ]
  }
}
```

The field `"freeze": true` indicates that rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn has enabled Individual Freeze on the USD trust line to rsA2LpzuawewSBQXkiju3YQTMzW13pAAdW. The lack of a field `"freeze_peer": true` indicates that the counterparty has _not_ frozen the trust line.


### Using RippleAPI ###

To see if a trust line has an Individual Freeze enabled, use the [`getTrustlines` method](rippleapi.html#gettrustlines) with the following parameters:

| Field         | Value   | Description |
|---------------|---------|-------------|
| address       | String  | The Ripple address of the issuing account |
| options.counterparty  | String  | The Ripple address of the counterparty account |

The response contains an array of trust lines, for each currency in which the issuing account and the counterparty are linked. Look for the following fields in each trust line object:

| Field                | Value   | Description |
|----------------------|---------|-------------|
| specification.frozen | Boolean | (May be omitted) `true` if the issuing account has frozen the trust line. |
| counterparty.frozen  | Boolean | (May be omitted) `true` if the counterparty has frozen the trust line. |

Example JavaScript (ECMAScript 6) code to check whether a trust line is frozen:

```
const {RippleAPI} = require('ripple-lib');
 
const api = new RippleAPI({
  server: 'wss://s1.ripple.com' // Public rippled server hosted by Ripple, Inc.
});

const my_address = "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn";
const counterparty_address = "rUpy3eEg8rqjqfUoLeBnZkscbKbFsKXC3v";
const frozen_currency = "USD";

api.connect().then(() => {
  
  // Look up current state of trust line
  var options = {counterparty: counterparty_address, currency: frozen_currency};
  console.log("looking up", frozen_currency, "trust line from", 
              my_address, "to", counterparty_address);
  return api.getTrustlines(my_address, options);
  
}).then(data => {

  if ( data.length > 1)
     throw "should only be 1 trust line per counterparty+currency pair";
  
  if ( data.length === 0 ) {
    console.log("No trust line found");
  } else {
      var trustline = data[0];
      console.log("Trust line frozen from our side?", 
                  trustline.specification.frozen === true);
      console.log("Trust line frozen from counterparty's side?", 
                  trustline.counterparty.frozen === true);
  }
  
}).then(() => {
  return api.disconnect();
}).catch(console.error);
```


## Checking for Global Freeze and No Freeze ##

### Using `rippled` ###

To see if an account has Global Freeze and/or No Freeze enabled, use the [`account_info` method](rippled-apis.html#account-lines) with the following parameters:

| Field    | Value   | Description |
|----------|---------|-------------|
| account  | String  | The Ripple address of the issuing account |
| ledger\_index | String | Use `validated` to get the most recently validated information. |

Check the value of the `account_data.Flags` field of the response using the [bitwise-AND](https://en.wikipedia.org/wiki/Bitwise_operation#AND) operator:

* If `Flags` AND `0x00400000` ([lsfGlobalFreeze](ripple-ledger.html#accountroot-flags)) is _nonzero_: Global Freeze is enabled.
* If `Flags` AND `0x00200000` ([lsfNoFreeze](ripple-ledger.html#accountroot-flags)) is _nonzero_: No Freeze is enabled.

Example WebSocket request:

```
{
  "id": 1,
  "command": "account_info",
  "account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
  "ledger_index": "validated"
}
```

WebSocket response:

```
{
  "id": 4,
  "status": "success",
  "type": "response",
  "result": {
    "account_data": {
      "Account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
      "AccountTxnID": "41320138CA9837B34E82B3B3D6FB1E581D5DE2F0A67B3D62B5B8A8C9C8D970D0",
      "Balance": "100258663",
      "Domain": "6D64756F31332E636F6D",
      "EmailHash": "98B4375E1D753E5B91627516F6D70977",
      "Flags": 12582912,
      "LedgerEntryType": "AccountRoot",
      "MessageKey": "0000000000000000000000070000000300",
      "OwnerCount": 4,
      "PreviousTxnID": "41320138CA9837B34E82B3B3D6FB1E581D5DE2F0A67B3D62B5B8A8C9C8D970D0",
      "PreviousTxnLgrSeq": 18123095,
      "Sequence": 352,
      "TransferRate": 1004999999,
      "index": "13F1A95D7AAB7108D5CE7EEAF504B2894B8C674E6D68499076441C4837282BF8",
      "urlgravatar": "http://www.gravatar.com/avatar/98b4375e1d753e5b91627516f6d70977"
    },
    "ledger_hash": "A777B05A293A73E511669B8A4A45A298FF89AD9C9394430023008DB4A6E7FDD5",
    "ledger_index": 18123249,
    "validated": true
  }
}
```

In the above example, the `Flags` value is 12582912. This indicates that has the following flags enabled: lsfGlobalFreeze, lsfDefaultRipple, as demonstrated by the following JavaScript code:

```js
var lsfGlobalFreeze = 0x00400000;
var lsfNoFreeze = 0x00200000;

var currentFlags = 12582912;

console.log(currentFlags & lsfGlobalFreeze); //4194304
//therefore, Global Freeze is enabled

console.log(currentFlags & lsfNoFreeze); //0
//therefore, No Freeze is not enabled
```

### Using RippleAPI ###

To see if an account has Global Freeze and/or No Freeze enabled, use the [`getSettings` method](rippleapi.html#getsettings) with the following parameters:

| Field         | Value   | Description |
|---------------|---------|-------------|
| address       | String  | The Ripple address of the issuing account |

Look for the following values in the response object:

| Field         | Value   | Description |
|---------------|---------|-------------|
| noFreeze      | Boolean | (May be omitted) `true` if No Freeze is enabled. |
| globalFreeze  | Boolean | (May be omitted) `true` if Global Freeze is enabled. |

Example code:

```
const {RippleAPI} = require('ripple-lib');
 
const api = new RippleAPI({
  server: 'wss://s1.ripple.com' // Public rippled server hosted by Ripple, Inc.
});

const my_address = "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn";

api.connect().then(() => {
  //Look up settings object
  return api.getSettings(my_address);
}).then(settings => {
    console.log("Got settings for address",my_address);
    console.log("Global Freeze enabled?", (settings.globalFreeze === true) );
    console.log("No Freeze enabled?", (settings.noFreeze === true) );
    
}).then(() => {
  return api.disconnect();
}).catch(console.error);
```

# See Also #

[Gateway Bulletin GB-2014-02 New Feature: Balance Freeze](https://ripple.com/files/GB-2014-02.pdf)
