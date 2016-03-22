# PHPBrake

[![Build Status](https://api.travis-ci.org/airbrake/phpbrake.svg)](https://travis-ci.org/airbrake/phpbrake)
[![Coding Style](https://img.shields.io/badge/phpcs-PSR--2-brightgreen.svg)](http://www.php-fig.org/psr/psr-2/)

![PHPBrake](http://f.cl.ly/items/0e2f2R2I0i081N2w3R0a/php.jpg)

## Installation

```bash
composer require airbrake/phpbrake
```

## Quickstart

```php
// Create new Notifier instance.
$notifier = new Airbrake\Notifier(array(
    'projectId' => 12345, // FIX ME
    'projectKey' => 'abcdefg', // FIX ME
));

// Set global notifier instance.
Airbrake\Instance::set($notifier);

// Register error and exception handlers.
$handler = new Airbrake\ErrorHandler($notifier);
$handler->register();

// Somewhere in the app...
try {
    throw new Exception('hello from phpbrake');
} catch(Exception $e) {
    Airbrake\Instance::notify($e);
}
```

## API

Notifier API constists of 4 methods:
- `buildNotice` - builds [Airbrake notice](https://airbrake.io/docs/#create-notice-v3).
- `sendNotice` - sends notice to Airbrake.
- `notify` - shortcut for `buildNotice` and `sendNotice`.
- `addFilter` - adds filter that can modify and/or filter notices.

## Adding custom data to the notice

```php
$notifier->addFilter(function ($notice) {
    $notice['context']['environment'] = 'production';
    return $notice;
});
```

## Filtering sensitive data from the notice

```php
$notifier->addFilter(function ($notice) {
    if (isset($notice['params']['password'])) {
        $notice['params']['password'] = 'FILTERED';
    }
    return $notice;
});
```

## Ignoring specific exceptions

```php
$notifier->addFilter(function ($notice) {
    if ($notice['errors'][0]['type'] == 'MyExceptionClass') {
        // Ignore this exception.
        return null;
    }
    return $notice;
});
```

## Error handler

Notifier can handle PHP errors, uncatched exceptions and shutdown. You can register appropriate handlers using following code:

```php
$handler = new Airbrake\ErrorHandler($notifier);
$handler->register();
```

Under the hood `$handler->register` does following:

```php
set_error_handler(array($this, 'onError'), error_reporting());
set_exception_handler(array($this, 'onException'));
register_shutdown_function(array($this, 'onShutdown'));
```

## Symfony integration

See https://github.com/aminin/airbrake-bundle

## Monolog integration

```php
$log = new Monolog\Logger('billing');
$log->pushHandler(new Airbrake\MonologHandler($notifier));

$log->addError('charge failed', array('client_id' => 123));
```

## Extra configuration options

### appVersion

The version of your application that you can pass to differentiate exceptions
between multiple versions. It's not set by default.

```php
$notifier = new Airbrake\Notifier([
    // ...
    'appVersion' => '1.2.3',
    // ...
]);
```

### host

By default, it is set to `api.airbrake.io`. A `host` is a web address containing a
scheme ("http" or "https"), a host and a port. You can omit the port (80 will be
assumed) and the scheme ("https" will be assumed).

```php
$notifier = new Airbrake\Notifier([
    // ...
    'host' => 'errbit.example.com', // put your errbit host here
    // ...
]);
```

### rootDirectory

Configures the root directory of your project. Expects a String or a Pathname,
which represents the path to your project. Providing this option helps us to
filter out repetitive data from backtrace frames and link to GitHub files
from our dashboard.

```php
$notifier = new Airbrake\Notifier([
    // ...
    'rootDirectory' => '/var/www/project',
    // ...
]);
```

### environment

Configures the environment the application is running in. Helps the Airbrake
dashboard to distinguish between exceptions occurring in different
environments. By default, it's not set.

```php
$notifier = new Airbrake\Notifier([
    // ...
    'environment' => 'staging',
    // ...
]);
```

## Running tests

```bash
composer install
vendor/bin/phpunit
```

## PHPDoc
```bash
composer require phpdocumentor/phpdocumentor
bin/phpdoc -d src
firefox output/index.html
```

## License

PHPBrake is licensed under [The MIT License (MIT)](LICENSE).
