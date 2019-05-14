# Bitcoin.com BIP70 Merchant Server, Beta version

A server for the creation and fulfillment of Bitcoin Cash invoices utilizing the [BIP70 Payment Protocol](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki)

Please note that this API is not final and _will_ be changed before final release.

## Invoice Creation

### Request

A POST request is made to `https://pay.bitcoin.com/create_invoice`

#### POST Data
JSON string of an output object with the following properties:
* `script` - (optional) Hex string of desired locking script
* `address` - (optional) Legacy or CashAddr format BCH address. P2PKH or P2SH
* `amount` - Amount in satoshis
* `fiat` - (optional) Three-letter fiat currency code. Defaults to 'USD'

Either script or address can be used. Only one is required.

If more than one output is desired send a JSON string of an object with the following properties:
* `outputs` - An array of output objects formatted as:
    * `script` - (optional) Hex string of desired locking script
    * `address` - (optional) Legacy or CashAddr format BCH address. P2PKH or P2SH
    * `amount` - Amount in satoshis
* `fiat` - (optional) Three-letter fiat currency code. Defaults to 'USD'

#### Headers

* `Content-Type` should be set to `application/json`.

### Response
The response will be a JSON format payload quite similar to the BIP70 format.

#### Body
* `network` - Which network is this request for (main / test / regtest)
* `currency` - Three digit currency code representing which coin the request is based on
* `outputs` - What output(s) your transaction must include in order to be accepted
* `time` - ISO Date format of when the invoice was generated
* `expires` - ISO Date format of when the invoice will expire
* `status` - The status of the invoice `open / paid / expired`
* `merchantId` - UUID of associated merchant. If no merchant, uuid consists of all zeroes
* `memo` - A plain text description of the payment request, can be displayed to the user / kept for records
* `paymentUrl` - The url where the payment should be sent
* `paymentId` - The invoice ID, can be kept for records

#### Response Body Example
```
{
  "network": "main",
  "currency": "BCH",
  "outputs": [
    {
      "amount": 39300,
      "script": "76a914018a532856c45d74f7d67112547596a03819077188ac",
      "address": "199PArEUmwmcch2LsjxVpegDXsomKdgYi",
      "type": "P2PKH"
    }
  ],
  "time": "2018-01-12T22:04:54.364Z",
  "expires": "2018-01-12T22:19:54.364Z",
  "status":"open",
  "merchantId":"00000000-0000-0000-0000-000000000000",
  "memo": "Payment request for invoice TmyrxFvAi4DjFNy3c7EjVm",
  "paymentUrl": "https://pay.bitcoin.com/i/TmyrxFvAi4DjFNy3c7EjVm",
  "paymentId": "TmyrxFvAi4DjFNy3c7EjVm"
}
```

### Curl Example
```
curl -v -L -H 'Content-Type: application/json' -d '{"script":"76a914018a532856c45d74f7d67112547596a03819077188ac","amount":25500}' https://pay.bitcoin.com/create_invoice
*   Trying 13.53.78.23...
* TCP_NODELAY set
* Connected to pay.bitcoin.com (13.53.78.23) port 443 (#0)
> POST /create_invoice HTTP/1.1
> Host: pay.bitcoin.com
> User-Agent: curl/7.58.0
> Accept: */*
> Content-Type: application/json
> Content-Length: 78
>
* upload completely sent off: 78 out of 78 bytes
< HTTP/1.1 200 OK
< Access-Control-Allow-Origin: *
< Content-Type: application/json
< Date: Tue, 07 May 2019 20:42:37 GMT
< Connection: keep-alive
< Content-Length: 487
<
* Connection #0 to host pay.bitcoin.com left intact
{"network":"main","currency":"BCH","outputs":[{"script":"76a914018a532856c45d74f7d67112547596a03819077188ac","amount":25500,"address":"199PArEUmwmcch2LsjxVpegDXsomKdgYi","type":"P2PKH"}],"time":"2019-05-07T20:42:35.174Z","expires":"2019-05-07T20:57:35.174Z","status":"open","merchantId":"00000000-0000-0000-0000-000000000000","memo":"Payment request for invoice Auo1tTBURpbPucr8rBbpyS","paymentUrl":"https://pay.bitcoin.com/i/Auo1tTBURpbPucr8rBbpyS","paymentId":"Auo1tTBURpbPucr8rBbpyS"}% 
```

## Payment

The payment process follows the [BIP70](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki) / [BIP71](https://github.com/bitcoin/bips/blob/master/bip-0071.mediawiki) / [BIP72](https://github.com/bitcoin/bips/blob/master/bip-0072.mediawiki) specification, with a slight modification to the headers in BIP71 as follows:

The Media Type (Content-Type in HTML/email headers) for Bitcoin Cash
protocol messages shall be:


| Message | Type/Subtype |
| --- | --- |
| PaymentRequest | application/bitcoincash-paymentrequest |
| Payment | application/bitcoincash-payment |
| PaymentACK | application/bitcoincash-paymentack |


## API Endpoints

### Check Status

#### Request Status (Server-Sent Events)

A GET request is made to `https://pay.bitcoin.com/s/{paymentID}`

#### Request Status (Websockets)

A Websocket Secure (WSS) request is made to `wss://pay.bitcoin.com/s/{paymentID}`

#### Response

Server will respond with a JSON response of the same type as the response during invoice creation.If status is "open," connection will remain open until closed by client or until status changes to either "expired" or "paid," at which time a new object will be returned and the connection will close.

### Invoice Web Page

Visiting `https://pay.bitcoin.com/i/{paymentID}` in a browser returns a web page. Currently this page has a centered QR code and creates a Server-Sent Events connection, logging status to console.

### QR Code

#### Request QR code

A GET request is made to `https://pay.bitcoin.com/qr/{paymentID}`

#### Response

A QR code representation of a URI formatted to BIP72 specification (non-backwards compatible) is returned as a PNG image
