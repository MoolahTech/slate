---
title: Savvy API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Savvy API
---

<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js">
  mermaid.initialize({startOnLoad:true});
</script>

# Introduction

Welcome to the Savvy API! You can use our API to access Savvy API endpoints, which helps you create any mutual funds usecase you like. Savvy is your one stop shop, no signups for any other external services are required. 

We have language bindings in Shell only for now. We don't have readymade clients as of now, but they'll be coming soon! You can view shell (curl) code examples in the dark area to the right.

# Authentication

> To authorize, you must pass a Base64 encoded combination of your access key and password. Use this code:

```shell
# With shell, you can just pass the correct header with each request
# encoded_key = Base64.encode("ACCESS_KEY:PASSWORD")
curl "https://surface.thesavvyapp.in/secure/tokens" \
  -H "Authorization: Basic <encoded_key>"
```
```json
Response:
{
  "token": "abcdeabcdeacbde"
}
```

```shell
# Use above token as Bearer token for secure paths:
curl "api_endpoint_here" \
  -H "Authorization: Bearer <token>"
```

Savvy has 2 ways of authenticating, depending on how the API is being used:

1. **Secure paths using Basic auth + Bearer token:** Using your API keys, you can request a new Bearer token that will give you access to partner level APIs (all investors under your partner account).
You can register a new Savvy API key combination at our [developer portal](http://developers.savvyapp.in) or send us an [email](mailto:hello@savvyapp.in) at hello@savvyapp.in.

2. **Individual paths using Bearer token:** Using your secure token in the previous step, you can request a Bearer token that will access to investor specific APIs. Refer to the investors section for information on how to get a token for an individual investor.

Savvy expects a Bearer token to be included in all API requests to the server in a header as follows:

`Authorization: Bearer <token>`

<aside class="notice">
You must replace <code>token</code> with your own secure token or individual token.
</aside>

# Onboardings

Most of our APIs require an onboarding to be done before communicating with them. An onboarding is essentially a KYC (Know Your Customer) workflow which authorizes a person to conduct a mutual fund transaction. The workflow is the following:

<div class="mermaid">
  graph TD;
  create_onboarding(Create onboarding with pan number and date of birth)-- Existing investor -->submit_bank_details(Submit bank details)
  create_onboarding-- Not an existing investor -->full_kyc(Full KYC);
  submit_bank_details-->submit_fatca(Submit FATCA info if required);
  submit_fatca-->create_transaction(Move ahead to create a transaction);
  full_kyc-->submit_pan(Submit picture of PAN card);
  submit_pan-->submit_address(Submit picture of an address proof);
  submit_address-->submit_signature(Submit picture of signature);
  submit_signature-->submit_selfie(Submit selfie image);
  submit_selfie-->verify_details(Verify details so far);
  verify_details-->submit_video(Submit video with OTP);
  submit_video-->aadhaar_sign(Aadhaar OTP verification);
  aadhaar_sign-->submit_full_kyc(Submit for verification);
</div>

<aside class="notice">
If doing Full KYC, you should wait for verification confirmation via webhook before creating transactions. While it is technically possible for you to accept transactions straight after full KYC submission, the user experience would be poor in case of rejections.
</aside>

## Onboarding object

Parameter | Description
--------- | ----------- 
uuid | `String` Unique identifier of the onboarding object
pan_number | `String` Customer PAN number. Note that the pan number is overwritten when parsed from the pan card.
existing_investor | `Boolean` If this customer is an existing investor in the KRA databases. Use this field to determine if you to do a full KYC for the customer or not.
name | `String` Customer's full name.
date_of_birth | `String` Format DD/MM/YYYY. Note that the dob is overwritten when parsed from the pan card.
email | `String` Customer's email address.
phone_number | `String` Customer's phone number with +91 prefix.
kyc_status | One of: `["success", "failure", "pending"]` Result of full KYC verification post submission
pan_card_image_url | `URL String` The pan card image submitted
fathers_name | `String` Father's name from the pan card
address_proof_image_url | `URL String` The address proof image submittted
address_proof_type | One of: `["aadhaar", "voter_id", "passport", "license"]` | The type of address proof that has been submitted
address | `String` Customer's street address.
city | `String` Customer's city.
pincode | `String` Customer's pincode.
signature_image_url | `URL String` The signature image submitted
selfie_image_url | `URL String` The selfie image submitted
cancelled_cheque_url | `URL String` The cancelled cheque image submitted
video_url | `URL String` The selfie video submitted
annual_income | `String` Annual income code (refer to enum list)
gender | `String` Gender code (refer to enum list)
occupation | `String` Occupation code (refer to enum list)
marital_status | `String` Marital status code (refer to enum list)

## Create onboarding

```json
// body
{ "onboarding": 
  {
    "pan_number": "ABCDE1234C"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the onboarding JSON object with the name filled in if the customer is an existing investor.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings`

### Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
pan_number | true | `String` Customer PAN number

## Create onboarding bank account

```json
// body
{ "onboarding": 
  {
    "account_number": "0012345678901",
    "ifsc_code": "HDFC0000873"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/bank_account" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the onboarding JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/bank_account`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
account_number | true | `String` A Valid bank account number
ifsc_code | true | `String` A valid IFSC code

## Start Full KYC

```json
// body
{ "onboarding":
  {
    "email": "batman@gmail.com",
    "name": "Bruce Wayne",
    "phone_number": "+919009012345"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/full_kyc" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> The above command returns the onboarding JSON object

This endpoint prepares the onboarding object for a full KYC

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/full_kyc`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Description
--------- | -----------
name | `String` Customer's name
email | `String` Customer's email
phone_number | `String` Customer's phone number

# AMCs

This API gives you a list of supported AMCs by the Savvy API. This list remains relatively static, but we are constantly adding AMCs, so we recommend having some mechanism of fetching this list every so often. A monthly update might be ideal, but this is up to you.

## AMC object

Parameter | Description
--------- | ----------- 
name | `String` Name of the AMC.
code | `String` Operational code of the AMC. This code must be submitted in API requests, **not** the name.

## Get all AMCs

```shell
curl "http://surface.thesavvyapp.in/amcs" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns an array of AMC JSON objects.

### HTTP Request

`GET http://surface.thesavvyapp.in/amcs`

# Funds

This API gives you lists of supported mutual funds, by AMC. There is also support for NFOs, along with their timelines. Please note that NFOs are updated 1-2 days before the launch date.

## Fund object

Parameter | Description
--------- | ----------- 
name | `String` Name of the fund.
active | `Boolean` Whether the fund is currently accepting purchases from customers.
nav | `Decimal` Current value of a unit of this fund.
code | `String` Global unique identifier of this fund. This code must be submitted in API requests, **not** the name.

## Get all funds

```shell
curl "http://surface.thesavvyapp.in/funds?amc_code=code" \
  -X GET \
  -H "Authorization: Bearer <token>"

```
> The above command returns an array of Fund JSON objects.

### HTTP Request

`GET http://surface.thesavvyapp.in/funds`

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
amc_code | true | `String` Code the AMC from the AMC list API.

# Accounts

For most usecases, the accounts API is a pull-only API. Accounts are created asynchronously; when a purchase transaction is successful, a new account number (Folio number in the MF world) is allocated. You should be listening to webhooks to listen for account creations.

## Account object

Parameter | Description
--------- | ----------- 
uuid | `String` Unique identifier of the account.
amc_code | `String` Code of the AMC which this account is of.
amc_name | `String` Name of the AMC which this account is of.
amount_deposited | `Integer` Original amount deposited.
amount_withdrawn | `Decimal` Amount withdrawn.
account_value | `Map` Fund code => Current investment amount mapping.
units | `Map` Fund code => Units mapping.
holding_mode | `Enum: ["SI"]` Only Single mode is supported as of now.

## Show account

```shell
curl "http://surface.thesavvyapp.in/accounts/<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns a account JSON object.

### HTTP Request

`GET http://surface.thesavvyapp.in/accounts/<UUID>`

# Deposits

The deposits API describes how to create a mutual fund purchase transaction. This API can only be accessed post the onboarding flow. The workflow for creating and completing a purchase is:

<div class="mermaid">
  graph TD;
  create_deposit(Create deposit with amount and fund details)-->redirect_user(Redirect user to payments page)
  redirect_user-- Failure -->user_failure(Your failure page);
  redirect_user-- Success-->user_success(Your success page);
  user_success-->listen(Listen for transaction updates via webhook);
</div>

## Deposit object

Parameter | Description
--------- | ----------- 
uuid | `String` Unique identifier of the deposit.
account_uuid | `String` Account which the deposit is part of. 
fund_code | `String` Code of the fund invested in.
fund_name | `String` Name of the fund invested in.
amount | `Integer` Original amount of the investment.
current_amount | `Decimal` Current value of the investment.
units | `Decimal` Units allocated for the investment.
status | `Enum: created, payment_made, submitted_to_rta, completed, error` Status of the investment.
status_description | `String` Reason in case of failure / rejections.
reinvest_mode | `Enum: payout: 'N', reinvestment: 'Y', growth: 'Z', bonus: 'B'` What should happen with any dividend that is paid out.
payment_link | `String` **Only applicable for create API** URL to which to take the user for payment checkout.
partner_transaction_id | `String` Your ID associated with the transaction.
user_completed_payment_at | `Datetime` Time at which payment was completed by the user.
transferred_to_amc_at | `Datetime` Time at which the payment was transferred to the AMC.
created_at | `Datetime` Time at which the deposit was created.  
sip_uuid | `String` Identifier of the SIP that the deposit is part of.
stp_uuid | `String` Identifier of the STP that the deposit is part of

## Get deposits

```shell
curl "http://surface.thesavvyapp.in/deposits?account_uuid=<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns an array of deposit JSON objects.

### HTTP Request

`GET http://surface.thesavvyapp.in/deposits?account_uuid=<UUID>`

### URL Parameters

Parameter | Required | Description
--------- | ------- | -----------
account_uuid | true | `String` All deposits associated with a account

## Show deposit

```shell
curl "http://surface.thesavvyapp.in/deposits/<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns a deposit JSON object.

### HTTP Request

`GET http://surface.thesavvyapp.in/deposits/<UUID>`

## Create deposit

```json
// body
{ "deposit": 
  {
    "amount": "1000",
    "fund_code": "1234",
    "account_uuid": "aaaaa-bbbb-cccc-dddd",
    "redirect_url": "https://example.com/payment_redirect"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/deposits" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the deposit JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/deposits`

### Parameters

<aside class="notice">
Note the <code>deposit</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
amount | true | `Integer` Amount to be invested
fund_code | true | `Date` Code of the fund to be invested in
redirect_url | true | `String` Where to direct the customer after payment. A field called `status` (as a query param) in the redirect URL will be available to indicate success or failure.
account_uuid | false | `String` If the deposit has to be created in an existing account.
partner_transaction_id | false | `String` Your custom ID to identify this transaction.

<aside class="warning">If account uuid is not specified, a new account will be created for the customer. Make sure this is the behavior you want. However, for first time customers, not passing the account id is the expected behavior.</aside>

# Withdrawals

The withdrawals API describes how to create a mutual fund redemption transaction. This API can only be accessed post the onboarding flow. The workflow for creating and completing a redemption is:

<div class="mermaid">
  graph TD;
  create_withdrawal(Create withdrawal with amount and fund details)-->verify_otp(Verify OTP on registered phone number and/or email)
  verify_otp-- Failure -->verify_otp;
  verify_otp-- Success-->listen(Listen for transaction updates via webhook);
</div>

## Withdrawal object

Parameter | Description
--------- | ----------- 
uuid | `String` Unique identifier of the withdrawal.
account_uuid | `String` Account which the withdrawal is part of. 
fund_code | `String` Code of the fund invested in.
fund_name | `String` Name of the fund invested in.
amount | `Integer` Amount to be withdrawn.
units | `Decimal` Units withdrawn for the transaction.
status | `Enum: created, otp_complete, submitted_to_rta, completed, error` Status of the withdrawal.
status_description | `String` Reason in case of failure / rejections.
partner_transaction_id | `String` Your ID associated with the transaction.
created_at | `Datetime` Time at which the withdrawal was created.  
swp_uuid | `String` Identifier of the SIP that the withdrawal is part of.
stp_uuid | `String` Identifier of the STP that the withdrawal is part of

## Get withdrawals

```shell
curl "http://surface.thesavvyapp.in/withdrawals?account_uuid=<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns an array of withdrawal JSON objects.

### HTTP Request

`GET http://surface.thesavvyapp.in/withdrawals?account_uuid=<UUID>`

### URL Parameters

Parameter | Required | Description
--------- | ------- | -----------
account_uuid | true | `String` All withdrawals associated with a account

## Show withdrawal

```shell
curl "http://surface.thesavvyapp.in/withdrawals/<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns a withdrawal JSON object.

### HTTP Request

`GET http://surface.thesavvyapp.in/withdrawals/<UUID>`

## Create withdrawal

```json
// body
{ "withdrawal": 
  {
    "amount": "1000",
    "fund_code": "1234",
    "account_uuid": "aaaaa-bbbb-cccc-dddd",
    "partner_transaction_id": "xyv-abc"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/withdrawals" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the withdrawal JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/withdrawals`

On success, an OTP is sent on both the users' registered email and phone number. This OTP has to be verified by the user and sent to us on API.

### Parameters

<aside class="notice">
Note the <code>withdrawal</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
amount | true | `Decimal` Amount to be invested
fund_code | true | `Date` Code of the fund to be invested in
account_uuid | true | `String` If the deposit has to be created in an existing account.
partner_transaction_id | false | `String` Your custom ID to identify this transaction.

## Verify withdrawal OTP

```json
// body
{ "withdrawal": 
  {
    "otp": "123456"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/withdrawals/<UUID>/verify_otp" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the withdrawal JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/withdrawals/<UUID>/verify_otp`

On success, the status of the withdrawal will change to `otp_complete`. The OTP verification will expire in 5 minutes, and the OTP can be resent in case of delivery failures after 30 seconds of first attempt. 

### Parameters

<aside class="notice">
Note the <code>withdrawal</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
otp | true | `String` The user inputted OTP

# One-click Checkout / Investment links

This API describes how to create instant investment links. This API can handle any scenario (existing folio, existing investor but no folio, and non-existing investor). This comes as a hosted solution under your logo, and no extra needs to be done from your end to enable this (other than listen for webhooks if you'd like).

# Systematic Investment Plans

This API describes how to create a SIP transaction. This API can only be accessed post the onboarding flow. The workflow for creating and registering an SIP is:

<div class="mermaid">
  graph TD;
  create_sip(Create SIP with amount and fund details)-->create_mandate(Create E-Nach / UPI Mandate)
  create_mandate -->redirect_user(Redirect user to bank page);
  redirect_user-- Failure-->user_failure(Your failure page);
  user_failure -->create_mandate; 
  redirect_user-- Success -->user_success(Your success page);
  user_success -->listen(Listen for mandate updates via webhook);
</div>

## SIP object

Parameter | Description
--------- | ----------- 
uuid | `String` Unique identifier of the withdrawal.
account_uuid | `String` Account which the SIP is part of.
fund_code | `String` Code of the fund invested in.
fund_name | `String` Name of the fund invested in.
start_date | `Date` Start date of the SIP.
end_date | `Date` End date of the SIP.
frequency | `Enum: monthly, weekly, daily, ad-hoc` Frequency of debits
amount | `Integer` Amount to be withdrawn at each installment.
payment_link | `String` **Only applicable for create API** URL to which to take the user for mandate checkout.
redirect_url | `String` Where to direct the customer after payment. A field called `status` (as a query param) in the redirect URL will be available to indicate success or failure.
active | `Boolean` Whether the SIP is actively debiting.
partner_transaction_id | `String` Your ID associated with the transaction.
created_at | `Datetime` Time at which the SIP was created.

## Get sips

```shell
curl "http://surface.thesavvyapp.in/sips?account_uuid=<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns an array of sip JSON objects.

### HTTP Request

`GET http://surface.thesavvyapp.in/sips?account_uuid=<UUID>`

### URL Parameters

Parameter | Required | Description
--------- | ------- | -----------
account_uuid | true | `String` All sips associated with a account

## Show sip

```shell
curl "http://surface.thesavvyapp.in/sips/<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns a sip JSON object.

### HTTP Request

`GET http://surface.thesavvyapp.in/sips/<UUID>`

## Create sip

```json
// body
{ "sip": 
  {
    "amount": "1000",
    "fund_code": "1234",
    "account_uuid": "aaaaa-bbbb-cccc-dddd",
    "partner_transaction_id": "xyv-abc",
    "start_date": "30/01/2022",
    "end_date": "30/12/2022",
    "frequency": "monthly",
    "redirect_url": "https://example.com/redirect"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/sips" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the sip JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/sips`

On success, a payment url is sent. The user must be redirected to this url to sign the e-nach mandate required for periodic auto-debits. If for some reason, the mandate fails to get signed, you can retry the mandate signing for a period of 24 hours, post which a new mandate must be created.

### Parameters

<aside class="notice">
Note the <code>sip</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
amount | true | `Decimal` Amount to be invested
fund_code | true | `Date` Code of the fund to be invested in
account_uuid | true | `String` If the deposit has to be created in an existing account.
partner_transaction_id | false | `String` Your custom ID to identify this transaction.
start_date | true | `Date` Start date of the SIP.
end_date | true | `Date` End date of the SIP.
frequency | true | `Enum: monthly, weekly, daily, ad-hoc` Frequency of debits 
redirect_url | true | `URL String` Where to direct the customer after payment. A field called `status` (as a query param) in the redirect URL will be available to indicate success or failure.

# Systematic Withdrawal Plans

This API describes how to create a SWP transaction. This API can only be accessed post the onboarding flow. The workflow for creating and registering an SWP is:

<div class="mermaid">
  graph TD;
  create_swp(Create SWP with amount and fund details)-->verify_otp(Verify OTP on registered phone number and/or email)
  verify_otp-- Failure -->verify_otp;
  verify_otp-- Success-->listen(Listen for transaction updates via webhook);
</div>

## SWP Object

Coming Soon!

# Systematic Transfer Plans

This API describes how to create a STP transaction, for periodic transfers from one fund to another in the same or different accounts. This API can only be accessed post the onboarding flow. The workflow for creating and registering an STP is:

<div class="mermaid">
  graph TD;
  create_stp(Create STP with amount and fund details)-->verify_otp(Verify OTP on registered phone number and/or email)
  verify_otp-- Failure -->verify_otp;
  verify_otp-- Success-->listen(Listen for transaction updates via webhook);
</div>

## STP Object

Coming soon!

# Risk Profiling

To improve the user experience in terms of fund selection, we offer a risk profiling API that takes customer profile information + a couple of risk related questions as inputs to output a maximum of 4 funds. While designed by our in-house MF experts, this API is provided "as-is", without any guarantees of correctness.

# Reports

We offer aggregated reports accross multiple categories for easy reconciliation and customer viewing.

# Frontend SDKs

For non-distributors, we offer the ability to use readymade SDKs for offering mutual fund onboarding and transactions. This exposes no private data of the customer. Marketing and non-fintech usecases should use this.

We also offer SDKs for simplified purchase transactions. Instead of redirecting the user to another page for payments, the SDK will allow you to complete the transaction on your assets itself.

# Webhooks

The webhooks API is an essential part of the Savvy API. There are several long running tasks and asynchronos activites that occur in the processing of onboardings and mutual fund transactions. For you to provide a good user experience, we offer webhooks that give you async updates about various activities you do via the Savvy API.

<aside class="warning">
Webhooks are sent on a "best-effort" basis, and are sent at least once. That means you may receive the same webhook twice, so make sure to have idempotency built in. For example, when receiving a deposit created webhook, make sure you haven't already seen the uuid before! 
</aside>

## Webhook structure

Parameter | Description
--------- | ----------- 
event | `String` Unique identifier of the withdrawal.
sent_at | `Datetime` Time at which the SIP was created.
payload | `JSON` Data sent as part of webhook.
signature | `String` SHA265 signature of the JSON payload.

## Events

Event | Description
--------- | ----------- 
`deposit.created` | Sent when a deposit is created
`deposit.status.updated` | Sent when a deposit status is updated. Use this for tracking success / failure of deposit transactions.
`withdrawal.created` | Sent when a withdrawal is created
`withdrawal.status.updated` | Sent when a withdrawal status is updated. Use this for tracking success / failure of withdrawal transactions.

## Webhook authentication

To make sure that the webhook is coming from Savvy, we append a signature to all webhooks in the `signature` parameter. The signature is a HMAC-SHA256 hash of the payload JSON string using your secret key:

`HMAC('sha256', secret_key, payload)`

Make sure to verify this signature before using the payload.

# Enums

## Annual Income Codes

## Gender Codes

## Occupation Codes

## Marital Status Codes

