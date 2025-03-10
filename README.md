# Introduction

**TonAPI.io** is an API for [TON blockchain](https://ton.org) developed by [Tonkeeper](https://tonkeeper.com) team.
It allows to work with indexed blockchain information via API.

You may also check [Tonkeeper documentation](https://github.com/tonkeeper/ton-connect) for interactions with user and performing user requests with wallet.

It is recommended to subscribe our [Telegram channel](https://t.me/tonapi_announcements) for announcements.

# Report a problem

In case of any problems with API or explorer, please submit an issue on github:
https://github.com/tonkeeper/tonapi/issues


# Registering your API key

At the moment you need special key to be able to use TonAPI, otherwise you requests will be limited.

To obtain special key which we call **serverSideKey** or **clientSideKey** in this doc - you need to use telegram bot https://t.me/tonapi_bot

Bot support two commands: **/get_server_key** and **/get_client_key**.

Tonapi can be used both from client side as well as from server side. From code perspective there is not much of a difference, but its important to not use **serverSideKey** anywhere in client side, and at the same time to use **clientSideKey** only on client side. The reason for this is because client side key has additional limitations per IP, while serverside key can be banned in case of large amount of flood requests to the api, so its usage should be limited by the developer.

# Performing API requests
Once you have API key you can perform simple requests to the api.

One of the basic methods is **/v1/blockchain/getAccount**

So you can make an **GET** http request to the url
```
https://tonapi.io/v1/blockchain/getAccount?account=EQCD39VS5jcptHL8vMjEXrzGaRcCVYto7HUn4bpAOg8xqB2N
```
But as mentioned above there an **Authorization** header should be passed to access the method:
```
Bearer eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9...
```
Here is an **javascript** code example to perform such request:
```javascript
fetch("https://tonapi.io/v1/blockchain/getAccount?" + new URLSearchParams({
    account: 'EQCD39VS5jcptHL8vMjEXrzGaRcCVYto7HUn4bpAOg8xqB2N',
}), {
    method: 'GET', 
    headers: new Headers({
        'Authorization': 'Bearer '+serverSideKey, 
    }), 
})
```
Also take a look at the same example but using **Go**:
```go
req, err := http.NewRequest("GET", "https://tonapi.io/v1/blockchain/getAccount", nil)
if err != nil {
    log.Println(err)
    os.Exit(1)
}

q := req.URL.Query()
q.Add("account", "EQCD39VS5jcptHL8vMjEXrzGaRcCVYto7HUn4bpAOg8xqB2N")
req.URL.RawQuery = q.Encode()

req.Header.Add("Authorization", "Bearer "+serverSideKey)

// Send req using http Client
client := &http.Client{}
resp, err := client.Do(req)
if err != nil {
    log.Println("Error on response.\n[ERROR] -", err)
}
defer resp.Body.Close()
```

# Using an SDK
To make things simpler for developers we introduced an SDK: https://github.com/startfellows/tonapi-sdk-js

Also since tonapi is build with **swagger** you can generate SDK for any language you prefer. Please use swaggerfile available on this URL: https://tonapi.io/swagger/swagger.json

# Authorization

Tonapi helps authorize user wallets via TON wallets. At the moment only [Tonkeeper](https://tonkeeper.com) is supported.

Authorization should be performed in 2 steps, redirect user to the auth page, and exchange **authToken** to regular API token and check ton account ownership.

## Serverside and clientside flows

Tonapi can be used both for the client side flow as well as for server side flow. This means that apps which doesnt have any backend can user tonapi authorization.

From code perspective there is not much of a difference, but its important to not use serverside token anywhere in client side, and at the same time to use clientside tokens only on client side.

The reason for this is because client side token has additional **limitations per IP**, while serverside token doesnt have such limitations but can be **banned in case of large amount of flood requests** to the api, so proper flood control and limits should be set by the developer.

### 1) Redirect user to the auth page

To perform auth user must be redirected to special page where auth can be checked
tonapi.io/login?\{params\}

**Params supported:**

- **redirect_url** – _[optional]_ _string_, url where user will be redirected after successful auth
- **callback_url** – _[optional]_ _string_, url which will be called from backend in after successfull auth
- **app_id** – _string_, identifier of the app. Name and icon of the app will be used on the authorization page. (not supported yet)

One of the params **redirect_url** or **callback_url** must be passed. Please note that **authToken** wich you will get after authorization flow is **ONE TIME USE**, **SHORT LIVING** token wich should be exchanged to persistent token serverside via tonapi.io/v1/oauth/getToken. Just receiving authToken is not a proof of successful user authorisation and can be possibly swapped or be stolen by attacker.

In case of success the **callback_url** or **redirect_url** will be triggered with following GET params added:

- **success** – boolean, true in case if auth was successfully performed (not supported yet)
- **auth_token** – _[optional]_ _string_, one-time-use token
- **error_code** – _[optional]_ _string_, in case of success=false short text code of error (not supported yet)
- **error_text** – _[optional]_ _string_, in case of success=false text human readable description of error (not supported yet)

**Examples:**

```json
{
  "success": true,
  "auth_token": "abcd..."
}
```

```json
{
  "success": false,
  "error_code": "auth_rejected",
  "error_text": "User canceled authorization"
}
```

### 2) Fetching persistent token via tonapi.io/v1/oauth/getToken method.

After successfully obtaining **auth_token** via process described below /auth method should be called from server side to check that the **auth_token** is valid.

Authorization header must be passed to this method the same way as any other methods in tonapi.io API. Token can be obtained with t.me/tonapi_bot telegram bot. [Learn more about serverside and clientside flows.](#serverside-and-clientside-flows)

Example header:

```
Authorization: Bearer AppTokenHere
```

**Serverside auth header:**

```javascript
var options = {
  host: "tonapi.io",
  path: "/v1/nft/getCollections",
  headers: {
    Authorization: "Bearer " + serverSideAppToken,
  },
};
http.request(options, () => {}).end();
```

**Clientside auth header:**

```javascript
var options = {
  method: "post",
  headers: new Headers({
    Authorization: "Bearer " + clientSideAppToken,
  }),
};
fetch("https://tonapi.io/v1/nft/getCollections", options);
```

There are two types of AppTokens that can be generated by t.me/tonapi_bot, serverside token and clientside token.

Following POST params needed by this method:

- **auth_token**, _string_, the token which was returned by the method below
- **rate_limit**, _number_, request per seconds
- **token_type**, _string [client, server]_, type of token which will be used to indicate the app

**Examples:**

```json
{
    "success": true,
    "user_token": "abcd...",
    "address": "EQrt...s7Ui",
    "pubkey": "Pub6...2k3y", // base64-encoded Ed25519 public key
    "signature": "Gt562...g5s8D=", // base64-encoded ed25519 signature
    "wallet_version": "v4R2", // supported values: "v3R1", "v3R2", "v4R1", "v4R2"
    "client_id": "abc"
}
```

```json
{
    "success": false,
    "error_code": "auth_rejected",
    "error_text": "User canceled authorization"
}
```

## Decentralised proof of ownership

It is possible to check proof of ownership, without fully relying in TONAPI. Here is the example of code needed to check signature and be sure that user have access to provided wallet.
https://github.com/tonkeeper/ton-connect/blob/main/tonconnect-server/src/TonConnectServerV1.ts#L36

## OAuth demo

Check out simple demo of Authorization flow:

[View Demo](https://tonapi-oauth.herokuapp.com/)

***
```javascript
Quick start guide:

1) git clone git@github.com:startfellows/tonapi-oauth-demo.git
2) cd tonapi-oauth-demo
3) yarn
4) yarn start
```

Look at the source code for more [details](https://github.com/startfellows/tonapi-oauth-demo/blob/master/src/App.tsx)

# Events

Events describe changes to the account balances in TON. Each event consists of one or more actions. Common transfers of coins and tokens are single-action events, while a trade on DEX produces a multi-action event.

Each event provides information from the perspective of all involved accounts. For example, an [NftItemTransfer](#NftItemTransfer) event will be returned for the sender’s account, recipient’s account and for the token address.

Events are offered in two flavors:

* Type [Event](#Event) contains complete set of all the actions and fees for all parties involved.
* Type [AccountEvent](#AccountEvent) contains a subset of actions relevant to a given account. Instead of a list of fees, it contains a single Fee object describing fees paid by the account.


## Events vs transactions

TON blockchain consists of _contracts_ that send _messages_ to each other and update their own states via _transactions_. These are low-level building blocks for the wallets, tokens and decentralized apps.

_Events_ and _actions_ are higher level abstractions that combine together multiple messages and transactions in a single entity that is easier to understand. For example, a single action `NftSend` consists of three transactions: (1) authentication on the sender’s wallet, (2) ownership update on the NFT contract, (3) notification sent to the new owner.

Transactions and messages form a tree, while events are flat lists of actions.


## Understanding fees

TON network has a complex system of fees. Sometimes both the sender and the recipient are paying fees for storage (_rent_) and processing costs (_gas_). When an additional contract is deployed, wallets typically send extra coins to cover the costs and initial balance of a new contract as _deposit_, while receiving unused back as _refund_.

One account may produce multiple actions while paying common fees for its storage and gas. To avoid any confusion, on API response for an event there can be return Array of [Fee](#Fee) objects, each for every account involved.

The fees are broken down in the following slots:

* **total**: total amount of fees paid by the account during the event. Calculated as `gas`+`rent`+`deposit`-`refund`.
* **gas**: total amount of coins spent for the processing of the messages by the account over all transactions within the event.
* **rent**: total amount of coins paid for the storage rent by the account.
* **deposit**: total amount of coins transferred by the wallet to cover the costs of the entire transaction chain. Some of these are left as balances of the newly created contracts. 
* **refund**: excess coins from the `deposit` that were returned to the sender after the operation is completed.

Note: sometimes `refund` may exceed `deposit` and the `total` fees could become negative. This may happen e.g. when a subscription contract is destroyed and its balance is returned to the merchant.


## Presenting events

To display the events correctly it is necessary to filter out actions that are not related to the account. This is done automatically by the API when you request [AccountEvents](#AccountEvent) instead of plain [Events](#Event).

The user’s wallet should check whether the transfer is incoming or outgoing using the sender/recipient addresses.


## Failed actions

Actions may fail and the appropriate messages may or may not bounce. Failure is indicated by the `status` field in each action. Bouncing is not indicated explicitly, but is reflected in `refund` field in the fees:

* If the message has failed and bounced, then the action is considered as not happened; the TONs sent are marked within the `deposit` in the `Fee` object and returned TONs are specified in the `refund` field.
* If the message has failed, but not bounced, then action is considered as not happened; the TONs sent are marked within the `deposit` in the `Fee` object.
* If the message is sent as non-bounceable to the uninitialized contract, then the action is successful.

## Versioning

Each version of API guarantees a fixed set of events.

E.g. if v1 does not recognize usage of a DEX, it will return a simple `JettonTransfer` with a recipient address being the DEX contract. When v2 adds `JettonPlaceOnSale` action, old clients will still render plain transfers.

From the API perspective all the newer actions are reformatted in terms of older actions on the fly for compatibility.

## Definitions

### Event

Complete description of the event for all accounts involved.

This event can be returned for any TON contract address: wallet account, NFT item, subscription contract, sale contract etc.

```json
{
    "event_id":    "<event-id>",
    "timestamp":   15612039230,   // timestamp of the initial transaction
    "fees": [Fee],                // 
    "actions": [
        Action<TonTransfer>,
        Action<TonDeploy>,
        Action<NftCollectionCreate>,
        Action<NftCollectionTransfer>,
        Action<NftItemCreate>,
        Action<NftItemTransfer>,
        Action<NftSalePlace>,
        Action<NftSaleFulfill>,
        Action<JettonMint>,
        Action<JettonTransfer>,
        Action<JettonBurn>,
        Action<SubscriptionCreate>,
        Action<SubscriptionCancel>,
        Action<SubscriptionPayment>,
    ]
}
```

### AccountEvent

Filtered-down description of the event for a given account. Contains a subset of actions that are relevant to the given account.

This event can only be returned for wallet contracts. If you want to receive events for a given token or a sale contract, request [Events](#event) instead.

```json
{
    "event_id":    "<event-id>",
    "timestamp":   15612039230,   // timestamp of the initial transaction
    "account":     Account,
    "fee": Fee,
    "actions": [
        Action<TonTransfer>,
        Action<TonDeploy>,
        Action<NftCollectionCreate>,
        Action<NftCollectionTransfer>,
        Action<NftItemCreate>,
        Action<NftItemTransfer>,
        Action<NftSalePlace>,
        Action<NftSaleFulfill>,
        Action<JettonMint>,
        Action<JettonTransfer>,
        Action<JettonBurn>,
        Action<SubscriptionCreate>,
        Action<SubscriptionCancel>,
        Action<SubscriptionPayment>,
    ]
}
```

### Fee

```json
{
    "account":   Address,   // account that paid the fees
    "total":     "100",     // = gas+rent+deposit-refund
    "gas":       "80",      // nanocoins cost of gas
    "rent":      "10",      // nanocoins cost of rent
    "deposit":   "110",     // nanocoins sent with the message from the account
    "refund":    "100",     // nanocoins returned to the account as a response msg or a bounced msg
}
```

### Action

```json
{
    "type":     "<action-type>",
    "status":   ActionStatus,
    "<action-type>": TonTransfer | TonDeploy | NftCollectionCreate...
}
```

### ActionStatus

Action could be pending, completed or failed.

```json
ActionStatus = "pending" | "ok" | "failed"
```

Pending status is for actions that map to queued outgoing messages that are not yet processed by the destination contract and shardchain. Pending status may also belong to all the actions when the event is in the mempool.

Failed action maps on a failed transaction. Such transaction may or may not bounce the message: the resulting difference in coins is reflected within the [Fee](#Fee) structure.



### TonTransfer

```json
{
    "amount":       "100000000",        // nanocoins
    "recipient":    Account,   // address of the recipient
    "sender":       Account,     // address of the sender
    "payload":      "<hex-encoded payload>",
    "comment":      "Hello, world", // optional
}
```


### TonDeploy

```json
{
    "amount":       "100000000",        // nanocoins
    "recipient":    Account,   // address of the recipient
    "sender":       Account,     // address of the sender
    "state_init":   "<hex-encoded stateinit>", // recipient’s StateInit
    "payload":      "<hex-encoded payload>",
    "comment":      "Hello, world", // optional
}
```


### NftCollectionCreate

TBD.


### NftCollectionTransfer

TBD.


### NftItemCreate

TBD.

### NftItemTransfer

```json
  {
    "nft":          NFTItem, 
    "sender":       Account,    // address of the sender
    "recipient":    Account,    // address of the recipient
    "payload":      "<hex-encoded payload>",
    "comment":      "Hello, world" // utf8 string from payload
  }
```



### NftSalePlace

```json
  {
    "owner":          Account, // owner of the token who places it on sale
    "nft_item":       NFTItem,
    "nft_collection": NFTCollection
    "sale_id":        Address,  // contract of the sale
    "price": {
        "currency": "TON",
        "amount":   "12345", // nanocoins
    },
  },
```


### NftSaleFulfill

When the sale is fulfilled, the payment is reflected in the `TonTransfer` action towards the seller’s account.

```json
  {
    "nft_item":       NFTItem,
    "nft_collection": NFTCollection
    "sale_id":        Address,        // contract of the sale
    "recipient":      Account,        // who received the token
    "owner":          Account,
  },
```


### JettonMint

TBD.

### JettonTransfer


```json
  {
    "jetton":       Jetton, 
    "amount":       "1000000",  // base units for this jetton
    "sender":       Account,    // address of the sender
    "recipient":    Account,    // address of the recipient
    "payload":      "<hex-encoded payload>",
    "comment":      "Hello, world" // utf8 string from payload
  }
```


### JettonBurn

TBD.


### SubscriptionCreate

When the subscription is created, full amount is deposited to the subscription contract. The base amount is left on the contract and the remaining coins are transferred out to the recipient. Hence, the difference is paid by the recipient and recorded as `FeePayment.deposit`.

```json
  {
    "recipient":    Account,
    "sender":       Account,
    "contract_id":  Address,
    "subscription": Subscription,
    "amount":       "1000000000", // nanocoins
    "interval":     "2629800", // seconds
  },
```


### SubscriptionCancel

Reminder of coins on the subscription contract are recorded as `FeePayment.refund` for the recipient.

Coins attached to the message by the user who cancels subscription are recorded as `FeePayment.deposit` on their end.

```json
  {
    "recipient":    Account,
    "sender":       Account,
    "contract_id":  Address, 
    "subscription": Subscription,
  },
```




### SubscriptionPayment

```json
  {
    "recipient":    Account,
    "sender":       Account,
    "contract_id":  Address, 
    "amount":       "1000000000",
    "subscription": Subscription,
  },
```


## Entities

### Address

Address represents TON contract location: workchain + hash.

```json
Address = "<wc>:<hex-encoded hash>"  // e.g. 0:afe8f8ade781324...
```

### Account

```json
{
    "address": Address,
    "name":    "Ton foundation" // optional
    "icon":    "https://ton.org/logo.png", // optional
    "isScam":  false, // optional
}
```

### NFTItem

```json
{
    "id":             Address, 
    "collection_id":  Address, // optional
    "name":           "Token name",
    ...
}
```

### NFTCollection

```json
{
    "id":      Address, 
    "name":    "Collection name",
    ...
}
```

### Jetton

```json
{
    "id":           Address, 
    "name":         "Token name",
    "symbol":       "TKN",
    "icon":         "https://example.com/tkn.png",
    "divisibility": 9,
    ...
}
```

### Subscription

```json
{
    "id":        Address, 
    "amount":    "10000000",      // nanocoins
    "interval":  23564800,        // seconds
    "name":      "Product name",  // optional
}
```


