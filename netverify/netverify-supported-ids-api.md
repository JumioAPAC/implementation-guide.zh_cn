![Jumio](/images/Jumio-ID-Verification-Banner.png)

# Global ID Verification APIs

## Table of contents
- [Supported and accepted ID documents](#supported-and-accepted-id-documents)
  - [Authentication and encryption](#authentication-and-encyption)
  - [Request headers](#request-headers)
  - [Supported IDs](#supported-ids)
  - [Accepted IDs](#accepted-ids)

---

# Supported and accepted ID documents
Call the RESTful API GET endpoints described below to request a list of the ID documents currently supported by Jumio, or to request a list of the ID documents currently configured by you as accepted ID documents. You can view and manage your accepted ID documents in the Customer Portal under **Settings** > **[Accepted IDs](/netverify/portal-settings.md#accepted-ids)**.
<br>

## Authentication and encryption
ID Verification API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings** > **API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md).

<br>

## Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

## Supported IDs

Call the RESTful HTTP GET API endpoint /**supportedIdTypes** to receive a JSON response including a list of all IDs supported for verification by Jumio.

**HTTP Request Method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/supportedIdTypes`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/supportedIdTypes`<br>
**REST URL (SGP)**: `https://core-sgp.jumio.com/api/netverify/v2/supportedIdTypes`<br>

<br>

### Response Parameters

**Required items appear in bold type.**

|Parameter|Type|Description|
|:----|:----|:----|
|**supportedIdTypes**|array|IDs supported for verification by Jumio.|
|**timestamp**|string|Timestamp of the response. <br> Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|

<br>

#### Parameter `supportedIdTypes`

|Parameter|Type|Max. length|Description|
|:----|:----|:----|:----|
|**countryName**|string|100|Possible names:<br>•	[ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1) country name<br>•	Kosovo|
|**countryCode**|string|3|Possible codes:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	`XKX` (Kosovo)|
|**idTypes**|array||Supported ID types for this country|

<br>

#### Parameter `idTypes`

|Parameter|Type|Max. length|Description|
|:----|:----|:----|:----|
|**idType**|string|255|Possible values:<br>•	`PASSPORT`<br>•	`DRIVING_LICENSE`<br>•	`ID_CARD`|
|**acquisitionConfig**|object||Acquisition configuration for this ID type|

<br>

#### Parameter `acquisitionConfig`

|Parameter|Type|Max. length|Description|
|:----|:----|:----|:----|
|**backSide**|Boolean||back side required: `true`<br> back side not required: `false`|

<br>


### Sample response
```
{
    "timestamp": "2019-03-01T14:13:38.938Z",
    "supportedIdTypes": [
        {
            "countryCode": "NLD",
            "countryName": "Netherlands",
            "idTypes": [
                {
                    "acquisitionConfig": {
                        "backSide": true
                    },
                    "idType": "ID_CARD"
                },
                {
                    "acquisitionConfig": {
                        "backSide": true
                    },
                    "idType": "DRIVING_LICENSE"
                },
                {
                    "acquisitionConfig": {
                        "backSide": false
                    },
                    "idType": "PASSPORT"
                }
            ]
        },
        {
            "countryCode": "FRO",
            "countryName": "Faroe Islands",
            "idTypes": [
                {
                    "acquisitionConfig": {
                        "backSide": false
                    },
                    "idType": "DRIVING_LICENSE"
                },
                {
                    "acquisitionConfig": {
                        "backSide": false
                    },
                    "idType": "PASSPORT"
                }
            ]
        }  
    ]
}
```

<br>

## Accepted IDs

Call the RESTful HTTP GET API endpoint /**acceptedIdTypes** to receive a JSON response including all [Accepted IDs](/netverify/portal-settings.md#accepted-ids) specified in your Customer Portal settings.

**HTTP Request Method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/acceptedIdTypes`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/acceptedIdTypes`<br>
**REST URL (SGP)**: `https://core-sgp.jumio.com/api/netverify/v2/acceptedIdTypes`<br>

<br>

### Response Parameters

**Required items appear in bold type.**

|Parameter|Type|Description|
|:----|:----|:----|
|**supportedIdTypes**|array|IDs you accept for verification.|
|**timestamp**|string|Timestamp of the response. <br> Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|

<br>

#### Parameter `acceptedIdTypes`

|Parameter|Type|Max. length|Description|
|:----|:----|:----|:----|
|**countryName**|string|100|Possible names:<br>•	[ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1) country name<br>•	Kosovo|
|**countryCode**|string|3|Possible codes:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	`XKX` (Kosovo)|
|**idTypes**|array||ID types you accept from this country.|

<br>

#### Parameter `idTypes`

|Parameter|Type|Max. length|Description|
|:----|:----|:----|:----|
|**idType**|string|255|Possible values:<br>•	`PASSPORT`<br>•	`DRIVING_LICENSE`<br>•	`ID_CARD`|
|**acquisitionConfig**|object||Acquisition configuration for this ID type.|

<br>

#### Parameter `acquisitionConfig`

|Parameter|Type|Max. length|Description|
|:----|:----|:----|:----|
|**backSide**|Boolean||back side required: `true`<br> back side not required: `false`|

<br>


### Sample response

```
{
    "timestamp": "2019-03-26T14:16:14.421Z",
    "acceptedIdTypes": [
        {
            "countryCode": "GAB",
            "countryName": "Gabon",
            "idTypes": [
                {
                    "acquisitionConfig": {
                        "backSide": false
                    },
                    "idType": "PASSPORT"
                }
            ]
        },
        {
            "countryCode": "SGP",
            "countryName": "Singapore",
            "idTypes": [
                {
                    "acquisitionConfig": {
                        "backSide": true
                    },
                    "idType": "ID_CARD"
                },
                {
                    "acquisitionConfig": {
                        "backSide": true
                    },
                    "idType": "DRIVING_LICENSE"
                },
                {
                    "acquisitionConfig": {
                        "backSide": false
                    },
                    "idType": "PASSPORT"
                }
            ]
        }
    ]
}
```


---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
