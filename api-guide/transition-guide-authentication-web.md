# Authentication Web Transition Guide
This document is intended as a reference manual for customers migrating __Authentication Web__ to the new platform API.

It provides an overview of the significant changes in the new client and describes the adjustments that will be needed to be made to your implementation to use the new platform API.

## Introduction
The Jumio KYX Platform API allows you to manage your user journeys. It allows you to create and update accounts for your users, prompt them to provide data such as a photo ID and selfie, and get identity verification results in real time so you can complete their onboarding.

The KYX Platform API is user-based and highly flexible, allowing various workflows that can be easily combined into a single user journey. Each workflow defines a single transaction executing a series of specific tasks, such as data extraction and a liveness check. Multiple workflows can be executed on the same account.

The following diagram shows several possible workflows:

![Workflow Overview](images/workflow_overview.png)

### Implementation Steps
1. Account Creation
2. Account Update
3. Data Acquisition
4. Finalization

## Table of Contents
- [Authentication Web](#authentication-web)
  - [Setup Test Account](#setup-test-account)
  - [Authentication and Encryption](#authentication-and-encryption)
  - [Create New Enrolments](#create-new-enrolments)
    - [Workflow 3: ID and Identity Verification](#workflow-3-id-and-identity-verification)
    - [Workflow 6: Standalone Liveness](#workflow-6-standalone-liveness)
  - [Displaying the Client](#displaying-the-client)
  - [Create an Authentication Workflow](#create-an-authentication-workflow)
    - [Workflow 9: Authentication](#workflow-9-authentication)
  - [Displaying Web Client](#displaying-web-client)
  - [Retrieving Data](#retrieving-data)

# Authentication Web

## Setup Test Account  
Reach out to Jumio Support for creating a test account which helps to run tests and play around with the new architecture.

## Authentication and Encryption
API calls are protected using either __HTTP Basic Authentication__ or __OAuth2__.

| ⚠️&nbsp;&nbsp; Never share your API token, API secret, or OAuth2 credentials with anyone — not even Jumio Support!
|:----------|

The Account Management Service initiates the acquisition process and returns:

* either a JSON Web Token (JWT), which can be used to authenticate to the Jumio backend system to use with the SDK
* or Redirect-URLs, which can be used to upload documents using the other channels

| ℹ️&nbsp;&nbsp; Some functionalities described in this document might be unavailable, depending on the scope of your license with Jumio. Contact your Jumio Solutions Engineer if you have any questions.
|:----------|

At the moment, your Basic Auth credentials are constructed using your API token as the User ID and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under:
* __Settings > API credentials > API Users__

## OAuth2
Your new OAuth2 credentials are constructed using your API token as the Client ID and your API secret as the Client secret. You can view and manage your API token and secret in the Customer Portal under:
* __Settings > API credentials > OAuth2 Clients__

Client ID and Client secret are used to generate an OAuth2 access token. OAuth2 has to be activated for your account. Contact your Jumio Account Manager for activation.

### Access Token URL (OAuth2)
* US: `https://auth.amer-1.jumio.ai/oauth2/token`
* EU: `https://auth.emea-1.jumio.ai/oauth2/token`
* SG: `https://auth.apac-1.jumio.ai/oauth2/token`

__Note:__ Never share your API token, API secret, or OAuth2 credentials with anyone — not even Jumio Support!

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [supported cipher suites](https://github.com/Jumio/implementation-guides/blob/master/netverify/supported-cipher-suites.md).

__Note:__ Calls with missing, incorrect or suspicious headers or parameter values will result in HTTP status code __400 Bad Request Error__ or __403 Forbidden__.

### Examples

#### Request Access Token
```
curl --request POST --location 'https://auth.amer-1.jumio.ai/oauth2/token' \
    --header 'Accept: application/json' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --data-raw 'grant_type=client_credentials' \
    --basic --user CLIENT_ID:CLIENT_SECRET
```

#### Response
```
{
  "access_token": "YOUR_ACCESS_TOKEN",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

### Access Token Timeout
Your OAuth2 access token is valid for 60 minutes. After the token lifetime is expired, it is necessary to [generate a new access token.](#authentication-and-encryption)

### Workflow (Transaction) Token
As soon as the workflow (transaction) starts, a 15 minutes session timeout starts. For each action performed (capture image, upload image) the session timeout will reset, and the 15 minutes will start again.

A token timeout can be configured via the [Jumio Customer Portal](https://github.com/Jumio/implementation-guides/blob/master/netverify/portal-settings.md) and can be overwritten using the API call. Within this token lifetime the token can be used to initialize the SDK, API or Web journey.

## Create New Enrolments
Currently, only new enrolments that are created in the new architecture can be used for the Next-Generation Authentication.

Use one of the following workflows to enroll a new transaction. Create a new account for a new end-user or use the update account API if the end-user already exists.

* Workflow 3: ID and Identity Verification
* Workflow 6: Standalone Liveness

### Workflow 3: ID and Identity Verification
This workflow verifies a photo ID document and returns a) whether that document is valid, and b) data extracted from that document. It also compares the user's face with the photo on the ID and performs a liveness check to ensure the person is physically present.

![Workflow 3 Overview](images/workflow_3.png)

#### Required Credentials
* ID
* Selfie
* Facemap

#### Stored Credentials
* ID
* Selfie
* Facemap

#### Prerequisites
* Identity Verification must be enabled for your account. (Contact your Jumio account manager for activation.)
* Allowed channels: SDK, WEB

#### Example

##### Initiate Account Creation

HTTP Request Method: __POST__
* US: `https://account.amer-1.jumio.ai/api/v1/accounts`
* EU: `https://account.emea-1.jumio.ai/api/v1/accounts`
* SG: `https://account.apac-1.jumio.ai/api/v1/accounts`

__Request:__
```
curl --request POST --location 'https://account.amer-1.jumio.ai/api/v1/accounts' \
    --header 'Content-Type: application/json' \
    --header 'User-Agent: User Demo' \
    --header 'Authorization: Bearer
    YOUR_ACCESS_TOKEN' \
    --data-raw '{
        "customerInternalReference": "CUSTOMER_REFERENCE",
        "workflowDefinition": {
            "key": 3,
            "credentials": [
                {
                    "category": "ID",
                    "type": {
                        "values": ["DRIVING_LICENSE", "ID_CARD", "PASSPORT"]
                    },
                    "country": {
                        "values": ["USA", "CAN", "AUT", "GBR"]
                    }
                }
            ]
        },
        "callbackUrl": "YOUR_CALLBACK_URL",
        "userReference": "YOUR_USER_REFERENCE",
    }'
```

__Response:__
```
{
    "timestamp": "2021-05-19T14:55:08.187Z",
    "account": {
        "id": "11111111-1111-1111-1111-aaaaaaaaaaaa"
    },
    "web": {
        "href": "https://mycompany.web.amer-1.jumio.ai/web/v4/app?authorizationToken=xxx&locale=es"
    },
    "sdk": {
        "token": "xxx"
    },
    "workflowExecution": {
        "id": "22222222-2222-2222-2222-aaaaaaaaaaaa",
        "credentials": [
            {
                "id": "33333333-3333-3333-aaaaaaaaaaaa",
                "category": "ID",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            },
            {
                "id": "33333333-3333-3333-bbbbbbbbbbbb",
                "category": "FACEMAP",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            },
            {
                "id": "33333333-3333-3333-cccccccccccc",
                "category": "SELFIE",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            }
        ]
    }
}
```

##### Initiate Account Update

HTTP Request Method: __PUT__
* US: `https://account.amer-1.jumio.ai/api/v1/accounts/<accountId>`
* EU: `https://account.emea-1.jumio.ai/api/v1/accounts/<accountId>`
* SG: `https://account.apac-1.jumio.ai/api/v1/accounts/<accountId>`

__Request:__
```
curl --request POST --location 'https://account.amer-1.jumio.ai/api/v1/accounts/11111111-1111-1111-1111-aaaaaaaaaaaa' \
    --header 'Content-Type: application/json' \
    --header 'User-Agent: User Demo' \
    --header 'Authorization: Bearer
    YOUR_ACCESS_TOKEN' \
    --data-raw '{
        "customerInternalReference": "CUSTOMER_REFERENCE",
        "workflowDefinition": {
            "key": 3,
            "credentials": [
                {
                    "category": "ID",
                    "type": {
                        "values": ["DRIVING_LICENSE", "ID_CARD", "PASSPORT"]
                    },
                    "country": {
                        "values": ["USA", "CAN", "AUT", "GBR"]
                    }
                }
            ]
        },
        "callbackUrl": "YOUR_CALLBACK_URL",
        "userReference": "YOUR_USER_REFERENCE",
    }'
```

__Response:__
```
{
    "timestamp": "2021-05-19T14:55:08.187Z",
    "account": {
        "id": "11111111-1111-1111-1111-aaaaaaaaaaaa"
    },
    "web": {
        "href": "https://mycompany.web.amer-1.jumio.ai/web/v4/app?authorizationToken=xxx&locale=es"
    },
    "sdk": {
        "token": "xxx"
    },
    "workflowExecution": {
        "id": "22222222-2222-2222-2222-aaaaaaaaaaaa",
        "credentials": [
            {
                "id": "33333333-3333-3333-aaaaaaaaaaaa",
                "category": "ID",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            },
            {
                "id": "33333333-3333-3333-bbbbbbbbbbbb",
                "category": "FACEMAP",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            },
            {
                "id": "33333333-3333-3333-cccccccccccc",
                "category": "SELFIE",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            }
        ]
    }
}
```

### Workflow 6: Standalone Liveness
This workflow captures a user's face to verify that the person is physically present and not presenting a photo or other fake as their selfie.

![Workflow 6 Overview](images/workflow_6.png)

#### Required Credentials
* Selfie
* Facemap

#### Stored Credentials
* Selfie
* Facemap

### Prerequisites
* Identity Verification must be enabled for your account. (Contact your Jumio account manager for activation.)
* Allowed channels: SDK, WEB

#### Example

##### Initiate Account Creation

HTTP Request Method: __POST__
* US: `https://account.amer-1.jumio.ai/api/v1/accounts`
* EU: `https://account.emea-1.jumio.ai/api/v1/accounts`
* SG: `https://account.apac-1.jumio.ai/api/v1/accounts`

__Request:__
```
curl --location --request PUT 'https://account.amer-1.jumio.ai/api/v1/accounts' \
--header 'Content-Type: application/json' \
--header 'User-Agent: User Demo' \
--header 'Authorization: Bearer xxx' \
--data-raw '{
    "customerInternalReference": "CUSTOMER_REFERENCE",
    "workflowDefinition": {
        "key": 6,
        "credentials": []
    }
}'
```

__Response:__
```
{
    "timestamp": "2021-05-19T15:11:18.072Z",
    "account": {
        "id": "11111111-1111-1111-1111-aaaaaaaaaaaa"
    },
    "web": {
        "href": "https://mycompany.web.amer-1.jumio.ai/web/v4/app?authorizationToken=xxx&locale=es"
    },
    "sdk": {
        "token": "xxx"
    },
    "workflowExecution": {
        "id": "22222222-2222-2222-2222-aaaaaaaaaaaa",
        "credentials": [
            {
                "id": "33333333-3333-3333-aaaaaaaaaaaa",
                "category": "FACEMAP",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            },
            {
                "id": "33333333-3333-3333-bbbbbbbbbbbb",
                "category": "SELFIE",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            }
        ]
    }
}
```

##### Initiate Account Update

HTTP Request Method: __PUT__
* US: `https://account.amer-1.jumio.ai/api/v1/accounts/<accountId>`
* EU: `https://account.emea-1.jumio.ai/api/v1/accounts/<accountId>`
* SG: `https://account.apac-1.jumio.ai/api/v1/accounts/<accountId>`

__Request:__
```
curl --location --request PUT 'https://account.amer-1.jumio.ai/api/v1/accounts/11111111-1111-1111-1111-aaaaaaaaaaaa' \
--header 'Content-Type: application/json' \
--header 'User-Agent: User Demo' \
--header 'Authorization: Bearer xxx' \
--data-raw '{
    "customerInternalReference": "CUSTOMER_REFERENCE",
    "workflowDefinition": {
        "key": 6,
        "credentials": []
    }
}'
```

__Response:__
```
{
    "timestamp": "2021-05-19T15:11:18.072Z",
    "account": {
        "id": "11111111-1111-1111-1111-aaaaaaaaaaaa"
    },
    "web": {
        "href": "https://mycompany.web.amer-1.jumio.ai/web/v4/app?authorizationToken=xxx&locale=es"
    },
    "sdk": {
        "token": "xxx"
    },
    "workflowExecution": {
        "id": "22222222-2222-2222-2222-aaaaaaaaaaaa",
        "credentials": [
            {
                "id": "33333333-3333-3333-aaaaaaaaaaaa",
                "category": "FACEMAP",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            },
            {
                "id": "33333333-3333-3333-bbbbbbbbbbbb",
                "category": "SELFIE",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            }
        ]
    }
}
```

## Displaying Web Client
When you initiate the web client, the API returns the `web.href` that you use to display the web client to the end user. You can provide this URL in several ways:

* within an iFrame on your web page
* as a link on your web page that opens a new browser tab or window
* as a link shared securely with an end-user
* in a native webview

### Embedding in an iFrame
No change to embed the generated `web.href` from the new architecture. See the current [Authentication for Web Implementation Guide](/netverify/netverify-authentication.md#using-authentication-in-an-iframe) for further information.

#### Width and Height
No change for recommended responsive breaking points. See the current [Authentication for Web Implementation Guide](/netverify/netverify-authentication.md#using-authentication-in-an-iframe) for further information.

#### Biometric Face Capture
For a better user experience when creating a three-dimensional map of your user's face, you must allow full screen mode. This will address the positioning and distance between the capture interface and the camera.

| ⚠️&nbsp;&nbsp; The `allow="camera;fullscreen;accelerometer;gyroscope;magnetometer" allowfullscreen` iFrame attributes must be included to enable biometric face capture in supported browsers.
|:----------|

| ⚠️&nbsp;&nbsp; In case you are nesting the iFrame in another iFrame the `allow="camera"` attribute must be added to every iFrame.
|:----------|

##### Example
```
<iframe src="https://yourcompany.netverify.com/web/v4/app?locale=en-GB&authorizationToken=xxx" width="70%" height="80%" allow="camera;fullscreen;accelerometer;gyroscope;magnetometer" allowfullscreen></iframe>
```

#### Optional iFrame logging
Setup and usage of the optional iFrame logging stays the same besides of some changes to a few properties in the `event.data` object listed below. See the current [Authentication for Web Implementation Guide](/netverify/netverify-authentication.md#optional-iframe-logging) for further information about Optional iFrame logging.

##### `event.data` object

###### New Parameters

**Required items appear in bold type.**

|New Property|Type|Description
|:-------|:---|:----------|
**accountId**|string|UUID of the account|

###### Minor Changed Parameters

**Required items appear in bold type.**

|Current Property|New Property|Type|Description
|:-------|:-------|:---|:----------|
|**transactionReference** |**workflowExecutionId**|string|Property has been renamed <br><br>UUID of the workflow|

###### Retained Parameters

**Required items appear in bold type.**  

|Property|Type|Description
|:-------|:---|:----------|
|**authorizationToken**|string|Authorization token, valid for a specified duration|
|**customerInternalReference**<sup>1</sup>|string| Internal reference for a request to link it in the customer backend (must not contain any PII |
|**eventType**|integer|Type of event that has occurred.<br>Possible values: <br>• `510` (application state-change)|
|**dateTime**|string|UTC timestamp of the event in the browser<br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|
|**payload**|JSON object|Information specific to the event generated <br>(see [`event.data.payload` object](#eventdatapayload-object))|

<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.


##### `event.data.payload` object

No change for the `event.data.payload` object.

##### `event.data.payload.metainfo` object

No change for the `event.data.payload.metainfo` object.

##### Example iFrame logging code
~~~javascript
function receiveMessage(event) {
	var data = window.JSON.parse(event.data);
  console.log('ID Verification Web was loaded in an iframe.');
  console.log('auth-token:', data.authorizationToken);
  console.log('event-type:', data.eventType);
  console.log('date-time:', data.dateTime);
  console.log('workflow-execution-id:', data.workflowExecutionId);
  console.log('account-id:', data.accountId);
  console.log('customer-internal-reference:', data.customerInternalReference);
  console.log('value:', data.payload.value);
  console.log('metainfo:', data.payload.metainfo);
}
window.addEventListener("message", receiveMessage, false);
~~~

### Using a Native WebView
No change for embedding Web within a WebView in your native mobile application. See the current [ID Verification Implementation Guide](/netverify/netverify-web-v4.md#using-id-verification-in-a-native-webview) for further information.

## Create an Authentication Workflow

### Workflow 9: Authentication
This workflow compares the facemap of a user to an already existing facemap that has already been captured.

The existing facemap must have been acquired during a previous workflow, e.g. [Workflow 3](workflow_descriptions.md#workflow-3-id-and-identity-verification) or [Workflow 5](workflow_descriptions.md#workflow-5-similarity-to-existing-id).

#### Required Credentials
* Selfie
* Facemap

#### Stored Credentials
* Selfie
* Facemap

#### Prerequisites
* Identity Verification must be enabled for your account. (Contact your Jumio account manager for activation.)
* Account already exists, `<accountId>` available
* Facemap already on file, acquired through previous workflow
* Allowed channels: SDK, WEB

### Initiating

#### Current architecture

HTTP Request Method: __POST__     
* US: `https://netverify.com/api/authentication/v1/web/initiate`<br>
* EU: `https://lon.netverify.com/api/authentication/v1/web/initiate`<br>
* SG: `https://core-sgp.jumio.com/api/authentication/v1/web/initiate`<br>

#### New architecture (to initiate all transactions)

HTTP Request Method: __PUT__   
* US: `https://account.amer-1.jumio.ai/api/v1/accounts/<accountId>`
* EU: `https://account.emea-1.jumio.ai/api/v1/accounts/<accountId>`
* SG: `https://account.apac-1.jumio.ai/api/v1/accounts/<accountId>`

### Request Headers
The request headers stay the same besides of the preferred Authorization method (Basic Authentication / OAuth2).

The following fields are required in the header section of your request:

`Accept: application/json`   
`Content-Type: application/json`    
`Content-Length:`  see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2)   
`Authorization:` see [RFC6749](https://tools.ietf.org/html/rfc6749)   
`User-Agent: YourCompany YourApp/v1.0`   

| ⚠️&nbsp;&nbsp; Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.
|:----------|

| ℹ️&nbsp;&nbsp; Calls with missing or suspicious headers, suspicious parameter values, or without OAuth2 will result in HTTP status code __403 Forbidden__
|:----------|

### Request Body
With this new architecture, we have begun the process of improving consistency across all of our APIs.

Some of the initiate API request fields have been retained, but have new names. We have also deprecated some fields and added a few new ones. Our changes are described below.

#### New Parameters

__Required fields appear in bold type.__

| Parameter                     | Type |  Notes |
|:-------------------------------|:---|:---|
| __workflowDefinition__         |object |Definition of the specific documents necessary to execute for the particular capabilities on them.|
| __workflowDefinition.key__     |object |Key of the workflow definition which you want to execute.<br>See [Workflow Definition Keys](#workflow-definition-keys)|
| workflowDefinition.credentials |array (object) |Optional workflow definition object part to customize acquiring process and workflow process.<br>Possible values: <br>See [workflowDefinition.credentials](#response-workflowdefinitioncredentials)|
| web |object|Web parameters are only relevant for the WEB channel.|

#### Request workflowDefinition.credentials

| Parameter              | Type           | Max. Length          | Notes                                                                     |
|------------------------|----------------|----------------------|---------------------------------------------------------------------------|
| category               | string         |                      | Possible values:<br>• ID<br>• FACEMAP<br>• DOCUMENT<br>•	SELFIE           |
| country                | object         |                      | Possible values:<br>• country.values                                      |
| country.predefinedType | string         |                      | Possible values:<br>• DEFINED (default: end user is not able to change country)<br>• RECOMMENDED (country is preselected, end user is still able to change it) |
| country.values         | array (string) | See possible values. | Define at least one ISO 3166-1 alpha-3 country code for the workflow definition.<br>Possible values: <br>•	[ISO 3166-1 alpha-3 country code](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) |
| type                   | object         |                      | Possible values:<br>•	 type.values                                       |
| type.predefinedType    | object         |                      | Possible values:<br>• DEFINED (default: end user is not able to change document type)<br>• RECOMMENDED (type is preselected, end user is still able to change it) |
| type.values            | array (string) | See possible values. | Defined number of credential type codes. <br>Possible values:<br>• ID_CARD<br>• DRIVING_LICENSE<br>• PASSPORT<br>• VISA       |

#### Minor Changed Parameters

__Required fields appear in bold type.__

| Current | New | Type | Max. Length | Changes |
|:---------|:--------|:--------|:--------|:--------|
| successUrl<sup>1</sup> | web.successUrl<sup>2</sup> | string | 255 | Parameter nested in `web` object<br><br>URL to which the browser will send the end-user at the end of a successful web acquisition user journey. |
| errorUrl<sup>1</sup> | web.errorUrl<sup>2</sup> | string | 255 |  Parameter nested in `web` object<br><br>URL to which the browser will send the end-user at the end of a failed web acquisition user journey. |
| locale<sup>1</sup> | web.locale<sup>2</sup> | string | 5 |  Parameter embedded in `web` object <br><br>Renders content in the specified language.<br><br>See [supported locale values](#supported-locale-values).|
| tokenLifetimeInMinutes | tokenLifetime | string | minimum: 5m, maximum: 60d,<br>default: 30m | Parameter has been renamed. Previously defined in minutes. <br><br> Should be a valid date period unit definition:<br>s - seconds<br>m - minutes<br>h - hours<br>d - days<br>Example: '1d' / '30m' / '600s'|
|enrollmentTransactionReference|workflowExecution.credentials.id|string||This parameter has been replaced with workflowExecution.credentials.id<br><br>The credentials ID from the onboarding ID verification to be used for authentication.|

<sup>1</sup> Overrides default setting from the Customer Portal.

##### Supported `locale` values

Parameter `web.locale` can be used to render the content of the client in the specified language.

Hyphenated combination of [ISO 639-1:2002 alpha-2](https://en.wikipedia.org/wiki/ISO_639-1) language code plus [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country (where applicable).

|Value  |Locale|
|:--------------|:--------------|
|ar|Arabic|
|bg|Bulgarian|
|cs|Czech|
|da|Danish|
|de|German|
|el|Greek|
|en|American English (**default**)|
|en-GB|British English|
|es|Spanish|
|es-MX|Mexican Spanish|
|et|Estonian|
|fi|Finnish|
|fr|French|
|he|Hebrew|
|hr|Croatian|
|hu|Hungarian|
|hy|Armenian|
|id|Indonesian|
|it|Italian|
|ja|Japanese|
|ka|Georgian|
|km|Khmer|
|ko|Korean|
|lt|Lithuanian|
|ms|Malay|
|nl|Dutch|
|no|Norwegian|
|pl|Polish|
|pt|Portuguese|
|pt-BR|Brazilian Portuguese|
|ro|Romanian|
|ru|Russian|
|sk|Slovak|
|sv|Swedish|
|th|Thai|
|tr|Turkish|
|vi|Vietnamese|
|zh-CN|Simplified Chinese|
|zh-HK|Traditional Chinese|


#### Retained Parameters

__Required fields appear in bold type.__

| Parameter | Type | Max. Length | Changes |
|:--------|:--------|:--------|:--------|
| userReference<sup>1</sup> | string | 100 | No change<br><br>Reference for the end user in the customer backend  |
| callbackUrl<sup>2</sup> | string | 255 | No change <br><br>Definition of the callback URL for this particular request|

<sup>1</sup> Values __must not__ contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>2</sup> Overrides default setting from the Customer Portal.

### Response
Unsuccessful requests will return the relevant HTTP status code and information about the cause of the error.

Successful requests will return HTTP status code __200 OK__ along with a JSON object containing the information described below.

#### New Parameters

| Parameter                     | Type | Notes |
|:-------------------------------|:-----|:-----|
| account                        |object|Possible values:<br>• account.id|
| account.id                     |string|UUID of the account|
| sdk                            |object|Possible values:<br>• sdk.token<br><br>*SDK parameters are only relevant for the SDK channel.*|
| sdk.token                      |string|JWT token for performing any action<br><br>*SDK parameters are only relevant for the SDK channel.*|
| workflowExecution              |object|Possible values:<br>• workflowExecution.id<br>• workflowExecution.credentials|
| workflowExecution.credentials  |array (object)|Credential response<br>See [workflowExecution.credentials](#response-workflowdefinition.credentials)|
| web.successUrl  |string|URL to which the browser will send the end-user at the end of a successful web acquisition user journey.|
| web.errorUrl  |string|URL to which the browser will send the end-user at the end of a failed web acquisition user journey.|

##### Response workflowDefinition.credentials
| Parameter             | Type                    | Notes                                                                                                                           |
|-----------------------|-------------------------|-----------------------------------------------------------------------------|
| id                    | string                  | UUID of the credentials                                                                                                         |
| category              | string                  | Credential category.<br>Possible values:<br>• ID<br>•	FACEMAP<br>• DOCUMENT<br>• SELFIE                                         |
| country               | object                  | Defined at least one ISO 3166-1 alpha-3 country code for the workflow definition.<br>Possible values: <br>•	[ISO 3166-1 alpha-3 country code](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3)                |
| type                  | object                  | Defined number of credential type codes.<br>Possible values: <br>• ID_CARD<br>•	DRIVING LICENSE<br>• PASSPORT<br>• VISA         |
| allowedChannels       | array                   | Channels which can be used to upload particular credential<br>Possible values:<br>• WEB<br>• API<br>•	SDK    |
| api                   | object                  | Available actions for the API calls, actions can be omitted due to unavailability<br>Possible values:<br>• api.token<br>api.parts<br>•	api.workflowExecution <br><br> _API parameters are only relevant for the API channel._ |
| api.token             | string                  | JWT token for performing any action for API<br><br>_API parameters are only relevant for the API channel._                         |
| api.parts             | object                  | href to manage parts for the account credential<br>Possible values:<br>• FRONT<br>•	BACK<br>• FACE <br><br> _API parameters are only relevant for the API channel._                |
| api.workflowExecution | string                  | href to manage the acquisition and workflow processing <br><br>_API parameters are only relevant for the API channel._

#### Minor Changed Parameters

| Current     | New   | Type | Changes |
|:-------------|:----------|:-----|:-----|
| timestamp    | timestamp |string |No change.<br>Timestamp (UTC) of the response.<br>Format: YYYY-MM-DDThh:mm:ss.SSSZ|
| redirectUrl  | web.href  |string|Parameter has been renamed.<br>URL used to load the ID Verification client.|
| transactionReference | workflowExecution.id |string|UUID of the workflow|

#### Example

##### Initiate Account Update

HTTP Request Method: __PUT__
* US: `https://account.amer-1.jumio.ai/api/v1/accounts/<accountId>`
* EU: `https://account.emea-1.jumio.ai/api/v1/accounts/<accountId>`
* SG: `https://account.apac-1.jumio.ai/api/v1/accounts/<accountId>`

__Request:__
```
curl --location --request PUT 'https://account.amer-1.jumio.ai/api/v1/accounts/11111111-1111-1111-1111-aaaaaaaaaaaa' \
--header 'Content-Type: application/json' \
--header 'User-Agent: User Demo' \
--header 'Authorization: Bearer xxx' \
--data-raw '{
    "customerInternalReference": "CUSTOMER_REFERENCE",
    "workflowDefinition": {
        "key": 9,
        "credentials": [
            {
                "id": "33333333-3333-3333-aaaaaaaaaaaa",
                "category": "ID"
            },
            {
                "category": "FACEMAP"
            }
        ]
    }
}'
```

__Response:__
```
{
    "timestamp": "2021-05-19T15:28:15.711Z",
    "account": {
        "id": "11111111-1111-1111-1111-aaaaaaaaaaaa"
    },
    "web": {
        "href": "https://mycompany.web.amer-1.jumio.ai/web/v4/app?authorizationToken=xxx&locale=es"
    },
    "sdk": {
        "token": "xxx"
    },
    "workflowExecution": {
        "id": "22222222-2222-2222-2222-aaaaaaaaaaaa",
        "credentials": [
            {
                "id": "33333333-3333-3333-bbbbbbbbbbbb",
                "category": "FACEMAP",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            },
            {
                "id": "33333333-3333-3333-cccccccccccc",
                "category": "SELFIE",
                "allowedChannels": [
                    "WEB",
                    "SDK"
                ]
            }
        ]
    }
}
```

## Displaying Web Client
See [Displaying Web Client](#displaying-web-client).

## Retrieving Data
Please refer to Implementation-Guide-Overview.pdf > [Available Retrieval APIs](api_guide.md#retrieval).

A result overview page can be found in the Customer Portal under __"Workflows"__.

---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
