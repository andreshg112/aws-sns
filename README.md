# Amazon SNS Notifications Channel for Laravel 6

[![Latest Version on Packagist](https://img.shields.io/packagist/v/lab123/aws-sns.svg?style=flat-square)](https://packagist.org/packages/lab123/aws-sns)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)
[![Build Status](https://img.shields.io/travis/lab123/aws-sns/master.svg?style=flat-square)](https://travis-ci.org/lab123/aws-sns)
[![Quality Score](https://img.shields.io/scrutinizer/g/lab123/aws-sns.svg?style=flat-square)](https://scrutinizer-ci.com/g/lab123/aws-sns)
[![Code Coverage](https://img.shields.io/scrutinizer/coverage/g/lab123/aws-sns/master.svg?style=flat-square)](https://scrutinizer-ci.com/g/lab123/aws-sns/?branch=master)
[![Total Downloads](https://img.shields.io/packagist/dt/lab123/aws-sns.svg?style=flat-square)](https://packagist.org/packages/lab123/aws-sns)

This package makes it easy to send notifications using [Amazon SNS](https://aws.amazon.com/pt/sns/) with Laravel 5.3.

## Contents

- [Installation](#installation)
	- [Setting up the AwsSns service](#setting-up-the-awssns-service)
- [Usage](#usage)
	- [Sending SMS](#sending-sms)
	- [Sending Topic](#sending-topic)
- [Changelog](#changelog)
- [Testing](#testing)
- [Security](#security)
- [Contributing](#contributing)
- [Credits](#credits)
- [License](#license)


## Installation

Can to install with commands:

	composer require lab123/aws-sns:dev-master

Or editing the `composer.json` file:

	"require": {
    	"lab123/aws-sns": "dev-master"
    }

You must install the service provider.

**Laravel 5.x**

configure in `config/app.php`:

	'providers' => [
	    ...
	    Lab123\AwsSns\AwsSnsServiceProvider::class,
	]

**Lumen 5.x**

configure in `bootstrap/app.php`:

	$app->register(\Lab123\AwsSns\AwsSnsLumenServiceProvider::class);

** Note: ** You go need make a wrapper [Laravel Notification](https://laracasts.com/discuss/channels/lumen/notification-in-lumen-53 "Laravel Notification") or use a package how
[Lumen Notification](https://packagist.org/packages/lab123/lumen-notification "Lumen Notification")

### Setting up the AwsSns service

Follow the Amazon Console generate the APIKEY and API SECRET, which is connecting both.

Create a new sns section inside `config/services.php`:

	...
	'sns' => [
	    'key' => env('SNS_KEY'),
	    'secret' => env('SNS_SECRET'),
	    'region' => env('SNS_REGION', 'us-east-1')
	],
	...

Next we need to add this keys to our Laravel environment. Edit file `.env` to config the keys:

	...
	SNS_KEY=YOUR_KEY
	SNS_SECRET=YOUR_SECRET
	SNS_REGION=YOUR_REGION
	...

### Default Configurations SMS (OPTIONAL) ###

You also can config defaults attributes for sending SMS running `php artisan vendor:publish --provider="Lab123\AwsSns\AwsSnsServiceProvider"` or creating file `config/aws-sns.php`:

	return [

	    'sms' => [
	        'monthlySpendLimit' => env('SNS_SMS_MONTHLY_LIMIT'),
	        'deliveryStatusIAMRole' => env('SNS_SMS_DELIVERY_STATUS_IAM_ROLE'),
	        'deliveryStatusSuccessSamplingRate' => env('SNS_SMS_DELIVERY_STATUS'),
	        'defaultSenderID' => env('SNS_SMS_SENDER'),
	        'defaultSMSType' => env('SNS_SMS_TYPE'),
	        'usageReportS3Bucket' => env('SNS_SMS_REPORT_S3')
	    ]
	]

And now you can set your default configuration for SMS in `.env`

** Note: ** More information how they work the settings at http://docs.aws.amazon.com/en/sns/latest/api/API_SetSMSAttributes.html

## Usage

### Sending SMS ###

To send sms without the need to create a topic, leave the `function via` as follows:

	// Notifications/Welcome.php
    /**
     * Get the notification channels.
     *
     * @param mixed $notifiable
     * @return array|string
     */
    public function via($notifiable)
    {
        return [
            AwsSnsSmsChannel::class
        ];
    }

Add function `toAwsSnsSms()` expected by class `AwsSnsSmsChannel` to send notification:

	// Notifications/Welcome.php
	/**
     * Get the AWS SNS SMS Message representation of the notification.
     *
     * @param mixed $notifiable
     * @return \Lab123\AwsSns\Messages\AwsSnsMessage
     */
    public function toAwsSnsSms($notifiable)
    {
        return (new AwsSnsMessage())->message('Message Here')->phoneNumber('+5511999999999');
    }

You also can ignore the `->phoneNumber()` in your notification and use function `routeNotificationForAwsSnsSms` in your Model Notifiable:

	// Models/User.php
	/**
     * Route notifications for the Aws SNS SMS channel.
     *
     * @return string
     */
    public function routeNotificationForAwsSnsSms()
    {
        return $this->phone_number;
    }

**Note.: ** The expected number use the standards-based international [E.123](https://en.wikipedia.org/wiki/E.123)

**eg.:** +5511999999999

### Sending Topic ###

To send notification to a topic, leave the `function via` as follows:

	// Notifications/Welcome.php
    /**
     * Get the notification channels.
     *
     * @param mixed $notifiable
     * @return array|string
     */
    public function via($notifiable)
    {
        return [
            AwsSnsTopicChannel::class
        ];
    }

Add function `toAwsSnsTopic()` expected by class `AwsSnsTopicChannel` to send notification:

	// Notifications/Welcome.php
	/**
     * Get the AWS SNS Topic Message representation of the notification.
     *
     * @param mixed $notifiable
     * @return \Lab123\AwsSns\Messages\AwsSnsMessage
     */
    public function toAwsSnsTopic($notifiable)
    {
        return (new AwsSnsMessage())->message('Message Here')->topicArn('arn:aws:sns:us-east-1:000000000000:name-topic');
    }

You also can ignore the `->topicArn()` in your notification and use function `routeNotificationForAwsSnsTopic` in your Model Notifiable:

	// Models/User.php
	/**
     * Route notifications for the Aws SNS Topic channel.
     *
     * @return string
     */
    public function routeNotificationForAwsSnsTopic()
    {
        return $this->topicArn;
    }

### Available methods


*   `topicArn($topicArn)`: Your Topic Arn from Amazon SNS;
*   `phoneNumber($phoneNumber)`: Phone number to send notification;
*   `message($message)`: Message to be sent;

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Testing

``` bash
$ composer test
```

## Security

If you discover any security related issues, please email jean.pierre@lab123.com.br instead of using the issue tracker.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Credits

- [Jean Pierre](https://github.com/jeanpfs)
- [Felipe Santos](https://github.com/felipeds2)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
