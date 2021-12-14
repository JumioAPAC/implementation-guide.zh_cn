![Jumio](/images/Jumio-Go-Banner.png)

# Jumio Go - Callback

The callback is the authorative answer from Jumio. Specify a callback URL (see [Configuring Settings in the Customer Portal](/netverify/portal-settings.md#callback-error-and-success-urls)) to automatically receive the result for each transaction.

## Table of contents

- [Jumio Callback IP List](#jumio-callback-ip-list)
- [Callback for Jumio Go](#callback-for-jumio-go)
  - [Supported Documents for Address Extraction](#supported-documents-for-address-extraction)
- [Callback for Authentication](#callback-for-authentication)
  - [Supported Documents for Data Extraction](#supported-documents-for-data-extraction)
- [ID Verification Retrieval API](#id-verification-retrieval-api)

___

## Jumio callback IP list
These are the IPs which Jumio sends callbacks from. Whitelist these IPs and use them to verify that the callback was sent from Jumio.

**US data center**

- 34.202.241.227<br>
- 34.226.103.119<br>
- 34.226.254.127<br>
- 52.52.51.178<br>
- 52.53.95.123<br>
- 54.67.101.173<br>
- Use the hostname *callback.jumio.com* to look up the most current IP addresses.

**EU data center**

- 34.253.41.236<br>
- 35.157.27.193<br>
- 52.48.0.25<br>
- 52.57.194.92<br>
- 52.58.113.86<br>
- 52.209.180.134<br>
- Use the hostname *callback.lon.jumio.com* to look up the most current IP addresses.

**SGP data center**

- 3.0.109.121<br>
- 52.76.184.73<br>
- 52.77.102.92<br>
- Use the hostname *callback.core-sgp.jumio.com* to look up the most current IP addresses.


## Callback for Jumio Go
An HTTP POST request is sent to your specified callback URL containing an `application/x-www-form-urlencoded` formatted string with the tranction result.

To specify a global callback URL in the Customer Portal, see [Configuring Settings in the Customer Portal](/netverify/portal-settings.md#callback-error-and-success-urls).

A callback URL can also be specified per transaction. See instructions for  [ID Verification Web v4](/netverify/netverify-web-v4.md#request-body), [ID Verification API](/netverify/performNetverify.md#request-body), and our SDK for [Android](https://github.com/Jumio/mobile-sdk-android/blob/master/docs/integration_authentication.md#callback) and [iOS](https://github.com/Jumio/mobile-sdk-ios/blob/master/docs/integration_authentication.md#configuration).

For Android/iOS: ID Verification must be enabled to receive the callback.

### Jumio Web Client

|User journey state|Transaction state|Callback|
|:---|:---|:---|
|Not started|Pending => Failed|Transaction will be cleaned-up from pending to failed after the authorization token lifetime expires. <br><br> Callback: Verification status = NO\_ID\_UPLOADED|
|Drop off during first attempt|Pending => Failed|Transaction will be cleaned-up after 15 minutes of no activity from the user. <br><br> Callback: Verification status = NO\_ID\_UPLOADED|
|Drop off during second or third attempt|Done|Transaction will be finished after 15 minutes of no activity from the user. <br><br> Callback: Verification status = ERROR\_NOT\_READABLE\_ID with previous reject reason|

### Jumio Mobile (Android/iOS)

|User journey state|Transaction state|Callback|
|:---|:---|:---|
|Drop off|Pending => Failed|Transaction will be cleaned-up from pending to failed after the authorization token lifetime expires. <br><br> Callback: Verification status = NO\_ID\_UPLOADED|
|Finished|Done|ID Verification will be performed|
|||Callback: Verification status depends on the resul (see table below)|

### Parameters

The following parameters are posted to your callback URL for ID Verification Web, ID Verification API, and ID Verification Mobile iOS/Android.

Required items appear in bold type.

|Parameter|Max Length|Description|Notes|
|:---|:---|:---|:---|
|**callbackType**||NETVERIFYID||
|**jumioIdScanReference**|36|Jumio's reference number for each transaction||
|**verificationStatus**||Possible states:<br> • APPROVED\_VERIFIED<br> • DENIED\_FRAUD<br> • DENIED\_UNSUPPORTED\_ID\_TYPE<sub>1</sub><br> • DENIED\_UNSUPPORTED\_ID\_COUNTRY<sub>1</sub><br> • ERROR\_NOT\_READABLE\_ID<br> • NO\_ID\_UPLOADED||
|**idScanStatus**||SUCCESS if verificationStatus = APPROVED\_VERIFIED, otherwise ERROR||
|**idScanSource**||Possible values:<br> • WEB (ID Verification Web before capture method selected)<br> • WEB\_CAM (ID Verification Web with camera capture)<br> • WEB\_UPLOAD (ID Verification Web with file upload)<br> • API (performNetverify)<br> • SDK (mobile)<br>||
|**idCheckDataPositions**||OK if verificationStatus = APPROVED\_VERIFIED, otherwise N/A||
|**idCheckDocumentValidation**||OK if verificationStatus = APPROVED\_VERIFIED, otherwise N/A||
|**idCheckHologram**||OK if verificationStatus = APPROVED\_VERIFIED, otherwise N/A||
|**idCheckMRZCode**||OK for passports and supported ID cards if verificationStatus = APPROVED\_VERIFIED, otherwise N/A||
|**idCheckMicroprint**||OK if verificationStatus = APPROVED\_VERIFIED, otherwise N/A||
|**idCheckSecurityFeatures**||OK if verificationStatus = APPROVED\_VERIFIED, otherwise N/A||
|**idCheckSignature**||OK if verificationStatus = APPROVED\_VERIFIED, otherwise N/A||
|**transactionDate**||Timestamp (UTC) of the transaction creation <br><br> format: [YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-1)||
|**callbackDate**||Timestamp (UTC) of the callback creation <br><br> format: [YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-1)||
|identityVerification||Identity Verification as JSON object<br><br>**ONLY** if verificationStatus = APPROVED\_VERIFIED<br><br> see [Identity Verification](#identity-verification) table below|activation required|
|idType||Possible types:<br> • PASSPORT <br> • DRIVING\_LICENSE<br> • ID\_CARD||
|idSubtype|255|Possible subtypes if idType = ID\_CARD<br> • NATIONAL\_ID<br> • CONSULAR\_ID<br> • ELECTORAL\_ID<br> • RESIDENT\_PERMIT\_ID<br> • TAX\_ID<br> • STUDENT\_ID<br> • PASSPORT\_CARD\_ID<br> • MILITARY\_ID<br> • PUBLIC\_SAFETY\_ID<br> • HEALTH\_ID<br> • OTHER\_ID<br> • VISA<br> • UNKNOWN<br><br>Possible subtypes if idType = DRIVING\_LICENSE<br> • REGULAR\_DRIVING\_LICENSE<br> • LEARNING\_DRIVING\_LICENSE<br><br> Possible subtypes if idType = PASSPORT<br> • E\_PASSPORT (only for mobile)||
|idCountry|3|Possible countries:<br> • [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3)<br> • eg. XKX (Kosovo)||
|rejectReason||Reject reason as JSON object if verificationStatus = DENIED\_FRAUD or ERROR\_NOT\_READABLE\_ID, see tables below||
|idScanImage|255|URL to retrieve the document image of transaction (JPEG or PNG) if available <sub>2</sub>||
|idScanImageFace|255|URL to retrieve the face image of transaction (JPEG or PNG) if available <sub>2</sub>||
idScanImageBackside|255|URL to retrieve the document backside image of transaction (JPEG or PNG) if available <sub>2</sub>||
|idNumber|200|Identification number of the document as available on the ID if enabled, otherwise if provided||
|idFirstName|200|First name of the customer as available on the ID if enabled, otherwise if provided||
|idLastName|200|Last name of the customer as available on the ID if enabled, otherwise if provided||
|idDob|10|Date of birth in the format YYYY-MM-DD as available on the ID if enabled, otherwise if provided<br><br>If idCountry = IND date of birth can be incomplete, possible values e.g.:<br> • Year-Month-Day: 1990-12-09<br> • Year only: 1990-01-01<br> • Year-Month: 1990-12-01<br> • Year-Day: 1990-01-09||
|idExpiry|10|Date of expiry in the format YYYY-MM-DD as available on the ID if enabled, otherwise if provided||
|idUsState|255|Possible values if idType = PASSPORT or ID\_CARD:<br> • Last two characters of [ISO 3166-2:US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br> • Last 2-3 characters of [ISO 3166-2:AU](http://en.wikipedia.org/wiki/ISO_3166-2:AU) state code<br> • Last two characters of [ISO 3166-2:CA](http://en.wikipedia.org/wiki/ISO_3166-2:CA) state code<br> • [ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country name<br> • e.g. XKX (Kosovo)<br> • Free text - if it can't be mapped to a state/country code<br><br>If idType = DRIVING\_LICENSE:<br> • Last two characters of ISO 3166-2:US state code<br> • Last 2-3 characters of ISO 3166-2:AU state code<br> • Last two characters of ISO 3166-2:CA state code||
|personalNumber|14|Personal number of the document<br> • if idType = PASSPORT and if data available on the document||
|idAddress||Address as JSON object in US, EU, or RAW format, see tables below<sub>3</sub>|activation required|
|merchantIdScanReference|100|Your reference for each transaction||
|merchantReportingCriteria|100|Your reporting criteria for each transaction||
|customerId|100|ID of the customer as provided||
|clientIp||IP address of the client in the format xxx.xxx.xxx.xxx||
|firstAttempt||Timestamp (UTC) of the first transaction attempt<br><br>format:[YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-1)||
|presetCountry|3|Possible countries:<br> • [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br> • e.g. XKX (Kosovo)||
|presetIdType||Possible ID types:<br> • PASSPORT<br> • DRIVING\_LICENSE<br> • ID\_CARD||
|issuingDate|10|Issuing date of the document|activation required|
|livenessImages||URLs to the liveness images of the transaction (JPEG or PNG/0 if available<sub>4</sub>||
|facemap|255|URL to the facemap of the transaction, if available|activation required|
|curp|255|CURP is available if idCountry = MEX and idType = PASSPORT, DRIVING\_LICENSE, or ID\_CARD and idSubtype = ELECTORAL\_ID|activation required|
|cpf|255|CPF number of the document|activation required|
|registrationNumber|255|Registration number of the document|activation required|
|personalIdentificationNumber|255|Personal identification number as available on the ID<br><br> • if idCountry = GEO and idSubtype = PASSPORT<br> • if idCountry = COL and idSubtype = ID\_CARD|activation required|
|rgNumber|255|"General Registration" number for idCountry = BRA|activation required|

<sup>1</sup> Transaction is declined as unsupported if the ID is not supported by Jumio Go, or not marked as accepted in your Customer Portal settings.

<sup>2</sup> For ID types that are configured to support a seperate scan of the frontside and backside, there is a separate image of the frontside (idScanImage) and backside (idScanImageBackside). If Identity Verification is enabled, there is also a picture of the face (idScanImageFace).

<sup>3</sup> Address recognition is performed for supported IDs, if enabled. The different address parameters are a part of the JSON object, if they are available on the ID.

<sup>4</sup> Liveness images are returned only for transactions containing Identity Verification submitted via the Android and iOS SDKs. The number of images can vary and may not be returned in chronological order.

#### Retrieving images

Use HTTP **GET** with **Basic Authorization** using your API token and secret as userid and password.<br>
**Header**: The following parameters are mandatory in the header section of your request.<br>
- `Accept: image/jpeg, image/png`<br>
- `User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br /><br />
The value for **User-Agent** must contain a reference to your business or entity for Jumio to be able to identify your requests. (e.g. YourCompanyName YourAppName/1.0.0). Without a proper User-Agent header, Jumio will take longer to diagnose API issues.<br>

The TLS protocol is required during the TLS handshake (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

#### Timestamp format

Timestamp are sent in format: `YYYY-MM-DDThh:mm:ss.SSSZ` with constraint that SSS (milliseconds) can either be
- Not included: `YYYY-MM-DDThh:mm:ssZ`
- 1 digit: `YYYY-MM-DDThh:mm:ss.SZ`
- 2 digits:  `YYYY-MM-DDThh:mm:ss.SSZ`
- 3 digits: `YYYY-MM-DDThh:mm:ss.SSSZ`

We encourage to use a standard library to convert the timestamp received from Jumio as the timeformat is valid with and without the SSS milliseconds. <br/><br>

### Supported documents for address extraction

|Country    |ID card    |Driving license    |Passport    |Callback format|
|:------------|:-------|:--------------|:--------------|:-------|
|Australia|No|Yes|No|Raw|
|Canada|No|Yes|No|Raw|
|France|Yes|No|No|Raw|
|United Kingdom|No|Yes|No|Raw|
|Indonesia|Yes|No|No|Raw|
|Ireland|No|Yes|No|Raw|
|Mexico|Yes|No|No|Raw|
|Malaysia|Yes|No|No|Raw|
|United States|Yes|Yes|No|Raw|


#### US address format

|Parameter `idAddress`       | Max. Length    | Description|
|:---------------|:--------|:------------|
|city   |64  |City |
|stateCode   |6  |[ISO 3166-2](http://en.wikipedia.org/wiki/ISO_3166-2) state code |
|streetName   |64  |Street name |
|streetSuffix   |14  |Street suffix abbreviation<br/>Examples: [US](http://www.gis.co.clay.mn.us/USPS.htm#suffix), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1423617), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|streetDirection   |255  |Street direction abbreviation<br/>Examples: US (E=EAST, W=WEST, N=NORTH, S=SOUTH), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1403220), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|streetNumber   |14  |Street number |
|unitDesignator   |14  |Unit designator abbreviation<br/>Examples: [US](http://www.gis.co.clay.mn.us/USPS.htm#secunitdesig), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1380473), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|unitNumber   |14  |Unit number |
|zip   |14  |Zip code |
|zipExtension   |20  |Zip extension |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo) |

#### EU address format

|Parameter `idAddress`       | Max. Length    | Description|
|:---------------|:--------|:------------|
|city   |64  |City |
|province   |64  |Province |
|streetName   |64  |Street name |
|streetNumber   |15  |Street number |
|unitDetails   |64  |Unit details |
|postalCode   |15  |Postal code |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo) |

#### Raw address format

|Parameter `idAddress`      | Max. Length    | Description|
|:---------------|:--------|:------------|
|line1   |100  |Line item 1 |
|line2   |100  |Line item 2 |
|line3   |100  |Line item 3 |
|line4   |100  |Line item 4 |
|line5   |100  |Line item 5 |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo)  |
|postalCode   |15  |Postal code |
|city   |64  |City |
|subdivision   |100  |Subdivision (Region, State, Province, Emirate, Department, ...) |

#### Upcoming Address format changes

On August 17, 2020 onwards, Jumio is going to streamline the EU and US format into a single Raw format. In addition to that, we will also be providing a new parameter called formattedAddress which will contain  the entire address in a comma separated format.

**Example**

|US Address Format|Raw Address Format|
|:------------------------|:--------|
|"streetNumber": "150"<br>"streetDirection":"N"<br>"streetName":"Alberny"<br>"streetSuffix":"Dr"<br>"unitDesignator":"APT"<br>"unitNumber": "7"|"line1":"150 N Alberny Dr APT. 7"|
|"city":"New Mexico"|city":"New Mexico"|
|"stateCode":"US-AL"|"subdivision":"US-AL"|
|"zip":"77358"<br>"zipExtension":"5555"|"postalCode":"77358-5555"|
|"country":"USA"|"country":"USA"|
||“formattedAddress”: “150 N Alberny Dr APT. 7, New Mexico, US-AL 77358-5555, United States”|

|EU Address Format|Raw Address Format|
|:------------------------|:--------|
|"streetNumber":"180"<br>"streetName":"Main Street"<br>"unitDetails":"ABC"|"line1": "180 Main Street ABC"|
|"city": "Graz"|"city": "Graz"|
|"province": "Steiermark"|"subdivision": "Steiermark"|
|"postalCode": "2424"|"postalCode": "2424"|
|"country": "AUT"|"country": "AUT"|
||“formattedAddress”: “180 Main Street ABC, Graz, Steiermark 2424, Austria”|

### Reject reason

|Parameter `rejectReason` | Type   | Max. Length    | Description|
|:------------------------|:--------|:--------|:------------|
|rejectReasonCode |String| 5  |see below |
|rejectReasonDescription |String |64  |Possible codes and descriptions for verification status DENIED\_FRAUD:<br>100	MANIPULATED\_DOCUMENT<br/>105	FRAUDSTER<br/>106	FAKE<br/>108	MRZ\_CHECK\_FAILED<br/>109	PUNCHED\_DOCUMENT<br/><br>Possible codes and descriptions for verificationStatus = ERROR\_NOT\_READABLE\_ID:<br/>102	PHOTOCOPY\_BLACK\_WHITE<br/>104	DIGITAL\_COPY<br/>200	NOT\_READABLE\_DOCUMENT<br/>201	NO\_DOCUMENT<br/>206	MISSING\_BACK<br/>|
|rejectReasonDetails |Object  |   |Reject reason details as JSON array containing JSON objects if rejectReasonCode = 100 or 200, see table below |

### Reject reason details

|Parameter `rejectReasonDetails` |  Type    | Max. Length    | Description|
|:-------------------------------|:---------|:---------------|:------------|
|detailsCode   |String | 5 | see below |
|detailsDescription|String|32| Possible codes and descriptions for rejectReasonCode = 100:<br/>1002	DOCUMENT\_NUMBER<br><br><br>Possible codes and descriptions for rejectReasonCode = 200:<br/>2001	BLURRED<br/>2002	BAD\_QUALITY<br/>2006 GLARE |

### Identity Verification

**Required items appear in bold type.**  

|Parameter `identityVerification`       | Max. Length    | Description|
|:---------------|:--------|:------------|
|**similarity** <sup>1</sup>   |  |Possible values:<br/> •	MATCH<br />•	NO\_MATCH<br />•	NOT\_POSSIBLE (not executed or bad quality of face on document)|
|**validity** <sup>2</sup>   |  |Possible values:<br/> •	TRUE<br />•	FALSE |
|reason   |  |Provided if validity = FALSE<br/>Possible values:<br />• SELFIE\_CROPPED\_FROM\_ID<br />•	ENTIRE\_ID\_USED\_AS\_SELFIE<br />•	MULTIPLE\_PEOPLE<br />•	SELFIE\_IS\_SCREEN\_PAPER\_VIDEO<br />•	SELFIE\_MANIPULATED<br />• AGE\_DIFFERENCE\_TOO\_BIG<br />•	NO\_FACE\_PRESENT<br />•	FACE\_NOT\_FULLY\_VISIBLE<br />•	BAD\_QUALITY<br />•	BLACK\_AND\_WHITE<br />•	LIVENESS\_FAILED <sup>3</sup>|

<sup>1</sup> Is the person on the selfie the same as the one on the document?<br>
<sup>2</sup> Is it a live person?<br>
<sup>3</sup> Potential reasons `LIVENESS_FAILED`:

- User tries to spoof the system
- User does not want to show his face at all but wants to complete the onboarding
- User does not look straight into the camera
- User does not finish the the Identity Verification process
- User has bad lighting conditions (too dark, too bright, reflections on face, not enough contrast, …)
- User is covering (parts) of his face with a scarf, hat, or something similar
- User is not able to align his face with the oval
- A different person is performing the Identity Verification in the second step than in the first one

### Sample Callbacks

#### Sample callback (URL-encoded POST): Approved and verified

```
idExpiry=2022-12-31&idType=PASSPORT&idDob=1990-01-01&idCheckSignature=OK&idCheckDataPositions=OK&idCheckHologram=OK&idCheckMicroprint=OK&idCheckDocumentValidation=OK&idCountry=USA&idScanSource=SDK&idFirstName=FIRSTNAME&verificationStatus=APPROVED_VERIFIED&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&personalNumber=N%2FA&merchantIdScanReference=YOURIDSCANREFERENCE&idCheckSecurityFeatures=OK&idCheckMRZcode=OK&idScanImage=https%3A%2F%2Fnetverify.com%2Frecognition%2Fv1%2Fidscan%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Ffront&callBackType=NETVERIFYID&clientIp=xxx.xxx.xxx.xxx&idLastName=LASTNAME&idAddress=%7B%22country%22%3A%22USA%22%2C%22stateCode%22%3A%22US%2DOH%22%7D&idScanStatus=SUCCESS&identityVerification=%7B%22similarity%22%3A%22MATCH%22%2C%22validity%22%3Atrue%7D&idNumber=P1234
```

#### Sample callback (URL-encoded POST): Fraud

```
idType=PASSPORT&idCheckSignature=N%2FA&rejectReason=%7B%20%22rejectReasonCode%22%3A%22100%22%2C%20%22rejectReasonDescription%22%3A%22MANIPULATED_DOCUMENT%22%2C%20%22rejectReasonDetails%22%3A%20%5B%7B%20%22detailsCode%22%3A%20%221001%22%2C%20%22detailsDescription%22%3A%20%22PHOTO%22%20%7D%2C%7B%20%22detailsCode%22%3A%20%221004%22%2C%20%22detailsDescription%22%3A%20%22DOB%22%20%7D%5D%7D&idCheckDataPositions=N%2FA&idCheckHologram=N%2FA&idCheckMicroprint=N%2FA&idCheckDocumentValidation=N%2FA&idCountry=USA&idScanSource=SDK&verificationStatus=DENIED_FRAUD&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&merchantIdScanReference=YOURSCANREFERENCE&idCheckSecurityFeatures=N%2FA&idCheckMRZcode=N%2FA&idScanImage=https%3A%2F%2Fnetverify.com%2Frecognition%2Fv1%2Fidscan%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Ffront&callBackType=NETVERIFYID&clientIp=xxx.xxx.xxx.xxx&idScanStatus=ERROR
```
---

## Callback for Authentication

An HTTP POST request is sent to your specified callback URL containing an `application/x-www-form-urlencoded` formatted string with the transaction result.

To specify a global callback URL in the Customer Portal, see [Configuring Settings in the Customer Portal](/netverify/portal-settings.md#callback-error-and-success-urls).

A callback URL can also be specified per transaction in our [Android](https://github.com/Jumio/mobile-sdk-android/blob/master/docs/integration_authentication.md#callback) and [iOS](https://github.com/Jumio/mobile-sdk-ios/blob/master/docs/integration_authentication.md#configuration) SDK.


### Parameters

**Required items appear in bold type.**  

|Parameter       | Type    | Max. Length|  Description|
|:---------------|:--------|:----------: |:------------|
|**callbackDate**| string  |  | Timestamp of the callback in the format: <br>[YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-1)|
|**transactionReference**| string  |36 |Jumio’s reference number for the Authentication transaction|
|**enrollmentTransactionReference**| string  |36 |Jumio’s reference number of the enrollment transaction (ID)|
|**transactionResult**| string  | |Possible values:<br>•	PASSED<br>•	FAILED<br>•	INVALID<br>•	EXPIRED|
|**transactionDate**| string  |  |Timestamp of the transaction in the format: <br>[YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-1)|
|**scanSource**|string||Possible values:<br>•	SDK<br>•	WEB|
|**callBackType**|string ||NETVERIFY_AUTHENTICATION|
|idScanImageFace | JSON array/object | |URL to retrieve the face image of the transaction (JPEG or PNG) <sup>1</sup> |
|livenessImages| JSON array  |  | URLs to the liveness images of the transaction (JPEG or PNG) <sup>1</sup> <sup>2</sup>|
|userReference |string  |  |Your internal reference for the user. |

<sup>1</sup> Retrieve the images of the transaction.<br>
<sup>2</sup> The number of images can vary and may not be returned in chronological order.

#### Retrieving Images
Use HTTP **GET** with **Basic Authorization** using your API token and secret, as userid and password.<br>

**Header**: The following parameters are mandatory in the header section of your request.<br>
- `Accept: image/jpeg, image/png`<br>
- `User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br /><br />
The value for **User-Agent** must contain a reference to your business or entity for Jumio to be able to identify your requests. (e.g. YourCompanyName YourAppName/1.0.0). Without a proper User-Agent header, Jumio will take longer to diagnose API issues.<br>

The TLS protocol is required during the TLS handshake (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

#### Timestamp format

Timestamp are sent in format: `YYYY-MM-DDThh:mm:ss.SSSZ` with constraint that SSS (milliseconds) can either be:

- Not included: `YYYY-MM-DDThh:mm:ssZ`
- 1 digit: `YYYY-MM-DDThh:mm:ss.SZ`
- 2 digits:  `YYYY-MM-DDThh:mm:ss.SSZ`
- 3 digits: `YYYY-MM-DDThh:mm:ss.SSSZ`

We encourage to use a standard library to convert the timestamp received from Jumio as the timeformat is valid with and without the SSS milliseconds. <br/><br>

### Sample Callback

#### Sample callback (URL-encoded POST): Passed

```
scanSource=SDK&transactionReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&enrollmentTransactionReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&callBackType=NETVERIFY_AUTHENTICATION&livenessImages=%5B%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F1%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F2%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F3%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F4%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F5%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F6%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F7%22%5D&idScanImageFace=https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fface&callbackDate=2019-05-30T08%3A37%3A35.822Z&transactionDate=2019-05-29T08%3A37%3A24.344Z&transactionResult=PASSED
```
---

## ID Verification Retrieval API
If your server was not able to receive or process the callback, you can use the [Retrieval API](/netverify/netverify-retrieval-api.md) to retrieve the results of your transaction.


---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
