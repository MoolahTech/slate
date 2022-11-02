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
  submit_bank_details-->update_onboarding(Submit user details and other FATCA details);
  update_onboarding-->create_transaction(Move ahead to create a transaction);
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
    "pan_number": "ABCDE1234C",
    "amc_code": "MOF"
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
amc_code | true | `String` AMC code for which this onboarding is being done

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

## Update onboarding

```json
// body
{ "onboarding": 
  {
    "address": "67 Baker Street",
    "city": "Mumbai",
    "pincode": "900900",
    "date_of_birth": "27/06/1981",
    "occupation_code": "05"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>" \
  -X PUT \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the onboarding JSON object

### HTTP Request

`PUT http://surface.thesavvyapp.in/onboardings/<UUID>`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
address | true | `String` Street address
city | true | `String` Residential city of the investor
pincode | true | `String` Residential pincode of the investor
date_of_birth | true | `Date` Date of birth of the investor
occupation_code | true | `Enum String` Occupation of the investor

## Start Full KYC

```json
// body
{ "onboarding":
  {
    "email": "batman@gmail.com",
    "name": "Bruce Wayne",
    "phone_number": "+919009012345",
    "full_kyc_redirect_url": "https://example.com/redirect"
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

Parameter | Required | Description
--------- | ----------- | -----------
name | true | `String` Customer's name
email | true | `String` Customer's email
phone_number | true | `String` Customer's phone number
full_kyc_redirect_url | true | `URL` Where to redirect the user once the KYC is submitted to the AMC.

## Upload file

```json
// body
{
  "upload": "<file to upload>"
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/upload_file" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> Notice that to upload files, you must make a multi-part request. This will *not* work without appropriate multi-part headers. 

This endpoint uploads files to our server for further processing and analysis.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/upload_file`

### JSON Parameters

<aside class="notice">
Note the absence of <code>onboarding</code>root key!
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
upload | true | `File` The file you want to upload

### JSON response

Parameter | Required | Description
--------- | ----------- | -----------
file | true | `String` URL of the uploaded file

## Read pan card

```json
// body
{
  "onboarding": {
    "image_urls": ["https://s3.images.com/123"]
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/read_pan_card" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns extra params along with the onboarding object. Please make sure you're reading the response correctly.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/read_pan_card`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
image_urls | true | `Array` List of images to process. In the case of PAN card, we only need the front of the card.

### JSON response

Parameter | Required | Description
--------- | ----------- | -----------
onboarding | true | `Object` The normal onboarding object
name | true | `String` Name on the pan card
fathers_name | true | `String` Fathers name on the pan card
date_of_birth | true | `String` Date of birth on the pan card
pan_number | true | `String` Pan number on the pan card

**Note that the scanned details are not guaranteed to match the PAN card exactly. They're on a best effort basis.**

## Submit pan card

```json
// body
{
  "onboarding": {
    "name": "John Smith",
    "fathers_name": "Mark Smith",
    "date_of_birth": "01/01/1991",
    "pan_number": "ABCD1234C"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/pan_card" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns the onboarding object.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/pan_card`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
name | true | `String` Name on the pan card
fathers_name | true | `String` Fathers name on the pan card
date_of_birth | true | `String` Date of birth on the pan card
pan_number | true | `String` Pan number on the pan card

**The details submitted and the actual details on the PAN card must match. If they don't, the KYC will be rejected.**

## Read address proof

```json
// body
{
  "onboarding": {
    "address_proof_type": "aadhaar",
    "image_urls": ["https://s3.images.com/123", "https://s3.images.com/456"]
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/read_address_proof" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns extra params along with the onboarding object, depending on the type of address proof. Please make sure you're reading the response correctly.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/read_address_proof`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
address_proof_type | true | `Enum` One of `[aadhaar, voter_id, passport, license]`
image_urls | true | `Array` List of images to process. In the case of address proof, we need both back and front of the document.

### JSON response

Parameter | Required | Description
--------- | ----------- | -----------
onboarding | true | `Object` The normal onboarding object.
aadhaar_uid | false | `String` Masked aadhaar UID.
license_number | false | `String` License number.
passport_number | false | `String` Passport number.
voter_id_number | false | `String` Voter ID number.
name | true | true | `String` Name on the proof.
date_of_birth | true | `Date` Date of birth on the proof.
pincode | true | `String` Pincode on the proof.
address | true | `String` Street address on the proof.
district | true | `String` District on the proof.
city | true | `String` City on the proof.
state | true | `String` State on the proof.
issue_date | false | `Date` Only present in license and passport.
expiry_date | false | `Date` Only present in license and passport.
fathers_name | false | `String` Only present in license, passport and voter id.


**Note that the scanned details are not guaranteed to be correct. They're on a best effort basis.**

## Submit address proof

```json
// body
{
  "onboarding": {
    "address_proof_type": "aadhaar",
     "name": "John Smith",
     "expiry_date": "01/01/2029",
     "date_of_birth": "01/01/1991",
     "issue_date": "01/01/2000",
     "address":"23 Baker Street",
     "city": "Mumbai",
     "state": "Maharashtra",
     "district": "Malad",
     "pincode": "400051",
     "license_number": "12345678",
     "aadhaar_uid": "",
     "passport_number": "",
     "voter_id_number": ""
   }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/address_proof" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns the onboarding object.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/address_proof`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
address_proof_type | true | `String` type of address proof being processed.
name | true | `String` Name on proof.
expiry_date | maybe (refer above) | `String` Expiry date of proof.
date_of_birth | maybe (refer above) | `String` Date of birth on proof.
issue_date | maybe (refer above) | `String` Issue date of proof.
address | true | `String` Street address on proof.
city | true | `String`  City on proof.
state | true | `String` State on proof.
district | true | `String` District on proof.
pincode | true | `String` Pincode on proof.
license_number | maybe (refer above) | `String` License number in case of license.
aadhaar_uid | maybe (refer above) | `String` Aadhaar UID in case of aadhaar
passport_number | maybe (refer above) | `String` Passport number in case of passort
voter_id_number | maybe (refer above) | `String` Voter ID number in case of voter ID

**The details submitted and the actual details on the address proof must match. If they don't, the KYC will be rejected.**

## Submit Investor Details

```json
// body
{
  "onboarding": {
    "gender": "M",
    "marital_status": "UNMARRIED",
    "occupation_description": "Professional",
    "occupation_code": "04",
    "citizenship_code": "101",
    "citizenship_country": "India",
    "application_status_code": "01",
    "application_status_description": "New",
    "annual_income": "31"
   }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/form" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns the onboarding object.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/form`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
gender | true | `Enum` Refer to Enum tables at the bottom
marital_status | true | `Enum` Refer to Enum tables at the bottom
occupation_description | true | `Enum` Refer to Enum tables at the bottom
occupation_code | true | `Enum` Refer to Enum tables at the bottom
citizenship_code | true | `Enum` Refer to Enum tables at the bottom
citizenship_country | true | `Enum` Refer to Enum tables at the bottom
application_status_code | true | `Enum` Refer to Enum tables at the bottom
application_status_description | true | `Enum` Refer to Enum tables at the bottom
annual_income | true | `Enum` Refer to Enum tables at the bottom

## Upload signature

```json
// body
{
  "onboarding": {
    "image_urls": ["https://s3.images.com/123"]
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/signature" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns the onboarding object.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/signature`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
image_urls | true | `Array` List of images to process. In the case of signature, we need just one image.

**Note that the signature must be against a white background. This can be either a physical signature on a white background, or a digital signature on a white background. The form factor is up to you**

## Upload selfie

```json
// body
{
  "onboarding": {
    "image_urls": ["https://s3.images.com/123"]
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/selfie" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns the onboarding object.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/selfie`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
image_urls | true | `Array` List of images to process. In the case of selfie, we need just one image.

## Start Video Verification

```json
// body
{}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/start_video_verification" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns extra params along with the onboarding object. Please make sure you're reading the response correctly.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/start_video_verification`

### **Important Info about this API**

The API will return a random 6 digit number. The user must say this number on video. This allows to do a "liveness check" (make sure it's not a pre-recorded video).

### JSON Response

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
onboarding | true | Regular onboarding object
transaction_id | true | ID uniquely identifying this video verification
random_number | true | Random number to say on video

## Submit Video Verification

```json
// body
{
  "onboarding": {
    "video_url": "https://s3.videos/123",
    "transaction_id": "12345"
   }
}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/video_verification" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns the onboarding object.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/video_verification`

### JSON Parameters

<aside class="notice">
Note the <code>onboarding</code>root key
</aside>

Parameter | Required | Description
--------- | ----------- | -----------
video_url | true | `URL` URL of the uploaded video via the upload file API.
transaction_id | true | `String` ID returned by the start video verification API.

## Generate KYC Contract

```json
// body
{}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/generate_contract" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns extra params along with the onboarding object. Please make sure you're reading the response correctly.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/generate_contract`

### **Important Info about this API**

The API will return a url which you must redirect the user to. The webpage will show all the information submitted so far, and the user must sign the contract displayed with their aadhaar e-signature. The page has been known to not play well with app-embedded browsers. So it's recommended that your redirect the user to the actual browser on the phone.

### JSON Response

Parameter | Required | Description
--------- | ----------- | -----------
onboarding | true | Regular onboarding object.
url | true | Aadhaar e-sign contract URL.
random_number | true | Random number to say on video

## Execute Verification

```json
// body
{}
```

```shell
curl "http://surface.thesavvyapp.in/onboardings/<UUID>/execute_verification" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body
```

> This API returns the onboarding object.

### HTTP Request

`POST http://surface.thesavvyapp.in/onboardings/<UUID>/execute_verification`

### **Important Info about this API**

On success response of this API, the investors' details are pushed to the KRA/AMC for verification. Wait for the webhook to be informed of success/failure and the reasons for failure, if any.
In case there is a failure, you can use the API to update the same onboarding. You will, however, have to generate a new contract, get a new aadhaar e-sign and execute verification again.

### JSON Response

Parameter | Required | Description
--------- | ----------- | -----------
onboarding | true | Regular onboarding object.

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
fund_code | `String` Code of the fund invested in.
fund_name | `String` Name of the fund invested in.
amount | `Integer` Original amount of the investment.
current_amount | `Decimal` Current value of the investment.
units | `Decimal` Units allocated for the investment.
status | `Enum: created, payment_made, submitted_to_rta, completed, error` Status of the investment.
status_description | `String` Reason in case of failure / rejections.
reinvest_mode | `Enum: payout: 'N', reinvestment: 'Y', growth: 'Z', bonus: 'B'` What should happen with any dividend that is paid out.
partner_transaction_id | `String` Your ID associated with the transaction.
user_completed_payment_at | `Datetime` Time at which payment was completed by the user.
transferred_to_amc_at | `Datetime` Time at which the payment was transferred to the AMC.
created_at | `Datetime` Time at which the deposit was created.  
sip_uuid | `String` Identifier of the SIP that the deposit is part of.
stp_uuid | `String` Identifier of the STP that the deposit is part of

## Get deposits

```shell
curl "http://surface.thesavvyapp.in/secure/deposits?account_uuid=<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns an array of deposit JSON objects.

### HTTP Request

`GET http://surface.thesavvyapp.in/secure/deposits?account_uuid=<UUID>`

### URL Parameters

Parameter | Required | Description
--------- | ------- | -----------
account_uuid | true | `String` All deposits associated with a account

## Show deposit

```shell
curl "http://surface.thesavvyapp.in/secure/deposits/<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns a deposit JSON object.

### HTTP Request

`GET http://surface.thesavvyapp.in/secure/deposits/<UUID>`

## Create deposit

```json
// body
{ "deposit": 
  {
    "amount": "1000",
    "fund_code": "1234",
    "account_uuid": "aaaaa-bbbb-cccc-dddd",
    "onboarding_uuid": "aaaaa-bbbb-cccc-dddd",
    "redirect_url": "https://example.com/payment_redirect"
  }
}
```

```shell
curl "http://surface.thesavvyapp.in/secure/deposits" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the deposit JSON object along with the payment URL

### HTTP Request

`POST http://surface.thesavvyapp.in/secure/deposits`

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
onboarding_uuid | false | `String` Mandatory if account uuid is not specified.
partner_transaction_id | false | `String` Your custom ID to identify this transaction.

### JSON response

Parameter | Required | Description
--------- | ----------- | -----------
deposit | true | `Object` Deposit object
url | true | `String` URL to redirect the customer to for payment

<aside class="warning">If account uuid is not specified, a new account will be created for the customer. Make sure this is the behavior you want. However, for first time customers, not passing the account id is the expected behavior.</aside>

## Create basket of deposits

```json
// body
{ "deposit": {
  "account_uuid": "aaaaa-bbbb-cccc-dddd",
  "onboarding_uuid": "aaaaa-bbbb-cccc-dddd",
  "redirect_url": "https://example.com/payment_redirect",
  "deposit_parts": [
    {
      "amount": "1000",
      "fund_code": "1234"
    },
    {
      "amount": "100",
      "fund_code": "xyz"
    }
  ]}
}
```

```shell
curl "http://surface.thesavvyapp.in/secure/deposits/create_basket" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the deposit JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/secure/deposits/create_basket`

### Parameters

<aside class="notice">
Note the <code>deposit</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
redirect_url | true | `String` Where to direct the customer after payment. A field called `status` (as a query param) in the redirect URL will be available to indicate success or failure.
account_uuid | false | `String` If the deposit has to be created in an existing account.
onboarding_uuid | false | `String` Mandatory if account uuid is not specified.
partner_transaction_id | false | `String` Your custom ID to identify this transaction.
deposit_parts | true | `Array` [amount, fund_code] to specify how much to invest in which funds

### JSON response

Parameter | Required | Description
--------- | ----------- | -----------
deposits | true | `Objects` Deposit object list
url | true | `String` URL to redirect the customer to for payment

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
curl "http://surface.thesavvyapp.in/secure/withdrawals?account_uuid=<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns an array of withdrawal JSON objects.

### HTTP Request

`GET http://surface.thesavvyapp.in/secure/withdrawals?account_uuid=<UUID>`

### URL Parameters

Parameter | Required | Description
--------- | ------- | -----------
account_uuid | true | `String` All withdrawals associated with a account

## Show withdrawal

```shell
curl "http://surface.thesavvyapp.in/secure/withdrawals/<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns a withdrawal JSON object.

### HTTP Request

`GET http://surface.thesavvyapp.in/secure/withdrawals/<UUID>`

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
curl "http://surface.thesavvyapp.in/secure/withdrawals" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the withdrawal JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/secure/withdrawals`

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
curl "http://surface.thesavvyapp.in/secure/withdrawals/<UUID>/verify_otp" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the withdrawal JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/secure/withdrawals/<UUID>/verify_otp`

On success, the status of the withdrawal will change to `otp_complete`. The OTP verification will expire in 5 minutes, and the OTP can be resent in case of delivery failures after 30 seconds of first attempt. 

### Parameters

<aside class="notice">
Note the <code>withdrawal</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
otp | true | `String` The user inputted OTP

# One-click Checkout (OCC) / Investment links

This API describes how to create instant investment links. This API can handle any scenario (existing folio, existing investor but no folio, and non-existing investor). This comes as a hosted solution under your logo.

## OCC object

Parameter | Description
--------- | ----------- 
account_uuid | `String` Account associated with the investment.
masked_folio_number | `String` Masked folio number, last 3 digits only.
fund_name | `String` Name of the fund invested in.
amc_name | `String` Name of AMC for the fund invested in.
checkout_type | `Enum: one_time, ongoing` Whether the checkout type is a lumpsum or a SIP.
start_date | `Date` Start date of the SIP.
end_date | `Date` End date of the SIP.
frequency | `Enum: monthly, weekly, daily, ad-hoc` Frequency of debits
amount | `Integer` Amount of lumpsum or monthly instalment of SIP
deposit_request_id | `String` Payment request ID of the investment.
deposit_status | `Enum: created, payment_made, submitted_to_rta, completed, error` Status of the investment.
partner | `Object` Partner details. See next table.
sip_request_id | `String` Payment returnsuest ID of the SIP.
sip_mandate_status | `Enum: pending, accepted_by_user, completed, error, cancelled` Status of the SIP auto-debit.
short_link | `URL string` This URL can be shared with the user to go to for transaction completion.

## Create OCC (Lumpsum)

```json
// body
{ "one_click_checkout": 
  {
    "onboarding": {
      "name": "James Bond",
      "phone_number": "9800011111",
      "email": "email@example.com",
      "pan_number": "ABCDE1234C"
    },
    "account": {
      "folio_number": "12345678",
      "amc_code": "MOS",
      "type": "SI"
    },
    "bank_account": {
      "account_number": "000111199999",
      "ifsc_code": "HDFC0000001",
      "bank_name": "HDFC BANK LTD"
    },
    "deposits": [{
      "amount": 1000,
      "mutual_fund_id": 10
    },
    {
      "amount": 2000,
      "mutual_fund_id": 20
    }]
  }
}
```

```shell
curl "https://surface.thesavvyapp.in/secure/one_click_checkouts" \
  -X POST \
  -d body
  -H "Authorization: Bearer <token>"
```
> The above command returns a OCC JSON object.

### HTTP Request

`POST https://surface.thesavvyapp.in/secure/one_click_checkouts`

### Parameters

<aside class="notice">
Note the <code>one_click_checkout</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
onboarding | false | `Object` Investor details
account | false | `Object` Account details
bank_account | false | `Object` Bank account details
deposit | true | `Object` Transaction details


Onboarding

Parameter | Required | Description
--------- | ------- | -----------
name | true | `String` Name of investor
phone_number | true | `String` Phone number of investor
email | true | `String` Email of investor
pan_number | true | `String` PAN of investor


Account

Parameter | Required | Description
--------- | ------- | -----------
folio_number | true | `String` Folio number of investor
amc_code | true | `String` AMC code of the folio
type | true | `String` Holding pattern of folio


Bank Account

Parameter | Required | Description
--------- | ------- | -----------
account_number | true | `String` Bank account number
ifsc_code | true | `String` IFSC code
bank_name | true | `String` Name of bank


Deposits (Array)

Parameter | Required | Description
--------- | ------- | -----------
amount | true | `Integer` Amount of the investment
mutual_fund_id | true | `String` Fund ID

## Create OCC (SIP)

```json
// body
{ "one_click_checkout": 
  {
    "onboarding": {
      "name": "James Bond",
      "phone_number": "9800011111",
      "email": "email@example.com",
      "pan_number": "ABCDE1234C"
    },
    "account": {
      "folio_number": "12345678",
      "amc_code": "MOS",
      "type": "SI"
    },
    "bank_account": {
      "account_number": "000111199999",
      "ifsc_code": "HDFC0000001",
      "bank_name": "HDFC BANK LTD"
    },
    "sips": [{
      "amount": 1000,
      "mutual_fund_id": 10
    },
    {
      "amount": 1000,
      "mutual_fund_id": 10
    }],
    "sip_day": 1,
    "number_of_installments": 12
  }
}
```

```shell
curl "https://surface.thesavvyapp.in/secure/one_click_checkouts/create_sip" \
  -X POST \
  -d body \
  -H "Authorization: Bearer <token>"
```
> The above command returns a OCC JSON object.

### HTTP Request

`POST https://surface.thesavvyapp.in/secure/one_click_checkouts/create_sip`

### Parameters

<aside class="notice">
Note the <code>one_click_checkout</code>root key
</aside>

Parameter | Required | Description
--------- | ------- | -----------
onboarding | false | `Object` Investor details
account | false | `Object` Account details
bank_account | false | `Object` Bank account details
sip | true if deposit is null | `Object` Transaction details
sip_day | false | `Integer` Day of the month that the SIP should be triggered
number_of_installments | false | `Integer` Number of instalments


Onboarding

Parameter | Required | Description
--------- | ------- | -----------
name | true | `String` Name of investor
phone_number | true | `String` Phone number of investor
email | true | `String` Email of investor
pan_number | true | `String` PAN of investor


Account

Parameter | Required | Description
--------- | ------- | -----------
folio_number | true | `String` Folio number of investor
amc_code | true | `String` AMC code of the folio
type | true | `String` Holding pattern of folio


Bank Account

Parameter | Required | Description
--------- | ------- | -----------
account_number | true | `String` Bank account number
ifsc_code | true | `String` IFSC code
bank_name | true | `String` Name of bank


SIPs (Array)

Parameter | Required | Description
--------- | ------- | -----------
amount | true | `Integer` 
mutual_fund_id | true | `String` Fund ID

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
amount | `Integer` Amount to be deposited at each installment.
payment_link | `String` **Only applicable for create API** URL to which to take the user for mandate checkout.
redirect_url | `String` Where to direct the customer after payment. A field called `status` (as a query param) in the redirect URL will be available to indicate success or failure.
active | `Boolean` Whether the SIP is actively debiting.
partner_transaction_id | `String` Your ID associated with the transaction.
created_at | `Datetime` Time at which the SIP was created.

## Get sips

```shell
curl "http://surface.thesavvyapp.in/secure/sips?account_uuid=<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns an array of sip JSON objects.

### HTTP Request

`GET http://surface.thesavvyapp.in/secure/sips?account_uuid=<UUID>`

### URL Parameters

Parameter | Required | Description
--------- | ------- | -----------
account_uuid | true | `String` All sips associated with a account

## Show sip

```shell
curl "http://surface.thesavvyapp.in/secure/sips/<UUID>" \
  -X GET \
  -H "Authorization: Bearer <token>"
```
> The above command returns a sip JSON object.

### HTTP Request

`GET http://surface.thesavvyapp.in/secure/sips/<UUID>`

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
curl "http://surface.thesavvyapp.in/secure/sips" \
  -X POST \
  -H "Authorization: Bearer <token>" \
  -d body

```
> The above command returns the sip JSON object

### HTTP Request

`POST http://surface.thesavvyapp.in/secure/sips`

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

# Save Now, Buy Later (SNBL)

SNBL is a unique proposition, where customers can save up (via SIP) to purchase products from merchants. This service is provided as an SDK to merchants so they can embed SNBL inside their purchase journeys.

## Importing the SDK

Get the SDK from our CDN: `https://cdn.savvyapp.in/lib/savvypayCoreSDK.min.js`

```html
<script
  defer="defer"
  src="https://cdn.savvyapp.in/lib/savvypayCoreSDK.min.js" />
```

## Using the SDK

The SDK accepts a number of parameters, depending on what information is available to the merchant:

```html
<script>
  function doPayment() {
      var config = {
        key: "XXXXX-XXXXX",
        offerUuid: "XXXXX-XXXXX",
        isProduction: false,
        isDisplayDetail: false,
        minAmount: 10000,
        maxAmount: 20000,
        endDate: "DD/MM/YYYY",
        productName: "Mauritius Vacation",
        multiplier: 1,
        amcCode: 'IPRU',
        merchantLogo: 'https://www.logo.com',
        merchantName: 'Name',
        onComplete: (data) => {
          console.log("ON_COMPLETE_OCCURED", data);
          alert("Savvy completed successfully!");
        },
        onUserExit: () => {
          console.log("ON_USER_EXITED");
        }}

      var savvySDK = new SavvySDK(config);
        savvySDK.start();
    }
  }
</script>
...

<button onclick="doPayment()">Do Payment</button>
```

## Track investment SDK usage

After SNBL setup, a customer may want to track their investment. In this case, pass one extra option, `isDisplayDetail` as `true` to the same SDK:

```html
<script>
  function showTracker() {
      var config = {
        key: "XXXXX-XXXXX",
        isProduction: false,
        isDisplayDetail: true,
        amcCode: 'IPRU',
        merchantLogo: 'https://www.logo.com',
        merchantName: 'Name',
        onUserExit: () => {
          console.log("ON_USER_EXITED");
        }}

      var savvySDK = new SavvySDK(config);
        savvySDK.start();
    }
  }
</script>
...

<button onclick="showTracker()">Show Tracker</button>
```

## SNBL SDK object

Parameter | Required | Description
--------- | -------- | ----------- 
key | true | `String` API key provided to the merchant.
offerUuid | true | `String` Product offer uuid. This can be static, or dynamic. It is either created by the merchant at purchase time, or created by Savvy and provided to the merchant.
isProduction | true | `Boolean` Production request or not
isDisplayDetail | false | `Boolean` Whether to open in track investment mode or not.
minAmount | false | `Decimal` Min amount, if the amount is a range.
maxAmount | false | `Decimal` Max amount, if the amount is a range, or the exact amount in case there is no range.
endDate | false | `Date` Targeted purchase date of the product.
productName | false | `String` Name of the product being purchased.
multiplier | false | `Integer` The amount of products being bought.
amcCode | true | `String` Code of the AMC being used.
merchantLogo: | true | `String` Image for the merchant logo.
merchantName: | true | `String` Name of the merchant.
onComplete(data) | true | `Function` Callback hook on completion of SIP setup.
onUserExit | true | `Function` Callback hook if user exited before completion.

## AMC name & fund

This API is used for displaying the name of the fund, AMC name and link.

```shell
curl "http://surface.thesavvyapp.in/amcs/<AMC_CODE>/name_and_liquid_fund" \
  -X GET \
  -H "X-PARTNER-ACCESS-KEY: <YOUR KEY>"
```
> The above command returns a JSON object.

### HTTP Request

`GET http://surface.thesavvyapp.in/amcs/<AMC_CODE>/name_and_liquid_fund`

### Response parameters

Parameter | Description
--------- | -----------
amc_name | AMC name
amc_logo | Link to amc logo
liquid_fund | Name of the fund
liquid_fund_link | Link to the fund

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

Code | Description
--------- | ----------- 
31 | Below 1 Lac
32 | 1-5 Lacs
33 | 5-10 Lacs
34 | 10-25 Lacs
35 | 25 Lacs-1 crore
36 | 1 crore

## Gender Codes

Code | Description
--------- | ----------- 
M | Male
F | Female
T | Transgender

## Occupation Codes

Code | Description
--------- | ----------- 
01 | Private Sector
02 | Public Sector
03 | Business
04 | Professional
06 | Retired
07 | Housewife
08 | Student
10 | Government Sector
99 | Others
11 | Self Employed
12 | Not Categorized

## Marital Status Codes

Code | Description
--------- | -----------
MARRIED | Married
UNMARRIED | Unmarried
OTHERS | Others

## Country Codes

Code | Description
--------- | -----------
India| 101
Albania| 003
Aland Islands| 002
Afghanistan| 001
Algeria| 004
American Samoa | 005
Andorra| 006
Angola | 007
Anguilla | 008
Antarctica | 009
Antigua And Barbuda| 010
Argentina| 011
Armenia| 012
Aruba| 013
Australia| 014
Austria| 015
Azerbaijan | 016
Bahamas| 017
Bahrain| 018
Bangladesh | 019
Barbados | 020
Belarus| 021
Belgium| 022
Belize | 023
Benin| 024
Bermuda| 025
Bhutan | 026
Bolivia| 027
Bosnia And Herzegovina | 028
Botswana | 029
Bouvet Island| 030
Brazil | 031
British Indian Ocean Territory | 032
Brunei Darussalam| 033
Bulgaria | 034
Burkina Faso | 035
Burundi| 036
Cambodia | 037
Cameroon | 038
Canada | 039
Cape Verde | 040
Cayman Islands | 041
Central African Republic | 042
Chad | 043
Chile| 044
China| 045
Christmas Island | 046
Cocos (Keeling) Islands| 047
Colombia | 048
Comoros| 049
Congo| 050
Congo, The Democratic Republic Of The| 051
Cook Islands | 052
Costa Rica | 053
Cote D'Ivoire| 054
Croatia| 055
Cuba | 056
Cyprus | 057
Czech Republic | 058
Denmark| 059
Djibouti | 060
Dominica | 061
Dominican Republic | 062
Ecuador| 063
Egypt| 064
El Salvador| 065
Equatorial Guinea| 066
Eritrea| 067
Estonia| 068
Ethiopia | 069
Falkland Islands (Malvinas)| 070
Faroe Islands| 071
Fiji | 072
Finland| 073
France | 074
French Guiana| 075
French Polynesia | 076
French Southern Territories| 077
Gabon| 078
Gambia | 079
Georgia| 080
Germany| 081
Ghana| 082
Gibraltar| 083
Greece | 084
Greenland| 085
Grenada| 086
Guadeloupe | 087
Guam | 088
Guatemala| 089
Guernsey | 090
Guinea | 091
Guinea-Bissau| 092
Guyana | 093
Haiti| 094
Heard Island And Mcdonald Islands| 095
Holy See (Vatican City State)| 096
Honduras | 097
Hong Kong| 098
Hungary| 099
Iceland| 100
Indonesia| 102
Iran, Islamic Republic Of| 103
Iraq | 104
Ireland| 105
Isle Of Man| 106
Israel | 107
Italy| 108
Jamaica| 109
Japan| 110
Jersey | 111
Jordan | 112
Kazakhstan | 113
Kenya| 114
Kiribati | 115
Korea, Democratic People’s Republic Of | 116
Korea, Republic Of | 117
Kuwait | 118
Kyrgyzstan | 119
Lao People’s Democratic Republic | 120
Latvia | 121
Lebanon| 122
Lesotho| 123
Liberia| 124
Libyan Arab Jamahiriya | 125
Liechtenstein| 126
Lithuania| 127
Luxembourg | 128
Macao| 129
Macedonia, The Former Yugoslav Republic Of | 130
Madagascar | 131
Malawi | 132
Malaysia | 133
Maldives | 134
Mali | 135
Malta| 136
Marshall Islands | 137
Martinique | 138
Mauritania | 139
Mauritius| 140
Mayotte| 141
Mexico | 142
Micronesia, Federated States Of| 143
Moldova, Republic Of | 144
Monaco | 145
Mongolia | 146
Montserrat | 147
Morocco| 148
Mozambique | 149
Myanmar| 150
Namibia| 151
Nauru| 152
Nepal| 153
Netherlands| 154
Netherlands Antilles | 155
New Caledonia| 156
New Zealand| 157
Nicaragua| 158
Niger| 159
Nigeria| 160
Niue | 161
Norfolk Island | 162
Northern Mariana Islands | 163
Norway | 164
Oman | 165
Pakistan | 166
Palau| 167
Palestinian Territory, Occupied| 168
Panama | 169
Papua New Guinea | 170
Paraguay | 171
Peru | 172
Philippines| 173
Pitcairn | 174
Poland | 175
Portugal | 176
Puerto Rico| 177
Qatar| 178
Reunion| 179
Romania| 180
Russian Federation | 181
Rwanda | 182
Saint Helena | 183
Saint Kitts And Nevis| 184
Saint Lucia| 185
Saint Pierre And Miquelon| 186
Saint Vincent And The Grenadines | 187
Samoa| 188
San Marino | 189
Sao Tome And Principe| 190
Saudi Arabia | 191
Senegal| 192
Serbia And Montenegro| 193
Seychelles | 194
Sierra Leone | 195
Singapore| 196
Slovakia | 197
Slovenia | 198
Solomon Islands| 199
Somalia| 200
South Africa | 201
South Georgia And The South Sandwich Islands | 202
Spain| 203
Sri Lanka| 204
Sudan| 205
Suriname | 206
Svalbard And Jan Mayen | 207
Swaziland| 208
Sweden | 209
Switzerland| 210
Syrian Arab Republic | 211
Taiwan, Province Of China| 212
Tajikistan | 213
Tanzania, United Republic Of | 214
Thailand | 215
Timor-Leste| 216
Togo | 217
Tokelau | 218
Tonga | 219
Trinidad And Tobago | 220
Tunisia | 221
Turkey | 222
Turkmenistan | 223
Turks And Caicos Islands | 224
Tuvalu | 225
Uganda | 226
Ukraine| 227
United Arab Emirates | 228
United Kingdom | 229
United States| 230
United States Minor Outlying Islands | 231
Uruguay| 232
Uzbekistan | 233
Vanuatu| 234
Venezuela| 235
Viet Nam | 236
Virgin Islands, British | 237
Virgin Islands, U.S. | 238
Wallis And Futuna | 239
Western Sahara | 240
Yemen | 241
Zambia | 242
Zimbabwe | 243
Côte D'ivoire | CI
Korea,Democratic People'sRepublicOf | KP
Lao People’s Democratic Republic | 12|0
