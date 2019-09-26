# Bitcoin.com BIP70 Merchant Server

A server for the creation and fulfillment of Bitcoin Cash invoices utilizing the [BIP70 Payment Protocol](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki)

## Invoice Creation

### Request

A POST request is made to `https://pay.bitcoin.com/create_invoice`

#### POST Data
JSON string of an output object with the following properties:
* `script` - (optional) Hex string of desired locking script
* `address` - (optional) Legacy or CashAddr format BCH address. P2PKH or P2SH
* `amount` - Amount in satoshis
* `fiatAmount` - (optional instead of `amount`) Amount denominated in `fiat`
* `fiat` - (optional) Main asset to settle price in. Defaults to BCH if null.
* `fiatRate` - (optional) Rate in [currency]/BCH, if currency is USD, amount is 10.0, then rate would be the USD/BCH price. If null, server will use default rate. Only available for select merchants.
* `webhook` - (optional) URL for webhook
* `memo` - (optional) Memo to be sent back
* `apiKey` - Merchant's/Store's API key. This will get the merchant's settings from server. This will populate the merchant data field in the BIP70 response.

Either script or address can be used. Only one is required.

If more than one output is desired send a JSON string of an object with the following properties:
* `outputs` - An array of output objects formatted as:
    * `script` - (optional) Hex string of desired locking script
    * `address` - (optional) Legacy or CashAddr format BCH address. P2PKH or P2SH
    * `amount` - Amount in satoshis
    * `fiatAmount` - (optional instead of `amount`) Amount denominated in `fiat`
* `currency` - (optional) Main asset to settle price in. Defaults to BCH if null.
* `fiat` - (optional) Main asset to settle price in. Defaults to BCH if null.
* `fiatRate` - (optional) Rate in [currency]/BCH, if currency is USD, amount is 10.0, then rate would be the USD/BCH price. If null, server will use default rate. Only available for select merchants.
* `webhook` - (optional) URL for webhook
* `memo` - (optional) Memo to be sent back
* `apiKey` - Merchant's/Store's API key. This will get the merchant's settings from server. This will populate the merchant data field in the BIP70 response.

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
* `paymentAsset` - BCH
* `fiatSymbol` - Counter currency (like USD)
* `fiatRate` - Rate in [currency]/BCH
* `fiatTotal` - Sum of [outputs] amounts, denominated in [fiatSymbol]
* `webhookUrl` - URL for webhook (if specified on invoice creation)

#### Response Body Example
```
{
   "network":"main",
   "currency":"BCH",
   "outputs":[
      {
         "script":"76a914018a532856c45d74f7d67112547596a03819077188ac",
         "amount":25500,
         "address":"199PArEUmwmcch2LsjxVpegDXsomKdgYi",
         "type":"P2PKH"
      }
   ],
   "time":"2019-06-19T17:57:41.573Z",
   "expires":"2019-06-19T18:12:41.573Z",
   "status":"open",
   "merchantId":"00000000-0000-0000-0000-000000000000",
   "memo":"Your message here",
   "fiatSymbol":"USD",
   "fiatRate":409.7,
   "fiatTotal":0.1044735
   "paymentUrl":"https://pay.bitcoin.com/i/DHL7iqo2CK3hDXZK34Sry8",
   "paymentId":"DHL7iqo2CK3hDXZK34Sry8",
   "webhookUrl":"http://somedomain.com/webhook"
}
```

### Curl Example
```
curl -v -L -H 'Content-Type: application/json' -d '{"script":"76a914018a532856c45d74f7d67112547596a03819077188ac","amount":25500, "webhook":"http://somedomain.com/webhook", "fiat":"USD", "memo":"Your message here"}' https://pay.bitcoin.com/create_invoice
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
< Date: Wed, 12 Jun 2019 22:51:36 GMT
< Connection: keep-alive
< Content-Length: 451
<
* Connection #0 to host pay.bitcoin.com left intact
{"network":"main","currency":"BCH","outputs":[{"script":"76a914018a532856c45d74f7d67112547596a03819077188ac","amount":25500,"address":"199PArEUmwmcch2LsjxVpegDXsomKdgYi","type":"P2PKH"}],"time":"2019-06-19T17:57:41.573Z","expires":"2019-06-19T18:12:41.573Z","status":"open","merchantId":"00000000-0000-0000-0000-000000000000","memo":"Your message here","fiatSymbol":"USD","fiatRate":409.7,"paymentUrl":"https://pay.bitcoin.com/i/DHL7iqo2CK3hDXZK34Sry8","paymentId":"DHL7iqo2CK3hDXZK34Sry8","webhookUrl":"http://somedomain.com/webhook"}% 
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

A GET request is made to `https://pay.bitcoin.com/s/{paymentId}`

#### Request Status (Websockets)

A Websocket Secure (WSS) request is made to `wss://pay.bitcoin.com/s/{paymentId}`

#### Response

Server will respond with a JSON response of the same type as the response during invoice creation.If status is "open," connection will remain open until closed by client or until status changes to either "expired" or "paid," at which time a new object will be returned and the connection will close.

### Invoice Web Page

Visiting `https://pay.bitcoin.com/i/{paymentId}` in a browser returns a web page. Currently this page has a centered QR code and creates a Server-Sent Events connection, logging status to console.

### Invoice Merchant Data

Request made to `https://pay.bitcoin.com/m/{paymentId}` receive a JSON response containing data about the invoice and the merchant that created the payment request

#### Response Body Example
```
{
   "merchantProcessor":"Bitcoin.com",
   "merchantName":"Unregistered merchant",
   "verification":"UNVERIFIED",
   "invoiceCurrency":"USD",
   "invoiceAmount":0.0498276,
   "paymentCurrency":"BCH",
   "paymentAmount":18000,
   "conversionRate":276.82,
   "conversionAssets":"USD/BCH",
   "itemDesc":"Payment request for invoice FFbFxhisukytSvAwGqZDsD",
   "email":"support@bitcoin.com",
   "merchantWebsite":"https://www.bitcoin.com",
   "phone":null,
   "createTime":1567108639293,
   "expiryTime":1567109539293,
   "status":"open",
   "outputs":
      [
         {
            "script":"76a914018a532856c45d74f7d67112547596a03819077188ac",
            "amount":7500,
            "address":"199PArEUmwmcch2LsjxVpegDXsomKdgYi",
            "type":"P2PKH"
         },
         {
            "script":"76a9145d02663da9af3acde02fcd138abd998ab9edd56d88ac",
            "amount":10500,
            "address":"19UniZ1obAjU1tgUydYLzhyvaMignd1oNE",
            "type":"P2PKH"
         }
      ]
}
```

### QR Code

#### Request QR code

A GET request is made to `https://pay.bitcoin.com/qr/{paymentID}`

#### Response

A QR code representation of a URI formatted to BIP72 specification (non-backwards compatible) is returned as a PNG image
