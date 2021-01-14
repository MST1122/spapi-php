# SPAPI
spapi-php is a http client for Amazon's Selling Partner API

Author: Lyubomir Slavilov

## Before you start
Spapi is released as Composer package `double-break/spapi-php` with **no warranties or promises**.

There are couple classes (such as `Signer`, `Credentials`) which are esential for the API calls to work. Most of the issues will live here.

All of the actual API clients are autogenerated and given the disclamer above - not heavely tested.

### Requirements
 - php 7.3 (never tested on lower version)
 - composer
 - a lot of time reading Amazon SP API documentation

## Install and update
Now you are ready to do
```bash
composer require double-break/spapi-php
```

### Package updates
Once you have successfully installed the package, the updates are simple as normal Composer package updates. Just execute:
```bash
composer update double-break/spapi-php
```
## Configuration

Name  | Description  |  Type
--|---|--
`http`  | Contains all the `Guzzle` configuration  |  GuzzleConfiguration
**LWA configuration** | |
`refresh_token` | Value of the refresh token issued by the Seller authorizing the application | string
`client_id` | The client id which is generated in the Seller Apps console | string
`client_secret`  | The client secret with which the client will identify itself  |  string
`access_token_longevity`  | The longevity in seconds of the token. It is basically the time period in which the token will be kept in the `TokenStorage`  |  integer<br/>Default: 3600
**STS configuration**  |   |  
`access_key`  | The IAM role access key  |  string
`secret_key`  | The IAM role secret key  |  string
`role_arn`  | The ARN of the IAM role  |  string
`sts_session _longevity`  |  The longevity of the STS session which will be created |  integer <br />Default: 3600
**API configuration**  |   |  
`region`  | The region identifier for the region you are going to execute your calls against  |  string <br /> Example: `eu-west-1`
`host`  | The region specific host of the Selling Partner API  |  string <br /> Example: `sellingpartnerapi-eu.amazon.com`



## Examples
Simple use
```php
<?php
  include __DIR__ . '/vendor/autoload.php';


  /** The Setup **/

  $config = [
    //Guzzle configuration
    'http' => [
      'verify' => false,
      'debug' => true
    ],

    //LWA: Keys needed to obtain access token from Login With Amazon Service
    'refresh_token' => '<YOUR-REFRESH-TOKEN>',
    'client_id' => '<CLINET-ID-IN-SELLER-CONSOLE>',
    'client_secret' => '<CLIENT_SECRET>',

    //STS: Keys of the IAM role which are needed to generate Secure Session
    // (a.k.a Secure token) for accessing and assuming the IAM role
    'access_key' => '<STS-ACCESS_KEY>',
    'secret_key' => '<STS-SECRET-KEY>',
    'role_arn' => '<ROLE-ARN>' ,

    //API: Actual configuration related to the SP API :)
    'region' => 'eu-west-1',
    'host' => 'sellingpartnerapi-eu.amazon.com'
  ];

  //Create token storage which will store the temporary tokens
  $tokenStorage = new DoubleBreak\Spapi\SimpleTokenStorage('./aws-tokens');

  //Create the request signer which will be automatically used to sign all of the
  //requests to the API
  $signer = new DoubleBreak\Spapi\Signer();

  //Create Credentials service and call getCredentials() to obtain
  //all the tokens needed under the hood
  $credentials = new DoubleBreak\Spapi\Credentials($tokenStorage, $signer, $config);
  $cred = $credentials->getCredentials();



  /** The application logic implementation **/


  //Create SP API Catalog client and execute one ot its REST methds.
  $catalogClinet = new DoubleBreak\Spapi\Api\Catalog($cred, $config);

  //Check the catalog info for B074Z9QH5F ASIN
  $result = $catalogClinet->getCatalogItem('B074Z9QH5F', [
    'MarketplaceId' => 'A1PA6795UKMFR9',
  ])['payload'];


  print_r($result);
```

### Feed API usage
For Feed API, user can follow [Feeds API Use Case Guide](https://github.com/amzn/selling-partner-api-docs/blob/main/guides/use-case-guides/feeds-api-use-case-guide-2020-09-04.md). 

And in this guide for step 2. Encrypt and upload the feed data: user can use below example:
```php
<?php

// content type of the feed data to be uploaded.
$contentType = 'text/xml; charset=UTF-8';

// create feed document
$feedClient = new \DoubleBreak\Spapi\Api\Feeds($cred, $config);
$response = $feedClient->createFeedDocument(["contentType" => $contentType]);
$payload = $response['payload'];

$feedContentFilePath = './testFeedDoc.xml';

$result = (new \DoubleBreak\Spapi\Helper\Feeder())->uploadFeedDocument($payload,$contentType,$feedContentFilePath);
print_r($result);
```
 
And for Step 6. Download and decrypt the feed processing report: user can use below example:
```php
<?php
$feedClient = new \DoubleBreak\Spapi\Api\Feeds($cred, $config);

// $resultFeedDocumentId: from response of getFeed() function.
$resultFeedDocumentId = 'amzn1.tortuga.3.ed4cd0d8-447b-4c22-96b5-52da8ace1207.T3YUVYPGKE9BMY';
$response = $feedClient->getFeedDocument($resultFeedDocumentId);
$payload = $response['payload'];

$result = (new \DoubleBreak\Spapi\Helper\Feeder())->downloadFeedProcessingReport($payload);
print_r($result);
```
 
