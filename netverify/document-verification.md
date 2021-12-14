![Jumio](/images/Jumio-Doc-Verification-Banner.png)

# Document Verification implementation guide

This is a reference manual and configuration guide for the Document Verification product. It illustrates how to use both the web client (standalone or embedded into your web page) for Document Verification and the API-only implementation.

### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).

## Table of contents
- [Global ID Verification settings](#global-id-verification-settings)
- [Authentication and encryption](#authentication-and-encryption)
- [Using the Document Verification web client](#using-the-document-verification-web-client)
    - [Initiating the transaction](#initiating-the-transaction)
    - [Displaying the Document Verification client](#displaying-your-document-verification-client)
    - [After the user journey](#redirecting-the-customer-after-the-user-journey)
- [Using the Document Verification API](#using-the-document-verification-api)
    - [Multi page upload: Initiating the transaction](#multi-page-upload-initiating-the-transaction)
    - [Multi-page upload: adding a page](#multi-page-upload-adding-a-page)
    - [Multi-page upload: adding a document](#multi-page-upload-adding-a-document)
    - [Multi page upload: finalizing the transaction](#multi-page-upload-finalizing-the-transaction)
    - [Single page upload](#single-page-upload)
- [Supported documents](#supported-documents)
- [Deleting transactions](#deleting-transactions)
- [Supported cipher suites](#supported-cipher-suites)


----

# Global ID Verification Settings

Configuration settings are located in the Jumio Customer Portal. The description of each of the settings are available via the link below.

[View the Customer Portal settings guide](/netverify/portal-settings.md)

# Authentication and encryption

All Document Verification API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [supported cipher suites](/netverify/supported-cipher-suites.md).

<br>


# Using the Document Verification web client

The Document Verification web client offers document upload and data extraction (see [supported documents](#supported-documents)) with a Jumio-hosted user interface. Use the RESTful API as described below to initiate a transaction and share the resulting URL securely with your user or embed it in an iFrame on your website. You can receive a callback when the transaction is complete containing extracted data (if applicable) and a link to the image uploaded by your user (see [Callback](/netverify/callback.md)).

Uploads are restricted to JPEG, PNG or PDF file types totaling of 10MB in size. Credit card uploads are limited to 2 images or PDF pages, and all other document types are limited to 30 images or PDF pages.

<br>


## Initiating the transaction

Call the RESTful API POST endpoint **/acquisitions** with a JSON object containing the properties described below to create a transaction for each user. You will receive a JSON object in the response containing a Jumio scan reference, and a URL which you can use to present the Document Verification web client to your user.

**HTTP request method:** `POST`<br>
**REST URL (US):** `https://upload.netverify.com/api/netverify/v2/acquisitions`<br>
**REST URL (EU):** `https://upload.lon.netverify.com/api/netverify/v2/acquisitions`<br>
**REST URL (SGP):** `https://upload.core-sgp.jumio.com/api/netverify/v2/acquisitions`<br>

<br>


## Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: application/json`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>


## Request body
The body of your **acquisitions** API request allows you to

- provide your own internal tracking information for the user and transaction.
- specify what type of document is being submitted (including [custom document types](/netverify/portal-settings.md#multi-documents) defined in the Customer Portal)
- indicate where the user should be directed after the user journey.
- set data extraction preferences.
- define the valid lifetime of the `clientRedirectUrl`.
- customize the header logo and colors to match your branding.
- localize the display language.
<br>

|⚠️ Credit cards and Credit card statements uploaded with incorrect `type` may pose a risk to your customers and your business!
|:----------|
|Jumio applies appropriate security controls to credit cards correctly uploaded as the `CC` document type, which was designed with sensitive credit card data in mind.<br><br>Submission of credit card data with any other **non-`CC`** document type is **not supported**. Any such transactions may present a risk to your business and are subject to deletion.<br><br>The same applies to Credit card statements, which are not uploaded as the `CCS ` document type.|

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

**Required items appear in bold type.**  

|Name               |Type   | Max. length|Description                                                                                                  |
|:---                      |:---    |:---        |:---                                                                                                          |
|**type**|string||Possible values:<br>See [supported documents](#supported-documents).|
|**country**<sup>1</sup>|string|3|Possible values:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|
|**merchantScanReference**<sup>2</sup>|string |100       |Your internal reference for the transaction.                                                               |
|**customerId**<sup>2</sup>           |string |100        |Your internal reference for the user.                                                                   |
|merchantReportingCriteria        |string |100        |Your reporting criteria for the transaction.                                                                      |
|successUrl<sup>3</sup>               |string |255        |Redirects to this URL after a successful transaction.<br>Overrides [Success URL](/netverify/portal-settings.md#callback-error-and-success-urls) in the Customer Portal.|
|errorUrl<sup>3</sup>                 |string |255        |Redirects to this URL after an unsuccessful transaction.<br>Overrides [Error URL](/netverify/portal-settings.md#callback-error-and-success-urls) in the Customer Portal.|
|callbackUrl<sup>3</sup>              |string |255        |Sends confirmation and any extracted data to this URL upon completion.<br>Overrides [Callback URL](/netverify/portal-settings.md#callback-error-and-success-urls) in the Customer Portal.|
|enableExtraction| Boolean ||Enables or disables data extraction<sup>4</sup> per transaction.<br><br>Possible values:<br>• `true` (extraction performed)<br>• `false` (no extraction performed)<br><br>**Data extraction will be performed by default if this parameter is not passed.**|
|authorizationTokenLifetime|integer|7|Duration of time (in seconds) for which your `clientRedirectUrl` remains valid.<br><br> default: 1800 (30 minutes)<br>maximum: 5184000 (60 days)|
|baseColor|string|6|Hex triplet value for custom main client color. <br>**Must be passed with** `bgColor`.|
|bgColor|string|6|Hex triplet value for custom background client color. <br>**Must be passed with** `baseColor`.|
|headerImageUrl<sup>3</sup>|string|255|URL of your custom header logo.<br><br>Logo must be:<br>•	landscape (16:9 or 4:3) <br>•	min. height of 192 pixels <br>•	size 8-64 KB|
|customDocumentCode|string|100|Your custom document code (see [Multi documents](/netverify/portal-settings.md#multi-documents)).<br>**Mandatory when** `type` = `CUSTOM`|
|locale                   |string |5          |Renders content in the specified language.<br>Overrides [Default locale](/netverify/portal-settings.md#default-locale) in the Customer Portal.<br>See [supported locale values](#supported-locale-values).|

<sup>1</sup> Optional for type `CC`.<br>
<sup>2</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>3</sup> See [URL constraints](#constraints-for-success-error-callback-and-headerimage-urls).<br>
<sup>4</sup> To activate Document Verification data extraction for your account, please contact your Account Manager or Jumio Support.

<br>


### Constraints for Success, Error, Callback, and headerImage URLs
#### Requirements:

* HTTPS using the [TLS Protocol](https://tools.ietf.org/html/rfc5246) (most recent version recommended)
* Valid [URL](https://tools.ietf.org/html/rfc3986) using [ASCII characters](https://en.wikipedia.org/wiki/ASCII) or [IDNA Punycode](https://tools.ietf.org/html/rfc3492)

#### Restrictions:

* IP addresses, ports, query parameters and fragment identifiers are not allowed.
* Personally identifiable information (PII) is not allowed in any form.

<br>


### Supported `locale` values
Combination of [ISO 639-1:2002 alpha-2](https://en.wikipedia.org/wiki/ISO_639-1) language code plus [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country (where applicable).

|Value  |Locale|
|:--------------|:--------------|
|bg|Bulgarian|
|cs|Czech|
|da|Danish|
|de|German|
|el|Greek|
|en|American English (**default**)|
|en_GB|British English|
|es|Spanish|
|es_MX|Mexican Spanish|
|et|Estonian|
|fi|Finnish|
|fr|French|
|hu|Hungarian|
|it|Italian|
|ja|Japanese|
|ko|Korean|
|lt|Lithuanian|
|nl|Dutch|
|no|Norwegian|
|pl|Polish|
|pt|Portuguese|
|pt_BR|Brazilian Portuguese|
|ro|Romanian|
|ru|Russian|
|sk|Slovak|
|sv|Swedish|
|tr|Turkish|
|vl|Vietnamese|
|zh_CN|Simplified Chinese|
|zh_HK|Traditional Chinese|

<br>


### Sample request — basic
```
POST https://upload.netverify.com/api/netverify/v2/acquisitions HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{
  "type": "BS",
  "country": "USA",
  "merchantScanReference": "transaction_1234",
  "customerId": "user_1234"
}
```
<br>

### Sample request — custom document type, colors, logo
```
POST https://upload.netverify.com/api/netverify/v2/acquisitions HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{
  "type": "CUSTOM",
  "country": "USA",
  "merchantScanReference": "transaction_1234",
  "customerId": "user_1234",
  "merchantReportingCriteria" : "myReport1234",
  "successUrl" : "https://www.yourcompany.com/success",
  "errorUrl" : "https://www.yourcompany.com/error",
  "callbackUrl" : "https://www.yourcompany.com/callback",
  "enableExtraction" : "true",
  "authorizationTokenLifetime" : "3600",
  "baseColor" : "00FF00",
  "bgColor" : "00FFFF",
  "headerImageUrl" : ""https://www.yourcompany.com/logo.png",
  "customDocumentCode" : "27B6"
}
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

<br>


## Response
Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:---------------|:--------|:-----------|:------------|
|**scanReference**|string|36|Jumio reference number for the transaction.|
|**clientRedirectUrl**|string|2000|URL used to load the Document Verification client.|

<br>



### Sample response
```
{
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"clientRedirectUrl": "https://upload.netverify.com/api/netverify/v2/acquisitions?initiationToken=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

<br>




## Displaying the Document Verification web client

To display the Document Verification, place the `clientRedirectUrl` into the `src` attribute of an iFrame tag, specifying an appropriate width and height, or use it to create a link on your web page.

We recommend considering the following dimensions:

-	Min. size: 320 x 480 pixels (or 480 x 320 if landscape on mobile)
-	Max. width of 900 pixels (for iFrame only)

<br>


## Redirecting the customer after the user journey

At the end of the user journey, the user is directed to your **Success URL** if the images they submitted were accepted for processing. If no **Success URL** has been defined, the Jumio default success page will be displayed, including any custom success [image](/netverify/portal-settings.md#images) you have specified in the Customer Portal.

In the event of an error, the user is directed to your , the user is directed to your **Error URL**. If no **Error URL** has been defined, the Jumio default error page will be displayed, including any custom error [image](/netverify/portal-settings.md#images) you have specified in the Customer Portal.

To display relevant information on your success or error page, you can use the following parameters which we append when redirecting to your `successUrl` or `errorUrl` as HTTP `GET` query string parameters<sup>1</sup>. It is also possible to set `successUrl` and `errorUrl` to the same address, by using the query parameter `idScanStatus`.<br>

<br>

**Required items appear in bold type.**

|Name|Description|
|:---|:---|
|**idScanStatus**|Possible values:<br>• `SUCCESS` <br> • `ERROR`|
|**jumioIdScanReference**|Jumio reference number for the transaction.|
|errorCode|Displayed when `transactionStatus` is `ERROR`.<br>Possible values: <br>• `211` (authorization token invalid)<br>• `310` (upload session has expired)<br>|

<sup>1</sup> Because HTTP `GET` parameters can be manipulated on the client side, they may be used for display purposes only.
<br>
<br>


### Sample success redirect
```
https://www.yourcompany.com/success?idScanStatus=SUCCESS&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Sample error redirect
```
https://www.yourcompany.com/error?idScanStatus=ERROR&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&errorCode=320
```

<br>

----

<br>


# Using the Document Verification API

The Document Verification API offers document upload and data extraction (see [supported documents](#supported-documents)) without a Jumio-hosted user interface. Use the RESTful APIs as described below to submit the user's document. You can receive a callback when the transaction is complete containing extracted data (if applicable) and a link to the image uploaded by your user (see [Callback](/netverify/callback.md)).

<br>


## Multi-page upload: initiating the transaction

Call the RESTful API POST endpoint **/acquisitions** with a JSON object containing the properties described below to create a transaction for each user.<br>
**HTTP request method:** `POST`<br>
**REST URL (US):** `https://acquisition.netverify.com/api/netverify/v2/acquisitions`<br>
**REST URL (EU):** `https://acquisition.lon.netverify.com/api/netverify/v2/acquisitions`<br>
**REST URL (SGP):** `https://acquisition.core-sgp.jumio.com/api/netverify/v2/acquisitions`<br>

<br>


### Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: application/json`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>


### Request body

The body of your multi-page **acquisitions** API request allows you to

- provide your own internal tracking information for the user and transaction.
- specify what type of document is being submitted (including [custom document types](/netverify/portal-settings.md#multi-documents) defined in the Customer Portal)
- set data extraction preferences.
<br>

|⚠️ Credit cards uploaded with incorrect `type` may pose a risk to your customers and your business!
|:----------|
|Jumio applies appropriate security controls to credit cards correctly uploaded as the `CC` document type, which was designed with sensitive credit card data in mind.<br><br>Submission of credit card data with any other **non-`CC`** document type is **not supported**. Any such transactions may present a risk to your business and are subject to deletion.|

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

**Required items appear in bold type.**  

|Name               |Type   |Max. length|Description                                                                                                  |
|:---                      |:---    |:---        |:---                                                                                                          
|**type**|string||Possible values:<br>See [supported documents](#supported-documents).|
|**country**<sup>1</sup>|string|3|Possible values:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|
|**merchantScanReference**<sup>2</sup>|string |100       |Your internal reference for the transaction.                                                               
|**customerId**<sup>2</sup>           |string |100        |Your internal reference for the user.                                                                 
|merchantReportingCriteria        |string |255        |Your reporting criteria for the transaction.                                                                      
|callbackUrl<sup>3</sup>              |string |255        |Sends confirmation and any extracted data to this URL upon completion.<br>Overrides [Callback URL](#callback-error-and-success-urls) in the Customer Portal.|
|enableExtraction| Boolean ||Enables or disables data extraction<sup>4</sup> per transaction.<br><br>Possible values:<br>• `true` (extraction performed)<br>• `false` (no extraction performed)<br><br>**Data extraction will be performed by default if this parameter is not passed.**|
|customDocumentCode|string|100|Your custom document code (see [Multi documents](/netverify/portal-settings.md#multi-documents)).<br>**Mandatory when** `type` = `CUSTOM`|
|clientIp|string|100|IP address of the client.|


<sup>1</sup> Optional for type `CC`.<br>
<sup>2</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>3</sup> See [URL constraints](#callback-error-and-success-urls).<br>
<sup>4</sup> To activate data extraction for your account, please contact your Account Manager or Jumio Support.

<br>


#### Sample request
```
POST https://acquisition.netverify.com/api/netverify/v2/acquisitions HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{
  "type": "BS",
  "country": "USA",
  "merchantScanReference": "transaction_1234",
  "customerId": "user_1234"
}
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

<br>


### Response
Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**


|Name      | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**|string||Timestamp of the response in the format *YYYY-MM-DDThh:mm:ss.SSSZ*.|
|**scanReference**|string|36|Jumio reference number for the transaction.|

<br>


#### Sample response
```
{
"timestamp": "2017-10-17T06:37:51.969Z",
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

<br>


## Multi-page upload: adding a page

Call the RESTful API POST endpoint **/pages** with a newly-created or recently updated scan reference and page number to add a JPEG, PNG, or single page PDF file with a maximum size of 10 MB and a maximum image resolution of 8000 x 8000.<br>
|⚠️ Adding a multi-page PDF file to the "adding a page" API call will end up in a HTTP status code `400 Bad Request`. Please use "adding a document" instead.
|:----------|

**HTTP request method:** `POST`<br>
**REST URL (US):** `https://acquisition.netverify.com/api/netverify/v2/acquisitions/<scanReference>/document/pages/<page>`<br>
**REST URL (EU):** `https://acquisition.lon.netverify.com/api/netverify/v2/acquisitions/<scanReference>/document/pages/<page>`<br>
**REST URL (SGP):** `https://acquisition.core-sgp.jumio.com/api/netverify/v2/acquisitions/<scanReference>/document/pages/<page>`<br>

<br>


### Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: multipart/form-data`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>


### Request path parameters

**Required items appear in bold type.**  

|Name               |Type   | Max. length|Description
|:---------------|:--------|:------------|:------------|
|**scanReference**|string|36|Jumio reference number for a transaction initiated/updated less than 5 minutes ago.|
|**page**|string|2 |Page number<br> (max. value 30)|

<br>


### Request body - `multipart/form-data`

**Required items appear in bold type.**  

|Key				|Value				|
|:------			|:------			|
|**Image**		|JPEG, PNG, PDF file (max. size 10 MB and max resolution of 8000 x 8000)|

<br>


#### Sample request
```
POST https://acquisition.netverify.com/api/netverify/v2/acquisitions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/document/pages/1 HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data;boundary=----xxxx
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
----xxxx
// your JPEG, PNG or PDF page
----xxxx
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

<br>


### Response

Unsuccessful requests will return HTTP status code `404 Not Found` if the scan is not available.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**  

|Name      | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**|string||Timestamp of the response in the format *YYYY-MM-DDThh:mm:ss.SSSZ*.|

<br>


#### Sample response
```
{
"timestamp": "2017-10-17T07:56:58.613Z"
}
```

<br>


## Multi-page upload: adding a document
Call the RESTful API POST endpoint **/document** for a newly-created or recently updated scan reference to add a PDF file with a maximum size of 10 MB and maximum of 30 pages.
<br>

**HTTP request method:** `POST`<br>
**REST URL (US):** `https://acquisition.netverify.com/api/netverify/v2/acquisitions/<scanReference>/document`<br>
**REST URL (EU):** `https://acquisition.lon.netverify.com/api/netverify/v2/acquisitions/<scanReference>/document`<br>
**REST URL (SGP):** `https://acquisition.core-sgp.jumio.com/api/netverify/v2/acquisitions/<scanReference>/document`<br>

<br>


### Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: multipart/form-data`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>


### Request path parameters

**Required items appear in bold type.**  

|Name               |Type   | Max. length|Description
|:---------------|:--------|:------------|:------------|
|**scanReference**|string|36|Jumio reference number for a transaction initiated/updated less than 5 minutes ago.|

<br>


### Request body — `multipart/form-data`

**Required items appear in bold type.**  

|Key				|Value				|
|:------			|:------			|
|**Image**		|PDF file with max size of 10 MB and 30 pages|

<br>


#### Sample request

```
POST https://acquisition.netverify.com/api/netverify/v2/acquisitions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/document HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data;boundary=----xxxx
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
----xxxx
// Your PDF document
----xxxx
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

<br>


### Response

Unsuccessful requests will return HTTP status code `404 Not Found` if the scan is not available.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**  

|Name       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**|string||Timestamp of the response in the format *YYYY-MM-DDThh:mm:ss.SSSZ*.|
|**pages**|integer|2|Number of extracted pages (max. 30).|

<br>


#### Sample response
```
{
"timestamp": "2017-10-17T07:56:58.613Z",
"pages": 6
}
```

<br>


## Multi-page upload: finalizing the transaction

Call the RESTful API PUT endpoint **/acquisitions** with a specified scan reference to finalize the transaction.<br>

**HTTP request method:** `PUT`<br>
**REST URL (US):** `https://acquisition.netverify.com/api/netverify/v2/acquisitions/<scanReference>`<br>
**REST URL (EU):** `https://acquisition.lon.netverify.com/api/netverify/v2/acquisitions/<scanReference>`<br>
**REST URL (SGP):** `https://acquisition.core-sgp.jumio.com/api/netverify/v2/acquisitions/<scanReference>`<br>

<br>


### Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>


### Request path parameters
**Required items appear in bold type.**

|Name    | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference**|string|36|Jumio reference number for a transaction initiated/updated less than 5 minutes ago.|

<br>


#### Sample request
```
PUT https://acquisition.netverify.com/api/netverify/v2/acquisitions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

<br>


### Response

Unsuccessful requests will return HTTP status code `404 Not Found` if the scan is not available.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**|string||Timestamp of the response in the format *YYYY-MM-DDThh:mm:ss.SSSZ*.|

<br>


#### Sample response
```
{
"timestamp": "2017-10-17T07:56:58.613Z"
}
```

<br>


## Single-page upload

Call the RESTful API POST endpoint **/complete** with a JSON object containing the properties described below to initialize, upload, and finalize a single-page document in one API call. You can add a JPEG, PNG, or a single-page PDF file with a maximum size of 10 MB and a maximum image resolution of 8000 x 8000.<br>

**HTTP request method:** `POST`<br>
**REST URL (US):** `https://acquisition.netverify.com/api/netverify/v2/acquisitions/complete`<br>
**REST URL (EU):** `https://acquisition.lon.netverify.com/api/netverify/v2/acquisitions/complete`<br>
**REST URL (SGP):** `https://acquisition.core-sgp.jumio.com/api/netverify/v2/acquisitions/complete`<br>

<br>


### Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: multipart/form-data`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>


### Request body — `multipart/form-data`

|Key			|Value				|
|:------		|:------			|
|**image**	|JPEG, PNG, PDF file with max size of 10 MB and max resolution of 8000 x 8000|
|**metadata**	|Details of the user's document specified as a JSON object|

The `metadata` object in the body of your single-page **complete** API request allows you to

- provide your own internal tracking information for the user and transaction.
- specify what type of document is being submitted (including [custom document types](/netverify/portal-settings.md#multi-documents) defined in the Customer Portal)
- set data extraction preferences.
<br>

|⚠️ Credit cards uploaded with incorrect `type` may pose a risk to your customers and your business!
|:----------|
|Jumio applies appropriate security controls to credit cards correctly uploaded as the `CC` document type, which was designed with sensitive credit card data in mind.<br><br>Submission of credit card data with any other **non-`CC`** document type is **not supported**. Any such transactions may present a risk to your business and are subject to deletion.|

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

**Required items appear in bold type.**  

### Request body — `metadata`
|Name               |Type   |Max. length|Description                                                                                                  
|:---                      |:---    |:---        |:---                                                                                                          
|**type**|string||Possible values:<br>See [supported documents](#supported-documents).|
|**country**<sup>1</sup>|string|3|Possible values:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|
|**merchantScanReference**<sup>2</sup>|string |100       |Your internal reference for the transaction.                                                               
|**customerId**<sup>2</sup>           |string |100        |Your internal reference for the user.                                                                 
|merchantReportingCriteria        |string |255        |Your reporting criteria for the transaction.                                                                      
|callbackUrl<sup>3</sup>              |string |255        |Sends confirmation and any extracted data to this URL upon completion.<br>Overrides [Callback URL](#callback-error-and-success-urls) in the Customer Portal.|
|enableExtraction| Boolean ||Enables or disables data extraction<sup>4</sup> per transaction.<br><br>Possible values:<br>• `true` (extraction performed)<br>• `false` (no extraction performed)<br><br>**Data extraction will be performed by default if this parameter is not passed.**|
|customDocumentCode|string|100|Your custom document code (see [Multi documents](/netverify/portal-settings.md#multi-documents)).<br>**Mandatory when** `type` = `CUSTOM`|
|clientIp|string|100|IP address of the client.|

<sup>1</sup> Optional for type `CC`.<br>
<sup>2</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>3</sup> See [URL constraints](#callback-error-and-success-urls).<br>
<sup>4</sup> To activate data extraction for your account, please contact your Account Manager or Jumio Support.

<br>


#### Sample request
```
POST https://acquisition.netverify.com/api/netverify/v2/acquisitions/complete HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data;boundary=----xxxx
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
----xxxx
Content-Disposition: form-data; name="image"; filename="xxx.png"
Content-Type: image/png
// Your JPEG, PNG or PDF page
----xxxx
Content-Disposition: form-data; name="metadata"
{
"type": "SSC",
"country": "USA",
"merchantScanReference":"YOURSCANREFERENCE",
"customerId": "CUSTOMERID",
}
----xxxx
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

<br>


### Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name      | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**|string||Timestamp of the response in the format *YYYY-MM-DDThh:mm:ss.SSSZ*.|
|**scanReference**|string|36|Jumio reference number for the transaction.|

<br>


#### Sample response
```
{
"timestamp": "2017-10-17T07:56:58.613Z",
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

---

# Supported documents

Several standard document types are supported for upload and data extraction.

- BC (Birth certificate)
- BS (Bank statement)
- CAAP (Cash advance application)
- CB (Council bill)
- CC (Credit card)
- CCS (Credit card statement)
- CRC (Corporate resolution certificate)
- CUSTOM (Custom document type - see below)
- HCC (Health care card)
- IC (Insurance card)
- LAG (Lease agreement)
- LOAP (Loan application)
- MEDC (Medicare card)
- MOAP (Mortgage application)
- PB (Phone bill)
- SEL (School enrolment letter)
- SENC (Seniors card)
- SS (Superannuation statement)
- SSC (Social security card)
- STUC (Student card)
- TAC (Trade association card)
- TR (Tax return)
- UB (Utility bill)
- VC (Voided check)
- VT (Vehicle title)
- WWCC (Working with children check)

You can also create your own [custom document types](/netverify/portal-settings.md#multi-documents) in the Customer Portal.

## Data extraction

**Name**, **Address**, and **Issuing Date** will be extracted for all documents printed in a Latin-script character set, provided that a minimum of one of these three data points are available for extraction. If the document does not meet these extraction criteria, only the document image will be saved — no data extraction will be performed.

For the following specific document types, additional data will be extracted.

|Type | Extracted data |
|:---------------|:----------|
|BS (bank statement) |name, issueDate, address, accountNumber, swiftCode |
|CC (credit card) <sup>1</sup>|name, pan, expiryDate (currently returned as issueDate)|
|UB (utility bill) |name, issueDate, address, dueDate |
|OTHER (Other document type) |name, issueDate, address |
|CCS (credit card statement) |name, issueDate, address, cardNumberLastFourDigits |
|SSC (Social Security card) <sup>1,</sup><sup>2</sup> |firstName, lastName, ssn, signatureAvailable  |

<sup>1</sup> For CC and SSC all data points need to be available for extraction.<br>
<sup>2</sup> USA only.

### Disabling data extraction

Should you wish to turn data extraction off for certain documents, this can be accomplished on a per-transaction level by setting `enableExtraction` to `false`.

## Supported document types

Call the RESTful HTTP GET API **supportedDocumentTypes** to receive a JSON response including the code and names of all supported standard document types.

**HTTP request method:** `GET`<br>
**REST URL (US):** `https://netverify.com/api/netverify/v2/supportedDocumentTypes`<br>
**REST URL (EU):** `https://lon.netverify.com/api/netverify/v2/supportedDocumentTypes`<br>
**REST URL (SGP):** `https://core-sgp.jumio.com/api/netverify/v2/supportedDocumentTypes`<br>

### Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

### Sample Request
```
GET https://netverify.com/api/netverify/v2/supportedDocumentTypes HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

<br>


### Response

**Required items appear in bold type.**

|Name      | Type        | Description|
|:---------------|:------------|:------------|
|**timestamp**|string|Timestamp of the response in the format *YYYY-MM-DDThh:mm:ss.SSSZ*.|
|**supportedDocumentTypes**|array|Array of supported documents.|

#### Parameter `supportedDocumentTypes`

**Required items appear in bold type.**

|Name| Type |Max. length       | Description|
|:---------------|:------------|:------------|:------------|
|**code**|string|6|Unique identifier of the document type.|
|**name**|string|255|Full name of the document type.|

<br>


### Sample Response
```
{
"timestamp":"2017-10-17T14:25:59.779Z",
"supportedDocumentTypes":[
  {"code":"BS","name":"Bank statement"},
  {"code":"CC","name":"Credit card"},
  {"code":"IC","name":"Insurance card"},
  {"code":"UB","name":"Utility bill"},
  {"code":"CAAP","name":"Cash advance application"},
  {"code":"CRC","name":"Corporate resolution certificate"},
  {"code":"CCS","name":"Credit card statement"},
  {"code":"LAG","name":"Lease agreement"},
  {"code":"LOAP","name":"Loan application"},
  {"code":"MOAP","name":"Mortgage application"},
  {"code":"TR","name":"Tax return"},
  {"code":"VT","name":"Vehicle title"},
  {"code":"VC","name":"Voided check"},
  {"code":"STUC","name":"Student card"},
  {"code":"HCC","name":"Health care card"},
  {"code":"CB","name":"Council bill"},
  {"code":"SENC","name":"Seniors card"},
  {"code":"MEDC","name":"Medicare card"},
  {"code":"BC","name":"Birth certificate"},
  {"code":"WWCC","name":"Working with children check"},
  {"code":"SS","name":"Superannuation statement"},
  {"code":"TAC","name":"Trade association card"},
  {"code":"SEL","name":"School enrolment letter"},
  {"code":"PB","name":"Phone bill"},
  {"code":"SSC","name":"Social security card"},
  {"code":"CUSTOM","name":"DOCUMENT"}
  ]
}
```

---
# Deleting transactions

You can delete transactions easily in the Customer Portal.

In **Verifications**, search for the transaction you want to delete. Click the trash can icon to the right of the scan details to remove sensitive data (e.g., name, address, date of birth, document number, etc.) and image(s) from the transaction record.

You can also implement the RESTful DELETE API to remove sensitive data and image(s) from a completed transaction.<br>

[View the Delete API implementation guide](/netverify/netverify-delete-api.md)

When deleting transaction data, the Jumio scan reference and timestamp will be retained for reporting purposes.


----
# Supported Cipher Suites
Jumio supported cipher suites during the TLS handshake.<p>
[View supported cipher suites](/netverify/supported-cipher-suites.md)

----
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
