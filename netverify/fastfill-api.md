![Jumio](/images/Jumio-Fastfill-Banner.png)

# Fastfill API Implementation Guide

This is a reference manual and configuration guide for the Fastfill API. It illustrates how to immediately retrieve information from an ID image.

### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).

## Table of Contents

- [Authentication and encryption](#authentication-and-encryption)
- [Using the Fastfill API](#using-the-fastfill-api)
  - [Request headers](#request-headers)
  - [Request body](#request-body)
  - [Response Parameter](#response-parameter)
  - [Examples](#examples)
- [Supported IDs](#supported-ids)
- [Supported Cipher Suites](#supported-cipher-suites)




---
# Authentication and encryption

All Document Verification API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [supported cipher suites](/netverify/supported-cipher-suites.md).

<br>

# Using the Fastfill API

Call the RESTful HTTP POST API **fastfill** from your server with the below parameters to create a transaction for each user. Simply submit front and/or back side image(s) of the user's document with a maximum file size of 5 MB and you will immediately receive the ID information as a JSON response.<br />
**Note:** Requests originating from the client are not supported (CORS is disabled) and the API credentials must never be exposed on the client side.

**HTTP request method:** `POST`<br>
**REST URL (US):** `https://netverify.com/api/netverify/v2/fastfill`<br>
**REST URL (EU):** `https://lon.netverify.com/api/netverify/v2/fastfill`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/fastfill`<br>

## Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: multipart/form-data`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in HTTP status code, **403 Forbidden**.

## Request body

### Request body — `multipart/form-data`

|Key			|Value				|
|:------		|:------			|
|**frontsideImage**	|JPEG, PNG file with max size of 5 MB<br>Front side image of the customer's document<br> <strong>Mandatory<br>-	if type = PASSPORT<br>-	If type = ID\_CARD or DRIVING\_LICENSE and backsideImage not provided</strong>|
|**backsideImage**|JPEG, PNG file with max size of 5 MB<br>Back side image of the customer's document if type = ID\_CARD or DRIVING\_LICENSE<br><strong>Mandatory if frontsideImage not provided</strong>|
|**metadata**	|Details of the user's document specified as a JSON object|


The `metadata` object in the body of your single-page **complete** API request allows you to

- specify what type of document is being submitted
- set the country of the uploaded document type.

<br>

**Required items appear in bold type.**  

### Request body — `metadata`

|Name               |Type   |Max. length|Description|
|:---               |:---   |:--        |:--        |             
|**type**|string||Possible types:<br>-	PASSPORT<br>-	ID\_CARD<br>- DRIVING\_LICENSE|
|**country**|string|3|Possible values:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|

## Response Parameter

**Note:** Mandatory parameters are marked with an asterisk * and highlighted bold.

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** *|String||Jumio’s reference number for each scan|
|type|String||Possible types:<br>-	PASSPORT<br>-	ID\_CARD<br>- DRIVING\_LICENSE|
|issueDate|String|255|Issue date in the format YYYY-MM-DD|
|issuingCountry|String|3|Possible countries:<br>- [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>-	XKX (Kosovo)|
|originatingCountry|||Possible countries:<br>- [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>-	XKX (Kosovo)|
|number|String|255|Identification number of the document|
|firstName|String|255|First name of the customer|
|lastName|String|255|Last name of the customer|
|dob|String|255|Date of birth in the format YYYY-MM-DD|
|expiry|String|255|Date of expiry in the format YYYY-MM-DD|
|optionalData1|String|255|Optional field of MRZ (TD1) line 1|
|optionalData2|String|255|Optional field of MRZ (TD1 and TD2) line 2|
|personalNumber|String|255|Personal number field of MRZ (MRP)|
|street|String|255|Street if type = DRIVING\_LICENSE|
|city|String|255|City if type = DRIVING\_LICENSE|
|state|String|255|State if type = DRIVING\_LICENSE|
|postalCode|String|255|Postal code if type = DRIVING\_LICENSE|
|gender|String|255|Possible values: <br>- M<br>-	F|

Optional parameters are returned:
-	If available on the ID
-	If readable
-	As readable (can be invalid)

## Examples

### Sample Request

```
POST https://netverify.com/api/netverify/v2/fastfill HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data;boundary=----xxxx
Content-Length: xxx
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
----xxxx
Content-Disposition: form-data; name="metadata"
Content-Type: application/json
{
  "type": "PASSPORT",
  "country": "USA"
}
----xxxx
Content-Disposition: form-data; name="frontsideImage"
Content-Type: image/jpeg
  // Your image
----xxxx
```

### Sample Response

```
{
  "dob": "1990-01-01",
  "expiry": "2022-12-31",
  "firstName": "FIRSTNAME",
  "issueDate": "2012-12-31",
  "issuingCountry": "USA",
  "lastName": "LASTNAME",
  "number": "P1234",
  "originatingCountry": "USA",
  "personalNumber": "12345",
  "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "PASSPORT"
}
```
---

# Supported IDs

The Fastfill API supports:

- All ID Verification supported passports

- ID cards from the following countries: Albania, Argentina, Austria, Belgium, Bosnia & Herzegovina, Bulgaria, Canada, Cameroon, Chile, Czech Republic, Denmark, Dominican Republic, Ecuador, El Salvador, Estonia, Finland, France, Georgia, Germany, Guatemala, Hungary, Italy, Kazakhstan, Kenya, Latvia, Liechtenstein, Macedonia, Malta, Mexico, Moldova, Montenegro, Netherlands, Pakistan, Paraguay, Peru, Poland, Portugal, Romania, Serbia, Slovakia, Slovenia, Spain, Sweden, Switzerland, Turkey, United Arab Emirates, United Kingdom.

- Driver licenses from the following countries: Austria, Belgium, Canada, Czech Republic, Denmark, Finland, France, Germany, Ireland, Italy, Netherlands, Poland, Portugal, Romania, Slovakia, Spain, Sweden, United Kingdom, United States.<br>
PDF417 bar code extraction from driver licenses back side from USA and Canada.

---
# Supported Cipher Suites
Jumio supported cipher suites during the TLS handshake.<p>
[View Supported Cipher Suites](/netverify/supported-cipher-suites.md)


---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
