# Bitcoin.com BIP70 Merchant Server

A server for the creation and fulfillment of BCH Simple Ledger Token invoices utilizing the [SLP Payment Protocol](https://github.com/vinarmani/slp-specifications/blob/payment-protocol/slp-payment-protocol.md)

## Invoice Creation

### Request

A POST request is made to `https://pay.bitcoin.com/create_invoice`

#### POST Data
JSON string with the following properties:
* `token_id` - ID of SLP token to be sent (BCH transaction hash)
* `slp_outputs` - An array of output objects formatted as:
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
* `currency` - Will be 'SLP'
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
   "network":"main",
   "currency":"SLP",
   "outputs":[
      {
         "script":"6a04534c500001010453454e44204de69e374a8ed21cbddd47f2338cc0f479dc58daa2bbe11cd604ca488eca0ddf08000000000000012c0800000000000000c8",
         "amount":0,
         "token_id":"4de69e374a8ed21cbddd47f2338cc0f479dc58daa2bbe11cd604ca488eca0ddf",
         "send_amounts":[
            300,
            200
         ],
         "type":"SLP"
      },
      {
         "script":"76a914c2fdfcd981a6f6cea21835ce1453139137aaceba88ac",
         "amount":546,
         "address":"1Jn2MbBv7wm7JobwiikX26y6qFdEMhcA9U",
         "type":"P2PKH"
      },
      {
         "script":"76a914018a532856c45d74f7d67112547596a03819077188ac",
         "amount":546,
         "address":"199PArEUmwmcch2LsjxVpegDXsomKdgYi",
         "type":"P2PKH"
      }
   ],
   "time":"2019-06-05T19:28:43.031Z",
   "expires":"2019-06-05T19:43:43.031Z",
   "memo":"Payment request for invoice Essp8fDKrzmqgNwYjowgia",
   "fiatSymbol":"BCH",
   "fiatRate":1,
   "paymentUrl":"https://pay.bitcoin.com/i/Essp8fDKrzmqgNwYjowgia",
   "paymentId":"Essp8fDKrzmqgNwYjowgia"
}
```

### Curl Example
```
curl -v -L -H 'Content-Type: application/json' -d '{"token_id":"4de69e374a8ed21cbddd47f2338cc0f479dc58daa2bbe11cd604ca488eca0ddf", "slp_outputs":[{"address":"1Jn2MbBv7wm7JobwiikX26y6qFdEMhcA9U","amount":300}, {"address":"199PArEUmwmcch2LsjxVpegDXsomKdgYi","amount":200}]}' https://pay.bitcoin.com/create_invoice
*   Trying 13.53.78.23...
* TCP_NODELAY set
* Connected to pay.bitcoin.com (13.53.78.23) port 443 (#0)
> POST /create_invoice HTTP/1.1
> Host: pay.bitcoin.com
> User-Agent: curl/7.58.0
> Accept: */*
> Content-Type: application/json
> Content-Length: 220
>
* upload completely sent off: 220 out of 220 bytes
< HTTP/1.1 200 OK
< Access-Control-Allow-Origin: *
< Content-Type: application/json
< Date: Wed, 05 Jun 2019 19:28:43 GMT
< Connection: keep-alive
< Content-Length: 857
<
* Connection #0 to host pay.bitcoin.com left intact
{"network":"main","currency":"SLP","outputs":[{"script":"6a04534c500001010453454e44204de69e374a8ed21cbddd47f2338cc0f479dc58daa2bbe11cd604ca488eca0ddf08000000000000012c0800000000000000c8","amount":0,"token_id":"4de69e374a8ed21cbddd47f2338cc0f479dc58daa2bbe11cd604ca488eca0ddf","send_amounts":[300,200],"type":"SLP"},{"script":"76a914c2fdfcd981a6f6cea21835ce1453139137aaceba88ac","amount":546,"address":"1Jn2MbBv7wm7JobwiikX26y6qFdEMhcA9U","type":"P2PKH"},{"script":"76a914018a532856c45d74f7d67112547596a03819077188ac","amount":546,"address":"199PArEUmwmcch2LsjxVpegDXsomKdgYi","type":"P2PKH"}],"time":"2019-06-05T19:28:43.031Z","expires":"2019-06-05T19:43:43.031Z","memo":"Payment request for invoice Essp8fDKrzmqgNwYjowgia","fiatSymbol":"BCH","fiatRate":1,"paymentUrl":"https://pay.bitcoin.com/i/Essp8fDKrzmqgNwYjowgia","paymentId":"Essp8fDKrzmqgNwYjowgia"} 
```

## Payment

The payment process follows the [SLP Payment Protocol](https://github.com/vinarmani/slp-specifications/blob/payment-protocol/slp-payment-protocol.md)


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
