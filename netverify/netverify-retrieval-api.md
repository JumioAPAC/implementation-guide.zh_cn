![Jumio](/images/Jumio-ID-Verification-Banner.png)

# ID Verification Retrieval API Implementation Guide

This guide describes how to implement the ID Verification Retrieval API.


### Revision history

| Date    | Description|
|:--------|:------------|
| 2021-12-07   |Added reject reason code 214 for MISSING\_FRONT |
| 2021-11-15   |Added reject reason code 116 for SUPERIMPOSED\_TEXT<br>Added reject reason code 118 for MISMATCH\_FRONT\_BACK<br>Added reject reason description code 2007 for MISSING\_MANDATORY\_DATAPOINTS |
| 2021-09-01   |Added reject reason code 115 for MISMATCH\_HRZ\_MRZ\_DATA |
| 2021-06-23   |Added reject reason code 113 for MISMATCHING\_DATA\_REPEATED\_FACE |
| 2021-05-21   |Added "personalNumberValidation" to "additionalChecks" |
| 2020-08-15   |Removed EU and US address format |
| 2020-05-26   |Added "enrollmentTransactionReference" to Authentication retrieval |
| 2020-03-03   |Added "faceSearchFindings" to "additionalChecks" |
| 2020-01-27   |Added "addressValidation" and "proofOfResidency" to "additionalChecks" |
| 2019-07-23   |Added response parameter "facemap" for ID Verification |
| 2019-05-28   |Added Retrieval API for Authentication|
| 2019-04-17   |Formatting and corrections|
| 2019-02-15   |Added value "OTHER" to parameter "type" for Document Verification|
| 2019-01-31   |Added new validity reason "LIVENESS\_FAILED"|
| 2019-01-15   |Added response parameter "livenessImages" for ID Verification|
| 2018-11-12   |Added response parameter "issuingAuthority", "issuingPlace", and value "PUBLIC\_SAFETY\_ID" to <br/>parameter "idSubtype", updated idSubtype "MILITARY\_ID"|
| 2018-10-31   |Updated usage guidelines|
| 2018-10-02   |Add swiftCode to Parameter "extractedData" for BS, removed deprecated faceMatch percentages|
| 2018-02-01   |Added response parameter "originalDocument" for Document Verification Retrieval <br />Retrieving document data only<br />Added Australia and Canada states to response parameter "usState" |
| 2018-01-17   |Added new validity reason BLACK\_AND\_WHITE<br> Updated supported countries for idSubtype LEARNING\_DRIVING\_LICENSE|
| 2017-11-23   |Added value "Visa" to response parameter "type" for ID Verification Retrieval - Retrieving Document <br>Data only|
| 2017-09-21   |Removed response parameter "additionalInformation"|
| 2017-08-24   |Added response parameter "identityVerification"|
| 2017-05-23   |Added response parameter "issuingDate" for retrieving scan details and<br />document data only|
| 2017-01-31   |Added value "PASSPORT\_CARD\_ID" and "MILITARY\_ID" to parameter "idSubType" |
| 2016-11-15   |Added reject reason detail code 10011 for PLACE\_OF\_BIRTH |
| 2016-06-21   |Removed Content-Length header for HTTP GET APIs |
| 2016-05-18   |Added value "STUDENT\_ID" to parameter "idSubType"<br />Removed TLS\_DHE ciphers |
| 2016-02-09   |Added reject reason code 111 for MISMATCH\_PRINTED\_BARCODE\_DATA |
| 2015-11-17   |Added parameter "idSubtype" to ID Verification Retrieval |
| 2015-10-21   |Added ECDHE ciphers to supported cipher suites |
| 2015-09-23   |Added parameter "country" and "customDocumentCode" for ID Verification Multi Document<br/>Retrieval |
| 2015-09-09   |Added ID Verification Multi Document Retrieval<br />Added second set of API credentials for retrieving transaction data |
| 2015-08-11   |Added new classifier "face" for retrieving scan images |
| 2015-07-14   |Added value "XKX" for Kosovo to parameters "country" and "issuingCountry"<br />Added value "Kosovo" to parameter "usState" |
| 2015-06-30   |Introduced EU data center |
| 2015-04-08   |Added reject reason code 109 for PUNCHED_DOCUMENT |
| 2015-03-24   |Removed cipher TLS\_RSA\_WITH\_RC4\_128\_SHA due to RC4 deprecation |
| 2014-03-05   |Added sources WEB and REDIRECT<br />Document type not mandatory anymore |
| 2014-09-25   |Changed parameter "maskhint" to apply for credit cards only |
| 2014-08-19   |Initial release |

---

## Table of Contents

- [Usage](#usage)
	- [Authentication and encryption](#authentication-and-encryption)
	- [Request headers](#request-headers)
- [ID Verification retrieval](#id-verification-retrieval)
    - [Retrieving status](#retrieving-status)
    - [Retrieving details](#retrieving-details)
    - [Retrieving document data only](#retrieving-document-data-only)
    - [Retrieving transaction data only](#retrieving-transaction-data-only)
    - [Retrieving verification data only](#retrieving-verification-data-only)
    - [Retrieving available images](#retrieving-available-images)
    - [Retrieving a specific image](#retrieving-a-specific-image)
	- [Retrieving liveness images](#retrieving-liveness-images)
- [Authentication retrieval](#authentication-retrieval)
	- [Retrieving details](#retrieving-details-1)
	- [Retrieving a specific image](#retrieving-a-specific-image-1)
	- [Retrieving liveness images](#retrieving-liveness-images-1)
- [Document Verification retrieval](#document-verification-retrieval)
    - [Retrieving status ](#retrieving-status-1)
    - [Retrieving details](#retrieving-details-2)
    - [Retrieving document data only](#retrieving-document-data-only-1)
    - [Retrieving transaction data only](#retrieving-transaction-data-only-1)
    - [Retrieving available images](#retrieving-available-images-1)
    - [Retrieving a specific image](#retrieving-a-specific-image-2)


---
# Usage

The Retrieval API is an alternative way to retrieve the results of a Jumio transaction for customers who choose not to implement our callback functionality, or in cases where a callback could not be received or processed.

Please contact Jumio Support at support@jumio.com to coordinate bulk requests to the Retrieval API.

### Best Practice:

- You must use the [status retrieval](/netverify/netverify-retrieval-api.md#retrieving-status) for all requests until the transaction is no longer in PENDING status.
- If the transaction status is DONE or FAILED (not pending), retrieve details and image(s) once.
-  Maximum of 10 consecutive retrieval attempts after successful image acquisition <br />(SDK/Web: User journey finshed; API: after ID Verification API call)<br />
  * If a final result is not available after 10 attempts, you are allowed to perform an additional retrieval call once per day.
- Request timings recommendation:
  * 40, 60, 100, 160, 240, 340, 460, 600, 760, 940 seconds
  * You are also allowed to set your own definition.

## Authentication and encryption
ID Verification API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password.
<br>

You can view and manage your API token and secret in the Customer Portal under **Settings** > **API credentials**. You can create a separate set of API credentials to be used specifically with the Retrieval API and [Delete API](/netverify/netverify-delete-api.md) in the Customer Portal under **Settings** > **API credentials** > **Transaction administration APIs**.


|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|


The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md).

<br>

## Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json` **or** `image/jpeg, image/png` when retrieving a specific image<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

<br>

---

# ID Verification retrieval

## Retrieving status

Call the RESTful API GET endpoint below to retrieve the status of a ID Verification transaction by specifying the Jumio transaction reference (scan reference) of an existing transaction from your account as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/scans/<scanReference>`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**timestamp** |string| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference** |string|36|Jumio’s reference number for the transaction|
|**status** |string| |Possible states:<br />•	PENDING<br />•	DONE<br />•	FAILED|

<br>

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx

```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

```
{
    "timestamp": "2019-01-01T16:23:55.039Z",
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "status": "DONE"
}
```

<br>

## Retrieving details

Call the RESTful API GET endpoint below to retrieve document, transaction, and verification details by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>/data`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>/data`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/scans/<scanReference>/data`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**timestamp** |string| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference** |string|36|Jumio’s reference number for the transaction|
|**document**| object| |As listed in the section [Retrieving document data only](#retrieving-document-data-only) <br> without timestamp and scanReference|
|**transaction**|object| |As listed in the section [Retrieving transaction data only](#retrieving-transaction-data-only) <br> without timestamp and scanReference|
|verification|object| |As listed in the section [Retrieving verification data only](#retrieving-verification-data-only) <br> without timestamp and scanReference|
|facemap|object| |facemap (if download of facemap is enabled), see table below|

<br>

#### Parameter `facemap`

**Required items appear in bold type.**

|Name| Type   | Max. Length    | Description|
|:------------------------|:--------|:--------|:------------|
|**classifier** |string| 5  |Possible value:<br/>•	facemap |
|**href** |string| 5  |REST URL which can be used to retrieve facemap content |


##### Retrieving facemap

Call the RESTful API GET endpoint to retrieve a raw facemap which will be returned in a binary format.

Use HTTP **GET** with **Basic Authorization** using your API token and secret, as userid and password.<br>
**Header**: The following parameters are mandatory in the header section of your request.<br>
- `User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br /><br />
The value for **User-Agent** must contain a reference to your business or entity for Jumio to be able to identify your requests. (e.g. YourCompanyName YourAppName/1.0.0). Without a proper User-Agent header, Jumio will take longer to diagnose API issues.<br>

The TLS protocol is required during the TLS handshake (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.<br/><br>

###### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available, does not contain a valid facemap or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing requested the information.


## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

```
{
    "timestamp": "2019-03-20T16:25:38.663Z",
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "document": {
        "type": "DRIVING_LICENSE",
        "dob": "1990-01-01",
        "expiry": "2022-12-31",
        "firstName": "FIRST NAME",
        "issuingCountry": "AUT",
        "issuingDate": "2013-01-01",
        "lastName": "LAST NAME",
        "number": "123456789",
        "status": "APPROVED_VERIFIED"
    },
    "transaction": {
        "clientIp": "XX.XX.X.XX",
        "customerId": "CUSTOMERID",
        "date": "2019-01-01T11:01:10.227Z",
        "merchantReportingCriteria": "YOURREPORTINGCRITERIA",
        "merchantScanReference": "YOURSCANREFERENCE",
        "source": "WEB_UPLOAD",
        "status": "DONE"
    },
    "verification": {
        "identityVerification": {
            "similarity": "MATCH",
            "validity": "true"
        },
        "mrzCheck": "NOT_AVAILABLE"
    },
		"facemap": {
        "classifier": "facemap",
        "href": "https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/facemap"
    }
}
```

<br>

## Retrieving document data only

Call the RESTful API GET endpoint below to retrieve document data by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>/data/document`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>/data/document`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/scans/<scanReference>/data/document`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**timestamp**| string| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| string|36|Jumio’s reference number for the transaction|
|**status**| string| |ID Verification:<br>• APPROVED\_VERIFIED<br>• DENIED\_FRAUD<br>• DENIED\_UNSUPPORTED\_ID\_TYPE<br>• DENIED\_UNSUPPORTED\_ID\_COUNTRY<br>• ERROR\_NOT\_READABLE\_ID<br>• NO\_ID\_UPLOADED |
|type| string| |ID Verification:<br>•	PASSPORT<br>• DRIVING\_LICENSE<br>• ID\_CARD<br>• VISA<br>• UNSUPPORTED |
|idSubtype| string|255|Possible subtypes if type = ID\_CARD:<br>•	NATIONAL\_ID<br>• CONSULAR\_ID<br>• ELECTORAL\_ID<br>• RESIDENT\_PERMIT\_ID<br>• TAX\_ID <br>• STUDENT\_ID <br>• PASSPORT\_CARD\_ID <br>• MILITARY\_ID <br>• PUBLIC\_SAFETY\_ID<br/>•	HEALTH\_ID <br>• OTHER\_ID<br>• VISA <br>• UNKNOWN<br><br>Possible subtypes if idType = DRIVING\_LICENSE<br/>•	REGULAR\_DRIVING\_LICENSE<br/>•	LEARNING\_DRIVING\_LICENSE <br/><br/>Possible subtypes if idType = PASSPORT<br/>•	E\_PASSPORT (only for mobile) |
|issuingCountry|string|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>- XKX (Kosovo)|
|firstName|string|255|Customer's first name|
|lastName|string|255|Customer's last name|
|dob|string||Date of birth in the format YYYY-MM-DD|
|expiry|string||Date of expiry in the format YYYY-MM-DD|
|issuingDate|string|10|Issue date in the format YYYY-MM-DD (if issuing date extraction is enabled)|
|number|string|255|Identification number of the document|
|usState|string|255|Possible values:<br/>•	Last two characters of [ISO 3166-2:US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br/>•	Last 2-3 characters of [ISO 3166-2:AU](http://en.wikipedia.org/wiki/ISO_3166-2:AU) state code<br/>•	Last two characters of [ISO 3166-2:CA](http://en.wikipedia.org/wiki/ISO_3166-2:CA) state code<br/>• [ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country name<br/>• XKX (Kosovo)<br/>• Free text - if it can't be mapped to a state/country code<br/>|
|personalNumber|string|255|Personal number of the document|
|optionalData1|string|255|Optional field of MRZ line 1|
|optionalData2|string|255|Optional field of MRZ line 2|
|address|object||Address in RAW format, see table below|
|issuingAuthority|string|50|Issuing authority of the document (if issuing authority extraction is enabled)|
|issuingPlace|string|50|Issuing place of the document (if issuing place extraction is enabled)|
|curp|string|18|CURP (if CURP extraction is enabled)|
|gender|string|1|Possible values: M, F|
|nationality|string|3|Nationality of the document holder in [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code (if extraction is enabled)|
|placeOfBirth|string|255|Place of birth of document holder|
|taxNumber|string|255|Tax number of the document<br>• if idCountry = ITA and idType = HEALTH\_ID or TAX\_ID <br> (if Tax number extraction is enabled)|
|cpf|string|255|CPF number of the document<br> (if CPF number extraction is enabled)|
|registrationNumber|string|255|Registration number of the document <br> (if Registration number extraction is enabled)|
|mothersName|string|255|Name of the document holder's mother <br> (if extraction is enabled)|
|fathersName|string|255|Name of the document holder's father <br> (if extraction is enabled)|
|personalIdentificationNumber|string|255|National personal identification number of the document <br> (if extraction is enabled) |
|rgNumber|string|255|"General Registration" number of Brazilian documents <br> (if extraction is enabled) |
|voterIdNumber|string|255|"Clave de elector" number for idCountry = MEX <br> (if extraction is enabled)|
|issuingNumber|string|255|"Numero de emission" number for idCountry = MEX <br> (if extraction is enabled)|


#### RAW address format

|Name      | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|line1| string|255 |Line item1|
|line2| string|255 |Line item2|
|line3| string|255 |Line item3|
|line4| string|255 |Line item4|
|line5| string|255 |Line item5|
|country|string|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• XKX (Kosovo)|
|postalCode|string|255|Postal code|
|city|string|255|City|
|formattedAddress|string||Complete address in a formatted way|

Note: On 2020-08-15 we removed old address formats (EU, US). Retrieving document data for transactions older than 2020-08-15 which were initially returned in EU or US format will be now returned in RAW format.

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/document HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

```
{
    "type": "DRIVING_LICENSE",
    "dob": "1990-01-01",
    "expiry": "2022-12-31",
    "firstName": "FIRST NAME",
    "issuingCountry": "AUT",
    "gender": "M",
    "issuingDate": "2013-01-01",
    "lastName": "LAST NAME",
    "nationality": "BHR",
    "personalNumber": "12345679",
    "number": "123456789",
    "status": "APPROVED_VERIFIED",
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-03-01T11:44:56.741Z"
}
```

<br>

## Retrieving transaction data only

Call the RESTful API GET endpoint below to retrieve transaction metadata by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>/data/transaction`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>/data/transaction`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/scans/<scanReference>/data/transaction`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**timestamp**| string| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| string|36 |Jumio’s reference number for the transaction|
|**status**| string| |Possible states:<br>• PENDING<br>• DONE<br>• FAILED|
|**source**| string| |ID Verification Web embedded:<br>• WEB<br>• WEB-CAM<br>• WEB-UPLOAD<br><br>ID Verification Web redirect:<br>• REDIRECT<br>• REDIRECT-CAM<br>• REDIRECT-UPLOAD<br><br>ID Verification API:<br>• API<br><br>ID Verification Mobile:<br>• SDK |
|**date**| string| |Timestamp of the scan creation in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|clientIp| string| |IP address of the client in the format xxx.xxx.xxx.xxx|
|customerId| string|255 |Your internal reference for the user.|
|merchantScanReference| string|255 |Your internal reference for each transaction|
|merchantReportingCriteria| string|255 |Your internal reporting criteria for each transaction|

<br>

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/transaction HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

```
{
    "clientIp": "xxx.xxx.xxx.xxx",
    "customerId": "CUSTOMERID",
    "date": "2019-02-01T11:01:10.227Z",
    "merchantReportingCriteria": "YOURMERCHANTREPORTINGCRITERIA",
    "merchantScanReference": "YOURSCANREFERENCE",
    "source": "WEB_UPLOAD",
    "status": "DONE",
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-03-01T11:46:57.127Z"
}
```

<br>

## Retrieving verification data only

Call the RESTful API GET endpoint below to retrieve verification data by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>/data/verification`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>/data/verification`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/scans/<scanReference>/data/verification`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**timestamp**| string| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| string|36 |Jumio’s reference number for the transaction|
|mrzCheck|string||Possible values:<br>• OK<br>• NOT\_AVAILABLE|
|rejectReason|object||Reject reason, see tables below|
|identityVerification|object||Identity verification, see table below|
|additionalChecks|object||Additional checks, see table below|

<!-- |additionalChecks|object||Additional checks, see watchlistScreening guide|-->

<br>

#### Parameter `rejectReason`

|Name| Type   | Max. Length    | Description|
|:------------------------|:--------|:--------|:------------|
|rejectReasonCode |string| 5  |see below |
|rejectReasonDescription |string |255  |Possible codes and descriptions for verification status DENIED\_FRAUD:<br>100	MANIPULATED\_DOCUMENT<br/>105	FRAUDSTER<br/>106	FAKE<br/>107	PHOTO\_MISMATCH<br/>108	MRZ\_CHECK\_FAILED<br/>109	PUNCHED\_DOCUMENT<br/>110	CHIP\_DATA\_MANIPULATED (only available for ePassport)<br/>111	MISMATCH\_PRINTED\_BARCODE\_DATA<br/>113	MISMATCHING\_DATA\_REPEATED\_FACE<sup>1</sup><br/>115 MISMATCH\_HRZ\_MRZ\_DATA<br/>116 SUPERIMPOSED\_TEXT<br/>118 MISMATCH\_FRONT\_BACK<br><br>Possible codes and descriptions for verificationStatus = ERROR\_NOT\_READABLE\_ID:<br/>102	PHOTOCOPY\_BLACK\_WHITE<br/>103	PHOTOCOPY\_COLOR<br/>104	DIGITAL\_COPY<br/>200	NOT\_READABLE\_DOCUMENT<br/>201	NO\_DOCUMENT<br/>202	SAMPLE\_DOCUMENT<br/>206	MISSING\_BACK<br/>207	WRONG\_DOCUMENT\_PAGE<br/>209	MISSING\_SIGNATURE<br/>210	CAMERA\_BLACK\_WHITE<br/>211	DIFFERENT\_PERSONS\_SHOWN<br/>213 INVALID\_WATERMARK<br/>214	MISSING\_FRONT<br/>300	MANUAL\_REJECTION|
|rejectReasonDetails |JSON object / JSON array  |   |Reject reason details as JSON object (if only one item is returned) or JSON array (containing JSON objects) if rejectReasonCode = 100 or 200, see table below |

<br>
<sup>1</sup>activation required

#### Parameter `rejectReasonDetails`

|Name |  Type    | Max. Length    | Description|
|:-------------------------------|:---------|:---------------|:------------|
|detailsCode   |string | 5 | see below |
|detailsDescription|string|255| Possible codes and description details for rejectReasonCode = 100:<br/>1001	PHOTO<br/>1002	DOCUMENT\_NUMBER<br>1003	EXPIRY<br/>1004	DOB<br/>1005	NAME<br/>1006	ADDRESS<br/>1007	SECURITY\_CHECKS<br/>1008	SIGNATURE<br>1009	PERSONAL\_NUMBER<br>10011	PLACE\_OF\_BIRTH<br><br>Possible codes and description details for rejectReasonCode = 200:<br/>2001	BLURRED<br/>2002	BAD\_QUALITY<br/>2003	MISSING\_PART\_DOCUMENT<br/>2004	HIDDEN\_PART\_DOCUMENT<br/>2005	DAMAGED\_DOCUMENT<br/>2006 GLARE<br/>2007	MISSING\_MANDATORY\_DATAPOINTS |

<br>

#### Parameter `identityVerification`

|Name      | Description|
|:---------------|:------------|
|similarity |Possible values:<br/>• MATCH<br />• NO\_MATCH<br />• NOT\_POSSIBLE|
|validity  |Possible values:<br/>• TRUE<br />• FALSE |
|reason   |Provided if validity = FALSE<br/>Possible values:<br />• SELFIE\_CROPPED\_FROM\_ID<br />•	ENTIRE\_ID\_USED\_AS\_SELFIE<br />•	MULTIPLE\_PEOPLE<br />•	SELFIE\_IS\_SCREEN\_PAPER\_VIDEO<br />•	SELFIE\_MANIPULATED<br />• AGE\_DIFFERENCE\_TOO\_BIG<br />•	NO\_FACE\_PRESENT<br />•	FACE\_NOT\_FULLY\_VISIBLE<br />• BAD\_QUALITY<br />• BLACK\_AND\_WHITE<br />• LIVENESS\_FAILED|


<!-- |similarityDecision   |Only visible if setting is turned on within your account. For questions about this feature, please contact Jumio Support. <br/><br/>Possible values:<br/> •	MANUAL<br />•	AUTOMATED|-->
<!-- |handwrittenNoteMatches	|Only visible if **Liveness Detection (with handwritten note)** setting is turned on within your account. For questions about this feature, please contact Jumio Support. <br/><br/>Possible values:<br/> •	TRUE<br />•	FALSE| -->

<br>

#### Parameter `additionalChecks`


|Name |  Type    | Description|Notes|
|:-------------------------------|:---------|:---------------|:------------|
|addressValidation   | JSON | Result of Address Validation<br>see table below |activation required|
|proofOfResidency | JSON | Result of Proof of Residency check<br>• if idCountry = AUS, BRA, CAN, DEU, ESP, GBR, MEX, <br>see table below |activation required|
|watchlistScreening | JSON | Result of Jumio Screening (see [ID Verification & ComplyAdvantage Screening Implementation Guide](netverify-screening.md)) |activation required|
|faceSearchFindings | JSON | Result of 1:n face search on previous transactions<br>see table below|activation required|
|personalNumberValidation | JSON | Result of validation for CPF number<br>see table below|activation required|

<br>

##### Parameter `addressValidation`

|Name |  Type    | Description|
|:-------------------------------|:---------|:---------------|
|searchResults   | enum | If verificationStatus = APPROVED\_VERIFIED possible values: <br />• POSITIVE\_RESULT<br />• PARTIAL\_RESULT<br />• NEGATIVE\_RESULT<br />else<br>• CHECK\_NOT\_DONE (see `searchResponse`)|
|searchResponse   | enum | Possible values: <br />• UNSUPPORTED\_COUNTRY<br />• NOT\_ENOUGH\_DATA<br />• TECHNICAL\_ERROR |

<br>

##### Parameter `proofOfResidency`

|Name |  Type    | Description|
|:-------------------------------|:---------|:---------------|
|searchResults   | enum | If verificationStatus = APPROVED\_VERIFIED possible values: <br />• POSITIVE\_RESULT<br />• PARTIAL\_RESULT<br />• NEGATIVE\_RESULT<br />else<br>• CHECK\_NOT\_DONE (see `searchResponse`)|
|searchResponse   | enum | Possible values: <br />• UNSUPPORTED\_COUNTRY<br />• NOT\_ENOUGH\_DATA<br />• TECHNICAL\_ERROR |

<br>

##### Parameter `faceSearchFindings`

|Name |  Type    | Description|
|:-------------------------------|:---------|:---------------|
|status   | enum | Possible values: <br />• DONE<br />• PENDING<br />• ERROR |
|findings   | String/JSON array |A face (on the ID or Selfie) is matching with another face (on the ID or Selfie) from a previous transaction<br><br>Possible values:<br />• No findings: parameter not returned<br />• Single finding: String of the transaction reference<br />• Multiple findings: Array of transaction references |

See [1:n Best Practice](https://support.jumio.com/hc/en-us/articles/1260803713790) for further details.

##### Parameter `personalNumberValidation`

|Name |  Type    | Description|
|:-------------------------------|:---------|:---------------|
|searchResults   | enum | If verificationStatus = APPROVED\_VERIFIED possible values: <br />• POSITIVE\_RESULT<br />• NEGATIVE\_RESULT<br />else<br>• CHECK\_NOT\_DONE (see `searchReason`)|
|searchReason   | enum |Possible values: <br />• NOT\_ENOUGH\_DATA<br />• TECHNICAL\_ERROR |

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/verification HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

#### Approved verified

```
{
	"additionalChecks": {
		"addressValidation": {
			"searchResults": "POSITIVE_RESULT"
		}
		"proofOfResidency": {
			"searchResults": "PARTIAL_RESULT"
		}
		"faceSearchFindings": {
			"status": "DONE",
			"findings": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx","xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"]
		}
		"personalNumberValidation": {
      	"searchResults": "POSITIVE_RESULT"
    	}
	},
	"identityVerification": {
        "similarity": "MATCH",
        "validity": "true"
    },
    "mrzCheck": "OK",
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-03-01T11:48:31.331Z"
}
```

#### Error (not readable ID)

```
{
    "mrzCheck": "OK",
    "rejectReason": {
        "rejectReasonCode": "201",
        "rejectReasonDescription": "NO_DOCUMENT"
    },
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-03-01T11:51:39.876Z"
}
```

<br>

## Retrieving available images

Call the RESTful API GET endpoint below to retrieve a list of the available images for a transaction by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>/images`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>/images`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/scans/<scanReference>/images`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.


**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| string| |Timestamp of the response in the format <br>YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference** | string|36 | |Jumio’s reference number for the transaction|
|images|JSON array/object||See table below|
|livenessImages| JSON array | | List of REST URLs to retrieve specific liveness images|

<br>

#### Parameter `images`
**Required items appear in bold type.**

|Name| Type    | Description|
|:---------------|:--------|:------------|
|**classifier**| string| ID Verification:<br/>•	front<br/>•	face<br/>•	back|
|**href** | string |REST URL to retrieve specific image (see [Retrieving a specific image](#retrieving-a-specific-image) section)|

<br>

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx

```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

```
{
    "timestamp": "2019-03-01T11:53:52.878Z",
    "images": [
        {
            "classifier": "back",
            "href": "https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/back"
        },
        {
            "classifier": "front",
            "href": "https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/front"
        },
        {
            "classifier": "face",
            "href": "https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/face"
        }
    ],
		"livenessImages": [
	        	"https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/7",
        		"https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/5",
        		"https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/6",
        		"https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/3",
        		"https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/4",
		        "https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/1",
        		"https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/2"
    ],
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

<br>

## Retrieving a specific image

Call the RESTful GET API endpoint below to retrieve a specific image from a transaction by specifying the image classifier and the Jumio transaction reference (scan reference) of an existing transaction from your account as path parameters.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>/images/<classifier>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>/images/<classifier>`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/scans/<scanReference>/images/<classifier>`<br>


### Request path parameters

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**scanReference**| string|36|Jumio’s reference number for the transaction|
|**classifier**| string| |ID Verification:<br/>•	front<br/>•	face<br/>•	back|

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available, has been deleted, or does not contain the image you requested.

Successful requests will return HTTP status code `200 OK` along with a JPG or PNG image and the appropriate header (e.g. `Content-Type: image/jpeg`).

<br>

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/front HTTP/1.1
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

## Retrieving liveness images

Call the RESTful API GET endpoint below to retrieve liveness images from a transaction by specifying the liveness number and the Jumio transaction reference (scan reference) of an existing transaction from your account as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>/images/liveness/<livenessNumber>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>/images/liveness/<livenessNumber>`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/scans/<scanReference>/images/liveness/<livenessNumber>`<br>


### Request path parameters

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**scanReference**| string|36|Jumio’s reference number for the transaction|
|**livenessNumber**| integer| |Number of image from the liveness sequence|

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JPG or PNG image and the appropriate header (e.g. `Content-Type: image/jpeg`).

<br>

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/1 HTTP/1.1
Accept: image/jpeg
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

---

# Authentication retrieval


## Retrieving details

Call the RESTful API GET endpoint below to retrieve the details of an Authentication transaction by specifying the Jumio transaction reference of an existing Authentication transaction from your account as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/authentications/<transactionReference>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/authentications/<transactionReference>`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/authentications/<transactionReference>`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**
<!---
|**timestamp** | string||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
-->

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**transactionReference** |string|36|Jumio’s reference number for the Authentication transaction|
|**enrollmentTransactionReference** |string|36|Jumio’s reference number of the enrollment transaction (ID)|
|**transactionResult** |string||Possible values:<br>• CREATED<br>• STARTED<br>• PASSED<br>• FAILED<br>• INVALID<br>• EXPIRED|
|**transactionDate** |string||Timestamp of the transaction in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanSource** |string||Possible value:<br>• SDK<br>• WEB|
|userReference |string||Your internal reference for the user|
|images |JSON array/object||See table below|
|livenessImages |JSON array||List of REST URLs to retrieve specific liveness images|

<br>

#### Parameter `images`
**Required items appear in bold type.**

|Name| Type    | Description|
|:---------------|:--------|:------------|
|**classifier**| string|Possible value:<br>• face |
|**href** | string |REST URL to retrieve specific image (see [Retrieving a specific image](#retrieving-a-specific-image-1))|

<br>

## Examples
### Sample request
```
GET https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response
<!---
timestamp will be added with the next release to be consistent with the other API calls

"timestamp": "2019-06-01T16:10:42.368Z",
-->
```
{
	"transactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
	"enrollmentTransactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
	"transactionResult": "PASSED",
    "scanSource": "SDK",
    "transactionDate": "2019-05-23T14:32:27.534",
    "images": [
        {
            "classifier": "face",
            "href": "https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/face"
        }
    ],
    "livenessImages": [
        "https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/1",
        "https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/2",
        "https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/3",
        "https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/4",
        "https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/5",
        "https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/6",
        "https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/7"
    ]
}
```
<br>

## Retrieving a specific image

Call the RESTful API GET endpoint below to retrieve the face image from an Authentication transaction by specifying the Jumio transaction reference of an existing Authentication transaction from your account as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/authentications/<transactionReference>/images/<classifier>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/authentications/<transactionReference>/images/<classifier>`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/authentications/<transactionReference>/images/<classifier>`<br>


### Request path parameters

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**transactionReference**| string|36|Jumio’s reference number for the Authentication transaction|
|**classifier**| string| |Possible value:<br/>•	face|

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JPG or PNG image and the appropriate header (e.g. `Content-Type: image/jpeg`).

<br>

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/face HTTP/1.1
Accept: image/jpeg
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

<br>

## Retrieving liveness images

Call the RESTful API GET endpoint below to retrieve liveness images from an Authentication transaction by specifying the liveness number and the Jumio transaction reference of an existing Authentication transaction from your account as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/authentications/<transactionReference>/images/liveness/<livenessNumber>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/authentications/<transactionReference>/images/liveness/<livenessNumber>`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/authentications/<transactionReference>/images/liveness/<livenessNumber>`<br>


### Request path parameters

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**transactionReference**| string|36|Jumio’s reference number for the Authentication transaction|
|**livenessNumber**| integer| |Number of image from the liveness sequence|

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JPG or PNG image and the appropriate header (e.g. `Content-Type: image/jpeg`).

<br>

## Examples
### Sample request

```
GET https://netverify.com/api/netverify/v2/authentications/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/liveness/1 HTTP/1.1
Accept: image/jpeg
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

---

# Document Verification retrieval


## Retrieving status

Call the RESTful API GET endpoint below to retrieve the status of a Document Verification transaction by specifying the Jumio transaction reference (scan reference) of an existing transaction from your account as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>`<br>
**REST URL (EU)**: `https://retrieval.lon.netverify.com/api/netverify/v2/documents/<scanReference>`<br>
**REST URL (SGP):** `https://retrieval.core-sgp.jumio.com/api/netverify/v2/documents/<scanReference>`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**timestamp** | string||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference** |string|36|Jumio’s reference number for the transaction|
|**status** |string||Possible states:<br>• DONE<br>• FAILED|

<br>

## Examples
### Sample request
```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response
```
{
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-01-01T15:52:48.864Z",
    "status": "DONE"
}
```
<br>

## Retrieving details

Call the RESTful API GET endpoint below to retrieve the document and transaction details of a Document Verification transaction by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/data`<br>
**REST URL (EU)**: `https://retrieval.lon.netverify.com/api/netverify/v2/documents/<scanReference>/data`<br>
**REST URL (SGP):** `https://retrieval.core-sgp.jumio.com/api/netverify/v2/documents/<scanReference>/data`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**|string||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**|string|36|Jumio’s reference number for the transaction|
|document|object||Same parameters as listed in the section [Retrieving document data only](#multi-retrieving-document-data-only) but without timestamp and scanReference|
|transaction|object||Same parameters as listed in the section [Retrieving transaction data only](#multi-retrieving-transaction-data-only) but without timestamp and scanReference|

<br>

## Examples
### Sample request
```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

#### Credit card (CC)
```
{
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-01-01T15:42:31.858Z",
    "document": {
        "status": "EXTRACTED",
        "type": "CC",
        "extractedData": {
            "firstName": "FULL NAME",
            "signatureAvailable": false,
            "extractedData": "{\"expiryDate\":\"01/22\",\"firstName\":\"FULL NAME\",\"pan\":\"XXXXXXXXXXXX1234\"
            }"
        },
        "country": "AUT"
    },
    "transaction": {
        "status": "DONE",
        "merchantScanReference": "YOURSCANREFERENCE",
        "customerId": "CUSTOMERID",
        "source": "DOC_UPLOAD"
    }
}
```

#### Social Security card (SSC)
```
{
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-01-01T15:40:49.124Z",
    "document": {
        "status": "EXTRACTED",
        "type": "SSC",
        "extractedData": {
            "firstName": "FIRST NAME",
            "lastName": "LAST NAME",
            "ssn": "123-45-6789",
            "signatureAvailable": false,
            "extractedData": "{\"lastName\":\"LAST NAME\",\"firstName\":\"FIRST NAME\",\"signaturePresent\":\"true\",\"ssn\":\"123-45-6789\"
            }"
        },
        "country": "USA"
    },
    "transaction": {
        "status": "DONE",
        "merchantScanReference": "YOURSCANREFERENCE",
        "customerId": "CUSTOMERID",
        "source": "DOC_UPLOAD"
    }
}
```

#### Bank statement (BS)
```
{
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-01-01T15:32:22.606Z",
    "document": {
        "status": "EXTRACTED",
        "type": "BS",
        "extractedData": {
            "firstName": "FULL NAME",
            "signatureAvailable": false,
            "extractedData": "{\"city\":\"CITY\",\"addressFormat\":\"RAW\",\"postalCode\":\"12345\",\"accountNumber\":\"123456789\",\"subdivision\":\"XX\",\"firstName\":\"FULL NAME\",\"issueDate\":\"2019-01-01\",\"line1\":\"12345 EASY STREET\"}"
        },
        "country": "USA",
        "originalDocument": "https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/original"
    },
    "transaction": {
        "status": "DONE",
        "merchantScanReference": "YOURSCANREFERENCE",
        "customerId": "CUSTOMERID",
        "source": "DOC_UPLOAD"
    }
}
```

<br>

## Retrieving document data only

Call the RESTful API GET endpoint below to retrieve the document data of a Document Verification transaction by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/data/document`<br>
**REST URL (EU)**: `https://retrieval.lon.netverify.com/api/netverify/v2/documents/<scanReference>/data/document`<br>
**REST URL (SGP):** `https://retrieval.core-sgp.jumio.com/api/netverify/v2/documents/<scanReference>/data/document`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**timestamp** |string||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| string|36|Jumio’s reference number for the transaction|
|**status**|string||Possible states:<br>•	UPLOADED (default)<br>•	EXTRACTED if the document provided is supported for data extraction<br>•	DISCARDED if the document provided is not supported for data extraction|
|type|string||Possible types:<br>• CC (Credit card, front and back side)<br>• BS (Bank statement, front side)<br>•	IC (Insurance card, front side)<br>•	UB (Utility bill, front side)<br>•	CAAP (Cash advance application, front and back side)<br>•	CRC (Corporate resolution certificate, front and back side)<br>•	CCS (Credit card statement, front and back side)<br>•	LAG (Lease agreement, front and back side)<br>•	LOAP (Loan application, front and back side)<br>•	MOAP (Mortgage application, front and back side)<br>•	TR (Tax return, front and back side)<br>•	VT (Vehicle title, front side)<br>•	VC (Voided check, front side)<br>•	STUC (Student card, front side)<br>•	HCC (Health care card, front side)<br>•	CB (Council bill, front side)<br>•	SENC (Seniors card, front side)<br>•	MEDC (Medicare card, front side)<br>•	BC (Birth certificate, front side)<br>•	WWCC (Working with children check, front side)<br>•	SS (Superannuation statement, front side)<br>•	TAC (Trade association card, front side)<br>•	SEL (School enrolment letter, front side)<br>•	PB (Phone bill, front side)<br>•	SSC (Social security card, front side)<br>•	CUSTOM (Custom document type)<br>•	OTHER (Other document type)|
|country|string|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|
|originalDocument|string|255|URL to retrieve the original document (PDF), if available |
|customDocumentCode|string|100|Your custom document code (configured in the [Customer Portal](/netverify/portal-settings.md#multi-documents)) if type = CUSTOM|
|extractedData|object||Extracted data when the document provided is supported for data extraction, see table below|

<br>

#### `extractedData` object
|Name| Type    | Max. Length| Description|
|:-------------------------|:--------|:------------|:------------|
|signatureAvailable |Boolean ||true if signature available, otherwise false|
|ssn |string |255|Social security number (if readable)|
|firstName |string |255|First name if readable for SSC type, full name if readable for all other types|
|lastName |string |255|Last name if readable (SSC type only).|
|extractedData |object||Extracted data when the document provided is supported for data extraction, see table below|

<br>

#### Nested `extractedData` object
|Name| Type    | Max. Length| Description|
|:-------------------------|:--------|:------------|:------------|
|addressFormat |string |255|RAW|  
|line1 |string |100 |Address line 1 |
|line2 |string |100 |Address line 2 |
|countryCode |string |3|Possible countries of residence: <br />• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br />• XKX (Kosovo)|
|postalCode |string |15 |Postal code|
|city |string |64 |City |
|subdivision |string |50 |Name of subdivision (US state is returned in this field) |
|firstName |string |255|First name if readable for SSC type, full name if readable for all other types|
|lastName |string |255|Last name if readable (SSC type only)|
|ssn |string |255|Social security number if readable (SSC type only)|
|signaturePresent |string |255|"true" if signature is present, otherwise "false"|
|accountNumber |string | |IBAN or account number|
|swiftCode |string |20|BIC/SWIFT code|
|issueDate  | string  |  |Issue date in the format YYYY-MM-DD|
|expiryDate |string | |Date of expiry in the format MM/YY (CC type only)|
|pan |string |20 |Personal account number of credit card (CC type only)|

<br>

## Examples
### Sample request
```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/document HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

#### Credit card (CC)
```
{
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-01-01T15:02:43.691Z",
    "status": "EXTRACTED",
    "type": "CC",
    "extractedData": {
        "firstName": "FULL NAME",
        "signatureAvailable": false,
        "extractedData": "{\"expiryDate\":\"01/22\",\"firstName\":\"FULL NAME\",\"pan\":\"XXXXXXXXXXXX1234\"}"
    },
    "country": "AUT"
}
```

#### Social Security card (SSC)
```
{
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-01-01T15:05:21.904Z",
    "status": "EXTRACTED",
    "type": "SSC",
    "extractedData": {
        "firstName": "FIRST NAME",
        "lastName": "LAST NAME",
        "ssn": "123-45-6789",
        "signatureAvailable": false,
        "extractedData": "{\"lastName\":\"LAST NAME\",\"firstName\":\"FIRST NAME\",\"signaturePresent\":\"true\",\"ssn\":\"123-45-6789\"}"
    },
    "country": "USA"
}
```

#### Bank statement (BS)
```
{
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-01-01T15:07:25.098Z",
    "status": "EXTRACTED",
    "type": "BS",
    "extractedData": {
        "firstName": "FULL NAME",
        "signatureAvailable": false,
        "extractedData": "{\"city\":\"CITY\",\"addressFormat\":\"RAW\",\"postalCode\":\"12345\",\"accountNumber\":\"123456789\",\"subdivision\":\"XX\",\"firstName\":\"FULL NAME\",\"issueDate\":\"2019-01-01\",\"line1\":\"12345 EASY STREET\"}"
    },
    "country": "USA",
    "originalDocument": "https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/original"
}
```

<br>

## Retrieving transaction data only

Call the RESTful API GET endpoint below to retrieve the transaction metadata of a Document Verification transaction by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/data/transaction`<br>
**REST URL (EU)**: `https://retrieval.lon.netverify.com/api/netverify/v2/documents/<scanReference>/data/transaction`<br>
**REST URL (SGP):** `https://retrieval.core-sgp.jumio.com/api/netverify/v2/documents/<scanReference>/data/transaction`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| string||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**|string|36|Jumio’s reference number for each transaction|
|**status**|string||Possible states:<br>•	DONE<br>•	FAILED |
|**source**|string||Possible status:<br>•	DOC\_API<br>• DOC\_UPLOAD |
|merchantReportingCriteria|string|255|Your internal reporting criteria for each transaction|
|merchantScanReference|string|255|Your internal reference for each transaction|
|customerId|string|255|Your internal reference for the user.|

<br>

## Examples
### Sample request
```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/transaction HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response
```
{
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "timestamp": "2019-01-01T16:02:25.646Z",
    "status": "DONE",
    "merchantScanReference": "YOURSCANREFERENCE",
    "customerId": "CUSTOMERID",
    "source": "DOC_UPLOAD"
}
```

<br>

## Retrieving available images

Call the RESTful API GET endpoint below to retrieve a list of the available images for a Document Verification transaction by specifying the Jumio transaction reference (scan reference) as a path parameter.

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/pages`<br>
**REST URL (EU)**: `https://retrieval.lon.netverify.com/api/netverify/v2/documents/<scanReference>/pages`<br>
**REST URL (SGP):** `https://retrieval.core-sgp.jumio.com/api/netverify/v2/documents/<scanReference>/pages`<br>

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available or has been deleted.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| string| |Timestamp of the response in the format <br>YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| string|36 |Jumio’s reference number for the transaction|
|images|JSON array/object||See table below|

<br>

#### Parameter `images`
**Required items appear in bold type.**

|Name| Type    | Description|
|:---------------|:--------|:------------|
|**classifier**| integer|Page number specifed when uploading the image|
|**href** | string |REST URL to retrieve specific image (see [Retrieving a specific image section](#retrieving-a-specific-image-2))|

<br>

## Examples
### Sample request

```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

#### Bank statement (BS)
```
{
    "timestamp": "2019-01-01T16:04:07.751Z",
    "images": [
        {
            "classifier": 1,
            "href": "https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages/1"
        }
        {
            "classifier": 2,
            "href": "https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages/2"
        }
    ],
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

#### Credit card (CC)

```
{
    "timestamp": "2019-01-01T16:10:42.368Z",
    "images": [
        {
            "classifier": 101,
            "href": "https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages/101"
        }
    ],
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

<br>

## Retrieving a specific image

Call the RESTful HTTP GET API below to receive a specific image from a Document Verification transaction by specifying the Jumio scan reference as a path parameter.

To retrieve an unmasked credit card image, append the query parameter `maskhint=unmasked`. By default, retrieval of unmasked credit card images is disabled, and will return HTTP status code `403 Forbidden`. If you want to enable it please contact Jumio Support. Retrieving unmasked credit card images might impose additional security requirements on your systems depending on whether you already store/transmit/process credit card data on your systems.

If you are unsure about the PCI DSS ramifications of retrieving unmasked credit card images, please refer to [Information Supplement: PCI DSS E-commerce Guidelines, version 2.0, January 2013](https://www.pcisecuritystandards.org/pdfs/PCI_DSS_v2_eCommerce_Guidelines.pdf), and/or contact your acquirer or a PCI DSS QSA (Qualified Security Assessor).

**HTTP request method:** `GET`<br>
**REST URL (US)**: `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/pages/<page_number>`<br>
**REST URL (EU)**: `https://retrieval.lon.netverify.com/api/netverify/v2/documents/<scanReference>/pages/<page_number>`<br>
**REST URL (SGP):** `https://retrieval.core-sgp.jumio.com/api/netverify/v2/documents/<scanReference>/pages/<page_number>`<br>

### Request path parameters

**Required items appear in bold type.**

|Name     | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** | string|36|Jumio’s reference number for the transaction|
|**page_number** | string|36|Page number specified when uploading the image|
|maskhint| string| |For credit cards:<br/>•	masked (default)<br/>•	unmasked|

<br>

### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error. HTTP status code `404 Not Found` will be returned if the transaction is not available, has been deleted, or does not contain the image you requested.

Successful requests will return HTTP status code `200 OK` along with a JPG or PNG image and the appropriate header (e.g. `Content-Type: image/jpeg`).

<br>

## Examples
### Sample request

```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages/1 HTTP/1.1
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>


---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
