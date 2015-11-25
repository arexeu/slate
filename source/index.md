---
title: AREX Issuer API Reference

language_tabs:
  - shell
  - javascript

toc_footers:
  - <a href='#'>Get a Developer Key</a>
  - <a href='http://arex.io'>Back to our main site</a>

includes:
  - errors

search: true
---

# AREX Issuer API

<aside class="notice">
<strong>All API requests must be made over HTTP over TLS (HTTPS)</strong>. Requests made over plain HTTP will fail.
</aside>

The AREX API for [Issuers](https://youtu.be/k2Nn4df1ZhM) is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). Our API is designed to have predictable, resource-oriented URLs and to use HTTP response codes and [JSON](http://www.json.org/)  payloads to indicate API errors. JSON will be returned in all responses from the API, including errors (*although client libraries will normally translate and map these automatically to language-specific objects or constructs*).

To make the AREX API as explorable and testable as possible, our sandbox API is available for development purposes. The sandbox keys need to be separately obtained and activated in your main account. Data created and sent to the sandbox will never hit the live market and will never cost any money. We have however a set of algorithmic investors in the sandbox, which provide you with a simulated market in order to test your processes end-to-end. The sandbox mirrors the live market in all aspects, except that the sandbox databases will be purged every midnight.

The sandbox API endpoint at: `https://issuer.arex.io/sandbox`

The live API endpoint at: `https://issuer.arex.io/api`

# Getting started

<aside class="warning">
<strong>Before you start:</strong> Make sure that you're authorized to enroll the company that you represent. We make the assumption that you've cleared your lines internally before starting to apply for any accounts, be it a sandbox-account or a live one.
</aside>

In order to access AREX you need to first register for an `Account`. An AREX  account is always denominated in a single currency, such as EUR, GBP, SEK, USD. Companies that trade in multiple currencies can therefore create several accounts. **The default account is denominated in EUR**.

The AREX account belongs to a `Company`, which in turn must have one or multiple `Users`. Users on the other hand can exist without having access to any companies.

Orphan users (_that are not part of any company_) cannot access any AREX accounts. They can nevertheless login and have normal access to user-specific actions, such as registration of new companies and their user settings (More about this later).

## Registration requirements

To register for an AREX account, you need the following information:

Input | Description
---------- | -------
**VAT number** | A valid value added tax identification number or VAT number for the company that you wish to apply an AREX account for.
**Email address** | The user's email address who will be responsible* for the account registration process
**Phone number** | The user's phone number (mobile or landline) that will responsible* for the account registration process

_* the user responsible for the account opening is the contact person AREX will liaise with, who does not need to be the authorized signatory of the company._

## Registration process

The registration process consists of following parts:

* Validation of the user
* Validation of the company

### User validation

AREX will automatically send a confirmation email to the provided email address after a successful registration request. The user must confirm the email address by clicking the confirmation link in the email. After the user confirms their email address, we will automatically initiate a phone number verification, where the user either receives an SMS or a call with a challenge token, that the user must provide back to us. If the challenge provided is correct, the user can create a password for their account. After this step, the user can now log in to the AREX service and initiate the next step; validation of the company.

### Company validation

In order to adhere to our [KYC (Know Your Customer)](https://en.wikipedia.org/wiki/Know_your_customer) requirements, AREX will send an unmarked letter to your company's registered office's address, that is addressed to the company's authorized signatory. The user cannot intervene with this process and the registered office's address is automatically requested by AREX. The letter includes a unique activation code, that the user must obtain and input in order to activate the company's account.

# Authentication

```shell
# With curl you just simply include the `apiKey` in all your requests
curl "https://sandbox.api.arex.io"
  -d "apiKey=ch72gsb320000udocl363eofy"
```

The API requests are authenticated using [JSON Web Tokens (JWTs)](http://jwt.io/). JWTs are an open, industry standard [RFC 7519](https://tools.ietf.org/html/rfc7519) method for representing claims securely between two parties. This information can be verified and trusted because it is digitally signed. Luckily, you don't need to know anything about JWTs in order to use AREX, as the JWTs are automatically issued and verified by us. You must however understand:

* How to obtain a JWT from the API
* What `Contexts` are and how you use them
* How to send the JWT with your API requests

The JWTs are not only useful for **authentication** (ascertaining of who the user is) but also **authorization** (ascertaining what the user is allowed to do). AREX uses JWTs for both authentication and authorization of the user's API requests.

## JWT Basics

JWTs are compact URL-safe tokens, that look like the following example:

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJjdHgiOnsidXNlciI6MzB9LCJpYXQiOjE0NDg0NDY2ODgsImV4cCI6MTQ0ODQ3NTQ4OCwiaXNzIjoiYXJleDppc3N1ZSIsInN1YiI6Imlzc3VlIn0.5HpnejRAZ68tc4ChfIwlrhMDd60-2UVJNTOM6X8xleo`

The string might look like random gibberish at a glance, but it actually consists of three [Base64](https://en.wikipedia.org/wiki/Base64) [Url-encoded](https://en.wikipedia.org/wiki/Base64#URL_applications) segments, each separated by a dot `.`. The segments are as following:

`{header}.{payload}.{signature}`

Next, let's demystify these segments.

### Header

```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```
The header declares that the encoded object is a JSON Web Token and which algorithm was used to create its signature. If we decode the header  we can inspect the contents on the right-hand side.

This simply asserts that the `type` is a 'JWT' and the `algorithm` used for the signature part is [HMAC-SHA256](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code)

### Payload

```json
{
  "ctx": {
    "user": 30
  },
  "iat": 1448446688,
  "exp": 1448475488,
  "iss": "arex:issue",
  "sub": "issue"
}
```
The payload declares the `claims`, and is equivalent to the body of the authentication / authorization object. AREX uses the following claims to authenticate and authorize the requests to its API.

Key | Description
---------- | -------
**ctx** | `Context`, declares the context (read more under contexts)
**iat** | `Issued at` declares the time when the token was generated by the server
**exp** | `Expires at` declares when this token shall expire. Tokens are handed out for a maximum duration of 8 hours. **Note** that tokens cannot cross the boundaries between days. All tokens are automatically null and void after midnight, whereafter the user needs to request a new token.
**iss** | `Issuer` declares who issued the token
**sub** | `Subject` declares what the token can be used for

### Signature

```json
"5HpnejRAZ68tc4ChfIwlrhMDd60-2UVJNTOM6X8xleo"
```
The signature is derived by concatenating the Base64 Url-encoded `header` and `payload` joined by a dot `.`. The resulting string is in this case hashed by a SHA256-HMAC function, using a secret key. The resulting SHA256-HMAC in this case is:

`e47a677a344067af2d7380a17c8c25ae130377ad3ed9454935338ce97f3195ea`

The SHA256-HMAC is Base64 Url-encoded and attached to the JWT. Each request that sends a JWT to the API, the server will then verify that it can derive the same signature by having access to the sent header and body and using the secret key known only by the server.

In case any of the values in the header or payload have been changed by the user or someone else, the resulting hash will not match the provided signature and the request will automatically fail and be denied of any access.

# Accounts

# Companies

# Users

# Invoices

`Invoice` is a synonym for the trade receivables (invoices) that are used to create marketable `Contracts`. Collateral may be uploaded to AREX by sending it through the API, by user uploading it through the browser or by an AREX provided service integration.

Newly uploaded `Collateral` are by default automatically turned into a `Contract` once the collateralization process has been completed. The process itself is asynchronous and will yield a new contract or an error when complete.  

## Create new Invoices

```shell
curl https://sandbox.api.arex.io/collateral \
  -d "apiKey=ch72gsb320000udocl363eofy" \
  -F invoice_id=102936A \
  -F file="@/path/to/invoice_102936A.pdf"
```

> The above request returns a JSON response:

```json
{
  "id": "col_15A3Gj2eZvKYlo2C0NxXGm4s",
  "created": 1433502667927,
  "size": 33278,
  "invoice": "102936A",
  "object": "file_upload",
  "type": "pdf"
}
```

This endpoint allows for a new collateral to be uploaded through the API.

To upload a file to AREX, you'll need to send a request of type `multipart/form-data`. The request should contain the file you would like to upload, as well as the parameters for creating the collateral.

All of AREX's officially supported client libraries should have support for sending `multipart/form-data`.

### API Request

`POST https://sandbox.api.arex.io/collateral`

### Arguments

Parameter | Type | Description
--------- | ------- | -----------
**invoice_id** | string | The Issuer's running sequential identitfication number for this invoice
**file** | file | A file to upload. The file should follow the specifications of [RFC 2388](https://www.ietf.org/rfc/rfc2388.txt) (which defines file transfers for the `multipart/form-data` protocol).

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## List all Invoices

```shell
curl https://sandbox.api.arex.io/collateral \
  -d "apiKey=ch72gsb320000udocl363eofy"
```

> The above request returns a JSON response:

```json
[
  {
    "id": "col_15A3Gj2eZvKYlo2C0NxXGm4s",
    "created": 1433502667927,
    "size": 33278,
    "invoice": "102936A",
    "object": "file_upload",
    "type": "pdf",
    "contractRef": null,
    "void": false,
    "values": {
      "value": "1580.89",
      "currency": "EUR",
      "dateIssue": "2015-06-01",
      "dateDue": "2015-06-21",
      "counterparty": "FI21291126",
      "destCountry": "FI"
      }
  }
]
```

This endpoint retrieves all collateral for the specific user.

<aside class="warning">If you're not using an administrator API key, note that some kittens will return 403 Forbidden if they are hidden for admins only.</aside>

### API Request

`GET https://sandbox.api.arex.io/collateral`

## Retrieve Invoice

```shell
curl https://sandbox.api.arex.io/collateral/col_15A3Gj2eZvKYlo2C0NxXGm4s \
  -d "apiKey=ch72gsb320000udocl363eofy"
```

> The above request returns a JSON response:

```json
  {
    "id": "col_15A3Gj2eZvKYlo2C0NxXGm4s",
    "created": 1433502667927,
    "size": 33278,
    "invoice": "102936A",
    "object": "file_upload",
    "type": "pdf",
    "contractRef": null,
    "void": false,
    "values": {
      "value": "1580.89",
      "currency": "EUR",
      "dateIssue": "2015-06-01",
      "dateDue": "2015-06-21",
      "counterparty": "FI21291126",
      "destCountry": "FI"
      }
  }
```

### API Request

`GET https://sandbox.api.arex.io/collateral/{COLLATERAL_ID}`

#ETRs

#Orders
