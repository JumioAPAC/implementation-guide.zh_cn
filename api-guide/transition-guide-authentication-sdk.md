# Authentication SDK Transition Guide
This document is intended as a reference manual for customers migrating __Authentication SDK__ to the new platform API.

It provides an overview of the significant changes in the new client and describes the adjustments that will be needed to be made to your implementation to use the new platform API.

## Introduction
The Jumio KYX Platform API allows you to manage your user journeys. It allows you to create and update accounts for your users, prompt them to provide data such as a photo ID and selfie, and get identity verification results in real time so you can complete their onboarding.

The KYX Platform API is user-based and highly flexible, allowing various workflows that can be easily combined into a single user journey. Each workflow defines a single transaction executing a series of specific tasks, such as data extraction and a liveness check. Multiple workflows can be executed on the same account.

The following diagram shows several possible workflows:

![Workflow Overview](images/workflow_overview.png)

| ℹ️&nbsp;&nbsp; Some functionalities described in this document might be unavailable, depending on the scope of your license with Jumio. Contact your Jumio Solutions Engineer if you have any questions.
|:----------|

### Implementation Steps
1. Account Creation
2. Account Update
3. Data Acquisition
4. Finalization

#### Workflow Overview

![Workflow Overview](images/workflow_overview.png)

## Table of Contents
- [Authentication SDK](#authentication-sdk)
  - [Setup Test Account](#setup-test-account)
  - [Authentication and Encryption](#authentication-and-encryption)
  - [Download Latest Moible SDK](#download-latest-mobile-sdk)
  - [Create New Enrolments](#create-new-enrolments)
    - [Workflow 3: ID and Identity Verification](#workflow-3-id-and-identity-verification)
    - [Workflow 6: Standalone Liveness](#workflow-6-standalone-liveness)
  - [Create an Authentication Workflow](#create-an-authentication-workflow)
    - [Workflow 9: Authentication](#workflow-9-authentication)
  - [Use Generated Token for SDK Setup](#use-generated-token-for-sdk-setup)
    - [Android](#android)
    - [iOS](#ios)
  - [Retrieving Data](#retrieving-data)

# Authentication SDK

## Setup Test Account  

Reach out to Jumio Support for creating a test account which helps to run tests and play around with the new architecture.

## Authentication and Encryption
API calls are protected using either __HTTP Basic Authentication__ or __OAuth2__.

| ⚠️&nbsp;&nbsp; Never share your API token, API secret, or OAuth2 credentials with anyone — not even Jumio Support!
|:----------|

The Account Management Service initiates the acquisition process and returns:

* either a JSON Web Token (JWT), which can be used to authenticate to the Jumio backend system to use with the SDK
* or Redirect-URLs, which can be used to upload documents using the other channels

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

### Access Token
Your OAuth2 access token is valid for 60 minutes. After the token lifetime is expired, it is necessary to [generate a new access token.](#authentication-and-encryption)

### Workflow (Transaction) Token
As soon as the workflow (transaction) starts, a 15 minutes session timeout starts. For each action performed (capture image, upload image) the session timeout will reset, and the 15 minutes will start again.

A token timeout can be configured via the [Jumio Customer Portal](https://github.com/Jumio/implementation-guides/blob/master/netverify/portal-settings.md) and can be overwritten using the API call. Within this token lifetime the token can be used to initialize the SDK, API or Web journey.

## Download Latest Mobile SDK
Download and use our latest SDK:

* [Android 3.9.2](https://github.com/Jumio/Mobile-SDK-ANDROID)
* [iOS 3.9.3](https://github.com/Jumio/Mobile-SDK-IOS)

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


## Use Generated Token for SDK Setup
After creating/updating a new account you will receive a `sdk.token` (JWT) for initializing the SDK. Use this token with your Android or iOS code.

### Android
```
try {
  sdk = JumioSDK(this)

  // Set your access token
  sdk.token = "yourAccessToken"

  // Set the dataCenter, default is US
  sdk.dataCenter = jumioDataCenter

} catch (e1: PlatformNotSupportedException) {
    // Handle exceptions here
  } catch (e2: NullPointerException) {
    // Handle exceptions here
}
```

### iOS
```
sdk = Jumio.SDK()

// Set your access token
sdk.token = "yourAccesToken"

// Set the dataCenter, default is US
sdk.dataCenter = jumioDataCenter
```

For more information on how to use the Jumio Mobile SDK please refer to our mobile guides for [iOS](https://github.com/Jumio/mobile-sdk-ios) and [Android](https://github.com/Jumio/mobile-sdk-android).

## Retrieving Data
Please refer to Implementation-Guide-Overview.pdf > [Available Retrieval APIs](api_guide.md#retrieval).

A result overview page can be found in the Customer Portal under "Workflows".

---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
