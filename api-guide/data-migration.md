# Data migration to new KYX platform
In order to move existing user data to the new platform, a migration step is necessary to import information. This step is especially important for Authentication customers, since they need to migrate their previous liveness users to Jumio’s new liveness provider iProov.

The old `scanReference` will be transitioned to the new `workflowExecutionId`. Optionally, the new `accountId` and `workflowExecutionId` can be returned via [callback](api_guide.md#callback), which is triggered for every migrated scan reference.

Once data is migrated, it can be retrieved with the [Retrieval API](api_guide.md#retrieval) the same way as a regular platform workflow.

## Table of Contents
- [Authentication and Encryption](#authentication-and-encryption)
- [Account Creation and Account Update](#account-creation-and-account-update)
  - [Request](#request-headers)
  - [Response](#response-1)
  - [Error Cases](#error-cases)
  - [Data Migration for Authentication](#data-migration-for-authentication)
- [Callback](#callback)
- [Retrieval](#rerieval)

## Authentication and Encryption
API calls are protected using HTTP Basic Authentication or OAuth2.

| ⚠️&nbsp;&nbsp; Never share your API token, API secret, or OAuth2 credentials with anyone — not even Jumio Support!
|:----------|

At the moment, your Basic Auth credentials are constructed using your API token as the User ID and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under:
* __Settings > API credentials > API Users__

### OAuth2
Your new OAuth2 credentials are constructed using your API token as the Client ID and your API secret as the Client secret. You can view and manage your API token and secret in the Customer Portal under:
* __Settings > API credentials > OAuth2 Clients__

Client ID and Client secret are used to generate an OAuth2 access token. OAuth2 has to be activated for your account. Contact your Jumio Account Manager for activation.

#### Access Token URL (OAuth2)
* US: `https://auth.amer-1.jumio.ai/oauth2/token`
* EU: `https://auth.emea-1.jumio.ai/oauth2/token`
* SG: `https://auth.apac-1.jumio.ai/oauth2/token`

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [supported cipher suites](https://github.com/Jumio/implementation-guides/blob/master/netverify/supported-cipher-suites.md).

| ℹ️&nbsp;&nbsp; Calls with missing, incorrect or suspicious headers or parameter values will result in HTTP status code __400 Bad Request Error__ or __403 Forbidden__
|:----------|

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

## Access Token Timeout
Your OAuth2 access token is valid for 60 minutes. After the token lifetime is expired, it is necessary to [generate a new access token.](#authentication-and-encryption)

## Account Creation and Account Update
Create a new account for your end-user by using the following API endpoint, request method POST and below mentioned request headers and request body. If an account already exists, use request method PUT and below mentioned request headers and request body to add scan references to the account.

| ℹ️&nbsp;&nbsp; Make sure to only add related transactions to the user account.
|:----------|

### Account Creation
HTTP Request Method: __POST__
* US: `https://account.amer-1.jumio.ai/api/v1/migrations/accounts`
* EU: `https://account.emea-1.jumio.ai/api/v1/migrations/accounts`
* SG: `https://account.apac-1.jumio.ai/api/v1/migrations/accounts`

### Account Update
HTTP Request Method: __PUT__
* US: `https://account.amer-1.jumio.ai/api/v1/migrations/accounts/<accountId>`
* EU: `https://account.emea-1.jumio.ai/api/v1/migrations/accounts/<accountId>`
* SG: `https://account.apac-1.jumio.ai/api/v1/migrations/accounts/<accountId>`

### Request Headers
The following fields are required in the header section of your Request

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

(Mandatory parameters in bold.)

| Parameter                      | Type           |  Notes          |
|--------------------------------|-----------------------|----------------|
| __scanReferences__  | array (string)    | Migrate one or multiple scan references to the new API<br><br>Old `scanReference` => new `workflowExecutionId`|
| callbackUrl  | string   | Sends verification result to this URL upon completion.<br><br>One callback sent per scan reference that is migrated.<br><br>[Overrides callback URL](https://github.com/Jumio/implementation-guides/blob/master/netverify/netverify-web-v4.md#callback-error-and-success-urls) in the Customer Portal.|

### Response

| Parameter                     | Type           | Notes                                                                     |
|-------------------------------|----------------|---------------------------------------------------------------------------|
| account                     | object         | Possible values:<br>• account.id      |
| account.id                       | string         | UUID of the account                                         |


### Examples

#### Request

```
curl --location --request PUT 'https://account.amer-1.jumio.ai/api/v1/accounts/<accountId>' \
--header 'Content-Type: application/json' \
--header 'User-Agent: User Demo' \
--header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
--data-raw '{
    "scanReferences": [
        "11111111-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "22222222-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    ],
    "callbackUrl": "https://www.yourcompany.com/callback"
}
```

#### Response
```
{
    "account": {
        "id": "11111111-1111-1111-1111-aaaaaaaaaaaa"
    }
}

```
### Error Cases
| Error message                 | Error Reason                    |
|-------------------------------|---------------------------------|
| Scans [`scan_reference`] are wiped out      | Scan has been deleted       |
| Account [`account_id`] not found | Account could not be found             |
| Scan references [`scan_reference`] not found | Scan could be not found    |
| Scans [`scan_reference`] not in final state | Scan has not been processed yet |
| Scan references [`scan_reference`] already migrated | Scan has already been migrated to the new platform |
| Scan references [`scan_reference`] are in progress or already migrated | Scan is already in the process of migrating or has been migrated to the new platform |

### Data Migration for Authentication
An [Authentication workflow](https://github.com/Jumio/implementation-guides-develop/blob/master/api-guide/workflow_descriptions.md#workflow-9-authentication) compares the facemap of a user to an already existing facemap, which has been captured beforehand. An Authentication migration call contains one or more scan references.

If iProov wasn’t the initial liveness provider, only transactions with the following preconditions can be migrated:
* `verificationStatus` = `APPROVED_VERIFIED`
* `validity` = `TRUE`
* `similarity` = `MATCH`

## Callback
See Implementation Guide > [Callback](api_guide.md#callback).

## Retrieval
See Implementation Guide > [Retrieval](api_guide.md#retrieval).

---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
