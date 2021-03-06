# HTTP API of the GEO Node

* [Repository](https://github.com/GEO-Protocol/GEO-node-CLI)
* [Postman Collection](https://github.com/GEO-Protocol/Documentation/blob/master/client/http-api/GEO%20node%20HTTP%20interface.postman_collection.json)

This API Interface is already included in the [docker image](https://hub.docker.com/u/geoprotocol) [(repo)](https://github.com/GEO-Protocol/docker-client), so there is no need for additional configuration.

<br/>
<br/>

# API methods (v1)

<br/>

## Open communication channel

```
POST /api/v1/node/contractors/init-channel/ 
```

_Communication channel_ is used for encypted communication with contractor (other GEO Node). <br/>
It is an abstract layer that is used for securing all trust lines related operations between this pair of node. <br/>
It must be called **before** any other trust line related operations with this contractor. <br/>
<br/>
This method performs **no communication** with remote node so no traffic sniffing is possible. <br/>
`secret key`, that is generated by this method, is used for traffic encryption between two nodes and must be **transferred securely** to the contractor node, so it would be able to accept the channel on its side.

<br/>

#### Parameters

| Parameter             | Type      | Required  | Multiple <br/> Allowed    | Description |
| ----                  | ----      | ----      | ----                      | ---- |
| `contractor_address`  | `Address` | +         | +                         | Specifies address of the remote GEO Node to which secure communication channel would be open. The format: `<Type>-<Address>`. At the moment, only IPv4 addressation is supported in the next representation: `12-127.0.0.1:2002` (example).
| `crypto_key`          | `String`  |           |                           | In case if called on contractor's side, used to establish an outgoing connection to the original GEO Node, that has initiated the operation.
| `contractor_id`       | `+Int`    |           |                           | In case if called on contractor's side, specifies channel id that should be used to establish connection to the original GEO Node, that has initiated the operation.

<br/>

#### Responses

| Code                  | Description |
| ----                  | ---- |
| `200 OK`              | Operation has been performed successfully.
| `401 Protocol Error`  | Node reports error. Please, check node's logs for the details.

In case of success node reports additional data:
```json
{
  "data": {
    "channel_id": "<id of created channel>",
    "crypto_key": "<channel crypto key>"
  }
}
```

<br/>

#### Examples

**Open channel**
```
POST http://127.0.0.1:3000/api/v1/node/contractors/init-channel/?contractor_address=12-127.0.0.1:2000
```

**Accept chanell**
```
POST http://127.0.0.1:3000/api/v1/node/contractors/init-channel/?contractor_address=12-127.0.0.2:2000&crypto_key=<key generated by initiator>&contractor_id=<channel id on contractor side>
```

-------

<br/>
<br/>

## List of Communication Channels

```
GET /api/v1/node/contractors/channels/
```

This method provides a list of communication channels that was open on current node.
For security reasons, this method does not provides secret keys, associated with the channels.

<br/>

#### Responses

| Code      |  Description |
| ----      | ---- |
| `200 OK`  | Operation has been performed successfully.

In case of success node reports additional data:

```json
{
  "data": {
    "count": <N>,
    "channels":
    [
      {
        "channel_id":"<id of channel 0>",
        "channel_addresses":"<addresses of channel 0>"
      },
      
      ...
        
      {
        "channel_id":"<id of channel N-1>",
        "channel_addresses":"<addresses of channel N-1>"
      }
    ]
  }
}
```

<br/>

#### Examples

```
GET http://127.0.0.1:3000/api/v1/node/contractors/channels/
```

-------

<br/>
<br/>

## Open Trust Line (TL)

```
POST /api/v1/node/contractors/{contractor_id}/init-trust-line/{equivalent_id}/
```

This method creates outgoing trust line from current node to contractor node, that is specififed by the `contractor_id`.

<br/>

#### URL Parameters

| Parameter         | Type      | Description |
| ----              | ----      | ---- |
| `contractor_id`   | `+Int`    | ID of the contractor towards which trust line should be open.
| `equivalent_id`   | `+Int`    | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) in which trust line would be open.

<br/>

#### Responses

| Code                  |  Description |
| ----                  | ---- |
| `200 OK`              | Operation has been performed successfully.
| `401 Protocol Error`  | Node reports error. Please, check node's logs for the details.

<br/>

#### Examples

```
POST http://127.0.0.1:1501/api/v1/node/contractors/0/init-trust-line/1/
```

-----

<br/>
<br/>

## Set Outgoing Trust Line

```
PUT /api/v1/node/contractors/{contractor_id}/trust-lines/{equivalent_id}/
```

This method sets (updates) outgoing trust amount of the trust line. <br/>
In case if new trust amount equals to 0 - outgoin trust line would be closed. <br/>
In case if incoming trust amount also equals to 0 - current trust line would be archieved. In this case no more operations would be allowed through this trust line.

<br/>

#### URL Parameters

| Parameter         | Type      | Description |
| ----              | ----      | ---- |
| `contractor_id`   | `+Int`    | ID of the contractor towards which trust line should be open.
| `equivalent_id`   | `+Int`    | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) that indicates the trust line.

<br/>

#### Parameters

| Parameter | Type      | Required  | Multiple <br/> Allowed    | Description |
| ----      | ----      | ----      | ----                      | ---- |
| `amount`  | `+Int`    | +         |                           | Specifies new outgoing amount of the trust line. Valid values are in range `0..(2^256)-1`. No float values are valid.

<br/>

#### Responses

| Code                  |  Description |
| ----                  | ---- |
| `200 OK`              | Operation has been performed successfully.
| `401 Protocol Error`  | Node reports error. Please, check node's logs for the details.
| `406 Ins. keys`       | Operation can't be done due to insufficient keys count.

<br/>

#### Examples

```
PUT http://127.0.0.1:1501/api/v1/node/contractors/0/trust-lines/1/?amount=1000
```

-----

<br/>
<br/>

## Close Incoming Trust Line

```
DELETE /api/v1/node/contractors/{contractor_id}/close-incoming-trust-line/{equivalent_id}/
```

This method closes (rejects) incoming trust line and sets incoming trust amount to 0. <br/>
In case if current trust line contains non-zero balance - trust line would not be closed, but incoming trust amount would be forced to 0. <br/> 
In case if outgoing trust amount also equals to 0 - current trust line would be archieved. In this case no more operations would be allowed through this trust line.

<br/>

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `contractor_id` | `+Int` | ID of the contractor towards which trust line should be open.
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) that indicates the trust line.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.
| `401 Protocol Error` | Node reports error. Please, check node's logs for the details.
| `406 Ins. keys` | Operation can't be done due to insufficient keys count.

<br/>

#### Examples

```
DELETE http://127.0.0.1:1502/api/v1/node/contractors/0/close-incoming-trust-line/1/
```

-----

<br/>
<br/>

## List Trust Lines

```
GET /api/v1/node/contractors/trust-lines/{equivalent_id}/
```

```
GET /api/v1/node/contractors/trust-lines/{offset}/{count}/{equivalent}/
```

This method returns list of **all** trust lines that are present on the node. <br/>
To retrieve only a part of this list you should use extended format of this method and specify `offset` and `count`.

<br/>

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) that indicates the trust lines.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.

In case of success node reports additional data:
```json
{
  "data": {
    "count": <N>,
    "trust_lines":
    [
      {
        "contractor_id":"<id of channel (contractor) 0>",
        "contractor":"<addresses of channel (contractor) 0>",
        "state":"<trust line state>",
        "own_keys_present":"<presance of full own keys set (1  or 0)>",
        "contractor_keys_present":"<presance of full contractor keys set (1  or 0)>",
        "incoming_trust_amount":"<incoming trust amount of current TL>",
        "outgoing_trust_amount":"<outgoing trust amount of current TL>",
        "balance":"<balance of current TL>"
      },
      
      ...
      {
        "contractor_id":"<id of channel (contractor) N-1>",
        "contractor":"<addresses of channel (contractor) N-1>",
        "state":"<trust line state>",
        "own_keys_present":"<presance of full own keys set (1  or 0)>",
        "contractor_keys_present":"<presance of full contractor keys set (1  or 0)>",
        "incoming_trust_amount":"<incoming trust amount of current TL>",
        "outgoing_trust_amount":"<outgoing trust amount of current TL>",
        "balance":"<balance of current TL>"
      },
    ]  
  }
}
```

#### Examples

```
GET http://127.0.0.1:1501/api/v1/node/contractors/trust-lines/0/10/1/
```

-------

<br/>
<br/>

## List Equivalents

```
GET /api/v1/node/equivalents/
```

This method returns list of **all** [equivalents ids](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) that are present on the node. <br/>

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.

In case of success, node reports additional data:
```json
{
  "data": {
    "count": <N>,
    "equivalents":
    [
      "<equivalent 0>",
      ...
      "equivalent N-1"
    ]
  }
}
```

#### Examples

```
￼http://127.0.0.1:1501/api/v1/node/equivalents/
```

-------

<br/>
<br/>

## List Contractors

```
GET /api/v1/node/contractors/{equivalent_id}/
```

This method returns list of all contractors, that are present on the node, and are operating in specififed `equivalent_id`.

<br/>

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) that indicates the trust line.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.

In case of success, node reports additional data:
```json
{
  "data": {
    "count": <N>,
    "contractors":
    [
      {
        "contractor_id":"<id of contractor (channel) 0>",
        "contractor_addresses":"<addresses of contractor (channel) 0>"
      },
      
      ...
      
      {
        "contractor_id":"<id of contractor (channel) N-1>",
        "contractor_addresses":"<addresses of contractor (channel) N-1>"
      }
    ]
  }
}
```

#### Examples

```
GET http://127.0.0.1:1501/api/v1/node/contractors/trust-lines/0/10/1/
```

-------

<br/>
<br/>


## Get Trust Line by Contractor ID

```
GET /api/v1/node/contractors/trust-line-by-id/{equivalent}/
```

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) that indicates the trust line.

<br/>

#### Parameters

| Parameter | Type | Required | Multiple <br/> Allowed | Description |
| ---- | ---- | ---- | ---- | ---- |
| `contractor_id` | `+Int` | + | | Specifies ID of the contractor trust line to/from which should be returned.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.
| `405 Trust Line Is Absent` | Node reports absense of requested trust line.

In case of success, node reports additional data:
```json
{
  "data": {
    "trust_line": {
      "id": "<id of channel (contractor)>",
      "state":"<trust line state>",
      "own_keys_present":"<presance of full own keys set (1  or 0)>",
      "contractor_keys_present":"<presance of full contractor keys set (1  or 0)>",
      "incoming_trust_amount":"<incoming trust amount of current TL>",
      "outgoing_trust_amount":"<outgoing trust amount of current TL>",
      "balance":"<balance of current TL>"
    }
  }
}
```

<br/>

#### Examples

```
GET http://127.0.0.1:1501/api/v1/node/contractors/trust-line-by-id/1/?contractor_id=0
```

-------

<br/>
<br/>


## Get Trust Line by Address(-es)

```
GET /api/v1/node/contractors/trust-line-by-address/{equivalent}/
```

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) that indicates the trust line.

<br/>

#### Parameters

| Parameter | Type | Required | Multiple <br/> Allowed | Description |
| ---- | ---- | ---- | ---- | ---- |
| `contractor_address` | `+Int` | + | + | Address(-es) of the contractor(s) to/form which trust lines are present.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.
| `405 Trust Line Is Absent` | Node reports absense of requested trust line.

In case of success, node reports additional data:
```json
{
  "data": {
    "trust_line": {
      "id": "<id of channel (contractor)>",
      "state":"<trust line state>",
      "own_keys_present":"<presance of full own keys set (1  or 0)>",
      "contractor_keys_present":"<presance of full contractor keys set (1  or 0)>",
      "incoming_trust_amount":"<incoming trust amount of current TL>",
      "outgoing_trust_amount":"<outgoing trust amount of current TL>",
      "balance":"<balance of current TL>"
    }
  }
}
```

----

<br/>

#### Examples

```
GET http://127.0.0.1:1501/api/v1/node/contractors/trust-line-by-address/1/?contractor_address=12-127.0.0.1:2002
```

-------

<br/>
<br/>


## (Re-)Generate Trsut Line Key Pairs

```
PUT /api/v1/node/contractors/{contractor_id}/keys-sharing/{equivalent}/
```

This method initiaates key pairs exchange: current node generates and shares key pairs (up to the keys pool limit) with contractor node, and expects the same behaviour from counterpart node.

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) that indicates the trust line.
| `contractor_id` | `+Int` | Specifies ID of the contractor trust line to/from which should be updated.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.

<br/>

#### Examples

```
PUT http://127.0.0.1:1501/api/v1/node/contractors/0/keys-sharing/1/
```

-------

<br/>
<br/>


## Max Flow Predicition

```
GET /api/v1/node/contractors/transactions/max/{equivalent}/
```

This method combines routing algorithm and flow prediciton algorithm and tries to efficiently scan the network topology and to calculate max. payment flow from the issuer node to each one node, that is listed in arguments. There is no need for direct TLs presence between issuer node and listed nodes: it would be enough if there would exist some paths that are connecting this nodes via some internal nodes. GEO Protocol supports paths that are up to 7 nodes long (icluding command issuer node and the destination node).

<br/>

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) in which max flow should be calculated.

<br/>

#### Parameters

| Parameter | Type | Required | Multiple <br/> Allowed | Description |
| ---- | ---- | ---- | ---- | ---- |
| `contractor_address` | `+Int` | + | + | Address(-es) of the contractor(s) to/form which max flow should be calculated.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.
| `401 Protocol Error` | Node reports error. Please, check node's logs for the details.

<br/>

In case of success, node reports additional data:
```json
{
  "data": {
    "count": <N>,
    "records": 
    [
      {
        "address_type":"<address type of contractor 0>",
        "contractor_address":"<address of contractor 0>",
        "max_amount":"<payment flow to contractor 0>"
      },
        
      ...
      
      {
        "address_type":"<address type of contractor N-1>",
        "contractor_address":"<address of contractor N-1>",
        "max_amount":"<payment flow to contractor N-1>"
      },
    ]
  }
}
```

<br/>

#### Examples

```
GET http://127.0.0.1:1502/api/v1/node/contractors/transactions/max/1/?contractor_address=12-127.0.0.1:2001
```

-------

<br/>
<br/>


## Payments / Transactions issuing

```
POST /api/v1/node/contractors/transactions/{equivalent}/
```

This method initilises payment operation from current node to the contractor. <br/>
Addresses received in arguments are used for the communication with remote node. Only one address is used at a time.

<br/>

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) in which max flow should be calculated.

<br/>

#### Parameters

| Parameter | Type | Required | Multiple <br/> Allowed | Description |
| ---- | ---- | ---- | ---- | ---- |
| `contractor_address` | `+Int` | + | | Address of the contractor to/form which max flow should be calculated.
| `amount` | `+Int` | + | | Specifies amount of the payment operation. Valid values are in range `0..(2^256)-1`. No float values are valid.
| `payload` | `String` | | | Specifies optional payment description or meta-information.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.
| `401 Protocol Error` | Node reports error. Please, check node's logs for the details.
| `409 No Consensus` | Consensus has not been reached (some nodes doesn't approved the operation).
| `412 Isufficient Funds` | Insufficient payment possibilities.
| `413 Keys Error` | Payment is impossible due to keys lack on one or several trust lines.
| `414 Keys Error` | Payment is impossible due to keys lack on one or several middleware nodes.
| `444 Remote Node Is Unrechable` | The contractor is not reachable (remote node seems to be down).
| `462 No Paths`| No payment paths (no payment connection is present between sender and receiver node).

In case of success, node reports additional data:
```json
{
  "data": {
    "transaction_uuid":<uuid of payment transaction>
  }
}
```

<br/>

#### Examples

```
POST http://127.0.0.1:1502/api/v1/node/contractors/transactions/1/?contractor_address=12-127.0.0.1:2001&amount=50
POST http://127.0.0.1:1502/api/v1/node/contractors/transactions/1/?contractor_address=12-127.0.0.1:2001&amount=50&payload=info
```

-------

<br/>
<br/>


## Payments History

```
GET /api/v1/node/history/transactions/payments/{offset}/{count}/{equivalent}/
```

<br/>

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) in which max flow should be calculated.
| `offset` | `+Int` | Offset from which operations should be read.
| `count` | `+Int` | How many (up to) operations should be returned from the database.

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.

In case of success, node reports additional data:
```json
{
  "data": {
    "count": <N>,
    "records":
    [
      {
        "transaction_uuid":"<UUID of payment 0>",
        "unix_timestamp_microseconds":"<count unix microseconds of payment 0>",
        "contractor":"<contractors addresses of payment 0>",
        "operation_direction":"<type of payment 0>",
        "amount":"<amount of payment 0>",
        "balance_after_operation":"<total balanse after payment 0>"
      },
    
      ...
      
      {
        "transaction_uuid":"<UUID of payment N-1>",
        "unix_timestamp_microseconds":"<count unix microseconds of payment N-1>",
        "contractor":"<contractors addresses of payment N-1>",
        "operation_direction":"<type of payment N-1>",
        "amount":"<amount of payment N-1>",
        "balance_after_operation":"<tottal balanse after payment N-1>"
      }
    ]
  }
}
```

Operation_direction could be one of:
* `incoming` — incoming payment;
* `outgoing` — outgoing payment;

<br/>

#### Examples

```
GET http://127.0.0.1:1501/api/v1/node/history/transactions/payments/0/10/1/
```

-------

<br/>
<br/>

## Trust Line Operations History

```
GET /api/v1/node/history/transactions/trust-lines/{offset}/{count}/{equivalent}/"
```

<br/>

#### URL Parameters

| Parameter | Type | Description |
| ---- | ---- | ---- |
| `equivalent_id` | `+Int` | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) in which max flow should be calculated.
| `offset` | `+Int` | Offset from which operations should be read.
| `count` | `+Int` | How many (up to) operations should be returned from the database.

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.

In case of success, node reports additional data:
```json
{
  "data": {
    "count": <N>,
    "records": 
    [
      {
        "transaction_uuid":"<UUID of TL operation 0>",
        "unix_timestamp_microseconds":"<count unix microseconds of TL operatoin 0>",
        "contractor":"<contractors addresses of TL operation 0>",
        "operation_direction":"<type of TL operation 0>",
        "amount":"<amount of TL operation 0>"
      },
      
      ...
           
      {
        "transaction_uuid":"<UUID of TL operation N-1>",
        "unix_timestamp_microseconds":"<count unix microseconds of TL operatoin N-1>",
        "contractor":"<contractors addresses of TL operation N-1>",
        "operation_direction":"<type of TL operation N-1>",
        "amount":"<amount of TL operation N-1>"
      }
    ]
  }
}
```

`operation_direction` could by one of:
* opening
* accepting
* setting
* updating
* closing
* rejecting
* closing_incoming
* rejecting_outgoing

<br/>

#### Examples

```
GET http://127.0.0.1:1501/api/v1/node/history/transactions/payments/0/10/1/
```

-------

<br/>
<br/>

## Trust Line Operations History

```
GET /api/v1/node/history/contractors/{offset}/{count}/{equivalent}/
```

This method returns all operations that are associated with a contractor (payment operations and trust lines operations).

<br/>

#### URL Parameters

| Parameter         | Type      | Description |
| ----              | ----      | ---- |
| `equivalent_id`   | `+Int`    | ID of the [equivalent](https://github.com/GEO-Protocol/specs-protocol/blob/master/trust_lines/trust_lines.md#trust-lines-equivalents) in which max flow should be calculated.
| `offset`          | `+Int`    | Offset from which operations should be read.
| `count`           | `+Int`    | How many (up to) operations should be returned from the database.

<br/>

#### Parameters

| Parameter | Type | Required | Multiple <br/> Allowed | Description |
| ---- | ---- | ---- | ---- | ---- |
| `contractor_address` | `+Int` | + | | Address of the contractor to/form which max flow should be calculated.

<br/>

#### Responses

| Code|  Description |
| ---- | ---- |
| `200 OK` | Operation has been performed successfully.

In case of success, node reports additional data:
```json
{
  "data": {
    "count":N,
    "records":
    [
      {
        "record_type":"<type of history record 0>",
        "transaction_uuid":"<UUID of operation 0>",
        "unix_timestamp_microseconds":"<count unix microseconds of operatoin 0>",
        "contractor":"<contractors addresses of operation 0>",
        "operation_direction":"<type of operation 0>",
        "amount":"<amount of operation 0>",
        "balance_after_operation":"<tottal balanse after operation 0>"
      },
	
	  ...
           
      {
        "record_type":"<type of history record N-1>",
        "transaction_uuid":"<UUID of operation N-1>",
        "unix_timestamp_microseconds":"<count unix microseconds of operatoin N-1>",
        "contractor":"<contractors addresses of operation N-1>",
        "operation_direction":"<type of operation N-1>",
        "amount":"<amount of operation N-1>",
        "balance_after_operation":"<total balanse after operation N-1>"
      }
    ]
  }
}
```

`record_type` could be one of:
* `trustline`
* `payment`

<br/>

#### Examples

```
GET http://127.0.0.1:1501/api/v1/node/history/contractors/0/10/1/?contractor_address=12-127.0.0.1:2002
```

-------

<br/>
<br/>

## Common Responses

| Code  |  Description |
| ----  | ---- |
| `400` | Invalid Request.
| `500` | Unexpected HTTP API error.
| `501` | Unexpected node error.
| `503` | Node is unreachable (command result is absent).
| `504` | Unexpected format of the command result (command parsing on the HTTP API level implemented with errors).
| `505` | Command transferring error. Node doesn't accepted the command.
| `603` | Transactions are disabled on the node's side.
| `604` | There is no trust line of this equivalent.
| `605` | Command has not been parsed correctly.
