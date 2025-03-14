
# Payzeni API Guide

## Introduction

We ​require **all merchants**​ to complete a full test cycle in our sandbox environment using the information provided below before any production information can be set up and released. Once testing has been completed, contact us to enable your live account.

Throughout this document, placeholders are used to represent variable elements.  These placeholders are enclosed with “{ }”.  Following are the explanations for these:

1. `{host}` – this refers to the host portion of a URL
   * For testing, use [https://test.payzeni.com](https://test.payzeni.com)
   * For production use [https://secure.payzeni.com](https://secure.payzeni.com)
2. `{merchantId}`
   * A merchant ID and API key will be issued to you upon creation of your account
   * Your merchant ID and API key in the test environment will be different from that of your production account
3. `{hashed token}`
   * This is a Hex-encoded SHA-256 hash of a string consisting of concatenated values of select attributes depending on the API.
   * The hashing requirements are explained in a later section of this document.
  


## Submit Payment Request 

This API call will require a `{hashed token}` to be passed as part of the request header as specified below. Following are the specifications for the `{hashed token}`.

* `merchantId`
* `orderId`
* `customerEmail`
* `apiKey`

Concatenate the attributes above in that order and generate the hex-encoded **SHA-256** string of the concatenated string. To illustrate, assume a `merchantId` of “1234567890123”, an `orderId` of “xyz123”, a `customerEmail` of “[test@somedomain.com](mailto:test@somedomain.com)”  and an `apiKey` of “abcde12345fghijk67890”. The concatenated string would be as follows:

`1234567890123xyz123test@somedomain.comabcde12345fghijk67890`.

The hash value would be:

`09a252e513f0c0eb051fe3915522f621ccc3f59c82a4e7fdeb72ca0bcfdc7936`.

Set the hash value in the request header with a header name of `X-payzeni-apiauthz`.

| Description                         | Submit a payment request |                                  
| ------------------------------- | ------------------------------------------------------- |
| HTTP Method                         | POST                                                   |
| Endpoint URL                        | `https://{host}/mxapi/{merchantId}/paymentrequest/submit` |
| Request Headers                     | `Content-Type: application/json;`              <br> `X-payzeni-apiauthz: {hashed token}` |                                             |
| Request Body      | In JSON format. Attributes as described below.             |

**_Response Attributes:_**

|Attribute Name	|Type	|Description
| ----------------------------------- |--- |---------------------------------------------------------- |
|txnId	| Numeric	| Payzeni’s transaction identifier |
|orderId	|String	|Your transaction/order identifier as specified in the request|
|createDate	|String|	The date/time the transaction was created|
|status	|String	|The current status of the transaction. Possible values: <br>`PENDING` <br>	`COMPLETED` <br>	`CANCELLED` <br>	`REJECTED` <br>	`TIMEDOUT` <br>	`SYSERR`
|statusDate |	String|	The date/time the current status was set
|requestedAmount |	Numeric	|The requested amount
|processedAmount	|Numeric|	The amount processed/fulfilled. This will be zero until the customer fulfills the payment request
|currency |	String	|This will always be CAD|
|customerEmail	| String	| The customer’s email as specified in the request |
|customerName	| String	| The customer’s full name as specified in the request|
|customerPhone	| String	| The customer’s phone if specified in the request. Otherwise it will be null|
|paymentLink	| String	| This is the URL that will take the customer to the Interac payment gateway page where this will select the financial institution from |which they will fulfill the request. You may display this link as the end-state of your payment flow or automatically redirect to it. If the “sendEmail” | attribute in the request is set to true, the customer would receive an email containing the same link.|
|message	| String	|This is a message relating to the status of the transaction| 

**_Sample Response_**


    {
    "txnId": 2584206965130,
    "createDate": "2025-03-10 23:31:36 UTC",
    "status": "PENDING",
    "statusDate": "2025-03-10 23:31:36 UTC",
    "orderId": "xyz123",
    "paymentLink": "https://test.payzeni.com/mockpaylinkpage.html?rID=FKWWV4RQIN",
    "requestedAmount": 10.00,
    "processedAmount": 0,
    "currency": "CAD",
    "customerEmail": "test@somedomain.com",
    "customerName": "Test Test",
    "customerPhone": null,
    "message": "Mock ETransfer Request Sent"
    }


**Callback**

When the status of a transaction changes, the system will send a callback if a “callbackUrl” was supplied in the request. This will be in the form of an HTTP Post with Content-Type of “application/json”. The body would contain the details of the transaction in JSON-formatted text. The attributes will be exactly the same as those of the response attributes described above. Your endpoint that will receive this callback need only return an HTTP 200. No content is necessary. Any content in your response will be ignored.



## Query Payment Request


This API call will require a **{hashed token}** to be passed as part of the request header as specified below. Following are the specifications for the **{hashed token}.**
* `merchantId`

* `txnId`

* `apiKey`

Concatenate the attributes above in that order and generate the hex-encoded **SHA-256** string of the concatenated string. To illustrate, assume a `merchantId` of “1234567890123”, a `txnId` of “2584206965130”, and an `apiKey` of “abcde12345fghijk67890”. The concatenated string would be as follows:

`12345678901232584206965130abcde12345fghijk67890`

The hash value would be:

`48d723ddfeaad371ac4981c58363d8a6c8d56d1d9e93ae56766c4dd042f8813e`

Set the hash value in the request header with a header name of `X-payzeni-apiauthz`.


|Description |Query a payment request |
|---|---|
| HTTP Method |POST|
| Endpoint URL | `https://{host}/mxapi/{merchantId}/paymentrequest/query` |
| Request Headers | `Content-Type: application/json;` <br> `X-payzeni-apiauthz: {hashed token}`
|Request Body |In JSON format. Attributes as described below. |

**_Request Body Attributes:_**


 | **Attribute Name** | **Required?** | **Type** |**Description**|
|--|---|--|--|
| txnId| Y |Numeric |The Payzeni transaction identifier|

**_Sample Request_**
```
{
"txnId": 2584206965130
}
```
**Response**

If the request succeeds, you will get an HTTP Status of 200. The response body will contain the details of the transaction in JSON-formatted text. The attributes will be exactly the same as those of the response attributes described above.
