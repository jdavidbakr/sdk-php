Authorize.Net PHP SDK
======================

[![Build Status] (https://travis-ci.org/AuthorizeNet/sdk-php.png?branch=master)]
(https://travis-ci.org/AuthorizeNet/sdk-php)


## License
Proprietary, see the provided `license.md`.

## Requirements

- PHP 5.3+ *(>=5.3.10 recommended)*
- cURL PHP Extension
- JSON PHP Extension
- SimpleXML PHP Extension
- An Authorize.Net Merchant Account or Sandbox Account. You can get a 
	free sandbox account at http://developer.authorize.net/sandbox/

## Autoloading

[`Composer`](http://getcomposer.org) currently has a [MITM](https://github.com/composer/composer/issues/1074)
security vulnerability.  However, if you wish to use it, require its autoloader in
your script or bootstrap file:
```php
require 'vendor/autoload.php';
```
*Note: you'll need a composer.json file with the following require section and to run
`composer update`.*
```json
"require": {
    "authorizenet/authorizenet": "~1.8"
}
```

Alternatively, we provide a custom `SPL` autoloader:
```php
require 'path/to/anet_php_sdk/autoload.php';
```

## Authentication
To authenticate with the Authorize.Net API you will need to retrieve your API Login ID and Transaction Key from the [`Merchant Interface`](https://account.authorize.net/).  You can find these details in the Settings section.
If you need a sandbox account you can sign up for one really easily [`here`](https://developer.authorize.net/sandbox/).

Once you have your keys simply plug them into the appropriate variables, as per the below code dealing with the authentication part of the flow.

````php
$merchantAuthentication = new AnetAPI\MerchantAuthenticationType();
$merchantAuthentication->setName("YOURLOGIN");
$merchantAuthentication->setTransactionKey("YOURKEY");
````
...

````php
$request = new AnetAPI\CreateTransactionRequest();
$request->setMerchantAuthentication($merchantAuthentication);
````
...

## Usage Examples

Apart from this README, you can find details and examples of using the SDK in the following places:
- [Developer Center Reference](http://developer.authorize.net/api/reference/index.html)
- [Github Sample Code Repositories](http://developer.authorize.net/api/samplecode/), [php](https://github.com/AuthorizeNet/sample-code-php)
      
### Quick Usage Example (with Charge Credit Card - Authorize and Capture)
- Save the below code to a php file named, say, `charge-credit-card.php`
- Open command prompt and navigate to your sdk folder
- Update dependecies - e.g., With composer, type `composer update`
- Type `php [<path to folder containing the php file>\]charge-credit-card.php`

```php
require 'vendor/autoload.php';
/* provide full path to the sdk in the above code, if you want to run it from some place other than the sdk folder
e.g. if sdk is in c:\anet-sdk-php,
require 'c:/anet-sdk-php/vendor/autoload.php'; */
use net\authorize\api\contract\v1 as AnetAPI;
use net\authorize\api\controller as AnetController;
define("AUTHORIZENET_LOG_FILE", "phplog");

// Common setup for API credentials
$merchantAuthentication = new AnetAPI\MerchantAuthenticationType();
$merchantAuthentication->setName("556KThWQ6vf2");
$merchantAuthentication->setTransactionKey("9ac2932kQ7kN2Wzq");

// Create the payment data for a credit card
$creditCard = new AnetAPI\CreditCardType();
$creditCard->setCardNumber( "4111111111111111" );
$creditCard->setExpirationDate( "2038-12");
$paymentOne = new AnetAPI\PaymentType();
$paymentOne->setCreditCard($creditCard);

// Create a transaction
$transactionRequestType = new AnetAPI\TransactionRequestType();
$transactionRequestType->setTransactionType( "authCaptureTransaction"); 
$transactionRequestType->setAmount(151.51);
$transactionRequestType->setPayment($paymentOne);

$request = new AnetAPI\CreateTransactionRequest();
$request->setMerchantAuthentication($merchantAuthentication);
$request->setTransactionRequest( $transactionRequestType);
$controller = new AnetController\CreateTransactionController($request);
$response = $controller->executeWithApiResponse( \net\authorize\api\constants\ANetEnvironment::SANDBOX);

if ($response != null)
{
	$tresponse = $response->getTransactionResponse();

	if (($tresponse != null) && ($tresponse->getResponseCode()=="1") )   
	{
		echo "Charge Credit Card AUTH CODE : " . $tresponse->getAuthCode() . "\n";
		echo "Charge Credit Card TRANS ID  : " . $tresponse->getTransId() . "\n";
	}
	else
	{
		echo  "Charge Credit Card ERROR :  Invalid response\n";
	}
}
else
{
	echo  "Charge Credit card Null response returned";
}
```

## Testing

Integration tests for the AuthorizeNet SDK are in the `tests` directory. These tests
are mainly for SDK development. However, you can also browse through them to find
more usage examples for the various APIs.

- Run `composer update --dev` to load the `PHPUnit` test library.
- Copy the `phpunit.xml.dist` file to `phpunit.xml` and enter your merchant
  credentials in the constant fields.
- Run `vendor/bin/phpunit` to run the test suite.

*You'll probably want to disable emails on your sandbox account.*
    
### Test Credit Card Numbers

| Card Type                  | Card Number      |
|----------------------------|------------------|
| American Express Test Card | 370000000000002  |
| Discover Test Card         | 6011000000000012 |
| Visa Test Card             | 4007000000027    |
| Second Visa Test Card      | 4012888818888    |
| JCB                        | 3088000000000017 |
| Diners Club/ Carte Blanche | 38000000000006   |

*Set the expiration date to anytime in the future.*

## PHPDoc

Add PhpDocumentor to your composer.json and run `composer update --dev`:
```json
"require-dev": {
    "phpdocumentor/phpdocumentor": "*"
}
```
To autogenerate PHPDocs run:
```shell
vendor/bin/phpdoc -t doc/api/ -d lib
```
