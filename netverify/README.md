![Jumio](/images/Jumio-ID-Verification-Banner.png)

# Revision history
The list below provides information on the changes to ID Verification features and improvements documented in each release.

## Contact

If you have any questions regarding our implementation guides please contact Jumio Support at support@jumio.com.<p>

The Jumio Knowledge Base also contains a wealth of information regarding our services: FAQs, best practices, demo videos, product descriptions, and additional information to help get you started with Jumio.<p>

[Jumio Knowledge Base](https://support.jumio.com)



---
## ID Verification Web and Document Verification

### Revision history

| Date           | Description       |
|:---------------|:------------------|
| 2020-12-02  |Provided images (front, back, selfie) with a width lower than 300 pixels will no longer be accepted. |
| 2020-11-12  |Added gender extraction for Peru ID |
| 2020-10-22  |Added nationality extraction for France and Belgium |
| 2020-10-15  |Added voter and issuing number extraction for Mexico ID documents<br> Added callback parameter "voterIdNumber" and "issuingNumber"|
| 2020-10-12  |Added personal number extraction for Romanian ID cards and driver licenses ("personalIdentificationNumber")|
| 2020-09-25  |Added address extraction for India ID cards  |
| 2020-09-04  |Added personal number extraction for Turkey ID cards and driver licenses ("personalIdentificationNumber")|
| 2020-08-05  |Added gender extraction for Chile ID<br>Added nationality extraction for Chile ID |
| 2020-07-17  |Added personal number extraction for Lithuania driver licenses ("personalIdentificationNumber")|
| 2020-06-19  |Added Croatian localization for ID Verification Web<br>Added support for Safari WebView for ID Verification Web<br>Added new callback value for "Not readable document" (GLARE)|
| 2020-04-17  |Added callback value "REGULAR\_DRIVING\_LICENSE" to parameter "idSubtype"|
| 2020-04-09  |Added support for native WebViews for ID Verification Web|
| 2020-02-26  |Added Arabic and Hebrew localizations for ID Verification Web|
| 2020-02-12  |Added place of birth extractioin for AUS, BEL, BRA, DEU, ESP, FRA, GBR, HKG, HUN, IDN,<br>IND, IRL, ITA, NGA, NLD, PAK, POL, PRT, ROU, RUS, SGP, TUR, UKR, USA|
| 2020-02-11  |Added issuing place extraction for Colombia|
| 2020-01-22  |Added callback parameter "rgNumber"|
| 2019-12-31  |Added address extraction for Malta ID cards and driver licenses |
| 2019-12-12  |Added new error code 9822 for ID Verification Web|
| 2019-12-02  |Added callback parameter "personalIdentificationNumber"|
| 2019-11-25  |Added callback parameter "similarityDecision", "similarityScore" to "identityVerification"|
| 2019-10-01  |Added callback parameter "cpf", "registrationNumber", "mothersName", and "fathersName"|
| 2019-09-19  |Added Authentication with Facemap Storage on Customer Premise for Web|
| 2019-09-04  |Added callback parameter "taxNumber", value "HEALTH\_ID" to <br>parameter "idSubtype"|
| 2019-08-28  |Renaming parameter for Authentication for Web from customerInternalReference (mandatory) to <br>userReference (optional) |
| 2019-08-22  |Added Indonesian, Khmer, Malay and Thai localizations for ID Verification Web |
| 2019-08-14  |Renaming endpoints for Authentication for Web and Authentication with Facemap<br>Storage on Customer Premise |
| 2019-08-05  |Added Authentication with Facemap Storage on Customer Premise for Mobile |
| 2019-07-23  |Added response parameter "facemap" for ID Verification |
| 2019-07-18  |Added Armenian and Georgian localizations for ID Verification Web|
| 2019-07-11  |Added Authentication for Web (Beta)|
| 2019-07-10  |Added value "EXPIRED" to callback parameter transactionResult|
| 2019-07-04  |Added Bulgaria, Latvia, Lithuania and Romania to extract DL categories<br>Added new callback parameter "placeOfBirth" for ID Verification<br>Added gender extraction for Hongkong ID<br>Added place of birth extraction for Hongkong ID|
| 2019-06-27  |Added full screen mode for Liveness (Beta)|
| 2019-06-13  |Added support for Samsung Internet Browser on Android for ID Verification Web|
| 2019-05-29  |Added Callback for Authentication|
| 2019-05-28  |Updated max. size for frontsideImage, faceImage and backsideImage to 15MB|
| 2019-04-08  |Added Liveness for Web (Beta)|
| 2019-02-15  |Added value "OTHER" to callback parameter "type" for Document Verification|
| 2019-02-08   |Added parameter "availability" to dlCategories <br>Added extraction of motorcycle permission from Taiwan driving licenses|
| 2019-02-06   |Updated supported document types section to list all available "types" |
| 2019-01-31   |Added new validity reason "LIVENESS\_FAILED"|
| 2019-01-30   |Formatting and grammar edits|
| 2019-01-29   |Added address extraction for Bahrain driver licenses|
| 2019-01-15   |Added response parameter "livenessImages" for ID Verification|
| 2018-11-29   | Added address extraction for Indonesian ID cards |
| 2018-11-20   | Added property "tokenLifetimeInMinutes" to ID Verification Web version 4 initiate request |
| 2018-11-12 |Added callback parameters "issuingAuthority" and "issuingPlace", value "PUBLIC\_SAFETY\_ID" to <br>parameter "idSubtype", clarified browser support|
| 2018-10-24 |Added possibility for ID Verification Web version 4 to use a Jumio redirectUrl (NV) or <br/>clientRedirectUrl (DV) as the successUrl for your transaction, allowing to daisy-chain multiple <br /> verifications together in your user journey|
| 2018-10-05 |Added Place of Birth extraction for France, address extraction for Romanian ID cards<br>Document Verification: Added data extraction of BIC/SWIFT code and callback parameter "swiftCode",<br> added data extraction for all document types with Latin-script characters<br>|
| 2018-07-24 |Release of ID Verification Web version 4|
| 2018-07-03 |Document Verification: Added data extraction for all Bank Statements (BS), Utility Bills (UB),<br> and Credit Card Statements (CCS) with Latin-script characters|
| 2018-06-19 |Added address extraction for Singapore ID cards|
| 2018-06-05 |Added callback parameter "originDob"|
| 2018-05-22 |Finally removed callback parameter "idFaceMatch" and "idFaceLiveness"|
| 2018-04-11 |Added address extraction for French passports, driver licenses and ID cards|
| 2018-03-01 | Added request parameter "enableExtraction" to Document Verification<br> Added additional countries for Bank Statement and Utility Bill for extraction |
| 2018-02-14 | Updated supported browsers |
| 2018-02-01 |Added callback parameter "originalDocument" for Document Verification<br />Added Australia and Canada states to callback parameter "idUsState"|
| 2018-01-17  |Added new validity reason BLACK\_AND\_WHITE<br>Updated supported countries for idSubtype LEARNING\_DRIVING\_LICENSE|
| 2017-11-23  |Added "Visa" to request parameter "idType" for performNetverify<br>Added "Visa" to callback parameter "idType"<br>Added credit card to supported documents for data extraction<br>Added callback parameters "pan" and "expiryDate" to extractedData for Document Verification|
| 2017-11-08  |Added credit card support for Document Verification and Document Verification API|
| 2017-10-30  |Added callback parameters "presetCountry" and "presetIdType"|
| 2017-10-03  |Added callback parameter "dlCategories"<br>Added embed code parameter "clientWidth" and "clientHeight" for ID Verification Web|
| 2017-09-21  |Removed API and callback parameter "additionalInformation"|
| 2017-09-05  |Added request parameters "presetCountry" and "presetIdType" for ID Verification embedded and<br> ID Verification redirect<br>Changed default value of "authorizationTokenLifetime" request parameter|
| 2017-08-24  |Added callback parameter "identityVerification"<br>Added callback IP addresses<br>Added Estonian and Spanish Mexican localizations|
| 2017-06-28  |Added iFrame logging (optional) for ID Verification embedded|
| 2017-05-23  |Added some callback parameters to extracted data for ID Verification Multi Document<br>Removed some callback IP addresses for the US data center|
| 2017-05-09  |Updated max. length of request and callback parameter "clientIp"|
| 2017-04-26  |Updated "idFirstName" and "idLastName" callback parameter description|
| 2017-04-11  |Added Lithuanian localization|
| 2017-03-28  |Added some callback parameters to extractedData for ID Verification Multi Document|
| 2017-02-28  |Added capture methods for supported browsers|
| 2017-01-31  |Added "errorCode" 338 to HTTP GET parameter for Multi Document error redirect URL<br>Added details for custom document type for ID Verification Multi Document|
| 2017-01-17  |Added value "MILITARY\_ID" to callback parameter "idSubtype"|
| 2016-11-15  |Added reject reason detail code 10011 for PLACE\_OF\_BIRTH|
| 2016-10-04  |performNetverify parameters "country" and "idType" are mandatory |
| 2016-08-17  |Updated "idFirstName" and "idLastName" callback parameter description|
| 2016-06-21  |Added callback IP addresses for the US data center<br>Added callback parameter "dni" for ID Verification|
| 2016-06-13  |Added value "PASSPORT\_CARD\_ID" to callback parameter "idSubtype"|
| 2016-05-18  |Added value "STUDENT\_ID" to callback parameter "idSubtype"<br>Removed TLS\_DHE ciphers|
| 2016-03-21  |Added value "TAX\_ID" to callback parameter "idSubtype"|
| 2016-02-24  |Added callback parameter "idFaceLiveness" for ID Verification|
| 2016-02-09  |Added reject reason code 111 for MISMATCH\_PRINTED\_BARCODE\_DATA|
| 2015-12-09  |Added request parameter "authorizationTokenLifetime" for ID Verification Multi Document<br>Updated max length of "successUrl" for ID Verification embedded and ID Verification redirect |
| 2015-11-17  |Added embed code parameter "responsiveLayout" for ID Verification Web<br>Added value "E_PASSPORT" to callback parameter idSubtype|
| 2015-10-21  |Removed name match functionality<br>Added ECDHE ciphers to supported cipher suites |
| 2015-10-06  |Removed value "UNSUPPORTED" from callback parameter "idType"|
| 2015-09-23  |Added callback parameter "gender" and "idSubtype" for ID Verification<br>Updated supported environments<br>Initial release of Delete API|
| 2015-09-09  |Added custom document type for ID Verification Multi Document|
| 2015-08-25  |Added callback parameters "optionalData1" and "optionalData2"<br>Added HTTP GET parameters to success and error redirect URLs for ID Verification Multi Document<br>Added Vietnamese localization|
| 2015-08-11  |Added callback parameter "idScanImageFace" for ID Verification|
| 2015-07-14  |Added value "XKX" for Kosovo to parameters "country", "idCountry" and "countryCode"<br>Added values "XK" and "XKX" to parameter "usState"<br>Added value "Kosovo" to parameters "usState", "idUsState" and "countryName" |
| 2015-06-30  |Introduced EU data center<br>Added "Content-Length" header for HTTP POST APIs|
| 2015-06-16  |Added recommended dimensions (width, height) for ID Verification Multi Document|
| 2015-06-02  |Added Windows Phone 8.1 support for ID Verification Multi Document|
| 2015-04-08  |Introduced face match for ID Verification Web uploads<br>Added reject reason code 109 for PUNCHED\_DOCUMENT and removed code 205|
| 2015-03-24  |Added reject reason detail code 1009 for PERSONAL_NUMBER<br>Removed cipher TLS\_RSA\_WITH\_RC4\_128\_SHA due to RC4 deprecation|
| 2015-03-09  |Added code 210 to HTTP GET parameter "errorCode" for error redirect URL|
| 2015-03-05  |Added values "WEB" and "REDIRECT" to callback parameter "idScanSource"<br>Callback parameters "idType" and "idCountry" not mandatory anymore |
| 2015-03-02  |Allow ID Verification Web embed code in body tag<br>Added parameter "captureMethod" for ID Verification Web<br>Added callback parameters "transactionDate", "firstAttemptDate" and "callbackDate" for<br /> ID Verification<br>Added callback parameter "images" for ID Verification Multi Document|
| 2015-02-10  |Added parameters "baseColor", "bgColor" and "headerImageUrl" for ID Verification<br />Multi Document customization<br>Added MRZ check for specific ID cards |
| 2015-01-27  |Added parameter "locale" for ID Verification Multi Document<br>Changed max. length of parameter "authorizationTokenLifetime" to 5184000 seconds (60 days)<br>Changed callback clean-up behavior for ID Verification Web|
| 2014-11-20  |Introduced new ID Verification Multi Document client<br>Added Brazilian Portuguese localization<br>Corrected iFrame dimensions|
| 2014-10-21 |Introduced ID Verification Multi Document API<br>Moved data retention from "Application settings" to "Data settings"|
| 2014-10-07  |Added Bulgarian, Chinese (China and Hong Kong), Czech, Danish, Greek, Hungarian, Japanese,<br> Korean, Romanian and Slovak localizations<br>Added social security card to supported documents for ID Verification Multi Document|
| 2014-09-25  |Added test IDs|
| 2014-09-03  |Added supported environments<br>Updated allowed ID Verification redirect domain name prefix characters|
| 2014-08-19  |Introduced ID Verification Retrieval API<br>Corrected max. length of parameter "clientIp" in the callback<br>|
| 2014-08-05  |Introduced name match for all ID Verification products<br>Introduced success and error URL redirect parameters for ID Verification redirect<br>Added HTTP GET parameter "errorCode" for error redirect URL<br>Added capture method in ID Verification settings|
| 2014-07-22  |Added callback parameter "stateCode" for US address format<br>Removed callback parameter "state" from US address format|
| 2014-07-08  |Added two-factor authentication<br>Added Dutch localization|
| 2014-06-10  |Added constraints for success, error and callback URLs<br>Added/updated allowed ID Verification redirect domain name prefix characters<br>Added US social security card to supported documents for ID Verification Multi Document<br>Updated supported cipher suites during SSL handshake<br>Corrected max. length of parameter "code" for supportedDocumentTypes API|
| 2014-04-23  |Added supported cipher suites during SSL handshake (TLS required)|
| 2014-03-11  |Introduced data deletion|
| 2014-02-03  |Introduced manual transaction setup for ID Verification redirect<br>Added "User-Agent" header for callback image URL requests<br>Enhanced supported documents for ID Verification Multi Document<br>initiateNetverifyRedirect parameter "successUrl" and "errorUrl" not mandatory anymore<br>Removed "callbackEmail" API parameters and related application setting|
| 2014-01-14  |Changed performNetverify RESTful URL from v1 to v2<br>Added parameters "faceImage" and "faceImageMimeType" for performNetverify face match|
| 2013-12-17  |Success and error URLs should be HTTPS using the TLS protocol<br>Removed custom HTTP GET ID Verification redirect landing page parameters<br>Added callback parameters "postalCode" and "city" for raw address format<br>Added HTTP status code 403 Forbidden for all RESTful APIs<br>Added scan state explanations in the sections "Redirecting the customer after the user journey" <br>and "Callback"<br>Added footnotes 7 and 8 in the ID Verification callback table|
| 2013-11-13  |Introduced RESTful supportedIdTypes and acceptedIdTypes APIs<br>Changed "Mandatory fields" to "Data settings" separating mandatory and optional fields<br>Added HTTP GET method restriction for callback image retrieval<br>Changed RESTful URL getSupportedDocumentTypes to supportedDocumentTypes<br>Parameters "customerId" and "additionalInformation" should not contain sensitive data<br>Parameter "merchantIdScanReference" must not contain sensitive data|
| 2013-10-29  |Added "User-Agent" header for all RESTful APIs<br>Added callback parameter "merchantReportingCriteria"<br>Added reject reason code 108 for MRZ\_CHECK\_FAILED<br>Removed reject reason code 208 for NOTHING\_SHOWN<br>Removed createDocumentAcquisition parameters "firstName", "lastName" and "country"<br>Added maximum file size for performNetverify parameters "frontsideImage" and "backsideImage"|
| 2013-10-14  |Added color customization in ID Verification settings<br>Deprecated callback email<br>createDocumentAcquisition parameter "country" not mandatory anymore|
| 2013-09-24  |Added parameter "merchantReportingCriteria" to the createDocumentAcquisition API and the <br>ID Verification Multi Document callback<br>Corrected values of "idUsState" parameter for initiateNetverify, initiateNetverfyRedirect, <br>performNetverify and ID Verification callback<br>Corrected data types for all APIs from Java types to JSON types|
| 2013-09-10  |Added callback parameter "rejectReason"<br>Enhanced "Accepted IDs" settings|
| 2013-08-13  |Enhanced "idAddress" callback parameter with EU and raw format<br>Added "additionalInformation" parameter for initiateNetverify, initiateNetverifyRedirect and <br>performNetverify<br>Added additional information input field for ID Verification redirect<br>Added callback parameter "additionalInformation"<br>Added value "UNSUPPORTED" for callback parameter "idType"|
| 2013-08-01  |Added locale specific settings for ID Verification redirect<br>Added value "idAddress" to "enabledFields" parameter<br>Added French, Italian, Portuguese and Spanish localizations|
| 2013-07-17  |Added maximum field lengths for all JSON parameters|
| 2013-06-27  |Corrected values of callback parameter "idScanSource"<br>Corrected value "ID\_CARD" of callback parameter "idType"|
| 2013-05-28  |Added Finnish, Norwegian, Polish, Russian and Swedish localization|
| 2013-05-14  |Introduced face match in Jumio merchant settings and callback<br>Added HTTP GET parameter "merchantIdScanReference" to success and error redirect URLs for <br>embedded ID Verification<br>Added new callback field "idScanSource" |
| 2013-05-02  |Added callback IP addresses<br>Added maximum parameter lengths for callback fields|
| 2013-04-16  |Added RESTful getSupportedDocumentTypes API for ID Verification Multi Document|
| 2013-04-03  |initiateNetverify API is mandatory for embedded and redirect<br>HTTPS callback URL using TLS<br>Added document types for ID Verification Multi Document|
| 2013-03-05  |Updated ID Verification Multi Document parameters|
| 2013-03-04  |Introduced ID Verification Multi Document<br>Added address recognition for US IDs|
| 2013-02-18  |Removed "showIntroductionText" parameter|
| 2013-02-14  |Removed API version 1.0|
| 2013-02-13  |Introduced client version 2.0 with improved UI|
| 2012-12-18  |Relocated and redesigned ID Verification customer portal<br>Added Turkish localization|
| 2012-11-27  |Corrected callback format|
| 2012-10-30  |Added personal number<br>Enhanced MRZ check configuration|
| 2012-10-03  |Added client FAQs<br>Updated "showIntroductionText" parameter|
| 2012-08-23  |Improved user experience of landing page|
| 2012-08-17  |Introduced RESTful performNetverify and initiateNetverify APIs|
| 2012-07-18  |Added ID Verification redirect settings<br>Updated callback images|
| 2012-06-27  |Added header and footer image dimensions<br>Updated JS link for ID Verification embedded|
| 2012-06-15  |Introduced ID Verification redirect|
| 2012-05-30  |Added "idScanImageBackside" callback parameter|
| 2011-03-06  |Introduced "showIntroductionText" parameter|
| 2011-03-01  |Initial release|
<!---

--->
---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
