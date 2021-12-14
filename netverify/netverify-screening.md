![Jumio](/images/screening/Jumio-Screening-Banner.png)

# ID Verification & ComplyAdvantage Screening Implementation Guide

This is a reference manual and configuration guide for the ComplyAdvantage PEPs/Sanctions Screening integration with ID Verification. It describes how to request a ComplyAdvantage account through Jumio, creating API keys through the ComplyAdvantage portal, setting up screening in the customer portal, and changes to the ID Verification APIs to include screening.

## Table of contents

- [Requesting a Jumio Screening Account](#Requesting-a-Jumio-Screening-Account)
- [ComplyAdvantage portal setup](#ComplyAdvantage-portal-setup)
	- [Creating API keys](#Creating-API-keys)
	- [Creating a screening profile](#Creating-a-screening-profile)
- [Customer portal setup](#customer-portal-setup)
- [Include screening in ID Verification API](#Include-screening-in-id-verification-API)
	- [Web screening](#Web-screening)
	- [API screening](#API-screening)
	- [Android SDK screening](https://github.com/Jumio/mobile-sdk-android/blob/master/docs/integration_id-verification-fastfill.md#jumio-watchlist-screening)
	- [iOS SDK screening](https://github.com/Jumio/mobile-sdk-ios/blob/master/docs/integration_id-verification-fastfill.md#jumio-watchlist-screening)
- [How to review results](#How-to-review-results)
	- [Callback with screening](#Callback-with-screening)
	- [Retrieval API with screening](#Retrieval-API-with-screening)
	- [Ongoing Monitoring](#Ongoing-Monitoring)

____

# Requesting a Jumio Screening Account

If you are interested in including PEPs/Sanctions screening with your ID Verification package, please contact your Jumio Account Manager.

When your discussions have concluded, your Account Manager will have your ComplyAdvantage account created.

# ComplyAdvantage portal setup

Once ComplyAdvantage creates your account, you'll receive a verification email, with a link, which will grant you access to your ComplyAdvantage customer portal.

The ComplyAdvantage customer portal will allow you to **update new users**, **manage API keys** and **manage your watchlist**.

## Creating API keys

API keys are used to allow access to ComplyAdvantage's APIs and to run searches. You can create as many as you want from the ComplyAdvantage portal and they can be deleted at any time, as an admin user.

Once you've logged into ComplyAdvantage's portal you will have a menu on the left side.

Under **API**, select **API Keys**

From this page, you will be able to manage all of the API keys that are associated with your ComplyAdvantage account.

To create a new API key, enter a **New Key Label** and click *Create*.

## Creating a screening profile

Next, you'll need to create a screening profile. Profiles are a collection of the various lists that you want to search against. You can choose which list you want to include in a profile based on your business. Examples could be based on geography or risk levels. There is no limit to the number of lists you can create.

(link to something to help them pick list)

To begin, select **Search Profiles** under **Settings** from the main page menu.

You can manage all of your search profiles from this page.

To create a new profile, click **Add new search profile**.

On this page, you will be asked to

- Select search profile ID
- Select source list by type and/or location
- Select group

## Creating ID Verification Tag

For full functionality, you will want to enable the ID Verification tags. To do so, you will have to create a tag within the ComplyAdvantage portal.

From the main portal, you will need to navigate to Tags, under Settings on the left menu bar.

Enter "netverify-link" as the name of the tag and select "Tag: Name combination" for Case Management Display.

Save the new tag.

# Customer portal setup

After you finished setting up your ComplyAdvantage portal, you will need to setup your ID Verification portal for screening also.

After logging into your ID Verification portal, go to **Settings**.

Under **Data settings**, click on the **Watchlist Screening** tab. If this tab is not visible, please contact Jumio Support.

You will be required to enter your **API Key** and have the option to define a **default search profile**.

![Jumio](/images/screening/screening_06.jpg)

Another optional setting is the **Search enabled for approved IDs**.

If enabled, this setting will make it so all transactions that are approved will be screened. Rejected documents will never be screened.

If you do not define the profile to screen against within the API call, transactions will be screened against the default profile which you can define within the portal.

Finally, you will want to turn on

# Include screening in ID Verification API

Finally, depending on which integrations you use, you will need to update your ID Verification APIs for all or select transactions which you want screening to occur.

If you've set your screening to search **if approved by default** within the portal, you do not have to make any changes to the API if you want all *Approved/Verified* transactions to be screened.

However, the portal setting can be overwritten in the API calls by including additional parameters.

## Web screening

The `additionalChecks` field must be included within the `presets` of the **ID Verification Web v4** API call with the following, under `watchlistScreening`.

Additionally, `userReference` will be used to associate Jumio Screening with ID Verification transactions and **must be included and populated** in API calls.

More information regarding the ID Verification Web v4 can be found here:
[ID Verification Web Implementation Guide](https://github.com/Jumio/implementation-guides/blob/master/netverify/netverify-web-v4.md)

|Field Name		| Values	|
|:--				|:--		|
|enabledSearch <br> **optional**	| true <br> false|
|searchProfile <br> **optional**| string <br> *Name of profile in ComplyAdvantage*|

**Sample Request Body**

~~~
{
 "customerInternalReference": "transaction_1234",
 "userReference": "user_1234",
 "workflowId": 100,
 "presets": [
     {
       "index": 1,
       "country": "USA",
       "type": "DRIVING_LICENSE",
       "additionalChecks": {
          "watchlistScreening": {
              "enableSearch": "true",
              "searchProfile": "sanctions-only"
          }
       }
     }
 ],
 "locale": "en-GB"
}
~~~


## API screening

The `additionalChecks` field must be included within the body of the **performNetverify** API call with the following, under `watchlistScreening`.

Additionally, `customerId` will be used to associate Jumio Screening to ID Verification transactions and **must be included and populated** in API calls.

More information regarding the ID Verification API can be found here:
[ID Verification API Implementation Guide](https://github.com/Jumio/implementation-guides/blob/master/netverify/performNetverify.md)


|Field Name		| Values	|
|:--				|:--		|
|enabledSearch <br> **optional**	| true <br> false|
|searchProfile <br> **optional**| string <br> *Name of profile in ComplyAdvantage*|

**Sample Request Body**

~~~
{
   "merchantIdScanReference": "YOURSCANREFERENCE",
   "customerId": "customer_1234",
   "country": "USA",
   "idType": "DRIVING_LICENSE",
   "additionalChecks": {
      "watchlistScreening": {
           "enableSearch": "true",
           "searchProfile": "sanctions-only"
       }
   }
}
~~~

# How to review results

The results of the screening will be available from the callback, in the portal, or using the Retrieval API.

## Callback with screening

The callback will only return the results of the screening search that was done as part of the transaction. Any other searches outside the transaction, either manual or with ComplyAdvantage API, will not be returned here.

The following fields may be additional to the original callback fields, if they apply to the search that was done.

More information regarding the callback can be found here:
[Callback Implementation Guide](https://github.com/Jumio/implementation-guides/blob/master/netverify/callback.md)


|Field Name		| Values	|
|:--				|:--		|
|**searchStatus**	| DONE <br> NOT\_DONE <br> ERROR|
|searchId| string <br> *only when searchStatus = DONE*|
|searchReference | string <br> *only when searchStatus = DONE*|
|searchResults | numeric <br> *only when searchStatus = DONE*|
|searchResultsLink | string, url <br> *only when searchStatus = DONE*|
|searchDate | timestamp <br> *only when searchStatus = DONE*|

**Full URL-encoded callback example**

~~~
idCheckMicroprint=OK&callBackType=NETVERIFYID&merchantIdScanReference=feb924fe-5377-4493-b660-8dc246c0a07a&idCheckDocumentValidation=OK&idLastName=MUSTERMANN&callbackDate=2018-10-25T13%3A43%3A21.131Z&idNumber=12345678&idCheckDataPositions=OK&merchantReportingCriteria=QA&customerId=98e59b44-aadf-4395-9982-36d07834f22c&idFirstName=KAL+UWE&firstAttemptDate=2018-10-25T13%3A43%3A11.411Z&idScanSource=API&idCheckMRZcode=N%2FA&idScanStatus=SUCCESS&idType=DRIVING_LICENSE&jumioIdScanReference=08680580-7b0c-498d-8715-07a9fe5ad123&additionalChecks=%7B%22watchlistScreening%22%3A%7B%22searchStatus%22%3A%22DONE%22%2C%22searchDate%22%3A%222018-10-25T14%3A43%3A20Z%22%2C%22searchId%22%3A%2281536971%22%2C%22searchReference%22%3A%221540475000-PyxBXf9K%22%2C%22searchResultUrl%22%3A%22https%3A%2F%2Fapp.complyadvantage.com%2Fpublic%2Fsearch%2F1540475000-PyxBXf9K%2Faf58b6eeda81%22%2C%22searchResults%22%3A0%7D%7D&verificationStatus=APPROVED_VERIFIED&idScanImage=https%3A%2F%2Fnetverify.test33.testlab.int.jumio.com%2Frecognition%2Fv1%2Fidscan%2F08680580-7b0c-498d-8715-07a9fe5ad123%2Ffront&identityVerification=%7B%7D&idDob=1984-08-20&transactionDate=2018-10-25T13%3A43%3A11.412Z&idCountry=AUT&idCheckSignature=OK&idCheckSecurityFeatures=OK&idCheckHologram=OK
~~~


## Retrieval API with screening

The Retrieval API will only return the screening results of the search that was done as part of the transaction. Any other searches outside the transaction, either manual or with ComplyAdvantage API, will not be returned here.

The following fields may be additional to the original callback fields, if they apply to the search that was done.

|Field Name		| Values	|
|:--				|:--		|
|**searchStatus**	| DONE <br> NOT\_DONE <br> ERROR|
|searchId| string <br> *only when searchStatus = DONE*|
|searchReference | string <br> *only when searchStatus = DONE*|
|searchResults | numeric <br> *only when searchStatus = DONE*|
|searchResultsLink | string, url <br> *only when searchStatus = DONE*|
|searchDate | timestamp <br> *only when searchStatus = DONE*|

**Full Retrieval API Response Body**

~~~
{
    "timestamp": "2018-10-25T13:53:17.050Z",
    "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "document": {
        "type": "DRIVING_LICENSE",
        "dob": "1984-08-20",
        "firstName": "",
        "issuingCountry": "USA",
        "lastName": "",
        "number": "12345678",
        "status": "APPROVED_VERIFIED"
    },
    "transaction": {
        "customerId": "customer_1234",
        "date": "2018-10-25T13:43:11.412Z",
        "merchantScanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "source": "API",
        "status": "DONE"
    },
    "verification": {
        "additionalChecks": {
            "watchlistScreening": {
                "searchDate": "2018-10-25T14:43:20Z",
                "searchId": "12345678",
                "searchReference": "xxxxxxxxxx-xxxxxxxx",
                "searchResultUrl": "https://app.complyadvantage.com/public/search/xxxxxxxxxx-xxxxxxxx/12345678",
                "searchResults": "0",
                "searchStatus": "DONE"
            }
        },
        "identityVerification": null,
        "mrzCheck": "NOT_AVAILABLE"
    }
}
~~~

**Full Retrieval API Response Body without search**

~~~
{
  "timestamp": "2018-11-05T14:28:38.899Z",
  "scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "document": {
    "type": "PASSPORT",
    "dob": "1956-10-11",
    "expiry": "2025-05-06",
    "firstName": "",
    "issuingCountry": "USA",
    "lastName": "",
    "number": "P12345678",
    "personalNumber": "123456789",
    "status": "APPROVED_VERIFIED"
  },
  "transaction": {
    "date": "2018-11-05T14:28:17.954Z",
    "merchantScanReference": "",
    "source": "API",
    "status": "DONE"
  },
  "verification": {
    "additionalChecks": {
      "watchlistScreening": {
        "searchStatus": "NOT_DONE"
      }
    },
    "identityVerification": null,
    "mrzCheck": "OK"
  }
}
~~~

## Ongoing Monitoring

This is an additional service that is offered that will notify you if your search appears in a list **after** the original search was executed.

For more information about including this in your contract, please contact your **Jumio Account Manager**.

If you have **Ongoing Monitoring** included in your contract, you have to turn on *'Auto monitor'* in your ComplyAdvantage portal, additional settings can be found there as well.

[ComplyAdvantage](https://app.complyadvantage.com/#/settings/monitor)

When monitoring is turned on, the transactions which have monitoring will be searched against the database and list each night. If a new entry is found, the admin users from the ComplyAdvantage portal will be notified through email.

---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
