![Jumio](/images/Jumio-ID-Verification-Banner.png)

# ID Verification Delete API Implementation Guide

This guide illustrates how to implement the ID Verification Delete API.

### Revision history

| Date    | Description|
|:--------|:------------|
| 2019-08-27  |Added Delete API for Authentication|
| 2019-01-30  |Document formatting|
| 2016-05-18  |Removed TLS\_DHE ciphers|
| 2015-10-20  |Added ECDHE ciphers to supported cipher suites|
| 2015-09-23  |Initial release|

---

## Table of Contents

- [Usage](#usage)
	- [Authentication and encryption](#authentication-and-encryption)
	- [Request headers](#request-headers)
- [Delete API for ID Verification](#delete-api-for-id-verification)
	- [Request body](#request-body)
	- [Response](#response)
	- [Example](#example)
- [Delete API for Authentication](#delete-api-for-authentication)
	- [Request body](#request-body-1)
	- [Response](#response-1)
	- [Example](#example-1)
- [Delete API for Document Verification](#delete-api-for-document-verification)
	- [Request body](#request-body-2)
	- [Response](#response-2)
	- [Example](#example-2)


---

# Usage

If you do not wish to store sensitive customer data after a verification has been completed, you can delete the image(s) and any extracted data in a transaction record either directly in the Customer Portal or via the Delete API.

|⚠️ Deletion is immediate and irreversible. Jumio cannot perform any quality or technical support investigations on transactions that have been deleted.
|:----------|
<br>

## Authentication and encryption
ID Verification API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.

You can generate a separate set of API credentials in the Customer Portal to use when retrieving or deleting transaction data under **Settings > API credentials > Transaction administration APIs**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|
<br>
The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md).<br>
<br>

## Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|
<br>

---

# Delete API for ID Verification

Use the HTTP `DELETE` method to call the RESTful API endpoint below. Specify the ID Verification transaction reference you want to delete as a path parameter.

**HTTP Request Method:** `DELETE`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<transactionReference>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<transactionReference>`<br>
**REST URL (SGP)**: `https://core-sgp.jumio.com/api/netverify/v2/scans/<transactionReference>`<br>
<br>

## Request body

**Required items appear in bold type.**  

|Name       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**transactionReference**| String|36|Jumio reference number for an existing ID Verification transaction in your account.|
<br>

## Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` as confirmation that you have successfully deleted the image(s) and extracted data from the specified transaction record.<br>
<br>

## Example
### Sample ID Verification delete request

~~~
DELETE https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
~~~
<br>

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

---

# Delete API for Authentication

Use the HTTP `DELETE` method to call the RESTful API endpoint below. Specify the Authentication transaction reference you want to delete as a path parameter.

**HTTP Request Method:** `DELETE`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/authentications/<transactionReference>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/authentications/<transactionReference>`<br>
**REST URL (SGP)**: `https://core-sgp.jumio.com/api/netverify/v2/authentications/<transactionReference>`<br>
<br>

## Request body

**Required items appear in bold type.**  

|Name       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**transactionReference**| String|36|Jumio reference number for an existing Authentication transaction in your account.|
<br>

## Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` as confirmation that you have successfully deleted PII data, images and facemap from the specified authentication transaction record.<br>
<br>

## Example
### Sample Authentication delete request

~~~
DELETE https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
~~~
<br>

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

---

# Delete API for Document Verification

Use the HTTP `DELETE` method to call the RESTful API endpoint below. Specify the Document Verification transaction reference you want to delete as a path parameter.

**HTTP Request Method:** `DELETE`<br>
**REST URL (US)**: `https://retrieval.netverify.com/api/netverify/v2/documents/<transactionReference>`<br>
**REST URL (EU)**: `https://retrieval.lon.netverify.com/api/netverify/v2/documents/<transactionReference>`<br>
**REST URL (SGP)**: `https://retrieval.core-sgp.jumio.com/api/netverify/v2/documents/<transactionReference>`<br>
<br>

## Request body

**Required items appear in bold type.**  

|Name       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**transactionReference**| String|36|Jumio reference number for an existing Document Verification transaction in your account.|
<br>

## Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` as confirmation that you have successfully deleted the image(s) and extracted data from the specified transaction record.<br>
<br>

## Example
### Sample Document Verification delete request

~~~
DELETE https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
~~~
<br>

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>



---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
